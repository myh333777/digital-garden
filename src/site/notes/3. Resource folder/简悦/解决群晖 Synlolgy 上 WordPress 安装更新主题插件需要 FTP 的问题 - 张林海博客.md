---
{"dg-publish":true,"permalink":"/3-resource-folder//synlolgy-word-press-ftp/","tags":"gardenEntry"}
---

sr-annote { all: unset; }

![](http://cdn.zhanglinhai.com/wp-content/uploads/2018/05/912-0.jpg?imageView2/2/w/840/q/85/format/png)

为什么运行在群晖 NAS 上的 WordPress 升级更新需要输入 FTP 用户名和密码？安装、更新 WordPress 插件也需要 FTP 用户名密码？到底该如何设置目录权限？本文将为你揭晓真正的原因。  
  
作为一个程序员，我在把 PHP Web 开发环境搭在了 NAS 上，这样可以让家里和公司的电脑自动同步，另外开发机也不用安装环境，运行效率更好。但随之而来的是，明明在服务器或开发机上跑的很好的 WordPress，为什么在群晖 NAS 上却发生各种奇怪的问题。

## 故障描述

WordPress 没有写权限，导致无论是升级 / 更新 WordPress、安装 / 更新插件都要求输入 FTP 密码。

![](http://cdn.zhanglinhai.com/wp-content/uploads/2018/05/912-1.png)

WordPress 要求输入 FTP 的用户名密码

就算是把 wwwroot 根目录的权限改成 http 用户可读可写也无效。

![](http://cdn.zhanglinhai.com/wp-content/uploads/2018/05/912-2.png)

群晖 DMS 的文件夹权限配置 UI

## 故障原因

因为某种神奇的原因，更新 WordPress 版本或安装插件时的用户身份是 `nobody`，而 WordPress 目录权限身份是 `http`，因此导致目录权限身份不符引发权限问题。

通常情况下运行 WordPress 的所有目录权限设置为 755 就够了，因为 WordPress 安装 / 更新插件需要 644（4+2）的权限，更新升级 WordPress 自身的话需要 7（4+2+1）权限。  
所有 WordPress 的文件设置为 664 即可，例外的是：wp-config.php 应设为 600，以防其它用户读取。

## 解决方法

找到问题的症结后，就能对症下药了。既然更新文件时使用了奇怪的用户身份，那我们来强制 WordPress 使用 PHP 的读写身份就行了。

## 1. 开启 Web station 的 zip/

1.  打开 `Web Station`——`PHP 设置`——`编辑`配置文件
2.  在`拓展名`里找到 `zip` 并打钩，开启 zip 支持；

![](http://cdn.zhanglinhai.com/wp-content/uploads/2018/05/912-3.jpg)

群晖 DMS 的 PHP 配置文件 UI

## 2. 修改 wp-config.php

1.  在 WordPress 的根目录下找到`wp-config.php`文件；
2.  在`好了！请不要再继续编辑`这行代码的前面新增这行代码：`define('FS_METHOD', 'direct');`，这行代码告诉 WordPress 强制使用 PHP 的读写身份运行文件读写操作。就像下面这样，保存后上传到你的服务器。

```
	define('FS_METHOD', 'direct');

	/* 好了！请不要再继续编辑。请保存本文件。使用愉快！ */

	/** WordPress 目录的绝对路径。 */
	if ( !defined('ABSPATH') )
		define('ABSPATH', dirname(__FILE__) . '/');

	/** 设置 WordPress 变量和包含文件。 */
	require_once(ABSPATH . 'wp-settings.php');



```

## 结束语

其实看得出来：群晖在引导用户从`套件中心`安装群晖打包的 WordPress，而不想让用户在 NAS 上安装 WordPress 官方版。群晖甚至在自己打包的 WordPress 的配置文件 config.php 中添加了禁用 WordPress 官方更新的代码。这意味着如果你选择群晖版 WordPress 就算在后台看到 “更新提示” 也无法升级，只能等群晖的包，然后从`套件中心`来升级。

欢迎转载，但请勿用于任何商业用途，谢谢你！请标注以下信息『 原文出处：[解决群晖 Synlolgy 上 WordPress 安装更新主题插件需要 FTP 的问题 - 张林海博客](http://zhanglinhai.com/archives/912) http://zhanglinhai.com/archives/912 』