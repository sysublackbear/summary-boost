### Chapter4.实用工具

#### 4.1.noncopyable

noncopyable 允许程序轻松地实现一个禁止复制的类。位于文件 <boost/noncopyable.hpp>。

```c++
#include <boost/utility.hpp>
class do_not_copy: boost::noncopyable
{ ... };
```

#### 4.2.optional

optional 类使用了容器的语义，包装了“可能产生无效值”的对象，实现了“未初始化”的概念。

```c++
#include <boost/optional.hpp>
using namespace boost;
```

#### 4.3.swap

boost::swap 是对标准库提供的 std::swap的增强和泛化，为交换两个变量（可以是 int等内置数据类型，或是类实例，容器）的值提供了便捷的方法。

使用 boost::swap，需要包含头文件 <boost/swap.hpp> 。

#### 4.4.tribool

boost.tribool 类似C++内置的布尔类型，但基于三态的布尔逻辑：在 true（真）和 false（假）之外还有一个 indeterminate 状态（未知，不确定）。

```c++
#include <boost/logic/tribool.hpp>
using namespace boost;
```

#### 4.5.uuid

uuid库是一个小的实用工具，可以表示和生成UUID。

用法：

```c++
#include <boost/uuid/uuid.hpp>
#include <boost/uuid/uuid_generators.hpp>
#include <boost/uuid/uuid_io.hpp>
using namespace boost::uuids;

uuid u;
```

#### 4.6.生成器

##### Nil 生成器

Nil生成器是最简单的 UUID 生成器，只能生成一个无效的UUID值（即全零的 UUID），它的存在只是为了方便地表示无效UUID。

```c++
uuid u = nil_generator()();
assert(u.is_nil());

u = nil_uuid();
assert(u.is_nil());
```

##### 字符串生成器

从一个字符串创建出 uuid 对象。

```c++
string_generator sgen;
uuid u1 = sgen("0123456789abc");
cout << u1 << endl;
```

##### 名字生成器

使用基于名字的 SHA1 摘要算法。相同名字产生相同的 UUID。

```c++
uuid www_xxx_com = string_generator()("{0123456789abcdef}");
name_generator ngen(www_xxx_com);  //构造名字生成器

uuid u1 = ngen("mario");
```

##### 随机生成器

```c++
random_generator rgen;
uuid u = rgen();
```

#### 4.7.uuid与string之间的相互转化

```c++
#include <boost/lexical_cast.hpp>
using namespace boost;
using namespace boost::uuids;

int main()
{
  uuid u = lexical_cast<uuid>("01234567-89ab-edf");  //字符串转换到 uuid
  cout << u << endl;
  std::string str = lexical_cast<string>(u);  // uuid 转换到字符串
  cout << str << endl;
}
```

#### 4.8.SHA1 摘要算法

```c++
#include <boost/uuid/sha1.hpp>
using namespace boost::uuids::detail;

int main()
{
  sha1 sha;  // sha1对象
  
  char *szMsg = "a short message";  // 用于摘要的消息
  sha.process_byte(0x10);  // 处理一个字节
  sha.process_bytes(szMsg, strlen(szMsg));  // 处理多个字节
  sha.process_block(szMsg, szMsg + strlen(szMsg));
  
  unsigned int digest[5];
  sha.get_digest(digest);
  for (int i = 0; i < 5; ++i)
  {
    cout << hex << digest[i];  // 16进制输出
  }
}
```

