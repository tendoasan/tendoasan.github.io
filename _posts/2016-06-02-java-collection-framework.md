---
layout: post
title: "Java基础--集合框架"
category: "笔记"
tags: "Java"
keywords: ["Java","Collection","Framework"]
description: "集合框架"
---
{% include JB/setup %}


#### 1.引言

数据结构(data structure)是以某种形式将数组组织在一起的集合，不仅存储数据，还支持那些访问和处理数据的操作。Java集合框架就是一组能有效地组织和操作数据的数据结构。
在面向对象思想里，一种数据结构也被认为是一个容器，它是一个能存储其他对象的对象，这里的其他对象是指数据或者元素。数据结构也可以称为容器对象。
定义一种数据结构在本质上讲就是定义一个类。数据结构类应该使用数据域存储数据，并提供方法支持查找、插入和删除等操作。
Java集合框架支持以下两种类型的容器：

+ 一种是为了存储一个元素集合，简称为集合(collection)
+ 另一种是为了存储键/值对，称为图(map)

#### 2.集合

Java集合框架支持三种主要类型的集合：规则集、线性表和队列。
Set的实例用于存储一组不重复的元素。
List的实例用于存储一个由元素构成的有序集合。
Queue的实例用于存储用先进先出方式处理的对象。
这些集合的通用特性都被定义在接口中，而它们的实现实在具体类中提供的。

#### 3.Collection接口和AbstractCollection类

Collection接口是处理对象集合的根接口。公共方法：

```java
// 返回该集合中元素所用的迭代器
iterator(): Iterator< E>

add(o: E): boolean
addAll(c: Collection<? extends E>): boolean
clear(): void
contains(o: Object): boolean
containsAll(c: Collection<?>): boolean
equals(o: Object): boolean
hashCode(): int
isEmpty(): boolean
remove(o: Object): boolean
removeAll(c: Collection<?>): boolean
retainAll(c: Collection<?>): boolean
size(): int
toArray(): Object[]
```

Iterator接口提供了对不同类型集合中的元素进行遍历的统一方法。

```java
hasNext(): boolean
next(): E
remove(): void
```

#### 4.规则集(Set)

Set接口扩展了Collection接口。它没有引入新的方法和常量，只是规定了Set的实例不包含重复的元素。
AbstractSet类是一个便利类，扩展了AbstractCollection类并实现了Set接口，提供equals方法和hashCode方法的具体实现。一个规则集的散列码是这个规则集中所有元素散列码的和。
Set接口的三个具体类是：散列集(HashSet)、链式散列集(LinkedHashSet)和树形集。

HashSet类可以用来存储互不相同的任何元素。考虑到效率的因素，添加到散列集中的对象必须以正确分散散列码的方式来实现hashCode()方法。散列集的方法：

```java
// 默认初始容量为16，客座率(loadFactor)为0.75
HashSet()
HashSet(c: Collection<? extends E>)
HashSet(initialCapacity: int)
HashSet(initialCapacity: int, loadFactor: float)
```

LinkedHashSet用一个链表实现来扩展HashSet类，支持对规则集内的元素排序。链式散列集的方法：

```java
LinkedHashSet()
LinkedHashSet(c: Collection<? extends E>)
LinkedHashSet(initialCapacity: int)
LinkedHashSet(initialCapacity: int, loadFactor: float)
```

SortedSet是Set的一个子接口，可以确保规则集中的元素是有序的。
TreeSet是实现了SortedSet接口的一个具体类，只要对象是可以互相比较的，就可以将它们添加到一个树形集中。两种比较对象的方法：使用Comparable接口和指定比较器。

```java
TreeSet()
TreeSet(c: Collection<? extends E>)
TreeSet(comparator: Comparator<? super E>)
TreeSet(s: SortedSet< E>)
```

#### 5.比较器接口(Comparator)

将元素插入到一个树集合中，如果这些元素不是`java.lang.Comparable`的实例，可以定义一个比较器来比较这些元素。

#### 6.线性表(List)

线性表不仅可以存储重复的元素，而且允许用户指定它们存储的位置。用户可以用下标来访问元素。List接口扩展了Collection接口，以定义一个允许重复的有序集合。List接口增加了面向位置(position-oriented)的操作，并且增加了一个能够双向遍历线性表的新列表迭代器。

```java
// 在指定下标处添加一个新元素
add(index: int, element: Object): boolean
// 在指定下标出添加c中的所有元素
addAll(index: int, c: Collection<? extends E>): boolean
// 返回列表中指定下标处的元素
get(index: int): E
// 返回第一个匹配的元素的下标
indexOf(element: Object): int
// 返回最后一个匹配的元素的下标
lastIndexOf(element: Object): int
// 返回这个列表的元素所用的列表迭代器
listIterator(): ListIterator< E>
// 返回从startIndex开始的元素所用的迭代器
listIterator(startIndex: int): ListIterator< E>
// 删除指定下标处的元素
remove(index: int): E
// 设置指定下标处的元素
set(index: int, element: Object): Object
// 返回从fromIndex到toIndex-1的子列表
subList(fromIndex: int, toIndex: int): List< E>
```

ListIterator接口扩展了Iterator接口，以增加对线性表的双向遍历能力。

```java
// 添加指定的对象
add(element: E): void
// 如果向后遍历时这个列表迭代器有更多的元素则返回true
hasPrevious(): boolean
// 返回下一个元素的下标
nextIndex(): int
// 返回这个列表迭代器中的前一个元素
previous(): E
// 返回前一个元素的下标
previousIndex(): int
// 使用指定的元素替换previous方法或next方法返回的最后一个元素
set(element: E): void
```

##### ArrayList和LinkedList

ArrayList用数组存储元素，这个数组是动态创建的(大小可变)。

```java
// 创建一个带默认初始容量的空列表
ArrayList()
// 从现有集合创建一个数组列表
ArrayList(c: Collection<? extends E>)
// 创建一个带指定初始容量的空列表
ArrayList(initialCapacity: int)
// 将这个ArrayList实例的容量缩小到这个列表的当前大小
trimToSize(): void
```

LinkedList除了实现List接口的方法外，还提供从线性表两端提取，插入和删除元素的方法。

```java
// 创建一个默认的空链表
LinkedList()
// 从现有集合创建一个链表
LinkedList(c: Collection<? extends E>)
// 将对象添加到这个列表头
addFirst(element: E): void
// 将对象添加到这个列表尾
addLast(element: E): void
// 返回这个列表的第一个元素
getFirst(): E
// 返回这个列表的最后一个元素
getLast(): E
// 返回和删除这个列表的第一个元素
removeFirst(): E
// 返回和删除这个列表的最后一个元素
removeLast(): E
```

若要提取元素或在线性表的尾部插入和删除元素，ArrayList的效率比较高。若要在线性表的任意位置上插入和删除元素，那么LinkedList的效率会高一些。

#### 7.线性表和集合的静态方法

Collections类中用于线性表的静态方法：

```java
// 对指定的列表进行排序
sort(list: List): void

// 使用比较器对指定的列表进行排序
sort(list: List, c: Comparator): void

// 使用二分查找法搜索有序列表中的键值
binarySearch(list: List, key: Object): int

// 使用比较器并使用二分查找法搜索有序列表中的键值
binarySearch(list: List, key: Object, c: Comparator): int

// 颠倒指定的列表
reverse(list: List): void

// 返回逆序的比较器
reverseOrder(): Comparator

// 随机打乱指定的列表
shuffle(list: List): void

// 用随机对象打乱指定列表
shuffle(list: List, rmd: Random): void

// 将源列表复制给目标列表
copy(des: List, src: List): void

// 返回包含某对象n个副本的列表
nCopies(n: int, o: Object): List

// 用对象填充列表
fill(list: List, o: Object): void
```

Collections类中用于集合的静态方法：

```java
// 返回集合中的max对象
max(c: Collection): Object

// 使用比较器返回max对象
max(c: Collection, c: Comparator): Object

// 返回集合中的min对象
min(c: Collection): Object

// 使用比较器返回min对象
min(c: Collection, c: Comparator): Object

// 如果c1和c2没有公共元素则返回true
disjoint(c1: Collection, c2: Collection): boolean

// 返回集合中指定元素的出现次数
frequency(c: Collection, o: Object): int
```

#### 8.规则集和线性表的性能

```java
import java.util.*;

public class SetListPerformanceTest {
	static final int N = 50000;

	public static void main(String[] args) {
		// 添加数字0-49999到这个数组线性表
		List<Integer> list = new ArrayList<>();
		for (int i = 0; i < N; i++)
			list.add(i);
		// 打乱这个线性表
		Collections.shuffle(list);

		// 创建一个散列集，然后测试其性能
		Collection<Integer> set1 = new HashSet<>(list);
		System.out.println("散列集的成员测试时间是 " + getTestTime(set1) + " 毫秒");
		System.out.println("散列集删除元素的时间是 " + getRemoveTime(set1) + " 毫秒");

		// 创建一个链式散列集，然后测试其性能
		Collection<Integer> set2 = new LinkedHashSet<>(list);
		System.out.println("链式散列集的成员测试时间是 " + getTestTime(set2) + " 毫秒");
		System.out.println("链式散列集删除元素的时间是 " + getRemoveTime(set2) + " 毫秒");

		// 创建一个树形集，然后测试其性能
		Collection<Integer> set3 = new TreeSet<>(list);
		System.out.println("树形集的成员测试时间是 " + getTestTime(set3) + " 毫秒");
		System.out.println("树形集删除元素的时间是 " + getRemoveTime(set3) + " 毫秒");

		// 创建一个数组线性表，然后测试其性能
		Collection<Integer> list1 = new ArrayList<>(list);
		System.out.println("数组线性表的成员测试时间是 " + getTestTime(list1) + " 毫秒");
		System.out.println("数组线性表删除元素的时间是 " + getRemoveTime(list1) + " 毫秒");

		// 创建一个链表，然后测试其性能
		Collection<Integer> list2 = new LinkedList<>(list);
		System.out.println("链表的成员测试时间是 " + getTestTime(list2) + " 毫秒");
		System.out.println("链表删除元素的时间是 " + getRemoveTime(list2) + " 毫秒");
	}

	public static long getTestTime(Collection<Integer> c) {
		long startTime = System.currentTimeMillis();
		// 测试数字是否在集合内
		for (int i = 0; i < N; i++)
			c.contains((int)(Math.random() * 2 * N));

		return System.currentTimeMillis() - startTime;
	}

	public static long getRemoveTime(Collection<Integer> c) {
		long startTime = System.currentTimeMillis();
		// 删除元素操作
		for (int i = 0; i < N; i++)
			c.remove(i);

		return System.currentTimeMillis() - startTime;
	}
}
/*
散列集的成员测试时间是 20 毫秒
散列集删除元素的时间是 16 毫秒
链式散列集的成员测试时间是 14 毫秒
链式散列集删除元素的时间是 19 毫秒
树形集的成员测试时间是 17 毫秒
树形集删除元素的时间是 25 毫秒
数组线性表的成员测试时间是 3609 毫秒
数组线性表删除元素的时间是 1387 毫秒
链表的成员测试时间是 6366 毫秒
链表删除元素的时间是 2568 毫秒
*/
```

如上所示，规则集比线性表更加高效。如果应用程序用规则集就足够，那就使用规则集。除此之外，如果程序不需要特别的顺序，就选择散列集。
这个程序测试了数组线性表和链表上常见的删除操作，它们的复杂度基本上是一样的。需要注意的是，在线性表中除了结尾以外的任意位置上进行插入或删除操作，链表会比数组线性表更加有效。

#### 9.向量类(Vector)和栈类(Stack)

除了包含用于访问和修改向量的同步方法之外，Vector类与ArrayList是一样的。同步方法用于防止两个或多个线程同时访问某个向量时引起数据损坏。对于许多不需要同步的应用来说，使用ArrayList比使用Vector效率更高。
Vector类实现了List接口，包含的方法如下：

```java
Vector()
Vector(c: Collection<? extends E>)
Vector(initialCapacity: int)
Vector(initCapacity: int, capacityIncr: int)
addElement(o: E): void
capacity(): int
copyInto(anArray: Object[]): void
elementAt(index: int): E
elements(): Enumeration< E >
ensureCapacity(): void
firstElement(): E
insertElementAt(o: E, index: int): void
lastElement(): E
removeAllElements(): void
removeElement(o: Object): boolean
removeElementAt(index: int): void
setElementAt(o: E, index: int): void
setSize(newSize: int): void
trimToSize(): void
```

在Java集合框架中，栈类Stack是作为Vector类的扩展来实现的：

```java
// 创建一个空栈
Stack()
// 如果栈为空则返回true
empty(): boolean
// 返回栈顶元素
peek(): E
// 返回并删除栈顶元素
pop(): E
// 在栈顶增加一个新元素
push(o: E): E
返回栈中指定元素的位置
search(o: Object): int
```

#### 10.队列和优先队列

队列是一种**先进先出**的数据结构。元素被追加到队列末位，然后从队列头删除。
在优先队列中，元素被赋予优先级。当访问元素时，拥有最高优先级的元素首先被删除。
Queue接口用附加的插入、提取和检验操作来扩展`java.util.Colection`：

```java
// 向队列插入一个元素
offer(element: E): boolean
// 获取并删除队列头，如果队列为空则返回null
poll(): E
// 获取并删除队列头，如果队列为空则抛出一个异常
remove(): E
// 获取但不删除队列头，如果队列为空则返回null
peek(): E
// 获取但不删除队列头，如果队列为空则抛出一个异常
element(): E
```

##### 双端队列(Deque)和链表(LinkedList)

LinkedList类实现了Deque接口，Deque又扩展了Queue接口，因此可以使用LinkedList创建一个队列。
Deque(double-ended queue)支持在两端的插入和删除元素。Deque接口用附加的从两段插入和删除元素的办法扩展Queue接口。附加方法：`addFirst(e), removeFirst(), addLast(e), removeLast(), getFirst(), getLast()`

PriorityQueue类实现一个优先队列。默认情况下，优先队列使用Comparable以元素的自然顺序进行排序。拥有最小数组的元素被赋予最高优先级，因此最先从队列中删除。

```java
// 创建一个初始容量为11的默认优先队列
PriorityQueue()
// 创建一个带指定容量的默认优先队列
PriorityQueue(initialCapacity: int)
// 创建一个带指定集合的优先队列
PriorityQueue(c: Collection<? extends E>)
// 创建一个带指定初始容量和比较器的优先队列
PriorityQueue(initialCapacity: int, comparator: Comparator<? super E>)
```

#### 11.映射(Map)

映射(或者叫图)是一种依照键值存储元素的容器。键值可以是任意类型的对象，但是不能重复。
映射的类型有三种：散列图(HashMap)、链式散列图(LinkedHashMap)和树形图(TreeMap)。
这些图的通用特性都定义在Map接口中。Map接口提供了查询、更新和获取集合的值和集合的键值的方法。

```java
/*
更新(update)方法
*/
// 删除图中所有的条目
clear(): void
// 将一个映射放入图中
put(key: K, value: V): V
// 将所有来自m的条目添加到图中
putAll(m: Map<? extends K,? extends V>): void
// 删除指定键值对应的条目
remove(key: Object): V

/*
查询(query)方法
*/
// 如果图包含指定键值对应的条目则返回true
containsKey(key: Object): boolean
// 如果图将一个或多个键值映射到特定值则返回true
containsValue(value: Object): boolean
// 如果图不包含如何条目则返回true
isEmpty(): boolean
// 返回图中条目个数
size(): int

// 获得一个包含图中键值的规则集
keySet(): Set< K >
// 获得一个包含图中值的集合
values(): Collection< V>
// 返回一个实现Map.Entry< K ,V>接口的对象集合，这里的Entry是Map接口的一个内部接口
// 该集合中的每个对象都是底层图中的一个特定的键/值对
entrySet(): Set< Map.Entry< K,V>>

// 获得指定键值对应的值
get(key: Object): V
```

Map.Entry< K, V>接口：

```java
// 返回对应到该条目的键值
getKey(): K
// 返回对应到该条目的值
getValue(): V
// 用一个新值替换该条目中的值
setValue(value: V): void
```

HashMap、LinkedHashMap和TreeMap类是Map接口的三个具体实现。
对于定位一个值、插入一个映射以及删除一个映射而言，HashMap类是高效的。

```java
HashMap()
HashMap(m: Map<? extends K, ? extends V>)
HashMap(initialCapacity: int,loadFactor: float)
```

LinkedHashMap类用链表实现来拓展HashMap类，它支持图中条目的排序。

```java
// 元素以插入图的顺序排序(插入顺序：insertion order)
LinkedHashMap()
LinkedHashMap(m: Map<? extends K,? extends V>)
// 按元素被最后一次访问时的顺序，从早到晚排序(访问顺序：access order)
LinkedHashMap(initialCapacity: int, loadFactor: float, accessOrder: boolean)
```

TreeMap在遍历排好顺序的键值时是很高效的。键值可以使用Comparable接口或Comparator接口来排序。

```java
// 假定元素的类实现了Comparable接口，则可以使用Comparable接口中的compareTo方法来排序
TreeMap()
TreeMap(m: Map<? extends K,? extends V>)
// 使用比较器来创建有序图
TreeMap(c: Comparator<? super K>)
```

三者比较：

```java
import java.util.*;

public class TestMap {
	public static void main(String[] args) {
		// 创建一个散列图
		Map<String, Integer> hashMap = new HashMap<>();
		hashMap.put("Smith", 30);
		hashMap.put("Anderson", 31);
		hashMap.put("Lewis", 29);
		hashMap.put("Cook", 29);

		System.out.println("显示散列图中的条目");
		System.out.println(hashMap + "\n");

		// 基于先前的散列图创建一个树形图
		Map<String, Integer> treeMap = new TreeMap<>(hashMap);
		System.out.println("按键值的升序显示条目");
		System.out.println(treeMap);

		// 创建一个链式散列图
		Map<String, Integer> linkedHashMap =
		new LinkedHashMap<>(16, 0.75f, true);
		linkedHashMap.put("Smith", 30);
		linkedHashMap.put("Anderson", 31);
		linkedHashMap.put("Lewis", 29);
		linkedHashMap.put("Cook", 29);

		// 显示Lewis的年龄
		System.out.println("\nLewis的年龄为 " + linkedHashMap.get("Lewis"));

		System.out.println("显示链式散列图中的条目");
		System.out.println(linkedHashMap);
	}
}
/*
显示散列图中的条目
{Smith=30, Lewis=29, Anderson=31, Cook=29}

按键值的升序显示条目
{Anderson=31, Cook=29, Lewis=29, Smith=30}

Lewis的年龄为 29
显示链式散列图中的条目
{Smith=30, Anderson=31, Cook=29, Lewis=29}
*/
```

由上可知，HashMap中条目的顺序是随机的，而TreeMap中的条目是按键值的升序排列的，LinkedHashMap中的条目则是按元素最后一次被访问的时间从早到晚排序的(最晚被访问的条目被放在图的末尾)。

#### 12.单例和不可变的集合与映射

Collections类还包含了用于创建单元素的规则集、线性表和图的方法，以及用于创建不可变(只读)规则集、线性表和图的方法。

```java
// 返回一个包含指定对象的单元素集合
singleton(o: Object): Set
// 返回一个包含指定对象的单元素列表
singletonList(o: Object): List
// 返回一个带键值对的单元素图
singletonMap(key: Object, value: Object): Map

// 返回一个不可更改的集合
unmodifiableCollection(c: Collection): Collection
// 返回一个不可更改的列表
unmodifiableList(list: List): List
// 返回一个不可更改的图
unmodifiableMap(m: Map): Map
// 返回一个不可更改的规则集
unmodifiableSet(s: Set): Set
// 返回一个不可更改的有序图
unmodifiableSortedMap(s: SortedMap): SortedMap
// 返回一个不可更改的有序规则集
unmodifiableSortedSet(s: SortedSet): SortedSet
```
































































































