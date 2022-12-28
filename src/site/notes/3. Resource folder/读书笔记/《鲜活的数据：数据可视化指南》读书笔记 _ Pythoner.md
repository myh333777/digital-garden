---
{"dg-home":true,"dg-publish":true,"permalink":"/3-resource-folder//pythoner/","tags":"gardenEntry","dgPassFrontmatter":true}
---

书名：Visualize This: The FlowingData Guide to Design, Visualization, and Statistics  
译名：《鲜活的数据：数据可视化指南》  
作者：Nathan Yau  
译者：向怡宁  
出版社：人民邮电出版社  
ISBN：978-7-115-29381-7  
页数：296

纸张较厚，正文黑白印刷，部分图放于前面的彩图插页。  
字体间距合适。插图中的中文基本使用了宋体，不太舒服。

本书勘误页（图灵社区）：[http://www.ituring.com.cn/book/819](http://www.ituring.com.cn/book/819 "鲜活的数据勘误")

阅读级别：粗读。  
推荐级别：程序员粗读，设计人员可细读。  
  
《鲜活的数据》这本书很长时间之前就打算看了，一直拖到现在。看完《Python Algorithm》之后，同步开了这本书和《机器学习实战》两本书。这两本书还是有一定相关性的，都是针对数据的。《鲜活的数据》是讲我们怎么把数据可视化，让大家更清晰地理解数据的含义；《机器学习实战》则是对数据进行机器学习，挖掘数据深层的含义，当然这其中实际上也少不了可视化的问题，毕竟一堆数字摆在大家面前不如一幅图像更清晰。

本书首先介绍了处理数据的方法和可视化的工具；然后分别从时间、比例、关系、差异、空间的角度介绍了可视化的方法。

本书中涉及到的可视化工具包括：Python、R、Illustrator、Flash/ActionScript、Protovis等。一般情况下都是使用R做基础创建，再使用Illustrator做一些改善的。其他几种工具使用的不多。Python除了空间一章使用的较多外，还在数据准备和整理阶段有很多使用。

本书中涉及到的可视化图表包括：柱形图、散点图、拟合线、阶梯图、饼图、板块层级图、堆叠面积图、气泡图、密度图、直方图、茎叶图、热点图、切尔诺夫脸谱图、星图、平行坐标图、地图等。

本书假定读者是一个设计人员，而非编程人员，因此对于所有涉及代码地方的讲解都非常细致，生怕读者看不明白，即便已经在前面的章节有过介绍。当然，也对Illustrator和Flex Builder（这货一看就是Eclipse的定制版本啊）的使用讲解的非常细致。因此，**本书非常适合初学者，尤其是不熟悉编程的设计人员来学习**。

本书虽然给出了很多可视化图表的绘制方法，但可能不太适合资深的设计人员寻找灵感。这本书更多的是教你如何创建这些可视化图表，并尽可能的让读者明白你的图表在讲什么。如果你想寻找一些灵感，可能像[《信息之美》](http://www.pythoner.com/73.html "《信息之美》读书笔记")这样的书更适合你。

本书舍弃了原书的彩印，对于色彩不太重要的图片只给出了灰度图，只对一些色彩重要的图片在前面给了彩图插页。实际上对于一本讲可视化的书来讲，还是全彩印刷更好一些，这样可以给读者更深刻的印象。我在看这本书的时候，同时打开着英文版的电子书，来看其中的图。当然，全彩印刷的话，这本书的定价会上涨的很多，出版社应该也是权衡过之后决定的。

此外，本书中图中的中文几乎都是使用了宋体，看起来不那么合适，不如英文原书中的字体看起来那么舒服。其中，还出现了4.2节的图片，最初字体已经和英文版一样了，后来数字突然就变成了宋体的问题。

总之，如果你是一个初学的设计人员，可以仔细阅读并实践书中的例子，作者对每一步（尤其是代码）讲解都很细致，可以很方便的教你入门。如果你是一个资深的设计人员，可能本书的帮助并不大。如果你是一个码农，可以粗读一下，看看自己的数据有什么表现形式，以及有什么工具可以实现。

1.用数据讲故事  
[![](http://www.pythoner.com/wp-content/uploads/2013/07/1.%E7%94%A8%E6%95%B0%E6%8D%AE%E8%AE%B2%E6%95%85%E4%BA%8B-300x125.png)
](http://www.pythoner.com/wp-content/uploads/2013/07/1.%E7%94%A8%E6%95%B0%E6%8D%AE%E8%AE%B2%E6%95%85%E4%BA%8B.png)

2.处理数据  
[![](http://www.pythoner.com/wp-content/uploads/2013/07/2.%E5%A4%84%E7%90%86%E6%95%B0%E6%8D%AE-250x300.png)
](http://www.pythoner.com/wp-content/uploads/2013/07/2.%E5%A4%84%E7%90%86%E6%95%B0%E6%8D%AE.png)

3.选择可视化工具  
[![](http://www.pythoner.com/wp-content/uploads/2013/07/3.%E9%80%89%E6%8B%A9%E5%8F%AF%E8%A7%86%E5%8C%96%E5%B7%A5%E5%85%B7-300x262.png)
](http://www.pythoner.com/wp-content/uploads/2013/07/3.%E9%80%89%E6%8B%A9%E5%8F%AF%E8%A7%86%E5%8C%96%E5%B7%A5%E5%85%B7.png)

4.有关时间趋势的可视化  
[![](http://www.pythoner.com/wp-content/uploads/2013/07/4.%E6%9C%89%E5%85%B3%E6%97%B6%E9%97%B4%E8%B6%8B%E5%8A%BF%E7%9A%84%E5%8F%AF%E8%A7%86%E5%8C%96-300x166.png)
](http://www.pythoner.com/wp-content/uploads/2013/07/4.%E6%9C%89%E5%85%B3%E6%97%B6%E9%97%B4%E8%B6%8B%E5%8A%BF%E7%9A%84%E5%8F%AF%E8%A7%86%E5%8C%96.png)

5.有关比例的可视化  
[![](http://www.pythoner.com/wp-content/uploads/2013/07/5.%E6%9C%89%E5%85%B3%E6%AF%94%E4%BE%8B%E7%9A%84%E5%8F%AF%E8%A7%86%E5%8C%96-300x134.png)
](http://www.pythoner.com/wp-content/uploads/2013/07/5.%E6%9C%89%E5%85%B3%E6%AF%94%E4%BE%8B%E7%9A%84%E5%8F%AF%E8%A7%86%E5%8C%96.png)

6.有关关系的可视化  
[![](http://www.pythoner.com/wp-content/uploads/2013/07/6.%E6%9C%89%E5%85%B3%E5%85%B3%E7%B3%BB%E7%9A%84%E5%8F%AF%E8%A7%86%E5%8C%96-300x129.png)
](http://www.pythoner.com/wp-content/uploads/2013/07/6.%E6%9C%89%E5%85%B3%E5%85%B3%E7%B3%BB%E7%9A%84%E5%8F%AF%E8%A7%86%E5%8C%96.png)

7.发现差异  
[![](http://www.pythoner.com/wp-content/uploads/2013/07/7.%E5%8F%91%E7%8E%B0%E5%B7%AE%E5%BC%82-300x221.png)
](http://www.pythoner.com/wp-content/uploads/2013/07/7.%E5%8F%91%E7%8E%B0%E5%B7%AE%E5%BC%82.png)

8.有关空间关系的可视化  
[![](http://www.pythoner.com/wp-content/uploads/2013/07/8.%E6%9C%89%E5%85%B3%E7%A9%BA%E9%97%B4%E5%85%B3%E7%B3%BB%E7%9A%84%E5%8F%AF%E8%A7%86%E5%8C%96-300x153.png)
](http://www.pythoner.com/wp-content/uploads/2013/07/8.%E6%9C%89%E5%85%B3%E7%A9%BA%E9%97%B4%E5%85%B3%E7%B3%BB%E7%9A%84%E5%8F%AF%E8%A7%86%E5%8C%96.png)

9.有目的的设计  
[![](http://www.pythoner.com/wp-content/uploads/2013/07/9.%E6%9C%89%E7%9B%AE%E7%9A%84%E7%9A%84%E8%AE%BE%E8%AE%A1-300x81.png)
](http://www.pythoner.com/wp-content/uploads/2013/07/9.%E6%9C%89%E7%9B%AE%E7%9A%84%E7%9A%84%E8%AE%BE%E8%AE%A1.png)

思维导图下载：  
[百度云网盘下载（图片版）](http://pan.baidu.com/share/link?shareid=2667452938&uk=3875896295 "鲜活的数据思维导图")  
[百度云网盘下载（mmap）](http://pan.baidu.com/share/link?shareid=2670462491&uk=3875896295 "鲜活的数据思维导图")