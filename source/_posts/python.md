---
title: python
tags:
  - python
categories:
  - python
date: 2018-04-20 09:55:25
---

## 基础
### 数据类型
6种标准数据类型:<!-- more -->
- **Boolean**: True, False. 任何对象都可以当做boolean型来验证, 默认一个对象被当做true, 除非它的class定义了一个返回False的`__bool__`方法 或者 一个返回0的`__len__`方法. 满足下列条件的, 对象会被当做false:
    - 对象为`None` 或 `False`
    - 任何值为0的数字类型: 0, 0.0, 0j, Decimal(0), Fraction(0, 1)
    - 空的序列和集合: "", (), [], {}, set(), range(0)
- **Number**: 数字, 支持 int, float, complex(复数), **Boolean是一种数字的子类型**. float通常使用C中的double实现, 范围与精度与系统有关, 可以通过`sys.float_info`查看. 
    ```python
    a, b, c, d, e = 100, 1024L, 3.1, True, 4+3j
    print(type(a), type(b), type(c), type(d), type(e))
    ```
- **String**: 字符串就是字符的序列, 
    - 使用单引号(')
    - 双引号(")
    - 三引号(''')(多行字符串)-
    - 转义符(\\)
    - 自然字符串(r或R), 如: r'This is a line. \n'
    - Unicode字符串(u或U), 如: u'Hello啊'
    ```python
    name = "Swaroop"
    if name.startswith("Swa"):
        print("yes")
    if 'a' in name:
        print("yes")
    if name.find("war) != -1:
        print("yes")
    
    delimiter = "_"
    names = ["Tom", "Sam"]
    delimiter.join(names)   # Tom_Sam
    ```
- **List**: 列表
```python
names = ["Tom", "John", "Alice", "Bob"]
print(len(names))
print(names.append("Sam"))
names.sort()
del names[0]
for i in names:
    print(i)
    
list(range(1, 10))
# 列表生成式
print [x * x for x in range(1, 11) if x % 2 == 0]       
print [m + n for m in 'ABC' for n in 'XYZ']     #两层循环

a, b, c = [1, 3, 9]     # 变量个数 必须 与元素个数一致
print a, b, c
a, b, c = (3, 6, 9)
print a, b, c
```
- **Tuple**: 元组, 与列表相似, 但元组是不可变的
```python
names = ("Tom", "Mike", "Tony")

age = 22
name = "Tom"
# 最常用的是打印
print("%s is %s years old" % (name, age))
```
- **Sets**: 集合
```python
s = set([1, 1, 2, 2, 3, 3]) #重复元素被自动过滤
s.add(4)
s.remove(1)

s1 = set([1, 2, 3])
s2 = set([2, 3, 4])
s1 & s2     # 取交集
s1 | s2     # 取并集
```
- **Dictionary**: 字典
```python
addr = {
    "Tony": "tony@qq.com",
    "john": "john@baidu.com",
    "sam": "sam@163.com"
}
print(addr["john"]) # 当没有john属性时会抛异常
print(len(addr))
# 遍历
for name, mail in addr.items():
    print("%s's mail is %s" % (name, mail))
    
# 增加属性
addr["Tom"] = "tom@xx.com"

# 键 值
addr.keys()
addr.values()

del addr["sam"]
pop addr["Tom"]
addr.clear()
del addr

# 判断是否包含属性
if "Tonny" not in addr:
    print("no Tonny")
    
# 推荐该方式获取属性值
print(addr.get("Tonny", "unknown"))
```

其中
- **不可变数据**: Number, String, Tuple, Sets
- **可变数据**: List, Dictionay

互相转换
```python
int("23")
long("0xFE", 16)
print float("3.14")
ord("a") # 字符转成整数值
oct(12) #转成8进制, 字符串格式
hex(12) #转成16进制, 字符串格式
str(122)    
chr(97) # 转成字符, 'a'
chr(0x597d) # 转成unicode字符, '好'
unichr(0x597d) # 转成unicode字符, '好'

atuple = (123, 'xyz', 'zara', 'abc')
print list(atuple)   # 创建列表
print tuple([1,2,3,4])  # 创建元组
print set('runoob')     # 转换为可变集合
print frozenset('runoob')   # 转换为不可变集合
print dict(a='a', b='b', t='t')     # 创建一个字典
```

序列切片(list/tuple/string)
```python
shopList = ["apple", "mango", "carrot", "banana"]
shopList[0]
shopList[-1]
shopList[1:3]
shopList[2:]
shopList[:3]
newList = shop[:]   # 复制新的序列
shop[:4:2]  # 前4个元素, 每2个取一个
"HelloWorld"[::2]   # 'Hlool'
```

### 操作符
#### and, or, not
 `x or y`, `x and y`, `not x`, `not`是低优先级操作父符, 所以: `not a == b` 等于 `not (a == b)`
#### 比较 
`<`, `<=`, `>`, `>=`, `==`, `!=`, `is`, `is not`
除了数字类型, 不同类型对象的比较结果永远不相等
#### 运算
`+`, `-`, `*`, `/`(得到完整结果), `//`取商, `%`取余, `<<`, `>>`, `&`, `|`, `^`异或, `~`翻转

### 语法
#### 标识符
数字, 字母, 下划线, 首字符不能是数字. 大小写敏感

#### if
```python
a = 12
if a > 10:
    print(a)
elif a < 5:
    print(a)
else: 
    pass    # pass:空的语句块, 保证没有语法错误
```
#### for
可应用于list, set, tuple, dict, str 和 生成器. 他们统称为可迭代对象: `Iterator `. list, dict, str等是Itrable, 但不是Iterator. 因为Python的Iterator对象表示的是一个数据流, 它可以被next方法调用, 直到抛出StopIteration错误. Iterator是一个惰性序列, 只在需要时才计算下一个值.
凡是可作用于for循环的对象都是Iterable类型；
凡是可作用于next()函数的对象都是Iterator类型，它们表示一个惰性计算的序列；
集合数据类型如list、dict、str等是Iterable但不是Iterator，不过可以通过iter()函数获得一个Iterator对象。
for in调用对象的`__iter__`方法, 获得迭代器对象.
```python
for i in range(0, 3):
    if i == 1:
        continue
    print(i)
else:
    print("Done")

d = {'a': 1, 'b': 2, 'c': 3}
# for v in d.values()  
# for k,v in d.items()
for k in d:     # 迭代字典时, 返回key. 
    print(k)

#判断对象是否可迭代
from collections import Iterable, Iterator
isinstance([1,2,3], Iterable)

# enumerate可以把list变成索引-元素对
for i,value in enumerate(['a', 'b', 'c']):
    print(i, value)
```
#### while
```python
count = 0
while count < 3:
    print(count)
    if count == 1:
        break
    count += 1
else:   # 当while正常结束, 即不是通过break/return结束时执行
    print("Done")
```
#### import
```python
import os

# 
try:
    import cPickle as pickle
except ImportError:
    print("no cPickle")
    import pickle
```
#### try
```python
fh = None
try:
    fh = open("testfile", "r")
    print(fh.read())

    if fh:
        raise Exception("Invalid level!", fh)   # 手动抛出
    except (IOError, Exception):
        print("Error: 没有找到文件或读取文件失败")
    except EnvironmentError as e:
        print(e)
    else:   # 没有任何异常抛出
        print("内容写入文件成功")
    finally:
        if fh:
            print("close")
            fh.close()
```
#### with
```python
# 打开文件, 不好的方式
f = None
try:
    f = open("testfile", "r")
    f.read()
finally:
    if f:
        f.close()

# 好的方式
with open("testfile") as f: #自动关闭
    '''with后的对象必须包含__enter()和__exit()__方法, 
    程序退出以及异常时__exit()__方法被执行'''
    f.read()
```

### 函数
#### 定义
```python
def max(a, b):
    '''方法的文档写在第一行, 
    用三个引号包裹'''
    
    global x    # 函数内定义全局变量, 不建议
    
    if not isinstance(a, (int, float)):
        raise TypeError("bad operand type")
    return a if a > b else b;   # 没有三目表达式
        
print(max(5, -1))
print(max.__doc__)  # 获取函数注释
```
#### 返回多个元素
因为在语法上, 返回一个元组是可以省略括号的, 所以函数返回多个值其实返回的是一个元组.
```python
def move(x, y, direction='forward'):
    if direction == "forward":
        return x+1, y
    if direction == "backward":
        return x-1, y
    if direction == "up":
        return x, y+1
    if direction == "down":
        return x, y-1
    else:
        raise Error("")
x, y = move(0, 0)
print(x)
print(y)
```
#### 默认参数
**只有在形参列表末尾的那些参数才可以有默认参数**, **默认参数必须指向不可变对象**
```python
def hi(name, msg='nothing'):
    print(name, "says:", msg)
hi("John")  # John says: nothing

def hi(name="Nobody", msg="Nothing"):
    print(name, "says:", msg) 
hi("John")  # John says: Nothing
hi(msg="Good")  # Nobody says: Good

def hi(a, b="doing", c="nothing"):
    print(a, b, c)
hi("John")  # John doing nothing
hi("John", "eating")    # John eating nothing
hi("John", c="something")   # John doing something
hi(b="eating", a="Tom") # Tom eating nothing

# 错误
def hi(name="Nobody", msg):
    print(name, "says:", msg)
hi(name="John", msg="Hello")

# 默认参数在函数定义时, 值就已经被计算出来了. 所以默认参数必须指向不可变对象.
def add_end(l=[]):
    l.append("END")
    return l
add_end()   # END
add_end()   # END, END

# 修复
def add_end(l=None):
    if l is None:
        l = []
    l.append("END")
    return l
```
#### `*args`(可变参数), `**xargs`(关键字参数) 和 `*`(命名关键字参数)
```python
def sum(*args):
    # 函数内接到的args是一个元组
    r = 0
    for n in args:
        r += n
    return r
sum()
sum(1, 3, 1)
nums = [11, 3, 10]
sum(*nums)      # 把list或tuple类型元素转成可变参数
```
同理, 使用\*\*将dict转成关键字参数
```python
def info(name, age, **kwargs):
    # 函数内获取到的kwargs是一个字典
    print("name:", name, "age:", age, "other", kwargs)
info("John", 10)
info("John", 10, city="London")

extra = {"city":"London", "job":"Engineer"}
info("John", 10, **extra)
```

如果要限制关键字参数的名字, 就可以使用命名关键字参数, 需要一个特殊分隔符`*`, 其后的参数被视为命名关键字参数.
```python
def info(name, age, *, city, job):
    print(name, age, city, job)
info("John", 10, city="London") #报错, 缺少job参数值
extra = {"city":"London", "job":"Engineer"}
info("John", 10, **extra)
```
```python
# 如果已经有一个可变参数, 则不再需要*分隔符
def info(name, age, *args, city, job):
    print(name, age, city, job)
info("John", 10, city="London") #报错, 缺少job参数值
extra = {"city":"London", "job":"Engineer"}
info("John", 10, **extra)
```
```python
# 任意长度的参数
def concat(*args, sep="/"):
    return sep.join(args)
concat("abc", "123")
concat("abc", "123", sep=".")

def fun(id, *args, **kwargs):
    print ("id = ", id)
    print ("args = ", args)
    print ("kwargs = ", kwargs)
    print("-------------------")  
fun(1,2,3,4)
fun(1, a=1,b=2,c=3)
fun('a','b','c', a=1,b=2,c=3)

a = (1,2,3,4)
b = {'a':1,'b':2,'c':3}
fun(*a, **b)
'''
id =  1
args =  (2, 3, 4)
kwargs =  {}
-------------------
id =  1
args =  ()
kwargs =  {'a': 1, 'b': 2, 'c': 3}
-------------------
id =  a
args =  ('b', 'c')
kwargs =  {'a': 1, 'b': 2, 'c': 3}
-------------------
id =  1
args =  (2, 3, 4)
kwargs =  {'a': 1, 'b': 2, 'c': 3}
-------------------
'''
```
#### 高阶函数
如果一个函数的参数列表含有函数, 则称该函数为高阶函数
```python
# add是高阶函数
def add(a, b, f)
    return f(a) + f(b)
```
##### map/reduce
map入参(function, sequence, *sequence_1), map作为高阶函数, 把运算规则抽象了. map返回的是`Iterator`
```python
list(map(str, [1, 2, 3]))   # ['1', '2', '3']
```
reduce(function, sequence, initial=None), 把一个函数作用在一个序列上, 该函数必须接收两个参数.
```python
reduce(f, [a, b, c]) = f(f(a, b), c)
# 累加
reduce(lambda x, y: x+y, [1, 2, 3, 4, 5])
# [1, 3, 5, 7, 9]变换成整数13579
reduce(lambda x, y: x*10 + y, [1, 3, 5, 7, 9])

# "123" -> 123
from functools import reduce
reduce(lambda x, y: x*10 + y, map(lambda x: int(x), "123"))
```
##### filter
filter也接收一个函数和一个序列, filter把函数作用于每个元素, 然后根据返回值为True或False来决定是否保留该元素.
```python
#保留偶数
list(filter(lambda x: x%2 == 0, [1, 2, 3, 4, 5, 6]))
```
##### sorted
排序
```python
sorted([43, -122, -12, 3, 123])
sorted([43, -122, -12, 3, 123], key=abs)    # 按绝对值排序
sorted(['bob', 'about', 'Zoo', 'Credit'], key=str.lower)    # 忽略大小写排序
sorted(['bob', 'about', 'Zoo', 'Credit'], key=str.lower, reverse=True) #倒序
# 按姓名排序
sorted([('Bob', 75), ('Adam', 92), ('Bart', 66)], key=lambda x:x[0].lower())
```

### 模块
```python
from sys import argv as arg     # 引用其他模块

if __name__ == "__main__":
    print("当前是主模块, 直接被用户运行")
else:
    print("当前模块是被其他模块引用的")
```

### 对象

### 其他
#### lambda

#### 函数式

#### 闭包

#### 生成器
可以通过列表生成式创建一个列表, 但由于内存限制, 容量有限且浪费. 可以创建生成器, 保存算法, 在循环过程中推断
```python
# 方式1: 把生成式的`[]`换成`()`
g = (x * x for x in range(10))
next(g) # 直接调用next()获取下一个返回值, 如果递归结束会抛出StopIteration错误
# 更常用的做法是在for循环中获取
for i in g:
    print i

# 方式2: yield关键字
# 斐波那契数列
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        print(b)
        a, b = b, a+b
        n += 1
#使用生成器
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        yield b  #每次next()遇到yield返回, 再次执行时从上次返回处继续执行
        a, b = b, a+b
        n += 1
f = fib(5)
next(f)
next(f)

# 杨辉三角
def triangles():
    b = [1]
    while True:
        yield b
        # b = [0] + b + [0]
        # b = [b[i] + b[i + 1] for i in range(len(b) -1)]
        b = [1] + [b[i] + b[i+1] for i in range(len(b)-1)] + [1]
n = 0
for t in triangles():
    print(t)
    n = n + 1
    if n == 10:
        break
```

## 核心
通过`dir()`函数列出模块定义的标识符, 包括函数, 类 和 变量
```python
import sys
print(dir(sys)) 
```
### collections
collections是Python内建的一个集合模块, 提供了许多有用的集合类
#### `namedtuple`
```python
p = (1, 2)   # 这表示一个坐标, 但单看(1, 2)不够明确, 定义1个class又太奢侈

from collections import namedtuple
Position = namedtuple('Position', ['x', 'y'])   # 属性个数任意
p = Position(1, 2)
p.x
p.y
```
#### `deque`
`list`结构查找快, 但插入和删除较慢. `deque`是为了实现高效插入和删除操作的双向列表, 适用于队列和栈
```python
from collections import deque
q = deque(['a', 'b', 'c'])
q.append('m')   # 尾部删除
q.appendleft('x')   #头部插入
print(q)    # deque(['x', 'a', 'b', 'c', 'm'])
q.pop()     #尾部删除
q.popleft()     # 头部删除
```
#### `defaultdict`
使用`dict`时, 如果key不存在会抛出`KeyError`, 使用`defaultdict`可以返回默认值
```python
from colelctions import defaultdict
dd = defaultdict(lambda: 'N/A')     # default(lambda:'N/A', **dict), 基于已有字典创建
dd['key1'] = 'abc'
pirnt(dd['key1'])   # 'abc'
print(dd['key2'])   # 'N/A'
```
#### `OrderedDict`
```python
# dict的key是无序的, 在迭代时无法确定key的顺序
d = dict([('c', 3), ('a', 1)])

od = OrderedDict([('c', 3), ('a', 1)])
od['b'] = 2
print(od.keys())   #odict_keys(['c', 'a', 'b']), key顺序是按插入的顺序排列, 而不是key本身的排序
```
#### `counter`
计数器, dict的一个子类
```python
from collections import Counter
c = Counter()
for ch in 'programming':
    c[ch] = c[ch] + 1
    
print(c)    # Counter({'r': 4, 'g': 4, 'm': 4, 'p': 2, 'o': 2, 'a': 2, 'i': 2, 'n': 2})
```

### `struct`
python没有专门处理字节的数据类型, 但`b'str'`可以表示字节, 所以, 字节数组=二进制str. 在python中, 如果要把一个32位无符号整数变成字节, 要这么写
```python
n = 10240099
b1 = (n & 0xff000000) >> 24
b2 = (n & 0xff0000) >> 16
b3 = (n & 0xff00) >> 8
b4 = n & 0xff
bs = bytes(b1, b2, b3, b4)
```
好在python提供了一个`struct`模块来解决`bytes`和其他二进制数据类型的转换
```python
import struct
struct.pack('>I', 10240099)
```
第一个参数是处理指令, '>I'是指: `>` 表示字节顺序是大端, `I`表示4字节无符号整数
```python
struct.unpack('>IH', b'\xf0\xf0\xf0\xf0\x80\x80')   # (4042322160, 32896)
# IH: 依次变为I:4字节无符号整数, H:2字节无符号整数
```

### `hashlib`
```python
import hmac     #还有hmac可用
import hashlib

md5 = hashlib.md5()
md5.update("Hello".encode('utf-8')) # 数据量大时可以多次update
print(md5.hexdigest())

sha1 = hashlib.sha1()   # 还有sha256, sha512
sha1.update("Hello".encode('utf-8'))
print(sha1.hexdigest())
```

### `itertools`
提供了非常方便的用于操作迭代对象的函数
```python
import itertools

c = itertools.count()   # 还可以指定起始值和步长
for i in c:     # c会一直生成数据, c是一个无限的迭代器
    print(i)

    if i == 3:  
        break
# 输出 1 2 3

cs = itertools.cycle('ABC')
for c in cs:   #又一个无限迭代器
    print(c)
# 输出  'A' 'B' 'C' 'A' 'B' 'C' 'A' 'B' 'C'...

r = itertools.repeat('A', 3)    # 可以限定重复次数
for i in r:
    print(i)
# 输出 'A' 'A' 'A'

# 可以使用takewhile函数获取一个有限的序列
c = itertools.count()
ns = itertools.takewhile(lambda x: x < 3, c)
list(ns)    # [0, 1, 2, 3]
```
#### 其他有用的函数
`chain()`可以把一组迭代对象串联起来, 形成一个更大的迭代对象
```python
for i in itertools.chain('abc', '123')
    print(i)
# 输出 'a' 'b' 'c' '1' '2' '3'
```
`groupby()`把迭代器中相邻的重复元素挑出来放一起
```python
for key, group in itertools.groupby('aabbccaaa'):
    print(key, list(group))

# 输出: 
a ['a', 'a']
b ['b', 'b']
c ['c', 'c']
a ['a', 'a', 'a']
# 挑选规则可以通过函数指定, 只要返回值相同, 则函数认为元素相同.
for key, group in itertools.groupby('AaaBb', lambda x: x.upper()):
    print(key, list(group))
# 输出
A ['A', 'a', 'a']
B ['B', 'b']
```

### contextlib
文件打开后需要关闭, 可以在`finally`里关闭, 也可以使用`with`语法, 自动关闭. 所有正确实现了上下文管理的对象, 都可以用于`with`语句. 实现上下文是通过`__enter__`和`__exit__`这两个方法实现的. 但依然繁琐, python的标准库`contextlib`提供了更简单的写法.
```python
from contextlib import contextmanager

class Query(object):
    def __init__(self, name):
        self.name = name
    
    def query(self):
        print("This is %s" % self.name)

@contextmanager
def create_query(name):
    print(">> before")  # 方法执行前
    q = Query(name)
    yield q
    print("<< after")   # 方法执行后

with create_query("Tom") as q:
    q.query()
    
# 输出
>> before
This is Tom
<< after
```
`@contextmanager`接收一个生成器(generator). 
如果一个对象没有实现上下文, 可以使用`closing()`来把该对象变为上下文的对象.
```python
from contextlib import closing
from urllib.request import urlopen

whth closing(urlopen("https://www.python.org")) as page:
    for line in page:
        print(line)
# 最终会调用对象的close()方法
```

### `urllib`
发送请求
```python
from urllib import request

with request.urlopen("https://api.douban.com/v2/book/2129650") as f:
    data = f.read()
    print("status:", f.status, f.reason)
    for k, v in f.getheaders():
        print("%s : %s" % (k, v))
    print("data:", data.decode('utf-8'))
    

# get
req = request.Request("https://www.douban.com")
req.add_header("User-Agent", "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36")
with request.urlopen(req) as f:
    print("status:", f.status, f.reason)
    for k,v in f.getheaders():
        print("%s : %s" % (k, v))
    print("data:", f.read().decode('utf-8'))
    
# post 把数据通过data传入
print('Login to weibo.cn...')
email = input('Email: ')
passwd = input('Password: ')
login_data = parse.urlencode([
    ('username', email),
    ('password', passwd),
    ('entry', 'mweibo'),
    ('client_id', ''),
    ('savestate', '1'),
    ('ec', ''),
    ('pagerefer', 'https://passport.weibo.cn/signin/welcome?entry=mweibo&r=http%3A%2F%2Fm.weibo.cn%2F')
])

req = request.Request('https://passport.weibo.cn/sso/login')
req.add_header('Origin', 'https://passport.weibo.cn')
req.add_header('User-Agent', 'Mozilla/6.0 (iPhone; CPU iPhone OS 8_0 like Mac OS X) AppleWebKit/536.26 (KHTML, like Gecko) Version/8.0 Mobile/10A5376e Safari/8536.25')
req.add_header('Referer', 'https://passport.weibo.cn/signin/login?entry=mweibo&res=wel&wm=3349&r=http%3A%2F%2Fm.weibo.cn%2F')

with request.urlopen(req, data=login_data.encode('utf-8')) as f:
    print('Status:', f.status, f.reason)
    for k, v in f.getheaders():
        print('%s: %s' % (k, v))
    print('Data:', f.read().decode('utf-8'))
```

### `HTMLParser`
```python
from html.parser import HTMLParser
from html.entities import name2codepoint

class myParser(HTMLParser):
    def handle_starttag(self, tag, attrs):
        print('<%s>' % tag)

    def handle_endtag(self, tag):
        print('<%s>' % tag)
        
    def handle_startendtag(self, tag):
        print('<%s>' % tag)
        
    def handle_data(self, data):
        print('<!--', data, '-->')
        
    def handle_entityref(self, name):
        print('&#%s;' % name)
        
    def handle_charref(self, name):
        print("&#%s;" % name)

parser = myParser()
parser.feed('''<html><head></head/><body>
<!-- test html -->
<p><a href=\"#\">html</a> HTML &nbsp;test<br> END </p>
</body></html>''')
```
`feed()`方法可以多次调用, 不必一次把整个html字符串都塞进去. 特殊字符有两种`&nbsp;`和`&#1234;`, 都可以解析出来

### `__builtin__`

### sys
```python
import sys
for i in sys.argv:  # 获取命令行参数
    print i
```
### os
```python
import os
os.name # 
os.uname #
os.environ
```

### IO
文件打开模式:
- `r`: 默认, 只读
- `w`: 先清空在写
- `x`: 不存在时创建, 已存在时抛出错误 
- `a`: 追加内容, 文件不存在时抛出错误
- `b`: 二进制模式
- `t`: 默认, 文本模式
- `+`: open a disk file for updating (reading and writing)
- `U`: 已遗弃
必须以'r', 'w', 'a' 或 'U'开头. `r+`读写都行
```python
with open('a.txt', 'r', encoding='utf-8', error='ignore') as f:
    f.read()    # 一次性读取全部文件内容
    f.read(1024)    # 
    for line in r.readlines():  # 按行读取
        line.strip()    # 去除\n
```
#### StringIO和ByteIO
内存IO
```python
from io import StringIO, BytesIO
f = StringIO("Hello\n会话")
print(f.read())

b = BytesIO()
b.write("Hello\n会话".encode('utf-8'))
print(b.getvalue())     # b'Hello\n\xe4\xbc\x9a\xe8\xaf\x9d'

# bytes <--> str
# 转字节
'Hello\n会话'.encode('utf-8')
# 转字符
'Hello\n会话'.encode('utf-8').decode('utf-8')
```
#### 文件和目录
```python
import os

os.sep  # 目录分隔符, 优先使用: os.path.split() 和 os.path.join()
os.pathsep  #多目录分隔符, "/a/b:/x/y", ':' unix/linux   ';' windows

os.getuid()     #返回当前进程用户id
os.getpid()     #返回当前进程id

os.mkdir("xxx")  # 创建目录, 默认777
os.makedirs("/Users/tonny/workspace/a/b/c") # 创建目录树
os.remove("xxx")     #删除文件或目录
os.removedirs("foo/bar/baz")    # 遍历删除目录, 如果目录为空, baz -> bar -> foo
os.rmdir("xx")  #删除目录, 如果目录为空, 否则抛出OSError
os.rename("test.txt", "t.txt")  # 文件重命名
os.renames("test.txt", "a/b/t.txt")  # 文件重命名, 创建a, b

os.chdir("/xx/yy")  # 修改当前工作目录到指定目录
os.getcwd()     #获取当前工作目录
os.chmod()
os.chown()
os.listdir()     # 返回list, 列出当前目录下文件及目录名称
os.scandir()  #效率更高, 返回迭代器, 元素类型os.DirEntry, 其有name, path, is_file(), is_dir()等属性和方法

# 遍历目录, os.walk返回的是生成器
for root, dirs, files in os.walk("./python_study"):
    # root: 当前被目录, dirs: root下的目录(不包括子目录), files: root下的文件(不包括子目录内的)
    # 获取文件全路径, 过滤.idea目录下的文件
    prefix = os.path.abspath(root)
    print(list(filter(lambda x: x.find(".idea") != 0, map(lambda x: os.path.join(prefix, x), files))))
```
os.path
```python
os.path.abspath(".")    # '/Users/tonny/workspace'
os.path.join("~", "work")   # 生成新路径, 不要直接拼接字符串
os.path.split('/Users/tonny/workspace/file.txt')    # ('/Users/tonny/workspace', 'file.txt'), 拆分路径
os.path.splitext('/Users/tonny/workspace/file.txt')    # ('/Users/tonny/workspace/file', 'txt') #得到文件扩展名
```
shuitl
```python
# 复制文件函数在os模块中不存在, 但是可以使用shutil模块
import shutil

shutil.copyfile("hello.log", "a.log")   #复制文件内容到新文件, 目的文件必须是文件名, 不能是目录
shutil.copymod("hello.log", "a.log")    #复制权限配置到新文件
shutil.copystat("hello.log", "a.log")   #复制权限配置, 最后访问时间, 最后更新时间等到新文件, 所属人和组不会复制
shutil.copy("hello.log", "a.log")   #复制文件到指定目录  
shutil.copy2("hello.log", "a.log")   #复制文件(包括元信息)到指定目录  
shutil.copytree("a.txt", "/a/b/c")  #复制目录树
shuitl.move("/a/b/c", "/a/d")   #移动目录及其下目录和文件
shutil.rmtree("/a/b/c") #删除目录树, 删除c以及其下的目录和文件
```
tempfile
```python
import tempfile

# 当程序需要一个临时文件存储数据, 不需要跟其他程序共享, 该程序关闭后, 该文件会自动删除
with tempfile.TemporaryFile() as temp:
    temp.write('Hello\n会话'.encode('utf-8'))
    temp.seek(0)
    print(str(temp.read(), 'utf-8'))
```

### Thread

### Net

### 时间/日期
```python
from datetime import datetime

dt = datetime(2018, 12, 10)  # 前3个必填
datetime(2018, 12, 10, 0, 0, 0, 123)  #构造时间日期
datetime.today()    #datetime.datetime(2018, 4, 24, 17, 2, 44, 923581)
datetime.now()  #datetime.datetime(2018, 4, 24, 17, 4, 31, 385321)
datetime.fromtimestamp(1524560935)  # 从时间戳构造

dt.date()   #日期部分, datetime.date(2018, 4, 24)
dt.time()   #时间部分, datetime.time(17, 8, 55)
dt.timestamp()  #时间戳, 1524561259.234084
dt.replace(year=2016)   #返回新的对象, 设置某个字段的值, datetime.datetime(2016, 4, 24, 17, 8, 55)

dt.isoformat()  # ISO8601标准的日期格式, '2018-04-24T17:14:19.234084'
dt.__str__()    # 等同于dt.isoformat()
# datetime <--> string
dt.strftime("%Y-%m-%d %H:%M:%S.%f")     #格式化, '2018-04-24 17:14:19.234084'
datetime.strptime('2018-04-24 17:14:19.234084', "%Y-%m-%d %H:%M:%S.%f") #从字符串转成datetime 
dt.year/month/day/hour/minute/second/microsecond/tzinfo

# 时间加减
from datetime import timedelta
timedelta(minutes=60).total_seconds()   # 时间间隔总秒数

dt + timedelta(hours=1)     # 加1小时
dt + timedelta(days=2, hours=12)    # 加1天12小时
dt - timedelta(days=1)      # 减1天

from datetime import date, time
d = date(2018, 10, 12)
t = time()
```

### Pillow
PIL是python事实上的图像处理库, 但只支持2.7. 3需要使用pilllow
```bash
pip install pillow
```
```python
from PIL import Image, ImageFilter, ImageFont, ImageDraw
im = Image.open("test.jpg")
w, h = im.size  #获得尺寸
im.thumbnail((w//2, h//2))  # 长宽缩小到50%
im.save('thumbnail.jpg', 'jpeg')    # 保存

im.filter(ImageFilter.BLUR)     # 图像模糊
im.save('blur.jpg', 'jpeg')

# 绘图
import random
import os

os.chdir('/Users/tonny/Downloads')

def rndChar():
    return chr(random.randint(65, 90))

def rndColor():
    return (random.randint(64, 255), random.randint(64, 255), random.randint(64, 255))

def rndColor2():
    return (random.randint(32, 147), random.randint(32, 147), random.randint(32, 147))
    
w = 60 * 4
h = 60
im = Image.new('RGB', (w, h), (255, 255, 255))
font = ImageFont.truetype('Arial.ttf', 36)
draw = ImageDraw.Draw(im)
for x in range(w):
    for y in range(h):
        draw.point((x, y), fill=rndColor())

for t in range(4):
    draw.text((60 * t + 10, 10), rndChar(), font=font, filel=rndColor2())
    
im = im.filter(ImageFilter.BLUR)
im.save("code.jpg", 'jpeg')
```
### requests
```python
import requests

# https://api.github.com/search/repositories?q=spring-framework+language:java&sort=stars&order=desc&page=1&per_page=2

cookies = {'token': 'xxxx'}
headers={'User-Agent': 'Mozilla/5.0 (iPhone; CPU iPhone OS 11_0 like Mac OS X) AppleWebKit'}
params = {
    'q':'spring-framework+language:java',
    'sort':'stars',
    'page':1,
    'per_page':5    
}
r = requests.get("https://api.github.com/search/repositories", params, headers=headers, cookies=cookies, timeout=2.5)   # 超时单位秒
r.status_code
r.encoding
r.content   # 二进制字节
r.cookie['ss']  #获取cookie
r.json()    # 可以对json格式的响应直接获取json对象

# request默认使用application/x-www-form-urlencoded对POST数据编码
r = requests.post('https://accounts.douban.com/login', data={'form_email': 'abc@example.com', 'form_password': '123456'})

# 可以传递JSON数据
params = {'key':'xx'}
r = request.post(url, json=params)

# 文件上传
upload_files = {'file': open('xx.txt', 'rb')}
r = request.post(url, files=upload_files)
```

### chardet
检测编码, 需要先安装
```python
import chardet
asc = b'Hello'      #{'confidence': 1.0, 'encoding': 'ascii', 'language': ''}
gbk = '会话就看看'.encode('gbk')     #{'confidence': 0.99, 'encoding': 'GB2312', 'language': 'Chinese'}
u8 = '会话'.encode('utf-8')   # {'confidence': 0.7525, 'encoding': 'utf-8', 'language': ''}
```

### psutil
获取系统信息, 需要安装
```python
import psutil

psutil.cpu_count()   # 8, cpu逻辑数量
psutil.cpu_count(logical=False)     #4, 物理cpu个数
psutil.cpu_times()  # cpu用户/系统/空闲时间 scputimes(user=10963.31, nice=0.0, system=5138.67, idle=356102.45)
# 实现类似top命令的cpu使用率, 每秒刷新一次
for i in range(10):
    psutil.cpu_percent(interval=1, percpu=True)

psutil.virtual_memory() # 物理内存, svmem(total=17179869184, available=5213442048, percent=69.7, used=15868260352, free=78065664, active=6409957376, inactive=5135376384, wired=4322926592)
psutil.swap_memory()    #交换内存, sswap(total=10737418240, used=9672851456, free=1064566784, percent=90.1, sin=894136324096, sout=12848443392)

psutil.disk_partitions()    #磁盘分区
psutil.disk_usage('/')  # 磁盘使用率
psutil.disk_io_counters()   #磁盘io

psutil.net_io_counters()    # 网络io
psutil.net_if_addrs()   # 网络接口信息
psutil.net_if_stats()   # 网络接口状态
psutil.net_connections()    #网络连接信息, 需要root权限

psutil.pids()   #所有进程id
p = psutil.Process(45801)
p.name()  # 进程名称
p.status()  #进程状态
p.username()    #进程用户名
p.create_time() #进程创建时间
p.terminal()    #进程终端
p.terminate()   #结束进程
p.exe() # 进程exe路径
p.cwd() #进程工作目录
p.cmdline() #进程启动的目录行
p.cpu_times()    #进程使用的cpu时间
p.memory_info() #进程使用的内容
p.open_files()  #进程打开的文件
p.connections() #进程相关的网络连接
p.num_threads()  #进程的线程数量
p.environ() #进程环境变量
p.ppid()    #父进程id
p.parent()  #父进程
p.children()    #子进程
```

### 字符串
#### 正则

### 其他
#### 日志
```python
import logging


# create file handler
handler = logging.FileHandler("hello.log")
handler.setLevel(logging.INFO)

# create a logging format
formatter = logging.Formatter("%(asctime)s - %(name)s - %(levelname)s - %(message)s")
handler.setFormatter(formatter)

logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)
logger.addHandler(handler)


logger.info("Hello Logger")
```

## Tips
#### 交换两个变量值
```python
x, y = 1, 3
print(x, y)
x, y = y, x
print(x, y)
```
#### 删除方法: remove del pop区别
`remove`只删除第一个匹配的元素; `del`删除指定下标的元素; `pop`删除指定下标的元素并返回
```python
ad = [1, 2, 4, 2]
ad.remove(2)
print(ad)   #[1, 4, 2]

ad = [1, 2, 4, 2]
del ad[2]
print(ad)   #[1, 2, 2]

ad = [1, 2, 4, 2]
ad.pop(2)
print(ad)  #[1, 2, 2]
```
#### 遍历序列
```python
list = ['a', 'b', 'c']
# 不好的做法
for i in range(0,len(list)):
    print(i, list[i])

# 好的做法
for i, item in enumerate(list):
    print(i, item)
```
