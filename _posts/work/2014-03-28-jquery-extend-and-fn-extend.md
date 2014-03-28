---
layout: post
title: 谈谈jQuery.extend和jQuery.fn.extend的插件写法
description: jQuery支持自定义的插件有另种写法，即jQuery.extend和jQuery.fn.extend，它们有什么区别呢？
category: work
tags:
- Javascript
- jQuery
---

##jQuery.extend和jQuery.fn.extend

一般来说，jQuery插件的开发分为两种：`jQuery.extend` 和 `jQueryf.fn.extend`。

 1. 前者是挂在jQuery命名空间下的全局函数，也可称为静态方法，静态方法是不需要new一个实例再去调用的，通过“类名+方法名”直接调用。比如 `$.Ajax`, `$.getJson`, `$.post`, `$.each`…

 2. 后者是jQuery对象级别的方法，即挂在jQuery原型下的方法，这样通过选择器获取的jQuery对象实例也能共享该方法。`jQuery.fn` 等于 `jQuery.prototype`，也就是说给 `function jQuery` 的原型 `prototype` 上挂了个 `extend` 方法。通过调用 `jQuery.fn.extend(object)` 来扩展时(注意只传一个参数object)，`extend` 函数中仍然会执行。

##jQuery.extend

`jQuery.extend` 有几种常见的使用方法，介绍一下。

###扩展jQuery本身

例如：

```javascript
jQuery.extend({nick: 'casper'});

//结果：
console.log(jQuery.nick);   //输出：'casper'
```

###扩展对象

以 `jQuery.extend(css1,css2)` 为例，`css1`, `css2` 个有一些属性(法照样会比处理,这里之讲属性)。

`extend` 函数会把 `css2` 有而 `css2` 没有的属性加到 `css1` 中，如果 `css2` 的某个属性与 `css1` 的某个属性名称享用，就会用 `css2` 的属性去覆盖 `css1` 的同名属性。`css1` 就是最后的整和对象。

```javascript
//用法: jQuery.extend(obj1,obj2,obj3,..)
var Css1={size: "10px",style: "oblique"}
var Css2={size: "12px",style: "oblique",weight: "bolder"}
$.jQuery.extend(Css1,Css2);

//结果:Css1的size属性被覆盖,而且继承了Css2的weight属性
Css1 = {size: "12px",style: "oblique",weight: "bolder"}
```

或者也可以用 ：

```javascript
var newcss = jquery.extend(css1,css2);  //newcss就是合并的新对象。
var newcss = jquery.extend({},css1,css2); //newcss就是合并的新对象.而且没有破坏css1的结构。
```

###浅度拷贝-当object中存在引用

如下代码，obj1.scores的值是个指向对象的引用，当obj2中存在同名应用时，默认obj2中的同名引用会覆盖obj1中那个。

```javascript
var obj1 = { nick: 'casper',  scores: { math: 100, English: 100 } },
    obj2 = { scores: { hitory: 100 } },
    obj3 = jQuery.extend(obj1, obj2);
```

打印下：

```javascript
console.log( JSON.stringify(obj1) );    // 输出 {"nick":"casper","scores":{"hitory":100}}
```

###深度拷贝-当obj中存在引用

不同的是，当第一个参数改成 `true`，就成了深拷贝，对象的子对象的属性也会被覆盖和合并。

例如以下代码：

```javascript
var obj1 = { nick: 'casper',  scores: { math: 100, English: 100 } },
    obj2 = { scores: { hitory: 100 } },
    obj3 = jQuery.extend( true, obj1, obj2 );
```

打印下：

```javascript
console.log( JSON.stringify(obj1) );    // 输出 {"nick":"casper","scores":{"math":100,"English":100,"hitory":100}}
```

###扩展jQuery静态方法

也是我们使用的最多的，例如代码：

```javascript
(function($){
    $.extend({
        test : function(){
            alert('test函数');
        }
    });
})(jQuery);

//调用：
$.test();  //或写成：jQuery.test();
```

##jQuery.fn.extend

`jQuery.fn` 等于 `jQuery.prototype`，也就是说给 `function jQuery` 的原型(prototype)上挂了个 `extend` 方法。通过调用 `jQuery.fn.extend` 来扩展时(注意只传一个参数object)，extend函数中仍然会执行 `target = this;`。

而这时的 `this` 则是 `jQuery.prototype`（上面则是jQuery函数自身）。即给 `jQuery.prototype` 上添加了许多属性，方法。当 `jQuery` 函数执行时，如 `$()` 或 `jQuery()`，更多时候是 `$()`。该函数会new一个jQuery（见上一篇jquery对象的组成）。这时则把扩展的属性，方法都附加到new生成的对象上了。也许下面这个示例更容易理解：

```javascript
function fun(){}            //定义一个类（函数）

//给该类原型上添加一个方法extned
fun.prototype.extend = function(obj){
    for(var a in obj)
        this[a] = obj[a];   //注意：这里的this即是fun.prototype
}

//调用extend方法给fun.prototype上添加属性，方法
fun.prototype.extend({name:"fun2",method1:function(){}})
//输出name,extend,method1
console.dir(new fun())
```

因此扩展的属性或方法都添加到jquery对象上了。
如 `bind`, `one`, `unbind`等可以通过`$("…").bind`, `$("…").one`, `$("…").unbind` 方式调用。却不能通过`$.bind`, `$.one`, `$.unbind` 方式调用。

jquery库与prototype库一样都是通过extend方法扩展出整个库的。相对来说jqueyr的扩展方式更难理解一些。总结如下:

 - `jQuery.extend({…})`是给function jQuery添加静态属性或方法
 -`jQuery().extend({…})`是给jquery对象添加属性或方法

而我们通常的写法为：

```javascript
//sample:扩展jquery对象的方法，bold()用于加粗字体。
        (function ($) {
            $.fn.extend({
                "bold" : function () {
                    ///<summary>
                    /// 加粗字体
                    ///</summary>
                    return this.css({ fontWeight: "bold" });
                }
            });
        })(jQuery);
```

调用方式：

```javascript
$(function(){
    $('p').bold();
});
```

##jQuery1.4.2中的extend函数源码及Ajax代码示例：

```javascript
/*!
 * jQuery源码分析-extend函数
 * jQuery版本:1.4.2
 *
 * ----------------------------------------------------------
 * 函数介绍
 * jQuery.extend与jQuery.fn.extend指向同一个函数对象
 * jQuery.extend是jQuery的属性函数(静态方法)
 * jQuery.fn.extend是jQuery函数所构造对象的属性函数(对象方法)
 *
 * ----------------------------------------------------------
 * 使用说明
 * extend函数根据参数和调用者实现功能如下：
 * 1.对象合并:
 * 对象合并不区分调用者,jQuery.extend与jQuery.fn.extend完全一致
 * 也就是说对jQuery对象本身及jQuery所构造的对象没有影响
 * 对象合并根据参数区分,参数中必须包括两个或两个以上对象
 * 如:$.extend({Object}, {Object}) 或 $.extend({Boolean},{Object}, {Object})
 * 对象合并返回最终合并后的对象,支持深度拷贝
 *
 * 2.为jQuery对象本身增加方法:
 * 这种方式从调用者和参数进行区分
 * 形式为 $.extend({Object})
 * 这种方式等同于 jQuery.{Fnction Name}
 *
 * 3.原型继承:
 * 原型继承方式可以为jQuery所构造的对象增加方法
 * 这种方式也通过调用者和参数进行区分
 * 形式为 $.fn.extend({Object})
 * 这种方式实际上是将{Object}追加到jQuery.prototype,实现原型继承
 *
 * ----------------------------------------------------------
 *
 */
// jQuery.fn = jQuery.prototype
// jQuery.fn.extend = jQuery.prototype.extend
jQuery.extend = jQuery.fn.extend = function(){
    //目标对象
    var target = arguments[0] || {},
    //循环变量,它会在循环时指向需要复制的第一个对象的位置,默认为1
    //如果需要进行深度复制,则它指向的位置为2
    i = 1,
    //实参长度
    length = arguments.length,
    //是否进行深度拷贝
    //深度拷贝情况下,会对对象更深层次的属性对象进行合并和覆盖
    deep = false,
    //用于在复制时记录参数对象
    options,
    //用于在复制时记录对象属性名
    name,
    //用于在复制时记录目标对象的属性值
    src,
    //用于在复制时记录参数对象的属性值
    copy;
    //只有当第一个实参为true时,即需要进行深度拷贝时,执行以下分支
    if (typeof target === "boolean") {
        //deep = true,进行深度拷贝
        deep = target;
        //进行深度拷贝时目标对象为第二个实参,如果没有则默认为空对象
        target = arguments[1] || {};
        //因为有了deep深度复制参数,因此i指向的位置为第二个参数
        i = 2;
    }
    //当目标对象不是一个Object且不是一个Function时(函数也是对象,因此使用jQuery.isFunction进行检查)
    if (typeof target !== "object" && !jQuery.isFunction(target)) {
        //设置目标为空对象
        target = {};
    }
    //如果当前参数中只包含一个{Object}
    //如 $.extend({Object}) 或 $.extend({Boolean}, {Object})
    //则将该对象中的属性拷贝到当前jQuery对象或实例中
    //此情况下deep深度复制仍然有效
    if (length === i) {
        //target = this;这句代码是整个extend函数的核心
        //在这里目标对象被更改,这里的this指向调用者
        //在 $.extend()方式中表示jQuery对象本身
        //在 $.fn.extend()方式中表示jQuery函数所构造的对象(即jQuery类的实例)
        target = this;
        //自减1,便于在后面的拷贝循环中,可以指向需要复制的对象
        --i;
    }
    //循环实参,循环从第1个参数开始,如果是深度复制,则从第2个参数开始
    for (; i < length; i++) {
        //当前参数不为null,undefined,0,false,空字符串时
        //options表示当前参数对象
        if ((options = arguments[i]) != null) {
            //遍历当前参数对象的属性,属性名记录到name
            for (name in options) {
                //src用于记录目标对象中的当前属性值
                src = target[name];
                //copy用于记录参数对象中的当前属性值
                copy = options[name];
                //存在目标对象本身的引用,构成死循环,结束此次遍历
                if (target === copy) {
                    continue;
                }
                //如果需要进行深度拷贝,且copy类型为对象或数组
                if (deep && copy && (jQuery.isPlainObject(copy) || jQuery.isArray(copy))) {
                    //如果src类型为对象或数组,则clone记录src
                    //否则colne记录与copy类型一致的空值(空数组或空对象)
                    var clone = src && (jQuery.isPlainObject(src) || jQuery.isArray(src)) ? src : jQuery.isArray(copy) ? [] : {};
                    //对copy迭代深度复制
                    target[name] = jQuery.extend(deep, clone, copy);
                    //如果不需要进行深度拷贝
                } else if (copy !== undefined) {
                    //直接将copy复制给目标对象
                    target[name] = copy;
                }
            }
        }
    }
    //返回处理后的目标对象
    return target;
};



/**
 * jQuery框架本身对extend函数的使用非常频繁
 * 典型示例为jQuery.ajax
 *
 */
//使用extend对jQuery对象本身进行扩展,只给了一个参数对象
//该对象中的属性将被追加到jQuery对象中
jQuery.extend({
    //jQuery.ajax
    //$.ajax
    //这里的origSettings参数是自定义的ajax配置
    //jQuery对象本身有一个ajaxSettings属性,是默认的ajax配置
    ajax: function(origSettings){
        //这里使用extend对ajax配置项进行合并
        //第一个参数表示进行深度拷贝
        //首先将第3个参数jQuery.ajaxSettings(即jQuery默认ajax配置)复制到第2个参数(一个空对象)
        //然后将第4个参数(自定义配置)复制到配置对象(覆盖默认配置)
        //这里的s就得到了最终的ajax配置项
        var s = jQuery.extend(true, {}, jQuery.ajaxSettings, origSettings);
        //其它相关代码...(省略)
    }
});
```