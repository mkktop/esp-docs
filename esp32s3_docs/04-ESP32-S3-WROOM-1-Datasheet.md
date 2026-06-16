# ESP32-S3-WROOM-1 & ESP32-S3-WROOM-1U 数据手册 (Datasheet)

> 来源: https://documentation.espressif.com/esp32-s3-wroom-1_wroom-1u_datasheet_en.html

---

## 1. 模块概述

ESP32-S3-WROOM-1 是一款功能强大的通用 Wi-Fi + Bluetooth LE MCU 模块，搭载双核 CPU、丰富的外设集，并提供神经网络计算和信号处理工作负载的加速能力。它是 AI + IoT (AIoT) 应用场景的理想选择。

### 1.1 主要特性

**CPU 和片上存储**
- ESP32-S3 系列 SoC，Xtensa® 双核 32-bit LX7 微处理器（单精度 FPU），最高 240 MHz
- 384 KB ROM
- 512 KB SRAM
- 16 KB RTC SRAM
- 最高 16 MB PSRAM

**Wi-Fi**
- 802.11b/g/n
- 速率: 802.11n 最高 150 Mbps
- A-MPDU 和 A-MSDU 聚合
- 0.4 µs 保护间隔支持
- 工作信道中心频率范围: 2412 ~ 2484 MHz

**Bluetooth**
- Bluetooth LE: Bluetooth 5, Bluetooth mesh
- 速率: 125 Kbps, 500 Kbps, 1 Mbps, 2 Mbps
- 广播扩展 (Advertising extensions)
- 多广播集 (Multiple advertisement sets)
- 信道选择算法 #2
- Wi-Fi 和 Bluetooth 共用天线的内部共存机制

**外设**
- 36 个 GPIO
- 4 个 Strapping GPIO
- SPI, LCD 接口, Camera 接口, UART, I2C, I2S, 远程控制, 脉冲计数器, LED PWM
- 全速 USB 2.0 OTG, USB Serial/JTAG 控制器
- MCPWM, SD/MMC 主机控制器, GDMA
- TWAI® 控制器 (兼容 ISO 11898-1)
- ADC, 触摸传感器, 温度传感器, 定时器和看门狗

**集成组件**
- 40 MHz 晶体振荡器
- 最高 16 MB Quad SPI Flash

**天线选项**
- ESP32-S3-WROOM-1: 板载 PCB 天线
- ESP32-S3-WROOM-1U: 外置天线连接器 (IPEX)

**工作条件**
- 工作电压: 3.0 ~ 3.6 V
- 工作环境温度:
  - 65°C 版本: –40 ~ 65°C
  - 85°C 版本: –40 ~ 85°C
  - 105°C 版本: –40 ~ 105°C

### 1.2 应用场景

- 唤醒词检测和语音命令识别
- 人脸检测和识别
- 智能家居
- 智能家电
- 智能控制面板
- 智能音箱

---

## 2. 功能框图

### ESP32-S3-WROOM-1 框图
- 3V3 输入
- EN 引脚
- 40 MHz 晶振
- ESP32-S3 系列 SoC (ESP32-S3, ESP32-S3R2, ESP32-S3R8, ESP32-S3R16V)
- PSRAM (可选, QSPI/OSPI)
- QSPI Flash (信号: SPICS0, SPICLK, SPID, SPIQ, SPIHD, SPIWP, VDD_SPI)
- RF 匹配电路
- GPIO
- 板载天线

### ESP32-S3-WROOM-1U 框图
- 与 WROOM-1 相同，但使用外置天线连接器替代板载 PCB 天线

---

## 3. 引脚定义

### 3.1 引脚布局

模块引脚编号 1~40 沿周边排列，中央有接地焊盘 (Pin 41)。

**注意:** ESP32-S3-WROOM-1 有天线禁区 (Keepout Zone)，ESP32-S3-WROOM-1U 没有。

### 3.2 引脚描述

模块共有 41 个引脚。

| 名称 | 引脚号 | 类型 | 功能 |
|------|--------|------|------|
| GND | 1 | P | 接地 |
| 3V3 | 2 | P | 电源 |
| EN | 3 | I | 高: 使能, 低: 关闭 (不可悬空) |
| IO4 | 4 | I/O/T | RTC_GPIO4, GPIO4, TOUCH4, ADC1_CH3 |
| IO5 | 5 | I/O/T | RTC_GPIO5, GPIO5, TOUCH5, ADC1_CH4 |
| IO6 | 6 | I/O/T | RTC_GPIO6, GPIO6, TOUCH6, ADC1_CH5 |
| IO7 | 7 | I/O/T | RTC_GPIO7, GPIO7, TOUCH7, ADC1_CH6 |
| IO15 | 8 | I/O/T | RTC_GPIO15, GPIO15, U0RTS, ADC2_CH4, XTAL_32K_P |
| IO16 | 9 | I/O/T | RTC_GPIO16, GPIO16, U0CTS, ADC2_CH5, XTAL_32K_N |
| IO17 | 10 | I/O/T | RTC_GPIO17, GPIO17, U1TXD, ADC2_CH6 |
| IO18 | 11 | I/O/T | RTC_GPIO18, GPIO18, U1RXD, ADC2_CH7, CLK_OUT3 |
| IO8 | 12 | I/O/T | RTC_GPIO8, GPIO8, TOUCH8, ADC1_CH7, SUBSPICS1 |
| IO19 | 13 | I/O/T | RTC_GPIO19, GPIO19, U1RTS, ADC2_CH8, CLK_OUT2, USB_D- |
| IO20 | 14 | I/O/T | RTC_GPIO20, GPIO20, U1CTS, ADC2_CH9, CLK_OUT1, USB_D+ |
| IO3 | 15 | I/O/T | RTC_GPIO3, GPIO3, TOUCH3, ADC1_CH2 |
| IO46 | 16 | I/O/T | GPIO46 |
| IO9 | 17 | I/O/T | RTC_GPIO9, GPIO9, TOUCH9, ADC1_CH8, FSPIHD, SUBSPIHD |
| IO10 | 18 | I/O/T | RTC_GPIO10, GPIO10, TOUCH10, ADC1_CH9, FSPICS0, FSPIIO4, SUBSPICS0 |
| IO11 | 19 | I/O/T | RTC_GPIO11, GPIO11, TOUCH11, ADC2_CH0, FSPID, FSPIIO5, SUBSPID |
| IO12 | 20 | I/O/T | RTC_GPIO12, GPIO12, TOUCH12, ADC2_CH1, FSPICLK, FSPIIO6, SUBSPICLK |
| IO13 | 21 | I/O/T | RTC_GPIO13, GPIO13, TOUCH13, ADC2_CH2, FSPIQ, FSPIIO7, SUBSPIQ |
| IO14 | 22 | I/O/T | RTC_GPIO14, GPIO14, TOUCH14, ADC2_CH3, FSPIWP, FSPIDQS, SUBSPIWP |
| IO21 | 23 | I/O/T | RTC_GPIO21, GPIO21 |
| IO47 | 24 | I/O/T | SPICLK_P_DIFF, GPIO47, SUBSPICLK_P_DIFF |
| IO48 | 25 | I/O/T | SPICLK_N_DIFF, GPIO48, SUBSPICLK_N_DIFF |
| IO45 | 26 | I/O/T | GPIO45 |
| IO0 | 27 | I/O/T | RTC_GPIO0, GPIO0 |
| IO35 | 28 | I/O/T | SPIIO6, GPIO35, FSPID, SUBSPID |
| IO36 | 29 | I/O/T | SPIIO7, GPIO36, FSPICLK, SUBSPICLK |
| IO37 | 30 | I/O/T | SPIDQS, GPIO37, FSPIQ, SUBSPIQ |
| IO38 | 31 | I/O/T | GPIO38, FSPIWP, SUBSPIWP |
| IO39 | 32 | I/O/T | MTCK, GPIO39, CLK_OUT3, SUBSPICS1 |
| IO40 | 33 | I/O/T | MTDO, GPIO40, CLK_OUT2 |
| IO41 | 34 | I/O/T | MTDI, GPIO41, CLK_OUT1 |
| IO42 | 35 | I/O/T | MTMS, GPIO42 |
| RXD0 | 36 | I/O/T | U0RXD, GPIO44, CLK_OUT2 |
| TXD0 | 37 | I/O/T | U0TXD, GPIO43, CLK_OUT1 |
| IO2 | 38 | I/O/T | RTC_GPIO2, GPIO2, TOUCH2, ADC1_CH1 |
| IO1 | 39 | I/O/T | RTC_GPIO1, GPIO1, TOUCH1, ADC1_CH0 |
| GND | 40 | P | 接地 |
| EPAD | 41 | P | 接地 |

**注释:**
- **P**: 电源; **I**: 输入; **O**: 输出; **T**: 高阻态
- **粗体** 标记的为默认引脚功能
- 引脚 28~30 的默认功能由 eFuse 位决定
- 对于 Octal SPI PSRAM 模块 (ESP32-S3R8 或 ESP32-S3R16V)，IO35、IO36、IO37 连接到 PSRAM，不可用于其他用途
- 对于 ESP32-S3R16V 模块，VDD_SPI 电压为 1.8V，GPIO47 和 GPIO48 的工作电压也是 1.8V

---

## 4. 系列型号对比

### 4.1 命名规则

ESP32-S3-WROOM-1(NxRy):
- **模块系列**: ESP32-S3-WROOM-1 / ESP32-S3-WROOM-1U
- **Flash 温度等级**: H (高温) / N (常温)
- **Flash 大小 (MB)**: 4, 8, 16
- **PSRAM**: R (可选)
- **PSRAM 大小 (MB)**: 2, 8, 16

### 4.2 型号列表

| 型号 | Flash | PSRAM | 温度范围 | 天线 |
|------|-------|-------|---------|------|
| ESP32-S3-WROOM-1-N4 | 4 MB | - | -40~85°C | PCB |
| ESP32-S3-WROOM-1-N8 | 8 MB | - | -40~85°C | PCB |
| ESP32-S3-WROOM-1-N16 | 16 MB | - | -40~85°C | PCB |
| ESP32-S3-WROOM-1-N4R2 | 4 MB | 2 MB | -40~85°C | PCB |
| ESP32-S3-WROOM-1-N8R2 | 8 MB | 2 MB | -40~85°C | PCB |
| ESP32-S3-WROOM-1-N16R2 | 16 MB | 2 MB | -40~85°C | PCB |
| ESP32-S3-WROOM-1-N4R8 | 4 MB | 8 MB | -40~65°C | PCB |
| ESP32-S3-WROOM-1-N8R8 | 8 MB | 8 MB | -40~65°C | PCB |
| ESP32-S3-WROOM-1-N16R8 | 16 MB | 8 MB | -40~65°C | PCB |
| ESP32-S3-WROOM-1-N16R16VA | 16 MB | 16 MB | -40~65°C | PCB |
| ESP32-S3-WROOM-1U-N4 | 4 MB | - | -40~85°C | IPEX |
| ESP32-S3-WROOM-1U-N8 | 8 MB | - | -40~85°C | IPEX |
| ESP32-S3-WROOM-1U-N16 | 16 MB | - | -40~85°C | IPEX |
| ESP32-S3-WROOM-1U-N4R2 | 4 MB | 2 MB | -40~85°C | IPEX |
| ESP32-S3-WROOM-1U-N8R2 | 8 MB | 2 MB | -40~85°C | IPEX |
| ESP32-S3-WROOM-1U-N16R2 | 16 MB | 2 MB | -40~85°C | IPEX |
| ESP32-S3-WROOM-1U-N4R8 | 4 MB | 8 MB | -40~65°C | IPEX |
| ESP32-S3-WROOM-1U-N8R8 | 8 MB | 8 MB | -40~65°C | IPEX |
| ESP32-S3-WROOM-1U-N16R8 | 16 MB | 8 MB | -40~65°C | IPEX |

---

## 5. 外设概述

ESP32-S3 集成了丰富的外设：
- SPI
- LCD 接口
- Camera 接口
- UART (3 个)
- I2C (2 个)
- I2S (2 个)
- 远程控制 (RMT)
- 脉冲计数器 (PCNT)
- LED PWM (LEDC)
- USB Serial/JTAG
- MCPWM
- SD/MMC 主机控制器
- TWAI® 控制器 (CAN 2.0)
- ADC (2 个, 12-bit)
- 触摸传感器 (14 个通道)
- 温度传感器
- 全速 USB 2.0 OTG

---

## 6. 原理图

### 6.1 组件说明

- **晶振 (Y1)**: 40 MHz (±10ppm)，C1 和 C4 值随晶振选择而变化
- **RF 匹配**: L3, C5, C11, L2, C12 值随实际 PCB 变化，50Ω 阻抗控制
- **电阻 R4**: 值随实际 PCB 变化，初始建议值 24 nH
- **NC**: 无组件

### 6.2 Flash 存储器 (U2)

| 引脚 | 信号 | 描述 |
|------|------|------|
| 1 | /CS | SPICS0 |
| 2 | DO | SPIQ |
| 3 | /WP | SPIWP |
| 4 | GND | 接地 |
| 5 | DI | SPID |
| 6 | CLK | SPICLK |
| 7 | /HOLD | SPIHD |
| 8 | VDD | VDD_SPI |

---

## 7. 封装信息

### ESP32-S3-WROOM-1
- **尺寸**: 18 × 25.5 × 3.1 mm
- **天线**: 板载 PCB 天线
- **引脚数**: 41

### ESP32-S3-WROOM-1U
- **尺寸**: 18 × 19.2 × 3.2 mm
- **天线**: IPEX 外置天线连接器
- **引脚数**: 41

---

## 8. 认证

- RF 认证: 见证书
- 绿色认证: RoHS/REACH

---

## 9. 测试

- HTOL/HTSL/uHAST/TCT/ESD

---

## 10. 相关文档

- [ESP32-S3 Datasheet](https://documentation.espressif.com/esp32-s3_datasheet_en.html)
- [ESP32-S3 Technical Reference Manual](https://www.espressif.com/sites/default/files/documentation/esp32-s3_technical_reference_manual_en.pdf)
- [ESP32-S3 Hardware Design Guidelines](https://www.espressif.com/sites/default/files/documentation/esp32-s3_hardware_design_guidelines_en.pdf)
- [ESP32-S3-WROOM-2 Datasheet](https://documentation.espressif.com/esp32-s3-wroom-2_datasheet_en.html)
