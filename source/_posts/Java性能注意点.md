---
title: Java性能注意点
date: 2016-11-04 15:56:32
categories: 性能进阶
tags:
- java
- 性能
---

参考：[http://www.importnew.com/16181.html](http://www.importnew.com/16181.html "")
* 1.stringBuilder.append 比 + 号的性能要优。

* 2.要避免使用正则表达式。

* 3.避免使用iterator迭代器来循环。尽量使用基本循环和增强for循环。

* 4.应该极力避免使用包装类。
> 这样做的坏处是给GC带来了很大的压力。GC将会为清除包装类生成的对象而忙得不可开交。
所以一个有效的优化方法是使用基本数据类型、定长数组，并用一系列分割变量来标识对象在数组中所处的位置。
遵循LGPL协议的 trove4j 是一个Java集合类库，它为我们提供了优于整形数组 int[] 更好的性能实现。

* 5.避免递归。

* 6.遍历map使用entrySet()。

* 7.使用EnumSet或EnumMap,来保存确定的少的键值。而不是使用hashSet货hashMap。

* 8.优化自定义hasCode()方法和equals()方法。
		// AbstractTable一个通用Table的基础实现：
		@Override
		public int hashCode() {

			// [#1938] 与标准的QueryParts相比，这是一个更加高效的hashCode()实现
			return name.hashCode();
		}
