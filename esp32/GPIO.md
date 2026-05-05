# ESP32-IDF GPIO 基础操作指南
## 1. 包含头文件
在使用 GPIO 相关的 API 之前，必须在文件头部引入驱动库：
```c
#include "driver/gpio.h"
```
## 2. GPIO 初始化配置
### 核心配置函数
```c
esp_err_t gpio_config(const gpio_config_t *pGPIOConfig);
```
### 对应的结构体 (`gpio_config_t`)
```c
typedef struct {
    uint64_t pin_bit_mask;       // 设置位掩码 (例如：1ULL << GPIO_NUM_X)
    gpio_mode_t mode;            // 设置工作模式 (输入/输出等)
    gpio_pullup_t pull_up_en;    // 上拉使能
    gpio_pulldown_t pull_down_en;// 下拉使能
    gpio_int_type_t intr_type;   // 中断类型
} gpio_config_t;
```
### 字段解释与实用技巧
* **`pin_bit_mask` (位掩码)**：用于指定要配置的引脚。
    * 配置单个引脚：`1ULL << GPIO_NUM_X` 或 `1ULL << X`。
    * **注**：可以通过按位或 `|` 同时配置多个引脚，例如 `(1ULL << GPIO_NUM_18) | (1ULL << GPIO_NUM_19)`。
* **`pull_up_en` (上拉使能)**：开启后，引脚在**悬空状态**下会被拉高到高电平。
* **`pull_down_en` (下拉使能)**：开启后，引脚在**悬空状态**下会被拉低到低电平。
* **`intr_type` (中断类型)**：设置为上升沿、下降沿或任意电平触发等。不需要中断时填 `GPIO_INTR_DISABLE`。
* `gpio_mode_t`相关介绍可以看下面的补充
* **快捷键**：按住 `Ctrl` + 鼠标左键 点击结构体、宏定义或枚举值，可以快速跳转到底层查看官方注释。
* 注意：在初始化了之后记得传参
  样例:
```c
void key_init(int gpio)
{
gpio_config_t gpio_init_struct = { 0 };
gpio_init_struct.intr_type = GPIO_INTR_DISABLE;
gpio_init_struct.mode = GPIO_MODE_INPUT;
gpio_init_struct.pull_up_en = 1;
gpio_init_struct.pull_down_en = 0;
gpio_init_struct.pin_bit_mask = 1ULL << gpio;
gpio_config(&gpio_init_struct);//传参
}
```

---
## 3. 设置输出电平 (Write)
**核心函数：**
```c
esp_err_t gpio_set_level(gpio_num_t gpio_num, uint32_t level);
```

**参数说明：**
* `gpio_num`：操作的引脚名称（标准格式为 `GPIO_NUM_X`，X 为具体的引脚号）。
* `level`：输出的电平值。`0` 表示低电平，`1` 表示高电平。
---
## 4. 读取输入电平 (Read)
**核心函数：**
```c
int gpio_get_level(gpio_num_t gpio_num);
```
**使用说明：**
* 返回值：`0` 或 `1`，代表当前引脚的实际电平状态。
* **注**：要正确读取外部电平，该引脚在初始化时，`mode` 必须配置为包含输入的模式（如 `GPIO_MODE_INPUT` 或 `GPIO_MODE_INPUT_OUTPUT`）。
---
## 补充：
1.在对引脚进行 `gpio_config` 复杂配置或复用之前，建议先调用重置函数。这会将引脚恢复到默认状态（通常是禁用输出/输入，开启上拉），避免被之前的程序或引导加载程序 (Bootloader) 残留的配置干扰：
```c
gpio_reset_pin(GPIO_NUM_X);
```
2.关于gpio_mode_t

| 模式                          | 含义      | 硬件行为                                                |
| --------------------------- | ------- | --------------------------------------------------- |
| `GPIO_MODE_INPUT`           | 仅输入     | 只能读，不能写。                                            |
| `GPIO_MODE_OUTPUT`          | 推挽输出    | 既能输出高电平，也能输出低电平。                                    |
| `GPIO_MODE_OUTPUT_OD`       | 开漏输出    | **只能输出低电平**。输出高电平时，内部处于高阻态（断开）。通常用于 I2C 或多个设备共用信号线。 |
| `GPIO_MODE_INPUT_OUTPUT`    | 输入+推挽输出 | 既能驱动引脚，又能通过读取引脚来确认当前真实状态。                           |
| `GPIO_MODE_INPUT_OUTPUT_OD` | 输入+开漏输出 | 允许读取引脚电平，同时以开漏方式驱动。                                 |
# GPIO中断相关操作
## 创建中断流程
1.在gpio_config()中配置中断引脚
2.通过`gpio_install_isr_service()`创建中断服务
3.通过`gpio_isr_handler_add()`添加中断处理函数
4.通过`gpio_intr_enable()`使能中断
