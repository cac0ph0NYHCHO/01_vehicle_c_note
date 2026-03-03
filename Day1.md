### 初见指针

```c
#include<stdio.h>

int main() {
	int a = 10;
	int *p = &a;
	
	printf("a = %d\n", a);
	printf("&a = %p\n", &a);
	printf("p = %p\n", p);
	printf("*p = %d\n", *p);
	
	*p = 20;
	printf("a = %d\n", a);
	
	return 0;
}
```

### 基本的if判断
```c
#include <stdio.h>

int main()
{
	int a;
	printf("输入电压值（整数）：");
	scanf("%d", &a);
	if (a<10){
		printf("低电压警告\n");
	} 
	else if(10<=a && a<=11){
		printf("电压略低\n");
	}
	else if(a>=12){
		printf("电压正常\n");
	}
	
	return 0;
}
```

### 浮点数指定小数点输出
```c
#include <stdio.h>

int main(){
	int a;
	float b,c;
	
	printf("请输入剩余油量：") ;
	scanf("%d", &a);
	printf("请输入百公里油耗：") ;
	scanf("%f", &b);
	
	if(b<=0){
		printf("输入错误，无法计算！\n");
		return 0;
	}
	
	printf("剩余续航：%.1f\n", (a*1.0)/b*100);
	
	return 0;
}
```

### 位操作
```c
#include <stdio.h>

int main(){
	unsigned char sensor = 0x00;
	//第 0 位：车速传感器 → 设为正常（置 1）
	sensor |= 1;
	//第 1 位：油量传感器 → 设为正常（置 1）
	sensor |= (1<<1);
	//第 2 位：水温传感器 → 设为故障（清 0）
	sensor &= ~(1<<2);
	
	printf("传感器状态：0x%02x\n", sensor);

	return 0;
}
```
