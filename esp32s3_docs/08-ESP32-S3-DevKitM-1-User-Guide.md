# ESP32-S3-DevKitM-1 用户指南

> **来源**: [乐鑫官方文档 - ESP32-S3-DevKitM-1 用户指南](https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32s3/esp32-s3-devkitm-1/user_guide.html)

---

## 目录

1. [开发板概述](#1-开发板概述)
2. [硬件描述](#2-硬件描述)
3. [管脚布局与管脚描述](#3-管脚布局与管脚描述)
4. [电源供电信息](#4-电源供电信息)
5. [入门指南](#5-入门指南)
6. [相关文档](#6-相关文档)

---

## 1. 开发板概述

### 1.1 简介

ESP32-S3-DevKitM-1 是乐鑫（Espressif）推出的一款**入门级开发板**，搭载的是 **Wi-Fi + 蓝牙 LE** 模组 **ESP32-S3-MINI-1** 或 **ESP32-S3-MINI-1U**，该款模组因小尺寸而得名。

板上模组的大部分管脚均已引出至开发板两侧排针，开发人员可根据实际需求，轻松通过跳线连接多种外围设备，也可将开发板插在面包板上使用。

### 1.2 产品状态说明

> **注意**: 由于 ESP32-S3-DevKitM-1 开发板与 ESP32-S3-DevKitC-1-N8R8 和 ESP32-S3-DevKitC-1U-N8R8 在功能和用途上基本一致，ESP32-S3-DevKitM-1 **已停止生产（EOL）**。开发者可使用 ESP32-S3-DevKitC-1-N8R8 或 ESP32-S3-DevKitC-1U-N8R8 进行软件及基础功能测试。上述调整不影响其所搭载 ESP32-S3-MINI-1/1U 模组的正常供货。

- **替代产品**: ESP32-S3-DevKitC-1-N8R8 / ESP32-S3-DevKitC-1U-N8R8

### 1.3 开发板变体

| 型号 | 模组 | 天线类型 | Flash | 尺寸 | 价格（参考） |
|------|------|----------|-------|------|-------------|
| ESP32-S3-DevKitM-1-N8 | ESP32-S3-MINI-1 | PCB 板载天线 | 8 MB | 62.74 x 25.4 mm | 约 88 CNY |
| ESP32-S3-DevKitM-1U-N8 | ESP32-S3-MINI-1U | IPEX 外部天线 | 8 MB | 62.74 x 25.4 mm | 约 88 CNY / 15 USD |

### 1.4 核心特性

- 搭载 ESP32-S3FN8 芯片，8 MB flash 直接封装在芯片中，模组尺寸小巧
- 支持 **2.4 GHz Wi-Fi** 和 **蓝牙 5 (LE)**
- 高性能 Xtensa 32 位 LX7 **双核处理器**
- **USB OTG** 接口（支持全速 USB 1.1）
- 板载 **RGB LED**（由 GPIO48 驱动）
- 丰富的 GPIO 管脚引出，兼容面包板使用
- 工作温度范围：-40 ~ 85 摄氏度

---

## 2. 硬件描述

### 2.1 主要组件

ESP32-S3-DevKitM-1 开发板上的主要组件如下（按照逆时针顺序介绍）：

| 主要组件 | 介绍 |
|---------|------|
| **ESP32-S3-MINI-1/1U** | ESP32-S3-MINI-1 和 ESP32-S3-MINI-1U 是通用型 Wi-Fi + 低功耗蓝牙 MCU 模组，具有丰富的外设接口。ESP32-S3-MINI-1 采用 PCB 板载天线，ESP32-S3-MINI-1U 采用连接器连接外部天线。两款模组的核心是 ESP32-S3FN8 芯片，该芯片带有 8 MB flash，由于 flash 直接封装在芯片中，因此模组具有较小的封装尺寸。 |
| **5V 转 3.3V LDO** | 电源转换器，输入 5V，输出 3.3V，为模组和板载器件供电。 |
| **排针（Pin Headers）** | 所有可用 GPIO 管脚（除 flash 的 SPI 总线）均已引出至开发板两侧排针（J1 和 J3）。 |
| **USB 转 UART 接口** | Micro-USB 接口，可用作开发板的供电接口，可烧录固件至芯片，也可作为通信接口，通过板载 USB 转 UART 桥接器与芯片通信。 |
| **Boot 键** | 下载按键。按住 **Boot** 键的同时按一下 **Reset** 键进入"固件下载"模式，通过串口下载固件。 |
| **Reset 键** | 复位按键。按下可复位 ESP32-S3 芯片。 |
| **ESP32-S3 USB 接口** | ESP32-S3 USB OTG 接口，支持全速 USB 1.1 标准。可用作开发板的供电接口，可烧录固件至芯片，可通过 USB 协议与芯片通信，也可用于 JTAG 调试。 |
| **USB 转 UART 桥接器** | 单芯片 USB 至 UART 桥接器，可提供高达 3 Mbps 的传输速率。 |
| **RGB LED** | 可寻址 RGB 发光二极管，由 **GPIO48** 驱动。 |
| **3.3V 电源指示灯** | 开发板连接 USB 电源后，该指示灯亮起，表明电源正常。 |

### 2.2 功能框图

ESP32-S3-DevKitM-1 的主要组件和连接关系如下：

```
                    +-----------------------------+
                    |      ESP32-S3-MINI-1/1U     |
                    |      (ESP32-S3FN8 芯片)      |
                    |         8 MB Flash          |
                    +------+-------------+--------+
                           |             |
               +-----------+             +-----------+
               | USB 转 UART   |         | ESP32-S3    |
               | 桥接器        |         | USB OTG     |
               +-------+------+         +------+------+
                       |                       |
               +-------+------+         +------+------+
               | Micro-USB    |         | Micro-USB    |
               | (UART 接口)  |         | (USB 接口)   |
               +--------------+         +--------------+
                       |
               +-------+------+
               | 5V -> 3.3V   |
               | LDO 稳压器   |
               +--------------+
                       |
         +-------------+-------------+
         |                           |
    +----+----+               +------+------+
    | J1 排针 |               | J3 排针     |
    | (GPIO   |               | (GPIO       |
    |  0-18,  |               |  19-48,     |
    |  3V3,   |               |  RST, TX,   |
    |  5V, G) |               |  RX, G)     |
    +---------+               +-------------+
```

### 2.3 硬件规格参数

| 参数 | 规格 |
|------|------|
| 板载模组 | ESP32-S3-MINI-1 或 ESP32-S3-MINI-1U |
| 核心芯片 | ESP32-S3FN8 |
| 处理器 | Xtensa 32 位 LX7 双核，主频最高 240 MHz |
| Flash | 8 MB（芯片内置） |
| PSRAM | 无 |
| Wi-Fi | 802.11 b/g/n (2.4 GHz) |
| 蓝牙 | 蓝牙 5 (LE) |
| USB 接口 | USB 转 UART（Micro-USB）+ ESP32-S3 USB OTG（Micro-USB） |
| GPIO 管脚数 | 多个可用 GPIO（详见管脚描述表） |
| ADC | 2 个 12 位 SAR ADC，共 20 个模拟输入通道 |
| 触摸传感器 | 14 个电容触摸 GPIO |
| SPI | 4 个 SPI 接口 |
| UART | 3 个 UART 接口 |
| I2C | 2 个 I2C 接口 |
| I2S | 2 个 I2S 接口 |
| LED | RGB LED（GPIO48）、3.3V 电源指示灯 |
| 按键 | Boot 键、Reset 键 |
| 开发板尺寸 | 62.74 x 25.4 mm |
| 工作温度 | -40 ~ 85 摄氏度 |

---

## 3. 管脚布局与管脚描述

### 3.1 管脚布局图

开发板两侧各有 22 个管脚，分别标记为 **J1**（左侧）和 **J3**（右侧）。管脚布局参考乐鑫官方提供的管脚布局图。

### 3.2 J1 排针管脚描述

下表列出了 J1 排针的序号、名称、类型和功能。排针的序号与开发板原理图一致。

| 序号 | 名称 | 类型 | 功能 |
|------|------|------|------|
| 1 | 3V3 | P | 3.3 V 电源 |
| 2 | 0 | I/O/T | RTC_GPIO0, GPIO0 |
| 3 | 1 | I/O/T | RTC_GPIO1, GPIO1, TOUCH1, ADC1_CH0 |
| 4 | 2 | I/O/T | RTC_GPIO2, GPIO2, TOUCH2, ADC1_CH1 |
| 5 | 3 | I/O/T | RTC_GPIO3, GPIO3, TOUCH3, ADC1_CH2 |
| 6 | 4 | I/O/T | RTC_GPIO4, GPIO4, TOUCH4, ADC1_CH3 |
| 7 | 5 | I/O/T | RTC_GPIO5, GPIO5, TOUCH5, ADC1_CH4 |
| 8 | 6 | I/O/T | RTC_GPIO6, GPIO6, TOUCH6, ADC1_CH5 |
| 9 | 7 | I/O/T | RTC_GPIO7, GPIO7, TOUCH7, ADC1_CH6 |
| 10 | 8 | I/O/T | RTC_GPIO8, GPIO8, TOUCH8, ADC1_CH7, SUBSPICS1 |
| 11 | 9 | I/O/T | RTC_GPIO9, GPIO9, TOUCH9, ADC1_CH8, FSPIHD, SUBSPIHD |
| 12 | 10 | I/O/T | RTC_GPIO10, GPIO10, TOUCH10, ADC1_CH9, FSPICS0, FSPIIO4, SUBSPICS0 |
| 13 | 11 | I/O/T | RTC_GPIO11, GPIO11, TOUCH11, ADC2_CH0, FSPID, FSPIIO5, SUBSPID |
| 14 | 12 | I/O/T | RTC_GPIO12, GPIO12, TOUCH12, ADC2_CH1, FSPICLK, FSPIIO6, SUBSPICLK |
| 15 | 13 | I/O/T | RTC_GPIO13, GPIO13, TOUCH13, ADC2_CH2, FSPIQ, FSPIIO7, SUBSPIQ |
| 16 | 14 | I/O/T | RTC_GPIO14, GPIO14, TOUCH14, ADC2_CH3, FSPIWP, FSPIDQS, SUBSPIWP |
| 17 | 15 | I/O/T | RTC_GPIO15, GPIO15, U0RTS, ADC2_CH4, XTAL_32K_P |
| 18 | 16 | I/O/T | RTC_GPIO16, GPIO16, U0CTS, ADC2_CH5, XTAL_32K_N |
| 19 | 17 | I/O/T | RTC_GPIO17, GPIO17, U1TXD, ADC2_CH6 |
| 20 | 18 | I/O/T | RTC_GPIO18, GPIO18, U1RXD, ADC2_CH7, CLK_OUT3 |
| 21 | 5V | P | 5 V 电源 |
| 22 | G | G | 接地 |

### 3.3 J3 排针管脚描述

| 序号 | 名称 | 类型 | 功能 |
|------|------|------|------|
| 1 | G | G | 接地 |
| 2 | RST | I | EN（使能/复位管脚，高电平使能，低电平关闭） |
| 3 | 46 | I/O/T | GPIO46 |
| 4 | 45 | I/O/T | GPIO45 |
| 5 | RX | I/O/T | U0RXD, GPIO44, CLK_OUT2 |
| 6 | TX | I/O/T | U0TXD, GPIO43, CLK_OUT1 |
| 7 | 42 | I/O/T | MTMS, GPIO42 |
| 8 | 41 | I/O/T | MTDI, GPIO41, CLK_OUT1 |
| 9 | 40 | I/O/T | MTDO, GPIO40, CLK_OUT2 |
| 10 | 39 | I/O/T | MTCK, GPIO39, CLK_OUT3, SUBSPICS1 |
| 11 | 38 | I/O/T | GPIO38, FSPIWP, SUBSPIWP |
| 12 | 37 | I/O/T | SPIDQS, GPIO37, FSPIQ, SUBSPIQ |
| 13 | 36 | I/O/T | SPIIO7, GPIO36, FSPICLK, SUBSPICLK |
| 14 | 35 | I/O/T | SPIIO6, GPIO35, FSPID, SUBSPID |
| 15 | 34 | I/O/T | SPIIO5, GPIO34, FSPICS0, SUBSPICS0 |
| 16 | 33 | I/O/T | SPIIO4, GPIO33, FSPIHD, SUBSPIHD |
| 17 | 26 | I/O/T | SPICS1, GPIO26 |
| 18 | 21 | I/O/T | RTC_GPIO21, GPIO21 |
| 19 | 20 | I/O/T | RTC_GPIO20, GPIO20, U1CTS, ADC2_CH9, CLK_OUT1, USB_D+ |
| 20 | 19 | I/O/T | RTC_GPIO19, GPIO19, U1RTS, ADC2_CH8, CLK_OUT2, USB_D- |
| 21 | 48 | I/O/T | SPICLK_N, GPIO48, SUBSPICLK_N_DIFF, RGB LED |
| 22 | 47 | I/O/T | SPICLK_P, GPIO47, SUBSPICLK_P_DIFF |

### 3.4 管脚类型说明

| 标记 | 含义 |
|------|------|
| **P** | 电源（Power） |
| **I** | 输入（Input） |
| **O** | 输出（Output） |
| **T** | 可设置为高阻态（Tri-state） |
| **G** | 接地（Ground） |

### 3.5 USB 接口管脚

| 接口 | 管脚 | 功能 |
|------|------|------|
| USB 转 UART 接口 | Micro-USB | 供电、固件下载、串口通信 |
| ESP32-S3 USB 接口 | Micro-USB | 供电、固件下载、USB 通信、JTAG 调试 |
| USB D+ | GPIO20 | USB 数据正极 |
| USB D- | GPIO19 | USB 数据负极 |

### 3.6 特殊管脚说明

- **Strapping 管脚**: GPIO0、GPIO45、GPIO46 等为 Strapping 管脚，在芯片上电和系统复位过程中，这些管脚根据其二进制电压值控制芯片启动模式。
  - **GPIO0**: 默认弱上拉。拉低时配合复位可进入固件下载模式。
  - **GPIO45**: 用于选择 VDD_SPI Flash 电压（1.8V 或 3.3V）。
  - **GPIO46**: 默认弱下拉。
- **EN (RST)**: 芯片使能管脚。高电平时芯片使能工作，低电平时芯片关闭。不能让此管脚浮空。
- **GPIO48**: 驱动板载 RGB LED。
- **GPIO19 / GPIO20**: USB D-/D+ 管脚，用于 ESP32-S3 内置 USB 外设功能。

---

## 4. 电源供电信息

### 4.1 电源选项

ESP32-S3-DevKitM-1 支持以下三种供电方式：

| 供电方式 | 说明 | 推荐程度 |
|---------|------|---------|
| **USB 转 UART 接口** 或 **ESP32-S3 USB 接口** | 通过 Micro-USB 线供电，可选择其一或同时供电 | 推荐（默认） |
| **5V 和 G (GND) 排针** | 通过排针的 5V 和 GND 管脚输入 5V 电源 | 可选 |
| **3V3 和 G (GND) 排针** | 通过排针的 3V3 和 GND 管脚直接输入 3.3V 电源 | 可选 |

### 4.2 电源管理

- 开发板内置 **5V 转 3.3V LDO 稳压器**，当通过 USB 接口或 5V 排针供电时，LDO 将 5V 转换为 3.3V 为模组供电。
- 当直接通过 3V3 排针供电时，电源绕过 LDO 直接为模组供电。
- **3.3V 电源指示灯** 在 USB 电源连接后会亮起，表明电源正常工作。

### 4.3 电源注意事项

- ESP32-S3 芯片的工作电压范围为 **3.0V ~ 3.6V**。
- 使用单电源供电时，建议电源电压为 **3.3V**，输出电流需达到 **500 mA** 及以上。
- 通过 3.3V 排针供电时，需确保外部电源稳定且纹波小。
- 若使用 USB 供电，请确保 USB 端口能提供足够的电流（建议 500 mA 以上）。

---

## 5. 入门指南

### 5.1 必备硬件

| 硬件 | 说明 |
|------|------|
| ESP32-S3-DevKitM-1 开发板 | 板载 ESP32-S3-MINI-1 或 ESP32-S3-MINI-1U 模组 |
| USB 2.0 数据线 | 标准 A 型转 Micro-B 型（确保支持数据传输，非纯充电线） |
| 电脑 | Windows、Linux 或 macOS |

### 5.2 硬件设置

1. **检查开发板**: 通电前，请确保开发板完好无损，无明显焊接缺陷或元件损坏。

2. **连接 USB 数据线**: 通过 **USB 转 UART 接口** 或 **ESP32-S3 USB 接口** 将开发板与电脑连接。
   - 默认推荐使用 **USB 转 UART 接口**。
   - 连接后，**3.3V 电源指示灯** 应亮起。

3. **确认串口识别**: 在电脑的设备管理器（Windows）或终端（Linux/macOS）中确认开发板已被识别为串口设备。

### 5.3 固件下载（进入下载模式）

ESP32-S3 支持 **UART0** 和 **USB** 两种固件下载方式。

#### 方式一：通过按键进入下载模式

1. 按住 **Boot** 键不放。
2. 同时按一下 **Reset** 键。
3. 松开 **Boot** 键。
4. 此时芯片进入"固件下载"模式，可通过串口或 USB 下载固件。

#### 方式二：通过 USB 接口下载

1. 将开发板通过 **ESP32-S3 USB 接口** 连接至电脑。
2. 首次使用时，需手动进入下载模式（按住 Boot 键再上电）。
3. 后续可通过软件自动进入下载模式。

#### 下载模式验证

进入下载模式后，UART0 将输出如下日志：

```
ESP-ROM:esp32s3-20210327
Build:Mar 27 2021
rst:0x1 (POWERON),boot:0x0 (DOWNLOAD(USB/UART0))
waiting for download
```

### 5.4 软件设置

#### 安装 ESP-IDF 开发环境

乐鑫提供完整的软件开发环境 **ESP-IDF**（Espressif IoT Development Framework），用于开发 ESP32-S3 应用程序。

请前往 [ESP-IDF 快速入门](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/get-started/index.html)，根据操作系统选择对应的安装指南：

- [在 Windows 上安装 ESP-IDF](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/get-started/windows-setup.html)
- [在 Linux 上安装 ESP-IDF](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/get-started/linux-setup.html)
- [在 macOS 上安装 ESP-IDF](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/get-started/macos-setup.html)

#### 构建和烧录示例项目

安装完成后，可通过以下方式构建和烧录应用程序：

1. **命令行方式**: 参考 [在 Windows 上通过命令行构建项目](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/get-started/windows-start-project.html) 或 [在 Linux/macOS 上通过命令行构建项目](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/get-started/linux-macos-start-project.html)。

2. **IDE 方式**:
   - [Espressif-IDE](https://docs.espressif.com/projects/espressif-ide/zh_CN/latest/)（基于 Eclipse CDT）
   - [VS Code ESP-IDF 扩展](https://docs.espressif.com/projects/vscode-esp-idf-extension/zh_CN/latest/index.html)

3. **Flash 下载工具**: 乐鑫提供的 PC 上位机 [Flash 下载工具](https://docs.espressif.com/projects/esp-test-tools/zh_CN/latest/esp32/production_stage/tools/flash_download_tool.html)，可直接下载固件（.bin）到 Flash 中。

### 5.5 固件运行验证

固件下载完成后：

1. 将 **GPIO0** 拉高（默认为高电平）。
2. 拉低再拉高 **EN（CHIP_PU）** 管脚进行硬件复位重启。
3. 芯片进入 Flash 启动模式，运行烧录的固件。
4. 使用串口调试软件查看 UART0 日志输出，检查固件运行状态。

### 5.6 包装信息

#### 零售订单

如购买样品，每个开发板将以防静电袋或零售商选择的其他方式包装。

- 零售订单入口: https://www.espressif.com/zh-hans/company/contact/buy-a-sample

#### 批量订单

如批量购买，开发板将以大纸板箱包装。

- 批量订单入口: https://www.espressif.com/zh-hans/contact-us/sales-questions

---

## 6. 相关文档

### 6.1 芯片和模组文档

| 文档 | 链接 |
|------|------|
| ESP32-S3 技术规格书（中文） | [PDF 下载](https://www.espressif.com/sites/default/files/documentation/esp32-s3_datasheet_cn.pdf) |
| ESP32-S3-MINI-1 & ESP32-S3-MINI-1U 技术规格书（中文） | [PDF 下载](https://www.espressif.com/sites/default/files/documentation/esp32-s3-mini-1_mini-1u_datasheet_cn.pdf) |
| ESP32-S3 硬件设计指南（中文） | [PDF 下载](https://www.espressif.com/sites/default/files/documentation/esp32-s3_hardware_design_guidelines_cn.pdf) |

### 6.2 开发板设计文档

| 文档 | 链接 |
|------|------|
| ESP32-S3-DevKitM-1 原理图 | [PDF 下载](https://dl.espressif.com/dl/schematics/SCH_ESP32-S3-DEVKITM-1_V1_20210310A.pdf) |
| ESP32-S3-DevKitM-1 PCB 布局图 | [PDF 下载](https://dl.espressif.com/dl/schematics/PCB_ESP32-S3-DevKitM-1_V1_20210310AC.pdf) |
| ESP32-S3-DevKitM-1 尺寸图 | [PDF 下载](https://dl.espressif.com/dl/schematics/DXF_ESP32-S3-DevKitM-1_V1_20210310AC.pdf) |
| ESP32-S3-DevKitM-1 尺寸图源文件 | [DXF 下载](https://dl.espressif.com/dl/schematics/DXF_ESP32-S3-DevKitM-1_V1_20210310AC.dxf)（可使用 [Autodesk Viewer](https://viewer.autodesk.com/) 查看） |

### 6.3 开发框架和工具

| 文档 | 链接 |
|------|------|
| ESP-IDF 编程指南 | [在线文档](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/get-started/index.html) |
| ESP-IDF 快速入门 | [在线文档](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/get-started/index.html) |
| Flash 下载工具用户指南 | [在线文档](https://docs.espressif.com/projects/esp-test-tools/zh_CN/latest/esp32/production_stage/tools/flash_download_tool.html) |

### 6.4 替代产品文档

| 文档 | 链接 |
|------|------|
| ESP32-S3-DevKitC-1 用户指南 | [在线文档](https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32s3/esp32-s3-devkitc-1/index.html) |

### 6.5 商务联系

有关本开发板的更多设计文档，请联系乐鑫商务部门: [sales@espressif.com](mailto:sales@espressif.com)

---

## 附录 A: ESP32-S3 芯片主要特性

| 特性 | 规格 |
|------|------|
| CPU | Xtensa 32 位 LX7 双核处理器，主频最高 240 MHz |
| 超低功耗协处理器 | 运行 RISC-V 或 FSM 内核 |
| Wi-Fi | 802.11 b/g/n (2.4 GHz)，支持 STA/AP/STA+AP 模式 |
| 蓝牙 | 蓝牙 5 (LE)，支持长距离通信、高速模式和广播扩展 |
| Flash | 8 MB（ESP32-S3FN8 芯片内置） |
| SRAM | 512 KB SRAM |
| GPIO | 最多 45 个可编程 GPIO |
| ADC | 2 个 12 位 SAR ADC，共 20 个通道 |
| 触摸传感器 | 14 个电容触摸 GPIO |
| SPI | 4 个 SPI 接口 |
| UART | 3 个 UART 接口 |
| I2C | 2 个 I2C 接口（软件实现） |
| I2S | 2 个 I2S 接口 |
| USB | USB OTG 全速 (1.1) 接口 + USB 串口/JTAG 控制器 |
| PWM | 8 个通道 LED PWM |
| 定时器 | 4 个 64 位通用定时器 |
| 看门狗 | 3 个看门狗定时器 |
| 安全特性 | AES、SHA、RSA、HMAC、数字签名、安全启动、Flash 加密 |
| 工艺 | 40 nm |

---

## 附录 B: 常见问题

### B.1 如何确认开发板是否正常工作？

1. 通过 USB 线连接开发板至电脑。
2. 确认 3.3V 电源指示灯亮起。
3. 在设备管理器中查看是否识别到串口设备。
4. 打开串口调试工具，按下 Reset 键，查看是否有启动日志输出。

### B.2 如何进入固件下载模式？

按住 **Boot** 键的同时按一下 **Reset** 键，然后松开 Boot 键。此时芯片进入下载模式，等待固件下载。

### B.3 开发板支持哪些操作系统？

ESP32-S3-DevKitM-1 的开发环境支持以下操作系统：
- Windows 10 及以上
- Linux（Ubuntu、Debian 等主流发行版）
- macOS

### B.4 如何使用板载 RGB LED？

板载 RGB LED 由 **GPIO48** 驱动，为可寻址 RGB LED。可通过 ESP-IDF 的 RMT 外设或专用 LED 驱动库进行控制。

---

> **文档版本**: 基于乐鑫官方 ESP32-S3-DevKitM-1 用户指南（最新版）
>
> **官方文档地址**: https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32s3/esp32-s3-devkitm-1/user_guide.html
