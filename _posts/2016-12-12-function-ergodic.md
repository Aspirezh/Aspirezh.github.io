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
function my_scandir($dir)  
{  
  $files = array();  
  if ( $handle = opendir($dir) ) { 
    while ( ($file = readdir($handle)) !== false ) {  
      if ( $file != ".." && $file != "." ) {  
        if ( is_dir($dir . "/" . $file) ) {  
        $files[$file] = scandir($dir . "/" . $file);  
        }else {  
        $files[] = $file;  
        }  
      }  
    }  
    closedir($handle);  
    return $files;  
  }  
}

$files=my_scandir('D:\www\moning');
print_r($files);
```