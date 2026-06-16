# ESP RainMaker 框架详解 — 端到端 AIoT 云平台

> 文档编号：30
> 适用芯片：ESP32 / ESP32-S2 / ESP32-S3 / ESP32-C2 / ESP32-C3 / ESP32-C5 / ESP32-C6 / ESP32-H2 等全系列
> 适用场景：AIoT 设备配网、远程控制、OTA、手机 App、第三方语音助手集成
> 来源：乐鑫官方 RainMaker 文档、GitHub 仓库、ESP-BOX 技术架构文档、ESP-IDF 库与框架页

---

## 目录

1. [ESP RainMaker 概述](#1-esp-rainmaker-概述)
2. [核心组件与架构](#2-核心组件与架构)
3. [核心功能](#3-核心功能)
4. [联网方案](#4-联网方案)
5. [RainMaker + ESP-Matter 融合](#5-rainmaker--esp-matter-融合)
6. [手机 App 与 Claiming](#6-手机-app-与-claiming)
7. [云端集成（ESP Insights）](#7-云端集成esp-insights)
8. [如何开始开发](#8-如何开始开发)
9. [应用场景](#9-应用场景)
10. [参考资源](#10-参考资源)

---

## 1. ESP RainMaker 概述

### 1.1 什么是 ESP RainMaker

ESP RainMaker 是乐鑫提供的**端到端（end-to-end）AIoT 解决方案**，为基于 ESP32 系列 SoC 的产品提供**无需云端配置即可实现的远程控制与监控**能力。它覆盖从芯片、设备固件、安全云中间件，到手机 App、第三方语音助手集成的完整链路。

### 1.2 核心理念

| 特性 | 说明 |
|------|------|
| **零云端配置** | 开发者在固件中自定义设备和参数，云端自动适配，无需任何云端配置 |
| **动态 UI** | 手机 App 根据设备信息动态渲染控制界面 |
| **私有云部署** | 基于 AWS Serverless 架构，"按用量付费"（pay-as-you-grow） |
| **一站式 AIoT** | 设备配网、控制、管理、诊断、OTA 全部覆盖 |

### 1.3 适用芯片

ESP32、ESP32-S2、ESP32-S3、ESP32-C2、ESP32-C3、ESP32-C6、ESP32-H2、ESP32-C5（preview）等全系列。

---

## 2. 核心组件与架构

ESP RainMaker 由四大核心组件构成：

```
┌───────────────────────────────────────────────────────────┐
│                    用户 / 第三方语音助手                     │
│         (Alexa / Google Home / 手机 App / CLI)             │
└────────────────────────▲──────────────────────────────────┘
                         │ 远程连接
┌────────────────────────┴──────────────────────────────────┐
│                ③ RainMaker Cloud（云端后端）                │
│           AWS Serverless 架构，安全云中间件                  │
└────────────────────────▲──────────────────────────────────┘
                         │ 加密通道
┌────────────────────────┴──────────────────────────────────┐
│  ② RainMaker Agent（设备端固件，本仓库）                    │
│  ① Claiming Service（获取云端连接凭证）                     │
│                ESP32-S3 等设备硬件                         │
└───────────────────────────────────────────────────────────┘
```

| 组件 | 说明 |
|------|------|
| **Claiming Service** | 设备首次联网时获取云端连接凭证（证书） |
| **RainMaker Agent** | 设备端固件库（esp-rainmaker 仓库），开发设备固件 |
| **RainMaker Cloud** | 后端云，提供安全远程连接，开发者无需管理证书与云基础设施 |
| **RainMaker Phone App / CLI** | 客户端工具，设备发现、配网、关联、控制 |

---

## 3. 核心功能

| 功能 | 说明 |
|------|------|
| **自定义设备与参数** | 在固件中定义任意类型设备与参数 |
| **零云端配置** | 云端自动识别设备模型，无需手工配置 |
| **动态手机 App** | App 根据设备信息动态渲染 UI |
| **OTA 升级** | 远程固件升级 |
| **场景自动化 / 时间调度** | 触发条件与定时任务 |
| **本地控制** | 本地网络内直连控制（不依赖云） |
| **第三方语音助手** | Alexa / Google Home 集成 |
| **用户节点共享** | 多用户共享设备控制权 |
| **ESP Insights 集成** | 远程诊断与可观测性 |

---

## 4. 联网方案

ESP RainMaker 支持多种网络拓扑，适配不同场景：

### 4.1 Wi-Fi 直连（Mesh-Lite）

- **适用芯片**：ESP32-C3、ESP32、ESP32-S3 等
- **特点**：Wi-Fi 协议联网上云，支持分区控制、组控制，无需网关
- **仓库**：esp-mesh-lite/examples/rainmaker/led_light

### 4.2 Thread 联网

- **硬件**：ESP32-H2 + Thread 边界路由器（ESP32-S3 SoC + ESP32-H2 RCP，或第三方边界路由器）
- **特点**：通过 802.15.4 Thread 协议经边界路由器上云，支持大规模组网、低功耗、快速组网
- **支持**：组控制、分区控制（本地组控用组播，远程组控用点对点）、Thread 网络内 OTA

> Thread 方案要求边界路由器支持 NAT64，详见 [38-ESP-Thread-Border-Router](38-ESP-Thread-Border-Router.md)。

---

## 5. RainMaker + ESP-Matter 融合

针对智能家居生态，RainMaker 提供两种与 ESP-Matter 结合的方案：

### 5.1 Fabric 方案（接入私有云）

- **场景**：需要接入私有云
- **机制**：设备使用标准 Matter 配网协议，RainMaker App 支持 Matter Controller 功能，配网并绑定到 RainMaker 云
- **控制**：远程控制和 OTA 通过 RainMaker 通道，本地控制用 Matter
- **硬件**：ESP32、ESP32-S3、ESP32-C 系列
- **仓库**：esp-rainmaker/examples/matter

### 5.2 Non-Fabric 方案（快速接入 Matter 生态）

- **场景**：需要快速接入 Matter 生态
- **机制**：设备使用蓝牙广播，同时支持标准 Matter 和 RainMaker 配网服务
- **特点**：开发简单、App 兼容性好，支持动态 Matter 二维码配网、Matter 生态音箱配网
- **硬件**：ESP32-C3、ESP32-S3
- **仓库**：esp-rainmaker/examples/matter/matter_light

---

## 6. 手机 App 与 Claiming

### 6.1 官方 App：ESP RainMaker Home

| 平台 | 链接 |
|------|------|
| Google Play（Android） | https://play.google.com/store/apps/details?id=com.espressif.rainmaker |
| App Store（iOS） | https://apps.apple.com/app/esp-rainmaker/id1497491540 |
| Android 直接 APK | 见 GitHub Wiki |
| App 源码（Android） | https://github.com/espressif/esp-rainmaker-android |
| App 源码（iOS） | https://github.com/espressif/esp-rainmaker-ios |

### 6.2 Claiming（设备凭证获取）

设备出厂或首次使用需通过 Claiming Service 获取云端证书，建立设备与用户账户的绑定关系。Claiming 可在产线批量完成，也可通过 App/CLI 完成。

---

## 7. 云端集成（ESP Insights）

ESP RainMaker 与 [32-ESP-Insights](32-ESP-Insights-Framework.md) 深度集成：ESP Insights 作为 RainMaker 生态的一部分，提供可选的远程诊断服务，增强固件开发质量、产品维护与数据分析能力。所有 RainMaker 示例默认已集成 ESP Insights 支持。

> ESP-VoCat、ESP-DualKey 等新一代开发板的出厂固件均基于 RainMaker 实现配网与远程控制。

---

## 8. 如何开始开发

### 8.1 环境准备

```bash
# 克隆 RainMaker 仓库（需 --recursive 拉取依赖）
git clone --recursive https://github.com/espressif/esp-rainmaker.git

# 兼容说明：master 分支不再支持 esp-idf v4.x（已 EOL）
# 需要 v4.x 兼容请切换 idf_4_x_compat 分支
```

### 8.2 支持 ESP-IDF 版本

ESP RainMaker 兼容 ESP-IDF 4.1 及以上版本。详细入门见官方文档 https://rainmaker.espressif.com/docs/get-started.html。

### 8.3 典型开发流程

```
1. 在固件中用 RainMaker API 定义设备节点（node）和参数（param）
2. 调用 esp_rmaker_start() 启动 Agent
3. 设备首次上电 → Claiming 获取凭证 → 连接云
4. 用户用 App 扫码配网 → 设备关联到用户账户
5. App 动态渲染控制界面 → 远程控制 / OTA / 诊断
```

### 8.4 在线体验

通过 ESP Launchpad 一键体验 RainMaker 示例：
https://espressif.github.io/esp-launchpad/?solution=rainmaker

---

## 9. 应用场景

| 场景 | 说明 |
|------|------|
| 智能家居 | 灯光、开关、传感器节点，App + 语音控制（ESP32-S3-BOX-3 / ESP-DualKey 均基于此） |
| 智能照明组网 | Mesh-Lite / Thread 大规模节点上云 |
| 工业设备远程监控 | 传感器数据上云 + OTA 维护 |
| 农业/环境监测 | 低功耗节点 + 边界路由器上云 |
| AI 语音设备 | ESP-VoCat 等通过 RainMaker 配网并管理 AI Agent |
| Matter 设备云控 | RainMaker + ESP-Matter Fabric/Non-Fabric 方案 |

---

## 10. 参考资源

| 资源 | 链接 |
|------|------|
| ESP RainMaker 官网 | https://rainmaker.espressif.com |
| RainMaker 文档 | https://rainmaker.espressif.com/docs/get-started.html |
| ESP RainMaker GitHub | https://github.com/espressif/esp-rainmaker |
| RainMaker 控制台（Console） | https://rainmaker.espressif.com |
| ESP RainMaker Home（Android） | https://play.google.com/store/apps/details?id=com.espressif.rainmaker |
| ESP RainMaker（iOS） | https://apps.apple.com/app/esp-rainmaker/id1497491540 |
| ESP32 论坛 RainMaker 板块 | https://www.esp32.com/viewforum.php?f=41 |
| ESP-Mesh-Lite 仓库 | https://github.com/espressif/esp-mesh-lite |
| ESP Thread Border Router | https://github.com/espressif/esp-thread-br |
| RainMaker 技术百科（中文） | https://docs.espressif.com/projects/esp-techpedia/zh_CN/latest/esp-friends/solution-introduction/esp-rainmaker/esp-rainmaker-wiki.html |
| Launchpad 体验 | https://espressif.github.io/esp-launchpad/?solution=rainmaker |

---

> **文档总结**：ESP RainMaker 是乐鑫端到端 AIoT 云平台，覆盖设备固件（RainMaker Agent）、云端后端（AWS Serverless）、手机 App、Claiming 凭证服务全链路。其核心卖点是"零云端配置"——在固件中定义设备参数，云端与 App 自动适配。支持 Wi-Fi/Mesh-Lite/Thread 多种联网，以及与 ESP-Matter 的 Fabric/Non-Fabric 融合方案，可对接 Alexa/Google Home 语音助手。ESP32-S3-BOX-3、ESP-DualKey、ESP-VoCat 等官方开发板均以 RainMaker 作为配网与远程控制底座，并与 ESP Insights 深度集成实现远程诊断。
