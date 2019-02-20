---
title: 不用装虚拟机了，Win10自带了Linux！
date: 2017-9-29 21:53:04
tags: [经验,Linux,Windows]
toc: true
---

## 前言
相信很多同学都受到过这样的困扰。在做开发的时候，如果你是安装的双系统，那么在系统切换的时候需要重启十分繁琐，特别是涉及到数据跨系统使用的时候十分让人头疼；如果用的虚拟机，那么一般的电脑也会因为性能问题出现难以忍受的卡顿。
<!-- more -->
那么现在，微软成功的为我们解决了这个问题。
现在Windows10已经有了内置的Linux，只需要在CMD下敲一个简单的命令就可以执行Linux下的的Shell命令了。


## 必须环境
* Windows10
由于这个功能不过是前段时间才公布的，所以实际上只有Windows10升级到一周年正式版以上才能够使用，如果是最新换成Windows10的可是要先更新才能用哦。

## 操作步骤

1. 右击开始按钮  -->   程序与功能  -->  启动或关闭Windows功能
将滚动条拖到最下
![Windows程序与功能](http://zjko-blog-img.oss-cn-beijing.aliyuncs.com/914FNBFO%40CIBBV9%290UWY5%28K.png?x-oss-process=style/blog-img)

2. 进入设置  -->  针对开发人员  -->  开发人员模式
选中之后需要等待一会儿，圆圈变成实心才算成功
![打开开发者模式](http://zjko-blog-img.oss-cn-beijing.aliyuncs.com/ARZ6A%7B%602%24%5BN4M9YJG%40HNXZC.png?x-oss-process=style/blog-img)

3. 打开CMD，输入bash即可自动下载（期间需要确认输入y）。由于默认数据源在国内不太稳定，所以可能需要多试几次。如果实在无法解决，可以参考手动更换数据源。


4. 安装完成之后，打开CMD，输入bash
Duang~
![Bash界面](http://zjko-blog-img.oss-cn-beijing.aliyuncs.com/B_37%60OAQZ%28%24016%5D4%7DF%7E0R36.png?x-oss-process=style/blog-img)

> 注意：
> 	默认的Ubuntu版本不包含图形界面，如果有需要请自行百度Xming。
>   默认不包含GCC环境，若需要请自行安装

