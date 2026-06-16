# ESP-Matter 框架详解 — Matter 智能家居方案 SDK

> 文档编号：31
> 适用芯片：ESP32-S3（Matter-over-Wi-Fi）/ 配合 ESP32-H2、ESP32-C5/C6（Thread）等
> 适用场景：跨生态智能家居、Matter 设备/网关/Thread 边界路由器、RainMaker 云控集成
> 来源：乐鑫官方 ESP-Matter 文档、GitHub 仓库、ESP32-S3-BOX-3 新闻、ESP-IDF OpenThread 指南

---

## 目录

1. [Matter 与 ESP-Matter 概述](#1-matter-与-esp-matter-概述)
2. [乐鑫 Matter 解决方案组成](#2-乐鑫-matter-解决方案组成)
3. [Matter 平台类型](#3-matter-平台类型)
4. [ESP-Matter SDK 架构](#4-esp-matter-sdk-架构)
5. [ESP32-S3 在 Matter 中的角色](#5-esp32-s3-在-matter-中的角色)
6. [Matter 与 RainMaker 集成](#6-matter-与-rainmaker-集成)
7. [开发与上手](#7-开发与上手)
8. [应用场景](#8-应用场景)
9. [参考资源](#9-参考资源)

---

## 1. Matter 与 ESP-Matter 概述

### 1.1 什么是 Matter

Matter（原 Project CHIP）是由 **CSA（连接标准联盟）** 主导的统一智能家居标准，旨在实现不同厂商、不同生态（Apple HomeKit、Google Home、Amazon Alexa、三星 SmartThings 等）智能设备的**安全、可靠互操作**。它统一了 Wi-Fi 与 Thread 两种底层网络，让设备一旦认证即可跨生态使用。

### 1.2 什么是 ESP-Matter

ESP-Matter 是乐鑫基于开源 Matter SDK（[project-chip/connectedhomeip](https://github.com/project-chip/connectedhomeip)）构建的**生产级 Matter SDK**，提供简化的 API、常用外设、安全/制造/生产工具及配套文档，目标是让用户以最短时间实现 Matter 产品量产。

### 1.3 核心价值

- **跨生态**：一次接入，Apple/Google/Amazon/三星等生态通用
- **生产就绪**：丰富的量产参考、证书生成与预置服务
- **全平台**：覆盖 Matter 设备、网关、Thread 边界路由器
- **云集成**：与 ESP RainMaker / ESP Insights 无缝结合

---

## 2. 乐鑫 Matter 解决方案组成

乐鑫的 Matter 解决方案是一套完整体系：

```
┌─────────────────────────────────────────────────────────┐
│              乐鑫 Matter 解决方案 (ESP-Matter)            │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ① 全谱系 Matter 设备平台                                │
│     Matter Wi-Fi 设备 / Matter Thread 设备              │
│     Thread Border Router / Bridge 网关                   │
│                                                         │
│  ② 生产就绪 SDK (ESP-Matter SDK)                         │
│     基于开源 Matter SDK + 简化 API + 工具 + 文档          │
│                                                         │
│  ③ Matter 与 ESP RainMaker 集成                          │
│     私有云设备管理 + 远程控制                             │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 3. Matter 平台类型

### 3.1 Matter Wi-Fi 设备

- **适用芯片**：ESP32、ESP32-C 系列、**ESP32-S 系列（含 ESP32-S3）**
- 基于内置 Wi-Fi 直接接入 Matter 生态

### 3.2 Matter Thread 设备

- **适用芯片**：ESP32-H 系列、ESP32-C5、ESP32-C6（带 802.15.4）
- 基于 Thread（802.15.4）低功耗接入

### 3.3 Thread Border Router（线程边界路由器）

- 由 ESP32-H（802.15.4）+ Wi-Fi SoC 高效组合而成，连接 Thread 网络与 Wi-Fi 网络
- 详见 [38-ESP-Thread-Border-Router](38-ESP-Thread-Border-Router.md)

### 3.4 Matter-Zigbee / Matter-BLE Mesh Bridge（桥接网关）

- **Matter-Zigbee Bridge**：ESP32-H + 另一颗 Wi-Fi SoC
- **Matter-BLE Mesh Bridge**：单颗同时具备 Wi-Fi + BLE 的 SoC 即可
- 让基于 Zigbee / BLE Mesh 的非 Matter 设备接入 Matter 生态

---

## 4. ESP-Matter SDK 架构

```
┌─────────────────────────────────────────────┐
│         应用层 (User Application)            │
├─────────────────────────────────────────────┤
│   ESP-Matter 简化 API + 常用外设 + 工具       │
│   (数据模型、配网、OTA、制造与生产工具)        │
├─────────────────────────────────────────────┤
│   开源 Matter SDK (connectedhomeip)          │
├──────────────┬──────────────────────────────┤
│  ESP RainMaker│  ESP Insights (远程诊断)     │
├──────────────┴──────────────────────────────┤
│   ESP-IDF / FreeRTOS                         │
├─────────────────────────────────────────────┤
│   ESP32-S3 (Wi-Fi) / ESP32-H2 (Thread) 等    │
└─────────────────────────────────────────────┘
```

SDK 集成 ESP RainMaker（云服务）与 ESP Insights（远程诊断），并提供丰富的**量产参考**（manufacturing references），简化 Matter 产品开发与上市流程。

---

## 5. ESP32-S3 在 Matter 中的角色

ESP32-S3 拥有 Wi-Fi（无 802.15.4），在 Matter 体系中主要承担：

| 角色 | 说明 |
|------|------|
| **Matter-over-Wi-Fi 设备** | 作为 Wi-Fi 终端设备接入 Matter 生态 |
| **Matter 网关 / 控制器** | 如 ESP32-S3-BOX-3 可作为 Matter 网关、Thread 边界路由器主机、Home Assistant 本地控制器 |
| **Thread 边界路由器主机** | ESP32-S3（Wi-Fi 主机）+ ESP32-H2（RCP）组合实现 Thread 边界路由 |
| **Matter-BLE Mesh Bridge** | 单芯片即可（同时有 Wi-Fi + BLE） |

> ESP32-S3-BOX-3 是官方主推的 Matter 多面手：可作为 Matter-over-Wi-Fi/Thread 终端、Thread 边界路由器、Matter 网关，并支持 Home Assistant 开源平台。

---

## 6. Matter 与 RainMaker 集成

乐鑫 AIoT 云平台 ESP RainMaker 可为 Matter 设备提供**远程控制与云端设备管理**，把 Matter 的本地控制能力扩展到云端：

| 方案 | 说明 |
|------|------|
| **Fabric 方案** | 设备走标准 Matter 配网，RainMaker App 作 Matter Controller，绑定到 RainMaker 云；远程控制/OTA 走 RainMaker，本地控制走 Matter |
| **Non-Fabric 方案** | 设备用 BLE 广播，同时支持标准 Matter 与 RainMaker 配网；开发简单、App 兼容好 |

详见 [30-ESP-RainMaker](30-ESP-RainMaker-Framework.md) 第 5 节。

---

## 7. 开发与上手

### 7.1 在线免设置体验

无需任何额外配置即可在乐鑫设备上测试 Matter：
[ESP Launchpad — ESP-Matter](https://espressif.github.io/esp-launchpad/?flashConfigURL=https://espressif.github.io/esp-matter/launchpad.toml)

### 7.2 Data Model Validator

乐鑫提供 Web UI 工具验证 Matter 设备的数据模型：
https://espressif.github.io/esp-matter-tools/

### 7.3 SDK 获取与开发

```bash
# ESP-Matter SDK 仓库（包含工具、示例、文档）
# 详见官方文档的 Get Started 章节
```

ESP32-S3 目标示例编译：

```bash
idf.py set-target esp32s3
idf.py build
idf.py -p <PORT> flash monitor
```

### 7.4 证书与制造

乐鑫提供证书生成与预置（pre-provisioning）服务，简化 Matter 兼容设备的制造流程。Matter 1.0 认证体系已就绪。

---

## 8. 应用场景

| 场景 | 说明 |
|------|------|
| 跨生态智能家居终端 | Matter-over-Wi-Fi 灯/开关/传感器（ESP32-S3） |
| Thread 终端 + 边界路由器 | 低功耗 Thread 设备 + ESP32-S3 主机的边界路由 |
| 智能家居中枢 / 网关 | ESP32-S3-BOX-3 作 Matter 网关、Home Assistant 控制器 |
| 桥接网关 | 让 Zigbee / BLE Mesh 老设备接入 Matter |
| Matter + 私有云 | RainMaker Fabric 方案，Matter 本地 + 云端远程 |
| 语音 + Matter | BOX-3 离线语音控制 Matter 设备 |

---

## 9. 参考资源

| 资源 | 链接 |
|------|------|
| ESP-Matter 文档（中文） | https://docs.espressif.com/projects/esp-matter/zh_CN/latest/esp32s3/introduction.html |
| ESP-Matter 文档（英文） | https://docs.espressif.com/projects/esp-matter/en/latest/esp32s3/introduction.html |
| ESP-Matter GitHub | https://github.com/espressif/esp-matter |
| Matter Launchpad 体验 | https://espressif.github.io/esp-launchpad/?flashConfigURL=https://espressif.github.io/esp-matter/launchpad.toml |
| Data Model Validator | https://espressif.github.io/esp-matter-tools/ |
| 开源 Matter SDK | https://github.com/project-chip/connectedhomeip |
| 乐鑫 Matter 方案官网 | https://www.espressif.com/en/solutions/device-connectivity/esp-matter-solution |
| ESP RainMaker | https://rainmaker.espressif.com |
| ESP Thread Border Router | https://github.com/espressif/esp-thread-br |
| ESP32-S3-BOX-3（Matter 网关） | https://github.com/espressif/esp-box |

---

> **文档总结**：ESP-Matter 是乐鑫基于开源 Matter SDK 打造的生产级智能家居 SDK，构成"设备平台 + SDK + RainMaker 云集成"完整方案。ESP32-S3 凭借 Wi-Fi 在其中担任 Matter-over-Wi-Fi 终端、Matter 网关/控制器、Thread 边界路由器主机、BLE Mesh Bridge 等多种角色。结合 RainMaker 的 Fabric/Non-Fabric 方案，可把 Matter 本地控制扩展到私有云远程控制。ESP32-S3-BOX-3 是官方主推的 Matter 多面手开发板，支持 Home Assistant 开源平台，是跨生态智能家居开发的理想起点。
