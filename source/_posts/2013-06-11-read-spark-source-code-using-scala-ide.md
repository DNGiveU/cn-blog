---
layout: post
title: "使用Scala IDE 阅读spark源码 -- 将sbt项目转化为eclipse项目"
date: 2013-06-11 11:48
comments: true
categories: Spark
tags: [scala, spark]
---
阅读Spark源代码，最简单的方式是下载源码包，解压后用纯文本方式来阅读源码。这样效率不高，可以用sbteclipse这个插件，将sbt项目文件转化为eclipse项目文件，然后导入到Scala IDE，用eclipse来阅读源码，效率大大提高。

**环境**：Windows 7, JDK 1.6

1. 安装 scala。去官网 <http://www.scala-lang.org/> ，下载MSI，安装，按默认设置即可。
2. 安装 sbt。去官网 <http://www.scala-sbt.org/> ， 下载MSI，安装，按默认设置即可。
Linux下可以省略以上两步，spark源码自带了一个sbt，且启动sbt时它会自动下载对应的scala编译器。

3. 安装 Scala IDE。 去官网 <http://scala-ide.org/>，点击"Get the SDK"绿色按钮，下载。这个IDE的好处是，自带了scala编译器，解压即可使用。

4. 下载spark源码。 去官网 <http://spark-project.org/> 下载源码，当前版本是 0.7.2, source package 大约4M左右。解压源码，例如 解压到 d:spark-0.7.0\

5. 添加 sbteclipse 插件依赖。spark已经添加了依赖，这一步什么也不需要做。

	这个插件的作用，就是能够读取sbt的配置文件，生成一个eclipse的工程文件。有了eclipse工程文件，就可以导入到eclipse了。

    spark已经添加了依赖，见 d:\spark-0.7.2\project\plugins.sbt，有一行

    addSbtPlugin("com.typesafe.sbteclipse" % "sbteclipse-plugin" % "2.1.1")


6. 启动cmd，启动sbt。

    > cd  d:\spark-0.7.0  
    sbt

    Linux下则是

    > cd  d:\spark-0.7.0  
	sbt/sbt

    开始下载各种依赖包，需要等待很长时间。

7. **翻@\_@墙。见本文最后一段。**

    <!--more-->

8. 等sbt提升符>出现后，输入eclipse命令，开始生成eclipse工程文件，需要耐心等待一段时间


    > \>eclipse  
    [info] About to create Eclipse project files for your project(s).  
    [info] Successfully created Eclipse project files for project(s):  
    [info] spark-examples    
    [info] spark-streaming   
    [info] spark-repl  
    [info] spark-bagel  
    [info] spark-core  


    这样就生成成功了，去core, bagel, streaming, repl, examples 五个文件夹下，可以看到有一个.project和.classpath文件。从这里也可以看出，spark源码由五个项目组成。

9. 用 Scala IDE 导入这5个工程，选择 d:\spark-0.7.2 文件夹，可以一次性导入5个项目。
项目图标上有红色感叹号，是因为jar包的路径不对，右击某个项目，选择 “Build Path -> Configure Build Path” ，删除所有的jar，点击 "Add external jars"，浏览到 d:\spark-0.7.2\lib_managed\jars，添加所有的jar，这是红色感叹号就消失了。每个项目都如此操作一番。


**注意：第8步在国内是无法成功的，因为一些maven仓库被墙，例如 twitter4j.org这个仓库就被墙了。因此需要翻@\_@墙。**

我平时用goagent翻@\_@墙，不过goagent只能让浏览器翻@\_@墙，如何让goagent变成全局代理呢？即所有http协议都经过goagent。可以用 Proxifier，它可以把goagent变成操作系统全局的http代理。

不过 spark 在访问maven仓库时，用的是https网址，即https协议，虽然goagent可以用来访问https页面，但 goagent 和 Proxifier 使用时，https协议总是链接不通（参考 <https://code.google.com/p/goagent/issues/detail?id=5210>）。

于是我又想到了另一个方法，用SSH翻@\_@墙。去网上找一个免费的ssh，安装 Bitvise SSH Client，然后在"Services"标签页面，勾选"SOCKS/HTTP Proxy Forwarding"，这样来翻@\_@墙，Proxifier  使用  Bitvise SSH Client 提供的代理。

翻@\_@墙成功后，再输入 eclipse，当到达 twitter4j.org 时，会发现 SUCCESS了。耐心等待，最后会成功生成.project文件。

用Scala IDE 导入项目，就可以开始阅读spark 源码了 :)

如果不想折腾，可以下载我已经生成好的项目, [spark-0.7.2.zip](http://pan.baidu.com/share/link?shareid=534521368&uk=2466605404)。解压，启动Scala IDE，选择菜单`File->Import->General->Existing projects into workspace`，浏览到 spark-0.7.2目录，批量导入5个项目。导入后项目图标有红色感叹号，这是因为你的电脑上路径和我的路径不一样，找不到引用的jar了。右击项目，选择`Build Path->Configure Build Path`，选择`Libraries`标签，这时可以看到所有jar都有红叉叉，全选，删除，然后点击`Add External Jars`，浏览到`spark-0.7.2\lib_managed\jars`，把所有jar都导入，导入后红色感叹号就消失了。对每个项目都执行上述操作。

**2013-07-27 更新**：eclipse项目上有红色小叉叉图标，之前一直没解决，今天解决了，主要原因是，**Scala IDE 版本不对！** scala-ide.org 官网最新的的3.0.1只支持scala 2.10，不再支持2.9.3。由于Spark目前使用scala 2.9.3写的，所以我们要下载支持 scala 2.9.3 版，scala ide 3.0.0是支持 2.9.3的，不过首选要下载 eclipse JUNO，不要使用新版的eclipse，例如eclipse Indigo, Kepler都不行。

因此，正确的做法是，先下载 eclipse juno，然后下载 3.0.0 的zip包，解压，然后启动eclipse，点击菜单"help->Install New Software"，浏览到刚刚解压的`site`文件夹，就可以安装scala ide插件了。
