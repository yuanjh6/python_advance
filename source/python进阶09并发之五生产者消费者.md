# python进阶09并发之五生产者消费者
这也是实际项目中使用较多的一种并发模式，用Queue(JoinableQueue)实现，是Python中最常用的方式(这里的queue特指multiprocess包下的queue，非queue.Queue)。


## Queue
```
# encoding:utf-8
__author__ = 'Fioman'
__time__ = '2019/3/7 14:06'
from multiprocessing import Process,Queue
import time,random

def consumer(q,name):
    while True:
        food = q.get()
        if food is None:
            print('接收到了一个空,生产者已经完事了')
            break

        print('\033[31m{}消费了{}\033[0m'.format(name,food))
        time.sleep(random.random())

def producer(name,food,q):
    for i in range(10):
        time.sleep(random.random())
        f = '{}生产了{}{}'.format(name,food,i)
        print(f)
        q.put(f)



if __name__ == '__main__':
    q = Queue(20)
    p1 = Process(target=producer,args=('fioman','包子',q))
    p2 = Process(target=producer,args=('jingjing','馒头',q))
    p1.start()
    p2.start()

    c1 = Process(target=consumer,args=(q,'mengmeng'))
    c2 = Process(target=consumer,args=(q,'xiaoxiao'))
    c1.start()
    c2.start()

    # 让主程序可以等待子进程的结束.
    p1.join()
    p2.join()
    # 生产者的进程结束,这里需要放置两个空值,供消费者获取,用来判断已经没有存货了
    q.put(None)
    q.put(None)

    print('主程序结束..........')
```
## JoinableQueue
创建可连接的共享进程队列,它们也是队列,但是这些队列比较特殊.它们可以允许消费者通知生产者项目已经被成功处理.注意,这里必须是生产者生产完了,生产者的进程被挂起,等到消费者完全消费的时候,生产者进程就结束,然后主程序结束.将消费者进程设置为守护进程,这样的话,主进程结束的时候,消费进程也就结束了.

JoinableQueue()比普通的Queue()多了两个方法:

```
q.task_done() 
使用者使用此方法发出信号，表示q.get()返回的项目已经被处理。如果调用此方法的次数大于从队列中删除的项目数量，将引发ValueError异常。

q.join() 
生产者将使用此方法进行阻塞，直到队列中所有项目均被处理。阻塞将持续到为队列中的每个项目均调用q.task_done()方法为止。 
# encoding:utf-8
__author__ = 'Fioman'
__time__ = '2019/3/7 14:06'
from multiprocessing import Process,JoinableQueue
import time,random

def consumer(q,name):
    while True:
        food = q.get()
        if food is None:
            print('接收到了一个空,生产者已经完事了')
            break

        print('\033[31m{}消费了{}\033[0m'.format(name,food))
        time.sleep(random.random())
        q.task_done()  # 向生产者发送信号,表示消费了一个

def producer(name,food,q):
    for i in range(10):
        time.sleep(random.random())
        f = '{}生产了{}{}'.format(name,food,i)
        print(f)
        q.put(f)
    q.join() # 当生产者生产完毕之后,会在这里阻塞.等待消费者的消费.



if __name__ == '__main__':
    q = JoinableQueue(20)
    p1 = Process(target=producer,args=('fioman','包子',q))
    p2 = Process(target=producer,args=('jingjing','馒头',q))
    p1.start()
    p2.start()

    c1 = Process(target=consumer,args=(q,'mengmeng'))
    c2 = Process(target=consumer,args=(q,'xiaoxiao'))
    c1.daemon = True # 将消费者设置为守护进程
    c2.daemon = True # 将消费者设置为守护进程
    c1.start()
    c2.start()

    # 让主程序可以等待生产子进程的结束.
    p1.join()
    p2.join()


    print('主程序结束..........')
```
个人不习惯使用JoinableQueue，为什么呢？因为他是通过生产者来“得知”，整个生产消费流程的终结.

在消费者调用q.task_done()时，会触发一次q.join()的检查(q.join()是用来阻塞进程的，最后一个任务完成后，q.task_done()＝》q.join()＝》阻塞解除)，之后生产者进程退出。而消费者呢？业务逻辑层面上是没有退出的（本例）。比如，本例中通过**设置为守护进程的方式进行退出**。但如果后续主进程还有其他任务，而没有退出呢？那么这些子进程则沦为僵尸进程，虽然对系统资源消耗很少(消费者的queue.get()也是阻塞的，所以不会执行循环，仅仅会“卡”在那里，但也不会自动消亡)，但感觉非常别扭的。所以个人还是倾向于用"生产者queue.put(None) ,消费者见到None则break(退出循环)"的传统方式 进行消费者进程触发退出。如果采用这种方式那么JoinableQueue相比Queue就没有优势了。 
## 一点思考
关于生产者和消费者，曾经思考过这么一种实现方式。

假如有一种队列，内置了**状态信息(存活生产者个数)**，设置目前存活的生产者个数

StatusableQueue(product_n=2,size=20)　#product_n=2含义：存活的生产者个数,size=20,队列长度

生产者：生产结束，q.product_n - 1(存活生产者个数-1)

消费者：存活生产者个数=0(生产者均已经完成生成) 且 队列长度=0(队列也已经消费结束)  则退出消费者进程.

这种情况下，只需要　消费者.join()　就可以保证整个生产消费进程的执行结束(这一点和JoinableQueue很像，不过JoinableQueue是 生产者.join())

一共只改动3处，就可以完成生产者消费者的并行化控制。 而且更符合逻辑，因为生产者是明确知道自己的退出条件的，而消费者依赖生产者，所以只需要观察消费者就可以知道（生成者是否结束）整个－生成消费链是否完成。


```

def consumer(q,name):
    while not (q.product_n==0 and q.size==0):# 存活生产者=0，意味着全部结束生产，队列不会新增数据,queue.size=0说明消费完毕
        food = q.get()
        print('\033[31m{}消费了{}\033[0m'.format(name,food))
        time.sleep(random.random())

def producer(name,food,q):
    for i in range(10):
        time.sleep(random.random())
        f = '{}生产了{}{}'.format(name,food,i)
        print(f)
        q.put(f)
    q.product_n -= 1 # 当生产者生产完毕之后,q.product_n - 1(存活生产者个数-1)



if __name__ == '__main__':
    q = StatusableQueue(product_n=2,size=20)#默认状态=正常,n=2含义：生产者个数,size=20,对列长度
    p1 = Process(target=producer,args=('fioman','包子',q))
    p2 = Process(target=producer,args=('jingjing','馒头',q))
    p1.start()
    p2.start()

    c1 = Process(target=consumer,args=(q,'mengmeng'))
    c2 = Process(target=consumer,args=(q,'xiaoxiao'))
    c1.start()
    c2.start()

    # 消费者消费结束（说明生产也一定结束了），则说明整个生产－消费逻辑完成
    c1.join()
    c2.join()
```
文中加注释地方为修改点，这样代码最简单，调用方面，含义清晰。

缺点：**必须知道生产者个数**，这个数据应该不难获取，毕竟后面在创建生产者时也需要使用这个变量控制。




## 参考

[Python的进程间通信](https://www.jianshu.com/p/acf67126d804)  