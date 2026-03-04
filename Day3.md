
- 指针函数：返回值是指针的函数
- 指针函数的返回值不要返回局部变量的地址
- 函数指针：指向函数的指针————函数返回值类型（*函数指针名）（参数列表）

### 函数指针 + typedef（车载C语言核心）
```c
#include <stdio.h>
#include <stdint.h>

void led_left(uint8_t pin) {
    printf("车载场景：左车灯（PB%d）亮\n", pin);
}
void led_right(uint8_t pin) {
    printf("车载场景：右车灯（PB%d）亮\n", pin);
}
void buzzer(uint8_t pin) {
    printf("车载场景：蜂鸣器（PB%d）响\n", pin);
}

// 给函数指针起外号（简化语法）
typedef void (*VehicleFunc)(uint8_t);

int main() {
    VehicleFunc func_p[] = { led_left, led_right, buzzer }; // 函数指针数组
    uint8_t pins[] = { 0, 1, 2 }; // 对应引脚

    // 遍历数组，调用所有设备函数（车载批量控制外设）
    for (uint8_t i = 0; i < 3; i++) {
        func_p[i](pins[i]); // 等价于：(*func_p[i])(pins[i])
    }

    return 0;
}
```

# Day3：函数指针 + typedef（车载C语言核心）
## 1. 核心知识点
- 函数指针：存函数入口地址，语法`返回值 (*指针名)(参数)`，括号是关键；
- typedef：给函数指针起别名（如`VehicleFunc`），简化语法，车载代码必用；
- 函数指针数组：车载批量管理外设（车灯/蜂鸣器），新增设备只需扩展数组，解耦代码。

## 2. 踩坑记录
- 函数指针括号不能少：`void *func()` 是普通函数，不是指针；
- 参数/返回值必须匹配：函数指针类型要和指向的函数完全一致；
- 数组遍历注意下标：车载外设引脚要和函数一一对应，避免下标越界。

## 3. 车载关联
- 函数指针：STM32中断回调函数注册、CAN报文处理函数匹配；
- 函数指针数组：车载MCU批量管理外设（车灯/蜂鸣器/传感器），符合开闭原则。
