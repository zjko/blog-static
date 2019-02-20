---
title: C语言操作数据库-ODBC
date: 2017-12-6 19:34:03
tags: [数据库,C]
toc: true
---

## 前言

本篇文章将通过一个代码案例让大家知道什么是ODBC，并教大家使用另一个姿势去使用C语言并操作数据库。以下是文章结构如下：
1. 什么是ODBC
3. 源代码
2. 代码详解


<!-- more -->
## 什么是ODBC

开放数据库连接（Open Database Connectivity，ODBC）是微软公司开放服务结构（WOSA，Windows Open Services Architecture）中有关数据库的一个组成部分，它建立了一组规范，并提供了一组对数据库访问的标准API（应用程序编程接口）。这些API利用SQL来完成其大部分任务。ODBC本身也提供了对SQL语言的支持，用户可以直接将SQL语句送给ODBC。开放数据库互连（ODBC）是Microsoft提出的数据库访问接口标准。开放数据库互连定义了访问数据库API的一个规范，这些API独立于不同厂商的DBMS，也独立于具体的编程语言（但是Microsoft的ODBC文档是用C语言描述的，许多实际的ODBC驱动程序也是用C语言写的。）

## 源代码

[源代码下载地址](https://github.com/zjko/ODBC.git)
本案例完全以对象的形式来进行数据库操作，将各种不同的问题交给不同的变量来解决，请参考具体代码。若有问题请在底部留言区留言，或者通过左侧电子邮件与我联系。
本程序仅提供代码文件，请自行创建工程，然后copy代码。
源代码一共分为Entrance.cpp(程序入口)，DBBean.h（数据库连接对象），Student.h(数据元组结构体)，StudentDao.h（Student表的操作对象），DBAccount（数据库账号和基本信息结构体），README.md（基本说明）。除此之外，还放了一款本人编写的开源数据库测试工具，可以十分小巧方便的测试数据库的连接状态，并完成一系列的数据库管理的基本功能，十分值得使用。源代码可以参考文章DBTest开发手册。

## 准备工作
### 设置登录名
如果不做任何设置，系统会默认用sa作为最高管理员账号。做自己的数据库应用的时候，不建议大家使用sa账号作为登录账号，大家可以新建一个登录名。在Microsoft SQL Server Management Studio中，可以用sa账号登录，然后右击左侧对象资源管理器的安全性->新建登录名，然后设置新用户名的权限，如下图所示。这些操作也完全可以通过SQL语句执行，新建一个查询输入语句，然后运行即可，在此不再赘述。
![新建用户名](http://zjko-blog-img.oss-cn-beijing.aliyuncs.com/Blog-Article/SQLServer%E6%96%B0%E5%BB%BA%E7%99%BB%E5%BD%95%E5%90%8D.png?x-oss-process=style/blog-img)
![设置权限](http://zjko-blog-img.oss-cn-beijing.aliyuncs.com/Blog-Article/SQLServer%E8%AE%BE%E7%BD%AE%E7%99%BB%E5%BD%95%E5%90%8D%E6%9D%83%E9%99%90.png?x-oss-process=style/blog-img)

### DBTest的使用
DBTest 是一款在Windows环境下小型开源的数据库连接与测试工具。用以解决开发者在进行各种数据库连接时，测试数据库连接状态。另外本软件还可以作为简单数据库可视化交互界面，用以直接测试与执行SQL语句。本软件源代码将会在我的Github上公开，另外我的个人博客也将不同步更新开发文档，欢迎大家关注。
[详细介绍](/2017/12/06/DBTest%E5%BC%80%E5%8F%91%E6%89%8B%E5%86%8C/)
如图所示，通过输入相关信息就可以进行数据库连接测试。
![DBTest登录](http://zjko-blog-img.oss-cn-beijing.aliyuncs.com/Blog-Article/DBTest%E7%99%BB%E5%BD%95.png?x-oss-process=style/blog-img)
如果连接失败并出现如图提示信息
![未开启服务](http://zjko-blog-img.oss-cn-beijing.aliyuncs.com/Blog-Article/%E6%9C%AA%E5%90%AF%E5%8A%A8%E6%9C%8D%E5%8A%A1.png?x-oss-process=style/blog-img)
则参考下一条。
### 检查是否打开了服务
右击我的电脑->管理,启动SQL Server服务。
![启动服务](http://zjko-blog-img.oss-cn-beijing.aliyuncs.com/Blog-Article/%E5%90%AF%E5%8A%A8SQL%E6%9C%8D%E5%8A%A1.png?x-oss-process=style/blog-img)
### 设置数据源
默认数据源可以填写.，若干出现数据源缺失，可以在控制面板->ODBC数据源，当中添加数据源，如下图所示。
![设置ODBC数据源](http://zjko-blog-img.oss-cn-beijing.aliyuncs.com/Blog-Article/%E8%AE%BE%E7%BD%AEODBC%E6%95%B0%E6%8D%AE%E6%BA%90.png?x-oss-process=style/blog-img)
## 操作ODBC
用C语言操作ODBC一共分为如下三步：
```c
    DBBean.getConnection();		//打开数据库连接
    OperatDB();					//操作数据库
    DBBean.closeConnection();	 //关闭数据库连接
```

### 连接数据库与关闭数据库
ODBC数据连接需要分以下几步：
具体代码参考 DBBean.h
#### 连接数据库

```c
	ret = SQLAllocHandle(SQL_HANDLE_ENV, NULL, &henv);			//分配环境句柄
	ret = SQLSetEnvAttr(henv, SQL_ATTR_ODBC_VERSION, (SQLPOINTER)SQL_OV_ODBC3, SQL_IS_INTEGER);	//
	ret = SQLAllocHandle(SQL_HANDLE_DBC, henv, &hdbc);			//分配连接句柄
	ret = SQLConnect(											//建立数据源
		hdbc,                                                   //连接句柄
		DBAccount.DBSOURCE, SQL_NTS,							//ODBC的DNS名称
		DBAccount.USERNAME, SQL_NTS,							//用户账号
		DBAccount.PASSWORD, SQL_NTS								//用户密码
	);
	if (SQL_SUCCEEDED(ret)) 							//连接失败时返回错误值
		puts("Connect Sucess!");
	else {
		puts("Conect Fail!");
	}
```
#### 关闭数据库连接
```c
	SQLFreeStmt(hstm, SQL_DROP);								//释放语句句柄
	SQLDisconnect(hdbc);										//断开与的连接
	SQLFreeConnect(hdbc);										//释放句柄资源
	SQLFreeEnv(henv);											//释放环境句柄
```


### 修改账户信息
参考DBAccount.h文件
```c
	SQLCHAR * DBSOURCE = (SQLCHAR *)"SQL Server Source";
	SQLCHAR * USERNAME = (SQLCHAR *)"username";
	SQLCHAR * PASSWORD = (SQLCHAR *)"password";
```


## 最后
关于代码重用的问题，案例代码当中的DBBean.h文件可以直接使用，不需要做任何修改，DBAccount文件当中，将信息改成自己的就可以。最后关于其他的内容的编写，请参考案例代码，根据不同的数据库设计稍作修改即可，在此不做赘述。

