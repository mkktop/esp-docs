# ESP32-C3 硬件设计指南详解

> 文档编号：03
> 适用范围：基于 ESP32-C3 芯片的产品硬件设计
> 来源：乐鑫官方《ESP32-C3 硬件设计指南》、原理图设计 checklist、Try Firmware 硬件接线

---

## 目录

1. [设计指南概述](#1-设计指南概述)
2. [核心电路组成](#2-核心电路组成)
3. [电源设计](#3-电源设计)
4. [上电时序与复位](#4-上电时序与复位)
5. [Flash 设计](#5-flash-设计)
6. [时钟源](#6-时钟源)
7. [射频（RF）设计](#7-射频rf设计)
8. [Strapping 管脚](#8-strapping-管脚)
9. [UART 与 USB 下载](#9-uart-与-usb-下载)
10. [GPIO 与 ADC](#10-gpio-与-adc)
11. [PCB 设计要点](#11-pcb-设计要点)
12. [参考资源](#12-参考资源)

---

## 1. 设计指南概述

《ESP32-C3 硬件设计指南》提供基于 ESP32-C3 芯片的产品设计规范，覆盖原理图设计、PCB 布局、射频、电源等。它帮助开发者把芯片正确集成到自有产品中，避免常见设计陷阱。

ESP32-C3 核心电路非常精简——**只需约 20 个电阻电容电感 + 1 个无源晶振 + 1 个 SPI flash**（封装内带 flash 的型号连外部 flash 都省了）。

---

## 2. 核心电路组成

ESP32-C3 核心电路重要组成部分：

| 部分 | 说明 |
|------|------|
| 电源 | 数字/模拟/RTC 三类电源域 |
| 上电时序与复位 | CHIP_EN 与上电稳定时序 |
| Flash | 封装内或封装外 SPI flash |
| 时钟源 | 主晶振 + 可选 32.768 kHz RTC 晶振 |
| 射频（RF） | 天线匹配网络 |
| UART | 下载与日志 |
| Strapping 管脚 | 启动模式配置 |
| GPIO / ADC / USB | 外设引脚 |

---

## 3. 电源设计

### 3.1 通用要点

- 单电源供电时，建议 **3.3 V**，最大输出电流**至少 500 mA**
- 总电源入口添加 **ESD 保护器件 + ≥10 µF 大电容**

### 3.2 三类电源域

| 电源域 | 管脚 | 工作电压 | 去耦建议 |
|--------|------|----------|----------|
| **数字电源** | VDD3P3_CPU（pin 17） | 3.0 ~ 3.6 V | 靠近管脚加 0.1 µF |
| **模拟电源** | VDDA、VDD3P3 | 3.0 ~ 3.6 V | VDD3P3 加 10 µF + 0.1 µF + LC 滤波（电感额定电流 ≥500 mA） |
| **RTC 电源** | VDD3P3_RTC | 3.0 ~ 3.6 V | 靠近管脚加 0.1 µF |

> VDD3P3 在 TX 时瞬间电流大，易引起电源轨道塌陷，必须加 10 µF 电容。

### 3.3 VDD_SPI

- 作为输出电源时：由 VDD3P3_CPU 经 RSPI 电阻供电，典型 3.3 V，靠近管脚加 1 µF
- 可接外部电源输入，或不用时作 GPIO
- 给 3.3 V flash 供电时，电压须保证 3.0 V 以上，此时不可作 GPIO

### 3.4 RTC 电源注意

VDD3P3_RTC **不可作为备用电源单独供电**。

---

## 4. 上电时序与复位

CHIP_EN（使能/复位）管脚控制芯片上电与复位：

| 参数 | 说明 | 最小值 |
|------|------|--------|
| tSTBL | CHIP_EN 拉高前，VDDA/VDD3P3/VDD3P3_RTC/VDD3P3_CPU 达到稳定所需时间 | 50 µs |
| tRST | CHIP_EN 电平低于 VIL_nRST 复位芯片的时间 | 50 µs |

> ⚠️ CHIP_PU（EN）**不能浮空**；3.3V 供电时 EN 必须为高电平。

---

## 5. Flash 设计

- **封装内 flash 型号**（C3FH4/FH4X/FH4X/FH8X）：flash 已在芯片封装内，无需外部 flash，SPI0/1 总线已内部连接
- **外接 flash 型号**（ESP32-C3）：需外接 SPI flash，通过 SPI/Dual SPI/Quad SPI/QPI 连接
- Flash ICP（在线编程）支持，便于产线烧录

---

## 6. 时钟源

| 时钟 | 频率 | 说明 |
|------|------|------|
| **主晶振** | 40 MHz | 无源晶振，连接 XTAL_P/XTAL_N |
| RTC 晶振（可选） | 32.768 kHz | 外部晶振，提供更精准 RTC 时钟；无外部 32K 时单 BLE Light-sleep 功耗为 mA 级，有则可降至 µA 级 |

> 主晶振外部电路配置基本的去耦电容即可（芯片内部已集成较多 RF 相关电路）。

---

## 7. 射频（RF）设计

ESP32-C3 已集成天线开关、射频巴伦（balun）、PA、LNA，**外部 RF 电路极简**：

- LNA_IN 射频引脚 → **π 型或 L 型阻抗匹配网络**（匹配至 50 Ω） → 天线
- 天线可选 PCB 天线、3D 天线或外接 IPEX 座子（模组如 WROOM-02U）
- 匹配网络需在 2.4 GHz 频段优化 Wi-Fi + BLE 共存

---

## 8. Strapping 管脚

GPIO2、GPIO8、GPIO9 为 strapping 管脚，控制启动模式：

| 启动模式 | GPIO9 | GPIO8 | GPIO2 |
|----------|:-----:|:-----:|:-----:|
| 默认（SPI Boot） | 1（上拉） | 浮空 | 浮空 |
| SPI Boot | 1 | 任意值 | 1 |
| Joint Download Boot | 0 | 1 | 1 |

### 设计要点

- **GPIO9 建议预留上拉电阻**
- **GPIO9 不要加大电容**（否则误进下载模式）
- GPIO2 虽不实际控制 SPI/Joint Download Boot，但**建议上拉**防毛刺
- ⚠️ GPIO8 与 GPIO9 **不可同时为低**
- 复位后 strapping 管脚作为普通 GPIO 使用

---

## 9. UART 与 USB 下载

ESP32-C3 支持 **UART0** 或 **USB** 两种固件下载方式：

### 9.1 UART0 下载接线

| ESP32-C3 | 3.3V 电源 | 串口工具 |
|----------|----------|----------|
| 3V3 | VDD | — |
| GND | GND | GND |
| EN | VDD | — |
| GPIO2（拉高） | VDD | — |
| GPIO8（拉高） | VDD | — |
| GPIO9（拉低） | GND | — |
| TXD0（GPIO21） | — | RXD |
| RXD0（GPIO20） | — | TXD |

### 9.2 USB 下载接线

| ESP32-C3 | 3.3V 电源 | USB 线 |
|----------|----------|--------|
| 同上供电与 strapping | — | — |
| GPIO18 | — | USB_D-（白） |
| GPIO19 | — | USB_D+（绿） |

### 9.3 上电日志判断

进入下载模式时 UART0 打印：
```
ESP-ROM:esp32c3-api1-20210207
rst:0x1 (POWERON),boot:0x4 (DOWNLOAD(USB/UART0/1))
waiting for download
```

---

## 10. GPIO 与 ADC

- GPIO 通过 IO MUX + GPIO 交换矩阵灵活复用
- ADC：2 个 12 位 SAR ADC，6 通道（ADC1_CH0~4、ADC2_CH0）
- strapping 管脚复位后可作普通 GPIO

---

## 11. PCB 设计要点

- 电源去耦电容**就近放置**在对应 VDD 管脚旁
- VDD3P3 走线加宽，加 10 µF 电容抑制瞬态塌陷
- RF 走线控制 50 Ω 阻抗，π 匹配网络靠近 LNA_IN
- 晶振走线短、包地，远离干扰源
- 高速 USB（D±）差分走线等长、控阻抗
- 建议 4 层板（信号/地/电源/信号）

---

## 12. 参考资源

| 资源 | 链接 |
|------|------|
| ESP32-C3 硬件设计指南（在线） | https://docs.espressif.com/projects/esp-hardware-design-guidelines/zh_CN/latest/esp32c3/index.html |
| 原理图设计 checklist | https://docs.espressif.com/projects/esp-hardware-design-guidelines/zh_CN/latest/esp32c3/schematic-checklist.html |
| 硬件设计指南 PDF | https://www.espressif.com/sites/default/files/documentation/esp32-c3_hardware_design_guidelines_cn.pdf |
| ESP32-C3 Try Firmware 硬件接线 | https://docs.espressif.com/projects/esp-techpedia/zh_CN/latest/esp-friends/get-started/try-firmware/try-firmware-hardware/esp32c3.html |
| 乐鑫 KiCad 库 | https://github.com/espressif/kicad-libraries |
| Flash 下载工具 | https://docs.espressif.com/projects/esp-test-tools/zh_CN/latest/esp32/production_stage/tools/flash_download_tool.html |

---

> **文档总结**：ESP32-C3 硬件设计指南规定了基于该芯片的产品设计规范。核心电路精简（~20 个阻容感 + 1 晶振 + flash）。电源分数字/模拟/RTC 三域，VDD3P3 须加 10 µF 防 TX 轨道塌陷。上电/复位有 50 µs 时序要求。strapping 管脚 GPIO2/8/9 决定 SPI Boot（默认）或 Joint Download Boot，GPIO9 须预留上拉且不可加大电容，GPIO8/9 不可同时低。下载支持 UART0（TXD0/RXD0）与原生 USB（GPIO18/19）。RF 电路极简（芯片已集成 balun/PA/LNA），只需 50 Ω π 匹配网络。配合 KiCad 库与 Flash 下载工具可完成从设计到量产。
