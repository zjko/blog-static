---
title: 使用hexo在GitHub上搭建个人博客网站
date: 2017-08-22 17:26:27
tags: 经验
toc: true
---

## 前言


为什么要使用GitHub Pages搭建博客？

1. GitHub使用免费，空间充足
2. 管理安全方便，基于GitHub版本控制
3. 定制化程度高，与其他博客网站相比没有什么约束
4. 可以自由指定域名且不许要备案
5. 基于全球最大的男性交友网站GitHub，方便交流。。。

<!-- more -->

## 所需环境
	node.js@5.5.0
	git@1.9.2
	hexo@3.2.2
	Github账号


## 新建一个repository
 repository名称为 username.github.io

![新建一个repository](http://i4.bvimg.com/608364/b042f1bd2bd6242d.png)

## 随便选择一个主题

![随便选择一个主题](http://i1.bvimg.com/608364/2306aefb8fe0777c.png)

此时在浏览器中输入 username.github.io 将会显示
…………
![你的博客](http://i4.bvimg.com/608364/ef3f0da4e8d240cf.png)

![成功了](http://i4.bvimg.com/608364/0ae79c2896aeb3d2.png)

别激动，这时只是创建了一个GitHub自带GitHub Pages主题，接下来配置安装hexo来搭建你更加定制的Blog主题。


## 安装Git和node.js
[Git官网下载地址](https://git-scm.com/downloads)
[Node.js官网下载地址](http://nodejs.cn/download/)
下载完成直接下一步下一步
最后将安装目录的bin文件加入到环境变量当中

关于设置环境变量
右击我的电脑->属性->高级系统设置->环境变量
找到Path编辑->新建->粘贴

## 安装hexo
打开Git Bash 输入
``` bash
npm install hexo-cli -g
```
等待数秒钟，中间可能会出现**WARN**没有关系

安装完成之后在CMD里面分别输入
``` bash
git --version
node -v
npm -v
```
来验证安装
结果如下图：
![安装成功](http://i1.bvimg.com/608364/4fb0b2d00d080173.png)

## 配置SSH
为了安全起见，我们来创建一个SSH安全连接
在Git Bash中输入
``` bash
cd ~/.ssh
```
来检测系统中是否已经存在了密钥。
若系统反馈为：**No such file or directory**
则我们需要创建一个
``` bash
ssh-keygen -t rsa -C "你的邮箱地址"
```
**注意C为大写**
一路三个回车键
然后按照反馈信息找到.ssh/id_rsa.pub使用txt或者sublime等文本处理软件打开
全选里面的内容，并复制
打开GitHub主页
点击右上角头像选择Setting
![Setting](http://i1.bvimg.com/608364/a7c4529943b91341.png)
选中左侧菜单SSH and GPG Keys
![SSH and GPG Keys](http://i1.bvimg.com/608364/fd3bd960e6fc6bc7.png)
将刚刚复制的内容粘贴到Key当中，Title可以不填,最后按Add SSH Key


## 测试SSH是否添加成功
在Git Bash中输入
``` bash
ssh -T git@github.com
```
如果得到反馈：Are you sure you want to continue connecting (yes/no)?
输入yes
如果看到
Hi liuxianan! You've successfully authenticated, but GitHub does not provide shell access.
则说明成功了
最后完善个人信息
``` bash
git config --global user.name "username"		//你的GitHub用户名
git config --global user.email  "user email"	//你的GitHub主邮箱
```



## 配置博客文件
为博客创建一个路径，例如 F:/hexo/username.github.io并初始化
在Git Bash中输入
``` bash
cd /f/hexo/username.github.io/
hexo init
```
此时hexo会下载一些文件
其中themes当中存放的是你的博客模板文件,source存放的是你的博客文章,_config.yml是你博客的一些参数配置，里面的参数直接按照提示修改即可(可以使用txt或者Sublime等字处理软件编辑)

## 启动生成博客网站
``` bash
hexo g	//生成
hexo s 	//本地浏览
```
此时在浏览器中输入[http://localhost:4000](http://localhost:4000)即可进行本地浏览


## hexo基本命令
缩写	|	全称			|功能
-|-|-
hexo n "new"|	hexo new "new"	|新建文章名为new
hexo p 		|	hexo publish	|草稿
hexo g 		| 	hexo generate	|生成
hexo c 		| 	hexo clean		|清空
hexo s 		|	hexo server		|开启本地服务(浏览)
hexo d 		|	hexo deploy		|部署

## 博客模板自定义
hexo的主题是可以更换的，默认使用的是landscape主题，我们替换成其他主题例如
>[有哪些好看的hexo主题](https://www.zhihu.com/question/24422335)

更换方法如下
``` bash
cd /f/hexo/zjko.github.io/
git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia

```
稍等片刻即可下载完成，所有已下载的主体都放在themes文件夹里。
通过修改博客目录里面的（在本例子当中为zjko.github.io文件夹下)_config.yml中的theme: landscape 为theme: yilia，然后重新生成。
通过修改主体文件夹中的_config.yml可以对主题进行定制。(本例中文件为theme/yilia/_config.yml)

## 编写上传配置
打开博客目录下的_config.yml，将一下内容复制进去
	deploy:
  		type: git
  		repository: git@github.com:zjko/zjko.github.io.git
  		branch: master

其中repository后面填写的内容与你的
![项目地址](http://i1.bvimg.com/608364/bc187f82e4b6dd52.png)
保持一致，修改成功之后之后上传都不需要修改。
**注意_config文件当中所有的设置参数‘:’之后均有空格**

## 安装deploy插件
``` bash
npm install hexo-deployer-git --save

```
安装完成之后，使用Git Bash进入博客目录输入hexo d即可部署到GitHub，此时即可通过username.github.io访问博客。

## 设置自己的域名
关于域名的购买可以参考一些域名服务商
关于购买域名的几点建议：

1. 不推荐大家使用国外服务商，因为可能会被墙，且不稳定
2. 若打算长期使用请注意价格，很多时候域名首年费用很低而续费很高，例如xxx域名第一年费用9元，第二年续费99元。这种情况并不少见。

购买了自己的域名之后，设置解析，将记录值设置为主机号。
可以通过ping username.github.io 来查看项目所在的主机号
![Blog的IP地址](http://i1.bvimg.com/608364/c8477d6ab9403d0c.png)
设置记录
![记录](http://i1.bvimg.com/608364/c17e851638ea3124.png)

解析可能需要一点时间，中间可以使用
	ping yourdomain
来检查是否解析成功

在这时可以回到自己的GitHub找到博客的这个Repository，进入Settings
![设置Domain](http://i1.bvimg.com/608364/80ed0a40da12515f.png)
在框框中写入自己的域名。
此时便已经完整的在GitHub上搭建了一个属于自己的Blog网站。


接下来你可能需要：
[关于编写博客](http://zjko.vip/2017/08/24/关于编写博客/)
