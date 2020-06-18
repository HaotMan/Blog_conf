---

title: Tensorflow的滑动平均模型
date: 2018/10/12 00:00:30 
categories: 

- [深度学习]

- [tensorflow]

tags: 

- 深度学习
- tensorflow
- python

permalink: tensorflow-sliding-average-model 

---

### **1. 什么是滑动平均法**
**基本原理**
滑动模型是一种简单的平滑预测技术。当一组数据在变化中存在较大波动起伏较大，不易显示出变化趋势，影响预测函数的平滑度时，使用移动平均法可以消除这些因素的影响，显示出事件的发展方向与趋势（即趋势线），然后依趋势线分析预测序列的长期趋势。

<!--more--> 


**简单滑动平均法**

简单移动平均的各元素的权重都相等。简单的移动平均的计算公式如下： Ft＝（At-1＋At-2＋At-3＋…＋At-n）/n式中，

　　·Ft：对下一期的预测值；

　　·n：移动平均的时期个数；

　　·At-1：前期实际值；

例子：
某类房地产2001年各月的价格如下表中第二列所示。由于各月的价格受某些不确定因素的影响，时高时低，变动较大。如果不予分析，不易显现其发展趋势。如果把每几个月的价格加起来计算其移动平均数，建立一个移动平均数时间序列，就可以从平滑的发展趋势中明显地看出其发展变动的方向和程度，进而可以预测未来的价格。
![](https://img-blog.csdn.net/2018101123391157?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01TRE5fdGFuZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**加权滑动平均法**

加权滑动平均模型给每个数据赋予不同的权重，将简单的移动平均法换为求取加权平均。


### **2、tensorflow中的滑动平均模型**
在tensorflow中，滑动平均相当于一个影子值(像是给参数加了影子，参数变化，影子随之追随逼近参数值），记录了每个参数一段时间内过往值的平均，增加了模型的泛化性。滑动平均不仅表现了当前值，还表现了过去一段时间内的平均值。

**滑动平均的计算**

影子 = 衰减率x影子 +（1-衰减率）x 参数 （影子初值=参数初值， 衰减率一般设置为非常接近1的数，比如0.999，0.9999等)

衰减率 = min{MOVING_AVERAGE_DECY，（1+轮数）/（10+轮数）}）

具体使用示例：

```python
import tensorflow as tf
# 1.定义变量及滑动平均类
# 定义一个32位浮点变量，初始值位0.0,这个代码就是不断更新w1参数，优化w1，滑动平均做了一个w1的影子
w1 = tf.Variable(0, dtype=tf.float32)

# 定义num_updates(NN的迭代轮数）,初始值为0，不可被优化，这个参数不训练
global_step = tf.Variable(0, trainable=False)

# 实例化滑动平均类，设衰减率为0.99，当前轮数global_step
MOVING_AVERAGE_DECAY = 0.99
ema = tf.train.ExponentialMovingAverage(MOVING_AVERAGE_DECAY,global_step)

# ema.apply后面的括号是更新列表，每次运行sess.run(ema_op)时，对更新列表求滑动平均值
# 在实际应用中会使tf.trainable_variables()自动将所有训练的参数汇总为列表
# ema_op = ema.apply([w1])
ema_op = ema.apply(tf.trainable_variables())

# 2.查看不同迭代中变量取值的变化
# 初始化变量，计算图节点要运用的会话实现
with tf.Session() as sess:
    # 初始化变量
    init_op = tf.global_variables_initializer()
    # 计算图节点运算
    sess.run(init_op)
    # 用ema.average(w1)获取w1滑动平均值
    # 打印出当前参数w1和w1的滑动平均
    print(sess.run([w1, ema.average(w1)]))
    # 参数w1赋值为1
    # tf.assign(A, new_number): 这个函数的功能主要是把A的值变为new_number
    sess.run(tf.assign(w1, 1))
    sess.run(ema_op)
    print(sess.run([w1, ema.average(w1)]))

    # 更新step和w1的值，模拟出100轮迭代后，参数w1变为10
    sess.run(tf.assign(global_step,100))
    sess.run(tf.assign(w1, 10))
    sess.run(ema_op)
    print(sess.run([w1, ema.average(w1)]))

    # 复制粘贴几次执行滑动平均节点的操作
    for i in range(100):
        sess.run(ema_op)
        print(sess.run([w1, ema.average(w1)]))

```
-------
**输出结果**

```
[0.0, 0.0]
[1.0, 0.9]
[10.0, 1.6445453]
[10.0, 2.3281732]
[10.0, 2.955868]
[10.0, 3.532206]
[10.0, 4.061389]
[10.0, 4.547275]
       .
       .
       .
[10.0, 9.997882]
[10.0, 9.998055]
[10.0, 9.998215]
[10.0, 9.998361]
```
可以看到，在每次参数w1变化的时候，影子参数都会随着迭代不断地向变化后的参数逼近。
因为影子参数平均了以往的参数可以有效地消除数值波动的影响，所以在测试集中使用影子参数代替原始变量会使得模型在测试集上表现得更为健壮。