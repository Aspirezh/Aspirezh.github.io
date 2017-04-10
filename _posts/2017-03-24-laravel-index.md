---
layout: post
title:  "关于laravel入口和源的自我理解"
date:   2016-03-24 15:26 +0800
categories: laravel
tags:  laravel
author: Aspirezh
mathjax: true
---

* content
{:toc}

关于laravel入口和源的自我理解。




## laravel5.2

laravel的入口文件是 Public下的index.php

先看下index.php:

```
<?php

header('Access-Control-Allow-Origin: *');   // cors 所需
header('Access-Control-Allow-Methods:GET, POST, PUT, DELETE, PATCH, OPTIONS');
// header('Access-Control-Allow-Methods: *');
header('Access-Control-Allow-Credentials:false');
// header('Access-Control-Allow-Headers:DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization,token, Content-Type, CUSTOM-HEADER');
header('Access-Control-Allow-Headers:DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization,token, Content-Type, CUSTOM-HEADER,origin');
// header("Content-Security-Policy:Ddefault-src *; style-src 'self' 'unsafe-inline'; script-src 'self' 'unsafe-inline' 'unsafe-eval' http://cdn.bootcss.com");
// header('Access-Control-Allow-Headers: *');
/**
 * Laravel - A PHP Framework For Web Artisans
 *
 * @package  Laravel
 * @author   Taylor Otwell <taylorotwell@gmail.com>
 */

/*
|--------------------------------------------------------------------------
| Register The Auto Loader
|--------------------------------------------------------------------------
|
| Composer provides a convenient, automatically generated class loader for
| our application. We just need to utilize it! We'll simply require it
| into the script here so that we don't have to worry about manual
| loading any of our classes later on. It feels nice to relax.
|
*/

require __DIR__.'/../bootstrap/autoload.php';

/*
|--------------------------------------------------------------------------
| Turn On The Lights
|--------------------------------------------------------------------------
|
| We need to illuminate PHP development, so let us turn on the lights.
| This bootstraps the framework and gets it ready for use, then it
| will load up this application so that we can run it and send
| the responses back to the browser and delight our users.
|
*/

$app = require_once __DIR__.'/../bootstrap/app.php';

/*
|--------------------------------------------------------------------------
| Run The Application
|--------------------------------------------------------------------------
|
| Once we have the application, we can handle the incoming request
| through the kernel, and send the associated response back to
| the client's browser allowing them to enjoy the creative
| and wonderful application we have prepared for them.
|
*/

$kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);

$response = $kernel->handle(
    $request = Illuminate\Http\Request::capture()
);

$response->send();

$kernel->terminate($request, $response);
```
其中，代码不多

```
require __DIR__.'/../bootstrap/autoload.php';
```
laravel引入composer类加载，laravel无需手动加载需要的类。

```
$app = require_once __DIR__.'/../bootstrap/app.php';
```
 $app指向的文件，应该是定义$app的文件：

```
<?php

/*
|--------------------------------------------------------------------------
| Create The Application
|--------------------------------------------------------------------------
|
| The first thing we will do is create a new Laravel application instance
| which serves as the "glue" for all the components of Laravel, and is
| the IoC container for the system binding all of the various parts.
|
*/

$app = new Illuminate\Foundation\Application(
    realpath(__DIR__.'/../')
);

/*
|--------------------------------------------------------------------------
| Bind Important Interfaces
|--------------------------------------------------------------------------
|
| Next, we need to bind some important interfaces into the container so
| we will be able to resolve them when needed. The kernels serve the
| incoming requests to this application from both the web and CLI.
|
*/

$app->singleton(
    Illuminate\Contracts\Http\Kernel::class,
    App\Http\Kernel::class
);

$app->singleton(
    Illuminate\Contracts\Console\Kernel::class,
    App\Console\Kernel::class
);

$app->singleton(
    Illuminate\Contracts\Debug\ExceptionHandler::class,
    App\Exceptions\Handler::class
);

/*
|--------------------------------------------------------------------------
| Return The Application
|--------------------------------------------------------------------------
|
| This script returns the application instance. The instance is given to
| the calling script so we can separate the building of the instances
| from the actual running of the application and sending responses.
|
*/

//$environment = getenv('LARAVEL_ENV') ? '.' . getenv('LARAVEL_ENV') : '';
//$app->loadEnvironmentFrom('.env'.$environment);
return $app;

```

可以发现，原来$app是一个 Illuminate\Foundation\Application 对象。
继续往下找，看 Illuminate\Foundation\Application 这里边定义了什么：

```
public function __construct($basePath = null)
{
    $this->registerBaseBindings();

    $this->registerBaseServiceProviders();

    $this->registerCoreContainerAliases();

    if ($basePath) {
        $this->setBasePath($basePath);
    }
}
```
它里边调用了3个方法，registerBaseBindings()；registerBaseServiceProviders()；registerCoreContainerAliases()

```
/**
  * Register the basic bindings into the container.
  *
  * @return void
  */
protected function registerBaseBindings()
{
    static::setInstance($this);

    $this->instance('app', $this);

    $this->instance('Illuminate\Container\Container', $this);
}
```

```
**
  * Register all of the base service providers.
  *
  * @return void
  */
protected function registerBaseServiceProviders()
{
    $this->register(new EventServiceProvider($this));

    $this->register(new RoutingServiceProvider($this));
}

/**
  * Register a service provider with the application.
  *
  * @param  \Illuminate\Support\ServiceProvider|string  $provider
  * @param  array  $options
  * @param  bool   $force
  * @return \Illuminate\Support\ServiceProvider
  */
public function register($provider, $options = [], $force = false)
{
    if ($registered = $this->getProvider($provider) && !$force) {
        return $registered;
    }

    // If the given "provider" is a string, we will resolve it, passing in the
    // application instance automatically for the developer. This is simply
    // a more convenient way of specifying your service provider classes.
    if (is_string($provider)) {
        $provider = $this->resolveProviderClass($provider);
    }

    $provider->register();

    // Once we have registered the service we will iterate through the options
    // and set each of them on the application so they will be available on
    // the actual loading of the service objects and for developer usage.
    foreach ($options as $key => $value) {
        $this[$key] = $value;
    }

    $this->markAsRegistered($provider);

    // If the application has already booted, we will call this boot method on
    // the provider class so it has an opportunity to do its boot logic and
    // will be ready for any usage by the developer's application logics.
    if ($this->booted) {
        $this->bootProvider($provider);
    }

    return $provider;
}
```
这些ServiceProvider是 Illuminate\Support\ServiceProvider 的子类，它接受一个 Application 对象作为构造函数参数，存储在实例变量 $app 中。

注入所有基础 Service Provider

在 register 方法中，每个ServiceProvider被调用了自身的 register 方法。在tServiceProvider 中：

```
public function register()
{
    $this->app->singleton('events', function ($app) {
        return (new Dispatcher($app))->setQueueResolver(function () use ($app) {
            return $app->make('Illuminate\Contracts\Queue\Factory');
        });
    });
}
```
将一个 Illuminate\Events\Dispatcher 对象以键 events 绑定到了容器 中，负责实现事件的调度。
有点乱，还得在实际项目中使用，一步步理解吧。