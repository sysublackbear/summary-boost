### Chapter9.数学

#### 9.1.rational

boost.rational 库实现了有理数，补充了 C++的数字概念。它基于 C++内建整数类型，数字运算时没有精度损失，某种程度上相当于 Java 语言中的 BigDecimal，可以应用于金融财务等需要准确值的领域。

rational 位于名字空间 boost，为了使用 rational，需要包含头文件 <boost/rational.hpp>。



#### 9.2.crc

CRC（循环冗余校验码）是一种被广泛应用的错误验证机制，它使用一定的规则处理数据，计算出一个校验和，并附在数据末尾发送给接收方。接收方使用同样的规则计算 CRC，如果两个计算结果一致则说明传输正确，否则表明传输过程发生了错误。

boost.crc 库实现了计算 CRC的功能。

```c++
#include <boost/crc.hpp>
using namespace boost;
```



#### 9.3.伪随机数发生器

random库位于名字空间 boost，为了使用 random 库，需要包含头文件 <boost/random.hpp>

random库提供了26个伪随机数发生器，使用的算法包括线性同余算法，逆同余算法，Mersenne Twister 算法，fibonacci 算法，ranlux算法以及它们的混合。这些随机数发生器的随机数质量，内存需求，产生速度都各不相同，程序员可以仔细评估以选择最合适自己应用的一个。





