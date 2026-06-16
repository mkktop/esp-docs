# ESP32-S3-MINI-1 & ESP32-S3-MINI-1U 技术规格书

> 本文档基于乐鑫官方技术规格书整理，适用于 ESP32-S3-MINI-1 和 ESP32-S3-MINI-1U 系列模组。
>
> 官方文档链接：https://www.espressif.com/documentation/esp32-s3-mini-1_mini-1u_datasheet_cn.pdf

---

## 目录

1. [模组概述与特性](#1-模组概述与特性)
2. [功能框图](#2-功能框图)
3. [管脚定义](#3-管脚定义)
4. [系列型号对比](#4-系列型号对比)
5. [外设概述](#5-外设概述)
6. [电气特性](#6-电气特性)
7. [射频特性](#7-射频特性)
8. [封装与尺寸信息](#8-封装与尺寸信息)
9. [模组原理图](#9-模组原理图)
10. [认证信息](#10-认证信息)
11. [参考设计资源](#11-参考设计资源)

---

## 1 模组概述与特性

ESP32-S3-MINI-1 和 ESP32-S3-MINI-1U 是乐鑫系统 (Espressif Systems) 推出的小尺寸通用型 Wi-Fi + 低功耗蓝牙 MCU 模组。两款模组内置 ESP32-S3 系列芯片，搭载 Xtensa 双核 32 位 LX7 处理器，功能强大，具有丰富的外设接口、超高的集成度和精简的设计，适合于可穿戴电子设备、智能家居等应用领域。

**核心亮点：**

- 小尺寸模组，集成 2.4 GHz Wi-Fi (802.11b/g/n) + 蓝牙 5 (Bluetooth LE)
- 内置 ESP32-S3 系列芯片，Xtensa 双核 32 位 LX7 处理器
- Flash 最大可选 8 MB，内置芯片可叠封 2 MB PSRAM
- 39 个 GPIO，丰富的外设
- 板载 PCB 天线或外部天线连接器

### 1.1 CPU 和片上存储器

| 参数 | 规格 |
| :--- | :--- |
| 处理器 | Xtensa 双核 32 位 LX7 微处理器 (支持单精度浮点运算单元) |
| 最高时钟频率 | 240 MHz |
| ROM | 384 KB |
| SRAM | 512 KB |
| RTC SRAM | 16 KB |
| Flash | 最大 8 MB Quad SPI flash |
| PSRAM | 2 MB 嵌入式 PSRAM（仅 ESP32-S3FH4R2 芯片） |

### 1.2 Wi-Fi 特性

| 参数 | 规格 |
| :--- | :--- |
| 标准 | IEEE 802.11b/g/n |
| 最大数据速率 | 150 Mbps (802.11n 模式) |
| 帧聚合 | TX/RX A-MPDU, TX/RX A-MSDU |
| 保护间隔 | 0.4 us |
| 工作信道中心频率范围 | 2412 ~ 2484 MHz |
| 安全协议 | WPA/WPA2/WPA2-Enterprise/WPA3/WPA3-Enterprise |

### 1.3 蓝牙特性

| 参数 | 规格 |
| :--- | :--- |
| 类型 | 低功耗蓝牙 (Bluetooth LE) |
| 版本 | Bluetooth 5、Bluetooth mesh |
| 速率支持 | 125 Kbps、500 Kbps、1 Mbps、2 Mbps |
| 广播扩展 | Advertising Extensions |
| 多广播 | Multiple Advertisement Sets |
| 信道选择 | Channel Selection Algorithm #2 |
| 共存 | Wi-Fi 与蓝牙共存，共用同一个天线 |

### 1.4 模组集成元件

- 40 MHz 集成晶振

### 1.5 天线选型

- **ESP32-S3-MINI-1**：板载 PCB 天线
- **ESP32-S3-MINI-1U**：通过连接器连接外部天线（IPEX 连接器）

### 1.6 工作条件

| 参数 | 最小值 | 典型值 | 最大值 | 单位 |
| :--- | :---: | :---: | :---: | :---: |
| 工作电压/供电电压 | 3.0 | 3.3 | 3.6 | V |
| 工作环境温度 | -40 | - | 85 | C |

### 1.7 认证

- RF 认证：详见官方认证证书
- 环保认证：RoHS / REACH
- 可靠性测试：HTOL / HTSL / uHAST / TCT / ESD

### 1.8 应用场景

- 智能家居
- 工业自动化
- 医疗保健
- 消费电子产品
- 智慧农业
- POS 机
- 服务机器人
- 音频设备
- 通用低功耗 IoT 传感器集线器
- 通用低功耗 IoT 数据记录器
- 摄像头视频流传输
- USB 设备
- 语音识别 / 图像识别
- Wi-Fi + 蓝牙网卡
- 触摸和接近感应

---

## 2 功能框图

### 2.1 ESP32-S3-MINI-1 功能框图

```
                         +-----------------------------+
                         |      ESP32-S3-MINI-1        |
                         |                             |
   3V3, EN, GPIOs <----->|  ESP32-S3FN8 /              |
                         |  ESP32-S3FH4R2              |
                         |      |                      |
                         |      +---> FLASH (QSPI)     |
                         |      +---> PSRAM (opt. QSPI)|
                         |      |                      |
                         |  40 MHz Crystal              |
                         |      |                      |
                         |  RF Matching ---> PCB Antenna|
                         +-----------------------------+
```

### 2.2 ESP32-S3-MINI-1U 功能框图

```
                         +-----------------------------+
                         |      ESP32-S3-MINI-1U       |
                         |                             |
   3V3, EN, GPIOs <----->|  ESP32-S3FN8 /              |
                         |  ESP32-S3FH4R2              |
                         |      |                      |
                         |      +---> FLASH (QSPI)     |
                         |      +---> PSRAM (opt. QSPI)|
                         |      |                      |
                         |  40 MHz Crystal              |
                         |      |                      |
                         |  RF Matching ---> Antenna   |
                         |                   Connector |
                         +-----------------------------+
```

**说明：**
- 核心组件为 ESP32-S3FN8（8 MB Flash 无 PSRAM）或 ESP32-S3FH4R2（4 MB Flash + 2 MB PSRAM）
- 存储通过 Quad SPI 总线连接
- ESP32-S3-MINI-1 使用板载 PCB 天线，ESP32-S3-MINI-1U 使用外部天线连接器
- 关于芯片与封装内 flash/PSRAM 的管脚对应关系，请参考《ESP32-S3 系列芯片技术规格书》

---

## 3 管脚定义

### 3.1 管脚布局

模组共有 65 个管脚，包括四周的信号管脚和中心底部的接地焊盘。

**说明：**
- ESP32-S3-MINI-1U 的管脚布局与 ESP32-S3-MINI-1 相同，但没有天线净空区。
- 关于底板上模组天线净空区的更多信息，请查看《ESP32-S3 硬件设计指南》。

### 3.2 管脚描述

**表 3-1. 管脚定义（65 个管脚）**

| 名称 | 序号 | 类型 | 功能 |
| :--- | :---: | :--- | :--- |
| GND | 1 | P | 接地 |
| GND | 2 | P | 接地 |
| 3V3 | 3 | P | 供电 (3.0 ~ 3.6 V) |
| IO0 | 4 | I/O/T | RTC_GPIO0, GPIO0 |
| IO1 | 5 | I/O/T | RTC_GPIO1, GPIO1, TOUCH1, ADC1_CH0 |
| IO2 | 6 | I/O/T | RTC_GPIO2, GPIO2, TOUCH2, ADC1_CH1 |
| IO3 | 7 | I/O/T | RTC_GPIO3, GPIO3, TOUCH3, ADC1_CH2 |
| IO4 | 8 | I/O/T | RTC_GPIO4, GPIO4, TOUCH4, ADC1_CH3 |
| IO5 | 9 | I/O/T | RTC_GPIO5, GPIO5, TOUCH5, ADC1_CH4 |
| IO6 | 10 | I/O/T | RTC_GPIO6, GPIO6, TOUCH6, ADC1_CH5 |
| IO7 | 11 | I/O/T | RTC_GPIO7, GPIO7, TOUCH7, ADC1_CH6 |
| IO8 | 12 | I/O/T | RTC_GPIO8, GPIO8, TOUCH8, ADC1_CH7, SUBSPICS1 |
| IO9 | 13 | I/O/T | RTC_GPIO9, GPIO9, TOUCH9, ADC1_CH8, FSPIHD, SUBSPIHD |
| IO10 | 14 | I/O/T | RTC_GPIO10, GPIO10, TOUCH10, ADC1_CH9, FSPICS0, FSPIIO4, SUBSPICS0 |
| IO11 | 15 | I/O/T | RTC_GPIO11, GPIO11, TOUCH11, ADC2_CH0, FSPID, FSPIIO5, SUBSPID |
| IO12 | 16 | I/O/T | RTC_GPIO12, GPIO12, TOUCH12, ADC2_CH1, FSPICLK, FSPIIO6, SUBSPICLK |
| IO13 | 17 | I/O/T | RTC_GPIO13, GPIO13, TOUCH13, ADC2_CH2, FSPIQ, FSPIIO7, SUBSPIQ |
| IO14 | 18 | I/O/T | RTC_GPIO14, GPIO14, TOUCH14, ADC2_CH3, FSPIWP, FSPIDQS, SUBSPIWP |
| IO15 | 19 | I/O/T | RTC_GPIO15, GPIO15, U0RTS, ADC2_CH4, XTAL_32K_P |
| IO16 | 20 | I/O/T | RTC_GPIO16, GPIO16, U0CTS, ADC2_CH5, XTAL_32K_N |
| IO17 | 21 | I/O/T | RTC_GPIO17, GPIO17, U1TXD, ADC2_CH6 |
| IO18 | 22 | I/O/T | RTC_GPIO18, GPIO18, U1RXD, ADC2_CH7, CLK_OUT3 |
| IO19 | 23 | I/O/T | RTC_GPIO19, GPIO19, U1RTS, ADC2_CH8, CLK_OUT2, USB_D- |
| IO20 | 24 | I/O/T | RTC_GPIO20, GPIO20, U1CTS, ADC2_CH9, CLK_OUT1, USB_D+ |
| IO21 | 25 | I/O/T | RTC_GPIO21, GPIO21 |
| IO26 | 26 | I/O/T | SPICS1, GPIO26 |
| IO47 | 27 | I/O/T | SPICLK_P_DIFF, GPIO47, SUBSPICLK_P_DIFF |
| IO33 | 28 | I/O/T | SPIIO4, GPIO33, FSPIHD, SUBSPIHD |
| IO34 | 29 | I/O/T | SPIIO5, GPIO34, FSPICS0, SUBSPICS0 |
| IO48 | 30 | I/O/T | SPICLK_N_DIFF, GPIO48, SUBSPICLK_N_DIFF |
| IO35 | 31 | I/O/T | SPIIO6, GPIO35, FSPID, SUBSPID |
| IO36 | 32 | I/O/T | SPIIO7, GPIO36, FSPICLK, SUBSPICLK |
| IO37 | 33 | I/O/T | SPIDQS, GPIO37, FSPIQ, SUBSPIQ |
| IO38 | 34 | I/O/T | GPIO38, FSPIWP, SUBSPIWP |
| IO39 | 35 | I/O/T | MTCK, GPIO39, CLK_OUT3, SUBSPICS1 |
| IO40 | 36 | I/O/T | MTDO, GPIO40, CLK_OUT2 |
| IO41 | 37 | I/O/T | MTDI, GPIO41, CLK_OUT1 |
| IO42 | 38 | I/O/T | MTMS, GPIO42 |
| TXD0 | 39 | I/O/T | U0TXD, GPIO43, CLK_OUT1 |
| RXD0 | 40 | I/O/T | U0RXD, GPIO44, CLK_OUT2 |
| IO45 | 41 | I/O/T | GPIO45 |
| GND | 42 | P | 接地 |
| GND | 43 | P | 接地 |
| IO46 | 44 | I/O/T | GPIO46 |
| EN | 45 | I | 高电平：芯片使能；低电平：芯片关闭 |
| GND | 46-60 | P | 接地 |
| EPAD | 61 | P | 底部焊盘（接地） |
| GND | 62-65 | P | 接地 |

**管脚类型说明：**
- P：电源
- I：输入
- O：输出
- T：可设置为高阻

**重要说明：**
- IO26 的说明：物料编号以 -N4R2 结尾的模组，IO26 用于连接至嵌入式 PSRAM，不可用于其他功能。
- 管脚 28 ~ 29、31 ~ 33 的默认功能由 eFuse 位决定。
- EN 管脚不能浮空。
- GPIO0、GPIO3、GPIO45、GPIO46 为 Strapping 管脚。
- GPIO19 和 GPIO20 默认用于 USB 串口/JTAG 接口。
- GPIO26 ~ GPIO37 通常用于 SPI flash 和 PSRAM，不推荐用于其他用途。

### 3.3 ADC 管脚映射

| GPIO | ADC 通道 |
| :--- | :--- |
| GPIO1 | ADC1_CH0 |
| GPIO2 | ADC1_CH1 |
| GPIO3 | ADC1_CH2 |
| GPIO4 | ADC1_CH3 |
| GPIO5 | ADC1_CH4 |
| GPIO6 | ADC1_CH5 |
| GPIO7 | ADC1_CH6 |
| GPIO8 | ADC1_CH7 |
| GPIO9 | ADC1_CH8 |
| GPIO10 | ADC1_CH9 |
| GPIO11 | ADC2_CH0 |
| GPIO12 | ADC2_CH1 |
| GPIO13 | ADC2_CH2 |
| GPIO14 | ADC2_CH3 |
| GPIO15 | ADC2_CH4 |
| GPIO16 | ADC2_CH5 |
| GPIO17 | ADC2_CH6 |
| GPIO18 | ADC2_CH7 |
| GPIO19 | ADC2_CH8 |
| GPIO20 | ADC2_CH9 |

### 3.4 触摸传感器管脚映射

| GPIO | 触摸通道 |
| :--- | :--- |
| GPIO1 | TOUCH1 |
| GPIO2 | TOUCH2 |
| GPIO3 | TOUCH3 |
| GPIO4 | TOUCH4 |
| GPIO5 | TOUCH5 |
| GPIO6 | TOUCH6 |
| GPIO7 | TOUCH7 |
| GPIO8 | TOUCH8 |
| GPIO9 | TOUCH9 |
| GPIO10 | TOUCH10 |
| GPIO11 | TOUCH11 |
| GPIO12 | TOUCH12 |
| GPIO13 | TOUCH13 |
| GPIO14 | TOUCH14 |

---

## 4 系列型号对比

### 4.1 ESP32-S3-MINI-1 系列型号对比

ESP32-S3-MINI-1 和 ESP32-S3-MINI-1U 均有两种变型，物料编号分别以 -N8 和 -N4R2 结尾，两种变型仅 flash 和 PSRAM 型号不同。

**表 4-1. 系列型号对比**

| 物料编号 | 天线类型 | Flash | PSRAM | 内置芯片 | 环境温度 (C) | 模组尺寸 (mm) |
| :--- | :--- | :--- | :--- | :--- | :---: | :--- |
| ESP32-S3-MINI-1-N8 | PCB 板载天线 | 8 MB (Quad SPI) | - | ESP32-S3FN8 | -40 ~ 85 | 15.4 x 20.5 x 2.4 |
| ESP32-S3-MINI-1-N4R2 | PCB 板载天线 | 4 MB (Quad SPI) | 2 MB (Quad SPI) | ESP32-S3FH4R2 | -40 ~ 85 | 15.4 x 20.5 x 2.4 |
| ESP32-S3-MINI-1U-N8 | 外部天线连接器 | 8 MB (Quad SPI) | - | ESP32-S3FN8 | -40 ~ 85 | 15.4 x 15.4 x 2.4 |
| ESP32-S3-MINI-1U-N4R2 | 外部天线连接器 | 4 MB (Quad SPI) | 2 MB (Quad SPI) | ESP32-S3FH4R2 | -40 ~ 85 | 15.4 x 15.4 x 2.4 |

**说明：**
1. 更多关于存储器规格的信息，请参考章节 6.5 存储器规格。
2. 默认情况下，模组 SPI flash 支持的最大时钟频率为 80 MHz，且不支持自动暂停功能。如需使用 120 MHz 的 flash 时钟频率或需要 flash 自动暂停功能，请联系乐鑫。
3. 环境温度指乐鑫模组外部的推荐环境温度。

### 4.2 ESP32-S3 系列模组对比

| 模组名称 | 内置芯片 | 尺寸 (mm) | GPIO | Flash | PSRAM | 天线类型 |
| :--- | :--- | :--- | :---: | :--- | :--- | :--- |
| ESP32-S3-MINI-1 | ESP32-S3FN8 / ESP32-S3FH4R2 | 15.4 x 20.5 x 2.4 | 39 | 8 MB / 4 MB | 0 / 2 MB | PCB 天线 |
| ESP32-S3-MINI-1U | ESP32-S3FN8 / ESP32-S3FH4R2 | 15.4 x 15.4 x 2.4 | 39 | 8 MB / 4 MB | 0 / 2 MB | 外部天线连接器 |
| ESP32-S3-WROOM-1 | ESP32-S3R8 / ESP32-S3FH4R2 | 18.0 x 25.5 x 3.2 | 39 | 8 MB / 4 MB | 0 / 2 MB | PCB 天线 |
| ESP32-S3-WROOM-1U | ESP32-S3R8 / ESP32-S3FH4R2 | 18.0 x 20.0 x 3.2 | 39 | 8 MB / 4 MB | 0 / 2 MB | 外部天线连接器 |
| ESP32-S3-WROOM-2 | ESP32-S3R16V / ESP32-S3R8V | 18.0 x 25.5 x 3.1 | 33 | 32 MB | 16 / 8 MB | PCB 天线 |

---

## 5 外设概述

### 5.1 外设接口列表

ESP32-S3-MINI-1 / MINI-1U 模组提供以下丰富的外设接口：

| 外设接口 | 说明 |
| :--- | :--- |
| GPIO | 39 个通用输入输出管脚，其中 4 个作为 strapping 管脚 |
| SPI | SPI 接口，支持连接外部 flash、PSRAM 及其他 SPI 设备 |
| LCD 接口 | 支持并行和串行 LCD 显示屏 |
| Camera 接口 | 支持 DVP 并行摄像头接口 |
| UART | 3 个 UART 接口 (UART0, UART1, UART2) |
| I2C | 2 个 I2C 接口，支持主机和从机模式 |
| I2S | 2 个 I2S 接口，支持音频数据传输 |
| 红外遥控 | 支持红外收发 |
| 脉冲计数器 (PCNT) | 脉冲计数功能 |
| LED PWM | 8 个通道的 LED PWM 控制 |
| USB 2.0 OTG | 全速 USB 2.0 OTG 接口 |
| USB 串口/JTAG | USB 串口/JTAG 控制器 |
| MCPWM | 电机控制 PWM |
| SD/MMC 主机控制器 | 支持 SD 卡和 MMC 存储卡 |
| GDMA | 通用 DMA 控制器 |
| TWAI | 兼容 ISO 11898-1 (CAN 规范 2.0) |
| ADC | 2 个 ADC 模块 (ADC1: 10 通道, ADC2: 10 通道) |
| 触摸传感器 | 14 个电容式触摸传感通道 |
| 温度传感器 | 片上温度传感器 |
| 定时器 | 4 个 64 位通用定时器 |
| 看门狗 | 3 个看门狗定时器 (1 个 RTC, 2 个 MWDT) |

### 5.2 外设管脚分配说明

ESP32-S3 通过 GPIO 交换矩阵和 IO MUX 可将外设信号灵活映射到任意 GPIO 管脚。管脚分配按优先级分为：

- **优先级 1 (P1)**：固定管脚，通过 IO MUX 或 RTC IO MUX 直接连接外设信号
- **优先级 2 (P2)**：GPIO 管脚，没有限制，可以自由分配
- **优先级 3 (P3)**：GPIO 管脚，使用时需要注意与重要功能是否冲突
- **优先级 4 (P4)**：已分配或不推荐使用的 GPIO 管脚

---

## 6 电气特性

### 6.1 绝对最大额定值

超出绝对最大额定值可能导致器件永久性损坏。

| 符号 | 参数 | 最小值 | 最大值 | 单位 |
| :--- | :--- | :---: | :---: | :---: |
| VDD33 | 电源管脚电压 | -0.3 | 3.6 | V |
| TSTORE | 存储温度 | -40 | 105 | C |

### 6.2 建议工作条件

| 符号 | 参数 | 最小值 | 典型值 | 最大值 | 单位 |
| :--- | :--- | :---: | :---: | :---: | :---: |
| VDD33 | 电源管脚电压 | 3.0 | 3.3 | 3.6 | V |
| IVDD | 外部电源的供电电流 | 0.5 | - | - | A |
| TA | 工作环境温度 | -40 | - | 85 | C |

### 6.3 直流电气特性 (3.3 V, 25 C)

| 参数 | 说明 | 最小值 | 典型值 | 最大值 | 单位 |
| :--- | :--- | :---: | :---: | :---: | :---: |
| CIN | 管脚电容 | - | 2 | - | pF |
| VIH | 高电平输入电压 | 0.75 x VDD | - | VDD + 0.3 | V |
| VIL | 低电平输入电压 | -0.3 | - | 0.25 x VDD | V |
| IIH | 高电平输入电流 | - | - | 50 | nA |
| IIL | 低电平输入电流 | - | - | 50 | nA |
| VOH | 高电平输出电压 | 0.8 x VDD | - | - | V |
| VOL | 低电平输出电压 | - | - | 0.1 x VDD | V |
| IOH | 高电平拉电流 (VDD=3.3V, VOH>=2.64V, PAD_DRIVER=3) | - | 40 | - | mA |
| IOL | 低电平灌电流 (VDD=3.3V, VOL=0.495V, PAD_DRIVER=3) | - | 28 | - | mA |
| RPU | 内部弱上拉电阻 | - | 45 | - | kohm |
| RPD | 内部弱下拉电阻 | - | 45 | - | kohm |
| VIH_nRST | 芯片复位释放电压 (EN 管脚) | 0.75 x VDD | - | VDD + 0.3 | V |
| VIL_nRST | 芯片复位电压 (EN 管脚) | -0.3 | - | 0.25 x VDD | V |

### 6.4 功耗特性

#### 6.4.1 Active 模式下的功耗

以下功耗数据基于 3.3 V 供电电源、25 C 环境温度条件测得。所有发射功耗数据基于 100% 占空比测得。

**Wi-Fi 功耗：**

| 工作模式 | 射频模式 | 描述 | 峰值 (mA) |
| :--- | :--- | :--- | :---: |
| Active | TX | 802.11b, 1 Mbps, @20.5 dBm | 355 |
| Active | TX | 802.11g, 54 Mbps, @18 dBm | 297 |
| Active | TX | 802.11n, HT20, MCS7, @17.5 dBm | 286 |
| Active | TX | 802.11n, HT40, MCS7, @17.5 dBm | 285 |
| Active | RX | 802.11b/g/n, HT20 | 95 |
| Active | RX | 802.11n, HT40 | 97 |

**低功耗蓝牙功耗：**

| 工作模式 | 射频模式 | 描述 | 峰值 (mA) |
| :--- | :--- | :--- | :---: |
| Active | TX | 低功耗蓝牙 @ 20.0 dBm | 340 |
| Active | TX | 低功耗蓝牙 @ 9.0 dBm | 204 |
| Active | TX | 低功耗蓝牙 @ 0 dBm | 189 |
| Active | TX | 低功耗蓝牙 @ -15.0 dBm | 118 |
| Active | RX | 低功耗蓝牙 | 93 |

#### 6.4.2 其他功耗模式下的功耗

**Modem-sleep 模式：**

| 频率 (MHz) | 说明 | 典型值1 (mA) | 典型值2 (mA) |
| :---: | :--- | :---: | :---: |
| 40 | WAITI（双核均空闲） | 13.2 | 18.8 |
| 40 | 单核执行 32 位数据访问指令，另一个核空闲 | 16.2 | 21.8 |
| 40 | 双核执行 32 位数据访问指令 | 18.7 | 24.4 |
| 80 | WAITI | 22.0 | 36.1 |
| 80 | 双核执行 32 位数据访问指令 | 33.1 | 47.3 |
| 160 | WAITI | 27.6 | 42.3 |
| 160 | 双核执行 32 位数据访问指令 | 49.6 | 64.1 |
| 240 | WAITI | 32.9 | 47.6 |
| 240 | 双核执行 32 位数据访问指令 | 66.2 | 81.3 |
| 240 | 双核执行 128 位数据访问指令 | 91.7 | 107.9 |

> 1. 所有外设时钟关闭时的典型值。
> 2. 所有外设时钟打开时的典型值。

**低功耗模式：**

| 工作模式 | 说明 | 典型值 |
| :--- | :--- | :---: |
| Light-sleep | VDD_SPI 和 Wi-Fi 掉电，所有 GPIO 设置为高阻状态 | 240 uA |
| Deep-sleep | ULP 协处理器处于工作状态 (ULP-FSM) | 170 uA |
| Deep-sleep | ULP 协处理器处于工作状态 (ULP-RISC-V) | 190 uA |
| Deep-sleep | 超低功耗传感器监测模式 | 18 uA |
| Deep-sleep | RTC 存储器和 RTC 外设上电 | 8 uA |
| Deep-sleep | RTC 存储器上电，RTC 外设掉电 | 7 uA |
| 关闭 | EN 管脚拉低，芯片关闭 | 1 uA |

### 6.5 存储器规格

以下数据来源于存储器供应商的数据手册。

**Flash 存储器规格：**

| 参数 | 说明 | 最小值 | 典型值 | 最大值 | 单位 |
| :--- | :--- | :---: | :---: | :---: | :---: |
| VCC | 电源电压 (1.8 V) | 1.65 | 1.80 | 2.00 | V |
| VCC | 电源电压 (3.3 V) | 2.7 | 3.3 | 3.6 | V |
| FC | 最大时钟频率 | 80 | - | - | MHz |
| - | 编程/擦除周期 | 100,000 | - | - | 次 |
| TRET | 数据保留时间 | 20 | - | - | 年 |
| TPP | 页编程时间 | - | 0.8 | 5 | ms |
| TSE | 扇区擦除时间 (4 KB) | - | 70 | 500 | ms |
| TBE1 | 块擦除时间 (32 KB) | - | 0.2 | 2 | s |
| TBE2 | 块擦除时间 (64 KB) | - | 0.3 | 3 | s |

**PSRAM 存储器规格：**

| 参数 | 说明 | 最小值 | 典型值 | 最大值 | 单位 |
| :--- | :--- | :---: | :---: | :---: | :---: |
| VCC | 电源电压 (1.8 V) | 1.62 | 1.80 | 1.98 | V |
| VCC | 电源电压 (3.3 V) | 2.7 | 3.3 | 3.6 | V |
| FC | 最大时钟频率 | 80 | - | - | MHz |

---

## 7 射频特性

射频数据是在天线端口处连接射频线后测试所得，包含了射频前端电路带来的损耗。除非特别说明，射频测试均是在 3.3 V (+/-5%) 供电电源、25 C 环境温度的条件下完成。

### 7.1 Wi-Fi 射频特性

| 参数 | 规格 |
| :--- | :--- |
| 工作信道中心频率范围 | 2412 ~ 2484 MHz |
| 无线标准 | IEEE 802.11b/g/n |

#### 7.1.1 Wi-Fi 发射器 (TX) 特性

| 速率 | 典型值 (dBm) |
| :--- | :---: |
| 802.11b, 1 Mbps | 20.5 |
| 802.11b, 11 Mbps | 20.5 |
| 802.11g, 6 Mbps | 20.0 |
| 802.11g, 54 Mbps | 18.0 |
| 802.11n, HT20, MCS 0 | 19.0 |
| 802.11n, HT20, MCS 7 | 17.5 |
| 802.11n, HT40, MCS 0 | 18.5 |
| 802.11n, HT40, MCS 7 | 17.0 |

**发射 EVM 特性：**

| 速率 | 典型值 (dB) | 标准限值 (dB) |
| :--- | :---: | :---: |
| 802.11b, 1 Mbps, @20.5 dBm | -24.5 | -10 |
| 802.11b, 11 Mbps, @20.5 dBm | -24.5 | -10 |
| 802.11g, 6 Mbps, @20 dBm | -23.0 | -5 |
| 802.11g, 54 Mbps, @18 dBm | -29.5 | -25 |
| 802.11n, HT20, MCS 0, @19 dBm | -24.0 | -5 |
| 802.11n, HT20, MCS 7, @17.5 dBm | -30.5 | -27 |
| 802.11n, HT40, MCS 0, @18.5 dBm | -25.0 | -5 |
| 802.11n, HT40, MCS 7, @17 dBm | -30.0 | -27 |

#### 7.1.2 Wi-Fi 接收器 (RX) 特性

802.11b 标准下的误包率 (PER) 不超过 8%，802.11g/n 标准下不超过 10%。

| 速率 | 典型灵敏度 (dBm) | 典型最大输入信号 (dBm) | 典型邻信道抑制 (dB) |
| :--- | :---: | :---: | :---: |
| 802.11b, 1 Mbps | -98.2 | 5 | 35 |
| 802.11b, 2 Mbps | -95.6 | 5 | 35 |
| 802.11b, 5.5 Mbps | -92.8 | 5 | - |
| 802.11b, 11 Mbps | -88.5 | 5 | 35 |
| 802.11g, 6 Mbps | -93.0 | 5 | 31 |
| 802.11g, 54 Mbps | -76.2 | 0 | 14 |
| 802.11n, HT20, MCS 0 | -93.0 | 5 | 31 |
| 802.11n, HT20, MCS 7 | -74.2 | 0 | 13 |
| 802.11n, HT40, MCS 0 | -90.0 | 5 | 19 |
| 802.11n, HT40, MCS 7 | -71.2 | 0 | 8 |

### 7.2 低功耗蓝牙射频特性

#### 7.2.1 低功耗蓝牙发射器 (TX) 特性

| 参数 | 1 Mbps | 2 Mbps | Coded (S=2) | Coded (S=8) |
| :--- | :---: | :---: | :---: | :---: |
| 典型输出功率 (dBm) | 20.0 | 20.0 | 20.0 | 20.0 |
| 典型 EVM (dB) | -17.5 | -17.5 | -17.5 | -17.5 |

#### 7.2.2 低功耗蓝牙接收器 (RX) 特性

| 参数 | 1 Mbps | 2 Mbps | Coded (S=2) | Coded (S=8) |
| :--- | :---: | :---: | :---: | :---: |
| 灵敏度 (dBm) | -96.5 | -92 | - | -103.5 / -100 |
| 最大接收信号 (dBm) | 8 | 3 | - | 8 |
| 共信道抑制比 C/I (dB) | 8 | 8 | - | 4 |
| 带外阻塞 30~2000 MHz (dBm) | -12 | -15 | - | - |
| 带外阻塞 2003~2399 MHz (dBm) | -18 | -21 | - | - |
| 带外阻塞 2484~2997 MHz (dBm) | -16 | -21 | - | - |
| 带外阻塞 3000~12750 MHz (dBm) | -10 | -9 | - | - |
| 互调 (dBm) | -29 | -29 | - | - |

---

## 8 封装与尺寸信息

### 8.1 模组尺寸

**ESP32-S3-MINI-1 (PCB 天线版本)：**

| 参数 | 数值 |
| :--- | :--- |
| 长度 | 20.5 mm |
| 宽度 | 15.4 mm |
| 高度 | 2.4 mm |
| 管脚数 | 65 |

**ESP32-S3-MINI-1U (外部天线版本)：**

| 参数 | 数值 |
| :--- | :--- |
| 长度 | 15.4 mm |
| 宽度 | 15.4 mm |
| 高度 | 2.4 mm |
| 管脚数 | 65 |

### 8.2 外部天线连接器尺寸 (仅 MINI-1U)

ESP32-S3-MINI-1U 采用第三代外部天线连接器，该连接器兼容：
- 广濑 (Hirose) 的 W.FL 系列连接器
- I-PEX 的 MHF III 连接器
- 安费诺 (Amphenol) 的 AMMC 连接器

ESP32-S3-MINI-1U 在认证测试过程中搭配使用的外部天线为第三代外接单极子天线，料号为 TFPD08H10060011。

**外部天线选用建议：**
- 2.4 GHz 频段
- 50 ohm 阻抗
- 最大增益不超过认证中所用天线的增益 2.33 dBi
- 接口规格与模组天线连接器接口匹配

### 8.3 PCB 布局建议

- 推荐 PCB 封装图源文件可用于测量图中未标注的尺寸
- ESP32-S3-MINI-1 和 ESP32-S3-MINI-1U 均提供 3D 模型 (.STEP 格式)
- 如产品采用模组进行 on-board 设计，需注意考虑模组在底板的布局，应尽可能减小底板对模组 PCB 天线性能的影响
- 关于 PCB 设计中模组位置摆放的更多信息，请参考《ESP32-S3 硬件设计指南》

---

## 9 模组原理图

### 9.1 ESP32-S3-MINI-1 原理图要点

**主要外围元件：**

| 元件 | 说明 |
| :--- | :--- |
| Y1 | 40 MHz 晶振 (精度 +/-10ppm) |
| D1 | ESD 保护器件 |
| R1 | 10K 上拉电阻 (NC, 未安装) |
| R3 | 499 ohm 串联电阻 (U0TXD 线路) |
| R4 | 阻抗匹配元件（初始建议值 24 nH） |
| L1 | 2.0 nH (+/-0.1nH) 电感 |
| L3, C5 | RF 匹配网络元件 |
| C1, C4 | 晶振负载电容（值随晶振选择而变化） |
| C2 | 10 nF 滤波电容 |
| C3 | 1 uF 滤波电容 |
| C6 | 10 uF 滤波电容 |
| C7 | 1 uF 滤波电容 |
| C8, C9, C10, C13, C15 | 0.1 uF 滤波电容 |
| C11, L2, C12 | RF 匹配网络元件（值随 PCB 板实际调整） |
| C14 | 1 uF 滤波电容 |

**设计注意事项：**
- 50 ohm 阻抗控制
- C1 和 C4 的值随晶振选择而变化
- R4 可以是电阻或电感，初始建议值为 24 nH
- L3, C5, C11, L2, C12 的值随实际 PCB 板调整
- NC 表示未安装元件

### 9.2 ESP32-S3-MINI-1U 原理图要点

ESP32-S3-MINI-1U 的内部电路与 MINI-1 基本相同，主要区别在于天线部分：
- MINI-1 使用板载 PCB 天线
- MINI-1U 使用外部天线连接器替代板载天线

---

## 10 认证信息

ESP32-S3-MINI-1 和 ESP32-S3-MINI-1U 已通过以下主要认证：

### 10.1 CE 认证 (欧盟)

| 项目 | 详情 |
| :--- | :--- |
| 认证编号 | ESP32-S3-MINI-1: 0370-RED-4972 / ESP32-S3-MINI-1U: 0370-RED-5007 |
| 指令 | Radio Equipment Directive 2014/53/EU |
| Wi-Fi 频率范围 | 2412 ~ 2472 MHz |
| Wi-Fi 最大输出功率 | 约 20 dBm |
| BLE 频率范围 | 2402 ~ 2480 MHz |
| BLE 最大输出功率 | 约 10 dBm |

### 10.2 TELEC 认证 (日本)

| 项目 | 详情 |
| :--- | :--- |
| 认证编号 | 201-230385 |
| 签发日期 | 2023 年 6 月 13 日 |
| 天线规格 | PCB 天线，最大增益 4.54 dBi (2.4 GHz) |

### 10.3 IC 认证 (加拿大)

| 项目 | 详情 |
| :--- | :--- |
| ISED 认证编号 | 21098-ESPS3MINI1 |
| 签发日期 | 2024 年 8 月 16 日 |
| 天线类型 | PCB 天线 |

### 10.4 环保认证

- RoHS 认证
- REACH 认证

### 10.5 可靠性测试

模组通过了以下可靠性测试：
- HTOL (High Temperature Operating Life)
- HTSL (High Temperature Storage Life)
- uHAST (Unbiased Highly Accelerated Stress Test)
- TCT (Temperature Cycling Test)
- ESD (Electrostatic Discharge)

---

## 11 参考设计资源

### 11.1 相关开发板

| 开发板 | 说明 |
| :--- | :--- |
| ESP32-S3-DevKitM-1 | 基于 ESP32-S3-MINI-1 的开发板 |
| ESP32-S3-USB-OTG | 基于 ESP32-S3-MINI-1 的 USB OTG 开发板 |
| ESP32-S3-USB-Bridge | 基于 ESP32-S3-MINI-1 的 USB Bridge 开发板 |

### 11.2 设计文件

| 资源 | 说明 |
| :--- | :--- |
| PCB 封装图 (.dxf) | 推荐 PCB 封装图源文件 |
| 3D 模型 (.STEP / .glb) | 模组 3D 模型文件 |
| KiCad 符号和封装 | Espressif 官方 KiCad 库支持 |
| 原理图 | 模组内部电路原理图 |

### 11.3 参考文档

| 文档 | 说明 |
| :--- | :--- |
| ESP32-S3 系列芯片技术规格书 | 芯片详细技术规格 |
| ESP32-S3 硬件设计指南 | 硬件设计参考 |
| ESP32-S3 技术参考手册 | 寄存器级别参考 |
| ESP32-S3 系列芯片勘误表 | 芯片版本标识及已知问题 |
| ESP 射频测试指南 | 射频测试配置指南 |

### 11.4 采购渠道

ESP32-S3-MINI-1 和 ESP32-S3-MINI-1U 系列模组可通过以下渠道采购：

| 渠道 | 网址 |
| :--- | :--- |
| 淘宝 | https://item.taobao.com/item.htm?id=660710967803 |
| Digi-Key | https://www.digikey.com |
| Mouser | https://www.mouser.com |
| AliExpress | https://www.aliexpress.com |

**采购信息：**

| 参数 | 数值 |
| :--- | :--- |
| 最小包装数量 (MPQ/SPQ) | 650 |
| 最小起订量 (MOQ) | 3250 |
| 交货周期 | 约 8 周 |
| 发布日期 | 2021 年 9 月 23 日 |

---

> **免责声明：** 本文档基于乐鑫官方技术规格书整理，仅供参考。设计产品时请以最新版官方技术规格书为准。乐鑫保留随时更改产品规格的权利。
>
> 官方文档地址：https://documentation.espressif.com/esp32-s3-mini-1_mini-1u_datasheet_cn.html
