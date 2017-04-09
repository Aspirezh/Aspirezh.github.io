---
layout: post
title:  "laravel关于服务提供者的再次理解"
date:   2017-03-27 23:23:32 +0800
categories: Laravel
tags:  IoC  laravel
author: Aspirezh
---

* content
{:toc}

关于laravel基本架构的理解，有助于我们的学习和使用laravel的许多功能
今天用到包开发的内容，感觉前面对IOC的理解又加深了一些。



## 介绍：

---
layout: post
title:  "laravel关于服务提供者的再次理解"
date:   2017-03-27 23:23:32 +0800
categories: Laravel—中间操作流
tags:  IoC laravel
author: Aspirezh
---

* content
{:toc}

关于laravel基本架构的理解，有助于我们的学习和使用laravel的许多功能，今天用到包开发的内容，感觉前面对IOC的理解又加深了一些。



## 介绍：

首先，要知道包开发是什么

Packages are the primary way of adding functionality to Laravel. Packages might be anything from a great way to work with dates like Carbon, or an entire BDD testing framework like Behat.

Of course, there are different types of packages. Some packages are stand-alone, meaning they work with any framework, not just Laravel. Both Carbon and Behat are examples of stand-alone packages. Any of these packages may be used with Laravel by simply requesting them in your  composer.json file.

On the other hand, other packages are specifically intended for use with Laravel. These packages may have routes, controllers, views, and configuration specifically intended to enhance a Laravel application. This guide primarily covers the development of those packages that are Laravel specific.
包是添加功能到 Laravel 的主要方式。包可以提供任何功能，小到处理日期如 Carbon，大到整个 BDD 测试框架如 Behat。

当然，有很多不同类型的包。有些包是独立的，意味着可以在任何框架中使用，而不仅是 Laravel。比如 Carbon 和 Behat 都是独立的包。所有这些包都可以通过在composer.json文件中请求以便被 Laravel 使用。
## 服务提供者
服务提供者是包和 Laravel 之间的连接点。服务提供者负责绑定对象到 Laravel 的服务容器并告知 Laravel 从哪里加载包资源如视图、配置和本地化文件。

服务提供者继承自Illuminate\Support\ServiceProvider类并包含两个方法：register和boot。ServiceProvider基类位于Composer包illuminate/support。

服务提供者可以在项目的任何地方设置:

eg:在根目录下创建文件夹 packages/jai/contact/src

进入src目录并创建一个服务提供者ContactServiceprovider.php

这就创建了一个服务提供者，它提供方法，连接你的包和项目

服务提供者具有这几种方法：

boot()方法：引入路由文件；
                    引入包视图-》loadViewsFrom()方法；
                    发布视图；
                    发布翻译文件；
                    发布前端资源
还可以自定义你的路由，或者一些注册方法：

```
public function setupRoutes(Router $router)
    {
        $router->group(['namespace' => 'Jai\Contact\Http\Controllers'], function($router)
        {
            require __DIR__.'/Http/routes.php';
        });
    }

    public function register()
    {
        $this->registerContact();
        config([
            'config/contact.php',
        ]);
    }
    private function registerContact()
    {
        $this->app->bind('contact',function($app){
            return new Contact($app);
        });
    }
```
## 路由
重要的路由文件，其实现在已经很简单了 
在服务提供者里引入路由文件后，在定义路由的时候就很方便了 ，和在框架中的路由设置一样：

```
public function boot(){
    if (! $this->app->routesAreCached()) {
        require __DIR__.'/../../routes.php';
    }
```
这样，路由就在我们的app/Http/route.php,也可以自己再定义一个路由文件，和我们原生的路由分开。