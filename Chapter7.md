#### 7.1.array

array 的模板参数指明了 array 的元素类型和大小，array<int, 5>相当于声明了一个普通数组 int a[5]。

array 包装了普通数组，并提供类似STL容器的接口：

+ begin() 和 end() 函数可以返回容器区间的开始和结束迭代器，相当于数组的首指针和末指针+1；
+ front() 和 back() 返回容器首末元素的引用，相当于 elems[0] 和 elems[N-1]；
+ size() 和 max_size() 返回容器的大小，因为是静态数组，故两者返回值相同，都是模版参赛N；
+ empty() 判断容器是否为空，因为是静态数组，该函数总会返回false；
+ operator=使用 std::copy 算法实现了赋值操作；
+ swap() 使用 boost::swap 可以高效地交换两个 array 对象；
+ assign() 使用标准算法 std::fill_n 将容器内所有元素赋值为 value；

此外，array 还重载了 ==, <, > 等比较操作符，使用字典序比较两个 array 对象。



#### 7.2.dynamic_bitset

C++98 标准为处理二进制数值提供了两个工具：vector<bool>和bitset。

vector<bool>是对元素类型为bool 的 vector 特化，它内部并不真正存储 bool 值，而是以 bit 来压缩保存，使用代理技术来操作 bit，造成的后果就是它很像容器，大多数情况下的行为与标准容器一致，但它不是容器，不满足容器的定义。

bitset与vector<bool>类似，同样存储二进制位，但它的大小固定，而且比 vector<bool> 支持更多的位运算。

vector<bool>和bitset各有优缺点：vector<bool>可以动态增长，但不能方便地进行位运算，bitset 则正好相反，可以方便地对容纳的二进制做位运算，但不能动态增长。

boost.dynamic_bitset 的出现恰好填补了这两者之间的空白，它类似标准库的 bitset，提供丰富的位运算，同时长度又是动态可变的。



#### 7.3.bimap

C++标准提供了映射型容器 map 和 multi_map，它们就像是一个关联数组，把一个元素(key)映射到另一个元素(value)，但这种映射关系是单向的，只能是key到value，而不能反过来。

boost.bimap 扩展了标准库的映射型容器，提供双向映射的能力，功能强大，其接口被特意设计为符合 STL 规范，以减少学习的负担。

使用方法：

```c++
#include <boost/bimap.hpp>
using namespace boost;

int main()
{
  bimap<int, string> bm;
  
  // 使用左视图 map<int, string>
  bm.left.insert(make_pair(1, "111"));  // 插入数据
  bm.left.insert(make_pair(2, "222"));
  
  // 使用右视图 map<string, int>
  bm.right.insert(make_pair("string", 10));  // 插入数据
  bm.right.insert(make_pair("bimap", 20));
  
  // 对左视图使用迭代器迭代
  for (BOOST_AUTO(pos, bm.left.begin()); pos != bm.left.end(); ++pos)
  {
    cout << "left[" << pos->first << "]=" << pos->second << endl;
  }
  
  bm.left.insert(make_pair(3, "3333"));  // 左视图插入数据
  bm.left.insert(make_pair(3, "4444"));  // 无效插入操作
  bm.right.insert(make_pair("string", 3));  // 无效插入操作
}
```



#### 7.4.circular_buffer

circular_buffer 实现了循环缓冲区的数据结构，支持标准的容器操作（如push_back）但大小是固定的，当到达容器末尾时将自动循环利用容器另一端的空间。

用法如下：

```c++
circular_buffer<int> cb(5);  // 声明一个大小为5的循环缓冲区
assert(cb.empty());  // 缓冲区目前无数据

cb.push_back(1);  // 向后端添加元素1
cb.push_front(2);  // 向前端添加元素2
assert(cb.front() == 2);
cb.insert(cb.begin(), 3);  // 向前端添加元素3

// 可以使用迭代器遍历容器
for (BOOST_AUTO(pos, cb.begin()); pos != cb.end(); ++pos)
{
  cout << *pos << ",";
}
cout << endl;

cb.pop_front();  // 弹出首元素3
assert(cb.size() == 2);
cb.push_back();  // 弹出末元素1
assert(cb[0] == 2);
```



#### 7.5.tuple

tuple（元组）定义了一个有固定数目元素的容器，其中的每个元素类型都可以不相同，这与其他容器有着本质的区别。访问元素使用这种方式：assert(t.get<0>() == 1); tuple在python已经很熟悉，也讲了很多，就不再补充了，需要用到的时候查询下api文档。



#### 7.6.any

any 是一种很特殊的容器，它只能容纳一个元素，但这个元素可以是任意的类型——int，double，string，STL 容器或者任何的自定义类型。程序可以用 any 保存任意的数据，在任何需要的时候将它取出。这种功能与 shared_ptr<void> 有些类似，但 any 是类型安全的。



#### 7.7.variant

variant 与 any 有些类似，是一种可变类型，是对C/C++中union概念的增强和扩展。普通的 union 只能持有 POD（普通数据类型），而不能持有如 string，vector 等复杂类型，variant 则没有这个限制。

variant 位于名字空间 boost，为了使用 variant 组件，需要包含头文件 <boost/variant.hpp>，即：

```c++
#include <boost/variant.hpp>
using namespace boost;
```









