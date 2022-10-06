1、首先是 Dockerfile的问题，本身是没有问题的，但是运行时出现[cgroups: cannot found cgroup mount destination: unknown](https://github.com/docker/for-linux/issues/219)
![](https://cdn.jsdelivr.net/gh/vaesong/Images/20220716222815.png)
说没有挂载点，因此需要执行下面两条命令进行挂载

```python
sudo mkdir /sys/fs/cgroup/systemd
sudo mount -t cgroup -o none,name=systemd cgroup /sys/fs/cgroup/systemd
```

2、是Dockerfile运行的时候出现了 下面这种问题，说明是网络问题，需要进行换源
<img src="https://img-blog.csdnimg.cn/e006cc5b730a4a14877a0866f09d6ddf.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6L-H6Lev5byg,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center" alt="" width="865" height="695" class="jop-noMdConv">
在这Dockerfile 的这两个位置进行更换清华源
![](https://cdn.jsdelivr.net/gh/vaesong/Images/20220716222727.png)

```python
RUN sed -i 's#http://archive.ubuntu.com/#http://mirrors.tuna.tsinghua.edu.cn/#' /etc/apt/sources.list \
&& rm -rf /var/lib/apt/lists/* \
&& apt-get update
```

3、下面的 pip3安装也需要换源，不过这次是直接换源，没有添加到路径换源，见代码
![](https://cdn.jsdelivr.net/gh/vaesong/Images/20220716222900.png)

```python
RUN pip3 install XXXXXX(包名加版本) -i https://pypi.tuna.tsinghua.edu.cn/simple
```

4、预处理数据集的时候运行脚本代码

```python
python3 ./scripts/downsize_skybox.py
```

5、docker运行本地镜像出现问题
![](https://cdn.jsdelivr.net/gh/vaesong/Images/20220704102936.png)
在/Matterport3DSimulator路径下运行

```Python
nvidia-docker run -it --mount type=bind,source=$MATTERPORT_DATA_DIR,target=/root/mount/Matterport3DSimulator/data/v1/scans --volume `pwd`:/root/mount/Matterport3DSimulator mattersim:9.2-devel-ubuntu18.04
```

6、docker容器运行中宕机
如果不能运行，首先找到缓存文件

```Python
find /var/run/docker -name id(jkhbljjkahoahda256a165a1651d6a)
```
然后删除文件
```Python
rm -rf /var/run/docker/runtime-runc/moby/322aa5b139b2d792c5c096a997387fffa94e501108f82381cc2746d63451e06a/
```

7、pip安装不了
换成国内的镜像，使用阿里源或者清华源，执行下面命令
版本号如果没有，需要自行修改
```Python
 pip install torch==1.8.1+cu101 torchvision==0.9.1+cu101 torchaudio==0.8.1 -f https://download.pytorch.org/whl/torch_stable.html -i http://mirrors.aliyun.com/pypi/simple/
```

8、Push 镜像到 docker hub上面

首先是登录到 docker hub 官网，然后创建一个仓库

![](https://cdn.jsdelivr.net/gh/vaesong/Images/20220913130356.png)

之后，需要在 linux 上面登录到自己的 docker hub 账号，输入账号密码

```dockerfile
docker login
```

然后注意，需要修改要上传的镜像名称，使用 tag 命令，一定要修改成 [账户名] [仓库名]：[tagname] 的形式

```dockerfile
docker tag [之前的名称]:[之前的tag] [修改的名称]:[修改的tag]
docker tag mattersim:9.2-devel-ubuntu18.04 vaesong/mattersim:9.2-devel-ubuntu18.04
```

之后，执行命令上传

```dockerfile
docker push vaesong/mattersim:9.2-devel-ubuntu18.04
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images/20220913131203.png)

最后退出登录

```dockerfile
docker logout
```



