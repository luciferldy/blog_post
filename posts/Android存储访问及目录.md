+++
date = "2015-05-29T08:13:07+08:00"
menu = "main"
tags = ["android"]
title = "Android 存储访问及目录"

+++

[文章的源地址](http://www.cnblogs.com/mengdd/p/3742623.html)

##### Android的外部存储

Android支持外部存储。

外部存储可以通过物理介质如（SD卡），也可以通过将内部存储的一部分封装而成，设备可以有多个外部存储实例

##### 访问外部存储的权限

写操作受权限 WRITE _EXTERNAL _STORAGE 保护

读操作受权限 READ_EXTERNAL_STORAGE 保护

从Android4.4开始，应用可以管理它在外部存储上的特定目录包，而不用获取  WRITE _EXTERNAL _STORAGE

比如说一个包名叫做 com.example.foo的应用，可以自由访问外存上的Android/data/com.example.foo/目录

外部存储对数据提供的保护较少，所以系统不应该存储敏感数据在外部存储上

特别的，配置和log文件应该存储在内部存储中，这样它们可以被有效的保护

对于多用户情况，一般每个用户都会有自己独立的外部存储，应用仅对当前用户的外部存储有访问权限

##### Environment API的目录

getDataDirectory() 用户数据目录

getDownloadCacheDirectory() 下载缓存的内容目录

getExternalStorageDirectory() 主要的外部存储目录

但是这个目录很可能当前不能访问，比如这个目录被用户的PC挂载，或者从设备中移除，或者其他问题发生，你可以通过 getExternalStorageState() 来获取当前状态

任何应用私有的文件都应该被放置 Cotext.getExternalFielDir 返回的目录下，在应用被卸载的时候，系统会清理的就是这个目录

另一些共享文件应该被放置在getExternalStoragePublicDirectory 返回的目录中

写这个路径需要 WRITE_EXTERNAL_STORAGE 权限，读需要 READ_EXTERNAL_STORAGE 权限。写权限包含了读权限

从 KITKAT 开始，如果你的应用知识需要存储一些内部数据，可以考虑使用 getExternalFilesDir() 或者 getExternalCacheDir() 他们不需要获得权限

getExternalStoragePublicDirectory(String type) 这个方法接收一个参数，标目目录所放的文件的类型，传入的参数是Enviroment类中的DIRECTORY_XXX 静态变量，比如 DIRECTORY_CDIM 等

getRootDirectory() 得到Android的根目录

isExternalStorageEmulated() 设备的外存是否用内存模拟的，是则返回true (API 11)

isExternalStorageRemovable() 设备的外存是否是可以卸载的，比如是SD卡，是则返回true (API 9)

##### Context API 中的目录

getExternalFilesDir(String type) 是应用在外部存储上的目录 个 Enviroment 类的 getExternalStoragePublicDirectory(String type) 方法类似，返回包含指定的特定类型文件的子目录

getExternalCacheDir() 是应用在外部存储上的缓存目录

从Android 4.4 这两个方法不需要读写权限，是针对本应用来说的，如果要访问其他应用的相关目录，还需要声明读写权限，Android 4.4 之前的版本要访问的话还是要声明读写权限的，如果没有在 AndroidManifest.xml 中添加权限，这两个都会返回 null

与上面两个方法形成对比的是下面的两个方法
getFileDir(), getCacheDir() 这两个方法得到的是内存上的目录