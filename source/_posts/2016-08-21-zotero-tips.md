---
title: zotero的一些使用经验
date: 2016-08-21 23:18:55
categories: 聊点感想
tags:
    - zotero
    - research
---

在科研活动中，我们需要参考大量的科技文献。科技文献管理一般要经历获取、存储、应用三个过程，不管怎么看，科技文献管理都是一件挺麻烦的事情：
1. 在查找文献时，我们通常会得到很多附加的信息，比如下载的网址、与文献相关的代码、期刊名称等等。这些信息对于我们应用文献有着比较重要的作用，但是它们记录起来可是相当麻烦。
2. 随着工作的开展，各种各样的文件堆积如山，而资源管理器一般只用目录名和文件名进行索引，丢失了大量的元数据信息，此时想要寻找特定内容时将会变得非常困难。
3. 撰写期刊论文、学位论文、项目申请报告的时候，标注参考文献出处是一项考验耐性、细心和记忆力的工作。如何高效地建立参考文献列表是所有研究者都会遇到的实际问题。

好在是有一些工具是能够帮助处理这些麻烦的，比如Endnote、Mendeley、Zotero、NoteExpress、NoteFirst等，这其中我觉得最顺手的就是Zotero。如果你也有遇到上面提到的那些问题，不妨试试它。

<!-- more -->

## Zotero是什么
zotero用于管理参考文献，本质上是FireFox浏览器的插件。它突破了传统管理参考文献的思路：参考文献应该直接从浏览器获取，而不是间接性地由人工下载添加进某个管理软件中，而且参考文献中的各项信息应该由计算机和软件自动识别完成，而不是由人主观识别复制粘贴到管理软件中。

它最初基于FireFox开发，现已有支持Chrome及Safari版的插件，但功能较FireFox版的少很多。现已推出有standalone版本，支持Win、Mac和Linux，功能完整。

已经有人写了一系列的好文章来介绍Zotero了：
* [Zotero（1）：文献管理软件Zotero基础及进阶示范](http://www.yangzhiping.com/tech/zotero1.html) 
* [Zotero（2）：作为知识管理工具的Zotero ](http://www.yangzhiping.com/tech/zotero2.html) 
* [Zotero（3）：平板与社交：再谈研究辅助工具Zotero兼配套APP](http://www.yangzhiping.com/tech/zotero3.html) 
* [Zotero（4）：Zotero之Zotfile插件的使用](http://www.yangzhiping.com/tech/zotero4.html) 
* [Zotero（5）：电子文献管理攻略](http://www.yangzhiping.com/tech/zotero5.html) 
* [Zotero（6）：如何批量下载PDF与组建个性化知识库](http://www.yangzhiping.com/tech/zotero6.html)

## Zotero数据目录
Zotero的管理架构是：**库-分类-条目**。库条目可以同时属于多个分类，条目可以设置多个标签，从而可以从多个角度组织文献。

Zotero数据目录主要包含以下文件和目录：
* zotero.sqlite 数据库文件，保存所有文献的相关信息；
* storage/ 数据目录，保存附件中的各种文件；
* styles/ 样式目录，存放参考文献样式；
* translators/ 抓取器目录，一系列js脚本，用来从各种网站上抓取信息。

## 第三方同步方案
Zotero的云同步功能可以同步笔记和文献pdf等到云端，从而解决了异地、异机、异设备终端的办公问题。该功能非常好，但Zotero只提供100 M的免费空间，该空间显然不够文献的存储使用，当然你可以通过购买增大空间。

我采用的是第三方的同步方案。Zotero官方建议在用第三方同步工具时，安全的办法是仅同步storage目录，因为如果同步数据库文件很容易造成损坏。Zotero的数据文件可以分为两个部分，一个是存放便签、笔记等的数据文件，一个是存放原始pdf文献文件的附件，后者存放在storage文件夹里。实现时很简单，就是将storage拷贝至第三方的同步盘目录中，然后在Zotero数据目录中建立符号链接即可。

## 强制设为英文界面
在安装目录下找到 defaults/preferences/perfs.js，将

    pref("intl.locale.matchOS", true)

改为

    pref("intl.locale.matchOS", false)

即可。

## 查看条目所属分类
一个条目可以指定很多分类，选中某个条目后，按住Ctrl键（Mac下是option键，Linux下是Alt键），包含该条目的分类将会高亮。

## 参考文献样式
可以在 http://www.zotero.org/styles/ 找到常见的参考文献样式，你可以自定义样式，比如在线编辑器 http://steveridout.com/csl/visualEditor/  。

## 常用扩展
### [Zotfile](http://www.columbia.edu/~jpl2136/zotfile.html) 
Zotero默认存储附件时用的是随机符号作为目录名，这可能会让你直接打开数据目录寻找文件时造成一定的困扰。而Zotfile可以按一定格式进行组织，并用链接的方式进行关联。Zotfile默认是用绝对路径的方式关联，这样会导致在不同的机器上找不到文件。我的做法是：
1. 更改Zotero的链接方式为相对路径，基准路径为Zotero数据目录所在位置；
2. 在Zotero数据目录中建立zotfile目录用来存储附件；
3. 更改Zotfile的文件路径为上述建立的zotfile目录位置，子目录按年份存放；
4. 为了实现第三方同步盘进行同步，可以将实际的目录放在同步盘中，只在Zotero数据目录下建立软链接即可。

### [abbreviations-for-zotero](http://citationstylist.org/abbreviations-for-zotero/) 
期刊名简写插件。

### [Better bibtex](https://zotplus.github.io/better-bibtex/) 
可以选择在导出bibtex格式时保留哪些域，去掉哪些域。

## 配合工具
### [Docear](http://www.docear.org)
仅仅用Zotero将文献按照一定的分类、标签组织起来其实还是不够的。要文献真正能派上用场，还需要进一步精细地对文献进行组织。比如文献[5]给出的这几张图：

![1.jpg][a]
![2.jpg][b]
![3.jpg][c]

Docear就是这样一种文献整理工具，并能够将Zotero中存储的大量信息链接起来。

## 参考文档
[1] [Zotero快速入门](https://www.zotero.org/support/zh/quick_start_guide)
[2] [Zotero常用技巧](https://www.zotero.org/support/zh/tips_and_tricks)
[3] [参考文献管理工具zotero的使用经验分享](http://emuch.net/html/201410/7981977.html)
[4] [Zotero文献管理、科研笔记不完全教程](http://blog.sina.com.cn/s/blog_565e747c01014toj.html)
[5] [科研文献资料的高效管理](http://blog.sina.com.cn/s/blog_6daf1c5b0100z8nn.html)
[6] [Zotero同步不足的解决方案](http://www.douban.com/group/topic/48495741/)
[7] [文献管理软件Zotero的一点使用感受](http://www.cnblogs.com/huashiyiqike/p/3265177.html)


  [a]: http://oc7urqs4c.bkt.clouddn.com/docear-1.jpg
  [b]: http://oc7urqs4c.bkt.clouddn.com/docear-2.jpg
  [c]: http://oc7urqs4c.bkt.clouddn.com/docear-3.jpg


