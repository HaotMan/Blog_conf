---
title: Tensorflow 两个交叉熵损失函数的区别
date: 2018/10/19 01:16:26 
categories: 

- [深度学习]

- [tensorflow]

tags: 

- 深度学习
- tensorflow
- python

permalink: tensorflow-cross-entropy-loss

---

## tf.nn.sparse_softmax_cross_entropy_with_logits

label：不含独热编码，shape：[batch_size, ]
logits：原始预测概率分布向量，shape：[batch_size, num_classes]
```python
logits = np.array([[0.3, 0.4, 0.3], [0.5, 0.2, 0.3], [0.1, 0.1, 0.8]])
labels = np.array([1, 0, 2])
```
<!--more--> 

这样的label参数决定了该函数只适合于每种样本只属于一种类别的情况

----
## tf.nn.softmax_cross_entropy_with_logits
label：独热编码，shape：[batch_size, num_classes]
logits：原始预测概率分布向量，shape：[batch_size, num_classes]

```python
logits = np.array([[0.3, 0.4, 0.3], [0.5, 0.2, 0.3], [0.1, 0.8, 0.1]])
labels = np.array([[0, 1, 0], [1, 0, 0], [0, 1, 0]])
```
独热编码的label参数决定了该函数可适合于每种样本属于多种类别的情况