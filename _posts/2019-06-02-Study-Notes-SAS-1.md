---
layout:     post
title:      Study Notes for "SAS Certification Prep Guide - Advance(Part1)"
subtitle:   Part 1 Chapter 1 - Chapter 12
author:     vincent_c
image: assets/images/studynotes1.jpg
catalog: true
categories: [ SAS, studynotes ]
tags: [SAS]
---  
This article recorded some study notes of book "SAS Certification Prep Guide - Advance" (Chapter 1 - Chapter 12)

## Foreword
since I had been preparing for SAS Advanced Programmer Exam, I write this study note which I study from the book "SAS Certification Prep Guide - Advance" in my blog, for recording this moments and experience.  

There are totally 2 parts for Chapter 1 to Chapter 24. Here is the Part One (for chapter 1 to chapter 12).

## Chapter 1 - Performing Queries Using PROC SQL 

1. sql语句中的where可以是Sas的where的有效语句， ne  gt 等也可以
	the expression in the WHERE clause can be any valid SAS expression.
2. SQL语句中，有group by，但没有sum function，效果等于普通order by, 在log中有提示
3. sql里面的从句必须按照下面的顺序：

	```
	SELECT XX
	FROM XX
	WHERE XX
	GROUP BY XX
	HAVING XX
	ORDER BY XX
	```

## Chapter 2 - Performing Advanced Queries Using PROC SQL
1. 使用feedback，可以在log将SQL语句中所有信息展开（比如select *）  
 
	```
	proc sql feedback;
	 	select *
	 	from sasuser.staffchanges;
	```
2. PROC SQL输出多少条数据，但还是全部读数据到sql中处理

	```
	PROC SQL OUTOBS= n;
	```
	PROC SQL中的操作符，除了ANY, ALL, and EXISTS，其他都可以在SAS中使用， between...and... , like.....
3. between后面没有大小顺序要求
	
	```
	select * from temp where no between 100 and 91;
	```
4. contains和？是等价的
	
	```
	sql-expression CONTAINS sql-expression
	sql-expression ? sql-expression
	```
5. 针对missing的判断
	
	```
	where boarded = .
	where flight = ’ ’
	```
	更好用下面的，不用担心是char还是numeric
	
	```
	where var IS MISSING
	where var IS NULL
	```
6. 通配的时候，_表示单个字符

	```
	LIKE ’D_an%’
	```
7. Sounds-Like (=*)会根据英文单词的拼写规则挑选出一些结果

	```
	where lastname =* ’Smith’;
	```
	>The sounds-like operator does not always select all possible values. For example, suppose you use the preceding WHERE clause to select rows from the following list of names that sound like Smith:
	  Schmitt   Smith
	  Smithson   Smitt
	  Smythe
8. 因为sql优先处理where从句，然后再select，所以会报错，没有total这个col

	```
	proc sql outobs=10;
	select flightnumber, date, destination,
	       boarded + transferred + nonrevenue
	       as Total
	    from sasuser.marchflights
	    where total < 100;
	```
	> ERROR: The following columns were not found in the contributing tables: total.  
	
	解决方法： 在计算的变量前面加上calculated
	
	```
	select flightnumber, date, destination,
	       boarded + transferred + nonrevenue
	       as Total
	from sasuser.marchflights
	where calculated total < 100;
	```
9. 将常量字符串恒定输出到sql结果的每一个row中  
	在select中直接用字符串代替var即可。  
	另外注意title位置，可以在proc sql里面，也可以在proc sql外面。  
	label的用法。      
	
	```
	proc sql outobs=15;
	title ’Current Bonus Information’;
	title2 ’Employees with Salaries > $75,000’;
	select empid label=’Employee ID’,
				jobcode label=’Job Code’,
				salary,
				’bonus is:’,
				salary * .10 format=dollar12.2
	from sasuser.payrollmaster
	where salary>75000
	order by salary desc;
	```  
10. 因为没有group by，select中计算的avg是所有行的salary, proc sql将所有数据看做一个group

	```
	proc sql outobs=10;
	   select jobcode, avg(salary)
	          as AvgSalary
	   from sasuser.payrollmaster;
	```
11. 下面结果返回单条数据，所有行的salary的avg

	```
	proc sql;
		select avg(salary)
	          as AvgSalary
		from sasuser.payrollmaster;
	```
12. 很多summary functions都忽略missing，不把missing计算在内
13. COUNT是proc sql中唯一一个可以用*作为参数的。
	>Note: The COUNT summary function is the only function that allows you to use an asterisk (*) as an argument.  
14. missing在group by的时候单独算一个值  
	count(*)显示的所有row的总数，count(var)是var不为missing的总数
	
	```
	proc sql;
		select count(JobCode) as Count
		from sasuser.payrollmaster;
	```
15. 如果下面的group by省略了，sas就把所有数据当成一个整个的group

	```
	proc sql;
		select jobcode,
			avg(salary) as AvgSalary
			format=dollar11.2
		from sasuser.payrollmaster
		group by jobcode
		having avg(salary) > 56000;
	```
16. Data Remerging  
	sql处理两遍，然后将两遍的结果merge
	下面例子是先算出来sum的相关，第二遍再筛选
	
	```
	proc sql;
		select empid, salary,
			(salary/sum(salary)) as Percent
			format=percent8.2
		from sasuser.payrollmaster
		where jobcode contains ’NA’;
	```
	会发生remerge的情况：  
	
	* The values returned by a summary function are used in a calculation.
	* The SELECT clause specifies a column that contains a summary function and other column(s) that are not listed in a GROUP BY clause.
	* The HAVING clause specifies one or more columns or column expressions that are not included in a subquery or a GROUP BY clause.
17. any的用法

	```
	where dateofbirth <小于号 any <subquery...>
	```
	例子：
	
	```
	proc sql;     select empid, jobcode, dateofbirth        from sasuser.payrollmaster        where jobcode in (’FA1’,’FA2’)              and dateofbirth < any                 (select dateofbirth                    from sasuser.payrollmaster                    where jobcode=’FA3’);
	```
	\> ANY(20，30，40) 大于20即可  
	< ANY(20，30，40) 小于40即可  
	= ANY(20，30，40) 等于20 or 30 or 40  
18. Non-Correlated Subqueries 和 Correlated Subqueries  
	Non-Correlated Subqueries：内外query是单独的，sql会计算最里面的，然后再一层一层的向外计算  
	比如下面的例子：
	
	```
	proc sql;
	   select empid, jobcode, dateofbirth
	      from sasuser.payrollmaster
	      where jobcode in (’FA1’,’FA2’)
	            and dateofbirth < all
	               (select dateofbirth
	                  from sasuser.payrollmaster
	                  where jobcode=’FA3’);
	```
	Correlated Subqueries：内外都不能单独计算，互相关联的  
	比如下面的例子：
	
	```
	proc sql;
	  select lastname, firstname
	     from sasuser.flightattendants
	     where not exists
	        (select *
	           from sasuser.flightschedule
	                  where flightattendants.empid=
	                        flightschedule.empid);   
	```
	上面的例子是计算在flightattendants中有数据，但在flightschedule没数据的  
19. 使用NOEXEC，不执行sql，只检查语法：  
在proc sql的后面，作用于后面所有的sql  

	```
	proc sql noexec;
	   select empid, jobcode, salary
	      from sasuser.payrollmaster
	      where jobcode contains ’NA’
	      order by salary;
	```
20. VALIDATE, 不执行sql，只检查语法，和NOEXEC一样，但位置不一样   
	在每个select的前面，没有；分号    
	只作用于紧跟在后面的一个select，如果有多个select，每个select之前都加一个  
	
	```
	proc sql;
	validate
	   select empid, jobcode, salary
	      from sasuser.payrollmaster
	      where jobcode contains ’NA’
	      order by salary;
	```
	
## Chapter 3 - Combining Tables Horizontally Using PROC SQL

1. join多个表，但没有where，产生笛卡尔乘积（Cartesian product），第一张表中的每条数据都和第二张表中的每个数据结合，类似所有可能
两张表中相同的变量不会被覆盖

	```
	proc sql;
	   select *
	      from one, two;
	```
2. Inner Join, 找出两张表的交集（最多32张表，不是32个view,一个view下面可能有多张表）

	```
	proc sql;
		select *
		from one, two
		where one.x = two.x;
	```
3. rename重复的var(两张表关联查询，但有相同的var)

	```
	proc sql;
		select one.x as ID, two.x, a, b
	  from one, two
	  where one.x = two.x;
	```
4. 如果inner join的时候，相同的关联var有多个重复的，则将内部进行笛卡尔乘积（没有rename）  
	例子：  
	x	var_1			x	var_2  
	a	123				a	234  
	a	567				a 789  
	关联后：  
	x var_1 	x var_2  
	a	123		a	234  
	a	123		a	789  
	a	567		a	234  
	a	567		a	789  
5. 关联查询时，给表起别名，table后面的as可以省略

	```
	proc sql;
	title ’Employee Names and Job Codes’;
		select s.empid, lastname, firstname, jobcode
		from sasuser.staffmaster as s,
			sasuser.payrollmaster as p
		where s.empid=p.empid
	```
	省略后变为：
	
	```
	from sasuser.staffmaster s,
		sasuser.payrollmaster p
	```
	下面两种情况 必须 给表其别名：
	
	* 自己关联自己查询  
	
	```
	from airline.staffmaster as s1,   airline.staffmaster as s2
	```
	
	* 在不同lib下面有相同的表名
	
	```
	from airline.flightdelays as af,work.flightdelays as wf
	where af.delay> wf.delay
	```
6. Outer Joins  
 
	* 	LEFT JOIN：在a中没有match的(ab的交集外)+ab的交集
	* 	RIGHT JOIN：在b中没有match的(ab的交集外)+ab的交集
	* 	FULL JOIN：在a中没有match的(ab的交集外)+ab的交集+在b中没有match的(ab的交集外) 

	![](https://cl.ly/ad144ffb7599/Image%2525202019-06-07%252520at%2525208.12.02%252520%2525E4%2525B8%25258A%2525E5%25258D%252588.png)
 
	用法：
	
	```
	SELECT column-1<,...column-n>
	FROM table-1 | view-1
	LEFT JOIN | rightHT JOIN | FULL JOIN table-2 | view-2
	ON join-condition(s)
	<other clauses>;
	```
	outer join(left join/right join/full join)只能操作2张表。  
	例子：
	
	```
	proc sql;
	 select one.x, a, b
	    from one
	    left join
	    two
	    on one.x=two.x;
	```
	
	left join, 左边表的所有记录，没有匹配上为missing，两个表的var：  
	![](https://cl.ly/45ce004581d7/Image%2525202019-06-07%252520at%2525208.13.26%252520%2525E4%2525B8%25258A%2525E5%25258D%252588.png)
	
	left join, 左边表的所有记录，没有匹配上为missing，指定的var：
	![](https://cl.ly/fd72a15ab8b8/Image%2525202019-06-07%252520at%2525208.14.46%252520%2525E4%2525B8%25258A%2525E5%25258D%252588.png)

	right join, 右边表的所有记录，没有匹配上为missing，两个表的var，默认右表var在后边：
	![](https://cl.ly/db5c65426472/Image%2525202019-06-07%252520at%2525208.15.37%252520%2525E4%2525B8%25258A%2525E5%25258D%252588.png)
	
	full join, 左表和右表的并集，重复的var只有一次，没有匹配上为missing，默认右表var在后边：
	![](https://cl.ly/d00b1fd59ab5/Image%2525202019-06-07%252520at%2525208.17.07%252520%2525E4%2525B8%25258A%2525E5%25258D%252588.png)
	
7. outer join默认不会覆盖相同的var，这也就是和data set merge的结果不同  
	如果需要outer join覆盖相同的var，需要COALESCE
	
	```
	select coalesce(three.x, four.x) as X, a, b
	from three full join four
	on three.x = four.x;
	```
8. sql join的好处：

	* 不需要sort或者index， merge需要
	* 不需要相同的var也可以手动关联，merger需要
	* 关联查询的时候，可以使用等号以外的条件，比如大于号等等
9. In-Line Views：相当于创建一张临时表：  
	例子：
	
	```
	from sasuser.flightschedule as f,
	       (select flightnumber, date
	               boarded/passengercapacity*100
	               as pctfull
	               format=4.1 label=’Percent Full’
	           from sasuser.marchflights) as m
	  where m.flightnumber=f.flightnumber
	        and m.date=f.date
	```
	
* 	但是in-line view中不能使用ORDER BY。
* 	可以使用as定义别名。

## Chapter 4 - Combining Tables Vertically Using PROC SQL

用法：

```
proc sql;
	select *
		from table1
  
  set-operator keywords <all> <corr>
	
	select *
		from table2;
``` 

1. 下面三个(EXCEPT/INTERSECT/UNION)默认是按照字段的顺序合并，不按照name，并且默认去掉重复的  

	* table1 EXCEPT table2 - 在table1中，不在table2中的 
	* table1 INTERSECT table2  - 两个的交集
	* table1 UNION table2 - 两个的合集   
2. OUTER不overlay字段，重复的也不overlay，全部var都display  
	table1 OUTER UNION table2 - 合集  
3. keywords ALL - 不去掉重复的字段

	```
	proc sql;   select *      from one   except all   select *      from two;
	```
4. keywords CORR (or CORRESPONDING)

	* with EXCEPT、INTERSECT、UNION，只按照相同name的变量合并，不相同的变量直接丢弃
	* with OUTER UNION, 相同name的var合并overlay

例子:  

EXCEPT - 在左不在右；去重；按相对顺序合并 
 
![](https://cl.ly/4f35e20b04be/Image%2525202019-06-04%252520at%25252010.17.52%252520%2525E4%2525B8%25258A%2525E5%25258D%252588.png)

EXCEPT ALL - 在左不在右；不去重；按相对顺序合并  

![](https://cl.ly/194853eefed5/Image%2525202019-06-04%252520at%25252011.02.05%252520%2525E4%2525B8%25258A%2525E5%25258D%252588.png)

EXCEPT CORR - 在左不在右；去重；只合并相同var的值    

![](https://cl.ly/8e4ea3c8f27d/Image%2525202019-06-04%252520at%25252011.02.58%252520%2525E4%2525B8%25258A%2525E5%25258D%252588.png)

EXCEPT ALL CORR - 在左不在右；不去重；只合并相同var的值    

![](https://cl.ly/bd0f0123ea25/Image%2525202019-06-04%252520at%25252011.04.06%252520%2525E4%2525B8%25258A%2525E5%25258D%252588.png)

INTERSECT - 两个的交集；去重；按相对顺序合并 

![](https://cl.ly/c34d3d26e474/Image%2525202019-06-04%252520at%25252011.05.28%252520%2525E4%2525B8%25258A%2525E5%25258D%252588.png)

INTERSECT ALL - 两个的交集；不去重；按相对顺序合并 

![](https://cl.ly/e5c2313a1e47/Image%2525202019-06-04%252520at%25252011.06.21%252520%2525E4%2525B8%25258A%2525E5%25258D%252588.png)

INTERSECT CORR - 两个的交集；去重；只合并相同var的值    

![](https://cl.ly/d4c8ac9b969d/Image%2525202019-06-04%252520at%25252011.07.11%252520%2525E4%2525B8%25258A%2525E5%25258D%252588.png)

INTERSECT ALL CORR - 两个的交集；不去重；只合并相同var的值    

![](https://cl.ly/5a7bdbc68dd2/Image%2525202019-06-04%252520at%25252011.08.26%252520%2525E4%2525B8%25258A%2525E5%25258D%252588.png)

注：  
关于重复数据的定义：
> in order to be considered a common row and to be included in the output, every duplicate row in one table must have a separate duplicate row in the other table. 
在右边的表中，没有重复的数据。但是如果在右边的表中增加一条重读的数据，比如1 w，这就算是有重复的数据了。  

UNION - 两个的并集（重复的数据只有一份）；去重；按相对顺序合并       

![](https://cl.ly/e774e95f9b68/Image%2525202019-06-04%252520at%25252011.39.07%252520%2525E4%2525B8%25258A%2525E5%25258D%252588.png)

UNION ALL- 两个的并集（重复的数据只有一份）；不去重；按相对顺序合并  

![](https://cl.ly/9e82d8e34597/Image%2525202019-06-04%252520at%25252011.41.56%252520%2525E4%2525B8%25258A%2525E5%25258D%252588.png)

UNION CORR - 两个的并集（重复的数据只有一份）；去重；只合并相同var的值      

![](https://cl.ly/8b61714e78fd/Image%2525202019-06-04%252520at%25252011.42.38%252520%2525E4%2525B8%25258A%2525E5%25258D%252588.png)

UNION CORR ALL - 两个的并集（重复的数据只有一份）；不去重；只合并相同var的值  

![](https://cl.ly/a8311b25050b/Image%2525202019-06-04%252520at%25252011.43.46%252520%2525E4%2525B8%25258A%2525E5%25258D%252588.png)


OUTER UNION - 两个表的直接相加；所有var都简单堆砌（重复的也不overlay，全部var都display）    

![](https://cl.ly/3352fcd1f840/Image%2525202019-06-04%252520at%25252011.44.39%252520%2525E4%2525B8%25258A%2525E5%25258D%252588.png)


OUTER UNION CORR - 两个表的直接相加；相同的varoverlay    

![](https://cl.ly/87165376ed3d/Image%2525202019-06-04%252520at%25252011.46.28%252520%2525E4%2525B8%25258A%2525E5%25258D%252588.png)


## Chapter 5 - Creating and Managing Tables Using PROC SQL

1. numeric的长度不能定义  
	PROC SQL allows you to specify a column width for character columns but not for numeric columns.

2. 创建空白表Creating an Empty Table by Defining Columns  

	```
	proc sql;
	create table work.departments
	      (Dept varchar(20) label=’Department’,
	       Code integer label=’Dept Code’,
	       Manager varchar(20),
	       AuditDate num format=date9.);
	```

3. DESCRIBE TABLE语句  
	The DESCRIBE TABLE statement会把表结构用create table的方式，写入log中（contents是写入report中）
	
	```
	proc sql;
	   describe table work.discount;
	```
	结果如下：  
	> NOTE: SQL table WORK.DISCOUNT was created like:  
	> create table WORK.DISCOUNT( bufsize=4096 ）
	> （  
	> Destination char(3),  
	>  BeginDate num format=DATE9.,  
	>  EndDate num format=DATE9.,  
	>  Discount num  
	>  ）  

4. 从别的表中copy表结构，创建新表(不包含数据)

	```
	proc sql;
	 create table work.flightdelays2
	    like sasuser.flightdelays; 
	```
5. 只复制部分字段过来，使用drop选项（不包含数据）

	```
	proc sql;
	 create table work.flightdelays3
	    (drop=delaycategory destinationtype)
	    like sasuser.flightdelays;
	```

6. 从别的表中select部分数据，复制到新表，同时包括表结构

	在log中不显示数据的query结果，只有创建表中的结果
	
	```
	proc sql;
	create table work.ticketagents as
	select lastname, firstname,
	    jobcode, salary
	from sasuser.payrollmaster,
	     sasuser.staffmaster
	where payrollmaster.empid
	      = staffmaster.empid
	      and jobcode contains ’TA’;
	```

7. copy数据和结果（没有where）

	如果没有order by，新表和旧表的数据row顺序可能不一样
	
	```
	proc sql;
	   create table work.supervisors2 as
	      select *
	         from sasuser.supervisors;
	```

8. insert数据 - 使用set

	```
	proc sql;
	   insert into work.discount
	      set destination=’LHR’,
	          begindate=’01MAR2000’d,
	          enddate=’05MAR2000’d,
	          discount=.33
	      set destination=’CPH’,
	          begindate=’03MAR2000’d,
	          enddate=’10MAR2000’d,
	          discount=.15;
	   select *
	     from discount;
	```

9. insert数据 - 使用values，完整的数据

	```
	insert into work.newtable
	   values (’WI’,’FLUTE’,6)
	   values (’ST’,’VIOLIN’,3);
	```

10. insert数据 - 使用values，部分数据

	```
	insert into work.newtable
	(item,qty)
	   values (’FLUTE’,6)
	   values (’VIOLIN’,3);
	或者values (’ ’, ., 45)
	```

11. Creating a Constraint in a Column Specification

	* 约束类型为not null，表示col不能为空或者missing  
	* 约束类型为check(expression)，表示col的值必须符合expression  

12. 创建约束的两种方法：

	方法一：Creating a Constraint in a Column Specification
	
	```
	proc sql;
	     create table work.employees
	        (ID char (5) primary key,
	        Name char(10),
	        Gender char(1) not null check(gender in (’M’,’F’)),
	        HDate date label=’Hire Date’);
	```
	
	方法二：Creating a Constraint by Using a Constraint Specification
	
	```
	proc sql;
	 create table work.discount3
	        (Destination char(3),
	        BeginDate num Format=date9.,
	        EndDate num format=date9.,
	        Discount num,
	        constraint ok_discount check (discount le .5),
	        constraint notnull_dest not null(destination));
	```

13. 插入多条数据时，有部分数据违反约束

	插入两条数据，如果有数据违反完整性约束，已经成功的数据最后也会被删除  
	在log中，会显示VALUES clause XXX错误，但只是第一条出现错误的数据  
	
	```
	proc sql;
	 insert into work.discount3
	        values(’CDG’,’03MAR2000’d,’10MAR2000’d,.15)
	        values(’LHR’,’10MAR2000’d,’12MAR2000’d,.55);
	```

14. UNDO_POLICY OPTIONS

	UNDO_POLICY = 
	
	* REQUIRED（默认值，有错误数据，回滚所有数据，在SAS/SHARE方式下，可能无效）
	* NONE （不需要回滚，正确的执行，错误的不管）
	* OPTIONAL （REQUIRED + NONE ）
	
	例子：
	在下面的例子中，第一条数据没有问题，成功插入，第2条插入失败，log中有提示
	
	```
	proc sql undo_policy=none;
	     create table work.discount4
	            (Destination char(3),
	            BeginDate num Format=date9.,
	            EndDate num format=date9.,
	            Discount num,
	            constraint ok_discount check (discount le .5),
	            constraint notnull_dest not null(destination));
	     insert into work.discount4
	            values(’CDG’,’03MAR2000’d,’10MAR2000’d,.15)
	            values(’LHR’,’10MAR2000’d,’12MAR2000’d,.55);
	```

15. 查看完整性约束：

	```
	proc sql;
	     describe table constraints work.discount4;
	```

16. 多条件update --- 有点像SAS中的SELECT

	```
	proc sql;
		update work.insure_new
		set pctinsured=pctinsured*
			case
	     when company=’ACME’
				then 1.10
			when company=’RELIABLE’
				then 1.15
			when company=’HOMELIFE’
				then 1.25
	     else  1
		end;
	```
	或者
	
	```
	proc sql;
		update work.payrollmaster_new2
		set salary=salary*
			case substr(jobcode,3,1)
				when ’1’
					then 1.05
				when ’2’
					then 1.10
				when ’3’
					then 1.15
				else 1.08
			end;
	```

17. 在select中也可以使用case，根据不同的值，返回显示不同的值

	```
	proc sql outobs=10;
		select lastname, firstname, jobcode,
			case substr(jobcode,3,1)
				when ’1’
					then ’junior’ 
				when ’2’
					then ’intermediate’
				when ’3’
					then ’senior’
				else ’none’
		as JobLevel
		from sasuser.payrollmaster,
			sasuser.staffmaster
		where staffmaster.empid=
			payrollmaster.empid;
	```

18. 修改表结构：ALTER语句不能在view中使用  
	> You cannot use the ALTER TABLE statement with views.

19. 增加新的列

	```
	proc sql;
		alter table work.payrollmaster4
	       add Bonus num format=comma10.2,
	       Level char(3);
	```

20. 删除列，如果有多个，中间使用comma隔开

	```
	proc sql;
	 alter table
	    work.payrollmaster4
			drop bonus, level;
	```

21. 修改表中列的属性：

	```
	proc sql;
		alter table work.payrollmaster4
		modifty salary format=dollar11.2 label="Salary Amt";
	```

22. 可以将上述statement放在一起：

	```
	proc sql;
	 alter table work.payrollmaster4
		add Age num
	  modifty  dateofhire date format=mmddyy10.
	  drop dateofbirth, gender;
	```

23. drop整表：
	
	```
	proc sql;
	drop table work.payrollmaster4;
	```


## Chapter 6 - Creating and Managing Indexes Using PROC SQL

1. Index的类型
	index有两种：simple  和   composite

2. 创建简单index：
	简单index的名字必须和col的名字一样，上面例子中index名字只能为 empid
	
	```
	proc sql;
	 create unique index EmpID
	    on work.payrollmaster(empid);
	```

3. 创建复合index：
	复合index的名字需要另外起一个,不能和现有的col名字一样  
	Index名字不区分大小写  
	
	```
	proc sql;
	   create unique index daily
	      on work.marchflights(flightnumber,date);
	```
	
	如果复合index的col值有重复的(重复的ob)，index不会被创建，上面例子中，如果flightnumber,date的pair值还有重复的
	log中提示：
	>ERROR: Duplicate values not allowed on index daily for file MARCHFLIGHTS.


4. DESCRIBE TABLE

	```
	proc sql;     describe table marchflights;
	```
	
	使用DESCRIBE TABLE可以查看index, 在log中找CREATE INDEX的部分，如果没有，就说明没有index。  
	在系统表 Dictionary.Indexes中也有所有的表的index信息。  
	PROC CONTENTS and PROC DATASETS 也可以再report中显示index信息。  

5. MSGLEVEL OPTIONS

	```
	OPTIONS MSGLEVEL=N | I;
	```
	
	* 默认是N，显示基本note、warning、error信息  
	* 如果设置为I，显示debug信息，index、merge等详细信息   
	* 在log中会显示index是否被使用
	
	```
	options msglevel=i;
	  proc sql;
	     select *
	        from marchflights
	        where flightnumber=’182’;
	```

6. 在表后面使用idxwhere，SAS不使用index

	```
	proc sql;
	   select *
	      from marchflights (idxwhere=no)
	      where flightnumber=’182’;
	```

7. idxname手动指定使用哪一个Index

	如果在一个col上面有多个index，简单index和复合index，可以在表后面使用idxname，告诉sas使用哪一个index
	
	```
	proc sql;
	   select *
	      from marchflights (idxname=daily)
	      where flightnumber=’182’;
	```

8. drop index

	```
	proc sql;
	 drop index daily
	    from work.marchflights;
	```

## Chapter 7 - Creating and Managing Views Using PROC SQL

1. 创建View

	不会querydata，只是创建一个reference
	
	```
	proc sql;
		create view sasuser.faview as
		  select lastname, firstname, gender,
		         int((today()-dateofbirth)/365.25) as Age,
		         substr(jobcode,3,1) as Level,
		         salary
		     from sasuser.payrollmaster,
		          sasuser.staffmaster
		     where jobcode contains ’FA’ and
		           staffmaster.empid=
		           payrollmaster.empid;
	```

2. 使用DESCRIBE VIEW查看view的创建

3. 创建view需要注意的：

	* 创建view的时候，不要order by，影响效率，否则每次query都会order by
	* 如有数据需要频繁被使用，将数据存放在临时表中湖更有效率，不然view每次都要query
	* view不要创建在经常会改变表结构的表

4. view和相关联的表如果在同一个libref下面  
	所以下面的例子中，from里面就不用libref了，因为create view中已经有libref了
	
	```
	proc sql;
	  create view sasuser.payrollv as
			select *
	    from sasuser.payrollmaster;
	```
	可以修改为：
	
	```
	proc sql;
	 create view sasuser.payrollv as
	    select *
	       from payrollmaster;
	```

5. 如果view和tables不在同一个libref下面，create和from中都必须有libref，即two-level name

6. 在create view中embedded定义libname

	定义的libname只在create view中有效，和全局同名的libname不会冲突
	
	```
	libname airline ’SAS-library one’;
	
	proc sql;
		create view sasuser.payrollv as
			select*
			from airline.payrollmaster
		using libname airline ’SAS-library two’;
	quit;
	
	proc print data=sasuser.payrollv;
	run;
	```

7. update view：更新视图

	* 语法和普通的update一样
	* 只能update单表的view，view中有多个表join的不行
	* view中有表达式计算的col不能update
	* 可以使用where从句update，但不能使用其他的从句，order by，having
	* 里面有group by的view不能update

8. drop view

	```
	proc sql;
	     drop view sasuser.raisev;
	```

## Chapter 8 - Managing Processing Using PROC SQL

1. INOBS控制每张表读入的数据, OUTOBS控制每张表输出的数据

	下面的例子中一共读入10条数据
	
	```
	proc sql inobs=5;
	   select *
	      from sasuser.mechanicslevel1
	   outer union corr
	   select *
	      from sasuser.mechanicslevel2;
	```

2. NUMBER , NONUMBER

	NUMBER,NONUMBER（默认）控制输出结果中是否包含行号，类似于proc print中的OBS , NOOBS
	
	```
	proc sql inobs=10 number;
	   select flightnumber, destination
	      from sasuser.internationalflights;
	```

3. DOUBLE , NODOUBLE

	DOUBLE , NODOUBLE（默认）输出时使用双空格作为分隔符，html不起作用，listing可以
	
	```
	proc sql inobs=10 double;
	     select flightnumber, destination
	        from sasuser.internationalflights;
	```

4. The FLOW , NOFLOW , FLOW=n , FLOW=n m

	The FLOW , NOFLOW , FLOW=n , FLOW=n m， 在listing下控制每列的宽度，避免某个col太长，导致换行
	
	```
	proc sql inobs=5 flow=10 15;
	    select ffid, membertype, name, address, city,
	           state, zipcode
	       from sasuser.frequentflyers
	       order by pointsused;
	```

5. STIMER , NOSTIMER（默认）

	STIMER , NOSTIMER（默认）默认log中只记录累计的总运行时长  
	STIMER可以详细地记录每个query的运行时长  
	如果要再proc sql中使用STIMER， 在SAS系统级options的STIMER也要先使用，否则显示的还是累计的总运行时长  
	
	```
	proc sql stimer;
	   select name, address, city, state, zipcode
	      from sasuser.frequentflyers;
	   select name, address, city, state, zipcode
	      from sasuser.frequentflyers
	      where pointsearned gt 7000 and pointsused lt 3000;
	quit;
	```

6. reset option

	下面的例子中第一个select使用options的outobs，第二个select，因为有reset，除了还使用之前的options（outobs）外，再使用number (reset语句只对语句后面的option进行追加，之前的参数设置不改变)  
	
	```
	proc sql outobs=5;
	     select flightnumber, destination
	        from sasuser.internationalflights;
	  reset number;
	     select flightnumber, destination
	        from sasuser.internationalflights
	        where boarded gt 200;
	```   
	
	下面的例子中的第二个select，追加了outobs= number（中间有空格，这是两个option），但 outobs=覆盖了之前option中outobs=5
	
	```
	proc sql outobs=5;
	      select flightnumber, destination
	         from sasuser.internationalflights;
	   reset outobs= number;
	      select flightnumber, destination
	         from sasuser.internationalflights
	         where boarded gt 200;
	```

7. Dictionary tables

	* Dictionary tables可以存放在Dictionary的libraray中(Dictionary.XXXX)，或者存储在Sashelp的表的view中查看
	
	* Dictionary tables：每次自动创建、update，只读
	 
	* dictionary表中的libname是以大写存储的
	
	```
	proc sql;
	select memname, modate, nvar, nobs
	         from dictionary.tables
	         where libname=’SASUSER’;
	```
	
	DICTIONARY表格|SASHELP视图|信息
	----|----|----
	Catalogs|Vcatalg|Catalog 条目的信息Columns|Vcolumn|变量及其属性的详细信息(如名称,类型, 长度,格式)Extfiles|Vextfl|当前分配的 FilerefIndexes|Vindex|数据文件定义的索引的信息Macros|Vmacro|用户以及系统定义的宏的信息
	Members|Vmember,Vsacces,Vscatlg,Vslib,Vstable,Vstabvw,Vsview|数据逻辑库的一般信息
	Options|Voption|当前的 SAS 系统选项Tables|Vtable|数据集的详细信息Titles|Vtitle|指定的标题以及脚注文本Views|Vview|数据视图的一般信息



## Chapter 9 - Introducing Macro Variables

1. 宏变量可以在任何地方定义、引用，除了datalines/cards里面

2. 宏变量必须被双引号括在一起，单引号不会被解析 

3. 引用不存在的宏变量，warning,但不会中断程序运行。

	例子：  
	最后第二个put语句输出的还是"—>(!!) outside macro &NEWNAME &SETNAME"，即变量没有解析，log中更有warning，但还继续  
	
	```
	%macro MAKEPGM(NEWNAME, SETNAME); data &NEWNAME;set &SETNAME; run;%put ->(!!) inside macro &NEWNAME &SETNAME; %mend;%MAKEPGM(WORK.NEW, SASHELP.CLASS)%put —>(!!) outside macro &NEWNAME &SETNAME;
	```

4. 引用错误命名的宏变量。比如，an invalid macro variable name，（不符合命名规则），error， 中断程序运行。

5. 宏变量的赋值

	```
	%let time=afternoon;
	```

	* 值为字符
	* 表达式、运算不会被计算
	* 大小写敏感
	* 即使有单双引号，也算是宏变量值得一部分
	* 等号后面的字符的开头、拖尾的空格会被自动去掉

6. NOSYMBOLGEN ， SYMBOLGEN

	设置后，宏变量运行时的值会被显示
	
	```
	OPTIONS NOSYMBOLGEN | SYMBOLGEN;
	```

7. The %PUT statement

	* 只写入log
	* 每次都另起一行，从头写
	* 没有参数就写空行
	* 后面的参数不需要用单双引号
	* 可以跟宏变量，会先解析，再put
	* 开头和拖尾的空格都被删除（除非使用了%str）
	* wraps lines 如果put的文本长度大于当前行的最大长度

8. %STR()

	下面语句会报错，宏变量赋值时，等号后面都是值，但是有分号，SAS会认为分号是这个语句的结束
	
	```
	%let prog=data new; x=1; run;
	```
	
	需要使用%STR()   可以保留一些特殊字符  
	
	Method One
	
	```
	   %let prog=%str(data new; x=1; run;);
	```
	Method Two
	
	```
	   %let prog=data new%str(;) x=1%str(;)run%str(;);
	```
	
	Method Three
	
	```
	   %let s=%str(;);
	   %let prog=data new&s x=1&s run&s;
	```
	
	保留’")( 等时的用法：
	
	```
	  %let text=%str(Joan%’s Report);
	  %let text=Joan%str(%’)s Report;
	```

9. %NRSTR

	The %NRSTR Function  保留 & 和 % （%STR不可以）
	下面程序的log报错：
	
	```
	%let Period=%str(May&Jun);
	%put Period resolves to: &period;
	```
	
	下面程序运行成功：
	
	```
	%let Period=%nrstr(May&Jun);
	%put Period resolves to: &period;
	```

10. %BQUOTE

	The %BQUOTE Function 保留引号，不用前面加%
	
	```
	%let text=%bquote(Joan’s Report);
	```
	
	
	%QUOTE 以及%NRQUOTE 方法会遮盖如下的操作符:  
	+ - * / < > = ^ ; ~ , # space
	+ AND OR NOT EQ NE LE LT GE GT IN  
	同时,会遮盖 '(单引号) 和 "(双引号):配对使用时、单独使用时、或者被前置的%(百分号)标记时。
	
	%NRQUOTE同时还会遮蔽 & 和 %,所以在参数可能含有不想被解析的Macro变量引用或者Macro调用时尤其有效。  

11. %UPCASE

	和普通的upcase一样 

12. %QUPCASE

	The %QUPCASE Function  和%UPCASE一样，但可以屏蔽% &

13. %SUBSTR

	和普通的substr一样

14. %QSUBSTR

	The %QSUBSTR Function  和%SUBSTR一样，可以屏蔽% &

15. The %INDEX Function

16. The %SCAN Function

17. The %QSCAN Function

18. %SYSFUNC

	The %SYSFUNC Function 在宏中调用系统函数（部分不能使用）

19. %QSYSFUNC

	Quoting with %QSYSFUNC（）和%SYSFUNC一样，但可以忽略参数中的字符串中的特殊字符

20. 宏变量和其他char的连接

	宏变量后面还有string，中间用.分隔
	
	```
	     %let endtime=end;
	     if year(&endtime.date)=&yr;
	```
	
	如果宏变量后面本来就需要有一个小数点，就再加一个
	
	```
	data &libref..temp&yr;
	```

21. 宏的解析过程

	先解析宏，再解析data，再执行，解析data的时候，所有宏变量就没有了，都已经是值了
	
	在下面的例子中，不管值是什么，foot的值永远都是"All Students Have Paid"
	
	```
	options symbolgen pagesize=30;
	  %let crsnum=3;
	  data revenue;
	     set sasuser.all end=final;
	     where course_number=&crsnum;
	     total+1;
	     if paid=’Y’ then paidup+1;
	     if final then do;
	        put total= paidup=; /* Write information
	                              to the log. */
	        if paidup<total then do;
	           %let foot=Some Fees Are Unpaid;  ---解析时，将值赋值为宏变量
	end; else do;
	           %let foot=All Students Have Paid;  ---解析时，又赋值，就覆盖了，解析data、执行的时候，就使用的是这个值
	end; end;
	run;
	```

22. 使用CALL SYMPUT赋值char(在宏解析时不执行，在执行时再执行)  

	使用CALL SYMPUT(‘macro-variable’, ‘text’);
	
	```
	call symput(’foot’,’Some Fees Are Unpaid’);
	```

23. 使用CALL SYMPUT赋值data步中的var

	```
	CALL SYMPUT(‘macro-variable’,DATA-step-variable);
	```
	
	* 宏变量（接收）32767个字符
	* var中的开头和拖尾的空格不会被处理，直接赋值给宏变量
	* 如果var是数字，隐式转换使用BEST12.，赋值给宏变量

24. CALL SYMPUTX

	CALL SYMPUTX 和 CALL SYMPUT 基本一样，但是会将开头和拖尾的空格去掉

25. CALL SYMPUT(expression1,expression2)

	参数是两个表达式，没有var,形如：
	
	```
	CALL SYMPUT(expression1,expression2);
	```
	
	表示： 使用expression1的结果作为宏变量的名字，同时使用expression2的结果作为宏变量的值  
	类似于python中的dict  
	下面例子中，读取sasuser.courses的数据，然后使用数据集（每一行数据）中的course_code的值作为宏变量名字，值为course_title
	
	```
	data _null_;
	     set sasuser.courses;
	     call symput(course_code, trim(course_title));
	   run;
	```
	这样就mapping起来了，会更有效率


26. Referencing Macro Variables Indirectly（间接调用宏变量）

	The Forward Re-Scan Rule：
	
	* 有两个&或者%的时候，先将&&变成&，%%变成%，然后开始re-scan模式
	* 从左到右，依次re-scan所有
	
	例子：
	
	```
	%let a=a2;
	%let a2=100;
	%let num=2;
	```
	
	解析过程：
	
	```
	&a --> a2
	&&a --> &a(begin re-scan) -->  a2
	&&&a --> &(&a with re-scan)->&a2(with re-scan)-> 100
	&&a&num -> &(a&num) with re-scan -> &a2 with re-scan ->100
	```
	
	例子：
	
	```
	%let sun=SHINE;	%let shine=MOON;	%let moon=earth;
	
	SAS运行log：
	1167  %put 1,&sun;	1,SHINE
	1168  %put 2,&&sun;
	2,SHINE
	1169  %put 3,&&&sun;
	3,MOON
	1170  %put 4,&&&&sun;
	4,SHINE
	1171  %put 5,&&&&&sun;
	5,MOON
	1172  %put 6,&&&&&&sun;
	6,MOON
	1173  %put 7,&&&&&&&sun;
	7,earth
	1174  %put 8,&&&&&&&&sun;
	8,SHINE
	1175  %put 9,&&&&&&&&&sun;
	9,MOON
	1176  %put 10,&&&&&&&&&&sun;
	10,MOON
	1177  %put 11,&&&&&&&&&&&sun;
	11,earth
	1178  %put 12,&&&&&&&&&&&&sun;
	12,MOON
	1179  %put 13,&&&&&&&&&&&&&sun;
	13,earth
	1180  %put 14,&&&&&&&&&&&&&&sun;
	14,earth
	1181  %put 15,&&&&&&&&&&&&&&&sun;
	WARNING: Apparent symbolic reference EARTH not resolved.
	15,&earth
	1182  %put 16,&&&&&&&&&&&&&&&&sun;
	16,SHINE
	1183  %put 17,&&&&&&&&&&&&&&&&&sun;
	17,MOON
	```

	解析：  
	从左到右，依次将&&变为&，如果有落单的，就结合为宏变量，然后re-scan，不断重复此过程。
	
	* 	&&&&&sun -> & & &sun -> &&shine -> &(shine) -> moon
	* 	&&&&&&sun -> & & & sun -> && &sun ->同上->moon
	* 	&&&&&&&&sun -> & & & & sun -> && && sun -> & & sun -> &&sun -> &(sun) -> shine
	* 	&&&&&&&&&&&sun -> & & & & & (&sun) ->&& && & (shine) -> & & (&shine) -> & moon -> earth
	* 	&&&&&&&&&&&&&&&sun -> & & & & & & & (&sun) -> && && && & (shine) -> & & & (&shine) -> && &(moon) -> & earth
	

27. symget

	symget和symput类似，都是告诉SAS编译的时候先跳过，等到执行的时候再编译。
	
	
	```
	data re;
	set test;
	length teacher $ 20;
	call symput('temp',num); --- 执行时再解释，但下面的语句是编译时解释
	teacher=&&teacher&temp; --编译时解释，temp的值还没有，num不能被解释成var，只能当做char
	run;
	```
	
	需要修改为：
	
	```
	data re;
	set test;
	length teacher $ 20;
	teacher=symget('teacher'||left(num)); --同样是执行的时候才解释，就可以使用num作为var
	run;
	```

28. Creating Macro Variables During PROC SQL Step Execution

	* 在select后面通过into，将select的结果存储在宏变量中，不要忘记冒号
	* 赋值的时候不会去掉开头和拖尾的空格，可以在后面的logic时再机上%let.
	
	```
	proc sql noprint;
	   select sum(fee) format=dollar10. into :totalfee
	      from sasuser.all;
	quit;
	%let totalfee=&totalfee;   --- let进行宏变量赋值会自动去掉开头和拖尾的空格
	```

## Chapter 10 - Processing Macro Variables at Execution Time 


### 1.Creating Macro Variables During PROC SQL Step Execution

在select后面通过into，将select的结果存储在宏变量中，**但是不要忘记冒号**， <u>并且不会去掉开头和拖尾的空格</u>  
**proc sql中的noprint选项可以不输出结果到report**


```
proc sql noprint;
   select sum(fee) format=dollar10. into :totalfee
      from sasuser.all;
quit;
%let totalfee=&totalfee;   --- let进行宏变量赋值会自动去掉开头和拖尾的空格
```
### 2.在proc sql中创建多个var(list)
下面的例子是将select的第一条row中的course_code, location, begin_date赋值给crsid1，place1，date1  
然后将第二条row的赋值为crsid2，place2，date2  
第三条的给crsid3，place3，date3  
相当于数组

```
proc sql;     select course_code, location, begin_date format=mmddyy10.
     into :crsid1-:crsid3,
          :place1-:place3,
          :date1-:date3        from sasuser.schedule        where year(begin_date)=2002        order by begin_date;quit;
```

如果想将表中的所有的col都赋值给var list，可以先获取表中一共有多少row，然后再进行赋值
**不要忘记使用into后，再配合使用%let将空格trim掉**

```
proc sql noprint;     select count(*) into :numrows        from sasuser.schedule        where year(begin_date)=2002;     %let numrows=&numrows;     %put There are &numrows courses in 2002;     select course_code, location,            begin_date format=mmddyy10.        into :crsid1-:crsid&numrows,             :place1-:place&numrows,             :date1-:date&numrows        from sasuser.schedule        where year(begin_date)=2002        order by begin_date;     %put _user_;  quit;
```
### 3.将select的结果合并存储，赋值给一个变量
select一个col，然后使用SEPARATED BY ‘delimiter1’，将结果合并存储

```
proc sql noprint;     select distinct location into :sites separated by ’ ’        from sasuser.schedule;quit;
```

### 4.在proc view中使用宏变量
下面使用方式导致sas只在编译的时候，处理了宏变量的值，编译的值是多少，就是多少。而不能在处理的过程中根据逻辑而取值。

```
proc sql;	create view subcrsid as		select student_name, student_company,paid		from sasuser.all		where course_code="&crsid";quit;
```
需要使用symget，在运行的过程中再取宏变量的值
这样每次使用view都可以使用不同的值

```
proc sql;
	 create view subcrsid as
	    select student_name,student_company,paid
	       from sasuser.all
	       where course_code=symget(’crsid’);
quit;

%let crsid=C003;
proc print data=subcrsid noobs;
title "Status of Students in Course Code &crsid";
run;

%let crsid=C004;
proc print data=subcrsid noobs;
title "Status of Students in Course Code &crsid";
run;
```

在创建view的使用也可以用symget宏变量，但需要注意：**如果使用宏变量的值进行数值判断和比较，要用Input显示转换。宏变量里是char**

```
proc sql;	create view subcnum as		select student_name, student_company, paid		from sasuser.all		where course_number=input(symget(’crsnum’),2.);quit;
```

### 5. Using Macro Variables in SCL Programs
**to be done**

### 做错的练习题

1 Which of the following is false?  >a) A %LET statement causes the macro processor to create a macro variable before the program is compiled.  b) To create a macro variable that is based on data calculated by the DATA step, you use the SYMPUT function.  c) Macro functions are always processed during the execution of the DATA step.  d) Macro variable references in a DATA step are always resolved prior to DATAstep execution.  Correct answer: c
>Most macro functions are handled by the macro processor before any SAS language statements in the DATA step are executed. For example, the %LET statement and any macro variable references (&macvar) are passed to the macro processor before the program is compiled. In order to create or update macro variables during DATA step execution, you use the SYMPUT routine.

大部分宏过程都是在编译阶段就运行了。  
宏处理器在编译阶段前就已经处理、创建好宏变量了。

## Chapter11 - Creating and Using Macro Programs
### 1.定义宏
使用%macro和%mend括起来，其中%mend后面的macro name可以省略(%mend不可以省略)

```
%macro prtlast;     proc print data=&syslast (obs=5);     title "Listing of &syslast data set";     run;%mend;
```
### 2.Compiling a Macro
在SAS编译前，宏处理器先编译Macro  
编译时：  
* 前检查语法，non-macro statements先不检查，等宏执行时再检查  
* 如果有语法错误，创建一个dummy(不能执行的宏)  
* 如果没有语法错误，创建宏在SAS catalog，默认 Work.Sasmacr，文件名为Macro-name.Macro  

### 3.打开编译宏时的日志
log中会有编译时的详细信息

```
options mcompilenote=all;%macro mymacro;%mend mymacro;
```

### 4. 宏调用

*  在宏名称前加%调用  
*  除了datalines/cards，其他任何地方都可以调用宏  
*  调用宏不是SAS statement，最后不用加分号

To execute the macro Prtlast you would call the macro as follows: 

```
%prtlast
```
注意：宏调用后面不要加分号，会导致分号和其他statement结合，导致意想不到的结果

宏调用有三种：  
1. name style  
2. command style  
3. statement style

其中name style(通过%调用)效率最高



### 5.宏相关的日志options
> OPTIONS MCOMPILENOTE= NONE | NOAUTOCALL | ALL;  
> OPTIONS MPRINT | NOMPRINT;  
> OPTIONS MLOGIC | NOMLOGIC;  
> OPTIONS SYMBOLGEN;  

SYMBOLGEN:
将宏变量解析的结果过程写在log  
例子：

```
SYMBOLGEN:  Macro variable REPTITLE resolves to Book Section
SYMBOLGEN:  Macro variable SYSDAY resolves to Tuesday
31   title “Frequencies by &reptitle as of &sysday”;
32   proc freq data=books.ytdsales;
SYMBOLGEN:  Macro variable REPVAR resolves to section
33     tables &repvar;
```

MPRINT:  
当宏执行的时候,将提交给编译器文本打印到 SAS LOG

例子：

```
MPRINT(ABC):title1 "Hello" ;MPRINT(ABC):title2 "There" ;MPRINT(ABC):proc print data=work.look ;MPRINT(ABC):run ;```

MLOGIC:  

* 宏执行的开始* 触发宏时提供的参数值
* 每一个宏程序的执行
* %IF条件的真值* 宏执行的结束

例子：

```
options nomprint mlogic;
%prtlast
MLOGIC(PRTLAST): Beginning execution.
NOTE: There were 1 observations read from the dataset
WORK.SALES.
NOTE: PROCEDURE PRINT used:
real time 0.02 seconds
cpu time 0.02 seconds
MLOGIC(PRTLAST): Ending execution.
```


### 6.宏的执行逻辑
1. searches the designated SAS catalog (Work.Sasmacr by default) for an entry named Macro-name.Macro  2. executes compiled macro language statements within Macro-name  3. sends any remaining text in Macro-name to the input stack for word scanning  4. suspends macro execution when the SAS compiler receives a global SAS statement or when it encounters a SAS step boundary  5. resumes execution of macro language statements after the SAS code executes.  

例子：

```
%macro prtlast;	proc print data=&syslast(obs=5);		title "Listing of &syslast data set";	run;%mend;
```
处理逻辑：
  
1. 遇到%，触发，宏处理器在指定的位置找编译好的宏  
2. 按照顺序执行宏里面的编译好的宏语句  
3. 宏处理器遇到SAS language statements（没有编译好的），放入input stack.  
4. 在上面的过程中，如果遇到宏变量，宏处理器去找对应的值，然后继续放入input stack.  
5. 整段语句(非宏语句)编译完了，执行，上面的例子是proc print  
6. 继续顺序执行，下面的宏语句，上面的例子中proc print后面就没有宏语句  
7. 到%MEND就结束.  

### 7.宏里面的注释(%*comment;)

```
%macro printit;
	%* The value of &syslast will be substituted appropriately ;	%* as long as a data set has been created during this session. ;	proc print data=&syslast(obs=5);	/* Print only the first 5 observations */	title "Last Created Data Set Is &syslast";run; %mend;```
### 8.Using Macro Parameters（按照参数定义的顺序调用）

需要注意，宏里面使用参数不要忘记&  
定义的时候，因为没有使用keyword，调用必须按照顺序
调用宏的时候，不要加引号，因为宏变量的值都为char  
注意下面例子中第二个实参，中间有空格，到了形参那里，直接作为var后面的char使用

```
%macro printdsn(dsn,vars);     proc print data=&dsn;        var &vars;     title "Listing of %upcase(&dsn) data set";     run;%mend;
%printdsn(sasuser.courses,course_code course_title days)
%printdsn(,course_code course_title days) --参数可以为空，直接逗号
```

下面的例子是错误的，会提示找不到"sasuser.courses"这个dataset，包含了引号：

```
%printdsn("sasuser.courses",course_code course_title days)
```
有keyword的定义，但没有缺省值：

```
%macro printdsn(dsn=,vars=);
```


### 9.调用宏(使用keyword，可以乱序)
可以在定义宏的时候，设置默认的缺省值：

```
%macro printdsn(dsn=sasuser.courses,vars=course_code                 course_title days);	proc print data=&dsn;	var &vars;	title "Listing of %upcase(&dsn) data set";run; %mend;
```
带keyword调用，实参的顺序可以乱序：

```
%printdsn(dsn=sasuser.schedule, vars=teacher           course_code begin_date)
```
使用默认的缺省值：

```
%printdsn()
```



### 10. 上面两种模式mix调用宏

调用的时候，keyword出现之前的所有实参，必须按照顺序：

```
%macro-name(value-1<,...,value-n>,              keyword-1=value-1<,...,keyword-n=value-n>)
```

### 11.使用PARMBUFF Option调用宏
在函数后面加/parmbuff  
调用时的实参全部存放在宏变量&syspbuff中  
在logic中，可以使用下面代码依次提取参数

```
      %let num=%eval(&num+1);
      %let dsname=%scan(&syspbuff,&num);
```

```
%macro printz/parmbuff;
   %put Syspbuff contains: &syspbuff;
   %let num=1;
   %let dsname=%scan(&syspbuff,&num);
   %do %while(&dsname ne);
      proc print data=sasuser.&dsname;
      run;
      %let num=%eval(&num+1);
      %let dsname=%scan(&syspbuff,&num);
   %end;
%mend printz;
```

### 12.创建全局宏变量
整个SAS的session都有效  
You can create a global macro variable with：
1. a %LET statement (used outside a macro definition)  2. a DATA step that contains a SYMPUT routine  3. a DATA step that contains a SYMPUTX routine   
4. a SELECT statement that contains an INTO clause in PROC SQL  5. a %GLOBAL statement （在宏定义的里面和外面都可以）

```
%macro printdsn;
	%global dsn vars;	%let dsn=sasuser.courses;	%let vars=course_title course_code days;	proc print data=&dsn;		var &vars;	title "Listing of &dsn data set";run; 
%mend;
```
如果想要删掉某个宏变量：

```
%symdel dsn;
```

### 13.Local宏变量
只在当前的宏定义中才有效  
You can create local macro variables withparameters in a macro definition  
1. a %LET statement within a macro definition2. a DATA step that contains a SYMPUT routine within a macro definition3. a DATA step that contains a SYMPUTX routine within a macro definition (beginning in SAS 9)4. a SELECT statement that contains an INTO clause in PROC SQL within a macro definition5. a %LOCAL statement.


> Note: The SYMPUT routine and the SYMPUTX routine can only create a local macro variable if a local symbol table already exists. If no local symbol table exists when the SYMPUT routine or SYMPUTX routine executes, it will create a global macro variable.

```
%let dsn=sasuser.courses;%macro printdsn;
	%local dsn;
	%let dsn=sasuser.register;	%put The value of DSN inside Printdsn is &dsn;%mend;%printdsn%put The value of DSN outside Printdsn is &dsn;
```

### 14.宏变量的引用和update机制
1. 如果是在open code：  
不管是引用还是update，都只在全局宏变量中查询。（在open code的编译的时候，还没有编译到宏定义的部分，不存在local macro table）  

2. 在宏定义里面：  
	* 先在local macro table中查找，有的话，就直接引用或者update，结束；没有的话，下一步。
	* 在global macro table中查找，有的话，引用或者update，结束；没有的话，下一步。
	* 如果是upadte的话，新创建一个全局的宏变量。如果是引用的话，在log报错。

eg:

```
%macro outer;     %local variX;     %let variX=one;     %inner%mend outer;%macro inner;     %local variY;     %let variY=&variX;%mend inner;```submit the following code:```	%let variX=zero;	%outer
```
下面是上面例子的执行步骤：    

1. 在最外面的地方，全局变量赋值：  

	**Global  Symbol Table**
	
	  var|value  
	----- | ----  
	variX | zero  

2. 按照顺序执行，outer，定义outer中local variX

	**Global  Symbol Table**
	
	var|value     
	----- | ----  
	variX | zero  
	
	**Outer Local  Symbol Table**
	
	var|value     
	----- | ----  
	variX | one

3. 在outer中调用宏inner，定义inner中local variY  
%let variY=&variX;赋值的时候，先在Inner Local Symbol Table中查找variX，如果有，就使用；如果没有，就去上一层nest的宏，也就是Outer Local Symbol Table中查找variX，如果有，就使用；如果还是没有，就再往上，一直到global symbol table，如果还没有，就报错。

	**Global  Symbol Table**
	
	var|value     
	----- | ----  
	variX | zero  
	
	**Outer Local  Symbol Table**
	
	var|value     
	----- | ----  
	variX | one
	
	**Inner Local  Symbol Table**
	
	var|value     
	----- | ----  
	variY | one

4. 然后，再一层一层的结束，每次结束%mend一个宏，就删除一个Local Symbol Table

例子1：  

```
%let a=cat;%macro animal; 
	%let a=dog;%mend; 
%animal%put a is &a;
```
解释： 在宏animal里面有对宏变量的赋值，SAS首先查找local table中有没有，发现没有，就去global table中找，最后发现了，赋值成功，最后再open code中的put的结果是dog.

例子2：

```
%macro trans;	%let type=Airplane;	proc print data=sasuser.activities;		where trans="&type"; 
	run;	%location(Automobile)	%put type is &type; 
%mend;%macro location(type); 
	data _null_;		call symput('type', 'Train'); 
	run;%mend; 
%trans
```
解释： 首先调用trans，在宏trans中定义宏变量type（因为在global table中没有），在trans的local table中赋值为airplane。然后调用宏location，由于type是宏location的形参，在宏location的local table中是有type的定义的，symput的时候，就把Train赋值给宏location的local table中的type。但在宏trans的local table中的值没有变，还是Airplane。


### 15.相关的日志开关
用于在MPRINT打印nest宏的log：
> OPTIONS MPRINTNEST , NOMPRINTNEST;    


用于在MLOGIC打印nest宏的log：
> OPTIONS MLOGICNEST , NOMLOGICNEST;

### 16.宏里面的条件判断

```
%IF expression %THEN %DO;	text and/or macro language statements%END; 
%ELSE %DO;	text and/or macro language statements%END;
```

%IF-%THEN...和IF-THEN...  

IF-THEN...  
>1. 只能在data步中使用  
>2. 在logic表达式中可以使用data set中的var  
>3. 在data步中控制走向  
>4. 如果在宏过程中使用，里面的statement会放在input stack中  


%IF-%THEN...  
> 1. 在宏过程中使用，里面可以有宏变量  
> 2. logic表达式中可以使用宏变量，但不能使用data步的var  
> 3. 里面的statement会放在input stack中

### 17.一些有趣的写法
注意下面freq的写法，与tables的结合，可以控制是1way还是2ways

```
%macro counts (cols=_all_,rows=,dsn=&syslast);	title "Frequency Counts for %upcase(&dsn) data set";
	proc freq data=&dsn;
		tables		%if &rows ne %then &rows *;		&cols; 
	run;%mend counts;
```
2-ways输出：

```
%counts(dsn=sasuser.all, cols=paid, rows=course_number)
```

1-way输出：

```  
%counts(dsn=sasuser.all, cols=paid)
```

### 18.宏过程中的循环
和SAS中的类似：

```
%DO index-variable=start %TO stop <%BY increment>;	text%END;
```
可以控制创建多个data set

```
%macro readraw(first=1999,last=2005);
	%local year;
	%do year=&first %to &last;
	  data year&year;
	     infile "raw&year..dat";
	     input course_code $4.
	     location $15.
	     begin_date date9.
	     teacher $25.;
		run;
		
		proc print data=year&year;
		   title "Scheduled classes for &year";
		   format begin_date date9.;
		run;
	%end;
%mend readraw;
```

### 19.The %EVAL Function
eg:
![](https://cl.ly/7e0a9f4c2bab/Image%2525202019-05-20%252520at%25252011.57.35%252520%2525E4%2525B8%25258B%2525E5%25258D%252588.png)

%eval(&numer/&denom); --> 2/8 = 0.25 eval取整数部分  
%eval(&numer/&denom\*&denom); --> 0  
%eval(&denom\*&numer/&denom); --> 2  

eval不能转换为数值，当：  

1. 数值中包含小数点或者E  
2. SAS date和time型的数值  

%eval(2.4+8); --->  就会报错

### 20.%SYSEVALF
和%EVAL类似，但是可以计算float  

%SYSEVALF(2.4+8) --->  10.4


## Chapter 12 - Storing Macro Programs  
### 1.使用外部文件中定义宏
如果外部文件中有宏的定义，在INCLUDE的时候，就会被编译，在后面的程序中可以调用

```
%INCLUDE file-specification </SOURCE2>;
```
注：使用/SOURCE2可以在log中显示调用的外部文件中的宏，更清楚

### 1.5 将宏定义保存在指定的CAT下面
在File->Save As Object->Save
>Make sure that the Entry Type is set to SOURCE entry (SOURCE), then click Save.


### 2.CATALOG PROC
```
PROC CATALOG CATALOG=libref.catalog; 
	CONTENTS;QUIT;
```
eg:

```
proc catalog cat=work.sasmacr;     contents;     title "Default Storage of SAS Macros";quit;
```

### 3.使用CAT中的宏
注意下面catalog的值为4层，不要忘记最后还有一个entry-type,可以是source（就是上面1.5章节中的保存的那个source），也可以是macro。可以使用proc catalog去查看具体的type

```
FILENAME fileref		CATALOG ‘libref.catalog.entry-name.entry-type’;%INCLUDE fileref;
```
eg: 一个CAT下的一个宏

```filename prtlast catalog ’sasuser.mymacs.prtlast.source’;
%include prtlast;
proc sort data=sasuser.courses out=bydays;     by days;run;%prtlast
```
eg: 一个CAT下的多个宏

```
filename prtsort catalog ’sasuser.mymacs’;
%include prtsort(prtlast) / source2;
%include prtsort(sortlast) / source2;

data current(keep=student_name course_title begin_date location);
	set sasuser.all;
	if year(begin_date)=2001;
	diff=year(today())-year(begin_date);
	begin_date=begin_date+(365*diff);
run;

%sortlast(begin_date)
%prtlast
```

### 4.Using the Autocall Facility
把宏定义放在Autocall里面，不用include，可以直接调用  
系统发现调用了，会先去Autocall里面找，找到了就编译（默认编译好了存放在Work.Sasmacr下面），再执行  
注：如果调用的时候，发现SAS已经编译过了，就不会再取Autocall里面面找了，直接执行

使用下面option打开autocall的功能：

```
OPTIONS MAUTOSOURCE | NOMAUTOSOURCE;
```
指定Autocall的路径：

```
OPTIONS SASAUTOS=library-1;OPTIONS SASAUTOS=(library-1,...,library-n);
```

系统本身sasautos里面存储了一些默认的Autocall的路径，如果在使用自己的路径时，要合并添加，不要覆盖：

```
options mautosource sasautos=(’c:\mysasfiles’,sasautos);%prtlast
```

### 5.预先存储编译好的宏
通过option，先编译好宏，放在一个permanently的地方，下次可以直接调用  

打开存储开关：

```
OPTIONS MSTORED | NOMSTORED;```

指定存储的libref，不可以是work

```
OPTIONS SASMSTORE=libref;
```

在宏定义时，加上option

```
%MACRO macro-name <(parameter-list)> /STORE
	<DES=‘description’>;	text%MEND <macro-name>;
```

注意事项：  

1. 存放在libref下面的cat的名字只能是Sasmacr  
1. 编译好的宏不能在操作系统层面编译、copy，只能在SAS里面重新编辑、编译宏  
1. 建议将宏的源代码放在Autocall下面，或者放在编译好的宏的同样的cat下面。  可以通过宏定义后面的SOURCE开关，将宏的源代码也同时存储（必须先在前面的options中定义MSTORED），但是存储的宏的源代码在操作系统和SAS层面都看不到
```%macro macro-name<(parameter list)> /STORE SOURCE;
```> SOURCE开关不能用于存储nest的宏定义

### 6.调用预先存储的编译好的宏

1. 使用libname指定libref（里面包含Sasmacr的CAT）  
1. options中指定MSTORED和SASMSTORE=libref  
1. 调用宏  

>Only one permanent catalog containing compiled macros can be accessed at any given time.

### 7.访问预先存储的编译好的宏的源代码
之前的宏定义需要用SOURCE的option存储

```
%COPY macro-name /SOURCE <other option(s)>;
```
```
%copy words/source;
```

### 8.调用宏时SAS处理的步骤

1. 先在CAT work中查找有没有宏定义，如果没有
1. 如果MSTORED和SASMSTORE有定义，在指定的CAT下面查找，如果没有  
1. 如果MAUTOSOURCE和SASAUTOS有定义，在Autocall下面查找

