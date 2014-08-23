---
layout: post
title: WordPress 用固定图片和文章首图弥补特色图片
description: 最近写了好几个 WordPress 主题，发现其中存在一个显示特色图片的问题，因为比较普遍，特此封装成一个方法，方便使用。
category: ['work','wordpress']
tags:
- WordPress
- PHP
---


##需求阐述

最近写了好几个 WordPress 主题，发现其中存在一个普遍的问题，就是在首页或者归档、分类页面展示文章的时候，如果信息中配带图片会比较麻烦。因为出于布局固定，那里无论如何要展示一张图片。而通常文章中不一定会有特色图片或者作者忘记设置特色图片，那么此时就需要设定另一张图片去弥补。

##解决方案

如果文章上传了特色图片，那么优先显示；如果未上传，则抓取改文章内容的第一张图片在此显示；而如果该文章中没有图片，则：A. 显示主题中自带的一张图片；B. 随机显示主题中自带的多张图片中的一张。OK，完美解决丢图问题。

##方法封装

将A,B两种方案分别封装方法。添加在主题的`functions.php`文件中。

###方案A

```php
<?php

function get_thumb($width = null, $height = null){
	global $post;

	$no_thumb = '<img src="'. get_template_directory_uri() .'/images/thumb.png" alt="'. get_the_title() .'" width="'. $width .'" height="'. $height .'"/>';

	if ($post) $id = $post -> ID;

	if( has_post_thumbnail($id) ) {

		echo get_the_post_thumbnail( $id, 'thumbnail', array(
				'alt' => get_the_title()
			) );
	} else {
		$content = get_the_content();
		preg_match_all('|<img.*?src=[\'"](.*?)[\'"].*?>|i', $content, $images);
		if($images[1][0]) {
			echo '<img src="'. $images[1][0] .'" alt="'. get_the_title() .'" width="'. $width .'" height="'. $height .'"/>';
		} else {
			echo $no_thumb;
		}
	}
}

?>
```

将图片`thumb.png`放在主题下的`images`文件夹即可。

###方案B

```php
<?php

function get_thumb($width = null, $height = null){
	global $post;

	$rand = rand(1,5); //修改参数，5 是所有随机图片的数量
	
	$no_thumb = '<img src="'. get_template_directory_uri() .'/images/thumb'. $rand .'.png" alt="'. get_the_title() .'" width="'. $width .'" height="'. $height .'"/>';

	if ($post) $id = $post -> ID;

	if( has_post_thumbnail($id) ) {

		echo get_the_post_thumbnail( $id, 'thumbnail', array(
				'alt' => get_the_title()
			) );
	} else {
		$content = get_the_content();
		preg_match_all('|<img.*?src=[\'"](.*?)[\'"].*?>|i', $content, $images);
		if($images[1][0]) {
			echo '<img src="'. $images[1][0] .'" alt="'. get_the_title() .'" width="'. $width .'" height="'. $height .'"/>';
		} else {
			echo $no_thumb;
		}
	}
}

?>
```

将所有图片依次命名为`thumb1.png`、`thumb2.png`、`thumb2.png`...放在主题下的`images`文件夹即可。

##关于方法调用

调用`<?php get_thumb(400 , 300); ?>`，带参数的话，则可以直接设定图片图片的显示大小。
调用`<?php get_thumb(); ?>`，不带参数的话，则原大小显示图片，可以通过css控制大小。
