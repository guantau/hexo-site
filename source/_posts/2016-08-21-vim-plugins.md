---
title: 武装你的vim
date: 2016-08-21 23:14:37
categories: 弄点工具
tags: vim
---

vim自带的编辑、快速移动等等强大的功能，但它还不够强大，它还可以有更多的武器。

## 1、vim插件相关：
（1）vundle：vim插件管理必备。
（2）genutils：提供了一些书写vim插件有用的函数。
（3）L9：同样提供了许多书写vim插件有用的函数。

<!-- more -->

## 2、浏览类：
（1）minibufexpl.vim：以不同的缓冲区显示不同的文件。
（2）Tabbar：对文件中出现的类、函数、变量等提供概要浏览
（3）taglist：类似Tagbar。
（4）The-NERD-Tree：对目录中的文件进行浏览。

## 3、查找类
（1）文件快速查找：
Command-T：根据部分文件名快速找到所需要的文件，需要vim编译时支持ruby。
（2）文件切换：
FSwitch：在头文件和源文件之间快速切换。
（3）类/函数/变量定义查找（类C语言）：
cscope.vim：需要有cscope可执行程序，创建数据库后可以实现查找。

## 4、补全类：
（1）neocomplcache：强大的补全功能，集所有的补全方式于一身。
（2）xptemplate：强大的snippet功能，帮助快速写代码。
（3）omnicppcomplete：为C/C++提供补全，在./->/::等等后面自动补全，需要ctags支持。
（4）pythoncomplete：为python提供补全功能。

## 5、括号配对
Auto-Pairs：自动补齐成对的符号，提供了flymode来跳出多层括号。

## 6、注释类
（1）The-NERD-Commenter：快速注释功能
（2）自动形成doxygen风格的注释：
DoxygenToolkit.vim：为文件、类、函数自动形成doxygen风格的注释。

## 7、可视化显示内容：
（1）Gundo：将undo内容用树形可读方式展示。
（2）Marks-Browser：将marks标记用可视化方式展示。
（3）Tasklist：对文件中标记TODO的内容可视化展示。
（4）YankRing.vim：可视化显示最近删除的内容。。

## 8、各种语言相关：
（1）PHP：
phpfolding.vim：为php提供代码折叠功能。
（2）python：
pyhon-mode-klen：为python提供了许多实用功能，包括了pylint（pep8规范性检查、语法错误检查）、rope（部分功能）、pydoc（帮助查找）
ropevim：为python提供了定义查找、帮助查找、代码重构等功能。
（3）HTML：
sparkup：为html代码书写提供一些实用功能。
（4）LaTeX：
vim-latex：书写latex必备插件。

## 9、版本控制：
vim-fugitive：提供Git相关功能

## 10、调试：
vimgdb：与gdb接口可实现可视化调试功能。

## 11、缩进：
vim-pasta：粘贴时自动根据语法进行缩进。
vim-indent-guides：可视化显示缩进级别。

## 12、状态栏：
vim-powerline：可以各种设置vim状态栏。
