# ESP32-C6-DevKitM-1 用户指南

> 文档编号：08
> 适用开发板：ESP32-C6-DevKitM-1（搭载 ESP32-C6-MINI-1 / MINI-1U）
> 来源：乐鑫官方 esp-dev-kits 用户指南

---

## 目录

1. [开发板概述](#1-开发板概述)
2. [核心规格](#2-核心规格)
3. [板载组件](#3-板载组件)
4. [电源与电流测量](#4-电源与电流测量)
5. [与 DevKitC-1 的区别](#5-与-devkitc-1-的区别)
6. [应用场景](#6-应用场景)
7. [参考资源](#7-参考资源)

---

## 1. 开发板概述

ESP32-C6-DevKitM-1 是乐鑫基于 **ESP32-C6-MINI-1 或 ESP32-C6-MINI-1U** 模组的入门级开发板，具备完整的 **Wi-Fi 6、低功耗蓝牙、Zigbee 及 Thread** 功能。相比 DevKitC-1，它使用超小模组，尺寸更紧凑。

板上模组大部分管脚引出至两侧排针，可跳线连接外设或插入面包板。

---

## 2. 核心规格

| 参数 | 规格 |
|------|------|
| 模组 | ESP32-C6-MINI-1（PCB 天线）/ MINI-1U（外接天线） |
| 芯片 | ESP32-C6（RISC-V HP+LP 双核） |
| 无线 | 2.4 GHz Wi-Fi 6 + BT 5.3 LE + 802.15.4 |
| Flash | 4 MB（模组内置） |
| USB | **USB Type-C**（双口：OTG + UART） |
| RGB LED | WS2812，GPIO8 |
| 电流测量 | J5 跳针 |

---

## 3. 板载组件

| 组件 | 说明 |
|------|------|
| **ESP32-C6-MINI-1/1U** | 超小模组（13.2×16.6 mm） |
| Pin Headers | 所有可用 GPIO 引出 |
| **5 V to 3.3 V LDO** | 电源转换 |
| **3.3 V Power On LED** | 通电指示 |
| **USB-to-UART Bridge** | USB-C UART 桥接 |
| **ESP32-C6 USB Type-C** | 原生 USB OTG |
| **Boot / Reset 按键** | 下载 + 复位 |
| **USB Type-C UART** | 供电+UART |
| **RGB LED** | WS2812，GPIO8 |
| **J5 电流测量跳针** | 测量模组电流 |

---

## 4. 电源与电流测量

与 DevKitC-1 完全相同：USB-C 双口供电，J5 跳针测量模组电流（推荐 Joulescope / PPK2）。

---

## 5. 与 DevKitC-1 的区别

| 维度 | DevKitM-1 | DevKitC-1 |
|------|------------|-----------|
| 模组 | **ESP32-C6-MINI-1**（超小，13.2×16.6 mm） | ESP32-C6-WROOM-1（18×25.5 mm） |
| Flash | 4 MB 内置 | 8 MB 外置 |
| 尺寸 | 更小 | 较大 |
| 定位 | 极小模组入门 | 通用 WROOM 模组入门 |

> 二者软件完全兼容，硬件仅模组和 GPIO 数量略有差异。

---

## 6. 应用场景

- ESP32-C6 三无线协议开发（MINI-1 极小模组版）
- Matter-over-Wi-Fi / Matter-over-Thread 产品原型
- 超小尺寸 Wi-Fi 6 + BLE + Thread IoT 设备评估
- 空间受限的原型设计

---

## 7. 参考资源

| 资源 | 链接 |
|------|------|
| DevKitM-1 用户指南（中文） | https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32c6/esp32-c6-devkitm-1/user_guide.html |
| DevKitM-1 索引页 | https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32c6/esp32-c6-devkitm-1/index.html |
| ESP-IDF 快速入门（C6） | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c6/get-started/index.html |

---

> **文档总结**：ESP32-C6-DevKitM-1 是 DevKitM-1 系列的 C6 版本，搭载 ESP32-C6-MINI-1/1U 超小模组（13.2×16.6 mm，内置 4 MB flash），与 DevKitC-1（搭载 WROOM-1，8 MB 外置 flash）互补。两者软件完全兼容，DevKitM-1 更适合极小尺寸原型。配套 USB Type-C 双口、J5 电流测量、RGB LED，适合 C6 Wi-Fi 6/Thread/Matter/BLE 全协议评估。
