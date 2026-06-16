# ESP32-C3-DevKitM-1 用户指南

> 文档编号：09
> 适用开发板：ESP32-C3-DevKitM-1（搭载 ESP32-C3-MINI-1 / MINI-1U）
> 来源：乐鑫官方 esp-dev-kits 用户指南、ESP32-C3-DevKitM-1 新闻

---

## 目录

1. [开发板概述](#1-开发板概述)
2. [核心规格](#2-核心规格)
3. [板载组件](#3-板载组件)
4. [电源选项](#4-电源选项)
5. [排针定义（J1 / J3）](#5-排针定义j1--j3)
6. [Strapping 管脚](#6-strapping-管脚)
7. [快速入门](#7-快速入门)
8. [应用场景](#8-应用场景)
9. [参考资源](#9-参考资源)

---

## 1. 开发板概述

ESP32-C3-DevKitM-1 是乐鑫一款**入门级开发板**，使用以尺寸小而得名的 **ESP32-C3-MINI-1 或 ESP32-C3-MINI-1U** 模组。该开发板具备完整的 Wi-Fi 和低功耗蓝牙功能，集成度高。

板上模组大部分管脚均已引出至两侧排针，开发者可轻松通过跳线连接外设，也可将开发板**插在面包板**上使用。

---

## 2. 核心规格

| 参数 | 规格 |
|------|------|
| 模组 | ESP32-C3-MINI-1（PCB 天线）/ MINI-1U（外接天线） |
| 芯片 | ESP32-C3（RISC-V 32 位单核，160 MHz） |
| 无线 | 2.4 GHz Wi-Fi + Bluetooth 5 (LE) |
| Flash | 4 MB（芯片封装内） |
| SRAM | 400 KB |
| USB 接口 | Micro-USB |
| RGB LED | WS2812，由 GPIO8 驱动 |

---

## 3. 板载组件

按逆时针顺序，主要组件：

| 组件 | 说明 |
|------|------|
| **ESP32-C3-MINI-1 / 1U** | 通用 Wi-Fi + BLE 模组，4 MB flash 封装于芯片内 |
| **5 V to 3.3 V LDO** | 电源转换，输入 5 V 输出 3.3 V |
| **5 V Power On LED** | 连接 USB 电源后点亮 |
| **Pin Headers（J1/J3）** | 所有可用 GPIO（除 flash SPI 总线）引出 |
| **Boot Button** | 下载键，按住 Boot + 按 Reset 进入固件下载模式 |
| **Micro-USB Port** | 供电或 PC 与芯片通信接口 |
| **Reset Button** | 复位按键 |
| **USB-to-UART Bridge** | 单芯片 USB 转 UART，最高 3 Mbps |
| **RGB LED** | 可寻址 RGB LED（WS2812），GPIO8 驱动 |

---

## 4. 电源选项

三种供电方式任选其一：

| 方式 | 说明 |
|------|------|
| **Micro-USB 接口**（默认、推荐） | 连接 USB 适配器或 PC |
| 5V + GND 排针 | 通过排针直接供电 |
| 3V3 + GND 排针 | 直接供 3.3 V |

---

## 5. 排针定义（J1 / J3）

### J1

| 序号 | 名称 | 类型 | 功能 |
|------|------|------|------|
| 1 | GND | G | 接地 |
| 2 | 3V3 | P | 3.3 V 电源 |
| 3 | 3V3 | P | 3.3 V 电源 |
| 4 | IO2 | I/O/T | GPIO2★, ADC1_CH2, FSPIQ |
| 5 | IO3 | I/O/T | GPIO3, ADC1_CH3 |
| 6 | GND | G | 接地 |
| 7 | RST | I | CHIP_PU |
| 8 | GND | G | 接地 |
| 9 | IO0 | I/O/T | GPIO0, ADC1_CH0, XTAL_32K_P |
| 10 | IO1 | I/O/T | GPIO1, ADC1_CH1, XTAL_32K_N |
| 11 | IO10 | I/O/T | GPIO10, FSPICS0 |
| 12 | GND | G | 接地 |
| 13 | 5V | P | 5 V 电源 |
| 14 | 5V | P | 5 V 电源 |
| 15 | GND | G | 接地 |

### J3

| 序号 | 名称 | 类型 | 功能 |
|------|------|------|------|
| 1 | GND | G | 接地 |
| 2 | TX | I/O/T | GPIO21, U0TXD |
| 3 | RX | I/O/T | GPIO20, U0RXD |
| 4 | GND | G | 接地 |
| 5 | IO9 | I/O/T | GPIO9★ |
| 6 | IO8 | I/O/T | GPIO8★, RGB LED |
| 7 | GND | G | 接地 |
| 8 | IO7 | I/O/T | GPIO7, FSPID, MTDO |
| 9 | IO6 | I/O/T | GPIO6, FSPICLK, MTCK |
| 10 | IO5 | I/O/T | GPIO5, ADC2_CH0, FSPIWP, MTDI |
| 11 | IO4 | I/O/T | GPIO4, ADC1_CH4, FSPIHD, MTMS |
| 12 | GND | G | 接地 |
| 13 | IO18 | I/O/T | GPIO18, USB_D- |
| 14 | IO19 | I/O/T | GPIO19, USB_D+ |
| 15 | GND | G | 接地 |

> 类型：P=电源，I=输入，O=输出，T=可设高阻。★ = strapping 管脚。

---

## 6. Strapping 管脚

ESP32-C3 系列的 **GPIO2、GPIO8、GPIO9** 为 strapping 管脚：

- GPIO2：浮空（建议上拉防毛刺）
- GPIO8：浮空
- GPIO9：弱上拉（默认 1）

| 启动模式 | GPIO9 | GPIO8 | GPIO2 |
|----------|:-----:|:-----:|:-----:|
| **SPI Boot（默认运行）** | 1 | x | 1 |
| Joint Download Boot（下载） | 0 | 1 | 1 |

- ⚠️ GPIO8 与 GPIO9 **不可同时为低**
- GPIO9 不要加大电容

> 详见 [02-ESP32-C3-Technical-Reference-Manual](02-ESP32-C3-Technical-Reference-Manual.md) 第 5 节。

---

## 7. 快速入门

### 硬件准备

- ESP32-C3-DevKitM-1
- USB 2.0 数据线（标准 A 转 Micro-B）
- 电脑（Windows / Linux / macOS）

> ⚠️ 务必用**数据线**，部分线仅充电无法下载。

### 下载模式

- 通过 USB-to-UART Bridge 下载：按住 **Boot** 键，短按 **Reset**，进入固件下载模式
- 也可用 ESP32-C3 原生 USB（GPIO18/19）下载，首次需手动拉低 GPIO9（按住 Boot 上电）

### 烧录固件

```bash
idf.py set-target esp32c3
idf.py -p <PORT> flash monitor
```

详细环境搭建见 ESP-IDF 快速入门（[14-ESP32-C3-Programming-Guide](14-ESP32-C3-Programming-Guide.md)）。

---

## 8. 应用场景

- ESP32-C3 入门学习与原型开发
- 低功耗 Wi-Fi + BLE 应用验证
- 面包板实验（适合插入面包板）
- 教学与培训

---

## 9. 参考资源

| 资源 | 链接 |
|------|------|
| DevKitM-1 用户指南（中文） | https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32c3/esp32-c3-devkitm-1/user_guide.html |
| DevKitM-1 索引页 | https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32c3/esp32-c3-devkitm-1/index.html |
| DevKitM-1 原理图（PDF） | https://dl.espressif.com/dl/schematics/SCH_ESP32-C3-DEVKITM-1_V1_20200915A.pdf |
| DevKitM-1 用户指南 PDF 下载 | https://www.espressif.com/zh-hans/support/download/documents/development-board |
| ESP-IDF 快速入门（C3） | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/get-started/index.html |
| DevKitM-1 上市新闻 | https://www.espressif.com/zh-hans/news/ESP32-C3-DevKitM-1 |

---

> **文档总结**：ESP32-C3-DevKitM-1 是 ESP32-C3 的入门级开发板，搭载超小模组 ESP32-C3-MINI-1/1U（4 MB 封装内 flash）。Micro-USB 供电/下载，板载 USB-to-UART Bridge（3 Mbps）、WS2812 RGB LED（GPIO8）、Boot/Reset 按键，GPIO 全引出至 J1/J3 排针，可直接插面包板。是 ESP32-C3 学习、原型与教学的首选开发板。
