ARG BUILD_FROM
FROM ${BUILD_FROM}

# Download and apply WOOWTECH UI overlay
ARG OVERLAY_VERSION=v1.0.0
RUN apk add --no-cache curl \
 && curl -fsSL \
      "https://github.com/WOOWTECH/Woow_ha_multi_ha_core_1/releases/download/${OVERLAY_VERSION}-overlay/woowtech-overlay.tar.gz" \
      -o /tmp/overlay.tar.gz \
 && rm -rf \
      /usr/src/homeassistant/homeassistant/components/analytics \
      /usr/src/homeassistant/homeassistant/components/homeassistant_alerts \
      /usr/src/homeassistant/homeassistant/components/intent_script \
      /usr/src/homeassistant/homeassistant/components/my \
      /usr/src/homeassistant/homeassistant/components/usage_prediction \
      /usr/src/homeassistant/homeassistant/components/cloud \
      /usr/src/homeassistant/homeassistant/components/conversation \
      /usr/src/homeassistant/homeassistant/components/intent \
      /usr/src/homeassistant/homeassistant/components/tts \
      /usr/src/homeassistant/homeassistant/components/stt \
      /usr/src/homeassistant/homeassistant/components/assist_satellite \
      /usr/src/homeassistant/homeassistant/components/wake_word \
      /usr/src/homeassistant/homeassistant/components/assist_pipeline \
 && rm -rf \
      /usr/local/lib/python3.14/site-packages/hass_frontend/frontend_latest \
      /usr/local/lib/python3.14/site-packages/hass_frontend/frontend_es5 \
      /usr/local/lib/python3.14/site-packages/hass_frontend/static \
      /usr/local/lib/python3.14/site-packages/hass_frontend/index.html \
      /usr/local/lib/python3.14/site-packages/hass_frontend/authorize.html \
      /usr/local/lib/python3.14/site-packages/hass_frontend/onboarding.html \
 && tar xzf /tmp/overlay.tar.gz -C / \
 && rm -f /tmp/overlay.tar.gz \
 && apk del curl

CMD mkdir -p /config/woow_ha_core_5 && python -m homeassistant --config /config/woow_ha_core_5
