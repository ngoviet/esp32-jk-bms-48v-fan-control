# ESP32 giám sát BMS 48V với điều khiển quạt tự động

<!-- readme-axi type=esp32 -->

[English](README.md) · **Tiếng Việt**

Cấu hình **ESPHome** để giám sát hệ thống quản lý pin **JK-BMS** (LiFePO4 16S) qua Modbus, với
điều khiển quạt tự động dựa trên nhiệt độ từ **Home Assistant**. Chạy trên **ESP32 Dev Module**
hoặc **ESP32-C3 SuperMini**.

> 🇻🇳 Dự án do một người Việt tự xây dựng và duy trì — nếu nó giúp ích cho bạn, hãy
> [ủng hộ dự án](#ủng-hộ-dự-án).

## Mục lục

- [Tính năng](#tính-năng)
- [Phần cứng](#phần-cứng)
- [Bắt đầu nhanh](#bắt-đầu-nhanh)
- [Cấu hình](#cấu-hình)
- [Cách hoạt động](#cách-hoạt-động)
- [Home Assistant](#home-assistant)
- [Xử lý sự cố](#xử-lý-sự-cố)
- [Cấu trúc dự án](#cấu-trúc-dự-án)
- [Phát triển](#phát-triển)

## Tính năng

- **Giám Sát JK-BMS**: Giám sát toàn diện hệ thống pin LiFePO4 16S qua Modbus
- **200+ Cảm Biến**: Điện áp cell, nhiệt độ, dòng điện, công suất, dung lượng, chu kỳ...
- **Điều Khiển Quạt Tự Động**: Điều khiển quạt theo nhiệt độ với logic hysteresis
- **Tích Hợp Home Assistant**: Kết nối trực tiếp qua ESPHome API
- **Hỗ Trợ Đa Board**: Cấu hình cho ESP32 và ESP32-C3 SuperMini
- **Cấu Hình Bảo Mật**: Thông tin WiFi được lưu trong file riêng biệt

## Phần cứng

### Cấu hình ESP32

- **Board**: ESP32 Dev Module
- **Chân UART**: GPIO16 (TX), GPIO17 (RX)
- **Chân Quạt**: GPIO25 (PWM)
- **Framework**: ESP-IDF

### Cấu hình ESP32-C3 SuperMini

- **Board**: ESP32-C3-DevKitM-1
- **Chân UART**: GPIO21 (TX), GPIO20 (RX)
- **Chân Quạt**: GPIO10
- **Framework**: ESP-IDF

### Kết nối JK-BMS

- **Giao Thức**: Modbus RTU qua UART
- **Tốc Độ**: 115200 baud
- **Địa Chỉ Thiết Bị**: 0x09
- **Tần Suất Cập Nhật**: 5-10 giây

## Bắt đầu nhanh

### 1. Tải Repository

```bash
git clone https://github.com/ngoviet/esp32-jk-bms-48v-fan-control.git
cd esp32-jk-bms-48v-fan-control
```

### 2. Thiết Lập Secrets

Sao chép và chỉnh sửa file secrets:

```bash
cp secrets.yaml.example secrets.yaml
# Chỉnh sửa secrets.yaml với thông tin WiFi của bạn
```

### 3. Flash Firmware

```bash
# Cho ESP32-C3
esphome run esp32-c3-bms-monitor.yaml

# Hoặc dùng file batch
flash-c3.bat  # Flash qua USB
log.bat       # Xem logs
```

## Cấu hình

### Cài đặt quạt

```yaml
substitutions:
  fan_pin: GPIO10                    # Fan control pin
  FAN_THR_HI_DEFAULT: "42.0"        # Auto ON threshold
  FAN_THR_LO_DEFAULT: "41.0"        # Auto OFF threshold
```

### Cài đặt BMS

```yaml
substitutions:
  device_addr: "9"                   # Địa chỉ Modbus BMS
  tx_pin: GPIO21                     # Chân TX UART
  rx_pin: GPIO20                     # Chân RX UART
  modbus_controller_update_interval: "10s"
```

### Thiết Lập WiFi (secrets.yaml)

```yaml
wifi_ssid: "Ten_WiFi_Cua_Ban"
wifi_password: "Mat_Khau_WiFi_Cua_Ban"
wifi_bssid: "aa:bb:cc:dd:ee:ff"    # Tùy chọn: AP cụ thể
wifi_channel: 11                    # Tùy chọn: kênh cụ thể
```

### Bảo mật

- **Quản Lý Secrets**: Thông tin WiFi được lưu riêng biệt
- **Bảo Vệ Git**: `.gitignore` ngăn chặn lộ thông tin nhạy cảm
- **Bảo Vệ OTA**: Tùy chọn bảo vệ mật khẩu cho cập nhật
- **Mã Hóa API**: Tùy chọn mã hóa giao tiếp API

## Cách hoạt động

Hệ thống quạt tự động điều khiển làm mát dựa trên nhiệt độ từ Home Assistant:

### Chế Độ Tự Động (Mặc Định)

- **Bật Quạt**: Khi nhiệt độ ≥ T_HIGH (mặc định: 42°C)
- **Tắt Quạt**: Khi nhiệt độ < T_LOW (mặc định: 41°C)
- **Hysteresis**: Ngăn chặn bật/tắt liên tục

### Chế Độ Thủ Công

- Ghi đè điều khiển tự động
- Điều khiển bật/tắt trực tiếp

### Nguồn Nhiệt Độ

- **Chính**: `sensor.device_h240909079_device_temperature` từ Home Assistant
- **Dự Phòng**: Không có (quạt tắt nếu không có dữ liệu để đảm bảo an toàn)

## Home Assistant

### Thực Thể Được Tạo

- **200+ cảm biến**: Tất cả thông số BMS
- **Điều khiển quạt**: Chế độ tự động/thủ công, ngưỡng nhiệt
- **Cảm biến nhị phân**: Trạng thái sạc, xả, cân bằng
- **Chẩn đoán**: Tín hiệu WiFi, địa chỉ IP, thông tin thiết bị

### Các Cảm Biến Có Sẵn

#### Giám Sát Pin

- Điện áp tế bào (1-16): Giám sát điện áp từng tế bào riêng lẻ
- Điện áp gói pin: Tổng điện áp của gói pin
- Dòng điện: Dòng sạc/xả có hướng
- Công suất: Tiêu thụ/phát điện thời gian thực
- SOC: Phần trăm mức sạc
- SOH: Phần trăm tình trạng sức khỏe pin

#### Giám Sát Nhiệt Độ

- Nhiệt độ MOSFET: Linh kiện chuyển mạch công suất
- Nhiệt độ pin: Nhiều đầu dò nhiệt độ
- Nhiệt độ môi trường: Giám sát môi trường

#### Bảo Vệ & Trạng Thái

- Cài đặt bảo vệ quá áp/thiếu áp
- Cài đặt bảo vệ quá dòng
- Giới hạn bảo vệ nhiệt độ
- Trạng thái và dòng cân bằng
- Trạng thái sạc/xả
- Điều kiện báo động và mã lỗi

### Cards Dashboard

```yaml
# Ví dụ card điều khiển quạt
type: entities
entities:
  - entity: switch.fan_auto_mode
  - entity: switch.fan_manual_control
  - entity: number.fan_t_high_c
  - entity: number.fan_t_low_c
  - entity: sensor.temperature_sensor_1
```

## Xử lý sự cố

### Các Vấn Đề Thường Gặp

**Lỗi Kiểm Tra CRC**

```
[W][modbus:135]: Modbus CRC Check failed! 6C0!=603
```

- Lỗi thỉnh thoảng do nhiễu điện bình thường
- Không ảnh hưởng chức năng nếu vẫn nhận được dữ liệu
- Cân nhắc thêm `send_wait_time: 50ms` để giảm tần suất

**Không Có Dữ Liệu Nhiệt Độ**

```
[W][fan]: No temperature data available, turning fan OFF
```

- Kiểm tra thực thể Home Assistant: `sensor.device_h240909079_device_temperature`
- Xác minh ESPHome có thể kết nối với Home Assistant
- Quạt sẽ an toàn TẮT cho đến khi có dữ liệu

**Lỗi Compilation**

- Đảm bảo tất cả tham chiếu `id:` khớp giữa các file
- Kiểm tra thụt lề YAML (dùng space, không phải tab)
- Xác minh phân công chân cho board cụ thể của bạn

### Lệnh Debug

```bash
# Kiểm tra cấu hình
esphome config esp32-c3-bms-monitor.yaml

# Xem logs
esphome logs esp32-c3-bms-monitor.yaml --device 192.168.x.x

# Xóa build cache
esphome clean esp32-c3-bms-monitor.yaml
```

### Cần hỗ trợ

- **Issues**: Mở [issue](https://github.com/ngoviet/esp32-jk-bms-48v-fan-control/issues) trên GitHub
- **Email**: vokupt@gmail.com
- **Tài liệu ESPHome**: https://esphome.io/

## Cấu trúc dự án

```
├── esp32-c3-bms-monitor.yaml    # Cấu hình chính ESP32-C3
├── base-setup.yaml              # Các thành phần ESPHome cơ bản
├── fan-control.yaml             # Logic điều khiển quạt
├── secrets.yaml                 # Thông tin WiFi (đã gitignore)
├── flash-c3.bat                 # Script flash
├── log.bat                      # Script xem log
└── .gitignore                   # File git ignore
```

## Phát triển

### Đóng góp

1. Fork repository
2. Tạo feature branch (`git checkout -b feature/amazing-feature`)
3. Commit thay đổi (`git commit -m 'Add amazing feature'`)
4. Push lên branch (`git push origin feature/amazing-feature`)
5. Mở Pull Request

<!-- support:start -->
## Ủng hộ dự án

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
  &nbsp;
  <img src="https://raw.githubusercontent.com/ngoviet/.github/main/assets/support/crypto-card.svg" width="300" alt="Crypto — BNB Smart Chain (BEP20) — 0xcbf0402b232cb24b4d998f09a4884f87b5f74750">
</p>

<p align="center"><b>Ví tiền mã hoá, chỉ gửi qua mạng BNB Smart Chain (BEP20):</b><br><code>0xcbf0402b232cb24b4d998f09a4884f87b5f74750</code></p>

<p align="center"><sub>🇻🇳 VietQR: quét bằng app ngân hàng bất kỳ, nội dung <code>Ung ho esp32-jk-bms-48v-fan-control</code>. Cảm ơn bạn đã giúp dự án tiếp tục phát triển!</sub></p>
<!-- support:end -->

## Giấy phép

[MIT](LICENSE) © Ngô Đức Việt.
