---
layout: post
title:  "写一个函数，能够遍历一个文件夹下的所有文件和子文件夹"
date:   2016-12-12  12:11 +0800
categories: 遍历
tags:  php函数 遍历
author: Aspirezh
mathjax: true
---

* content
{:toc}




##这个
初学php遇到的一个问题，以后会补充
遍历文件夹，遍历什么都一样


```
    function my_scandir($dir){
    $files = array();
    if ( $handle = opendir($dir)){
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
    
    
    
    
    
    
    
    
    
    
    
    
    

