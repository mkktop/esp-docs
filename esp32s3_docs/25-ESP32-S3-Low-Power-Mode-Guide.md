# ESP32-S3 低功耗模式专项指南

> 本文档基于 ESP32-S3 技术参考手册、ESP-IDF 官方低功耗模式文档及数据手册整理，全面介绍 ESP32-S3 支持的各种低功耗模式、配置方法、唤醒源、功耗测量及优化策略。

---

## 目录

1. [功耗模式概述](#1-功耗模式概述)
2. [电源域架构](#2-电源域架构)
3. [Active 模式](#3-active-模式)
4. [Modem-sleep 模式](#4-modem-sleep-模式)
5. [Light-sleep 模式](#5-light-sleep-模式)
6. [Deep-sleep 模式](#6-deep-sleep-模式)
7. [休眠模式](#7-休眠模式-hibernation)
8. [动态频率调节 (DFS)](#8-动态频率调节-dfs)
9. [唤醒源详解](#9-唤醒源详解)
10. [Wi-Fi 场景低功耗](#10-wi-fi-场景低功耗)
11. [BLE 场景低功耗](#11-ble-场景低功耗)
12. [Deep-sleep 唤醒存根](#12-deep-sleep-唤醒存根)
13. [功耗测量数据](#13-功耗测量数据)
14. [配置步骤详解](#14-配置步骤详解)
15. [低功耗优化最佳实践](#15-低功耗优化最佳实践)

---

## 1. 功耗模式概述

ESP32-S3 支持 **4 种预设功耗模式**，从高功耗到低功耗依次为：

| 功耗模式 | CPU | RF 模块 | 典型功耗 | 唤醒延迟 |
|----------|-----|--------|---------|---------|
| Active | 运行 | 开启 | 20~340 mA | 即时 |
| Modem-sleep | 运行 | 关闭 | 13~107 mA | 即时 |
| Light-sleep | 暂停 | 关闭 | ~240 uA | <1 ms |
| Deep-sleep | 关闭 | 关闭 | ~7 uA | <1 ms (重新启动) |

功耗模式关系图：

```
Active  →  Modem-sleep  →  Light-sleep  →  Deep-sleep
高功耗                                          低功耗
高性能                                          低性能
即时唤醒                                        需重新启动
```

ESP32-S3 默认复位后进入 **Modem-sleep** 模式，当有收发包任务时可配置为 Active 模式。CPU 空闲时可进入更低功耗模式。

---

## 2. 电源域架构

ESP32-S3 共有三大类共 10 个电源域：

### 2.1 RTC 类
- 电源管理单元 (PMU) - 常开状态，不可关闭
- RTC 外设（RTC GPIO、RTC I2C、温度传感器、触摸传感器、RTC ADC、ULP 协处理器）

### 2.2 数字类
- 数字内核（CPU、ROM、SRAM）
- 无线通信 MAC 和 BB
- 部分数字外设（SPI2、GDMA、SHA、RSA、AES、USB OTG 等）

### 2.3 模拟类
- 快速 RC 振荡器 (RC_FAST_CLK)
- 外部晶振 (XTAL_CLK)
- 锁相环 (PLL)
- RF 电路

### 各模式下电源域状态

| 电源域 | Active | Modem-sleep | Light-sleep | Deep-sleep |
|--------|--------|-------------|-------------|------------|
| PMU | ON | ON | ON | ON |
| RTC 外设 | ON | ON | ON | ON* (可配置) |
| CPU | ON | ON | OFF* | OFF |
| 无线数字电路 | ON | ON* | OFF* | OFF |
| 数字内核 | ON | ON | ON* | OFF |
| RC_FAST_CLK | ON | ON | OFF | OFF |
| XTAL_CLK | ON | ON | OFF | OFF |
| PLL | ON | ON | OFF | OFF |
| RF 电路 | ON | OFF | OFF | OFF |

> `*` 表示可配置

---

## 3. Active 模式

Active 模式是芯片全功能运行状态，所有模块均处于上电工作状态。

### 功耗特性

| 工作场景 | 描述 | 峰值电流 |
|----------|------|---------|
| Wi-Fi TX | 802.11b, 1 Mbps, @21 dBm | 340 mA |
| Wi-Fi TX | 802.11g, 54 Mbps, @19 dBm | 291 mA |
| Wi-Fi TX | 802.11n, HT20, MCS7, @18.5 dBm | 283 mA |
| Wi-Fi RX | 802.11b/g/n, HT20 | 88 mA |
| BLE TX | @21 dBm | 335 mA |
| BLE TX | @0 dBm | 176 mA |
| BLE RX | 接收模式 | 93 mA |

Active 模式下的底电流大约为 20~40 mA（所有模块空闲时）。

---

## 4. Modem-sleep 模式

Modem-sleep 是 ESP32-S3 默认的功耗模式，当无线模块没有数据传输任务或不进行扫描时，RF 射频模块自动关闭以降低功耗，CPU 保持运行。

### 4.1 工作原理

Modem-sleep 基于 **DTIM 机制**：
1. Wi-Fi 任务结束后自动进入休眠
2. 关闭 PHY（RF 模块）
3. 按 DTIM 周期或监听间隔自动唤醒
4. CPU 和其他模块保持正常运行

```
电流 ▲
     |          ┌─────┐        ┌─────┐
     |          │     │        │     │
     |   DTIM   |     |        |     |
     |   到来    |     |  Wi-Fi |     |
     |       \  |     |  任务   |     |
     |        \ │     |  结束   |     │
     |  ────────┘     └────────┘     └──────
     └─────────────────────────────────────────► 时间
```

### 4.2 两种节能子模式

| 模式 | API | 说明 |
|------|-----|------|
| 最小节能模式 | `WIFI_PS_MIN_MODEM` | 每个 DTIM 间隔唤醒，不丢广播数据 |
| 最大节能模式 | `WIFI_PS_MAX_MODEM` | 每个监听间隔唤醒，可能丢广播数据 |

### 4.3 Modem-sleep 功耗数据

| 频率 | 说明 | 典型值 (外设时钟关闭) | 典型值 (外设时钟打开) |
|------|------|---------------------|---------------------|
| 40 MHz | WAITI (双核空闲) | 13.2 mA | 18.8 mA |
| 40 MHz | 双核 32 位 | 18.7 mA | 24.4 mA |
| 80 MHz | WAITI | 22.0 mA | 36.1 mA |
| 160 MHz | WAITI | 27.6 mA | 42.3 mA |
| 240 MHz | WAITI | 32.9 mA | 47.6 mA |
| 240 MHz | 双核 128 位 | 91.7 mA | 107.9 mA |

### 4.4 适用场景

- CPU 需要持续工作（如本地语音唤醒、音频处理）
- 需要保持 Wi-Fi 连接
- 对实时性要求较高的应用

---

## 5. Light-sleep 模式

Light-sleep 模式下，数字外设、CPU 及大部分 RAM 使用时钟门控，供电电压降低，RF 模块关闭。

### 5.1 Light-sleep 特点

- CPU 暂停工作，但**内部状态被保留**
- 唤醒后 CPU 接着入睡前的上下文继续运行
- 唤醒延迟低于 1 ms
- 典型功耗：**240 uA**

### 5.2 手动 Light-sleep

```c
#include "esp_sleep.h"

// 配置唤醒源（必须至少配置一个）
esp_sleep_enable_timer_wakeup(5 * 1000000); // 5 秒后唤醒

// 进入 Light-sleep
esp_light_sleep_start();
```

### 5.3 Auto Light-sleep 模式

Auto Light-sleep 是基于 FreeRTOS Tickless IDLE 功能的自动低功耗模式：

1. 当所有任务进入阻塞态或挂起态
2. 系统获取下一个就绪事件的时间点
3. 判定空闲时间超过设定值 (`CONFIG_FREERTOS_IDLE_TIME_BEFORE_SLEEP`)
4. 自动配置定时器唤醒源并进入 Light-sleep

```
┌──────────┐     系统空闲      ┌──────────┐   超过设定时间    ┌──────────┐
│          │  ─────────────►  │          │  ────────────►  │   auto   │
│  active  │                  │   IDLE   │                 │  light   │
│          │  ◄─────────────  │          │                 │   sleep  │
└──────────┘    系统非空闲      └──────────┘                 └────┬─────┘
    ▲                                                           │
    │                     配置唤醒源唤醒                          │
    └───────────────────────────────────────────────────────────┘
```

### 5.4 启用 Auto Light-sleep

```c
// 在 DFS 配置中同时启用 Auto Light-sleep
#if CONFIG_PM_ENABLE
    esp_pm_config_t pm_config = {
        .max_freq_mhz = 240,
        .min_freq_mhz = 40,
        .light_sleep_enable = true  // 启用 Auto Light-sleep
    };
    ESP_ERROR_CHECK(esp_pm_configure(&pm_config));
#endif
```

### 5.5 Light-sleep 子模式

| 子模式 | ULP/触摸 | RTC IO 输入 | ADC/TSEN Monitor | 8MD256 | 8MHz RC | XTAL |
|--------|---------|------------|-----------------|--------|---------|------|
| LSLP_DEFAULT | Y | Y | - | - | - | - |
| LSLP_ADC_TSENS | Y | Y | Y | - | - | - |
| LSLP_8MD256 | Y | Y | Y | Y | - | - |
| LSLP_LEDC8M/XTAL_FPU | Y | Y | Y | Y | Y | Y |

---

## 6. Deep-sleep 模式

Deep-sleep 模式是追求极致低功耗的模式，休眠时仅保留 RTC/LP 相关内存及外设，其余模块全部关闭。

### 6.1 Deep-sleep 特点

- CPU、大部分 RAM、所有数字外设断电
- 仅保留：RTC 控制器、ULP 协处理器、RTC 高速/低速内存
- 典型功耗：**7 uA**（RTC 外设掉电时）
- 唤醒后丢失 CPU 运行上下文，需重新运行引导加载程序

### 6.2 进入 Deep-sleep

```c
#include "esp_sleep.h"

// 配置唤醒源
esp_sleep_enable_timer_wakeup(30 * 1000000ULL); // 30 秒后唤醒

// 可选：配置 RTC 外设电源域
esp_sleep_pd_config(ESP_PD_DOMAIN_RTC_PERIPH, ESP_PD_OPTION_OFF);

// 进入 Deep-sleep
esp_deep_sleep_start();
// 此行之后的代码不会立即执行，需等唤醒后重新启动
```

### 6.3 Deep-sleep 功耗数据

| 工作模式 | 说明 | 典型值 (uA) |
|----------|------|------------|
| Deep-sleep | RTC 存储器和 RTC 外设上电 | 8 |
| Deep-sleep | **RTC 存储器上电，RTC 外设掉电** | **7** |
| Deep-sleep | 超低功耗传感器监测模式 | 18 |
| Deep-sleep | ULP-FSM 协处理器工作 | 170 |
| Deep-sleep | ULP-RISC-V 协处理器工作 | 190 |
| 关闭 | CHIP_PU 拉低 | 1 |

> **注意**：封装内有 PSRAM 的芯片功耗更高：
> - 8 MB 8 线 PSRAM (3.3V): +140 uA
> - 8 MB 8 线 PSRAM (1.8V): +200 uA
> - 2 MB 4 线 PSRAM: +40 uA

### 6.4 Deep-sleep 子模式

| 子模式 | ULP/触摸 | RTC IO 输入/高温 RTC 内存 | ADC/TSEN Monitor | 8MD256 |
|--------|---------|-------------------------|-----------------|--------|
| DSLP_ULTRA_LOW | Y | - | - | - |
| DSLP_DEFAULT | Y | Y | - | - |
| DSLP_8MD256/DSLP_ADC_TSENS | Y | Y | Y | Y |

配置超低功耗模式：
```c
// 启用超低功耗模式（关闭 RTC IO 输入和高温 RTC 内存）
esp_sleep_sub_mode_config(ESP_SLEEP_ULTRA_LOW_MODE, true);
```

### 6.5 适用场景

- 低功耗传感器应用（周期性采集数据）
- 大部分时间不需要数据传输的场景
- 待机模式

---

## 7. 休眠模式 (Hibernation)

休眠模式是最极端的低功耗模式（参考 ESP32 技术参考手册），虽然 ESP32-S3 的数据手册中主要列出 4 种预设模式，但在概念上：

- 内部 8 MHz 振荡器、40 MHz 高速晶振、PLL 及射频模块均禁用
- 数字内核断电，CPU 内容丢失
- RTC 外设域断电
- RTC 内核供电电压降至 0.7V
- 8 x 32 位数据保存在通用保留寄存器中
- 仅支持 RTC 计时器唤醒

> ESP32-S3 可以通过关闭尽可能多的电源域来接近休眠模式的功耗水平。

---

## 8. 动态频率调节 (DFS)

DFS (Dynamic Frequency Scaling) 是 ESP-IDF 电源管理机制的基础功能，根据应用程序持有电源锁的情况自动调整 APB 和 CPU 频率。

### 8.1 DFS 工作原理

```
持有 CPU 和 APB MAX 锁           释放 CPU MAX 锁
       │                        /
       ▼                       /
  ──────────┐                 /    释放 APB MAX 锁
             │               /    /
             │              /    /
             └───────────┐ /    /
                          ▼    /
                          ────────
```

### 8.2 启用 DFS

```c
#if CONFIG_PM_ENABLE
    esp_pm_config_t pm_config = {
        .max_freq_mhz = 240,       // 最大 CPU 频率
        .min_freq_mhz = 40,        // 最小 CPU 频率
        .light_sleep_enable = false // 是否启用 Auto Light-sleep
    };
    ESP_ERROR_CHECK(esp_pm_configure(&pm_config));
#endif
```

### 8.3 电源锁

DFS 根据应用程序持有的电源锁决定运行频率：

```c
#include "esp_pm.h"

// 获取高性能锁（阻止降频）
esp_pm_lock_handle_t lock;
esp_pm_lock_create(ESP_PM_CPU_FREQ_MAX, 0, "perf_lock", &lock);
esp_pm_lock_acquire(lock);
// 高性能操作...
esp_pm_lock_release(lock);
```

### 8.4 DFS 配置选项

```
# menuconfig
Component config → Power Management → Support for power management
CONFIG_PM_ENABLE=y
CONFIG_FREERTOS_USE_TICKLESS_IDLE=y
CONFIG_FREERTOS_HZ=100  # 影响调频灵敏度
```

---

## 9. 唤醒源详解

ESP32-S3 支持多种唤醒源，可单独使用或组合使用。

### 9.1 唤醒源总览

| 唤醒源 | Light-sleep | Deep-sleep | API |
|--------|-------------|------------|-----|
| 定时器 (RTC Timer) | Y | Y | `esp_sleep_enable_timer_wakeup()` |
| GPIO | Y | Y* | `esp_sleep_enable_gpio_wakeup()` |
| EXT0 (单个 RTC GPIO) | Y | Y | `esp_sleep_enable_ext0_wakeup()` |
| EXT1 (多个 RTC GPIO) | Y | Y | `esp_sleep_enable_ext1_wakeup()` |
| UART0 | Y | - | `esp_sleep_enable_uart_wakeup()` |
| UART1 | Y | - | `esp_sleep_enable_uart_wakeup()` |
| 触摸传感器 | Y | Y | `esp_sleep_enable_touchpad_wakeup()` |
| ULP-FSM | Y | Y | - |
| ULP-RISC-V | Y | Y | - |
| Wi-Fi | Y | - | 自动 (Modem-sleep) |
| BT | Y | - | 自动 |

> *Deep-sleep 模式下 GPIO 唤醒仅限 RTC GPIO

### 9.2 定时器唤醒

```c
// 唤醒时间精度为微秒，实际分辨率取决于 RTC_SLOW_CLK
esp_sleep_enable_timer_wakeup(10 * 1000000ULL); // 10 秒
```

### 9.3 EXT0 唤醒（单个 RTC GPIO）

```c
// 当指定 RTC GPIO 达到预定义电平时唤醒
esp_sleep_enable_ext0_wakeup(GPIO_NUM_0, 1); // GPIO0 为高电平时唤醒

// 可使用内部上拉/下拉电阻
rtc_gpio_pullup_en(GPIO_NUM_0);
rtc_gpio_pulldown_dis(GPIO_NUM_0);
```

### 9.4 EXT1 唤醒（多个 RTC GPIO）

```c
// 配置多个 RTC GPIO 作为唤醒源
#define PIN_BITMASK(x) (1ULL << x)
uint64_t pin_mask = PIN_BITMASK(GPIO_NUM_2) | PIN_BITMASK(GPIO_NUM_4);

// 任一选定引脚为高电平时唤醒
esp_sleep_enable_ext1_wakeup(pin_mask, ESP_EXT1_WAKEUP_ANY_HIGH);
```

### 9.5 GPIO 唤醒（Light-sleep）

```c
// Light-sleep 模式下的 GPIO 唤醒（不限于 RTC GPIO）
esp_sleep_enable_gpio_wakeup();
```

### 9.6 触摸传感器唤醒

```c
// 启用触摸传感器唤醒
esp_sleep_enable_touchpad_wakeup();

// ESP-IDF v5.x 新驱动配置
touch_sleep_config_t deep_slp_cfg = TOUCH_SENSOR_DEFAULT_DSLP_CONFIG();
touch_sensor_config_sleep_wakeup(sens_handle, &deep_slp_cfg);
```

### 9.7 UART 唤醒

```c
// ESP-IDF v6.1 新 API
#include "driver/uart_wakeup.h"

uart_wakeup_cfg_t wakeup_cfg = {
    .wakeup_mode = UART_WK_MODE_ACTIVE_THRESH,
    .rx_edge_threshold = 3,
};
uart_wakeup_setup(UART_NUM_0, &wakeup_cfg);
esp_sleep_enable_uart_wakeup(UART_NUM_0);
```

### 9.8 ULP 协处理器唤醒

ESP32-S3 支持两种 ULP 协处理器：
- **ULP-FSM**：使用汇编编程
- **ULP-RISC-V**：使用 C 语言编程，17.5 MHz

```c
// ULP-RISC-V 可以在 Deep-sleep 期间周期性唤醒
// 执行传感器读取、GPIO 控制等任务
// 当 ULP 需要主 CPU 处理时，通过中断唤醒主 CPU
```

### 9.9 唤醒源组合

多个唤醒源可以组合使用，任一唤醒源触发都会唤醒芯片：

```c
// 同时配置定时器和 GPIO 唤醒
esp_sleep_enable_timer_wakeup(60 * 1000000ULL);
esp_sleep_enable_ext0_wakeup(GPIO_NUM_0, 0);

// 获取唤醒原因
esp_sleep_wakeup_cause_t cause = esp_sleep_get_wakeup_cause();
if (cause == ESP_SLEEP_WAKEUP_TIMER) {
    // 定时器唤醒
} else if (cause == ESP_SLEEP_WAKEUP_EXT0) {
    // GPIO 唤醒
}
```

---

## 10. Wi-Fi 场景低功耗

### 10.1 Wi-Fi 场景低功耗模式选择

| 项目 | Modem-sleep | Modem-sleep+DFS | Auto Light-sleep | Deep-sleep |
|------|-------------|-----------------|-----------------|------------|
| 休眠 | 自动 | 自动 | 自动 | 手动 |
| 唤醒 | 自动 | 自动 | 自动 | 配置唤醒源 |
| Wi-Fi 连接 | 保持 | 保持 | 保持 | 断开 |
| CPU | 开 | 开/降频 | 暂停 | 关 |
| DTIM1 平均电流 | 40.1 mA | 20.7 mA | 2.45 mA | / |
| DTIM3 平均电流 | 38.7 mA | 19.9 mA | 1.33 mA | / |
| DTIM10 平均电流 | 38.2 mA | 19.5 mA | 0.93 mA | / |
| Deep-sleep 平均 | / | / | / | 6.8 uA |

### 10.2 Modem-sleep + DFS 配置

```c
// 启用电源管理
#if CONFIG_PM_ENABLE
    esp_pm_config_t pm_config = {
        .max_freq_mhz = 160,
        .min_freq_mhz = 80,
        .light_sleep_enable = false
    };
    ESP_ERROR_CHECK(esp_pm_configure(&pm_config));
#endif

// 配置 Wi-Fi Modem-sleep
esp_wifi_set_ps(WIFI_PS_MIN_MODEM);
```

### 10.3 Auto Light-sleep + Wi-Fi 配置

```c
#if CONFIG_PM_ENABLE
    esp_pm_config_t pm_config = {
        .max_freq_mhz = 80,
        .min_freq_mhz = 40,
        .light_sleep_enable = true  // 启用 Auto Light-sleep
    };
    ESP_ERROR_CHECK(esp_pm_configure(&pm_config));
#endif

// Wi-Fi 驱动会自动在空闲时请求 Light-sleep
esp_wifi_set_ps(WIFI_PS_MIN_MODEM);
```

### 10.4 Deep-sleep + Wi-Fi 场景

Deep-sleep 会断开 Wi-Fi 连接。典型工作流程：

```
唤醒 → 连接 Wi-Fi → 上传数据 → 断开 Wi-Fi → 进入 Deep-sleep → 等待下次唤醒
```

```c
void app_main() {
    // 检查唤醒原因
    esp_sleep_wakeup_cause_t cause = esp_sleep_get_wakeup_cause();

    // 连接 Wi-Fi 并上传数据
    wifi_init();
    wifi_connect();
    upload_data();

    // 断开 Wi-Fi
    esp_wifi_stop();

    // 配置定时器唤醒
    esp_sleep_enable_timer_wakeup(300 * 1000000ULL); // 5 分钟

    // 进入 Deep-sleep
    esp_deep_sleep_start();
}
```

### 10.5 非连接状态下的休眠

通过 `CONFIG_ESP_WIFI_STA_DISCONNECTED_PM_ENABLE` 可以在 station 未连接状态下也关闭 RF/PHY/BB：

```
# menuconfig
Component config → Wi-Fi → Power saving for disconnected station
```

默认已开启，共存模式下强制打开。

---

## 11. BLE 场景低功耗

### 11.1 时钟源选择

BLE 协议栈的功耗与时钟源选择密切相关：

| 时钟源 | 精度 | 功耗影响 |
|--------|------|---------|
| 外部 32 kHz 晶振 | 高 | 低（BLE 协议栈能更精确地控制唤醒时间） |
| 内部 RC 振荡器 | 低 | 较高（需要更长的唤醒窗口补偿） |

建议在硬件设计时使用外部 32 kHz 晶振以降低 BLE 功耗。

### 11.2 BLE 连接参数优化

- **连接间隔 (Connection Interval)**：较长间隔降低功耗，但增加延迟
- **从机延迟 (Slave Latency)**：允许从机跳过若干个连接事件
- **超时 (Supervision Timeout)**：连接超时时间

### 11.3 BLE + DFS 模式

BLE 协议栈内部已集成了类似 Modem-sleep 的机制，在不需要 RF 时自动关闭 PHY。结合 DFS 可以进一步降低功耗。

---

## 12. Deep-sleep 唤醒存根

Deep-sleep 唤醒存根 (Wake Stub) 允许在芯片从 Deep-sleep 唤醒后、在任何正常初始化之前，快速执行一些代码。

### 12.1 工作原理

```
Deep-sleep 唤醒
    │
    ▼
部分初始化
    │
    ▼
CRC 验证 RTC 快速内存 ──失败──→ 正常启动
    │
    成功
    │
    ▼
执行唤醒存根代码
    │
    ▼
回到 Deep-sleep 或继续正常启动
```

### 12.2 实现唤醒存根

**方法一：使用 RTC_IRAM_ATTR**

```c
void RTC_IRAM_ATTR esp_wake_deep_sleep(void) {
    // 首先调用默认函数
    esp_default_wake_deep_sleep();

    // 添加自定义逻辑
    // 注意：只能调用 ROM 中或 RTC 快速内存中的函数
}
```

**方法二：使用独立源文件**

将代码放入以 `rtc_wake_stub` 开头的源文件中，链接器会自动将其放入 RTC 快速内存。

### 12.3 唤醒存根示例

```c
#include "esp_sleep.h"
#include "rom/rtc.h"

static RTC_FAST_ATTR int s_count = 0;
static RTC_FAST_ATTR int s_max_count = 3;

void RTC_IRAM_ATTR esp_wake_deep_sleep(void) {
    esp_default_wake_deep_sleep();

    s_count++;

    if (s_count < s_max_count) {
        // 设置下次唤醒时间
        esp_wake_stub_set_wakeup_time(
            esp_wake_stub_get_wakeup_cause() == ESP_SLEEP_WAKEUP_TIMER
            ? 5000000 : 0);

        // 回到 Deep-sleep
        esp_wake_stub_sleep(esp_wake_deep_sleep);
    }

    // 达到最大次数后，恢复计数并继续正常启动
    s_count = 0;
}
```

### 12.4 注意事项

- 唤醒存根代码**必须**驻留在 RTC 快速内存中
- 只能调用 ROM 中或加载到 RTC 快速内存中的函数
- SPI Flash 未被映射，不能访问 Flash 中的数据
- 存根大小受 RTC 快速内存大小限制 (8 KB)
- 字符串常量必须用 `RTC_RODATA_ATTR` 标记并声明为数组
- 启用 `CONFIG_BOOTLOADER_SKIP_VALIDATE_IN_DEEP_SLEEP` 可减少唤醒时间

---

## 13. 功耗测量数据

### 13.1 完整功耗对比表

| 模式 | 条件 | 功耗 |
|------|------|------|
| Active (Wi-Fi TX) | 802.11b, 1 Mbps, @21 dBm | 340 mA (峰值) |
| Active (Wi-Fi RX) | 802.11n, HT20 | 88 mA (峰值) |
| Active (BLE TX) | @0 dBm | 176 mA (峰值) |
| Modem-sleep (40 MHz) | WAITI, 外设时钟关闭 | 13.2 mA |
| Modem-sleep (240 MHz) | WAITI, 外设时钟关闭 | 32.9 mA |
| Light-sleep | VDD_SPI 和 Wi-Fi 掉电 | 240 uA |
| Deep-sleep | RTC 外设掉电 | 7 uA |
| Deep-sleep | 超低功耗传感器监测模式 | 18 uA |
| Deep-sleep | ULP-FSM 工作 | 170 uA |
| 关闭 | CHIP_PU 拉低 | 1 uA |

### 13.2 Wi-Fi 场景平均电流（屏蔽箱测试）

| 模式 | DTIM1 | DTIM3 | DTIM10 |
|------|-------|-------|--------|
| Modem-sleep | 40.1 mA | 38.7 mA | 38.2 mA |
| Modem-sleep + DFS | 20.7 mA | 19.9 mA | 19.5 mA |
| Auto Light-sleep | 2.45 mA | 1.33 mA | 0.93 mA |

### 13.3 PSRAM 对功耗的影响

| 配置 | Light-sleep 额外功耗 | Deep-sleep 额外功耗 |
|------|---------------------|---------------------|
| 8 MB 8 线 PSRAM (3.3V) | +140 uA | +140 uA |
| 8 MB 8 线 PSRAM (1.8V) | +200 uA | +200 uA |
| 2 MB 4 线 PSRAM | +40 uA | +40 uA |

---

## 14. 配置步骤详解

### 14.1 纯系统低功耗配置

**步骤 1：启用电源管理**

```
# menuconfig
Component config → Power Management → Support for power management = y
```

**步骤 2：配置 DFS**

```c
esp_pm_config_t pm_config = {
    .max_freq_mhz = 240,
    .min_freq_mhz = 40,
    .light_sleep_enable = true  // 启用 Auto Light-sleep
};
esp_pm_configure(&pm_config);
```

**步骤 3：配置 Light-sleep（可选）**

```c
// 手动进入 Light-sleep
esp_sleep_enable_timer_wakeup(5000000); // 5 秒
esp_light_sleep_start();
```

**步骤 4：配置 Deep-sleep**

```c
// 配置唤醒源
esp_sleep_enable_timer_wakeup(60000000); // 60 秒

// 配置电源域
esp_sleep_pd_config(ESP_PD_DOMAIN_RTC_PERIPH, ESP_PD_OPTION_OFF);

// 进入 Deep-sleep
esp_deep_sleep_start();
```

### 14.2 公共配置选项

```
# menuconfig 公共配置
CONFIG_PM_ENABLE=y                    # 电源管理
CONFIG_FREERTOS_USE_TICKLESS_IDLE=y   # Tickless IDLE
CONFIG_FREERTOS_HZ=100                # 系统调度频率
CONFIG_FREERTOS_IDLE_TIME_BEFORE_SLEEP=3  # 进入睡眠的最小空闲时间
CONFIG_PM_SLP_IRAM_OPT=y              # 将睡眠代码放入 IRAM
CONFIG_PM_PRESSURE_TOUCH=y            # 触摸传感器睡眠支持
```

### 14.3 保持特定模块在睡眠期间开启

```c
// 在 Deep-sleep 中保持外部 40 MHz 晶振开启
esp_sleep_pd_config(ESP_PD_DOMAIN_XTAL, ESP_PD_OPTION_ON);

// 在 Light-sleep 中保持内部 8 MHz 振荡器开启
esp_sleep_pd_config(ESP_PD_DOMAIN_RTC8M, ESP_PD_OPTION_ON);
```

---

## 15. 低功耗优化最佳实践

### 15.1 硬件层面

1. **使用外部 32 kHz 晶振**：提高 RTC 定时器精度，降低 BLE 唤醒窗口
2. **选择合适的 Flash/PSRAM**：根据应用需求选择最小容量和合适类型
3. **优化 PCB 设计**：减少漏电流，确保电源稳定性
4. **断开不必要的外设**：在睡眠前断开 LED、传感器等外设的供电

### 15.2 软件层面

1. **合理选择功耗模式**：
   - 需要保持 Wi-Fi 连接 + CPU 工作 → Modem-sleep + DFS
   - 需要保持 Wi-Fi 连接 + 允许 CPU 暂停 → Auto Light-sleep
   - 不需要 Wi-Fi 连接 + 周期性唤醒 → Deep-sleep

2. **优化 Wi-Fi DTIM 周期**：
   - 如果 AP 支持较大的 DTIM，使用 Auto Light-sleep 效果更好
   - DTIM10 + Auto Light-sleep 可低至 0.93 mA

3. **使用 ULP 协处理器**：
   - 在 Deep-sleep 期间使用 ULP 进行传感器监测
   - 只有在需要复杂处理时才唤醒主 CPU

4. **使用 Deep-sleep 唤醒存根**：
   - 对于不需要完整启动的周期性任务
   - 可以大幅减少唤醒后的处理时间

5. **关闭不必要的电源域**：
   ```c
   esp_sleep_pd_config(ESP_PD_DOMAIN_RTC_PERIPH, ESP_PD_OPTION_OFF);
   esp_sleep_pd_config(ESP_PD_DOMAIN_XTAL, ESP_PD_OPTION_OFF);
   ```

6. **使用电源锁管理性能需求**：
   ```c
   // 仅在需要高性能时获取锁
   esp_pm_lock_acquire(perf_lock);
   // 执行高性能操作
   esp_pm_lock_release(perf_lock);
   ```

### 15.3 系统级优化策略

```
应用场景决策树：

是否需要 Wi-Fi/BLE 连接？
├── 是 → 是否允许 CPU 暂停？
│       ├── 是 → Auto Light-sleep (最优，~0.93 mA @ DTIM10)
│       └── 否 → Modem-sleep + DFS (~19.5 mA @ DTIM10)
└── 否 → 数据采集频率？
        ├── 高（< 1 分钟）→ Light-sleep
        └── 低（> 1 分钟）→ Deep-sleep (~7 uA)
                             + ULP 周期性监测
```

---

## 参考资料

- [ESP32-S3 技术规格书](https://www.espressif.com/sites/default/files/documentation/esp32-s3_datasheet_cn.pdf)
- [ESP32-S3 技术参考手册 - 低功耗管理](https://www.espressif.com/sites/default/files/documentation/esp32-s3_technical_reference_manual_cn.pdf)
- [ESP-IDF 低功耗模式指南](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/api-guides/low-power-mode/index.html)
- [ESP-IDF 睡眠模式 API](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/api-reference/system/sleep_modes.html)
- [ESP-IDF Deep-sleep 唤醒存根](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/api-guides/deep-sleep-stub.html)
- [ESP-IDF 电源管理](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/api-reference/system/power_management.html)
- [ESP 低功耗入门](https://docs.espressif.com/projects/esp-techpedia/zh_CN/latest/esp-friends/advanced-development/performance/lowpower/lowpower-fundamental.html)
- [ESP-IDF Deep-sleep 示例](https://github.com/espressif/esp-idf/tree/master/examples/system/deep_sleep)
- [ESP-IDF Light-sleep 示例](https://github.com/espressif/esp-idf/tree/master/examples/system/light_sleep)
- [ESP-IDF 唤醒存根示例](https://github.com/espressif/esp-idf/tree/master/examples/system/deep_sleep_wake_stub)
