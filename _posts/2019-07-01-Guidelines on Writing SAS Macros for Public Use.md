---
layout:     post
title:      Guidelines on Writing SAS Macros for Public Use 
subtitle:   SAS Study Notes
author:     vincent_c
image:			assets/images/guide-line1.jpg
categories: [ SAS, tips ]
catalog: true
tags: [SAS]
---  
如果你或者你的团队经常需要编写一些公开宏，供团队里其他的人使用，那么你可以参考一些文档。对于个人的编写者，部分内容或者准则也值得借鉴。  
注:
此文翻译自[http://www2.sas.com/proceedings/sugi29/047-29.pdf](http://www2.sas.com/proceedings/sugi29/047-29.pdf)，翻译水平有限，而且有部分内容没有翻译。  

## 摘要：

SAS宏应用的实用性是毋庸置疑的，当我们编写的宏被分享时，他们的价值就会大大增加。
    
考虑到码农为他们的团队提供了有效的方案时而节省的时间，宏的有效性就不能单单局限于“这个宏完成了它的特定目标”，而是应该体现在这个宏被其他人和其他程序有效地应用。

如果宏的编写、注释十分晦涩难懂，那它的实用性就大打折扣，即使这个宏的功能性或者创意性很强。宏的编写者在编写的时候，通常只是考虑自己的使用，而没有考虑这个宏可以被其他人使用，这点就造成了糟糕的设计。

本文主要针对宏的优秀设计方法提供了一个建议，比如如何使你编写的宏被被人方便的使用。本文的目标对象是SAS的入门者、正在学习SAS宏语言的人。


## 介绍：

众所周知，宏是SAS中应用性最广、最强大的功能之一。从表面上看，宏就是对一些重复性的编码进行了简单的替换，但宏在SAS中有着相当广泛的应用。

从完成重复任务，到复杂的self-contained的应用，宏都可以发挥着强大的作用。宏的应用性和灵活性是宏的主要特点。一旦你熟悉了宏语言，你就会发现编写一个宏，会让你在不同的场景中，有效的重复的利用一段复杂的代码。更重要的是，正确地编写宏可以让你和同事、世界上任何人分享。

在宏的逻辑上和语法上多做一些，少讨论一些，这会让你的宏有更强的公开应用性。通常，我编写宏的时候，我会考虑让它有很强的公开应用性，可以被别人有效地使用。缺乏考虑宏使用时的上下文，会让公开的宏有很多的问题。比如，一些公开的宏可以在不同的操作系统上使用，这可能会作者刚刚开始编写宏时的考虑不同。你的宏可能有很多不同种类的潜在的使用者。所以清楚的、正确的说明十分重要。

大部分情况下，你都应该可以正确地使用一个公开宏，尽可能少的出问题。最终，公开宏的目标应该是它能被所有有需要的人使用。如果一个公开宏的说明文档很差，或者复杂得难以使用，就算最有创意的、最强大的宏也没有多少实用性。

本文总结了一些如何使宏公开化的方法，其中很多方法都不是新的。在一些例子中，它们都被应用在好的变成实践中(see McConnell, 1993)。然而，基于我的经验来看，有很多原因都造成了公开宏经常缺少一些特定的属性。一般来看，有一个主要的原因是但初学者发现他们的宏可以正常作用时，他们就满足了，但他们没有考虑宏的鲁棒性和适用性。

而且，我也和其他人一样，书上和网上很多的例子都只是告诉我代码的逻辑和正确的语法。慢慢积累的经验让我们理解了很多设计上微妙的问题。

本文的建议都是从我这些年使用公开宏、编写公开宏而来。它们可能并不能适用于所有的情况。但它至少可以让你在编写公开宏时有些新的思考。

## 正文:

### 1. DOCUMENTATION 文档说明

说明文档对任何类型的编程语言都很重要，但对公开宏来说，它尤为重要。公开宏的潜在使用者需要知道它是怎么工作的，怎么被正确使用的。在开始编写宏时，一个说明文档的文档可以很好地帮助你标准化地描述一些重要信息。格式和分级的种类很多，但关于作者、目的、范围和设计的相关信息都是必不可少的。

下面是Whitney在1996年写的一个标准化的模板。

```

/*******************************************************************************
Macro name: In addition to the file name, include the full path of the
directory in which it resides.

Written by: Your name, as well as relevant contact information (email,
telephone number).

Creation date: The date of original program.

As of date: The date of the most recent revision.

SAS version: The version and platform used to write the macro.

Purpose: Clearly and concisely describe what the macro actually does.

Method: Describe the steps necessary to achieve the purpose. Additional comments may also be included in the code.

Format: List the components of the actual macro call. For example:
 %print_obs (data =
 , keep =
 , nobs =
 )
An added benefit of this is that users can simply cut and paste
the above code when they invoke the macro.

Required Parameters: Define all the required parameters needed for the macro
 to run properly, e.g.
 data - Name of SAS data set.

 keep - List of variables that you want printed.

Optional Parameters: Define all the optional parameters and any default settings.
 For example:
nobs - Number of observations to be printed. Default is 50.
Sub-macros

called: Are any additional macros called within the current macro. If so, list them and their purpose.

Data sets created: List the library and name(s) of any data sets created by the macro, if applicable.

Limitations: Discuss any limitations of the macro.

Notes: References, background information or rationale of the macro.
\
History: List bug fixes, coding changes, etc.

Sample Macro call: What the actual macro call would look like, e.g.
 %print_obs (data =mydata
 , keep = var1 var7
 , nobs = 100
 )
You may also want to consider including sample data/output to
fully demonstrate how the macro should work.
*********************************************************************************/
```

上面的模板中清楚地列举了一些点。在实际的使用中，你可以根据实际情况合并或者重新编排这些点。你也可以去掉一些不适合的点。虽然上面模板中的注释描述了宏的目标和设计，你还是是尽量根据你实际的宏代码进行self-documenting，可以通过使用清楚的命名风格来实现。

### 2. 命名习惯

首先，宏的名称应该清晰地描述他是做什么的。
比如，如果你编写的宏注要是用于找出一些重复的记录，你可以使用PRINT_DUPS，这比你使用CHECKDATA或者DUPLICATES都要好，

宏参数的命名也同样如此，要能帮助你更好的理解宏。我发现在SAS语句或者option后面命名宏参数很有用，比如，我常常使用data_用来命名和数据集有关的参数。不管你用什么方式命名，但请在你所有的代码中保持一致。使用一个通用的命名习惯，可以让你的宏更容易被别人使用。

最后，在宏代码中，也要使用这些风格来命名宏里面的变量，这将帮助你在以后的时间里理解和调试你的宏代码。比如，不要使用i和j作为计数器，要根据计数器的使用目的来命名，比如YEARS或者VISITS。

### 3. 宏的options

另外一点需要考虑的是，使用什么样的命令或者options来将宏的信息打印到log中。相关的options有三种：

* MPRINT
* MLOGIC
* SYMBOLGEN

其中

* MPRINT会将宏创建的SAS语句打印出来，
* MLOGIC对于逻辑判断和%Do循环是最有效的。
* SYMBOLGEN会将宏中所有宏变量的解析结果打印出来。

在上面三个options中，我一般只使用MPRINT，它可以在log中显示宏究竟做了什么。另外两个options一般会输出很多的信息，有时会使log的可读性变差。需要说明的是，他们三个对编写宏和调试宏都很有用。

### 4. 打印调式信息

另一个也很好的方法是使用%PUT，除了可以打印指定的宏变量，它还可以打印_USER_ ,_LOCAL_ ,_GLOBAL_ ,_AUTOMATIC_和_ALL_的值。

另外，有一些其他的语句也有同样的作用，比如，当你想知道自定义的宏变量的值时，你可以使用：

```
%put You defined the following macro variables: ;
%put _user_ ; 
```

除此之外，还有很多使用%PUT语句的场景，比如记备忘录，列举初始options，告警等等。

### 5. 增加一些信息让你的宏更有可读性

除了上述建议，你还可以根据宏的实际代码增加注释。你可以使用/* 注释内容 */，或者%\* 注释内容。但记住不要使用\* 注释内容，那是给open code注释使用的。

对于长的或者复杂的宏，你可以根据需要增加一些说明文档。这时，最好专门创建一个文档，或者一个单独的文本文件或者html文件。

一般来说，你在其中增加一些关于理论、算法、使用方法或者输出的说明。创建了专门的说明文档之后，记住要在你的宏中写好引用。

如果你创建了一系列内容相关的宏，建议再写一个readme文件，用于说明每个文件的用途和使用方法。如果你对你的宏的目标使用者定位不清楚的话，这一点尤为重要。

### 6. ERROR CAPTURE 错误捕获

如果调用不当的话，任何宏都会出错误。怎么对待错误会大大影响易用性。一方面，你可以专注于让你的宏完成指定目标，另一方面，你也可以写一些错误捕获代码来处理错误。

这样做有两个原因。

第一，它可以在调试时更有效地帮助你，毕竟有时SAS的log输出比较隐晦。
对于那些不熟悉你的宏的用户，他们也会感到方便。

第二，当错误出现时，它可以让你避免很多麻烦，尤其是你在处理大的数据集或者密集性计算时。宏的前半部分需要多考虑考虑，因为后面很可能会由于缺失一个参数的值导致你的调试的焦头烂额。

虽然预防和正确的捕获所有类型的错误是一件困难的事情，但我们还是可以根据一些标准来尽可能的做到。

* **检查是否所有的宏参数都有值。这可以通过%LENGTH来实现。**

	比如：
	
	```
	%if %length(&data) = 0 %then %do ;
	 %put ERROR: Value for macro parameter DATA is missing ;
	 %goto finish ;
	%end ;
	/* other macro statements */
	%finish: 
	```
	
	如果一个变量的长度为0，则说明这个变量为空值。通过上面的方法可以正确的处理错误。
	
	使用%GOTO语句来跳转、而不用再处理后面不用的代码是很明智的。在最后使用一段扫尾代码来处理%GOTO跳转过来的logic很必要。


* **检查宏需要的实体是否都存在**

	比如：一个宏需要一个数据集作为输入，那么这个数据集就必须先存在。
	使用%SYSFUNC和EXIST很容易完成检查：
	
	```
	%let error = 0 ;
	%if %sysfunc(exist(&data)) = 0 %then %do ;
	 %put ERROR: data set &data does not exist ;
	 %let error = 1 ;
	%end ;
	/* other error checking code */
	%if &error = 1 %then %goto finish ;
	/* macro code */
	%finish: 
	```
	
	上面的代码和处理错误的过程有一点点不一样，上述的例子中定义了一个变量error，用来标志代码处理过程中是否出现了错误。在代码的最后，完成了所有可能出现的错误检查，再检查这个变量。
	
	这些例子都是一些比较简单的例子。根据宏的实际代码，错误捕获会相对比较复杂。

* **一些其他的检查项**

	除此之外，还有一些其他的检查项，比如检查变量是否是字符型，libname和format的命名是否符合SAS定义。
	建议在你的宏代码中进行相应的检查。
	
	有一些短的实用性的宏可以在另一个宏中定义。比如，你可以定义一个子宏来检查被引用的数据集是否已经存在。这样你的代码风格可以保持一致，还可以使你的代码看起来整洁。而且，它也可以被其他宏重复利用，完成最普通的检查，提供清楚的解释说明。
	
	另外一个比较有效的减少出错的方法是提供你的宏的变量的兼容性，比如，让你的宏代码处理字符变量的值的大小写，这比你要求你的用户指定好大小写要好的多。这可以通过使用%UPCASE来完成
	
	```
	%let debug = %upcase(&debug) ; 
	```
	
	你可以扩展一下，完成单次的首字母大小写兼容的问题，比如，Yes和yes是一样的
	
	```
	%let debug = %upcase( %substr(&debug,1,1 ) ) ;
	```

* **避免无意重写数据集**

	一个宏的完成方式多种多样，但一定不能影响其他的代码逻辑
	比如在你的宏当中创建了一个数据集，但不小心和其他的数据集重名了，这样就会覆盖原始数据集。一个比较常见的方法是使用下划线开头的方式来命名你的数据集，虽然有时不能保证万无一失，但还是可以大大降低错误的几率的。
	
	除此之外，你还可以在创建你的数据集之前，先检查一下这个数据集是否已经存在，如果已经存在，可以在log中报警，并采取相应的措施，比如中断你的代码。
	
	如果你的宏只是创建一个临时使用的数据集，那么在宏完成这个临时数据集的处理之后，要及时的删除它。这样做，不仅可以节省空间，还能避免数据集的冲突。

* **避免无意重写变量**

	另外一个容易出现冲突的就是全局变量。简单来说，全局变量可以在你的代码中的任何地方调用，比如open code。而局部变量只是在你的宏的逻辑处理中存在。
	
	```
	%let x = 5 ;
	
	%macro check ;
		%let x = 1 ;
	%mend check ;
	
	%check
	%put x = &x ; 
	```
	
	上面的代码中就犯了这个错误，程序中先在open code中定义了一个全局变量x，并赋值为5，但是在宏代码中，引用了x，根据SAS的处理逻辑，SAS在全局变量中发现有x的存在，于是宏代码的logic就改变了全局变量x的值。
	
	**解决的方法:**
	
	1. 在宏代码中，对于局部变量要使用local来提前定义
	
		```
		%let x = 5 ;
		
		%macro check ;
			%local x ;
			%let x = 1 ;
		%mend check ;
		
		%check
		%put x = &x ; 
		```
	
	1. 使用下划线开头的方式来命名全局变量
	
	1. 使用%SYMEXIST函数来检查变量是否已经存在


* **避免修改一些全局设置**

	1. 修改了全局的options
	
		如果你的宏修改了一些option的设置，那么在你的宏结束之前，应该把options恢复成原本的设置。
		
		**解决的方法:**
		
		先用getoption保存设置，在宏代码结束之前，使用options重新设置。
		
		```
		%macro test ;
			%local option ;
			%let option = %sysfunc( getoption( mprint, keyword ) ) ;
		
			options mprint ;
			/* macro code */
			options &option ;
		%mend test ; 
		```
	
	1. 修改了全局的titles或者footnotes
	
		如果你的宏修改了titles或者footnotes，可以：
		
		**方法一：**
		在你的宏之前，没有指定的titles和footnotes，但在你的宏中指定了titles和footnotes，那么在你的宏结束之前，清空titles和footnotes
		
		**方法二：**
		如果在你的宏之前，本来就有指定好的titles和footnotes，而且在你的宏中修改了titles和footnotes，那么在你的宏结束之前，应该把titles和footnotes恢复成之前的值。
		
		在DICTIONARY.TITLES或者SASHELP.VTITLE中可以获取到titles和footnotes的值，在你的宏修改titles和footnotes之前，先保存原先的值，等你的宏处理完之后，再恢复titles和footnotes的值。
		
		比如：
		
		```
		title1 'Original title';
		/* Next, within the macro, capture the title number and text of the title. */
		
		%macro test ;
			proc sql noprint;
				select number, text into: num ,:text
				from dictionary.titles
				where number = 1;
			quit;
		
			%let num = &num;
			%let text = &text ;
		
			/* The above %let statements are simply a convenient method for trimming the macro
		variables. Next comes the new title defined in the macro. 	*/
			
			title1 'Macro title';
			/* But before existing macro, restore the original title: */
			
			title&num "&text";
		%mend test; 
		```
		根据你的代码逻辑，根据number的值，保存和恢复指定的title和footnotes。比如你的代码只用到了title1，那就利用proc sql的where number = 1的条件，只保存和恢复title1。



### 7. 提高易用性

调用函数的方式主要有两种， 分别是keyword和positional，定义时的格式如下：

```
/* positional */ %macro recode (data, out, var, groups );

/* keyword */ %macro recode (data =, out =, var =, groups = ); 
```

调用时的格式如下：

```
/* positional */ %recode(sales, salesr, revenue, 5)

/* keyword */ %recode(data = sales, out = salesr, var = revenue, groups = 5 ) 
```

使用keyword方式指定参数的宏定义有一些好处，keywords的参数是自描述的，以上面的例子说明，你可以很清楚地知道，sales是一个数据集的名字，revenue是一个变量的名字，而并非数据集（因为在以keywords形式调用的时候，data=和var=就有自描述性质的）    
而positional形式，参数的位置很重要，不能乱序，而且你不会清楚参数的大概含义。
除此之外，keywords形式还有一个好处是：它可以定义参数的默认值。在下面的例子中，因为5是组数的一个大概率值，将默认值设置为5会比较方便。

```
%macro recode(data=, out=, var, groups = 5) ; 
```

另一方面，要保证宏的参数尽可能的少。太多参数会让你的宏非常难用。那么多少参数算多呢？根据经验主义，你的宏参数不要多于7个，如果你发现你的宏需要用到很多参数，那么你可以：

1. 根据你的宏的真实需要，确定宏真正需要的参数。有些参数，在你设计时觉得有用，但在宏功能实现上，没有什么真正的用途。
1. 如果所有的宏参数都有用，那么你可以筛选一下哪些参数可以设置默认值的，这样，别人调用你的宏的时候就不会面对一堆参数而一头雾水。
1. 在宏定义的时候，使用注释将参数代表的意义写出来，比如

```
%print_vars (data = /* name of input data set */
 , var = /* list of variables to print */
 , nobs = /* number of observations */
 , byvar = /* name of by variable */ 
 )
```


### 8. 权衡和最后的结论

在创建公开宏的时候，你要根据实际情况去权衡功能性和实用性。这一点也经常体现在不同的SAS的版本中。
比如说，SAS9.0有很多新的命令、功能和options，但是你要考虑到你的宏的用户有可能还是在使用SAS8.0。在大部分情况，你的宏都考虑到尽可能多的SAS版本，比如：在SAS9中使用的命令：

```
call symputx ('count', count) ; 
```

但是考虑到SAS9以前的版本的用户，你需要使用下面的代码来完成左对齐和去左右空格的功能：

```
call symput ('count', trim(left(put(count,8.)))) ; 
```

如果你还使用了SAS的其他组件的功能，那你要考虑到你的用户可能没有安装这些组件，比如：SAS/STAT、SAS/GRAPH、SAS/QC、SAS/IML、SAS/ETS、SAS/OR等等。利用PROC SURVEYSELECT很容易实现选择随机样本，但这个功能需要SAS/STAT功能组件。

记得提醒你的用户你的宏需要的前提条件。

还有就是你的宏的鲁棒性。你的宏可能会被使用在不同的环境中，这是一个值得去努力的目标。但是，你的宏要实现灵活性和鲁棒性需要更多的时间，也会增加宏的复杂性。最终，你会发现这一切都取决于实际情况。但如果你发现你的宏不能很好地处理所有潜在的问题，记得在你的宏中写明你宏的局限性。


### 9. 一些建议

1. 在你的代码中保持风格一致性，包括说明文档、命名风格等等。如果你的宏是多人团队协作完成的，更要考虑一致性。

1. 在发布你的宏之前，完整的测试你的宏，不仅要测试有没有bug，还要考虑到功能性和效率。有一次，我的宏使用PROC FREQ来完成一些统计功能。这个宏在测试数据不多的情况下，可以正常使用，但在实际情况中，在百万数据面前，这个宏就瘫痪了，后来我不得不使用PROC SQL重写了一遍。

1. 如果你在编写之前，知道你的宏的使用者是谁，在编写之后，和之后都要和他们仔细商讨一下。这件帮助你能更好的实现宏的功能。

1. 把你写好的公开宏保存在一个公用、只读的地方。这可以防止一些无意的修改，而且每个人都可以正常使用你的宏。

1. 定期地复查、优化你的宏。随着你的知识、水平增加，你可能会找到更好的方法来优化你的宏。


