# Day4 结构体 + 结构体嵌套函数指针（车载驱动核心）
## 1. 核心知识点
- 结构体：封装外设的属性（pin/name）和行为（函数指针），实现对象化；
- 结构体数组：批量管理多外设（车灯/蜂鸣器），新增外设只需扩展数组；
- sizeof公式：`sizeof(数组)/sizeof(数组[0])`计算元素个数，避免硬编码。

## 2. 踩坑记录
- 结构体初始化用`.成员名=值`，避免顺序错乱；
- 函数指针类型必须和绑定的函数完全匹配（返回值+参数）；
- 数组传参后会退化成指针，不能用sizeof算长度（需手动传）。

## 3. 车载关联
- 结构体+函数指针：STM32外设驱动（GPIO/CAN/UART）的标准写法；
- 结构体数组：车载MCU批量初始化/控制外设（车灯/蜂鸣器/传感器）。

### typedef结构体基本用法
```c
#include <stdio.h>
#include <stdint.h>

// 1. 定义结构体：打包车载车灯的所有属性
typedef struct {
    uint8_t pin;        // 引脚号（比如PB0=0）
    uint8_t status;     // 状态（0=灭，1=亮）
    char name[20];      // 车灯名称（左车灯/右车灯）
} CarLed; // 给结构体起别名（车载命名规范：CarXXX）

int main() {
    // 2. 创建结构体变量并初始化（车载左车灯）
    CarLed left_led = {
        .pin = 0,        // PB0
        .status = 0,     // 初始灭
        .name = "左车灯" // 名称
    };

    // 3. 访问结构体成员（. 操作符）
    printf("车载场景：%s（PB%d）当前状态：%s\n",
        left_led.name,
        left_led.pin,
        left_led.status == 0 ? "灭" : "亮");

    // 4. 修改结构体成员（控制车灯亮）
    left_led.status = 1;
    printf("车载场景：%s（PB%d）当前状态：%s\n",
        left_led.name,
        left_led.pin,
        left_led.status == 0 ? "灭" : "亮");

    return 0;
}
```
- typedef第一次赋值时中间用`,`而不是`;`

### 结构体嵌套函数指针
```c
#include <stdio.h>
#include <stdint.h>

// 1. 先定义函数指针类型：车灯的初始化/控制函数
typedef void (*LedInitFunc)(uint8_t);  // 初始化函数：参数=引脚
typedef void (*LedCtrlFunc)(uint8_t);  // 控制函数：参数=引脚

// 2. 结构体封装：车灯的属性 + 行为（车载驱动核心）
typedef struct {
    uint8_t pin;          // 属性：引脚号
    char name[20];        // 属性：名称
    LedInitFunc init;     // 行为：初始化函数（函数指针）
    LedCtrlFunc on;       // 行为：开灯函数（函数指针）
    LedCtrlFunc off;      // 行为：关灯函数（函数指针）
} CarLed;

// 3. 实现车灯的具体函数（行为）
void led_init(uint8_t pin) {
    printf("车载场景：%s初始化（PB%d）→ 开时钟+配置推挽输出\n", __func__, pin);
}

void led_on(uint8_t pin) {
    printf("车载场景：开灯（PB%d）→ GPIO_SetBits\n", pin);
}

void led_off(uint8_t pin) {
    printf("车载场景：关灯（PB%d）→ GPIO_ResetBits\n", pin);
}

int main() {
    // 4. 创建车灯对象：绑定属性+行为（车载驱动注册）
    CarLed left_led = {
        .pin = 0,
        .name = "左车灯",
        .init = led_init,  // 绑定初始化函数
        .on = led_on,      // 绑定开灯函数
        .off = led_off     // 绑定关灯函数
    };

    // 5. 调用结构体里的函数指针（控制车灯）
    left_led.init(left_led.pin);  // 初始化左车灯
    left_led.on(left_led.pin);    // 开灯
    left_led.off(left_led.pin);   // 关灯

    return 0;
}
```
- `printf("%s\n", __func__);`输出函数名称

### 进阶实战：结构体数组管理多个车载外设
```c
#include <stdio.h>
#include <stdint.h>

// 1. 先定义函数指针类型：车灯的初始化/控制函数
typedef void (*LedInitFunc)(uint8_t);  // 初始化函数：参数=引脚
typedef void (*LedCtrlFunc)(uint8_t);  // 控制函数：参数=引脚

// 2. 结构体封装：车灯的属性 + 行为（车载驱动核心）
typedef struct {
    uint8_t pin;          // 属性：引脚号
    char name[20];        // 属性：名称
    LedInitFunc init;     // 行为：初始化函数（函数指针）
    LedCtrlFunc on;       // 行为：开灯函数（函数指针）
    LedCtrlFunc off;      // 行为：关灯函数（函数指针）
} CarLed;

// 3. 实现车灯的具体函数（行为）
void led_init(uint8_t pin) {
    printf("车载场景：%s初始化（PB%d）→ 开时钟+配置推挽输出\n", __func__, pin);
}

void led_on(uint8_t pin) {
    printf("车载场景：开灯（PB%d）→ GPIO_SetBits\n", pin);
}

void led_off(uint8_t pin) {
    printf("车载场景：关灯（PB%d）→ GPIO_ResetBits\n", pin);
}

int main() {
    // 1. 创建多个车灯对象
    CarLed left_led = { 0, "左车灯", led_init, led_on, led_off };
    CarLed right_led = { 1, "右车灯", led_init, led_on, led_off };

    // 2. 结构体数组：批量管理所有车灯
    CarLed car_leds[] = { left_led, right_led };
    uint8_t led_num = sizeof(car_leds) / sizeof(car_leds[0]); // 外设数量

    // 3. 批量初始化+控制所有车灯（车载启动流程）
    for (uint8_t i = 0; i < led_num; i++) {
        printf("===== 控制%s =====\n", car_leds[i].name);
        car_leds[i].init(car_leds[i].pin); // 批量初始化
        car_leds[i].on(car_leds[i].pin);   // 批量开灯
        car_leds[i].off(car_leds[i].pin);  // 批量关灯
    }

    return 0;
}
```
