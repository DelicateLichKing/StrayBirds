---
layout: post
title: Eclipse 2 AS 迁徙
category: Android
comments: false
---

##一.乱码
 Eclipse 项目导入 Android Studio 1.4 时出现乱码，Build 提示
Error:(2, 5) 閿欒: 缂栫爜UTF-8鐨勪笉鍙槧灏勫瓧绗?。
原因是源代码是 GBK 格式，Android Studio Encoding 方式是 UTF-8。
解决方式如下：
1、点击 Android Studio 右下角的 File Encoding UTF-8，在出现提示时点击 GBK；
2、在出现的对话框中点击 Reload -> Reload anyway，此时源代码乱码消失，出现中文；
3、点击 Android Studio 右下角的 File Encoding GBK，在出现提示时点击 UTF-8；
4、在出现的对话框中点击 Convert。
Build一切OK。注意顺序不能出错。

##二.导入.so文件
由海康第三方包除了jar还有armebi一大堆.so动态链接库文件。导入方法：copy文件到libs下，在
build.gradle的main里加一句jniLibs.srcDirs=['libs']即可。

##三.jar导入
放入libs文件夹，重写运行工程或add as library。

##另附：cmd ge连接错误 
因为ruby的软件源rubygems.org因为使用亚马逊的云服务，被我天朝屏蔽了，需要更新一下ruby的源，过程如下：
$ gem sources -l (查看当前ruby的源)
$ gem sources --remove https://rubygems.org/
$ gem sources -a https://ruby.taobao.org/
$ gem sources -l
