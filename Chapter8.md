### Chapter8.算法

#### 8.1.foreach

foreach库提供了两个宏 BOOST_FOREACH 和 BOOST_REVERSE_FOREACH，分别来实现对序列的正向遍历和反向遍历，使用起来非常简单方便，用法如下：

```c++
#include <boost/foreach.hpp>
#include <boost/assign.hpp>
#define foreach BOOST_FOREACH
#define reverse_foreach BOOST_REVERSE_FOREACH

int main()
{
  using namespace boost::assign;
  
  //遍历普通数组
  int ar[] = {1,2,3,4,5};
  foreach(int& x, ar)
  {
    cout << x << " ";
  }
  cout << endl;
  
  // 遍历map
  map<int, string> m = map_list_of(1, "111")(2, "222")(3, "333");
  
  pair<int, string> x;
  foreach(x, m)
  {
    cout << x.first << x.second << endl;
  }
  
  // 遍历用vector实现的二维数组
  vector< vector<int> > v = list_of( list_of(1)(2))(list_of(3)(4));
  foreach(vector<int> & row, v)
  {
    reverse_foreach(int & z, row)
    {
      cout << z << ",";
    }
    cout << endl;
  }
}
```

