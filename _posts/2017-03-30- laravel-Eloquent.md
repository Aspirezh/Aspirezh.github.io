---
layout: post
title:  "写一个函数，能够遍历一个文件夹下的所有文件和子文件夹"
date:   2016-12-12 12:11 +0800
categories: 遍历
tags:  遍历 php函数
author: Aspirezh
mathjax: true
---

* content
{:toc}

怎么说呢，自己的记录远远没有文档说的好，也没有文档给的详细，全当自己理解的一个记录吧。




## 理解

laravel Eloquent:Collection
<h3>**详细文档地址**<h3>

<h4>[链接](https://laravel.com/docs/5.3/eloquent-collections)

下面是我关于对Eloquent:collection的理解：
在上一篇中提到过一个对象builder，Illuminate\Database\Eloquent\Builder
它是一个对象构造器，或者是说它是一个中间操作流，他产生了一个对象就做builder。
而对于collecttion来说，Illuminate\Support\Collection类为处理数组数据提供了平滑、方便的封装。Collection类允许你使用方法链对底层数组执行匹配和减少操作，通常，没个Collection方法都会返回一个新的Collection实例。
下面就写在项目中遇到的collection方法，全部方法，请看源文档，在链接的地方：
集合方法;
1 all()

```
collect([1, 2, 3])->all();

// [1, 2, 3]
```
2 chunk()
将一个大集合分成几个小集合

```
$collection = collect([1, 2, 3, 4, 5, 6, 7]);
$chunks = $collection->chunk(4);
$chunks->toArray();
// [[1, 2, 3, 4], [5, 6, 7]]
```


应用实例的话就是在你的blade中很好用：

```
@foreach ($products->chunk(3) as $chunk)
    <div class="row">
        @foreach ($chunk as $product)
            <div class="col-xs-4">{{ $product->name }}</div>
        @endforeach
    </div>
@endforeach
```

3 collapse()
取值的时候比较方便  比较好调用

```
$collection = collect([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

$collapsed = $collection->collapse();

$collapsed->all();

// [1, 2, 3, 4, 5, 6, 7, 8, 9]
```
4 count()
不多说，计数用的
5 first()
取第一行的值，数据库如果取一行值的话比较好用
6 flatten()
flatten方法将多维度的集合变成一维

```
$collection = collect(['name' => 'taylor', 'languages' => ['php', 'javascript']]);

$flattened = $collection->flatten();

$flattened->all();

// ['taylor', 'php', 'javascript'];
```
7 get()
get方法返回给定键的数据项，如果不存在，返回null：

```
$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

$value = $collection->get('name');

// taylor
```
8 sort()
数组排序，不多赘述
9 toArray() toJson()
转化为数组、json


<h4>**手上能经常碰到的 遇到问题的话  最好去找手册，把常用的记住就ok了**