### Chapter2.时间与日期

#### 2.1.timer

```c++
#include <boost/timer.hpp>
using namespace boost;
timer t;
t.elapsed_max();  // timer能够测量的最大时间范围
t.elapsed_min();  // timer测量时间的最小精度
t.elapsed();  // timer对象一旦被声明，它的构造函数就启动了计时工作。之后随时使用elapsed()函数简单地测量自对象创建后所流逝的时间。
```

#### 2.2.progress_timer

progress_timer 继承于 timer，会在析构时自动输出时间。如果要在一个程序中测量多个时间，可以运用花括号{}以限定 progress_timer 的生命周期

```c++
#include <boost/progress.hpp>
using namespace std;
boost::progress_timer t;
cout << t.elapsed() << end;
{
  boost::progress_timer t;
  // do something
}

class progress_timer : public timer, noncopyable  // noncopyable是boost库里面的一个单例基类，继承它能够轻松实现单例
{
  public:
  	explicit progress_timer();
  	progress_timer( std::ostream & os);
  	~progress_timer();
};

// 看到上面的构造函数，因此可以这样使用
stringstream ss;
{
  progress_timer t(ss);
}
cout << ss.str() << endl;
```

#### 2.3.progress_display

快速在控制台显示程序执行的进度条，例子如下：

```c++
#include <boost/progress.hpp>
using namespace boost;

int main()
{
  std::vector<std::string> v(100);
  std::ofstream fs('/tmp/file');
  
  // 声明progress_display对象
  progress_display pd(v.size());
  
  std::vector<std::string>::iterator itr;
  for (itr = v.begin(); itr != v.end(); itr++)
  {
    fs << *itr << std::endl;
    ++pd;
  }
}
```



#### 2.4.date_time

date_time库比较多，具体还是查文档。



