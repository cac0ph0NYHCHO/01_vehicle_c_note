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

### Test1
```c
练习题 1：基础 - 宏 + 枚举 + 位运算（10 分钟）
题目要求：
用#define封装：刹车灯引脚（PB5）、倒车灯引脚（PB6）；
用enum定义车灯状态：灭（0）、亮（1）、闪烁（2）；
用位运算实现：
打开刹车灯（置 1）；
关闭倒车灯（置 0）；
翻转刹车灯状态；
读取倒车灯状态并打印。
答题要求：
写完整可运行代码，模拟 STM32 寄存器（用unsigned int GPIO_ODR）。
```
```c
#include<stdio.h>

#define BRAKE_LED_PIN 5
#define REVERSE_LED_PIN 6

enum LedStatus {
	LED_OFF = 0,
	LED_ON = 1,
	LED_BLINK = 2
};

int main() {
	unsigned int GPIO_ODR = 0;

	GPIO_ODR |= (1 << BRAKE_LED_PIN);
	GPIO_ODR &= ~(1 << REVERSE_LED_PIN);
	GPIO_ODR ^= (1 << BRAKE_LED_PIN);

	printf("%d\n", (GPIO_ODR >> REVERSE_LED_PIN) & 1);

	printf("刹车灯状态：%s\n",
		((GPIO_ODR >> REVERSE_LED_PIN) & 1 )== LED_OFF ? "灭" : "亮");


	return 0;
}
```

### Test2
```c
练习题 2：进阶 - 结构体 + 动态内存（15 分钟）
题目要求：
定义CarLed结构体，包含：引脚（pin）、名称（name [20]）、状态（LedStatus 枚举）；
动态申请内存创建包含 3 个车灯的数组（刹车灯、倒车灯、示宽灯）；
初始化数组：
刹车灯：pin=5，name="刹车灯"，status = 亮；
倒车灯：pin=6，name="倒车灯"，status = 灭；
示宽灯：pin=7，name="示宽灯"，status = 闪烁；
遍历数组打印所有车灯信息；
释放内存并置 NULL。
答题要求：
必须做 malloc 判空，避免野指针。
```
```c
#define _CRT_SECURE_NO_WARNINGS
#include<stdio.h>
#include<stdint.h>
#include<stdlib.h>
#include<string.h>

enum LedStatus {
	LED_OFF = 0,
	LED_ON = 1,
	LED_BLINK = 2,
};

typedef struct {
	uint8_t pin;
	char name[20];
	enum LedStatus status;
} CarLed;

int main() {
	CarLed *leds = malloc(3 * (sizeof(CarLed)));
	if (leds == NULL) {
		perror("内存申请失败：");
		return -1;
	}
	
	leds[0].pin = 5;
	strcpy(leds[0].name, "刹车灯");
	leds[0].status = LED_ON;

	leds[1].pin = 6;
	strcpy(leds[1].name, "倒车灯");
	leds[1].status = LED_OFF;

	leds[2].pin = 7;
	strcpy(leds[2].name, "示宽灯");
	leds[2].status = LED_BLINK;

	printf("所有车灯信息如下：\n");
	for (int i = 0; i < 3; i++) {
		printf("%s：pin=%d,status=%d\n",
			leds[i].name, leds[i].pin, leds[i].status);
	}

	free(leds);
	leds = NULL;

	return 0;
}
```
- malloc返回的是`void*`类型的指针
- 用malloc要包含头文件#include<stdlib.h>
- malloc申请内存后要判空，最后要free+置NULL

### Test3
```c
练习题 3：综合 - 文件操作 + 结构体（20 分钟）
题目要求：
基于练习题 2 的结构体，实现：
把 3 个车灯的信息写入car_led_config.txt（开头加 2 行注释：# 车载车灯配置、# 格式：名称 引脚 状态）；
读取该文件（跳过注释行），把数据读入新的结构体数组；
打印读取后的车灯信息，验证和写入的一致。
答题要求：
文件操作全程判空，fclose 后置 NULL，fscanf 校验返回值（可选）。
```
```c
#define _CRT_SECURE_NO_WARNINGS
#include<stdio.h>
#include<stdint.h>
#include<stdlib.h>
#include<string.h>

enum LedStatus {
	LED_OFF = 0,
	LED_ON = 1,
	LED_BLINK = 2,
};

typedef struct {
	uint8_t pin;
	char name[20];
	enum LedStatus status;
} CarLed;

int main() {
	CarLed* leds = malloc(3 * (sizeof(CarLed)));
	if (leds == NULL) {
		perror("内存申请失败：");
		return -1;
	}
	CarLed* leds_data = malloc(3 * (sizeof(CarLed)));
	if (leds_data == NULL) {
		perror("内存申请失败：");
		free(leds);
		return -1;
	}

	leds[0].pin = 5;
	strcpy(leds[0].name, "刹车灯");
	leds[0].status = LED_ON;
	leds[1].pin = 6;
	strcpy(leds[1].name, "倒车灯");
	leds[1].status = LED_OFF;
	leds[2].pin = 7;
	strcpy(leds[2].name, "示宽灯");
	leds[2].status = LED_BLINK;

	FILE* fp_write = fopen("car_led_config.txt", "w");
	if (fp_write == NULL) {
		perror("文件指针申请失败：");
		free(leds);
		free(leds_data);
		leds = NULL;
		leds_data = NULL;
		return -1;
	}

	fprintf(fp_write, "#车载车灯配置\n");
	fprintf(fp_write, "#格式：名称 引脚 状态\n");
	for (int i = 0; i < 3; i++) {
		fprintf(fp_write, "%s %d %d\n",
			leds[i].name, leds[i].pin, leds[i].status);
	}
	fclose(fp_write);
	fp_write = NULL;

	FILE* fp_read = fopen("car_led_config.txt", "r");
	if (fp_read == NULL) {
		perror("文件指针申请失败：");
		free(leds);
		free(leds_data);
		leds = NULL;
		leds_data = NULL;
		return -1;
	}

	char temp[50];
	fgets(temp, sizeof(temp), fp_read);
	fgets(temp, sizeof(temp), fp_read);
	for (int i = 0; i < 3; i++) {
		fscanf(fp_read, "%s %hhu %d",
			leds_data[i].name, &leds_data[i].pin, &leds_data[i].status);
		printf("%s：pin=%d,status=%d\n",
			leds_data[i].name, leds_data[i].pin, leds_data[i].status);
	}
	fclose(fp_read);
	fp_read = NULL;

	free(leds);
	leds = NULL;
	free(leds_data);
	leds_data = NULL;
	
	return 0;
}
```
- leds_data[i].pin/status是数值类型，fscanf读取时必须传地址（&）
- 用"w"模式打开文件后，直接尝试读取："w"模式是只写，会清空文件且无法读取，读取必须重新以"r"模式打开
- %hhu 是 C 语言中专门匹配 unsigned char/uint8_t（8 位无符号字符型）的格式控制符
- 写入用"%s %d %d\n"，读取用"%s %hhu %d"（去掉\n，fscanf会自动忽略空白符）

### Test4
```c
练习题 4：高阶 - 函数指针 + 位运算（20 分钟）
题目要求：
在CarLed结构体中新增函数指针：void (*control)(CarLed *led, enum LedStatus status)；
实现控制函数led_control：根据传入的状态，用位运算修改模拟寄存器GPIO_ODR，并更新结构体的 status；
初始化结构体数组时，绑定函数指针；
调用每个车灯的 control 函数：
刹车灯设为闪烁；
倒车灯设为亮；
示宽灯设为灭；
打印控制后的寄存器值和车灯状态。
```
```c
#define _CRT_SECURE_NO_WARNINGS
#include<stdio.h>
#include<stdint.h>
#include<stdlib.h>
#include<string.h>

enum LedStatus {
	LED_OFF = 0,
	LED_ON = 1,
	LED_BLINK = 2,
};

typedef struct CarLed CarLed;
typedef void (*LedControlFunc)(CarLed* led, enum LedStatus status);

struct CarLed {
	uint8_t pin;
	char name[20];
	enum LedStatus status;
	LedControlFunc control;
};

unsigned int GPIO_ODR = 0;

void led_control(CarLed* led, enum LedStatus status) {
	if (led == NULL) return;
	
	led->status = status;
	switch (status) {
		case LED_OFF:
			GPIO_ODR &= ~(1 << led->pin);
			break;
		case LED_ON:
			GPIO_ODR |= (1 << led->pin);
			break;
		case LED_BLINK:
			GPIO_ODR ^= (1 << led->pin);
			break;
	}
	printf("%s已执行：%s -> 寄存器值=0x%08X\n",
		led->name,
		status == LED_ON ? "亮" : (status == LED_OFF ? "灭" : "闪烁"),
		GPIO_ODR);
}

int main() {
	CarLed* leds = malloc(3 * (sizeof(CarLed)));
	if (leds == NULL) {
		perror("内存申请失败：");
		return -1;
	}

	leds[0].pin = 5; strcpy(leds[0].name, "刹车灯"); leds[0].status = LED_ON;
	leds[1].pin = 6; strcpy(leds[1].name, "倒车灯"); leds[1].status = LED_OFF;
	leds[2].pin = 7; strcpy(leds[2].name, "示宽灯"); leds[2].status = LED_BLINK;

	leds[0].control = led_control;
	leds[1].control = led_control;
	leds[2].control = led_control;

	printf("开始控制车灯\n");
	leds[0].control(&leds[0], LED_BLINK);
	leds[1].control(&leds[1], LED_ON);
	leds[2].control(&leds[2], LED_OFF);

	free(leds);
	leds = NULL;

	return 0;
}
```
- 解释`typedef struct CarLed CarLed;`看下面`struct`的基本用法

### struct基本用法
#### 基本语法
- 先定义后使用
```c
// 示例：定义"学生"结构体
struct Student {
    int id;         // 学号（整型）
    char name[20];  // 姓名（字符数组）
    float score;    // 成绩（浮点型）
};

 // 2. 定义结构体变量（使用模板创建具体的"数据实例"）
    struct Student stu1;  // 方式1：先定义类型，再定义变量
    struct Student stu2 = {2, "李四", 92.5};  // 方式2：定义变量时直接初始化
```
#### typedef 简化结构体类型名（避免重复写 struct）
```c
// 定义时用typedef，后续可直接用"Student"代替"struct Student"
typedef struct {
    int id;
    char name[20];
    float score;
} Student;

Student stu;  // 无需写struct，更简洁
```
#### 结构体数组（批量管理多个结构体对象）
```c
typedef struct {
    int id;
    char name[20];
    float score;
} Student;

// 定义并初始化结构体数组
    Student stus[3] = {
        {1, "张三", 88.0},
        {2, "李四", 92.5},
        {3, "王五", 85.5}
    };
```
#### 结构体指针（通过指针访问成员，用 "->" 代替 "."）
```c
typedef struct {
    int id;
    char name[20];
    float score;
} Student;

Student stu = {1, "张三", 88.0};
Student *p = &stu;  // 定义结构体指针，指向stu
```
