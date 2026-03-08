# Day7 预处理指令+枚举（#define/enum/#ifndef）- 车载代码工程化核心
## 1. 核心知识点
- #define宏定义：文本替换型常量定义，语法`#define 宏名 常量/表达式`（末尾不加;），可封装引脚、状态、表达式等；
- enum枚举：批量定义关联状态/错误码，语法`enum 枚举名 {枚举值1, 枚举值2}`，枚举值默认从0递增，可显式赋值；
- #ifndef头文件保护：工程化必备，核心模板`#ifndef _文件名_H_ → #define _文件名_H_ → 头文件内容 → #endif`，防止重复包含。

## 2. 踩坑记录
- 宏定义错误：末尾加`;`导致编译报错；表达式宏未加括号（如`#define NUM 10+2`），运算优先级出错；
- 枚举错误：枚举值重复赋值（如`enum {A=1, B=1}`）引发编译冲突；未显式赋值时误解枚举值默认规律；
- 头文件保护错误：宏名重复（不同头文件用同一保护宏）导致保护失效；头文件写函数实现而非声明，引发重定义错误。

## 3. 车载关联
- 宏定义：STM32外设引脚（如`#define LED_PIN 2`）、故障码、硬件参数（如闪烁间隔）的标准化定义，改参数仅需修改宏；
- 枚举：车载外设状态（车灯OFF/ON/BLINK）、故障码（ERR_NONE/ERR_PIN）的强语义定义，提升代码可读性与协作效率；
- 头文件保护：车载多文件开发（如CarLed.h/CAN.h）的标配，避免头文件重复包含导致编译错误，适配工程化开发规范。

### #define的基础用法
```c
#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>

// ===== 核心：用宏封装车载车灯常量 =====
#define BRAKE_LED_PIN 2    // 刹车灯引脚
#define REVERSE_LED_PIN 3  // 倒车灯引脚
#define LED_ON 1           // 灯亮状态
#define LED_OFF 0          // 灯灭状态
#define PB_PREFIX "PB"     // 引脚前缀

int main() {
    // 直接用宏，不用记数字
    printf("刹车灯引脚：%s%d，默认状态：%d（亮）\n", PB_PREFIX, BRAKE_LED_PIN, LED_ON);
    printf("倒车灯引脚：%s%d，默认状态：%d（灭）\n", PB_PREFIX, REVERSE_LED_PIN, LED_OFF);

    // 改参数只需改宏定义，不用改业务代码（车载工程化核心）
    return 0;
}
```

### enum的基础用法
```c
#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>

// ===== 核心：枚举定义车灯状态 =====
enum LedStatus {
    LED_OFF = 0,   // 显式赋值，更清晰
    LED_ON = 1,
    LED_BLINK = 2
};

// 枚举定义故障码
enum LedErrCode {
    ERR_NONE = 0,    // 无故障
    ERR_PIN = 1,     // 引脚错误
    ERR_POWER = 2    // 供电错误
};

int main() {
    // 用枚举变量存储状态，代码可读性拉满
    enum LedStatus brake_led_status = LED_ON;
    enum LedErrCode err = ERR_NONE;

    printf("刹车灯状态：%d（%s）\n", brake_led_status,
        brake_led_status == LED_ON ? "亮" : "灭");
    printf("故障码：%d（%s）\n", err,
        err == ERR_NONE ? "无故障" : "异常");

    // 枚举也能作为函数参数（车载驱动常用）
    return 0;
}
```
- enum赋值中间用`,`

### #ifndef的基础用法
`main.c`
```c
#define _CRT_SECURE_NO_WARNINGS
#include"CarLed.h"

int main() {
	set_led_status(BRAKE_LED_PIN, LED_ON);
	set_led_status(REVERSE_LED_PIN, LED_OFF);

	return 0;
}
```
`CarLed.c`
```c
#include"CarLed.h"
#include<stdio.h>

void set_led_status(int pin, enum LedStatus status) {
	printf("设置PB%d车灯状态：%d(%s)\n", pin, status,
		status == LED_ON ? "亮" : "灭");
}
```
`CarLed.h`
```c
#ifndef _CAR_LED_H
#define _CAR_LED_H

#define BRAKE_LED_PIN 2
#define REVERSE_LED_PIN 3

enum LedStatus {
	LED_OFF = 0,
	LED_ON = 1
};

void set_led_status(int pin, enum LedStatus status);

#endif
```
