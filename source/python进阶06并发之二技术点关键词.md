# python进阶06并发之二技术点关键词
## GIL,线程锁
python中存在GIL这个"线程锁",

关键地方可以使用c语言解决 GIL问题 然后可以提高cpu占用效率


## 守护进程
主进程创建守护进程

1）守护进程会在主进程代码执行结束后就终止

2）守护进程内无法再开启子进程,否则抛出异常：AssertionError: daemonic processes are not allowed to have children

注意：进程之间是互相独立的，主进程代码运行结束，守护进程随即终止

```
#主进程代码运行完毕,守护进程就会结束
from multiprocessing import Process
from threading import Thread
import time
def foo():
    print(123)
    time.sleep(1)
    print("end123")

def bar():
    print(456)
    time.sleep(3)
    print("end456")


p1=Process(target=foo)
p2=Process(target=bar)

p1.daemon=True
p1.start()
p2.start()
print("main-------") #打印该行则主进程代码结束,则守护进程p1应该被终止,可能会有p1任务执行的打印信息123,因为主进程打印main----时,p1也执行了,但是随即被终止
```

## 互斥锁(mutex)

为了方式上面情况的发生，就出现了互斥锁(Lock)

```
import threading
import time
 
 
def run(n):
    lock.acquire()  #获取锁
    global num
    num += 1
    lock.release()  #释放锁
 
lock = threading.Lock()     #实例化一个锁对象
 
num = 0
t_obj = []  
 
for i in range(20000):
    t = threading.Thread(target=run, args=("t-%s" % i,))
    t.start()
    t_obj.append(t)
 
for t in t_obj:
    t.join()
 
print "num:", num
```
## RLock 递归锁(了解)
## 队列(推荐)

Queue是多进程的安全队列，可以使用Queue实现多进程之间的数据传递。

```
Queue.qsize()：返回当前队列包含的消息数量；
Queue.empty()：如果队列为空，返回True，反之False ；
Queue.full()：如果队列满了，返回True,反之False；
Queue.get():获取队列中的一条消息，然后将其从列队中移除，可传参超时时长。
Queue.get_nowait()：相当Queue.get(False),取不到值时触发异常：Empty；
Queue.put():将一个值添加进数列，可传参超时时长。
Queue.put_nowait():相当于Queue.get(False),当队列满了时报错：Full。

from multiprocessing import Process, Queue
import time

def write(q):
   for i in ['A', 'B', 'C', 'D', 'E']:
      print('Put %s to queue' % i)
      q.put(i)
      time.sleep(0.5)

def read(q):
   while True:
      v = q.get(True)
      print('get %s from queue' % v)

if __name__ == '__main__':
   q = Queue()
   pw = Process(target=write, args=(q,))
   pr = Process(target=read, args=(q,))
   print('write process = ', pw)
   print('read  process = ', pr)
   pw.start()
   pr.start()
   pw.join()
   pr.join()
   pr.terminate()
   pw.terminate()
```

## 管道(了解)

## 共享数据(Manager)

```
if __name__ == '__main__':
    with multiprocessing.Manager() as MG: #重命名
        mydict=MG.dict()#主进程与子进程共享这个字典
        mylist=MG.list(range(5))#主进程与子进程共享这个LIST

        p=multiprocessing.Process(target=func,args=(mydict,mylist))

        p.start()
        p.join()

        print(mylist)
        print(mydict)
```
## 信号量(了解）

## 事件
事件（Event类）
python线程的事件用于主线程控制其他线程的执行，事件是一个简单的线程同步对象，其主要提供以下几个方法：

| 方法 | 注释 |
| --- | --- |
| clear | 将flag设置为“False” |
| set | 将flag设置为“True” |
| is_set | 判断是否设置了flag |
| wait | 会一直监听flag，如果没有检测到flag就一直处于阻塞状态 |

事件处理的机制：全局定义了一个“Flag”，当flag值为“False”，那么event.wait()就会阻塞，当flag值为“True”，那么event.wait()便不再阻塞。

```
#利用Event类模拟红绿灯
import threading
import time
 
event = threading.Event()
  
def lighter():
    count = 0
    event.set()     #初始值为绿灯
    while True:
        if 5 < count <=10 :
            event.clear()  # 红灯，清除标志位
            print("\33[41;1mred light is on...\033[0m")
        elif count > 10:
            event.set()  # 绿灯，设置标志位
            count = 0
        else:
            print("\33[42;1mgreen light is on...\033[0m")
 
        time.sleep(1)
        count += 1
 
def car(name):
    while True:
        if event.is_set():      #判断是否设置了标志位
            print("[%s] running..."%name)
            time.sleep(1)
        else:
            print("[%s] sees red light,waiting..."%name)
            event.wait()
            print("[%s] green light is on,start going..."%name)
 
light = threading.Thread(target=lighter,)
light.start()
 
car = threading.Thread(target=car,args=("MINI",))
car.start()
```
## fork
Unix/Linux操作系统提供了一个fork()系统调用，它非常特殊。普通的函数调用，调用一次，返回一次，但是fork()调用一次，返回两次，因为操作系统自动把当前进程（称为父进程）复制了一份（称为子进程），然后，分别在父进程和子进程内返回。

子进程永远返回0，而父进程返回子进程的ID。这样做的理由是，一个父进程可以fork出很多子进程，所以，父进程要记下每个子进程的ID，而子进程只需要调用getppid()就可以拿到父进程的ID。

Python的os模块封装了常见的系统调用，其中就包括fork，可以在Python程序中轻松创建子进程：

```
import os

print('Process (%s) start...' % os.getpid())
# Only works on Unix/Linux/Mac:
pid = os.fork()
if pid == 0:
    print('I am child process (%s) and my parent is %s.' % (os.getpid(), os.getppid()))
else:
    print('I (%s) just created a child process (%s).' % (os.getpid(), pid))

运行结果如下：

Process (876) start...
I (876) just created a child process (877).
I am child process (877) and my parent is 876.
```
由于Windows没有fork调用，上面的代码在Windows上无法运行。而Mac系统是基于BSD（Unix的一种）内核，所以，在Mac下运行是没有问题的，


## Process模块
​1.注意：Process对象可以创建进程，**但Process对象不是进程**，其**删除与否与系统资源是否被回收没有直接的关系**。

2.主进程执行完毕后会默认等待子进程结束后回收资源，不需要手动回收资源；join()函数用来控制子进程结束的顺序,其**内部也有一个清除僵尸进程的函数，可以回收资源**；

3.Process进程创建时，子进程会将主进程的Process对象完全复制一份，这样在主进程和子进程各有一个 Process对象，**但是p.start()启动的是子进程，主进程中的Process对象作为一个静态对象存在，不执行**。

4.当**子进程执行完毕后，会产生一个僵尸进程，其会被join函数回收**，或者再有一条进程开启，**start函数也会回收僵尸进程**，所以不一定需要写join函数。

5.windows系统在子进程结束后会立即自动清除子进程的Process对象，而**linux系统子进程的Process对象如果没有join函数和start函数的话会在主进程结束后统一清除**。


进程直接的内存空间是隔离的

```
from multiprocessing import Process
n=100 #在windows系统中应该把全局变量定义在if __name__ == '__main__'之上就可以了
def work():
    global n
    n=0
    print('子进程内: ',n)


if __name__ == '__main__':
    p=Process(target=work)
    p.start()
    print('主进程内: ',n)
```


## multiprocessing模块
Process模块是一个创建进程的模块,借助这个模块可以创建进程

```
Process([group [, target [, name [, args [, kwargs]]]]])，由该类实例化得到的对象，表示一个子进程中的任务（尚未启动）
```
强调：

1. 需要使用关键字的方式来指定参数

2. args指定的为传给target函数的位置参数，是一个元组形式，必须有逗号


参数介绍：

group参数未使用，值始终为None

target表示调用对象，即子进程要执行的任务

args表示调用对象的位置参数元组，args=(1,2,'egon',)

kwargs表示调用对象的字典,kwargs={'name':'egon','age':18}

name为子进程的名称


方法介绍

p.start()：启动进程，并调用该子进程中的p.run()

p.run():进程启动时运行的方法，正是它去调用target指定的函数，我们自定义类的类中一定要实现该方法

p.terminate():强制终止进程p，不会进行任何清理操作，如果p创建了子进程，该子进程就成了僵尸进程，

使用该方法需要特别小心这种情况。如果p还保存了一个锁那么也将不会被释放，进而导致死锁

p.is_alive():如果p仍然运行，返回True

p.join([timeout]):主线程等待p终止（强调：是主线程处于等的状态，而p是处于运行的状态）。

timeout是可选的超时时间，需要强调的是，p.join只能join住start开启的进程，而不能join住run开启的进程


属性介绍

p.daemon：默认值为False，如果设为True，代表p为后台运行的守护进程，当p的父进程终止时，p也随之终止，

并且设定为True后，p不能创建自己的新进程，必须在p.start()之前设置

p.name:进程的名称

p.pid：进程的pid

p.exitcode:进程在运行时为None、如果为–N，表示被信号N结束(了解即可)

p.authkey:进程的身份验证键,默认是由os.urandom()随机生成的32字符的字符串。这个键的用途是

为涉及网络连接的底层进程间通信提供安全性，这类连接只有在具有相同的身份验证键时才能成功（了解即可）


window中使用Process注意事项:

在Windows操作系统中由于没有fork(linux操作系统中创建进程的机制)，在创建子进程的时候会自动 import 启动它的这个文件，而在 import 的时候又执行了整个文件。因此如果将process()直接写在文件中就会无限递归创建子进程报错。所以必须把创建子进程的部分使用if __name__ ==‘__main__’ 判断保护起来，import 的时候 ，就不会递归运行了。



## 进程池
由于进程启动的开销比较大，使用**多进程的时候会导致大量内存空间被消耗**。为了防止这种情况发生可以使用进程池，（由于启动线程的开销比较小，所以不需要线程池这种概念，多线程只会频繁得切换cpu导致系统变慢，并不会占用过多的内存空间）

进程池中常用方法：

```
1 p.apply(func [, args [, kwargs]])
在一个池工作进程中执行func(*args,**kwargs),然后返回结果。
需要强调的是：此操作并不会在所有池工作进程中并执行func函数。如果要通过不同参数并发地执行func函数，必须从不同线程调用p.apply()函数或者使用p.apply_async()
2 p.apply_async(func [, args [, kwargs]]):
在一个池工作进程中执行func(*args,**kwargs),然后返回结果。
此方法的结果是AsyncResult类的实例，callback是可调用对象，接收输入参数。当func的结果变为可用时，
将理解传递给callback。callback禁止执行任何阻塞操作，否则将接收其他异步操作中的结果。   
如果传递给apply_async()的函数如果有参数，需要以元组的形式传递 并在最后一个参数后面加上 , 号，如果没有加, 号，提交到进程池的任务也是不会执行的  
3 p.close():关闭进程池，防止进一步操作。如果所有操作持续挂起，它们将在工作进程终止前完成
4 P.jion():等待所有工作进程退出。此方法只能在close（）或teminate()之后调用

from  multiprocessing import Process,Pool
import time
 
def Foo(i):
    time.sleep(2)
    return i+100
 
def Bar(arg):
    print('-->exec done:',arg)
 
pool = Pool(5)  #允许进程池同时放入5个进程
 
for i in range(10):
    pool.apply_async(func=Foo, args=(i,),callback=Bar)  #func子进程执行完后，才会执行callback，否则callback不执行（而且callback是由父进程来执行了）
    #pool.apply(func=Foo, args=(i,))
 
print('end')
pool.close()
pool.join() #主进程等待所有子进程执行完毕。必须在close()或terminate()之后。
```
进程池内部维护一个进程序列，当使用时，去进程池中获取一个进程，如果进程池序列中没有可供使用的进程，那么程序就会等待，直到进程池中有可用进程为止。在上面的程序中产生了10个进程，但是只能有5同时被放入进程池，剩下的都被暂时挂起，并不占用内存空间，等前面的五个进程执行完后，再执行剩下5个进程。

回调函数：进程池支持回调函数


## 协程(gevent)
协程 :

能够在一个线程中实现并发效果的效果,提高cpu的利用率

无需原子操作锁定及同步的开销

能够规避一些任务中的IO操作

方便切换控制流，简化编程模型

协程相比于多线程的优势  切换的效率更快了 　

缺点：

无法利用多核资源：协程的本质是个单线程,它不能同时将 单个CPU 的多个核用上,协程需要和进程配合才能运行在多CPU上.当然我们日常所编写的绝大部分应用都没有这个必要，除非是cpu密集型应用。


线程和进程的操作是由程序触发系统接口，最后的执行者是系统，它本质上是操作系统提供的功能。而协程的操作则是程序员指定的，在python中通过yield，人为的实现并发处理。

协程存在的意义：对于多线程应用，CPU通过切片的方式来切换线程间的执行，线程切换时需要耗时。协程，则只使用一个线程，分解一个线程成为多个“微线程”，在一个线程中规定某个代码块的执行顺序。

协程的适用场景：当程序中存在大量不需要CPU的操作时（IO）。

常用第三方模块gevent和greenlet。（本质上，gevent是对greenlet的高级封装，因此一般用它就行，这是一个相当高效的模块。）


**greenlet**

```
from greenlet import greenlet
 
def test1():
    print(12)
    gr2.switch()
    print(34)
    gr2.switch()
 
def test2():
    print(56)
    gr1.switch()
    print(78)
 
gr1 = greenlet(test1)
gr2 = greenlet(test2)
gr1.switch()
```
实际上，greenlet就是通过switch方法在不同的任务之间进行切换。


**gevent**

```
from gevent import monkey; monkey.patch_all()
import gevent
import requests
 
def f(url):
    print('GET: %s' % url)
    resp = requests.get(url)
    data = resp.text
    print('%d bytes received from %s.' % (len(data), url))
 
gevent.joinall([
        gevent.spawn(f, 'https://www.python.org/'),
        gevent.spawn(f, 'https://www.yahoo.com/'),
        gevent.spawn(f, 'https://github.com/'),
])
```
通过joinall将任务f和它的参数进行统一调度，实现单线程中的协程。代码封装层次很高，实际使用只需要了解它的几个主要方法即可。


## ThreadLocal 
创建一个全局的ThreadLocal对象,每个线程有独立的存储空间,每个线程对ThreadLocal对象都可以读写，但是互不影响.

```
import threading

# 创建全局ThreadLocal对象:
local = threading.local()

def process_student():
    # 获取当前线程关联的student:
    print('local.student: %s , current_thread : %s' % (local.student, threading.current_thread().name))

def process_thread(stu_name):
    # 绑定ThreadLocal的student:
    local.student = stu_name
    process_student()

t1 = threading.Thread(target= process_thread, args=('Alice',), name='Thread-A')
t2 = threading.Thread(target= process_thread, args=('Bob',), name='Thread-B')
t1.start()
t2.start()
t1.join()
t2.join()
# local.student: Alice , current_thread : Thread-A
# local.student: Bob , current_thread : Thread-B
```
全局变量local就是一个ThreadLocal对象，每个Thread对它都可以读写student属性，但互不影响。 
你可以把local看成全局变量，但每个属性如local.student都是线程的局部变量，

可以任意读写而互不干扰，也不用管理锁的问题，ThreadLocal内部会处理。

可以理解为全局变量local是一个dict，不但可以用local.student，还可以绑定其他变量，

如local.teacher等等。

应用:ThreadLocal最常用的地方就是为每个线程绑定一个数据库连接，HTTP请求，用户身份信息等，

这样一个线程的所有调用到的处理函数都可以非常方便地访问这些资源。


小结

一个ThreadLocal变量虽然是全局变量，但每个线程都只能读写自己线程的独立副本，互不干扰。

ThreadLocal解决了参数在一个线程中各个函数之间互相传递的问题。

```
import threading, time
local = threading.local()  # 创建一个全局的ThreadLocal对象
num = 0  # 将线程中需要访问的变量绑定到全局ThreadLocal对象上

def run(x, n):
    x = x + n
    x = x - n
    return  x

def func(n):
    #每个线程都有local.x，就是线程的局部变量
    local.x = num							 # 在线程调用的函数中, 将访问的变量和ThreadLock绑定
    for i in range(1000000):
        run(local.x, n)
    print("%s-  local.x =%d"%(threading.current_thread().name, local.x))

if __name__ == "__main__":
    iTimeStart = time.time()
    t1 = threading.Thread(target=func, args=(6,))
    t2 = threading.Thread(target=func, args=(9,))
    t1.start()
    t2.start()
    t1.join()
    t2.join()

    print("num =",num)
    iTimeEnd = time.time()
    print(iTimeEnd - iTimeStart)   # 1.6630573272705078
# 不仅不会导致数据混乱, 而且所用时间已经接近不加锁的时间.
```

## 参考
[进程和线程、协程的区别](https://www.cnblogs.com/lxmhhy/p/6041001.html)

[进程 vs. 线程](https://www.liaoxuefeng.com/wiki/1016959663602400/1017631469467456)

[以Python爬取数据为例，多线程和多进程的优劣](https://blog.csdn.net/u014603907/article/details/99747107)

[在多核CPU下，同一进程下的多个线程可以并行运行吗](https://bbs.csdn.net/topics/270083226)

[python并发编程之多进程(实践篇)](https://www.cnblogs.com/jiangfan95/p/11439207.html)

[python多进程原理及其实现（1-6总结，较好）](https://blog.csdn.net/qq_31362767/article/details/87474466)

[python之路多进程和多线程总结(四)](https://www.cnblogs.com/zhen1996/articles/9692988.html)

[一文看懂Python多进程与多线程编程(工作学习面试必读)](https://blog.csdn.net/weixin_42134789/article/details/82992326)

[搞定python多线程和多进程（详细）](https://blog.csdn.net/qq_34802511/article/details/81233324)

[Python的进程间通信](https://www.jianshu.com/p/acf67126d804)

[Python进程间共享数据（三）（dict、list）](https://blog.csdn.net/ssssSFN/article/details/93517467)

[多进程,多线程,协程实现简单举例](https://blog.csdn.net/qq_31362767/article/details/88728839#_1)

[异步IO、多线程、多进程](https://blog.csdn.net/m0_37886429/article/details/82385500)

[Python开发【第九章】：线程、进程和协程](https://www.cnblogs.com/lianzhilei/p/5881434.html)

[multiprocess模块使用进程池调用apply_async()提交的函数及回调函数不执行问题](https://blog.csdn.net/weixin_40976261/article/details/89006082)  