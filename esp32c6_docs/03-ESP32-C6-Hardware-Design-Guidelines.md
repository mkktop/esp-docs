# ESP32-C6 硬件设计指南详解

> 文档编号：03
> 适用范围：基于 ESP32-C6 芯片的产品硬件设计
> 来源：乐鑫官方《ESP32-C6 硬件设计指南》、原理图设计 checklist

---

## 目录

1. [设计指南概述](#1-设计指南概述)
2. [核心电路组成](#2-核心电路组成)
3. [电源设计](#3-电源设计)
4. [双核供电域](#4-双核供电域)
5. [Strapping 管脚](#5-strapping-管脚)
6. [RF 设计](#6-rf-设计)
7. [USB 与下载](#7-usb-与下载)
8. [PCB 布局要点](#8-pcb-布局要点)
9. [参考资源](#9-参考资源)

---

## 1. 设计指南概述

ESP32-C6 硬件设计指南覆盖基于该芯片的产品设计规范。与 C3 相比，主要区别：

| 维度 | ESP32-C6 | ESP32-C3 |
|------|-----------|-----------|
| 内核 | HP + LP 双核 | 单核 |
| 电压域 | 更多（VDD33N、VDD33S 等） | 3 域 |
| USB | **USB 2.0 OTG**（全速 12 Mbps） | USB Serial/JTAG |
| 天线共用 | Wi-Fi + BT + 802.15.4 三合一 | Wi-Fi + BT |

---

## 2. 核心电路组成

| 部分 | 说明 |
|------|------|
| 电源 | VDD33N（网络）、VDD33S（系统）、VDD33RF（射频）等 |
| 上电时序 | CHIP_EN、VDD 稳定时序 |
| Flash | 外接 SPI flash（C6 无内置 flash 型号） |
| 时钟源 | 主晶振 40 MHz、可选 32.768 kHz RTC 晶振 |
| 射频 | 天线匹配，Wi-Fi/BT/802.15.4 共用 |
| Strapping | 启动模式 |
| GPIO / USB | 外设 |

---

## 3. 电源设计

### 3.1 关键电源管脚

| 电源域 | 管脚 | 说明 |
|--------|------|------|
| VDD33N | 网络/无线 | Wi-Fi/BT/802.15.4 供电 |
| VDD33S | 系统 | HP 核、数字外设供电 |
| VDD33RF | 射频 | RF 模块供电 |
| VDD33A | 模拟 | ADC 等模拟外设 |
| VDD33RTC | RTC | LP 核、RTC 相关供电 |

### 3.2 设计要点

- 推荐 **3.3 V** 供电，输出电流 ≥ 500 mA
- 各电源域就近加 **0.1 µF** 去耦电容
- VDD33RF 在 TX 期间瞬态电流大，需加 **10 µF** 大电容防轨道塌陷
- 建议加 **ESD 保护** + **10 µF** 入口电容

---

## 4. 双核供电域

ESP32-C6 的 HP + LP 双核带来更细粒度的电源管理：

- HP 核和 LP 核可**独立上下电**
- LP 核可在 HP 核 Deep-sleep 时**独立运行**
- 设计时确保 LP 域供电稳定（LP 核独立低功耗任务需要）

---

## 5. Strapping 管脚

ESP32-C6 的 strapping 管脚控制启动模式，具体引脚和组合与 C3 不同，请以官方 TRM 为准。设计时注意：

- Strapping 管脚在复位时采样，复位后作普通 GPIO
- 建议在 strapping 管脚预留上拉/下拉电路
- GPIO8/9 等不可同时为特定组合时需规避

---

## 6. RF 设计

ESP32-C6 的 Wi-Fi + Bluetooth + 802.15.4 **共用同一 2.4 GHz 射频和天线**：

- RF 匹配网络：π 型或 L 型，匹配 50 Ω
- 天线接口：PCB 天线（模组内置）或 IPEX 座子（外接）
- 三协议共存时时分复用射频，需在 firmware 侧处理时分调度

---

## 7. USB 与下载

### 7.1 USB 全速 OTG（C6 独有）

ESP32-C6 支持 **USB 2.0 OTG**（全速 12 Mbps），可作为：
- USB 设备（烧录、CDC 串口）
- USB 主机（连接 USB 设备）

### 7.2 下载方式

| 方式 | 接口 |
|------|------|
| **USB OTG 烧录** | USB-C 接口（推荐） |
| UART0 下载 | GPIO20/21 |
| JTAG 调试 | 内置 USB JTAG |

> C6 相比 C3 的 USB Serial/JTAG（只设备），增加了 **OTG 主机模式**。

---

## 8. PCB 布局要点

- 电源去耦电容就近放置
- VDD33RF 走线加宽，TX 电流峰值可达数百 mA
- RF 走线控制 50 Ω 阻抗，靠近 LNA_IN
- USB D± 差分对等长（90 Ω 差分阻抗）
- 建议 4 层板

---

## 9. 参考资源

| 资源 | 链接 |
|------|------|
| ESP32-C6 硬件设计指南（在线） | https://docs.espressif.com/projects/esp-hardware-design-guidelines/zh_CN/latest/esp32c6/index.html |
| 原理图设计 checklist | https://docs.espressif.com/projects/esp-hardware-design-guidelines/zh_CN/latest/esp32c6/schematic-checklist.html |
| 硬件设计指南 PDF | https://www.espressif.com/sites/default/files/documentation/esp32-c6_hardware_design_guidelines_cn.pdf |
| 乐鑫 KiCad 库 | https://github.com/espressif/kicad-libraries |
| Flash 下载工具 | https://docs.espressif.com/projects/esp-test-tools/zh_CN/latest/esp32/production_stage/tools/flash_download_tool.html |

---

> **文档总结**：ESP32-C6 硬件设计核心是双核供电域（HP/LP 独立）、三无线协议共用 RF（Wi-Fi 6 + BT 5.3 + 802.15.4）、USB 2.0 OTG 全速 12 Mbps（主机/设备模式）三大新特性。电源分 VDD33N/S/RF/A/RTC 多域，TX 峰值电流大需重点注意去耦。USB OTG 是 C6 相对 C3 的重大升级。配合 TRM（[02](02-ESP32-C6-Technical-Reference-Manual.md)）与 Datasheet（[01](01-ESP32-C6-Datasheet.md)）使用。
