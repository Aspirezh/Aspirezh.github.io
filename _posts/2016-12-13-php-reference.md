﻿---
layout: post
title:  "基础php引用"
date:   2016-12-13 22:58:18 +0800
categories: 基础php引用
tags:递归 引用  
author: Aspirezh
---

* content
{:toc}

对于PHP的引用，主要表现在变量、函数、对象的引用，
在变量、函数或者是对象前面加&，就相当于引用了变量
删除引用的变量，只会影响访问的变量，内容不会销毁。
整理以前的记录，以后会修改






## php的引用允许两个不同的变量指向同一内存内容：
### 代码，简单例子
```
?php
   $a = p;
   $b = &$a;
   echo $a;//p
   echo $b;//p
   $b = q;
   echo $a;//q
?> 
```
### 在自己学习的过程中，感觉引用和递归一起用比较舒服：
所以写了一个简单递归的例子： 
```
function kv(a=0,&re=array()){
a++;if(a<5) { 
re[]=a; 
kv(a,re); 
} 
echo a;
return re; 
} 
```
稍微整理了下以前用过的，关于函数和对象的引用，其实是差不多的，引用对象的话 在php的面向对象中还有个函数，__clone();函数，可以学习下。

