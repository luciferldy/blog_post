+++
date = "2015-06-17T21:25:00+08:00"
menu = "main"
tags = ["linux"]
title = "Vi 编辑器的基本指令"

+++

Vi编辑器是基于shell的一款高逼格编辑器，用好了可以极大的提高编程的效率，下面简单介绍一下Vi编辑器的一些指令

##### 命令模式下的方向键

k(up), h(left), j(down), l(right)

##### 命令模式下的一些指令

- delete a single character under cursor **x**
- delete a single character left of cursor **X**
- replace a single character **r**
- undo the last change **u**
- repeat the last command **.**
- 显示行号 **:set nu**

##### 切换到编辑模式

- insert text at the begin of line **I**
- insert text before cursor **i**
- append text after cursor **a**
- append text at the end of line **A**
- 返回命令模式 **ESC**

##### 命令模式下的搜索

- /单词 正向查找单词
- ?单词 反向查找单词

##### 替换

- **'s/{old value}/{new value}/'**
- **：%s/old/new/g** %表示所有行，g表示全部替换

##### 剪切，拷贝和粘贴

- cut the whole line **dd**
- copy the whole line **yy**
- cut a word from the current cursor position to the end **dw**
- paste contents of buffers here **p**

##### 退出

- save and exit in command line mode **ZZ**
- save in ex mode **:w**
- forcefully save the file **:w!**
- quit without save **:q**
- force quit without saving **:q!**
- save and exit **:wq**
- save and exit in ex mode shorter **:x**