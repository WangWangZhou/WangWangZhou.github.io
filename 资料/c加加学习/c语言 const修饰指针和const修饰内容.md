c语言 const修饰指针和const修饰内容

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

int main601() {

	//1、这种方式不安全 可以通过指针修改
	//通过指针const修饰的常量
	const int a = 10;
	int *p = &a;
	*p = 100;
	printf("%d\n",a);
	printf("%d\n",*p);

	return 0;
}

int main602() {

	int a = 10;
	int b = 20;
	//2、如果const 修饰 int * 不能改变指针变量指向的内存地址的值
	// 但是可以改变指针指向的地址
	const int * p;
	p = &a;
//	*p = 100;
	p = &a;
	p = &b;
	printf("%d\n",*p);

	return 0;
}

int main603() {

	int a = 10;
	int b = 20;
	//3、const修饰指针变量 能改变指针变量指向地址的值
	// 但是不能改变指针指向的地址
	int * const p = &a;
	*p = 100;
	//p = &b;

	printf("%d\n",*p);
	return 0;
}

int main604() {

	int a = 10;
	int b = 20;
	//4、const修饰指针类型  也修饰指针变量 那么不能改变指针指向的地址 也不能修改指针指向的值
	const int * const p = &a;
	//p = &b;
	//*p = 100;

	return 0;
}

```

![](https://xinqianpingtaib2btest.oss-cn-shenzhen.aliyuncs.com/xinqianpingtaib2btest/blogimg/2020/微信截图_20200430002705.jpg)

被红色方块圈起来的表示不能这样操作。