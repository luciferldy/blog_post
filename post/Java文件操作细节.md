+++
date = "2015-05-26T09:00:49+08:00"
menu = "main"
tags = ["java"]
title = "Java 文件操作细节"

+++

Java中文件操作：

* File file = new File("test.txt") 使用这个这个构造方法实例化一个文件类，file.exist() 用来判断文件或者目录是否存在，这里需要注意的是，Java中并不像C语言里会通过构造方法去创建新文件， file = new File("test.txt") 只是在内存中实例化了一个类，如果不向 test.txt 写入信息或者不适用createNewFile()这个函数，test.txt 是不会存在的。并且File file = new File("sdcard/miplayer/")这样的话就是声明了一个目录，可以使用file.exist()判断目录是否存在，然后使用file.mkdir()创建这个目录。在Android中使用时注意 **权限**