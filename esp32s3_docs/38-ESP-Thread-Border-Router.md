# ESP Thread Border Router 详解 — Thread 边界路由器方案

> 文档编号：38
> 适用芯片：ESP32-S3（Wi-Fi 主机）+ ESP32-H2（802.15.4 RCP）；也支持 ESP32-C6 / ESP32-P4 等
> 适用场景：Thread 边界路由器、Matter-over-Thread、Thread 设备上云、Thread 1.4 认证产品
> 来源：乐鑫官方 ESP-Thread-BR 文档、GitHub 仓库、博客、ESP-IDF OpenThread 指南、DevCon23 演讲
> 关联文档：[31-ESP-Matter-Framework](31-ESP-Matter-Framework.md)、[30-ESP-RainMaker](30-ESP-RainMaker-Framework.md)

---

## 目录

1. [Thread 与边界路由器概述](#1-thread-与边界路由器概述)
2. [ESP Thread Border Router 方案](#2-esp-thread-border-router-方案)
3. [架构原理](#3-架构原理)
4. [核心网络功能](#4-核心网络功能)
5. [硬件方案](#5-硬件方案)
6. [ESP32-S3 在边界路由器中的角色](#6-esp32-s3-在边界路由器中的角色)
7. [与 Matter / RainMaker 的关系](#7-与-matter--rainmaker-的关系)
8. [开发与上手](#8-开发与上手)
9. [参考资源](#9-参考资源)

---

## 1. Thread 与边界路由器概述

### 1.1 什么是 Thread

Thread 是基于 **IEEE 802.15.4** 的低功耗、安全的**网状（mesh）网络协议**，广泛应用于智能家居（Matter-over-Thread）。它需要一颗 802.15.4 无线电芯片（如 ESP32-H2、ESP32-C6）。

### 1.2 什么是 Thread 边界路由器（Border Router）

根据 Thread Group 白皮书定义：

> **边界路由器（Border Router）** 是一种能在 Thread mesh 网络与其它 IP 承载接口（如 Wi-Fi、以太网、蜂窝）之间路由数据包的设备。

Thread 网络中的设备要访问互联网或与其它网络通信，**必须经过边界路由器**。它是 Thread 网络"走出去"的门户。

---

## 2. ESP Thread Border Router 方案

ESP Thread Border Router（ESP Thread BR）是乐鑫提供的 **Thread 边界路由器 SDK**，基于乐鑫 Wi-Fi SoC + 802.15.4 SoC 的组合，构建在 **ESP-IDF** 与开源 **OpenThread** 协议栈之上。

### 2.1 认证

- 已获得 Thread Group 颁发的 **Thread 1.4 Certified Component Certificate**（兼容最新 Thread 1.4 标准）
- 同时支持 **Thread 1.3** 互操作性认证
- 支持 Matter 应用场景

### 2.2 区别于 ot-br-posix

```
ot-br-posix（业界常见）：基于 Linux/Unix，需树莓派等主机
ESP Thread BR（乐鑫）  ：基于 ESP-IDF + FreeRTOS，纯嵌入式，低成本
```

ESP Thread BR 是**基于 RTOS 的 Thread 边界路由器方案**，相比 Linux 方案成本极低，且功能完备。

---

## 3. 架构原理

```
┌─────────────────────────────────────────────────────┐
│   Host Wi-Fi SoC（如 ESP32-S3）                      │
│   ┌─────────────────────────────────────────────┐   │
│   │  ESP Thread BR + OpenThread Core Stack      │   │
│   │  + LwIP + mDNS + Service Discovery ...      │   │
│   └──────────────────┬──────────────────────────┘   │
│                      │ Spinel 协议（UART）           │
└──────────────────────┼──────────────────────────────┘
                       ▼
┌─────────────────────────────────────────────────────┐
│   802.15.4 SoC（ESP32-H2）                           │
│   运行 OpenThread RCP（Radio Co-processor）          │
└──────────────────────┬──────────────────────────────┘
                       │ 802.15.4 Thread mesh
                       ▼
              Thread 终端设备（ESP32-H2/C6 等）
```

- **Host Wi-Fi SoC（ESP32-S3）**：运行 Espressif Thread BR 和 OpenThread Core 协议栈
- **802.15.4 SoC（ESP32-H2）**：运行 OpenThread **RCP**（Radio Co-processor，射频协处理器）
- 二者通过 **Spinel 协议**（经 UART）通信

---

## 4. 核心网络功能

ESP Thread BR 提供完整的边界路由功能：

| 功能 | 说明 |
|------|------|
| **双向 IPv6 连接** | 实现 Thread 网络与非 Thread 网络（Wi-Fi/以太网骨干链路）间双向 IPv6 通信 |
| **服务发现** | 双向服务发现，含 SRP 服务器、Advertising Proxy、Discovery Proxy，零配置发现服务 |
| **组播转发** | 实现 MLDv2（Multicast Listener Discovery v2），跨 Thread/Wi-Fi/以太网无缝 IPv6 组播 |
| **NAT64** | 让 Thread 设备经边界路由器访问 IPv4 互联网 |
| **Credential Sharing** | Thread 网络凭证共享 |
| **TREL** | Thread over TCP/Ethernet 链路 |
| **RCP Update** | RCP 固件 OTA 升级（两 SoC 固件合并为单一二进制，一步 OTA） |
| **RF 共存** | 3 线 PTA 射频共存（多无线电设计） |
| **Web GUI** | 用户友好的 Web 配置界面 |

> 提供的 REST API 与 ot-br-posix 兼容，确保无缝集成。

---

## 5. 硬件方案

### 5.1 基于 Wi-Fi 的 Thread 边界路由器（推荐，双 SoC）

默认需要两颗 SoC：

| 角色 | 芯片 | 加载示例 |
|------|------|----------|
| Wi-Fi Host | ESP32 系列（ESP32、ESP32-C、**ESP32-S3** 等） | ot_br 示例 |
| 802.15.4 RCP | ESP32-H2 | ot_rcp 示例 |
| Thread 终端（可选） | ESP32-H2 | ot_cli 示例 |

两 SoC 经 UART 连接（示例：ESP32 DevKitC + ESP32-H2 DevKitC）：

| ESP32 引脚 | ESP32-H2 引脚 |
|------------|---------------|
| GND | G |
| GPIO4 | TX |
| GPIO5 | RX |

### 5.2 基于以太网的 Thread 边界路由器

类似 Wi-Fi 方案，但需具备以太网接口的设备，如 ESP32-Ethernet-Kit 或 ESP32-P4-Function-EV-Board。

### 5.3 单 SoC 方案（不推荐）

ESP32-C6 等单芯片同时支持 Wi-Fi 和 Thread，但**只有一条射频通路**，Wi-Fi 和 Thread 无法同时接收，丢包率高、性能差。故**强烈推荐双 SoC 方案**。

---

## 6. ESP32-S3 在边界路由器中的角色

ESP32-S3 具备 Wi-Fi（无 802.15.4），在 ESP Thread BR 方案中担任**Wi-Fi Host 主机**：

| 职责 | 说明 |
|------|------|
| 运行 Thread BR + OpenThread Core | 边界路由器主逻辑 |
| Wi-Fi 骨干链路 | 连接家庭 Wi-Fi 路由器，提供 IPv6/NAT64 出口 |
| 服务发现 / mDNS | 局域网服务发现 |
| 管理 ESP32-H2 RCP | 经 Spinel/UART 控制 RCP，并支持 RCP OTA |
| 与 RainMaker 集成 | 边界路由器本身可接入 RainMaker 云 |

> ESP32-S3-BOX-3 即可作为 Thread 边界路由器主机（配合 ESP32-H2 RCP），实现 Matter-over-Thread 智能家居中枢。

---

## 7. 与 Matter / RainMaker 的关系

### 7.1 与 ESP-Matter

Thread 边界路由器是 **Matter-over-Thread** 场景的必要基础设施：Matter Thread 终端设备需经边界路由器才能接入 Matter 网络。详见 [31-ESP-Matter-Framework](31-ESP-Matter-Framework.md)。

### 7.2 与 ESP RainMaker

Thread 设备可通过任何支持 NAT64 的边界路由器接入 RainMaker 云：

- 远程控制、OTA、设备管理
- RainMaker 支持 Thread 组网方案（见 [30-ESP-RainMaker](30-ESP-RainMaker-Framework.md) 第 4.2 节）
- **唯一要求：边界路由器必须支持 NAT64**

---

## 8. 开发与上手

### 8.1 ESP-IDF 内置示例

OpenThread 边界路由器示例位于 ESP-IDF：

```
examples/openthread/ot_br    # 边界路由器（Host）
examples/openthread/ot_rcp   # RCP（ESP32-H2）
examples/openthread/ot_cli   # Thread 终端
```

支持目标：ESP32 / ESP32-C2 / ESP32-C3 / ESP32-C5 / ESP32-C6 / ESP32-P4 / **ESP32-S2 / ESP32-S3** / ESP32-S31。

### 8.2 配置

```bash
idf.py menuconfig

# OpenThread CLI 默认经 UART 输出，也支持 USB JTAG：
# Component config → ESP System Settings → Channel for console output → USB Serial/JTAG Controller

# 单 SoC（如 C6）需启用：
# CONFIG_ESP_COEX_SW_COEXIST_ENABLE=y
# CONFIG_OPENTHREAD_RADIO_NATIVE=y

# 自动启动模式：配置 Wi-Fi SSID/密码后开机自动连 Wi-Fi + 组 Thread 网
# OPENTHREAD_NETWORK_AUTO_START=y
# CONFIG_EXAMPLE_WIFI_SSID / CONFIG_EXAMPLE_WIFI_PASSWORD

# 手动模式：启用 wifi 命令手动连网
# OPENTHREAD_NETWORK_AUTO_START=n + OPENTHREAD_CLI_ESP_EXTENSION=y
```

### 8.3 关键 API

```c
// 初始化边界路由器（启用所有边界路由功能）
esp_openthread_border_router_init();
```

### 8.4 ESP-Thread-BR 生产级 SDK

ESP Thread BR SDK（esp-thread-br 仓库）在 ESP-IDF 示例之上提供额外生产组件与示例：

- RCP 固件合并 OTA
- Web GUI
- REST API（ot-br-posix 兼容）
- RF 共存设计

---

## 9. 参考资源

| 资源 | 链接 |
|------|------|
| ESP Thread Border Router 文档 | https://docs.espressif.com/projects/esp-thread-br/en/latest/index.html |
| ESP Thread BR GitHub | https://github.com/espressif/esp-thread-br |
| ESP-IDF OpenThread 指南（中文） | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/api-guides/openthread.html |
| ot_br 示例 | https://github.com/espressif/esp-idf/tree/master/examples/openthread/ot_br |
| ot_rcp 示例 | https://github.com/espressif/esp-idf/tree/master/examples/openthread/ot_rcp |
| ot_cli 示例 | https://github.com/espressif/esp-idf/tree/master/examples/openthread/ot_cli |
| Thread Border Router 博客 | https://developer.espressif.com/blog/espressif-thread-border-router/ |
| Thread 1.4 认证证书 | https://documentation.espressif.com/Espressif%20Thread%20Border%20Router%20Thread%20V1.4%20Interoperability%20Certification.pdf |
| Thread 1.3 认证证书 | https://www.espressif.com/sites/default/files/Espressif%20Thread%20Border%20Router%20Thread%20V1.3%20Interoperability%20Certification_0.pdf |
| OpenThread 官方 | https://github.com/openthread/openthread |
| ESP-FAQ Thread 章节 | https://docs.espressif.com/projects/esp-faq/zh_CN/latest/software-framework/thread.html |
| DevCon23 Thread 介绍（视频） | https://www.youtube.com/watch?v=OuzF993uV4w |

---

> **文档总结**：ESP Thread Border Router 是乐鑫基于 ESP-IDF + 开源 OpenThread 的纯嵌入式 Thread 边界路由器 SDK（区别于 Linux 版 ot-br-posix，成本极低）。架构为"Wi-Fi Host SoC（ESP32-S3）运行 Thread BR + OpenThread Core，经 Spinel/UART 控制 ESP32-H2 RCP（射频协处理器）"。提供双向 IPv6 连接、服务发现、组播转发、NAT64、RCP 一步 OTA、RF 共存、Web GUI 等完整功能，已通过 Thread 1.4 / 1.3 认证。ESP32-S3 在其中担任 Wi-Fi 骨干主机（如 ESP32-S3-BOX-3），是 Matter-over-Thread 智能家居与 Thread 设备上云（经 RainMaker NAT64）的核心基础设施。
