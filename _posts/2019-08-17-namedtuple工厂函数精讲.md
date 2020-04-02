---
layout: post
title: "namedtuple工厂函数精讲"
author: zhouzhaoxin
categories: ["后端"]
image: assets/images/animal-4972730_1920.jpg
featured: true
comments: false
---
首先，我会介绍下使用`namedtuple`所需要了解的基本概念，然后讲解如何使用`namedtuple`，最后使用`namedtuple`来创建一摞纸牌。理解这些之后，就可以权衡利弊，并在生产中使用

## 基本概念
1. `namedtuple`是一个 **工厂函数**，定义在`python`标准库的`collections`模块中，使用此函数可以创建一个可读性更强的元组
2. `namedtuple`函数所创建（返回）的是一个 **元组的子类**（python中基本数据类型都是类，且可以在`buildins`模块中找到）
3. `namedtuple`函数所创建元组，中文名称为 **具名元组**
4. 在使用普通元组的时候，我们只能通过`index`来访问元组中的某个数据
5. 使用具名元组，我们既可以使用`index`来访问，也可以使用具名元组中每个字段的名称来访问
6. 值得注意的是，具名元组和普通元组所需要的内存空间相同，所以 **不必使用性能来权衡是否使用具名元组**

## 如何使用
`namedtuple`是一个函数，我们先来看下他的参数

#### 参数解析
```python
def namedtuple(typename, field_names, *, rename=False, defaults=None, module=None):
```
有两个必填参数`typename`和`field_names`
**`typename`**
- 参数类型为字符串
- 具名元组返回一个元组子对象，我们要为这个对象命名，传入`typename`参数即可

**`field_names`**
- 参数类型为字符串序列
- 用于为创建的元组的每个元素命名，可以传入像`['a', 'b']`这样的序列，也可以传入`'a b'`或`'a, b'`这种被**逗号**或**空格**分割的单字符串
- 必须是合法的标识符。不能是关键字如`class,def`等

**`rename`**
- 注意的参数中使用了`*`，其后的所有参数必须指定关键字
- 参数为布尔值
- 默认为`False`。当我们指定为`True`时，如果定义**`field_names`**参数时，出现非法参数时，会将其替换为位置名称。如`['abc', 'def', 'ghi', 'abc']`会被替换为`['abc', '_1', 'ghi', '_3']`

**`defaults`**
- 参数为`None`或者可迭代对象
- 当此参数为`None`时，创建具名元组的实例时，必须要根据`field_names`传递指定数量的参数
- 当设置`defaults`时，我们就为具名元组的元素赋予了默认值，被赋予默认值的元素在实例化的时候可以不传入
- 当`defaults`传入的序列长度和`field_names`不一致时，函数默认会右侧优先
- 如果`field_names`是`['x', 'y', 'z'] `，`defaults`是`(1, 2)`，那么`x`是实例化必填参数，`y`默认为`1`，`z`默认为`2`

#### 基本使用
理解了`namedtuple`函数的参数，我们就可以创建具名元组了
```python
>>> Point = namedtuple('Point', ['x', 'y'])  # 返回一个名为`Point`的类，并赋值给名为`Point`的变量
>>> p = Point(11, y=22)     # 可以根据参数的位置，或具名参数来实例化（像普通的类一样）
>>> p[0] + p[1]             # 具名元组可以像普通元组一样通过`index`访问
33
>>> x, y = p                # 具名元组可以像普通元组一样解包
>>> x, y
(11, 22)
>>> p.x + p.y               # 具名元组还可以通过属性名称访问元组内容
33
>>> p                       # 具名元组在调用`__repr__`,打印实例时，更具可读性
Point(x=11, y=22)
```
具名元组在存储[`csv`](https://docs.python.org/3.8/library/csv.html#module-csv "csv: Write and read tabular data to and from delimited files.")或者[`sqlite3`](https://docs.python.org/3.8/library/sqlite3.html#module-sqlite3 "sqlite3: A DB-API 2.0 implementation using SQLite 3.x.")返回数据的时候特别有用
```
EmployeeRecord = namedtuple('EmployeeRecord', 'name, age, title, department, paygrade')

import csv
for emp in map(EmployeeRecord._make, csv.reader(open("employees.csv", "rb"))):
    print(emp.name, emp.title)

import sqlite3
conn = sqlite3.connect('/companydata')
cursor = conn.cursor()
cursor.execute('SELECT name, age, title, department, paygrade FROM employees')
for emp in map(EmployeeRecord._make, cursor.fetchall()):
    print(emp.name, emp.title)
```

#### 特性
具名元组除了拥有继承自基本元组的所有方法之外，还提供了额外的三个方法和两个属性，为了防止命名冲突，这些方法都会以下划线开头

**`_make(iterable)`**
这是一个类函数，参数是一个迭代器，可以使用这个函数来构建具名元组实例
```
>>> t = [11, 22]
>>> Point._make(t)
Point(x=11, y=22)
```

**`_asdict()`**
实例方法，根据具名元组的名称和其元素值，构建一个[`OrderedDict`](https://docs.python.org/3.8/library/collections.html#collections.OrderedDict "collections.OrderedDict")返回
```python
>>> p = Point(x=11, y=22)
>>> p._asdict()
OrderedDict([('x', 11), ('y', 22)])
```

**`_replace(**kwargs)`**
实例方法，根据传入的关键词参数，替换具名元组的相关参数，然后返回一个新的具名元组
```
>>> p = Point(x=11, y=22)
>>> p._replace(x=33)
Point(x=33, y=22)

>>> for partnum, record in inventory.items():
...     inventory[partnum] = record._replace(price=newprices[partnum], timestamp=time.now())
```
**`_fields`**
这是一个实例属性，存储了此具名元组的元素名称元组，在根据已经存在的具名元组创建新的具名元组的时候使用
```
>>> p._fields            # view the field names
('x', 'y')

>>> Color = namedtuple('Color', 'red green blue')
>>> Pixel = namedtuple('Pixel', Point._fields + Color._fields)
>>> Pixel(11, 22, 128, 255, 0)
Pixel(x=11, y=22, red=128, green=255, blue=0)
```

**`_fields_defaults`**
查看具名元组类的默认值
```
>>> Account = namedtuple('Account', ['type', 'balance'], defaults=[0])
>>> Account._fields_defaults
{'balance': 0}
>>> Account('premium')
Account(type='premium', balance=0)
```

#### 使用技巧
1. 使用`getattr`获取具名元组元素值
```
>>> getattr(p, 'x')
11
```

2. 将字典转换为具名元组
```
>>> d = {'x': 11, 'y': 22}
>>> Point(**d)
Point(x=11, y=22)
```

3. 既然具名元组是一个类，我们当然可以随心所欲的进行定制

```
>>> class Point(namedtuple('Point', ['x', 'y'])):
...     __slots__ = ()
...     @property
...     def hypot(self):
...         return (self.x ** 2 + self.y ** 2) ** 0.5
...     def __str__(self):
...         return 'Point: x=%6.3f  y=%6.3f  hypot=%6.3f' % (self.x, self.y, self.hypot)

>>> for p in Point(3, 4), Point(14, 5/7):
...     print(p)
Point: x= 3.000  y= 4.000  hypot= 5.000
Point: x=14.000  y= 0.714  hypot=14.018
```

`__slots__`值的设置可以保证具名元组保持最小的内存占用

## namedtuple纸牌
```
import collections
# 将纸牌定义为具名元组，每个纸牌都有等级和花色
Card = collections.namedtuple('Card', 'rank suit')

class FrenchDeck:
    # 等级2-A
    ranks = [str(n) for n in range(2,11)] + list('JQKA')
    # 花色红黑方草
    suits = 'spades diamonds clubs hearts'.split()
    # 构建纸牌
    def __init__(self):
        self._cards = [Card(rank, suit) for suit in self.suits for rank in self.ranks]
    # 获取纸牌
    def __getitem__(self, position):
        return self._cards[position]

>>> french_deck = FrenchDeck()
>>> french_deck[0]
Card(rank='2', suit='spades')
>>> french_deck[0].rank
'2'
>>> french_deck[0].suit
'spades'
```
