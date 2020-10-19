# python进阶20之actor
actor模型。actor模式是一种最古老的也是最简单的并行和分布式计算解决方案。

优点：充分利用单线程＋事件机制，达到了多线程效果。

缺点，对python而言，由于GIL的存在，毕竟只是单线程，难以匹敌多进程，目前使用并不多。

## 简单任务调度器

```
class TaskScheduler:
    def __init__(self):
        self._task_queue = deque()

    def new_task(self, task):
        '''
        Admit a newly started task to the scheduler
        '''
        self._task_queue.append(task)

    def run(self):
        '''
        Run until there are no more tasks
        '''
        while self._task_queue:
            task = self._task_queue.popleft()
            try:
                # Run until the next yield statement
                next(task)
                self._task_queue.append(task)
            except StopIteration:
                # Generator is no longer executing
                pass

# Example use
sched = TaskScheduler()
sched.new_task(countdown(10))
sched.new_task(countdown(5))
sched.new_task(countup(15))
sched.run()
```

## 协程生产者消费者
廖雪峰的python官网教程里面的协程生产者消费者

```
def consumer():
    r = ''
    while True:
        n = yield r
        if not n:
            return
        print('[CONSUMER] Consuming %s...' % n)
        r = '200 OK'
 
def produce(c):
    c.send(None)
    n = 0
    while n < 5:
        n = n + 1
        print('[PRODUCER] Producing %s...' % n)
        r = c.send(n)
        print('[PRODUCER] Consumer return: %s' % r)
    c.close()
 
c = consumer()
produce(c)
```

## 并发网络应用程序
演示了使用生成器来实现一个并发网络应用程序：

```
class ActorScheduler:
    def __init__(self):
        self._actors = {}
        self._msg_queue = deque()
 
    def new_actor(self, name, actor):
        self._msg_queue.append((actor, None))
        self._actors[name] = actor
 
    def send(self, name, msg):
        actor = self._actors.get(name)
        if actor:
            self._msg_queue.append((actor, msg))
 
    def run(self):
        while self._msg_queue:
            # print("队列：", self._msg_queue)
            actor, msg = self._msg_queue.popleft()
            # print("actor", actor)
            # print("msg", msg)
            try:
                 actor.send(msg)
            except StopIteration:
                 pass
 
if __name__ == '__main__':
    def say_hello():
        while True:
            msg = yield
            print("say hello", msg)
 
    def say_hi():
        while True:
            msg = yield
            print("say hi", msg)
 
    def counter(sched):
        while True:
            n = yield
            print("counter:", n)
            if n == 0:
                break
            sched.send('say_hello', n)
            sched.send('say_hi', n)
            sched.send('counter', n-1)
 
    sched = ActorScheduler()
    # 创建初始化 actors
    sched.new_actor('say_hello', say_hello())
    sched.new_actor('say_hi', say_hi())
    sched.new_actor('counter', counter(sched))
 
    sched.send('counter', 10)
    sched.run()
```


## 参考
扩展Python Gevent的Actor模型：https://www.dazhuanlan.com/2020/02/29/5e5a7f241ed15/

终结python协程----从yield到actor模型的实现：https://www.bbsmax.com/A/n2d9bQaYzD/

12.12 使用生成器代替线程：https://python3-cookbook.readthedocs.io/zh_CN/latest/c12/p12_using_generators_as_alternative_to_threads.html

