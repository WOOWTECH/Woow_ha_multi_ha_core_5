# 優化 HA Core 2-5 提示詞（基於 HA Core 1 已完成的優化）

## 背景

HA Core 1 已完成以下優化並通過 11 項測試。現在需要對 HA Core 2-5 執行相同的優化。

HAOS 主機：`192.168.2.12`
SSH：`root@192.168.2.12`，密碼：`woowtech`
主 HAOS 帳號：`admin` / `admin`

## 各實例對照表

| 項目 | HA1 (已完成) | HA2 | HA3 | HA4 | HA5 |
|------|-------------|-----|-----|-----|-----|
| GitHub Repo | `WOOWTECH/Woow_ha_multi_ha_core_1` | `WOOWTECH/Woow_ha_multi_ha_core_2` | `WOOWTECH/Woow_ha_multi_ha_core_3` | `WOOWTECH/Woow_ha_multi_ha_core_4` | `WOOWTECH/Woow_ha_multi_ha_core_5` |
| 舊目錄名 | ~~hassception~~ → `woow_ha_core_1` | `hassception2` → `woow_ha_core_2` | `hassception3` → `woow_ha_core_3` | `hassception4` → `woow_ha_core_4` | `hassception5` → `woow_ha_core_5` |
| 舊 slug | ~~hassception~~ → `woow_ha_core_1` | `hassception2` → `woow_ha_core_2` | `hassception3` → `woow_ha_core_3` | `hassception4` → `woow_ha_core_4` | `hassception5` → `woow_ha_core_5` |
| 舊 add-on slug (HAOS) | ~~10409bfc_hassception~~ | `3a60229b_hassception2` | `9ddb52a6_hassception3` | `3f07a9d0_hassception4` | `e72b4c43_hassception5` |
| 外部 Port | 8124 | 8125 | 8126 | 8127 | 8128 |
| Config 目錄 | `/config/woow_ha_core_1/` | `/config/woow_ha_core_2/` | `/config/woow_ha_core_3/` | `/config/woow_ha_core_4/` | `/config/woow_ha_core_5/` |
| 舊 Config 目錄 | — | `/config/hassception2/` | `/config/hassception3/` | `/config/hassception4/` | `/config/hassception5/` |
| 目前版本 | 1.0.1 | 1.0.0 | 1.0.0 | 1.0.0 | 1.0.0 |

## 需要執行的優化任務（每個 HA 實例都要做）

### 任務 1：Rebrand — 將 hassception 改為 woow_ha_core_N

在 GitHub Repo 中：

1. **重命名目錄**：`hassception{N}/` → `woow_ha_core_{N}/`
2. **修改 `woow_ha_core_{N}/config.yaml`**：
   - `slug` 從 `hassception{N}` 改為 `woow_ha_core_{N}`
   - 確認 `name`、`description`、`version`、`ports` 正確
3. **修改 `woow_ha_core_{N}/Dockerfile`**：
   - config 路徑從 `/config/hassception{N}` 改為 `/config/woow_ha_core_{N}`
4. **推送到 GitHub**

完成後 config.yaml 應該長這樣（以 HA2 為例）：
```yaml
name: "Woow HA Core 2"
description: "Woow Home Assistant Core instance 2"
version: "1.0.1"
slug: "woow_ha_core_2"
init: false
arch:
  - aarch64
  - amd64
  - armhf
  - armv7
  - i386
ports:
  8123/tcp: 8125
map:
  - "config:rw"
```

完成後 Dockerfile 應該長這樣（以 HA2 為例）：
```dockerfile
ARG BUILD_FROM
FROM ${BUILD_FROM}

CMD mkdir -p /config/woow_ha_core_2 && python -m homeassistant --config /config/woow_ha_core_2
```

### 任務 2：HAOS 上移除舊 add-on、重新安裝新 add-on

```bash
# SSH 進入 HAOS
ssh root@192.168.2.12  # 密碼: woowtech

# 以 HA2 為例（其他以此類推）：

# 1. 停止並移除舊 add-on
ha addons stop 3a60229b_hassception2
ha addons uninstall 3a60229b_hassception2

# 2. 移除舊的 repo（找到 slug ID）
ha store remove 3a60229b   # 只取前 8 碼

# 3. 重新整理 store
ha store reload

# 4. 重新加入 repo
ha store add https://github.com/WOOWTECH/Woow_ha_multi_ha_core_2

# 5. 安裝新的 add-on（新 slug 由 HAOS 自動生成，格式為 {hash}_woow_ha_core_2）
ha store reload
# 查看可用的 add-on
ha addons list --raw-json | grep woow_ha_core_2
# 安裝
ha addons install {新slug}_woow_ha_core_2
ha addons start {新slug}_woow_ha_core_2
```

**注意**：`ha addons` 在新版 HAOS 中可能顯示 `The use of 'addons' is deprecated, please use 'apps' instead!`，兩者都可以用。

### 任務 3：Onboarding（首次設定）

每個 HA Core 實例首次啟動需要完成 onboarding。可以透過 API 自動化：

```bash
# 以 HA2 (port 8125) 為例
PORT=8125
ADMIN_USER="admin-core2"    # HA2 用 admin-core2，HA3 用 admin-core3，以此類推
PASSWORD="woowtech"

# Step 1: 建立使用者
curl -X POST http://192.168.2.12:$PORT/api/onboarding/users \
  -H "Content-Type: application/json" \
  -d "{\"client_id\":\"http://192.168.2.12:$PORT/\",\"name\":\"Admin\",\"username\":\"$ADMIN_USER\",\"password\":\"$PASSWORD\",\"language\":\"en\"}"
# 回傳 auth_code，用它取得 token

# Step 2: 取得 token
AUTH_CODE="<上一步回傳的 auth_code>"
curl -X POST http://192.168.2.12:$PORT/auth/token \
  -d "grant_type=authorization_code&code=$AUTH_CODE&client_id=http://192.168.2.12:$PORT/"
# 取得 access_token

# Step 3: 完成 onboarding 其餘步驟
TOKEN="<上一步的 access_token>"
curl -X POST http://192.168.2.12:$PORT/api/onboarding/core_config \
  -H "Authorization: Bearer $TOKEN"
curl -X POST http://192.168.2.12:$PORT/api/onboarding/analytics \
  -H "Authorization: Bearer $TOKEN"
curl -X POST http://192.168.2.12:$PORT/api/onboarding/integration \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"client_id":"http://192.168.2.12:'$PORT'/","redirect_uri":"http://192.168.2.12:'$PORT'/?auth_callback=1"}'
```

### 任務 4：安裝 Config Editor 自訂元件（關鍵！含 HA1 發現的 bug fix）

**重要**：HA1 測試時發現 `__init__.py` 被部署為空檔（0 bytes），導致 Config Editor 的 WebSocket 後端（`config_editor/ws`）從未註冊，所有檔案操作（list/load/save）靜默失敗。務必從 GitHub 下載正確的完整檔案。

```bash
# SSH 進入 HAOS
ssh root@192.168.2.12  # 密碼: woowtech

# 以 HA2 為例，CONFIG_DIR=/config/woow_ha_core_2
CONFIG_DIR=/config/woow_ha_core_2

# 1. 建立目錄
mkdir -p $CONFIG_DIR/custom_components/config_editor
mkdir -p $CONFIG_DIR/www

# 2. 下載 config_editor 後端（3 個檔案）— 注意 __init__.py 必須是完整的 4319 bytes！
curl -sL "https://raw.githubusercontent.com/junkfix/config-editor/main/custom_components/config_editor/__init__.py" \
  > $CONFIG_DIR/custom_components/config_editor/__init__.py
curl -sL "https://raw.githubusercontent.com/junkfix/config-editor/main/custom_components/config_editor/config_flow.py" \
  > $CONFIG_DIR/custom_components/config_editor/config_flow.py
curl -sL "https://raw.githubusercontent.com/junkfix/config-editor/main/custom_components/config_editor/manifest.json" \
  > $CONFIG_DIR/custom_components/config_editor/manifest.json

# 3. 驗證 __init__.py 不是空檔！
wc -c $CONFIG_DIR/custom_components/config_editor/__init__.py
# 應該顯示 4319 bytes，如果是 0 就表示下載失敗，需要重新下載

# 4. 下載 config-editor-card.js 前端
curl -sL "https://raw.githubusercontent.com/junkfix/config-editor-card/main/config-editor-card.js" \
  > $CONFIG_DIR/www/config-editor-card.js

# 5. 在 configuration.yaml 加入 config_editor
echo "" >> $CONFIG_DIR/configuration.yaml
echo "config_editor:" >> $CONFIG_DIR/configuration.yaml

# 6. 建立 .storage 檔案

# 6a. Lovelace 資源
cat > $CONFIG_DIR/.storage/lovelace_resources << 'EOF'
{
  "version": 1,
  "minor_version": 1,
  "key": "lovelace_resources",
  "data": {
    "items": [
      {
        "id": "config_editor_card",
        "type": "module",
        "url": "/local/config-editor-card.js?v=1"
      }
    ]
  }
}
EOF

# 6b. Lovelace dashboards
cat > $CONFIG_DIR/.storage/lovelace_dashboards << 'EOF'
{
  "version": 1,
  "minor_version": 1,
  "key": "lovelace_dashboards",
  "data": {
    "items": [
      {
        "id": "config_editor",
        "url_path": "config-editor",
        "title": "Config Editor",
        "icon": "mdi:file-edit-outline",
        "show_in_sidebar": true,
        "require_admin": true,
        "mode": "storage"
      }
    ]
  }
}
EOF

# 6c. Dashboard 設定
cat > $CONFIG_DIR/.storage/lovelace.config_editor << 'EOF'
{
  "version": 1,
  "minor_version": 1,
  "key": "lovelace.config_editor",
  "data": {
    "config": {
      "views": [
        {
          "title": "Config Editor",
          "panel": true,
          "cards": [
            {
              "type": "custom:config-editor-card",
              "depth": 3
            }
          ]
        }
      ]
    }
  }
}
EOF
```

### 任務 5：重啟 add-on 並註冊 Config Editor Integration

```bash
# 重啟 add-on 讓新的 custom_components 生效
ha addons restart {slug}_woow_ha_core_{N}

# 等待 30 秒讓 HA Core 啟動完成
sleep 30

# 取得 token（用 refresh_token 或重新登入）
TOKEN=$(curl -s -X POST http://192.168.2.12:$PORT/auth/token \
  -d "grant_type=refresh_token&client_id=http://192.168.2.12:$PORT/&refresh_token=$REFRESH_TOKEN" \
  | python3 -c "import sys,json;print(json.load(sys.stdin)['access_token'])")

# 手動註冊 config_editor integration（如果自動發現沒觸發）
curl -X POST http://192.168.2.12:$PORT/api/config/config_entries/flow \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"handler":"config_editor","show_advanced_options":false}'

# 取得 flow_id 後完成
FLOW_ID="<回傳的 flow_id>"
curl -X POST http://192.168.2.12:$PORT/api/config/config_entries/flow/$FLOW_ID \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{}'
```

### 任務 6：驗證

每個 HA Core 實例完成後驗證以下項目：

```bash
PORT=8125  # HA2=8125, HA3=8126, HA4=8127, HA5=8128

# 1. 基本連線
curl -s -o /dev/null -w "%{http_code}" http://192.168.2.12:$PORT/api/
# 預期: 401

# 2. API 認證
TOKEN=$(curl -s -X POST http://192.168.2.12:$PORT/auth/token \
  -d "grant_type=refresh_token&client_id=http://192.168.2.12:$PORT/&refresh_token=$REFRESH_TOKEN" \
  | python3 -c "import sys,json;print(json.load(sys.stdin)['access_token'])")
curl -s http://192.168.2.12:$PORT/api/ -H "Authorization: Bearer $TOKEN"
# 預期: {"message": "API running."}

# 3. Config Editor WebSocket 測試（用 python3 + websockets）
python3 << PYEOF
import asyncio, json, websockets
async def test():
    async with websockets.connect("ws://192.168.2.12:$PORT/api/websocket") as ws:
        await ws.recv()
        await ws.send(json.dumps({"type":"auth","access_token":"$TOKEN"}))
        r = json.loads(await ws.recv())
        print(f"Auth: {r['type']}")
        await ws.send(json.dumps({"id":1,"type":"config_editor/ws","action":"list","file":"","data":"","ext":"yaml","depth":3}))
        r = json.loads(await ws.recv())
        print(f"List files: success={r.get('success')}, count={len(r.get('result',{}).get('file',[]))}")
asyncio.run(test())
PYEOF
# 預期: Auth: auth_ok, List files: success=True, count>0

# 4. 檢查 add-on 狀態
ha addons info {slug}_woow_ha_core_{N} --raw-json | python3 -c "
import sys,json
d=json.load(sys.stdin)['data']
print(f'Name: {d[\"name\"]}')
print(f'Version: {d[\"version\"]}')
print(f'State: {d[\"state\"]}')
print(f'Icon: {d[\"icon\"]}')
print(f'Logo: {d[\"logo\"]}')
"
# 預期: Name=Woow HA Core N, Version=1.0.1, State=started, Icon=True, Logo=True
```

### 任務 7：版本更新測試（選做）

在 GitHub 上將 config.yaml 的 version 從 `1.0.1` 改為 `1.0.2`，push 後：

```bash
ha store reload
sleep 10
ha addons info {slug}_woow_ha_core_{N} --raw-json | grep update_available
# 預期: "update_available": true
```

## 各實例帳號規劃

| 實例 | Port | 使用者名稱 | 密碼 |
|------|------|-----------|------|
| HA Core 1 | 8124 | admin-core1 | woowtech |
| HA Core 2 | 8125 | admin-core2 | woowtech |
| HA Core 3 | 8126 | admin-core3 | woowtech |
| HA Core 4 | 8127 | admin-core4 | woowtech |
| HA Core 5 | 8128 | admin-core5 | woowtech |

## 重要注意事項

1. **__init__.py 不能是空檔**：這是 HA1 測試中發現的最關鍵 bug。下載後一定要用 `wc -c` 確認是 4319 bytes。
2. **slug 變更後需要完全重裝**：不能只改檔案，需要 uninstall 舊的 → remove repo → reload → add repo → install 新的。
3. **config 目錄遷移**：rebrand 後 config 目錄路徑會從 `/config/hassception{N}/` 變為 `/config/woow_ha_core_{N}/`。如果舊實例有重要資料，需要先備份 `/config/hassception{N}/` 目錄。
4. **`ha store remove` 要用 slug ID**（前 8 碼），不是 URL。
5. **Onboarding 只能做一次**：如果已經 onboard 過，重裝後如果 config 目錄保留了舊資料就不需要重新 onboard。如果是全新安裝才需要。
6. **repository.json 已經正確**：HA2-5 的 `repository.json` 已經是標準格式（name: "WOOWTECH HA Add-Ons", maintainer: "WOOWTECH"），不需要修改。
7. **icon.png 和 logo.png 已存在**：HA2-5 已有 icon/logo 檔案（14365 bytes），不需要重新添加，只需要確保 rebrand 時隨目錄一起搬移。
