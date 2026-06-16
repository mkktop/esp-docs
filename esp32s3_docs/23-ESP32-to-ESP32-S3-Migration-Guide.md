# ESP32 到 ESP32-S3 迁移指南

> 本文档基于 Espressif 官方文档、ESP-IDF 迁移指南以及 ESP32-S3 技术规格书整理而成，旨在帮助开发者从 ESP32 平台平滑迁移到 ESP32-S3 平台。

---

## 目录

1. [概述](#1-概述)
2. [硬件差异对比](#2-硬件差异对比)
3. [CPU 与内存差异](#3-cpu-与内存差异)
4. [GPIO 与引脚映射差异](#4-gpio-与引脚映射差异)
5. [外设差异](#5-外设差异)
6. [无线通信差异](#6-无线通信差异)
7. [安全特性差异](#7-安全特性差异)
8. [API 变更与迁移](#8-api-变更与迁移)
9. [构建配置变更](#9-构建配置变更)
10. [迁移步骤清单](#10-迁移步骤清单)
11. [常见迁移问题](#11-常见迁移问题)

---

## 1. 概述

ESP32-S3 是乐鑫科技专为 AIoT 市场打造的芯片，搭载 Xtensa 32 位 LX7 双核处理器，主频高达 240 MHz，内置 512 KB SRAM，具有 45 个可编程 GPIO 管脚。与 ESP32 相比，ESP32-S3 在 AI 算力、安全加密、GPIO 数量和外设接口等方面都有显著提升。

ESP32-S3 的主要提升点：

- 新增用于加速神经网络计算和信号处理的**向量指令 (Vector Instructions)**
- 支持**更大容量的高速 Octal SPI Flash 和片外 PSRAM**
- 新增 **USB OTG** 和 **USB 串口/JTAG 控制器**
- 新增**世界控制器 (World Controller)** 模块，支持可信执行环境
- GPIO 从 34 个增加到 **45 个**
- 支持 **Bluetooth 5 (LE)**，包含远距离模式和 2 Mbps 高速模式

---

## 2. 硬件差异对比

### 2.1 核心规格对比表

| 特性 | ESP32 | ESP32-S3 |
|------|-------|----------|
| CPU | Xtensa LX7 双核 | Xtensa LX7 双核 |
| 主频 | 最高 240 MHz | 最高 240 MHz |
| 浮点运算单元 (FPU) | 有 | 有 |
| AI 向量指令 | 无 | **有** |
| SRAM | 520 KB | 512 KB |
| ROM | 448 KB | 384 KB |
| RTC SRAM | 16 KB | 16 KB |
| GPIO | 34 | **45** |
| 触摸传感器 GPIO | 10 | **14** |
| UART | 3 | 3 |
| SPI | 4 | 4 |
| I2C | 2 | 2 |
| I2S | 2 | 2 |
| ADC | 2x 12 位 | 2x 12 位 |
| DAC | 2x 8 位 | **无 (已移除)** |
| USB OTG | 无 | **有 (全速)** |
| USB 串口/JTAG | 无 | **有** |
| SD/MMC 主机 | 有 | 有 (2 个卡槽) |
| MCPWM | 无 | **有 (2 个)** |
| LCD 接口 | 无 | **有** |
| 摄像头接口 | 无 | **有 (DVP 8~16 位)** |
| 封装 | QFN48 (6x6 mm)/QFN38 (5x5 mm) | QFN56 (7x7 mm) |
| 超低功耗协处理器 | ULP-FSM | **ULP-FSM + ULP-RISC-V** |
| 世界控制器 (TEE) | 无 | **有** |

### 2.2 关键硬件差异说明

#### DAC 被移除

ESP32-S3 **移除了 DAC 功能** (2022年1月修订)。ESP32 上使用的 `dac_output_*()` 系列 API 在 ESP32-S3 上不可用。若项目依赖 DAC，需要使用外部 DAC 芯片或改用 LEDC + RC 滤波方案。

#### USB 支持

ESP32-S3 内置了两个 USB 接口：
- **USB OTG**：全速 USB OTG 控制器，可用作 USB 主机或设备
- **USB 串口/JTAG**：固定功能的 USB 设备，同时集成 USB-to-Serial 和 USB-to-JTAG 功能，无需外部桥接芯片即可进行固件下载和调试

---

## 3. CPU 与内存差异

### 3.1 CPU 架构

虽然 ESP32 和 ESP32-S3 都使用 Xtensa LX7 双核处理器，但 ESP32-S3 有以下改进：

- **128 位数据总线位宽**，专用的 SIMD 指令
- **向量指令**：用于加速神经网络计算和信号处理
- 五级流水线架构

### 3.2 内部 SRAM

| 参数 | ESP32 | ESP32-S3 |
|------|-------|----------|
| 内部 SRAM | 520 KB | 512 KB (含 TCM) |
| RTC SRAM | 16 KB | 16 KB (8 KB 快速 + 8 KB 慢速) |
| ROM | 448 KB | 384 KB |

> **注意**：ESP32-S3 的内部 SRAM 比 ESP32 略少 (512 KB vs 520 KB)，但 ESP32-S3 支持更大容量的外部 PSRAM (最高 8 MB Octal PSRAM)，可以弥补内部 SRAM 的不足。

### 3.3 外部 Flash 与 PSRAM

| 参数 | ESP32 | ESP32-S3 |
|------|-------|----------|
| Flash 接口 | Quad SPI | Quad SPI / **Octal SPI** |
| PSRAM 接口 | Quad SPI | Quad SPI / **Octal SPI** |
| 最大 Flash 容量 | 16 MB | 16 MB+ |
| 最大 PSRAM 容量 | 4 MB (Quad) | **8 MB (Octal) / 2 MB (Quad)** |

ESP32-S3 引入了 cache 机制的 Flash 控制器，支持用户配置数据缓存与指令缓存：

```
# menuconfig 中配置
CONFIG_ESP32S3_INSTRUCTION_CACHE_32KB=y
CONFIG_ESP32S3_DATA_CACHE_64KB=y
```

---

## 4. GPIO 与引脚映射差异

### 4.1 GPIO 数量变化

ESP32-S3 有 **45 个可编程 GPIO** (GPIO0 ~ GPIO21, GPIO26 ~ GPIO48)，相比 ESP32 的 34 个增加了 11 个。

### 4.2 Strapping 管脚

ESP32-S3 有 4 个 Strapping 管脚，上电时的电平状态决定芯片的启动模式：

| Strapping 管脚 | 默认状态 | 功能说明 |
|----------------|----------|----------|
| GPIO0 | 弱上拉 | 启动模式选择 |
| GPIO45 | 弱下拉 | VDD_SPI Flash 电压选择 (1.8V/3.3V) |
| GPIO46 | 弱下拉 | 启动模式控制 |
| GPIO3 | 弱上拉 | JTAG 信号源选择 |

> **注意**：GPIO45 用于选择 VDD_SPI Flash 电压。当使用 1.8V Flash 时需拉高，使用 3.3V Flash 时需拉低。这与 ESP32 的设计不同。

### 4.3 关键引脚差异

| 功能 | ESP32 引脚 | ESP32-S3 引脚 |
|------|-----------|---------------|
| UART0 TX | GPIO1 (TXD0) | GPIO43 (TXD0) |
| UART0 RX | GPIO3 (RXD0) | GPIO44 (RXD0) |
| USB D+ | 无 | GPIO20 |
| USB D- | 无 | GPIO19 |
| Flash/PSRAM | GPIO6~GPIO11 | GPIO26~GPIO32 (Octal) |
| ADC1 通道 | GPIO32~GPIO39 | GPIO1~GPIO10 |
| ADC2 通道 | GPIO0,2,4,12~15,25~27 | GPIO11~GPIO20 |

> **重要**：ESP32-S3 的 Flash/PSRAM 引脚范围与 ESP32 不同，在设计 PCB 时必须重新规划引脚分配。

### 4.4 RTC GPIO

ESP32-S3 有 21 个 RTC GPIO (GPIO0~GPIO21)，用于 RTC 功能和低功耗唤醒。

---

## 5. 外设差异

### 5.1 新增外设

| 外设 | 说明 |
|------|------|
| USB OTG | 全速 USB OTG 控制器 |
| USB 串口/JTAG | 固定功能 USB 设备，支持固件烧录和调试 |
| LCD 接口 | 支持 I8080 和 RGB 接口 |
| 摄像头接口 | DVP 8~16 位 |
| MCPWM | 2 个电机控制 PWM |
| 世界控制器 | TEE 支持 |
| ULP-RISC-V | 17.5 MHz RISC-V 超低功耗协处理器 |

### 5.2 移除的外设

| 外设 | ESP32 | ESP32-S3 |
|------|-------|----------|
| DAC | 2x 8 位 | **已移除** |
| 霍尔传感器 | 有 | **已移除** |

### 5.3 ADC 差异

ESP32-S3 的 ADC 从 ESP32 的 12 位升级，通道分配完全不同：

```c
// ESP32-S3 ADC 配置示例
#include "esp_adc/adc_oneshot.h"
#include "esp_adc/adc_continuous.h"

// ADC 单次模式 (ESP-IDF v5.x 新驱动)
adc_oneshot_unit_handle_t adc1_handle;
adc_oneshot_unit_init_cfg_t init_config = {
    .unit_id = ADC_UNIT_1,
};
ESP_ERROR_CHECK(adc_oneshot_new_unit(&init_config, &adc1_handle));
```

---

## 6. 无线通信差异

### 6.1 Wi-Fi

| 特性 | ESP32 | ESP32-S3 |
|------|-------|----------|
| Wi-Fi 协议 | 802.11 b/g/n | 802.11 b/g/n |
| 带宽 | 20/40 MHz | 20/40 MHz |
| 虚拟接口 | 4 个 | 4 个 |
| WPA3 | 支持 | 支持 |
| FTM | 不支持 | **支持 (802.11mc)** |

### 6.2 蓝牙

| 特性 | ESP32 | ESP32-S3 |
|------|-------|----------|
| 经典蓝牙 | **支持** | **不支持** |
| BLE 版本 | 4.2 | **5.0** |
| BLE Mesh | 支持 | 支持 |
| 远距离模式 | 不支持 | **支持** |
| 2 Mbps PHY | 不支持 | **支持** |
| 广播扩展 | 不支持 | **支持** |

> **关键差异**：ESP32-S3 **不支持经典蓝牙 (Classic Bluetooth)**，仅支持低功耗蓝牙。如果项目依赖 SPP、A2DP 等经典蓝牙协议，将无法直接迁移到 ESP32-S3。

---

## 7. 安全特性差异

| 特性 | ESP32 | ESP32-S3 |
|------|-------|----------|
| Flash 加密算法 | AES | **AES-XTS** |
| 安全启动 | Secure Boot V1/V2 | **Secure Boot V2** |
| 世界控制器 (TEE) | 无 | **有** |
| 数字签名 | RSA | RSA |
| HMAC | 有 | 有 |
| AES 密钥大小 | 256 位 | **256 位 / 512 位** |

### Flash 加密变化

ESP32-S3 使用 AES-XTS 算法替代了 ESP32 的 AES 算法。密钥大小支持 256 位 (XTS_AES_128) 或 512 位 (XTS_AES_256)。

---

## 8. API 变更与迁移

### 8.1 头文件路径变更

从 ESP-IDF v5.0 开始，许多 API 的头文件路径发生了变化。以下是 ESP32-S3 迁移时需要注意的关键变更：

```c
// 旧 (ESP-IDF v4.x)
#include "driver/adc.h"
#include "driver/periph_ctrl.h"
#include "driver/rtc_cntl.h"
#include "soc/cpu.h"
#include "esp_system.h"
#include "esp_clk.h"

// 新 (ESP-IDF v5.x)
#include "esp_adc/adc_oneshot.h"
#include "esp_adc/adc_continuous.h"
#include "esp_private/periph_ctrl.h"
#include "esp_private/rtc_ctrl.h"
#include "esp_cpu.h"
#include "esp_random.h"      // 需要单独包含
#include "esp_mac.h"         // 需要单独包含
#include "esp_chip_info.h"   // 需要单独包含
#include "esp_private/esp_clk.h"
```

### 8.2 驱动组件拆分

从 ESP-IDF v5.3 开始，原先位于 `driver` 组件下的驱动被拆分到独立组件：

| 旧组件 | 新组件 |
|--------|--------|
| driver/uart | esp_driver_uart |
| driver/i2c | esp_driver_i2c |
| driver/spi | esp_driver_spi |
| driver/gpio | esp_driver_gpio |
| driver/gptimer | esp_driver_gptimer |
| driver/ledc | esp_driver_ledc |
| driver/rmt | esp_driver_rmt |
| driver/i2s | esp_driver_i2s |
| driver/dac | esp_driver_dac |

在 `CMakeLists.txt` 中需要更新依赖：

```cmake
# 旧
idf_component_register(SRCS "main.c"
                       INCLUDE_DIRS "."
                       REQUIRES driver)

# 新
idf_component_register(SRCS "main.c"
                       INCLUDE_DIRS "."
                       REQUIRES esp_driver_uart esp_driver_gpio esp_driver_i2c)
```

### 8.3 PSRAM API 变更

```c
// 旧 (ESP-IDF v4.x)
#include "esp_spiram.h"
size_t size = esp_spiram_get_size();

// 新 (ESP-IDF v5.x)
#include "esp_psram.h"
size_t size = esp_psram_get_size();
```

### 8.4 ROM 头文件路径

```c
// 旧
#include "rom/uart.h"

// 新
#include "esp32s3/rom/uart.h"
```

### 8.5 ADC 校准 API 迁移

```c
// 旧
#include "esp_adc_cal.h"
esp_adc_cal_characterize(...);
esp_adc_cal_raw_to_voltage(...);
esp_adc_cal_get_voltage(...);

// 新
#include "esp_adc/adc_cali.h"
#include "esp_adc/adc_cali_scheme.h"
adc_cali_create_scheme_curve_fitting(...);
adc_cali_raw_to_voltage(...);
adc_oneshot_get_calibrated_result(...);
```

### 8.6 GPIO 中断处理变更

ESP-IDF v5.0 后，GPIO 中断状态寄存器在调用用户回调函数 **之前** 被清空。用户无法再在回调函数中读取中断状态寄存器来确定触发引脚，应通过回调函数参数获取：

```c
// 新方式：通过回调参数获取 GPIO 编号
static void IRAM_ATTR gpio_isr_handler(void *arg) {
    uint32_t gpio_num = (uint32_t) arg;
    // 处理中断
}
```

---

## 9. 构建配置变更

### 9.1 设置目标芯片

```bash
# 设置目标为 ESP32-S3
idf.py set-target esp32s3
```

### 9.2 关键 Kconfig 变更

```
# PSRAM 配置 (menuconfig)
Component config → ESP32S3 Specific → Support for external, SPI connected RAM → SPI RAM config

# Octal PSRAM 配置
SPI RAM config → Mode (QUAD/OCT) of SPI RAM chip in use → Octal Mode PSRAM

# Flash 配置
Serial flasher config → Flash size → 8 MB
Serial flasher config → Flash mode → DIO / DOUT / QIO / QOUT

# USB 串口/JTAG 控制台
Component config → ESP System Settings → Channel for console output → USB Serial/JTAG Controller
```

### 9.3 分区表

ESP32-S3 的分区表与 ESP32 格式相同，但由于支持更大的 Flash 和 PSRAM，通常需要调整分区大小：

```
# partitions.csv 示例
nvs,      data, nvs,     0x9000,  0x6000,
phy_init, data, phy,     0xf000,  0x1000,
factory,  app,  factory, 0x10000, 1M,
```

---

## 10. 迁移步骤清单

### 步骤 1：硬件评估

- [ ] 确认是否使用了经典蓝牙（ESP32-S3 不支持）
- [ ] 确认是否使用了 DAC（ESP32-S3 不支持）
- [ ] 确认是否使用了霍尔传感器（ESP32-S3 不支持）
- [ ] 检查 GPIO 引脚分配是否需要重新映射
- [ ] 确认 Flash/PSRAM 引脚是否冲突

### 步骤 2：软件迁移

- [ ] 执行 `idf.py set-target esp32s3`
- [ ] 更新 PSRAM 配置（Quad/Octal 模式）
- [ ] 更新 Flash 配置（大小、模式、频率）
- [ ] 迁移 ADC 驱动到新 API (`esp_adc/adc_oneshot.h`)
- [ ] 更新头文件引用路径（v4.x → v5.x 变更）
- [ ] 更新 CMakeLists.txt 中的组件依赖
- [ ] 更新 linker.lf 文件中的驱动路径

### 步骤 3：功能验证

- [ ] 验证 Wi-Fi 连接功能
- [ ] 验证 BLE 功能（注意协议差异）
- [ ] 验证 GPIO、UART、I2C、SPI 等外设
- [ ] 验证 PSRAM 访问
- [ ] 验证低功耗模式
- [ ] 验证 Flash 加密和安全启动（如使用）

### 步骤 4：性能优化

- [ ] 利用 AI 向量指令优化算法（如适用）
- [ ] 配置指令缓存和数据缓存大小
- [ ] 评估是否需要使用 Octal PSRAM 提升性能
- [ ] 利用 USB OTG 或 USB 串口/JTAG 简化开发

---

## 11. 常见迁移问题

### 11.1 经典蓝牙不可用

**问题**：ESP32-S3 不支持经典蓝牙，仅支持 BLE 5.0。

**解决方案**：
- SPP (串口协议) → 改用 BLE GATT 或 Wi-Fi
- A2DP (音频分发) → 改用 Wi-Fi 传输或 BLE Audio
- HID over GATT → 使用 BLE HID

### 11.2 PSRAM 配置错误

**问题**：ESP32-S3R8V (8 MB PSRAM) 需要配置为 Octal 模式。

**解决方案**：
```
# menuconfig 路径
Component config → ESP32S3 Specific → Support for external, SPI connected RAM
→ SPI RAM config → Mode (QUAD/OCT) of SPI RAM chip in use → Octal Mode PSRAM
```

### 11.3 Flash 模式不匹配

**问题**：烧录后程序无法启动，日志显示 boot 错误。

**解决方案**：
- 确认 Flash 模式 (DIO/DOUT/QIO/QOUT)
- 使用 `-fm dio` 参数烧录
- 检查 Flash 频率配置

### 11.4 USB 下载模式

ESP32-S3 支持 USB 下载，但首次使用或 Flash 为空时需要手动进入下载模式：

1. 按住 BOOT 按键
2. 短按 RST 按键
3. 释放 BOOT 按键
4. 执行烧录命令

### 11.5 ADC 通道映射不同

ESP32-S3 的 ADC 通道与 ESP32 完全不同，必须参考 ESP32-S3 数据手册重新映射 ADC 通道。

---

## 参考资料

- [ESP32-S3 技术规格书](https://www.espressif.com/sites/default/files/documentation/esp32-s3_datasheet_cn.pdf)
- [ESP32-S3 技术参考手册](https://www.espressif.com/sites/default/files/documentation/esp32-s3_technical_reference_manual_cn.pdf)
- [ESP-IDF 迁移指南 (ESP32-S3)](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/migration-guides/index.html)
- [ESP32-S3 硬件设计指南](https://www.espressif.com/sites/default/files/documentation/esp32-s3_hardware_design_guidelines_cn.pdf)
- [ESP32-S3 产品页面](https://www.espressif.com/zh-hans/products/socs/esp32-s3)
