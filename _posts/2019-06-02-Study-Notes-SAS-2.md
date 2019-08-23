---
layout:     post
title:      Study Notes for "SAS Certification Prep Guide - Advance"(Part 2)
subtitle:   Part 2 Chapter 13 - Chapter 24
author:     vincent_c
image: assets/images/studynotes2.jpg
catalog: true
categories: [ SAS, studynotes ]
tags: [SAS]
---  
This article recorded some study notes of book "SAS Certification Prep Guide - Advance" (Chapter 13 - Chapter 24)

## Foreword
since I had been preparing for SAS Advanced Programmer Exam, I write this study note which I study from the book "SAS Certification Prep Guide - Advance" in my blog, for recording this moments and experience.  

There are totally 2 parts for Chapter 2 to Chapter 24. Here is the Part Two (for chapter 13 to chapter 24).

## Chapter 13 - Creating Samples and Indexes

### 1.读取指定行
data set的结束标志为：读到文件EOF或者STOP
所以，不要忘记STOP(退出)+POINT(读指定行)+OUTPUT(输出)一起使用  

```
SET data-set-name POINT= point-variable;
```

在数据步execution的时候，执行到point语句之前，必须给var赋值
>You must use program statements to assign a value to this variable during execution of the DATA step, before execution of the SET statement

### 2. data集的总行数
其中variable可以在这个SET之前使用，因为SAS先进行编译的时候就已经从PDV中获取这个值了。

```
SET SAS-data-set NOBS=variable;
```
### 3. 随机数
seed必须有，不能为空,like RANUNI()  
seed不变，每次获取的随机数都一样。除非seed设置为0

```
RANUNI (seed)
```

配合CEIL函数，获取1~50的整数：

```
ceil(ranuni(0)*50)
```

### 4. 从data集中抽取部分随机样本
#### 4.1 可能有相同的row
比较简单，但每次获取随机数的时候，有可能会获得相同的数字，结果中有可能有相同的row，特别是抽取的样本比较多的时候

```
data work.rsubset (drop=i sampsize);	sampsize=10;	do i=1 to sampsize;		pickit=ceil(ranuni(0)*totobs);		set sasuser.revenue point=pickit nobs=totobs;		output;	end;	stop;
run;
```

#### 4.2 没有重复的随机样本
其中方法比较聪明，需要仔细研读、掌握

```
data test;	do i=1 to 1000;		value=ceil(ranuni(0)*100);		output;	end;run;data take_rand4;	sampsize=10;	obsleft=totobs;	do while(sampsize>0);		pickit+1; 		if ranuni(0)<sampsize/obsleft then do;			set test point=pickit nobs=totobs;			output;			sampsize=sampsize-1;		end;		obsleft=obsleft-1;	end;	stop; run;
```

### 5.在DATA步中创建Index
语法：

```
DATA SAS-data-file-name 
	(INDEX= (index-specification-1</UNIQUE><...index-specification-n></UNIQUE>));
```

创建一个简单Index： 
 
```
data simple (index=(division));
	set sasuser.empdata;
run;
```

创建2个简单Index：  

```
data simple2 (index=(division empid/unique));     set sasuser.empdata;run;
```
创建联合Index：

```
data composite (index=(Empdiv=(division empid)));     set sasuser.empdata;run;
```

#### 5.1 执行的顺序

建立 DATA 步视图，DATA 步仅被部分编译,中间代码在指定的 SAS 逻辑库当中存储为 VIEW 成员类型。  
引用 DATA 步视图:编译器解析中间代码并为主机环境产生可执行代码,生成的代码随后被执行。  

注意:若在 DATA 语句当中指定了其他的数据文件,则当后续的 DATA 或者 PROC 步调用了该视图时 SAS 才会创建这 些数据文件。所以,若想使用此类数据文件,必须先引用 DATA 步视图。  

```
data _null_ WORK.BAD_DATA / view=WORK.BAD_DATA; 
	set SASUSER.LOOK(keep=Xa Xb Xc);	length _Check_ $ 10 ;	if Xa=. then _check_=trim(_Check_)!!" Xa" ; 
	if Xb=. then _check_=trim(_Check_)!!" Xb" ; 
	if Xc=. then _check_=trim(_Check_)!!" Xc" ; 
	put Xa= Xb= Xc= _check_= ;run ;
```
上述例子中执行的时候，只在 work 下创建了一个名为 temp 的视图,视图中也没有观测。当你双击该视图后,才写入观测,并创建了另一个名为 Error 的数据集。


#### 5.2 相关的日志开关
显示操作Index等相关操作的日志，like创建Index、Merge、Sort
并且能够在log中提示Index may Help,是否要创建Index


```
OPTIONS MSGLEVEL= N|I;
```

#### 5.3 不需要Index的时候


1. with a subsetting IF statement in a DATA step
1. with particular WHERE expressions
1. if SAS determines it is more efficient to read the data sequentially.


### 6. 在PROC DATASETS管理Index
在这个proc中管理Index，比起rebuild data set更有效率


```
PROC DATASETS LIBRARY= libref <NOLIST>;MODIFY SAS-data-set-name; 
INDEX DELETE index-name; 
INDEX CREATE index-specification;QUIT;
```

eg:

```
proc datasets library=sasuser nolist;	modify sale2000;	index delete origin;	index create flightid;	index create Fromto=(origin dest);quit;
```

### 7.Managing Indexes with PROC SQL

```
PROC SQL;CREATE <UNIQUE > INDEX index-nameON table-name(column-name-1<...,column-name-n>); 
DROP INDEX index-name FROM table-name;QUIT;
```

### 8. Documenting and Maintaining Indexes

* Index和数据集存放在同一个libref下面，并且名字也一样，只是Index的后缀类型不一样  
* 一个数据集只有一个Index文件，即使这个数据集有多个Index，也是存放在那一个Index文件下面  
* 但是Index文件在SAS的Explorer window看不到  

可以通过下面两个方法查看具体信息：

```
PROC CONTENTS DATA=<libref.>SAS-data-set-name; 
RUN;
```

```
PROC DATASETS <LIBRARY=libref> <NOLIST>;CONTENTS DATA=<libref.>SAS-data-set-name; 
QUIT;
```

上面两个方法一样，输出的结果也一样，包含：

1. general and summary information 
1. engine/host dependent information 
1. alphabetic list of variables and attributes 
1. alphabetic list of integrity constraints alphabetic 
1. list of indexes and attributes.

### 9. Copy数据集(数据+Index)


```
PROC DATASETS LIBRARY=old-libref <NOLIST>;COPY OUT=new-libref; 
SELECT SAS-data-set-name;QUIT;
```

和上面的效果一样，但如果带上MOVE的option，copy完数据，删除Index，再重建Index

```
PROC COPY OUT=new-libref IN=old-libref <MOVE>;SELECT SAS-data-set-name(s); 
RUN;QUIT;
```

eg:  下面两个效果一样，把sasuser. sale2000->work. sale2000

```
proc datasets library=sasuser nolist;     copy out=work;
     select sale2000;  quit;proc copy out=work in=sasuser;     select sale2000;quit;
```

注：
>If you copy and paste a data set in either SAS Explorer or in SAS Enterprise Guide, a new index file is automatically created for the new data file.

注意：  
但是如果是在data步中通过set的方式来copy数据，是不会有index过来的。  

例子：下面的数据集test原本是有index的，但通过set的方式，index会被删除了（不会copy过来）    

```
data WORK.TEST;set WORK.TEST(keep=Id Var_1 Var_2 rename=(Id=Id_Code)); Total=sum(Var_1, Var_2);run;
```


### 10. Renaming Data Sets

修改了数据集的名字，同名的Index也跟着修改

```
PROC DATASETS LIBRARY=libref <NOLIST>;CHANGE old-data-set-name = new-data-set-name; 
QUIT;
```

### 11. Renaming Variables

```
PROC DATASETS LIBRARY=libref <NOLIST>;MODIFY SAS-data-set-name;RENAME old-var-name-1 = new-var-name-1 <...old-var-name-n = new-var-name-n>;QUIT;
```
注：

1. 如果修改的VAR是Index，Index名字也同时rename  
1. 如果修改的Var是联合Index中的其中一个，同时修改了Var的引用  
1. 如果修改后的Var的名字和联合Index名字一样，报错  


## Chapter 14 - Combining Data Vertically

### 1. 使用FILENAME合并

```
FILENAME fileref (‘external-file1’ ‘external-file2’ ...‘external-filen’);
```

注：  

1. 不要忘记了所有多个fileref都要在括号和引号之间。   
1. 多个fileref都有引号，但中间使用空格分隔，不是逗号  
1. 不了解file内容的话，可以用proc fslist先查看。  

```
proc fslist file=fileref;
run;
```

### 2. INFILE with FILEVAR
可以根据程序logic，在infile时，改变FILEVAR中的文件名，来控制读取不同的raw文件。

```
INFILE file-specification FILEVAR= variable;
```

### 3. COMPRESS函数
去掉字符串中的某个字符(可以是多个)，如果第二个参数为空，默认去掉空格

```
COMPRESS(source, <characters-to-remove>);
```
eg:下面的例子去掉空格和小数点。  

```
compress("123.456 789"," .");
```

### 4.利用循环读取多个raw文件的数据

```
data work.quarter (drop=monthnum midmon lastmon);
	monthnum=month(today());
	midmon=month(intnx(’month’,today(),-1));
	lastmon=month(intnx(’month’,today(),-2));

	do i = monthnum, midmon, lastmon;
		nextfile="c:\sasuser\month"!!compress(put(i,2.)!!".dat",’ ’);
		
		do until (lastobs);
			infile temp filevar=nextfile end=lastobs;
			input Flight $ Origin $ Dest $ Date : date9.
			RevCargo : comma15.2;
			output;
		end; 
	end;
	stop; 
run;	
```

其中：
  
1. 利用intnx根据时间间隔，推算新的日期  
1. 利用第一层do读取每个文件  
1. 利用第二层do until读取文件中的每行数据  
1. 利用end=var控制每个文件中的最后一行数据  


### 5. 利用APPEND合并数据集

```
PROC APPEND BASE=SAS-data-set DATA=SAS-data-set;RUN;
```
注：  

1. BASE的数据集为结果数据集   
1. PROC APPEND只读DATA的数据集，所以更有效率  
1. 如果BASE中有的var，但DATA中没有这个var，合并后为missing,日志中有提示。  
1. 如果DATA中有比BASE多出来的var，合并失败，BASE数据没有改变。日志中有提示。  

* 如果DATA有多出来的VAR，需要使用FORCE。

```
PROC APPEND BASE=SAS-data-set DATA=SAS-data-set force;RUN;
```

* 如果某个var的长度在BASE中短，在DATA中长，也需要用FORCE，并且合并后，DATA中的那个var会被trunc掉。

* 如果base的var和data的var类型不一样，必须用force，data的var会变成missing，存入base中。SAS并不会进行隐式转换。  

### 6. 将raw file文件名存放在data set，再读取合并
eg:
data set的内容如下：  

obs|readit  
------|-------
1|rawfile1.dat
2|rawfile2.dat
3|rawfile3.dat

可以从数据集中读取出文件名，配合filevar：

```
data work.newroute;
   set sasuser.rawdata;
   infile in filevar = readit end = lastfile;
   do while(lastfile = 0);
      input  @1 RouteID $7. @8 Origin $3. @11 Dest $3.
             @14 Distance 5. @19 Fare1st 4.
             @23 FareBusiness 4. @27 FareEcon 4.
             @31 FareCargo 5.;
output; end;
run;
```


### 7. 将raw file文件名存放在外部文件中，再读取合并

```
data work.newroute;     infile ’rawdatafiles.dat’;     input readit $10.;     infile in filevar=readit end=lastfile;     do while(lastfile = 0);        input  @1 RouteID $7. @8 Origin $3. @11 Dest $3.               @14 Distance 5. @19 Fare1st 4.               @23 FareBusiness 4. @27 FareEcon 4.               @31 FareCargo 5.;output; end;run;
```


## Chapter 15 - Combining Data Horizontally

### 1. 几种简单的mapping，可以看做横向合并

* 使用IF-ELSE：

```
data mylib.employees_new;	set mylib.employees;	if IDnum=1001 then Birthdate=’01JAN1963’d;		else if IDnum=1002 then Birthdate=’08AUG1946’d;		else if IDnum=1003 then Birthdate=’23MAR1950’d;		else if IDnum=1004 then Birthdate=’17JUN1973’d;run;
```

* 使用数组：

```
data mylib.employees_new;	array birthdates{1001:1004} _temporary_ (’01JAN1963’d           ’08AUG1946’d ’23MAR1950’d ’17JUN1973’d);	set mylib.employees;	Birthdate=birthdates(IDnum);run;```

* 使用自定义Format：

```
proc format;	value birthdate 
		1001 = ’01JAN1963’		1002 = ’08AUG1946’		1003 = ’23MAR1950’		1004 = ’17JUN1973’;run;data mylib.employees_new;	set mylib.employees;
	Birthdate=input(put(IDnum,birthdate.),date9.);run;
```

### 2. 使用Data Merge：  
但需要注意的是，more-to-more横向合并的时候，不是笛卡尔乘积（使用SQL才可以）  

### 3. 使用proc sql:  

### 4. Data Merge和Proc SQL的比较：  

1. Data Merge：    
	* 优点：  
		* 有没有需要合并的data set的数量要求，只占内存  
		* data步中可以使用数组、数据集中的变量等完成复杂logic  
		* BY后面可以加多个VAR，根据多个VAR进行关联  
	* 缺点：
		* 因为有by，必须先排序
		* by后面的var必须在所有数据集中都有，而且名字要一样
		* 关联的var的值必须一样（不可以先sql中where的其他逻辑条件）
		   		
2. PROC SQL：    
	* 优点：  
		* 不用先排序（但如果有index，会有帮助）  
		* 一个sql就可以完成复杂的多表关联，而且不用有相同的var  
		* 关联后的结果可以输出数据集、创建表或者view    
	* 缺点：
		* 一次最多关联32张表  
		* sql中不能用数据集的var、数组、loop等
		* 相比Data Merge，需要更多的系统资源

3. more-to-more关联：  
	Data Merge的不是笛卡尔乘积，merge的过程可以理解成set1和set2各有一个游标，匹配上了就都向下移动一个。  
当两个set的游标都因为by中var的值下移了，SAS会初始化PDV，将所有值写为missing。  
有一个set的游标因为var改变移动了，另一个没有下移，PDV里面的值不会被初始化，merge的时候，直接填充或者覆盖  。

	PROC SQL是笛卡尔乘积。因为它的logic是，先计算两个表的笛卡尔乘积，然后再根据where中的条件，将笛卡尔乘积中不符合条件的row删除，最后得到目标结果。   

![](https://cl.ly/d4a461e91a5d/Image%2525202019-05-25%252520at%25252010.10.22%252520%2525E4%2525B8%25258A%2525E5%25258D%252588.png)  

![](https://cl.ly/ab9ecccb538e/Image%2525202019-05-25%252520at%25252010.14.58%252520%2525E4%2525B8%25258A%2525E5%25258D%252588.png)

### 5. Using Multiple SET Statements

```
data combine;     set dataset1;     set dataset2;run;
```

1. SAS同时读dataset1的一条ob，和dataset2的一条ob，然后用dataset2的var覆盖掉dataset1中相同的var，不同的var就直接写入。  
1. 一直读到其中一个文件EOF为止，所以结果数据集的行数为dataset1和dataset2的最小行数。  
1. dataset1的所有var和dataset2的所有var都会在结果数据集中出现。  
1. 但第二个set语句执行时，PDV的所有var不会初始化（变为missing）。    
可以使用下面的方式，使Data Merge也能完成more-to-more的笛卡尔合并：

```
data flightemps3(drop=empnum jobcode);	set sasuser.flightschedule;	do i=1 to num;		set sasuser.flightattendants				(rename=(empid=empnum))				nobs=num point=i;		if empid=empnum then output;	end;run;
```

### 6. Combining Summary Data and Detail Data
思路：  

1. 先使用proc means或者其他方式获得你需要的means，或者std或者sum。  
1. 然后再和原dataset合并。  

先使用proc means获取平均数：

```
proc means data=sasuser.monthsum noprint;	var revcargo;	output out=sasuser.summary sum=Cargosum;run;
```
和原数据集合并，利用的是_n_=1时先set一个数据集，后面set的时候，前面PDV的值不会被去掉。

```
data sasuser.percent1(drop=cargosum);	if _N_=1 then set sasuser.summary(keep=cargosum);	set sasuser.monthsum(keep=salemon revcargo);	PctRev=revcargo/cargosum;run;
```

利用_n_=1时，先loop计算sum，然后再重新读原数据集，合并

```
data sasuser.percent2(crop=totalrev);	if _N_=1 then do until (LastObs);		set sasuser.monthsum(keep=revcargo) end=lastobs;		TotalRev+revcargo;	end;        
	set sasuser.monthsum(keep=salemon revcargo);	PctRev=revcargo/totalrev;run;
```

### 7. 利用Index合并数据

```
SET SAS-data-set-name KEY=index-name;
```
set的时候，SAS利用index的值（可能是多个var的复合index）在目前PDV的值匹配，找到和PDV中的值相同的值为止。所以这个set的时候，PDV中一定要有index的var的值（即目标值）。  

如果一个数据步中set指定了key，在同一个数据步中不能同时使用WHERE。  

* 正确的例子：

```
data work.profit;	set sasuser.dnunder;	set sasuser.sale2000(keep=routeid flightid date rev1st revbusiness 
		revecon revcargo) key=flightdate;	Profit=sum(rev1st, revbusiness, revecon, revcargo,		-expenses);run;
```

1. 先set sasuser.dnunder，PDV中就有值，包括index的var的值。  
1. 再读sale2000，因为有key，直接读到和index的var相同的值。  
1. 根据logic，进行关联，计算相关变量。  

* 错误的例子：

```
data work.profit2;	set sasuser.sale2000(keep=routeid flightid date		rev1st revbusiness revecon revcargo)		key=flightdate;	set sasuser.dnunder;	Profit=sum(rev1st, revbusiness, revecon, revcargo,		-expenses);run;
```

1. 刚开始的时候，PDV中没有值。也不会有index的var的值。
1. 读取sasuser.sale2000，因为有key，但PDV中没有index的var的值，所有没有数据从这个数据集中读取到。
1. 读取第二个SET dnunder，将这个数据集中所有var的值放入PDV。  
1. 第一次loop完成，output数据（没有sale2000的数据，只有dnunder的数据）
1. 每次data set loop的时候，PDV的var不会被初始化。
1. 第二次loop的时候，读取sasuser.sale2000，因为有key，所以从PDV中查找index的var的值，这个值其实是上一次dnunder的值，而且这个值已经被写入结果数据集了。 从sale2000找到匹配的值，放入PDV。
1. 读取第二个SET dnunder，将这个数据集中所有var的值放入PDV。  
1. 第二次loop完成，output数据（这时数据已经有错位了）

### 8. The _ IORC _  Variable(INPUT/OUTPUT Return Code)

>还是上面的逻辑，如果数据没有错位，但最后一条数据是有问题的。  
从dataset1中读取var放入PDV，然后从dataset2中读取数据，因为有key=，但是假设在dataset2没有找到相同的var的值，PDV中已经有dataset1的var的值，最后loop的时候还是会output的，但这个不是我们想要的数据。  

在KEY=或者MODIFY statement with the KEY=时，SAS会自动产生这个值。  
_ IORC _=0表示找到了和index相应的值，为1表示没有找到。  

完整的程序应该为：

```
data work.profit3 work.errors;	set sasuser.dnunder;	set sasuser.sale2000(keep=routeid flightid date rev1st		revbusiness revecon revcargo)key=flightdate;	if _iorc_=0 then do;		Profit=sum(rev1st, revbusiness, revecon, revcargo,			-expenses);		output work.profit3;	end;	else do;		_error_=0;		output work.errors;	end;run;
```

### 9. Using the UPDATE Statement

```
DATA master-data-set;	UPDATE master-data-set transaction-data-set;	BY by-variable(s); 
RUN;
```

1. update只能有2个dataset
1. 注意2个dataset的顺序，主表必须第一个
1. 必须有BY  1. 必须先sort或者有index1. 主表中的BY的var必须是唯一的  


## Chapter 16 - Using Lookup Tables to Match Data

如果两张表的结构不一样，不能直接用merge或者proc sql合并  

### 1. 利用多维数组合并

```
ARRAY array-name {rows,cols,...} <$> <length><array-elements> <(initial values)>;
```

1. 数组的名字不能和所在dataset中的var重复。  
1. 数组的所有值必须是同一个类型，char or numeric。  
1. 可以指定数组的初始值，放在括号中。  

用_TEMPORARY_定义临时数组：  

1. 不会output到数据集中。  
1. 不可以使用{*}，必须指定数组的长度。  
1. 临时数组的值会自动retain（每次的loop）  

eg:

```
data work.lookup1;	array Targets{1997:1999,12} _temporary_;	if _n_=1 then do i= 1 to 3;		set sasuser.ctargets;		array Mnth{*} Jan--Dec;		do j=1 to dim(mnth);			targets{year,j}=mnth{j};		end;
	end;
	
	set sasuser.monthsum(keep=salemon revcargo monthno);
	year=input(substr(salemon,4),4.);	Ctarget=targets{year,monthno}; 	format ctarget dollar15.2;run;
```
在_ n _=1的时候，将dataset的值读取到多维数组中，然后在后面的logic中再处理合并的logic### 2. Using PROC TRANSPOSE

有时候，两张表的表结构不一样的话，可以先处理，比如转置，然后再merge  

```
PROC TRANSPOSE <DATA=input-data-set>		<OUT=output-data-set> 
		<NAME=variable-name>  
		<PREFIX=variable-name>;	BY <DESCENDING> variable-1 
		<...<DESCENDING> variable-n> <NOTSORTED>;	VAR variable(s); 
RUN;
```

例子：

* 没有指定关键字VAR，所有numeric的var都转置，结果如下：

```
data a;	input year Jan Feb Mar;	cards;	1997 10 11 12	1998 20 21 22	1999 30 31 32;proc transpose data=a out=b;run;
```
  

![](https://cl.ly/74fc4d75879a/Image%2525202019-05-25%252520at%2525203.12.00%252520%2525E4%2525B8%25258B%2525E5%25258D%252588.png)  


* 利用name=修改转置后的_ name _的名字，利用prefix=修改转置后col1,col2...的前缀。  

```
proc transpose data=a out=b	name=Month	prefix = value;run;```

结果如下：  

![](https://cl.ly/a691474670dc/Image%2525202019-05-25%252520at%2525203.15.38%252520%2525E4%2525B8%25258B%2525E5%25258D%252588.png)  

* 利用关键字BY=，指定不要转置的var  

```
proc transpose data=a out=b		name=Month		prefix = value;	by year;run;
```
结果如下：
![](https://cl.ly/42ca0d514a80/Image%2525202019-05-25%252520at%2525203.18.42%252520%2525E4%2525B8%25258B%2525E5%25258D%252588.png)

### 3. Using Hash Objects as Lookup Tables

#### 3.1 创建并手动赋值hash

例子： 

```
data work.difference (drop= goalamount);     length goalamount 8;     if _N_ = 1 then do;        declare hash goal( );        goal.definekey("QtrNum");        goal.definedata("GoalAmount");        goal.definedone( );        call missing(qtrnum, goalamount);        goal.add(key:’qtr1’, data:10 );        goal.add(key:’qtr2’, data:15 );        goal.add(key:’qtr3’, data: 5 );        goal.add(key:’qtr4’, data:15 );    end;    set sasuser.contrib;    goal.find();    Diff = amount - goalamount;run;```

先定义hash：

```
declare hash goal;
```

再初始化：

```
goal= _new_ hash();
```

也可以直接定义+初始化：

```
declare hash Goal();
```

定义key和value的值，并且 + done：

```
goal.definekey ("QtrNum");
goal.definedata ("GoalAmount");goal.definedone();``` 

初始赋值：

```
call missing(qtrnum, goalamount);
```

手动赋值： 

```
goal.add(key:’qtr1’, data:10 );goal.add(key:’qtr2’, data:15 );goal.add(key:’qtr3’, data: 5 );goal.add(key:’qtr4’, data:15 );
```
获取hash中的值，因为是在dataset中，dataset的var作为hash的key，获取的值放到hash的data的var中。

```
goal.find();
```

#### 3.2 将dataset的值赋值给hash  

* Using a Non-Executing SET Statement
利用if 0可以set到PDV中，但不读取任何数据  

```
if 0 then set sasuser.acities(keep=Code City Name);
```
例子：  

```
data work.report;
	if _N_=1 then do;
		if 0 then set sasuser.acities(keep=Code City Name);
		
		declare hash airports (dataset: "sasuser.acities");
		airports.definekey ("Code");
		airports.definedata ("City", "Name");
		airports.definedone();
	end;
	
	set sasuser.revenue;
	rc=airports.find(key:origin);
	if rc=0 then do;
		OriginCity=city;
		OriginAirport=name;
	end;
	else do;
		OriginCity=’’;
		OriginAirport=’’;
	end;
	
	rc=airports.find(key:dest);
	if rc=0 then do;
		DestCity=city;
		DestAirport=name;
	end;
	else do;
		DestCity=’’;
		DestAirport=’’;
	end; 
run;
```

利用dataset:数据集名称，将dataset的内容赋值给hash  
其中，可以一个key，对应多个data  

```
declare hash airports (dataset: "sasuser.acities");
airports.definekey ("Code");
airports.definedata ("City", "Name");
airports.definedone();
```
可以用下面的语句，将所有var都存放在data中  

```
hashobject.DEFINEDATA (ALL:“YES”);
```

根据dataset的var，作为hash的key，获取值放入data中（一个或者多个，depend具体定义）  

```
airports.find(key:origin);
airports.find(key:dest);
```
如果find的返回值为0，表示找到了；如果不为0，表示没有找到  

```
rc=hashobject.find (key:keyvalue);
```

## Chapter 17 - Formatting Data

### 1. Creating Non-Overlapping Formats

例子：

```
libname library ’c:\sas\newfmts’;
proc format lib=library;
	value $routes
		’Route1’ = ’Zone 1’
		’Route2’ - ’Route4’ = ’Zone 2’
		’Route5’ - ’Route7’ = ’Zone 3’
		’ ’ = ’Missing’
		other = ’Unknown’;
		
	value $dest
		’LHR’,’LIS’,’MAD’,’NBO’,’PEK’,’PRG’,
		’SIN’,’SYD’,’VIE’,’WLG’ = ’International’
		’MSY’,’ORD’,’PWM’,’RDU’,’SEA’,’SFO’ = ’Domestic’;
		
	value revfmt
		. = ’Missing’
		low - 10000 = ’Up to $10,000’
		10000 <- 20000 = ’$10,000+ to $20,000’
		50000 <- 60000 = ’$50,000+ to $60,000’
		60000 <- HIGH  = ’More than $60,000’;
run;
```

1. 字符型的range，可以使用减号  
1. numeric的range，不可以直接使用减号，要变成"<-"，但是有low和high的话，可以使用减号(上面例子high的部分可以换成减号)
1. 注意考虑char和numeric的missing的定义。  
1. char的多个值，可以用逗号分隔开  

### 2. Creating a Format with Overlapping Ranges
range有相互重叠的部分，在定义时，加上(MULTILABEL)

```
VALUE format-name (MULTILABEL);
```
例子：

```
proc format;	value dates (multilabel)
		’01jan2000’d - ’31mar2000’d = ’1st Quarter’		’01apr2000’d - ’30jun2000’d = ’2nd Quarter’		’01jul2000’d - ’30sep2000’d = ’3rd Quarter’		’01oct2000’d - ’31dec2000’d = ’4th Quarter’		’01jan2000’d - ’30jun2000’d = ’First Half of Year’		’01jul2000’d - ’31dec2000’d = ’Second Half of Year’;
run;
```

在下面几个proc中可以使用，但必须带上MLF的option：

* PROC TABULATE 
* PROC MEANS 
* PROC SUMMARY

例子：

```
proc tabulate data = sasuser.sale2000 format = dollar15.2;	format Date dates.;	class Date / mlf;	var RevCargo;	table Date, RevCargo*(mean median);run;
```


### 3. Creating Custom Formats Using the PICTURE Statement

```
PROC FORMAT; 
	PICTURE format-name		value-or-range=‘picture’;RUN;
```
* digit selectors  

	99/999....表示非0，是需要在前面填充0的  
	00/000....表示没有，前面不需要填充0  

picture|Data value | Formatted value
----|----|----|
picture month 1-12=’99’;|01|01
picture month 1-12=’99’;|1|01
picture month 1-12=’99’;|12|12
picture month 1-12=’00’;|01|1
picture month 1-12=’00’;|1|1
picture month 1-12=’00’;|12|12

* message characters

picture|Data value | Formatted value
----|----|----|
picture month 1=’99 JAN’;|1|01 JAN

* directives

directive | result
----|----
%a|abbreviated weekday name %A full weekday name%b|abbreviated month name %B full month name%d|day of the month as a number 1-31, with no leading zero%H|24-hour clock as a number 0-23, with no leading zero%I|12-hour clock as a number 1-12, with no leading zero%j|day of the year as a number 1-366, with no leading zero %m month as a number 1-12, with no leading zero%M|minute as a decimal number 0-59, with no leading zero%p|AM or PM%S|second as a number 0-59, with no leading zero%U|week number of the year (Sunday is the first day of the week) as a number 0-53, with no leading zero%w|weekday as a number (1=Sunday, to 7)
%y|year without century as a number 0-99, with no leading zero 
%Y|year with century as a number

eg:

```
proc format;	picture mydate		low-high=’%0d-%b%Y  ’ (datatype=date);run;proc print data=sasuser.empdata		(keep=division hireDate lastName obs=5);	format hiredate mydate.;run;
```
注：

* low到high是所有的值。  
* %0d中的0是为了在前面补一个0，1号就变成了01号。  
* ’%0d-%b%Y  ’，引号中的长度就是结果char的最大长度，超出部分就会被trunc掉，如果是’%0d-%b%Y’，就只有8位，就会变成11-MAR19。  

### 4. Managing Custom Formats
* 将CAT下面所有format都dispay出来：

```
PROC FORMAT LIB=library FMTLIB; 
	SELECT format-name;	EXCLUDE format-name; 
RUN;
```
eg：  
	displaylibrary下面的format：  

```
libname library ’c:\sas\newfmt’;	proc format lib=library fmtlib;run;
```
display指定的format-name：

```
libname library ’c:\sas\newfmt’;proc format lib=library fmtlib;	select $routes;run;
```

format也是放在CAT里面的，所有和dataset一样，可以使用PROC CATALOG操作：  

```
PROC CATALOG CATALOG=libref.catalog;	CONTENTS <OUT=SAS-data-set>; 
	COPY OUT=libref.catalog <options>; 
	SELECT entry-name.entry-type(s); 
	EXCLUDE entry-name.entry-type(s); 
	DELETE entry-name.entry-type(s);RUN;QUIT;
```

修改dataset中已经绑定的format

```
PROC DATASETS LIB=SAS-library <NOLIST>;	MODIFY SAS-data-set; 
	FORMAT variable(s) format;QUIT;
```
注： 

1. PROC DATASETS不需要RUN，需要QUIT  
1. FORMAT中的var后面，如果有没有指定新的format，表示移除旧的format  

eg:

```
proc datasets lib=Mylib;	modify flights;	format dest $dest.;	format baggage;quit;
```

### 5. 系统查找format的顺序
比较好做下面的事情：  

1. 指定Library的libref，指定为你SAS运行的程序所在的地方。  
1. 使用PROC FORMAT创建format的时候，带上LIB=LIBRARY。  
1. 使用LIBNAME指定你自定义的format所在的地方，指定Library。

默认的查找顺序：  
先在Work.Formats下面找，然后去Library.Formats下面找

可以通过下面的options改变、添加查找format的顺序：  

```
OPTIONS FMTSEARCH= (catalog-1 catalog-2...catalog-n);
```
注：  

* 如果指定的值没有work或者library，SAS还是先在Work.Formats下面找，然后去Library.Formats下面找。
* 如果指定的自定义的lib在library前面，则还是先work，然后再按照指定的顺序。

如：  
fmtsearch=(mylib) -> 1. Work.formats 2. library.formats 3. mylib  
fmtsearch=(mylib, library)->1. Work.formats 2. mylib 3.library.formats

### 6. 相关的options

如果指定了自定义的format，但没有找到（没有libname正确的路径，或是拼写错误），SAS会在log中error，然后中断。  

可以通过下面的options，让SAS自己找一个format($\$w. or w.)替代，然后继续运行。  


```
OPTIONS FMTERR(默认) | NOFMTERR;
```

### 7. 从dataset中的数据，创建自定义format
LIBRARY后面是新的format，CNTLIN是数据集（有数据的）

```
PROC FORMAT LIBRARY=libref.catalog CNTLIN=SAS-data-set;
```

但是，dataset的格式是有要求的：  

1. 必须有变量名为FmtName(format的名字), Start(format range的开始值), Label（format对应的输出值）  
1. 如果是range，必须要有一个变量名为end。如果是单值，end可以省略。  
1. 如果FmtName的值以$\$开头（表示是format里面的range是一个char的值）。如果format的range是char，但FmtName的值不以$开头，就必须再指定一个新变量，变量名为Type。  

1. 如果里面有多个format，即多个FmtName的值，就必须sort by FmtName.  

例子，已经有一个dataset，但有些var名称不符合要求：

```
data sasuser.aports;
   keep Start Label FmtName;
   retain FmtName ’$airport’;
   set sasuser.acities (rename=(Code=Start
       City= Label));
run;
```

1. 可以keep想要的字段。
1. 用rename改变量的名字。
1. 手动添加retain的FmtName。


### 8. 从将自定义format导出到dataset中

应用场景：  
	  对已经的自定义的format，要修改。可以先导入，再用proc sql或者data修改。最后再导入。
	  
```
PROC FORMAT LIBRARY=libref.catalog 
		CNTLOUT=SAS-data-set; 
	SELECT format-name format-name...;
	EXCLUDE format-name format-name...; 
RUN;
```

和导入过程中的注意事项一样，主要是var的名字。

## Chapter 18 - Modifying SAS Data Sets and Tracking Changes

下面例子中，SAS为A复制一个copy，然后从这个copy中读数据，写入数据集a，最后把copy删除掉。

```
data a;
	set a;
	var=XXX;
run;
```

### 1. 修改所有的数据
```
DATA SAS-data-set; 
	MODIFY SAS-data-set;
	existing-variable = expression; 
RUN;
```

### 2. 修改数据Using a Transaction Data Set

主表紧跟在modify后面，transaction表在主表的后面

```
DATA SAS-data-set;	MODIFY SAS-data-set transaction-data-set;	BY key-variable; 
RUN;
```

1. SAS会根据BY var自动生产一个where从句，进行刷选，定位。定位好之后，进行replaced, deleted, or appended。 默认是replace。  
1. 虽然有BY，但不用先sort。但sort或者index对性能有帮助。 
 
* 如果主表有相同的BY VAR的值，只更新主表中where查找到的第一条数据  

主表：

a|b
----|----
1|10
1|20
2|30

trans表：

a|b
----|----
1|999

最后的主表为：

a|b
----|----
1|999
1|20
2|30


* 如果trans表有相同的BY VAR的值，只更新主表中where查找到的第一条数据，更新值为trans相同BY VAR的最后一条数据，即后面的覆盖前面的

主表：

a|b
----|----
1|10
1|20
2|30

trans表：

a|b
----|----
1|999
1|888

最后的主表为：

a|b
----|----
1|888
1|20
2|30


* trans表中有missing   

默认trans表中的missing不会更新主表中相应的var，但可以通过options指定：

```
MODIFY master-data-set transaction-data-set 
		UPDATEMODE=MISSINGCHECK | NOMISSINGCHECK;
```
MISSINGCHECK(默认)，不处理missing  
NOMISSINGCHECK，强制更新missing的var  


### 3. 修改数据By Index

```
MODIFY SAS-data-set KEY=index-name;
```
和前面的BY trans表不一样的是：  

1. 必须显式指定replace，BY trans默认的就是replace，不用指定。  
1. trans表中所有数据都必须在主表中找的到。
1. 如果trans表中有重复数据，只有第一条数据会用来replace主表的数据，然后log报错，中断（除非使用了UNIQUE选项）。  

例子：

```
data cargo99;	set sasuser.newcgnum (rename =		(capcargo = newCapCargo		cargowgt = newCargoWgt		cargorev = newCargoRev));	modify cargo99 key=flghtdte;	capcargo = newcapcargo;	cargowgt = newcargowgt;	cargorev = newcargorev;run;```

* 主表中有重复数据，和BY trans一样：

![](https://cl.ly/0947597320e1/Image%2525202019-05-26%252520at%2525203.30.06%252520%2525E4%2525B8%25258B%2525E5%25258D%252588.png)

* trans表中有重复数据，但不连续，和BY trans一样，覆盖：

![](https://cl.ly/d0ab0d4107c9/Image%2525202019-05-26%252520at%2525203.31.28%252520%2525E4%2525B8%25258B%2525E5%25258D%252588.png)

* trans表中有重复数据，连续，使用第一条，报错，中断：

![](https://cl.ly/886624e5b544/Image%2525202019-05-26%252520at%2525203.32.12%252520%2525E4%2525B8%25258B%2525E5%25258D%252588.png)

* 使用UNIQUE：

```
MODIFY SAS-data-set KEY=index-name /UNIQUE;
```

就可以和By trans一样了，后面覆盖前面的

![](https://cl.ly/3e926fb7ce1a/Image%2525202019-05-26%252520at%2525203.33.59%252520%2525E4%2525B8%25258B%2525E5%25258D%252588.png)


### 4. Controlling the Update Process
一共有三种模式，可以根据logic显式增加。  
可以一起用多种模式，但OUTPUT必须放在REPLACE和REMOVE的后面，用来保证logic上的数据完整性。  

```
OUTPUT; 
REPLACE; 
REMOVE;
```

例子：

```
data master;     set transaction;     modify master key = id;     a = b;     if code = ’no’ then remove;     else if code = ’yes’ then replace;     else if code = ’new’ then output;run;
```

### 5. Using \_IORC\_ with %SYSRC

```
data master;	set transaction;	modify master key = id;	if _IORC_=%sysrc(_sok) then do;		a = b;		replace;	end;	else if _IORC_=%sysrc(_dsenom) then do; 
		output;		_ERROR_ = 0;	end;
run;
```

Mnemonic|Meaning
----|----
\_DSENMR|The observation in the transaction data set does not exist in the master data set (used only with the MODIFY and BY statements).
\_DSEMTR|Multiple transaction data set observations do not exist on the master data set (used only with the MODIFY and BY statements).\_DSENOM|No matching observation (used with the KEY= option).
\_SOK|The observation was located. _SOK has a value of 0.


### 6. 创建完整性约束

* the SQL procedure  
在创建表的同时，也创建完整性约束。也可以在现有表上创建。

* the DATASETS procedure  
  只能在现有表上创建。

```
PROC DATASETS LIB=libref <NOLIST>;	MODIFY SAS-data-set;	IC CREATE constraint-name=constraint		<MESSAGE=‘Error Message’>; 
QUIT;
```

其中IC和INTEGRITY CONSTRAINT等价  

例子：  

创建主键和一个check

```
proc datasets nolist;	modify capinfo;	ic create PKIDInfo=primary key(routeid)		message=’You must supply a Route ID Number’;	ic create Class1=check(where=(cap1st<capbusiness or capbusiness=.))		message=’Cap1st must be less than CapBusiness’;quit;
```
其中创建主键primary等效于UNIQUE and NOT NULL  

### 7. 完整性约束执行的时机  

1. DATA步配合MODIFY。  
1. 在SAS windows上编辑数据。  
1. 含有INSERT INTO, SET, or UPDATE的PROC SQL
1. PROC APPEND.  


### 8. Copying a Data Set and Preserving Integrity Constraints

使用APPEND, COPY, CPORT, CIMPORT, and SORT procedures操作数据的时候，都会保留表的完整性约束。  
在SAS Explorer window操作时，也会。  

### 9. 打印表中的完整性约束  

```
PROC DATASETS LIB=libref <NOLIST>;	CONTENTS DATA=SAS-data-set; 
QUIT;
```

用CONTENTS procedure也可以。  

### 10.删除表中的完整性约束  

```
PROC DATASETS LIB=libref <NOLIST>;	MODIFY SAS-data-set;	IC DELETE constraint-name;QUIT;
```

### 11. Audit Trails

以下情况会记录表改动的audit，将变动记录到一个特殊的表中。  

1. the VIEWTABLE window  
1. the data grid  
1. the MODIFY statement in the DATA step  1. the UPDATE, INSERT, or DELETE statement in PROC SQL.  

注：  

* 任何直接replace数据集的操作（such as the DATA step, CREATE TABLE in PROC SQL, or SORT without the OUT= option)是会自动删除aduit的，所以不会有aduit.  
* aduit trail在SAS是不能用system tool删除的。

aduit trail：
只读文件。
通过PROC DATASETS创建。
必须和要审计的表在同一个lib下面。
和要审计的表有相同的名字，但type是ADUIT。

### 12. Initiating and Reading Audit Trails

指定要审计的dataset(SAS-password可以限定一些操作权限)

```
PROC DATASETS LIB=libref <NOLIST>;	AUDIT SAS-data-set <SAS-password>; 
	INITIATE;RUN;QUIT;
```

Aduit trail就类似一个普通的dataset，读取的时候加上(type=audit)：

```
proc contents data=mylib.sales (type=audit);run;

proc print data=capinfo (type=audit);run;
```

### 13. Controlling Data in the Audit Trail

Audit trail表中有3中VAR：  
data set variables： 要审计表中的所有字段。  
audit trail variables： 记录一些变动的info，都是系统自动VAR。  
user variables： 可以自定义的用户数据。  

![](https://cl.ly/d7750e59d5cd/Image%2525202019-05-26%252520at%25252010.11.57%252520%2525E4%2525B8%25258B%2525E5%25258D%252588.png)  


Audit trail variable|Information stored
----|----\_ATDATETIME\_|date and time of a modification\_ATUSERID\_|login user ID associated with a modification\_ATOBSNO\_|observation number affected by the modification unless REUSE=YES\_ATRETURNCODE\_|event return code\_ATMESSAGE\_|SAS log message at the time of the modification 
\_ATOPCODE\_|code describing the type of operation

其中\_ATOPCODE\_中记录了详细的变动info、原因：  

\_ATOPCODE\_|Event
----|----DA|added data record image 
DD|deleted data record image 
DR|before-update record image 
DW|after-update record image 
EA|observation add failedED|observation delete failed 
EU|observation update failed

利用LOG Statement，设定需要记录的变动info：

* BEFORE_IMAGE=YES|NO controls storage of before-update record images (the ‘DR’ operation).  * DATA_IMAGE=YES|NO controls storage of after-update record images (for example, other operations starting with ‘D’).  * ERROR_IMAGE=YES|NO controls storage of unsuccessful update record images (for example, operations starting with ‘E’).  

### 14. 操作Aduit Trail
有的时候，比如操作大数据的时候，可以先suspend，最后再resume  

```
PROC DATASETS LIB=libref<NOLIST>;	AUDIT SAS-data-set <SAS-password>; 
	SUSPEND | RESUME | TERMINATE;QUIT;
```

### 15. 历史Generation Data Sets

下面例子中，SAS为A复制一个copy，然后从这个copy中读数据，写入数据集a，最后把copy删除掉。

```
data a;
	set a;
	var=XXX;
run;
```
但是可以通过设置，记录每次replace时的历史dataset。  
最新的version叫做base version。  
历史generation和指定表的名字相同，只是带了4个字符的后缀，like #001 (其中001是版本号)  

### 16. Initiating Generation Data Sets

对于新的dataset，或者replace的dataset：

```
data SAS-data-set (GENMAX=n); 
	XXX
RUN；
```

对于已有的dataset，追加：

```
PROC DATASETS LIB=libref <NOLIST>;	MODIFY SAS-data-set (GENMAX=n); 
QUIT;
```

对于上面GENMAX=n中的n:  

* n=0: 不记录历史generation。  
* n>0: 记录历史generation的max数，包括了base version。  

什么时候会创建历史generation:（是replace，不是modify）

1. a DATA step with a SET statement1. a DATA step with a MERGE statement1. PROC SORT without the OUT= option1. PROC SQL with a CREATE TABLE statement.

### 17. 读取、操作指定的历史generation

```
GENNUM=n
```

指定绝对版本(absolute reference):

```
proc print data=year(gennum=4);  run;
```

指定相对版本(relative reference):

```
proc print data=year(gennum=-1);  run;```

指定的dataset为A。  
第一次replace的版本为A#001  
第二次replace的版本为A#002  

相对版本号：

* 0表示base version，也就是current version。  
* -1表示最近的一次的replace的version。  

例子：

```
版本号是一直往上递增的。  
比如： GENMAX=3
A#001  第一次replace  
A#002  第二次replace  
A#003  第三次replace  
A#004  第四次replace  
但是因为只保留3个，所有系统中只有A#002，A#003，A#004(base version)  
如果这个时候修改了GENMAX=2，那就只保留2个：A#003，A#004
```

### 18. 操作历史generation

```
PROC DATASETS LIB=libref <NOLIST>;	CHANGE SAS-data-set<(GENNUM=n)>=new-data-set-name; 
	DELETE SAS-data-set<(GENNUM=n | HIST | ALL)>;QUIT;
```

如果dataset的名字修改了，它的历史generation的名字也同时修改：  

```
proc datasets library=quarter1 nolist;     change salesData=sales;quit;
```

修改指定历史版本generation的名字：

```
proc datasets library=quarter1 nolist;     change sales(gennum=2)=newsales;quit;
```

删除所有历史版本generation：

```
proc datasets library=quarter1 nolist;     delete newsales(gennum=HIST);quit;
```
删除所有generation，包括base version：

```
proc datasets library=quarter1 nolist;     delete newsales(gennum=ALL);quit;
```

## Chapter 19 - Introduction to Efficient SAS Programming

一堆方法论.....

一些用来监测的options

Option|UNIX and Windows
----|----
STIMER|pecifies that CPU time and real-time statistics are to be tracked and written to the SAS log throughout the SAS session. Can be set either at invocation or by using an OPTIONS statement.Is the default setting.
MEMRPT|Not available as a separate option;this functionality is part of the FULLSTIMER option.
FULLSTIMER|Specifies that all available resource usage statistics are to be tracked and written to the SAS log throughout the SAS session.Can be set either at invocation or by using an OPTIONS statement.In Windows operating environments, some statistics will not be calculated accurately unless FULLSTIMER is specified at invocation.
STATS|Not available as a separate option.



## Chapter 20 - Controlling Memory Usage

### 1. 查看Page Size
可以使用PROC CONTENTS或者DATASETS中的CONTENTS语句来查看。  

### 2. BUFSIZE=
设置BUF的大小。 
使用BUFSIZE=0可以reset  

### 3. BUFNO=
设置可以用来读写dataset的buffer的数量。  

除了在options中设置外，还可以在data步中直接设置：  

```
data work.orders (bufsize=12288 bufno=10);   set retail.order_fact;run;
```

### 4. Using the SASFILE Statement

可以将一个dataset（业务logic中经常会用到的）读入内存，然后常驻内存，一直到：

* 使用SASFILE CLOSE关闭。  
* 程序结束运行，SAS自动释放。  

```
SASFILE SAS-data-file <(password-option(s))> OPEN | LOAD | CLOSE;
```

* OPEN: 打开文件，分配内存，但不读入数据，等logic需要。  * LOAD: 打开文件，分配内存，读数据入内存。  * CLOSE: 关闭文件，释放内存。  


例子：

```
sasfile retail.medium load;proc print data=retail.medium;   where cm=100;   var Customer_Age_Group;run;proc tabulate data=retail.medium;   class Customer_Age_Group;   var Customer_BirthDate;   table Customer_Age_Group,Customer_BirthDate*(mean median);run;proc means data=retail.medium;   var Customer_Age;   class Customer_Group;   output out=summary sum=;run;proc freq data=retail.medium;   tables Customer_Country;run;   sasfile retail.medium close;
```

注：  
如果一个dataset很大，但不是里面所有的数据都频繁被使用，可以读取里面部分数据，常驻内存。  


### 5. IBUFSIZE=
可以专门为Index设置buffer大小。  
如果：  

* Index中有很多Level。  
* 或者Index的长度很大。



## Chapter 21 - Controlling Data Storage Space

### 1. SAS分配char var的长度  
如果没有定义的话，以第一次出现的长度为准。  
如果第一次出现没有长度，就使用默认长度8.  

### 2. 定义char长度

需要在var出现前定义，如果var已经出现了，PDV中已经有长度了，length statement就没有作用了。

```
LENGTH variable(s) $ length;
```

### 3. 减少char var的空间  

normalizing the data -- 对于经常使用，但属于展示型的var，可以用code代替，再创建一张mapping表，将code和真实的长的名字对应起来。


### 4. SAS怎么存储numeric
mainframe的定义(共8 byte)

* bit 1: sign  
* bit 2-8: exponent  
* bit 9-64: mantissa  


### 5. 定义numeric长度

```
LENGTH variable(s) length <DEFAULT=n>;
```

* 定义长度2到8（最小值是2是3，取决于操作系统）  
* 定义的长度值影响output，在内部计算的时候，永远都是8
* 定义的长度如果小于8，可能在dataset中处理的时候导致trunc


例子：

```
data reducedsales;     length default=4;     set sales;     Sale_Percent=15;run;
```
其中length default=4;只影响在当前data步的numeric var。所有的numeric var的长度都是4.

### 6. 对numeric的影响

建议如果存储的不是整数的话，就不要修改numeric的长度，这样会影响浮点数的精度。  
即使是整数，减少numeric的长度，也有可能会影响存储的最大值。

### 7. Using PROC COMPARE

```
PROC COMPARE BASE=SAS-data-set-one COMPARE=SAS-data-set-two;RUN;
```

1. 先使用默认的长度生成benchmark的数据集。  
1. 再减少长度，再生成对照的数据集。
1. 再使用PROC COMPARE，看看output中Maximum difference是不是可以接受（减少的精度）。  

### 8. Compressing Data Files
* 存储的数据集如果是压缩后的数据，可以节省空间，读取的时候，再解压缩即可，但解压缩的过程可能会提高CPU。  
* 在某些情况下，压缩的数据有可能并不能节省空间。  

#### 8.1 未压缩的数据

* 每个VAR占（分配）相同的空间。
* 每个ROW占（分配）相同的空间。
* char var用空格补齐。
* numeric var用二进制0补齐。
* 每一个page前有16-byte空白。
* 每个page的最后，有给在此page的所有row有1-bit的标志位，用来标志在此page中，这条row是否被删除。* 新增的row在file的最后。* 如果新的row在当前page不够了，整个新的row全部都放在新增的page中（单条row的数据不跨page）* descriptor portion存储在第一个page的最后部分。

![](https://cl.ly/e170ed612420/Image%2525202019-05-28%252520at%2525206.17.26%252520%2525E4%2525B8%25258B%2525E5%25258D%252588.png)


#### 8.2 Compressed Data File Structure

* 把单条ob当成single string of bytes，忽略var类型和边界。
* 将连续的、重复的char进行折叠。  
* 每页开头的OH变成28-byte。    
* contain a 28-byte overhead at the beginning of each page.  
* 每页给一个12或者24（具体看操作系统）OH给当前页的OB，用来标记删除等信息。


![](https://cl.ly/28ea46ebe284/Image%2525202019-05-28%252520at%2525206.26.43%252520%2525E4%2525B8%25258B%2525E5%25258D%252588.png)  
注：  
每页的ob长度变得不一样。但还是不跨page。  


#### 8.3 决定是否要压缩

* 很大
* 有很多long character values
* 有没有重复的char，或者二进制0
* 有很多missing values
* 有很多重复的value，在物理上存储在一起的

#### 8.4 COMPRESS= 

在options中设置，对整个session中创建的数据集都压缩（包括临时lib下面的）：  

```
OPTIONS COMPRESS= NO | YES | CHAR | BINARY;
```
指定数据集创建时压缩：

```
DATA SAS-data-set (COMPRESS= NO | YES | CHAR | BINARY);
```

选项：  

> YES or CHAR： 对多次连续重复的char(单个char)很有效果。对数字有多个二进制0重复也有效果。    
> BINARY： 对于large blocks of binary data (numeric variables)  , 比如一条ob超级长的(observation length is several hundred bytes or more)  ，也可以是2个或者3个char的pattern重复，BINARY can also be very effective with character data that contains patterns rather than simple repetitions.
> BINARY要比Yes or CHAR相对多占CPU。


注：  

* 使用COMPRESS=的时候，SAS会自动根据实际情况判断，如果没有必要，不会进行压缩，同时在log提示。  
* 一旦表被压缩了，是永久的属性。怎么重复读写都是压缩的表，除非re-create。
* 单单针对表中的ob进行压缩的功能是不存在的。


### 9. POINTOBS= in Data Set Option

有时候可以通过point=n，来直接访问指定行的数据（SAS默认是从开头顺序一行一行读取的）， 但是在compressed表中进行直接查询，非常消耗CPU。  
可以在dataset创建时，关闭这个功能，即不能使用point=来读取：  

```
DATA SAS-data-set (POINTOBS= YES | NO);
```

例子：

```
data company.customer_compressed (compress=yes pointobs=no);     set company.customer;run;
```

### 10. REUSE=
SAS默认删除ob后的磁盘空间不重复利用，还是空着的，后续新增的ob在最后面写入。可以通过REUSE=来重复利用，节省空间。


系统级：

```
OPTIONS REUSE= NO | YES;```
数据集级：

```
DATA SAS-data-set (COMPRESS=YES REUSE=NO | YES);
```

注：    

* 当一个压缩的dataset创建后，就不能再修改这个dataset的reuse属性了（最后创建的时候就想好） 
* 仅对压缩数据集可用 
* 即使创建时指定了REUSE=YES，PROC APPEND还是将新增的ob写在最后面。  
* 指定了REUSE=YES后，自动将POINTOBS=YES失效，默认将POINTOBS=NO，即reuse后就不能通过指定行查找，必须顺序查找。  

例子：

```
data company.customer_compressed (compress=yes reuse=yes);     set company.customer;run;
```
### 11. Data Set View视图

* 数据集中存储数据，view中没有数据  
* 如果有一个大的raw文件，存储在数据集中，如果文件内容修改了，需要重新读入或者修改数据集，但是可以直接在raw文件上创建view，中间跳过了数据集。即使raw文件中改变，view也可以直接反映出来。  
* view不占太多空间，但运行时占CPU  
* 创建view时不执行，调用的时候，读取数据

data set view可以创建在：  

* raw data files  * SAS data files  * PROC SQL views  * SAS/ACCESS views  * DB2, ORACLE, or other DBMS data  



```
DATA SAS-data-view <SAS-data-file-1 ... SAS data-file-n> / VIEW=SAS-data-view;<SAS statements> RUN;
```

#### 11.1 The DESCRIBE Statement可以通过describe语句，将view中的数据在log中输出一份copy```
data view=company.newdata;     describe;run;
```

#### 11.2 在一个程序中频繁调用一部分数据  
可以优先考虑使用临时表，而不是view。  
可以将部分数据通过where或者其他筛选条件创建成view，但每次调用都相当于对一张大表执行了多次筛选条件，效率不高。  

#### 11.3 Making Multiple Passes Through

有些proc或者操作，对数据进行多次的pass through（不是简单的一次读取、处理、输出），这样的话，SAS会把所有数据放在spill cache中，所以使用view的话，效率不高。

#### 11.4 Creating Data Views on Unstable Data

如果有些表的字段经常变来变去，尤其是其他的表会关联过来，这样的话，就不适合用view。 比如，merge的时候，by的var需要一样，有一个张表的var修改了，view中的merge语句要手动修改。  

#### 11.5 结论

* Create a SAS DATA step view to avoid storing a raw data file and a copy of that data in a SAS data file.    
* Use a SAS DATA step view if the content, but not the structure, of the flat file is dynamic  


## Chapter 22 - Utilizing Best Practices

### 1. 只执行有必要的语句

#### 1.1 在logic上尽可能早的subset你的数据
注意下面例子中的if语句，要放在前面，尽可能早的if

方法一：

```
data profit;
   retain TotalProfit TotalDiscount TotalWait Count 0;
   set retail.order_fact;
   MonthOfOrder=month(order_date);
   WaitTime=sum(delivery_date,-order_date);
   if discount gt . then
      CalcProfit=sum((total_retail_price*discount),-costprice_per_unit)*quantity;
   else 
   		CalcProfit=sum(total_retail_price,-costprice_per_unit)*quantity;
   
   TotalProfit=sum(totalprofit,calcprofit);
   TotalDiscount=sum(totaldiscount,discount);
   TotalWait=sum(totalwait,waittime);
   Count+1;
   if monthoforder=12;
run;
```
方法二：

```
data profit;
   retain TotalProfit TotalDiscount TotalWait Count 0;
   set retail.order_fact;
   MonthOfOrder=month(order_date);
   if monthoforder=12;
   WaitTime=sum(delivery_date,-order_date);
   if discount gt . then
      CalcProfit=sum((total_retail_price*discount),-costprice_per_unit)*quantity;
   else 
   		CalcProfit=sum(total_retail_price,-costprice_per_unit)*quantity;
   
   TotalProfit=sum(totalprofit,calcprofit);
   TotalDiscount=sum(totaldiscount,discount);
   TotalWait=sum(totalwait,waittime);
   Count+1;
run;
```
#### 1.2 有效的利用判断的logic

判断语句有IF-ELSE和SELECT：  

Use IF-THEN/ELSE statements when

* 判断的值都是char。  
* 值的分布是不均匀分布。  
* 检查的条件不是很多。

需要注意的地方：

* 尽量使用if和else if的配合语句，比连续使用多个if要好。  
* 先判断高频的值（很有可能出现的值）
* then后面如果有多个执行语句，要用DO和END包含起来。

Use SELECT statements when

* 判断的值都是numeric。  
* 值的分布是均匀分布。 

对要判断的数据，先使用FREQ、GCHART、GPLOT、UNIVARIATE观察频率。

#### 1.3 如果多次需要某个函数的结果，先将函数的结果赋值给变量。

例子：

* 多次调用相同的函数
* 多次连续使用if语句，而不是if-else if

```
data retail.orders;   set retail.order_fact;
   if month(order_date)=1 then Month=’Jan’;	if month(order_date)=2 then Month=’Feb’;	if month(order_date)=3 then Month=’Mar’;	if month(order_date)=6 then Month=’Jun’;	if month(order_date)=7 then Month=’Jul’;	if month(order_date)=8 then Month=’Aug’;	if month(order_date)=9 then Month=’Sep’;	if month(order_date)=10 then Month=’Oct’;	if month(order_date)=11 then Month=’Nov’;	if month(order_date)=12 then Month=’Dec’;run;
```

比上面的例子要好些，但还有多次调用同一个函数的问题。  

```
data retail.orders;   set retail.order_fact;
   if month(order_date)=1 then Month=’Jan’;	else if month(order_date)=2 then Month=’Feb’;	else if month(order_date)=3 then Month=’Mar’;	else if month(order_date)=4 then Month=’Apr’;	else if month(order_date)=5 then Month=’May’;	else if month(order_date)=6 then Month=’Jun’;	else if month(order_date)=7 then Month=’Jul’;	else if month(order_date)=10 then Month=’Oct’;	else if month(order_date)=11 then Month=’Nov’;	else if month(order_date)=12 then Month=’Dec’;run;
```

比上一个例子要好一些，将函数的结果赋值给var， 但数值型var的判断最好用select

```
data retail.orders(drop=mon);
	set retail.order_fact;
	mon=month(order_date);
	if mon=1 then Month=’Jan’;
	else if mon=2 then Month=’Feb’;
	else if mon=3 then Month=’Mar’;
	else if mon=4 then Month=’Apr’;
	else if mon=5 then Month=’May’;
	else if mon=6 then Month=’Jun’;
	else if mon=7 then Month=’Jul’;
	else if mon=8 then Month=’Aug’;
	else if mon=9 then Month=’Sep’;
	else if mon=10 then Month=’Oct’;
	else if mon=11 then Month=’Nov’;
	else if mon=12 then Month=’Dec’;
run;
``` 
比之前的所有例子都要好，对数值var用select，将多次使用的函数值先赋值给var  

```
data retail.orders;
	set retail.order_fact;
	select(month(order_date));
		when (1) Month=’Jan’;
		when (2) Month=’Feb’;
		when (3) Month=’Mar’;
		when (4) Month=’Apr’;
		when (5) Month=’May’;
		when (6) Month=’Jun’;
		when (7) Month=’Jul’;
		when (8) Month=’Aug’;
		when (11) Month=’Nov’;
		when (12) Month=’Dec’;
		otherwise;
	end;
run;
```
#### 1.4 不要过多使用if，消耗资源

还可以：

```
data retail.customers;
	length Customer_Group $ 26 Customer_Activity $ 15;
	set retail.customer_hybrid;
	select(substr(put(customer_type_ID,4.),1,2));
		when (’10’)
			do;
				customer_group=’Orion Club members’;
				select(substr(put(customer_type_ID,4.),3,2));
					when (’10’) customer_activity=’inactive’;
					when (’20’) customer_activity=’low activity’;
					when (’30’) customer_activity=’medium activity’;
					when (’40’) customer_activity=’high activity’;
					otherwise;
				end;
			end;
		when (’20’)
		do;
			customer_group=’Orion Club Gold members’;
			select(substr(put(customer_type_ID,4.),3,2));
				when (’10’) customer_activity=’inactive’;
				when (’20’) customer_activity=’low activity’;
				when (’30’) customer_activity=’medium activity’;
				when (’40’) customer_activity=’high activity’;
				otherwise;
			end;
		end;
		when (’30’) customer_group=’Internet/Catalog Customers’;
		otherwise;
	end;
run;
```

不是很好，比较占用资源  

```
data retail.customers;
	length Customer_Group $ 26 Customer_Activity $ 15;
	set retail.customer_hybrid;
	if substr(put(customer_type_ID,4.),1,2)=’10’ then
		customer_group=’Orion Club members’;
	if substr(put(customer_type_ID,4.),1,2)=’20’ then
		customer_group=’Orion Club Gold members’;
	if substr(put(customer_type_ID,4.),1,2)=’30’ then
		customer_group=’Internet/Catalog Customers’;
	if substr(put(customer_type_ID,4.),1,2) in (’10’, ’20’) and
		substr(put(customer_type_ID,4.),3,2)=’10’ then
			customer_activity=’inactive’;
	if substr(put(customer_type_ID,4.),1,2) in (’10’, ’20’) and
		substr(put(customer_type_ID,4.),3,2)=’20’ then
			customer_activity=’low activity’;
	if substr(put(customer_type_ID,4.),1,2) in (’10’, ’20’) and
		substr(put(customer_type_ID,4.),3,2)=’30’ then
			customer_activity=’medium activity’;
	if substr(put(customer_type_ID,4.),1,2) in (’10’, ’20’) and
		substr(put(customer_type_ID,4.),3,2)=’40’ then
			customer_activity=’medium activity’;
run;
```
 
### 2. 去掉不必要的Passes through数据

#### 2.1 读取一次数据，output多个数据

相比于写多个data set，要有效率，因为他只需要读取一次数据

```
data Sales_managers Account_managers Finance_managers;	set company.organization;	if job_title=’Sales Manager’ then		output Sales_managers;	else if job_title=’Account Manager’ then		output Account_managers;	else if job_title=’Finance Manager’ then		output Finance_managers;run;
```

#### 2.2 sort时用where（根据具体logic）

如果在logic上需要用where先截取部分数据，然后再将这部分数据sort的话，可以合并成一个proc，比在data set中where，再sort要有效率（不用创建临时表）

```
proc sort data=company.organization		out=company.managers;	by job_title;	where job_title in(’Sales Manager’,			’Account Manager’,			’Finance Manager’);run;
```

#### 2.3 在PROC DATASETS中修改表中的属性

在data step中也可以修改表中的属性，但需要占用更多的资源，因为读取的不只是descriptor portion，也读取了具体的数据。  
而PROC DATASETS是读取descriptor portion，然后进行修改。

```
proc datasets lib=company;     modify organization;     rename Job_title=Title;quit;
```
注：  
不能在PROC DATASETS中修改var的type，长度，和具体位置。因为上述属性直接影响到具体的数据。所以还是需要用DATA STEP修改。

### 3. 只读和写Essential Data

#### 3.1 WHERE和IF区别

![](https://cl.ly/36b4b8a6a67e/Image%2525202019-05-29%252520at%2525209.20.59%252520%2525E4%2525B8%25258B%2525E5%25258D%252588.png)

* WHERE和IF都对INPUT的I/O没有影响，但对OUTPUT有影响。
* WHERE在从INPUT的buffer读取数据，进行筛选，然后才放入PDV。
* IF等SAS把数据放入PDV之后，再进行筛选，然后才放入output的buffer中。
* WHERE只能处理SAS存在的VAR（因为where还没有等到PDV的步骤）。  
* IF能处理所有PDV中存在的var，包括数据集的VAR。
* WHERE相对IF更有效率。
* IF除了筛选功能，还有logic判断功能（WHERE没有）
* when Grouping Data Using a BY Statement: IF - "produces the same data set when a BY statement accompanies a SET, MERGE, or UPDATE statement.", WHERE - "produces a different data set when a BY statement accompanies a SET, MERGE, or UPDATE statement."

#### 3.2 WHERE和OBS= and FIRSTOBS一起

* where语句在firstobs和obs之前起作用。
* firstobs和obs读取的是按照logic上的序号，不是文件上物理的序号，是从where的subset里面读取的。


```
proc print data=company.organization(firstobs=5 obs=8);   var employee_id employee_gender salary;   where salary>40000;run;
```

#### 3.3 从外部文件读部分数据

方法一： 读取所有row的信息，到PDV，然后再判断，是否要写到output中。没有效率。

```
data retail.UnitedKingdom;   infile customerdata;   input @1   Customer_ID         12.         @13  Country             $2.         @15  Gender              $1.         @16  Personal_ID        $15.         @31  Customer_Name      $40.         @71  Customer_FirstName $20.         @91  Customer_LastName  $30.         @121 Birth_Date       date9.         @130 Customer_Address   $45.         @175 Street_ID           12.         @199 Street_Number       $8.         @207 Customer_Type_ID     8.;    if country=’GB’;run;
```

方法二： 只读一部分数据到PDV，然后判断，有需要就读全部的信息，没有需要，就跳出这条ob，不output

```
data retail.UnitedKingdom;   infile customerdata;   input @13 Country $2. @;   if country=’GB’;   input @1   Customer_ID         12.         @15  Gender              $1.         @16  Personal_ID        $15.         @31  Customer_Name      $40.         @71  Customer_FirstName $20.         @91  Customer_LastName  $30.         @121 Birth_Date       date9.         @130 Customer_Address   $45.         @175 Street_ID           12.         @199 Street_Number       $8.         @207 Customer_Type_ID     8.;run;
```

#### 3.4 使用KEEP和DROP进行subset

![](https://cl.ly/9387438ae3ea/Image%2525202019-05-29%252520at%25252010.09.11%252520%2525E4%2525B8%25258B%2525E5%25258D%252588.png)

* 在SET或者MERGE语句中的KEEP/DROP的option，对input的buffer进行筛选，将筛选后的数据写入PDV。  
* 在data步的KEEP/DROP的option，和data步中的KEEP/DROP语句，是在PDV之后，进行筛选，将筛选后的数据写入output。  


方法一： 没有subset，处理全部的数据，效率差

```
data retail.profit;
	set retail.order_fact;
	if discount=. then
		Profit=(total_retail_price-costPrice_Per_Unit)*quantity;
	else Profit=((total_retail_price*discount)-costprice_per_unit)*quantity;
run;
proc means data=retail.profit mean median maxdec=2;
	title ’Order Information’;
	class employee_id;
	var profit;
run;
```

方法二： 在处理后的output数据集中，只keep了后续means需要的2个字段，有效率。

```
data retail.profit(keep=employee_id profit);   set retail.order_fact;   if discount=. then      Profit=(total_retail_price-costprice_per_unit)*quantity;   else Profit=((total_retail_price*discount)-costprice_per_unit)*quantity;run;proc means data=retail.profit mean median maxdec=2;   title ’Order Information’;   class employee_id;   var profit;run;
```

方法三： 比方法二更有效率，在处理output的时候，set的时候，只读取了logic需要的部分var，不是全部。

```
data retail.profits(keep=employee_id profit);   set retail.order_fact(keep=employee_id total_retail_price discount                              costprice_per_unit quantity);      if discount=. then         Profit=(total_retail_price-costprice_per_unit)*quantity;      else Profit=((total_retail_price*discount)-costprice_per_unit)*quantity;run;proc means data=retail.profit mean median maxdec=2;   title ’Order Information’;   class employee_id;   var profit;run;
```
方法四：在logic上更有兼容性。在处理output的logic上，set的时候，只读取logic上需要的部分var。在output的数据集上没有subset，而是在means上进行subset，是为了output的数据集在业务可能被其他场景使用，所有保留了所有的var。

```
data retail.profit;  set retail.order_fact(keep=employee_id total_retail_price discount                             costprice_per_unit quantity);  if discount=. then     Profit=(total_retail_price-costprice_per_unit)*quantity;  else Profit=((total_retail_price*discount)-costprice_per_unit)*quantity;run;proc means data=retail.profit(keep=employee_id profit) mean median maxdec=2;   title ’Order Information’;   class employee_id;   var profit;run;
```

### 4. 将数据存储在data set中（不是重复读取）

* 如果程序需要经常处理部分数据，更好的方式是将数据存储在data set，不是每次都重新读取。
* 在数据存储在data set中，还可以使用更多功能，比如merge

### 5. 避免不必要的PROC调用

最好使用一个或者比较少的PROC处理尽可能多的功能，避免没有效率的堆砌使用众多PROC。

#### 5.1 Executing the DATASETS Procedure

```
PROC DATASET .....
	<process1...>
	<process1...>
	<process1...>
	run;
	
	<process2...>	
	<process2...>
	<process2...>
	run;
quit;
```

* 读取statement，一直到run语句，或者隐式RUN。
* 执行刚才读取的statement
* 重复上述过程，一直到quit，或者下一个data步，或者proc步。

#### 5.2 支持RUN-Group Processing的PROC

* CHART, GCHART 
* PLOT, GPLOT 
* GIS, GMAP* GLM* REG
* DATASETS.

#### 5.3 导致隐式的RUN（以datasets为例）

* PROC DATASETS这个单个语句就是一个隐式run，读完这个直接执行。
* MODIFY和它的子句，读完直接执行。
* APPEND, CONTENTS, and COPY，读完直接执行。
* AGE，EXCHANGE，CHANGE，REPAIR，读完直接执行。

但是上面这些同时和他们之前的语句形成run group，也就是说，如果他们这些语句之前还有一些语句没有被执行，就和之前的并在一起，等着一起执行，就不是读完直接执行了。

下面是一个简单的例子：

```
SAVE XXXXX  --- 这句不是隐式run，先不执行
REPAIR  ----这个是隐式RUN，但前面的不是，所以和前面的SAVE并在一起，先不执行
SAVE XXX --- 读取，但不执行
RUN;或者quit;或者数据步、proc步-----执行之前的RUN GROUP
```


## Chapter 23 - Selecting Efficient Sorting Strategies


### 1. Avoiding Unnecessary Sorts


#### 1.1 Using BY-Group Processing with an Index

如果表中有index，又能满足你的要求，就不用sort，可以直接by index。  
注：  

* 需要额外的空间来存储index  
* by中使用index，相对来说，没有by sorted数据有效率。
* 如果by中有DESCENDING or NOTSORTED的options，不能使用index（因为index一定是升序的）  
* MODIFY语句不用sort，但有sorted的数据会好一些。

结论：

1. 为了节省资源，尽量使用sort（如果index没有非必要的时候）  
1. 但有些时候，资源紧张，就不用sorted数据，可以直接使用index。

#### 1.2 Using the NOTSORTED Option

原理：

不用排序，但是NOTSORTED会按照顺序将紧挨在一起的var进行group，例如下面的数据比较适合（不一样是按照alpha排序）

```
zxc
zxc
abc
abc
edf
edf
jkl
```
但下面的数据就不适合：

```
zxc
abc
zxc
abc
edf
jkl
edf
```
比较适合，数据已经按照var group好了的，但对数据又没有按照alpha排序的要求。

注：

* NOTSORTED不能在MERGE或者UPDATE语句中使用。

例子：

```
proc print data=retail.europe; by country_name notsorted;run;```


#### 1.3 Using FIRST. and LAST.
可以配合BY一起使用，然后再logic上判断。

#### 1.4 Using the GROUPFORMAT Option

将var的值，先format，然后根据format后的值进行by，不是按照数据集中var的真实的值进行by。可以配合FIRST.variable and LAST.variable一起使用。

```
BY variable(s) GROUPFORMAT (NOTSORTED);
```

注：

* GROUPFORMAT只能使用在data步中。  
* 非常适合有format数据的需求。  
* 后面可以根据需要增加NOTSORTED。没有NOTSORTED就表示数据集中的数据必须要按照format的值先sort。如果NOTSORTED，就表中数据集中的数据最好（非必须）按照format的值是group（紧挨着）的。  

例子：  
要定义自定义的format，然后在data步中先关联format。

```
proc format;   value qtrfmt ’01jan2002’ d - ’31mar2002’ d = ’1’                ’01apr2002’ d - ’30jun2002’ d = ’2’                ’01jul2002’ d - ’30sep2002’ d = ’3’                ’01oct2002’ d - ’31dec2002’ d = ’4’;run;data company.quarters(keep=count order_date           rename=(order_date=Quarter));   set retail.orders;   format order_date qtrfmt.;   by  order_date groupformat notsorted;   where year(order_date)=2002;   if first.order_date then Count=0;   Count +1;   if last.order_date;run;
```

#### 1.5 Using the CLASS Statement

class不需要sort，也不需要index （pre-sort对class语句没有明显的帮助,没必要presort）  

可以在下面的PROC使用CLASS：

* MEANS 
* TABULATE 
* SUMMARY 
* UNIVARIATE

class的var可以是numeric，也可以是char。   
class的var的值可以是连续的，但从统计上看，最好是一些离散的。  

#### 1.6 Using the SORTEDBY= Data Set Option

如果外部数据或者其他的数据是已经排序的，在data set读取的时候，可以加上SORTEDBY  

```
SORTEDBY=by-clause </collate-name> | _NULL_
```

![](https://cl.ly/db5658507e1c/Image%2525202019-05-30%252520at%2525205.37.22%252520%2525E4%2525B8%25258B%2525E5%25258D%252588.png)

注：

* SORTEDBY可以修改Sorted flag，但是不能修改sort information里面的Validated sort flag（proc sort才可以。）
* SORTEDBY=_NULL_可以去掉Sorted flag

#### 1.7 Using a Threaded Sort(多线程)
利用多线程sort。

system option级别：

```
options THREADS | NOTHREADS
```

proc sort级别：

```
PROC SORT SAS-data-set-name THREADS | NOTHREADS;
```

#### 1.8 Using the CPUCOUNT= System Option
指定可用CPU的个数，SAS会根据指定thread数量

```
CPUCOUNT= n | ACTUAL;
```

注：

* CPUCOUNT= ACTUAL表示SAS会自动检测CPU个数。
* 如果手动指定n超过了实际的CPU个数，可能反而会影响性能。


### 2. Calculating and Allocating Sort Resources

#### 2.1 计算sort时需要的空间
在sort时，SAS会copy2份原数据，另外需要2倍至4倍的数据大小的workplace。  （在SAS9以前的版本，需要3到4倍，在SAS9以后的版本需要2倍）

计算公式：

```
bytes required = (key-variable-length + observation-length)		* number-of-observations * 4
```

#### 2.2 Using the SORTSIZE= Option

可以在options中，或者proc sort时，指定分配给sort的空间：

```
SORTSIZE=memory-specification;
```

注：

默认的SORTSIZE的大小取决于具体的操作系统。  
如果分配的大小 大于 sort需要的空间大小，则会很有效率，一次在内存中处理完成。  
如果分配的大小 小于 sort需要的空间大小，很影响效率，SAS会多次在内存中处理，再合并。

### 3 Handling Large Data Sets

#### 3.1 拆分大的dataset
如果一个dataset非常大，sort的空间不够用，可以将大的dataset拆分成多个小的dataset，然后依次sort小的dataset，最后再合并。

拆分大的dataset可以用：

* 在data步中使用FIRSTOBS= + OBS= + OUTPUT  
* 在proc sort中使用WHERE  
* 在data步中使用IF-THEN/ELSE or SELECT-WHEN输出多个数据集

合并多个小的dataset可以用：

* 使用set dataset1 dataset2进行顺序合并。
* 使用set dataset1 dataset2 + by var进行合并。
* 用proc APPEND进行合并。


#### 3.2 Using the TAGSORT Option

原理：
sort的时候，只sort by中的一个或者多个var，其他的数据先不处理，在临时表中间by中的var排序完了最后，再关联ob中其他字段，最后再输出。  
把by中的var看做tag，首先，只排序tag。


```
PROC SORT DATA=SAS-data-set-name TAGSORT;
```
注：

* 会造成CPU和IO占用的提高。
* 当数据中大部分都是乱序的时候，使用tagsort会明显影响CPU和IO。
* 当数据中小部分都是乱序的时候，使用tagsort会稍微影响一点CPU和IO。
* TAGSORT不能和多线程THREAD排序一起使用。


### 4. Removing Duplicate Observations Efficiently

sort可以去除重复的数据：

* used with the NODUPKEY option
* used with the NODUPRECS option
* followed by FIRST. processing in the DATA step

一般使用NODUPKEY会比使用FIRST.var占用少的IO和CPU。

#### 4.1 Using the NODUPKEY Option

会判断by中所有的var，如果完成相同，后面的数据算作重复数据，不输出。不会判断其他的var。按照顺序。

```
PROC SORT DATA=SAS-data-set-name NODUPKEY;
```

例子： by a b;

a|b|c
----|----|----
1|10|202|10|201|10|30

如果带上NODUPKEY的输出结果为：

a|b|c
----|----|----
1|10|202|10|20#### 4.2 Using the NODUPRECS/NODUP Option

和NODUPKEY类似，但检查的ob中所有的var，不仅仅是by中的要求排序的var。 但是由于是按照by中的var进行的排序，如果by中的var相同，ob还是按照原来的顺序输出，如果连续紧挨着两个ob其他var也相同，就算重复；但如果间隔重复了，不算。

```
PROC SORT DATA=SAS-data-set-name NODUPRECS;
```例子1 ：by a b;

a|b|c
----|----|----
1|10|202|10|201|10|20

算作重复，因为按照a和b排序后，两条ob是紧挨着的。

a|b|c
----|----|----
1|10|202|10|20

例子2 ：by a b;

a|b|c
----|----|----
1|10|202|10|201|10|30
1|10|20

不算作重复，因为按照a和b排序后，两条ob是不是紧挨着的。

a|b|c
----|----|----
1|10|202|10|201|10|30
1|10|20

#### 4.3 Using the EQUALS | NOEQUALS Option

EQUALS（默认）: 保证原来的顺序，去掉重复的，除了第一条ob外，其他的都算是重复数据。

NOEQUALS： 不保证顺序，去掉的重复数据都是随机的（具体看SAS内部算法，但测试了多次，结果都是随机的）

例子1 ：nodupkey noequals by a b;

a|b|c
----|----|----
1|10|202|10|201|10|30
1|10|20

每次修改dataset后，得到的结果都不一样。


#### 4.4 Using FIRST. LAST. Processing in the DATA Step

```
proc sort data=company.onorder          out=work.sorted;   by product_name;run;
data work.onorder2;   set work.sorted;
   by product_name; if first.product_name;run;
```

#### 4.5 性能比较

* 使用sort nodupkey要比使用first.var要好。
* 使用noequals更快，但要注意equals和noequals的结果可能不一样。

 
### 5. 一些options
SAS在sort时，默认使用的是SAS sort utilities。 在某些情况下，使用操作系统自己的Host Sort Utility可能更有效率。

#### 5.1 Using the SORTPGM= System Option

```
OPTIONS SORTPGM= BEST | HOST | SAS;
```

BEST: SAS自己选择最好最适合的。
HOST: 使用操作系统的。
SAS: 使用SAS自己的。

#### 5.2 Using the SORTCUTP= System Option
当使用HOST自己的utilities时，需要设置的参数：

```
OPTIONS SORTCUTP=n / nK / nM / nG / MIN / MAX / hexX;
```

#### 5.3 Using the SORTCUT= System Option
当使用HOST自己的utilities时，需要设置的参数：

```
OPTIONS SORTCUT=n | nK | nM | nG | MIN | MAX | hexX;```
#### 5.4 Using the SORTNAME= System Option
告诉SAS要使用HOST的哪一个sort引擎。

```
OPTIONS SORTNAME=host-sort-utility name;
```

## Chapter 24 - Querying Data Efficiently

### 1. Using an Index for Efficient WHERE Processing

要善用index，有时候index不一样有好性能。  

BY也可以使用index，by var的时候，SAS会自动先检查表中(contents)的sorted flag, 如果为false，SAS再看有没有index，使用哪一个index。

#### 1.1 Benefits and Costs of Using an Index

index的好处：

* 快速定位。
* 返回的数据（如果是多条）是按照顺序的。
* 可以保证唯一性。

index的代价：

* 需要额外的CPU和IO创建、维护Index。
* 需要额外的CPU和IO去使用Index读取数据。
* 需要额外的磁盘空间存放Index。* 需要额外的内存去转载index的page。

当一个数据集打开的时候，如果有index，先打开index文件，同时会给index分配一个的buffer大小，这时没有具体的buffer给Index。等确定coding中要使用index的时候，再给具体的buffer（使用时的buffer是三倍大小，update时时四倍大小）  

#### 1.2 How SAS Selects an Access Method


1. identifies available indexes1. identifies conditions that can be optimized1. estimates the number of observations that qualify1. compares probable resource usage for both methods.

### 2. Identifying Available Indexes

当WHERE从句满足下面条件中的任何一个，就可以使用index optimize：  

* where中的var在表中有simple index。
* where中的var在表的联合index是first key variable。

注：

* 无论表中有多少index，where从句中有多少条件，SAS每次都只能使用一个Index。
* 当where时，表中没有index，顺序。
* where中的var在表中只有一个index，SAS会评估是否要使用Index，如果顺序查找的性能好的话，还是不会使用Index。
* where中有多个var在表中都分别有Simple Index，SAS只能使用一个，SAS会评估。
* where中的一个或者多个var在表中有联合Index，如果满足条件的话，会使用联合Index。


**例子一：**

SAS 语句：

```
where order_date=’01jan2000’d and delivery_date=’02jul2000’d’;
```
表中已有的Index：

```
composite index defined on the following variables:Order_Date (first key variable) 
Delivery_Date (second key variable) 
Product_ID (third key variable)
```
优化，使用Index。

<br/>
**例子二：**

SAS语句：

```
where order_date=’01jan2000’d and product_id=’220101400106’;
```
表中已有的Index：

```
composite index defined on the following variables:Order_Date (first key variable) 
Delivery_Date (second key variable) 
Product_ID (third key variable)
```

因为where中的order_date在联合Index是first key variable，所以优化，使用Index。

<br/>
**例子三：**

SAS语句：

```
where delivery_date=’02jul2000’d’ and product_id=’220101400106’;
```
表中已有的Index：

```
composite index defined on the following variables:Order_Date (first key variable) 
Delivery_Date (second key variable) 
Product_ID (third key variable)
```

因为where中的delivery_date在联合Index不是first key variable，所以不能使用Index。

### 3. Identifying Conditions That Can Be Optimized

除了前面第二部分的条件外，SAS还会根据where中的一些情况进行判断，是否使用Index。

#### 3.1 Requirements for Optimizing a Single WHERE Condition

当where从句中出现下面的任何一种情况时才会评估是否使用Index：  

Operator|Example
----|----
comparison operators and the IN operator|where quarter = ’1998Q1’; or > XXX or IN (XX,XX)
comparison operators with NOT| var ne XXX or var not in (XX,XX)
comparison operators with the colon modifier(通配符，类似sql中的like，在sql中不能用)| XX =:'1998'
CONTAINS operator| var contains 'xx'
fully-bounded range conditions that specify both an upper and lower limit, which includes the BETWEEN-AND operator|1<var<10 or var between 1 and 10
pattern-matching operator LIKE| var like '%a'
IS NULL or IS MISSING operator| var is null or var is missing
TRIM function| trim(var)='1998'
SUBSTR function(非替换功能,并且必须只能从第一个字符可以)| substr(var,1,4)='1998'

#### 3.2 SAS不会optimize的where从句

Element in WHERE Condition|Example
----|----
除了TRIM or SUBSTR其他的function|weekday(var)=2
substr不是从第一个字符开始查找的|substr(var,2,2)='ss'
the sounds-like operator (=\*)基于英文文法的判断|var=\*'vincent'
arithmetic operators| where var=var+1
a variable-to-variable condition|where var2 > var1

#### 3.3 联合index优化的条件

和3.1的single index的要求差不多，下面是额外的要求：  

* where中条件中的var，使用and连接，可以是不同，也可以相同的。
* where中条件中的var，使用or连接，但必须是相同的var
* where至少有一个条件的判断是EQ或者IN。

以下情况，SAS对where不能使用联合index优化：

* 有CONTAINS。
* 有LIKE或者NOT LIKE。
* 有IS NULL或者IS MISSING。
* 含有任何函数。


例子1：
lastname和frstname是联合index，是and连接，并且是eq，两个var的顺序和联合index中的first var/second var一样，所以可以使用index。

```
where lastname eq ’Smith’ and frstname eq ’John’;
```

例子2：
lastname和frstname是联合index，是and连接，并且是eq，虽然两个var的顺序和联合index中的first var/second var不一样，所以可以使用index。所以顺序没有关系。

```
where frstname eq ’John’ and lastname eq ’Smith’;
```

例子3：
使用or连接，但两个var不一样，所以不能使用index。

```
where frstname eq ’John’ or lastname eq ’Smith’;
```

### 4. Estimating the Number of Observations
如果访问的是小数据的subset，使用Index更有效率；如果访问的是大数据的subset，使用顺序查找更有效率。

![](https://cl.ly/7defabf76f1a/Image%2525202019-05-31%252520at%25252012.00.20%252520%2525E4%2525B8%25258B%2525E5%25258D%252588.png)

SAS会判断：

* 如果访问的数据小于3%，就会使用Index,不会再评估 resource usage。
* 如果访问的数据大于33%，就不使用Index。
* 如果访问的数据量在3%和33%之间，倾向于使用Index，结合评估 resource usage。


#### 4.1 Printing Centile Information

可以在contents语句或者proc contents中使用centiles的options，来查看数据的百分比分布，可以帮助优化自己的程序。  

```
proc contents data=company.organization centiles;run;
```

### 5. Comparing Probable Resource Usage
#### 5.1 Factors That Affect I/O
#### 5.2 Subset Size Relative to Data Set Size
#### 5.3 Number of Pages in the Data File
#### 5.4 Order of the Data
#### 5.5 Cost to Uncompress a Compressed File for a Sequential Read

### 6. Deciding Whether to Create an Index

* 尽量可能少的创建index，只对经常要查询的，或者因为某种原因不能被sort的，在where或者by中的var.
* 创建index，当需要从大的dataset中查询到小的数据。
* 如果dataset的数据比较小（小于3个pages），不要创建index
* 针对经常会判断的var创建index
* 创建index前，先针对var排序，再创建index，会有效率些。
* 不要想着能用一个index，optimize所有where的条件。

#### 6.1  IDXWHERE= and IDXNAME=
大部分情况下，建议让SAS决定是否使用、使用哪一个index。但是可以通过options指定

#### 6.2 MSGLEVEL=I
使用MSGLEVEL=I可以在log中查看SAS是否使用、使用了哪个index

### 7 Comparing Procedures That Produce Detail Reports

PROC PRINT简单、简易，消耗资源少。
PROC SQL可以实现复杂的功能。

例子1：

先sort，再print

```
proc sort data=company.products     out=product;   by supplier_country;run;proc print data=product;   var product_id product_name       supplier_name;run;
```

在proc sql中完成sort和print

```
proc sql;   select product_id product_name          supplier_name      from company.products      order by supplier_country;quit;
```

PROC PRINT相对占用少的内存和CPU，两个proc在IO方面都差不多。

### 8 Comparing Tools for Summarizing Data

