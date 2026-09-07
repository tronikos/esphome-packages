# ESPHome packages

This repo holds the source of various packages I'm using.

Every package takes a `..._prefix` substitution that is prepended to each entity
name. It defaults to empty, so a single package works out of the box, but when
you combine packages you have to set at least one prefix: `bme68x_bsec2` and
`scd4x` both publish a `Temperature` and a `Humidity`, and ESPHome rejects
duplicate entity names within a platform. See the `## Combining packages`
example at the end.

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

Needs an `i2c:` bus in your own config.

```yaml
substitutions:
  bme680_prefix: ''
  bme680_temperature_offset: 2.5°C  # bme68x_bsec2.yaml only

packages:
  bme680:
    url: https://github.com/tronikos/esphome-packages
    ref: main
    file: bme68x_bsec2.yaml
    # file: bme680_bsec.yaml
    # file: bme680.yaml
```

Three variants, in order of preference:

- `bme68x_bsec2.yaml` -- BSEC2, real 0-500 IAQ plus CO2 and VOC estimates. Use
  this one.
- `bme680_bsec.yaml` -- the older BSEC v1 component. **Arduino framework only**;
  it will not compile on `esp-idf`.
- `bme680.yaml` -- no BSEC. IAQ is a heuristic from gas resistance and humidity
  mapped onto the 0-500 scale, not a calibrated measurement.

Each also publishes internal, unfiltered `*_raw` sensors
(`bme680_temperature_raw`, `bme680_pressure_raw`, `bme680_humidity_raw`,
`bme680_iaq_raw`) for use in displays and in the SCD4x pressure compensation
below, where the smoothing filters would only add lag.

## SCD40/SCD41

Needs an `i2c:` bus in your own config.

```yaml
substitutions:
  scd4x_prefix: ''
  scd4x_temperature_offset: 3°C
  scd4x_calibration_ppm: '426'

packages:
  scd4x:
    url: https://github.com/tronikos/esphome-packages
    ref: main
    file: scd4x.yaml
```

Runs in `low_power_periodic` mode, one measurement every 30s, which self-heats
less than the default 5s `periodic` mode. Exposes a `calibrate_co2_value` API
action, a **Calibrate CO2** button that forces calibration to
`scd4x_calibration_ppm`, and a hidden **Factory Reset CO2** button. Forced
calibration is only meaningful with the sensor outdoors in fresh air.

## LD2410

Needs a `uart:` in your own config. The LD2410 defaults to 256000 baud.

```yaml
substitutions:
  ld2410_prefix: ''
  # ld2410-zones.yaml only: zone boundaries in cm
  ld2410_z1_end: '10'
  ld2410_z2_end: '36'
  ld2410_z3_end: '100'

uart:
  tx_pin: GPIO17
  rx_pin: GPIO16
  baud_rate: 256000

packages:
  ld2410:
    url: https://github.com/tronikos/esphome-packages
    ref: main
    files: [ld2410.yaml, ld2410-zones.yaml]
    # files: [ld2410.yaml]
```

`ld2410.yaml` exposes the full component: presence, per-gate energies, gate
thresholds, and a `set_ld2410_bluetooth_password` API action. Almost everything
is `disabled_by_default` so the device does not flood Home Assistant with 60+
entities.

`ld2410-zones.yaml` adds three distance-based occupancy zones on top, with the
boundaries exposed as numbers. It requires `ld2410.yaml`.

## CC1101 fan controller

Bridges a 300-350MHz RF ceiling-fan remote through a CC1101, so the fan can be
driven from Home Assistant while the physical remote stays in sync. ESP32 only:
it uses the RMT peripheral (`rmt_symbols`, `buffer_size`).

```yaml
substitutions:
  rc_switch_protocol: "6"
  cc1101_frequency_rx: "303.87"
  cc1101_frequency_tx: "307.0"

  pin_spi_clk: GPIO18
  pin_spi_mosi: GPIO23
  pin_spi_miso: GPIO19
  pin_cc1101_cs: GPIO5
  # CC1101 GDO0 / GDO2
  pin_remote_rx: GPIO27
  pin_remote_tx: GPIO26

  # 12-bit codes, as decimals
  code_fan_off: "125"
  code_fan_low: "119"
  code_fan_med: "111"
  code_fan_high: "95"
  code_light_hold: "126"
  code_rev_hold: "123"
  code_common_release: "127"

packages:
  cc1101_fan:
    url: https://github.com/tronikos/esphome-packages
    ref: main
    file: cc1101_fan_controller.yaml
```

To find the codes for a different remote, set `logger: level: DEBUG` and press
its buttons; every received frame is logged with its protocol and code before
the protocol filter is applied.

RX and TX frequencies, and the number of times each burst is repeated, are
exposed as config numbers so they can be tuned without reflashing. The 50-repeat
default is generous; it can usually be lowered.

## Combining packages

```yaml
substitutions:
  # bme68x_bsec2 and scd4x would otherwise both publish Temperature/Humidity.
  scd4x_prefix: 'SCD41 '
  ld2410_prefix: 'Radar '

packages:
  tronikos-sensors:
    url: https://github.com/tronikos/esphome-packages
    ref: main
    files:
      - bme68x_bsec2.yaml
      - scd4x.yaml
      - ld2410.yaml
      - ld2410-zones.yaml
    refresh: 0s

sensor:
  # Compensate the CO2 reading with the unfiltered pressure from the BME680.
  - platform: scd4x
    id: !extend scd4x_sensor
    ambient_pressure_compensation_source: bme680_pressure_raw
```

## Dev notes

```sh
python3 -m venv .venv
source .venv/bin/activate
pip install esphome pre-commit
pre-commit install

# Validate the same configs CI does
esphome config tests/test-*.yaml
```
