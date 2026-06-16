# ESP32-C6 Zigbee 开发指南

> 文档编号：12
> 适用芯片：ESP32-C6（IEEE 802.15.4 + Zigbee 3.0）
> 来源：乐鑫官方 ESP-IDF Zigbee 文档、Zigbee 规格

---

## 目录

1. [Zigbee 概述](#1-zigbee-概述)
2. [ESP32-C6 Zigbee 能力](#2-esp32-c6-zigbee-能力)
3. [Zigbee 设备类型](#3-zigbee-设备类型)
4. [开发要点](#4-开发要点)
5. [与 Thread / Matter 的关系](#5与-thread--matter-的关系)
6. [参考资源](#6-参考资源)

---

## 1. Zigbee 概述

Zigbee 是基于 IEEE 802.15.4 的短距离、低功耗 mesh 网络协议，广泛用于智能照明与传感网络。

| 特性 | 说明 |
|------|------|
| 频段 | 2.4 GHz（全球通用） |
| 速率 | 250 Kbps |
| 拓扑 | 星型/树型/mesh（Self-healing） |
| 功耗 | 低（休眠电流 µA 级） |
| 设备类型 | Coordinator / Router / End Device |

---

## 2. ESP32-C6 Zigbee 能力

ESP32-C6 集成 **802.15.4 基带 + MAC**，支持 **Zigbee 3.0** 认证协议栈：

| 能力 | ESP32-C6 | ESP32-C3 |
|------|-----------|---------|
| 802.15.4 | ✓ | ✗ |
| Zigbee 3.0 | ✓ | ✗ |
| Thread | ✓ | ✗ |
| BLE 共存 | ✓ | ✓ |

---

## 3. Zigbee 设备类型

| 类型 | 功耗 | 说明 |
|------|------|------|
| Coordinator（协调器） | 高（常电） | 网络管理者，创建网络 |
| Router（路由器） | 中（常电） | 路由数据，扩展网络 |
| End Device（终端） | **极低**（电池） | 只与父节点通信，可休眠 |

---

## 4. 开发要点

```bash
# menuconfig → Enable Zigbee
idf.py build
idf.py flash monitor
```

ESP-IDF 提供完整 Zigbee 协议栈例程（zigbee/）。

---

## 5. 与 Thread / Matter 的关系

| 协议 | 底层 | 特点 |
|------|------|------|
| **Zigbee** | 802.15.4 | 智能照明/传感，生态成熟 |
| **Thread** | 802.15.4 | 更开放，Matter 基础层 |
| **Matter** | Wi-Fi / Thread | 统一智能家居，不依赖具体协议 |

ESP32-C6 同时支持 Zigbee 和 Thread，两者共享 802.15.4 硬件。

---

## 6. 参考资源

| 资源 | 链接 |
|------|------|
| ESP-IDF Zigbee 指南（C6） | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c6/api-guides/zigbee.html |
| Zigbee 3.0 规格 | https://csa-iot.org/ |

---

> **文档总结**：ESP32-C6 支持 Zigbee 3.0（IEEE 802.15.4），可作为 Coordinator / Router / End Device，配合 BLE 用于配网。Zigbee 与 Thread 共用 802.15.4 硬件，C6 同时支持两者。与 Matter 生态互通。
