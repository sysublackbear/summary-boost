### Chapter12.并发编程

#### 12.1.互斥量

mutex 的基本用法如下：

```c++
mutex mu;   // 声明一个互斥量对象
try
{
  mu.lock();  // 锁定互斥量
  cout << "some operations" << endl;  // 临界区操作
  mu.unlock();  // 解锁互斥量
}
catch (...)  //必须使用try-catch块保证解锁互斥量
{
  mu.unlock();
}
```

mutex 类使用内部类型定义 scoped_lock 和 scoped_try_lock 定义了两种 lock_guard 对象，分别对应执行 lock() 和 try_lock()。

```c++
mutex mu;
mutex::scoped_lock lock(mu);   // 使用RAII型的 lock_guard
cout << "some operations" << endl;
```



#### 12.2.asio

##### 同步定时器

```c++
#include <boost/asio.hpp>
#include <boost/date_time/posix_time/posix_time.hpp>
using namespace boost::asio;  // 打开asio名字空间
int main()
{
  io_service ios;  //所有 asio 程序必须要有一个io_service对象
  deadline_timer t(ios, posix_time::seconds(2));  //定时器，io_service作为构造函数的参数，两秒钟后定时器终止
  cout << t.expires_at() << endl;  // 查看终止的绝对时间
  t.wait();  // 调用wait()同步等待
  cout << "hello asio" << endl;  // 输出一条消息
}
```

##### 异步定时器

```c++
#include <boost/asio.hpp>
#include <boost/date_time/posix_time/posix_time.hpp>
using namespace boost::asio;  // 打开asio名字空间
int main()
{
  io_service ios;  // io_service对象
  deadline_timer t(ios, posix_time::second(2));  // 定时器
  t.async_wait(print);  // 异步等待，传入回调函数
  // 立即返回
  cout << "it show before t expired." << endl;
}
```

当然，异步定时器也可以使用bind函数，适配接口做更多的事情。

```c++
t.async_wait(bind(&a_timer::call_func, this, placeholders::error));  // 注册回调函数
```



#### 12.3.IP地址和端口

下面的代码示范了 address 类的用法：

```c++
ip::address addr;  // 声明一个ip地址对象
addr = addr.from_string("127.0.0.1");  // 从字符串产生ip地址
assert(addr.is_v4());  // ipv4的地址
cout << addr.to_string() << endl;  // 转换字符串输出
addr = addr.from_string("ab::12::34::56");  // ipv6的地址字符串
assert(addr.is_v6());
```

有了ip地址，加上通信用的端口号就构成了一个 socket 端点。

```c++
ip::address addr;
addr = addr.from_string("127.0.0.1");  // 一个ipv4的地址
ip::tcp::endpoint ep(addr, 6688);  // 创建端口对象，端口为6688
assert(ep.address() == addr）;
assert(ep.port() == 6688);
```



#### 12.4.同步socket处理

##### 服务器端：

```c++
int main()
{
  try  // function-try块
  {
    cout << "server start." << endl;
    io_service ios;  // asio程序必需的io_service对象
    
    ip::tcp::acceptor acceptor(ios, ip::tcp::endpoint(ip::tcp::v4(), 6688));  // 接受6688端口
    cout << acceptor.local_endpoint().address() << endl;
    
    while(true)  // 循环执行服务
    {
      ip::tcp::socket sock(ios);  // 一个socket对象
      acceptor.accept(sock);  // 阻塞等待socket连接
      
      cout << "client:";
      cout << sock.remote_endpoint().address() << endl;
      sock.write_some(buffer("hello asio"));  // 发送数据
    }
  }
  catch (std::exception & e)  // 捕获可能发生的异常
  {
    cout << e.what() << endl;
  }
}
```

##### 客户端：

```c++
void client(io_service &ios)  // 传入 io_service 对象
{
  try
  {
    cout << "client start." << endl;
    ip::tcp::socket sock(ios);  // 创建socket对象
    ip::tcp::endpoint ep(ip::address::from_string("127.0.0.1"), 6688);  // 创建连接端点
    
    sock.connect(ep);  // socket连接到端点
    vector<char> str(100, 0);  // 定义一个vector缓冲区
    sock.read_some(buffer(str));  // 使用buffer()包装缓冲区接收数据
    cout << "receive from " << sock.remote_endpoint().address();
    cout << &str[0] << endl; // 输出接收到的字符串
  }
  catch (std::exception & e)
  {
    cout << e.what() << endl;
  }
}

int main()
{
  io_service ios;
  a_timer at(ios, 5, bind(client, ref(ios)));  // 启动定时器
  ios.run();
}
```



#### 12.5.异步socket处理

##### 服务器端：

```c++
class server
{
  private:
  	io_service &ios;
  	ip::tcp::acceptor acceptor;
  	typedef shared_ptr<ip::tcp::socket> sock_ptr;
  
  public:
  	server(io_serve & io): ios(io), acceptor(ios, ip::tcp::endpoint(ip::tcp::v4(), 6688))
    {
     	start(); 
    }
}

void start()
{
  sock_pt sock(new ip::tcp::socket(ios));  // 智能指针
  
  acceptor.async_accept(*sock, bind(&server::accept_handler, this, placeholders::error, sock));
  // 异步侦听服务
}

void accept_handler(const system::error_code & ec, sock_pt sock)
{
  if (ec)
  {
    return ;
  }
  
  cout << "client:";  // 输出连接的客户端信息
  cout << sock->remote_endpoint().address() << endl;
  sock->async_write_some(buffer("hello asio")),
  	bind(&server::write_handler, this, placeholders::error));
  start();  // 再次启动异步接受连接
}

void write_handler(const system::error_code&)
{
  cout << "send msg complete." << endl;
}

int main()
{
  try
  {
    cout << "server start." << endl;
    io_service ios;  // io_service对象
    
    server serv(ios);  // 构造server对象
    ios.run();
  }
  catch (std::exception& e)
  {
    cout << e.what() << endl;
  }
}
```



##### 客户端

```c++
class client
{
  private:
  	io_service & ios;
  	ip::tcp::endpoint ep;
  	typedef shared_ptr<ip::tcp::socket> sock_ptr;
  public:
  	client(io_service& io):ios(io), ep(ip::address::from_string("127.0.0.1"), 6688)
    {
      start();
    }
}

void start()
{
  sock_pt sock(new ip::tcp::socket(ios));  // 创建socket对象
  sock->async_connect(ep, bind(&client::conn_handler, this, placeholders::error, sock));
}

void conn_handler(const system::error_code & ec, sock_pt sock)
{
  if (ec)
  {
    return ;
  }
  
  cout << "receive from " << sock->remote_endpoint().address();
  shared_ptr<vector<char> > str(new vector<char>(100, 0));  // 建立接收数据的缓冲区
  sock->async_read_some(buffer(*str)),  // 异步读取数据
  	bind(&client::read_handler, this, placeholders::error, str));
  start();  // 再次启动异步连接
}

void read_handler(const system::error_code& ec, shared_ptr<vector<char> > str)
{
  if (ec)  // 处理错误代码
  {
    return ;
  }
  
  cout << &(*str)[0] << endl;
}

int main()
{
  try
  {
    cout << "client start." << endl;
    io_service ios;
    client cl(ios);
    ios.run();
  }
  catch (std::exception & e)  // 捕获可能发生的异常
  {
    cout << e.what() << endl;
  }
}
```



#### 12.5.UDP协议通信

代码示范如下：

```c++
int main()  // 服务端主函数
{
  cout << "udp server start." << endl;
  io_service ios;
  
  ip::udp::socket sock(ios, ip::udp::endpoint(ip::udp::v4(), 6699));  // 创建一个udp的socket对象，使用ipv4协议，端点6699
  
  for (;;)
  {
    char buf[1];  // 一个临时用缓冲区
    ip::udp::endpoint ep;  // 要接受连接的远程端点
    
    system::error_code ec;
    sock.receive_from(buffer(buf), ep, 0, ec);  // 阻塞等待远程连接，连接的端点信息保存在ep对象中
    
    if (ec && ec != error::message_size)
    {
      throw system::system_error(ec);
    }
    
    cout << "send to " << ep.address() << endl;  // 得到远程端点
    sock.send_to(buffer("hello asio udp"), ep);  // 发送数据
  }
}

int main()  //客户端主函数
{
  cout << "client start:" << endl;
  ios_service ios;
  
  ip::udp::endpoint send_ep(ip::address::from_string("127.0.0.1"), 6699);  // 连接端点
  ip::udp::socket sock(ios);  // 创建udp socket对象
  sock.open(ip::udp::v4());  // 使用ipv4打开socket
  
  char buf[1];
  sock.send_to(buffer(buf), send_ep);  // 向连接端点发送连接数据
  
  vector<char> v(100, 0);
  ip::udp::endpoint recv_ep;
  sock.receive_from(buffer(v), recv_ep);  // 接收数据
  cout << "recv from " << recv_ep.address() << " ";
  cout << &v[0] << endl;
}
```









