---
{"dg-publish":true,"permalink":"/3-resource-folder//nas-word-press/","tags":"gardenEntry"}
---

![](https://pic3.zhimg.com/v2-470cea9a48961c32a41aef46a8fa371a_b.jpg)

## 前言

由于群晖 NAS 套件中心提供的 WordPress 套件版本更新不如官方及时, 本文将详解如何在群晖 NAS 中手动安装配置官方 WordPress 博客.

见过尝试，总体比较简单，关键是一个读取权限的问题。

WordPress 是一个以 PHP 和 MySQL 为平台的自由开源的博客软件和内容管理系统。WordPress 具有插件架构和模板系统。截至 2018 年 4 月，排名前 1000 万的网站超过 30.6% 使用 WordPress。WordPress 是最受欢迎的网站内容管理系统。WordPress 是当前因特网上最流行的博客系统。WordPress 在最着名的网络发布阶段中脱颖而出。如今，它被使用在超过 7000 万个站点上。

本文为 landiao 结合网络文章根据实践, 测试, 整理发布. 如需转载请联系作者获得授权, 并注明转载地址.

## 一、环境套件安装

群晖套件中心安装以下套件

*   Web Station
*   Apache HTTP Server 2.4
*   PHP 7.3
*   MariaDB 10
*   phpMyAdmin

网络文章都是写的 PHP 7.2，这这里用的 7.3。完全没有区别可自由选择。图片就不更新了

![](https://pic2.zhimg.com/v2-720014bfad12eeb3ba87fdeea9736579_b.jpg)

二、创建数据库

浏览器输入 [http://192.168.x.x/phpmyadmin](http://192.168.x.x/phpmyadmin)

也可以从群晖桌面图标进入

账号 root

密码为安装 MariaDB 10 套件时设置的密码

选择服务器 - MariaDB 10

192.168.x.x 的 IP 为群晖的内网 IP

![](https://pic3.zhimg.com/v2-39181a13d2b619d78cbc0e546249fd2e_b.jpg)

1.  新建
2.  填写数据库名称
3.  创建

数据库名称可自行设置, 建议使用 wordpress 等方便识别.

![](https://pic4.zhimg.com/v2-86d281efb488826828f8b6501c952bb3_b.jpg)

## 三、下载并安装 WordPress 主程序

1.  访问 `[WordPress官方网站](https://wordpress.org/)`, 下载最新版.
2.  在群晖 Web 文件夹下创建博客主目录, 命名为 wordpress(或其他任意文件名).
3.  将下载并解压的文件夹内的所有文件及文件夹拷贝至博客主目录.

![](https://pic1.zhimg.com/v2-427f1027a95cba34ddd2fc3381ee39f8_b.jpg)

1.  设置 wordpress 文件夹权限为 http（重要）
2.  应用到这个文件夹, 子文件夹和子文件
3.  确定

![](https://pic4.zhimg.com/v2-1091dda3e64ad0f8cbe5ccc338c8a2a7_b.jpg)

## 四、配置 PHP

1.  打开 Web Station
2.  PHP 设置, 选中 Default Profile, 版本为 7.3, 编辑.
3.  扩展名勾选全选, 安装扩展.

## 五、配置 Web 站点

1.  打开 Web Station
2.  虚拟主机
3.  新增
4.  填写你的域名
5.  文档根目录选择刚才在 Web 下创建的文件夹
6.  HTTP 后端服务器选择 Apache HTTP Server 2.4
7.  PHP 选择 Default Profile(PHP 7.3)

![](https://pic1.zhimg.com/v2-e3d07c07a1efc22be5e2b30acfa47de8_b.jpg)

## 六、配置 WordPress 博客

在浏览器中输入你的域名开始配置 WordPress 博客

![](https://pic3.zhimg.com/v2-d6c836bb490dd5153e871ac0a28be1de_b.jpg)

1.  数据库名称填写刚才 phpMyAdmin 创建的数据库
2.  用户名 root
3.  密码填写群晖安装 MariaDB 10 时设置的密码
4.  数据库地址填写 localhost:/run/mysqld/mysqld10.sock
5.  提交

数据库地址如下图所示在群晖 NAS 中打开 MariaDB 10 所显示的路径. 如诺不行, 可尝试以下几种:  
localhost:3307/run/mysqld/mysqld10.sock  
127.0.0.1:/run/mysqld/mysqld10.sock  
127.0.0.1:3307/run/mysqld/mysqld10.sock

如果安装的数据库套件为 MariaDB 5, 那么此处数据库地址填写 localhost 即可.

![](https://pic3.zhimg.com/v2-cf6ac7c691be0cb45e483ae37692df32_b.jpg)

![](https://pic2.zhimg.com/v2-635c0cebf6e9477cfc50e4fae57fe41d_b.jpg)

  
现在安装  

![](https://pic3.zhimg.com/v2-3f8631841d772c397fdec538a50b8ffa_b.jpg)

1.  填写站点标题
2.  填写用户名
3.  填写密码
4.  填写电子邮件
5.  安装 WordPress；

![](https://pic1.zhimg.com/v2-f0615bdd33736d02a5293d69caa4060c_b.jpg)

  
安装成功  

![](https://pic4.zhimg.com/v2-3fba155e26b195718f6c31e2b77eb80f_b.jpg)

  
登录  

![](https://pic1.zhimg.com/v2-2ff60e0e3cecc848709e4aecb0efc1f8_b.jpg)

  
登录成功

![](https://pic3.zhimg.com/v2-d0ae80535217536e7774499c348280ba_b.jpg)

  
访问你的 WordPress 博客吧!

![](https://pic1.zhimg.com/v2-e6ca215ea87c7efa3bca6aa2ca7782e8_b.jpg)

**关键来了**

更新或者主题操作报错。

主要是权限的问题。

## **故障原因**

因为某种神奇的原因，更新 WordPress 版本或安装插件时的用户身份是 `nobody`，而 WordPress 目录权限身份是 `http`，因此导致目录权限身份不符引发权限问题。

![](https://pic2.zhimg.com/v2-e1bddfad14374b33028eb908c3ce3c39_r.jpg)

通常情况下运行 WordPress 的所有目录权限设置为 755 就够了，因为 WordPress 安装 / 更新插件需要 644（4+2）的权限，更新升级 WordPress 自身的话需要 7（4+2+1）权限。  
所有 WordPress 的文件设置为 664 即可，例外的是：wp-config.php 应设为 600，以防其它用户读取。

**解决方法**

找到问题的症结后，就能对症下药了。既然更新文件时使用了奇怪的用户身份，那我们来强制 WordPress 使用 PHP 的读写身份就行了。

## **修改 wp-config.php**

1.  在 WordPress 的根目录下找到`wp-config.php`文件；

在好了！请不要再继续编辑这行代码的前面新增这行代码：

```
define('FS_METHOD', 'direct');，

```

这行代码告诉 WordPress 强制使用 PHP 的读写身份运行文件读写操作。就像下面这样，保存后上传到你的服务器。

1.  在最后添加一段代码：

```
define(“FS_METHOD”, “direct”);
define(“FS_CHMOD_DIR”, 0777);
define(“FS_CHMOD_FILE”, 0777);

```

## 结语

这个教程主要是整理完善。现在大家都在用 docker 部署，但是个人觉得这样也挺好的。

引用了下列文章

[群晖 NAS 手动安装配置官方 WordPress 博客教程 - HDTVMV](http://www.hdtvmv.com/9.html)

[解决群晖 Synlolgy 上 WordPress 安装更新主题插件需要 FTP 的问题 | 张林海博客 (zhanglinhai.com)](http://zhanglinhai.com/archives/912)