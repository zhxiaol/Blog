---
layout: post
title: StringBuffer
tags:
- java

categories: java
description:
---
## 主题介绍
可变的字符序列，线程安全，有final修饰不能有子类
char[]数组 初始容量16 每次扩容会容量*2+2 扩容调用System.arrayCopy();

## length(),capacity()
StringBuffer sb=new StringBuffer()
sb.length():实际长度 return count; 0
sb.capacity:理论长度 return value.length;16

## toString()
return new String(value,0,count);
底层还是调用System.arrayCopy(),
将StringBuffer中的char数组从0，到实际长度count复制给String中的value数组

## append(String str)
str.getChars(0, len, value, count);
将String中的字符复制到value数组

## insert(int index,String str)
System.arraycopy(value, offset, value, offset + len, count - offset);
System.arraycopy(str, 0, value, offset, len);

## deleteAt(int index),delete(int start,int end)
{0,1,2,3} 1-2
System.arraycopy(value, index+1, value, index, count-index-1);
count--
System.arraycopy(value, start+len, value, start, count-end);
count -= len;

## replce(int start,int end,String str)
System.arraycopy(value, end, value, start + len, count - end);
str.getChars(value, start);

## String substring(int start)
return new String(value, start, end - start);
