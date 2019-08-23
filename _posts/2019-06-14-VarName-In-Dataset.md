---
layout:     post
title:      Technique of Var Name in SAS Data Set 
subtitle:   for SAS using
author:     vincent_c
image: assets/images/tip1.jpg
categories: [ SAS, tips ]
tags: [SAS]
catalog: true
---  
在SAS的实际应用,经常会出现在程序中使用在dataset中的变量名做进一步逻辑处理的情况,下面三种方法可以获取dataset中的所有变量.

## 1. 获取已有数据集中的所有变量

### 1.1 方法一: dictionary.columns

读取表Dictionary.columns. (Dictionary表里保存着库名及其成员的名字。)

可以通过下面sql查询表中的各个字段的含义（输出在log中）

```
proc sql;describe table dictionary.columns;quit;
```
查询代码如下：

```
proc sql;select * from dictionary.columnswhere libname="SS" and memname='ALL_P5_SS';quit;
```
结果如下图：  
![](https://cl.ly/a79f2ae56fd4/Image%2525202019-06-14%252520at%25252011.13.41%252520%2525E4%2525B8%25258A%2525E5%25258D%252588.png)

说明：

* 可以根据libname和memname查询，但注意需要全部大写。  
* name表示变量名。
* varnum表示是第几个变量，是按照创建表时的顺序。


### 1.2 方法二: 读取View SASHELP.VCOLUMN
视图SASHELP.VCOLUMN是基于dictionary.columns创建，字段信息都一样。

* 通过sql：

```
proc sql;select * from sashelp.vcolumnwhere libname="SS" and memname='ALL_P5_SS';quit;```

* 通过DATA步：

```
data test;set sashelp.vcolumn;where libname="SS" and memname='ALL_P5_SS';run;
```

### 1.3 方法三: 通过proc contents
比前两个方法要灵活，效率高

* 将变量信息存入data set：

	```	proc contents data=ss.ALL_P5_SS	out=test(keep=varnum name)	noprint;	run;
	```

* 将变量名合并取出，放入一个宏变量，以作他用：

	```
	proc sql noprint;	select name into : var_name separated by ' ' from test;	quit;	%put &var_name;
	```

* 将表中变量分别放入宏变量：

	```
	proc sql noprint;	select count(*) into :amt_num from test;	%let amt_num=&amt_num;	select name into :var_name1 - :var_name&amt_num from 	test;	quit;	%put _global_;
	```


* 提取部分变量，放入宏变量：

	```
	proc sql noprint;	select name into : var_name1 - : var_name3 from test	where varnum in (1,2,3);	quit;	%put _global_;
	```

