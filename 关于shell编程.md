+++
date = "2015-06-19T08:43:59+08:00"
menu = "main"
tags = ["code", "linux"]
title = "关于shell编程"

+++

Shell是一个命令解释器，是介于操作系统Kernel与用户之间的绝缘层。今天特拿老师课堂上讲的一些知识点，给大家分享一下，勿喷。

##### 什么时候不用Shell

- 项目由连串的依赖关系各部分分组
- 需要大规模的文件操作
- 需要多维数组的支持
- 需要数据结构的支持
- 需要I/O或Socket接口
- 使用库
- 私人的闭源的应用

##### 激活shell脚本

	bash script
	chmod a+x script
	 ./script(. script)(source script)
	#!/bin/bash

##### 脚本参数

- $1, $2, $3 指向响应的参数
- $# 表示参数的个数
- $* 表示所有参数的整体

##### Shift命令

如果有很多参数时，可以使用shift切换参数。比如说原来输入4个参数，调用shift一次，切掉第一个参数，变成3个参数。

##### 条件执行

1. elif 是else if 的缩减形式

		if [ condition1 ]
		then
			command1
			command2
			command3
		elif [ condition2 ]
		then
			command4
			command5
		else
			default-command
		fi

2. while do 命令的语法

		while [ expression is true ]
		do
			command(s)
		done

3. for
	
		for indentifier in list
		do
			command
		done

##### 转义字符和通配字符

* 转义字符 (metacharacters) 指的是 shell 中有特殊含义的字符，例如 <> | ; ! ? * [] $ \ " ' ` ~ () {}
* 通配字符
	* 通配一个字符 `？`
	* 通配多个字符 `*`

##### 特殊字符

- `\` 转义字符
- `/` 文件名路径分隔符，除法
- `"` `'` 双引号只解释变量名前缀$，后引符，转义，单引号和双引号类似，单引号不能解释$
- `;` 可以在一行中写多个命令
- `;;` 终止 case 选择

		case "$variable" in
			abc) echo "\$variable=abc" ;;
			xyz) echo "\$variable=zyx" ;;
		esac

- `.` 隐藏文件，当前文件
- `,` 连接运算操作
- `:` 空命令
- `!` 取反
- `*` 万能匹配符
- `$` 正则表达式中行结束符
	- `${}` 参数替换
	- `$*` 位置参数
	- `$?` 退出状态
	- `$$` 进程ID变量
- `()` 命名组，在其中的变量赋值之类的很像局部变量
- `{}` 扩展或者代码块

		cat {file1, file2, file3} > combine_line

- `{}/` 路径名
- 重定向符号
	-  `> filename` 重定向脚本的输出到文件中，覆盖文件原来内容
	-  `&> filename` 重定向 `stdout` 和 `stderr` 到文件中
	-  `>&2` 重定向 `command` 的 `stdout` 到 `stderr`
	-  `>> filename` 重定向脚本的输出到文件中，添加到文件尾端，如果没有文件，则创建这个文件
- `<, >` ASCII 比较
- `%` 取模
- `&` 在命令后加上这个表示后台执行
- `~ HOME`
- `condition1 && condition2 condition1` 执行返回 `true` 才执行 `condition2`
- `condition1 || condition2 condition1` 执行返回 `false` 才执行 `condition2` 