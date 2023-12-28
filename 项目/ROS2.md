安装 `vcs import --input https://raw.githubusercontent.com/ros2/ros2/humble/ros2.repos src` 过程中碰到

![](https://raw.githubusercontent.com/vaesong/Images/master/20230821161526.png)

首先安装一下证书

```shell
sudo apt install --reinstall ca-certificates
```

然后声明一下

```shell
export SSL_CERT_FILE=/usr/lib/ssl/certs/ca-certificates.crt
```





