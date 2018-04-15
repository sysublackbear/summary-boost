### Chpater3.内存管理

#### 3.1.scoped_ptr 与 auto_ptr的区别

scoped_ptr 与 auto_ptr 的根本性区别在于指针的所有权。

```c++
auto_ptr<int> ap(new int(10));  // 一个int自动指针
scoped_ptr<int> sp(ap);  // 从auto_ptr获得原始指针
assert(ap.get() == 0);  // 原auto_ptr 不再拥有指针

ap.reset(new int(20));  // auto_ptr 拥有新的指针
cout << *ap << "," << *sp << endl;

auto_ptr<int> ap2;  
ap2 = ap;  // ap2从ap获得原始指针，发生所有权转移
assert(ap.get() == 0);  // ap不再拥有指针
scoped_str<int> sp2;  // 另一个scoped_str
sp2 = sp;  // 赋值操作，无法通过编译！
```



#### 3.2.scoped_array

scoped_array 很像 scoped_ptr，它包装了 new[] 操作符（不是单纯的 new) 在堆上分配的动态数组，为动态数组提供了一个代理，保证可以正确地使用内存。

scoped_array 弥补了标准库中没有指向数组的智能指针的缺憾。但它的功能很有限，不能动态增长，也没有迭代器支持，不能搭配STL算法。动态空间还是推荐使用std::vector



#### 3.3.shared_ptr

shared_ptr 与 scoped_ptr 一样包装了 new 操作符在堆上分配的对象，但它实现的是引用计数型的智能指针，可以自由被拷贝和赋值。

支持工厂方法(make_shared)，防止多次使用 new 操作符

```c++
#include <boost/make_shared.hpp>
int main()
{
  shared_ptr<string> sp = make_shared<string>("make_shared");  // 创建 string 的共享指针
  shared_ptr<vector<int> > spv = make_shared<vector<int> >(10, 2);  // 创建 vector 的共享指针。
}
```



#### 3.4.shared_ptr 应用于桥接模式

桥接模式是一种结构型的设计模式，它把类的具体实现细节对用户隐藏起来，以达到类之间的最小耦合关系。在具体编程实践中桥接模式也被称为“pimpl" 或者 handle/body 惯用法，它可以将头文件的依赖关系降到最小，减少编译时间，并且可以不使用虚函数实现多态。

scoped_ptr 和 shared_ptr 都可以用来实现桥接模式，但 shared_ptr 通常更合适，因为它支持拷贝和赋值，这在很多情况下都是有用的，比如可以配合容器工作。

首先我们声明一个类 sample，它仅向外界暴露了最小细节，真正的实现在内部类 impl，sample 用一个 shared_ptr 来保存它的指针：

```c++
class sample
{
  private:
  	class impl;  // 不完整的内部类声明
  	shared_ptr<impl> p;  // shared_ptr 成员变量
  public:
  	sample();
  	void print();
};
```

在 sample 的 cpp 中完整定义 impl 类和其他功能：

```c++
class sample::impl  // 内部类的实现
{
  public:
  	void print()
    {
      cout << "impl print" << endl;
    }
};

sample::sample():p(new impl) {}  // 构造函数初始化 shared_ptr
void sample::print()
{
  p->print();
}
```

最后是桥接模式的使用，很简单：

```c++
sample s;
s.print();
```



#### 3.5.shared_ptr 应用于工厂模式

使用 shared_ptr ，只需要修改工厂方法的接口，不再返回一个原始指针，而是返回一个被 shared_ptr 包装的智能指针，这样可以很好地保护系统资源，而且会更好地控制对接口的使用。

```c++
class abstract  // 接口类定义
{
  public:
  	virtual void f() = 0;
  	virtual void g() = 0;
  protected:
  	virtual ~abstract(){}
}
```

注意这里 abstract 的析构函数，被定义为保护的，意味着除了它自己和它的子类，其他任何对象都无权调用 delete 来删除它。

然后看下 abstract 的实现子类

```c++
class impl:public abstract
{
	public:
		virtual void f() { cout << "class impl f" << endl; }
		virtual void g() { cout << "class impl g" << endl; }
};
```

随后的工厂函数返回基类的 shared_ptr

```c++
shared_ptr<abstract> create()
{
  return shared_ptr<abstract>(new impl);
}
```

这样我们就完成了全部工厂模式的实现，现在可以把这些组合起来：

```c++
int main()
{
  shared_ptr<abstract> p = create();  // 工厂函数创建对象
  p->f();
  p->g();  // 不用担心资源泄露
  
  abstract * q = p.get();  // 正确
  delete q;  // 错误,析构函数为protected 方法。无法通过编译
}
```

shared_ptr还支持定制删除器，在构造函数多传一个参数。

#### 3.6.shared_array

shared_array 类似 shared_ptr，它包装了 new[] 操作符在堆上分配的动态数组，同样使用引用计数机制为动态数组提供了一个代理，可以在程序的生命周期里长期存在，直到没有任何引用后才释放内存。

####3.7.weak_ptr

weak_ptr 是为配合 shared_ptr 而引入的一种智能指针，它更像是 shared_ptr 的一个助手而不是智能指针，因为它不具有普通指针的行为，没有重载 operator* 和 ->，它的最大作用在于协助 shared_ptr 工作，像旁观者那样观测资源的使用情况。

weak_ptr本身不占用引用计数，只用于观测，用法如下：

```c++
shared_ptr<int) sp(new int(10));  // 一个shared_ptr
assert(sp.use_count() == 1);

weak_ptr<int> wp(sp);  // 从shared_ptr创建weak_ptr
assert(wp.use_count() == 1);  // weak_ptr不影响引用计数

if (!wp.expired())  // 判断 weak_ptr 观察的对象是否失效
{
  shared_ptr<int> sp2 = wp.lock();  // 获得一个shared_ptr
  *sp2 = 100;
  assert(wp.use_count() == 2);
}

assert(wp.use_count() == 1);
```



#### 3.8.内存池

boost 也提供了各种 pool的使用方法，详细可以看文档，pool有以下几种：

pool：简单数据类型；

object_pool：类实例（对象）；（重要）

singleton_pool：简单数据类型的内存指针，但它是一个单件，并提供线程安全。

