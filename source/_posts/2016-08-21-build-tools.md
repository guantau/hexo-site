---
title: 那些构建工具们
date: 2016-08-21 22:29:59
categories: 弄点工具
tags:
---

构建工具指的是能够帮助程序员自动完成程序编译过程的工具，而并非编译器本身。其目标是更方便更快捷地完成整个编译过程。

围绕这个目标，目前已有的构建工具可谓种类繁多。诸如Eclipse、Visual Studio等大型IDE本身含有构建工具，但不纳入下面讨论的范围。

构建工具的总体思路都是利用配置文件描述编译规则，然后通过输入简单的命令完成目标。各个工具在灵活性、复杂度、移植性、执行效率等方面各有千秋。下面列出的工具并没有都尝试过，有些是通过资料所得，仅供参考。

<!-- more -->

## 1. Make
Make是最经典的构建工具，Make的实现版本有不少，最有名的是GNU Make。

通过书写Makefile描述依赖关系和编译命令从而自动完成编译过程。GNU Make强大之处在于可以使用丰富的Shell命令，且它可以通过文件更改时间自动判断是否需要执行编译命令。

Makefile的移植性比较差，并且当源代码结构比较复杂时，手工书写Makefile是一件挺繁琐的事情。

## 2. GNU Autotools
Autotools是Linux系统下大型软件的默认构建工具，通过生成Makefile来完成编译过程。

Autotools造就了经典的源码安装软件三步法（./configure,make,make install）。相比于传统的Make，Autotools扩展了默认编译规则，并添加了依赖检查功能，在编译各种类型目标时都要更方便。

## 3. CMake
CMake可以根据不同的平台、不同的编译器，生成相应的Makefile或者项目文件（如VCproj）。

通过编写CMakeLists.txt，从而控制编译过程。CMake本身就可以看做是一种脚本语言，它不依赖于其它语言。相比GNU Autotools，CMake实现架构要更简单，运行更快。

## 4. Ant
Ant是由Apache发布的，主要用于Java开发，其编译文件格式为xml。

相比Makefile写起来要长一些，但能与IDE较好的兼容，且跨平台的移植性更好。目前也有Ant的变种用来编译C/C++、C#等等。

## 5. Maven
Maven也是Apache开发的用于Java的构建工具，还包含打包及发布功能。

Maven的可定制性很强，这也意味着它比较复杂，学习曲线很陡。使用时，每个工程和模块都需要定义POM（Project Object Model，采用xml描述），声明元数据、插件和库的依赖关系。首次编译时，依赖的插件和库会自动通过网络下载，并形成本地源。Maven还提供了不错的版本控制功能。这些都使得Maven适合作为工业级或大型开源软件的使用。

## 6. SCons
SCons的目标是代替经典的make，它是基于python实现的，且不需要中间步骤来完成编译过程（如生成Makefile等）。SCons的可移植性好，但在大型工程中使用时构建速度比较慢。

## 7. AAP
AAP是由Vim作者开发的基于python的构建工具，可看做是更高级的Make。

AAP同样使用包含依赖关系及编译命令的配置文件，但它提供更高级的功能比如网站维护、软件分发、版本控制等。同Make相比，AAP还做了一些变化，比如通过文件签名而不是修改时间来判断是否需要重新编译，使用python脚本而不是Shell命令避免移植性问题。

## 8. qmake
qmake是由Trolltech为QT套件做的构建工具，最后生成Makefile或相关工程文件。

## 9. KConfig
KConfig是基于Make的构建工具，常用来构建Linux内核以及其它底层工具（如uclibc、busybox等），它的特点是具有可视化配置界面，且构建速度很快。

其它的构建工具还非常多，比如Jam、Makeit、Waf等等，在做相关的项目时，大家可以根据自己的需求进行选择。

--------------
## 参考资料：

1. [跟我一起写Makefile](http://www.chinaunix.net/old_jh/23/408225.html)
2. [Makefile是如何自动生成的](http://blog.chinaunix.net/uid-20544507-id-3494422.html)
3. [CMake入门指南](http://www.cnblogs.com/sinojelly/archive/2010/05/22/1741337.html) 
4. [用Ant实现Java项目的自动构建和部署](http://tech.it168.com/j/2007-11-09/200711091344781.shtml)
5. [Apache Maven入门](http://www.oracle.com/technetwork/cn/community/java/apache-maven-getting-started-1-406235-zhs.html)
6. [Scons VS Other Build Tools](http://www.scons.org/wiki/SconsVsOtherBuildTools)
7. [AAP](http://www.a-a-p.org/) 
8. [Build Manager Tools](http://www.dmoz.org/Computers/Software/Build_Management/Build_Manager_Tools/) 
