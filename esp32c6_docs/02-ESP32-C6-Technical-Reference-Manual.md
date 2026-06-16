# ESP32-C6 技术参考手册详解

> 文档编号：02
> 适用范围：ESP32-C6 全系列芯片
> 来源：乐鑫官方《ESP32-C6 技术参考手册》、ESP-IDF 编程指南（esp32c6）

---

## 目录

1. [技术参考手册概述](#1-技术参考手册概述)
2. [HP RISC-V 处理器](#2-hp-risc-v-处理器)
3. [LP RISC-V 处理器](#3-lp-risc-v-处理器)
4. [双核协作与启动](#4-双核协作与启动)
5. [低功耗管理（HP + LP）](#5-低功耗管理hp--lp)
6. [安全架构（TEE）](#6-安全架构tee)
7. [外设控制器](#7-外设控制器)
8. [Wi-Fi 6 与 802.15.4](#8-wi-fi-6-与-802154)
9. [参考资源](#9-参考资源)

---

## 1. 技术参考手册概述

《ESP32-C6 技术参考手册》（TRM）是芯片全功能的权威使用指南。与 C3 的主要区别：

| 维度 | ESP32-C6 | ESP32-C3 |
|------|-----------|-----------|
| CPU 核 | HP + LP 双核 | 单核 |
| Wi-Fi | **Wi-Fi 6** (802.11ax) | Wi-Fi 4 (802.11n) |
| BLE | Bluetooth 5.3 | Bluetooth 5 |
| 802.15.4 | **Thread + Zigbee** | 无 |
| 安全 | **TEE + AES-256-XTS** | 无 TEE |
| USB | **USB OTG** | USB Serial/JTAG |
| RAM | 512 KB HP + 16 KB LP | 400 KB 单块 |

---

## 2. HP RISC-V 处理器

高性能核是主应用处理器，负责 Wi-Fi/BT/802.15.4 协议栈和用户应用程序。

| 参数 | 规格 |
|------|------|
| 架构 | RISC-V 32 位（RV32IMC），4 级流水线 |
| 频率 | 最高 **160 MHz** |
| ROM | 320 KB |
| HP SRAM | 512 KB |
| Cache | 可配置 I/D cache |
| 中断 | 向量中断，可配置优先级 |
| 调试 | JTAG + trace |

### HP 系统组件

| 组件 | 说明 |
|------|------|
| GDMA | 3 通道 RX + 3 通道 TX |
| 通用定时器 × 2 | 54 位 |
| 系统定时器 | 52 位 |
| 主系统看门狗 | |
| 权限控制 | PMP，16 区域 |
| Cache | I/D cache 加速 flash 访问 |
| Event Task Matrix | 核间/外设事件高效路由 |

---

## 3. LP RISC-V 处理器

低功耗核是 ESP32-C6 独有设计，可在 HP 核休眠时**独立处理低功耗外设任务**，实现极低功耗。

| 参数 | 规格 |
|------|------|
| 架构 | RISC-V 32 位（RV32IMC） |
| 频率 | 最高 **20 MHz** |
| LP SRAM | 16 KB |

### LP 专属外设

| 外设 | 说明 |
|------|------|
| LP_I2C | LP 核专属 I2C（低功耗 I2C） |
| LP_UART | LP 核专属 UART（低功耗 UART） |
| LP RTC Timer | LP 核定时器（RTC 时钟） |
| LP Watchdog | LP 看门狗 |

---

## 4. 双核协作与启动

### 4.1 HP + LP 协作模式

ESP32-C6 支持 HP 和 LP 核的多种协作方式：

| 模式 | 说明 |
|------|------|
| HP 独立运行 | HP 核跑全部，LP 核关闭 |
| LP 独立运行 | HP 核休眠，LP 核处理低功耗任务（BLE Light-sleep 监听等） |
| 并行运行 | HP 跑应用 + Wi-Fi，LP 跑 BLE 协议栈 |
| LP 唤醒 HP | LP 核满足条件后唤醒 HP 核 |

### 4.2 启动流程

1. ROM bootloader（HP 核）在 ROM 中执行
2. 加载二级 bootloader（可选）到 SRAM
3. HP 核启动用户应用程序
4. LP 核可由 HP 核配置后独立运行低功耗任务

---

## 5. 低功耗管理（HP + LP）

### 5.1 低功耗模式（全芯片）

| 模式 | HP 核 | LP 核 | 说明 |
|------|--------|--------|------|
| Active | 运行 | 运行/休眠 | 全速 |
| Modem-sleep | 休眠 | 可运行 | Wi-Fi/BT 关闭 |
| Light-sleep | 休眠 | 可运行 | RTC 工作，状态保留 |
| Deep-sleep | 关闭 | 独立运行 | 仅 LP 核 + RTC |
| Hibernation | 关闭 | 关闭 | 仅 RTC 定时器 |
| Off | 关闭 | 关闭 | CHIP_EN 低 |

### 5.2 LP 核的独特价值

LP 核在 Deep-sleep 中**独立运行低功耗任务**是 C6 的核心创新：
- BLE 广播监听（LP 核独立处理，HP 核可保持休眠）
- 定时唤醒上报传感器数据
- 低功耗外设轮询

### 5.3 Wi-Fi 6 TWT

目标唤醒时间（TWT）是 Wi-Fi 6 专为低功耗设计：
- 设备与 AP 协商"唤醒时间表"
- 设备只在约定时间唤醒通信，其余时间深度休眠
- 比传统 DTIM 省电 3~5 倍

---

## 6. 安全架构（TEE）

ESP32-C6 在 C3 安全机制基础上增加了 **TEE（可信执行环境）**：

| 安全组件 | 说明 |
|----------|------|
| Secure Boot v2 | RSA-3072 / ECC 验签 |
| Flash 加密 | **AES-128/256-XTS** |
| TEE | 可信执行环境，硬件隔离关键资产 |
| DS | 数字签名外设 |
| HMAC | 消息认证 |
| SHA / AES / ECC / RSA | 全套硬件加密加速 |
| RNG | 随机数 |
| 时钟毛刺检测 | 抗故障注入 |
| eFuse | 1792 位用户可用 |

---

## 7. 外设控制器

### HP 外设（完整列表）

| 外设 | 说明 |
|------|------|
| USB 2.0 OTG | 全速 12 Mbps，支持主机和设备模式 |
| USB Serial/JTAG | 免调试器下载 |
| SDIO 2.0 从机 | 连接外部 SDIO 主机 |
| SPI × 3 | SPI0/1（flash）、SPI2（通用） |
| UART × 2 | |
| I2C × 2 | |
| I2S × 1 | PDM/TDM 音频 |
| LED PWM | 多通道 |
| RMT | 红外 TX/RX + WS2812 时序 |
| TWAI® | CAN 2.0 兼容 |
| MCPWM | 电机控制 |
| PARLIO | 并行高速 IO |
| PCNT | 脉冲计数 |
| 12 位 SAR ADC | 多通道 |
| 温度传感器 | 内置 |

### LP 外设

| 外设 | 说明 |
|------|------|
| LP_I2C | 低功耗 I2C |
| LP_UART | 低功耗 UART |
| LP RTC Timer | LP 定时器 |
| LP Watchdog | |

---

## 8. Wi-Fi 6 与 802.15.4

### 8.1 Wi-Fi 6 协议栈

| 特性 | 说明 |
|------|------|
| 802.11ax | OFDMA（上行/下行）、MU-MIMO 下行、TWT、BSS Color |
| 802.11b/g/n | 向下兼容 |
| 4 虚拟接口 | Station / SoftAP / Station+SoftAP / 混杂 |
| FTM | 精细时间测量（测距） |

### 8.2 802.15.4 协议栈

| 协议 | 说明 |
|------|------|
| Thread 1.3 | Matter-over-Thread 设备必备 |
| Zigbee 3.0 | 智能照明与传感 |
| CSL | 协调器/路由/终端设备 |

> C6 可同时作为 Wi-Fi STA + Thread 终端，实现 Matter Wi-Fi + Thread 双协议设备。

---

## 9. 参考资源

| 资源 | 链接 |
|------|------|
| ESP32-C6 技术参考手册（PDF） | https://www.espressif.com/sites/default/files/documentation/esp32-c6_technical_reference_manual_cn.pdf |
| ESP-IDF 编程指南（C6） | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c6/index.html |
| ESP32-C6 H/W 参考 | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c6/hw-reference/index.html |
| 低功耗模式指南 | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c6/api-guides/low-power-mode/index.html |
| Wi-Fi 6 (802.11ax) 指南 | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c6/api-guides/wifi.html |

---

> **文档总结**：ESP32-C6 TRM 的核心是双核 RISC-V 架构——HP 核（160 MHz，512 KB SRAM，运行主应用与 Wi-Fi/BT/802.15.4 协议栈）和 LP 核（20 MHz，16 KB SRAM，可在 HP 休眠时独立处理 BLE 监听/定时等低功耗任务）。新增 TEE 可信执行环境（硬件隔离）和 AES-256-XTS（比 C3 更强）。Wi-Fi 6 协议栈支持 OFDMA/MU-MIMO/TWT/BSS Color。802.15.4 支持 Thread 1.3 + Zigbee 3.0。双核 + LP 核独立运行使 C6 在保持 BLE 监听的同时实现极低功耗。USB OTG 是另一亮点（12 Mbps 全速）。配合 [01-Datasheet](01-ESP32-C6-Datasheet.md) 构成完整硬件参考。
