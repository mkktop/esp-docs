# ESP-CSI 框架详解 — Wi-Fi 感知（Wi-Fi Sensing）方案

> 文档编号：36
> 适用芯片：ESP32 / ESP32-S3 / ESP32-C3 / ESP32-C5 / ESP32-C6 等（CSI 性能：C5 ＞ C6 ＞ C3 ≈ S3 ＞ ESP32）
> 适用场景：人体存在检测、动作识别、入侵检测、室内定位、呼吸/微动感知、智能家居无感联动
> 来源：乐鑫官方 ESP-CSI 方案介绍、GitHub 仓库、ESP-IDF Wi-Fi Vendor 特性、ESP-CSI 微信公众号

---

## 目录

1. [ESP-CSI 概述](#1-esp-csi-概述)
2. [什么是 CSI](#2-什么是-csi)
3. [方案优势](#3-方案优势)
4. [常见应用场景](#4-常见应用场景)
5. [核心组件](#5-核心组件)
6. [ESP-CSI 数据结构与获取](#6-esp-csi-数据结构与获取)
7. [共晶振多天线方案（esp-crab）](#7-共晶振多天线方案esp-crab)
8. [ESP32-S3 在 CSI 中的角色](#8-esp32-s3-在-csi-中的角色)
9. [典型应用：智能无感联动](#9-典型应用智能无感联动)
10. [参考资源](#10-参考资源)

---

## 1. ESP-CSI 概述

### 1.1 什么是 ESP-CSI

ESP-CSI 是乐鑫提供的 **Wi-Fi 感知（Wi-Fi Sensing）技术**，利用 ESP 系列芯片的 Wi-Fi 信道状态信息（Channel State Information）实现**人体存在检测、动作识别**等感知能力。它是一项实验性框架，把 Wi-Fi 从单纯的"数据传输"升级为"空间感知"工具——无需任何额外传感器，仅通过软件即可让 ESP 设备具备"看见"环境变化的能力。

### 1.2 核心理念

ESP32 系列芯片可利用 CSI 数据实现**动作检测（motion detection）**和**存在检测（presence detection）**，无论是自动调节灯光、风扇，还是节能控制，CSI 都为智能家居带来新可能。它甚至能感知手指摆动、头部转动、呼吸起伏等细微动作。

---

## 2. 什么是 CSI

**CSI（Channel State Information，信道状态信息）** 是描述 Wi-Fi 无线信道状态的详细数据。Wi-Fi 信号从路由器传到 ESP 芯片时，会被墙壁、家具、人体反射/折射/遮挡，发生扭曲衰减。Wi-Fi 协议内置"导频（pilot）"机制——发送已知参考信号，接收端对比理想与实际信号，得到信道变化的细节，即 CSI。

```
路由器(TX) ──Wi-Fi 信号──> [墙壁/家具/人体 多径反射] ──> ESP(RX)
                                                            │
                                            比对导频 → CSI（子载波幅度+相位）
```

### CSI vs RSSI

| 指标 | 信息量 | 说明 |
|------|--------|------|
| RSSI（信号强度） | 粗糙 | 仅测信号强度，单一数值 |
| **CSI** | 丰富 | 含**每个子载波**的幅度、相位，频域细粒度描述 |

ESP 芯片最多可提供 **306 个子载波**的 CSI 数据。CSI 振幅实质是信道衰减系数，对环境变化极其敏感，对电源适配器/跳频器干扰很稳健。

---

## 3. 方案优势

对比毫米波雷达、UWB 等感知技术，ESP-CSI 基于 Wi-Fi 的优势：

| 优势 | 说明 |
|------|------|
| **零额外硬件成本** | 直接利用 ESP 芯片自身 Wi-Fi 特性，无需额外传感器 |
| **实时本地响应** | ESP 芯片本地实时处理 CSI，快速响应、低延迟 |
| **低功耗** | 相比高频雷达功耗更低，适合大规模/电池部署 |
| **抗干扰** | CSI 振幅是信道衰减系数，对电源适配器/跳频器干扰稳健 |
| **感知范围广** | Wi-Fi 覆盖广，不受视距限制，**支持非视距（NLoS）穿墙感知** |
| **细粒度** | 以子载波为单位采样频率响应，频域精细描述 |

---

## 4. 常见应用场景

| 场景 | 说明 |
|------|------|
| **人体存在检测（Presence Detection）** | 静坐不动也能感知（动作停止后仍判断有人），用于"人走灯灭" |
| **动作检测（Motion Detection）** | 检测行走、奔跑、举手、点头、摇头等大幅度/中幅度动作 |
| **微动作/呼吸感知** | 感知手指摆动、头部转动、呼吸胸腔起伏（新一代芯片如 C5 可达） |
| **入侵检测（Intruder Detection）** | 高灵敏子载波组合 + 非视距方向信号，被动式人员检测安防方案 |
| **室内定位与测距** | CSI 作丰富指纹（多子载波幅度+相位）或频率选择性衰减模型 |
| **智能家居无感联动** | 人来灯亮、走近台灯自动亮、风扇根据位置调风向 |
| **活动识别** | 边缘部署轻量神经网络，本地识别手势/日常活动 |

---

## 5. 核心组件

ESP-CSI 生态提供以下可复用组件（发布在 ESP Component Registry）：

| 组件 | 作用 |
|------|------|
| **esp-radar** | ESP-CSI 的人体运动检测组件，实现 CSI 数据获取 + 人体移动检测 |
| **esp_csi_gain_ctrl** | 射频接收增益补偿控制组件，补偿 AGC 自动增益，方便产品集成 |
| **esp_wifi_sensing** | 基于 CSI 的 Wi-Fi 感知状态机，封装多通道管理、运动/存在检测、现场标定、事件回调 |

---

## 6. ESP-CSI 数据结构与获取

ESP-IDF Wi-Fi 驱动提供 CSI 回调接口，CSI 数据由子载波的信道频率响应组成。每个子载波响应用 **2 字节有符号数**记录（第一字节虚部，第二字节实部）。

### 6.1 CSI 数据字段

CSI 数据按长训练字段（LTF）分三类，根据数据包类型可能全部或部分存在：

| LTF 类型 | 说明 |
|----------|------|
| LLTF | 传统长训练字段（非 HT） |
| HT-LTF | HT 长训练字段 |
| STBC-HT-LTF | STBC HT 长训练字段 |

### 6.2 子载波数（按 PHY 标准）

| PHY 标准 | 子载波范围 | 总数 / 可用 |
|----------|-----------|-------------|
| 802.11a/g | -26 ~ +26 | 52 总 / 48 可用 |
| 802.11n, 20 MHz | -28 ~ +28 | 56 总 / 52 可用 |
| 802.11n, 40 MHz | -57 ~ +57 | 114 总 / 108 可用 |

### 6.3 获取 CSI 的 API

```c
// 配置 CSI
esp_wifi_set_csi_config(const wifi_csi_config_t *config);
// 注册 CSI 回调
esp_wifi_set_csi_rx_cb(...);
// 启用/禁用 CSI
esp_wifi_set_csi(bool enable);

// 回调中拿到 wifi_csi_info_t：
//   - buf：CSI 数据缓冲（虚部/实部成对）
//   - len：总字节数
//   - rx_ctrl：含 RSSI、底噪、接收时间、天线、带宽、STBC 等元信息
//   - first_word_invalid：前 4 字节是否无效（硬件限制）
```

---

## 7. 共晶振多天线方案（esp-crab）

业界 CSI 研究多基于多天线网卡。乐鑫通过**多芯片共晶振（co-crystal）**设计实现等效效果，`esp-crab` 提供 Wi-Fi CSI 射频相位同步方案：

| 模式 | 说明 |
|------|------|
| **自发自收模式** | 两片 ESP32-C5 分别收发，计算 CSI 相位，**毫米级感知**射频路径扰动，适合近距离精确感知 |
| **单发双收模式** | 一片 C5 发、两片 C5 收，分散部署实现**大范围空间感知**，对接高级算法 |

> 共晶振方案通过时钟缓冲器消除多芯片间时钟不同步引起的频率偏移，使系统具备毫米级空间分辨能力。

### 运动检测开发板（ESP32-C3 + ESP32-S3）

官方多天线共晶振开发板：**ESP32-C3**（经天线开关连 3 个板载定向天线，持续发 ESP-NOW 数据包）+ **ESP32-S3**（从环境反射包包中获取 CSI，运行检测算法利用相位信息）。两芯片间经时钟缓冲器连接消除相对频偏，结果实时显示在屏幕上。

---

## 8. ESP32-S3 在 CSI 中的角色

ESP32-S3 具备 Wi-Fi，CSI 性能排序中 **ESP32-C3 ≈ ESP32-S3**（仅次于 C5/C6），在 ESP-CSI 方案中主要担任：

| 角色 | 说明 |
|------|------|
| **CSI 接收 + 检测算法主机** | 在多天线共晶振方案中负责获取 CSI、运行动作/存在检测算法（C3 发、S3 收） |
| **智能设备主控** | 如 ESP32-S3 智能电扇，CSI 检测有人即开灯/开风扇，无人自动关电器节能 |
| **边缘 AI 推理** | 利用向量指令本地运行轻量神经网络做动作识别 |

> 新一代 ESP32-C5 / C6 在 CSI 上做了射频电路优化（更高接收灵敏度、更低静态噪声、支持 5 GHz CSI、固定增益模式），CSI 性能更优；ESP32-S3 仍是成熟可用的主力平台。

---

## 9. 典型应用：智能无感联动

以 ESP32-S3 智能无感风扇为例（方案已开源至立创平台 [esp-bldc](https://oshwhub.com/esp-college/esp-bldc)）：

```
1. CSI 实时采集 → 动作检测算法 / 存在检测算法计算结果
2. 结果超过阈值 → 判定有人 → 自动开灯 + 开风扇
3. 人停止动作但仍在场 → 动作检测低于阈值，存在检测仍判有人
   → 灯保持点亮，风扇切换自然风
4. 人离开 → 检测结果低于阈值 → 确认无人 → 自动关电器节能
```

实时界面四条曲线：动作检测结果（绿）、动作检测阈值（紫）、存在检测结果（蓝）、存在检测阈值（黄）。

---

## 10. 参考资源

| 资源 | 链接 |
|------|------|
| ESP-CSI GitHub | https://github.com/espressif/esp-csi |
| ESP-CSI 中文 README | https://github.com/espressif/esp-csi/blob/master/README_cn.md |
| ESP-CSI 方案介绍（中文） | https://docs.espressif.com/projects/esp-techpedia/zh_CN/latest/esp-friends/solution-introduction/esp-csi/esp-csi-solution.html |
| esp-radar 组件 | https://components.espressif.com/components/espressif/esp-radar |
| esp_csi_gain_ctrl 组件 | https://components.espressif.com/components/espressif/esp_csi_gain_ctrl |
| esp_wifi_sensing 组件 | https://components.espressif.com/components/espressif/esp_wifi_sensing |
| ESP-IDF Wi-Fi CSI（Vendor 特性） | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32/api-guides/wifi-driver/wifi-vendor-features.html#wi-fi |
| esp-crab 共晶振示例 | https://github.com/espressif/esp-csi/tree/master/examples/esp-crab |
| 智能无感风扇开源（esp-bldc） | https://oshwhub.com/esp-college/esp-bldc |
| CSI 人体检测（照明方案） | https://github.com/espressif/esp-csi/blob/master/examples/esp-radar/connect_rainmaker/README_cn.md |

### 视频资源

- ESP-CSI 通感一体化，让 Wi-Fi"看见"万物：https://www.bilibili.com/video/BV1Y3hHzVEbb/
- 乐鑫 ESP-CSI 智能人体感知检测方案：https://www.bilibili.com/video/BV1ui4y1o7fz/
- ESP-CSI: Transforming Wi-Fi into a Sensing Platform（YouTube）

---

> **文档总结**：ESP-CSI 是乐鑫的 Wi-Fi 感知（Wi-Fi Sensing）技术，利用 ESP 芯片 Wi-Fi 信道状态信息（CSI，子载波级幅度+相位，比 RSSI 丰富得多）实现人体存在检测、动作识别、入侵检测、室内定位甚至呼吸微动感知。优势是零额外硬件、本地实时、低功耗、抗干扰、支持穿墙非视距感知。核心组件包括 esp-radar（人体运动检测）、esp_csi_gain_ctrl（增益补偿）、esp_wifi_sensing（感知状态机），并通过 esp-crab 共晶振多芯片方案实现毫米级相位同步感知。ESP32-S3 在其中常作 CSI 接收与检测算法主机（与 ESP32-C3 配合），是智能家居无感联动（人来灯亮/人走灯灭）的低成本感知底座。
