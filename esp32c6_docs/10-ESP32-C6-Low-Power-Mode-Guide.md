# ESP32-C6 低功耗模式指南

> 文档编号：10
> 适用芯片：ESP32-C6（HP + LP 双核架构）
> 来源：乐鑫官方 ESP-IDF 低功耗模式指南

---

## 目录

1. [低功耗概述](#1-低功耗概述)
2. [六种功耗模式](#2-六种功耗模式)
3. [HP + LP 双核低功耗策略](#3-hp--lp-双核低功耗策略)
4. [Wi-Fi 6 TWT 省电](#4-wi-fi-6-twt-省电)
5. [BLE 监听与 LP 核](#5-ble-监听与-lp-核)
6. [Deep-sleep 与 Hibernation](#6-deep-sleep-与-hibernation)
7. [参考资源](#7-参考资源)

---

## 1. 低功耗概述

ESP32-C6 是乐鑫**低功耗标杆芯片**，相比 C3 更进一步：

| 创新 | 说明 |
|------|------|
| **LP 核** | HP 核休眠时，LP 核可独立运行 BLE 监听等低功耗任务 |
| **Wi-Fi 6 TWT** | 设备与 AP 协商唤醒时间表，深度睡眠时间最长 |
| **三协议时分复用** | Wi-Fi + BT + 802.15.4 共存时的精细功耗调度 |

---

## 2. 六种功耗模式

| 模式 | HP 核 | LP 核 | 说明 |
|------|-------|-------|------|
| Active | 运行 | 运行/休眠 | 全速 |
| Modem-sleep | 休眠 | 可运行 | Wi-Fi/BT 关闭，LP 可处理事件 |
| Light-sleep | 休眠 | 可运行 | RTC 工作，状态保留 |
| Deep-sleep | 关闭 | **独立运行** | LP 核接管 BLE 监听/TWT |
| Hibernation | 关闭 | 关闭 | 仅 RTC 定时器 |
| Off | 关闭 | 关闭 | CHIP_EN 低 |

---

## 3. HP + LP 双核低功耗策略

这是 C6 相对 C3 的核心优势：**HP 和 LP 核可以独立休眠**。

### 典型场景

| 场景 | HP 核 | LP 核 | 功耗优势 |
|------|-------|-------|----------|
| BLE 广播监听 | Deep-sleep | **独立运行 BLE 监听** | HP 关闭，LP 处理事件 |
| Wi-Fi TWT | Light-sleep | LP 可运行 | TWT 协商后 HP 长睡 |
| 传感器上报 | Deep-sleep | 定时唤醒 | LP 独立处理传感器 |

### LP 核独立任务

LP 核可处理：BLE 监听、TWT 事件、低功耗传感器轮询。

---

## 4. Wi-Fi 6 TWT 省电

TWT（Target Wake Time）是 Wi-Fi 6 最关键的节能特性：

- 设备与 AP 协商"唤醒时间表"（双方约定下次通信时间）
- 设备只在约定时间唤醒，其余时间深度睡眠
- **TWT 相比传统 DTIM**：深度睡眠时间可达 90% 以上
- 适合：定期上报的传感器、电池供电设备

---

## 5. BLE 监听与 LP 核

ESP32-C6 的 BLE 监听可由 **LP 核独立处理**：

- HP 核可以完全关闭（Deep-sleep）
- LP 核独立运行 BLE Host + Controller
- BLE 收到特定广播包后唤醒 HP 核处理
- 功耗：**LP 核 20 MHz 运行 BLE 监听**，远低于 HP 核 160 MHz

---

## 6. Deep-sleep 与 Hibernation

| 模式 | LP 核 | RTC | 保留寄存器 |
|------|-------|-----|------------|
| Deep-sleep | **独立运行** | 工作 | 可保留 |
| Hibernation | 关闭 | 仅 RTC 定时器 | 仅 8 个保留寄存器 |

### Deep-sleep 唤醒源

| 唤醒源 | 支持 |
|--------|:----:|
| RTC Timer | ✓ |
| GPIO | ✓ |
| Wi-Fi 唤醒（TWT） | ✓（由 LP 核处理） |
| BLE 唤醒 | ✓（由 LP 核处理） |

---

## 7. 参考资源

| 资源 | 链接 |
|------|------|
| ESP-IDF 低功耗模式（C6） | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c6/api-guides/low-power-mode/index.html |
| LP Core 例程 | https://github.com/espressif/esp-idf/tree/master/examples/system/lp_core |
| Wi-Fi 6 TWT | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c6/api-guides/wifi.html#wi-fi-6-twt |
| Joulescope | https://www.joulescope.com/ |

---

> **文档总结**：ESP32-C6 的低功耗核心是 HP+LP 双核独立休眠 + Wi-Fi 6 TWT + BLE LP 核独立监听。HP 核完全关闭时，LP 核可独立处理 BLE 广播监听、TWT 事件、传感器轮询。Wi-Fi 6 TWT 使设备与 AP 协商唤醒时间表，深度睡眠可达 90%+。Deep-sleep（LP 核独立运行）与 Hibernation（双核关闭）构成完整低功耗层级。配合 [09-Programming-Guide](09-ESP32-C6-Programming-Guide.md) 使用。
