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

初学php遇到的一个问题，以后会补充，遍历文件夹，遍历什么都一样




## 代码

首先要明确使用定时的这种业务场景：
        规定的时间执行某个操作或者执行一条sql语句。
1 创建一条命令
        php artisan make:console HelloLaravelAcademy --command=laravel:academy
        执行完成后会在**app/Console/Commands**下生成一个 HelloLaravelAcademy的文件。
```
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Foundation\Inspiring;

class HelloLaravelAcademy extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'inspire';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Display an inspiring quote';

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        //$this->comment(PHP_EOL.Inspiring::quote().PHP_EOL);
        return "my name id Aspire";
    }
}

```
其中$signature是在控制台的命令名 $description是命令的描述，$handle是命令所执行的方法

在运行命令之前  还需要将注册到App\Console\Kernel的$commands属性中，

```
class Kernel extends ConsoleKernel
{
    /**
     * The Artisan commands provided by your application.
     *
     * @var array
     */
    protected $commands = [
        // Commands\Inspire::class,
    ];

    /**
     * Define the application's command schedule.
     *
     * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
     * @return void
     */
}
```

执行控制台的方法 php artisan laravel:academy，可以看到 my name is Aspire.
设置定时任务：
执行 crontab -e  

在文档的末尾输入*/1 * * * * PHP的完整路径/usr/bin/php /home/xjk/php/leguanzhu/artisan  laravel:academy 1>> /dev$> /dev/null 2>&1


*/1 * * * * 表示每分钟执行一次（可以看看cron语法）
cron的语法

分 小时 日 月 星期 命令
0-59 0-23 1-31 1-12 0-6 command (取值范围,0表示周日一般一行对应一个任务)

cron中几个特殊符号的含义

"*"代表取值范围内的数字,
"/"代表"每",
"-"代表从某个数字到某个数字