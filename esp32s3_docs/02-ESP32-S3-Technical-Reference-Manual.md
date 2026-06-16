# ESP32-S3 技术参考手册 (Technical Reference Manual)

> 来源: https://documentation.espressif.com/esp32-s3_technical_reference_manual_en.html
> 版本: v1.8

---

## 1. 目录概览

ESP32-S3 Technical Reference Manual 包含以下主要章节：

| 章节 | 内容 |
|------|------|
| Chapter 1 | Xtensa 处理器 (CPU) |
| Chapter 2 | ULP 低功耗协处理器 |
| Chapter 3 | GDMA 控制器 |
| Chapter 4 | 存储系统 (Flash/PSRAM) |
| Chapter 5 | eFuse 控制器 |
| Chapter 6 | IO MUX 和 GPIO 矩阵 |
| Chapter 7 | 时钟和复位 |
| Chapter 8 | 启动模式和 Strapping 引脚 |
| Chapter 9 | 中断矩阵 |
| Chapter 10 | 电源管理 (RTC) |
| Chapter 11 | Timer Group |
| Chapter 12 | 系统定时器 |
| Chapter 13 | UART 控制器 |
| Chapter 14 | I2C 控制器 |
| Chapter 15 | I2S 控制器 |
| Chapter 16 | SPI 控制器 |
| Chapter 17 | 系统寄存器 |
| Chapter 18 | ADC 控制器 |
| Chapter 19 | 温度传感器 |
| Chapter 20 | 触摸传感器 |
| Chapter 21 | LED PWM 控制器 |
| Chapter 22 | MCPWM 控制器 |
| Chapter 23 | USB 串口/JTAG 控制器 |
| Chapter 24 | USB OTG |
| Chapter 25 | TWAI 控制器 |
| Chapter 26 | SD/MMC 主机控制器 |
| Chapter 27 | LCD 和摄像头控制器 |
| Chapter 28 | RMT 控制器 |
| Chapter 29 | 脉冲计数器 |

---

## 2. Xtensa 处理器 (Chapter 1)

### 2.1 处理器概述

ESP32-S3 搭载 Xtensa LX7 双核处理器，运行频率最高 240 MHz。

### 2.2 扩展指令集

ESP32-S3 在标准 Xtensa 指令集基础上扩展了向量指令，用于 AI 加速：

#### 向量指令分类

| 指令类型 | 描述 |
|---------|------|
| 读取指令 (Read) | 向量数据加载 |
| 写入指令 (Write) | 向量数据存储 |
| 数据交换指令 (Data Exchange) | 向量数据交换 |
| 向量加法 (Vector Addition) | 向量加法运算 |
| 向量乘法 (Vector Multiplication) | 向量乘法运算 |
| 向量复数乘法 (Vector Complex Multiplication) | 复数乘法 |
| 向量乘累加 (Vector MAC) | 乘累加运算 |
| 向量-标量乘累加 (Vector-Scalar MAC) | 标量乘累加 |
| 比较指令 (Comparison) | 向量比较 |
| 位逻辑指令 (Bitwise Logical) | 位运算 |
| 移位指令 (Shift) | 移位运算 |
| 蝶形运算 (Butterfly) | FFT 蝶形运算 |
| 位反转 (Bit Reverse) | 位反转 |
| 实数 FFT (Real FFT) | 实数 FFT 运算 |
| GPIO 控制 | GPIO 操作指令 |

### 2.3 流水线

Xtensa 处理器采用五级流水线：
1. **IF** (Instruction Fetch) - 取指
2. **RF** (Register Fetch) - 寄存器读取
3. **EX** (Execute) - 执行
4. **MEM** (Memory) - 访存
5. **WB** (Write Back) - 写回

扩展指令流水线阶段：
- 向量指令通常需要多个时钟周期完成
- 流水线深度取决于指令复杂度

---

## 3. ULP 低功耗协处理器 (Chapter 2)

### 3.1 概述

ESP32-S3 包含两个 ULP (Ultra Low Power) 协处理器：
- **ULP-RISC-V**: 基于 RISC-V 架构的低功耗协处理器
- **ULP-FSM**: 基于有限状态机的低功耗协处理器

### 3.2 ULP-RISC-V 协处理器

#### 特性
- RISC-V RV32I 指令集
- 独立的指令和数据存储器
- 支持 ADC 采集、GPIO 操作
- 可在 Deep-sleep 模式下运行
- 支持中断唤醒主 CPU

#### ALU 操作

| 操作类型 | 描述 |
|---------|------|
| 寄存器间操作 | ADD, SUB, AND, OR, XOR 等 |
| 立即数操作 | ADDI, ANDI, ORI 等 |
| 阶梯计数器操作 | 阶梯计数器相关运算 |

#### 数据存储模式
- **自动存储模式**: 自动将 ADC 结果存储到指定地址
- **手动存储模式**: 软件控制数据存储

#### 中断源

| 中断源 | 描述 |
|--------|------|
| SARADC1 | ADC1 采样完成 |
| SARADC2 | ADC2 采样完成 |
| TOUCH | 触摸传感器中断 |
| RTC Timer | RTC 定时器中断 |
| GPIO | GPIO 状态变化 |

### 3.3 ULP-FSM 协处理器

- 基于有限状态机架构
- 更低功耗，更简单的程序模型
- 适合简单的传感器采集任务

---

## 4. GDMA 控制器 (Chapter 3)

### 4.1 概述

GDMA (General DMA) 控制器提供高速数据传输能力，支持外设与存储器之间的数据搬运。

### 4.2 特性
- 支持多个 DMA 通道
- 支持链式描述符 (Linked List Descriptor)
- 支持突发传输 (Burst Transfer)
- 可配置优先级

### 4.3 外设 DMA 通道分配

| 外设 | DMA 通道 |
|------|---------|
| SPI2 | 专用通道 |
| SPI3 | 专用通道 |
| I2S0 | 专用通道 |
| I2S1 | 专用通道 |
| LCD_CAM | 专用通道 |
| AES | 专用通道 |
| SHA | 专用通道 |
| ADC | 专用通道 |

---

## 5. 存储系统 (Chapter 4)

### 5.1 内部存储器地址映射

| 存储器 | 地址范围 | 大小 | 描述 |
|--------|---------|------|------|
| SRAM0 | 0x40370000 - 0x40377FFF | 32 KB | 可配置为 I-Cache 缓冲区 |
| SRAM1 | 0x3FC88000 - 0x3FCBFFFF | 224 KB | 指令和数据总线均可访问 |
| SRAM2 | 0x3FC00000 - 0x3FC87FFF | 544 KB | 可配置为 D-Cache |
| RTC FAST | 0x600FE000 - 0x600FFFFF | 8 KB | RTC 快速存储器 |
| RTC SLOW | 0x50000000 - 0x50001FFF | 8 KB | RTC 慢速存储器 |
| ROM | 0x40000000 - 0x4006FFFF | 448 KB | 只读存储器 |

### 5.2 外部存储器地址映射

| 存储器 | 地址范围 | 描述 |
|--------|---------|------|
| Flash (I-Cache) | 0x42000000 - 0x42FFFFFF | 通过指令缓存访问 Flash |
| Flash (D-Cache) | 0x3C000000 - 0x3CFFFFFF | 通过数据缓存访问 Flash |
| PSRAM (I-Cache) | 0x42000000 - 0x43FFFFFF | 通过指令缓存访问 PSRAM |
| PSRAM (D-Cache) | 0x3C000000 - 0x3DFFFFFF | 通过数据缓存访问 PSRAM |

### 5.3 模块/外设地址映射

外设基地址：
- 0x3FF40000 - 0x3FF7FFFF: 外设寄存器区域
- 每个外设占用 4 KB 地址空间

---

## 6. IO MUX 和 GPIO 矩阵 (Chapter 6)

### 6.1 概述

ESP32-S3 的 GPIO 系统包含：
- **IO MUX**: 提供固定的引脚功能映射，信号完整性最好
- **GPIO 矩阵**: 提供灵活的引脚功能映射，可将任意 GPIO 映射到任意外设

### 6.2 Light-sleep 模式 IO MUX 控制

在 Light-sleep 模式下，可通过配置位控制 IO MUX 引脚状态。

### 6.3 GPIO 矩阵外设信号

ESP32-S3 通过 GPIO 矩阵支持以下外设信号映射：

| 外设 | 信号数量 | 描述 |
|------|---------|------|
| UART0 | TX, RX, CTS, RTS | 串口 0 |
| UART1 | TX, RX, CTS, RTS | 串口 1 |
| UART2 | TX, RX, CTS, RTS | 串口 2 |
| I2C0 | SDA, SCL | I2C 总线 0 |
| I2C1 | SDA, SCL | I2C 总线 1 |
| SPI2 | CS0, CS1, CLK, D, Q, WP, HD | SPI 2 |
| SPI3 | CS0, CS1, CLK, D, Q, WP, HD | SPI 3 |
| I2S0 | BCK, WS, DATA_IN, DATA_OUT | I2S 0 |
| I2S1 | BCK, WS, DATA_IN, DATA_OUT | I2S 1 |
| LEDC | LSIG0~LSIG7 | LED PWM |
| RMT | TX0~TX3, RX0~RX3 | 远程控制 |
| MCPWM | PWM0A, PWM0B, PWM1A, PWM1B | 电机控制 PWM |
| USB | D+, D- | USB OTG |

### 6.4 IO MUX 引脚功能表

IO MUX 引脚具有固定的默认功能，详见完整引脚功能表。

### 6.5 RTC IO MUX 引脚功能

RTC IO MUX 引脚支持：
- 数字 GPIO 功能
- RTC GPIO 功能
- 模拟功能 (ADC, Touch)
- 32K 晶振功能

---

## 7. 时钟和复位 (Chapter 7)

### 7.1 复位源

| 复位源 | 描述 |
|--------|------|
| 上电复位 (POR) | 电源上电时的复位 |
| 看门狗复位 (WDT) | 看门狗超时复位 |
| 软件复位 | 软件触发的复位 |
| 欠压复位 (BOD) | 电压过低时的复位 |
| 外部复位 | CHIP_PU 引脚复位 |

### 7.2 时钟源

#### CPU 时钟源

| 时钟源 | 频率 | 描述 |
|--------|------|------|
| PLL_F160M | 160 MHz | PLL 160M 时钟 |
| PLL_F240M | 240 MHz | PLL 240M 时钟 |
| XTAL | 40 MHz | 外部晶振 |
| RC_FAST | ~8.5 MHz | 内部快速 RC 振荡器 |
| RC_SLOW | ~136 kHz | 内部慢速 RC 振荡器 |

#### CPU 时钟频率配置

| 分频系数 | 时钟源 | CPU 频率 |
|---------|--------|---------|
| 1 | PLL_F240M | 240 MHz |
| 2 | PLL_F240M | 120 MHz |
| 2 | PLL_F160M | 80 MHz |
| 3 | PLL_F160M | 53.3 MHz |
| 4 | PLL_F160M | 40 MHz |

#### 外设时钟

| 外设 | 时钟源 | 频率 |
|------|--------|------|
| APB_CLK | PLL_F80M | 80 MHz |
| UART_CLK | PLL_F80M | 80 MHz |
| SPI_CLK | PLL_F80M | 80 MHz |
| I2C_CLK | APB_CLK | 80 MHz |
| I2S_CLK | PLL_F160M | 可配置 |
| ADC_CLK | RC_FAST | ~8.5 MHz |
| RTC_SLOW_CLK | RC_SLOW | ~136 kHz |
| RTC_FAST_CLK | RC_FAST | ~8.5 MHz |

---

## 8. 启动模式和 Strapping 引脚 (Chapter 8)

### 8.1 Strapping 引脚默认配置

| 引脚 | 默认状态 | 描述 |
|------|---------|------|
| GPIO0 | 上拉 | 启动模式选择 |
| GPIO46 | 下拉 | 启动模式选择 |

### 8.2 启动模式控制

| GPIO0 | GPIO46 | 启动模式 |
|-------|--------|---------|
| 1 | X | 从 Flash 启动 |
| 0 | 0 | 下载模式 (UART) |
| 0 | 1 | 下载模式 (SPI) |

### 8.3 ROM 消息打印控制

ROM 消息可通过以下方式输出：
- UART0
- USB Serial/JTAG 控制器

### 8.4 JTAG 信号源控制

JTAG 信号可通过以下方式提供：
- 内置 USB Serial/JTAG 控制器
- 外部 JTAG 调试器

---

## 9. 中断矩阵 (Chapter 9)

### 9.1 概述

ESP32-S3 的中断矩阵允许将任意外设中断源映射到任意 CPU 中断线。

### 9.2 CPU 外设中断配置

| 寄存器 | 描述 |
|--------|------|
| 中断源选择寄存器 | 选择映射到每个 CPU 中断的外设中断源 |
| 中断使能寄存器 | 使能/禁用各中断 |
| 中断状态寄存器 | 读取中断状态 |

### 9.3 CPU 中断列表

ESP32-S3 支持 32 个可屏蔽中断 (CPU 中断 0~31)。

---

## 10. 电源管理 (Chapter 10)

### 10.1 低功耗时钟

| 时钟 | 频率 | 描述 |
|------|------|------|
| RTC_SLOW_CLK | ~136 kHz | RTC 慢速时钟 |
| RTC_FAST_CLK | ~8.5 MHz | RTC 快速时钟 |

### 10.2 RTC 定时器触发条件

| 触发源 | 描述 |
|--------|------|
| 软件触发 | 软件设置触发位 |
| 外部唤醒 | GPIO 唤醒 |
| 定时器唤醒 | RTC 定时器超时 |
| Touch 唤醒 | 触摸传感器唤醒 |

### 10.3 电源模式

| 模式 | CPU | Wi-Fi | BT | 晶振 | RTC | 功耗 |
|------|-----|-------|-----|------|-----|------|
| Active | 运行 | 可开启 | 可开启 | 开启 | 开启 | ~50 mA |
| Modem-sleep | 运行 | 关闭 | 关闭 | 开启 | 开启 | ~15 mA |
| Light-sleep | 暂停 | 关闭 | 关闭 | 关闭 | 开启 | ~0.8 mA |
| Deep-sleep | 关闭 | 关闭 | 关闭 | 关闭 | 开启 | ~10 μA |
| Hibernation | 关闭 | 关闭 | 关闭 | 关闭 | 部分关闭 | ~5 μA |

### 10.4 唤醒源

| 唤醒源 | Light-sleep | Deep-sleep | Hibernation |
|--------|------------|------------|-------------|
| RTC 定时器 | ✓ | ✓ | ✓ |
| GPIO 唤醒 | ✓ | ✓ | ✓ |
| Touch 唤醒 | ✓ | ✓ | ✗ |
| UART 唤醒 | ✓ | ✗ | ✗ |
| USB 唤醒 | ✓ | ✗ | ✗ |

---

## 11. 系统寄存器 (Chapter 17)

### 11.1 概述

ESP32-S3 系统寄存器用于控制：
- 系统和存储器配置
- 时钟管理
- 软件中断
- 低功耗管理
- 外设时钟门控和复位
- CPU 控制

### 11.2 外设时钟门控和复位

ESP32-S3 采用低功耗设计，部分外设时钟默认关闭。使用外设前必须：
1. 使能外设时钟
2. 释放外设复位状态

#### 外设时钟和复位位映射

| 外设 | 时钟使能位 | 复位控制位 |
|------|-----------|-----------|
| EDMA | SYSTEM_EDMA_CLK_ON | SYSTEM_EDMA_RESET |
| DCACHE | SYSTEM_DCACHE_CLK_ON | SYSTEM_DCACHE_RESET |
| ICACHE | SYSTEM_ICACHE_CLK_ON | SYSTEM_ICACHE_RESET |
| Timer Group0 | SYSTEM_TIMERGROUP_CLK_EN | SYSTEM_TIMERGROUP_RST |
| Timer Group1 | SYSTEM_TIMERGROUP1_CLK_EN | SYSTEM_TIMERGROUP1_RST |
| System Timer | SYSTEM_SYSTIMER_CLK_EN | SYSTEM_SYSTIMER_RST |
| UART0 | SYSTEM_UART_CLK_EN | SYSTEM_UART_RST |
| UART1 | SYSTEM_UART1_CLK_EN | SYSTEM_UART1_RST |
| SPI0, SPI1 | SYSTEM_SPI01_CLK_EN | SYSTEM_SPI01_RST |
| SPI2 | SYSTEM_SPI2_CLK_EN | SYSTEM_SPI2_RST |
| SPI3 | SYSTEM_SPI3_CLK_EN | SYSTEM_SPI3_RST |
| I2C0 | SYSTEM_I2C_EXT0_CLK_EN | SYSTEM_I2C_EXT0_RST |
| I2C1 | SYSTEM_I2C_EXT1_CLK_EN | SYSTEM_I2C_EXT1_RST |
| I2S0 | SYSTEM_I2S0_CLK_EN | SYSTEM_I2S0_RST |
| I2S1 | SYSTEM_I2S1_CLK_EN | SYSTEM_I2S1_RST |
| TWAI | SYSTEM_CAN_CLK_EN | SYSTEM_CAN_RST |
| USB | SYSTEM_USB_CLK_EN | SYSTEM_USB_RST |
| RMT | SYSTEM_RMT_CLK_EN | SYSTEM_RMT_RST |
| PCNT | SYSTEM_PCNT_CLK_EN | SYSTEM_PCNT_RST |
| PWM0 | SYSTEM_PWM0_CLK_EN | SYSTEM_PWM0_RST |
| PWM1 | SYSTEM_PWM1_CLK_EN | SYSTEM_PWM1_RST |
| LED_PWM | SYSTEM_LEDC_CLK_EN | SYSTEM_LEDC_RST |
| ADC | SYSTEM_APB_SARADC_CLK_EN | SYSTEM_APB_SARADC_RST |
| USB_DEVICE | SYSTEM_USB_DEVICE_CLK_EN | SYSTEM_USB_DEVICE_RST |
| UART2 | SYSTEM_UART2_CLK_EN | SYSTEM_UART2_RST |
| LCD_CAM | SYSTEM_LCD_CAM_CLK_EN | SYSTEM_LCD_CAM_RST |
| SDIO_HOST | SYSTEM_SDIO_HOST_CLK_EN | SYSTEM_SDIO_HOST_RST |
| DMA | SYSTEM_DMA_CLK_EN | SYSTEM_DMA_RST |
| HMAC | SYSTEM_CRYPTO_HMAC_CLK_EN | SYSTEM_CRYPTO_HMAC_RST |
| Digital Signature | SYSTEM_CRYPTO_DS_CLK_EN | SYSTEM_CRYPTO_DS_RST |
| RSA | SYSTEM_CRYPTO_RSA_CLK_EN | SYSTEM_CRYPTO_RSA_RST |
| SHA | SYSTEM_CRYPTO_SHA_CLK_EN | SYSTEM_CRYPTO_SHA_RST |
| AES | SYSTEM_CRYPTO_AES_CLK_EN | SYSTEM_CRYPTO_AES_RST |

**注意事项:**
1. 时钟使能寄存器设为 1 使能时钟，设为 0 关闭时钟
2. 复位寄存器设为 1 复位外设，设为 0 释放复位
3. 复位寄存器不会被硬件自动清除，需要软件手动清除
4. DMA 时钟在外设通信时也需要使能
5. HMAC 复位也会复位 SHA 加速器
6. Digital Signature 复位也会复位 AES、SHA 和 RSA 加速器

---

## 12. UART 控制器 (Chapter 13)

### 12.1 概述

ESP32-S3 有 3 个 UART 控制器 (UART0, UART1, UART2)。

### 12.2 特性
- 3 个可分频时钟源
- 可编程波特率
- 1024×8-bit RAM，由 3 个 UART 的 TX/RX FIFO 共享
- 全双工异步通信
- 自动波特率检测
- 数据位 5~8 位
- 停止位 1, 1.5, 2, 3 位
- 奇偶校验位
- AT_CMD 特殊字符检测
- RS485 协议
- IrDA 协议
- GDMA 高速数据传输
- UART 唤醒源
- 软件和硬件流控

---

## 13. I2C 控制器 (Chapter 14)

### 13.1 概述

ESP32-S3 有 2 个 I2C 总线接口，支持主/从模式。

### 13.2 特性
- 标准模式 (100 kbit/s)
- 快速模式 (400 kbit/s)
- 最高 800 kbit/s (受 SCL/SDA 上拉强度限制)
- 7 位和 10 位寻址模式
- 双寻址模式
- 硬件命令抽象层简化使用

---

## 14. SPI 控制器 (Chapter 16)

### 14.1 概述

ESP32-S3 有 4 个 SPI 接口：
- **SPI0**: GDMA 和 Cache 访问 Flash/PSRAM
- **SPI1**: CPU 访问 Flash/PSRAM
- **SPI2**: 通用 SPI 主/从控制器
- **SPI3**: 通用 SPI 主/从控制器

### 14.2 SPI0/SPI1 特性
- 支持 Single, Dual, Quad, Octal SPI, QPI, OPI 模式
- 用于 Flash 和 PSRAM 访问

### 14.3 SPI2/SPI3 特性
- 支持主/从模式
- 可配置时钟频率和数据位序 (MSB/LSB)
- 主模式: 2 线全双工，最高 80 MHz
- 从模式: 2 线全双工，最高 60 MHz

---

## 15. ADC 控制器 (Chapter 18)

### 15.1 概述

ESP32-S3 包含 2 个 SAR ADC：
- **ADC1**: 10 个通道 (CH0~CH9)
- **ADC2**: 10 个通道 (CH0~CH9)

### 15.2 特性
- 12 位分辨率
- 支持多种衰减配置
- 支持 DMA 传输
- 支持 ULP 协处理器访问

---

## 16. 触摸传感器 (Chapter 20)

### 16.1 概述

ESP32-S3 支持 14 个电容触摸引脚 (GPIO0~GPIO14)。

### 16.2 特性
- 支持触摸检测
- 支持触摸唤醒
- 可配置灵敏度
- 支持 ULP 协处理器访问

---

## 17. LED PWM 控制器 (Chapter 21)

### 17.1 概述

LED PWM 控制器 (LEDC) 用于 LED 亮度控制和信号生成。

### 17.2 特性
- 8 个通道 (高速通道 4 个 + 低速通道 4 个)
- 1~20 位分辨率
- 可配置频率
- 支持渐变 (Fade) 功能
- 可用作通用定时器

---

## 18. TWAI 控制器 (Chapter 25)

### 18.1 概述

TWAI (Two-Wire Automotive Interface) 控制器兼容 ISO 11898-1 (CAN 2.0)。

### 18.2 特性
- 支持标准帧和扩展帧
- 支持数据帧和远程帧
- 可配置波特率
- 支持错误检测和处理

---

## 19. USB OTG (Chapter 24)

### 19.1 概述

ESP32-S3 集成全速 USB 2.0 OTG 控制器。

### 19.2 特性
- 支持 USB Host 和 Device 模式
- 全速 (12 Mbps) 和低速 (1.5 Mbps) 模式
- 支持 USB CDC, HID, MSC 等类
- 支持 USB Serial/JTAG

---

## 20. 相关文档

- [ESP32-S3 Datasheet](https://documentation.espressif.com/esp32-s3_datasheet_en.html)
- [ESP32-S3 Programming Guide](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/api-reference/index.html)
- [ESP-IDF 编程指南](https://docs.espressif.com/projects/esp-idf/en/latest/)
