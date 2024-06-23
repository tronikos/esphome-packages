# ESPHome packages

This repo holds the source of various packages I'm using.

## Voice assistant

- ESP32-S3-DevKitC-1-N16R8V
- INMP441
- MAX98357A

Create a new device in the ESPHome dashboard and add the following
at the end of its config:

```yaml
substitutions:
  micro_wake_word_model: okay_nabu
  # micro_wake_word_model: hey_jarvis
  # micro_wake_word_model: alexa

  # https://www.home-assistant.io/voice_control/troubleshooting/#to-tweak-the-assist-audio-configuration-for-your-device
  voice_assistant_noise_suppression_level: "2"
  voice_assistant_auto_gain: 31dBFS
  voice_assistant_volume_multiplier: "2.0"

  # INMP441
  mic_ws_pin: GPIO5
  mic_sck_pin: GPIO6
  mic_sd_pin: GPIO4

  # MAX98357A
  spk_lrc_pin: GPIO12
  spk_bclk_pin: GPIO13
  spk_din_pin: GPIO14

packages:
  voice-assistant:
    url: https://github.com/tronikos/esphome-packages
    ref: main
    files: [esp32-s3-voice-assistant.yaml]
    # Or to use the device as a media player:
    # files: [esp32-s3-voice-assistant.yaml, esp32-s3-media-player.yaml]
    refresh: 0s
```

To use the beta model for "Hey Mycroft" add:

```yaml
external_components:
  - source: github://pr#6653
    components: [micro_wake_word]
    refresh: 0s

micro_wake_word:
  model: !remove
  models:
    - model: https://github.com/kahrendt/microWakeWord/releases/download/hey_mycroft/hey_mycroft.json
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
