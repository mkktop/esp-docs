# ESP-Insights 框架详解 — 远程诊断与可观测性框架

> 文档编号：32
> 适用芯片：ESP32 全系列（ESP32 / ESP32-S2 / ESP32-S3 / ESP32-C3 / ESP32-C5 / ESP32-C6 / ESP32-H2 等）
> 适用场景：现场部署设备的远程健康监控、崩溃诊断、日志上报、固件质量分析
> 来源：乐鑫官方 RainMaker ESP-Insights 文档、GitHub 仓库、ESP-BOX / BOX-3 新闻、ESP-IDF 库与框架页

---

## 目录

1. [ESP-Insights 概述](#1-esp-insights-概述)
2. [核心能力](#2-核心能力)
3. [与 ESP RainMaker 的关系](#3-与-esp-rainmaker-的关系)
4. [工作原理与数据流](#4-工作原理与数据流)
5. [如何启用 ESP-Insights](#5-如何启用-esp-insights)
6. [固件诊断包（Insights Firmware Package）](#6-固件诊断包insights-firmware-package)
7. [应用场景](#7-应用场景)
8. [参考资源](#8-参考资源)

---

## 1. ESP-Insights 概述

### 1.1 什么是 ESP-Insights

ESP-Insights 是乐鑫提供的**远程诊断（remote diagnostics）与可观测性（observability）框架**，允许用户远程监控现场部署 ESP 设备的健康状况。它解决了嵌入式设备出厂后"黑盒难调试"的痛点——开发者无需回收设备，即可在云端看到设备运行时的崩溃、错误、关键日志与系统指标。

ESP-Insights 充分利用 ESP RainMaker 平台的设备认证与云传输能力，作为 RainMaker 生态的一部分提供，是一个**可选服务**，用于进一步增强固件开发质量、产品维护与数据分析。

### 1.2 解决的核心痛点

| 痛点 | ESP-Insights 提供的能力 |
|------|------------------------|
| 现场设备崩溃无法复现 | 上报崩溃栈、寄存器、回溯（backtrace） |
| 日志只能本地串口看 | 关键日志/错误周期性上报云端 |
| 难以批量了解设备健康状况 | 仪表盘聚合查看全量设备健康状态 |
| 问题定位缺乏数据 | 上报系统指标、诊断信息 |

---

## 2. 核心能力

| 能力 | 说明 |
|------|------|
| **崩溃诊断** | 自动捕获 panic / exception / watchdog 触发，上报崩溃回溯与寄存器现场 |
| **错误与日志上报** | 通过 diagnostics 机制将关键日志、错误周期性上报云端 |
| **系统指标** | 内存（堆）、任务、Wi-Fi 等运行时指标采集 |
| **云端仪表盘** | 在 ESP Insights Dashboard 远程查看设备健康与诊断数据 |
| **与 RainMaker 集成** | 复用 RainMaker 设备认证与安全云传输，所有 RainMaker 示例默认集成 |

---

## 3. 与 ESP RainMaker 的关系

ESP-Insights **不是独立的云**，而是构建在 ESP RainMaker 之上：

```
┌──────────────────────────────────────┐
│   ESP Insights Dashboard（仪表盘）    │
│   insights.espressif.com             │
└──────────────────▲───────────────────┘
                   │ 诊断数据上报
┌──────────────────┴───────────────────┐
│   ESP RainMaker Cloud（云传输/认证）   │
└──────────────────▲───────────────────┘
                   │ 加密通道
┌──────────────────┴───────────────────┐
│   设备端：app_insights 组件           │
│   + esp-insights components           │
│   （ESP32-S3 等设备）                  │
└──────────────────────────────────────┘
```

- ESP-Insights 完全复用 RainMaker 的设备认证（claiming）与安全云传输
- 已集成于所有 ESP RainMaker 示例中，默认存在但默认关闭
- ESP-Matter SDK 也集成了 ESP Insights 用于远程诊断

---

## 4. 工作原理与数据流

```
设备运行
   │
   ├─ 崩溃/异常 ──> 自动捕获 backtrace + 寄存器
   ├─ DIAG 诊断日志 ──> 缓冲存储
   └─ 系统指标 ──> 周期采集
                    │
                    ▼
          打包为 Insights 固件诊断包（.zip）
                    │
                    ▼
          经 RainMaker 安全通道上报云端
                    │
                    ▼
          ESP Insights Dashboard 可视化展示
```

设备端的关键日志通过 `DIAG` 机制（而非全部 `printf`）筛选上报，避免上传海量无用日志，节省带宽与云端存储。

---

## 5. 如何启用 ESP-Insights

### 5.1 前置条件

确保 `esp-rainmaker` 仓库为最新：

```bash
cd /path/to/esp-rainmaker
git pull origin master
git submodule update --init --recursive
```

这会拉取 `esp-rainmaker/components` 下的 `esp-insights` 子模块，并为所有默认示例配置好 ESP Insights 支持。

### 5.2 修改 CMakeLists.txt（自定义项目）

在自定义项目中，将 esp-insights 组件目录加入 `EXTRA_COMPONENT_DIRS`：

```cmake
# 原：
set(EXTRA_COMPONENT_DIRS ${CMAKE_CURRENT_LIST_DIR}/../../components ${CMAKE_CURRENT_LIST_DIR}/../common)
# 改为：
set(EXTRA_COMPONENT_DIRS ${CMAKE_CURRENT_LIST_DIR}/../../components ${CMAKE_CURRENT_LIST_DIR}/../common ${CMAKE_CURRENT_LIST_DIR}/../../components/esp-insights/components)
```

> 更推荐的做法：直接参考任意 RainMaker 示例的最新 CMakeLists.txt，复制相关片段。

### 5.3 启用配置与代码调用

ESP Insights 逻辑由 `examples/common/app_insights` 组件抽象，默认关闭。启用步骤：

```bash
# menuconfig 启用
idf.py menuconfig
# Component config → ESP Insights → Enable ESP Insights
# 设置 CONFIG_ESP_INSIGHTS_ENABLED=y
```

在代码中，于 `esp_rmaker_start()` 之前调用：

```c
#include "app_insights.h"

// 在 esp_rmaker_start() 之前
app_insights_enable();
```

> 所有 RainMaker 示例已内置上述调用，只需在 menuconfig 中开启 `CONFIG_ESP_INSIGHTS_ENABLED`。

### 5.4 编译烧录

```bash
idf.py build flash monitor
```

构建日志中会出现生成 Insights 固件诊断包的提示：

```
===================== Generating insights firmware package build/<project>-v1.0.zip ======================
```

---

## 6. 固件诊断包（Insights Firmware Package）

构建生成的 `build/<project>-v1.0.zip` 即为固件诊断包，包含：

| 文件 | 说明 |
|------|------|
| `<project>.bin` | 应用固件 |
| `sdkconfig` | 编译配置 |
| `partition_table/partition-table.bin` | 分区表 |
| `bootloader/bootloader.bin` | bootloader |
| `partitions.csv` | 分区 CSV |
| `<project>.elf` | ELF 符号文件（用于解析崩溃栈回溯） |
| `project_description.json` | 工程描述 |

该 zip 包会在云端用于**解析崩溃回溯地址 → 源码位置**，是远程诊断的关键。

---

## 7. 应用场景

| 场景 | 说明 |
|------|------|
| 现场设备售后维护 | 远程获取崩溃栈，无需回收设备即可定位 bug |
| 大规模设备质量监控 | 仪表盘聚合查看全量设备健康状态、故障率 |
| 固件 OTA 后回归 | 对比 OTA 前后崩溃率与诊断数据，评估固件质量 |
| 墨盒级调试 | 结合 DIAG 日志快速定位偶发性问题 |
| Matter / RainMaker 设备运维 | ESP-Matter、BOX-3、VoCat、DualKey 等基于 RainMaker 的设备天然支持 |

---

## 8. 参考资源

| 资源 | 链接 |
|------|------|
| ESP-Insights GitHub | https://github.com/espressif/esp-insights |
| ESP Insights Dashboard | https://insights.espressif.com |
| RainMaker ESP-Insights 集成文档 | https://docs.rainmaker.espressif.com/docs/product_overview/integrated_solutions/esp-insights/ |
| ESP RainMaker GitHub | https://github.com/espressif/esp-rainmaker |
| ESP-Matter GitHub | https://github.com/espressif/esp-matter |
| ESP RainMaker 官网 | https://rainmaker.espressif.com |

---

> **文档总结**：ESP-Insights 是乐鑫的远程诊断与可观测性框架，构建于 ESP RainMaker 云平台之上，复用其设备认证与安全传输通道。它能自动捕获现场设备的崩溃回溯、关键诊断日志与系统指标，打包成固件诊断包上报云端，在 ESP Insights Dashboard 可视化展示。默认集成于所有 RainMaker 示例（menuconfig 中 `CONFIG_ESP_INSIGHTS_ENABLED=y` 即可启用），并已被 ESP-Matter、ESP32-S3-BOX-3、ESP-VoCat、ESP-DualKey 等采用，是 IoT 设备量产售后与远程运维的关键基础设施。
