# ESP32-S3 系列数据手册 (Datasheet)

> 来源: https://documentation.espressif.com/esp32-s3_datasheet_en.html
> 版本: v2.2

## 1. 概述

ESP32-S3 是一款双核 Xtensa LX7 MCU，主频可达 240 MHz。集成了 2.4 GHz Wi-Fi (802.11 b/g/n) 和 Bluetooth 5 (LE) 连接功能，支持长距离通信。

### 1.1 主要特性

- **处理器**: Xtensa® 32-bit LX7 双核处理器，最高 240 MHz
- **内存**: 512 KB SRAM，384 KB ROM，支持 SPI/Dual SPI/Quad SPI/Octal SPI/QPI/OPI 接口连接 Flash 和外部 RAM
- **AI 加速**: 额外支持向量指令，加速神经网络计算和信号处理工作负载
- **无线连接**:
  - Wi-Fi: IEEE 802.11b/g/n, 2.4 GHz, HT20/40, 最高 150 Mbps
  - Bluetooth: Bluetooth LE v5.0
- **外设**: 45 个可编程 GPIO，SPI, I2S, I2C, PWM, RMT, ADC, UART, SD/MMC host, TWAI™
- **安全**: RSA 安全启动, AES-XTS Flash 加密, 数字签名, HMAC, World Controller
- **工作电压**: 3.0 ~ 3.6 V
- **封装**: QFN56 (7×7 mm)

### 1.2 芯片型号变体

| 型号 | Flash | PSRAM | 温度范围 | 状态 |
|------|-------|-------|---------|------|
| ESP32-S3 | 0 (Quad) | 0 (Quad) | -40 ~ 105°C | 量产 |
| ESP32-S3R8 | 0 (Quad) | 8 MB (Octal) | -40 ~ 65°C | 量产 |
| ESP32-S3RH2 | 0 (Quad) | 2 MB (Quad) | -40 ~ 105°C | 量产 |
| ESP32-S3FN8 | 8 MB (Quad) | 0 (Quad) | -40 ~ 85°C | 量产 |
| ESP32-S3FH4R2 | 4 MB (Quad) | 2 MB (Quad) | -40 ~ 85°C | 量产 |
| ESP32-S3R16V | 0 | 16 MB (Octal) | - | 量产 |

---

## 2. 引脚定义

### 2.1 引脚布局

ESP32-S3 采用 QFN56 封装，引脚布局（顶视图）如下：

### 2.2 引脚概述

ESP32-S3 芯片集成多种外设，引脚类型包括：
- **IO 引脚**: 具有预定义功能集（IO MUX, RTC IO MUX, Analog）
- **模拟引脚**: 专用模拟功能
- **电源引脚**: 为芯片组件供电

### 2.3 引脚列表

| 引脚号 | 引脚名称 | 类型 | 供电域 | 复位设置 | IO MUX | RTC IO MUX | Analog |
|--------|---------|------|--------|---------|--------|-----------|--------|
| 1 | LNA_IN | Analog | — | — | — | — | — |
| 2 | VDD3P3 | Power | — | — | — | — | — |
| 3 | VDD3P3 | Power | — | — | — | — | — |
| 4 | CHIP_PU | Analog | VDD3P3_RTC | — | — | — | — |
| 5 | GPIO0 | IO | VDD3P3_RTC | WPU, IE | **IO MUX** | RTC IO MUX | — |
| 6 | GPIO1 | IO | VDD3P3_RTC | IE | **IO MUX** | RTC IO MUX | Analog |
| 7 | GPIO2 | IO | VDD3P3_RTC | IE | **IO MUX** | RTC IO MUX | Analog |
| 8 | GPIO3 | IO | VDD3P3_RTC | IE | **IO MUX** | RTC IO MUX | Analog |
| 9 | GPIO4 | IO | VDD3P3_RTC | — | **IO MUX** | RTC IO MUX | Analog |
| 10 | GPIO5 | IO | VDD3P3_RTC | — | **IO MUX** | RTC IO MUX | Analog |
| 11 | GPIO6 | IO | VDD3P3_RTC | — | **IO MUX** | RTC IO MUX | Analog |
| 12 | GPIO7 | IO | VDD3P3_RTC | — | **IO MUX** | RTC IO MUX | Analog |
| 13 | GPIO8 | IO | VDD3P3_RTC | — | **IO MUX** | RTC IO MUX | Analog |
| 14 | GPIO9 | IO | VDD3P3_RTC | IE | **IO MUX** | RTC IO MUX | Analog |
| 15 | GPIO10 | IO | VDD3P3_RTC | IE | **IO MUX** | RTC IO MUX | Analog |
| 16 | GPIO11 | IO | VDD3P3_RTC | IE | **IO MUX** | RTC IO MUX | Analog |
| 17 | GPIO12 | IO | VDD3P3_RTC | IE | **IO MUX** | RTC IO MUX | Analog |
| 18 | GPIO13 | IO | VDD3P3_RTC | IE | **IO MUX** | RTC IO MUX | Analog |
| 19 | GPIO14 | IO | VDD3P3_RTC | IE | **IO MUX** | RTC IO MUX | Analog |
| 20 | VDD3P3_RTC | Power | — | — | — | — | — |
| 21 | XTAL_32K_P | IO | VDD3P3_RTC | — | **IO MUX** | RTC IO MUX | Analog |
| 22 | XTAL_32K_N | IO | VDD3P3_RTC | — | **IO MUX** | RTC IO MUX | Analog |
| 23 | GPIO17 | IO | VDD3P3_RTC | IE | **IO MUX** | RTC IO MUX | Analog |
| 24 | GPIO18 | IO | VDD3P3_RTC | IE | **IO MUX** | RTC IO MUX | Analog |
| 25 | GPIO19 | IO | VDD3P3_RTC | — | **IO MUX** | RTC IO MUX | Analog |
| 26 | GPIO20 | IO | VDD3P3_RTC | USB_PU | **IO MUX** | RTC IO MUX | Analog |
| 27 | GPIO21 | IO | VDD3P3_RTC | — | **IO MUX** | — | — |
| 28 | SPICS1 | IO | VDD_SPI | WPU, IE | **IO MUX** | — | — |
| 29 | VDD_SPI | Power | — | — | — | — | — |
| 30 | SPIHD | IO | VDD_SPI | WPU, IE | **IO MUX** | — | — |
| 31 | SPIWP | IO | VDD_SPI | WPU, IE | **IO MUX** | — | — |
| 32 | SPICS0 | IO | VDD_SPI | WPU, IE | **IO MUX** | — | — |
| 33 | SPICLK | IO | VDD_SPI | WPU, IE | **IO MUX** | — | — |
| 34 | SPIQ | IO | VDD_SPI | WPU, IE | **IO MUX** | — | — |
| 35 | SPID | IO | VDD_SPI | WPU, IE | **IO MUX** | — | — |
| 36 | SPICLK_N | IO | VDD_SPI/VDD3P3_CPU | IE | **IO MUX** | — | — |
| 37 | SPICLK_P | IO | VDD_SPI/VDD3P3_CPU | IE | **IO MUX** | — | — |
| 38 | GPIO33 | IO | VDD_SPI/VDD3P3_CPU | IE | **IO MUX** | — | — |
| 39 | GPIO34 | IO | VDD_SPI/VDD3P3_CPU | — | **IO MUX** | — | — |
| 40 | GPIO35 | IO | VDD_SPI/VDD3P3_CPU | — | **IO MUX** | — | — |
| 41 | GPIO36 | IO | VDD_SPI/VDD3P3_CPU | — | **IO MUX** | — | — |
| 42 | GPIO37 | IO | VDD_SPI/VDD3P3_CPU | — | **IO MUX** | — | — |
| 43 | GPIO38 | IO | VDD3P3_CPU | — | **IO MUX** | — | — |
| 44 | MTCK | IO | VDD3P3_CPU | — | **IO MUX** | — | — |
| 45 | MTDO | IO | VDD3P3_CPU | — | **IO MUX** | — | — |
| 46 | VDD3P3_CPU | Power | — | — | — | — | — |
| 47 | MTDI | IO | VDD3P3_CPU | — | **IO MUX** | — | — |
| 48 | MTMS | IO | VDD3P3_CPU | — | **IO MUX** | — | — |
| 49 | U0TXD | IO | VDD3P3_CPU | WPU, IE | **IO MUX** | — | — |
| 50 | U0RXD | IO | VDD3P3_CPU | WPU, IE | **IO MUX** | — | — |
| 51 | GPIO45 | IO | VDD3P3_CPU | WPD, IE | **IO MUX** | — | — |
| 52 | GPIO46 | IO | VDD3P3_CPU | WPD, IE | **IO MUX** | — | — |
| 53 | XTAL_N | Analog | — | — | — | — | — |
| 54 | XTAL_P | Analog | — | — | — | — | — |
| 55 | VDDA | Power | — | — | — | — | — |
| 56 | VDDA | Power | — | — | — | — | — |
| 57 | GND | Power | — | — | — | — | — |

**注释:**
1. **粗体** 标记的引脚功能集是默认启动模式下的默认功能。
2. 缩写说明: IE – 输入使能; WPU – 内部弱上拉电阻使能; WPD – 内部弱下拉电阻使能; USB_PU – USB 上拉电阻使能。

---

## 3. 功能框图

ESP32-S3 功能框图包含以下主要模块：
- **双核 Xtensa LX7 处理器** (Core 0 & Core 1)
- **Wi-Fi 子系统** (支持 Station, SoftAP, SoftAP+Station, Promiscuous 模式)
- **Bluetooth LE 子系统** (支持 Bluetooth 5)
- **存储系统** (512 KB SRAM, 384 KB ROM, Flash/PSRAM 接口)
- **安全模块** (AES, RSA, SHA, Digital Signature, HMAC, World Controller)
- **外设总线** (GPIO, SPI, I2S, I2C, PWM, RMT, ADC, UART, USB OTG, USB Serial/JTAG, TWAI, SD/MMC)
- **ULP 低功耗核心**

---

## 4. 外设引脚配置

### 4.1 GPIO 矩阵

ESP32-S3 的 GPIO 矩阵允许将任意 GPIO 引脚映射到任意外设功能，提供了极大的灵活性。

### 4.2 IO MUX 功能

IO MUX 引脚具有固定的默认功能映射，可提供更好的信号完整性。

---

## 5. 电气特性

- **工作电压**: 3.0 ~ 3.6 V
- **工作温度**: -40 ~ 105°C (视型号而定)
- **Wi-Fi 发射功率**: 最高 20 dBm
- **BLE 发射功率**: 最高 20 dBm

---

## 6. 封装信息

- **封装类型**: QFN56
- **尺寸**: 7 × 7 mm
- **引脚数**: 56 + 裸露焊盘

---

## 7. 相关文档

- [ESP32-S3 Technical Reference Manual](https://www.espressif.com/sites/default/files/documentation/esp32-s3_technical_reference_manual_en.pdf)
- [ESP32-S3 Hardware Design Guidelines](https://www.espressif.com/sites/default/files/documentation/esp32-s3_hardware_design_guidelines_en.pdf)
- [ESP32-S3 Programming Guide](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/api-reference/index.html)
