# 阻塞I/O 与 非阻塞 I/O

- 阻塞I/O

> 会发出系统调用，询问当前的缓冲区是否有数据，如果没有的话，会一直等待，知道内核缓冲区中存在数据

- 非阻塞 I/O

> 同样会发生系统调用，询问缓冲区中是否有数据，如果没有，就会退出系统调用，继续执行之前的逻辑，过段时间继续询问内核是否有数据

- I/O 复用

> 把需要查询的缓冲区注册到内核中，等待通知，有数据内核会通知，从而减少了频繁的系统调用

- 同步 I/O

> 内核向应用程序通知的是，就绪事件，事件需要由应用程序自行完成

- 异步 I/O 

> 内核向应用程序通知的是，完成事件，（事件已经由内核完成了）

# Select 和 Epoll

- select

```C++
//创建 select 需要监听的事件集合，并且初始化
fd_set readfds;
fd_set writefds;
FD_ZERO(&readfds);
FD_ZERO(&writefds);
struct timeval timeout;
int fd_max, fd_num;

//然后在 select 函数之前，需要把需要监听的文件描述符放到相应的集合里面，并且设置超时时间
FD_SET(clnt_sock, &read_fds);
timeout.tv_sec = 10;
timeout.tv_usec = 0;

//之后调用 select 函数,发生错误时返回-1，超时的时候返回0
fd_num = select(fd_max, &readfds, &writefds, NULL, &timeout);

//查看套接字是否发生变化，也就是是否有数据可读，可写，异常
FD_ISSET(clnt_sock, &read_fds)
```

- epoll

```C++
//首先创造一个事件，然后创造就绪的事件集合，以及一个事件注册表的文件描述符
struct epoll_event event;
struct epoll_event ep_events[EPOLL_SIZE];

//创建 epoll 实例
int epfd = epoll_create(EPOLL_SIZE);

//定义事件的类型，以及相应的文件描述符，加入到注册表中
event.events = EPOLLIN;
event.data.fd = serv_sock;
epoll_ctl(epfd, EPOLL_CTL_ADD, serv_sock, &event);

//监听事件，并且把就绪的事件返回到集合中，函数返回值是就绪的个数
int event_cnt = epoll_wait(epfd, ep_events, EPOLL_SIZE, -1);
```

# 信号

```C++
#include<signal.h>
int sigaction(int signo, const struct sigaction * act, struct sigaction* oldact)
// 成功时返回 0， 失败时返回 -1
/*
signo: 传递信号信息, 例如，SIGURG, SIGCHLD, SIGALRM
act: 处理函数的信息, 包括信号处理函数，收到信号的行为，设置信号掩码
oldact：通过这个参数获取之前注册的信号处理函数指针，不需要则传递 0
*/
```

```C++
struct sigaction
{
    //保存信号处理函数的指针
    void (*sa_handler)(int);
    //收到信号的行为，信号集类型
    sigset_t sa_mask;
    //设置信号掩码, 指定哪些信号不能发送给本进程
    int sa_flags;
}
```

```C++
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
struct sigaction act;
//设置信号处理方式
act.sa_handler = resd_childproc;
sigemptyset(&act.sa_mask);
act.sa_flags = 0;
int state = sigaction(SIGCHLD, &act, 0);


//上面的这么多操作可以通过函数封装，使得每次处理信号信息的时候，不用这么繁琐
void addsig(int sig, void(*sig_handler)(int)){
    struct sigaction sa;
    memset(&sa, '\0', sizeof(sa));
    sa.sa_handler = sig_handler;
    sa.sa_flags |= SA_RESTART;
    sigfillset(&sa.sa_mask);
    assert(sigaction(sig, &sa, NULL) != -1);
}
```
