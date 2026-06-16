# ESP32-S3 编程指南 (ESP-IDF Programming Guide)

> 来源: https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/index.html

---

## 1. ESP-IDF 概述

ESP-IDF (Espressif IoT Development Framework) 是乐鑫官方的物联网开发框架，用于开发 ESP32、ESP32-S、ESP32-C、ESP32-H 和 ESP32-P 系列 SoC 的应用程序。

ESP32-S3 使用 ESP-IDF 框架进行开发，本指南介绍如何使用 ESP-IDF 开发 ESP32-S3 应用程序。

### 1.1 文档结构

| 模块 | 内容 |
|------|------|
| Get Started (入门指南) | 环境搭建、Hello World 示例 |
| API Reference (API 参考) | 完整的 API 文档 |
| API Guides (API 指南) | 使用指南和最佳实践 |

---

## 2. 开发环境搭建

### 2.1 系统要求

ESP-IDF 支持 Windows、Linux 和 macOS 操作系统。

### 2.2 安装步骤

1. **获取 ESP-IDF**
   ```bash
   git clone --recursive https://github.com/espressif/esp-idf.git
   ```

2. **设置工具链**
   ```bash
   cd esp-idf
   ./install.sh esp32s3
   ```

3. **设置环境变量**
   ```bash
   . ./export.sh
   ```

### 2.3 目标芯片设置

```bash
idf.py set-target esp32s3
```

---

## 3. 构建系统 (CMake)

ESP-IDF 使用 CMake 作为构建系统。

### 3.1 项目结构

```
my_project/
├── CMakeLists.txt
├── sdkconfig
├── components/
│   └── my_component/
│       ├── CMakeLists.txt
│       ├── include/
│       └── src/
└── main/
    ├── CMakeLists.txt
    └── main.c
```

### 3.2 项目级 CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.16)

include($ENV{IDF_PATH}/tools/cmake/project.cmake)
project(my-esp32s3-project)
```

### 3.3 组件级 CMakeLists.txt

```cmake
idf_component_register(SRCS "my_component.c"
                       INCLUDE_DIRS "include"
                       REQUIRES driver)
```

### 3.4 构建命令

```bash
# 配置项目
idf.py menuconfig

# 编译项目
idf.py build

# 烧录固件
idf.py -p /dev/ttyUSB0 flash

# 监视串口输出
idf.py -p /dev/ttyUSB0 monitor

# 一键构建、烧录、监视
idf.py -p /dev/ttyUSB0 flash monitor
```

---

## 4. 核心 API

### 4.1 GPIO (通用输入输出)

GPIO 驱动提供配置和操作 GPIO 引脚的功能。

```c
#include "driver/gpio.h"

// 配置 GPIO 为输出
gpio_config_t io_conf = {
    .pin_bit_mask = (1ULL << GPIO_NUM_2),  // 选择 GPIO2
    .mode = GPIO_MODE_OUTPUT,               // 输出模式
    .pull_up_en = GPIO_PULLUP_DISABLE,
    .pull_down_en = GPIO_PULLDOWN_DISABLE,
    .intr_type = GPIO_INTR_DISABLE,
};
gpio_config(&io_conf);

// 设置输出电平
gpio_set_level(GPIO_NUM_2, 1);

// 配置 GPIO 为输入
io_conf.pin_bit_mask = (1ULL << GPIO_NUM_4);
io_conf.mode = GPIO_MODE_INPUT;
io_conf.pull_up_en = GPIO_PULLUP_ENABLE;
gpio_config(&io_conf);

// 读取输入电平
int level = gpio_get_level(GPIO_NUM_4);
```

#### GPIO 中断

```c
// 定义 ISR 处理函数
static void IRAM_ATTR gpio_isr_handler(void* arg) {
    uint32_t gpio_num = (uint32_t) arg;
    // 处理中断
}

// 配置中断
io_conf.intr_type = GPIO_INTR_NEGEDGE;  // 下降沿触发
gpio_config(&io_conf);

// 安装 ISR 服务
gpio_install_isr_service(0);
gpio_isr_handler_add(GPIO_NUM_4, gpio_isr_handler, (void*) GPIO_NUM_4);
```

---

### 4.2 UART (通用异步收发器)

ESP32-S3 有 3 个 UART 控制器 (UART0, UART1, UART2)。

```c
#include "driver/uart.h"

#define UART_PORT_NUM      UART_NUM_1
#define UART_BAUD_RATE     115200
#define UART_TX_PIN        17
#define UART_RX_PIN        18
#define BUF_SIZE           1024

// 配置 UART
uart_config_t uart_config = {
    .baud_rate = UART_BAUD_RATE,
    .data_bits = UART_DATA_8_BITS,
    .parity = UART_PARITY_DISABLE,
    .stop_bits = UART_STOP_BITS_1,
    .flow_ctrl = UART_HW_FLOWCTRL_DISABLE,
    .source_clk = UART_SCLK_DEFAULT,
};
uart_param_config(UART_PORT_NUM, &uart_config);

// 设置引脚
uart_set_pin(UART_PORT_NUM, UART_TX_PIN, UART_RX_PIN, 
             UART_PIN_NO_CHANGE, UART_PIN_NO_CHANGE);

// 安装驱动
uart_driver_install(UART_PORT_NUM, BUF_SIZE * 2, 0, 0, NULL, 0);

// 发送数据
const char* data = "Hello UART";
uart_write_bytes(UART_PORT_NUM, data, strlen(data));

// 接收数据
uint8_t* rx_data = (uint8_t*) malloc(BUF_SIZE);
int len = uart_read_bytes(UART_PORT_NUM, rx_data, BUF_SIZE, 100 / portTICK_PERIOD_MS);
```

---

### 4.3 I2C (集成电路总线)

ESP32-S3 有 2 个 I2C 总线接口，支持主/从模式。

```c
#include "driver/i2c.h"

#define I2C_MASTER_SCL_IO    9
#define I2C_MASTER_SDA_IO    8
#define I2C_MASTER_NUM       I2C_NUM_0
#define I2C_MASTER_FREQ_HZ   100000
#define I2C_MASTER_TX_BUF_DISABLE  0
#define I2C_MASTER_RX_BUF_DISABLE  0

// 初始化 I2C 主机
i2c_config_t conf = {
    .mode = I2C_MODE_MASTER,
    .sda_io_num = I2C_MASTER_SDA_IO,
    .scl_io_num = I2C_MASTER_SCL_IO,
    .sda_pullup_en = GPIO_PULLUP_ENABLE,
    .scl_pullup_en = GPIO_PULLUP_ENABLE,
    .master.clk_speed = I2C_MASTER_FREQ_HZ,
};
i2c_param_config(I2C_MASTER_NUM, &conf);
i2c_driver_install(I2C_MASTER_NUM, conf.mode, 
                   I2C_MASTER_RX_BUF_DISABLE, I2C_MASTER_TX_BUF_DISABLE, 0);

// 写数据到从机
i2c_cmd_handle_t cmd = i2c_cmd_link_create();
i2c_master_start(cmd);
i2c_master_write_byte(cmd, (slave_addr << 1) | I2C_MASTER_WRITE, true);
i2c_master_write(cmd, data_buf, data_len, true);
i2c_master_stop(cmd);
i2c_master_cmd_begin(I2C_MASTER_NUM, cmd, 1000 / portTICK_PERIOD_MS);
i2c_cmd_link_delete(cmd);
```

---

### 4.4 SPI (串行外设接口)

ESP32-S3 有 4 个 SPI 接口，其中 SPI2 和 SPI3 是通用 SPI 控制器。

```c
#include "driver/spi_master.h"

#define SPI_HOST     SPI2_HOST
#define PIN_NUM_MOSI 11
#define PIN_NUM_MISO 13
#define PIN_NUM_CLK  12
#define PIN_NUM_CS   10

// 初始化 SPI 主机
spi_bus_config_t buscfg = {
    .mosi_io_num = PIN_NUM_MOSI,
    .miso_io_num = PIN_NUM_MISO,
    .sclk_io_num = PIN_NUM_CLK,
    .quadwp_io_num = -1,
    .quadhd_io_num = -1,
    .max_transfer_sz = 4096,
};
spi_bus_initialize(SPI_HOST, &buscfg, SPI_DMA_CH_AUTO);

// 添加设备
spi_device_interface_config_t devcfg = {
    .clock_speed_hz = 10 * 1000 * 1000,  // 10 MHz
    .mode = 0,                             // SPI 模式 0
    .spics_io_num = PIN_NUM_CS,
    .queue_size = 7,
};
spi_device_handle_t spi;
spi_bus_add_device(SPI_HOST, &devcfg, &spi);

// 发送/接收数据
spi_transaction_t t = {
    .length = 8 * 4,            // 4 字节
    .tx_buffer = sendbuf,
    .rx_buffer = recvbuf,
};
spi_device_transmit(spi, &t);
```

---

### 4.5 I2S (集成电路音频)

ESP32-S3 有 2 个 I2S 接口，用于音频数据传输。

```c
#include "driver/i2s_std.h"

// I2S 标准模式配置
i2s_chan_handle_t tx_handle;
i2s_chan_config_t chan_cfg = I2S_CHANNEL_DEFAULT_CONFIG(I2S_NUM_0, I2S_ROLE_MASTER);
i2s_new_channel(&chan_cfg, &tx_handle, NULL);

i2s_std_clk_config_t clk_cfg = I2S_STD_CLK_DEFAULT_CONFIG(44100);
i2s_std_slot_config_t slot_cfg = I2S_STD_PHILIPS_SLOT_DEFAULT_CONFIG(I2S_DATA_BIT_WIDTH_16BIT, 
                                                                       I2S_SLOT_MODE_STEREO);
i2s_std_gpio_config_t gpio_cfg = {
    .bclk = 4,
    .ws = 5,
    .dout = 6,
    .din = 7,
    .mclk = I2S_GPIO_UNUSED,
};

i2s_std_config_t std_cfg = {
    .clk_cfg = &clk_cfg,
    .slot_cfg = &slot_cfg,
    .gpio_cfg = &gpio_cfg,
};
i2s_channel_init_std_mode(tx_handle, &std_cfg);
i2s_channel_enable(tx_handle);

// 写入音频数据
size_t bytes_written;
i2s_channel_write(tx_handle, audio_buffer, buffer_size, &bytes_written, portMAX_DELAY);
```

---

### 4.6 ADC (模数转换器)

ESP32-S3 有 2 个 12-bit SAR ADC。

```c
#include "esp_adc/adc_oneshot.h"

adc_oneshot_unit_handle_t adc_handle;
adc_oneshot_unit_init_cfg_t init_config = {
    .unit_id = ADC_UNIT_1,
};
adc_oneshot_new_unit(&init_config, &adc_handle);

adc_oneshot_chan_cfg_t config = {
    .atten = ADC_ATTEN_DB_12,
    .bitwidth = ADC_BITWIDTH_12,
};
adc_oneshot_config_channel(adc_handle, ADC_CHANNEL_0, &config);

// 读取 ADC 值
int adc_raw;
adc_oneshot_read(adc_handle, ADC_CHANNEL_0, &adc_raw);
```

---

### 4.7 LED PWM (LEDC)

LEDC 用于 LED 亮度控制和信号生成。

```c
#include "driver/ledc.h"

#define LEDC_TIMER              LEDC_TIMER_0
#define LEDC_MODE               LEDC_LOW_SPEED_MODE
#define LEDC_OUTPUT_IO          2
#define LEDC_CHANNEL            LEDC_CHANNEL_0
#define LEDC_DUTY_RES           LEDC_TIMER_13_BIT
#define LEDC_DUTY               (4096)
#define LEDC_FREQUENCY          (4000)

// 配置定时器
ledc_timer_config_t ledc_timer = {
    .speed_mode = LEDC_MODE,
    .timer_num = LEDC_TIMER,
    .duty_resolution = LEDC_DUTY_RES,
    .freq_hz = LEDC_FREQUENCY,
    .clk_cfg = LEDC_AUTO_CLK,
};
ledc_timer_config(&ledc_timer);

// 配置通道
ledc_channel_config_t ledc_channel = {
    .speed_mode = LEDC_MODE,
    .channel = LEDC_CHANNEL,
    .timer_sel = LEDC_TIMER,
    .intr_type = LEDC_INTR_DISABLE,
    .gpio_num = LEDC_OUTPUT_IO,
    .duty = 0,
    .hpoint = 0,
};
ledc_channel_config(&ledc_channel);

// 设置占空比
ledc_set_duty(LEDC_MODE, LEDC_CHANNEL, LEDC_DUTY);
ledc_update_duty(LEDC_MODE, LEDC_CHANNEL);
```

---

### 4.8 定时器 (GPTimer)

```c
#include "driver/gptimer.h"

gptimer_handle_t timer = NULL;
gptimer_config_t timer_config = {
    .clk_src = GPTIMER_CLK_SRC_DEFAULT,
    .direction = GPTIMER_COUNT_UP,
    .resolution_hz = 1000000, // 1 MHz, 1 tick = 1 us
};
gptimer_new_timer(&timer_config, &timer);

// 定时器回调
static bool IRAM_ATTR timer_callback(gptimer_handle_t timer, 
                                      const gptimer_alarm_event_data_t *edata, 
                                      void *user_data) {
    // 定时器到期处理
    return true;
}

gptimer_event_callbacks_t cbs = {
    .on_alarm = timer_callback,
};
gptimer_register_event_callbacks(timer, &cbs, NULL);

gptimer_alarm_config_t alarm_config = {
    .reload_count = 0,
    .alarm_count = 1000000, // 1 秒
    .flags.auto_reload_on_alarm = true,
};
gptimer_set_alarm_action(timer, &alarm_config);

gptimer_enable(timer);
gptimer_start(timer);
```

---

### 4.9 触摸传感器

ESP32-S3 支持 14 个电容触摸引脚。

```c
#include "driver/touch_sensor.h"

// 初始化触摸传感器
touch_sensor_config_t touch_cfg = TOUCH_SENSOR_DEFAULT_CONFIG();
touch_sensor_initialize(&touch_cfg);

// 配置触摸通道
touch_sensor_channel_config_t ch_cfg = {
    .channel_num = TOUCH_PAD_NUM0,
    .charge_speed = TOUCH_CHARGE_SPEED_8V,
    .charge_times = 10,
};
touch_sensor_config_channel(&ch_cfg);
```

---

## 5. Wi-Fi 编程

### 5.1 Wi-Fi 模式

ESP32-S3 Wi-Fi 支持以下模式：
- **Station 模式**: 连接到 AP
- **SoftAP 模式**: 作为热点
- **SoftAP + Station 模式**: 同时作为 Station 和热点
- **Promiscuous 模式**: 监听模式

### 5.2 Station 模式连接

```c
#include "esp_wifi.h"
#include "esp_event.h"
#include "nvs_flash.h"

// 初始化 NVS
nvs_flash_init();

// 初始化网络接口
esp_netif_init();

// 创建默认事件循环
esp_event_loop_create_default();

// 创建默认 Station
esp_netif_create_default_wifi_sta();

// Wi-Fi 初始化
wifi_init_config_t cfg = WIFI_INIT_CONFIG_DEFAULT();
esp_wifi_init(&cfg);

// 配置 Wi-Fi
wifi_config_t wifi_config = {
    .sta = {
        .ssid = "your_ssid",
        .password = "your_password",
    },
};
esp_wifi_set_mode(WIFI_MODE_STA);
esp_wifi_set_config(WIFI_IF_STA, &wifi_config);
esp_wifi_start();
```

### 5.3 Wi-Fi 省电模式

```c
// 最小省电模式 (默认)
esp_wifi_set_ps(WIFI_PS_MIN_MODEM);

// 最大省电模式
esp_wifi_set_ps(WIFI_PS_MAX_MODEM);

// 关闭省电模式
esp_wifi_set_ps(WIFI_PS_NONE);
```

| 省电模式 | 描述 |
|---------|------|
| `WIFI_PS_NONE` | 不省电，最低延迟 |
| `WIFI_PS_MIN_MODEM` | 最小省电，每个 DTIM 唤醒 |
| `WIFI_PS_MAX_MODEM` | 最大省电，每个监听间隔唤醒 |

---

## 6. Bluetooth (蓝牙)

ESP32-S3 支持 Bluetooth LE 5.0 和 Bluetooth mesh。

### 6.1 NimBLE 协议栈

NimBLE 是推荐的轻量级 Bluetooth LE 协议栈。

### 6.2 主要功能

- **Controller & HCI**: 控制器和主机控制接口
- **Bluetooth Common**: 通用蓝牙功能
- **Bluetooth Low Energy (BLE)**: 低功耗蓝牙
- **NimBLE Host APIs**: NimBLE 主机 API
- **ESP-BLE-MESH**: BLE Mesh 网络

---

## 7. 电源管理

### 7.1 电源模式

ESP32-S3 支持以下电源模式：

| 模式 | 描述 | 典型功耗 |
|------|------|---------|
| **Active** | CPU、RF 和所有外设开启 | ~50 mA |
| **Modem-sleep** | CPU 开启，可降低时钟频率，RF 周期性开启 | ~15-25 mA |
| **Light-sleep** | CPU 暂停，保留状态，可快速唤醒 | ~240 µA |
| **Deep-sleep** | 仅 RTC 开启，唤醒后重启 | ~7-8 µA |
| **Power off** | 芯片完全关闭 | ~1 µA |

### 7.2 Modem-sleep 模式

Modem-sleep 模式基于 DTIM 机制，自动进入和退出休眠：

```
┌───────────┐      Wi-Fi 任务完成      ┌───────────┐
│           ├──────────────────────────►│   modem   │
│   active  │                           │   sleep   │
│           │◄──────────────────────────┤           │
└───────────┘   DTIM 间隔触发/发送数据  └───────────┘
```

### 7.3 Light-sleep 模式

```c
#include "esp_sleep.h"

// 配置 GPIO 唤醒源
esp_sleep_enable_gpio_wakeup();

// 进入 Light-sleep
esp_light_sleep_start();
```

在 Light-sleep 模式下：
- 数字外设、大部分 RAM 和 CPU 时钟关闭
- 供电电压降低
- 退出后恢复运行，内部状态保留

### 7.4 Deep-sleep 模式

```c
#include "esp_sleep.h"

// 配置唤醒源 (定时器唤醒，10 秒)
esp_sleep_enable_timer_wakeup(10 * 1000000ULL);

// 配置 GPIO 唤醒源
esp_sleep_enable_ext0_wakeup(GPIO_NUM_0, 0);

// 进入 Deep-sleep
esp_deep_sleep_start();
```

在 Deep-sleep 模式下：
- CPU、大部分 RAM 和数字外设关闭
- 仅 RTC 控制器、ULP 协处理器、RTC FAST/SLOW memory 保持供电
- 唤醒后重新运行 bootloader

### 7.5 唤醒源

| 唤醒源 | Light-sleep | Deep-sleep | API |
|--------|------------|------------|-----|
| RTC 定时器 | ✓ | ✓ | `esp_sleep_enable_timer_wakeup()` |
| GPIO (EXT0) | ✓ | ✓ | `esp_sleep_enable_ext0_wakeup()` |
| GPIO (EXT1) | ✓ | ✓ | `esp_sleep_enable_ext1_wakeup()` |
| GPIO (普通) | ✓ | ✗ | `esp_sleep_enable_gpio_wakeup()` |
| Touch 唤醒 | ✓ | ✓ | `esp_sleep_enable_touchpad_wakeup()` |
| UART 唤醒 | ✓ | ✗ | `esp_sleep_enable_uart_wakeup()` |
| Wi-Fi MAC | ✓ | ✗ | `esp_sleep_enable_wifi_wakeup()` |

### 7.6 电源域配置

```c
// 配置电源域
esp_sleep_pd_config(ESP_PD_DOMAIN_RTC_PERIPH, ESP_PD_OPTION_OFF);
esp_sleep_pd_config(ESP_PD_DOMAIN_RTC_SLOW_MEM, ESP_PD_OPTION_ON);
esp_sleep_pd_config(ESP_PD_DOMAIN_RTC_FAST_MEM, ESP_PD_OPTION_ON);
```

| 选项 | 描述 |
|------|------|
| `ESP_PD_OPTION_OFF` | 强制关闭 |
| `ESP_PD_OPTION_ON` | 强制开启 |
| `ESP_PD_OPTION_AUTO` | 自动管理 (默认) |

---

## 8. OTA 固件升级

ESP-IDF 支持 OTA (Over-The-Air) 固件升级。

### 8.1 主要 API

```c
#include "esp_ota_ops.h"
#include "esp_https_ota.h"

// HTTPS OTA 升级
esp_https_ota_config_t ota_config = {
    .http_config = &http_config,
};
esp_err_t ret = esp_https_ota(&ota_config);
if (ret == ESP_OK) {
    esp_restart();
}
```

### 8.2 相关组件

- **OTA**: 基本 OTA 功能
- **ESP HTTPS OTA**: 基于 HTTPS 的 OTA
- **App Update**: 应用更新 API

---

## 9. 安全特性

### 9.1 Flash 加密

ESP32-S3 支持 AES-XTS-based Flash 加密，保护存储在 Flash 中的代码和数据。

### 9.2 安全启动 (Secure Boot)

RSA-based 安全启动确保只运行经过签名的固件。

### 9.3 数字签名外设 (DS)

```c
#include "esp_ds.h"

// 使用数字签名外设进行 RSA 签名
```

### 9.4 HMAC

```c
#include "esp_hmac.h"

// 使用 HMAC 硬件加速器
```

### 9.5 加密加速器

- **AES 加速器**: AES 加密/解密
- **RSA 加速器**: RSA 运算
- **SHA 加速器**: SHA 哈希运算
- **HMAC 加速器**: HMAC 运算

---

## 10. 存储系统

### 10.1 NVS (非易失性存储)

```c
#include "nvs_flash.h"
#include "nvs.h"

nvs_handle_t my_handle;
nvs_open("storage", NVS_READWRITE, &my_handle);

// 写入数据
nvs_set_i32(my_handle, "restart_count", 0);

// 读取数据
int32_t restart_count = 0;
nvs_get_i32(my_handle, "restart_count", &restart_count);

// 提交并关闭
nvs_commit(my_handle);
nvs_close(my_handle);
```

### 10.2 SPIFFS 文件系统

```c
#include "esp_spiffs.h"

esp_vfs_spiffs_conf_t conf = {
    .base_path = "/spiffs",
    .partition_label = NULL,
    .max_files = 5,
    .format_if_mount_failed = true,
};
esp_vfs_spiffs_register(&conf);
```

### 10.3 FAT 文件系统

```c
#include "esp_vfs_fat.h"

esp_vfs_fat_sdmmc_mount_config_t mount_config = {
    .format_if_mount_failed = true,
    .max_files = 5,
    .allocation_unit_size = 16 * 1024,
};
```

---

## 11. FreeRTOS 实时操作系统

ESP-IDF 基于 FreeRTOS，提供多任务支持。

### 11.1 任务创建

```c
void task_function(void* pvParameters) {
    while (1) {
        // 任务代码
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}

// 创建任务
xTaskCreate(task_function, "task_name", 4096, NULL, 5, NULL);
```

### 11.2 队列

```c
QueueHandle_t queue = xQueueCreate(10, sizeof(uint32_t));

// 发送数据
uint32_t data = 100;
xQueueSend(queue, &data, portMAX_DELAY);

// 接收数据
uint32_t received;
xQueueReceive(queue, &received, portMAX_DELAY);
```

### 11.3 信号量

```c
SemaphoreHandle_t sem = xSemaphoreCreateBinary();

// 释放信号量
xSemaphoreGive(sem);

// 获取信号量
xSemaphoreTake(sem, portMAX_DELAY);
```

### 11.4 互斥量

```c
SemaphoreHandle_t mutex = xSemaphoreCreateMutex();

xSemaphoreTake(mutex, portMAX_DELAY);
// 临界区
xSemaphoreGive(mutex);
```

---

## 12. ULP 协处理器编程

### 12.1 ULP-RISC-V

```c
#include "ulp_riscv.h"
#include "ulp_main.h"

// 加载 ULP-RISC-V 程序
ulp_riscv_config_t cfg = {
    .wakeup_period_us = 10000,
};
ulp_riscv_load_binary(ulp_main_bin_start, 
                      (ulp_main_bin_end - ulp_main_bin_start));
ulp_riscv_run(&cfg);
```

### 12.2 ULP-FSM

```c
#include "ulp.h"

// 加载 ULP-FSM 程序
ulp_load_binary(0, ulp_main_bin_start, 
                (ulp_main_bin_end - ulp_main_bin_start) / sizeof(uint32_t));
ulp_set_wakeup_period(0, 1000);
ulp_run(0);
```

---

## 13. USB 外设

### 13.1 USB Serial/JTAG

内置 USB Serial/JTAG 控制器，无需外部芯片即可进行编程和调试。

### 13.2 USB OTG

ESP32-S3 集成全速 USB 2.0 OTG 控制器：
- **USB Host**: USB 主机模式
- **USB Device**: USB 设备模式

---

## 14. 网络协议

### 14.1 MQTT

```c
#include "mqtt_client.h"

esp_mqtt_client_config_t mqtt_cfg = {
    .broker.address.uri = "mqtt://broker.example.com",
};
esp_mqtt_client_handle_t client = esp_mqtt_client_init(&mqtt_cfg);
esp_mqtt_client_register_event(client, ESP_EVENT_ANY_ID, mqtt_event_handler, NULL);
esp_mqtt_client_start(client);
```

### 14.2 HTTP 客户端

```c
#include "esp_http_client.h"

esp_http_client_config_t config = {
    .url = "http://example.com",
    .event_handler = http_event_handler,
};
esp_http_client_handle_t client = esp_http_client_init(&config);
esp_http_client_perform(client);
esp_http_client_cleanup(client);
```

### 14.3 HTTP 服务器

```c
#include "esp_http_server.h"

esp_err_t get_handler(httpd_req_t *req) {
    const char* resp = "Hello World";
    httpd_resp_send(req, resp, HTTPD_RESP_USE_STRLEN);
    return ESP_OK;
}

httpd_uri_t uri_get = {
    .uri = "/",
    .method = HTTP_GET,
    .handler = get_handler,
};

httpd_config_t config = HTTPD_DEFAULT_CONFIG();
httpd_handle_t server = NULL;
httpd_start(&server, &config);
httpd_register_uri_handler(server, &uri_get);
```

---

## 15. 完整 API 列表

### 15.1 外设 API (Peripherals API)

| API | 描述 |
|-----|------|
| ADC | 模数转换器 |
| Asynchronous Memory Copy | 异步内存拷贝 |
| Clock Tree | 时钟树 |
| GPIO & RTC GPIO | GPIO 控制 |
| GPTimer | 通用定时器 |
| Dedicated GPIO | 专用 GPIO |
| HMAC | 哈希消息认证码 |
| RSA_DS | RSA 数字签名 |
| I2C | I2C 接口 |
| I2S | I2S 音频接口 |
| LCD | LCD 显示接口 |
| LEDC | LED PWM 控制 |
| MCPWM | 电机控制 PWM |
| PCNT | 脉冲计数器 |
| RMT | 远程控制收发器 |
| SDMMC Host | SD/MMC 主机驱动 |
| SD SPI Host | SD SPI 主机驱动 |
| SDM | Sigma-Delta 调制 |
| SPI Flash | SPI Flash API |
| SPI Master | SPI 主机驱动 |
| SPI Slave | SPI 从机驱动 |
| Temperature Sensor | 温度传感器 |
| Capacitive Touch Sensor | 电容触摸传感器 |
| TWAI | 两线汽车接口 (CAN) |
| UART | 通用异步收发器 |
| USB Device | USB 设备栈 |
| USB Host | USB 主机 |

### 15.2 系统 API (System API)

| API | 描述 |
|-----|------|
| Console | 控制台 |
| eFuse Manager | eFuse 管理 |
| ESP HTTPS OTA | HTTPS OTA 升级 |
| Event Loop | 事件循环 |
| FreeRTOS | 实时操作系统 |
| Heap Memory | 堆内存管理 |
| ESP Timer | 高精度定时器 |
| Interrupt Allocation | 中断分配 |
| Logging | 日志系统 |
| OTA | 空中升级 |
| Power Management | 电源管理 |
| Sleep Modes | 休眠模式 |
| System Time | 系统时间 |
| ULP Coprocessor | 超低功耗协处理器 |
| Watchdogs | 看门狗定时器 |

### 15.3 网络 API (Networking APIs)

| API | 描述 |
|-----|------|
| Wi-Fi | Wi-Fi 连接 |
| Ethernet | 以太网 |
| ESP-NETIF | 网络接口 |
| Thread | Thread 协议 |

### 15.4 蓝牙 API (Bluetooth API)

| API | 描述 |
|-----|------|
| Controller & HCI | 控制器和 HCI |
| Bluetooth Common | 通用蓝牙 |
| Bluetooth LE | 低功耗蓝牙 |
| NimBLE Host | NimBLE 主机 |
| ESP-BLE-MESH | BLE Mesh |

### 15.5 应用协议 API

| API | 描述 |
|-----|------|
| ESP-MQTT | MQTT 客户端 |
| ESP-TLS | TLS 连接 |
| ESP HTTP Client | HTTP 客户端 |
| HTTP Server | HTTP 服务器 |
| mDNS | mDNS 服务 |
| Mbed TLS | TLS/SSL 库 |
| ESP-Modbus | Modbus 协议 |
| ASIO Port | ASIO 端口 |

### 15.6 存储 API (Storage API)

| API | 描述 |
|-----|------|
| FAT Filesystem | FAT 文件系统 |
| NVS | 非易失性存储 |
| SPIFFS | SPIFFS 文件系统 |
| VFS | 虚拟文件系统 |
| Wear Levelling | 磨损均衡 |
| Partition | 分区表 |

---

## 16. 硬件抽象层 (Hardware Abstraction)

ESP-IDF 提供硬件抽象 API，支持不同层级的硬件控制：

1. **LL (Low Level) Layer**: 底层寄存器操作
2. **HAL (Hardware Abstraction Layer)**: 硬件抽象层
3. **Driver Layer**: 高层驱动接口

**注意:** 硬件抽象 API (除 driver 和 xxx_types.h 外) 是实验性功能，可能在不同版本间变更。

---

## 17. 相关文档

- [ESP-IDF Get Started (入门指南)](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/get-started/index.html)
- [ESP-IDF API Reference (API 参考)](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/api-reference/index.html)
- [ESP-IDF API Guides (API 指南)](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/api-guides/index.html)
- [Low Power Mode Guide (低功耗指南)](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/api-guides/low-power-mode/index.html)
- [Wi-Fi Driver Guide (Wi-Fi 驱动指南)](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/api-guides/wifi/index.html)
- [ESP32-S3 Datasheet (数据手册)](https://documentation.espressif.com/esp32-s3_datasheet_en.html)
- [ESP32-S3 Technical Reference Manual (技术参考手册)](https://www.espressif.com/sites/default/files/documentation/esp32-s3_technical_reference_manual_en.pdf)
- [GitHub: ESP-IDF](https://github.com/espressif/esp-idf)
- [ESP-IDF 中文文档](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/index.html)
