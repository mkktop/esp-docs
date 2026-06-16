# ESP32-C6 Wi-Fi 6 / Matter / Thread 开发指南

> 文档编号：11
> 适用芯片：ESP32-C6（Wi-Fi 6 + BT 5.3 + 802.15.4）
> 来源：乐鑫官方 ESP-IDF Wi-Fi 指南、ESP-Matter、OpenThread 文档

---

## 目录

1. [三协议生态概述](#1-三协议生态概述)
2. [Wi-Fi 6 开发](#2-wi-fi-6-开发)
3. [Matter 概述](#3-matter-概述)
4. [Matter-over-Wi-Fi](#4-matter-over-wi-fi)
5. [Matter-over-Thread](#5-matter-over-thread)
6. [Thread 边界路由器](#6-thread-边界路由器)
7. [选型建议](#7-选型建议)
8. [参考资源](#8-参考资源)

---

## 1. 三协议生态概述

ESP32-C6 是乐鑫 **Matter 全协议芯片**，同时支持：

```
Wi-Fi 6  ←→  Matter-over-Wi-Fi
802.15.4 ←→  Matter-over-Thread
BT 5.3   ←→  BLE 配置/配网
```

| 协议 | 作用 | C6 支持 |
|------|------|--------|
| **Matter** | 统一智能家居协议，跨平台/生态 | Wi-Fi + Thread 双支持 |
| **Wi-Fi 6** | 高带宽、本地/云通信 | 802.11ax OFDMA/MU-MIMO/TWT |
| **Thread** | 低功耗 mesh，专为 Matter 设计 | 802.15.4，Self-healing mesh |
| **Bluetooth LE** | 设备配网、BLE 控制 | BT 5.3 |

---

## 2. Wi-Fi 6 开发

### Wi-Fi 6 核心特性

| 特性 | 说明 |
|------|------|
| OFDMA | 上行/下行多用户同时传输，高密度场景 |
| MU-MIMO | 下行多用户空间复用 |
| TWT | 目标唤醒时间，长续航电池设备 |
| BSS Color | 空间复用，减少同信道干扰 |
| 20 MHz only | 2.4 GHz 抗干扰模式 |

### Wi-Fi 6 应用场景

- 高密度设备环境（多设备同时通信）
- 电池供电（TWT 协商）
- 需要低延迟的实时控制

---

## 3. Matter 概述

**Matter**（前称 CHIP）是 CSA 联盟推出的统一智能家居协议：

| 特性 | 说明 |
|------|------|
| 跨平台 | Alexa / Google Home / Apple HomeKit / Home Assistant |
| 本地通信 | 不依赖云 |
| 强安全 | AES-128、应用层加密 |
| 设备类型 | 灯/开关/插座/传感器/锁/暖通等 |

ESP32-C6 支持 **Matter-over-Wi-Fi** 和 **Matter-over-Thread** 两种设备类型。

---

## 4. Matter-over-Wi-Fi

ESP32-C6 可作为 **Matter Wi-Fi 终端设备**（灯/开关/插座等）：

- 直接连接 Wi-Fi AP，通过 Matter 协议通信
- 无需额外网关
- 适合：墙壁开关、插座、传感器等电池或常电设备

### Matter Wi-Fi 设备开发

```bash
# ESP-Matter SDK（C6 支持）
# 参考 esp-matter 文档
```

---

## 5. Matter-over-Thread

ESP32-C6 可作为 **Matter Thread 终端设备**：

- 通过 Thread mesh 网络通信
- 低功耗、Self-healing mesh
- 适合：电池供电的传感器/开关
- 需要 Thread Border Router 才能接入 Matter 网络

### Thread 网络架构

```
Thread 网络
  ├── Thread 终端设备（低功耗）
  ├── Thread 路由器（路由）
  └── Thread Border Router（Wi-Fi/Ethernet → Matter）
              ↑
         ESP32-C6（或搭配 ESP32-S3）
```

---

## 6. Thread 边界路由器

Thread Border Router（边界路由器）是 Thread 网络接入 Matter 的网关：

| 方案 | 说明 |
|------|------|
| **ESP32-C6 + ESP32-S3** | C6 做 Thread BR，S3 做 Matter 控制器 |
| ESP32-C6 独立 | C6 同时跑 Thread BR + Matter Controller（资源有限） |

> 注意：ESP32-C6 **不适合独立作 Thread Border Router**（资源/性能限制），建议搭配 ESP32-S3 或专用网关 SoC。

---

## 7. 选型建议

| 需求 | 推荐方案 |
|------|----------|
| Matter Wi-Fi 终端设备 | ESP32-C6 |
| Matter Thread 终端设备 | ESP32-C6 + Thread BR |
| Thread Border Router | **ESP32-S3** 或专用网关 |
| Wi-Fi 6 高密度设备 | ESP32-C6 |
| Wi-Fi 6 TWT 长续航 | ESP32-C6 |
| BLE + Wi-Fi 双模 | ESP32-C6 |

---

## 8. 参考资源

| 资源 | 链接 |
|------|------|
| ESP-IDF Wi-Fi 指南（C6） | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c6/api-guides/wifi.html |
| OpenThread 文档 | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c6/api-guides/openthread.html |
| ESP-Matter | https://github.com/espressif/esp-matter |
| Matter 协议规格 | https://csa-iot.org/builders/ |
| Thread 规格 | https://www.threadgroup.org/ |

---

> **文档总结**：ESP32-C6 是乐鑫 Matter 全协议芯片，支持 Matter-over-Wi-Fi（直接 Wi-Fi 连接）和 Matter-over-Thread（通过 Thread mesh 连接）。Wi-Fi 6 提供 OFDMA/MU-MIMO/TWT/BSS Color 能力；Thread 基于 802.15.4 低功耗 Self-healing mesh；BLE 5.3 用于配网。C6 适合 Matter Wi-Fi/Thread 终端设备；Thread Border Router 建议用 ESP32-S3 等更高性能芯片。配合 [09-Programming-Guide](09-ESP32-C6-Programming-Guide.md) 使用。
