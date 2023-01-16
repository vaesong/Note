# 运行github的Webserver

## 首先安装MySQL

[参考博客](https://blog.csdn.net/lyouhuan/article/details/124868523)

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

**密码：** MySQL@1234



# 模仿webserver

## lock

该文件是对信号量，互斥锁以及条件变量的封装



# epoll中EPOLLSHOT的使用

对于注册了EPOLLONESHOT事件的文件描述符，**操作系统最多触发其上注册的一个事件，且只触发一次**

EPOLLSHOT相当于说，某次循环中epoll_wait唤醒该事件fd后, 就不会出现多个线程同时处理一个fd的情况

**对于主线程，主线程的职责在于：**

- 当前唤醒的事件fd为监听套接字时，由主线程来转换套接字，在转换套接字的过程中，主线程的epoll循环是处于停顿的，因为listenfd的工作线程实际上就是主线程，因此对于listenfd不需要注册EPOLLONESHOT，因为在listenfd上工作时主线程的epoll处于停顿，不会出现多个线程同时处理listenfd的情况。同时，在将监听套接字转换为套接字connfd后，将connfd注册进入epoll，此时需要附加上EPOLLONESHOT标志，因为对connfd的处理是由工作线程来处理的，要保证同时只有一个线程在处理某个fd

**对于工作线程：**

- 当前工作线程工作未完成，如在EPOLLIN中读出数据后。对于整个工作流程来说，读出数据只是工作的一半，还需要将处理后的数据写到对端，但由于注册了EPOLLONESHOT，所以对于尚未完成的工作，要重新注册，比如这里，重新注册fd为EPOLL|EPOLLET|EPOLLONESHOT。EPOLLONESHOT是必须的，因为要保证向对端写入时只有一个线程在该fd上工作

- 当前工作线程已经完成，此时该fd已经无意义，因此将该fd从epollfd中remove掉
