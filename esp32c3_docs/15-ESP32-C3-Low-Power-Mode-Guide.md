# ESP32-C3 低功耗模式指南

> 文档编号：15
> 适用芯片：ESP32-C3（RISC-V 单核，低功耗 IoT）
> 来源：乐鑫官方 ESP-IDF 低功耗模式指南、Datasheet 功耗章节、ESP-FAQ、低功耗入门

---

## 目录

1. [低功耗概述](#1-低功耗概述)
2. [四种功耗模式与功耗数据](#2-四种功耗模式与功耗数据)
3. [Active 与 Modem-sleep](#3-active-与-modem-sleep)
4. [Light-sleep 模式](#4-light-sleep-模式)
5. [Deep-sleep 模式](#5-deep-sleep-模式)
6. [唤醒源](#6-唤醒源)
7. [Wi-Fi 场景低功耗策略](#7-wi-fi-场景低功耗策略)
8. [BLE 场景低功耗策略](#8-ble-场景低功耗策略)
9. [SPI Flash 深度休眠（DPD）](#9-spi-flash-深度休眠dpd)
10. [功耗测量](#10-功耗测量)
11. [参考资源](#11-参考资源)

---

## 1. 低功耗概述

ESP32-C3 是低功耗 IoT 芯片，支持四种工作模式（按平均功耗由高到低）：**Active → Modem-sleep → Light-sleep → Deep-sleep**。不同模式下各功能模块的上下电状态不同，从而产生功耗差异。

待机性能（sleep 电流）对电池供电 IoT 极其关键。ESP32-C3 在 Deep-sleep 下可低至 **5 µA**，是同类竞品中的领先水平。

---

## 2. 四种功耗模式与功耗数据

| 模式 | 功耗典型值 | 说明 |
|------|-----------|------|
| **Active**（RF 工作） | 见 RF 表，TX 峰值 335 mA | 全模块工作 |
| **Modem-sleep** | 13 ~ 28 mA | CPU 运行，Wi-Fi/BT 基带与射频关闭 |
| **Light-sleep** | **130 µA** | CPU 暂停，RTC 工作，状态保留 |
| **Deep-sleep** | **5 µA** | 仅 RTC 定时器 + RTC 内存 |
| 关闭 | 1 µA | CHIP_EN 拉低 |

### Wi-Fi Active RF 功耗（3.3V，25°C，100% 占空比）

| 模式 | 描述 | 峰值 |
|------|------|------|
| TX | 802.11b, 1 Mbps, 21 dBm | 335 mA |
| TX | 802.11g, 54 Mbps, 19 dBm | 285 mA |
| TX | 802.11n HT20 MCS7, 18.5 dBm | 276 mA |
| RX | 802.11b/g/n HT20 | 84 mA |

### Modem-sleep 功耗（CPU 频率相关）

| CPU 频率 | CPU 状态 | 外设时钟全关 | 全开 |
|----------|----------|:---:|:---:|
| 160 MHz | 工作 | 23 mA | 28 mA |
| 160 MHz | 空闲 | 16 mA | 21 mA |
| 80 MHz | 工作 | 17 mA | 22 mA |
| 80 MHz | 空闲 | 13 mA | 18 mA |

---

## 3. Active 与 Modem-sleep

- **Active**：芯片所有模块上电工作，底电流约 20 ~ 40 mA，RF 工作时随 TX/RX 叠加峰值电流
- **Modem-sleep**：CPU 保持运行，当无线模块无数据/不扫描时，**RF 射频自动关闭**。芯片仍可处理应用任务
  - flash 访问会增加功耗（80 Mbit/s SPI 2 线约 +10 mA）

---

## 4. Light-sleep 模式

- 数字外设、CPU、大部分 RAM **时钟门控**，供电电压降低
- RF 模块关闭
- **状态保留**：退出后 CPU/RAM/外设恢复运行，内部状态不丢失
- 底电流 **130 µA**

```c
esp_sleep_enable_timer_wakeup(us);
esp_sleep_enable_gpio_wakeup(...);
esp_light_sleep_start();   // 唤醒后继续运行
```

> Light-sleep 适合需快速恢复且保持连接的场景（如 Wi-Fi Auto Light-sleep）。

---

## 5. Deep-sleep 模式

- CPU、大部分 RAM、所有 APB_CLK 驱动的数字外设**断电**
- 仅保留：**RTC 控制器 + RTC 快速内存（8 KB）**
- 底电流 **5 µA**
- ⚠️ 唤醒后**丢失 CPU 运行上下文**，需重新运行 bootloader（相当于复位）

```c
esp_sleep_enable_timer_wakeup(us);   // 定时唤醒
esp_deep_sleep_start();              // 唤醒后重启
```

主要应用：低功耗传感器，长时间休眠一次醒来采集并上传，再回去休眠。可用 deep-sleep-stub 在 RTC 内存运行快速代码。

---

## 6. 唤醒源

### Light-sleep 唤醒源（ESP32-C3）

| 唤醒源 | 支持 |
|--------|:----:|
| RTC Timer | ✓ |
| GPIO | ✓ |
| UART | ✓ |

### Deep-sleep 唤醒源（ESP32-C3）

| 唤醒源 | 支持 |
|--------|:----:|
| Timer | ✓ |
| GPIO（仅 RTC GPIO） | ✓ |

> ⚠️ ESP32-C3 **不支持** ULP-FSM / ULP-RISC-V / EXT0 / EXT1 / Touch 唤醒（这些是 S2/S3/P4 的特性）。C3 的 Deep-sleep GPIO 唤醒需使用 RTC GPIO。

---

## 7. Wi-Fi 场景低功耗策略

ESP-IDF 提供针对 Wi-Fi 优化的低功耗组合：

| 策略 | 说明 |
|------|------|
| **Modem-sleep** | 保持 Wi-Fi 连接，RF 按 AP 的 DTIM 定期关闭 |
| **DFS + Modem-sleep** | 动态调频 + Modem-sleep，进一步降功耗 |
| **Auto Light-sleep + Wi-Fi** | Wi-Fi 驱动按需唤醒系统，保持连接同时进入 Light-sleep（推荐） |
| **Deep-sleep + Wi-Fi** | 长时间不通信时进入 Deep-sleep，定时醒来重连 |

保持连接时建议启用 Wi-Fi Modem-sleep + Auto Light-sleep：

```c
// menuconfig
// CONFIG_PM_ENABLE=y
// 唤醒锁管理由 Wi-Fi 驱动自动处理
```

---

## 8. BLE 场景低功耗策略

- **时钟源选择**：单 BLE Light-sleep 时，配置**外部 32 kHz 晶振**可将功耗降至 µA 级；无外部 32 kHz 则用内部主晶振作 RTC 时钟，功耗约 mA 级
- **广播间隔优化**：增大广播间隔降低功耗
- **连接间隔协商**：协商较长连接间隔

---

## 9. SPI Flash 深度休眠（DPD）

为降低 Light-sleep 期间 SPI Flash 功耗，ESP-IDF **优先推荐** **Deep Power-Down（DPD）**：

- 启用 `CONFIG_ESP_SLEEP_SET_FLASH_DPD`：flash 供电保持但进入器件内部深度休眠
- 多数 SPI flash 在 DPD 下电流可降至 <1 µA

⚠️ DPD 与 flash 彻底断电（`CONFIG_ESP_SLEEP_POWER_DOWN_FLASH`）**互斥**，二者不可同时启用。使用前确认 flash 型号支持 DPD。

---

## 10. 功耗测量

### 测量建议

- **不建议直接用开发板测模组功耗**（板上电路有额外功耗），需先切断电源电路
- 用裸模组 + ESP-Prog 烧录 `deep_sleep` 示例
- 推荐电流表：**Joulescope**（A~nA 九数量级，1A 时压降仅 25 mV）或 **Nordic PPK2**

### 硬件连接（裸模组）

```
PC ── ESP-Prog ── 模组（VPROG）
                    │
              Joulescope IN+ ← VPROG
                       OUT+ → 模组 3.3V
模组 UART TX/RX、IO9（SPI Boot）、EN、GND → ESP-Prog
```

烧录 `deep_sleep` 示例（每 20 s 唤醒一次），用 Joulescope 软件查看电流波形。

---

## 11. 参考资源

| 资源 | 链接 |
|------|------|
| ESP-IDF 低功耗模式指南（C3） | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/api-guides/low-power-mode/index.html |
| 系统低功耗模式介绍 | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/api-guides/low-power-mode/low-power-mode-soc.html |
| Wi-Fi 场景低功耗 | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/api-guides/low-power-mode/low-power-mode-wifi.html |
| BLE 场景低功耗 | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/api-guides/low-power-mode/low-power-mode-ble.html |
| 睡眠模式 API（sleep_modes） | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/api-reference/system/sleep_modes.html |
| 测量模组功耗 | https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/api-guides/current-consumption-measurement-modules.html |
| ESP 低功耗入门（techpedia） | https://docs.espressif.com/projects/esp-techpedia/zh_CN/latest/esp-friends/advanced-development/performance/lowpower/lowpower-fundamental.html |
| deep_sleep 示例 | https://github.com/espressif/esp-idf/tree/master/examples/system/deep_sleep |
| light_sleep 示例 | https://github.com/espressif/esp-idf/tree/master/examples/system/light_sleep |
| Joulescope | https://www.joulescope.com/ |
| Nordic PPK2 | https://www.nordicsemi.com/Products/Development-hardware/Power-Profiler-Kit-2 |

---

> **文档总结**：ESP32-C3 支持四种功耗模式——Active（RF 峰值 335 mA）、Modem-sleep（13~28 mA，CPU 运行+RF 关）、Light-sleep（130 µA，状态保留）、Deep-sleep（5 µA，仅 RTC）。Wi-Fi 场景推荐 Modem-sleep + DFS + Auto Light-sleep 保持连接又省电；BLE 场景配外部 32 kHz 晶振可把 Light-sleep 降至 µA 级。C3 的 Deep-sleep 唤醒源为 Timer 与 RTC GPIO（不支持 ULP/EXT0/EXT1/Touch）。Light-sleep 可启用 SPI Flash DPD 进一步降功耗。测量用裸模组 + ESP-Prog + Joulescope/PPK2。是电池供电 IoT 的低功耗优选。
