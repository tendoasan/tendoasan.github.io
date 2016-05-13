---
layout: post
title: 指针小结
category: 笔记
tags: C语言
keywords: C语言,指针
description: 
---

## 间接
指针提供了一个间接的方法来存取一个特定数据项中的数值。

---

## 定义指针变量
定义如下的一个变量:

	int count = 10;

定义另一个变量，可以使用它以间接的方式来存取count的值:

	int *int_pointer;

在C语言中，以上语句中的*代表`int_pointer`是一个整型指针变量。这样，程序就可以用`int_pointer`来间接地存取一个或多个整型数值。

`&`操作符为地址运算符, 在C语言中用于生成指向某个目标的指针。

若`x`是一个特定类型的变量，则表达式`&x`就是这个变量的指针。

表达式`&x`可以被赋予任何一个指针变量，只要它被声明为和`x`的类型相同的指针。

用如下语句建立int_pointer和count之间的间接引用关系。

	int_pointer = &count;

上面的表达式将一个指向变量count的指针，而不是变量count的值赋予变量int_pointer。

为了获取指针变量int_pointer指向的变量count所包含的内容，可以使用*操作符，即指针运算符。如果定义x为整型变量，则以下语句:
	x = *int_pointer;
将int_pointer通过间接关系所指向的值赋予变量x。因为int_pointer先前被设为指向count，所以这个语句将包含在变量count的值10，赋予变量x。

---

## 在表达式中运用指针

	int i1 = 5, i2, *p1, *p2;
	p1 = &i1;
	i2 = *p1 / 2 + 10;
	p2 = p1;

结果为：

	i1 = 5, i2 = 12, *p1 = 5, *p2 = 5；

如果一个指针px指向一个变量x，且px被定义为和x相同类型的指针，那么表达式中*px的用法等同于x。

---

## 使用指针和结构
定义一个如下的数据结构：

	struct date{
		int month;
		int day;
		int year;
	};

定义一个结构变量：

	Struct date todaysDate;

定义一个指向结构的指针：

	Struct date *datePtr;

使该指针指向todaysDate:

	datePtr = &todaysDate;

存取结构中的一个成员：

	(*datePtr).day = 21;

结构指针运算符：->，即(*x).y可以表示为x->y.

### 包含指针的结构
定义一个结构intPtrs，包含三个整型指针。

	struct intPtrs{
		int *p1;
		int *p2;
		int *p3;
	};

定义一个结构类型的变量intPtrs:

	struct intPtrs pointers;

将该变量内的p2指针指向的整型数的值设为-97:

	*pinters.p2 = -97;

### 链表
定义一个如下结构:

	struct entry{
		int value;
		struct entry *next;
	};

创建一个链表:

	int main(void){
		struct entry n1, n2, n3;
		struct entry *list_pointer = &n1; // 显示链表的头指针
	
		n1.value = 100;
		n1.next = &n2;
	
		n2.value = 200;
		n2.next = &n3;
	
		n3.value = 300;
		n3.next = (struct entry *) 0; // 用空指针来标识链表的表尾
	
		while(list_pointer != (struct entry *) 0){
			printf("%i\n", list_pointer->value);
			list_pointer = list_pointer->next;;
		}
	
		return 0;
	}

---

## 关键字const和指针
假定有如下声明:

	char c = 'x'
	char *charPtr = &c;

指针变量charPtr被设为指向变量c，如果它总是指向c，则可被声明为一个指针常量如下:

	char * const charPtr = &c;

(读法为“charPtr是一个指向字符的指针常量”)因此，类似的下面的一个语句:

	charPtr = &d; // 不是有效的，即该指针不会指向其他变量

如果charPtr指向的位置的值不会通过使用指针变量charPtr被改变，则可使用如下声明:

	const char *charPtr = &c;
(读法为“charPtr指向一个字符常量”)因此，类似下面的语句:

	*charPtr = 'Y'; // 不是有效的，即该指针指向的变量值不能通过指针来改变

在指针变量和它所指向的单元都不改变的情况下，可应用下面的声明:

	const char * const *charPtr = &c;

第一个const表明指针指向的单元的内容不会被改变，第二个const表明指针本身不会被改变。

---

## 指针和函数
1.将指针作为参数传递给函数:

	void print_list(struct entry *pointer){
		...
	}

接下来接可以将形式参数pointer当作一个正常的指针变量使用了。
在调用函数时，指针的值将被复制到形式参数中。
尽管函数不能改变指针参数，但却可以改变指针指向的数据元素:

	void exchange(int * const pint1, int * const pint2){
		int temp;
	
		temp = *pint1;
		*pint1 = *pint2;
		*pint2 = temp;
	}

	int main(void){
		void exchange(int * const pint1, int * const pint2);
		int i1 = -5, i2 = 66, *p1 = &i1, *p2 = &i2;
	
		printf("i1 = %i, i2 = %i\n", i1, i2); // -5, 66
	
		exchange(p1, p2);
		printf("i1 = %i, i2 = %i\n", i1, i2); // 66, -5
	
		exchange(&i1, &i2);
		printf("i1 = %i, i2 = %i\n", i1, i2); // -5, 66
	
		return 0;
	}

函数exchange的功能时交换由它的两个指针参数(const)所指向的整型变量的值。
若是不用指针，我们就无法写出exchange函数来交换两个整型的值，因为函数被限制为只能返回一个单值，另外函数不能改变它的参数。

2.函数返回一个指针
定义一个函数，查找一个链表以找出给定的值，返回值是一个指向结构entry的指针:

	struct entry *findEntry(struct entry *listPtr, int match){
		while(listPtr != (struct entry *) 0){
			if(listPtr->value == match)
				return (listPtr);
			else
				listPtr = listPtr->next;
		}
		return (struct entry *) 0;
	}

	int main(void){
		struct entry *findEntry(struct entry *listPtr, int match);
		struct entry n1, n2, n3;
		struct entry *listPtr, *listStart = &n1;
		int search;
	
		n1.value = 100;
		n1.next = &n2;
	
		n2.value = 200;
		n2.next = &n3;
	
		n3.value = 300;
		n3.next = 0;
	
		printf("Enter value to locate: ");
		scanf("%i", &search);
	
		listPtr = findEntry(listStart, search);
	
		if(listPtr != (struct entry *) 0)
			printf("Found %i.\n", listPtr->value);
		else
			printf("Not found.\n");
		
		return 0;
	}

可以将字典组织成链表，方便插入和删除，缺点是不方便查找，比如无法使用二分查找。
树可以对元素进行方便的插入和删除，其查找速度也很快。

---

## 指针和数组
如果a是一个元素为x类型的数组，px是x的指针类型，而i和n时整型常量或变量，则语句:

	px = a; // px = &a[0];

用来将指针px指向a的第一个元素，表达式:

	*(px + i)

用来引用包含在a[i]中的值。进而，语句:

	px += n;

用来将px指向数组中的第n号元素，不管数组中的元素到底是那种类型。

### 关于程序的优化
通常情况下，(按照顺序)使用下标变量访问数组元素的过程会比使用指针访问花费更多的时间。
使用指针来存取数组元素的主要原因就是这样产生的代码通常效率更高。

### 数组还是指针
如果我们想用下标来引用传递给一个函数的数组元素，需要将相应的形式参数声明为数组。这样就更准确地表明了函数中使用的是数组。如下所示:

	int arraySum(int array[], const int n); 

相似的，如果我们将形式参数作为一个指向数组的指针使用，则要将它声明为一个指针类型:

	int arraySum(int *array, const int n); //声明符号array为一个整型指针

### 指向字符串的指针
用指针来编写copyString:

	void copyString(char *to, char *from){
		for(; *from != '\0'; ++from, ++to)
			*to = *from;
		
		*to = '\0';
	}

	int main(void){
		void copyString(char *to, char *from);
		char string1[] = "A string to be copied.";
		char string2[50];
	
		copyString(string2, string1);
		printf("%s\n", string2);
	
		copyString(string2, "So is this.");
		printf("%s\n", string2);
	
		return 0;
	}

### 字符串常量和指针
如果text是一个字符数组，用下面语句初始化text:

	char text[80] = "This is okay.";

C语言将这些字符串的字符本身保存在数组array的对应位置。

如果text是一个字符指针，用下面语句初始化text:

	char *text = "This is okay.";

则将一个指向字符串"This is okey."的指针赋值到text。

用如下方法建立一个叫做days的数组，它包含指向一个星期中每一天的名字的字符指针:

	char *days[] = {"Sunday","Monday","Tuesday","Wednesday","Thursday","Friday","Saturday"};

数组days被定义为包含7个元素，每个元素都是一个指向字符串的指针。于是，
days[0]包含指向字符串"Sunday"的指针，days[1]包含指向字符串"Monday"的指针，等等。
可以用下面的语句显示一个星期第三天的名字:

	printf("%s\n", days[3]);

### 递增和递减运算符
`++i`，++放置在操作数的前面，递增运算符可以看成是“前加”；
`i++`，++放置在操作数的后面，递增运算符可以看成是“后加”；
前置和后置运算符的差别：
1.
假定有两个整型变量i和j，如果我们将i设为0，则下面语句：

	j = ++i;

将1赋值给j，而不是0。在前置运算符的情况下，变量的值首先被加1，然后再参加到表达式的求值中。因此前面的表达式中，变量i的值首先由0递增为1，然后其值被赋给j，等价于如下语句：
	++i;
	j = i;

如果我们在语句中用了“后加”运算符：

	j = i++;

则i的值实在先赋值给j之后才递增的。因此前面的表达式中，先将0赋值给j，然后对i进行递增操作，即加1，等价于如下语句：

	j = i;
	++i;

2.
另举一个例子。如果i为1，则下面语句：

	x = a[--i];

将a[0]的值赋给x。因为，变量i在其值作为下标使用之前，已经进行了递减操作。而语句：

	x = a[i--];

将a[1]的值赋给x。因为，变量i在其值作为下标使用过之后，才进行递减操作。

3.
再举一个例子说明前置和后置递增和递减运算符的差别。函数调用：

	printf("%i\n", ++i); // 如果i为100，则printf调用显示101，语句执行完毕i为101

在i递增后才将其值传递给printf函数，然而，函数调用：

	printf("%i\n", i++); // 如果i为100，则printf调用显示100，语句执行完毕i为101

在将i的值传递给printf函数之后，才对i进行递增操作。

4.
最后一个例子：如果textPtr时一个字符指针，则表达式：

	*(++textPtr)
首先递增textPtr，然后取出它所指向的字符。然而，表达式：

	*(textPtr++)

在它的值递增之前就取回了textPtr所指向的字符。不管是哪种情况，圆括号都不是必需的，因为*和++运算符具有相同的优先级，而且它们是“自右向左”结合的。

在赋值语句里使用后置递增运算符来重写copyString函数：

	// copyString函数的修订版本
	#include <stdio.h>
	void copyString(char *to, char *from){
		// 空字符的值等价于值0
		while(*from)
			*to++ = *from++;
		
		*to = '\0';
	}

	int main(void){
		void copyString(char *to, char *from);
		char string1[] = "A string to be copied.";
		char string2[50];
	
		copyString(string2, string1);
		printf("%s\n", string2);
	
		copyString(string2, "So is this.");
		printf("%s\n", string2);
	
		return 0;
	}

---

## 指针运算
C语言中两个指针减法操作的结果是它们之间所包含元素的个数。
由两个指针的减操作产生的结果的实际类型时ptrdiff_t，它在标准头函数文件<stddef.h>里定义。
因此，如果a是一个指向任意元素类型的数组，而b时另一个同类型的指针，该指针指向同一数组更远的一个位置，则表达式b-a代表这两个指针之间的元素个数。
例如，如果p时一个指向数组x的某个元素的指针，则语句：

	n = p - x;

将p所指的元素的下标数赋值给变量n(假定n是一个整型变量)。因此，如果p由以下语句设置为指向第100个元素：

	p = &x[99];

那么，变量n在前面的减操作完成后值为99.
指针相加同理？

---

## 指向函数的指针
为了使用指向函数的指针，C语言编译器不仅需要知道指向函数的指针变量，还要知道函数返回值的类型，还包括它的参数的数量和类型。为了声明符号fnPtr为一个“指向一个返回整数且不带任何参数的函数”的指针，我们使用如下声明：

	int (*fnPtr)(void);

*fnPtr两边的小括号时必需的，否则，因为函数调用运算符()比指针运算符*的优先级高，C语言编译器将会把前面的语句作为一个函数的声明，这个函数叫做fnPtr，它返回一个指向整型数的指针。
为了将函数指针指向一个指定的函数，我们可以简单地将函数的名字赋值给它。因此如果lookup是一个函数，它返回一个整型，且没有参数，则语句：
	
	fnPtr = lookup;

将一个指向函数的指针存储到函数指针变量fnPtr之中。
如果需要通过函数指针变量间接调用函数，我们可以在函数指针变量后面加上函数调用操作符--即一对小括号，然后在小括号里列出要传递给函数的参数。例如：

	entry = fnPtr();

调用了由fnPtr指向的函数，并存储返回值到变量entry之中。

函数指针的一个最常用的用途时将函数作为参数传递给其他函数。例如标准C语言库函数qsort就用到这个特性。
函数指针的另一个常见应用时用来产生一个派遣表。我们不能将函数存储到一个数组元素中，但是我们可以将函数指针存储到数组中。通过函数指针，我们可以生成一个包含函数指针的表。

---

## 指针和内存地址
在C语言里，当我们对一个变量运用地址运算符时，得到的结果就是这个变量在计算机内存里的实际地址。所以，下面的语句：

	intPtr = &count;

将计算机分配给变量count的内存地址赋值给intPtr。因此，如果count存储在地址0x8868，包含了一个值为10，则这个语句将值0x8868赋予intPtr.
用对一个指针变量运用指针运算符，如下表达式所示：

	*intPtr

则告诉计算机，将包含在指针变量的值看作是一个内存地址。于是计算机取出内存地址里存储的值，并根据指针变量的类型对该值作出解释。
