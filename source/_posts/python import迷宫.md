---

title: python import迷宫
date: 2019/06/25 08:43:22  
categories: python
tags: 

- python
- grammar
- import

permalink: python_maze

---



在https://wiki.woodpecker.org.cn/moin/MiscItems/2008-11-25上看到一个关于import机制的问题，觉得很经典，遂记下来

A.py
```python
from B import D
class C:
	pass
```

<!--more--> 

B.py

```python
from A import C
class D:
	pass
```
这时候执行A.py就会报错 ImportError: cannot import name 'D'
而将A.py中的'from B import D' 改成'import B' 便不会出现该错误

**执行A.py的过程如下：**
```b
1. 执行A.py中的from B import D
	由于是执行的python A.py，所以在sys.modules中并没有<module B>存在，
	首先为B.py创建一个module对象(<module B>)，
	 注意，这时创建的这个module对象是空的，里边啥也没有，
	在Python内部创建了这个module对象之后，就会解析执行B.py，其目的是填充<module B>这个dict。	

2. 执行B.py中的from A import C
	在执行B.py的过程中，会碰到这一句，
	首先检查sys.modules这个module缓存中是否已经存在<module A>了，
	由于这时缓存还没有缓存<module A>，
	所以类似的，Python内部会为A.py创建一个module对象(<module A>)，
	然后，同样地，执行A.py中的语句

3. 再次执行A.py中的from B import D
	这时，由于在第1步时，创建的<module B>对象已经缓存在了sys.modules中，
	所以直接就得到了<module B>，
	但是，注意，从整个过程来看，我们知道，这时<module B>还是一个空的对象，里面啥也没有，
	所以从这个module中获得符号"D"的操作就会抛出异常。
	如果这里只是import B，由于"B"这个符号在sys.modules中已经存在，所以是不会抛出异常的。
```

ZQ图解
![ZQ图](https://img-blog.csdnimg.cn/20190625084442568.png?type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01TRE5fdGFuZw==,size_16,color_FFFFFF,t_70)