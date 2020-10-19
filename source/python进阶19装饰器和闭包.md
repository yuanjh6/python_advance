# python进阶19装饰器和闭包
## Nested functions
Python允许创建嵌套函数，这意味着我们可以在函数内声明函数并且所有的作用域和声明周期规则也同样适用。

```
>>> def outer():
...     x = 1
...     def inner():
...         print x # 1
...     inner() # 2
...
>>> outer()
```
这看起来稍显复杂，但其行为仍相当直接，易于理解。考虑一下在#1处发生了什么——Python寻找一个名为x的local变量，失败了，然后在最邻近的外层作用域里搜寻，这个作用域是另一个函数！变量x是函数outer的local变量，但是和前文提到的一样，inner函数拥有对外层作用域的访问权限（最起码有读和修改的权限）。在#2处我们调用了inner函数。请记住inner也只是一个变量名，它也遵从Python的变量查找规则——Python首先在outer的作用域里查找之，找到了一个名为inner的local变量。


## Closures
让我们不从定义而是从另一个代码示例开始。如果我们将上一个例子稍加修改会怎样呢？

```
>>> def outer():
...     x = 1
...     def inner():
...         print x # 1
...     return inner
>>> foo = outer()
>>> foo.func_closure # doctest: +ELLIPSIS
(\<cell at 0x.... int object at 0x...>,)
```

从上一个例子中我们看到inner是一个由outer返回的函数，存储于一个名为foo的变量，我们可以通过foo()调用它。但是它能运行吗？让我们先来考虑一下作用域规则。

一切都依照Python的作用域规则而运行——x是outer函数了一个local变量。当inner在#1处打印x时，Python在inner中寻找一个local变量，没有找到；然后它在外层作用域即outer函数中寻找并找到了它。

但是自此处从变量生命周期的角度来看又会如何呢？变量x是函数outer的local变量，这意味着只有当outer函数运行时它才存在。只有当outer返回后我们才能调用inner，因此依照我们关于Python如何运作的模型来看，在我们调用inner的时候x已经不复存在了，  那么某个运行时错误可能会出现。
事实与我们的预想并不一致，返回的inner函数的确正常运行。Python支持一种称为闭包(function closures)的特性，这意味着定义于非全局作用域的inner函数在定义时记得记得它们的外层作用域长什么样儿。这可以通过查看inner函数的func_closure属性来查看，它包含了外层作用域里的变量。

请记住，每次当outer函数被调用时inner函数都被重新定义一次。目前x的值没有改变，因此我们得到的每个inner函数和其它的inner函数拥有相同的行为，但是如果我们将它做出一点改变呢？

```
>>> def outer(x):
...     def inner():
...         print x # 1
...     return inner
>>> print1 = outer(1)
>>> print2 = outer(2)
>>> print1()
1
>>> print2()
2
```
从这个例子中你可以看到closures——函数记住他们的外层作用域的事实——可以用来构建本质上有一个硬编码参数的自定义函数。我们没有将数字1或者2传递给我们的inner函数但是构建了能"记住"其应该打印数字的自定义版本。

closures独自就是一个强有力的技术——你甚至想到在某些方面它有点类似于面向对象技术：outer是inner的构造函数，x扮演着一个类似私有成员变量的角色。它的作用有很多，如果你熟悉Python的sorted函数的key参数，你可能已经写过一个lambda函数通过第二项而不是第一项来排序一些列list。也许你现在可以写一个itemgetter函数，它接收一个用于检索的索引并返回一个函数，这个函数适合传递给key参数。


但是让我们不要用闭包做任何噩梦般的事情！相反，让我们重新从头开始来写一个decorator!


## 闭包之坑_延迟绑定
这是一个经（很）典（大）的问题（坑）

```
def multipliers():
    return [lambda x: i * x for i in range(4)]
print([m(2) for m in multipliers()])  
```
以上代码会输出什么？

答案:[6,6,6,6]

为何?闭包的延迟绑定！

有解药么？有4种，分别如下

```
#生成器
def multipliers():
    return (lambda x: i * x for i in range(4))
print([m(2) for m in multipliers()])  

#匿名函数的入参绑定
def multipliers():
    return [lambda x, j=i: j * x for i in range(4)]
print([m(2) for m in multipliers()])  

# 偏函数
from functools import partial
def multipliers_ch2():
    return [partial(lambda m, x: m * x, i) for i in range(4)]
print([m(2) for m in multipliers()])  

# yield生成器
def multipliers_ch3():
    for m in range(4):
        yield lambda x: m * x
print([m(2) for m in multipliers()])  
```
其实这里最难理解的是第一个生成器，为何·[]改成()就对了？

其实第一个和第四个(yield生成器)是同一个道理，都是“依次执行”，依次执行时i会“挨个”，“真实的取得”0123。熟悉yield的应该深有体会。至于第二种的入参绑定和偏函数其实相对容易理解。


## Decorators!

一个decorator只是一个带有一个函数作为参数并返回一个替换函数的闭包。我们将从简单的开始一直到写出有用的decorators。

```
>>> def outer(some_func):  
...     def inner():  
...         print "before some_func"  
...         ret = some_func() # 1  
...         return ret + 1  
...     return inner  
>>> def foo():  
...     return 1  
>>> decorated = outer(foo) # 2  
>>> decorated()  
before some_func  
2  
```
请仔细看我们的decorator实例。我们定义了一个接受单个参数some\_func的名为outer的函数。在outer内部我们定义了一个名为inner的嵌套函数。inner函数打印一个字符串然后调用some\_func，在#1处缓存它的返回值。some\_func的值可能在每次outer被调用时不同，但是无论它是什么我们都将调用它。最终，inner返回some_func的返回值加1，并且我们可以看到，当我们调用存储于``#2``处decorated里的返回函数时我们得到了输出的文本和一个返回值2而不是我们期望的调用foo产生的原始值1.



我们可以说“装饰”的变量是foo的一个装饰版本——由foo加上一些东西构成。实际上，如果我们写了一个有用的decorator，我们可能想用装饰了的版本一起来替换foo，从而我们可以总是得到foo的“增添某些东西”的版本。我们可以不用学习任何新语法而做到这一点——重新将包含我们函数的变量进行赋值：



``` Python

>>> foo = outer(foo)

>>> foo # doctest: +ELLIPSIS

<function inner at 0x...>

```
现在任何对foo()的调用都不会得到原始的foo，而是会得到我们经过装饰的版本！领悟到了一些decorator的思想吗？让我们写一个更加有用的装饰器。  
假设我们有一个提供坐标对象的库，它们可能只是由x, y两个坐标对组成。令人沮丧的是，这个坐标对象并不支持算术运算，并且我们无法修改这个库的源代码，因此我们不能添加这些对运算的支持。我们将做大量的运算，但是我们现在只想实现加、减函数，它们可以带两个坐标最想作为参数并做相应的算术运算。这些函数可能很容易写（为了描述我将提供一个简单的Coordinate类。  
```
>>> class Coordinate(object):

...     def __init__(self, x, y):

...         self.x = x

...         self.y = y

...     def __repr__(self)

...         return "Coord:" + str(self.__dict__)

>>> def add(a, b):

...     return Coordinate(a.x + b.x, a.y + b.y)

>>> def sub(a, b):

...      return Coordinate(a.x - b.x, a.y - b.y)

>>> one = Coordinate(100, 200)

>>> two = Coordinate(300, 200)

>>> add(one, two)

Coord:{'y': 400, 'x': 400}

```
但是，我们想当one和two都是{x: 0, y: 0},one和three的和为{x: 100, y: 200}，在不修改one, two, three的前提下结果有所不同（实在没弄明白原作者此处是什么意思^ ^）。让我们写一个边界检查decorator而不用为每个函数添加一个对输入参数做边界检查然后返回函数值！  
```
>>> def wrapper(func):

...     def checker(a, b): # 1

...         if a.x < 0 or a.y < 0:

...             a = Coordinate(a.x if a.x > 0 else 0, a.y if a.y > 0 else 0)

...         if b.x < 0 or b.y < 0:

...             b = Coordinate(b.x if b.x > 0 else 0, b.y if b.y > 0 else 0)

...         ret = func(a, b)

...         if ret.x < 0 or ret.y < 0:

...             ret = Coordinate(ret.x if ret.x > 0 else 0, ret.y if re> 0 else 0)

...         return ret

...     return checker

>>> add = wrapper(add)

>>> sub = wrapper(sub)

>>> sub(one, two)

Coord: {'y': 0, 'x': 0}

>>> add(one, three)

Coord: {'y': 200, 'x': 100}

```
这个装饰器的效果和前面实例的一样——返回一个修改过了的函数，只是在上例中对输入参数和返回值做了一些有用的检查和规范化，至于这样做是否让我们的代码变得更加简洁是一件可选择的事情：将边界检查隔绝在它自己的函数里，然后将其应用到通过用一个decorator包装将我们所关心的函数上。另一个可能的方法是每次调用算数函数时对每一个输入参数和输出结果前对参数或者结果做边界检查，毫无疑问的是使用decorator至少在对一个函数进行边界检查的代码量上重复更少。实际上，如果是装饰我们自己的函数，我们可以将装饰器应用程序写的更明显一点。

## 含参装饰器
可以在装饰器里传入一个参数，指明国籍，并在函数执行前，用自己国家的母语打一个招呼。  
```
# 小明，中国人
@say_hello("china")
def xiaoming():
    pass

# jack，美国人
@say_hello("america")
def jack():
    pass
那我们如果实现这个装饰器，让其可以实现 传参 呢？

会比较复杂，需要两层嵌套。

def say_hello(contry):
    def wrapper(func):
        def deco(*args, **kwargs):
            if contry == "china":
                print("你好!")
            elif contry == "america":
                print('hello.')
            else:
                return

            # 真正执行函数的地方
            func(*args, **kwargs)
        return deco
    return wrapper

来执行一下

xiaoming()
print("------------")
jack()
```

看看输出结果。  
```
你好!
------------
hello.
```


## 用偏函数与类实现装饰器
绝大多数装饰器都是基于函数和闭包实现的，但这并非制造装饰器的唯一方式。  
事实上，Python 对某个对象是否能通过装饰器（ @decorator）形式使用只有一个要求：decorator 必须是一个“可被调用（callable）的对象。  
对于这个 callable 对象，我们最熟悉的就是函数了。  
除函数之外，类也可以是 callable 对象，只要实现了__call__ 函数（上面几个例子已经接触过了）。  
还有容易被人忽略的偏函数其实也是 callable 对象。  
```
import time
import functools

class DelayFunc:
    def __init__(self,  duration, func):
        self.duration = duration
        self.func = func

    def __call__(self, *args, **kwargs):
        print(f'Wait for {self.duration} seconds...')
        time.sleep(self.duration)
        return self.func(*args, **kwargs)

    def eager_call(self, *args, **kwargs):
        print('Call without delay')
        return self.func(*args, **kwargs)

def delay(duration):
    """
    装饰器：推迟某个函数的执行。
    同时提供 .eager_call 方法立即执行
    """
    # 此处为了避免定义额外函数，
    # 直接使用 functools.partial 帮助构造 DelayFunc 实例
    return functools.partial(DelayFunc, duration)
我们的业务函数很简单，就是相加

@delay(duration=2)
def add(a, b):
    return a+b
```
来看一下执行过程  
```
>>> add    # 可见 add 变成了 Delay 的实例
<__main__.DelayFunc object at 0x107bd0be0>
>>>
>>> add(3,5)  # 直接调用实例，进入 __call__
Wait for 2 seconds...
8
>>>
>>> add.func # 实现实例方法
<function add at 0x107bef1e0>
```


## 参考
【翻译】12步理解Python Decorators：https://harveyqing.gitbooks.io/python-read-and-write/content/python_advance/python_decorator_in_12_steps.html  
装饰器进阶用法详解：python.iswbm.com/en/latest/c03/c03_01.html  
python中的闭包:https://blog.csdn.net/weixin_44141532/article/details/87116038  

