# ESPHome packages

This repo holds the source of various packages I'm using.

## Voice assistant

- ESP32-S3-DevKitC-1-N16R8V
- INMP441
- MAX98357A

Create a new device in the ESPHome dashboard and add the following as its config:

```yaml
substitutions:
  name: voice-assist
  friendly_name: Voice Assist

  # INMP441
  mic_ws_pin: GPIO5
  mic_sck_pin: GPIO6
  mic_sd_pin: GPIO4

  # MAX98357A
  spk_lrc_pin: GPIO12
  spk_bclk_pin: GPIO13
  spk_din_pin: GPIO14

  # Touch pins
  action_pin: GPIO11
  action_pin_threshold: "160000"
  vol_down_pin: GPIO09
  vol_down_pin_threshold: "150000"
  vol_up_pin: GPIO10
  vol_up_pin_threshold: "150000"

packages:
  voice-assistant:
    url: https://github.com/tronikos/esphome-packages
    ref: main
    files: [esp32-s3-voice-assistant.yaml]
    refresh: 0s

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
```

To [tweak the assist audio configuration for your device](https://www.home-assistant.io/voice_control/troubleshooting#to-tweak-the-assist-audio-configuration-for-your-device), add and adjust:

```yaml
voice_assistant:
  noise_suppression_level: 2
  auto_gain: 31dBFS
  volume_multiplier: 2.0
```

## BME680

```yaml
packages:
  bme680:
    url: https://github.com/tronikos/esphome-packages
    ref: main
    file: bme68x_bsec2.yaml
    # file: bme680_bsec.yaml
    # file: bme680.yaml
```
