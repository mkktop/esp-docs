# ESP32-C3-DevKitC-02 用户指南

> 文档编号：10
> 适用开发板：ESP32-C3-DevKitC-02（搭载 ESP32-C3-WROOM-02 / 02U）
> 来源：乐鑫官方 esp-dev-kits 用户指南

---

## 目录

1. [开发板概述](#1-开发板概述)
2. [核心规格](#2-核心规格)
3. [板载组件](#3-板载组件)
4. [电源选项](#4-电源选项)
5. [排针定义（J1 / J3）](#5-排针定义j1--j3)
6. [Strapping 管脚与下载](#6-strapping-管脚与下载)
7. [与 DevKitM-1 的区别](#7-与-devkitm-1-的区别)
8. [快速入门](#8-快速入门)
9. [应用场景](#9-应用场景)
10. [参考资源](#10-参考资源)

---

## 1. 开发板概述

ESP32-C3-DevKitC-02 是乐鑫一款**入门级开发板**，搭载通用型 **ESP32-C3-WROOM-02**（或 02U）模组。它是 ESP32 经典 DevKitC 系列的 C3 版本，模组大部分管脚引出至两侧排针，便于通过跳线连接外设或插入面包板使用。

---

## 2. 核心规格

| 参数 | 规格 |
|------|------|
| 模组 | ESP32-C3-WROOM-02（PCB 天线）/ 02U（外接天线） |
| 芯片 | ESP32-C3（RISC-V 32 位单核，160 MHz） |
| 无线 | 2.4 GHz Wi-Fi + Bluetooth 5 (LE) |
| Flash | 4 MB（外部 SPI flash） |
| SRAM | 400 KB |
| USB 接口 | Micro-USB |
| RGB LED | WS2812，由 GPIO8 驱动 |

---

## 3. 板载组件

| 组件 | 说明 |
|------|------|
| **ESP32-C3-WROOM-02 / 02U** | 通用 Wi-Fi + BLE 模组，4 MB 外部 SPI flash，PCB 天线（02U 为 U.FL 座） |
| **5 V to 3.3 V LDO** | 电源转换 |
| **Pin Headers（J1/J3）** | 所有可用 GPIO（除 flash SPI 总线）引出 |
| **Boot Button** | 下载键，Boot + Reset 进入固件下载模式 |
| **Micro-USB Port** | 供电与通信 |
| **Reset Button** | 复位 |
| **USB-to-UART Bridge** | 单芯片 USB 转 UART |
| **RGB LED** | WS2812，GPIO8 |

---

## 4. 电源选项

| 方式 | 说明 |
|------|------|
| **Micro-USB 接口**（默认、推荐） | USB 适配器或 PC |
| 5V + GND 排针 | 排针直接供电 |
| 3V3 + GND 排针 | 直接供 3.3 V |

---

## 5. 排针定义（J1 / J3）

### J1

| 序号 | 名称 | 类型 | 功能 |
|------|------|------|------|
| 1 | G | G | 接地 |
| 2 | 3V3 | P | 3.3 V |
| 3 | 3V3 | P | 3.3 V |
| 4 | RST | I | CHIP_PU |
| 5 | G | G | 接地 |
| 6 | 4 | I/O/T | GPIO4, ADC1_CH4, FSPIHD, MTMS |
| 7 | 5 | I/O/T | GPIO5, ADC2_CH0, FSPIWP, MTDI |
| 8 | 6 | I/O/T | GPIO6, FSPICLK, MTCK |
| 9 | 7 | I/O/T | GPIO7, FSPID, MTDO |
| 10 | G | G | 接地 |
| 11 | 8 | I/O/T | GPIO8★, RGB LED |
| 12 | 9 | I/O/T | GPIO9★ |
| 13 | 5V | P | 5 V |
| 14 | 5V | P | 5 V |
| 15 | G | G | 接地 |

### J3

| 序号 | 名称 | 类型 | 功能 |
|------|------|------|------|
| 1 | G | G | 接地 |
| 2 | 0 | I/O/T | GPIO0, ADC1_CH0, XTAL_32K_P |
| 3 | 1 | I/O/T | GPIO1, ADC1_CH1, XTAL_32K_N |
| 4 | 2 | I/O/T | GPIO2★, ADC1_CH2, FSPIQ |
| 5 | 3 | I/O/T | GPIO3, ADC1_CH3 |
| 6 | G | G | 接地 |
| 7 | 10 | I/O/T | GPIO10, FSPICS0 |
| 8 | G | G | 接地 |
| 9 | RX | I/O/T | GPIO20, U0RXD |
| 10 | TX | I/O/T | GPIO21, U0TXD |
| 11 | G | G | 接地 |
| 12 | 18 | I/O/T | GPIO18, USB_D- |
| 13 | 19 | I/O/T | GPIO19, USB_D+ |
| 14 | G | G | 接地 |
| 15 | G | G | 接地 |

> ★ = strapping 管脚（GPIO2、GPIO8、GPIO9）。

---

## 6. Strapping 管脚与下载

- Strapping：GPIO2、GPIO8、GPIO9
  - GPIO9 弱上拉（默认 1），GPIO2/8 浮空
  - SPI Boot（默认）：GPIO9=1
  - Download Boot：GPIO9=0, GPIO8=1
- 下载方式：UART0（GPIO20/21）或原生 USB（GPIO18/19）
- 首次用 USB 下载需手动拉低 GPIO9（按住 Boot 上电）
- ⚠️ GPIO8 与 GPIO9 不可同时为低

---

## 7. 与 DevKitM-1 的区别

| 维度 | DevKitC-02 | DevKitM-1 |
|------|------------|-----------|
| 模组 | ESP32-C3-WROOM-02（4 MB 外部 flash） | ESP32-C3-MINI-1（4 MB 封装内 flash） |
| 尺寸 | 较大（WROOM-02 模组） | 较小（MINI-1 超小模组） |
| 引脚 | 基本一致 | 基本一致 |
| 定位 | 经典 DevKitC 风格，GPIO 多 | 极小模组入门板 |

---

## 8. 快速入门

### 硬件准备

- ESP32-C3-DevKitC-02
- USB 2.0 数据线（A 转 Micro-B）
- 电脑

### 下载与烧录

```bash
idf.py set-target esp32c3
idf.py -p <PORT> flash monitor
```

进入下载模式：按住 Boot + 短按 Reset。

---

## 9. 应用场景

- ESP32-C3 入门学习与原型（GPIO 丰富）
- 从 ESP8266 / ESP-WROOM-02 迁移评估
- 低功耗 Wi-Fi + BLE 应用开发
- 面包板实验

---

## 10. 参考资源

| 资源 | 链接 |
|------|------|
| DevKitC-02 用户指南（中文） | https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32c3/esp32-c3-devkitc-02/user_guide.html |
| DevKitC-02 原理图（PDF） | https://dl.espressif.com/dl/schematics/SCH_ESP32-C3-DEVKITC-02_V1_1_20210126A.pdf |
| DevKitC-02 用户指南 PDF | https://www.espressif.com/zh-hans/support/download/documents/development-board |
| ESP-IDF 快速入门（C3） | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/get-started/index.html |

---

> **文档总结**：ESP32-C3-DevKitC-02 是 ESP32-C3 的入门级开发板，搭载 ESP32-C3-WROOM-02/02U 模组（4 MB 外部 flash）。Micro-USB 供电/下载，USB-to-UART Bridge、WS2812 RGB LED（GPIO8）、Boot/Reset 按键，GPIO 全引出至 J1/J3。与 DevKitM-1（超小 MINI-1 模组）相比，它用经典 WROOM-02 模组，适合 ESP8266 迁移评估与 GPIO 丰富的原型开发。
