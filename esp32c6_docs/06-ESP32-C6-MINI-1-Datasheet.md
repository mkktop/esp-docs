# ESP32-C6-MINI-1 / MINI-1U 模组数据手册

> 文档编号：06
> 适用模组：ESP32-C6-MINI-1（PCB 天线）、ESP32-C6-MINI-1U（外接天线座子）
> 来源：乐鑫官方《ESP32-C6-MINI-1 & MINI-1U 技术规格书》、产品页

---

## 目录

1. [模组概述](#1-模组概述)
2. [核心规格](#2-核心规格)
3. [MINI-1 vs MINI-1U](#3-mini-1-vs-mini-1u)
4. [三无线协议](#4-三无线协议)
5. [与 C3-MINI-1 的兼容性](#5-与-c3-mini-1-的兼容性)
6. [对应开发板](#6-对应开发板)
7. [应用场景](#7-应用场景)
8. [参考资源](#8-参考资源)

---

## 1. 模组概述

ESP32-C6-MINI-1 / MINI-1U 是乐鑫基于 **ESP32-C6** 芯片的超小尺寸三无线模组，支持 **2.4 GHz Wi-Fi 6、Bluetooth 5.3 LE、Zigbee 3.0 及 Thread 1.3**，**管脚兼容 ESP32-C3-MINI 系列**，是空间受限的 Matter / Thread / Wi-Fi 6 IoT 产品理想选择。

- **ESP32-C6-MINI-1**：PCB 板载天线
- **ESP32-C6-MINI-1U**：外部天线连接器

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
| **Flash** | **内置 4/8 MB（芯片封装内）** |
| 工作电压 | 3.0 ~ 3.6 V |
| 尺寸 | **13.2×16.6×2.4 mm**（MINI-1） |

---

## 3. MINI-1 vs MINI-1U

| 项目 | MINI-1 | MINI-1U |
|------|---------|-----------|
| 天线 | PCB 板载天线 | 外部天线座子（IPEX） |
| 尺寸 | 13.2×16.6×2.4 mm | 13.2×12.5×2.4 mm |
| GPIO | 22（QFN32 封装，53 个功能引脚） | 22 |
| Flash | 4/8 MB 内置 | 4 MB 内置 |

---

## 4. 三无线协议

| 协议 | 说明 |
|------|------|
| **Wi-Fi 6** | OFDMA、TWT、MU-MIMO 下行、BSS Color、802.11b/g/n 兼容 |
| **Bluetooth 5.3 LE** | 2M PHY、Coded PHY（远距离）、LE Power Control |
| **IEEE 802.15.4** | Thread 1.3 认证 + Zigbee 3.0 认证 |

---

## 5. 与 C3-MINI-1 的兼容性

ESP32-C6-MINI-1 **管脚兼容 ESP32-C3-MINI-1**，主要区别：

| 维度 | ESP32-C6-MINI-1 | ESP32-C3-MINI-1 |
|------|------------------|-------------------|
| 芯片 | ESP32-C6（双核） | ESP32-C3（单核） |
| Wi-Fi | Wi-Fi **6** (802.11ax) | Wi-Fi 4 (802.11n) |
| BLE | BT 5.3 | BT 5 |
| 802.15.4 | **Thread + Zigbee** | 无 |
| USB | USB OTG | USB Serial/JTAG |
| SRAM | 512 KB HP + 16 KB LP | 400 KB |
| 升级 | 从 C3-MINI-1 升级 | — |

> 管脚兼容 = 同一 PCB 可以换焊 C3-MINI-1 ↔ C6-MINI-1（需软件适配）。

---

## 6. 对应开发板

| 开发板 | 说明 |
|--------|------|
| **ESP32-C6-DevKitM-1** | 入门级开发板，搭载 MINI-1/1U，USB-C + RGB LED |

---

## 7. 应用场景

- 超小 Matter-over-Wi-Fi / Matter-over-Thread 终端设备
- 空间受限的智能家居传感器/执行器
- Thread/Zigbee 网格网络节点
- Wi-Fi 6 + BLE 双模设备
- 电池供电、需 TWT 省电的长续航 IoT

---

## 8. 参考资源

| 资源 | 链接 |
|------|------|
| ESP32-C6-MINI-1 规格书（中文 PDF） | https://www.espressif.com/sites/default/files/documentation/esp32-c6-mini-1_mini-1u_datasheet_cn.pdf |
| MINI-1 PCB Footprint（dxf） | https://www.espressif.com/sites/default/files/modules-dxf/ESP32-C6-MINI-1%20PCB%20Footprint.dxf |
| MINI-1 3D 模型（STEP） | https://www.espressif.com/sites/default/files/3dmodel/ESP32-C6-MINI-1.STEP |
| 模组产品页 | https://www.espressif.com/zh-hans/products/modules?id=ESP32-C6 |

---

> **文档总结**：ESP32-C6-MINI-1/1U 是 ESP32-C6 的超小尺寸模组（13.2×16.6 mm），内置 4/8 MB flash，管脚兼容 C3-MINI-1（同一 PCB 可换）。Wi-Fi 6 + BT 5.3 + 802.15.4（Thread/Zigbee）三无线协议共存，比 C3-MINI-1 大幅升级（Wi-Fi 6 TWT 省电、Thread/Matter 全支持、USB OTG）。DevKitM-1 是配套开发板。是超小尺寸 Matter / Thread / Wi-Fi 6 IoT 产品的首选模组。
