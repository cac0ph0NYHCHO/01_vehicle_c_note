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
	printf("ÊäÈëµçÑ¹Öµ£¨ÕûÊý£©£º");
	scanf("%d", &a);
	if (a<10){
		printf("µÍµçÑ¹¾¯¸æ\n");
	} 
	else if(10<=a && a<=11){
		printf("µçÑ¹ÂÔµÍ\n");
	}
	else if(a>=12){
		printf("µçÑ¹Õý³£\n");
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
	
	printf("ÇëÊäÈëÊ£ÓàÓÍÁ¿£º") ;
	scanf("%d", &a);
	printf("ÇëÊäÈë°Ù¹«ÀïÓÍºÄ£º") ;
	scanf("%f", &b);
	
	if(b<=0){
		printf("ÊäÈë´íÎó£¬ÎÞ·¨¼ÆËã£¡\n");
		return 0;
	}
	
	printf("Ê£ÓàÐøº½£º%.1f\n", (a*1.0)/b*100);
	
	return 0;
}
```

### 位操作
```c
#include <stdio.h>

int main(){
	unsigned char sensor = 0x00;
	//µÚ 0 Î»£º³µËÙ´«¸ÐÆ÷ ¡ú ÉèÎªÕý³££¨ÖÃ 1£©
	sensor |= 1;
	//µÚ 1 Î»£ºÓÍÁ¿´«¸ÐÆ÷ ¡ú ÉèÎªÕý³££¨ÖÃ 1£©
	sensor |= (1<<1);
	//µÚ 2 Î»£ºË®ÎÂ´«¸ÐÆ÷ ¡ú ÉèÎª¹ÊÕÏ£¨Çå 0£©
	sensor &= ~(1<<2);
	
	printf("´«¸ÐÆ÷×´Ì¬£º0x%02x\n", sensor);

	return 0;
}
```
