# Ch 9：套接字的可选项

本章讲的是套接字的一些选项

## 套接字的可选项与 I/O 缓冲大小

下面是一部分套接字的可选项，其中套接字的可选项又进行了分层，包括 IP层，TCP层，socket层

![](https://cdn.jsdelivr.net/gh/vaesong/Images/20220902212945.png)

其中最常用的两个方法，一个是 get，一个是 set

### getsockopt

![](https://cdn.jsdelivr.net/gh/vaesong/Images/20220902212722.png)

### setsockopt

![](https://cdn.jsdelivr.net/gh/vaesong/Images/20220902212751.png)

## SO_REUSEADDR（Timewait）

在服务器-客户端通信的过程中，如果先关闭服务器端，那么再次运行服务器端的时候，再进行绑定的过程中会出现 bind error。可能需要等待三分钟左右才能正常运行

![](https://cdn.jsdelivr.net/gh/vaesong/Images/20220902213145.png)

在四次挥手的过程中，谁先断开连接，谁就会进入 Timewait 状态。Timewait 状态是确保，当最后一个信息传递失败时，主机B会以为主机A 没有接受到而持续发送 ACK，因此设置了等待时间，占用了端口

**注意，客户端的端口是任意指定的，即使存在 Time-wait 状态也没关系**

SO_REUSEADDR 默认是 0， 设置为 1 的时候，可将 Time-wait 状态下的套接字端口号重新分配给新的套接字

## Nagle算法

![](https://cdn.jsdelivr.net/gh/vaesong/Images/20220902213832.png)

不使用：数据到达缓冲区后，直接发送，并不会等待 ACK，因此，每个单词可能都会发送一个包

使用：数据到达缓冲区后，也是发送，但是需要等待 ACK，在等待的过程中，剩余的数据到达缓冲区，再一起发送

TCP_NODELAY可以设置算法是否打开或关闭，0 为打开， 1为关闭

# Ch10：多进程服务器端

## 僵尸进程

### 产生原因

 僵尸进程是当子进程比父进程先结束，而父进程又没有回收子进程，释放子进程占用的资源，此时子进程将成为一个僵尸进程。如果父进程先退出 ，子进程被init接管，子进程退出后init会回收其占用的相关资源

 当一个进程创建了一个子进程时，他们的运行是异步的。父进程无法预知子进程会在什么时候结束，那么如果父进程很繁忙来不及wait 子进程时，那么当子进程结束时，**会丢失子进程结束时的状态信息**，处于这种考虑unix提供了一种机制可以保证只要父进程想知道子进程结束时的信息，它就可以得到

 这种机制是：在每个进程退出的时候，内核释放该进程所有的资源，包括打开的文件，占用的内存。但是仍然保留了一些信息，留下一个称为僵尸进程（Zombie）的数据结构（如进程号pid 退出状态 运行时间等），也就是说僵尸进程是为了让父进程能够获得结束时的状态信息

这些保留的信息直到进程通过调用wait/waitpid时才会释放。这样就导致了一个问题，如果没有调用wait/waitpid的话，那么保留的信息就不会释放

### Wait（）和 Waitpid（）

**Wait**

![](https://cdn.jsdelivr.net/gh/vaesong/Images/20220903220842.png)

调用该函数的时候，如果有子进程终止，那么其终止时的返回值（return或 exit) 会保存到该参数所指的内存空间

但是可能空间中还保存其他的信息，需要通过下列宏进行分离

> WIFEXITED 子进程正常终止时返回 true
>
> WEXITSTATUS 返回子进程的返回值

```C++
int status;
...
wait(&status)
...
if(WIFEXITED(status))
    cout << "子进程的返回值：" << WEXITSTATUS(status) << endl;

```

注意，调用 wait 函数的时候，如果没有已经终止的子进程，那么程序就会阻塞，直到有子进程终止

**Waitpid**

![](https://cdn.jsdelivr.net/gh/vaesong/Images/20220903221551.png)

```C++
#include<sys/wait.h>
int status;
...
int success = waitpid(-1, &status, WNOHANG);
...
if(WIFEXITED(status))
    cout << "子进程的返回值：" << WEXITSTATUS(status) << endl;
```

## 信号处理

信号机制的出现，当子进程终止的时候，**让操作系统调用函数处理**，而不是要在程序的某一段去单独处理

三种特殊情况:

- SIGALRM，已经到了调用 alarm 函数注册的时间
- SIGINT，输入了 Ctrl + C
- SIGCHLD，子进程终止

alarm 是一个函数，向其传递一个正整形参数，会在设定的时间产生 SIGALRM 信号

```C++
alarm(2)
```

### sigaction

这个函数在不同操作系统上完全相同

```C++
#include<signal.h>
int sigaction(int signo, const struct sigaction * act, struct sigaction* oldact)
// 成功时返回 0， 失败时返回 -1
/*
signo: 传递信号信息
act: 处理函数的信息
oldact：通过这个参数获取之前注册的信号处理函数指针，不需要则传递 0
*/
```

其中 sigaction 结构体的定义

```C++
struct sigaction
{
    //保存处理函数的指针
    void (*sa_handler)(int);
    //下面两个初始化为 0 就行
    sigset_t sa_mask;
    int sa_flags;
}
```

调用示例：

```C++
//处理子进程所需要的参数
pid_t pid;
struct sigaction act;
int state;

//发生信号时，父进程调用函数处理子进程
void resd_childproc(int sig){
    pid_t pid;
    int status;
    pid = waitpid(-1, &status, WNOHANG);
    if(WIFEXITED(status)){
        cout << "removed child proc: " << pid << endl;
        cout << "Child send: " << WEXITSTATUS(status) << endl;
    }
}

//绑定子进程信号终止时 调用的函数参数
act.sa_handler = resd_childproc;
sigemptyset(&act.sa_mask);
act.sa_flags = 0;
state = sigaction(SIGCHLD, &act, 0);
```

## 基于多任务的并发服务器模型

![](https://cdn.jsdelivr.net/gh/vaesong/Images/20220906101924.png)

**注意**：这里服务器端首先 accept 客户端的套接字，之后才 fork 一个子进程，把处理的内容放到子进程中

而父进程这是需要关闭和客户端套接字的连接，然后继续监听是否有新的客户端连接请求

![image-20220906102204589](C:\Users\18365\AppData\Roaming\Typora\typora-user-images\image-20220906102204589.png)

这里是示意图，因为子进程会复制父进程的文件描述符，因此 fork 之后，相当于两个文件描述符指向同一个套接字

如果只关闭其中一个的话，并不会关闭套接字，**只有两个都关闭，才能正确关闭**

因此，在父进程 fork 之后，需要把客户端的连接关掉，保证只有子进程拥有客户端套接字的文件描述符

而子进程同样，需要把 服务端套接字的文件描述符关闭

# Ch11：进程间通信

## 管道通信

![](https://cdn.jsdelivr.net/gh/vaesong/Images/20220908220449.png)

管道相当于一个通道，可以从一边输入消息，另外一边读取消息

```C++
#include<unistd.h>
int pipe(int filedes[2]);
/*
成功返回 0 ， 失败返回 -1
filedes[0]	接收管道数据时使用的文件描述符，管道的 出口
filedes[1]	传输到管道时使用的文件描述符， 管道的 入口
*/
```

**管道的数据会成为无主数据，谁先 read 谁就会先取走数据**

因此，进程之间进行通信的时候，使用两个管道更加安全

![](https://cdn.jsdelivr.net/gh/vaesong/Images/20220908221004.png)

# Ch 12：I/O 复用

之前服务端可以同时服务多个客户端，主要是创建了子进程，那么如果在不创建子进程的情况下，和多个客户端通信

让服务端一直监听 服务器与客户端 I/O 通信的套接字，发生变化的话就进行处理

![](https://cdn.jsdelivr.net/gh/vaesong/Images/20220911212327.png)

![](https://cdn.jsdelivr.net/gh/vaesong/Images/20220911212358.png)



## fd_set

这个数据结构，主要是把一些文件描述符集中到一起，方便进行监视

```C++
FD_ZERO(fd_set* fdset)		将fd_set变量的所有位初始化为0
FD_SET(int fd, fd_set* fdset) 在参数fdset指向的变量中注册文件描述符fd的信息
FD_CLR(int fd, fd_set* fdset) 从参数fdset指向的变量中清除文件描述符fd的信息
FD_ISSET(int fd, fd_set* fdset) 若参数fdset指向的变量中包含文件描述符的信息，返回 true
```



![](https://cdn.jsdelivr.net/gh/vaesong/Images/20220911212851.png)

## select 函数

使用 select 函数可以将多个文件描述符集中到一起统一监视，如果某个套接字发生了变化，select会发现

- 是否存在套接字接收数据
- 无需阻塞传输数据的套接字有哪些
- 哪些套接字发生了异常

```C++
int select(int maxfd, fd_set * readset, fd_set * writeset, fd_set * exceptest, const struct timeval * timeout);
/*
成功时返回大于 0 的值，失败时返回 -1
maxfd: 监视对象文件描述符数量
readset: 将所有关注「是否存在待读取数据」的文件描述符注册到 fd_set 型变量，并传递其地址值。
writeset: 将所有关注「是否可传输无阻塞数据」的文件描述符注册到 fd_set 型变量，并传递其地址值。
exceptset: 将所有关注「是否发生异常」的文件描述符注册到 fd_set 型变量，并传递其地址值。
timeout: 调用 select 函数后，为防止陷入无限阻塞的状态，传递超时(time-out)信息
返回值: 发生错误时返回 -1,超时时返回0,。因发生关注的时间返回时，返回大于0的值，该值是发生事件的文件描述符数。
*/
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images/20220911213705.png)

select 函数的超时时间与 select 函数的最后一个参数有关，其中 timeval 结构体定义如下：

```c++
struct timeval
{
    long tv_sec;
    long tv_usec;
};
```

调用select之后，其实就相当把没有发生变化的文件描述符清零，发生变化的仍然不变

- timeout 每次复制都要在 select 的前面，不然会造成时间不对与浪费
- 因为调用select函数会改变 fd_set, 所以需要拷贝一份



# Ch17: epoll

为了实现 I/O 的复用，在 LInux 平台上提供了另外一种方法，epoll

首先是 **select** 的优点以及缺点：

优点：

- 兼容性好，大部分操作系统都支持 select 

缺点：

- 需要循环去监视状态的变化
- 每次调用 select 都需要传递监视对象的信息

而 epoll 的优点恰恰就是 select 函数的缺点，弥补了其不足

在介绍 epoll 之前，还需要介绍一个新的结构体：**epoll_event**

```C++
struct epoll_event
{
    __uint32_t events;
    epoll_data_t data;
};
typedef union epoll_data {
    void *ptr;
    int fd;
    __uint32_t u32;
    __uint64_t u64;
} epoll_data_t;
```

这里的 events 的参数是下面几种：

- EPOLLIN：需要读取数据的情况
- EPOLLOUT：输出缓冲为空，可以立即发送数据的情况
- EPOLLPRI：收到 OOB 数据的情况
- EPOLLRDHUP：断开连接或半关闭的情况，这在边缘触发方式下非常有用
- EPOLLERR：发生错误的情况
- EPOLLET：以边缘触发的方式得到事件通知
- EPOLLONESHOT：发生一次事件后，相应文件描述符不再收到事件通知。因此需要向 epoll_ctl 函数的第二个参数传递 
- EPOLL_CTL_MOD ，再次设置事件

这里的 epoll_event 结构体，主要用来保存监听的事件，和文件描述符

对应于 select，epoll 也有相对应的函数

- epoll_create 对应于 fd_set，主要是用来**创建**一个实例
- epoll_ctl 对应于 fd_clr，主要是用来向实例中**注册**
- epoll_wait 对应于 select ，主要是用来**等待**事件的发生

### epoll_create

```C++
#include <sys/epoll.h>
int epoll_create(int size);
/*
成功时返回 epoll 的文件描述符，失败时返回 -1
size：epoll 实例的大小
*/
```

### epoll_ctl

```C++
#include <sys/epoll.h>
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
/*
成功时返回 0 ，失败时返回 -1
epfd：用于注册监视对象的 epoll 例程的文件描述符
op：用于指定监视对象的添加、删除或更改等操作
fd：需要注册的监视对象文件描述符
event：监视对象的事件类型
*/
epoll_ctl(A,EPOLL_CTL_ADD,B,C);
epoll_ctl(A,EPOLL_CTL_DEL,B,NULL);
```

下面是第二个参数的含义：

- EPOLL_CTL_ADD：将文件描述符注册到 epoll 例程
- EPOLL_CTL_DEL：从 epoll 例程中删除文件描述符
- EPOLL_CTL_MOD：更改注册的文件描述符的关注事件发生情况

### epoll_wait

```C++
#include <sys/epoll.h>
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
/*
成功时返回发生事件的文件描述符个数，失败时返回 -1
epfd : 表示事件发生监视范围的 epoll 例程的文件描述符
events : 保存发生事件的文件描述符集合的结构体地址值
maxevents : 第二个参数中可以保存的最大事件数
timeout : 以 1/1000 秒为单位的等待时间，传递 -1 时，一直等待直到发生事件
*/

int event_cnt;
struct epoll_event *ep_events;
...
ep_events=malloc(sizeof(struct epoll_event)*EPOLL_SIZE);//EPOLL_SIZE是宏常量
...
event_cnt=epoll_wait(epfd,ep_events,EPOLL_SIZE,-1);
...
```

调用函数后，返回发生事件的文件描述符个数，同时在第二个参数指向的缓冲中保存发生事件的文件描述符集合。因此，无需像 select 一样插入针对所有文件描述符的循环

### 条件触发和边缘触发

**条件触发：**对于读取数据时间来说，只要输入缓冲中有数据，就一直注册该实践

> 一开始缓冲中收到50字节数据，注册该事件，读取20字节后还剩30字节，仍然会注册该事件

**边缘触发：**只有刚输入的时候，才注册该事件

> 一开始是空，收到50字节后注册该事件，读取后20字节后还剩30字节，不会注册该事件，下次收到数据后才会注册

### 更改I/O为非阻塞

在边缘触发中，因为当从缓冲读取 4 字节数据后，如果退出的话，即使缓冲区有数据也不会注册事件，只有等到缓冲区再次接收到数据时，才会触发事件，仍然只会读取 4 字节的数据

因此，**当触发事件的时候，需要使用 while 循环去读取缓冲中的数据，直到缓冲中的数据全部被读取出来**

但是，这又会引发一个新的问题，当 while 不停读取的时候，**只有客户端主动关闭连接，发送 EOF 之后，才能够结束这个 I/O**，这样就会**导致 I/O 的阻塞**，即使有新的连接进来，却并不能显示

于是，需要把 I/O 的方式改成非阻塞 I/O

```C++
#include <fcntl.h>
int fcntl(int fields, int cmd, ...);
/*
成功时返回 cmd 参数相关值，失败时返回 -1
filedes : 属性更改目标的文件描述符
cmd : 表示函数调用目的
*/
```

```C++
//把套接字改为非阻塞的方式
void setnonblockingmode(int fd){
    //首先获得 该套接字 之前设置的属性信息
    int flag = fcntl(fd, F_GETFL, 0);
    //在此基础上，添加非阻塞 O_NONBLOCK 标志
	fcntl(fd, F_SETFL, flag | O_NONBLOCK);
}
```

# Ch18 多线程

### 进程与线程

![](https://cdn.jsdelivr.net/gh/vaesong/Images/20220916111840.png)

![](https://cdn.jsdelivr.net/gh/vaesong/Images/20220916111908.png)

**进程**分为三个区域：

- 数据区：存放全局变量
- 堆区：存放 malloc 等开辟内存的 局部变量
- 栈区：函数运行时候使用的栈

而**线程**通过把栈区分离，数据区和堆区仍然共用进程的

线程的优点：

- 上下文切换的时候，不需要切换数据区和堆
- 可以利用数据区和堆交换数据

![](https://cdn.jsdelivr.net/gh/vaesong/Images/20220916112418.png)

### 线程的创建

线程具有单独的执行流，因此**需要单独定义线程的 main 函数**，还需要请求操作系统在单独的执行流中执行该函数，完成函数功能的函数如下：

```C++
#include <pthread.h>

int pthread_create(pthread_t *restrict thread,
                   const pthread_attr_t *restrict attr,
                   void *(*start_routine)(void *),
                   void *restrict arg);
/*
成功时返回 0 ，失败时返回 -1
thread : 保存新创建线程 ID 的变量地址值。线程与进程相同，也需要用于区分不同线程的 ID
attr : 用于传递线程属性的参数，传递 NULL 时，创建默认属性的线程
start_routine : 相当于线程 main 函数的、在单独执行流中执行的函数地址值（函数指针）
arg : 通过第三个参数传递的调用函数时包含传递参数信息的变量地址值
*/

pthread_create(&t_id, NULL, thread_main, (void *)&thread_param)
```

主进程必须每次都 sleep 来等待线程执行完毕？并不需要!!!

```C++
#include <pthread.h>
int pthread_join(pthread_t thread, void **status);
/*
成功时返回 0 ，失败时返回 -1
thread : 该参数值 ID 的线程终止后才会从该函数返回
status : 保存线程的 main 函数返回值的指针的变量地址值
*/
void *thr_ret;
pthread_join(t_id, &thr_ret)
```

**编译线程的时候，需要在后面加上 -lpthread！！！**

### 线程同步

**临界区：**多个线程同时运行时，能够引发错误的代码块

线程同步用于解决线程访问顺序引发的问题。需要同步的情况可以从如下两方面考虑。

- 同时访问同一内存空间时发生的情况
- 需要指定访问同一内存空间的线程顺序的情况

**互斥量**就是一把优秀的锁，当临界区被占据的时候就上锁，等占用完毕然后再放开

```C++
#include <pthread.h>
int pthread_mutex_init(pthread_mutex_t *mutex,
                       const pthread_mutexattr_t *attr);
int pthread_mutex_destroy(pthread_mutex_t *mutex);
/*
成功时返回 0，失败时返回其他值
mutex : 创建互斥量时传递保存互斥量的变量地址值，销毁时传递需要销毁的互斥量地址
attr : 传递即将创建的互斥量属性，没有特别需要指定的属性时传递 NULL
*/
pthread_mutex_t mutex
```

```C++
#include <pthread.h>
int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);
/*
成功时返回 0 ，失败时返回其他值
*/
```

```C++
pthread_mutex_lock(&mutex);
//临界区开始
//...
//临界区结束
pthread_mutex_unlock(&mutex);
```

**信号量：**是一个同步对象，用于保持在0至指定最大值之间的一个计数值。当线程完成一次对该semaphore对象的等待（wait）时，该计数值减一；当线程完成一次对semaphore对象的释放（release）时，计数值加一

```C++
#include <semaphore.h>
int sem_init(sem_t *sem, int pshared, unsigned int value);
int sem_destroy(sem_t *sem);
/*
成功时返回 0 ，失败时返回其他值
sem : 创建信号量时保存信号量的变量地址值，销毁时传递需要销毁的信号量变量地址值
pshared : 传递其他值时，创建可由多个继承共享的信号量；传递 0 时，创建只允许 1 个进程内部使用的信号量。需要完成同一进程的线程同步，故为0
value : 指定创建信号量的初始值
*/
```

```C++
#include <semaphore.h>
int sem_post(sem_t *sem);
int sem_wait(sem_t *sem);
/*
成功时返回 0 ，失败时返回其他值
sem : 传递保存信号量读取值的变量地址值，传递给 sem_post 的信号量增1，传递给 sem_wait 时信号量减一
*/
```

```C++
sem_wait(&sem);//信号量变为0...
// 临界区的开始
//...
//临界区的结束
sem_post(&sem);//信号量变为1...
```

### 多线程服务器端



# Ch24 制作HTTP服务器端

```C++
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<string.h>
#include<sys/socket.h>
#include<pthread.h>
#include<arpa/inet.h>

#include<iostream>
#include<string>
using namespace std;

#define BUFF_SIZE 1024
#define SMALL_BUF 100

//函数声明
void error_handling(char * message);
void send_error(FILE * fp);
char * content_type(char* file);
void send_data(FILE* fp, char* ct, char* file_name);
void* request_handler(void * arg);

int main(int argc, char* argv[]){
    int serv_sock, clnt_sock;
    struct sockaddr_in serv_adr, clnt_adr;
    socklen_t adr_sz;
    char buf[BUFF_SIZE];
    //线程的 id
    pthread_t t_id;

    if(argc != 2){
        cout << "Usage : " << argv[0] << " <PORT>" << endl;
        exit(0);
    }

    serv_sock = socket(PF_INET, SOCK_STREAM, 0);
    memset(&serv_adr, 0, sizeof(serv_adr));
    serv_adr.sin_family = AF_INET;
    serv_adr.sin_addr.s_addr = htonl(INADDR_ANY);
    serv_adr.sin_port = htons(atoi(argv[1]));
    if(bind(serv_sock, (struct sockaddr *)&serv_adr, sizeof(serv_adr)) == -1)
        error_handling("bind() error !!!");
    if(listen(serv_sock, 20) == -1)
        error_handling("listen() error!!!");

    //这里就开始持续监听端口，然后对请求的客户端链接，然后创建线程处理
    while(1){
        adr_sz = sizeof(clnt_adr);
        clnt_sock = accept(serv_sock, (struct sockaddr*)&clnt_adr, &adr_sz);
        if(clnt_sock == -1)
            error_handling("accept() error!");
        else
            cout << "Connect client IP: " << inet_ntoa(clnt_adr.sin_addr) << " PORT : "<< ntohs(clnt_adr.sin_port) << endl;

        //创建线程进行处理
        pthread_create(&t_id, NULL, request_handler, &clnt_sock);
        //脱离线程
        pthread_detach(t_id);
    }
    close(serv_sock);
    return 0;
}

void error_handling(char * message){
    fputs(message, stderr);
    fputc('\n', stderr);
    exit(1);
}

//处理客户端的连接
void * request_handler(void * arg){
    //首先接收 客户端链接的 I/O
    int clnt_sock = *((int *) arg);
    char req_line[SMALL_BUF];
    FILE* clnt_read;
    FILE* clnt_write;
    //读取请求头的信息，方法，文件名, content-type等
    char method[10];
    char ct[5];
    char file_name[30];

    //使用标准I/O函数，打开客户端的文件描述符
    clnt_read = fdopen(clnt_sock, "r");
    clnt_write = fdopen(dup(clnt_sock), "w");
    fgets(req_line, SMALL_BUF, clnt_read);
    //如果接收的请求头不是 HTTP 协议的，说明出现问题了
    if(strstr(req_line, "HTTP/") == NULL){
        send_error(clnt_write);
        fclose(clnt_read);
        fclose(clnt_write);
        return 0;
    }
    //strtok()用来将字符串分割成一个个片段,第一次调用时，strtok()必需给予参数s字符串，往后的调用则将参数s设置成NULL
    //每次调用成功则返回指向被分割出片段的指针
    strcpy(method, strtok(req_line, " /"));
    strcpy(file_name, strtok(NULL, " /"));
    strcpy(ct, content_type(file_name));
    if(strcmp(method, "GET") != 0){
        send_error(clnt_write);
        fclose(clnt_read);
        fclose(clnt_write);
        return 0;
    }
    //这里关闭接收端
    fclose(clnt_read);
    send_data(clnt_write, ct, file_name);
}

void send_error(FILE * fp){
    //设置传输的错误消息
    char protocol[] = "HTTP/1.0 400 Bad Request\r\n";
    char server[] = "Server: linux Web Server \r\n";
    char cnt_len[] = "Content-length: 2048 \r\n";
    char cnt_type[] = "Content-type: text/html \r\n\r\n";
    char content[] = "<html><head<title>NETWORK</title></head><body><front size=+5><br> 发生错误！查看请求文件名和请求方式！</front></body></html>";

    //把消息一个个传输
    fputs(protocol, fp);
    fputs(server, fp);
    fputs(cnt_len, fp);
    fputs(cnt_type, fp);
    //刷新缓冲区，加快传输
    fflush(fp);
}

char* content_type(char* file){
    char extension[SMALL_BUF];
    char file_name[SMALL_BUF];
    strcpy(file_name, file);
    strtok(file_name, ".");
    strcpy(extension, strtok(NULL, "."));

    if(!strcmp(extension, "html") || !strcmp(extension, "htm"))
        return "text/html";
    else
        return "text/plain";
}

void send_data(FILE* fp, char* ct, char* file_name){
    char protocol[] = "HTTP/1.0 200 OK\r\n";
    char server[] = "Server: Linux Web Server \r\n";
    char cnt_len[] = "Content-length: 2048\r\n";
    char cnt_type[SMALL_BUF];
    char buf[BUFF_SIZE];
    FILE* send_file;

    sprintf(cnt_type, "Content-type:%s\r\n\r\n", ct);
    send_file = fopen(file_name, "r");

    if(send_file == NULL){
        send_error(fp);
        return;
    }

    fputs(protocol, fp);
    fputs(server, fp);
    fputs(cnt_len, fp);
    fputs(cnt_type, fp);

    while(fgets(buf, BUFF_SIZE, send_file) != NULL){
        fputs(buf, fp);
        fflush(fp);
    }
    fflush(fp);
    fclose(fp);
}
```





















