# ESPHome packages

This repo holds the source of various packages I'm using.

## Voice assistant

- ESP32-S3-DevKitC-1-N16R8V
- INMP441
- MAX98357A

Create a new device in the ESPHome dashboard and add the following as its config:

```yaml
substitutions:
substitutions:
  name: voice-assist
  friendly_name: Voice Assist
  # alexa, hey_jarvis, hey_mycroft, okay_nabu
  micro_wake_word_model: hey_mycroft

  # INMP441
  mic_ws_pin: GPIO5
  mic_sck_pin: GPIO6
  mic_sd_pin: GPIO4

  # MAX98357A
  spk_lrc_pin: GPIO12
  spk_bclk_pin: GPIO13
  spk_din_pin: GPIO14

  mute_pin: GPIO11
  mute_pin_threshold: "160000"

packages:
  voice-assistant:
    url: https://github.com/tronikos/esphome-packages
    ref: main
    files: [esp32-s3-voice-assistant.yaml]
    # Or to use the device as a media player:
    # files: [esp32-s3-voice-assistant-media-player.yaml]
    refresh: 0s

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
```

To use additional Micro Wake Word models add:

```yaml
micro_wake_word:
  models:
    - model: hey_jarvis
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
i2c:
  sda: GPIO7
  scl: GPIO15

packages:
  bme680:
    url: https://github.com/tronikos/esphome-packages
    ref: main
    # file: bme680.yaml
    file: bme680_bsec.yaml
    # file: bme68x_bsec.yaml
```
