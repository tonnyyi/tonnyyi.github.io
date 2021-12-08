---
title: python代码片段
tags:
  - python
  - 片段
categories:
  - python
date: 2021-11-05 09:58:32
---

### 列表映射后求平均值

```python
def average(lst, fn):
	return sum(map(fn, lst), 0.0) / len(lst)

# 调用
average([1, 3, 4], lambda x: x)
average([{'n': 1}, {'n': 3}, {'n': 4}, lambda x: x['n'])
```

形如`lambda parameters: expression`的表达式可以创建一个匿名函数, 有时这比先调用函数再当做参数传入要方便许多. 比如算平方`list(map(lambda x: x * x, [1, 2, 3, 4, 5, 6, 7, 8, 9]))`.

```python
# 使用lambda方式定义一个函数
g = lambda x: x * x
g(2)	# 4
(lambda x: x * x)(2)	# 4
```

`map`的类似应用

```python
def max(lst, fn):
    return max(map(fn, lst))

max([{'n': 1}, {'n': 3}, {'n': 4}, lambda x: x['n'])
```

