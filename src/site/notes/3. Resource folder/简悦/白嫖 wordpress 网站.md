---
{"dg-home":true,"dg-publish":true,"permalink":"/3-resource-folder//wordpress/","tags":"gardenEntry","dgPassFrontmatter":true}
---


首先，什么是 word press

`WordPress` 是使用 `PHP` 语言开发的博客平台，用户可以在支持 `PHP` 和 `MySQL` 数据库的服务器上架设属于自己的网站。也可以把 `WordPress` 当作一个内容管理系统（CMS）来使用。  
`WordPress` 是一款[个人博客系统](https://so.csdn.net/so/search?q=%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2%E7%B3%BB%E7%BB%9F&spm=1001.2101.3001.7020)，并逐步演化成一款内容管理系统软件，它是使用 PHP 语言和 MySQL 数据库开发的，用户可以在支持 `PHP` 和 `MySQL` 数据库的服务器上使用自己的博客。  
`WordPress` 有许多第三方开发的免费模板，安装方式简单易用。不过要做一个自己的模板，则需要你有一定的专业知识。比如你至少要懂的标准通用标记语言下的一个应用 `HTML` 代码、`CSS`、`PHP` 等相关知识。  
`WordPress` 官方支持中文版，同时有爱好者开发的第三方中文语言包，如 `wopus` 中文语言包。`WordPress` 拥有成千上万个各式插件和不计其数的主题模板样式。

或许有些人会考虑 `GitHub Pages`，但由于其无法配置数据库，无法用于 `wordpress` 的搭建。

而既然如此，为什么不考虑 [https://byet.host/free-hosting/news](https://byet.host/free-hosting/news) 呢？

byethost，是美国的一款免费主机，支持 FTP，MySQL，绑定域名（虽然现在我还不会搞，有会的在评论区 @我），完全够安置一款 `wordpress` 了。

## 下载 wordpress

大家可以去[官网](https://cn.wordpress.org/)下载。

## 获取免费主机

打开 [https://byet.host/free-hosting/news](https://byet.host/free-hosting/news)，填入你的信息如：  

![](https://img-blog.csdnimg.cn/img_convert/74a8941eec09e33f4b8b4b0a37876f14.png)

  
点击提交后会跳转到一个页面：  

![](https://img-blog.csdnimg.cn/img_convert/b615943c4b893bfd437c0e1803ea7435.png)

  
接着你会收到一份邮件，按着邮件的指示去做就行了。

打开 [http://cpanel.byethost.com/](http://cpanel.byethost.com/) ，填入邮箱上的信息，登入：  

![](https://img-blog.csdnimg.cn/img_convert/f1f946ee815e067259941ec5d594f493.png)

## 上传

随便下载一个 FTP 客户端，我用的是 SmartFTP Client。

新建一个连接 FTPupload.net 连接，用户名为 cpanle 的用户名，密码同样也是最开始设置的密码

1.  首先从官网上下载 zip 文件，解压。
2.  把其中的 wordpress 文件夹拖放到 htdocs 文件夹，**注意不要在根目录上操作**。
3.  等待传输完成

期间我们可以配置数据库。

![](https://img-blog.csdnimg.cn/img_convert/7e6e6c3aa64328b1946969afe97823dc.png)

点击 “MySQL 数据库”，点击 Create Database 新建数据库，并且几下这个数据库的名字。

## 配置

现在大概上传完毕了，我们来配置一下。

因为一些玄学原因，我就不在虚拟主机上演示了。

打开你的域名 `/wordpress`，你会看到一下页面：

![](https://img-blog.csdnimg.cn/img_convert/eaf2ae0585351a3c8c3fc31b85756bea.png)

凭着直觉走，一定不会有错，点击 现在就开始 按钮。

![](https://img-blog.csdnimg.cn/img_convert/04a19d25d66448958144dd97fe041f0d.png)

填写数据库名（之前创建的数据库），用户名（你的 cpanel 用户名），密码（cpanel 密码）和数据库主机（在 cpanel 的右侧的账号资料里面，通常为 sqlXXX.byethostXX.com）。

然后就按着直觉走啦，绝对不会错。

最后给大家推荐一个主题，Argon，很漂亮，很赞。

### 优点

免费，且容易上手。

### 缺点

速度稍微慢一点，但几乎没影响。  
还有就是貌似绑定不了免费域名 ＞︿＜ 谁会记得在评论区叫我，或者给我[发邮件](mailto:ztrztr2011@163.com)。

## 成功列表

照着这个手册成功的在评论区发布一下评论

1.  [http://ztrblog.byethost31.com/wordpress/](http://ztrblog.byethost31.com/wordpress/)