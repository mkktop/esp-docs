# ESP32-S3-DevKitC-1 用户指南

> 本文档基于乐鑫官方文档整理，涵盖 ESP32-S3-DevKitC-1 开发板 v1.0 和 v1.1 版本。
>
> 官方文档链接：
> - [ESP32-S3-DevKitC-1 v1.1 用户指南（中文）](https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32s3/esp32-s3-devkitc-1/user_guide_v1.1.html)
> - [ESP32-S3-DevKitC-1 v1.0 用户指南（中文）](https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32s3/esp32-s3-devkitc-1/user_guide_v1.0.html)

---

## 目录

1. [开发板概述](#1-开发板概述)
2. [硬件描述](#2-硬件描述)
   - [主要组件](#21-主要组件)
   - [功能框图](#22-功能框图)
3. [管脚布局与管脚描述](#3-管脚布局与管脚描述)
   - [J1 排针](#31-j1-排针)
   - [J3 排针](#32-j3-排针)
   - [管脚类型说明](#33-管脚类型说明)
   - [重要备注](#34-重要备注)
4. [电源供应](#4-电源供应)
5. [入门指南](#5-入门指南)
   - [必备硬件](#51-必备硬件)
   - [硬件设置](#52-硬件设置)
   - [软件设置](#53-软件设置)
   - [固件下载模式](#54-固件下载模式)
6. [订购信息](#6-订购信息)
7. [硬件版本历史](#7-硬件版本历史)
8. [相关文档](#8-相关文档)

---

## 1. 开发板概述

ESP32-S3-DevKitC-1 是乐鑫（Espressif）推出的一款入门级开发板，搭载 **Wi-Fi + Bluetooth LE** 模组 **ESP32-S3-WROOM-1**、**ESP32-S3-WROOM-1U** 或 **ESP32-S3-WROOM-2**。

该开发板具有以下特点：

- 基于 ESP32-S3 芯片，支持 2.4 GHz Wi-Fi (802.11 b/g/n) 和 Bluetooth 5 (LE)
- 搭载 Xtensa 32 位 LX7 双核处理器，主频最高可达 240 MHz
- 具有丰富的外设接口、强大的神经网络运算能力和信号处理能力，专为人工智能（AI）和 AIoT 市场打造
- 板上模组的大部分管脚均已引出至开发板两侧排针
- 开发人员可根据实际需求，轻松通过跳线连接多种外围设备
- 可将开发板直接插在面包板上使用
- 支持 USB OTG 功能

> **ESP32-S3-WROOM-1** 和 **ESP32-S3-WROOM-2** 采用 PCB 板载天线，**ESP32-S3-WROOM-1U** 采用连接器连接外部天线。

---

## 2. 硬件描述

### 2.1 主要组件

以下按照逆时针顺序依次介绍 ESP32-S3-DevKitC-1 开发板上的主要组件：

| 主要组件 | 介绍 |
| --- | --- |
| **ESP32-S3-WROOM-1/1U/2** | 通用型 Wi-Fi + 低功耗蓝牙 MCU 模组，具有丰富的外设接口、强大的神经网络运算能力和信号处理能力，专为人工智能和 AIoT 市场打造。ESP32-S3-WROOM-1 和 ESP32-S3-WROOM-2 采用 PCB 板载天线，ESP32-S3-WROOM-1U 采用连接器连接外部天线。 |
| **5 V to 3.3 V LDO** | 电源转换器，输入 5 V，输出 3.3 V，为板上各组件供电。 |
| **Pin Headers（排针）** | 所有可用 GPIO 管脚（除 Flash 的 SPI 总线）均已引出至开发板的排针，方便连接外部设备。 |
| **USB-to-UART Port（USB 转 UART 接口）** | Micro-USB 接口，可用作开发板的供电接口，可烧录固件至芯片，也可作为通信接口，通过板载 USB 转 UART 桥接器与芯片通信。 |
| **Boot Button（Boot 键）** | 下载按键。按住 **Boot** 键的同时按一下 **Reset** 键进入"固件下载"模式，通过串口下载固件。 |
| **Reset Button（Reset 键）** | 复位按键，按下后芯片复位重启。 |
| **USB Port（USB 接口）** | ESP32-S3 USB OTG 接口，支持全速 USB 1.1 标准。可用作开发板的供电接口，可烧录固件至芯片，可通过 USB 协议与芯片通信，也可用于 JTAG 调试。 |
| **USB-to-UART Bridge（USB 转 UART 桥接器）** | 单芯片 USB 至 UART 桥接器，可提供高达 3 Mbps 的传输速率。 |
| **RGB LED** | 可寻址 RGB 发光二极管。v1.0 版本由 GPIO48 驱动，v1.1 版本由 GPIO38 驱动。 |
| **3.3 V Power On LED（3.3 V 电源指示灯）** | 开发板连接 USB 电源后，该指示灯亮起，表示电源正常。 |

### 2.2 功能框图

ESP32-S3-DevKitC-1 的主要组件和连接方式如下图所示：

```
+---------------------------------------------------------------+
|                  ESP32-S3-DevKitC-1 功能框图                    |
+---------------------------------------------------------------+
|                                                                 |
|   USB 转 UART 接口       ESP32-S3 USB 接口                      |
|   (Micro-USB)            (Micro-USB)                            |
|        |                      |                                 |
|        v                      v                                 |
|   USB 转 UART 桥接器    ESP32-S3 USB OTG                        |
|        |                      |                                 |
|        +------+  +------------+                                 |
|               |  |                                               |
|               v  v                                               |
|        +------------------+                                      |
|        |   ESP32-S3 芯片   |<------- 5V -> 3.3V LDO <--- 5V/GND |
|        |  (WROOM-1/1U/2)  |             ^                       |
|        +------------------+             |                       |
|           |    |    |    |          USB 供电                     |
|           v    v    v    v                                       |
|         J1 排针    J3 排针    RGB LED    3.3V LED                |
|                                                                 |
|   Boot 键 ----> GPIO0 (下载模式)                                |
|   Reset 键 --> CHIP_PU (复位)                                   |
+---------------------------------------------------------------+
```

功能框图展示了以下关键连接：

- **USB 转 UART 接口** 通过板载 USB 转 UART 桥接器连接至 ESP32-S3 芯片的 UART0（TX/RX）
- **ESP32-S3 USB 接口** 直接连接至 ESP32-S3 芯片的 USB OTG 引脚（GPIO19/20）
- **5V 转 3.3V LDO** 将 USB 输入的 5V 电源转换为 3.3V，为 ESP32-S3 模组供电
- **两侧排针（J1 和 J3）** 引出大部分可用 GPIO 管脚
- **Boot 键** 连接至 GPIO0，用于进入固件下载模式
- **Reset 键** 连接至 CHIP_PU（EN）引脚，用于芯片复位

---

## 3. 管脚布局与管脚描述

### 3.1 J1 排针

下表列出了 J1 排针的名称和功能，排针序号与开发板原理图一致。

| 序号 | 名称 | 类型 | 功能 |
| ---: | ---: | :---: | --- |
| 1 | 3V3 | P | 3.3 V 电源 |
| 2 | 3V3 | P | 3.3 V 电源 |
| 3 | RST | I | EN（芯片使能/复位引脚） |
| 4 | 4 | I/O/T | RTC_GPIO4, GPIO4, TOUCH4, ADC1_CH3 |
| 5 | 5 | I/O/T | RTC_GPIO5, GPIO5, TOUCH5, ADC1_CH4 |
| 6 | 6 | I/O/T | RTC_GPIO6, GPIO6, TOUCH6, ADC1_CH5 |
| 7 | 7 | I/O/T | RTC_GPIO7, GPIO7, TOUCH7, ADC1_CH6 |
| 8 | 15 | I/O/T | RTC_GPIO15, GPIO15, U0RTS, ADC2_CH4, XTAL_32K_P |
| 9 | 16 | I/O/T | RTC_GPIO16, GPIO16, U0CTS, ADC2_CH5, XTAL_32K_N |
| 10 | 17 | I/O/T | RTC_GPIO17, GPIO17, U1TXD, ADC2_CH6 |
| 11 | 18 | I/O/T | RTC_GPIO18, GPIO18, U1RXD, ADC2_CH7, CLK_OUT3 |
| 12 | 8 | I/O/T | RTC_GPIO8, GPIO8, TOUCH8, ADC1_CH7, SUBSPICS1 |
| 13 | 3 | I/O/T | RTC_GPIO3, GPIO3, TOUCH3, ADC1_CH2 |
| 14 | 46 | I/O/T | GPIO46 |
| 15 | 9 | I/O/T | RTC_GPIO9, GPIO9, TOUCH9, ADC1_CH8, FSPIHD, SUBSPIHD |
| 16 | 10 | I/O/T | RTC_GPIO10, GPIO10, TOUCH10, ADC1_CH9, FSPICS0, FSPIIO4, SUBSPICS0 |
| 17 | 11 | I/O/T | RTC_GPIO11, GPIO11, TOUCH11, ADC2_CH0, FSPID, FSPIIO5, SUBSPID |
| 18 | 12 | I/O/T | RTC_GPIO12, GPIO12, TOUCH12, ADC2_CH1, FSPICLK, FSPIIO6, SUBSPICLK |
| 19 | 13 | I/O/T | RTC_GPIO13, GPIO13, TOUCH13, ADC2_CH2, FSPIQ, FSPIIO7, SUBSPIQ |
| 20 | 14 | I/O/T | RTC_GPIO14, GPIO14, TOUCH14, ADC2_CH3, FSPIWP, FSPIDQS, SUBSPIWP |
| 21 | 5V | P | 5 V 电源 |
| 22 | G | G | 接地 |

### 3.2 J3 排针

下表列出了 J3 排针的名称和功能：

| 序号 | 名称 | 类型 | 功能 |
| ---: | ---: | :---: | --- |
| 1 | G | G | 接地 |
| 2 | TX | I/O/T | U0TXD, GPIO43, CLK_OUT1 |
| 3 | RX | I/O/T | U0RXD, GPIO44, CLK_OUT2 |
| 4 | 1 | I/O/T | RTC_GPIO1, GPIO1, TOUCH1, ADC1_CH0 |
| 5 | 2 | I/O/T | RTC_GPIO2, GPIO2, TOUCH2, ADC1_CH1 |
| 6 | 42 | I/O/T | MTMS, GPIO42 |
| 7 | 41 | I/O/T | MTDI, GPIO41, CLK_OUT1 |
| 8 | 40 | I/O/T | MTDO, GPIO40, CLK_OUT2 |
| 9 | 39 | I/O/T | MTCK, GPIO39, CLK_OUT3, SUBSPICS1 |
| 10 | 38 | I/O/T | GPIO38, FSPIWP, SUBSPIWP, RGB LED (v1.1) |
| 11 | 37 | I/O/T | SPIDQS, GPIO37, FSPIQ, SUBSPIQ |
| 12 | 36 | I/O/T | SPIIO7, GPIO36, FSPICLK, SUBSPICLK |
| 13 | 35 | I/O/T | SPIIO6, GPIO35, FSPID, SUBSPID |
| 14 | 0 | I/O/T | RTC_GPIO0, GPIO0 |
| 15 | 45 | I/O/T | GPIO45 |
| 16 | 48 | I/O/T | GPIO48, SPICLK_N, SUBSPICLK_N_DIFF, RGB LED (v1.0) |
| 17 | 47 | I/O/T | GPIO47, SPICLK_P, SUBSPICLK_P_DIFF |
| 18 | 21 | I/O/T | RTC_GPIO21, GPIO21 |
| 19 | 20 | I/O/T | RTC_GPIO20, GPIO20, U1CTS, ADC2_CH9, CLK_OUT1, USB_D+ |
| 20 | 19 | I/O/T | RTC_GPIO19, GPIO19, U1RTS, ADC2_CH8, CLK_OUT2, USB_D- |
| 21 | G | G | 接地 |
| 22 | G | G | 接地 |

### 3.3 管脚类型说明

| 标记 | 含义 |
| :---: | --- |
| **P** | 电源（Power） |
| **I** | 输入（Input） |
| **O** | 输出（Output） |
| **T** | 可设置为高阻（Tri-state） |
| **G** | 接地（Ground） |

有关管脚功能名称的详细解释，请参考 [ESP32-S3 技术规格书](https://www.espressif.com/sites/default/files/documentation/esp32-s3_datasheet_cn.pdf) (PDF)。

### 3.4 重要备注

> **注意：** 在板载 ESP32-S3-WROOM-1/1U 模组系列（使用 8 线 SPI Flash/PSRAM）的开发板和板载 ESP32-S3-WROOM-2 模组系列的开发板中，管脚 **GPIO35**、**GPIO36** 和 **GPIO37** 已用于内部 ESP32-S3 芯片与 SPI Flash/PSRAM 之间的通信，**外部不可使用**。

> **v1.0 与 v1.1 差异：**
> - v1.0 版本的 RGB LED 由 **GPIO48** 驱动
> - v1.1 版本的 RGB LED 改为由 **GPIO38** 驱动
> - v1.1 版本中 GPIO38 标注了 RGB LED 功能，GPIO48 不再标注 RGB LED

---

## 4. 电源供应

ESP32-S3-DevKitC-1 支持以下三种供电方式（任选其一即可）：

| 供电方式 | 说明 |
| --- | --- |
| **USB 转 UART 接口供电** | 通过 Micro-USB 数据线连接电脑或 USB 电源适配器供电（推荐，默认方式） |
| **ESP32-S3 USB 接口供电** | 通过 ESP32-S3 USB OTG 接口供电（v1.1 版本支持，v1.0 版本软件暂不支持） |
| **5V 和 GND 排针供电** | 通过 J1 排针的 5V（序号 21）和 GND（序号 22）引脚外接 5V 电源 |
| **3V3 和 GND 排针供电** | 通过 J1 排针的 3V3（序号 1、2）和 GND（序号 22）引脚外接 3.3V 电源 |

**电源说明：**

- 板载 **5V 转 3.3V LDO** 稳压器将 USB 输入的 5V 电源转换为 3.3V，为 ESP32-S3 模组供电
- 当通过 USB 接口供电时，**3.3V 电源指示灯** 会亮起，表示电源正常
- 两个 USB 接口可以同时供电，也可选择其中一个供电

> **注意：** 请确保使用适当的 USB 数据线。部分数据线仅可用于充电，无法用于数据传输和编程。推荐使用 USB 2.0 数据线（标准 A 型转 Micro-B 型）。

---

## 5. 入门指南

### 5.1 必备硬件

- **ESP32-S3-DevKitC-1** 开发板
- **USB 2.0 数据线**（标准 A 型转 Micro-B 型）
- **电脑**（Windows、Linux 或 macOS）

### 5.2 硬件设置

1. 使用 USB 数据线将开发板的 **USB 转 UART 接口** 连接至电脑
2. 电脑会自动识别开发板的串口设备
3. 确认 **3.3V 电源指示灯** 亮起，表示开发板已正常供电

> **v1.1 版本说明：** 也可以通过 **ESP32-S3 USB 接口** 连接开发板与电脑，用于供电、烧录固件、USB 通信或 JTAG 调试。

### 5.3 软件设置

ESP32-S3-DevKitC-1 使用乐鑫官方的 **ESP-IDF** 开发框架进行开发。

**设置步骤：**

1. 前往 [ESP-IDF 快速入门指南](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/get-started/index.html)
2. 按照 [详细安装步骤](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/get-started/index.html#get-started-step-by-step) 小节的说明，安装 ESP-IDF 开发环境
3. 配置开发环境，选择目标芯片为 ESP32-S3
4. 编译并烧录示例应用程序至开发板

**ESP-IDF 开发框架支持：**

- Windows、Linux 和 macOS 操作系统
- C 语言开发
- 丰富的示例代码和组件库
- 完善的调试工具链

### 5.4 固件下载模式

要将固件烧录至 ESP32-S3-DevKitC-1，请按以下步骤操作：

1. 按住 **Boot** 键不放
2. 同时按一下 **Reset** 键
3. 松开 **Boot** 键

此时开发板进入"固件下载"模式，可通过串口下载固件。ESP-IDF 的烧录工具会自动处理下载过程。

烧录完成后，按一下 **Reset** 键即可正常运行新固件。

---

## 6. 订购信息

ESP32-S3-DevKitC-1 开发板有多种型号可供选择：

| 订购代码 | 搭载模组 | Flash | PSRAM | SPI 电压 | 天线类型 |
| --- | --- | --- | --- | --- | --- |
| ESP32-S3-DevKitC-1-N8R8 | ESP32-S3-WROOM-1-N8R8 | 8 MB QD | 8 MB OT | 3.3 V | PCB 板载天线 |
| ESP32-S3-DevKitC-1-N32R16V | ESP32-S3-WROOM-2-N32R16V | 32 MB OT | 16 MB OT | 1.8 V | PCB 板载天线 |
| ESP32-S3-DevKitC-1U-N8R8 | ESP32-S3-WROOM-1U-N8R8 | 8 MB QD | 8 MB OT | 3.3 V | 外部天线连接器 |

> **说明：** QD 指代 Quad SPI，OT 指代 Octal SPI。

**购买渠道：**

- **零售购买：** [淘宝](https://item.taobao.com/item.htm?spm=a1z10.5-c-s.w4002-22443450244.12.49e867d8WoFnsC&id=653155344338) | [Digi-Key](https://www.digikey.com/en/products/detail/espressif-systems/ESP32-S3-DEVKITC-1-N8R8/15295894) | [Mouser](https://www.mouser.com/ProductDetail/Espressif-Systems/ESP32-S3-DevKitC-1-N8R8?qs=7D1LtPJG0i2PiuUUKucutQ%3D%3D)
- **乐鑫官方样品订购：** <https://www.espressif.com/zh-hans/company/contact/buy-a-sample>
- **批量订购：** <https://www.espressif.com/zh-hans/contact-us/sales-questions>

---

## 7. 硬件版本历史

### v1.1（最新版本）

- RGB LED 驱动引脚从 GPIO48 更改为 **GPIO38**
- 支持通过 **ESP32-S3 USB 接口** 进行固件烧录和 JTAG 调试
- 原理图下载：[SCH_ESP32-S3-DevKitC-1_V1.1_20221130.pdf](https://dl.espressif.com/dl/schematics/SCH_ESP32-S3-DevKitC-1_V1.1_20221130.pdf)

### v1.0

- RGB LED 由 **GPIO48** 驱动
- 软件暂不支持通过 ESP32-S3 USB 接口连接
- 原理图下载：[SCH_ESP32-S3-DEVKITC-1_V1_20210312C.pdf](https://dl.espressif.com/dl/SCH_ESP32-S3-DEVKITC-1_V1_20210312C.pdf)

---

## 8. 相关文档

### 芯片与模组技术规格书

- [ESP32-S3 技术规格书](https://www.espressif.com/sites/default/files/documentation/esp32-s3_datasheet_cn.pdf) (PDF)
- [ESP32-S3-WROOM-1 & ESP32-S3-WROOM-1U 技术规格书](https://www.espressif.com/sites/default/files/documentation/esp32-s3-wroom-1_wroom-1u_datasheet_cn.pdf) (PDF)
- [ESP32-S3-WROOM-2 技术规格书](https://www.espressif.com/sites/default/files/documentation/esp32-s3-wroom-2_datasheet_cn.pdf) (PDF)

### 开发板设计文件

**v1.1 版本：**

- [ESP32-S3-DevKitC-1 V1.1 原理图](https://dl.espressif.com/dl/schematics/SCH_ESP32-S3-DevKitC-1_V1.1_20221130.pdf) (PDF)
- [ESP32-S3-DevKitC-1 V1.1 PCB 布局图](https://dl.espressif.com/dl/schematics/PCB_ESP32-S3-DevKitC-1_V1.1_20220429.pdf) (PDF)
- [ESP32-S3-DevKitC-1 V1.1 尺寸图](https://dl.espressif.com/dl/schematics/esp_idf/DXF_ESP32-S3-DevKitC-1_V1.1_20220429.pdf) (PDF)
- [ESP32-S3-DevKitC-1 V1.1 尺寸图源文件](https://dl.espressif.com/dl/schematics/esp_idf/DXF_ESP32-S3-DevKitC-1_V1.1_20220429.dxf) (DXF)

**v1.0 版本：**

- [ESP32-S3-DevKitC-1 V1.0 原理图](https://dl.espressif.com/dl/SCH_ESP32-S3-DEVKITC-1_V1_20210312C.pdf) (PDF)
- [ESP32-S3-DevKitC-1 V1.0 PCB 布局图](https://dl.espressif.com/dl/PCB_ESP32-S3-DevKitC-1_V1_20210312CB.pdf) (PDF)
- [ESP32-S3-DevKitC-1 V1.0 尺寸图](https://dl.espressif.com/dl/DXF_ESP32-S3-DevKitC-1_V1_20210312CB.pdf) (PDF)
- [ESP32-S3-DevKitC-1 V1.0 尺寸图源文件](https://dl.espressif.com/dl/DXF_ESP32-S3-DevKitC-1_V1_20210312CB.dxf) (DXF)

### 开发框架与工具

- [ESP-IDF 编程指南](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/get-started/index.html)
- [ESP-IDF GitHub 仓库](https://github.com/espressif/esp-idf)

### 其他资源

- [ESP32-S3-DevKitC-1 官方用户指南（v1.1 中文版）](https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32s3/esp32-s3-devkitc-1/user_guide_v1.1.html)
- [ESP32-S3-DevKitC-1 官方用户指南（v1.1 英文版）](https://docs.espressif.com/projects/esp-dev-kits/en/latest/esp32s3/esp32-s3-devkitc-1/user_guide_v1.1.html)

> 有关本开发板的更多设计文档，请联系乐鑫商务部门：[sales@espressif.com](mailto:sales@espressif.com)。

---

*本文档最后更新：基于 ESP32-S3-DevKitC-1 v1.1 用户指南整理*
