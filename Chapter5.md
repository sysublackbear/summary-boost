### Chapter5.字符串与文本处理

#### 5.1.lexical_cast

lexical_cast 库进行“字面量”的转换，可以进行字符串，整数或浮点数之间的字面转换。

```c++
#include <boost/lexical_cast.hpp>
using namespace boost;
int main()
{
  int x = lexical_cast<int>("100");  // 字符串->整数
  long y = lexical_cast<long>("2000");
  float pai = lexical_cast<float>("3.1415926e5");
  
  cout << lexical_cast<string>(456);
}
```

#### 5.2.format

format 字符串模板类，已经在业务代码有所使用：

```c++
std::string sCopyWriting = "你在%s的账号%s将于明天进行服务续期，请保持银行卡或零钱额度充足";
boost::format formater(sCopyWriting);
formater % m_poMchSimpleInfo->smchname() % m_sDisplayAccount;
// 忽略多了参数或者少了参数的异常
formater.exceptions(boost::io::all_error_bits ^ (boost::io::too_many_args_bit | boost::io::too_few_args_bit));
sCopyWriting = formater.str();
```

除了会抛异常（多了或少了参数，可以忽略），用起来挺方便的。



#### 5.3.string_algo

一个非常全面的字符串算法库。共分五大类：

+ 大小写转换；
+ 判断式与分类；
+ 修剪；
+ 查找与替换；
+ 分割与合并。

##### 1.大小写转换

to_upper_copy(str)  // 原字符串不改变

to_lower_copy(str)



##### 2.判断式

+ starts_with：判断是否前缀；
+ ends_with：判断是否后缀；
+ contains：判断是否包含；
+ equals：判断是否相等；
+ lexicographical_compare：根据字典顺序检测一个字符串是否小于另一个；
+ all：检测一个字符串中的所有元素是否满足指定的判断式。

除了all，其他算法都有个前缀 i 的版本，代表大小写无关字符串比较。



##### 3.判断式（函数对象）

+ is_equal：类似equals算法，比较两个对象是否相等；
+ is_less：比较两个对象是否具有小于关系；
+ is_not_greater：比较两个对象是否具有不大于关系；



##### 4.分类

+ is_space：字符是否空格；
+ is_alnum：字符是否为字母和数字字符；
+ is_alpha：字符是否为字母；
+ is_cntrl：字符是否为控制字符；
+ is_digit：字符是否为十进制数字；
+ is_graph：字符是否为图形字符；
+ is_lower：字符是否为小写字母；
+ is_print：字符是否为可打印字符；
+ is_punct：字符是否为标点符号字符；
+ is_upper：字符是否为大写字符；
+ is_xdigit：字符是否为十六进制字符；
+ is_any_of：字符是否参数字符序列中的任意字符；
+ is_from_range：字符是否位于指定区间内，即 from <= ch <= to。



##### 5.修剪

trim_left，trim_right 和 trim。



##### 6.替换与删除

+ replace/erase_first：替换/删除一个字符串在输入中的第一次出现；
+ replace/erase_last：替换/删除一个字符串在输入中的最后一次出现；
+ replace/erase_nth：替换/删除一个字符串在输入中的第n次（从0开始）出现；
+ replace/erase_all：替换/删除一个字符串在输入中的所有出现；
+ replace/erase_head：替换/删除输入的开头；
+ replace/erase_tail：替换/删除输入的末尾；



#### 5.4.tokenizer

tokenizer 库是一个专门用于分词的字符串处理库，可以使用简单易用的方法把一个字符串分解成若干个单词。

用法示例如下：（由于 tokenizer 具有类似容器的接口，所以可以使用 BOOST_AUTO）

```c++
#include <boost/tokenizer.hpp>
using namespace boost;
int main()
{
  std::string str("Link raise the master-sword.");
  
  tokenizer<> tok(str);
  
  // 可以像遍历一个容器一样使用 for 循环获得分词结果
  for (BOOST_AUTO(pos, tok.begin()); pos != tok.end(); ++pos)
  {
    cout << "[" << *pos ""]";
  }
}
```

此外，tokenizer 库也提供了定义好的四个分词对象：

+ char_delimiters_separator;
+ char_separtator;
+ escaped_list_separator;
+ offset_separtator。



#### 5.5.xpress

正则表达式，后面有需要再看吧。