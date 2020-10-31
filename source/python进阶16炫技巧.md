# python进阶16炫技巧
原则：**可读性第一**（效率固然重要，除非非常明显的效率差异，否则可读性优先)

学习炫技巧，更多为了读懂他人代码，自己开发过程中，相似代码量（可读性），建议使用通俗写法。反对为炫而炫。



## 可直接运行的 zip 包
有 Python 包，居然可以以 zip 包进行发布，并且可以不用解压直接使用。

这个zip 是如何制作的呢，请看下面的示例。

```
[root@localhost ~]# ls -l demo
total 8
-rw-r--r-- 1 root root 30 May  8 19:27 calc.py
-rw-r--r-- 1 root root 35 May  8 19:33 __main__.py

[root@localhost ~]# cat demo/__main__.py
import calc
print(calc.add(2, 3))
[root@localhost ~]# cat demo/calc.py
def add(x, y):
    return x+y
[root@localhost ~]# python -m zipfile -c demo.zip demo/*
```
制作完成后，我们可以执行用 python 去执行它

```
[root@localhost ~]# python demo.zip
5
```

## 懒人必备技能：使用 “_”
大家对于他的印象都是用于 占位符，省得为一个不需要用到的变量，绞尽脑汁的想变量名。

今天要介绍的是他的第二种用法，就是在 console 模式下的应用。

示例如下：

```
>>> 3 + 4
7
>>> _
7
>>> name='公众号: Python编程时光'
>>> name
'公众号: Python编程时光'
>>> _
'公众号: Python编程时光'
```
它可以返回上一次的运行结果。

但是，如果是print函数打印出来的就不行了。

```
>>> 3 + 4
7
>>> _
7
>>> print("公众号: Python编程时光")
ming
>>> _
7
```

## 最快查看包搜索路径的方式
```
python3 -m site
sys.path = [
    '/home/wangbm',
    '/usr/local/Python3.7/lib/python37.zip',
    '/usr/local/Python3.7/lib/python3.7',
    '/usr/local/Python3.7/lib/python3.7/lib-dynload',
    '/home/wangbm/.local/lib/python3.7/site-packages',
    '/usr/local/Python3.7/lib/python3.7/site-packages',
]
USER_BASE: '/home/wangbm/.local' (exists)
USER_SITE: '/home/wangbm/.local/lib/python3.7/site-packages' (exists)
ENABLE_USER_SITE: True
```
## and 和 or 的取值顺序
当一个 or 表达式中所有值都为真，Python会选择第一个值

当一个 and 表达式 所有值都为真，Python 会选择第二个值。

示例如下：

```
>>>(2 or 3) * (5 and 7)
14  # 2*7
```

## 访问类中的私有方法
```
# 调用私有方法，以下两种等价
ins._Kls__private()
ins.call_private()
```


## 一行代码实现FTP服务器
python3 -m http.server 8888


## for else逻辑
for else 和 try else 相同，只要代码正常走下去不被 break，不抛出异常，就可以走else。(所以和去掉else，直接写后面没区别?)

优点在于，**可以识别正常退出还是break退出**（一般业务含义不同）


## 嵌套上下文管理的另类写法
with test_context('aaa'), test_context('bbb'):
    print('========== in main ============')
## 连接多个列表最极客的方式

```
>>> b = [3,4]
>>> c = [5,6]
>>>
>>> sum((a,b,c), [])
[1, 2, 3, 4, 5, 6]
```
另外几种连接列表的方式

```
>>> list01 + list02 + list03
[1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> list(chain(list01, list02, list03))
>>> [*list01, *list02]
>>> list01.extend(list02)
>>> [x for l in (list01, list02, list03) for x in l]
>>> from heapq import merge
>>> list(merge(list01, list02, list03))
sorted(itertools.chain(*iterables))
```

## 在程序退出前执行代码的技巧
使用 atexit 这个内置模块，可以很方便的注册退出函数。

如果clean()函数有参数，那么你可以不用装饰器，而是直接调用atexit.register(clean_1, 参数1, 参数2, 参数3='xxx')。


## 合并字典的几种方法
```
profile.update(ext_info)
full_profile01 = {**profile, **ext_info}
dict(itertools.chain(profile.items(), ext_info.items()))
dict(ChainMap(profile, ext_info))
full_profile = dict(profile.items() | ext_info.items())
```
## 条件语句的几种写法
```
<on_true> if <condition> else <on_false>
<condition> and <on_true> or <on_false>
(<on_true>, <on_false>)[condition]
(lambda: <on_false>, lambda:<on_true>)[<condition>]()
>>> msg2 = (lambda:"未成年", lambda:"已成年")[age2 > 18]()
>>> print(msg2)
未成年
 msg2 = {True: "已成年", False: "未成年"}[age2 > 18]
>>> print(msg2)
未成年
```

## 让我爱不释手的用户环境
当你在机器上并没有 root 权限时，如何安装 Python 的第三方包呢？

可以使用 pip install --user pkg 将你的包安装在你的用户环境中，该用户环境与全局环境并不冲突，并且多用户之间相互隔离，互不影响。

## with 与 上下文管理器
样例:

```
import contextlib

@contextlib.contextmanager
def open_func(file_name):
    # __enter__方法
    print('open file:', file_name, 'in __enter__')
    file_handler = open(file_name, 'r')

    try:
        yield file_handler
    except Exception as exc:
        # deal with exception
        print('the exception was thrown')
    finally:
        print('close file:', file_name, 'in __exit__')
        file_handler.close()

        return

with open_func('/Users/MING/mytest.txt') as file_in:
    for line in file_in:
        1/0
        print(line)
```
## 在 linux 上看 json 文件
cat test.json | python -m json.tool


## sh，最优雅的命令调用方式
```
 >>> sh.glob("/etc/*.conf")
['/etc/mke2fs.conf', '/etc/dnsmasq.conf', '/etc/asound.conf']
>>> r=sh.Command('/root/test.py')
>>> r()
hello,world
```
## 判断是否包含子串的七种方法
```
1. 使用 in 和 not in
2. 使用 find 方法
3. 使用 index 方法
4. 使用 count 方法
5，借助 operator
```
operator模块是python中内置的操作符函数接口，它定义了一些算术和比较内置操作的函数。operator模块是用c实现的，所以执行速度比 python 代码快。

在 operator 中有一个方法 contains 可以很方便地判断子串是否在字符串中。

```
>>> import operator
>>>
>>> operator.contains("hello, python", "llo")
```
## 使用json.dumps打印字典
打印中文在 json.dumps 这里，却只要加个参数就好了

具体的代码示例如下：

```
>>> import json
>>> print json.dumps(info, indent=4, ensure_ascii=False)
```
## slots
JIT即时编译器，当虚拟机发现某个方法或代码块运行特别频繁时，就会把这些代码认定为热点代码，为了提高热点代码的运行效率，在运行时，虚拟机将会把这些代码编译成与本地平台的相关带代码，并进行各层次的优化。

正如上面所说的，默认情况下,Python的新式类和经典类的实例都有一个dict来存储实例的属性。这在一般情况下还不错，而且非常灵活，

乃至在程序中可以随意设置新的属性。但是，对一些在”编译”前就知道有几个固定属性的小class来说，这个dict就有点浪费内存了。

当需要创建大量实例的时候，这个问题变得尤为突出。一种解决方法是在新式类中定义一个__slots__属性。

__slots__声明中包含若干实例变量，并为每个实例预留恰好足够的空间来保存每个变量；这样Python就不会再使用dict，从而节省空间。

##  stateful service 排障

来个真的有点高（黑）级（暗）的技巧吧，开发一个 stateful service 的时候怎么排障？比如出现死锁了，或者某个线程意外退出。这个时候可以从

gc.get_objects()

掏出所有活着的对象，其中当然就包括活着的线程。



例如，打印出所有 Greenlet 的 stack：

```

import os

import gc

import greenlet

import traceback



greenlets = [

    o for o in gc.get_objects() if isinstance(o, greenlet.greenlet) if o]

stack = '\n\n'.join(

    ''.join(traceback.format_stack(o.gr_frame)) for o in greenlets)

open('/tmp/stack-%d.txt' % os.getpid(), 'w').write(stack)

```

这个方法结合 gevent.backdoor 堪称排障利器。如果愿意冒一定的风险，甚至可以使用调试器对事先没有埋过点的进程做活体检测——挂上 GIL 再打印一份线程 stack。详见 GitHub - wooparadog/pstack: Tool to dump python thread and greenlet stacks.



## 调试利器，显示调用栈

有时候BUG隐藏的太深，需要对上下文都有清晰的展示来帮助判断。用pdb调试不方便，用print不直观。可以使用如下函数获取当前调用栈：

```

import sys

def get_cur_info():

	print sys._getframe().f_code.co_filename # 当前文件名

	print sys._getframe(0).f_code.co_name # 当前函数名

	print sys._getframe(1).f_code.co_name # 调用该函数的函数的名字，如果没有被调用，则返回module

	print sys._getframe().f_lineno # 当前行号

```

## x入参
```
def testa(*v):
    print(list(v))


testa(1, 2, 3)
testa(list('abc'))
testa('a', 'b', list('cde'))

[1, 2, 3]
[['a', 'b', 'c']]
['a', 'b', ['c', 'd', 'e']]
```
## 内存占用  
下面的代码块可以检查变量 variable 所占用的内存。  
```  
import sys   
  
variable = 30   
print(sys.getsizeof(variable)) # 24  
```  
  
## 打印 N 次字符串  
该代码块不需要循环语句就能打印 N 次字符串。  
```
n = 2;

s ="Programming";



print(s * n);

# ProgrammingProgramming

```
## 解包  
如下代码段可以将打包好的成对列表解开成两组不同的元组。  
```
array = [['a', 'b'], ['c', 'd'], ['e', 'f']]

transposed = zip(*array)

print(transposed)

# [('a', 'c', 'e'), ('b', 'd', 'f')]

```
## 链式函数调用  
你可以在一行代码内调用多个函数。  
```
def add(a, b):

    return a + b



def subtract(a, b):

    return a - b



a, b = 4, 5

print((subtract if a > b else add)(a, b)) # 9

```
## 回文序列  
以下方法会检查给定的字符串是不是回文序列，它首先会把所有字母转化为小写，并移除非英文字母符号。最后，它会对比字符串与反向字符串是否相等，相等则表示为回文序列。  
```
def palindrome(string):

    from re import sub

    s = sub('[\W_]', '', string.lower())

    return s == s[::-1]



palindrome('taco cat') # True

```
## 不使用 if-else 的计算子  
这一段代码可以不使用条件语句就实现加减乘除、求幂操作，它通过字典这一数据结构实现：  
```
import operator

action = {

    "+": operator.add,

    "-": operator.sub,

    "/": operator.truediv,

    "*": operator.mul,

    "**": pow

}

print(action['-'](50, 25)) # 25

```


## 参考
Python 黑魔法指南 50 例:python.iswbm.com/en/latest/c01/c01_10.html  
Python 炫技操作：连接列表的八种方法：python.iswbm.com/en/latest/c01/c01_41.html  
Python 炫技操作：判断是否包含子串的七种方法：python.iswbm.com/en/latest/c01/c01_40.html   
Python 炫技操作：合并字典的七种方法：python.iswbm.com/en/latest/c01/c01_39.html  
Python 炫技操作：条件语句的七种写法：python.iswbm.com/en/latest/c01/c01_37.html  
每日一库：sh，最优雅的命令调用方式：python.iswbm.com/en/latest/c01/c01_34.html  
Python 开发中有哪些高级技巧？:https://www.zhihu.com/question/23760468  
python __slots__ 详解（上篇）:https://blog.csdn.net/sxingming/article/details/52892640  
30段极简Python代码：这些小技巧你都Get了么:https://zhuanlan.zhihu.com/p/109016233
