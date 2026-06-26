ARG BUILD_FROM
FROM ${BUILD_FROM}

# Overlay the exact woowtech rebrand (hass_frontend + homeassistant trees,
# extracted from woowtech-ha:2026.3.4) onto the pinned stock HA Core image.
# Served box-locally over the hassio bridge during the Supervisor build.
ADD http://172.30.32.1:8099/woowtech-overlay.tgz /tmp/woowtech-overlay.tgz
RUN rm -rf /usr/local/lib/python3.14/site-packages/hass_frontend /usr/src/homeassistant/homeassistant && tar xzf /tmp/woowtech-overlay.tgz -C / && rm /tmp/woowtech-overlay.tgz

CMD mkdir -p /config/woow_ha_core_5 && python -m homeassistant --config /config/woow_ha_core_5
