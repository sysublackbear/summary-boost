### Chapter11.函数与回调

#### 11.1.result_of

result_of 是一个很小但很有用的组件，可以帮助程序员确定一个调用表达式的返回类型，主要用于泛型编程和其他 Boost 库组件，它已被收入TR1。

result_of 位于名字空间boost，为了使用 result_of 组件，需要包含头文件<boost/utility/result_of.hpp>

##### 原理：

所谓“调用表达式”，是指一个含有 operator() 的表达式，函数调用或函数对象调用都可以称为调用表达式，而result_of 可以确定这个表达式所返回的类型。

单从这一点来理解，result_of 的功能有些类似 typeof 库，typeof 库可以确定一个表达式的类型，但它不具备推演调用表达式的能力。

```c++
#include <boost/utility/result_of.hpp>
using namespace boost;
int main()
{
  typeof double (*Func)(double d);
  Func func = sqrt;
  result_of<Func(double)>::type x = func(5.0);  // type被自动推导成double
  cout << typeid(x).name();
}
```



#### 11.2.bind

##### 绑定普通函数

```c++
#include <boost/bind.hpp>
using namespace boost;

cout << bind(f, 1, 2)() << endl;
cout << bind(g, 1, 2, 3)() << endl;

//相当于
cout << f(1,2) << endl;
cout << g(1,2,3) << endl;
```

##### 使用占位符

```c++
bind(f, _1, 9)(x);  // f(x, 9)，相当于bind2nd(f, 9)
bind(f, _1, _2)(x, y);  // f(x, y)
bind(f, _2, _1)(x, y);  // f(y, x)
bind(f, _1, _1)(x, y);  // f(x, x),y参数被忽略
bind(g, _1, 8, _2)(x,y);  // g(x,8,y)
bind(g, _3, _2, _2)(x, y, z);  // g(z, y, y);x参数被忽略，第一个参数是第三个占位符z，其余两个参数都是第二个占位符y
```

bind一般结合函数对象一起使用。



#### 11.3.signals2

signals2 基于 Boost 的另一个库 signals，实现了线程安全的观察者模式。在signals2 库中，观察者模式被称为信号/插槽，它是一种函数回调机制，一个信号关联了多个插槽，当信号发出时，所有关联它的插槽都会被调用。

使用方法如下：

```c++
#include <boost/signals2.hpp>
using namespace boost::signals2;

int main()
{
  signal<void()> sig;  // 一个信号对象
  sig.connect(&slots1);  // 链接插槽1
  sig.connect(&slots2);  // 链接插槽2
  sig();  // 调用operator()，产生信号（事件），触发插槽调用
}
```

输出结果：

slot1 called

slot2 called

明确传入 at_front 位置标志可以显式放到最前：

sig.connect(&slot2, at_front);



添加组号的signal 连接代码：

```c++
sig.connect(slots<1>(), at_back);  //最后被调用
sig.connect(slots<100>(), at_front);  //第一个被调用

sig.connect(5, slots<51>(), at_back);  //组号5，该组最后一个
sig.connect(5, slots<55>(), at_front);  //组号5,该组第一个
sig.connect(3, slots<30>(), at_front);  // 组号3，该组第一个
sig.connect(3, slots<33>(), at_back);  // 组号3，该组最后一个

sig.connect(10, slots<10>());  // 组号10，该组仅有一个
```



管理信号的连接

接下来的代码示范了插槽的连接与断开：

```c++
#include <boost/signals2.hpp>
using namespace boost::signals2;

int main()
{
  signal<int(int)> sig;
  assert(sig.empty());  //刚开始没有连接任何插槽
  
  sig.connect(0, slots<10>());  // 连接两个组号为0的插槽
  sig.connect(0, slots<20>());
  sig.connect(1, slots<30>());  // 连接组号为1的插槽
  
  assert(sig.num_slots() == 3);  // 目前有3个插槽
  sig.disconnect(0);  // 断开第0组插槽，共两个
  aseert(sig.num_slots() == 1);
  sig.disconnect(slots<30>());  //断开一个插槽
  assert(sig.empty());  // 信号不再连接任何插槽
  
  // 使用connection对象进行管理
  connection c1 = sig.connect(0, slots<10>());
  connection c2 = sig.connect(0, slots<20>());
  connection c3 = sig.connect(1, slots<30>());
  
  c1.disconnect();  // 断开第一个连接
  assert(sig.num_slots() == 2);  // sig现在连接两个插槽
  assert(!c1.connected());  // c1不再连接信号
  assert(c2.connected());  // c2仍然连接
  
  // 另一种连接管理对象是 scoped_connection，插槽与信号的连接仅在作用域内生效，当离开作用域连接就会自动断开。
  signal<int(int) > sig;
  
  sig.connect(0, slots<10>());
  assert(sig.num_slots() == 1);
  {  // 进入局部作用域，建立临时连接
    scoped_connection sc = sig.connect(0, slots<20>());
    assert(sig.num_slots() == 2);
  }  // 离开局部作用域，临时连接自动断开
}
```







