---
title: vim-latex-suite使用手册
date: 2016-08-21 22:33:47
categories: 弄点工具
tags:
  - vim
  - latex
---

## 导语
> 快捷键是提高效率的不二法则，毕竟十个手指头比两个手指头要快得多。
使用vim-latex-suite的关键亦是如此。

说明：

* `||`中为在普通模式下输入的命令；
* `<>`中为按键，C表示Ctrl，A表示Alt，S表示Shift；

<!-- more -->

## 一、模板

### 1. 使用方法
模板存放在$VIM/ftplugin/latex-suite/templates/中，使用命令`|:TTemplate|`或者从菜单中可以调出可用的模板。

### 2. 定制方法
在$VIM/ftplugin/latex-suite/templates/中建立相应的文件即可。

## 二、包
### 1. 使用方法
宏包存放在$VIM/ftplugin/latex-suite/packages/中，从菜单中或者使用按键`<F5>`、命令`|:TPackage|`都可以调出可用宏包。

### 2. 定制方法
在$VIM/ftplugin/latex-suite/packages/中建立相应的文件即可，可参考exmpl。

## 三、环境

### 1. 使用方法

#### （1）插入
* 方法1：`<F5>`，读取当前行的单词并形成环境，如果是空行，则给出环境列表；
* 方法2：`<S-F1>`-`<S-F4>`，每一个对应一个自定义的环境；
* 方法3：使3字母序列`Exx`，第一个字母E代表Environment，后两个字母是环境名的简写，比如EFI插入figure环境。

#### （2）包围
* 方法1：选中需要放入环境中的内容，按`<F5>`；
* 方法2：选中需要放入环境中的内容，按3字母序列，这里的3字母序列和插入中不同的在于首字母需要改为`<,>`，后两个字母保持不变（小写即可）；

#### （3）修改
* 方法1：选中需要修改环境名的内容，然后按`<S-F5>`，多重环境时先改变最内层环境；

### 2.定制方法
设置变量`g:Tex_Env_name`即可，其中'name'是环境名，例如

    let g:Tex_Env_frame = "\\begin{frame}\<cr>\\frametitle{<+title+>}\<cr><++>\<cr>\\end{frame}<++>"

有些带标签的环境可设置变量`g:Tex_EnvLabelprefix_name`，例如

    figure, table, theorem, definition,lemma, proposition, corollary, assumption, remark, equation, eqnarray, align, multline

默认给出的环境列表由变量`g:Tex_PromptedEnvironments`设置，默认值为

    'eqnarray\*,eqnarray, equation,equation\*,\[,$$,align,align\*'

`<S-F1>`-`<S-F4>`对应的环境名由`g:Tex_HotKeyMappings`设置，默认值为

    'eqnarray*,eqnarray,bmatrix'


## 四、命令

### 1. 使用方法

#### （1）插入

* 方法1：`<F7>`，提取当前光标所在单词构成命令，如果是空单词，则给出命令列表；

#### （2）包围
* 方法1：选中需要放入命令的内容，按`<F7>`；

#### （3）修改
* 方法1：选中需要修改命令名的内容，按`<S-F7>`。

### 2. 定制方法
设置变量`g:Tex_Com_name`即可，其中'name'是变量名。

默认给出的命令列表由变量`g:Tex_PromptedCommands`控制，默认值为

	'footnote,cite,pageref,label'

## 五、参考文献

### 1. 使用方法
共提供四种插入模式：`BBB`、`BBL`、`BBH`和`BBX`。
它们的插入方式是一致的，输入后会提示需要插入的文献类型。

* `BBB`仅插入该种文献所需的最少字段；
* `BBL`插入该种文献常用的字段；
* `BBH`插入一些更多的字段；
* `BBX`则插入所有的字段。

### 2. 定制方法
如果需要定制不同插入模式下的字段，那么需要修改全局变量`g:Bib_{type}_options`

该变量在文件$VIM/ftplugin/bib.vim中定义，{type}是文献类型，比如'article'、'book'等。
变量取值如下表所示：

字符  |  对应的字段
-----|------------
w    |       address
a    |        author
b    |       booktitle
c    |        chapter
d    |       edition
e    |       editor
h    |       howpublished
i    |       institution
k    |       isbn
j    |       journal
m    |       month
z    |       note
n    |       number
o    |       organization
p    |       pages
q    |       publisher
r    |       school
s    |       series
t    |       title
u    |       type
v    |       volume
y    |       year

比如，默认条件下使用`BBB`插入'article'

    @ARTICLE{<+key+>,
        author = {<++>},
        title = {<++>},
        journal = {<++>},
        year = {<++>},
        otherinfo = {<++>}
    }<++>

当定义`g:Bib_article_options`为'mnp'，则使用`BBB`插入'article'为

    @ARTICLE{<+key+>,
        author = {<++>},
        title = {<++>},
        journal = {<++>},
        year = {<++>},
        month = {<++>},
        number = {<++>},
        pages = {<++>},
        otherinfo = {<++>}
    }<++>

如果还有一些上面没有列出来的字段需要插入，则需要定义全局变量
`g:Bib_article_extrafields`

比如定义

    let g:Bib_article_extrafields = "crossref\nabstract"

则'article'的模板会多出两个字段

    crossref = {<++>},
    abstract = {<++>},


## 六、编译及查看
使用按键`\ll`开始编译。
变量`g:Tex_CompileRule_<format>`设置编译规则，<format>是"pdf"、"dvi"等。
设置编译依赖，比如

    .tex -> .dvi -> .ps -> .pdf

可以设置为

    let g:Tex_FormatDependency_pdf = 'dvi,ps,pdf'

同时需要设定编译规则

    let g:Tex_CompileRule_dvi = 'latex --interaction=nonstopmode $*'
    let g:Tex_CompileRule_ps = 'dvips -Ppdf -o $*.ps $*.dvi'
    let g:Tex_CompileRule_pdf = 'ps2pdf $*.ps'

只编译部分文件，选择模式下选择一部分内容，然后使用\ll编译这一部分内容，用\lv来查看结果。对应的命令是`|:TPartComp|`和`|:TPartView|`。

查看使用`\lv`。规则使用变量`g:Tex_ViewRule_<format>`来定义。

前向搜索使用`\ls`。在Mac上，需要设置`g:Tex_TreatMacViewerAsUNIX`为1

反向搜索需要设置查看器与vim的沟通方式，比如

    "C:\Program Files\vim\vim61\gvim" -c ":RemoteOpen +%l %f"

## 七、折叠
Latex-Suite用插件SyntaxFolds.vim来进行语法折叠。
折叠是手动的，新写的内容需要按`<F6>`或`\rf`来开启折叠。

有一系列变量用来控制折叠
`g:Tex_FoldedSections`控制哪些节需要折叠，默认值为

    part,chapter,section,subsection,subsubsection,paragraph

`g:Tex_FoldedEnvironments`控制哪些环境需要折叠，默认值为

    verbatim,comment,eq,gather,
    align,figure,table,thebibliography,
    keywords,abstract,titlepage

`g:Tex_FoldedCommands`控制哪些命令需要折叠，默认值为空。
`g:Tex_FoldedMisc`控制一些其他需要折叠的内容，默认值为

    item,preamble,<<<

## 八、多文件工程
假设有如下工程结构

    thesis/
        main.tex
        abstract.tex
        intro/
            intro.tex
            figures/
                fig1.eps
                fig2.eps
        chapter1/
            chap1.tex
            figures/
                fig1.eps
        conclusion/
            conclusion.tex
            figures/

main.tex文件如下

    % file: main.tex
    \documentclass{report}
    \begin{document}

    \input{abstract.tex}
    \input{intro/intro.tex}
    \input{chapter1/chap1.tex}
    \input{conclusion/conclusion.tex}

    \end{document}

只需要创建一个空文件main.tex.latexmain就可以表明main.tex是主文件。


## 九、常用快捷键

### 1. 章节
可使用3字母序列`Sxx`进行插入和修改。

### 2. 字体
可使用3字母序列`Fxx`进行插入和修改。

### 3. 希腊字母
\`a至\`z分别代表\alpha到\zeta，大写情况也类似（但不支持所有大写希腊字母）。

### 4. 智能按键
`...`在数学模式外是\ldots，在数学模式中是\cdots

### 5. 补全
用`<F9>`可以进行各种类型的补全，包括引用补全（\ref、\eqref、\cite）、文件名补全、命令参数补全。通常需要设置

	set grepprg=grep\ -nH\ $*

### 6. Auc-Tex中的一些快捷键
数学环境中：

快捷键 | 对应的命令
------|------------
\`^   |   \Hat{<++>}<++>
\`_   |   \bar{<++>}<++>
\`6   |   \partial
\`8   |   \infty
\`/   |   \frac{<++>}{<++>}<++>
\`%   |   \frac{<++>}{<++>}<++>
\`@   |   \circ
\`0   |   ^\circ
\`=   |   \equiv
\`\   |   \setminus
\`.   |   \cdot
\`*   |   \times
\`&   |   \wedge
\`-   |   \bigcap
\`+   |   \bigcup
\`(   |   \subset
\`)   |   \supset
\`<   |   \le
\`>   |   \ge
\`,   |   \nonumber
\`~   |   \tilde{<++>}<++>
\`;   |   \dot{<++>}<++>
\`:   |   \ddot{<++>}<++>
\`2   |   \sqrt{<++>}<++>
\`&#124;   |   \Big&#124;
\`I   |   \int_{<++>}^{<++>}<++>

visual模式下：

快捷键 | 对应的命令
------|----------
\`(   | \left( \right)
\`[   | \left[ \right]
\`{   | \left\\{ \right\\}
\`$   | 普通选择 $$，行选择 \\[ \\]


### 7. Alt相关
默认条件下Alt键是菜单栏的热键，如果有冲突则需要设置

	set winaltkeys=no

* `<Alt-L>`
在插入模式下，根据当前光标前的字符，插入不同的命令

光标之前的字符 | 对应的命令
------------|-----------
(           | \left( <++> \right)
[           | \left[ <++> \right]
&#124;      | \left&#124; <++> \right&#124;
{           | \left\{ <++> \right\}
<           | \langle <++> \rangle
q           | \lefteqn{<++>}<++>

如果当前光标前面没有任何字符，则插入\label{<++>}。

* `<Alt-B>`
插入模式中将前面的字符包含在命令\mathbf{}中。

* `<Alt-C>`
在插入模式下，
如果前面的字符是字母或数字，则变成大写并包含在命令\mathcal{}中；
其它情况下插入\cite{}。
在选择模式下，将选择的字符包含在\mathcal{}中。

* `<Alt-I>`
根据不同的环境插入\item

环境名           |  样式
----------------|--------
itemize         |   \item
enumerate       |   \item
theindex        |   \item
thebibliography |   \item[<+biblabel+>]{<+bibkey+>} <++>
description     |   \item[<+label+>] <++>

可以通过变量`g:TeX_ItemStyle_environment`进行修改。


## 十、宏定制方法

### 1. 宏文件
在$VIM/ftplugin/latex-suite/macros/中，每一个文件就是一个宏。
用命令`|:TMacro|`或从菜单上可以选择使用哪个宏。
可以用`|:TMacroNew|`、`|:TMacroEdit|`、`|:TMacroDelete|`进行操作。

### 2. IMAP
可以通过IMAP()定制宏，其语法为

    call IMAP (lhs, rhs, ft [, phs, phe])
    lhs 缩写
    rhs 展开的代码
    ft 适用的文件类型
    phs,phe 用来表示插入点的起始和终止符号，默认为<+和+>

例如

	:call IMAP('EFE', "\\begin{figure}\<CR><++>\\end{figure}<++>", 'tex')

复杂一点的情况

	call IMAP('FOO', "\<C-r>=AskVimFunc()\<CR>", 'vim')
	" Askvimfunc: Asks For Function Name And Sets Up Template
	" Description:
	function! AskVimFunc()
	    let name = input('Name of the function : ')
	    if name == ''
	        let name = "<+Function Name+>"
	    end
	    let islocal = input('Is this function scriptlocal ? [y]/n : ', 'y')
	    if islocal == 'y'
	        let sidstr = '<SID>'
	    else
	        let sidstr = ''
	    endif
	    return IMAP_PutTextWithMovement(
	        \ "\" ".name.": <+short description+> \<cr>" .
	        \ "Description: <+long description+>\<cr>" .
	        \ "\<C-u>function! ".name."(<+arguments+>)<++>\<cr>" .
	        \       "<+function body+>\<cr>" .
	        \ "endfunction \" "
	        \ )
	endfunction
