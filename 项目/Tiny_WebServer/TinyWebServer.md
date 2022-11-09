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
