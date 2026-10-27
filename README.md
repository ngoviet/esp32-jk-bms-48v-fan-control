# ESP32 JK-BMS 48V Monitor with Automatic Fan Control

<!-- readme-axi type=esp32 -->

**English** · [Tiếng Việt](README.vi.md)

An **ESPHome** configuration that monitors a **JK-BMS** battery management system (16S LiFePO4)
over Modbus and drives a cooling fan automatically from a temperature reported by **Home
Assistant**. Runs on an **ESP32 Dev Module** or an **ESP32-C3 SuperMini**.

> 🇻🇳 Built and maintained by one person in Việt Nam — if it saves you time, please
> [support the project](#support-the-project).

## Contents

- [Features](#features)
- [Hardware](#hardware)
- [Quick start](#quick-start)
- [Configuration](#configuration)
- [How it works](#how-it-works)
- [Home Assistant](#home-assistant)
- [Troubleshooting](#troubleshooting)
- [Project layout](#project-layout)
- [Development](#development)

## Features

- **JK-BMS monitoring**: full monitoring of a 16S LiFePO4 battery system over Modbus
- **200+ sensors**: cell voltages, temperatures, current, power, capacity, cycles...
- **Automatic fan control**: temperature-based fan control with hysteresis logic
- **Home Assistant integration**: direct connection through the ESPHome API
- **Multi-board support**: configurations for ESP32 and ESP32-C3 SuperMini
- **Secure configuration**: WiFi credentials are kept in a separate file

## Hardware

### ESP32 configuration

- **Board**: ESP32 Dev Module
- **UART pins**: GPIO16 (TX), GPIO17 (RX)
- **Fan pin**: GPIO25 (PWM)
- **Framework**: ESP-IDF

### ESP32-C3 SuperMini configuration

- **Board**: ESP32-C3-DevKitM-1
- **UART pins**: GPIO21 (TX), GPIO20 (RX)
- **Fan pin**: GPIO10
- **Framework**: ESP-IDF

### JK-BMS connection

- **Protocol**: Modbus RTU over UART
- **Speed**: 115200 baud
- **Device address**: 0x09
- **Update interval**: 5-10 seconds

## Quick start

### 1. Get the repository

```bash
git clone https://github.com/ngoviet/esp32-jk-bms-48v-fan-control.git
cd esp32-jk-bms-48v-fan-control
```

### 2. Set up secrets

Copy and edit the secrets file:

```bash
cp secrets.yaml.example secrets.yaml
# Edit secrets.yaml with your WiFi details
```

### 3. Flash the firmware

```bash
# For ESP32-C3
esphome run esp32-c3-bms-monitor.yaml

# Or use the batch files
flash-c3.bat  # Flash over USB
log.bat       # View logs
```

## Configuration

### Fan settings

```yaml
substitutions:
  fan_pin: GPIO10                    # Fan control pin
  FAN_THR_HI_DEFAULT: "42.0"        # Auto ON threshold
  FAN_THR_LO_DEFAULT: "41.0"        # Auto OFF threshold
```

### BMS settings

```yaml
substitutions:
  device_addr: "9"                   # BMS Modbus address
  tx_pin: GPIO21                     # UART TX pin
  rx_pin: GPIO20                     # UART RX pin
  modbus_controller_update_interval: "10s"
```

### WiFi setup (secrets.yaml)

```yaml
wifi_ssid: "Ten_WiFi_Cua_Ban"
wifi_password: "Mat_Khau_WiFi_Cua_Ban"
wifi_bssid: "aa:bb:cc:dd:ee:ff"    # Optional: specific AP
wifi_channel: 11                    # Optional: specific channel
```

### Security

- **Secrets management**: WiFi credentials are stored separately
- **Git protection**: `.gitignore` keeps sensitive information from leaking
- **OTA protection**: optional password protection for updates
- **API encryption**: optional encryption of API communication

## How it works

The fan system controls cooling automatically from a temperature provided by Home Assistant:

### Automatic mode (default)

- **Fan on**: when temperature ≥ T_HIGH (default: 42°C)
- **Fan off**: when temperature < T_LOW (default: 41°C)
- **Hysteresis**: prevents rapid on/off cycling

### Manual mode

- Overrides the automatic control
- Direct on/off control

### Temperature source

- **Primary**: `sensor.device_h240909079_device_temperature` from Home Assistant
- **Fallback**: none (the fan turns off when there is no data, for safety)

## Home Assistant

### Entities created

- **200+ sensors**: every BMS parameter
- **Fan control**: auto/manual mode, temperature thresholds
- **Binary sensors**: charging, discharging, balancing state
- **Diagnostics**: WiFi signal, IP address, device information

### Available sensors

#### Battery monitoring

- Cell voltages (1-16): voltage of each individual cell
- Pack voltage: total voltage of the battery pack
- Current: directional charge/discharge current
- Power: realtime consumption/generation
- SOC: state of charge, in percent
- SOH: state of health, in percent

#### Temperature monitoring

- MOSFET temperature: the power switching components
- Battery temperature: several temperature probes
- Ambient temperature: environment monitoring

#### Protection & status

- Over-voltage / under-voltage protection settings
- Over-current protection settings
- Temperature protection limits
- Balancing state and balancing current
- Charge/discharge state
- Alarm conditions and error codes

### Dashboard cards

```yaml
# Example fan control card
type: entities
entities:
  - entity: switch.fan_auto_mode
  - entity: switch.fan_manual_control
  - entity: number.fan_t_high_c
  - entity: number.fan_t_low_c
  - entity: sensor.temperature_sensor_1
```

## Troubleshooting

### Common issues

**CRC check error**

```
[W][modbus:135]: Modbus CRC Check failed! 6C0!=603
```

- Occasional errors from electrical noise are normal
- No effect on functionality as long as data still arrives
- Consider adding `send_wait_time: 50ms` to reduce how often it happens

**No temperature data**

```
[W][fan]: No temperature data available, turning fan OFF
```

- Check the Home Assistant entity: `sensor.device_h240909079_device_temperature`
- Verify that ESPHome can connect to Home Assistant
- The fan stays safely OFF until data is available

**Compilation errors**

- Make sure every `id:` reference matches across files
- Check the YAML indentation (spaces, not tabs)
- Verify the pin assignments for your specific board

### Debug commands

```bash
# Validate the configuration
esphome config esp32-c3-bms-monitor.yaml

# View logs
esphome logs esp32-c3-bms-monitor.yaml --device 192.168.x.x

# Clear the build cache
esphome clean esp32-c3-bms-monitor.yaml
```

### Getting help

- **Issues**: open an [issue](https://github.com/ngoviet/esp32-jk-bms-48v-fan-control/issues) on GitHub
- **Email**: vokupt@gmail.com
- **ESPHome Docs**: https://esphome.io/

## Project layout

```
├── esp32-c3-bms-monitor.yaml    # Main ESP32-C3 configuration
├── base-setup.yaml              # Basic ESPHome components
├── fan-control.yaml             # Fan control logic
├── secrets.yaml                 # WiFi credentials (gitignored)
├── flash-c3.bat                 # Flash script
├── log.bat                      # Log viewer script
└── .gitignore                   # Git ignore file
```

## Development

### Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

<!-- support:start -->
## Support the project

<p align="center">
  <img src="https://raw.githubusercontent.com/ngoviet/.github/main/assets/support/banner.svg" width="720" alt="Built and maintained by one person in Việt Nam">
</p>

<p align="center">
  <a href="https://buymeacoffee.com/ngoviet"><img src="https://raw.githubusercontent.com/ngoviet/.github/main/assets/support/btn-coffee.svg" height="56" alt="Buy me a coffee"></a>
  &nbsp;
  <a href="https://github.com/sponsors/ngoviet"><img src="https://raw.githubusercontent.com/ngoviet/.github/main/assets/support/btn-sponsors.svg" height="56" alt="GitHub Sponsors"></a>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/ngoviet/.github/main/assets/support/vietqr-card.svg" width="300" alt="VietQR — MB Bank — NGO DUC VIET — 911051212">
</p>

<p align="center"><sub>🇻🇳 VietQR: scan with any Vietnamese banking app, transfer note <code>Ung ho esp32-jk-bms-48v-fan-control</code>. Thank you for keeping this project alive!</sub></p>
<p align="center"><sub>Crypto (BEP20 network): <code>0x57f07d44fb581cddc028a0c67d63a8cc05aa6caa</code> · USDT · USDC · BNB · ETH · BTC</sub></p>
<!-- support:end -->

## License

[MIT](LICENSE) © Ngô Đức Việt.
