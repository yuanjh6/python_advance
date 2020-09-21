# python进阶26代码洁癖
## 缘由
先说垃圾代码：表面是代码垃圾，实际是逻辑垃圾，思路不清或不够简单。  
简单代码需要更深入思考，建立更简单，简洁，明了的思路。python具有很高的灵活性，所以这一点尤其重要。  

## 类型注解
大型程序中非常必要，弥补python弱类型在开发中的沟通成本（接口调用方需阅读接口实现或代码注释才知传参类型）。  
```
from typing import List, Sequence
 
def square(elems: Sequence[float]) -> List[float]:
    return [x**2 for x in elems]
```

## 列表生成式替代循环([i for xx])
大多数循环都可以用列表生成式替代，列表生成式另一个好处是很方便的就可以实现并行化.  

## 拍平嵌套的多重循环(product)笛卡尔积
itertools.product()  
```
In [68]: list(product('ABCD', 'xy'))
Out[68]:
[('A', 'x'),
 ('A', 'y'),
 ('B', 'x'),
 ('B', 'y'),
 ('C', 'x'),
 ('C', 'y'),
 ('D', 'x'),
 ('D', 'y')]

```
## _替代无用临时变量i
其他语言对于无意义的循环个数常用临时变量i,j等表示，python中推荐使用\_(开源代码大多此规则，认为是一种约定习惯)   
```
[x for x in range(3) for _ in range(2)]
[0,0,1,1,2,2]
```

## 批量化一致性操作优于依次ifelse判断
循环内部做if-else劣于先批量做处理再filter过滤。（不绝对）  
对于统一性的批量处理，numpy等本身会做一定优化，可以通过Numba进行进一步优化。而且一致性的批量处理符合cpu的cache的LRU规则，大概率命中cache。所以不建议内部有if-else这样的判断逻辑（如果不同分支时间差异很大，另当别论）  
简单来说就是操作集中化，每个步骤都做简单的，多步骤组合。优于一个大循环内做不同的分支处理逻辑。  

猜测如下代码那个速度最快   
```
import time
a = time.time()
b, c = list(), list()
for i in range(10000):
    if not i % 7:
        b.append(i)
    else:
        c.append(i)
print(time.time() - a)

a = time.time()
b = list()
c = list()
[b.append(i) if not i % 7 else c.append(i) for i in range(10000)]
print(time.time() - a)

a = time.time()
b = [i for i in range(10000) if not i % 7]
c = [i for i in range(10000) if i % 7]
print(time.time() - a)
```
结论:最慢，中间，最快  
比较奇怪的是第一个和第二个，为何第二个比第一个快，测试多次都是同样结果，本人也不大理解！！  


## 用a and b(a or b)替代if a:b(a if a is not none else b)的简单表达式
副作用mypy静态类型检查会报错(在b没有返回值时)  

