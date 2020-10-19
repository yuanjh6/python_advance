# python进阶01偏函数
定义：偏函数的第二个部分(可变参数)，按原有函数的参数顺序进行补充，参数将作用在原函数上，最后偏函数返回一个新函数（类似于，装饰器decorator，对于函数进行二次包装，产生特殊效果；但又不同于装饰器，偏函数产生了一个新函数，而装饰器，可改变被装饰函数的函数入口地址也可以不影响原函数）


效果：固定一部分参数，在后续调用时只需传递少量参数即可


个人倾向于按照重构函数行为来理解，比如需要3个函数，一个是x的平反，一个是x的3次方，一个是x的四次方，那么一个函数将2,3,4当做参数穿进去 ，生成一个新函数。这样的话可以把原函数看做函数集合，偏函数才是真正使用的函数具体对象。


举例01：


```
from functools import partial
 
def mod( n, m ):
  return n % m
 
mod_by_100 = partial( mod, 100 )
 
print mod( 100, 7 )  # 2
print mod_by_100( 7 )  # 2
```

举例02：

```
from functools import partial
 
bin2dec = partial( int, base=2 )
print bin2dec( '0b10001' )  # 17
print bin2dec( '10001' )  # 17
 
hex2dec = partial( int, base=16 )
print hex2dec( '0x67' )  # 103
print hex2dec( '67' )  # 103
```