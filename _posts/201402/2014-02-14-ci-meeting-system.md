---
layout: post
title: 基于 CodeIgniter 的会议室预定微系统
date: 2014-02-20 10:00:00
categories: php
tags: CodeIgniter
author: Aisin
---

##前言

实习期间加入了公司的QQ群，平时宣布个通知啊、喊谁拿个快递啊什么的，还是挺方便的。但是也常常会有一些对于大多数人不需要关注的信息，比如说预定几点到几点的某个会议室，每天都会有很多条。开始没有在意，但是过了几个月之后，发现所有人都是这样用，每次大家都怕错过一些重要信息，但是打开之后常常是预定会议室的信息，很容易被打扰。还有关键是喊一下基本起不到什么作用，因为你上午预定了某个会议室的某个时段，而另一个人在预定这个会议室的相同时段时很少会去翻看聊天记录，即便去翻你也不知道去翻哪天的，因为他有可能在周一就预定了下周五的会议室。所以总之使用QQ群预定简直太低效了。

我感觉作为一家IT公司，使用这样原始而且几乎起不到任何作用的协作方式，实在说不过去。所以，上个月我在OA中提了一个需求，通过一种很幽默的方式把这个需求场景描述的很详细，领导看过后，哈哈一笑，觉得深有同感。其他人也大都觉得有改良的必要。结果呢，领导在批复意见上说：需求不错，建议也很好，这个月的需求提报我先给你3分。你尝试着去做这样一个东西，做好之后，下个月的需求提报我给你10分。。。

顿时感觉领导激励员工的方法好高(jī)明(zéi)啊……

算啦，做就做吧，那就使用最近一直在研究的 CodeIgniter 尝试一下。

##实例

公司在3楼、8楼和12楼一共有6个会议室，我设计的是每个会议室是一个独立的类，也就是一个独立的页面。这样在 `application/controllers` 里面就有了6个方法类。

```html
application/controllers/
|--m12.php
|--m12_small.php
|--m8.php
|--m8_small.php
|--m3.php
|--mc.php
```

在预定的时候，我设计的展现形式是 Google 日历的形式，以每周为单位，展示本周中该会议室每天的使用情况，包括会议的起止时间、会议主题和预定人。最终前端使用 `jQuery.weekCalendar` 插件。效果如下：

![{{ page.title }}]({{site.cdn_img}}201402/ci-meeting-system-1.png)

那么这样数据库表结构也就相应出来了，一共6张结构一样的表。其实如果是实际比较大的生产项目一张表比较合理，但是这里考虑以后维护清晰一些，并且只有这一个业务，也就无所谓了。

```sql
CREATE TABLE IF NOT EXISTS `m12` (
	`sid`  varchar(50) NOT NULL ,
	`start`  varchar(100) NULL ,
	`end`  varchar(100) NULL ,
	`topic`  varchar(500) NULL ,
	`person`  varchar(50) NULL ,
	PRIMARY KEY (`ID`) )
ENGINE=MyISAM DEFAULT CHARSET=utf8 ;
```

其实，预定就是一个增删改查的基本业务，并不复杂。起初想做一个用户权限的功能，但是后来想想感觉必要性不大，那样不仅需要每个人分配一个帐号，并且每次预定前都需要先登录，实在麻烦了很多，毕竟大家使用它只是预定一下会议室或者看看会议室的使用情况嘛，所以干脆做成完全开放的。

###显示预定会议

打开每个会议室的页面，从数据库中查询出该会议室的所有会议，以数组的形式赋给页面对象的数组属性 `eventData.events` 中，用来显示当前会议室的所有会议。另外查询每个会议室的会议，并统计在今天该会议室有几项尚未结束的会议，用来显示 Badge 数字。

```php
//打开页面查询各会议室的会议 -- Controller
function index() {
	$dt['title'] = '内部会议室预定平台';
	//get M8
	$data['meetM8'] = $this->M12_mod->get_meeting('m8');
	//get Mc
	$data['meetMc'] = $this->M12_mod->get_meeting('mc');
	//get M3
	$data['meetM3'] = $this->M12_mod->get_meeting('m3');
	//get M8_small
	$data['meetM8_small'] = $this->M12_mod->get_meeting('m8_small');
	//get M12
	$data['meetM12'] = $this->M12_mod->get_meeting('m12');
	//get M12_small
	$data['meetM12_small'] = $this->M12_mod->get_meeting('m12_small');
	
	$this->load->view('header',$dt);
	$this->load->view('m12_vie',$data);
	$this->load->view('main');
}

//查询所有会议 -- Model
function get_meeting($tb){
	$query = $this->db->get($tb);
	return $query->result();
}
```

前端处理查询的数据：

```javascript
var eventData = {
	events : [
		{'id':15022814005937,'start':Date.parse('02/18/2014 02:00PM'),'end':Date.parse('02/18/2014 04:45PM'),'title':'部门会议','person':'王海利'},
		{'id':15022813005284,'start':Date.parse('02/18/2014 01:00PM'),'end':Date.parse('02/18/2014 01:30PM'),'title':'隆达骨质瓷建议书讨论','person':'侯强'},
		{'id':15030209009738,'start':Date.parse('02/19/2014 09:00AM'),'end':Date.parse('02/19/2014 11:45AM'),'title':'综合部内部会议','person':'陈雨薇'},
	]
};
```

初始化 `eventData` 数据之后，`jQuery.weekCalendar` 插件即会展现在日历中。

```javascript
//Badge 的展示

$(function(){
	var today = Date.today().toString('yyMMdd');
	var sidM8 = new Array();
	//后台查询出的数据
	sidM8 = [
				Date.parse('02/18/2014 02:00PM').toString('yyMMdd'),
				Date.parse('02/18/2014 01:00PM').toString('yyMMdd'),
				Date.parse('02/19/2014 09:00AM').toString('yyMMdd'),
			];
			
	var i, j = 0;
	
	for(i=0; i<sidM8.length; i++){
		var id = sidM8[i];
		if(id == today) j++;
	}
	
	if(j == 0){
		$('li.m8 a').attr('title','今日无会议');
		//$('#newM8').hide();
	}else{
		//$('#newM8').text(j);
		$('li.m8 a').append('<span id="newM8">'+j+'</span>');
		$('#newM8').parent('a').attr('title','今日有 '+j+' 项会议');
	}
});
```

###预定功能

点击或者选中一个时间区段即可弹出信息框，填写预定信息。如图：

![{{ page.title }}]({{site.cdn_img}}201402/ci-meeting-system-2.png)

```php
//Ajax 写入 -- Controller

function ajadd(){

	$this->M12_mod->ajadd($_POST['id'],$_POST['st'],$_POST['ed'],$_POST['tp'],$_POST['ps']);
	return $this->db->affected_rows() ? true : false;
	
}

//Ajax 写入 -- Model

function ajadd($id,$st,$ed,$tp,$ps, $table='m12'){

	$data = array(
		'sid' => $id,
		'start' => $st,
		'end' => $ed,
		'topic' => $tp,
		'person' => $ps
	);
	
	return $this->db->insert($table, $data);
}
```

因为不想在提交一个预定后还得刷新页面，所以增删改都使用 `Ajax` 实现，并在操作后有 callback 提示。

```javascript
//Ajax写入 - callback

function ajadd(){
	$.ajax({
		type:"post",
		data:"id="+sid+'&st='+sstart+'&ed='+send+'&tp='+stopic+'&ps='+sperson,
		url:"index.php/m12/ajadd/",
		success:function(msg){
			if(msg){
				callback('会议室预定创建成功！','tip-success');
			}else{
				callback('会议室预定创建失败！','tip-fail');
			}
		}
	});
}
```

###修改功能

修改会议的功能包括拖动修改、拉动修改和自定义修改两种。拖动支持跨天，修改开始时间和结束时间；拉动只能快捷修改会议的结束时间；自定义修改是指点击会议会弹出和新建时一样的框，可以任意修改所有信息。

![{{ page.title }}]({{site.cdn_img}}201402/ci-meeting-system-3.png)

![{{ page.title }}]({{site.cdn_img}}201402/ci-meeting-system-4.png)

![{{ page.title }}]({{site.cdn_img}}201402/ci-meeting-system-5.png)

```php
//Ajax更新（拖动）-- Controller

function ajUpdateDrop(){
	$this->M12_mod->ajUpdateDrop($_POST['id'],$_POST['st'],$_POST['ed']);
	return $this->db->affected_rows() ? true : false;
}

//Ajax更新（拉动）-- Controller

function ajUpdateRe(){
	$this->M12_mod->ajUpdateRe($_POST['id'],$_POST['ed']);
	return $this->db->affected_rows() ? true : false;
}

//Ajax更新（自定义）-- Controller
function ajUpdateClk(){
	$this->M12_mod->ajUpdateClk($_POST['id'],$_POST['st'],$_POST['ed'],$_POST['tp'],$_POST['ps']);
	return $this->db->affected_rows() ? true : false;
}

//Ajax更新（拖动）-- Model

function ajUpdateDrop($id,$st,$ed, $table='m12'){
	$data = array(
		'start' => $st,
		'end' => $ed
	);
	$this->db->where('sid', $id);
	return $this->db->update($table, $data);
}

//Ajax更新（拉动）-- Model

function ajUpdateRe($id,$ed, $table='m12'){
	$data = array(
		'end' => $ed
	);
	$this->db->where('sid', $id);
	return $this->db->update($table, $data);
}

//Ajax更新（自定义）-- Model

function ajUpdateClk($id,$st,$ed,$tp,$ps, $table='m12'){
	$data = array(
		'start' => $st,
		'end' => $ed,
		'topic' => $tp,
		'person' => $ps
	);
	$this->db->where('sid', $id);
	return $this->db->update($table, $data);
}
```

###删除功能

点击会议，弹出信息窗口，点击“删除”按钮，确认之后即可修改。

![{{ page.title }}]({{site.cdn_img}}201402/ci-meeting-system-6.png)

```php
//Ajax删除 -- Controller

function ajdel(){
	$this->M12_mod->ajdel($_POST['id']);
	return $this->db->affected_rows() ? true : false;
}

//Ajax删除 -- Model

function ajdel($id, $table="m12"){
	$this->db->where('sid', $id);
	return $this->db->delete($table);
}
```

修改和删除之后，同样会将状态返回前端，前端根据状态调用 callback 告诉用户是否成功修改或者删除。

##记录操作客户端IP

做好之后，放在公司内部服务器上跑，大家反响都很好，毕竟会议室的使用情况一目了然还是很方便的，也让QQ群安静了不少。因为是不需要注册的，所有同事都可以在上面预定和发布内容，相当于办公室的写字板。开始没有觉得有什么不妥，但没多久一天，很多人在QQ群讨论得很热烈，还有人私信我说去预定平台上看看。原来，有人在周末的预定区域匿名留言，留言内容大概是对另一个同事的人身攻击。由此，我才想到应该完善一下，既然不使用用户权限，那么就记录所有操作人的客户端IP吧，毕竟公司内部IP是和人绑定的，这样一旦有什么需要追查的，直接去看iplog就好了。

页面中获取客户端的IP，然后在增删改时将会议主题、预定人、操作时间、IP等信息发送给 `iplog.php` 处理。

```php
<?php
	//获取IP地址
	if(getenv('HTTP_CLIENT_IP') && strcasecmp(getenv('HTTP_CLIENT_IP'), 'unknown')) {
		$onlineip = getenv('HTTP_CLIENT_IP');
	} elseif(getenv('HTTP_X_FORWARDED_FOR') && strcasecmp(getenv('HTTP_X_FORWARDED_FOR'), 'unknown')) {
		$onlineip = getenv('HTTP_X_FORWARDED_FOR');
	} elseif(getenv('REMOTE_ADDR') && strcasecmp(getenv('REMOTE_ADDR'), 'unknown')) {
		$onlineip = getenv('REMOTE_ADDR');
	} elseif(isset($_SERVER['REMOTE_ADDR']) && $_SERVER['REMOTE_ADDR'] && strcasecmp($_SERVER['REMOTE_ADDR'], 'unknown')) {
		$onlineip = $_SERVER['REMOTE_ADDR'];
	}
?>
<input type="hidden" id="userIP" value="<?php echo $onlineip ?>" /><!--Get User IP-->
```

在


```php
//iplog.php

$ip = $_POST['ip'];
$topic = $_POST['tp'];
$person =  $_POST['ps'];
$filename = 'iplog.txt';  //存入与 iplog.php 同目录的 iplog.txt 文件
date_default_timezone_set("PRC");
$time = date("Y-m-d H:i:s");
$content = "Topic:$topic || IP:$ip || Wtite Person:$person || Time:$time \r\n";
// 确定文件存在并且可写
if (is_writable($filename)) {
    // 我们将使用添加模式打开$filename，
    // 因此，文件指针将会在文件的开头，
    // 那就是当我们使用fwrite()的时候，$content将要写入的地方。
    // a表示写入方式打开，将文件指针指向文件末尾。如果文件不存在则尝试创建之。
    if (!$handle = fopen($filename, 'a')) exit;
    // 将$content写入到我们打开的文件中。
    if (fwrite($handle, $content) === FALSE) exit;
    fclose($handle);
}
```