# ESP32-C6 系列芯片技术规格书详解

> 文档编号：01
> 适用范围：ESP32-C6 / C6FH4 / C6FH8 全系列芯片
> 来源：乐鑫官方《ESP32-C6 系列芯片技术规格书》v1.5、产品选型页

---

## 目录

1. [产品概述](#1-产品概述)
2. [芯片型号与封装](#2-芯片型号与封装)
3. [双核架构（HP + LP）](#3-双核架构hp--lp)
4. [无线连接三合一](#4-无线连接三合一)
5. [存储与内存](#5-存储与内存)
6. [外设接口](#6-外设接口)
7. [安全机制](#7-安全机制)
8. [功耗模式](#8-功耗模式)
9. [应用场景](#9-应用场景)
10. [参考资源](#10-参考资源)

---

## 1. 产品概述

ESP32-C6 是乐鑫**首款支持 Wi-Fi 6 的 SoC**，集成 2.4 GHz Wi-Fi 6、Bluetooth 5 和 IEEE 802.15.4（Thread/Zigbee），是**三无线协议共存**的旗舰 RISC-V IoT 芯片。

### 核心亮点

- **Wi-Fi 6 (802.11ax)**：OFDMA 上行/下行、MU-MIMO 下行、TWT 目标唤醒时间（电池续航）、BSS Color 空间复用
- **Bluetooth 5.3 LE**：远距离 Coded PHY、2 Mbps、LE Power Control、Wi-Fi 共存
- **IEEE 802.15.4**：**Thread 1.3 + Zigbee 3.0**，Matter 全协议栈支持
- **双核 RISC-V**：HP（高性能，160 MHz）+ LP（低功耗，20 MHz）
- **完善安全**：RSA-3072 Secure Boot、AES-128/256-XTS Flash 加密、TEE 可信执行环境、ECC/RNG/HMAC/DS
- **原生 USB OTG**：全速 12 Mbps

### 功能框图

```
┌──────────────────────────────────────────────────────────┐
│                    ESP32-C6 SoC                          │
├──────────────┬──────────────┬────────────────────────────┤
│  HP RISC-V  │  LP RISC-V  │        RF                   │
│  160 MHz   │  20 MHz    │  Wi-Fi 6 (1T1R 2.4G)      │
│  512 KB HP │  16 KB LP  │  BT 5.3 LE                  │
│  SRAM      │  SRAM       │  802.15.4 (Thread/Zigbee)  │
├──────────────┴──────────────┴────────────────────────────┤
│ HP外设: SPI/I2C/UART/I2S/RMT/LEDC/ADC/USB/TWAI/MCPWM │
│ LP外设: LP_I2C/LP_UART/LP_RTC/Timer/WDT               │
├───────────────────────────────────────────────────────┤
│ 安全: Secure Boot / Flash Enc / TEE / AES / SHA / ECC  │
│      RSA / RNG / HMAC / DS / Clock Glitch Detection    │
└───────────────────────────────────────────────────────┘
```

---

## 2. 芯片型号与封装

| 型号 | 封装 | GPIO | Flash | 说明 |
|------|------|------|-------|------|
| ESP32-C6 | QFN40 5×5 mm | **30** | 外接 | 基础型号 |
| ESP32-C6FH4 | QFN40 5×5 mm | 30 | 4 MB 内置 | 停产(EOL) |
| ESP32-C6FH8 | QFN40 5×5 mm | 30 | 8 MB 内置 | 停产(EOL) |
| ESP32-C6FN4 | QFN32 5×5 mm | 22 | 4 MB 内置 | QFN32 小封装 |

> 主力在产型号为裸芯片（ESP32-C6 外接 flash）或 QFN40/QFN32 封装。

---

## 3. 双核架构（HP + LP）

这是 ESP32-C6 区别于 C3 的核心创新：**一颗芯片两个 RISC-V 处理器**。

| 处理器 | 频率 | 用途 | SRAM | ROM |
|--------|------|------|------|-----|
| **HP RISC-V**（高性能） | 最高 160 MHz | 运行主应用程序、Wi-Fi/BT/802.15.4 协议栈 | 512 KB | 320 KB |
| **LP RISC-V**（低功耗） | 最高 20 MHz | 运行 LP 核相关外设、低功耗管理、蓝牙轻量协议 | 16 KB | — |

两者可以协作运行（HP 睡眠时 LP 接管低功耗任务），也可以独立休眠。

---

## 4. 无线连接三合一

ESP32-C6 是乐鑫唯一同时支持 **Wi-Fi 6 + BT 5.3 + 802.15.4** 的芯片，三者共用同一 2.4 GHz 射频和天线。

### 4.1 Wi-Fi 6 (802.11ax)

| 特性 | 说明 |
|------|------|
| 协议 | 802.11ax 向下兼容 b/g/n |
| 频段 | 2.4 GHz，20 MHz 仅 20 MHz 信道 |
| MCS | MCS0~MCS9（1T1R，max 150 Mbps） |
| OFDMA | 上行/下行正交频分多址（高密度多用户场景） |
| MU-MIMO | 下行多用户多输出 |
| TWT | **目标唤醒时间**：设备商定唤醒时间，深度睡眠时间更长，电池续航大幅提升 |
| BSS Color | 空间复用，减少同信道干扰 |
| Beamformee | 波束成形接收端 |
| CQI / DCM | 信道质量指示/双载波调制 |

### 4.2 Bluetooth 5.3 LE

| 特性 | 说明 |
|------|------|
| 版本 | Bluetooth 5.3 LE |
| 模式 | LE Peripheral / Central / Broadcaster / Observer |
| PHY | 1M / 2M / Coded PHY（125K / 500K） |
| 发射功率 | 高功率模式 **+20 dBm** |
| 特色 | LE Power Control、Advertising Extensions、Channel Selection Algorithm #2 |

### 4.3 IEEE 802.15.4（Thread + Zigbee）

| 特性 | 说明 |
|------|------|
| 协议 | 802.15.4-2015 |
| 频段 | 2.4 GHz，OQPSK，250 Kbps |
| **Thread** | **Thread 1.3** 认证，Matter-over-Thread 设备 |
| **Zigbee** | **Zigbee 3.0** 认证 |
| 共存 | Wi-Fi + BT + 802.15.4 三协议时分共存 |

---

## 5. 存储与内存

| 类型 | 容量 |
|------|------|
| HP ROM | 320 KB |
| HP SRAM | 512 KB |
| LP SRAM | 16 KB |
| eFuse | 用户可用高达 1792 位 |
| 外部 Flash | 通过 SPI/Dual SPI/Quad SPI/QPI 连接，最大 16 MB（模组内） |

---

## 6. 外设接口

### HP（高性能核）外设

| 类型 | 说明 |
|------|------|
| GPIO | 最多 30 个（QFN40）/ 22 个（QFN32） |
| **USB 2.0 OTG** | 全速 12 Mbps，主机/设备模式 |
| **USB Serial/JTAG** | 免调试器下载与 JTAG |
| SDIO 2.0 | 从机 |
| SPI × 3 | SPI0/1（flash）、SPI2（通用 FSPI） |
| UART × 2 | |
| I2C × 2 | |
| I2S × 1 | |
| LED PWM | 多通道 |
| RMT | TX/RX 红外 + WS2812 |
| **TWAI®** | 兼容 CAN 2.0 |
| **MCPWM** | 电机控制 PWM |
| **PARLIO** | 并行 IO（高速数据） |
| **PCNT** | 脉冲计数器 |
| ADC | 12 位 SAR，多通道 |
| 温度传感器 | 内置 |
| GDMA | 通用 DMA |

### LP（低功耗核）外设

| 类型 | 说明 |
|------|------|
| LP_I2C | LP 核专属 I2C |
| LP_UART | LP 核专属 UART |
| LP RTC Timer | LP 核定时器 |
| LP 看门狗 | |

---

## 7. 安全机制

ESP32-C6 安全机制比 C3 更强，新增 **TEE（可信执行环境）**：

| 安全特性 | 说明 |
|----------|------|
| 安全启动 | 基于 **RSA-3072** |
| Flash 加密 | 基于 **AES-128/256-XTS**（比 C3 的 AES-128-XTS 更强） |
| **TEE** | **可信执行环境控制器**，基于硬件特性分配访问权限，软件隔离更安全 |
| 数字签名（DS） | 设备身份安全 |
| HMAC | 消息认证 |
| SHA | FIPS PUB 180-4 |
| AES | FIPS PUB 197 |
| **ECC** | **椭圆曲线加密**（新增，比 RSA 更高效的签名） |
| RNG | 随机数 |
| 时钟毛刺检测 | 抗故障注入 |

---

## 8. 功耗模式

ESP32-C6 支持六种功耗模式（硬件预设）：

| 模式 | 说明 |
|------|------|
| **Active** | 所有模块工作 |
| **Modem-sleep** | CPU 运行，Wi-Fi/BT 基带与射频关闭 |
| **Light-sleep** | CPU 暂停，RTC 工作，状态保留 |
| **Deep-sleep** | 仅 RTC，LP 核可独立运行 |
| **Hibernation** | 仅 RTC 定时器 + 8 个保留寄存器 |
| **Off** | CHIP_EN 拉低 |

> Wi-Fi 6 的 **TWT（目标唤醒时间）** 是专为低功耗设计：设备与 AP 协商唤醒时间表，可在协商的时间点精确唤醒，大幅延长电池续航。

---

## 9. 应用场景

- **Matter 智能家居**：Matter-over-Wi-Fi 终端 + Matter-over-Thread 终端（Matter 全栈）
- **Thread/Zigbee 设备**：Thread 1.3 节点、Zigbee 3.0 照明与传感
- **Wi-Fi 6 物联网**：TWT 省电、长续航电池供电设备
- **BLE + Wi-Fi 双模**：远程 + 近场混合场景
- **Matter 网关/边界路由器**：搭配 ESP32-S3 等作 Thread BR 或 Zigbee BR
- 智能家居、工业自动化、医疗保健

---

## 10. 参考资源

| 资源 | 链接 |
|------|------|
| ESP32-C6 规格书（PDF） | https://www.espressif.com/sites/default/files/documentation/esp32-c6_datasheet_cn.pdf |
| ESP32-C6 规格书（在线） | https://documentation.espressif.com/esp32-c6_datasheet_cn.html |
| ESP32-C6 产品页 | https://www.espressif.com/zh-hans/products/socs/esp32-c6 |
| 模组产品页 | https://www.espressif.com/zh-hans/products/modules?id=ESP32-C6 |
| 开发板产品页 | https://www.espressif.com/zh-hans/products/devkits?id=ESP32-C6 |
| 乐鑫产品选型工具 | https://products.espressif.com/#/product-selector?language=zh |
| ESP32-C6 芯片勘误表 | https://docs.espressif.com/projects/esp-chip-errata/zh_CN/latest/esp32c6/index.html |

---

> **文档总结**：ESP32-C6 是乐鑫三无线协议旗舰（Wi-Fi 6 + BT 5.3 + 802.15.4 Thread/Zigbee），双 RISC-V 核（HP 160 MHz + LP 20 MHz）协同工作，是 Matter 全协议（Wi-Fi/Thread）支持的核心芯片。512 KB HP SRAM + 320 KB ROM + 16 KB LP SRAM，外接最大 16 MB flash。安全上比 C3 增配 TEE 可信执行环境 + AES-256-XTS + ECC。Wi-Fi 6 的 TWT 特性使电池续航大幅提升。三协议共存 + LP 核独立低功耗管理，使 C6 成为 Matter 智能家居、Thread/Zigbee 终端、Wi-Fi 6 长续航电池设备的头号选择。
