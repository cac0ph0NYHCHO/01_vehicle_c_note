### 位运算基础用法一览
```c
#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>

// 模拟STM32的GPIO输出寄存器（32位，仅用低16位控制引脚）
// 二进制：0000 0000 0000 0100 → 十进制4 → 表示PB2灯亮
unsigned int GPIO_ODR = 0;

// 车载标准化宏定义（复用Day7知识点）
#define BRAKE_LED_PIN 2  // 刹车灯-PB2
#define REVERSE_LED_PIN 3 // 倒车灯-PB3

// 1. 打开车灯（置1）
void led_on(int pin) {
    GPIO_ODR |= (1 << pin); // 把pin位设为1，其他位不变
    printf("打开PB%d车灯 → ODR寄存器值：0x%08X\n", pin, GPIO_ODR);
}

// 2. 关闭车灯（置0）
void led_off(int pin) {
    GPIO_ODR &= ~(1 << pin); // 把pin位设为0，其他位不变
    printf("关闭PB%d车灯 → ODR寄存器值：0x%08X\n", pin, GPIO_ODR);
}

// 3. 翻转车灯（闪烁）
void led_toggle(int pin) {
    GPIO_ODR ^= (1 << pin); // 翻转pin位（0变1/1变0）
    printf("翻转PB%d车灯 → ODR寄存器值：0x%08X\n", pin, GPIO_ODR);
}

// 4. 读取车灯状态
int led_get_status(int pin) {
    return (GPIO_ODR >> pin) & 1; // 提取pin位的值（0=灭/1=亮）
}

int main() {
    // 车载车灯控制流程模拟
    printf("===== 车载车灯控制系统 =====\n");

    // 控制刹车灯（PB2）
    led_on(BRAKE_LED_PIN);
    printf("PB%d状态：%d（亮）\n\n", BRAKE_LED_PIN, led_get_status(BRAKE_LED_PIN));

    led_toggle(BRAKE_LED_PIN);
    printf("PB%d状态：%d（灭）\n\n", BRAKE_LED_PIN, led_get_status(BRAKE_LED_PIN));

    // 控制倒车灯（PB3）
    led_on(REVERSE_LED_PIN);
    printf("PB%d状态：%d（亮）\n", REVERSE_LED_PIN, led_get_status(REVERSE_LED_PIN));

    return 0;
}
```
- 核心代码解读
- `1 << pin`：把数字 1 左移pin位，比如1<<2=4（二进制100），精准定位到第 2 位（PB2）；
- `~(1 << pin)`：取反后，只有pin位是 0，其他位都是 1（比如~4=0xFFFFFFFB）；
- `GPIO_ODR &= ~(1<<2)`：按位与后，第 2 位被置 0（灯灭），其他位保持不变（不影响其他灯）；
- `(GPIO_ODR >> 2) & 1`：先右移 2 位把第 2 位移到最低位，再和 1 按位与，只保留这一位的值（0/1）。
