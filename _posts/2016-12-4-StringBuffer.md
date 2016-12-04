---
layout: post
title: StringBuffer
tags:
- java

categories: java
description:
---
##主题介绍
char[]数组 初始容量16 每次扩容会容量*2+2 扩容调用System.arrayCopy();
线程安全，有final修饰不能有子类
##length(),capacity()
StringBuffer sb=new StringBuffer()
sb.length():实际长度 return count; 0
sb.capacity:理论长度 return value.length;16
##toString()
return new String(value,0,count);
底层还是调用System.arrayCopy(),
将StringBuffer中的char数组从0，到实际长度count复制给String中的value数组
##append
