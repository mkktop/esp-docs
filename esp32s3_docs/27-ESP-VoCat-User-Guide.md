# ESP-VoCat（喵伴）用户指南 — 大模型 AI 语音交互开发套件

> **文档来源**：基于乐鑫官方 esp-dev-kits 文档、ESP-VoCat v1.0/v1.2 用户指南、ESP-Brookesia 复刻教程及 GitHub 仓库整理。
> **文档编号**：27

---

## 目录

- [1. 产品概述](#1-产品概述)
- [2. 硬件架构与核心规格](#2-硬件架构与核心规格)
- [3. 板载组件详解](#3-板载组件详解)
- [4. 显示屏与触摸](#4-显示屏与触摸)
- [5. 音频系统](#5-音频系统)
- [6. 电源系统](#6-电源系统)
- [7. 接口与扩展](#7-接口与扩展)
- [8. 硬件版本差异（v1.0 / v1.2）](#8-硬件版本差异v10--v12)
- [9. 软件框架与开发](#9-软件框架与开发)
- [10. 快速入门](#10-快速入门)
- [11. 应用场景](#11-应用场景)
- [12. 相关资源与链接](#12-相关资源与链接)

---

## 1. 产品概述

ESP-VoCat（喵伴）是乐鑫携手火山引擎扣子大模型团队打造的智能 AI 开发套件，定位为**桌面语音交互终端 / 陪伴式 AI 设备**。它适用于玩具、智能音箱、智能中控等需要大模型赋能的语音交互类产品。

设备搭载 ESP32-S3 模组、1.85 英寸 QSPI 圆形触摸屏、双麦克风阵列，支持**离线语音唤醒与声源定位算法**。结合火山引擎扣子、Amazon Nova、OpenAI、小智 AI、Gemini 等大模型能力，喵伴可实现**全双工语音交互、多模态识别与智能体控制**，为开发者打造完整的端侧 AI 应用体验提供坚实基础。

### 核心亮点

- **大模型 AI 交互**：对接扣子 / OpenAI / Gemini / 小智等大模型，支持全双工对话、情绪识别、长记忆能力。
- **离线语音唤醒 + 声源定位**：双麦阵列 + ESP-SR 算法，配合旋转底座可实现 180° 方向跟踪与视觉对视。
- **圆形彩屏 + 原生触摸**：1.85" 360×360 QSPI 屏，ESP32-S3 原生电容触摸。
- **丰富的智能体能力**：MCP 协议 + Function Call，可作本地控制中枢对接智能设备；自定义语音角色、动作姿态感知（IMU）。

---

## 2. 硬件架构与核心规格

### 2.1 核心规格总览

| 参数 | 规格 |
|------|------|
| 主控 SoC | ESP32-S3（240 MHz Xtensa 32-bit LX7 双核） |
| 主控模组 | ESP32-S3-WROOM-1-N16R16VA（v1.2）/ WROOM-2-N32R16V（v1.0） |
| 无线连接 | 2.4 GHz Wi-Fi (802.11 b/g/n) + Bluetooth 5 (LE) |
| Flash | 16 MB（v1.2）/ 32 MB（v1.0） |
| PSRAM | 16 MB Octal PSRAM |
| 内置 SRAM | 512 KB |
| AI 加速 | 专用向量指令，神经网络/信号处理加速 |

### 2.2 存储与扩展

- **microSD 卡槽**：支持高达 32 GB，SDIO 协议（1 线 SD 总线），可用于存放大模型配置（如 Coze 配置）、音频资源等。

### 2.3 功能框图

ESP-VoCat 由三块 PCB 组成（CoreBoard 核心板 / MicBoard 麦克风板 / BaseBoard 底板），主要组件连接关系如下：

```
                 ┌──────────────────────────┐
                 │    ESP32-S3-WROOM-1       │
                 │  (双核 LX7 240MHz)        │
                 │  16MB Flash + 16MB PSRAM  │
                 └─────────────┬────────────┘
                               │
        ┌──────────┬───────────┼───────────┬──────────┐
        ▼          ▼           ▼           ▼          ▼
   ┌─────────┐ ┌────────┐ ┌─────────┐ ┌────────┐ ┌─────────┐
   │ 1.85"   │ │双麦阵列│ │ 3W 扬声器│ │ USB-C  │ │ microSD │
   │ 圆形LCD │ │(I2S)   │ │ + 功放   │ │ 供电/  │ │  卡槽   │
   │ 360×360 │ │ ESP-SR │ │         │ │ 下载   │ │         │
   │ QSPI    │ │ 声源   │ │         │ │        │ │         │
   │ ST77916 │ │ 定位   │ │         │ │        │ │         │
   └─────────┘ └────────┘ └─────────┘ └────────┘ └─────────┘
        │          │
   原生电容触摸  底板电源管理：BQ27220 电池计量 / TP4057 充电 /
                 TLV62569 DCDC(5V→3.3V) / SAM8108 开关机控制
```

---

## 3. 板载组件详解

ESP-VoCat 采用三板结构，以下按板卡分组介绍主要组件：

### 3.1 CoreBoard（核心板）

| 组件 | 描述 |
|------|------|
| ESP32-S3-WROOM-1 模组 | 主控，集成 Flash 和 PSRAM，支持 Wi-Fi + BLE |
| Battery Connector（电池连接器） | 连接 3.7 V 锂电池，上正下负 |
| LCD FPC Connector（屏幕连接器） | 连接 1.85 英寸圆形 LCD（360×360） |
| MicBoard Connector（麦克风连接器） | 连接双麦克风阵列与状态 LED |
| Touch Connector（触摸连接器） | 连接触摸铜箔，实现触摸交互 |
| Speaker Connector（扬声器连接器） | 2 线连接器，连接内置 3 W 扬声器 |

### 3.2 MicBoard（麦克风板）

| 组件 | 描述 |
|------|------|
| MIC（麦克风阵列） | 双 LMA3729T381-OY3S 麦克风，支持本地语音唤醒与声源定位 |
| Green LED | 状态指示灯（如触摸固件升级成功时常亮） |

### 3.3 BaseBoard（底板）

| 组件 | 描述 |
|------|------|
| BQ27220（电池管理芯片） | 电池电量检测、充电管理、电源状态监控 |
| TP4057（锂电池充电芯片） | 锂电池充电，充电电流 250 mA |
| TLV62569（DCDC 芯片） | 降压转换，5 V → 3.3 V，为系统稳定供电 |
| SAM8108（开关机控制芯片） | 单击 POWER 按键即可切换开关机状态 |
| Type-C（USB-C 接口） | 供电、编程下载、调试，支持对锂电池充电 |

---

## 4. 显示屏与触摸

### 4.1 LCD 规格

| 参数 | 规格 |
|------|------|
| 屏幕尺寸 | 1.85 英寸 |
| 分辨率 | 360 × 360（圆形） |
| 接口 | QSPI |
| LCD 驱动 IC | ST77916 |
| 背光控制 | `LCD_BLK` (GPIO44) 可软件调节背光 |

> 屏幕型号 UE018HV-RB39-A002A，详细参数见显示屏规格书。

### 4.2 触摸

ESP-VoCat 使用 **ESP32-S3 原生电容触摸传感器**（非外接触摸 IC），提供直观丰富的交互体验。

> **注意**：部分批次屏幕存在触摸位置偏移问题，可烧录"触摸升级固件"更新屏幕触摸芯片固件。程序运行后顶部绿灯常亮表示更新成功，快速闪烁表示失败。

---

## 5. 音频系统

### 5.1 输入：双麦克风阵列

- **型号**：双 LMA3729T381-OY3S
- **功能**：本地语音唤醒（WakeNet）、声源定位（DOA）、波束成形
- **方向跟踪**：配合电机控制模块（旋转底座），可实现 180° 范围内精准方向跟踪，自动识别声源方位并视觉对视

### 5.2 输出：3W 扬声器

内置 3 W 扬声器，用于语音播报与音频反馈，配合音视频框架 `esp-gmf` 实现流畅的音频播放。

---

## 6. 电源系统

ESP-VoCat 电源系统兼容多种供电方式，且任意外部供电均可为锂电池充电：

| 供电方式 | 说明 |
|----------|------|
| Type-C（USB-C） | 主供电方式，5V DC，同时支持下载调试 |
| Magnetic Connector（磁吸连接器） | 连接磁吸充电座供电 |
| 锂电池 | 内置 3.7 V 锂电池，按下 POWER 按键开启供电 |

- **电池容量**：700 mAh（3.7 V 锂电池）
- **充电管理**：TP4057，充电电流 250 mA
- **电量监测**：BQ27220 燃料计芯片实时监控电量、充电状态
- **开关机**：SAM8108 芯片，单击 POWER 按键切换开关机

---

## 7. 接口与扩展

| 接口 | 说明 |
|------|------|
| USB-C | 供电、编程下载、调试、电池充电 |
| 磁吸连接器 | 磁吸供电 / 充电 |
| Pogopin 接口 | 预留，方便功能扩展 |
| microSD 卡槽 | 最高 32 GB，SDIO 协议 |
| IMU 传感器 | 感知动作与姿态（身体语言交互） |
| 旋转底座接口 | 配合旋转底座固件实现声源跟踪转动 |

---

## 8. 硬件版本差异（v1.0 / v1.2）

ESP-VoCat 有两个硬件版本，可通过主板丝印版本号确认：

| 项目 | v1.0 | v1.2 |
|------|------|------|
| 主控模组 | ESP32-S3-WROOM-2-N32R16V | ESP32-S3-WROOM-1-N16R16VA |
| Flash | 32 MB | 16 MB |
| PSRAM | 16 MB | 16 MB |
| 用户指南 | user_guide_v1.0.html | user_guide_v1.2.html |

> v1.2 版本调整了分区结构，优化了分区逻辑与资源分配，并新增了从 SD 卡读取 Coze 配置的功能。

---

## 9. 软件框架与开发

### 9.1 软件栈

ESP-VoCat 的出厂固件基于以下框架构建：

| 框架 | 作用 |
|------|------|
| **ESP-Brookesia** | 面向 IoT 设备的人机交互开发框架，承担 UI 构建与渲染，深度融合扣子平台与 `esp-gmf` 音视频框架 |
| **ESP-SR** | 离线语音唤醒、声源定位 |
| **ESP-GMF** | 乐鑫全新音视频框架，负责音频播放 |
| **ESP RainMaker** | 设备配网、远程控制、Agent 管理 |
| **ESP-IDF** | 底层开发框架（基于 release/v5.5） |

### 9.2 智能体能力

依托 ESP-Brookesia，喵伴集成了多项端侧优化的智能功能：

| 能力 | 说明 |
|------|------|
| 智能对讲与情绪识别 | 识别用户意图与情绪变化，拟人化动态表情与语音反馈 |
| 长记忆能力 | 持续记录多轮对话，记住姓名、偏好等，提供个性化体验 |
| 离线语音唤醒与声源定位 | 180° 方向跟踪，配合旋转底座视觉对视 |
| 自定义语音角色 | 基于扣子平台灵活切换音色与风格，DIY 专属 AI 形象 |
| MCP 协议 + Function Call | 对接本地智能设备，远程控制/任务下发/状态反馈，作为本地控制中枢 |
| 动作与姿态感知 | 内置 IMU，用身体语言与用户互动 |

### 9.3 固件形态

ESP-VoCat 提供多种出厂/示例固件：

| 固件 | 基础 | 说明 |
|------|------|------|
| Coze 固件 | 火山引擎扣子智能体 | 需自行注册 KEY，基于 ESP-Brookesia |
| 小智固件 | 小智 AI 机器人 | 支持中/英/日文字库，更多表情 |
| 屏幕触摸固件 | - | 修复触摸偏移问题 |
| 旋转底座固件 | - | 旋转底座电机控制 |

---

## 10. 快速入门

### 10.1 首次配网（出厂固件）

设备首次使用必须通过 **ESP RainMaker Home** App 进行配网：

1. 设备上电后，屏幕显示配网二维码。
2. 打开 ESP RainMaker Home，点击右上角 **"+"** → **Scan QR Code** 扫描设备二维码。
3. 选择 Wi-Fi 网络并输入密码，完成配网。
4. 配网完成后，说 **"Hi, ESP"** 唤醒设备进行语音对话，也可点击屏幕唤醒；支持触控打断、约 15 秒无操作自动休眠。

### 10.2 恢复出厂 / 切换 Agent

- **App 内恢复出厂**：配网完成后进入对话界面，长按触摸进入 App 界面，在 Settings 中执行恢复出厂。
- **切换 Agent**：在 [ESP Private Agents](https://agents.espressif.com) 创建并分享 Agent，手机扫描分享二维码，然后在 ESP RainMaker Home 中选择本设备更新 Agent。

### 10.3 烧录固件开发

```bash
# 克隆 ESP-VoCat 出厂固件仓库（基于 esp-dev-kits，依赖 ESP-IDF release/v5.5）
git clone --recursive https://github.com/espressif/esp-dev-kits.git
cd esp-dev-kits/examples/esp-vocat

idf.py set-target esp32s3
idf.py build
idf.py -p <PORT> flash monitor
```

或在浏览器通过 [ESP-Launchpad](https://espressif.github.io/esp-launchpad/?flashConfigURL=https://dl.espressif.com/AE/esp-dev-kits/launchpad.toml) 一键体验各示例。

---

## 11. 应用场景

| 应用领域 | 说明 |
|----------|------|
| AI 智能玩具 | 带大模型对话与情绪识别的陪伴玩具 |
| 智能音箱 / 中控 | 语音控制 + 触屏交互的桌面终端 |
| 智能家居中枢 | 通过 MCP / Function Call 控制本地智能设备 |
| 教育与原型 | 端侧 AI 应用快速验证、复刻学习 |
| 桌面陪伴机器人 | 配合旋转底座，视觉对视 + 身体语言交互 |

---

## 12. 相关资源与链接

### 官方资源

| 资源 | 链接 |
|------|------|
| ESP-VoCat 用户指南（中文） | https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32s3/esp-vocat/index.html |
| ESP-VoCat v1.2 用户指南 | https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32s3/esp-vocat/user_guide_v1.2.html |
| ESP-VoCat v1.0 用户指南 | https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32s3/esp-vocat/user_guide_v1.0.html |
| ESP-VoCat 内置固件用户指南 | https://espressif.craft.me/tuJafhHDAiLKGz |
| 出厂固件仓库（examples） | https://github.com/espressif/esp-dev-kits/tree/master/examples/esp-vocat |
| ESP-Brookesia（UI 框架） | https://github.com/espressif/esp-brookesia/tree/master/products/speaker |
| ESP-VoCat v1.0 原理图 (PDF) | https://dl.espressif.com/AE/esp-dev-kits/ESP-VoCat_SCH_V1_0.pdf |
| ESP-VoCat v1.0 PCB 布局图 (ZIP) | https://dl.espressif.com/AE/esp-dev-kits/ESP-VoCat_pcb_V1_0.zip |
| 显示屏规格书 (PDF) | https://dl.espressif.com/AE/esp-dev-kits/UE018HV-RB39-A002A%20%20V1.0%20SPEC.pdf |
| 复刻教程（立创开源） | https://oshwhub.com/esp-college/echoear |

### 配套 App

| App | 平台 |
|-----|------|
| ESP RainMaker Home | [App Store (iOS)](https://apps.apple.com/in/app/esp-rainmaker-home/id1563728960) / [Google Play (Android)](https://play.google.com/store/apps/details?id=com.espressif.novahome) |

### 视频资源

- ESP-VoCat（喵伴）发布视频：https://www.bilibili.com/video/BV17AMwzBEqG/

### 相关项目

| 项目 | 描述 |
|------|------|
| esp-agents-firmware | ESP 私有 Agent 设备固件与示例（RainMaker 配网、语音对话、本地工具） |
| esp-brookesia | 面向 AIoT 的人机交互框架（HMI、Agent、MCP 协议等） |

---

> **文档总结**：ESP-VoCat（喵伴）是乐鑫面向大模型 AI 语音交互的旗舰开发套件，硬件上集成 ESP32-S3-WROOM-1（16M Flash + 16M PSRAM）、1.85" 360×360 圆形 QSPI 触摸屏（ST77916）、双麦阵列（声源定位）、3W 扬声器、IMU 与 700mAh 电池，三板结构设计。软件上以 ESP-Brookesia 为核心，深度融合扣子/OpenAI/Gemini 等大模型，实现全双工对话、情绪识别、长记忆、自定义角色、MCP 协议控制等能力，是端侧 AI 陪伴/中控类产品的理想参考平台。
