# ESP32-S3-USB-OTG 用户指南

> **文档来源**: Espressif 官方文档 (docs.espressif.com) | **语言**: 中文 | **更新日期**: 2026-06-14

---

## 目录

- [1. 开发板概述](#1-开发板概述)
- [2. 核心特性](#2-核心特性)
- [3. USB OTG 功能详解](#3-usb-otg-功能详解)
- [4. 硬件组件介绍](#4-硬件组件介绍)
- [5. 功能框图](#5-功能框图)
- [6. GPIO 分配](#6-gpio-分配)
- [7. 供电系统](#7-供电系统)
- [8. USB 接口选择电路](#8-usb-接口选择电路)
- [9. LCD 显示屏](#9-lcd-显示屏)
- [10. SD 卡接口](#10-sd-卡接口)
- [11. 电池与充电](#11-电池与充电)
- [12. 快速入门](#12-快速入门)
- [13. 应用程序开发](#13-应用程序开发)
- [14. 应用示例与应用场景](#14-应用示例与应用场景)
- [15. 相关文档与资源](#15-相关文档与资源)

---

## 1. 开发板概述

ESP32-S3-USB-OTG 是乐鑫（Espressif）推出的一款**侧重于 USB-OTG 功能验证和应用开发**的开发板。该开发板基于 **ESP32-S3 SoC**，支持 Wi-Fi 和 BLE 5.0 无线功能，同时支持 **USB 主机（Host）** 和 **USB 从机（Device）** 双重角色。

ESP32-S3-USB-OTG 可广泛用于开发以下应用：

- 无线存储设备（USB 无线 U 盘）
- Wi-Fi 网卡 / USB Dongle
- LTE MiFi 设备
- 多媒体设备（USB 摄像头、USB 音频设备）
- 虚拟键盘和鼠标（HID 设备）
- USB 扩展屏

该开发板结合 ESP32-S3 提供的 Wi-Fi 功能，USB 接口可用于 Wi-Fi 视频流传输、通过 4G 热点接入互联网、连接无线 USB 磁盘等多种应用场景。

### 产品基本信息

| 参数 | 说明 |
| --- | --- |
| **产品型号** | ESP32-S3-USB-OTG |
| **状态** | 量产（Mass Production） |
| **工作温度** | -40 ~ 85 °C |
| **Flash 容量** | 8 MB |
| **板载模组** | ESP32-S3-MINI-1-N8 |
| **开发板尺寸** | 65 mm × 32 mm |
| **接口** | LCD 屏、LED、按键、USB 设备、USB 主机、SD 卡 |
| **参考价格** | 169.0 CNY |

---

## 2. 核心特性

ESP32-S3-USB-OTG 开发板具有以下核心特性：

### 2.1 硬件特性一览

| 特性 | 详细说明 |
| --- | --- |
| **板载模组** | ESP32-S3-MINI-1-N8 模组，内置 **8 MB flash** |
| **USB 接口** | 板载 USB Type-A **主机**和**从机**接口，内置 USB 接口切换电路 |
| **调试接口** | 板载 USB 转串口调试芯片（Micro USB 接口） |
| **LCD 显示屏** | 板载 **1.3 英寸 LCD 彩屏**（ST7789，240×240 分辨率），支持 GUI |
| **SD 卡接口** | 板载 Micro SD 卡槽，兼容 **SDIO** 和 **SPI** 接口 |
| **电池支持** | 板载充电 IC，可外接**锂电池**供电 |
| **按键** | MENU 按键、OK(Boot) 按键、UP+ 按键、DW- 按键、Reset 按键 |
| **LED 指示灯** | 绿色 LED（GPIO15）、黄色 LED（GPIO16）、充电指示灯 |
| **无线功能** | Wi-Fi 2.4 GHz + Bluetooth 5 (LE) |

### 2.2 USB OTG 能力

| 能力 | 支持状态 | 说明 |
| --- | --- | --- |
| **USB Host（主机）** | 支持 | 可连接 USB 键盘、鼠标、U 盘、摄像头等外设 |
| **USB Device（设备）** | 支持 | 可作为 HID 设备、MSC 设备、CDC 设备等 |
| **USB 速度** | Full Speed | 总线传输速率 12 Mbps |
| **接口切换** | 硬件切换电路 | 通过 GPIO18 (USB_SEL) 动态切换主机/从机模式 |

### 2.3 BSP 能力依赖表

| 能力 | 可用 | 控制器/编解码器 | 组件 | IDF 版本 |
| --- | --- | --- | --- | --- |
| **DISPLAY** | 支持 | ST7789 | idf | >= 5.2 |
| **LVGL_PORT** | 支持 | - | espressif/esp_lvgl_port | ^2 |
| **TOUCH** | 不支持 | - | - | - |
| **BUTTONS** | 支持 | - | espressif/button | ^4 |
| **AUDIO** | 不支持 | - | - | - |
| **SDCARD** | 支持 | - | idf | >= 5.2 |
| **IMU** | 不支持 | - | - | - |
| **LED** | 支持 | - | idf | >= 5.2 |
| **BAT** | 支持 | - | idf | >= 5.2 |

---

## 3. USB OTG 功能详解

### 3.1 USB-OTG 外设概述

ESP32-S3 芯片内置 **USB-OTG 外设**，包含 USB 控制器和 USB PHY，支持通过 USB 线连接到 PC 或其他 USB 设备，实现 USB Host 和 USB Device 双重功能。

### 3.2 USB-OTG 传输速率

ESP32-S3 USB-OTG 总线传输速率为 **Full Speed（12 Mbps）**，实际有效传输速率取决于传输类型：

| 传输类型 | 控制传输 | 中断传输 | 批量传输 | 同步传输 |
| --- | --- | --- | --- | --- |
| **适用场景** | 设备初始化和管理 | 鼠标和键盘 | 打印机和批量存储 | 流式音频和视频 |
| **校验重传** | 有 | 有 | 有 | 无 |
| **最大传输尺寸** | 64 字节 | 64 字节 | 64 字节 | ~512 字节 |
| **理论有效速率** | - | 64000 Bytes/s | 1216000 Bytes/s | 512000 Bytes/s |

### 3.3 USB Host 功能

作为 USB 主机，ESP32-S3-USB-OTG 可以：
- 连接 USB HID 设备（键盘、鼠标、游戏手柄）
- 读写 USB 大容量存储设备（U 盘、SD 卡读卡器）
- 连接 USB 摄像头（UVC）
- 连接 USB 音频设备（UAC）
- 连接 USB 转串口设备（CDC-ACM、CH34x、CP210x、PL2303、FTDI）

### 3.4 USB Device 功能

作为 USB 设备，ESP32-S3-USB-OTG 可以模拟为：
- **HID 设备**：虚拟键盘、鼠标、游戏手柄
- **MSC 存储设备**：无线 U 盘、读卡器
- **CDC 串口设备**：虚拟串口
- **UAC 音频设备**：USB 麦克风、USB 扬声器
- **UVC 视频设备**：USB 摄像头
- **复合设备**：多种功能组合

### 3.5 ROM 内置 USB 功能

ESP32-S3 的 ROM Code 中内置以下 USB 功能：

- **USB OTG Console**：通过 USB CDC 替代 UART，实现 Log 打印、Console 交互和固件下载
- **USB DFU**：标准的 Device Firmware Upgrade 模式，用于固件烧录

---

## 4. 硬件组件介绍

ESP32-S3-USB-OTG 开发板包括**主板**和**子板**两部分：

### 4.1 主板正面组件

以 USB_HOST 接口为起点，逆时针顺序的主要组件：

| 主要组件 | 描述 |
| --- | --- |
| **USB_HOST 接口** | USB Type-A 母口，用来连接其它 USB 设备（ESP32-S3 作为主机） |
| **ESP32-S3-MINI-1 模组** | 通用型 Wi-Fi + 低功耗蓝牙 MCU 模组，具有丰富的外设接口、强大的神经网络运算能力和信号处理能力，专为 AIoT 市场打造。采用 PCB 板载天线，与 ESP32-S2-MINI-1 pin-to-pin 兼容 |
| **MENU 按键** | 菜单按键（GPIO14） |
| **Micro SD 卡槽** | 可插入 Micro SD 卡，支持 4-线 SDIO 和 SPI 模式 |
| **USB Switch IC** | USB 接口切换芯片，通过 USB_SEL (GPIO18) 电平切换 USB 连接到 USB_DEV 或 USB_HOST 接口 |
| **Reset 按钮** | 用于重启系统 |
| **USB_DEV 接口** | USB Type-A 公口，可连接其它 USB 主机（ESP32-S3 作为设备），也作为锂电池充电接口 |
| **电池供电开关** | 拨向 ON：使用电池供电；拨向 GND：通过其它方式供电 |
| **Boot 按键** | 按住 Boot + 按一下 Reset 进入固件下载模式；正常使用中作为确认（OK）按钮（GPIO0） |
| **DW- 按键** | 向下按键（GPIO11） |
| **屏幕排座** | 用于连接 1.3 英寸 LCD 屏 |
| **UP+ 按键** | 向上按键（GPIO10） |
| **USB 转 UART 接口** | Micro-USB 接口，可用作供电、烧录固件和通信接口 |

### 4.2 主板背面组件

| 主要组件 | 描述 |
| --- | --- |
| **黄色指示灯** | 设置 GPIO16 为高电平，指示灯亮 |
| **绿色指示灯** | 设置 GPIO15 为高电平，指示灯亮 |
| **充电指示灯** | 为电池充电时亮红灯，充电完成红灯熄灭 |
| **电池焊点** | 可焊接 3.6 V 锂电池，为主板供电 |
| **充电电路** | 用于为锂电池充电 |
| **空闲管脚** | 可自定义的空闲管脚（GPIO45, GPIO46, GPIO48, GPIO26, GPIO47, GPIO3） |
| **USB 转 UART 桥接器** | 单芯片 USB 至 UART 桥接器，提供高达 3 Mbps 传输速率 |

### 4.3 子板

- **ESP32-S3-USB-OTG-SUB**：贴装 1.3 英寸 LCD 彩屏的子板，通过排线连接到主板

---

## 5. 功能框图

ESP32-S3-USB-OTG 的主要组件和连接方式如下：

```
                    ┌─────────────────────────────────────────┐
                    │              ESP32-S3-MINI-1             │
                    │                                         │
   USB DEV ────────┤  USB D+ (GPIO20)   USB D- (GPIO19)      ├──────── USB HOST
  (Type-A 公口)     │          │                  │           │  (Type-A 母口)
                    │     ┌────┴────────────────────┴────┐     │
                    │     │     USB Switch IC            │     │
                    │     │   (USB_SEL = GPIO18)         │     │
                    │     └──────────────────────────────┘     │
                    │                                         │
                    │  SPI2 ──── LCD (ST7789, 240×240)        │
                    │  SPI3/SDIO ──── Micro SD Card            │
                    │  ADC1_CH0 (GPIO1) ── HOST_VOL            │
                    │  ADC1_CH1 (GPIO2) ── BAT_VOL             │
                    │  GPIO15 ── LED_GREEN                     │
                    │  GPIO16 ── LED_YELLOW                    │
                    │  GPIO13 ── BOOST_EN                      │
                    │  GPIO12 ── DEV_VBUS_EN                   │
                    │  GPIO17 ── LIMIT_EN                      │
                    └─────────────────────────────────────────┘
```

> **注意**：功能框图中的 `USB_HOST D+ D-` 信号对应的外部接口是 `USB DEV`，指 ESP32-S3 作为设备接收其他 USB 主机的信号。`USB_DEV D+ D-` 信号对应的外部接口是 `USB HOST`，指 ESP32-S3 作为主机控制其他设备。

---

## 6. GPIO 分配

### 6.1 功能引脚

| 编号 | ESP32-S3-MINI-1 管脚 | 功能说明 |
| --- | --- | --- |
| 1 | GPIO18 | **USB_SEL**：切换 USB 接口，高电平使能 USB_HOST，低电平（默认）使能 USB_DEV |
| 2 | GPIO19 | USB_D- 信号 |
| 3 | GPIO20 | USB_D+ 信号 |
| 4 | GPIO15 | **LED_GREEN**：控制绿色 LED，高电平亮 |
| 5 | GPIO16 | **LED_YELLOW**：控制黄色 LED，高电平亮 |
| 6 | GPIO0 | **BUTTON_OK**：OK/Boot 按键，按下为低电平 |
| 7 | GPIO11 | **BUTTON_DW**：Down 按键，按下为低电平 |
| 8 | GPIO10 | **BUTTON_UP**：UP 按键，按下为低电平 |
| 9 | GPIO14 | **BUTTON_MENU**：MENU 按键，按下为低电平 |
| 10 | GPIO8 | **LCD_RET**：复位 LCD，低电平复位 |
| 11 | GPIO5 | **LCD_EN**：使能 LCD，低电平使能 |
| 12 | GPIO4 | **LCD_DC**：切换数据和命令状态 |
| 13 | GPIO6 | **LCD_SCLK**：LCD SPI 时钟信号 |
| 14 | GPIO7 | **LCD_SDA**：LCD SPI MOSI 信号 |
| 15 | GPIO9 | **LCD_BL**：LCD 背光控制信号 |
| 16 | GPIO36 | **SD_SCK**：SD SPI CLK / SDIO CLK |
| 17 | GPIO37 | **SD_DO**：SD SPI MISO / SDIO Data0 |
| 18 | GPIO38 | **SD_D1**：SDIO Data1 |
| 19 | GPIO33 | **SD_D2**：SDIO Data2 |
| 20 | GPIO34 | **SD_D3**：SD SPI CS / SDIO Data3 |
| 21 | GPIO1 | **HOST_VOL**：USB_DEV 电压监测（ADC1 通道 0） |
| 22 | GPIO2 | **BAT_VOL**：电池电压监测（ADC1 通道 1） |
| 23 | GPIO17 | **LIMIT_EN**：使能限流芯片，高电平使能 |
| 24 | GPIO21 | **OVER_CURRENT**：电流超限信号，高电平代表超限 |
| 25 | GPIO12 | **DEV_VBUS_EN**：高电平选择 DEV_VBUS 电源 |
| 26 | GPIO13 | **BOOST_EN**：高电平使能 Boost 升压电路 |

### 6.2 扩展功能引脚（空闲管脚）

| 编号 | GPIO | 说明 |
| --- | --- | --- |
| 1 | GPIO45 | FREE_1：空闲，可自定义 |
| 2 | GPIO46 | FREE_2：空闲，可自定义（Strapping 引脚） |
| 3 | GPIO48 | FREE_3：空闲，可自定义 |
| 4 | GPIO26 | FREE_4：空闲，可自定义 |
| 5 | GPIO47 | FREE_5：空闲，可自定义 |
| 6 | GPIO3 | FREE_6：空闲，可自定义 |

### 6.3 LCD SPI 接口配置

| 参数 | 值 |
| --- | --- |
| SPI 主机 | SPI2_HOST |
| SPI 时钟频率 | 40 MHz |
| MISO | 无（-1） |
| MOSI | GPIO7 |
| CLK | GPIO6 |
| CS | GPIO5 |
| DC | GPIO4 |
| RESET | GPIO8 |
| 背光 | GPIO9 |
| 屏幕控制器 | ST7789 |
| 分辨率 | 240 × 240 |

### 6.4 SD 卡接口配置

| 模式 | 引脚 |
| --- | --- |
| SPI 模式 | SCK=GPIO36, MOSI(CMD)=GPIO35, MISO(D0)=GPIO37, CS=GPIO34 |
| SDIO 4-线模式 | CLK=GPIO36, CMD=GPIO35, D0=GPIO37, D1=GPIO38, D2=GPIO33, D3=GPIO34 |

---

## 7. 供电系统

### 7.1 三种供电方式

ESP32-S3-USB-OTG 开发板支持以下三种供电方式：

#### 方式 1：通过 Micro_USB 接口供电

- 使用 USB 电缆（标准 A 转 Micro-B）连接主板至供电设备
- 将电源开关置于 **OFF**
- **注意**：该供电模式仅为主板和显示屏供电

#### 方式 2：通过 USB_DEV 接口供电

- 将 `DEV_VBUS_EN`（GPIO12）设置为高电平
- 将电源开关设置为 **OFF**
- 该模式可同时向 `USB HOST` 接口供电
- 如已安装锂电池，会同时进行充电

#### 方式 3：通过锂电池供电

- 将 `BOOST_EN`（GPIO13）设置为高电平
- 将电源开关设置为 **ON**
- 将 1S 锂电池（3.7 V ~ 4.2 V）焊接于主板背面预留的电源焊点
- 该模式可同时向 `USB HOST` 接口供电

### 7.2 USB HOST 接口供电

`USB HOST` 接口（Type-A 母口）可向已连接的 USB 设备供电：

- **供电电压**：5 V
- **最大电流**：500 mA
- **供电电源选择**：

| BOOST_EN | DEV_VBUS_EN | 电源来源 |
| --- | --- | --- |
| 0 | 1 | USB_DEV 接口 |
| 1 | 0 | 锂电池（经 Boost 升压） |
| 0 | 0 | 无输出 |
| 1 | 1 | 未定义 |

### 7.3 500 mA 限流电路

限流 IC **MIC2005A** 可将 USB HOST 接口最大输出电流限制为 500 mA。必须设置 `LIMIT_EN`（GPIO17）为高电平，使能限流 IC，USB HOST 接口才有电压输出。

---

## 8. USB 接口选择电路

开发板通过硬件切换电路实现 USB 主机/从机模式的切换：

| USB_SEL (GPIO18) | 电平 | USB D+/D- 连接到 | 可用的外部接口 | ESP32-S3 角色 |
| --- | --- | --- | --- | --- |
| 低电平（默认） | 0 | USB_HOST D+ D- | USB DEV（Type-A 公口） | USB 设备 |
| 高电平 | 1 | USB_DEV D+ D- | USB HOST（Type-A 母口） | USB 主机 |

> **注意**：默认 `USB_SEL` 为低电平，即默认 ESP32-S3 作为 USB 设备工作。

---

## 9. LCD 显示屏

### 9.1 基本信息

- **屏幕型号**：ST7789
- **屏幕尺寸**：1.3 英寸
- **分辨率**：240 × 240
- **接口类型**：SPI
- **SPI 时钟频率**：40 MHz
- **背光控制**：GPIO9（LCD_BL）

### 9.2 LVGL 支持

开发板支持通过 LVGL（Light and Versatile Graphics Library）进行 GUI 开发。BSP 提供了 `esp_lvgl_port` 组件，简化 LVGL 的移植和使用。

### 9.3 屏幕控制 API

```c
/* 使能 LCD 功能 */
bsp_feature_enable(BSP_FEATURE_LCD, true);

/* 设置 LCD 背光 */
bsp_display_brightness_set(100); // 0-100%
```

---

## 10. SD 卡接口

### 10.1 基本信息

- **兼容模式**：1-线 SDIO、4-线 SDIO、SPI
- **接口电压**：3.3 V signaling
- **文件系统**：支持 FAT32 等标准文件系统

### 10.2 模式选择

上电后，SD 卡处于 3.3 V signaling 模式。发送第一个 CMD0 命令选择总线模式：SD 模式或 SPI 模式。

### 10.3 挂载路径

- BSP 默认挂载路径：`/sdcard`
- 最大打开文件数：9
- 磁盘块大小：512 字节

---

## 11. 电池与充电

### 11.1 电池规格

- **电池类型**：1S 锂电池
- **电压范围**：3.6 V ~ 4.2 V
- **安装方式**：焊接于主板背面预留焊点

### 11.2 充电功能

- 通过 USB_DEV 接口充电
- 充电指示灯：充电时亮红灯，充电完成红灯熄灭
- 电压监测：通过 GPIO2（ADC1_CH1）进行电池电压监测

### 11.3 Boost 升压电路

锂电池 3.6 V ~ 4.2 V 电压可通过 Boost 电路升压到 5 V，为 USB HOST 接口供电。Boost IC 由 `BOOST_EN`（GPIO13）控制。

---

## 12. 快速入门

### 12.1 硬件准备

- ESP32-S3-USB-OTG 开发板（主板 + 子板）
- 一根 USB 2.0 数据线（标准 A 转 Micro-B）
- 电脑（Windows、Linux 或 macOS）

### 12.2 软件设置

请前往 [ESP-IDF 快速入门](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/get-started/index.html) 查看 [详细安装步骤](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/get-started/index.html#get-started-step-by-step)，完成开发环境的搭建。

### 12.3 上电检查

1. 确认开发板完好无损
2. 将 LCD 子板安装到主板屏幕排座上
3. 使用 USB 数据线连接 Micro-USB 接口与电脑
4. 开发板上电后，屏幕应正常显示

### 12.4 进入下载模式

1. 按住 Boot 按键（GPIO0）不放
2. 按一下 Reset 按键
3. 松开 Boot 按键
4. 开发板进入固件下载模式

---

## 13. 应用程序开发

### 13.1 编译和烧录

```bash
# 设置 ESP-IDF 环境变量
. $HOME/esp/esp-idf/export.sh

# 设置目标芯片
idf.py set-target esp32s3

# 配置工程选项（可选）
idf.py menuconfig

# 编译、烧录并监控
idf.py -p PORT flash monitor
```

> 将 `PORT` 替换为实际的串口名称（如 Windows 的 `COM3`，Linux 的 `/dev/ttyUSB0`）

### 13.2 通过 USB OTG 烧录

ESP32-S3 支持通过 USB-OTG 接口直接烧录固件，无需外部 USB 转 UART 芯片：

- USB D+ 使用 **GPIO20**
- USB D- 使用 **GPIO19**

首次烧录需要手动进入下载模式（按住 Boot + Reset），之后可通过 USB 自动进入下载模式。

### 13.3 使用 BSP（Board Support Package）

通过 ESP Component Registry 获取 BSP：

```yaml
# idf_component.yml
dependencies:
  espressif/esp32_s3_usb_otg: "^1.0.0"
```

BSP 提供的核心 API：

```c
/* 初始化 I2C 总线 */
bsp_i2c_init();

/* 初始化 ADC */
bsp_adc_initialize();

/* 使能/禁用硬件功能 */
bsp_feature_enable(BSP_FEATURE_LCD, true);
bsp_feature_enable(BSP_FEATURE_SD, true);
bsp_feature_enable(BSP_FEATURE_BATTERY, true);

/* 获取 I2C 总线句柄 */
i2c_master_bus_handle_t i2c = bsp_i2c_get_handle();

/* 获取 ADC 句柄 */
adc_oneshot_unit_handle_t adc = bsp_adc_get_handle();
```

---

## 14. 应用示例与应用场景

### 14.1 官方应用示例

| 示例名称 | 描述 | 链接 |
| --- | --- | --- |
| **factory** | 工厂演示程序，展示开发板各项功能 | [GitHub](https://github.com/espressif/esp-dev-kits/tree/master/examples/esp32-s3-usb-otg/examples/factory) |
| **Display** | 在屏幕上显示图像，带启动动画（LVGL） | [GitHub](https://github.com/espressif/esp-bsp/tree/master/examples/display) |
| **USB HID Demo** | USB HID 演示（键盘、鼠标、游戏手柄可视化） | [GitHub](https://github.com/espressif/esp-bsp/tree/master/examples/display_usb_hid) |
| **USB Device Demos** | USB 设备模式示例集合 | [GitHub](https://github.com/espressif/esp-iot-solution/tree/master/examples/usb/device) |
| **USB Host Demos** | USB 主机模式示例集合 | [GitHub](https://github.com/espressif/esp-iot-solution/tree/master/examples/usb/host) |

### 14.2 USB Host 应用场景

#### 14.2.1 USB HID 主机（键盘/鼠标）

通过 USB HOST 接口连接 USB 键盘或鼠标，在 LCD 屏幕上显示输入内容。

```
USB 键盘/鼠标 ──→ USB HOST 接口 ──→ ESP32-S3 ──→ LCD 显示
```

#### 14.2.2 USB MSC 主机（U 盘读写）

作为 USB 主机读写 U 盘，结合 Wi-Fi 功能可实现无线文件共享。

#### 14.2.3 USB 摄像头主机

连接 USB UVC 摄像头，通过 Wi-Fi 进行视频流传输。

#### 14.2.4 USB 音频主机

连接 USB 音频设备，实现音频播放和录制。

### 14.3 USB Device 应用场景

#### 14.3.1 USB 无线 U 盘（MSC Device）

| 特性 | 说明 |
| --- | --- |
| 协议标准 | USB Mass Storage Class (MSC) |
| 读写方式 | USB - Wi-Fi 双向访问 |
| 支持多设备接入 | 是 |
| 测试读写速度 | 读 540 KB/s，写 350 KB/s |
| 示例 | [usb_msc_wireless_disk](https://github.com/espressif/esp-iot-solution/tree/master/examples/usb/device/usb_msc_wireless_disk) |

#### 14.3.2 USB HID 设备（虚拟键鼠）

| 特性 | 说明 |
| --- | --- |
| 协议标准 | USB HID |
| 支持设备类型 | 键盘、鼠标、游戏手柄 |
| 双模支持 | USB HID + BLE HID 双模 |
| 自定义 | 支持自定义 HID 描述符 |
| 示例 | [usb_hid_device](https://github.com/espressif/esp-iot-solution/tree/master/examples/usb/device/usb_hid_device) |

#### 14.3.3 USB 音频设备（UAC）

| 特性 | 说明 |
| --- | --- |
| 协议标准 | UAC 2.0 |
| 支持格式 | 多种音频格式和采样率 |
| 方向 | 支持音频输入和输出 |
| 示例 | [usb_uac](https://github.com/espressif/esp-iot-solution/tree/master/examples/usb/device/usb_uac) |

#### 14.3.4 USB 视频设备（UVC）

| 特性 | 说明 |
| --- | --- |
| 协议标准 | UVC 1.5 |
| 传输模式 | 同步和批量 |
| 应用 | USB 门铃摄像头、USB + Wi-Fi 双模网络摄像头 |
| 示例 | [usb_webcam](https://github.com/espressif/esp-iot-solution/tree/master/examples/usb/device/usb_webcam) |

#### 14.3.5 UF2 拖拽升级

| 特性 | 说明 |
| --- | --- |
| 功能 | 拖拽 UF2 固件到虚拟 U 盘实现 OTA |
| NVS 支持 | 支持将 NVS 数据映射到 U 盘文件 |
| 示例 | [usb_uf2_ota](https://github.com/espressif/esp-iot-solution/tree/master/examples/usb/device/usb_uf2_ota) |

#### 14.3.6 USB 扩展屏

| 特性 | 说明 |
| --- | --- |
| 功能 | 通过 USB 总线作为扩展副屏 |
| 支持数据 | 图像、音频、触摸信息 |
| 平台 | Windows（使用 IDD 驱动） |
| 示例 | [usb_extend_screen](https://github.com/espressif/esp-iot-solution/tree/master/examples/usb/device/usb_extend_screen) |

### 14.4 USB-OTG 模式动态切换

USB-OTG 外设支持动态切换模式，通过动态注册 USB Host Driver 或 USB Device Driver 实现：

- 示例代码：[usb_host_device_mode_manual_switch](https://github.com/espressif/esp-iot-solution/tree/master/examples/usb/otg/usb_host_device_mode_manual_switch)

---

## 15. 相关文档与资源

### 15.1 官方文档

| 文档 | 链接 |
| --- | --- |
| ESP32-S3-USB-OTG 用户指南（中文） | https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32s3/esp32-s3-usb-otg/user_guide.html |
| ESP32-S3-USB-OTG 用户指南（英文） | https://docs.espressif.com/projects/esp-dev-kits/en/latest/esp32s3/esp32-s3-usb-otg/user_guide.html |
| ESP-IDF 快速入门 | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/get-started/index.html |
| ESP-IDF USB Host 驱动 | https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/api-reference/peripherals/usb_host.html |

### 15.2 源码仓库

| 仓库 | 链接 |
| --- | --- |
| ESP-Dev-Kits（开发板示例） | https://github.com/espressif/esp-dev-kits/tree/master/examples/esp32-s3-usb-otg |
| ESP-BSP（板级支持包） | https://github.com/espressif/esp-bsp/blob/master/bsp/esp32_s3_usb_otg/README.md |
| ESP-IoT-Solution（USB 方案） | https://github.com/espressif/esp-iot-solution |
| ESP Component Registry | https://components.espressif.com/components/espressif/esp32_s3_usb_otg |

### 15.3 数据手册

| 文档 | 链接 |
| --- | --- |
| ESP32-S3 技术规格书 | https://www.espressif.com/sites/default/files/documentation/esp32-s3_datasheet_cn.pdf |
| ESP32-S3-MINI-1 数据手册 | https://www.espressif.com/sites/default/files/documentation/esp32-s3-mini-1_mini-1u_datasheet_en.pdf |
| ST7789 屏幕芯片手册 | https://dl.espressif.com/AE/esp-dev-kits/ST7789VW芯片手册.pdf |

### 15.4 购买渠道

- 零售订单：https://www.espressif.com/zh-hans/company/contact/buy-a-sample
- 批量订单：https://www.espressif.com/zh-hans/contact-us/sales-questions
- AliExpress / Digi-Key / Mouser / Amazon US

### 15.5 包装清单

零售 ESP32-S3-USB-OTG 开发套件包含：

- 主板：ESP32-S3-USB-OTG
- 子板：ESP32-S3-USB-OTG_SUB（带 LCD 屏）
- 紧固件：安装螺栓（x4）、螺丝（x4）、螺母（x4）

### 15.6 社区支持

- ESP32 论坛：https://esp32.com/
- GitHub Issues：https://github.com/espressif/esp-dev-kits/issues

---

> **备注**：本文档基于 Espressif 官方文档整理编写，如有差异请以官方文档为准。
