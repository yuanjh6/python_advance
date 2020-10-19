# python进阶08并发之四map, apply, map_async, apply_async差异
## 差异矩阵
python封装了4种常用方法，用于实现并发

其差异如下


|             | Multi-args | Concurrence | Blocking | Ordered-results |
| ----------- | ---------- | ----------- | -------- | -------------- |
| map         | no         | yes         | yes      | yes             |
| apply       | yes        | no          | yes      | no              |
| map_async   | no         | yes         | no       | yes             |
| apply_async | yes        | yes         | no       | no              |

需要注意：map 和 map_async 入参为迭代器类型，可以批量调用。而apply和apply_async只能一个个调用。

```
# map
results = pool.map(worker, [1, 2, 3])

# apply
for x, y in [[1, 1], [2, 2]]:
    results.append(pool.apply(worker, (x, y)))

def collect_result(result):
    results.append(result)

# map_async
pool.map_async(worker, jobs, callback=collect_result)

# apply_async
for x, y in [[1, 1], [2, 2]]:
    pool.apply_async(worker, (x, y), callback=collect_result)
```
## apply和apply_async
Pool.apply_async：调用立即返回而不是等待结果。AsyncResult返回一个对象。你调用其get()方法以检索函数调用的结果。该get()方法将阻塞直到功能完成。

因此，pool.apply(func, args, kwargs)等效于pool.apply_async(func, args, kwargs).get()。

相比Pool.apply，该Pool.apply_async方法还具有一个回调，则在函数完成时调用该回调。可以使用它来代替get()。


```
import multiprocessing as mp
import time

def foo_pool(x):
    time.sleep(2)
    return x*x

result_list = []
def log_result(result):
    # This is called whenever foo_pool(i) returns a result.
    # result_list is modified only by the main process, not the pool workers.
    result_list.append(result)

def apply_async_with_callback():
    pool = mp.Pool()
    for i in range(10):
        pool.apply_async(foo_pool, args = (i, ), callback = log_result)
    pool.close()
    pool.join()
    print(result_list)

if __name__ == '__main__':
    apply_async_with_callback()
可能会产生如下结果

[1, 0, 4, 9, 25, 16, 49, 36, 81, 64]
```
还要注意，可使用调用许多不同的函数Pool.apply_async（并非所有调用都需要使用同一函数）。

相反，Pool.map将相同的函数应用于许多参数。但是，与不同Pool.apply_async，返回结果的顺序与参数的顺序相对应。


## 参考

[Python multiprocessing.Pool: Difference between map, apply, map_async, apply_async](http://blog.shenwei.me/python-multiprocessing-pool-difference-between-map-apply-map_async-apply_async/)

[Python-multiprocessing.Pool：何时使用apply，apply_async或map？](http://codingdict.com/questions/1325)

