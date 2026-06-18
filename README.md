# Woow HA Core 5 — WOOWTECH UI Edition

Home Assistant Core instance running inside HAOS as an add-on, with the **WOOWTECH branded frontend** (compile-time custom UI).

## What's Different

This branch (`woowtech-ui`) extends the standard add-on with:

- Custom compiled HA frontend with WOOWTECH branding
- Modified backend constants (`APPLICATION_NAME = "woowtech"`)
- Stub components replacing cloud-dependent integrations
- Custom icons, colors (#6183fc), and translations
- OHF badge replaced with WOOWTECH branding

## Installation

1. In HAOS: **Settings > Add-ons > Add-on Store > Repositories**
2. Add: `https://github.com/WOOWTECH/Woow_ha_multi_ha_core_5`
3. Install **WOOWTECH HA Core 5** from the store
4. Start the add-on
5. Access at `http://<HAOS_IP>:8128`

## Details

| Item | Value |
|------|-------|
| Slug | `woow_ha_core_5` |
| Internal port | 8123 |
| External port | 8128 |
| Config directory | `/config/woow_ha_core_5/` |
| HA Version | 2026.3.4 |
| Overlay Version | v1.0.0 |

## Architecture

The WOOWTECH UI overlay (84 MB compressed) is downloaded during Docker build from a GitHub Release asset and applied on top of the official HA base image. All modifications are architecture-independent (Python/JS/HTML/CSS), so the same overlay works on both amd64 and aarch64.

The overlay is verified with SHA256 checksum to ensure integrity.
