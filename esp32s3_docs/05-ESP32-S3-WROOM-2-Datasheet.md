# ESP32-S3-WROOM-2 技术规格书 (Datasheet)

> 来源: https://documentation.espressif.com/esp32-s3-wroom-2_datasheet_cn.html

---

## 1. 模组概述

ESP32-S3-WROOM-2 是通用型 Wi-Fi + 低功耗蓝牙 MCU 模组，具有丰富的外设接口，强大的神经网络运算能力和信号处理能力，是专为人工智能和 AIoT 市场打造的一款模组，适用于多种应用场景，例如唤醒词检测和语音命令识别、人脸检测和识别、智能家居、智能家电、智能控制面板、智能扬声器等。

ESP32-S3-WROOM-2 采用 PCB 板载天线，模组配置 ESP32-S3R8V 或 ESP32-S3R16V 芯片，可选 16/32 MB flash、8/16 MB PSRAM。请注意，若开启 PSRAM ECC 功能，模组最大环境温度可以提高到 85 °C，但是 PSRAM 的可用容量将减少 1/16。

### 1.1 主要特性

- 2.4 GHz Wi-Fi (802.11b/g/n) + 蓝牙 5 (LE) 模组
- 内置 ESP32-S3 系列芯片，Xtensa 双核 32 位 LX7 处理器
- Flash 最大可选 32 MB (Octal SPI)，PSRAM 最大可选 16 MB (Octal SPI)
- 33 个 GPIO，丰富的外设
- 板载 PCB 天线

**CPU 和片上存储器**

- 内置 ESP32-S3 芯片，Xtensa 双核 32 位 LX7 微处理器（支持单精度浮点运算单元），支持高达 240 MHz 的时钟频率
- 384 KB ROM
- 512 KB SRAM
- 16 KB RTC SRAM
- 最大 16 MB PSRAM

**Wi-Fi**

- 802.11b/g/n
- 802.11n 模式下数据速率高达 150 Mbps
- 帧聚合 (TX/RX A-MPDU, TX/RX A-MSDU)
- 0.4 us 保护间隔
- 工作信道中心频率范围：2412 ~ 2484 MHz

**蓝牙**

- 低功耗蓝牙 (Bluetooth LE)：Bluetooth 5、Bluetooth mesh
- 速率支持 125 Kbps、500 Kbps、1 Mbps、2 Mbps
- 广播扩展 (Advertising Extensions)
- 多广播 (Multiple Advertisement Sets)
- 信道选择 (Channel Selection Algorithm #2)
- Wi-Fi 与蓝牙共存，共用同一个天线

**外设**

- 33 个 GPIO（其中 4 个作为 strapping 管脚）
- SPI、LCD 接口、Camera 接口、UART、I2C、I2S、红外遥控、脉冲计数器、LED PWM
- 全速 USB 2.0 OTG、USB 串口/JTAG 控制器
- MCPWM、SD/MMC 主机控制器、GDMA
- TWAI 控制器（兼容 ISO 11898-1）
- ADC、触摸传感器、温度传感器、定时器和看门狗

**模组集成元件**

- 40 MHz 集成晶振
- 最大 32 MB Octal SPI flash

**天线选型**

- 板载 PCB 天线

**工作条件**

- 工作电压/供电电压：3.0 ~ 3.6 V
- 工作环境温度：-40 ~ 65 °C

**认证**

- RF 认证：FCC / CE / SRRC / MIC 等（见证书）
- 环保认证：RoHS/REACH
- 测试：HTOL / HTSL / uHAST / TCT / ESD

### 1.2 应用场景

- 智能家居
- 工业自动化
- 医疗保健
- 消费电子产品
- 智慧农业
- POS 机
- 服务机器人
- 音频设备
- 通用低功耗 IoT 传感器集线器
- 通用低功耗 IoT 数据记录器
- 摄像头视频流传输
- USB 设备
- 语音识别
- 图像识别
- Wi-Fi + 蓝牙网卡
- 触摸和接近感应

---

## 2. 功能框图

ESP32-S3-WROOM-2 模组的功能框图如下所示：

```
                         +---------------------------+
                         |      ESP32-S3-WROOM-2     |
                         |                           |
   3V3  ----[Power]----->|  3V3                      |
                         |                           |
   EN   ----[Enable]---->|  EN (CHIP_PU)             |
                         |                           |
                         |  +---------------------+  |
   40MHz Crystal ------->|  |   ESP32-S3R8V /     |  |
                         |  |   ESP32-S3R16V      |  |
                         |  |                     |  |
                         |  |  Xtensa Dual-Core   |  |
                         |  |  LX7 @ 240MHz       |  |
                         |  +---------------------+  |
                         |         |    |            |
                         |    +----+    +----+       |
                         |    |              |       |
                         | +--+--+     +-----+-----+|
                         | |OSPI |     | OSPI      ||
                         | |Flash|     | PSRAM     ||
                         | |16/32|     | 8/16 MB   ||
                         | | MB  |     |           ||
                         | +-----+     +-----------+|
                         |                           |
                         |  RF Matching --> Antenna  |
                         |       (PCB Antenna)       |
                         |                           |
   GPIO [33 pins] <----->|  GPIO0~GPIO48             |
                         |                           |
                         +---------------------------+
```

**框图说明：**

- **3V3 供电**：模组供电电压 3.0 ~ 3.6 V
- **EN 使能信号**：高电平使能芯片，低电平关闭芯片，不能浮空
- **40 MHz 晶振**：集成在模组内部
- **ESP32-S3R8V / ESP32-S3R16V 芯片**：搭载 Xtensa 双核 32 位 LX7 处理器
- **RF 匹配电路**：射频匹配网络，50 ohm 阻抗控制
- **板载 PCB 天线**：2.4 GHz 天线
- **OSPI Flash**：通过 SPICS0, SPICLK, SPID, SPIQ, SPIHD, SPIWP, VDD_SPI 连接
- **OSPI PSRAM**：通过 SPIIO4, SPIIO5, SPIIO6, SPIIO7, SPIDQS 连接
- **GPIO**：33 个可用 GPIO 管脚

---

## 3. 型号对比

### 3.1 ESP32-S3-WROOM-2 系列型号

| 物料编号 | Flash | PSRAM | 内置芯片 | 环境温度 (°C) | 模组尺寸 (mm) | 状态 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| ESP32-S3-WROOM-2-N32R16V | 32 MB (Octal SPI) | 16 MB (Octal SPI) | ESP32-S3R16V | -40 ~ 65 | 18.0 x 25.5 x 3.1 | 在产 |
| ESP32-S3-WROOM-2-N32R8V | 32 MB (Octal SPI) | 8 MB (Octal SPI) | ESP32-S3R8V | -40 ~ 65 | 18.0 x 25.5 x 3.1 | 停产 |
| ESP32-S3-WROOM-2-N16R8V | 16 MB (Octal SPI) | 8 MB (Octal SPI) | ESP32-S3R8V | -40 ~ 65 | 18.0 x 25.5 x 3.1 | 停产 |

> **说明：**
> 1. 默认情况下，模组 SPI flash 支持的最大时钟频率为 120 MHz，且不支持自动暂停功能。
> 2. 该模组使用封装在芯片中的 PSRAM。
> 3. 环境温度指乐鑫模组外部的推荐环境温度。
> 4. 若开启 PSRAM ECC 功能，模组最大环境温度可提高到 85 °C，但 PSRAM 可用容量将减少 1/16。

### 3.2 ESP32-S3-WROOM-2 与 WROOM-1 系列对比

| 参数 | ESP32-S3-WROOM-2 | ESP32-S3-WROOM-1 | ESP32-S3-WROOM-1U |
| :--- | :--- | :--- | :--- |
| 内置芯片 | ESP32-S3R8V / ESP32-S3R16V | ESP32-S3 / ESP32-S3R2 / ESP32-S3R8 | ESP32-S3 / ESP32-S3RH2 / ESP32-S3R8 |
| Flash | 16/32 MB (Octal SPI) | 4/8/16 MB (Quad SPI) | 4/8/16 MB (Quad SPI) |
| PSRAM | 8/16 MB (Octal SPI) | 0/2/8 MB | 0/2/8 MB |
| GPIO | 33 | 36 | 36 |
| 天线 | 板载 PCB 天线 | 板载 PCB 天线 | 外接天线座子 (IPEX) |
| 模组尺寸 | 18 x 25.5 x 3.1 mm | 18 x 25.5 x 3.1 mm | 18 x 19.2 x 3.2 mm |
| 管脚数 | 41 (含 EPAD) | 41 (含 EPAD) | 41 (含 EPAD) |

### 3.3 与 ESP32-S3 其他模组系列对比

| 模组 | 内置芯片 | Flash | PSRAM | GPIO | 天线 | 尺寸 (mm) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| ESP32-S3-WROOM-2 | ESP32-S3R16V / ESP32-S3R8V | 16/32 MB | 8/16 MB | 33 | PCB 天线 | 18 x 25.5 x 3.1 |
| ESP32-S3-WROOM-1 | ESP32-S3 / S3R2 / S3R8 | 4/8/16 MB | 0/2/8 MB | 36 | PCB 天线 | 18 x 25.5 x 3.1 |
| ESP32-S3-WROOM-1U | ESP32-S3 / S3RH2 / S3R8 | 4/8/16 MB | 0/2/8 MB | 36 | 外接天线 | 18 x 19.2 x 3.2 |
| ESP32-S3-MINI-1 | ESP32-S3FN8 / S3FH4R2 | 4/8 MB | 0/2 MB | 39 | PCB 天线 | 15.4 x 20.5 x 2.4 |

---

## 4. 管脚定义

### 4.1 管脚布局

模组共有 41 个管脚，包含 40 个外围管脚和 1 个中心接地焊盘 (EPAD)。管脚编号 1~40 沿模组两侧排列。

### 4.2 管脚描述

| 名称 | 序号 | 类型 | 功能 |
| :--- | :--- | :--- | :--- |
| GND | 1 | P | 接地 |
| 3V3 | 2 | P | 供电 (3.0 ~ 3.6 V) |
| EN | 3 | I | 高电平：芯片使能；低电平：芯片关闭。注意不能让 EN 管脚浮空 |
| IO4 | 4 | I/O/T | RTC_GPIO4, GPIO4, TOUCH4, ADC1_CH3 |
| IO5 | 5 | I/O/T | RTC_GPIO5, GPIO5, TOUCH5, ADC1_CH4 |
| IO6 | 6 | I/O/T | RTC_GPIO6, GPIO6, TOUCH6, ADC1_CH5 |
| IO7 | 7 | I/O/T | RTC_GPIO7, GPIO7, TOUCH7, ADC1_CH6 |
| IO15 | 8 | I/O/T | RTC_GPIO15, GPIO15, U0RTS, ADC2_CH4, XTAL_32K_P |
| IO16 | 9 | I/O/T | RTC_GPIO16, GPIO16, U0CTS, ADC2_CH5, XTAL_32K_N |
| IO17 | 10 | I/O/T | RTC_GPIO17, GPIO17, U1TXD, ADC2_CH6 |
| IO18 | 11 | I/O/T | RTC_GPIO18, GPIO18, U1RXD, ADC2_CH7, CLK_OUT3 |
| IO8 | 12 | I/O/T | RTC_GPIO8, GPIO8, TOUCH8, ADC1_CH7, SUBSPICS1 |
| IO19 | 13 | I/O/T | RTC_GPIO19, GPIO19, U1RTS, ADC2_CH8, CLK_OUT2, USB_D- |
| IO20 | 14 | I/O/T | RTC_GPIO20, GPIO20, U1CTS, ADC2_CH9, CLK_OUT1, USB_D+ |
| IO3 | 15 | I/O/T | RTC_GPIO3, GPIO3, TOUCH3, ADC1_CH2 |
| IO46 | 16 | I/O/T | GPIO46 |
| IO9 | 17 | I/O/T | RTC_GPIO9, GPIO9, TOUCH9, ADC1_CH8, FSPIHD, SUBSPIHD |
| IO10 | 18 | I/O/T | RTC_GPIO10, GPIO10, TOUCH10, ADC1_CH9, FSPICS0, FSPIIO4, SUBSPICS0 |
| IO11 | 19 | I/O/T | RTC_GPIO11, GPIO11, TOUCH11, ADC2_CH0, FSPID, FSPIIO5, SUBSPID |
| IO12 | 20 | I/O/T | RTC_GPIO12, GPIO12, TOUCH12, ADC2_CH1, FSPICLK, FSPIIO6, SUBSPICLK |
| IO13 | 21 | I/O/T | RTC_GPIO13, GPIO13, TOUCH13, ADC2_CH2, FSPIQ, FSPIIO7, SUBSPIQ |
| IO14 | 22 | I/O/T | RTC_GPIO14, GPIO14, TOUCH14, ADC2_CH3, FSPIWP, FSPIDQS, SUBSPIWP |
| IO21 | 23 | I/O/T | RTC_GPIO21, GPIO21 |
| IO47 | 24 | I/O/T | SPICLK_P_DIFF, GPIO47, SUBSPICLK_P_DIFF |
| IO48 | 25 | I/O/T | SPICLK_N_DIFF, GPIO48, SUBSPICLK_N_DIFF |
| IO45 | 26 | I/O/T | GPIO45 |
| IO0 | 27 | I/O/T | RTC_GPIO0, GPIO0 |
| NC | 28 | -- | 空管脚 |
| NC | 29 | -- | 空管脚 |
| NC | 30 | -- | 空管脚 |
| IO38 | 31 | I/O/T | GPIO38, FSPIWP, SUBSPIWP |
| IO39 | 32 | I/O/T | MTCK, GPIO39, CLK_OUT3, SUBSPICS1 |
| IO40 | 33 | I/O/T | MTDO, GPIO40, CLK_OUT2 |
| IO41 | 34 | I/O/T | MTDI, GPIO41, CLK_OUT1 |
| IO42 | 35 | I/O/T | MTMS, GPIO42 |
| RXD0 | 36 | I/O/T | U0RXD, GPIO44, CLK_OUT2 |
| TXD0 | 37 | I/O/T | U0TXD, GPIO43, CLK_OUT1 |
| IO2 | 38 | I/O/T | RTC_GPIO2, GPIO2, TOUCH2, ADC1_CH1 |
| IO1 | 39 | I/O/T | RTC_GPIO1, GPIO1, TOUCH1, ADC1_CH0 |
| GND | 40 | P | 接地 |
| EPAD | 41 | P | 接地 (中心焊盘) |

> **说明：**
> - P：电源；I：输入；O：输出；T：可设置为高阻。
> - 加粗字体为管脚的默认功能。
> - 由于 ESP32-S3R8V 和 ESP32-S3R16V 芯片的 VDD_SPI 电压已设置为 1.8 V，所以，不同于其他 GPIO，在 VDD_SPI 电源域中的 GPIO47 和 GPIO48 的工作电压也为 1.8 V。

### 4.3 Strapping 管脚

ESP32-S3 芯片上有 4 个 strapping 管脚，用于控制芯片的启动模式：

| Strapping 管脚 | 模组管脚 | 功能说明 |
| :--- | :--- | :--- |
| GPIO0 | IO0 (27) | 启动模式选择：低电平进入下载模式，高电平进入 Flash 运行模式 |
| GPIO46 | IO46 (16) | 启动模式选择 |
| GPIO3 | IO3 (15) | JTAG 信号源选择 |
| GPIO45 | IO45 (26) | VDD_SPI 电压选择 |

> **注意：** 在芯片上电时，strapping 管脚的电平被锁存。如果需要更改启动模式，请在 EN 信号上升沿之前设置好 strapping 管脚的电平。

### 4.4 电源管脚

| 管脚 | 说明 |
| :--- | :--- |
| 3V3 (管脚 2) | 模组供电管脚，供电电压 3.0 ~ 3.6 V，典型值 3.3 V |
| GND (管脚 1, 40) | 接地管脚 |
| EPAD (管脚 41) | 中心接地焊盘，建议焊接到底板 GND 以获得更好的散热特性 |

---

## 5. 外设概述

ESP32-S3 集成了丰富的外设，包括 SPI、LCD、Camera 接口、UART、I2C、I2S、红外遥控、脉冲计数器、LED PWM、USB 串口/JTAG、MCPWM、SD/MMC 主机控制器、TWAI 控制器、ADC、触摸传感器和温度传感器。此外，还有一个全速 USB 2.0 On-The-Go (OTG) 接口。

### 5.1 通讯接口

#### 5.1.1 UART 控制器

ESP32-S3 有三个 UART 控制器，即 UART0、UART1、UART2，支持异步通信（RS232 和 RS485）和 IrDA，通信速率可达到 5 Mbps。

- 支持三个可预分频的时钟源
- 可编程收发波特率
- 三个 UART 的发送 FIFO 以及接收 FIFO 共享 1024 x 8-bit RAM
- 全双工异步通信
- 支持输入信号波特率自检功能
- 支持 5/6/7/8 位数据长度
- 支持 1/1.5/2/3 个停止位
- 支持奇偶校验位
- 支持 AT_CMD 特殊字符检测
- 支持 RS485 协议
- 支持 IrDA 协议
- 支持 GDMA 高速数据通信
- 支持 UART 唤醒模式
- 支持软件流控和硬件流控

#### 5.1.2 I2C 接口

ESP32-S3 有两个 I2C 总线接口，可以用作 I2C 主机或从机模式。

- 标准模式 (100 Kbit/s)
- 快速模式 (400 Kbit/s)
- 速度最高可达 800 Kbit/s，但受制于 SCL 和 SDA 上拉强度
- 7 位寻址模式和 10 位寻址模式
- 双地址寻址模式

#### 5.1.3 I2S 接口

ESP32-S3 有两个标准 I2S 接口，可以以主机或从机模式工作，支持频率从 10 kHz 到 40 MHz。

- 支持 TDM PCM、TDM MSB 对齐、TDM LSB 对齐、TDM Phillips、PDM 接口

#### 5.1.4 串行外设接口 (SPI)

ESP32-S3 具有四个 SPI 接口：

- **SPI0**：供 GDMA 控制器与 Cache 访问封装内或封装外 flash/PSRAM
- **SPI1**：供 CPU 访问封装内或封装外 flash/PSRAM
- **SPI2 (通用)**：支持主机或从机模式，支持多种 SPI 模式 (Dual, Quad, Octal, QPI, OPI)。主机模式时钟频率可达 80 MHz
- **SPI3 (通用)**：支持主机或从机模式，支持多种 SPI 模式

#### 5.1.5 双线汽车接口 (TWAI)

兼容 ISO 11898-1 协议 (CAN 规范 2.0)。

- 支持标准帧 (11 位 ID) 和扩展帧 (29 位 ID)
- 1 Kbit/s 到 1 Mbit/s 比特率
- 具有工作模式、监听模式、自检模式
- 64 字节接收 FIFO

### 5.2 显示与摄像头接口

#### 5.2.1 LCD 控制器

- 支持 8 位 ~ 16 位并行 RGB、I8080、MOTO6800 接口
- 支持频率小于 40 MHz
- 支持颜色格式转换

#### 5.2.2 Camera 控制器

- 支持 8 位 ~ 16 位 DVP 图像传感器接口
- 支持频率小于 40 MHz
- 支持颜色格式转换

### 5.3 USB 接口

#### 5.3.1 USB 2.0 OTG 全速接口

符合 USB 2.0 规范，集成了收发器。

- 支持全速和低速速率
- 支持 HNP 和 SRP
- 6 个附加端点，可配置为 IN 或 OUT
- 支持 Scatter/Gather DMA 模式

#### 5.3.2 USB 串口/JTAG 控制器

- USB 全速标准
- 可配置为使用 ESP32-S3 内部 USB PHY 或通过 GPIO 交换矩阵使用外部 PHY
- 固定功能，包含 CDC-ACM 和 JTAG 适配器功能
- 共 2 个 OUT 端点、3 个 IN 端点和 1 个控制端点 EP_0，可实现最大 64 字节的数据载荷
- 包含内部 PHY，基本无需其他外部组件连接主机计算机
- CDC-ACM 的虚拟串行功能在大多数现代操作系统上可实现即插即用
- JTAG 接口可使用紧凑的 JTAG 指令实现与 CPU 调试内核的快速通信
- CDC-ACM 支持主机控制芯片复位和进入下载模式

### 5.4 模拟外设

#### 5.4.1 ADC

- 两个 SAR ADC 模块：ADC1 (10 通道) 和 ADC2 (10 通道)
- 12 位分辨率
- 支持多种衰减比配置

#### 5.4.2 触摸传感器

- 支持多达 14 个电容式触摸传感通道
- 可用于接近检测

#### 5.4.3 温度传感器

- 内置温度传感器，用于测量芯片内部温度

### 5.5 定时器与 PWM

#### 5.5.1 LED PWM 控制器

- 可用于生成八路独立的数字波形
- 波形的周期和占空比可配置，在信号周期为 1 ms 时，占空比精确度可达 14 位
- 多种时钟源选择
- 可在 Light-sleep 模式下工作
- 支持硬件自动步进式地增加或减少占空比

#### 5.5.2 电机控制脉宽调制器 (MCPWM)

- 两个 MCPWM 模块
- 每个 MCPWM 包含三个 PWM 定时器、三个 PWM 操作器和一个捕捉模块
- 可用于驱动数字马达和智能灯

### 5.6 其他外设

#### 5.6.1 红外遥控 (RMT)

- 四个通道支持发送，四个通道支持接收
- 八个通道共享 384 x 32-bit 的 RAM
- 发送脉冲支持载波调制
- 接收脉冲支持滤波和载波解调

#### 5.6.2 脉冲计数控制器 (PCNT)

- 四个脉冲计数控制器（单元），各自独立工作
- 计数范围 1 ~ 65535
- 每个单元有两个独立的通道

#### 5.6.3 SD/MMC 主机控制器

- SD 卡 3.0 和 3.01 版本
- SDIO 3.0 版本
- 高达 80 MHz 的时钟输出
- 支持 1 位、4 位、8 位数据总线模式

#### 5.6.4 GDMA

- 通用 DMA 控制器，支持高速数据传输

---

## 6. 电气特性

### 6.1 绝对最大额定值

| 符号 | 参数 | 最小值 | 最大值 | 单位 |
| :--- | :--- | :--- | :--- | :--- |
| VDD33 | 电源管脚电压 | -0.3 | 3.6 | V |
| T_STORE | 存储温度 | -40 | 105 | °C |

### 6.2 建议工作条件

| 符号 | 参数 | 最小值 | 典型值 | 最大值 | 单位 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| VDD33 | 电源管脚电压 | 3.0 | 3.3 | 3.6 | V |
| I_VDD | 外部电源的供电电流 | 0.5 | -- | -- | A |
| T_A | 工作环境温度 | -40 | -- | 65 | °C |

### 6.3 直流电气特性 (3.3 V, 25 °C)

| 参数 | 说明 | 最小值 | 典型值 | 最大值 | 单位 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| C_IN | 管脚电容 | -- | 2 | -- | pF |
| V_IH | 高电平输入电压 | 0.75 x VDD | -- | VDD + 0.3 | V |
| V_IL | 低电平输入电压 | -0.3 | -- | 0.25 x VDD | V |
| I_IH | 高电平输入电流 | -- | -- | 50 | nA |
| I_IL | 低电平输入电流 | -- | -- | 50 | nA |
| V_OH | 高电平输出电压 | 0.8 x VDD | -- | -- | V |
| V_OL | 低电平输出电压 | -- | -- | 0.1 x VDD | V |
| I_OH | 高电平拉电流 (VDD=3.3V, V_OH>=2.64V, PAD_DRIVER=3) | -- | 40 | -- | mA |
| I_OL | 低电平灌电流 (VDD=3.3V, V_OL=0.495V, PAD_DRIVER=3) | -- | 28 | -- | mA |
| R_PU | 内部弱上拉电阻 | -- | 45 | -- | kohm |
| R_PD | 内部弱下拉电阻 | -- | 45 | -- | kohm |
| V_IH_nRST | 芯片复位释放电压 | 0.75 x VDD | -- | VDD + 0.3 | V |
| V_IL_nRST | 芯片复位电压 | -0.3 | -- | 0.25 x VDD | V |

> **说明：**
> - VDD -- 各个电源域电源管脚的电压。
> - V_OH 和 V_OL 为负载是高阻条件下的测试值。

### 6.4 功耗特性

#### 6.4.1 Active 模式下的功耗

下列功耗数据是基于 3.3 V 供电电源、25 °C 环境温度的条件下测得。所有发射功耗数据均基于 100% 占空比测得。所有接收功耗数据均是在外设关闭、CPU 空闲的条件下测得。

**Wi-Fi 功耗：**

| 工作模式 | 描述 | 峰值 (mA) |
| :--- | :--- | :--- |
| Active (射频工作) TX | 802.11b, 1 Mbps, @20.5 dBm | 355 |
| Active (射频工作) TX | 802.11g, 54 Mbps, @18 dBm | 297 |
| Active (射频工作) TX | 802.11n, HT20, MCS7, @17.5 dBm | 286 |
| Active (射频工作) TX | 802.11n, HT40, MCS7, @17.5 dBm | 285 |
| Active (射频工作) RX | 802.11b/g/n, HT20 | 95 |
| Active (射频工作) RX | 802.11n, HT40 | 97 |

**低功耗蓝牙功耗：**

| 工作模式 | 描述 | 峰值 (mA) |
| :--- | :--- | :--- |
| Active (射频工作) TX | 低功耗蓝牙 @ 20.0 dBm | 344 |
| Active (射频工作) TX | 低功耗蓝牙 @ 9.0 dBm | 202 |
| Active (射频工作) TX | 低功耗蓝牙 @ 0 dBm | 187 |
| Active (射频工作) TX | 低功耗蓝牙 @ -15.0 dBm | 119 |
| Active (射频工作) RX | 低功耗蓝牙 | 93 |

#### 6.4.2 Modem-sleep 模式下的功耗

| 频率 (MHz) | 说明 | 典型值 (mA) 外设关闭 | 典型值 (mA) 外设开启 |
| :--- | :--- | :--- | :--- |
| 40 | WAITI (双核均空闲) | 13.2 | 18.8 |
| 40 | 单核执行 32 位数据访问指令 | 16.2 | 21.8 |
| 40 | 双核执行 32 位数据访问指令 | 18.7 | 24.4 |
| 80 | WAITI | 22.0 | 36.1 |
| 80 | 单核执行 32 位数据访问指令 | 28.4 | 42.6 |
| 80 | 双核执行 32 位数据访问指令 | 33.1 | 47.3 |
| 160 | WAITI | 27.6 | 42.3 |
| 160 | 单核执行 32 位数据访问指令 | 39.9 | 54.6 |
| 160 | 双核执行 32 位数据访问指令 | 49.6 | 64.1 |
| 240 | WAITI | 32.9 | 47.6 |
| 240 | 单核执行 32 位数据访问指令 | 51.2 | 65.9 |
| 240 | 双核执行 32 位数据访问指令 | 66.2 | 81.3 |

> **说明：** Modem-sleep 模式下，Wi-Fi 设有时钟门控。该模式下，访问 flash 时功耗会增加。若 flash 速率为 80 Mbit/s，SPI 双线模式下 flash 的功耗为 10 mA。

#### 6.4.3 低功耗模式下的功耗

| 工作模式 | 说明 | 典型值 (uA) |
| :--- | :--- | :--- |
| Light-sleep | VDD_SPI 和 Wi-Fi 掉电，所有 GPIO 设置为高阻状态 | 240 |
| Deep-sleep | ULP 协处理器处于工作状态 (ULP-FSM) | 170 |
| Deep-sleep | ULP 协处理器处于工作状态 (ULP-RISC-V) | 190 |
| Deep-sleep | 超低功耗传感器监测模式 | 18 |
| Deep-sleep | RTC 存储器和 RTC 外设上电 | 8 |
| Deep-sleep | RTC 存储器上电，RTC 外设掉电 | 7 |
| 关闭 | EN 管脚拉低，芯片关闭 | 1 |

> **说明：**
> - Light-sleep 模式下，SPI 相关管脚上拉。封装内有 PSRAM 的芯片请在典型值的基础上添加相应的 PSRAM 功耗：8 MB 8 线 PSRAM (3.3 V) 为 140 uA；8 MB 8 线 PSRAM (1.8 V) 为 200 uA。
> - Deep-sleep 模式下，仅 ULP 协处理器处于工作状态时，可以操作 GPIO 及低功耗 I2C。
> - 当系统处于超低功耗传感器监测模式时，ULP 协处理器或传感器周期性工作。触摸传感器以 1% 占空比工作，系统功耗典型值为 18 uA。

### 6.5 存储器规格

**Flash 规格：**

| 参数 | 说明 | 最小值 | 典型值 | 最大值 | 单位 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| VCC | 电源电压 (1.8 V) | 1.65 | 1.80 | 2.00 | V |
| VCC | 电源电压 (3.3 V) | 2.7 | 3.3 | 3.6 | V |
| F_C | 最大时钟频率 | 80 | -- | -- | MHz |
| -- | 编程/擦除周期 | 100,000 | -- | -- | 次 |
| T_RET | 数据保留时间 | 20 | -- | -- | 年 |
| T_PP | 页编程时间 | -- | 0.8 | 5 | ms |
| T_SE | 扇区擦除时间 (4 KB) | -- | 70 | 500 | ms |
| T_BE1 | 块擦除时间 (32 KB) | -- | 0.2 | 2 | s |
| T_BE2 | 块擦除时间 (64 KB) | -- | 0.3 | 3 | s |
| T_CE | 芯片擦除时间 (32 Mb) | -- | 20 | 60 | s |
| T_CE | 芯片擦除时间 (256 Mb) | -- | 70 | 300 | s |

**PSRAM 规格：**

| 参数 | 说明 | 最小值 | 典型值 | 最大值 | 单位 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| VCC | 电源电压 (1.8 V) | 1.62 | 1.80 | 1.98 | V |
| VCC | 电源电压 (3.3 V) | 2.7 | 3.3 | 3.6 | V |
| F_C | 最大时钟频率 | 80 | -- | -- | MHz |

---

## 7. 射频特性

### 7.1 Wi-Fi 射频

| 名称 | 描述 |
| :--- | :--- |
| 工作信道中心频率范围 | 2412 ~ 2484 MHz |
| 无线标准 | IEEE 802.11b/g/n |

#### 7.1.1 Wi-Fi 射频发射器 (TX) 特性

| 速率 | 最小值 (dBm) | 典型值 (dBm) | 最大值 (dBm) |
| :--- | :--- | :--- | :--- |
| 802.11b, 1 Mbps | -- | 20.5 | -- |
| 802.11b, 11 Mbps | -- | 20.5 | -- |
| 802.11g, 6 Mbps | -- | 20.0 | -- |
| 802.11g, 54 Mbps | -- | 18.0 | -- |
| 802.11n, HT20, MCS 0 | -- | 19.0 | -- |
| 802.11n, HT20, MCS 7 | -- | 17.5 | -- |
| 802.11n, HT40, MCS 0 | -- | 18.5 | -- |
| 802.11n, HT40, MCS 7 | -- | 17.0 | -- |

**发射 EVM 测试：**

| 速率 | 典型值 (dB) | 标准限值 (dB) |
| :--- | :--- | :--- |
| 802.11b, 1 Mbps, @20.5 dBm | -24.5 | -10 |
| 802.11b, 11 Mbps, @20.5 dBm | -24.5 | -10 |
| 802.11g, 6 Mbps, @20 dBm | -23.0 | -5 |
| 802.11g, 54 Mbps, @18 dBm | -29.5 | -25 |
| 802.11n, HT20, MCS 0, @19 dBm | -24.0 | -5 |
| 802.11n, HT20, MCS 7, @17.5 dBm | -30.5 | -27 |
| 802.11n, HT40, MCS 0, @18.5 dBm | -25.0 | -5 |
| 802.11n, HT40, MCS 7, @17 dBm | -30.0 | -27 |

#### 7.1.2 Wi-Fi 射频接收器 (RX) 特性

802.11b 标准下的误包率 (PER) 不超过 8%，802.11g/n 标准下不超过 10%。

**接收灵敏度：**

| 速率 | 典型值 (dBm) |
| :--- | :--- |
| 802.11b, 1 Mbps | -98.2 |
| 802.11b, 2 Mbps | -95.6 |
| 802.11b, 5.5 Mbps | -92.8 |
| 802.11b, 11 Mbps | -88.5 |
| 802.11g, 6 Mbps | -93.0 |
| 802.11g, 9 Mbps | -92.0 |
| 802.11g, 12 Mbps | -90.8 |
| 802.11g, 18 Mbps | -88.5 |
| 802.11g, 24 Mbps | -85.5 |
| 802.11g, 36 Mbps | -82.2 |
| 802.11g, 48 Mbps | -78.0 |
| 802.11g, 54 Mbps | -76.2 |
| 802.11n, HT20, MCS 0 | -93.0 |
| 802.11n, HT20, MCS 1 | -90.6 |
| 802.11n, HT20, MCS 2 | -88.4 |
| 802.11n, HT20, MCS 3 | -84.8 |
| 802.11n, HT20, MCS 4 | -81.6 |
| 802.11n, HT20, MCS 5 | -77.4 |
| 802.11n, HT20, MCS 6 | -75.6 |
| 802.11n, HT20, MCS 7 | -74.2 |
| 802.11n, HT40, MCS 0 | -90.0 |
| 802.11n, HT40, MCS 1 | -87.5 |
| 802.11n, HT40, MCS 2 | -85.0 |
| 802.11n, HT40, MCS 3 | -82.0 |
| 802.11n, HT40, MCS 4 | -78.5 |
| 802.11n, HT40, MCS 5 | -74.4 |
| 802.11n, HT40, MCS 6 | -72.5 |
| 802.11n, HT40, MCS 7 | -71.2 |

**最大接收电平：**

| 速率 | 典型值 (dBm) |
| :--- | :--- |
| 802.11b, 1 Mbps | 5 |
| 802.11b, 11 Mbps | 5 |
| 802.11g, 6 Mbps | 5 |
| 802.11g, 54 Mbps | 0 |
| 802.11n, HT20, MCS 0 | 5 |
| 802.11n, HT20, MCS 7 | 0 |
| 802.11n, HT40, MCS 0 | 5 |
| 802.11n, HT40, MCS 7 | 0 |

**接收邻道抑制：**

| 速率 | 典型值 (dB) |
| :--- | :--- |
| 802.11b, 1 Mbps | 35 |
| 802.11b, 11 Mbps | 35 |
| 802.11g, 6 Mbps | 31 |
| 802.11g, 54 Mbps | 14 |
| 802.11n, HT20, MCS 0 | 31 |
| 802.11n, HT20, MCS 7 | 13 |
| 802.11n, HT40, MCS 0 | 19 |
| 802.11n, HT40, MCS 7 | 8 |

### 7.2 低功耗蓝牙射频

| 名称 | 描述 |
| :--- | :--- |
| 工作信道中心频率范围 | 2402 ~ 2480 MHz |
| 射频发射功率范围 | -24.0 ~ 20.0 dBm |

#### 7.2.1 低功耗蓝牙射频发射器 (TX) 特性

| 参数 | 1 Mbps 典型值 | 2 Mbps 典型值 | 125 Kbps 典型值 | 500 Kbps 典型值 | 单位 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 载波频率偏移 (f_n 最大值) | 2.50 | 2.50 | 0.80 | 0.80 | kHz |
| 调制特性 (f1_avg) | 249.00 | 499.00 | 248.00 | -- | kHz |
| 调制特性 (f2_avg) | -- | -- | -- | 213.00 | kHz |
| 带内杂散发射 (偏移 +/-2 MHz) | -37.00 | -- | -37.00 | -37.00 | dBm |
| 带内杂散发射 (偏移 +/-3 MHz) | -42.00 | -- | -42.00 | -42.00 | dBm |
| 带内杂散发射 (偏移 > +/-3 MHz) | -44.00 | -- | -44.00 | -44.00 | dBm |

#### 7.2.2 低功耗蓝牙射频接收器 (RX) 特性

| 参数 | 1 Mbps | 2 Mbps | 125 Kbps | 500 Kbps | 单位 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 灵敏度 @30.8% PER | -96.5 | -92.5 | -103.5 | -100 | dBm |
| 最大接收信号 @30.8% PER | 8 | 3 | 8 | 8 | dBm |
| 共信道抑制比 C/I | 9 | 10 | 6 | 4 | dB |
| 镜像频率抑制 | -32 | -31 | -35 | -37 | dB |
| 带外阻塞 (30 MHz ~ 2000 MHz) | -9 | -15 | -- | -- | dBm |
| 带外阻塞 (2003 MHz ~ 2399 MHz) | -18 | -19 | -- | -- | dBm |
| 带外阻塞 (2484 MHz ~ 2997 MHz) | -15 | -15 | -- | -- | dBm |
| 带外阻塞 (3000 MHz ~ 12.75 GHz) | -5 | -6 | -- | -- | dBm |
| 互调 | -29 | -29 | -- | -- | dBm |

---

## 8. 模组原理图

### 8.1 原理图说明

- **晶振电路**：Y1 为 40 MHz (+/-10ppm)。C1 和 C4 的值根据晶振选型而变化。
- **射频匹配**：L3, C16, C11, L2 和 C12 的值根据实际 PCB 板调整。NC: No component。50 ohm 阻抗控制。
- **电源滤波**：VDD33 包含 C6 (10uF), C7 (1uF), C8 (0.1uF), C9 (0.1uF) 等滤波电容。
- **芯片引脚配置**：R4 的值根据实际 PCB 板调整。R4 可以是电阻或电感，初始建议值为 24 nH。
- **上拉/下拉电阻**：R1 为 10K (NC)，R11 为 10K，R12 为 10K。

### 8.2 Flash 连接 (U2 FLASH_1V8)

| 引脚编号 | 引脚名称 | 引脚编号 | 引脚名称 |
| :--- | :--- | :--- | :--- |
| A4 | RESET# | B4 | VCC |
| A5 | ECS# | D1 | VCC |
| C3 | DQS | E4 | VCC |
| B2 | CS# | A2 | NC1 |
| D3 | SI(IO0) | A3 | NC2 |
| D2 | SO(IO1) | B1 | NC3 |
| C4 | IO2 | B5 | NC4 |
| D4 | IO3 | C5 | WP# |
| D5 | IO4 | B3 | VSS |
| E3 | IO5 | C1 | VSS |
| E2 | IO6 | E5 | VSS |
| E1 | IO7 | | |

> **注意：** 从 BOM v0.7 开始，ESP32-S3-WROOM-2 GPIO45 上的外部上拉电阻 R1 已改为不上件。

---

## 9. 外围设计原理图

### 9.1 外围电路说明

模组与外围器件（如电源、天线、复位按钮、JTAG 接口、UART 接口等）连接的应用电路设计要点：

- **电源设计**：C1 (22uF), C3 (0.1uF) 用于 VDD33 滤波
- **RC 延时电路**：R1 (TBD), C2 (TBD) 连接至 EN，用于确保上电复位正常。通常建议 R = 10 kohm, C = 1 uF
- **USB OTG (JP3)**：1: USB_D-, 2: USB_D+
- **UART (JP1)**：1: VDD33, 2: TXD0, 3: RXD0, 4: GND
- **JTAG (JP2)**：1: TMS (IO42), 2: TDI (IO41), 3: TDO (IO40), 4: TCK (IO39)
- **Boot Option (JP4)**：1: IO0, 2: GND
- **复位按键 (SW1)**：连接至 EN，配合 C8 (0.1uF) 和 R7 (0 ohm)

### 9.2 设计注意事项

- EPAD 可以不焊接到底板，但是焊接到底板的 GND 可以获得更好的散热特性。如果要将 EPAD 焊接到底板，请确保使用适量焊膏，避免过量焊膏造成模组与底板距离过大，影响管脚与底板之间的贴合。
- 为确保 ESP32-S3 芯片上电时的供电正常，EN 管脚处需要增加 RC 延迟电路。RC 通常建议为 R = 10 kohm, C = 1 uF，但具体数值仍需根据模组电源的上电时序和芯片的上电复位时序进行调整。
- IO47/IO48 在 1.8V 电压域下工作（使用 ESP32-S3R8V 或 ESP32-S3R16V 芯片时）。

---

## 10. 模组尺寸

### 10.1 尺寸参数

| 参数 | 值 | 单位 |
| :--- | :--- | :--- |
| 模组长度 | 25.5 +/- 0.2 | mm |
| 模组宽度 | 18.0 +/- 0.2 | mm |
| 模组厚度 | 3.1 +/- 0.15 | mm |
| PCB 厚度 | 0.8 | mm |
| 管脚间距 | 1.27 | mm |
| 天线区域宽度 | 18 | mm |
| 天线区域高度 | 6 | mm |

### 10.2 焊盘规格

| 参数 | 值 | 单位 |
| :--- | :--- | :--- |
| 外侧焊盘数量 | 40 | 个 |
| 外侧焊盘尺寸 (长 x 宽) | 1.5 x 0.9 | mm |
| 外侧焊盘间距 | 1.27 | mm |
| 底部散热焊盘数量 | 9 (3x3 阵列) | 个 |
| 底部散热焊盘尺寸 | 0.5 x 0.5 | mm |
| 底部散热焊盘间距 | 1.0 | mm |

### 10.3 PCB 布局建议

**PCB 封装图标注说明（单位：mm）：**

- **天线区域 (Antenna Area)**：宽度 18，高度 6
- **外框尺寸**：18 x 25.5
- **焊盘规格**：
  - 外侧焊盘 (1-40): 40 x 1.5 (长) x 0.9 (宽)
  - 外侧焊盘间距: 1.27
  - 底部中心焊盘: 9 个 (3x3 阵列)，每个 0.5 x 0.5，间距 1.0
- **定位尺寸**：
  - 天线区底沿到第一排焊盘距离: 7.49
  - 左右焊盘中心跨度: 17.5
  - 焊盘到模组边缘距离: 0.5

如产品采用模组进行 on-board 设计，则需注意考虑模组在底板的布局，应尽可能地减小底板对模组 PCB 天线性能的影响。关于 PCB 设计中模组位置摆放的更多信息，请参考《ESP32-S3 硬件设计指南》。

### 10.4 封装信息

| 参数 | 说明 |
| :--- | :--- |
| MPQ (最小包装数量) | 650 |
| MOQ (最小订购数量) | 3250 |
| 封装形式 | SMD |
| 引脚数 | 41 (含 EPAD) |

---

## 11. 认证信息

### 11.1 RF 认证

| 认证 | 编号/状态 | 说明 |
| :--- | :--- | :--- |
| FCC | 已认证 | 美国联邦通信委员会认证 |
| CE | 已认证 | 欧盟认证 |
| SRRC | CMIIT ID: 2022DP8369 | 中国无线电型号核准，有效期至 2028-12-31 |
| MIC | 认证号: 020-250321 | 日本无线电设备认证 |
| ISED | 认证号: 21098-ESPS3WROOM2 | 加拿大创新科学和经济发展部认证 |

### 11.2 环保认证

- RoHS 认证
- REACH 认证

### 11.3 可靠性测试

| 测试项目 | 说明 |
| :--- | :--- |
| HTOL | 高温工作寿命测试 |
| HTSL | 高温存储寿命测试 |
| uHAST | 非饱和高压蒸煮测试 |
| TCT | 温度循环测试 |
| ESD | 静电放电测试 |

---

## 12. 焊接信息

### 12.1 回流焊温度曲线

- **升温区**：温度 25 ~ 150 °C，时间 60 ~ 90 s，升温斜率 1 ~ 3 °C/s
- **预热恒温区**：温度 150 ~ 200 °C，时间 60 ~ 120 s
- **回流焊接区**：温度 > 217 °C，时间 60 ~ 90 s；峰值温度 235 ~ 250 °C，时间 30 ~ 70 s
- **冷却区**：从峰值温度到 180 °C，降温斜率 -1 ~ -5 °C/s
- **焊料**：锡银铜合金无铅焊料 (SAC305)

> **说明：** 建议模组只过一次回流焊。如果 PCBA 需要多次回流焊，则在最后一次回流焊时将模组放在 PCB 上方。

---

## 13. 参考资源

### 13.1 相关文档

- [ESP32-S3 系列芯片技术规格书](https://www.espressif.com/sites/default/files/documentation/esp32-s3_datasheet_cn.pdf)
- [ESP32-S3 技术参考手册](https://www.espressif.com/sites/default/files/documentation/esp32-s3_technical_reference_manual_cn.pdf)
- [ESP32-S3 硬件设计指南](https://www.espressif.com/sites/default/files/documentation/esp32-s3_hardware_design_guidelines_cn.pdf)
- [ESP32-S3-WROOM-2 PCB Footprint](https://www.espressif.com/sites/default/files/modules-dxf/ESP32-S3-WROOM-2%20PCB%20Footprint.dxf)
- [ESP32-S3-WROOM-2 3D Model (STEP)](https://www.espressif.com/sites/default/files/3dmodel/ESP32-S3-WROOM-2.STEP)

### 13.2 开发板

- [ESP32-S3-DevKitC-1](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/hw-reference/esp32s3/user-guide-devkitc-1.html)

### 13.3 购买渠道

| 渠道 | 链接 |
| :--- | :--- |
| 淘宝 | https://item.taobao.com/item.htm?ft=t&id=659527736594 |
| Mouser | https://www.mouser.com/ProductDetail/Espressif-Systems/ESP32-S3-WROOM-2-N32R16V |
| Digi-Key | https://www.digikey.com/en/products/detail/espressif-systems/ESP32-S3-WROOM-2-N32R16V |
| AliExpress | https://www.aliexpress.com/item/1005006334720108.html |
