# ESP32-S3 硬件设计指南 (Hardware Design Guidelines)

> 来源: https://docs.espressif.com/projects/esp-hardware-design-guidelines/en/latest/esp32s3/index.html
> 版本: v1.8

本文档提供 ESP32-S3 SoC 的硬件设计指南，涵盖原理图设计、PCB 布局和常见问题解决。

---

## 1. 概述

ESP32-S3 的高度集成使其外围电路设计非常简单。一个基础的 ESP32-S3 电路仅需：
- **20 个电气元件**（电阻、电容、电感）
- **1 个晶振**
- **1 个 SPI Flash**

### 主要设计模块

1. [电源设计](#2-电源设计)
2. [上电和复位时序](#3-上电和复位时序)
3. [Flash 和 PSRAM](#4-flash-和-psram)
4. [时钟源](#5-时钟源)
5. [RF 射频](#6-rf-射频设计)
6. [UART](#7-uart)
7. [Strapping 引脚](#8-strapping-引脚)
8. [GPIO](#9-gpio)
9. [ADC](#10-adc)
10. [USB](#11-usb)
11. [触摸传感器](#12-触摸传感器)

---

## 2. 电源设计

### 2.1 通用建议

- 使用单电源时，推荐电压 **3.3 V**，输出电流 **不低于 500 mA**
- 电源入口建议添加 **ESD 保护二极管** 和至少 **10 µF 电容**

### 2.2 数字电源

| 引脚 | 名称 | 说明 | 电压范围 |
|------|------|------|---------|
| Pin 46 | VDD3P3_CPU | 数字电源 | 3.0 ~ 3.6 V |
| Pin 20 | VDD3P3_RTC | RTC 及部分数字电源 | 3.0 ~ 3.6 V |
| Pin 29 | VDD_SPI | 外部 Flash/PSRAM 电源 | 1.8 V 或 3.3 V (默认) |

**建议**: 在数字电源引脚附近添加 **0.1 µF 去耦电容**。在 VDD_SPI 附近添加额外的 0.1 µF 和 1 µF 去耦电容，不要使用过大的电容。

### 2.3 VDD_SPI 电压控制

VDD_SPI 有两种工作模式：

| 模式 | 电压 | 供电来源 |
|------|------|---------|
| 1.8 V | 1.8 V | ESP32-S3 内部 LDO (最大 40 mA) |
| 3.3 V (默认) | 3.3 V | 通过 14 Ω 电阻由 VDD3P3_RTC 供电 |

#### VDD_SPI 电压控制表

| EFUSE_VDD_SPI_FORCE | GPIO45 | EFUSE_VDD_SPI_TIEH | 电压 | VDD_SPI 电源来源 |
|---------------------|--------|--------------------|-----|-----------------|
| 0 | 0 | 忽略 | 3.3 V | VDD3P3_RTC 经 RSPI (默认) |
| 0 | 1 | 忽略 | 1.8 V | Flash 电压调节器 |
| 1 | 忽略 | 0 | 1.8 V | Flash 电压调节器 |
| 1 | 忽略 | 1 | 3.3 V | VDD3P3_RTC 经 RSPI |

**注意**:
- VDD_SPI 作为 3.3 V Flash/PSRAM 电源时，需确保 VDD3P3_RTC 保持在 3.0 V 以上（考虑压降）
- VDD3P3_RTC 不能单独供电，所有电源必须同时上电

---

## 3. 上电和复位时序

ESP32-S3 上电和复位时序要求：
- **CHIP_PU (复位) 引脚**: 不可悬空
- 电源稳定后，CHIP_PU 需要保持低电平一段时间再拉高
- 建议使用 RC 延迟电路确保正确的上电时序

---

## 4. Flash 和 PSRAM

ESP32-S3 需要封装内或封装外 Flash 来存储固件和数据。封装内或封装外 PSRAM 是可选的。

### 4.1 封装内 Flash 引脚映射

**Quad SPI Flash 映射表**:

| ESP32-S3FN8 / ESP32-S3FH4R2 | 封装内 Flash (Quad SPI) |
|------------------------------|------------------------|
| SPICLK | CLK |
| SPICS0 | CS# |
| SPID | DI |
| SPIQ | DO |
| SPIWP | WP# |
| SPIHD | HOLD# |

### 4.2 封装内 PSRAM 引脚映射

**Quad SPI PSRAM (2 MB) 映射表**:

| ESP32-S3R2 / ESP32-S3FH4R2 | 封装内 PSRAM (Quad SPI) |
|-----------------------------|------------------------|
| SPICLK | CLK |
| SPICS1 | CE# |
| SPID | SI/SIO0 |
| SPIQ | SO/SIO1 |
| SPIWP | SIO2 |
| SPIHD | SIO3 |

**Octal SPI PSRAM (8 MB) 映射表**:

| ESP32-S3R8 / ESP32-S3R8V | 封装内 PSRAM (Octal SPI) |
|---------------------------|--------------------------|
| SPICLK | CLK |
| SPICS1 | CE# |
| SPID | DQ0 |
| SPIQ | DQ1 |
| SPIWP | DQ2 |
| SPIHD | DQ3 |
| GPIO33 | DQ4 |
| GPIO34 | DQ5 |
| GPIO35 | DQ6 |
| GPIO36 | DQ7 |
| GPIO37 | DQS/DM |

### 4.3 PCB 布局建议

- 在 SPI 走线上靠近 ESP32-S3 放置 **0 Ω 串联电阻**
- SPI 走线尽量布在 **内层**（如第三层），并在时钟和数据走线周围添加地铜和地过孔
- **Octal SPI 走线需等长匹配**
- 如果 Flash/PSRAM 距 ESP32-S3 较远，建议在 VDD_SPI 和 Flash/PSRAM 电源附近放置合适的去耦电容

---

## 5. 时钟源

### 5.1 外部晶振 (必需)

ESP32-S3 需要 **40 MHz** 晶振作为主时钟源。

### 5.2 晶振 PCB 布局

- 确保 RF、晶振和芯片有完整的地平面
- 晶振应远离时钟引脚，间距至少 **2.0 mm**，避免干扰芯片
- 时钟走线周围添加高密度地过孔以增强隔离
- 时钟输入输出走线不应有换层过孔
- 与晶振走线串联的元件应靠近芯片侧放置
- 外部匹配电容应放在晶振两侧，最好在时钟走线末端
- 晶振下方不要走高频数字信号，最好不走任何信号
- 晶振是敏感元件，附近不要放置磁性元件（如大电感），确保周围有干净的大面积地平面

### 5.3 32.768 kHz 晶振 (可选)

ESP32-S3 可选连接 32.768 kHz 晶振（连接到 XTAL_32K_P/N 引脚），用于提高 RTC 精度。

---

## 6. RF 射频设计

### 6.1 RF 走线要求

- RF 走线特性阻抗 **50 Ω**
- 需要 CLC 匹配电路进行芯片调谐
- 使用 0201 元件，靠近引脚呈"之"字形放置
- RF 走线应宽度一致，不分支
- RF 走线尽量短，周围密集地过孔屏蔽干扰
- RF 走线布在外层，不换层
- 走线拐弯用 135° 角或圆弧
- 相邻层的地平面必须完整，RF 走线下方不走任何信号

### 6.2 谐波抑制 (Stub)

- 在靠近芯片侧的接地电容上添加 stub 来抑制二次谐波
- Stub 长度建议 **15 mil**
- Stub 特性阻抗 100 Ω ± 10%
- Stub 过孔连接到第三层，第一层和第二层保持禁布区
- **注意**: 0402 及以上封装不需要 stub

### 6.3 天线布局

- **IPEX 连接器**: 所有层在下方保持禁布区
- **PCB 天线**: 需通过仿真和实际测试验证，建议增加额外的 CLC 匹配电路
- 天线远离高频元件（晶振、DDR SDRAM、高频时钟等）
- USB 端口、USB 转串口芯片、UART 信号线必须尽量远离天线
- UART 信号线应被地铜和地过孔包围

---

## 7. UART

ESP32-S3 有 3 个 UART 接口。设计建议：
- UART0 通常用于固件下载和日志输出
- 连接 USB 转串口芯片（如 CP2102、CH340）时，确保信号完整性
- UART 走线远离 RF 走线和晶振

---

## 8. Strapping 引脚

ESP32-S3 有多个 Strapping 引脚，用于控制启动模式、VDD_SPI 电压等：

| 引脚 | 默认状态 | 功能 |
|------|---------|------|
| GPIO0 | 上拉 | 启动模式选择 |
| GPIO45 | 下拉 | VDD_SPI 电压选择 |
| GPIO46 | 下拉 | 启动模式选择 |

### 启动模式控制

| GPIO0 | GPIO46 | 启动模式 |
|-------|--------|---------|
| 1 | X | SPI Flash 启动 (正常运行) |
| 0 | 0 | 下载模式 (UART) |
| 0 | 1 | 下载模式 (SPI) |

**注意**: 上电时 Strapping 引脚电平被采样，外部电路不应在这些引脚上加下拉或强制电平，否则可能导致启动异常。

---

## 9. GPIO

ESP32-S3 有 45 个可编程 GPIO，通过 GPIO 矩阵可灵活映射到任意外设。

### GPIO 设计建议

- GPIO 矩阵允许将任意 GPIO 映射到任意外设信号
- 注意 GPIO 的供电域（VDD3P3_RTC、VDD_SPI、VDD3P3_CPU）
- 对于 ESP32-S3R16V，VDD_SPI 为 1.8 V，GPIO47 和 GPIO48 工作电压也是 1.8 V
- 使用 ESP32-S3R8/R16V 时，GPIO35/36/37 被 Octal SPI PSRAM 占用，不可用于其他功能

---

## 10. ADC

ESP32-S3 有 2 个 12-bit SAR ADC：
- **ADC1**: GPIO1~GPIO10 (10 个通道)
- **ADC2**: GPIO11~GPIO20 (10 个通道)

**注意**: Wi-Fi 工作时，ADC2 不可用。

---

## 11. USB

ESP32-S3 集成全速 USB 2.0 OTG 控制器和 USB Serial/JTAG 控制器：
- **USB D-**: GPIO19
- **USB D+**: GPIO20

USB 设计建议：
- USB D+/D- 走线差分等长
- 阻抗 90 Ω 差分
- 添加 ESD 保护

---

## 12. 触摸传感器

ESP32-S3 支持 14 个电容触摸引脚 (GPIO1~GPIO14)。

---

## 13. PCB 布局设计

### 13.1 推荐四层 PCB 设计

| 层 | 名称 | 用途 |
|----|------|------|
| Layer 1 (TOP) | 信号层 | 信号走线和元件 |
| Layer 2 (GND) | 地层 | 不走信号，确保完整地平面 |
| Layer 3 (POWER) | 电源层 | 地平面隔离 RF 和晶振，走电源线和少量信号线 |
| Layer 4 (BOTTOM) | 底层 | 走少量信号线，不建议放元件 |

### 13.2 两层 PCB 设计

| 层 | 名称 | 用途 |
|----|------|------|
| Layer 1 (TOP) | 信号层 | 信号走线和元件 |
| Layer 2 (BOTTOM) | 底层 | 不放元件，走线最少化，确保芯片、RF、晶振有完整地平面 |

### 13.3 电源走线建议

- 四层板优先，电源走线尽量在内层（非地层），通过过孔连接到芯片引脚
- 主电源走线换层至少 **2 个过孔**，过孔钻径不小于电源走线宽度
- **3.3 V 主电源走线宽度**: 不小于 **25 mil**
- **VDD3P3 (Pin 2, 3) 电源走线宽度**: 不小于 **20 mil**
- **其他电源走线宽度**: 推荐 **10 mil**
- 电源走线周围用地铜包围
- 电源入口附近放置 ESD 保护二极管，添加 10 µF 电容
- Pin 2 和 Pin 3 是 RF 相关电源，各放置一个 10 µF 电容（可并联 0.1 µF 或 1 µF）
- Pin 2 和 Pin 3 附近添加 CLC/LC 滤波电路抑制高频谐波
- 除 10 µF 电容外，推荐使用 0201 元件
- 芯片底部接地焊盘通过至少 **9 个地过孔** 连接到地平面

---

## 14. 常见布局问题与解决方案

### 问题 1: 发送数据包时电压纹波小，但 RF TX 性能差

**原因**: RF TX 性能受晶振影响。晶振质量差、频偏大，或晶振时钟被干扰信号（如高速输入输出、SDIO 走线、UART 走线）干扰。敏感元件（电感、天线）也可能降低 RF 性能。

**解决**: 重新布局晶振区域，参见 [时钟源](#5-时钟源)。

### 问题 2: 发送功率远高于或低于目标值，EVM 较差

**原因**: RF 引脚到天线传输线阻抗不匹配导致信号反射，影响内部 PA 工作状态，使 PA 过早进入饱和区，信号失真导致 EVM 变差。

**解决**: 用 RF 走线上的 π 型电路匹配天线阻抗，使天线阻抗与芯片阻抗接近，最小化反射。

### 问题 3: TX 性能正常，但 RX 灵敏度低

**原因**: RX 灵敏度低可能是外部耦合到天线。晶振信号谐波可能耦合到天线。UART 的 TX/RX 走线与 RF 走线交叉也会影响 RX 性能。板上高频干扰源会影响信号完整性。

**解决**: 天线远离晶振，高频信号走线远离 RF 走线，参见 [RF 射频设计](#6-rf-射频设计)。

---

## 15. 相关文档

- [ESP32-S3 Datasheet](https://www.espressif.com/sites/default/files/documentation/esp32-s3_datasheet_en.pdf)
- [ESP32-S3 Schematic Checklist](https://docs.espressif.com/projects/esp-hardware-design-guidelines/en/latest/esp32s3/schematic-checklist.html)
- [ESP32-S3 PCB Layout Design](https://docs.espressif.com/projects/esp-hardware-design-guidelines/en/latest/esp32s3/pcb-layout-design.html)
- [ESP32-S3 Download Guidelines](https://docs.espressif.com/projects/esp-hardware-design-guidelines/en/latest/esp32s3/download-guidelines.html)
- [ESP32-S3 中文硬件设计指南](https://docs.espressif.com/projects/esp-hardware-design-guidelines/zh_CN/latest/esp32s3/index.html)
