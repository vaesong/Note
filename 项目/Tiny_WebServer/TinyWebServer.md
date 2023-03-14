# 运行github的Webserver

## 首先安装MySQL

[参考博客](https://blog.csdn.net/lyouhuan/article/details/124868523)

[Debian参考的这篇](https://blog.csdn.net/weixin_43961117/article/details/126755140)

## 编译运行

找不到 `mysql.h` 的包，需要手动下载

```Shell
# centos
sudo yum install mysql-devel -y
# ubuntu,debian
sudo apt-get install libmysqlclient-dev -y 
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20221026104331.png)

然后可能出现下面的问题

![image-20221026110718148](/home/vaesong/.config/Typora/typora-user-images/image-20221026110718148.png)

这时候输入 `mysql_config` 查看 `lmysqlclient`的位置

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20221026110859.png)

然后把`libmysqlclient.so` 复制到 `/usr/lib` 下面

```Shell
sudo cp /usr/lib64/mysql/libmysqlclient.so /usr/lib
```

登录 `mysql` **密码：** MySQL@123

```Shell
mysql -u root -p
```





# epoll中EPOLLSHOT的使用

对于注册了EPOLLONESHOT事件的文件描述符，**操作系统最多触发其上注册的一个事件，且只触发一次**

EPOLLSHOT相当于说，某次循环中epoll_wait唤醒该事件fd后, 就不会出现多个线程同时处理一个fd的情况

**对于主线程，主线程的职责在于：**

- 当前唤醒的事件fd为监听套接字时，由主线程来转换套接字，在转换套接字的过程中，主线程的epoll循环是处于停顿的，因为listenfd的工作线程实际上就是主线程，因此对于listenfd不需要注册EPOLLONESHOT，因为在listenfd上工作时主线程的epoll处于停顿，不会出现多个线程同时处理listenfd的情况。同时，在将监听套接字转换为套接字connfd后，将connfd注册进入epoll，此时需要附加上EPOLLONESHOT标志，因为对connfd的处理是由工作线程来处理的，要保证同时只有一个线程在处理某个fd

**对于工作线程：**

- 当前工作线程工作未完成，如在EPOLLIN中读出数据后。对于整个工作流程来说，读出数据只是工作的一半，还需要将处理后的数据写到对端，但由于注册了EPOLLONESHOT，所以对于尚未完成的工作，要重新注册，比如这里，重新注册fd为EPOLL|EPOLLET|EPOLLONESHOT。EPOLLONESHOT是必须的，因为要保证向对端写入时只有一个线程在该fd上工作

- 当前工作线程已经完成，此时该fd已经无意义，因此将该fd从epollfd中remove掉



# Mywebserver

## lock

该文件是对信号量，互斥锁以及条件变量的封装，其中互斥锁用于线程池中的线程对请求队列操作的时候，防止多个线程同时操作

而信号量是 用于线程之前的同步，相当于是操作系统的 PV操作。需要从队列中获得请求的时候，需要进行 P 操作。添加请求进队列后需要进行 V 操作

条件变量：



## 线程池

线程池就是用来存放线程以及请求队列的地方

需要创建多个线程，然后把请求放进队列，获得请求并执行处理函数，



## 定时器

定时器的逻辑是这样的

- 首先注册一个 SIGALRM 信号，当经过给定的事件就会触发一次 SIGALRM 事件
- 为了让该事件和 I/O 复用的事件统一起来，那么触发该事件的处理函数定义为，把信号值通过定义的管道发送给主线程
- 主线程收到管道的 EPOLLIN 事件后，判断是什么信号
- 如果是 SIGTERM 信号，就结束服务，如果是 SIGALRM 信号，那么就会调用 定时器类的处理函数
- 该函数就会让维护的定时器容器进行一次 tick()，相当于判断容器中有哪些已经过期了，删除那些过期的 定时器
- 对于定时器来说，删除的话，就调用自身的回调函数，从 epoll 事件表中删除客户端连接，并且关闭连接，用户数量减少



## main

 



## 压力测试

### 云服务器

- Proactor + LT + LT

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230307115901.png)

- Proactor + LT + ET

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230307120502.png)



- Proactor + ET + LT

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230307120626.png)

- Proactor + ET + ET

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230307120823.png)

- Reactor + LT + LT

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230307120952.png)



- Reactor + LT + ET

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230307121038.png)



- Reactor + ET + LT

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230307121124.png)



- Reactor + ET + ET

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230307121217.png)









### 本地

- Proactor + LT + LT

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230307144629.png)



- Proactor + LT + ET

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230307144728.png)



- Proactor + ET + LT

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230307144819.png)



- Proactor + ET + ET

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230307144909.png)



- Reactor + LT + LT
- ![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230307145055.png)



- Reactor + LT + ET

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230307145156.png)



- Reactor + ET + LT

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230307145250.png)



- Reactor + ET + ET

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230307145002.png)



## 问题

### html显示中文乱码

在头部信息的 `Content-Type` 后面加入 `;charset=utf-8`

```C++
Content-Type:text/html;charset=utf-8
```



### 图片不显示

图片不显示，类似于这种，然后把图片链接换成网页的图片就行

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230225111806.png)

原因是生成响应报文的时候忘记加空行了...



### 每次都要先注册再登录

有个错误，就是每次都要重新注册，才能进行登录

初步排查是，注册的时候可能只保存到了 map 数组中，而没有把信息成功插入到数据库里面。最后发现确实是这个问题

后续又发现还是不行

排查得知，之前把数据库的用户名和密码存储到了 m_users 里面，这样在一开始初始化的时候，就只会把客户数组地址的 m_users 给初始化了。等新来的客户连接之后，其成员变量 m_users 是空的

于是想办法，要么是定义一个 类的静态成员变量数组，让所有的对象共享 m_users，后面发现对 map 初始化不太方便

于是在 `http_conn.cpp` 的文件中定义了一个全局变量 `users` ，这样也可以让所有的对象访问到



### Segmentation fault   段错误

在某次修改了存放数据库的全局变量 users 中，出现该错误。还有之前进行字符串拼接的时候也是该问题

一般出现这种错误是一下几个原因。

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230304172333.png)

最后排查得知，是存放定时器的容器  `sort_timer_lst` 的 head 和 tail 在初始化的时候出错，把 `head = NULL` 写成了 `head == NULL`，这就造成了使用未初始化的指针。但是不修改数据库用户数组之前为什么可以呢？？？

难道是定义 `head` 的时候，`expire` 自动设置为 0，但是 `user_data` 并没有办法初始化。但是之前的时候，每次都是一启动用浏览器连接，这样就相当于给其的 `user_data` 赋值了？？？？？？？？



### webbench 编译的错误

```shell
# 首先先删除
make clean

# 然后重新编译
make && make install
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230307114141.png)

一开始会说没有这个 `ctags` 安装一下就好

```shell
sudo yum install ctags
```



然后提示说不能创建路径，手动创建

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230307115425.png)
