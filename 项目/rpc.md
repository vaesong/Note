# RPC 分布式网络通信框架项目

## 理论

### 集群

集群：每一台服务器独立的运行一个工程的所有模块

优点：

- 多个服务器，可以同时承受的并发量大大增加

缺点：

- 项目代码需要整体重新编译，而且需要多次部署
- 仍然不能解决 不同模块需要不同的硬件资源



### 分布式

分布式：一个工程拆分成多个模块，每一个或多个模块独立部署在一个服务器主机上，所有服务器协同工作共同提供服务，每一台服务器称作分布式的一个节点，根据节点的并发要求，对一个节点可以再做节点模块集群部署

优点：

- 一个服务器节点集群部署，可以提高并发量
- 每个服务器节点出现问题，只需要在该节点上重新编译和部署
- 可以解决不同模块对于不同硬件资源的需求



### RPC 通信原理

RPC（Remote  Call）



## 环境配置

### 安装 muduo

可能出现 ByteSize 的问题

![](https://raw.githubusercontent.com/vaesong/Images/master/20230601192227.png)

先修改为 ByteLong

注释掉 CMakeLists.txt 里面的 Werror

![](https://raw.githubusercontent.com/vaesong/Images/master/20230601192538.png)



### 运行protobuf demo

如果是编译的时候报很多错误，看一下升级 g++ 版本

出现下面错误，编译的时候加上，`-lpthread`

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230517151920.png)

创建test.proto文件，添加以下信息：
```proto
//使用的版本号
syntax = "proto3";

//相当于打包后的 C++ 命名空间
package fixbug;


//定义登录请求消息类型
message LoginRequest{
    string name = 1;
    string pwd = 2;
}

//定义登录相应消息类型
message LoginResponse{
    int32 errcode = 1;
    string errmsg = 2;
    bool success = 3;
}
```

```C++
#include "test.pb.h"
#include <iostream>
#include <string>

int main(){
    //首先实例化一个登录类
    fixbug::LoginRequest req;
    req.set_name("zhang san");
    req.set_pwd("123456");

    //序列化，需要有字符串接受
    std::string send_str;
    if(req.SerializeToString(&send_str)){
        std::cout << send_str << std::endl;
    }
    
    //反序列化
    fixbug::LoginRequest req2;
    if(req2.ParseFromString(send_str)){
        std::cout << req2.name() << std::endl;
        std::cout << req2.pwd() << std::endl;
    }

}
```

demo2: 主要是学习列表的使用

`add_XXXList()` 函数会返回一个列表元素的指针

`XXX_list_size()` 返回列表的元素个数

```protobuf
//使用的版本号
syntax = "proto3";

//相当于打包后的 C++ 命名空间
package fixbug;

//定义错误码和错误消息
message ResultCode{
    int32 errcode = 1;
    bytes errmsg = 2;
}

//定义登录请求消息类型
message LoginRequest{
    string name = 1;
    string pwd = 2;
}

//定义登录响应消息类型
message LoginResponse{
    ResultCode result = 1;
    bool success = 2;
}

//定义用户类型
message User{
    bytes name = 1;
    uint32 age = 2;
    enum Sex{
        MAN = 0;
        WOMAN = 1;
    }
    Sex sex = 3;
}

//定义获得朋友的请求类型
message GetFriendListsRequest{
    uint32 userid = 1;
}

//定义朋友的返回类型
message GetFriendListsResponse{
    ResultCode result = 1;
    repeated User friend_list = 2;
}
```

```C++
#include "test.pb.h"
#include <iostream>
#include <string>

int main(){

    // fixbug::LoginResponse rsp;
    // fixbug::ResultCode *rc = rsp.mutable_result();
    // rc->set_errcode(1);
    // rc->set_errmsg("登录处理失败");

    fixbug::GetFriendListsResponse rspB;
    fixbug::ResultCode *rc = rspB.mutable_result();
    rc->set_errcode(0);

    fixbug::User *user1 = rspB.add_friend_list();
    user1->set_name("zhang san");
    user1->set_age(20);
    user1->set_sex(fixbug::User::MAN);

    fixbug::User *user2 = rspB.add_friend_list();
    user2->set_name("li si");
    user2->set_age(22);
    user2->set_sex(fixbug::User::WOMAN);

    std::cout << rspB.friend_list_size() << std::endl;

    return 0;
}
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230517175908.png)



### protobuf 原理

message 定义的是方法的 **调用参数和返回的类**，继承于 **Message**

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230522152627.png)



Service 定义会产生两种，Callee ServiceProvider rpc（rpc 服务提供者）、Caller ServiceConsumer（rpc 服务消费者）

Callee ServiceProvider rpc（rpc 服务提供者）：

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230522153146.png)

Caller ServiceConsumer 构造函数需要传递 RPCChannel指针，并且需要该 chanel去调用 CallMethod

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230522153319.png)

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230522153912.png)

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230522153525.png)



## 一些模块

整个框架的模块主要是几个

- `MprpcApplication` 模块，全局的 `Init` 函数，调用 `MprpcConfig` 模块加载配置文件，初始化整个框架
- `MprpcConfig` 模块，配置模块，加载配置
- `MprpcProvider` 模块
  - 主要是 `NotifyService` 方法来注册 RPC 服务
  - `Run` 方法用来连接 `zookeeper` 创建 `znode` 节点。创建 TcpServer 对象，绑定连接回调函数，接受消息的回调函数，发送消息的回调函数
    - 其中 `OnMessage` 回调函数用来对网络接收的 request 反序列化，分析调用的服务和方法
    - `SendRpcResponse` 回调函数绑定到 `google::protobuf::Closure* done` 指针上，重写 `google::protobuf::NewCallback` 函数来绑定。接收 response 然后序列化通过网络发送
- `Mprpcchannel` 模块，在 RPC 调用端使用，通过继承了 `google::protobuf::RpcChannel` 类，重写其 `CallMethod` 方法，用于客户端 `request` 参数对象的序列化，构建发送请求的格式，连接 `zookeeper` 获得提供服务的 ip 和 port，建立 TCP 连接，发送数据，接受序列化数据，反序列化得到 `response`
- `Mprpccontroller` 模块，调用方 针对错误的情况，设置相应的错误信息，例如当调用方序列化失败的时候，或者 TCP 发送失败等
- `Logger` 模块，日志模块，采用单例模式，用于记录服务端的日志信息
- `zookeeperutil` 模块，被调用方连接后，创建 `znode` 节点，记录被调用方的 ip 和 port，调用方连接后，根据服务和方法名查询 `znode` 节点保存的数据



`NotifyService` 方法，就是把已经发布的类和方法，都保存到一个表里面，该表的结构是

`std::unordered_map<服务名称，服务的信息>`，其中服务信息是 `服务指针、std::unordered_map<函数名称，函数描述信息>`

```C++
// 框架提供给外部使用的，可以发布 rpc 方法的接口
void RpcProvider::NotifyService(google::protobuf::Service * service){

    //用来保存该服务的指针对象和方法信息
    ServiceInfo service_Info;
    service_Info.m_service = service;

    //获得该 Service 的服务对象的描述信息
    const google::protobuf::ServiceDescriptor * pserviceDesc = service->GetDescriptor();
    //获得该服务的名字
    std::string service_name = pserviceDesc->name();
    //获得该服务的方法的数量
    int methodCnt = pserviceDesc->method_count();

    //遍历所有的方法，存到 info 信息里面
    for(int i = 0; i < methodCnt; ++i){
        //获得每个方法的描述信息
        const google::protobuf::MethodDescriptor * pmethodDesc = pserviceDesc->method(i);
        //获得方法的名字
        std::string method_name = pmethodDesc->name();
        //存到 serviceInfo 信息里面
        service_Info.m_methodMap.insert({method_name, pmethodDesc});
    }

    //存到表里
    m_serviceMap.insert({service_name, service_Info});
}
```



### MPRPCConfig 模块

该模块是在 `MpRpcApplication ` 启动的时候，根据传递的文件，调用 `LoadConfigFile` 方法读取里面的配置

```C++
// rpcserverip rpcserverport zookeeperip zookeeperport
//框架读取配置文件类
class MyRpcConfig{
public:
    //负责解析加载的配置文件
    void LoadConfigFile(const char* config_file);
    //负责根据 key 值查询
    std::string Load(const std::string& key);

private:
    std::unordered_map<std::string, std::string> m_configMap;
    void Trim(std::string& src_buf);
};
```

有两个主要的方法，一个是 `LoadConfigFile` 从传递的配置文件中，读取配置参数，保存到 Map 中。另外一个是 `Load`，就是根据传入的 key，从 Map 中查找对应的 Value，并返回。还有一个是 `Trim` 函数，用来处理配置文件中的空格等

首先是 `LoadConfigFile` 方法

```C++
void MyRpcConfig::LoadConfigFile(const char* config_file){
    FILE *fp = fopen(config_file, "r");
    if(fp == nullptr){
        std::cout << config_file << " is not exist!" << std::endl;
        exit(EXIT_FAILURE);
    }

    //读取文件内容，一行一行读取，去掉每行的开头和末尾的空格，然后在判断是否合法
    // feof 到达文件结尾返回 非 0，否则返回 0
    while (!feof(fp))
    {
        char buf[CONFIG_BUF_SIZE] = {0};
        fgets(buf, CONFIG_BUF_SIZE, fp);

        //去掉字符串前面多余的空格
        std::string read_buf(buf);
        Trim(read_buf);

        //如果是注释
        if(read_buf[0] == '#' || read_buf.empty()){
            continue;
        }

        //解析配置项
        int idx = read_buf.find('=');
        //如果配置不合法
        if(idx == -1){
            continue;
        }

        std::string key;
        std::string value;
        //找到 key，并且去除空格
        key = read_buf.substr(0, idx);
        Trim(key);

        //找到 value，并且去除空格，查找是否存在 '\n'
        int end_idx = read_buf.find('\n', idx);
        //没有 '\n'，就用最后的大小减
        if(idx == -1)
            value = read_buf.substr(idx+1, read_buf.size() - idx - 1);
        else
            value = read_buf.substr(idx+1, end_idx - idx - 1);
        Trim(value);

        m_configMap.insert({key, value});
    }
    
}
```

然后是 `Trim` 方法

```C++
//去掉前后多余的空格
void MyRpcConfig::Trim(std::string& src_buf){
    int idx = src_buf.find_first_not_of(' ');
    if(idx != -1){
        //说明前面有空格
        src_buf = src_buf.substr(idx, src_buf.size() - idx);
    }

    //去掉字符串后面多余的空格
    idx = src_buf.find_last_not_of(' ');
    if(idx != -1){
        //说明后面有空格
        src_buf = src_buf.substr(0, idx + 1);
    }
}
```

最后是 `Load` 方法

```C++
//负责根据 key 值查询
std::string MyRpcConfig::Load(const std::string& key){
    auto it = m_configMap.find(key);
    if(it == m_configMap.end()){
        return "";
    }
    return it->second;
}
```



### MprpcApplication 模块

主要是使用静态函数初始化，mprpc 框架的基础类，主要是初始化框架，加载配置文件，设计成为单例模式。

```C++
// mprpc 框架的基础类，主要是初始化框架，设计成为单例模式
class MprpcApplication{
public:
    static void Init(int argc, char** argv);
    static MprpcApplication& GetInstance();
    MyRpcConfig& GetConfig();

private:
    //删除默认构造函数，拷贝构造函数，移动构造函数
    MprpcApplication(){};
    MprpcApplication(const MprpcApplication& ) = delete;
    MprpcApplication(MprpcApplication&& ) = delete;

    static MyRpcConfig m_config;
};
```

首先是`GetInstance` 方法，创建一个静态的类对象，返回引用

```C++
MprpcApplication& MprpcApplication::GetInstance(){
    static MprpcApplication app;
    return app;
}
```

然后是 `Init` 方法，判断命令行的参数是否正确，然后根据参数加载配置文件

```C++
void MprpcApplication::Init(int argc, char** argv){
    //如果参数量不够，直接报错
    if(argc < 2){
        //打印错误信息
        ShowArgsHelp();
        exit(EXIT_FAILURE);
    }

    int opt = 0;
    const char* optstring = "i:";
    std::string config_file;

    while((opt = getopt(argc, argv, optstring)) != -1){
        switch (opt)
        {
        case 'i':
            config_file = optarg;
            break;
        case '?':
            std::cout << "invalid arg" << std::endl;
            ShowArgsHelp();
            exit(EXIT_FAILURE);
        case ':':
            std::cout << "need <configfile" << std::endl;
            ShowArgsHelp();
            exit(EXIT_FAILURE);
        default:
            break;
        }
    }

    //加载配置文件，获得 rpcserver_ip、rpcserver_port、zookeeper_ip、zookeeper_port
    m_config.LoadConfigFile(config_file.c_str()); 
}
```

最后是 `GetConfig` 方法，返回 `MprpcConfig` 的指针

```C++
MyRpcConfig& MprpcApplication::GetConfig(){
    return m_config;
}
```



### MprpcChannel 模块

定义一个 `MprpcChannel` 类，继承了 `google::protobuf::RpcChannel` ，然后把其中的 `CallMethod` 方法进行重写

```C++
class MprpcChannel : public google::protobuf::RpcChannel{
public:
    void CallMethod(const google::protobuf::MethodDescriptor* method,
                          google::protobuf::RpcController* controller, const google::protobuf::Message* request,
                          google::protobuf::Message* response, google::protobuf::Closure* done);
};
```

重写 `CallMethod` 方法

- 根据 `method` 描述符，获得 `service_name` 和 `method_name`，传输过去后才知道调用哪个服务和方法
- 把 `request` 序列化成 `args_str`，
- 由于 TCP 粘包问题（为了区分`service_name`  `method_name` 和序列化的`request`  ），构建传输的数据格式，并且序列化，和`args_str`组成发送数据
- 通过 `zookeeper` 查询 ip 和 port，创建 TCP 连接，发送数据
- 然后 `recv` 等待接收数据，反序列化得到 response

```C++
/*
发送的请求头的数据格式
header: header_size + (service_name, method_name, args_size) + args
*/
void MprpcChannel::CallMethod(const google::protobuf::MethodDescriptor* method,
                          google::protobuf::RpcController* controller, const google::protobuf::Message* request,
                          google::protobuf::Message* response, google::protobuf::Closure* done)
{
    //根据 method 描述符，获得 service_name 和 method_name
    const google::protobuf::ServiceDescriptor* sd = method->service();
    std::string service_name = sd->name();
    std::string method_name = method->name();
	// request 参数的序列化
    uint32_t args_size = 0;
    std::string args_str;
    if(request->SerializeToString(&args_str)){
        args_size = args_str.size();
    }
    else{
        controller->SetFailed("serialize request error!");
        return;
    }

    //构建请求头格式
    mprpc::RpcHeader rpcheader;
    rpcheader.set_service_name(service_name);
    rpcheader.set_method_name(method_name);
    rpcheader.set_args_size(args_size);

    uint32_t header_size = 0;
    std::string header_str;
    if(rpcheader.SerializeToString(&header_str)){
        header_size = header_str.size();
    }
    else{
        controller->SetFailed("serialize header error!");
        return;
    }

    //组织发送的数据格式
    std::string send_rpc_str;
    send_rpc_str.insert(0, std::string((char*)&header_size, 4));
    send_rpc_str += header_str;
    send_rpc_str += args_str;

    //打印调试信息
    std::cout << "=============================================" << std::endl;
    std::cout << "header_size: " << header_size << std::endl;
    std::cout << "rpc_header_str: " << header_str << std::endl;
    std::cout << "service_name: " << service_name << std::endl;
    std::cout << "method_name: " << method_name << std::endl;
    std::cout << "args_size: " << args_size << std::endl;
    std::cout << "args_str: " << args_str << std::endl;
    std::cout << "=============================================" << std::endl;
    
    // //通过 TCP 网络发送
    // std::string ip = MprpcApplication::GetInstance().GetConfig().Load("rpcserver_ip");
    // uint16_t port = atoi(MprpcApplication::GetInstance().GetConfig().Load("rpcserver_port").c_str());

    int clientfd = socket(AF_INET, SOCK_STREAM, 0);
    if(clientfd == -1){
        std::cout << "create socket error! errno: " << errno << std::endl;
        close(clientfd);
        exit(EXIT_FAILURE);
    }


    //通过 TCP 网络发送,不过是使用的 zookeeper 查询 ip和port
    ZkClient zkcli;
    zkcli.Start();
    std::string method_path = "/" + service_name + "/" + method_name;
    std::string host_data = zkcli.GetData(method_path.c_str());
    if(host_data == ""){
        controller->SetFailed(method_path + " is not exist!");
        return;
    }

    int idx = host_data.find(':');
    if(idx == -1){
        controller->SetFailed(method_path + " address is invalid!!");
        return;
    }

    //划分出 ip 和 port
    std::string ip = host_data.substr(0, idx);
    int port = atoi(host_data.substr(idx+1, host_data.size()-idx).c_str());

    struct sockaddr_in server_addr;
    bzero(&server_addr, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = inet_addr(ip.c_str());
    server_addr.sin_port = htons(port);

    if(connect(clientfd, (struct sockaddr *)&server_addr, sizeof(server_addr)) == -1){
        char errtxt[512] = {0};
        sprintf(errtxt, "connect error! errno: %d", errno);
        controller->SetFailed(errtxt);
        close(clientfd);
        return;
    }
    else
        std::cout << "clientfd: " << clientfd << std::endl;

    //发送请求
    if(send(clientfd, send_rpc_str.c_str(), send_rpc_str.size(), 0) == -1){
        char errtxt[512] = {0};
        sprintf(errtxt, "send error! errno: %d", errno);
        controller->SetFailed(errtxt);
        return;
    }


    //接受返回结果，反序列化得到 response
    char recv_buf[1024] = {0};
    int recv_size = 0;
    if((recv_size = recv(clientfd, recv_buf, 1024, 0)) == -1){
        char errtxt[512] = {0};
        sprintf(errtxt, "recv error! errno: %d", errno);
        controller->SetFailed(errtxt);
        close(clientfd);
        return;
    }

    // std::string response_str(recv_buf, 0, recv_size);
    if(response->ParseFromArray(recv_buf, recv_size)){
        std::cout << "parse success!" << std::endl;
    }
    else{
        char errtxt[512] = {0};
        sprintf(errtxt, "parse error! response_str: %s", recv_buf);
        controller->SetFailed(errtxt);
        close(clientfd);
        return;
    }

    close(clientfd);
}
```



### MprpcProvider 模块

该模块是框架最主要的模块，总体上提供两个方法，一个是注册 RPC 服务，一个是 RUN 方法，进行监听并处理

```C++
//框架提供的专门发布 rpc 服务的网络对象类
class MprpcProvider{
public:
    // 框架提供给外部使用的，可以发布 rpc 方法的接口
    void NotifyService(google::protobuf::Service * service);

    //启动rpc服务节点，开始提供 rpc 远程网络调用服务
    void Run();

private:
    //保存发布的 Service 的指针（也就是调用的类的指针），和该 service 所有的 method （也就是类的所有方法） 
    struct ServiceInfo{
        google::protobuf::Service * m_service;
        std::unordered_map<std::string, const google::protobuf::MethodDescriptor *> m_methodMap;
    };

    //定义一个 m_map 来保存，发布的 Service 名，和其 ServiceInfo 信息
    std::unordered_map<std::string, ServiceInfo> m_serviceMap;

private:
    muduo::net::EventLoop m_eventloop;
    void OnConnection(const muduo::net::TcpConnectionPtr & conn);
    void OnMessage(const muduo::net::TcpConnectionPtr&, muduo::net::Buffer*, muduo::Timestamp);
    void SendRpcResponse(const muduo::net::TcpConnectionPtr& conn, google::protobuf::Message* response);
};
```

首先是定义一个哈希表，用来保存注册的 `Service` 名和相应的 结构体信息，其中这里的 `Service` 是定义的 RPC 服务类，继承了 `google::protobuf::Service` , 结构体里面是该 `Service` 的指针和相应的 `Method` 哈希表

首先是 `NotifyService` 函数，注册需要相应的 RPC 服务，通过对象的描述信息指针，获取 `Service` 和 `Method` 的 `name` 

保存到哈希表里面

```C++
// 框架提供给外部使用的，可以发布 rpc 方法的接口
void MprpcProvider::NotifyService(google::protobuf::Service * service){

    //用来保存该服务的指针对象和方法信息
    ServiceInfo service_Info;
    service_Info.m_service = service;

    //获得该 Service 的服务对象的描述信息
    const google::protobuf::ServiceDescriptor * pserviceDesc = service->GetDescriptor();
    //获得该服务的名字
    std::string service_name = pserviceDesc->name();
    //获得该服务的方法的数量
    int methodCnt = pserviceDesc->method_count();

    //遍历所有的方法，存到 info 信息里面
    for(int i = 0; i < methodCnt; ++i){
        //获得每个方法的描述信息
        const google::protobuf::MethodDescriptor * pmethodDesc = pserviceDesc->method(i);
        //获得方法的名字
        std::string method_name = pmethodDesc->name();
        //存到 serviceInfo 信息里面
        service_Info.m_methodMap.insert({method_name, pmethodDesc});

        LOG_INFO("service: %s、method: %s", service_name.c_str(), method_name.c_str());
    }

    //存到表里
    m_serviceMap.insert({service_name, service_Info});

}
```

然后是主函数，用来启动 `MprpcProvider` 模块

- 加载配置文件的 ip 和 port，创建 TcpServer 对象
- 绑定连接回调 `OnConnection` 和消息读写回调方法`OnMessage` ，设置线程库的数量
- 创建 `ZkClient` 对象，连接 `zookeeper` 并且根据哈希表创建 `znode` 节点
- 启动网络服务

```C++
//启动rpc服务节点，开始提供 rpc 远程网络调用服务
void MprpcProvider::Run(){
    std::string ip = MprpcApplication::GetInstance().GetConfig().Load("rpcserver_ip");
    uint16_t port = atoi(MprpcApplication::GetInstance().GetConfig().Load("rpcserver_port").c_str());
    muduo::net::InetAddress address(ip, port);

    //创建 TcpServer 对象
    // muduo::net::TcpServer server(&m_eventloop, address, "MprpcProvider");
    muduo::net::TcpServer* server = new muduo::net::TcpServer(&m_eventloop, address, "MprpcProvider");

    //绑定连接回调和消息读写回调方法， 分离了网络代码和业务代码
    server->setConnectionCallback(std::bind(&MprpcProvider::OnConnection, this, std::placeholders::_1));
    server->setMessageCallback(std::bind(&MprpcProvider::OnMessage, this, std::placeholders::_1, std::placeholders::_2, std::placeholders::_3));

    //设置 muduo 库的线程数量
    server->setThreadNum(0); 

    //定义 zookeeper, session timeout 30s, zkclient 网络I/O线程， 1/3 * timeout 时间发送 ping 消息
    ZkClient zkcli;
    zkcli.Start();
    
    // service_name 永久性节点，method_name 临时性节点
    for(auto &sp : m_serviceMap){
        //获得 service_name
        std::string service_path = "/" + sp.first;
        zkcli.Create(service_path.c_str(), nullptr, 0);
        
        //遍历该 service 的所有的 method
        for(auto &mp : sp.second.m_methodMap){
            std::string method_path = service_path + "/" + mp.first;
            char method_path_data[128] = {0};
            // ip port 拼接成数据，作为 znode 节点的数据
            sprintf(method_path_data, "%s:%d", ip.c_str(), port);
            zkcli.Create(method_path.c_str(), method_path_data, strlen(method_path_data), ZOO_EPHEMERAL);
        }
    }

    LOG_INFO("Rpcservice start service at ip: %s port: %d", ip.c_str(), port);
    //启动网络服务
    server->start();
    m_eventloop.loop();
}
```

接着是 `OnConnection` 模块，没啥好说的，就是新连接来了

```C++
//新的连接回调
void MprpcProvider::OnConnection(const muduo::net::TcpConnectionPtr& conn){
    if(!conn->connected()){
        //如果和 rpc client 的连接断开了
        conn->shutdown();
    }
}
```

最主要的 `OnMessage` 回调函数，在这里处理业务，为了解决 TCP 的粘包问题，这里自定义了消息的结构

- 从网络上接收的远程 RPC 调用请求的字符流
- 解析出 `header_str`, `request`序列化的 `args_str`
- 反序列化 `header`,得到 `service_name`, `method_name`，然后在注册表中查找 `service` 和 `method`
- 获取 `Service` 对象和 `Method` 对象，根据 `Service` 和 `Method` 获得 `request` 和 `response` 属性，然后创建
- 把请求参数 反序列化成 `request` Message
- 据远端的 rpc 请求，调用当前 rpc 节点上发布的方法。给 `method` 方法的调用，绑定一个 `Closure` 的回调函数(`SendRpcResponse`)

```C++
/*
需要协商好通信用的 protobuf 数据类型

首先是四个字节的数字，代表了 header 的大小，然后是 header_str，最后是 args_str
header_size(4字节，一个int) + header_str(里面包含了service_name，method_name，arg_size) + args_str
例如（18UserServiceLogin15zhang san123456）
*/

//已建立连接用户的读写事件回调,如果远端有 rpc 服务的调用请求，那么就会相应 OnMessage
void MprpcProvider::OnMessage(const muduo::net::TcpConnectionPtr& conn, muduo::net::Buffer* buffer, muduo::Timestamp){

    //从网络上接收的远程 RPC 调用请求的字符流
    std::string recv_buf = buffer->retrieveAllAsString();

    //从字符流中读取前四个字节的数据
    uint32_t header_size = 0;
    recv_buf.copy((char *)&header_size, 4, 0);

    //根据 header_size 读取数据头的原始字符流，反序列化数据，得到 RPC 请求的详细信息
    std::string rpc_header_str = recv_buf.substr(4, header_size);
    mprpc::RpcHeader rpcheader;
    std::string service_name;
    std::string method_name;
    uint32_t args_size;

    if(rpcheader.ParseFromString(rpc_header_str)){
        //数据头反序列化成功
        service_name = rpcheader.service_name();
        method_name = rpcheader.method_name();
        args_size = rpcheader.args_size();
    }
    else{
        //数据头反序列化失败
        LOG_ERROR("rpc_header_str: %s parse error!", rpc_header_str.c_str());
        return;
    }

    //获取 RPC 方法参数的字符流数据
    std::string args_str = recv_buf.substr(4 + header_size, args_size);

    //打印调试信息
    LOG_INFO("=============================================");
    LOG_INFO("header_size: %d", header_size);
    LOG_INFO("rpc_header_str: %s", rpc_header_str.c_str());
    LOG_INFO("service_name: %s", service_name.c_str());
    LOG_INFO("method_name: %s", method_name.c_str());
    LOG_INFO("args_size: %d", args_size);
    LOG_INFO("args_str: %s", args_str.c_str());
    LOG_INFO("=============================================");

    //然后在注册表中查找 service 和 method
    auto it = m_serviceMap.find(service_name);
    //如果 service 不存在
    if(it == m_serviceMap.end()){
        LOG_ERROR("%s is not exist!", service_name.c_str());
        return;
    }
    auto mit = it->second.m_methodMap.find(method_name);
    if(mit == it->second.m_methodMap.end()){
        LOG_ERROR("%s exist! But %s is not exist!", service_name.c_str(), method_name.c_str());
        return;
    }

    //获取 Service 对象和 method 对象
    google::protobuf::Service * service = it->second.m_service;
    const google::protobuf::MethodDescriptor * method = mit->second;

    //生成 RPC 方法调用的请求 request 和相应 response 参数
    //根据 service 和 method 获得 request 和 response 属性，然后创建
    google::protobuf::Message *request = service->GetRequestPrototype(method).New();
    google::protobuf::Message *response = service->GetResponsePrototype(method).New();
    
    //然后把请求参数 反序列化成 request Message
    if(request->ParseFromString(args_str)){
        LOG_INFO("request parse success, content: %s", args_str.c_str());
    }

    //然后给下面的 Method 方法的调用，绑定一个 Closure 的回调函数
    google::protobuf::Closure* done = google::protobuf::NewCallback<MprpcProvider,
                                                                    const muduo::net::TcpConnectionPtr&, 
                                                                    google::protobuf::Message*>
                                                                    (this, &MprpcProvider::SendRpcResponse, conn, response);

    //在框架上根据远端的 rpc 请求，调用当前 rpc 节点上发布的方法
    service->CallMethod(method, nullptr, request, response, done);
}
```

最后是 `SendRpcResponse` 函数，`method` 方法结束的最后，调用 `do->Run()`，就会调用该回调函数

该函数把 `response` 序列化，然后直接发送给 RPC 调用方

```C++
//Closure的回调操作，用于序列化 RPC 的响应和网络发送
void MprpcProvider::SendRpcResponse(const muduo::net::TcpConnectionPtr& conn, google::protobuf::Message* response){
    
    //接收 response 然后序列化并发送
    std::string response_str;
    if(response->SerializeToString(&response_str)){
        //序列化成功后，通过网络把 RPC 方法执行的结果发送到 RPC 的调用方
        conn->send(response_str);
    }
    else{
        LOG_ERROR("serialize response_str error!");
    }
    
    //模拟 http 的短连接服务，由 rpcprovider 主动断开连接
    conn->shutdown();
}
```



### MprpcController 模块

调用方 针对错误的情况，设置相应的错误信息，例如当调用方序列化失败的时候，或者 TCP 发送失败等

```C++
class MprpcController : public google::protobuf::RpcController{
public:
    MprpcController();
    void Reset();
    bool Failed() const;
    std::string ErrorText() const;
    void SetFailed(const std::string& reason);

    //目前未实现具体的功能
    void StartCancel();
    bool IsCanceled() const;
    void NotifyOnCancel(google::protobuf::Closure* callback);

private:
    // RPC 方法执行过程中的状态
    bool m_failed;

    // RPC 方法执行过程中的错误信息
    std::string m_errText;
};
```

首先是 `SetFailed` ，设置错误，以及错误信息

```C++
void MprpcController::SetFailed(const std::string& reason){
    m_failed = true;
    m_errText = reason;
}
```

然后是 `Failed` 和 `ErrorText`，返回是否错误和错误信息

```C++
std::string MprpcController::ErrorText() const{
    return m_errText;
}

void MprpcController::SetFailed(const std::string& reason){
    m_failed = true;
    m_errText = reason;
}
```



### Logger 模块

日志模块，服务器端记录日志，定义两个日志级别，`INFO` `ERROR` 

采用异步日志，先把数据写到队列里，然后再创建一个线程去写到文件中。注意线程安全

单例模式，定义两个宏定义，可以直接进行数据的写入

```C++
//日志的两个级别
enum LogLevel{
    //普通信息
    INFO,
    //错误信息
    ERROR,
};

//日志系统
class Logger{
public:

    //单例模式获取实例
    static Logger& GetInstance();
    //设置日志级别
    void SetLogLevel(LogLevel level);
    //写日志到日志队列
    void Log(std::string msg);

private:
    //日志级别
    int m_loglevel;
    //异步写日志的日志队列
    LockQueue<std::string> m_lockQue;

private:
    //设置成单例模式，private 构造函数，删掉拷贝构造，移动构造
    Logger();
    Logger(const Logger&) = delete;
    Logger(Logger&& ) = delete;

};

//定义宏,do...while(0)有什么意义呢?
//让后面的代码变成一个整体，相当于一个 大括号变成了一条语句
#define LOG_INFO(logmsgformat, ...) \
    do{ \
        Logger &logger = Logger::GetInstance(); \
        logger.SetLogLevel(INFO); \
        char c[1024]; \
        snprintf(c, 1024, logmsgformat, ##__VA_ARGS__); \
        logger.Log(c); \
    } while(0)

#define LOG_ERROR(logmsgformat, ...) \
    do{ \
        Logger &logger = Logger::GetInstance(); \
        logger.SetLogLevel(ERROR); \
        char c[1024]; \
        snprintf(c, 1024, logmsgformat, ##__VA_ARGS__); \
        logger.Log(c); \
    } while(0)
```

`GetInstance` 获取实例

```C++
//单例模式获取实例
Logger& Logger::GetInstance(){
    static Logger logger;
    return logger;
}
```

`Logger` 初始化日志模块，设置线程，设置工作函数，不停地从队列里取数据

```C++
//初始化日志模块，设置线程，设置工作函数
Logger::Logger(){
    std::thread writeLogTask([&](){
        for(;;){
            //总任务就是向队列里面写入日志
            //首先获取时间，根据时间打开文件
            time_t now = time(nullptr);
            tm *nowtm = localtime(&now);

            //拼接文件名
            char filename[128];
            sprintf(filename, "%d-%d-%d-log.txt", nowtm->tm_year+1900, nowtm->tm_mon+1, nowtm->tm_mday);

            //打开文件
            FILE *fp = fopen(filename, "a+");
            if(fp == nullptr){
                std::cout << "logger file: " << filename << " open error!" << std::endl;
                exit(EXIT_FAILURE);
            }

            //获得消息队列里面的字符串
            std::string msg = m_lockQue.Pop();

            //当前时间，写入的时间
            char time_buf[64] = {0};
            sprintf(time_buf, "[%d:%d:%d] ==> [%s]", 
                                nowtm->tm_hour, 
                                nowtm->tm_min, 
                                nowtm->tm_sec, 
                                (m_loglevel == INFO ? "info" : "error"));

            //在错误信息前面插入当前的时间
            msg.insert(0, time_buf);
            msg.append("\n");

            //把错误信息写入到文件里面
            fputs(msg.c_str(), fp);
            fclose(fp);
        }
    });

    //设置线程分离
    writeLogTask.detach();
}
```

`SetLogLevel` 和 `Log` 在宏定义的时候调用，设置日志级别，并且把日志放到队列中

```C++
//设置日志级别
void Logger::SetLogLevel(LogLevel level){
    m_loglevel = level;
}

//写日志到日志队列
void Logger::Log(std::string msg){
    m_lockQue.Push(msg);
}
```



### zookeeperutil 模块

`znode` 节点存储提供该服务的 ip 和 port ，格式是 `ip:port`

举例：服务端需要在 `zookeeper` 上面注册，使用 `Create` 函数，例如 `/Userservice/Login` ，然后存储的内容是 `127.0.0.1:9190`

客户端可以使用 `GetData` 获得 `/service_name/method_name` 的内容，然后切分出 ip 和 port，再进行 TCP 连接

```C++
class ZkClient{
public:
    ZkClient();
    ~ZkClient();

    //启动 zookeeper 服务
    void Start();
    //根据指定的 path，创建 znode 节点，state表示永久性节点还是临时性节点
    void Create(const char *path, const char *data, int datalen, int state=0);
    //获得指定 path znode 的值
    std::string GetData(const char* path);

    //设置指定 path znode 的值
    void SetData(const char *path, const char *data);
    //删除指定的 znode
    void DelData(const char *path);

private:
    //zk客户端句柄
    zhandle_t * m_zhandle;
};
```

`Start` 方法，启动 zookeeper 服务。根据配置文件的 `zookeeper_ip` 和 `zookeeper_port` ，进行初始化连接，绑定信号量

然后绑定回调函数 `global_watcher`，是一个全局的观察器，是一个单独的线程，等待会话相关的消息类型，判断是否连接 `zookeeper` 成功，然后增加信号量

```C++
//全局的watcher观察器，回调函数，是一个单独的线程
void global_watcher(zhandle_t *zh, int type, int state, const char *path, void *watcherCtx){
    //回调消息类型是和会话相关的消息类型
    if(type == ZOO_SESSION_EVENT){
        // ZkClient 和 zkserver 连接成功
        if(state == ZOO_CONNECTED_STATE){
            sem_t *sem = (sem_t*) zoo_get_context(zh);
            sem_post(sem);
        }
    }
}

//启动 zookeeper 服务
void ZkClient::Start(){
    std::string host = MprpcApplication::GetInstance().GetConfig().Load("zookeeper_ip");
    std::string port = MprpcApplication::GetInstance().GetConfig().Load("zookeeper_port");
    std::string connstr = host + ":" + port;

    /*
    zookeeper_mt 多线程版本
    zookeeper 的 API 客户端程序提供了三个线程
    API调用线程
    网络I/O线程
    watcher回调线程
    */
    m_zhandle = zookeeper_init(connstr.c_str(), global_watcher, 30000, nullptr, nullptr, 0);
    if(m_zhandle == nullptr){
        LOG_ERROR("zookeeper_init error!");
        exit(EXIT_FAILURE);
    }

    sem_t sem;
    sem_init(&sem, 0, 0);
    zoo_set_context(m_zhandle, &sem);

    //主线程等待回调函数，对于信号量的增加
    sem_wait(&sem);
    LOG_INFO("zookeeper_init success!");
}
```

`Create` 根据指定的 path，创建 znode 节点，state表示永久性节点还是临时性节点

- 0 是永久性节点
- ZOO_EPHEMERAL 临时性节点

```C++
//根据指定的 path，创建 znode 节点，state表示永久性节点还是临时性节点
void ZkClient::Create(const char *path, const char *data, int datalen, int state){
    char path_buffer[128];
    int buff_len = sizeof(path_buffer);
    int flag;

    //首选判断节点是否存在
    flag = zoo_exists(m_zhandle, path, 0, nullptr);
    
    //如果该节点并不存在
    if(flag == ZNONODE){
        //那么就创建该节点
        flag = zoo_create(m_zhandle, path, data, datalen, &ZOO_OPEN_ACL_UNSAFE, state, path_buffer, buff_len);
        if(flag == ZOK){
            LOG_INFO("znode create success... path: %s", path);
        }
        else{
            LOG_ERROR("znode create error... path: %s, flag: %d", path, flag);
            exit(EXIT_FAILURE);
        }
    }
}
```

`GetData` 获得指定 path znode 的值

```C++
//获得指定 path znode 的值
std::string ZkClient::GetData(const char* path){
    char buffer[128];
    int buffer_len = sizeof(buffer);
    int flag = zoo_get(m_zhandle, path, 0, buffer, &buffer_len, nullptr);
    if(flag != ZOK){
        LOG_ERROR("get znode error... path: %s", path);
    }
    else{
        return buffer;
    }
    return "";
}
```



### Zookeeper 分布式协调服务



#### 存在问题

- 会在 1/3 的 Timeout 事件发送 ping 心跳消息
- 设置监听 watcher 只能是一次性的，每次触发后需要重复设置
- znode 节点只存储简单的 byte 字节数组，如果存储对象，需要自己转换对象生成字节数组



## 难点！

### Segmentation Fault



## 问题

### 宏定义与 do ... while(0)

[参考博客](https://www.cnblogs.com/lizhenghn/p/3674430.html)

主要原因是会把宏定义展开，如果有两个语句，没有括号的话，会产生错误，第二条指令永远被执行

如果加上括号，那么使用的时候带分号也会出现问题



### unique_lock、lock_guard





### backslash-newline at end of file

在 .h 文件最后面添加一个空行



