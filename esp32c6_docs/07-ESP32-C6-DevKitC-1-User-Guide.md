# ESP32-C6-DevKitC-1 用户指南

> 文档编号：07
> 适用开发板：ESP32-C6-DevKitC-1 v1.2（搭载 ESP32-C6-WROOM-1 / WROOM-1U）
> 来源：乐鑫官方 esp-dev-kits 用户指南（v1.2 最新版）

---

## 目录

1. [开发板概述](#1-开发板概述)
2. [核心规格](#2-核心规格)
3. [板载组件](#3-板载组件)
4. [电源与电流测量](#4-电源与电流测量)
5. [排针定义](#5-排针定义)
6. [快速入门](#6-快速入门)
7. [应用场景](#7-应用场景)
8. [参考资源](#8-参考资源)

---

## 1. 开发板概述

ESP32-C6-DevKitC-1 是乐鑫基于 **ESP32-C6-WROOM-1 或 ESP32-C6-WROOM-1U** 模组的入门级开发板，具备完整的 **Wi-Fi 6、低功耗蓝牙、Zigbee 及 Thread 功能**。板上模组大部分管脚引出至两侧排针，可跳线连接外设或插入面包板。

v1.2 为当前最新版本。

---

## 2. 核心规格

| 参数 | 规格 |
|------|------|
| 模组 | ESP32-C6-WROOM-1（WROOM-1U 兼容） |
| 芯片 | ESP32-C6（RISC-V HP+LP 双核，160 MHz HP） |
| 无线 | 2.4 GHz Wi-Fi 6 + BT 5.3 LE + 802.15.4 |
| Flash | 8 MB（板载模组） |
| USB | **USB Type-C**（双口：OTG + UART） |
| RGB LED | WS2812，由 GPIO8 驱动 |
| 电流测量 | J5 跳针用于测量模组电流 |

---

## 3. 板载组件

| 组件 | 说明 |
|------|------|
| **ESP32-C6-WROOM-1/1U** | 三无线模组（Wi-Fi 6 + BT 5.3 + 802.15.4） |
| Pin Headers | 所有可用 GPIO 引出 |
| **5 V to 3.3 V LDO** | 电源转换 |
| **3.3 V Power On LED** | 通电指示 |
| **USB-to-UART Bridge** | USB-C UART 桥接，最高 3 Mbps |
| **ESP32-C6 USB Type-C** | 原生 USB OTG（烧录+调试+JTAG） |
| **Boot / Reset 按键** | Boot+Reset 进入下载模式 |
| **USB Type-C UART** | 供电+UART 通信 |
| **RGB LED** | WS2812，GPIO8 |
| **J5 电流测量跳针** | 测量模组电流（断开后外接电流表） |

---

## 4. 电源与电流测量

| 供电方式 | 说明 |
|----------|------|
| USB-C（OTG 口）供电 | 推荐 |
| USB-C（UART 口）供电 | |
| 5V + GND 排针供电 | |
| 3.3V + GND 排针供电 | |

### 电流测量

- J5 跳针：断开后可在 J5 处串入电流表测量模组实际电流（不含板上 LDOs/LED 功耗）
- 推荐用 **Joulescope** 或 **Nordic PPK2** 测动态电流

---

## 5. 排针定义

> 具体引脚排列请查阅官方用户指南或原理图。排针引出了 ESP32-C6 所有可用 GPIO（除 flash SPI 总线），包括 USB、UART、TWAI、MCPWM、PARLIO 等。

---

## 6. 快速入门

### 硬件准备

- ESP32-C6-DevKitC-1
- USB-C 数据线（数据传输型，非仅充电）
- 电脑

### 软件设置

```bash
idf.py set-target esp32c6
idf.py -p <PORT> flash monitor
```

> 详细请参考 [09-ESP32-C6-Programming-Guide](09-ESP32-C6-Programming-Guide.md) 与 [15-ESP32-C6-Getting-Started](15-ESP32-C6-Getting-Started.md)。

---

## 7. 应用场景

- ESP32-C6 三无线协议开发与评估
- Matter-over-Wi-Fi / Matter-over-Thread 产品原型
- Wi-Fi 6 + BLE 双模设备开发
- Thread/Zigbee 网格网络研究
- 电池供电 IoT（TWT 评估）

---

## 8. 参考资源

| 资源 | 链接 |
|------|------|
| DevKitC-1 用户指南（v1.2 最新） | https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32c6/esp32-c6-devkitc-1/user_guide.html |
| DevKitC-1 索引页 | https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32c6/esp32-c6-devkitc-1/index.html |
| 原理图（PDF） | https://dl.espressif.com/dl/schematics/ESP32-C6-DevKitC-1_v1.2.pdf |
| ESP-IDF 快速入门（C6） | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c6/get-started/index.html |

---

> **文档总结**：ESP32-C6-DevKitC-1 v1.2 是 C6 的入门级开发板，搭载 ESP32-C6-WROOM-1/1U（Wi-Fi 6 + BT 5.3 + 802.15.4），8 MB Flash。USB Type-C 双口（原生 OTG + USB-UART Bridge），RGB LED（GPIO8），J5 跳针支持模组电流测量，Boot/Reset 双按键。适合 Matter/Thread/Wi-Fi 6/BLE 三无线开发与评估。
