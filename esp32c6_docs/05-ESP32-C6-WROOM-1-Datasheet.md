# ESP32-C6-WROOM-1 / WROOM-1U 模组数据手册

> 文档编号：05
> 适用模组：ESP32-C6-WROOM-1（PCB 天线）、ESP32-C6-WROOM-1U（外接天线座子）
> 来源：乐鑫官方《ESP32-C6-WROOM-1 & WROOM-1U 技术规格书》v1.4

---

## 目录

1. [模组概述](#1-模组概述)
2. [核心规格](#2-核心规格)
3. [WROOM-1 vs WROOM-1U](#3-wroom-1-vs-wroom-1u)
4. [Wi-Fi 6 + BT 5.3 + 802.15.4 三无线](#4-wi-fi-6--bt-53--802154-三无线)
5. [外设接口](#5-外设接口)
6. [模组型号对比](#6-模组型号对比)
7. [对应开发板](#7-对应开发板)
8. [应用场景](#8-应用场景)
9. [参考资源](#9-参考资源)

---

## 1. 模组概述

ESP32-C6-WROOM-1 / WROOM-1U 是乐鑫基于 **ESP32-C6** 芯片的通用型三无线模组，支持 **2.4 GHz Wi-Fi 6、Bluetooth 5、Zigbee 3.0 及 Thread 1.3**，管脚**兼容 ESP32-WROOM 系列**（经典尺寸），是老产品线升级到 Wi-Fi 6 + Matter 的理想方案。

- **ESP32-C6-WROOM-1**：PCB 板载天线
- **ESP32-C6-WROOM-1U**：外部天线连接器（IPEX）

---

## 2. 核心规格

| 参数 | 规格 |
|------|------|
| 集成芯片 | ESP32-C6（RISC-V HP+LP 双核） |
| 主频 | HP 核最高 160 MHz |
| 无线 | 2.4 GHz Wi-Fi 6 (802.11ax) + BT 5.3 LE + 802.15.4 |
| ROM | 320 KB |
| HP SRAM | 512 KB |
| LP SRAM | 16 KB |
| **Flash** | **外接 Quad SPI，最大 8 MB / 16 MB** |
| 工作电压 | 3.0 ~ 3.6 V（推荐 3.3 V） |
| GPIO | 23（WROOM-1）/ 29（WROOM-1U） |

> 注意：WROOM-1/G 的 GPIO 数不同，因为某些 GPIO 用于连接外置 flash。

---

## 3. WROOM-1 vs WROOM-1U

| 项目 | WROOM-1 | WROOM-1U |
|------|---------|-----------|
| 天线 | PCB 板载天线 | IPEX 座子（外接天线） |
| 尺寸 | 18×25.5×3.1 mm | 18×19.2×3.2 mm |
| GPIO | 23 | 29 |
| Flash | 4/8/16 MB | 4/8/16 MB |

---

## 4. Wi-Fi 6 + BT 5.3 + 802.15.4 三无线

| 协议 | 特性 |
|------|------|
| **Wi-Fi 6 (802.11ax)** | OFDMA、MU-MIMO、TWT、BSS Color、向下兼容 b/g/n |
| **Bluetooth 5.3 LE** | 2M PHY、Coded PHY、远距离、LE Power Control |
| **IEEE 802.15.4** | Thread 1.3（认证）、Zigbee 3.0（认证） |

三协议共用 2.4 GHz 天线，firmware 侧时分调度共存。

---

## 5. 外设接口

| 接口 | 说明 |
|------|------|
| GPIO | 23/29 可编程 GPIO |
| SPI × 3 | SPI0/1（flash）、SPI2（通用） |
| UART × 2 | |
| I2C × 2 | |
| I2S × 1 | |
| LED PWM | 多通道 |
| RMT | TX/RX |
| TWAI® | CAN 2.0 兼容 |
| MCPWM | 电机控制 |
| PARLIO | 并行 IO |
| PCNT | 脉冲计数器 |
| ADC | 12 位 SAR |
| USB 2.0 OTG | 全速 12 Mbps |

---

## 6. 模组型号对比

| 型号 | Flash | 环境温度 | 尺寸 |
|------|-------|----------|------|
| ESP32-C6-WROOM-1-**N4** | 4 MB | -40~85 °C | 18×25.5×3.1 mm |
| ESP32-C6-WROOM-1-**N8** | 8 MB | -40~85 °C | 18×25.5×3.1 mm |
| ESP32-C6-WROOM-1-**N16** | 16 MB | -40~85 °C | 18×25.5×3.1 mm |
| ESP32-C6-WROOM-1U-N4/8/16 | 同上 | 同上 | 18×19.2×3.2 mm |

---

## 7. 对应开发板

| 开发板 | 说明 |
|--------|------|
| **ESP32-C6-DevKitC-1** | 入门级开发板，搭载 WROOM-1/1U，USB-C + RGB LED |

---

## 8. 应用场景

- Wi-Fi 6 + Matter-over-Wi-Fi 智能家居设备
- Thread/Zigbee 终端（Matter-over-Thread）
- Wi-Fi 6 长续航电池设备（TWT）
- BLE + Wi-Fi 双模产品
- 工业物联网（Wi-Fi + Thread 双协议网关）

---

## 9. 参考资源

| 资源 | 链接 |
|------|------|
| ESP32-C6-WROOM-1 规格书（中文 PDF） | https://www.espressif.com/sites/default/files/documentation/esp32-c6-wroom-1_wroom-1u_datasheet_cn.pdf |
| WROOM-1 在线规格书 | https://documentation.espressif.com/esp32-c6-wroom-1_wroom-1u_datasheet_cn.html |
| WROOM-1 PCB Footprint（dxf） | https://www.espressif.com/sites/default/files/modules-dxf/ESP32-C6-WROOM-1%20PCB%20Footprint.dxf |
| 模组产品页 | https://www.espressif.com/zh-hans/products/modules?id=ESP32-C6 |

---

> **文档总结**：ESP32-C6-WROOM-1/1U 是 ESP32-C6 的通用三无线模组（Wi-Fi 6 + BT 5.3 + 802.15.4 Thread/Zigbee），外接最大 16 MB SPI flash。WROOM-1（18×25.5 mm，PCB 天线）与 WROOM-1U（18×19.2 mm，IPEX 座）管脚兼容经典 ESP32-WROOM 系列，便于从老产品线迁移。GPIO 23/29，支持 USB OTG、UART/I2C/SPI/TWAI/MCPWM/PARLIO 等丰富外设。是 Matter Wi-Fi/Thread 智能家居、Wi-Fi 6 长续航电池设备、双模物联网产品的优选模组。
