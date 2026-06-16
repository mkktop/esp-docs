# ESP32-S3-PICO-1 数据手册 (Datasheet)

> 来源: https://documentation.espressif.com/esp32-s3-pico-1_datasheet_en.html
> 版本: v1.1

---

## 1. 产品概述

ESP32-S3-PICO-1 是一款系统级封装 (SiP) 器件，基于 ESP32-S3 芯片，集成了 2.4 GHz Wi-Fi 和 Bluetooth® Low Energy (Bluetooth LE)。内置 8 MB SPI Flash 和最高 8 MB SPI PSRAM。

### 1.1 主要特性

- **CPU**: Xtensa® 双核 32-bit LX7 微处理器（单精度 FPU），最高 240 MHz
- **存储**: 384 KB ROM, 512 KB SRAM, 16 KB RTC SRAM
- **Flash**: 8 MB Quad SPI Flash
- **PSRAM**: 最高 8 MB PSRAM
- **Wi-Fi**: 802.11b/g/n, 最高 150 Mbps
- **Bluetooth**: Bluetooth LE 5.0, Bluetooth mesh
- **集成组件**: 40 MHz 晶振, 去耦电容, RF 匹配电路
- **封装**: LGA56 (7×7 mm)
- **工作电压**: 3.0 ~ 3.6 V

### 1.2 应用场景

- 通用低功耗 IoT 传感器
- 低功耗数据记录器
- 视频流摄像头
- OTT 设备
- USB 设备
- 语音识别
- 图像识别
- Mesh 网络
- 智能家居
- 智能建筑
- 工业自动化
- 智慧农业
- 音频应用
- 医疗健康
- 可穿戴电子设备

---

## 2. 系列型号对比

### 2.1 命名规则

ESP32-S3-PICO-1 H/N x R x:
- **SiP 系列**: ESP32-S3-PICO-1
- **Flash 温度等级**: H (高温) / N (常温)
- **Flash 大小 (MB)**: x
- **PSRAM**: R
- **PSRAM 大小 (MB)**: x

### 2.2 型号对比表

| 型号 | 内置 Flash | 内置 PSRAM | 工作温度 | SPI 电压 |
|------|-----------|-----------|---------|---------|
| ESP32-S3-PICO-1-N8R2 | 8 MB (Quad SPI) | 2 MB (Quad SPI) | –40 ~ 85°C | 3.3 V |
| ESP32-S3-PICO-1-N8R8 | 8 MB (Quad SPI) | 8 MB (Octal SPI) | –40 ~ 65°C | 3.3 V |

**注:** 对于 ESP32-S3-PICO-1-N8R8，如果启用 PSRAM ECC 功能，最高环境温度可提高到 85°C，但可用 PSRAM 大小将减少 1/16。

---

## 3. 功能框图

ESP32-S3-PICO-1 内部包含：
- **ESP32-S3 SoC**: 主控芯片
- **40 MHz Crystal**: 晶体振荡器
- **QSPI Flash**: 8 MB 四线 SPI Flash
- **QSPI/OSPI PSRAM**: 可选 PSRAM
- **RF Matching**: RF 匹配电路

---

## 4. 引脚定义

### 4.1 引脚布局

ESP32-S3-PICO-1 采用 LGA56 封装，引脚按逆时针方向排列（顶视图）。

### 4.2 引脚描述

| 名称 | 引脚号 | 类型 | 供电域 | 功能 |
|------|--------|------|--------|------|
| LNA_IN | 1 | I/O | — | RF LNA 输入/输出信号 |
| VDD3P3 | 2 | PA | — | 模拟电源 |
| VDD3P3 | 3 | PA | — | 模拟电源 |
| CHIP_PU | 4 | I | VDD3P3_RTC | 高: 使能, 低: 关闭 (不可悬空) |
| GPIO0 | 5 | I/O/T | VDD3P3_RTC | RTC_GPIO0, **GPIO0** |
| GPIO1 | 6 | I/O/T | VDD3P3_RTC | RTC_GPIO1, **GPIO1**, TOUCH1, ADC1_CH0 |
| GPIO2 | 7 | I/O/T | VDD3P3_RTC | RTC_GPIO2, **GPIO2**, TOUCH2, ADC1_CH1 |
| GPIO3 | 8 | I/O/T | VDD3P3_RTC | RTC_GPIO3, **GPIO3**, TOUCH3, ADC1_CH2 |
| GPIO4 | 9 | I/O/T | VDD3P3_RTC | RTC_GPIO4, **GPIO4**, TOUCH4, ADC1_CH3 |
| GPIO5 | 10 | I/O/T | VDD3P3_RTC | RTC_GPIO5, **GPIO5**, TOUCH5, ADC1_CH4 |
| GPIO6 | 11 | I/O/T | VDD3P3_RTC | RTC_GPIO6, **GPIO6**, TOUCH6, ADC1_CH5 |
| GPIO7 | 12 | I/O/T | VDD3P3_RTC | RTC_GPIO7, **GPIO7**, TOUCH7, ADC1_CH6 |
| GPIO8 | 13 | I/O/T | VDD3P3_RTC | RTC_GPIO8, **GPIO8**, TOUCH8, ADC1_CH7, SUBSPICS1 |
| GPIO9 | 14 | I/O/T | VDD3P3_RTC | RTC_GPIO9, **GPIO9**, TOUCH9, ADC1_CH8, SUBSPIHD, FSPIHD |
| GPIO10 | 15 | I/O/T | VDD3P3_RTC | RTC_GPIO10, **GPIO10**, TOUCH10, ADC1_CH9, FSPIIO4, SUBSPICS0, FSPICS0 |
| GPIO11 | 16 | I/O/T | VDD3P3_RTC | RTC_GPIO11, **GPIO11**, TOUCH11, ADC2_CH0, FSPIIO5, SUBSPID, FSPID |
| GPIO12 | 17 | I/O/T | VDD3P3_RTC | RTC_GPIO12, **GPIO12**, TOUCH12, ADC2_CH1, FSPIIO6, SUBSPICLK, FSPICLK |
| GPIO13 | 18 | I/O/T | VDD3P3_RTC | RTC_GPIO13, **GPIO13**, TOUCH13, ADC2_CH2, FSPIIO7, SUBSPIQ, FSPIQ |
| GPIO14 | 19 | I/O/T | VDD3P3_RTC | RTC_GPIO14, **GPIO14**, TOUCH14, ADC2_CH3, FSPIDQS, SUBSPIWP, FSPIWP |
| VDD3P3_RTC | 20 | PA | — | 模拟电源 |
| XTAL_32K_P | 21 | I/O/T | VDD3P3_RTC | RTC_GPIO15, **GPIO15**, U0RTS, ADC2_CH4, XTAL_32K_P |
| XTAL_32K_N | 22 | I/O/T | VDD3P3_RTC | RTC_GPIO16, **GPIO16**, U0CTS, ADC2_CH5, XTAL_32K_N |
| GPIO17 | 23 | I/O/T | VDD3P3_RTC | RTC_GPIO17, **GPIO17**, U1TXD, ADC2_CH6 |
| GPIO18 | 24 | I/O/T | VDD3P3_RTC | RTC_GPIO18, **GPIO18**, U1RXD, ADC2_CH7, CLK_OUT3 |
| GPIO19 | 25 | I/O/T | VDD3P3_RTC | RTC_GPIO19, **GPIO19**, U1RTS, ADC2_CH8, CLK_OUT2, **USB_D-** |
| GPIO20 | 26 | I/O/T | VDD3P3_RTC | RTC_GPIO20, **GPIO20**, U1CTS, ADC2_CH9, CLK_OUT1, **USB_D+** |
| GPIO21 | 27 | I/O/T | VDD3P3_RTC | RTC_GPIO21, **GPIO21** |
| SPICS1 | 28 | I/O/T | VDD_SPI | SPICS1, **GPIO26** |
| VDD_SPI | 29 | PD | — | 输出电源: VDD3P3_RTC |
| NC | 30~35 | — | — | 未连接 |
| SPICLK_N | 36 | I/O/T | VDD3P3_CPU/VDD_SPI | SPICLK_N_DIFF, **GPIO48**, SUBSPICLK_N_DIFF |
| SPICLK_P | 37 | I/O/T | VDD3P3_CPU/VDD_SPI | SPICLK_P_DIFF, **GPIO47**, SUBSPICLK_P_DIFF |
| GPIO33 | 38 | I/O/T | VDD3P3_CPU/VDD_SPI | SPIIO4, **GPIO33**, FSPIHD, SUBSPIHD |
| GPIO34 | 39 | I/O/T | VDD3P3_CPU/VDD_SPI | SPIIO5, **GPIO34**, FSPICS0, SUBSPICS0 |
| GPIO35 | 40 | I/O/T | VDD3P3_CPU/VDD_SPI | SPIIO6, **GPIO35**, FSPID, SUBSPID |
| GPIO36 | 41 | I/O/T | VDD3P3_CPU/VDD_SPI | SPIIO7, **GPIO36**, FSPICLK, SUBSPICLK |
| GPIO37 | 42 | I/O/T | VDD3P3_CPU/VDD_SPI | SPIDQS, **GPIO37**, FSPIQ, SUBSPIQ |
| GPIO38 | 43 | I/O/T | VDD3P3_CPU | **GPIO38**, FSPIWP, SUBSPIWP |
| MTCK | 44 | I/O/T | VDD3P3_CPU | **MTCK**, GPIO39, CLK_OUT3, SUBSPICS1 |
| MTDO | 45 | I/O/T | VDD3P3_CPU | **MTDO**, GPIO40, CLK_OUT2 |
| VDD3P3_CPU | 46 | PD | — | CPU IO 输入电源 |
| MTDI | 47 | I/O/T | VDD3P3_CPU | **MTDI**, GPIO41, CLK_OUT1 |
| MTMS | 48 | I/O/T | VDD3P3_CPU | **MTMS**, GPIO42 |
| U0TXD | 49 | I/O/T | VDD3P3_CPU | **U0TXD**, GPIO43, CLK_OUT1 |
| U0RXD | 50 | I/O/T | VDD3P3_CPU | **U0RXD**, GPIO44, CLK_OUT2 |
| GPIO45 | 51 | I/O/T | VDD3P3_CPU | **GPIO45** |
| GPIO46 | 52 | I/O/T | VDD3P3_CPU | **GPIO46** |
| NC | 53~54 | — | — | 未连接 |
| VDDA | 55~56 | PA | — | 模拟电源 |
| GND | 57 | G | — | 接地 |

**注释:**
- **粗体** 标记的为默认引脚功能
- P: 电源; I: 输入; O: 输出; T: 高阻态
- 对于 ESP32-S3-PICO-1-N8R8，IO35-IO37 连接到 Octal SPI PSRAM，不可用于其他用途

---

## 5. 电气特性

### 5.1 绝对最大额定值

| 参数 | 描述 | 最小 | 最大 | 单位 |
|------|------|------|------|------|
| VDDA, VDD3P3, VDD3P3_RTC, VDD3P3_CPU, VDD_SPI | 允许输入电压 | –0.3 | 3.6 | V |
| Ioutput | 累计 IO 输出电流 | — | 1500 | mA |
| TSTORE | 存储温度 | –40 | 150 | °C |

### 5.2 推荐电源特性

| 参数 | 描述 | 最小 | 典型 | 最大 | 单位 |
|------|------|------|------|------|------|
| VDDA, VDD3P3 | 推荐输入电压 | 3.0 | 3.3 | 3.6 | V |
| VDD3P3_RTC | 推荐输入电压 | 3.0 | 3.3 | 3.6 | V |
| VDD_SPI (输入) | — | 1.8 | 3.3 | 3.6 | V |
| VDD3P3_CPU | 推荐输入电压 | 3.0 | 3.3 | 3.6 | V |
| IVDD | 累计输入电流 | 0.5 | — | — | A |

### 5.3 DC 特性 (3.3V, 25°C)

| 符号 | 参数 | 最小 | 典型 | 最大 | 单位 |
|------|------|------|------|------|------|
| CIN | 引脚电容 | — | 2 | — | pF |
| VIH | 高电平输入电压 | 0.75×VDD | — | VDD+0.3 | V |
| VIL | 低电平输入电压 | –0.3 | — | 0.25×VDD | V |
| VOH | 高电平输出电压 | 0.8×VDD | — | — | V |
| VOL | 低电平输出电压 | — | — | 0.1×VDD | V |
| IOH | 高电平源电流 | — | 40 | — | mA |
| IOL | 低电平灌电流 | — | 28 | — | mA |
| RPU | 内部弱上拉电阻 | — | 45 | — | kΩ |
| RPD | 内部弱下拉电阻 | — | 45 | — | kΩ |

---

## 6. 封装信息

- **封装类型**: LGA56
- **尺寸**: 7.0 × 7.0 mm
- **高度**: 0.86 ~ 1.06 mm
- **引脚间距**: 0.4 mm
- **引脚数**: 56 + 中央接地焊盘

---

## 7. 原理图

ESP32-S3-PICO-1 原理图包含：
- ESP32-S3 SoC
- 8 MB QSPI Flash (U3 或 U4)
- 可选 QSPI/OSPI PSRAM
- 40 MHz 晶振
- RF 匹配电路
- 电源去耦电容

### 引脚映射表

| ESP32-S3 芯片引脚 | ESP32-S3-PICO-1 引脚 |
|-------------------|---------------------|
| LNA_IN | LNA_IN |
| VDD3P3 | VDD3P3 |
| CHIP_PU | CHIP_PU |
| GPIO0~GPIO21 | GPIO0~GPIO21 |
| SPICS1 | SPICS1 |
| VDD_SPI | VDD_SPI |
| SPIHD, SPIWP, SPICS0, SPICLK, SPIQ, SPID | NC (内部连接) |
| SPICLK_N, SPICLK_P | SPICLK_N, SPICLK_P |
| GPIO33~GPIO38 | GPIO33~GPIO38 |
| MTCK, MTDO, MTDI, MTMS | MTCK, MTDO, MTDI, MTMS |
| U0TXD, U0RXD | U0TXD, U0RXD |
| GPIO45, GPIO46 | GPIO45, GPIO46 |
| XTAL_N, XTAL_P | NC (内部连接) |

**注意:**
- ESP32-S3-PICO-1-N8R2: 使用 U3 (QSPI PSRAM), U4 未连接
- ESP32-S3-PICO-1-N8R8: 使用 U4 (OSPI PSRAM), U3 未连接

---

## 8. 相关文档

- [ESP32-S3 Datasheet](https://documentation.espressif.com/esp32-s3_datasheet_en.html)
- [ESP32-S3 Technical Reference Manual](https://www.espressif.com/sites/default/files/documentation/esp32-s3_technical_reference_manual_en.pdf)
- [ESP32-S3 Hardware Design Guidelines](https://www.espressif.com/sites/default/files/documentation/esp32-s3_hardware_design_guidelines_en.pdf)
