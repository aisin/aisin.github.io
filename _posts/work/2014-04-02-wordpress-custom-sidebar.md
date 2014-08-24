---
layout: post
title: WordPress 让主题支持自定义侧边栏小工具的功能
description: WordPress 让主题支持自定义侧边栏小工具的功能
category: ['work', 'wordpress']
tags:
- WordPress
- PHP
---

##注册侧边栏 register_sidebar


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
	
    //注册侧边栏函数 register_sidebar
    function aisin_add_widget(){
        register_sidebar( $args ); 
    }
?>
```

##调用侧边栏


```php
<?php if ( is_active_sidebar( 'aisin_home' ) ) { 
	dynamic_sidebar( 'aisin_home' ); 
} ?>
```

或者：

```php
<?php if (function_exists('dynamic_sidebar') && dynamic_sidebar('aisin_home')) {
	dynamic_sidebar( 'aisin_home' ); 
} ?>
```
