---
title: "Python语言规范"
date: 2019-06-14T20:26:46+08:00
tags: ["Python", "Pylint"]
categories: [""]
series: [""]
summary: "pylint是一个可以查找py文件中部分错误以及不规范的语法，虽然pylint还不够完美但是我们可以借助它修正不规范的地方。"
draft: false
editPost:
  URL: "https://github.com/Zhoutao822/zhoutao822.github.io/tree/main/content/"
  Text: "Suggest Changes"
  appendFilePath: true 
---

## 1. Pylint

pylint是一个可以查找py文件中部分错误以及不规范的语法，虽然pylint还不够完美但是我们可以借助它修正不规范的地方。

使用方式

```shell
pylint xxx.py
```

输出类似于

```
************* Module wine
wine.py:20:0: C0304: Final newline missing (missing-final-newline)
wine.py:1:0: C0111: Missing module docstring (missing-docstring)
wine.py:3:0: E0401: Unable to import 'numpy' (import-error)
wine.py:4:0: E0401: Unable to import 'sklearn.datasets' (import-error)
wine.py:5:0: E0401: Unable to import 'sklearn.mixture' (import-error)
wine.py:6:0: C0103: Constant name "rawData" doesn't conform to UPPER_CASE naming style (invalid-name)
wine.py:8:0: C0103: Constant name "data" doesn't conform to UPPER_CASE naming style (invalid-name)
wine.py:9:0: C0103: Constant name "target" doesn't conform to UPPER_CASE naming style (invalid-name)
wine.py:11:0: C0103: Constant name "gmm" doesn't conform to UPPER_CASE naming style (invalid-name)
wine.py:15:0: C0103: Constant name "prediction" doesn't conform to UPPER_CASE naming style (invalid-name)
wine.py:19:0: C0103: Constant name "acc" doesn't conform to UPPER_CASE naming style (invalid-name)

---------------------------------------------------------------------
Your code has been rated at -9.17/10 (previous run: 10.00/10, -19.17)
```

也可以通过行注释来抑制警告

```shell
dict = 'something awful'  # Bad Idea... pylint: disable=redefined-builtin
```

要抑制”参数未使用”告警, 你可以用”_”作为参数标识符, 或者在参数名前加”unused_”. 遇到不能改变参数名的情况, 你可以通过在函数开头”提到”它们来消除告警. 例如:

```python
def foo(a, unused_b, unused_c, d=None, e=None):
    _ = d, e
    return a
```

## 2. 导入

* 使用 `import x` 来导入包和模块.

* 使用 `from x import y`, 其中x是包前缀, y是不带前缀的模块名.

* 使用 `from x import y as z`, 如果两个要导入的模块都叫做y或者y太长了.

例如, 模块 `sound.effects.echo` 可以用如下方式导入:

```python
from sound.effects import echo
...
echo.EchoFilter(input, output, delay=0.7, atten=4)
```

导入时不要使用相对名称. 即使模块在同一个包中, 也要使用完整包名. 这能帮助你避免无意间导入一个包两次.

## 3. 包

所有的新代码都应该用完整包名来导入每个模块.

应该像下面这样导入:

```python
# Reference in code with complete name.
import sound.effects.echo

# Reference in code with just module name (preferred).
from sound.effects import echo
```

## 4. 异常

异常必须遵守特定条件:

1. 像这样触发异常: `raise MyException("Error message")` 或者 `raise MyException` . 不要使用两个参数的形式( `raise MyException, "Error message"` )或者过时的字符串异常( `raise "Error message"` ).
2. 模块或包应该定义自己的特定域的异常基类, 这个基类应该从内建的Exception类继承. 模块的异常基类应该叫做”Error”.

```python
class Error(Exception):
    pass
```

3. 永远不要使用 `except:` 语句来捕获所有异常, 也不要捕获 `Exception` 或者 `StandardError` , 除非你打算重新触发该异常, 或者你已经在当前线程的最外层(记得还是要打印一条错误消息). 在异常这方面, Python非常宽容, `except:` 真的会捕获包括Python语法错误在内的任何错误. 使用 `except:` 很容易隐藏真正的bug.
4. 尽量减少try/except块中的代码量. try块的体积越大, 期望之外的异常就越容易被触发. 这种情况下, try/except块将隐藏真正的错误.
5. 使用`finally`子句来执行那些无论try块中有没有异常都应该被执行的代码. 这对于清理资源常常很有用, 例如关闭文件.
6. 当捕获异常时, 使用 `as` 而不要用逗号. 例如

```python
try:
    raise Error
except Error as error:
    pass
```

## 5. 全局变量

避免使用全局变量, 用类变量来代替. 但也有一些例外:

1. 脚本的默认选项.
2. 模块级常量. 例如:　PI = 3.14159. 常量应该全大写, 用下划线连接.
3. 有时候用全局变量来缓存值或者作为函数返回值很有用.
4. 如果需要, 全局变量应该仅在模块内部可用, 并通过模块级的公共函数来访问.

## 6. 嵌套/局部/内部类或函数

鼓励使用嵌套/本地/内部类或函数

类可以定义在方法, 函数或者类中. 函数可以定义在方法或函数中. 封闭区间中定义的变量对嵌套函数是只读的.

嵌套类或局部类的实例不能序列化(pickled).

## 6. 列表推导

适用于简单情况. 每个部分应该单独置于一行: 映射表达式, for语句, 过滤器表达式. 禁止多重for语句或过滤器表达式. 复杂情况下还是使用循环.

```python
Yes:
  result = []
  for x in range(10):
      for y in range(5):
          if x * y > 10:
              result.append((x, y))

  for x in xrange(5):
      for y in xrange(5):
          if x != y:
              for z in xrange(5):
                  if y != z:
                      yield (x, y, z)

  return ((x, complicated_transform(x))
          for x in long_generator_function(parameter)
          if x is not None)

  squares = [x * x for x in range(10)]

  eat(jelly_bean for jelly_bean in jelly_beans
      if jelly_bean.color == 'black')
```

```python
No:
  result = [(x, y) for x in range(10) for y in range(5) if x * y > 10]

  return ((x, y, z)
          for x in xrange(5)
          for y in xrange(5)
          if x != y
          for z in xrange(5)
          if y != z)
```

## 7. 默认迭代器和操作符

如果类型支持, 就使用默认迭代器和操作符, 例如列表, 字典和文件. 内建类型也定义了迭代器方法. 优先考虑这些方法, 而不是那些返回列表的方法. 当然，这样遍历容器时，你将不能修改容器.

```python
Yes:  for key in adict: ...
      if key not in adict: ...
      if obj in alist: ...
      for line in afile: ...
      for k, v in dict.iteritems(): ...
```

```python
No:   for key in adict.keys(): ...
      if not adict.has_key(key): ...
      for line in afile.readlines(): ...
```

## 8. 生成器

所谓生成器函数, 就是每当它执行一次生成(yield)语句, 它就返回一个迭代器, 这个迭代器生成一个值. 生成值后, 生成器函数的运行状态将被挂起, 直到下一次生成.

鼓励使用. 注意在生成器函数的文档字符串中使用”Yields:”而不是”Returns:”.

## 9. Lambda函数

适用于单行函数. 如果代码超过60-80个字符, 最好还是定义成常规(嵌套)函数.

对于常见的操作符，例如乘法操作符，使用 `operator` 模块中的函数以代替lambda函数. 例如, 推荐使用 `operator.mul` , 而不是 `lambda x, y: x * y` .

## 10. 条件表达式

适用于单行函数. 在其他情况下，推荐使用完整的if语句.

## 11. 默认参数值

鼓励使用, 不过有如下注意事项:

不要在函数或方法定义中使用可变对象作为默认值.

```python
Yes: def foo(a, b=None):
         if b is None:
             b = []
No:  def foo(a, b=[]):
         ...
No:  def foo(a, b=time.time()):  # The time the module was loaded???
         ...
No:  def foo(a, b=FLAGS.my_thing):  # sys.argv has not yet been parsed...
         ...
```

## 12. 属性

你通常习惯于使用访问或设置方法来访问或设置数据, 它们简单而轻量. 不过我们建议你在新的代码中使用属性. 只读属性应该用 `@property` 装饰器 来创建.

如果子类没有覆盖属性, 那么属性的继承可能看上去不明显. 因此使用者必须确保访问方法间接被调用, 以保证子类中的重载方法被属性调用(使用模板方法设计模式).

```python
Yes: import math

     class Square(object):
         """A square with two properties: a writable area and a read-only perimeter.

         To use:
         >>> sq = Square(3)
         >>> sq.area
         9
         >>> sq.perimeter
         12
         >>> sq.area = 16
         >>> sq.side
         4
         >>> sq.perimeter
         16
         """

         def __init__(self, side):
             self.side = side

         def __get_area(self):
             """Calculates the 'area' property."""
             return self.side ** 2

         def ___get_area(self):
             """Indirect accessor for 'area' property."""
             return self.__get_area()

         def __set_area(self, area):
             """Sets the 'area' property."""
             self.side = math.sqrt(area)

         def ___set_area(self, area):
             """Indirect setter for 'area' property."""
             self._SetArea(area)

         area = property(___get_area, ___set_area,
                         doc="""Gets or sets the area of the square.""")

         @property
         def perimeter(self):
             return self.side * 4
```

## 13. True/False的求值

尽可能使用隐式的false, 例如: 使用 `if foo:` 而不是 `if foo != []:` . 不过还是有一些注意事项需要你铭记在心:

1. 永远不要用==或者!=来比较单件, 比如None. 使用is或者is not.

2. 注意: 当你写下 `if x:` 时, 你其实表示的是 `if x is not None` . 例如: 当你要测试一个默认值是None的变量或参数是否被设为其它值. 这个值在布尔语义下可能是false!

3. 永远不要用==将一个布尔量与false相比较. 使用 `if not x:` 代替. 如果你需要区分false和None, 你应该用像 `if not x and x is not None:` 这样的语句.

4. 对于序列(字符串, 列表, 元组), 要注意空序列是false. 因此 `if not seq:` 或者 `if seq:` 比 `if len(seq):` 或 `if not len(seq):` 要更好.

5. 处理整数时, 使用隐式false可能会得不偿失(即不小心将None当做0来处理). 你可以将一个已知是整型(且不是len()的返回结果)的值与0比较.

```python
Yes: if not users:
         print 'no users'

     if foo == 0:
         self.handle_zero()

     if i % 10 == 0:
         self.handle_multiple_of_ten()
No:  if len(users) == 0:
         print 'no users'

     if foo is not None and not foo:
         self.handle_zero()

     if not i % 10:
         self.handle_multiple_of_ten()
```

6. 注意‘0’(字符串)会被当做true.

## 14. 词法作用域

嵌套的Python函数可以引用外层函数中定义的变量, 但是不能够对它们赋值. 变量绑定的解析是使用词法作用域, 也就是基于静态的程序文本. 对一个块中的某个名称的任何赋值都会导致Python将对该名称的全部引用当做局部变量, 甚至是赋值前的处理. 如果碰到global声明, 该名称就会被视作全局变量.

一个使用这个特性的例子:

```python
def get_adder(summand1):
    """Returns a function that adds numbers to a given number."""
    def adder(summand2):
        return summand1 + summand2

    return adder
```

(译者注: 这个例子有点诡异, 你应该这样使用这个函数: `sum = get_adder(summand1)(summand2)` )

## 15. 函数与方法装饰器

用于函数及方法的装饰器 (也就是@标记). 最常见的装饰器是@classmethod 和@staticmethod, 用于将常规函数转换成类方法或静态方法. 不过, 装饰器语法也允许用户自定义装饰器. 特别地, 对于某个函数 `my_decorator` , 下面的两段代码是等效的:

```python
class C(object):
   @my_decorator
   def method(self):
       # method body ...
```

```python
class C(object):
    def method(self):
        # method body ...
    method = my_decorator(method)
```

## 16. 线程

虽然Python的内建类型例如字典看上去拥有原子操作, 但是在某些情形下它们仍然不是原子的(即: 如果__hash__或__eq__被实现为Python方法)且它们的原子性是靠不住的. 你也不能指望原子变量赋值(因为这个反过来依赖字典).

优先使用Queue模块的 `Queue` 数据类型作为线程间的数据通信方式. 另外, 使用threading模块及其锁原语(locking primitives). 了解条件变量的合适使用方式, 这样你就可以使用 `threading.Condition` 来取代低级别的锁了.

## 参考：

1. [Google Python语言规范](https://zh-google-styleguide.readthedocs.io/en/latest/google-python-styleguide/python_language_rules/)

