+++
date = "2016-08-13T13:44:55+08:00"
menu = "main"
tags = ["vps", "shadowsocks"]
title = "使用搬瓦工 + shadowsocks 搭建 vps"

+++

#### 前言 

身为一名 Android 开发者，应该掌握一些科学访问 Google 的方法。陆续使用过 GoAgent，xx-net等项目，最开始只能通过浏览器访问 Google，更新 SDK 的话使用国内的一些镜像网站。后来又使用过一段时间 host，解决了更新 SDK 的问题。但是 host 的 ip 经常被封，每次使用脚本更新 host 地址实在麻烦，在朋友的推荐下，决定使用搬瓦工 + shadowsocks 搭建一个 vps。

#### 正文

搭建 Vps 我主要参考这两篇博客 [搬瓦工搭建 Shadowsocks 服务器详细图文教程,尽情的翻墙吧!](http://www.banwagong.me/27.html) 和 [搬瓦工VPS面板教程 - 只需15秒钟一键安装Shadowsocks账户](http://www.banwagong.me/27.html)。 这两篇博客介绍的相当详细，所以我在这里也不赘述，主要工作是在在搬瓦工上买一个服务器方案，购买完成后在 KiwiVM Control Panel 上一键安装 shadowsocks，之后下载 shadowsocks 的 win 客户端，配置之后即可运行。后面的一篇博客上面有可供选择的服务方案，我选择的 256M 内存，10G SSD，500G/Monthly 的方案。

在访问搬瓦工官网的时如果遇到了困难，可以使用 GreenVpn 有免费200M流量或者使用 [xx-net](https://github.com/XX-net/XX-Net) 项目。

在 KiwiVM 一键安装 shadowsocks server 有一个缺点就是无法实现多用户，如果想让多台设备使用 shadowsocks server 的话，需要在 KiwiWM 上手动安装 shadowsocks server。 win 用户参考 [CentOS VPS新手教程（1）VPS登录](http://my.oschina.net/zetaplusae/blog/141376) 先下载 putty 并 ssh 到 vps，使用 root 用户登录到 vps 时，vps 是没有初始密码的，需要先在 KiViVM 的控制面板的 root password modification 生成 root 密码，登录之后可以修改密码。 成功连接到 vps 之后可以参考 [Shadowsocks Python版一键安装脚本](https://teddysun.com/342.html) 完成多用户的配置。

#### 更多

* Github 上维护的 google host 项目
	* [https://github.com/txthinking/google-hosts](https://github.com/txthinking/google-hosts)
	* [https://github.com/racaljk/hosts](https://github.com/racaljk/hosts)
* 现在可以得科学上网项目
	* [xx-net](https://github.com/XX-net/XX-Net) 基于 GoAgent，有比较详细的安装使用教程。


