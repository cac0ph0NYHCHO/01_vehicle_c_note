# Day6 文件操作（fopen/fprintf/fscanf）- 车载数据持久化核心
## 1. 核心知识点
- 文件操作流程：`fopen`（打开）→ `fprintf/fscanf`（读写）→ `fclose`（关闭）→ `fp=NULL`（防野指针），全程需判空/校验；
- 核心打开模式："w"（写配置，清空原有内容）、"r"（读配置，只读）、"a"（追加日志，末尾添加）；
- 读写规则：`fprintf`/`fscanf`格式需与文件内容严丝合缝，一行一个数据最稳；数值变量读需加`&`，字符串数组读不加`&`。

## 2. 踩坑记录
- 格式不匹配：`fscanf`读含中文/字母的混合内容（如PB2）未跳过无关字符，导致读取乱数；
- 野指针问题：仅`fclose`未置`fp=NULL`，后续误操作易引发MCU异常；
- 校验缺失：未校验`fscanf`返回值，文件内容不全时输出乱数；未校验`fopen`结果，文件不存在直接操作崩溃；
- 路径错误：相对路径找不到文件，车载场景需用绝对路径。

## 3. 车载关联
- 核心应用：车灯配置保存/读取、行车日志追加、传感器参数持久化（程序重启数据不丢失）；
- 工程化规范：读写逻辑封装成函数，配置文件加注释行，读取后校验数据合法性（如引脚范围）；
- 安全要求：`fopen`判空、`fscanf`返回值校验、`fclose+置NULL`是车载文件操作铁律，防止MCU死机/数据丢失。

### 打开文件（写模式）
```c
#define _CRT_SECURE_NO_WARNINGS
#include<stdio.h>
#include<stdint.h>
#include<string.h>

typedef struct {
	uint8_t pin;
	char name[20];
	uint8_t default_status; // 0=灭，1=亮
} CarLed;

int main() {
	// 1. 定义车灯对象
	CarLed brake_led = { 2, "刹车灯", 1 }; // 默认亮

	// 2. 打开文件（写模式，车载配置文件常用）
	// FILE*是文件指针，和结构体指针用法类似
	FILE* fp = fopen("car_led_config.txt", "w");
	// 必做：判空（文件打开失败，比如权限不足）
	if (fp == NULL) {
		printf("车载场景：配置文件打开失败！\n");
		return -1;
	}

	// 3. 写数据到文件（格式化，和printf用法几乎一样）
	// fprintf(文件指针, 格式字符串, 变量);
	fprintf(fp, "车载车灯配置文件\n");
	fprintf(fp, "车灯名称：%s\n", brake_led.name);
	fprintf(fp, "引脚编号：PB%d\n", brake_led.pin);
	fprintf(fp, "默认状态：%d（1=亮，0=灭）\n", brake_led.default_status);

	// 4. 关闭文件（车载代码必做，防止数据丢失）
	fclose(fp);
	fp = NULL; // 置NULL，避免野指针

	printf("车载场景：车灯配置已保存到文件！\n");
	return 0;
}
```

### scanf基础用法
```c
#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>

int main() {
    char s[10];
    int num;

    FILE* fp = fopen("test.txt", "r");

    if (fp == NULL) {
        perror("文件打开失败：");
        fp = NULL;
        return -1;
    }

    fscanf(fp, "%s %d", s, &num);
    printf("第一组：s=%s,num=%d\n", s, num);

    fscanf(fp, "%s %d", s, &num);
    printf("第二组：s=%s,num=%d\n", s, num);


    fclose(fp);
    fp = NULL;

    return 0;
}
```
- `perror("车载场景：配置文件打开失败");` // 打印具体错误原因
- fopen 打开文件后，必须调用 fclose 关闭文件
