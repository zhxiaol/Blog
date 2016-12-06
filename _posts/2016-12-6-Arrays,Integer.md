---
layout: post
title: Arrays,Integer
tags:
- java

categories: java
description:
---
##冒泡排序
就是找最大值，把最大值放在依次最右边
##选择排序
就是找最小值，把最小值依次放在最左边
##Arrays.toString()
创建一个StirngBuild 遍历append
收获:能用‘[’的时候别用"["
##sort()
用的不是冒泡也不是选择，复杂程度令人发指，懒得看，听说效率很高
##binarySearch()
二分法查找，跟自己写的差不多，就是mid=(low+high)>>>1而不是/2 位运算效率更高
##Integer
int的包装类，jdk1.5之后添加了自动封箱，和自动拆箱
IntegerCache类 缓冲池中有一个长度255的数组存储范围-128~127（存储在常量池）
如果取值范围在-128~127,自动装箱不会新创建对象，会从常量池中取
new Integer(97)==new Integer(97) flase

Integer i1=97;
Integer i2=97;
i1==i2                           true

Integer i1=197;
Integer i2=197;
i1==i2                           flase
