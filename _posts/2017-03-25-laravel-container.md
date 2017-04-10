---
layout: post
title:  "关于laravel Container 的最初想法"
date:   2017-03-25 10:46 +0800
categories: laravel
tags:  laravel Container
author: Aspirezh
mathjax: true
---

* content
{:toc}

关于laravel Container 的最初想法。



## 开始
初学laravel框架，感觉很烦，各种调用   目前虽然没开始项目，但是记录一些东西  总是好的
对于laravel容器IoC的一些理解，自己的理解，至于对不对，还得在实际项目中应用：

Laravel容器是用来放Service的地方，这些Service就是一个个绑定到容器中的实例对象或闭包或其他的，绑定方式主要三种：bind(),singleton() and instance()，从容器中解析Service方式：make()，这些都在\Illuminate\Container\Container，而\Illuminate\Foundation\Application extends Container。

关于Container是如何工作的，当在Controller中使用Constructor Injection或Method Injection时，就已经在使用Container了，因为Container会自动帮你把这些服务解析出来，如：

```
class AccountController extends Controller
{
    // 这里是Method Injection，Container会自动解析出Request，而不需要去new Request获得对象
    // 从容器中解析服务是用的Container::make()方法
    public function test(Request $request) 
    {
        return $request->ip();
    }
}
```

所以容器服务一直都在用，总的来说容器就是分离了服务构造的场所，传统是在被依赖对象内构造所需要的服务，而现在是在容器这个场所内构造所需依赖，而且是自动化的，并自动化注入到被依赖对象内，这样就实现了解耦。

更好的理解，可以去看下[源码](http://laravelacademy.org/post/769.html)

```
<?php

namespace App\Http\Controllers;

use App\User;
use App\Repositories\UserRepository;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    /**
     * User Repository 的实现。
     *
     * @var UserRepository
     */
    protected $users;

    /**
     * 创建新的控制器实例。
     *
     * @param  UserRepository  $users
     * @return void
     */
    public function __construct(UserRepository $users)
    {
        //这里已经注入
        $this->users = $users;
    }

    /**
     * 显示指定用户的详细信息。
     *
     * @param  int  $id
     * @return Response
     */
    public function show($id)
    {    
        //开始使用！你看见我有new过users对象吗？
        $user = $this->users->find($id);

        return view('user.profile', ['user' => $user]);
    }
}
```