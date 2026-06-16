# ESP-DualKey 用户指南 — 智能双键控制器开发板

> **文档来源**：基于乐鑫官方 esp-dev-kits 文档、ESP-DualKey 用户指南、BSP 头文件 `esp_dualkey.h` 及 M5Stack Chain DualKey 文档整理。
> **文档编号**：28

---

## 目录

- [1. 产品概述](#1-产品概述)
- [2. 硬件架构与核心规格](#2-硬件架构与核心规格)
- [3. 三种工作模式](#3-三种工作模式)
- [4. 板载组件详解](#4-板载组件详解)
- [5. GPIO 引脚分配](#5-gpio-引脚分配)
- [6. 电源管理与低功耗](#6-电源管理与低功耗)
- [7. 出厂固件使用](#7-出厂固件使用)
- [8. 软件开发](#8-软件开发)
- [9. 快速入门](#9-快速入门)
- [10. 应用场景](#10-应用场景)
- [11. 相关资源与链接](#11-相关资源与链接)

---

## 1. 产品概述

ESP-DualKey 是乐鑫基于 ESP32-S3 芯片开发的**智能双键控制器开发板**，集成了智能家居控制、蓝牙键盘、USB 键盘等多种功能，可通过物理开关在不同工作模式间切换。该开发板配备双按键、RGB 灯效、电池管理、电源监控等功能，为开发者提供完整的智能按键解决方案。

ESP-DualKey 主控采用乐鑫 ESP32-S3 芯片（封装 ESP32-S3FN8），支持 2.4 GHz Wi-Fi 和 Bluetooth 5 (LE) 无线连接，集成 8 MB flash 存储。配备 2×1 按键矩阵和 WS2812 RGB 灯效，提供直观丰富的交互体验。

### 核心亮点

- **三模切换**：物理拨动开关一键切换 蓝牙键盘 / USB 键盘 / 智能家居（RainMaker）三种模式。
- **双按键矩阵**：支持独立按键检测与组合按键功能。
- **RGB 灯效**：WS2812 可编程 LED，支持热力图、呼吸灯、流水灯等多种灯效。
- **完整电源方案**：内置 350 mAh 锂电池 + TP4057 充电管理 + 电池/VBUS 电压监控 + Deep-sleep 低功耗。
- **可扩展设计**：两个 HY2.0-4P 接口，支持向外供电与外接传感器。

> ESP-DualKey 也是 M5Stack **Chain DualKey（SKU:C147）** 的姊妹产品，二者硬件一致，M5Stack 版本出厂预置 Chain 宏键盘固件。

---

## 2. 硬件架构与核心规格

### 2.1 核心规格

| 参数 | 规格 |
|------|------|
| 主控芯片 | ESP32-S3FN8 |
| 处理器 | 240 MHz Xtensa 32-bit LX7 双核 |
| 无线连接 | 2.4 GHz Wi-Fi (802.11 b/g/n) + Bluetooth 5 (LE) |
| Flash | 8 MB |
| 内置 SRAM | 512 KB |
| USB | 原生 USB Serial/JTAG（GPIO19/20），无需外部转串口芯片 |

### 2.2 存储与交互

- **存储**：8 MB Flash
- **交互**：2 个可热插拔青轴机械按键（2×1 矩阵）+ 2 颗 WS2812B RGB LED（串联）

---

## 3. 三种工作模式

ESP-DualKey 通过一个三档物理拨动开关切换工作模式：

| 开关档位 | 模式 | 说明 |
|----------|------|------|
| **左档位（BLE）** | 蓝牙键盘模式 | 作为蓝牙 HID 设备，低功耗，按键映射同 USB HID |
| **中档位（OFF/USB）** | USB 键盘模式 | 接入电脑即被识别为 USB 键盘，即插即用免配对 |
| **右档位（RainMaker）** | 智能家居模式 | 接入 ESP RainMaker 云平台，可作智能开关/灯泡，支持场景自动化与远程控制 |

> **按键出厂映射示例（USB/BLE 键盘）**：音量控制（默认）、复制粘贴、撤销重做、标签页切换、窗口切换、缩放、翻页、媒体控制、方向键、Home/End 等。

---

## 4. 板载组件详解

### 4.1 主要组件

| 主要组件 | 描述 |
|----------|------|
| **ESP32-S3** | 主控芯片，Wi-Fi + BLE，集成 8 MB flash |
| **双按键** | 2×1 按键矩阵，支持独立按键和组合按键 |
| **模式切换开关** | 三档位，切换 蓝牙键盘 / 智能家居 / USB 键盘 |
| **WS2812 RGB LED** | 可编程 RGB LED，支持多种灯效（热力图、呼吸灯、流水灯等） |
| **TP4057 锂电池充电芯片** | 支持 USB-C 充电，充电电流管理 |
| **USB-C 接口** | 供电、编程下载、调试，支持对锂电池充电 |
| **两个 HY2.0-4P 接口** | 支持向外供电，用于连接外置传感器 |
| **电池电压监控** | 实时监控电池电压和充电状态 |
| **VBUS 监控** | 监控 USB 供电状态 |
| **Deep-sleep** | 支持 Deep-sleep 模式节省功耗 |

### 4.2 结构特性

- M5Stack Chain 系列可拓展设计，两个 HY2.0-4P 拓展接口支持横向级联
- 背部 LEGO 兼容孔设计，便于结构固定
- 挂绳设计

---

## 5. GPIO 引脚分配

以下引脚定义基于乐鑫官方 BSP 头文件 `bsp/esp_dualkey.h`：

| 功能 | GPIO | 说明 |
|------|------|------|
| USB D- | GPIO19 | 原生 USB |
| USB D+ | GPIO20 | 原生 USB |
| 按键 Key2 | GPIO17 | 普通输入，支持中断 |
| 按键 Key1 | GPIO0 | 兼任 Boot/下载模式触发 |
| WS2812 数据 | GPIO21 | RGB LED 串联数据线 |
| WS2812 电源控制 | GPIO40 | MOSFET 控制 LED 供电（深睡眠关断） |
| UART1 TX | GPIO47 | HY2.0-4P Port1 |
| UART1 RX | GPIO48 | HY2.0-4P Port1 |
| UART2 TX | GPIO6 | HY2.0-4P Port2 |
| UART2 RX | GPIO5 | HY2.0-4P Port2 |
| 模式开关 BLE 检测 | GPIO8（ADC1_CH7） | ADC 读取开关档位 |
| 模式开关 RainMaker 检测 | GPIO7（ADC1_CH6） | ADC 读取开关档位 |
| VBUS 监控 | GPIO2（ADC1_CH1） | 检测 USB 电源接入 |
| 电池电压监控 | GPIO10（ADC1_CH9） | 分压电路检测电池电压（比例 1.51） |
| 充电状态 | GPIO9（ADC1_CH8） | 充电中 1.65V / 充满 2.2V / 未充电 3.3V |
| 电源维持 SWITCH_1 | GPIO8 | 电源自锁电路 |
| 电源维持 SWITCH_2 | GPIO7 | 电源自锁电路 |

### ADC 阈值与电压换算

```
ADC_SWITCH_THRESHOLD       = 2000   // 开关档位判定阈值
ADC_CHARGER_FULL_THRESHOLD = 2200   // 充满
ADC_CHARGING_THRESHOLD     = 1650   // 充电中
ADC_CHARGER_NO_THRESHOLD   = 3300   // 未充电
ADC_BATTERY_VOLTAGE_RATIO  = 1.51   // 电池分压比
ADC_BATTERY_VOLTAGE_MIN    = 3.0V   // 电量 0%
ADC_BATTERY_VOLTAGE_MAX    = 4.2V   // 电量 100%
```

> ⚠️ **固件禁忌**：严禁将 SWITCH_1（GPIO8）和 SWITCH_2（GPIO7）配置为高电平输出，否则会触发 PMOS 锁定导通，导致物理开关无法切断电源（无法关机）。

---

## 6. 电源管理与低功耗

### 6.1 供电与充电

| 供电方式 | 说明 |
|----------|------|
| USB-C | 5V 供电，同时充电，支持下载调试 |
| 内置锂电池 | 350 mAh 锂电池 |

- **充电芯片**：TP4057（线性充电管理）
- **电池计量**：通过 ADC（GPIO10）分压采样换算电压
- **充电独立**：只要接入外部 5V 电源，无论开关拨到哪档，电池都会持续充电

### 6.2 低功耗实测

得益于电源开关电路（MOSFET 隔离）与 LDO 选型：

| 模式 | 电流消耗 |
|------|----------|
| 关机模式（Power Off） | ≈ 8.97 µA |
| 深度睡眠（Deep Sleep，RTC 域工作） | ≈ 107.64 µA |

### 6.3 Deep-sleep 配置

固件支持配置空闲超时后自动进入 Light-sleep / Deep-sleep：

```c
// 配置项（idf.py menuconfig）
// CONFIG_LIGHT_SLEEP_TIMEOUT_MS：无按键操作超过该时长进入 Light-sleep
// CONFIG_DEEP_SLEEP_TIMEOUT_S  ：无按键操作超过该时长进入 Deep-sleep

// BSP 提供 Deep-sleep 唤醒配置结构体
typedef struct {
    int32_t sleep_time_ms;
    int32_t wakeup_pin_1;
    int32_t wakeup_pin_2;
} bsp_deep_sleep_config_t;
```

---

## 7. 出厂固件使用

### 7.1 键盘连接

- **有线（USB）**：用 USB-C 数据线连接 DualKey 与电脑/手机，即被识别为键盘。
- **蓝牙**：开关拨至左侧，在主机上搜索蓝牙设备 `ESP_DualKey`（M5 版为 `DualKey-XXXX`）配对连接。支持同时有线 + 蓝牙连接两个设备。

### 7.2 智能家居模式（RainMaker）

1. 开关拨至**右侧**进入 RainMaker 模式。
2. 两颗 LED 交替闪烁表示等待配网/连接。
3. 在 RainMaker App 中选择 `Add Device` → `BLE (no QR code)`，配网时设备名前缀设为 `M_Key`。
4. 配网成功后按键灯效切换为预设模式（默认：热力图模式）。

可控制参数：亮度、色调（Hue）、饱和度、左右按键开关、键盘层（USB/BLE HID 映射）、灯效模式。

### 7.3 配置网页（M5 Chain 固件）

连接外部电源后，连接 Wi-Fi AP `DualKey_XXXX`（密码 `12345678`），浏览器访问 `192.168.4.1` 即可配置按键功能、LED 颜色、Wi-Fi 等。两按键同时长按 5 秒可重置 Wi-Fi。

---

## 8. 软件开发

### 8.1 开发框架

| 框架/SDK | 说明 |
|----------|------|
| ESP-IDF | 底层开发框架（推荐 release/v5.5） |
| ESP RainMaker | 智能家居模式云平台 |
| ESP-HID（esp_hid） | USB/BLE HID 键盘组件 |
| ESP-BSP | 板级支持包，封装按键、LED、电源监控 API |

### 8.2 BSP 关键 API

```c
// 模式开关状态
typedef enum {
    BSP_SWITCH_STATE_MIDDLE = 0,  // USB 键盘模式
    BSP_SWITCH_STATE_LEFT,        // 蓝牙键盘模式
    BSP_SWITCH_STATE_RIGHT,       // 智能家居模式
} bsp_switch_state_t;

// 获取开关档位
esp_err_t bsp_get_switch_state(bsp_switch_state_t *state);

// 电池状态查询
esp_err_t bsp_get_battery_state(bool *is_charging, float *voltage);
esp_err_t bsp_get_vbus_state(bool *is_charging);

// 监控回调（电池 / VBUS / 开关状态变化）
typedef void (*battery_monitor_cb_t)(bool is_charging, float voltage);
typedef void (*vbus_monitor_cb_t)(bool is_charging);
typedef void (*switch_monitor_cb_t)(bsp_switch_state_t state);
```

### 8.3 出厂示例（factory_demo）

`esp-dev-kits/examples/esp-dualkey/examples/factory_demo` 演示了完整的智能按钮控制器功能，集成 RainMaker、BLE/USB HID，支持物理开关切换。

---

## 9. 快速入门

### 9.1 硬件准备

- ESP-DualKey 开发板
- USB 2.0 数据线（标准 A 转 Type-C）
- 电脑（Windows / Linux / macOS）

### 9.2 进入下载模式

- 开发板使用 ESP32-S3 原生 USB。若已烧录为 HID 设备，上传新固件需重新进入下载模式：**断开 USB-C，开关拨到中间，按住 Key1（GPIO0）再连接 USB-C，然后松开 Key1**。

### 9.3 编译烧录

```bash
git clone --recursive https://github.com/espressif/esp-dev-kits.git
cd esp-dev-kits/examples/esp-dualkey/examples/factory_demo

idf.py set-target esp32s3
idf.py build
idf.py -p <PORT> flash monitor

# RainMaker 模式需设置路径
export RMAKER_PATH=/path/to/rainmaker
```

---

## 10. 应用场景

| 应用领域 | 说明 |
|----------|------|
| 智能家居控制 | 作为物理按钮控制灯光、开关、场景（配合 RainMaker） |
| 宏键盘 / 快捷键面板 | 自定义按键映射，提升办公/创作效率 |
| 键盘外设 | BLE/USB 双模便携小键盘 |
| 流媒体/演示遥控 | 音量、播放、翻页、标签切换等远程控制 |
| 低功耗 IoT 触发器 | 电池供电 + Deep-sleep，作为事件触发节点 |

---

## 11. 相关资源与链接

### 官方资源

| 资源 | 链接 |
|------|------|
| ESP-DualKey 用户指南（中文） | https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32s3/esp-dualkey/user_guide.html |
| ESP-DualKey 索引页 | https://docs.espressif.com/projects/esp-dev-kits/zh_CN/latest/esp32s3/esp-dualkey/index.html |
| ESP-DualKey 出厂固件用户指南 | https://espressif.craft.me/5gX6vzhfn5QL93 |
| factory_demo 示例 | https://github.com/espressif/esp-dev-kits/tree/master/examples/esp-dualkey/examples/factory_demo |
| BSP 头文件 esp_dualkey.h | https://github.com/espressif/esp-dev-kits/blob/master/examples/esp-dualkey/components/esp_dualkey/include/bsp/esp_dualkey.h |
| ESP-BSP 仓库 | https://github.com/espressif/esp-bsp |

### M5Stack Chain DualKey 资源

| 资源 | 链接 |
|------|------|
| M5ChainDualKey 产品页 | https://docs.m5stack.com/en/chain/Chain_DualKey |
| 出厂固件使用教程 | https://docs.m5stack.com/en/guide/input_device/chain_dualkey |
| USB HID 编程示例 | https://docs.m5stack.com/en/arduino/chain_dualkey/hid |
| 电源管理示例 | https://docs.m5stack.com/en/arduino/chain_dualkey/power |

### 相关框架

| 框架 | 说明 |
|------|------|
| ESP RainMaker | 智能家居模式云平台，https://rainmaker.espressif.com |
| arduino-esp32 USB 库 | M5 版本可用，https://github.com/espressif/arduino-esp32/tree/master/libraries/USB |

---

> **文档总结**：ESP-DualKey 是乐鑫基于 ESP32-S3FN8 的智能双键控制器，最大特色是通过一个三档物理开关在蓝牙键盘 / USB 键盘 / 智能家居（RainMaker）三种模式间无缝切换。硬件上集成 2×1 按键矩阵、WS2812 RGB LED、TP4057 充电与 ADC 电池/VBUS 监控，内置 350mAh 锂电池，关机仅 8.97µA、Deep-sleep 仅 107µA。两个 HY2.0-4P 接口支持级联扩展。它是宏键盘、智能家居物理按钮、便携 HID 外设等场景的理想参考平台，亦是 M5Stack Chain DualKey 的官方姊妹产品。
