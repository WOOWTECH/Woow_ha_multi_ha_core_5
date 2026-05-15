# Woow HA Core 5

Home Assistant Core instance running inside HAOS as an add-on. Based on [jgoakley/hassio-addons](https://github.com/jgoakley/hassio-addons).

## Installation

1. In HAOS: **Settings > Add-ons > Add-on Store > Repositories**
2. Add: `https://github.com/WOOWTECH/Woow_ha_multi_ha_core_5`
3. Install **Woow HA Core 5** from the store
4. Start the add-on
5. Access at `http://<HAOS_IP>:8128`

## Details

| Item | Value |
|------|-------|
| Slug | `woow_ha_core_5` |
| Internal port | 8123 |
| External port | 8128 |
| Config directory | `/config/woow_ha_core_5/` |

## Post-Install

After starting the add-on, complete the HA onboarding wizard at port 8128. The config files will be stored under `/config/woow_ha_core_5/` on the host HAOS.
