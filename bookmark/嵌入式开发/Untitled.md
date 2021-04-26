当我们需要定义常量时，一个办法是用大写变量通过整数来定义，例如月份：

```
JAN = 1
FEB = 2
MAR = 3
...
NOV = 11
DEC = 12
```

好处是简单，缺点是类型是`int`，并且仍然是变量。

更好的方法是为这样的枚举类型定义一个class类型，然后，每个常量都是class的一个唯一实例。Python提供了`Enum`类来实现这个功能：

```
from enum import Enum

Month = Enum('Month', ('Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'))
```

这样我们就获得了`Month`类型的枚举类，可以直接使用`Month.Jan`来引用一个常量，或者枚举它的所有成员：

```
for name, member in Month.__members__.items():
    print(name, '=>', member, ',', member.value)
```

`value`属性则是自动赋给成员的`int`常量，默认从`1`开始计数。

如果需要更精确地控制枚举类型，可以从`Enum`派生出自定义类：

```
from enum import Enum, unique

@unique
class Weekday(Enum):
    Sun = 0 # Sun的value被设定为0
    Mon = 1
    Tue = 2
    Wed = 3
    Thu = 4
    Fri = 5
    Sat = 6
```

`@unique`装饰器可以帮助我们检查保证没有重复值。



**一般来说，带宽分配的策略可以按照下面的方法进行：**

- 1）总共的带宽由码率自适应 ABC 模块估算得出；
- 2）丢包重传 ARQ 的重传数据包所占带宽根据 RTT 和 PLR 估算得出；
- 3）前向纠错 FEC 的校验数据包所占带宽根据 RTT，ARQ 恢复后的 PLR，和总共的带宽估算得出；
- 4）原始数据包所占的带宽根据 ARQ、FEC 和总共的带宽计算得出。

下面是一个例子，展示随着 RTT 和 PLR 的增加，如何在原始数据包、ARQ 和 FEC 之间分配带宽。

![如何优化传输机制来实现实时音视频的超低延迟？_7.png](_assets/Untitled/161417hzoikpaitotiih8i.png)

 

