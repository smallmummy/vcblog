---
layout:     post
title:      Some tips of Python(VC)
subtitle:   recordind study notes
author:     vincent_c
image: assets/images/study1.jpg
catalog: true
categories: [studeynotes, python ]
tags: [python]
--- 
本文记录了一些python的小技巧.

#### 1. 二向赋值
普通写法：

```
condition = False
if condition:
    x = 1
else:
    x = 0
print(x)
```

简化为

```
x = 1 if condition else 0
```

#### 2.千分位数字：

普通写法，不直观

```
num1=100000000
num2=100000000000

total=num1+num2
print(total)
```

使用下划线将数字按照千分位隔开，直观，而且不影响业务logic。  
print的时候，使用f'{total:,}'  按照千分位输出。

```
num1=100_000_000
num2=100_000_000_000
total=num1+num2

print(f'{total:,}')
```

结果如下：

>100,100,000,000  


#### 3. 关闭文件
养成习惯，读完了文件，如果不再操作文件，直接close，再处理

```
f=open('test.txt','r')
file_content=f.read()
f.close()

#the process of file content here
XXXXXXXXX
```
或者使用python里面的context manager:

```
with open('test.txt','r') as f:
	file_content=f.read()

#the process of file content here
XXXXXXXXX
```

#### 4.遍历dict以及相关

普通写法，自定义index，用于遍历或者输出：

```
names = ['vin', 'col', 'ken', 'den']

index = 0
for name in names:
    print(index, name)
    index += 1
```

输出结果如下：

> 0 vin  
> 1 col  
> 2 ken  
> 3 den  


可以使用python的enumerate函数，获取自动的index

```
names = ['vin', 'col', 'ken', 'den']


for index,name in enumerate(names):
    print(index, name)
```
即可获得和上面一样的结果。

注：  
if you dont want to start the index with 0,then use parameter "start="

```
names = ['vin', 'col', 'ken', 'den']


for index,name in enumerate(names,start=10):
    print(index, name)
```

结果如下：

> 10 vin  
> 11 col  
> 12 ken  
> 13 den  


#### 5.两个dict的mapping遍历

普通写法，取另一个dict中相对应的值  

```
names = ['vin', 'col', 'ken', 'den']
place = ['1st','2nd','3rd','4th']

for index,name in enumerate(names):
    print(f"the {place[index]} is {name}")
```
结果：
> the 1st is vin  
> the 2nd is col  
> the 3rd is ken  
> the 4th is den  


可以使用zip函数，pack，然后一一对应

```
names = ['vin', 'col', 'ken', 'den']
place = ['1st','2nd','3rd','4th']

for name,place in zip(names,place):
    print(f"the {place} is {name}")
```

一个pack的例子：

```
a,b=(1,2)
print(a)
print(b)
```

如果unpack了所有变量，但没有使用所有的，编译器会提示有变量没有被用到（不影响程序逻辑和运行）。对于没有用到的变量可以用下划线代替，告诉编译器，unpack，但后续logic上不用。( *_ 告诉python，后面所有的内容都不用了)

```
a , _ =(1,2)
print(a)

a , *_ =(1,2，3，4)
print(a)
```

可以使用unpack：

```
names = ['vin', 'col', 'ken', 'den']
place = ['1st', '2nd', '3rd', '4th']

for val in zip(names, place):
    n, p = val
    print(n,p)
```


#### 6.类属性和动态添加类属性

实例化类之后添加类的属性，再使用：

```
class person():
    pass

p = person()
p.name = 'val'
p.sex = 'female'

print(p.name)
print(p.sex)
```
结果如下:
> val  
> female  



下面的例子使用dict和setattr函数，动态的添加实例化后的类的属性，可以更灵活的配合logic:

```
class person():
    pass

p = person()
content={'name':'val','sex':'female'}

for key,value in content.items():
    setattr(p,key,value)

print(p.name)
print(p.sex)
```

还可以配合getattr函数，动态的取指定的类的属性：

```
class person():
    pass

p = person()
content={'name':'val','sex':'female'}

for key,value in content.items():
    setattr(p,key,value)

for key in content.keys():
    print(f"the {key} is {getattr(p,key)}")
```

#### 7.包的相关

在python下，先import，然后使用help(pack_name)，可以查看包的说明信息：

```
$ python3
Python 3.6.2 (v3.6.2:5fd33b5926, Jul 16 2017, 20:11:06) 
[GCC 4.2.1 (Apple Inc. build 5666) (dot 3)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import time
>>> help(time)
```
使用dir，可以查看包的属性和方法：

```
>>> dir(time)
['_STRUCT_TM_ITEMS', '__doc__', '__loader__', '__name__', '__package__', '__spec__', 'altzone', 'asctime', 'clock', 'ctime', 'daylight', 'get_clock_info', 'gmtime', 'localtime', 'mktime', 'monotonic', 'perf_counter', 'process_time', 'sleep', 'strftime', 'strptime', 'struct_time', 'time', 'timezone', 'tzname', 'tzset']
```


