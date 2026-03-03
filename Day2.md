### 数组和指针
```c
#include<stdio.h>

int main() {
	int arr[5] = {10, 20, 30, 40, 50};
	int *p = arr;
	int i,j;
	
// 方式1：下标访问	
	printf("下标访问：");
	for(i = 0;i<5;i++){
		printf("%d ", arr[i]);
	} 
	printf("\n");
	
// 方式2：指针访问	
	printf("指针访问：");
	for(j = 0;j<5;j++){
		printf("%d ", *(p+j));
	}
	printf("\n");

// 验证：数组名是首元素地址	
	printf("arr = %p\n", arr);
	printf("&arr[0] = %p\n", &arr[0]);
	printf("p = %p\n", p);
	
	return 0;
}
```
- %p传入地址

### 数组型字符串和指针型字符串
```c
#include<stdio.h>

int main() {
// 两种字符串定义方式
	char str1[] = "abs"; // 数组型字符串
	char *str2 = "abs";	 // 指针型字符串		
	
// 1. 输出字符串	
	printf("str1 = %s\n", str1);
	printf("str2 = %s\n", str2);
	
// 2. 指针访问单个字符
	printf("str1[2] = %c\n", str1[2]);
	printf("str1[3] = %c\n", str1[3]);
	printf("*(str2+2) = %c\n", *(str2+2));	
	printf("*(str2+3) = %c\n", *(str2+3));
	
// 3. 注意：指针型字符串不可修改（面试坑点）
	//str2[0] = 'n'; // 取消注释会崩溃！原因：字符串常量存只读区
	str1[0] = 'n';
	printf("修改后的str1 = %s\n", str1);
	
	return 0;
}
```
- %s 传的是字符串首地址，识别到'\0'停止输出
- %c 只传一个字符
- char *p 指向只读区，不可更改

### （题目1）找最大数最小数，熟悉指针的用法
```c
#include<stdio.h>

int main() {
	int scores[8] = {92, 78, 85, 98, 66, 80, 91, 88};
	int *p = scores;
	int max,min = *p;
	int i;
	
	for(i=0;i<8;i++){
		if(*(p+i)<min){
			min = *(p+i);
		}
		if(*(p+i)>max){
			max = *(p+i);
		}
	}
	
	printf("最大值 = %d\n", max);
	printf("最小值 = %d\n", min);
	
	return 0;
}
```

### （题目2）双指针逆序
```c
#include<stdio.h>

int main() {
	char msg[] = "CAN_FRAME_O1";
	char *left = msg;
	char *right = msg;
	char temp;
	
	while(*right != '\0'){
		right++;
	}
	right--;
	
	printf("反转前：%s\n", msg);
	
	while(left<right){
		temp = *right;
		*right = *left;
		*left = temp;
		
		left++;
		right--;
	}
	
	printf("反转后：%s\n", msg);
	
	return 0;
}
```
- 双指针逆序节省内存
- 重点：找到字符串的最后一个字符
```c
	while(*right != '\0'){
		right++;
	}
	right--;
```

### （题目3）通过函数修改字符串第一个字符
```c
#include<stdio.h>

void modify_arr_str(char a[]);
void modify_ptr_str(char *p);

int main() {
	char arr_str[] = "UART_MSG";
	char *ptr_str = "CAN_MSG";
	
	printf("修改前的字符数组：%s\n", arr_str);
	modify_arr_str(arr_str);
	printf("修改后的字符数组：%s\n", arr_str);
	
	printf("修改前的字符指针：%s\n", ptr_str);
	modify_ptr_str(ptr_str);
	printf("修改后的字符指针：%s\n", ptr_str);
	
	return 0;
}

void modify_arr_str(char a[])
{
	*a = 'c';
}

void modify_ptr_str(char *p)
{
	*p = 'c';
}
```
- 传数组参数时加了[]，只要写数组名就行了，传指针时也只要写指针名
