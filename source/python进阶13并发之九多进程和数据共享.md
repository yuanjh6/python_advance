# python进阶13并发之九多进程和数据共享
使用进程，大概率出现情况是，想当然以为共享了，实际没共享。所以最终程序大概率卡死（部分逻辑没有数据进来，导致的业务逻辑性卡住，并非程序死锁）


## 哪些共享，哪些不共享
默认进程是**都不共享，包括全局变量**。

父子进程其实处于**不同的资源空间**（进程是系统分配资源的最小单位），所以2进程其实是完全独立的资源空间，数据自然无法直接交互。如果要交互，必须**超越进程内部**，进入到操作系统层面，比如文件方式，等进行交互。

其实父子进程是一种非常**松散的关系**，在一个app中启动另一个app的关系就可以看做父子进程。只是开发过程中，大多数父子进程都是强业务关联的，所以自然会有一种**虚幻的“亲密关系”**，实际上这只是想当然的，子进程一旦启动，基本就和父进程没什么关系了，虽然不是完全没有，比如守护进程需要在父进程退出时也退出，但这个是操作系统的权利，而非父进程的权利。

所以，可以认为进程内变量都不共享!


##  如何实现共享(参考博文:python进阶06并发之二技术点关键词)

## 共享数据的同步
这个问题其实线程也存在，所以解决方法其实和线程类似的。

优点是进程数据默认独立的，凡是共享数据，其实都是非常“突出”的，容易定位，找到多个进程的共享变量，相关临界区加上锁即可(视情况，未必一定是“加锁的方式”，或一定需要加锁，原子操作依然不需加锁)

另外需要留意以下几点:

01,虽然使用了manager()封装好的dict()或list()对象，这些对象只能保证数据的共享，不能保证数据的同步，也就是说也需要自己考虑加锁问题。

02,dict=manager().dict(),dict['abc']=list('asdf'),这个赋值操作是进程安全的，而dict['abc'][2]='f',就未必了，应当避免使用后者的赋值方式。(不要尝试绕过manager()的操作，最好采用最为稳妥的赋值方式)

03,创建进程时，参数必须能够被pickle，所以有些自定义的类对象实例是不能被作为参数的。和threading不同，multiprocessing Process参数必须能够被pickle进行序列化


## 使用了multiprocessing.Value为何还需要加锁

本质上而言，**共享和锁没有必然关系**!，只是**不共享不需加锁**，**共享协程不加锁**，**共享普通进程（线程）该加锁还是要加锁**

在multiprocessing库中的Value是细粒度的，Value中有一个ctypes类型的对象，拥有一个value属性来表征内存中实际的对象。Value可以保证同时只有一个单独的线程或进程在读或者写value值。这么看起来没有什么问题。

然而在第一个进程加载value值的时候，程序却不能阻止第二个进程加载旧的值。两个进程都会把value拷贝到自己的私有内存然后进行处理，并写回到共享值里。

共享变量虽然**做了一定锁封装**，但是其**锁粒度都是非常细**的，而程序中**涉及临界区可能比较大**，**仅依靠共享变量自身锁是无法限制住**的。

## 使用类变量可以实现跨进程共享?
否。类变量仅能实现同进程内，跨实例共享! (仔细想想很清楚，但开发过程中容易忽略，想当然以为是全局的，实际只是进程内部的全局，而非web应用层面的全局)


## 什么可以被 pickle
参见[what is pickable](https://docs.python.org/2/library/pickle.html#what-can-be-pickled-and-unpickled)。
```
None，True，False，内建数字类型，字符串
picklable 对象组成的 tuple、list、set、dict
在模块顶层中定义的函数、内建函数、类
其 __dict__ 或 __getstate__() 是可 pickle 的类实例
特别地，numpy ndarray
注意 lambda 匿名函数不可被 pickle，除非使用 dill 之类的包，见此。
详见 http://luly.lamost.org/blog/python_multiprocessing.html
```

## 参考
[Python多进程相关的坑](https://www.cnblogs.com/li-dp/p/5837823.html)

[Python分布式进程中你会遇到的坑](https://blog.csdn.net/licheetools/article/details/82946312)

[Python多进程开发中使用Manager进行数据共享的陷阱](https://www.jianshu.com/p/52676b93430d)

[python多进程Pool使用遇到的坑](https://blog.csdn.net/u013887652/article/details/104007145)

[python multiprocessing多进程变量共享与加锁](https://www.jianshu.com/p/a682b03e32b9)

[python 多进程共享对象(bug日记)](https://blog.csdn.net/yournevermore/article/details/88718745)


