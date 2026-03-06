# Day5 结构体指针 + 动态内存分配（车载内存管理核心）
## 1. 核心知识点
- 结构体指针：访问成员用`->`，普通变量用`.`，指针需先绑定有效地址再使用；
- 动态内存（malloc/free）：堆区按需申请内存，核心四步「申请→判空→使用→释放置NULL」；
- 动态数组：`malloc(元素个数*sizeof(结构体))`创建，可通过`p_leds[i]`或`*(p_leds+i)`访问成员。

## 2. 踩坑记录
- 字符串赋值：结构体字符串成员需用`strcpy`，禁止直接赋值，且必须包含`string.h`头文件；
- 内存操作：malloc后未判空会导致MCU死机，free后未置NULL易产生野指针；
- 传参错误：strcpy第一个参数必须是字符串地址（如`p_led->name`），而非结构体指针。

## 3. 车载关联
- 结构体指针：STM32驱动传参首选方式，节省车载MCU有限内存，支持远程修改外设属性；
- 动态内存：车载场景按需加载外设（倒车雷达/空调），避免静态分配浪费内存；
- 内存规范：malloc判空+free置NULL是车载代码稳定性核心要求，防止内存泄漏导致MCU异常。

### 结构体指针核心语法
```c
#include <stdio.h>
#include <stdint.h>

// 车载车灯结构体（Day4的定义）
typedef struct {
    uint8_t pin;
    char name[20];
} CarLed;

// 结构体指针传参：控制车灯（车载驱动标准写法）
void led_ctrl(CarLed *p_led, uint8_t status) {
    if(status == 1) {
        printf("车载场景：%s（PB%d）亮\n", p_led->name, p_led->pin);
    } else {
        printf("车载场景：%s（PB%d）灭\n", p_led->name, p_led->pin);
    }
}

int main() {
    // 1. 定义普通结构体对象
    CarLed left_led = {0, "左车灯"};
    
    // 2. 定义结构体指针，绑定对象地址
    CarLed *p_led = &left_led;
    
    // 3. 传指针调用函数（只传4字节地址，高效）
    led_ctrl(p_led, 1); // 开灯
    led_ctrl(p_led, 0); // 关灯
    
    // 直接访问指针成员（车载代码高频操作）
    printf("指针访问：%s引脚=%d\n", p_led->name, p_led->pin);
    
    return 0;
}
```
- 函数指针 = 函数名(相当于函数首地址);
- 结构体指针 = &结构体名;
- 结构体指针访问：p_led->name, p_led->pin

### `. `和 `->`的区别
```c
#define _CRT_SECURE_NO_WARNINGS
#include<stdio.h>
#include<stdint.h>
#include<string.h>

typedef struct {
	uint8_t pin;
	char name[20];
} CarLed;

int main() {
	CarLed left_led = { 0, "左车灯" };
	CarLed* p_led = &left_led;

	p_led->pin = 1;
	strcpy(p_led->name, "右车灯");

	printf("普通变量：%s(PB%d)\n", left_led.name, left_led.pin);
	printf("指针访问：%s(PB%d)\n", p_led->name, p_led->pin);

	return 0;
}
```
- **#define _CRT_SECURE_NO_WARNINGS**(VS里用strcpy加在第一行预编译)

### 动态内存分配（malloc/free）
```c
#define _CRT_SECURE_NO_WARNINGS
#include<stdio.h>
#include<stdint.h>
#include<string.h>
#include<stdlib.h> // malloc/free 必须加这个头文件！

typedef struct {
	uint8_t pin;
	char name[20];
} CarLed;

int main() {
	// ===== 步骤1：向堆内存“租房”（动态分配）=====
	// 格式：(结构体指针类型)malloc(sizeof(结构体类型))
	CarLed* p_led = (CarLed*)malloc(sizeof(CarLed));

	// ===== 步骤2：必做！检查是否租房成功（判空）=====
	// 车载代码中：内存不足时malloc返回NULL，直接访问会死机
	if (p_led == NULL) {
		printf("内存不够了！创建车灯失败\n");
		return -1; // 退出程序，避免崩溃
	}

	// ===== 步骤3：使用动态创建的对象（和之前的指针用法完全一样）=====
	p_led->pin = 2; // 车灯引脚设为PB2
	strcpy(p_led->name, "刹车灯"); // 赋值字符串

	// 打印验证
	printf("动态车灯：%s(PB%d)\n", p_led->name, p_led->pin);

	// ===== 步骤4：必做！退房（释放内存）=====
	free(p_led); // 把内存还给系统
	p_led = NULL; // 关键：清空指针，防止“野指针”（车载代码铁律）

	return 0;
}
```
**TIPS**
```c
// 车灯开灯函数（传void*，适配任意外设）
void led_on(void *dev) {
    CarLed *p_led = (CarLed *)dev; // 强制类型转换（车载常用）
    printf("车载场景：%s（PB%d）亮\n", p_led->name, p_led->pin);
}
```
- 为什么要做这个转换？
- 最根本的原因是：dev 是无类型的通用指针，而我们需要访问 CarLed 结构体的具体成员（比如 name、status）—— 但 void* 无法直接解引用、无法访问结构体成员，必须转换成具体类型的指针才能操作。
- 字符串赋值：`strcpy(p_led->name, "刹车灯");`

### 动态创建 2 个车灯对象（刹车灯 + 倒车灯），用指针数组管理
```c
#define _CRT_SECURE_NO_WARNINGS
#include<stdio.h>
#include<stdint.h>
#include<string.h>
#include<stdlib.h>

typedef struct {
	uint8_t pin;
	char name[20];
} CarLed;

int main() {
	// 动态创建2个车灯的内存空间（数组）
	CarLed* p_leds = (CarLed*)malloc(2 * sizeof(CarLed));
	if (p_leds == NULL) {
		printf("内存不足！\n");
		return -1;
	}

	// 初始化第一个车灯（刹车灯）
	p_leds[0].pin = 2;
	strcpy(p_leds[0].name, "刹车灯");

	// 初始化第二个车灯（倒车灯）
	p_leds[1].pin = 3;
	strcpy(p_leds[1].name, "倒车灯");

	// 打印
	printf("车灯1：%s(PB%d)\n", p_leds[0].name, p_leds[0].pin);
	printf("车灯2：%s(PB%d)\n", p_leds[1].name, p_leds[1].pin);

	// 释放内存
	free(p_leds);
	p_leds = NULL;

	return 0;
}
```
**TIPS**
- malloc：动态申请内存，格式(类型*)malloc(sizeof(类型)*个数)，必须加stdlib.h
- 必做两步：malloc后判空 + free后置NULL
- 动态对象的使用：和普通结构体指针完全一样，用->访问成员
