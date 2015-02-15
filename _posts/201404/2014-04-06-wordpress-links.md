---
layout: post
title: WordPress 通过 get_option 取值格式化输出
date: 2014-04-06 11:23:10
categories: php
tags: wordpress
author: Aisin
---

最近在给客户开发网站，底部需要加一个友情链接。当然对于WordPress来说，有现成的功能，很简单。但是对于客户来说，希望有一种傻瓜似的添加方式。于是我就想到了这种方式。

后台通过 options 设置一个文本域：

```php
<?php 
    function as_links(){
    $all_the_links =  get_option('as_links');//后台文本域取值
    if($all_the_links){
        $all_links = explode("\n", $all_the_links);
        foreach ($all_links as $link) {
            $link = explode("|", $link );
            echo '&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;<a href="'.trim($link[0]).'" title="'.esc_attr(trim($link[1])).'">'.trim($link[1]).'</a>';
        }
    }
}
?>
```

后台文本域的填写格式：

```html
http://aisin.me | Aisin's Blog
http://www.baidu.com | 百度
http://www.qq.com | 腾讯
```

其实，这只是一种思路，取到值之后前台处理即可。可以通过这种方式取代很多繁琐的设置。尤其适用于对 WordPress 不熟悉的人。

PS：WordPress 后台开启友情链接功能，在主题的`functions.php`中添加这句话即可：

```php
<?php 
    add_filter( 'pre_option_link_manager_enabled', '__return_true' );
?>
```
