title: with语句上下文管理器详解
date: 2015-09-04 11:42:27
tags: [Python, with, contextlib, 上下文管理器]
categories: Python
---

写Python代码一直强调pythonic，使用Python特色的内容。
With语句上下文管理器便是python中比较特色的内容之一。

# With语句上下文管理器

With语句上下文管理器经过各方的争辩和讨论，在[PEP343](https://www.python.org/dev/peps/pep-0343/)最终被确立。

目前，Python很多地方使用了它：
 
+ file
+ thread.LockType
+ threading.Lock
+ ...

其语法如下：

```Python
with EXPR as VAR:
     BLOCK
```

本文将对Python这部分内容，结合自己的所知作一个全面的说明。

# 从打开文件写入说起

Python很多函数内置了`with`语句的使用，`open`函数便是其中一个比较常见的。这里由open函数使用例子，一步步说明`with`的独特优势。

假设，我们需要写这样的内容，打开一个‘test.txt’文件，写入‘test’，并关闭文件。

## 版本一：陈述写法
按照要求可以看出，基本需要三个功能，打开文件，写入文件以及关闭文件，一个普通的陈述性质Python写法是这个样子的：

```Python
writer = open('test.txt', 'w')
writer.write('test')
wtiter.close()
```
三行语句分别完成一个功能。

但是这是后应该会想到，如果打开文件写入失败了怎么办，应该怎么处理。


## 版本二：加上异常的写法

对于这些异常的处理，自然会想到`try...except...finally`的语句来进行异常的处理，一个添加了异常处理的写法会是这个样子：

```Python
writer = open('test.txt', 'w')
try:
    writer.write('test')
except:
    pass
finally:
    writer.close()
```
无论是否发生异常都会关闭，保证了代码的安全。

但是现实里很多不仅仅是一个`try...except...finally`能够解决的，大量的使用会使代码变的很难看，那么如何能更优雅地处理这些呢？

## 版本三：更优雅的写法

这里就使用到了我们提到的`with`语句了，把出错控制等等交给上下文管理器来解决，使用`with`语句的写法，就会变成这样：

```Python
with open('test.txt', 'w') as writer:
    writer.write('test')
```
这样结束关闭和出错控制都不需要你来做了，代码变的更加的简洁。

但是`with`语句究竟做了什么呢？


# With语句处理的背后

## 内置的方法

Python类通过两个特殊的内置方法实现了`with`语句的功能：`__enter__`、`__exit__`(参见：[上下文管理器](https://docs.python.org/2/reference/datamodel.html#context-managers))，其中：

+ `__enter__`:定义了代码执行前的处理；
+ `__exit__`:定义了代码结束部分的处理。

回归到我们之前的打开文件写入，我们就要实现这样的类：

```Python

class Writer(object):
   def __init__(self, filename, mode):
       self.filename = filename
       self.mode = mode
       
   def __enter__(self):
       self.file = open(self.filename, self.mode)
       return self.file
   
   def __exit__(self, exc_type, exc_value, exc_traceback):
       if exc_type:
          pass # 这里代表异常部分处理
       self.file.close()
       return False
```
注：`__exit__`方法中的参数`exc_type`, `exc_value`, `exc_traceback`分别为异常类型，异常值以及异常追踪信息，如果在处理的时候遇到异常，`sys.exc_info()`将会传入这些参数。此方法返回值为True或者False，表明异常是否得到处理。如果返回False，引发的异常会被传递出上下文。

这样构成的打开文件的上下文管理器，就能这样使用：

```Python
with Writer('test.txt', 'w') as writer:
    writer.write('test')
```


功能是实现了没有错，但是这里就有问题了，这个是类的方法，但是`open`是函数不是类，怎么能用`with`呢？

## open函数上下文管理器实现

事实上`open`上下文管理器也是利用了类里面的这两种方法，具体实现会有很多，这里提供实现这里提供两种思路：

### 装饰器

这个比较pythonic，官方也是这种方式：

```Python
class Context(object):

   def __init__(self, func):
       self.func = func
       
   def __enter__(self):
       return self.func
   
   def __exit__(self, exc_type, exc_value, exc_traceback):
       if exc_type:
          pass # 这里代表异常部分处理
       self.func.close()
       return False
       
def context(func):
    def wrap(*args, **kwargs):
        return Context(func(*args, **kwargs))
    return wrap
```

这样通过装饰器把函数放入到类中，实现open函数：

```Python
@context
def new_open(filename, mode):
    return open(filename, mode)
    
with new_open('test.txt', 'w') as writer:
    writer.write('test')
```

### 实例化调用


```Python
class Open(object):

   def __call__(self, filename, mode):
       self.file = open(filename, mode)
       
   def __enter__(self):
       return self.file
   
   def __exit__(self, exc_type, exc_value, exc_traceback):
       if exc_type:
          pass # 这里代表异常部分处理
       self.file.close() 
       return False
       
new_open = Open()
```
这样就可以直接来使用`with`语句了:

```Python
with new_open('test.txt', 'w') as writer:
    writer.write('test')
```

# Contextlib介绍

[contextlib](https://hg.python.org/cpython/file/2.7/Lib/contextlib.py)是Python中内置的库，提供了一些方法，能更快的实现上下文管理器，如同上一部分装饰器方法类似用`Generator`实现了一个`GeneratorContextManager`的类，并提供了`contextmanager`的装饰器来为函数创建上下文管理器。

下面介绍这个库里的几个方法：

## contextmanager

contestmanager的使用方法如下：

```Python
@contextmanager
def some_generator(<arguments>):
    <setup>
    try:
        yield <value>
    finally:
        <cleanup>
```
之后就能这样使用：

```Python
with some_generator(<arguments>) as <variable>:
    <body>
```

这样使用`contextmanager`的`new_open`就变成：

```Python
@contextlib.contextmanager
def new_open(filename, mode):
    file = open(filename, mode)
    try:
        yield file
    finally:
        file.close()
```

## closing
`closing`是简单的处理关闭上下文管理器，对于有关闭的方法可以管理处理完关闭。

其使用方法如下：

```Python
with closing(<module>.open(<arguments>)) as f:
    <block>
```

譬如在`socket`中，为`socket`创建了生成socket的`create_socket`函数和发送数据结束后返回`socket`对象的`send_data函数`，那么可以这样使用上下文：

```Python
with contextlib.closing(create_socket()) as socket:
    socket.send_data(data)
```

## nested

`nested`是为了处理多个with嵌套的，使用方法如下：

```Python
with nested(*managers):
    do_something()
```

例如：

```Python
with contextlib.nested(open('read.txt', 'r'), open('write.txt', 'w')) as (reader, writer):
    writer.write(reader.read())
```

由于Python 2.7之后，添加了新的语法可以直接嵌套。这种函数就被废弃了：

```Python
with (open('read.txt', 'r') as reader,
      open('write.txt', 'w')) as writer:
    writer.write(reader.read())
```

以上便是with语句上下文管理器的全部内容，总之，善于利用上下文管理器能够使你的代码更有质量。

***

版本修改记录：

+ 2015-09-05 完成本次文章全部内容。 By Octavian