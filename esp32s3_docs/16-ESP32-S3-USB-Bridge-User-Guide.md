# ESP32-S3-USB-Bridge 用户指南

> **文档来源**: Espressif 官方文档 (docs.espressif.com) | **语言**: 中文 | **更新日期**: 2026-06-14

---

## 目录

- [1. 开发板概述](#1-开发板概述)
- [2. 核心特性](#2-核心特性)
- [3. USB 桥接功能详解](#3-usb-桥接功能详解)
- [4. 硬件组件介绍](#4-硬件组件介绍)
- [5. 功能框图](#5-功能框图)
- [6. GPIO 分配](#6-gpio-分配)
- [7. 供电系统](#7-供电系统)
- [8. 快速入门](#8-快速入门)
- [9. 应用程序开发](#9-应用程序开发)
- [10. 工作模式详解](#10-工作模式详解)
- [11. 无线桥接功能](#11-无线桥接功能)
- [12. 应用场景与应用示例](#12-应用场景与应用示例)
- [13. 硬件版本历史](#13-硬件版本历史)
- [14. 相关文档与资源](#14-相关文档与资源)

---

## 1. 开发板概述

ESP32-S3-USB-Bridge 是乐鑫（Espressif）推出的一款**基于 ESP32-S3 芯片的 USB 桥接工具开发板**。该开发板通过在计算机（PC）和目标微控制器（MCU）之间建立桥接，可以**替代 USB 转 UART 芯片（如 CP210x、FT232）或专用调试器**。

### 1.1 核心定位

ESP32-S3-USB-Bridge 是一款体积小巧、功能强大的开发工具板，尺寸仅为 **23.3 mm × 31.5 mm**。它通过模拟成为 **USB 复合设备**，支持多种桥接功能，并增加了**无线桥接**能力。

### 1.2 四大核心功能

| 功能 | 说明 |
| --- | --- |
| **USB 转 UART 桥接** | 通过 USB 转 UART 桥接，实现计算机与目标芯片的串口数据收发 |
| **JTAG 适配器** | 通过 JTAG 桥接，实现计算机与目标芯片之间双向传输 JTAG 通信 |
| **MSC 存储设备** | 将 UF2 固件文件拖放到开发板的 USB 存储设备中，实现固件升级 |
| **无线桥接** | 通过 ESP-NOW，实现无线烧录以及无线串口数据收发 |

### 1.3 与 CP210x 的对比

| 特性 | CP210x | ESP32-S3-USB-Bridge |
| --- | --- | --- |
| USB 转串口 | 支持 | 支持 |
| JTAG 调试 | 不支持 | 支持 |
| MSC 拖拽烧录 | 不支持 | 支持 |
| 无线桥接 | 不支持 | 支持（ESP-NOW） |
| 可编程/可定制 | 否 | 是（ESP-IDF） |
| Wi-Fi 功能 | 无 | 有 |
| 蓝牙功能 | 无 | 有（BLE 5.0） |

---

## 2. 核心特性

### 2.1 硬件特性

| 特性 | 详细说明 |
| --- | --- |
| **板载模组** | ESP32-S3-MINI-1-N4R2，内置 **4 MB flash** 和 **2 MB PSRAM** |
| **指示灯** | 板载一颗 **WS2812** RGB 指示灯（GPIO42），以及两颗串口数据指示灯（TX/RX） |
| **USB 接口** | 板载 USB 转 UART 桥接器及 JTAG 适配器，支持 **USB Type-C** 和 **USB Type-A** 接口 |
| **扩展连接器** | 提供 12 个外接接口，包含 JTAG、串口、TX/RX、Boot、Reset 及系统电压管脚 |
| **按键** | Reset 按键（IO8）、Boot 按键（IO9）、模组 Boot 按键（IO0） |
| **电压转换** | 5 V 转 3.3 V 电路，为模组供电 |
| **板载尺寸** | 23.3 mm × 31.5 mm（极其紧凑） |
| **无线功能** | Wi-Fi 2.4 GHz + Bluetooth 5 (LE) + ESP-NOW |

### 2.2 ESP32-S3-MINI-1-N4R2 模组

| 参数 | 说明 |
| --- | --- |
| 模组型号 | ESP32-S3-MINI-1-N4R2 |
| 芯片 | ESP32-S3 系列 |
| Flash | 4 MB |
| PSRAM | 2 MB |
| 天线 | PCB 板载天线 |
| 特点 | 通用型 Wi-Fi + 低功耗蓝牙 MCU 模组，拥有强大的神经网络运算能力和信号处理能力 |
| 应用领域 | AIoT 领域多种应用场景 |

---

## 3. USB 桥接功能详解

### 3.1 USB 复合设备概念

ESP32-S3-USB-Bridge 连接到 PC 时，会创建一个**USB 复合设备**，通过单一 USB 连接提供多种功能。PC 通过 USB 电缆即可访问以下所有功能：

```
              ┌───────────────────────────────────┐
              │       ESP32-S3-USB-Bridge          │
              │       (USB 复合设备)                │
              │                                    │
   PC  ◄──────┤  ┌──────────┐  ┌──────────────┐   │
  (USB)       │  │ CDC 串口  │  │ JTAG 适配器   │   │
              │  │ (虚拟COM) │  │ (openocd)    │   │
              │  └─────┬────┘  └──────┬───────┘   │
              │        │              │            │
              │  ┌─────┴────┐  ┌──────┴───────┐   │
              │  │ MSC 存储  │  │ ESP-NOW 无线 │   │
              │  │ (UF2烧录) │  │ (无线桥接)   │   │
              │  └──────────┘  └──────────────┘   │
              └───────────────────────────────────┘
                            │
                    ┌───────┴───────┐
                    │   目标 MCU     │
                    │ (ESP32/ESP32-S3│
                    │   等)          │
                    └───────────────┘
```

### 3.2 USB 转 UART 桥接（Serial Bridge）

这是 ESP32-S3-USB-Bridge 最基本也是最重要的功能。

#### 工作原理

1. PC 通过 USB 连接到 ESP32-S3-USB-Bridge
2. Bridge MCU 通过 USB CDC 创建一个虚拟串口（Windows: `COMx`，Linux: `/dev/ttyACMx`，macOS: `/dev/cu.*`）
3. 开发者可以运行 `esptool` 或串口终端程序连接到该虚拟串口
4. Bridge MCU 将数据在 PC（USB）和目标 MCU（UART）之间双向透明传输

#### 关键特性

| 特性 | 说明 |
| --- | --- |
| 最大波特率 | 3,000,000 bps（3 Mbps） |
| 自动下载 | 支持 esptool 的自动固件下载功能 |
| 双向透传 | USB 和 UART 之间双向透明数据通信 |
| 兼容性 | 可用于烧录各种 ESP 系列芯片 |

#### 自动下载功能

自动下载功能默认开启，通过控制目标芯片的 Boot 和 EN（Reset）引脚自动进入下载模式：

- `AUTODLD_EN_PIN`（IO8）连接目标芯片的 EN（RST）引脚
- `AUTODLD_BOOT_PIN`（IO9）连接目标芯片的 IO0（Boot）引脚

可在 `menuconfig` 中禁用：`USB UART Bridge Demo → Enable auto download`

### 3.3 JTAG 适配器（JTAG Bridge）

#### 工作原理

1. PC 运行 `openocd-esp32` 连接到 ESP32-S3-USB-Bridge
2. Bridge MCU 作为 PC 和目标 MCU 之间的桥梁
3. 双向传输 JTAG 通信

#### JTAG 引脚映射

| JTAG 信号 | Bridge GPIO | 功能 |
| --- | --- | --- |
| TCK | IO4 | 同步测试数据传输 |
| TMS | IO5 | 测试模式选择 |
| TDI | IO3 | 测试数据输入 |
| TDO | IO2 | 测试数据输出 |

#### 使用场景

- 在线调试 ESP32 系列芯片
- 断点调试、单步执行
- 变量监视、寄存器查看
- 通过 GDB 进行源码级调试

### 3.4 MSC 存储设备（Mass Storage Device）

#### 工作原理

1. ESP32-S3-USB-Bridge 创建一个 USB 大容量存储设备
2. PC 文件浏览器中会出现一个新的磁盘驱动器
3. 将 **UF2 格式**的固件文件拖放到该磁盘
4. Bridge MCU 读取 UF2 文件并自动烧录到目标 MCU

#### 优势

- **无需安装任何工具**：不需要 esptool 或 idf.py
- **无需命令行操作**：只需拖拽文件
- **跨平台**：Windows、Linux、macOS 均可使用
- **支持芯片**：各种乐鑫微控制器

### 3.5 UF2 固件升级

基于 `esp-tinyuf2` 组件，ESP32-S3-USB-Bridge 可以作为虚拟 U 盘：

- 拖拽 UF2 固件实现 OTA 升级
- 支持将 NVS 数据映射到 U 盘文件，通过修改文件改写 NVS

---

## 4. 硬件组件介绍

### 4.1 开发板正面组件（顺时针顺序）

| 主要组件 | 描述 |
| --- | --- |
| **ESP32-S3-MINI-1-N4R2 模组** | 通用型 Wi-Fi + 低功耗蓝牙 MCU 模组，搭载 ESP32-S3 系列芯片，内置 4 MB flash 和 2 MB PSRAM。具有强大的神经网络运算能力和信号处理能力 |
| **TX/RX 指示灯** | 用于指示串口数据的收发状态 |
| **扩展连接器** | 可供连接的 JTAG 管脚、串口管脚、TX/RX 管脚、Boot 管脚、Reset 管脚以及系统电压管脚 |
| **Reset 按键** | 连接目标芯片的 Reset 按键，与模组的 IO8 相连。单独按下可复位目标芯片 |
| **USB 转 USB 接口** | 为整个系统提供电源，该端口用于 PC 与 ESP32-S3-MINI-1 模组的 USB 通信 |
| **Boot 按键** | 连接目标芯片的 Boot 按键，与模组的 IO9 相连。长按 Boot + 按 Reset 可启动固件上传模式 |

### 4.2 开发板背面组件

| 主要组件 | 描述 |
| --- | --- |
| **5 V 转 3.3 V** | 用于将 USB 电压转换为 3.3 V 电压，为 ESP32-S3-MINI-1 模组供电 |
| **模组 Boot 按键** | 连接模组的 IO0 按键。长按此按键再重新给开发板上电，即可让开发板进入下载模式，上传新固件 |
| **WS2812** | 与模组的 IO42 相连，用于指示开发板当前的状态（颜色编码不同工作模式） |

### 4.3 接口类型

ESP32-S3-USB-Bridge 提供两种 USB 接口选择：

#### Type-C 连接

通过 USB Type-C 接口连接 PC 和目标芯片，适合现代设备使用。

#### Type-A 连接

开发板还支持 USB Type-A 接口，更换方便，适合传统设备。

---

## 5. 功能框图

ESP32-S3-USB-Bridge 的主要组件和连接方式：

```
     ┌────────────────────────────────────────────────────────┐
     │                    PC (计算机)                          │
     │   esptool / openocd / 串口终端 / 文件管理器             │
     └──────────────────────┬─────────────────────────────────┘
                            │ USB
                            │
     ┌──────────────────────┴─────────────────────────────────┐
     │              ESP32-S3-USB-Bridge                        │
     │         ┌──────────────────────────┐                    │
     │         │    ESP32-S3-MINI-1-N4R2  │                    │
     │         │                          │                    │
     │         │  USB 复合设备:            │                    │
     │         │  ├── CDC (虚拟串口)       │                    │
     │         │  ├── JTAG 适配器          │                    │
     │         │  ├── MSC (U盘)           │                    │
     │         │  └── ESP-NOW (无线)      │                    │
     │         │                          │                    │
     │         │  WS2812 (状态指示)       │                    │
     │         │  TX/RX LED               │                    │
     │         └──────────┬───────────────┘                    │
     │                    │ 扩展连接器                          │
     └────────────────────┼────────────────────────────────────┘
                          │
              ┌───────────┼───────────┐
              │           │           │
         UART TX/RX   JTAG        Boot/Reset
         (IO40/IO41)  (IO2-5)     (IO9/IO8)
              │           │           │
     ┌────────┴───────────┴───────────┴────────┐
     │              目标 MCU                     │
     │         (ESP32 / ESP32-S3 / 等)           │
     └──────────────────────────────────────────┘
```

---

## 6. GPIO 分配

### 6.1 完整 GPIO 分配表

下表为 ESP32-S3-MINI-1 模组管脚及外接接口的 GPIO 分配列表：

| 管脚编号 | 管脚名称 | 功能 |
| --- | --- | --- |
| 1 | GND | 接地 |
| 2 | 3V3 | 供电 |
| 3 | IO0 | 模组 Boot 按键，用于进入下载模式，以及作为按键输入管脚 |
| 4 | IO2 | JTAG 管脚 **TDO**，用于测试数据输出 |
| 5 | IO3 | JTAG 管脚 **TDI**，用于测试数据输入 |
| 6 | IO4 | JTAG 管脚 **TCK**，用于同步测试数据传输 |
| 7 | IO5 | JTAG 管脚 **TMS**，用于测试模式选择 |
| 8 | IO8 | 连接目标芯片的 **Reset** 管脚，按下为低电平 |
| 9 | IO9 | 连接目标芯片的 **Boot** 管脚，按下为低电平 |
| 10 | IO19 | 与 **USB_D-** 接口相连 |
| 11 | IO20 | 与 **USB_D+** 接口相连 |
| 12 | IO40 | **RX**，用于连接目标芯片的 UART TX 管脚 |
| 13 | IO41 | **TX**，用于连接目标芯片的 UART RX 管脚 |
| 14 | IO42 | **WS2812** 控制管脚 |

> **注意**：管脚 3-14 为开发板提供的外接接口。除上表所列内容外，所有引出 IO 均可作为其他用途，其中 GPIO5 和 GPIO8 与外部按键相连。

### 6.2 引脚连接示意图（与目标芯片）

```
    ESP32-S3-USB-Bridge                    目标 MCU
    ┌───────────────┐                   ┌──────────────┐
    │          IO40 ├──── RX ◄──── TX ───┤ TX           │
    │          IO41 ├──── TX ────► RX ───┤ RX           │
    │           IO9 ├──── Boot ── Boot ──┤ IO0(Boot)    │
    │           IO8 ├──── Reset ─ EN ────┤ EN(Reset)    │
    │           IO2 ├──── TDO ◄─── TDO ──┤ TDO          │
    │           IO3 ├──── TDI ───► TDI ──┤ TDI          │
    │           IO4 ├──── TCK ───► TCK ──┤ TCK          │
    │           IO5 ├──── TMS ───► TMS ──┤ TMS          │
    │          GND  ├──── GND ──── GND ──┤ GND          │
    │          3V3  ├──── 3V3 ──── 3V3 ──┤ 3V3          │
    └───────────────┘                   └──────────────┘
```

---

## 7. 供电系统

### 7.1 USB 供电方式

ESP32-S3-USB-Bridge 支持两种 USB 供电方式：

#### 方式 1：通过 Type-A 端口供电

使用 USB Type-A 接口连接 PC 或电源适配器，为整个系统提供电源。

#### 方式 2：通过 Type-C 端口供电

使用 USB Type-C 接口连接 PC 或电源适配器，为整个系统提供电源。

### 7.2 电压转换

ESP32-S3-USB-Bridge 内置 **5 V 转 3.3 V** 电压转换电路，将 USB 接口的 5 V 电压转换为 3.3 V，为 ESP32-S3-MINI-1 模组供电。

---

## 8. 快速入门

### 8.1 硬件准备

- ESP32-S3-USB-Bridge 开发板
- 目标 MCU 开发板（如 ESP32-DevKitC、ESP32-S3-DevKitC-1 等）
- USB 数据线（Type-C 或 Type-A）
- 杜邦线（用于连接 Bridge 和目标 MCU）
- 电脑（Windows、Linux 或 macOS）

### 8.2 硬件连接

1. 使用 USB 线将 ESP32-S3-USB-Bridge 连接到 PC
2. 使用杜邦线连接 ESP32-S3-USB-Bridge 与目标 MCU：

| Bridge 管脚 | 目标 MCU 管脚 | 用途 |
| --- | --- | --- |
| RX (IO40) | TX | 串口数据接收 |
| TX (IO41) | RX | 串口数据发送 |
| Boot (IO9) | IO0 (Boot) | 下载模式控制 |
| Reset (IO8) | EN (RST) | 复位控制 |
| GND | GND | 共地 |
| 3V3 | 3V3（可选） | 供电 |

3. PC 应识别到一个新的 USB 复合设备
   - Windows：设备管理器中出现新的 `COMx`
   - Linux：`/dev/ttyACMx`
   - macOS：`/dev/cu.*`

### 8.3 验证串口功能

连接成功后，使用串口终端工具（如 PuTTY、minicom、screen 等）打开虚拟串口，即可与目标 MCU 进行串口通信。

### 8.4 验证烧录功能

使用 `esptool` 验证烧录功能：

```bash
# 查看设备信息
esptool.py -p /dev/ttyACMx chip_id

# 或使用 idf.py
idf.py -p /dev/ttyACMx flash monitor
```

### 8.5 进入下载模式（烧录 Bridge 固件）

如需烧录 ESP32-S3-USB-Bridge 自身的固件：

1. **按住** ESP32-S3 Boot 按键（背面的模组 Boot 按键，IO0）
2. **重新上电**（拔插 USB 线）
3. 松开 Boot 按键
4. 开发板进入下载模式
5. 使用 esptool 或 idf.py 烧录新固件

> **注意**：请不要按住模组自身的 Boot 按键后上下电，防止默认固件被替换。

---

## 9. 应用程序开发

### 9.1 软件框架

ESP32-S3-USB-Bridge 的开发框架为 **ESP-IDF**（Espressif IoT Development Framework）。ESP-IDF 是基于 FreeRTOS 的乐鑫 SoC 开发框架，具有众多组件，包括 LCD、ADC、RMT、SPI 等。

### 9.2 ESP-IDF 版本要求

ESP32-S3-USB-Bridge 支持以下 ESP-IDF 版本：

| ESP-IDF 版本 | 支持状态 |
| --- | --- |
| release/v5.0 及所有 Bugfix 版本 | 支持 |
| release/v5.1 及所有 Bugfix 版本 | 支持 |
| release/v5.2 及所有 Bugfix 版本 | 支持 |
| release/v5.3 及所有 Bugfix 版本 | 支持 |
| release/v5.4 及所有 Bugfix 版本 | 支持 |

### 9.3 获取示例代码

```bash
# 克隆 esp-dev-kits 仓库
git clone https://github.com/espressif/esp-dev-kits.git

# 进入 usb_wireless_bridge 示例
cd esp-dev-kits/examples/esp32-s3-usb-bridge/examples/usb_wireless_bridge
```

### 9.4 编译和烧录

```bash
# 设置 ESP-IDF 环境变量
. $HOME/esp/esp-idf/export.sh

# 设置目标芯片
idf.py set-target esp32s3

# 针 ESP32-S3-USB-Bridge 开发板进行配置
rm -rf build sdkconfig sdkconfig.old
idf.py -D SDKCONFIG_DEFAULTS="sdkconfig.defaults;sdkconfig.esp32-s3-usb-bridge" reconfigure

# 编译、烧录并监控
idf.py -p PORT flash monitor
```

> **注意**：对于 ESP32-S3-USB-Bridge 开发板，需要**按住 ESP32-S3 Boot 按键，重新上电**才能烧录。

### 9.5 工程配置

在示例目录下输入以下命令可以配置工程选项：

```bash
idf.py menuconfig
```

项目特定的设置在 **"Bridge Configuration"** 子菜单中，包括：
- 引脚编号设置
- 供应商 ID（Vendor ID）
- 产品 ID（Product ID）
- 自动下载功能开关
- USB 描述符配置

### 9.6 在线固件烧录

也可以通过浏览器直接烧录固件，无需安装 ESP-IDF：

- 在线烧录地址：https://espressif.github.io/esp-launchpad/?flashConfigURL=https://dl.espressif.com/AE/esp-dev-kits/config.toml

### 9.7 基于 ESP USB Bridge 项目的开发

ESP32-S3-USB-Bridge 的固件基于 [ESP USB Bridge](https://github.com/espressif/esp-usb-bridge) 项目二次开发，增加了通过 ESP-NOW 进行无线烧录和串口通信的功能。

ESP USB Bridge 项目的核心特点：

- 使用 ESP-IDF v5.0 或更新版本编译
- 支持 ESP32-S2 和 ESP32-S3 芯片
- 每块开发板应有自己的供应商 ID 和产品 ID
- 可在 [Espressif USB Vendor PID 仓库](https://github.com/espressif/usb-pids) 注册产品 ID

---

## 10. 工作模式详解

ESP32-S3-USB-Bridge 支持**三种工作模式**：

### 10.1 模式功能对比

| 功能 | 有线模式 | 无线主机模式 | 无线从机模式 |
| --- | --- | --- | --- |
| **USB 串口通信** | 支持 | 支持 | 支持 |
| **串口烧录** | 支持 | 支持 | 支持 |
| **JTAG 调试** | 支持 | - | - |
| **MSC** | 支持 | 支持 | - |
| **UF2 升级** | 支持 | - | - |

### 10.2 有线模式（Wired Mode）

有线模式是最常用的工作模式，支持所有功能：

```
PC ◄──── USB ────► ESP32-S3-USB-Bridge ◄──── UART/JTAG ────► 目标 MCU
```

支持的功能：
- USB 转串口通信
- JTAG 调试
- UF2 固件升级
- MSC 存储设备

### 10.3 无线主机模式（Wireless Host Mode）

无线主机模式通过 ESP-NOW 与无线从机进行通信：

```
PC ◄──── USB ────► ESP32-S3-USB-Bridge (主机) ◄══ ESP-NOW ══► ESP32-S3-USB-Bridge (从机) ◄──── UART ────► 目标 MCU
                                                                              (无线连接)
```

支持的功能：
- USB 串口通信（无线转发）
- 串口烧录（无线烧录）
- MSC 存储设备

### 10.4 无线从机模式（Wireless Slave Mode）

无线从机模式接收来自无线主机的指令：

```
PC ◄──── USB ────► ESP32-S3-USB-Bridge (主机) ══ ESP-NOW ══► ESP32-S3-USB-Bridge (从机) ◄──── UART ────► 目标 MCU
```

支持的功能：
- 串口通信（通过无线主机转发）
- 串口烧录（通过无线主机转发）

### 10.5 模式切换

#### 默认模式颜色

WS2812 指示灯颜色对应的工作模式：

| 颜色 | 工作模式 |
| --- | --- |
| **紫色** | 有线模式 |
| **蓝色** | 无线主机模式 |
| **绿色** | 无线从机模式 |

#### 切换方法

- **双击开发板 Boot 按键**：切换工作状态
- 指示灯变色代表模式切换成功

---

## 11. 无线桥接功能

### 11.1 ESP-NOW 概述

ESP-NOW 是乐鑫定义的一种无线通信协议，支持无连接的数据收发。ESP32-S3-USB-Bridge 利用 ESP-NOW 实现无线烧录和无线串口通信。

### 11.2 无线模式架构

```
    有线端                                    无线端
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  PC ─USB─► [无线主机] ═══════ ESP-NOW ═══════ [无线从机] ─UART─► 目标 MCU
│                                                          │
│  主机功能:                    从机功能:                    │
│  - 接收 PC 串口数据            - 接收主机数据               │
│  - 通过 ESP-NOW 转发           - 通过 UART 转发到目标 MCU   │
│  - 接收 ESP-NOW 回传           - 接收目标 MCU UART 数据     │
│  - 回传给 PC                   - 通过 ESP-NOW 回传主机       │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### 11.3 主从机绑定与解绑

#### 绑定流程

1. 未绑定时，**指示灯闪烁**
2. 长按无线主从开发板 Boot 按键开始绑定
3. 指示灯呼吸，绑定过程持续 **10 秒**
4. 绑定成功，**指示灯常亮**

#### 解绑流程

1. 长按 Boot 按键 **2 秒**
2. 指示灯恢复闪烁

### 11.4 注意事项

> 数据传输可靠性与 Wi-Fi 环境是否良好有关。在 Wi-Fi 干扰严重的环境中，无线模式的稳定性可能受影响。

---

## 12. 应用场景与应用示例

### 12.1 典型应用场景

#### 12.1.1 替代 USB 转 UART 芯片

最常见的应用场景，用 ESP32-S3-USB-Bridge 替代 CP210x、FT232、CH340 等传统 USB 转串口芯片：

```
PC ──USB──► ESP32-S3-USB-Bridge ──UART──► 目标 ESP 芯片
```

#### 12.1.2 嵌入式开发调试

同时使用串口通信和 JTAG 调试功能：

```
PC ──USB──► ESP32-S3-USB-Bridge ─┬──UART──► 目标 ESP 芯片 (串口通信)
                                  └──JTAG──► 目标 ESP 芯片 (在线调试)
```

#### 12.1.3 无线固件烧录

在无法直接接触设备的场景中，通过无线方式烧录固件：

```
PC ──USB──► [主机 Bridge] ══无线══ [从机 Bridge] ──UART──► 目标设备
```

#### 12.1.4 批量生产烧录

利用 UF2 拖拽烧录功能，无需安装任何软件即可批量烧录：

```
PC ──拖拽 UF2 文件──► ESP32-S3-USB-Bridge (U盘) ──自动烧录──► 目标 MCU
```

#### 12.1.5 远程串口监控

通过无线模式远程监控目标设备的串口输出：

```
监控 PC ──USB──► [主机 Bridge] ══无线══ [从机 Bridge] ──UART──► 远程设备
```

### 12.2 应用示例

| 示例 | 描述 | 链接 |
| --- | --- | --- |
| **usb_wireless_bridge** | 完整的 USB 无线桥接固件 | [GitHub](https://github.com/espressif/esp-dev-kits/tree/master/examples/esp32-s3-usb-bridge/examples/usb_wireless_bridge) |
| **ESP USB Bridge** | 基础 USB 桥接项目 | [GitHub](https://github.com/espressif/esp-usb-bridge) |
| **USB to UART Bridge** | USB 转串口桥接工具 | [GitHub](https://github.com/espressif/esp-iot-solution/blob/master/examples/usb/device/usb_uart_bridge/README.md) |

### 12.3 USB 转 UART Bridge 默认引脚配置

使用 `ESP32-S3-OTG` 开发板作为 USB UART Bridge 时的引脚配置（兼容参考）：

| ESP32-Sx IO | 功能 | 说明 |
| --- | --- | --- |
| GPIO20 | USB D+ | 固定 |
| GPIO19 | USB D- | 固定 |
| GPIO46 | UART-RX | 连接目标板 TX |
| GPIO45 | UART-TX | 连接目标板 RX |
| GPIO26 | Boot | 连接目标板 Boot |
| GPIO3 | EN (RST) | 连接目标板 EN(RST) |

> **注意**：以上为 ESP32-S3-OTG 开发板的默认配置。ESP32-S3-USB-Bridge 的引脚配置请参考第 6 节。

### 12.4 使用 esptool 烧录目标芯片

通过 ESP32-S3-USB-Bridge 创建的虚拟串口，可以像使用普通串口一样烧录目标芯片：

```bash
cd YOUR_ESP_PROJECT

# 编译
idf.py build

# 通过 Bridge 创建的虚拟串口烧录
idf.py -p /dev/ttyACMx flash monitor
```

> 将 `/dev/ttyACMx` 替换为 Bridge 创建的虚拟串口名称（Windows 为 `COMx`）。

### 12.5 使用 openocd-esp32 进行 JTAG 调试

通过 ESP32-S3-USB-Bridge 的 JTAG 功能调试目标芯片：

```bash
# 启动 openocd
openocd -f interface/esp_usb_bridge.cfg -c "esp_usb_bridge_serial <SERIAL>" -f target/esp32s3.cfg

# 另一终端启动 GDB
xtensa-esp32s3-elf-gdb your_app.elf
(gdb) target remote :3333
(gdb) monitor reset halt
(gdb) load
(gdb) continue
```

---

## 13. 硬件版本历史

### 13.1 版本演进

| 版本 | 说明 |
| --- | --- |
| **Version 2.1**（当前） | 将 ESP32-S3 芯片替换为 ESP32-S3-MINI-1-N4R2 模组，减少元器件，优化射频性能 |
| **Version 1.0** | 使用 ESP32-S3 芯片，体积小巧，支持陶瓷天线和 IPEX 天线 |

### 13.2 开源硬件

ESP32-S3-USB-Bridge 为开源硬件项目，原理图和 PCB 文件已公开：

- 开源链接：https://github.com/espressif/esp-dev-kits
- 立创开源硬件平台：https://oshwhub.com/esp-college/esp32s3_usb_flash_tool

---

## 14. 相关文档与资源

### 14.1 官方文档

| 文档 | 链接 |
| --- | --- |
| ESP32-S3-USB-Bridge 用户指南（中文） | https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32s3/esp32-s3-usb-bridge/user_guide.html |
| ESP32-S3-USB-Bridge 用户指南（英文） | https://docs.espressif.com/projects/esp-dev-kits/en/latest/esp32s3/esp32-s3-usb-bridge/user_guide.html |
| ESP32-S3-USB-Bridge 索引页 | https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32s3/esp32-s3-usb-bridge/index.html |
| ESP-IDF 快速入门 | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/get-started/index.html |
| 与 ESP32-S3 创建串口连接 | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/get-started/establish-serial-connection.html |

### 14.2 源码仓库

| 仓库 | 链接 |
| --- | --- |
| ESP-Dev-Kits（开发板示例） | https://github.com/espressif/esp-dev-kits/tree/master/examples/esp32-s3-usb-bridge |
| ESP USB Bridge（基础项目） | https://github.com/espressif/esp-usb-bridge |
| ESP-IoT-Solution（USB UART Bridge） | https://github.com/espressif/esp-iot-solution/blob/master/examples/usb/device/usb_uart_bridge/README.md |

### 14.3 相关工具

| 工具 | 说明 | 链接 |
| --- | --- | --- |
| esptool | ESP 固件烧录工具 | https://github.com/espressif/esptool |
| openocd-esp32 | ESP JTAG 调试工具 | https://github.com/espressif/openocd-esp32 |
| ESP Launchpad | 在线固件烧录 | https://espressif.github.io/esp-launchpad/ |
| esp-tinyuf2 | UF2 固件支持 | https://github.com/espressif/esp-tinyuf2 |

### 14.4 获取样品

- 样品获取：https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32s3/esp32-s3-usb-bridge/user_guide.html#id18

### 14.5 社区支持

| 渠道 | 链接 |
| --- | --- |
| ESP32 论坛（技术问题） | https://esp32.com/ |
| GitHub Issues（功能请求/Bug 报告） | https://github.com/espressif/esp-dev-kits/issues |
| 乐鑫新闻 - ESP USB Bridge 介绍 | https://www.espressif.com/zh-hans/news/ESP_USB_Bridge |

### 14.6 USB PID 注册

如需为自定义开发板注册 USB 产品 ID，请访问：
- Espressif USB Vendor PID 仓库：https://github.com/espressif/usb-pids

---

## 附录 A：常见问题

### Q1: ESP32-S3-USB-Bridge 支持哪些目标芯片？

A: 支持各种乐鑫 ESP 系列芯片，包括 ESP32、ESP32-S2、ESP32-S3、ESP32-C3 等。只要目标芯片支持 UART 下载或 JTAG 调试，即可使用本开发板。

### Q2: 无线桥接的距离有多远？

A: 无线桥接基于 ESP-NOW，在空旷环境下通信距离可达约 200 米。实际距离取决于 Wi-Fi 环境和物理障碍。

### Q3: 能否同时使用串口和 JTAG 功能？

A: 可以。有线模式下，USB 复合设备同时提供 CDC 串口和 JTAG 适配器功能，两者可以并行使用。

### Q4: MSC 拖拽烧录支持哪些固件格式？

A: 支持 UF2（USB Flashing Format）格式。需要将标准的 bin 固件转换为 UF2 格式。

### Q5: 如何查看 Bridge 创建的虚拟串口号？

A:
- **Windows**：设备管理器中查看 COM 端口列表
- **Linux**：`ls /dev/ttyACM*`
- **macOS**：`ls /dev/cu.*`

### Q6: 自动下载功能不工作怎么办？

A: 某些开发板由于 RC 电路造成的延迟，自动下载功能可能无法正常工作。可以尝试：
1. 在 `menuconfig` 中禁用再启用自动下载
2. 在代码中调整 `gpio_set_level` 的软件延时
3. 手动进入下载模式

---

> **备注**：本文档基于 Espressif 官方文档整理编写，如有差异请以官方文档为准。
