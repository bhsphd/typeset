# LaTex FAQ

* Makeatletter and Makeatother

>  On Wed, 2 Jan 2002 at 13:05, K. N. Manoj wrote:

>      What do the commands \makeatletter and \makeatother mean? I
>      searched the four volume `TeX in Practice', but could not find

>   \makeatletter and \makeatother are LaTeX commands and you cant
>   expect that to be explained in TeX in Practice which deals with
>   plain TeX alone.

>      anything. I have seen it being used regularly in LaTeX style
>      files.

>   \makeatletter changes the category code of '@' character to 11 
>   (which is the catcode of ordinarary characters a-z,A-Z). 
>   \makeatother reverts this to its original catcode of 12.   

>   Knuth assigns a category code for each and every character like 0 
>   for escape '\', 1 for begining of a group '{', 2 for end of group 
>   '}', 3 for math shift '$', 4 for alignmet tab `&', 5 for end of 
>   line, 6 for paramter '#', 7 for superscript '^', 8 for subscript 
>   '_', 9 for ignored character, 10 for space, 11 for letters, 13 for 
>   active character '~', 14 for comment character '%', 15 for invalid 
>   character and 12 for characters other than the above.

>   Knuth gives the freedom to change the catcode of any character 
>   anywhere. You can change the catcode of \ to 11 (ie, letter) assign 
>   the catcode 0 to | so that |section becomes a function or control 
>   sequence.

>   You might have noted that an escape character combined with the
>   characters of catcode 11 becomes a control sequence. As such, all
>   the user defined control sequences or macros will be of this nature.

>   This raises the problems of risks of an user defined macro having
>   the same name as that of a macro in a package or even LaTeX kernel.  
>   even. This can break down packages. 

>   In order to circumvent this foreseeable problem, package writers
>   always use the character '@' in their control sequences by changing
>   the cat code of '@' character to 11 which is the catcode of alpha
>   characters. This is accomplished by the command \makeatletter. 

>   At the end of the package, the author will revert the catcode of '@'
>   to 12 with the command \makeatother. So you will find lots of macros
>   with '@' like \@title, etc., which you cant define in a document
>   without changing the catcode. So novice users will not create macros
>   that might clash with kernal macros.

>   Hope you got the idea of this safety scheme. 


All characters in TeX are assigned a "category code" or catcode. There are 16 catcodes in all, some containing just a single character, e.g. `\` is (normally) catcode 0, `{`, catcode 1 etc. Normal characters are catcode 11; this category normally comprises all of the letter characters. The `@` symbol is given the catcode of 12, which means it is *not*treated as a normal letter. The effects of this are that `@` cannot normally be used in user document files as part of a multicharacter macro name. (All other non-letter characters are also forbidden in macro names: for example, `\foo123`, and `\foo?!` are not valid macro names.)

In LaTeX class and package files, however, `@` is treated as a normal letter (catcode 11) and this allows package writers to make macro-names with `@`. The advantage of this is that such macro names are automatically protected from regular users: since they cannnot use `@` as a normal letter, there is no accidental way for a user to override or change a macro that is part of the internal workings of a package.

However, it is sometimes necessary in user documents to *have* access to such package-internal macros, and so the commands `\makeatletter` and `\makeatother` change the catcode of `@` from 12 to 11 and 11 to 12, respectively.

In practical terms, if you need to modify a package internal macro that contains the `@`symbol in its name, you will need to surround your modifications by these commands:

```
\makeatletter % changes the catcode of @ to 11
<your changes here>
\makeatother % changes the catcode of @ back to 12
```

The commands should not be used within `.sty` and `.cls` files themselves as they may conflict with the catcode changes that occurs when package and class files are loaded. For more information on this see [Is it really bad to use \makeatletter and \makeatother in a package or class file?](https://tex.stackexchange.com/q/62583/2693).

* What is the meaning of \fussy, \sloppy, \emergencystretch, \tolerance, \hbadness?

  `\sloppy` is a latex macro that does

  ```
  \def\sloppy{%
    \tolerance 9999%
    \emergencystretch 3em%
    \hfuzz .5\p@
    \vfuzz\hfuzz}

  ```

  `\fussy` sets the values back to the latex defaults

  ```
  \def\fussy{%
    \emergencystretch\z@
    \tolerance 200%
    \hfuzz .1\p@
    \vfuzz\hfuzz}

  ```

  `\tolerance` sets the maximum "badness" that tex is allowed to use while setting the paragraph, that is it inserts breakpoints allowing white space to stretch and penalties to be taken, so long as the badness keeps below this threshold. If it can not do that then you get overfull boxes. So different values produce different typeset result.

  `\emergencystretch` (added at TeX3) is used if TeX can not set the paragraph below the `\tolerance` badness, but rather than make overfull boxes it tries an extra pass "pretending" that every line has an additional `\emergencystretch` of stretchable glue, this allows the overall badness to be kept below 1000 and stops TeX "giving up" and putting all stretch into one line. So `\emergencystretch` does not change the setting of "good" paragraphs, it only changes the setting of paragraphs that would have produced over-full boxes. Note that you get warnings about the *real* badness calculation from TeX even though it retries with `\emergencystretch` the extra stretch is used to control the typesetting but it is not considered as good for the purposes of logging.

  `\hfuzz` does not affect the typesetting in any way but just stops TeX complaining if the box is is only slightly over-full.

  `\hbadness` not used in `\sloppy` but used in the final example below` is similar, it does not affect the typesetting but stops TeX warning about underfull boxes if the badness is below the given amount.

  ------

  The main problem with `\sloppy` is the setting of `\tolerance` to 9999 which is almost infinitely bad. That does not distribute the white space evenly but encourages individual lines of the paragraph to stretch the white space to arbitrarily large amounts. This was an acknowledged deficiency in TeX and why in TeX3 the new parameter `\emergencystretch` was added. Unfortunately when adding `\emergencystretch` to `\sloppy`when definining it for LaTeX2e, we didn't reduce the `\tolerance` setting, but left it at 9999, which is probably good for compatibility with LaTeX2.09, but bad for everything else.

  Note final settings show that often you get a better setting with just setting `\emergencystretch` and not changing `\tolerance` at all, the setting is the same in both cases, but `\hbadness=10000` silences the warnings. Not everyone would *want* the final setting without being warned about it, but in automatic typesetting contexts where changing the input is not a possibility and you never want over-full boxes, this is a possibility.

  ![enter image description here](https://i.stack.imgur.com/yOeO1.png)

  ```
  \documentclass[12pt]{article}
  \usepackage{showframe}
  \begin{document}

  \def\test#1{{#1

  \texttt{\detokenize{#1}}

  Long made-up words, sloppy: 

  abcdefghijklabcdefghijkl abcdefghijklabcdefghijkl abcdefghijklabcdefghijkl

  \textit{abcdefghijklabcdefghijkl abcdefghijklabcdefghijkl abcdefghijklabcdefghijkl abcdefghijklabcdefghijkl}

  Long made-up words, textit, sloppy: 

  \textit{abcdefghijklabcdefghijkl abcdefghijklabcdefghijkl abcdefghijklabcdefghijkl abcdefghijklabcdefghijkl}

  Here is a line with long unbreakable math, sloppy. ${a+b+c+d a+b+c+d}$ ${a+b+c+d a+b+c+d }$

  Extreme unbreakable math, sloppy. ${a+b+c+d+e a+b+c+d+e a+b+c+d+e a+b+c+d+e }$ break ${a+b+c+d+e a+b+c+d+e a+b+c+d+e a+b+c+d+e }$

  \nobreak\bigskip\hrule\filbreak}}

  \test{}
  ```


  \test{\sloppy}

  \test{\setlength\emergencystretch{\hsize}}

  \test{\setlength\emergencystretch{\hsize}\hbadness=10000}

  \end{document}
  ```

  * fancyhdr

    | 在用 LaTeX 排版文章、书籍时，缺省定义了四种页眉页脚的格式：empty没有页眉和页脚plain没有页眉，页脚中部放置页码。headings没有页脚，页眉包含章节的标题和页码。myheadings没有页脚，页眉页码和使用者所定义的信息。article 缺省使用 plain 格式，而 book 则使用headings 格式。 也可用 \pagestyle 命令在你的文档中设定所用的格式，例如在文档中使用 \pagestyle{empty} 则使得此后的页面没有页眉和页脚。 一般情况下，这四种格式基本可满足排版的要求。但在某些情况下，特别是 使用者想定义自己的页眉和页脚格式时，就会遇到很多限制和麻烦。这时， 使用 fancyhdr 宏包可以很容易地达到目的。利用 fancyhdr 宏包提供的命令，可以方便的作到：自定义页眉和页脚。为页眉和页脚加上装饰性的横线。页眉和页脚的宽度可以超过正文文本的宽度。多行的页眉和页脚。奇偶页使用不同格式的页眉和页脚。每章的首页使用不同格式的页眉和页脚。浮动对象页使用不同格式的页眉和页脚。控制页眉和页脚的字体，包括字形，字族，大小写等。现在的大多数 TeX 软件如 MikTeX，fpTeX，teTeX等，都包括 fancyhdr 宏包。如果你的 TeX 软件是较旧的 emTeX 等，则需要自己安装。安装的方法很简单， 只要将 **fancyhdr.sty** 放到 LaTeX 能够找到的目录下就行了。**基本用法：**\documentclass{book}......\usepckage{fancyhdr}\pagestyle{fancy}\begin{document}......\end{document}由 fancyhdr 所定义的页眉和页脚的形式与位置如图所示：![img](http://www.ctex.org/documents/packages/images/fancyhdr1.gif)其中 LeftHeader 和 LeftFooter 为左对齐，CenteredHeader 和 CenteredFooter 为中间对齐，RightHeader 和 RightFooter 为右对齐。上述六个区域的内容和两条装饰线可由用户自己定义。**简单的例子：**\documentclass{article}\usepackage{fancyhdr}\pagestyle{fancy}\lhead{} \chead{} \rhead{\bfseries The performance of new graduates} \lfoot{From: K. Grant} \cfoot{To: Dean A. Smith}\rfoot{\thepage} \renewcommand{\headrulewidth}{0.4pt} \renewcommand{\footrulewidth}{0.4pt}......\begin{document}......\end{document}结果如图所示：![img](http://www.ctex.org/documents/packages/images/fancyhdr-ex1.gif)上面例子中，\thepage 给出了当前页的页码，而 \bfseries 则使 LaTeX 使用粗体字排版页眉。如果想在文档中改用其它形式，比如在第一页不要页眉和页脚，则可在 \begin{document} 和 \maketitle 后使用命令\thispagestyle{empty}缺省情况下，\maketitle 命令会自动设置其所在页的格式为 plain 。因此，如果你想在该页使用 fancy 格式的话，应该在 \maketile 后面使用命令 \thispagestyle{fancy}。下面是一个双面页版式下的例子：\documentclass{book} \usepackage{fancyhdr}\fancyhead{} % clear all fields \fancyhead[RO,LE]{\bfseries The performance of new graduates} \fancyfoot[LE,RO]{\thepage} \fancyfoot[LO,CE]{From: K. Grant} \fancyfoot[CO,RE]{To: Dean A. Smith} \renewcommand{\headrulewidth}{0.4pt} \renewcommand{\footrulewidth}{0.4pt}\begin{document}......\end{document}这里方括号中字母代表的意义为：E偶数页O奇数页L页眉或页脚的左边部分C页眉或页脚的中间部分R页眉或页脚的右边部分H页眉F页脚结果如图所示：![img](http://www.ctex.org/documents/packages/images/fancyhdr-ex2.gif)在配合 CJK 排版中文文档时，要把带有中文的页眉和页脚的定义用 \begin{CJK}{...}{...} 和 \end{CJK} 括起来。最简单的办法是将其放到 \begin{document} 和 \begin{CJK}{...}{...} 之后。如：\documentclass{book} \usepackage{CJK}\usepackage{fancyhdr}......\begin{document}\begin{CJK}{GBK}{song}\pagestyle{fancy}\fancyhead{} % clear all fields \fancyhead[RO,LE]{\CJKfamily{hei} \bfseries \LaTeX{} 排版系统} \fancyhead[LO,RE]{\CJKfamily{hei>} \bfseries \leftmark}\fancyfoot[LE,RO]{\thepage} \fancyfoot[LO,RE]{\CJKfamily{kai} 公元二零零零年七月} \renewcommand{\headrulewidth}{0.4pt} \renewcommand{\footrulewidth}{0.4pt} ......\end{CJK}\end{document} |
    | ---------------------------------------- |
    | ![img](http://www.ctex.org/documents/packages/images/fancyhdr-ex3.gif) |
  ```