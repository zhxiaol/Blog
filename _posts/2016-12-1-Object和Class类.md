---
layout: post
title: Object类和Class类
tags:
- java

categories: java
description:
---
## 主题介绍
Object类是所有类的祖宗(直接父类或者间接父类)
Class类是对象的字节码文件，Class类的父类也是Object

<!-- more -->

## hashCode()
返回该对象的哈希码值，该方法会根据对象地址来计算
不同对象的hashCode()一般来说不会相同（亲测String类不同对象hashCode竟然有可能相同,尼玛太神奇了 new String("abc")堆内存中的对象和常量池"abc"对象hashCode()相同）。
同一个对象的hashCode肯定相同

## getClass()
返回对象运行时类，可以通过Class类中的getName()方法获取对象的真实类全名称

## toString()
return getClass().getName()+"@"+Integer.toHexString(hashCode()); return com.zhxiaol.Person@673a3322
