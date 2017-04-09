---
layout: post
title:  "Laravel—中间操作流(Builder)miss error"
date:   2017-03-30 17:35:32 +0800
categories: Laravel—中间操作流
tags:  Laravel Builder 中间操作流
author: Aspirezh
---

* content
{:toc}

首先，数据库大都是链式操作，在laravel中，出现一个builder的定义，可以理解为中间流操作，或者是中间构造器。



## 自己遇到的错误：

```
Undefined property: Illuminate\Database\Eloquent\Builder::$id (View: D:\xampp\htdocs\tell\resources\views\admin\Carindex.blade.php)
```
我的代码：

```
$car = Car::orderby('id','desc')->get();

```
其中，代码中::orderby('id','desc')就是中间流操作，它会生成一个对象builder，这个builder最后通过一个get方法触发数据库操作，其实它是类似于构造函数，如果没有这个对象，是不可能调用get方法的，构造函数同样是如果没有new一个出来（不论你是new还是怎么产生的对象），都不能调用构造函数。

```
$car = Car::orderby('id','desc')->ID;

```
这里的ID是不会调用的，因为它还是builder对象，并不是Car对象。所以可以使用触发的方法，来将这个对象给它终止操作，`first()` `get()` `paginate()` `count()` `delete()`等等。

然而有时候还是会出事，例如我加了get方法后，出现一个collection对象，在表单进行操作还是会报错，所以，我使用了：

```
@foreach ($cars->chunk(3) as $chunk)
            <div class="row">
                @foreach ($chunk as $cars)
                    <div class="col-xs-4">{{ $cars->id }}</div>
                @endforeach
            </div>
        @endforeach
```
方法，OK