---
layout: post
title: "CentOS 6.5 升级内核到 3.10.28"
date: 2014-01-23 00:02
comments: true
categories: 
---

本文适用于CentOS 6.4, CentOS 6.5，亲测可行，估计也适用于其他Linux发行版。

## 1. 准备工作

### 1.1 下载源码包
Linux内核版本有两种：稳定版和开发版 ，Linux内核版本号由3个数字组成：r.x.y

* r: 主版本号
* x: 次版本号，偶数表示稳定版本；奇数表示开发中版本。
* y: 修订版本号 ， 表示修改的次数


去 <http://www.kernel.org> 首页，可以看到有stable, longterm等版本，longterm是比stable更稳定的版本，会长时间更新，因此我选择 3.10.28，

	wget  https://www.kernel.org/pub/linux/kernel/v3.x/linux-3.10.28.tar.xz

###1.2 解压

	tar -xf linux-3.10.28.tar.xz

###1.3 更新当前系统

	sudo yum update
	sudo yum upgrade

###1.4 安装编译内核所需要的软件包

	sudo yum groupinstall "Development Tools" # 一口气安装编译时所需的一切工具
	sudo yum install ncurses-devel #必须这样才能让 make *config 这个指令正确地执行。
	sudo yum install qt-devel #如果你没有 X 环境，这一条可以不用
	sudo yum install hmaccalc zlib-devel binutils-devel elfutils-libelf-devel #创建 CentOS-6 内核时需要它们


##2 配置文件

###2.1 查看当前系统内核
	
	uname -r
	2.6.32-358.11.1.el6.x86_64

###2.2 将当前系统的配置文件拷贝到当前目录

	cp /boot/config-2.6.32-358.11.1.el6.x86_64 .config


###2.3 使用旧内核配置，并自动接受每个新增选项的默认设置

	sh -c 'yes "" | make oldconfig'

`make oldconfig`会读取当前目录下的`.config`文件，在`.config`文件里没有找到的选项则提示用户填写，然后备份`.config`文件为`.config.old`，并生成新的`.config`文件，参考 <http://stackoverflow.com/questions/4178526/what-does-make-oldconfig-do-exactly-linux-kernel-makefile>


##3 编译

	sudo make -j8 bzImage #生成内核文件
	sudo make -j8 modules #编译模块
	sudo make -j8 modules_install #编译安装模块

要严格按照这个顺序进行编译，**不能合并成一句**，`sudo make -j8 bzImage modules modules_install`。

`-j`后面的数字是线程数，用于加快编译速度，一般的经验是，有多少G内存，就填写那个数字，例如有8G内存，则为`-j8`。


##4 安装

	sudo make install

如果出现了 `ERROR: modinfo: could not find module xxx`，数量少的话，可以忽略。


##5 修改Grub引导顺序

安装完成后，需要修改Grub引导顺序，让新安装的内核作为默认内核。

编辑 `grub.conf`文件，

	sudo vim /etc/grub.conf

数一下刚刚新安装的内核在哪个位置，从0开始，然后设置default为那个数字，一般新安装的内核在第一个位置，所以设置`default=0`。


##6 重启

	sudo reboot

重启后，看一下当前内核版本号，

	uname -r
	3.10.28

成功啦！！


##7 如果失败，则重新循环

如果失败，重新开始的话，要清理上次编译的现场 

	make mrproper #清理上次编译的现场 

然后转到第2步，重新开始。


## 参考资料 
* [How to upgrade the kernel on CentOS](http://xmodulo.com/2013/07/how-to-upgrade-the-kernel-on-centos.html)
* [CentOS 6.4 升级到 3.x Kernel](http://winotes.net/centos-64-upgrade-to-kernel-3x.html)
* [CentOS Linux 升级内核步骤和方法](http://my.oschina.net/qichang/blog/101542)
