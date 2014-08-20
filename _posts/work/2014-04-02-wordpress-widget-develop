---
layout: post
title: WordPress 小工具的开发教程
description: WordPress 小工具的开发教程笔记
category: work/wordpress
tags:
- WordPress
- PHP
---

##注册小工具register_sidebar

```php
<?php
 $args = array(
	//侧边栏的名字（默认是 'Sidebar' 加 数字 ID）
	'name'          => sprintf(__('Sidebar %d'), $i ),
	//侧边栏 ID，必须全部小写，不带空格（默认是sidebar-ID）
	'id'            => 'aisin_home',
	//描述
	'description'   => '',
	//分配到小工具 HTML输出 中的CSS选择器名字（默认为空）
    'class'         => '',
	//小工具之前的html代码
	'before_widget' => '<li id="%1$s" class="widget %2$s">',
	//小工具之后的html代码
	'after_widget'  => '</li>',
	//小工具title之前的html代码
	'before_title'  => '<h2 class="widgettitle">',
	//小工具title之后的html代码
	'after_title'   => '</h2>' ); 
	
    //注册小工具
    function aisin_add_widget(){
        register_sidebar( $args ); 
    }
    
    //添加action
    add_action( 'widgets_init', 'aisin_add_widget' );
?>
```

##调用小工具

```php
    <?php if ( is_active_sidebar( 'aisin_home' ) ) { 
	    dynamic_sidebar( 'aisin_home' ); 
    }?>
```
