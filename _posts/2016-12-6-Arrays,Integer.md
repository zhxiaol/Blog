---
layout: post
title: Arrays,Integer
tags:
- java

categories: java
description:
---
## 冒泡排序
就是找最大值，把最大值放在依次最右边
```java
public void sort(int[]a){
       int temp = 0;
       for (int i = a.length - 1; i > 0; --i){
           for (int j = 0; j < i; ++j){
               if (a[j + 1] < a[j]){
                   temp = a[j];
                   a[j] = a[j + 1];
                   a[j + 1] = temp;
               }
           }
       }
   }
```

## 选择排序
就是找最小值，把最小值依次放在最左边
```java
public static void selectSort(int[]a)
{
    int minIndex=0;
    int temp=0;
    if((a==null)||(a.length==0))
        return;
    for(int i=0;i<a.length-1;i++)
    {
        minIndex=i;//无序区的最小数据数组下标
        for(intj=i+1;j<a.length;j++)
        {
            //在无序区中找到最小数据并保存其数组下标
            if(a[j]<a[minIndex])
            {
                minIndex=j;
            }
        }
        if(minIndex!=i)
        {
            //如果不是无序区的最小值位置不是默认的第一个数据，则交换之。
            temp=a[i];
            a[i]=a[minIndex];
            a[minIndex]=temp;
        }
    }
}¡¡
```

## Arrays.toString()
创建一个StirngBuild 遍历append
收获:能用‘[’的时候别用"["

## sort()
用的不是冒泡也不是选择，复杂程度令人发指，懒得看，听说效率很高

## binarySearch()
二分法查找，跟自己写的差不多，就是mid=(low+high)>>>1而不是/2 位运算效率更高
```java
int low = fromIndex;
        int high = toIndex - 1;

        while (low <= high) {
            int mid = (low + high) >>> 1;
            byte midVal = a[mid];

            if (midVal < key)
                low = mid + 1;
            else if (midVal > key)
                high = mid - 1;
            else
                return mid; // key found
        }
        return -(low + 1);  // key not found.
```

## Integer
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
