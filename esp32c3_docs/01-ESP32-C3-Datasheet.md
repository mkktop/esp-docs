# ESP32-C3 系列芯片技术规格书详解

> 文档编号：01
> 适用范围：ESP32-C3 全系列芯片（ESP32-C3 / C3FH4 / C3FH4X / C3FN4 / C3FH4AZ / C3FH8X）
> 来源：乐鑫官方《ESP32-C3 系列芯片技术规格书》v2.4、产品选型页、ESP-RISC-V CPU 文档

---

## 目录

1. [产品概述](#1-产品概述)
2. [芯片型号与封装](#2-芯片型号与封装)
3. [CPU 与存储](#3-cpu-与存储)
4. [Wi-Fi 与蓝牙](#4-wi-fi-与蓝牙)
5. [外设接口](#5-外设接口)
6. [安全机制](#6-安全机制)
7. [功耗管理](#7-功耗管理)
8. [RF 模块](#8-rf-模块)
9. [Strapping 管脚与启动配置](#9-strapping-管脚与启动配置)
10. [应用场景](#10-应用场景)
11. [参考资源](#11-参考资源)

---

## 1. 产品概述

ESP32-C3 是一款低功耗、高集成度的 MCU 系统级芯片（SoC），集成 **2.4 GHz Wi-Fi** 和**低功耗蓝牙（Bluetooth LE）**无线通信。它是乐鑫面向物联网（IoT）推出的 **RISC-V 架构**芯片，主打**低成本、低功耗、强安全**。

### 核心亮点

- **RISC-V 32 位单核处理器**：四级流水线架构，主频高达 160 MHz
- **行业领先的低功耗与射频性能**：802.11b 发射功率 +21 dBm
- **完善安全机制**：RSA-3072 安全启动、AES-128-XTS flash 加密、数字签名、HMAC、硬件加密加速器
- **内置存储**：400 KB SRAM、384 KB ROM，支持外部 SPI/Dual SPI/Quad SPI/QPI flash
- **丰富外设**：USB Serial/JTAG、TWAI（CAN 2.0）、LED PWM、RMT 红外、I2S 等

### 功能框图

```
┌─────────────────────────────────────────────────────┐
│                   ESP32-C3 SoC                       │
├──────────────┬──────────────┬───────────────────────┤
│ CPU & Memory │     RF       │ Wireless Digital      │
│  RISC-V 32b  │ 2.4G Balun   │ Wi-Fi MAC/Baseband    │
│  Cache/SRAM  │ 2.4G RX/TX   │ BLE Link Ctrl/Baseband│
│  JTAG/ROM    │ RF Synth     │                       │
├──────────────┴──────────────┴───────────────────────┤
│ Peripherals: GDMA, RMT, SPI0/1/2, I2C, I2S, UART×2, │
│   TWAI, Timers, LED PWM, SysTimer, GPIO, ADC,        │
│   USB Serial/JTAG, WDT, eFuse, Temp Sensor           │
├──────────────────────────────────────────────────────┤
│ Security: SHA, RSA, AES, RNG, HMAC, DS, Secure Boot, │
│           Flash Encryption                            │
├──────────────────────────────────────────────────────┤
│ RTC: RTC Memory, PMU, Brownout Detector              │
└──────────────────────────────────────────────────────┘
```

---

## 2. 芯片型号与封装

| 型号 | 状态 | Flash | GPIO | 说明 |
|------|------|-------|------|------|
| ESP32-C3 | 在产 | 外接 | 22 | 无内置 flash 版本 |
| **ESP32-C3FH4X** | **推荐** | 4 MB（封装内） | 16 | 推荐用于新设计 |
| ESP32-C3FH4 | 在产 | 4 MB（封装内） | 22 | 封装内 4 MB flash |
| ESP32-C3FH8X | 在产 | 8 MB（封装内） | 16 | 封装内 8 MB flash |
| ESP32-C3FN4 | EOL（停产） | 4 MB（封装内） | 22 | 不推荐新设计 |
| ESP32-C3FH4AZ | NRND | 4 MB（封装内） | 16 | 不推荐新设计 |

- **封装**：QFN32，**5×5 mm**
- **GPIO 数量**：FH4X/FH4AZ 为 16 个（封装内 flash 占用引脚被复用），其余 22 个
- **衍生芯片**：ESP8685 是 ESP32-C3 的更小封装（QFN 4×4）版本，详见 [05-ESP8685-Datasheet](05-ESP8685-Datasheet.md)

---

## 3. CPU 与存储

### 3.1 ESP-RISC-V CPU

| 参数 | 规格 |
|------|------|
| 架构 | RISC-V 32 位（RV32IMC：基本整数 I + 乘除法 M + 压缩 C） |
| 核心 | 单核，四级有序标量流水线 |
| 主频 | 最高 160 MHz |
| CoreMark 性能 | 483.27 CoreMarks（@160 MHz），3.02 CoreMark/MHz |
| 中断控制器 | 多达 31 个向量中断，可配置优先级与阈值 |
| 调试 | RISC-V Debug v0.13，支持 JTAG/USB，8 个断点/观察点 |
| 内存保护 | PMP（物理内存保护），最多 16 个区域 |
| 系统总线 | 32 位 AHB |

### 3.2 片上存储

| 类型 | 容量 |
|------|------|
| ROM | 384 KB |
| SRAM | 400 KB（其中 16 KB 专用于 cache） |
| RTC SRAM | 8 KB |
| eFuse | 4096 位（用户可用高达 1792 位） |

### 3.3 Flash 与 DMA

- **封装内 flash**：不同型号 4/8 MB（见型号表）
- **外部 Flash 接口**：支持 SPI、Dual SPI、Quad SPI、QPI 连接多个封装外 flash
- **Cache 加速**：通过 cache 加速 flash 访问
- **ICP**：支持 flash 在线编程
- **GDMA**：通用 DMA 控制器，3 个接收通道 + 3 个发送通道

---

## 4. Wi-Fi 与蓝牙

### 4.1 Wi-Fi

- 协议：**IEEE 802.11b/g/n**
- 频段：2.4 GHz，支持 20 MHz 和 40 MHz 频宽
- 模式：1T1R，数据速率高达 **150 Mbps**
- **4 个虚拟 Wi-Fi 接口**：同时支持 Station / SoftAP / Station+SoftAP / 混杂模式
  - 注意：Station 模式扫描时，SoftAP 信道会同时改变
- 特性：WMM、A-MPDU/A-MSDU 帧聚合、Immediate Block ACK、分片重组、TXOP、Beacon 自动监测（硬件 TSF）、天线分集、**802.11mc FTM**

### 4.2 蓝牙（Bluetooth LE）

- **Bluetooth 5** + **Bluetooth mesh**
- 高功率模式，发射功率最高 **20 dBm**
- 速率：125 Kbps、500 Kbps、1 Mbps、2 Mbps
- 广播扩展（Advertising Extensions）、多广播（Multiple Advertisement Sets）
- 信道选择算法 #2
- **Wi-Fi 与蓝牙共存**，共用同一天线

---

## 5. 外设接口

### 5.1 通信接口

| 接口 | 数量/说明 |
|------|-----------|
| UART | 2 个 |
| SPI | 3 个（SPI0/1 用于 flash，SPI2 通用） |
| I2C | 1 个 |
| I2S | 1 个 |
| **USB Serial/JTAG** | 全速 USB 串口/JTAG 控制器（免外部调试器） |
| **TWAI®** | 兼容 ISO 11898-1（CAN 规范 2.0） |
| LED PWM | 多达 6 通道 |
| RMT（红外） | 2 个发送通道 + 2 个接收通道 |

### 5.2 模拟与定时器

| 类型 | 说明 |
|------|------|
| ADC | 2 个 12 位 SAR ADC，多达 6 通道 |
| 温度传感器 | 内置 |
| 通用定时器 | 2 个 54 位 |
| 看门狗 | 3 个数字 WDT + 模拟 WDT + XTAL32K WDT |
| 系统定时器 | 52 位 |

### 5.3 GPIO

- ESP32-C3 / C3FH4 / C3FN4 / C3FH8X：**22 个**（3 个 strapping + 6 个用于封装内 flash）
- ESP32-C3FH4X / C3FH4AZ：**16 个**（3 个 strapping）

---

## 6. 安全机制

| 安全特性 | 说明 |
|----------|------|
| 安全启动（Secure Boot） | 基于 **RSA-3072**，内外存储器权限控制 |
| Flash 加密 | 基于 **AES-128-XTS** |
| AES 加速器 | AES-128/256（FIPS PUB 197） |
| SHA 加速器 | FIPS PUB 180-4 |
| RSA 加速器 | — |
| RNG | 随机数生成器 |
| HMAC | 硬件加速 |
| 数字签名（DS） | 创新的数字签名外设 |
| 时钟毛刺检测 | 抗故障注入 |

---

## 7. 功耗管理

ESP32-C3 支持四种功耗模式（按平均功耗由高到低）：

| 模式 | 功耗（典型值） | 说明 |
|------|----------------|------|
| **Active**（RF 工作） | 见 RF 功耗表 | 全模块工作，Wi-Fi TX 峰值可达 335 mA |
| **Modem-sleep** | 13 ~ 28 mA | CPU 运行，Wi-Fi/BT 基带与射频关闭 |
| **Light-sleep** | **130 µA** | CPU 暂停，RTC 外设/内存工作，状态保留 |
| **Deep-sleep** | **5 µA** | 仅 RTC 定时器 + RTC 内存工作 |
| 关闭 | 1 µA | CHIP_EN 拉低 |

### Active 模式 RF 功耗（3.3V，25°C，100% 占空比）

| 工作模式 | 描述 | 峰值电流 |
|----------|------|----------|
| TX | 802.11b, 1 Mbps, @21 dBm | 335 mA |
| TX | 802.11g, 54 Mbps, @19 dBm | 285 mA |
| TX | 802.11n, HT20, MCS7, @18.5 dBm | 276 mA |
| TX | 802.11n, HT40, MCS7, @18.5 dBm | 278 mA |
| RX | 802.11b/g/n, HT20 | 84 mA |
| RX | 802.11n, HT40 | 87 mA |

> 低功耗详细配置与唤醒源见 [15-ESP32-C3-Low-Power-Mode-Guide](15-ESP32-C3-Low-Power-Mode-Guide.md)。

---

## 8. RF 模块

| 特性 | 规格 |
|------|------|
| 集成度 | 天线开关、射频巴伦（balun）、PA、LNA 集成在芯片内 |
| 802.11b 发射功率 | 高达 **+21 dBm** |
| 802.11n 发射功率 | 高达 +20 dBm |
| BLE 接收灵敏度（125 Kbps） | 高达 **-105 dBm** |

---

## 9. Strapping 管脚与启动配置

ESP32-C3 有 3 个 strapping 管脚：**GPIO2、GPIO8、GPIO9**，在复位时被采样锁存。

| Strapping 管脚 | 默认状态 |
|----------------|----------|
| GPIO2 | 浮空 |
| GPIO8 | 浮空 |
| GPIO9 | 弱上拉（默认 1） |

### 启动模式控制

| 启动模式 | GPIO9 | GPIO8 | GPIO2 |
|----------|:-----:|:-----:|:-----:|
| **SPI Boot（默认）** | 1 | 任意值 | 1 |
| Joint Download Boot | 0 | 1 | 1 |

- **SPI Boot**：从 SPI flash 读取程序启动
- **Joint Download Boot**：支持 USB-Serial-JTAG / UART 下载
- ⚠️ **GPIO8 和 GPIO9 不可同时为低电平**
- 建议在 GPIO9 处预留上拉电阻，且**不要加大电容**（否则误入下载模式）

> 详细时序、ROM 日志打印控制、下载接线见 [03-ESP32-C3-Hardware-Design-Guidelines](03-ESP32-C3-Hardware-Design-Guidelines.md) 与 [02-ESP32-C3-Technical-Reference-Manual](02-ESP32-C3-Technical-Reference-Manual.md)。

---

## 10. 应用场景

ESP32-C3 专为物联网设备设计，典型领域：

- 智能家居、工业自动化、医疗保健
- 消费电子产品、智慧农业
- POS 机、服务机器人
- 音频设备
- 通用低功耗 IoT 传感器集线器、数据记录器
- 小家电旋钮屏（洗衣机、体脂秤、电动牙刷）
- 电池供电的低功耗 BLE/Wi-Fi 设备

---

## 11. 参考资源

| 资源 | 链接 |
|------|------|
| ESP32-C3 技术规格书（PDF） | https://www.espressif.com/sites/default/files/documentation/esp32-c3_datasheet_cn.pdf |
| ESP32-C3 技术规格书（在线） | https://documentation.espressif.com/esp32-c3_datasheet_cn.html |
| ESP32-C3 产品页（芯片） | https://www.espressif.com/zh-hans/products/socs/esp32-c3 |
| ESP32-C3 产品页（模组） | https://www.espressif.com/zh-hans/products/modules?id=ESP32-C3 |
| 乐鑫产品选型工具 | https://products.espressif.com/#/product-selector?language=zh |
| ESP32-C3 芯片勘误表 | https://docs.espressif.com/projects/esp-chip-errata/zh_CN/latest/esp32c3/index.html |
| PCN / Advisories | https://espressif.com/zh-hans/support/documents/pcns?keys=ESP32-C3 |

---

> **文档总结**：ESP32-C3 是乐鑫 RISC-V 架构主力 IoT 芯片，单核 160 MHz RV32IMC，内置 400 KB SRAM + 384 KB ROM，QFN32 5×5 封装。集成 2.4 GHz Wi-Fi（4 虚拟接口，+21 dBm）与 BLE 5（+20 dBm，-105 dBm 灵敏度），外设含 USB Serial/JTAG、TWAI（CAN）、6 通道 LED PWM、RMT 红外等。安全上具备 RSA-3072 安全启动、AES-128-XTS flash 加密及全套硬件加密加速器。功耗上 Light-sleep 仅 130 µA、Deep-sleep 低至 5 µA，是低成本、低功耗、强安全物联网设备的优选。
