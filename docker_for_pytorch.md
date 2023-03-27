
问题：阿里云镜像无法配置完成
```shell
sed -i "s|EXTRA_ARGS='|EXTRA_ARGS='--registry-mirror=|g" /var/lib/boot2docker/profile
```

## **docker quick start**
### **一、基本操作**

拉取pytorch镜像

```shell
docker pull pytorch/pytorch
```

查看镜像
```shell
docker images 
```

在docker运行镜像
```shell
docker run -it pytorch/pytorch
```
进入的是一个类linux系统

```shell
docker ps // 看哪个容器是在运行的
docker stop
docker rm/rmi
```

### **二、复制所需配置到容器中**

在所要复制的文件夹，右键使用`git bash here`\
将本地文件复制到容器当中
```shell
docker cp 本地文件夹 容器ID:文件夹
```

在使用httpserver后需要再次进入环境，可以再cmd中输入下面命令再次进入容器
```shell
docker exec -it containerID /bin/bash
```

### 下载配置好的项目

1、登录阿里云docker registry
```shell
docker login --username=username registry.cn-hangzhou.aliyuncs.com 
```

2、将镜像推动到Registry中
```shell
docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/命名空间/pytorch:[镜像版本号]

docker push registry.cn-hangzhou.aliyuncs.com/命名空间/pytorch:[镜像版本号]
```

3、从Registry拉取镜像
```shell
docker pull registry.cn-hangzhou.aliyuncs.com/命名空间/pytorch:[镜像版本号]
```

### **三、DockerFile**
dockerfile是用来构建docker镜像的文件。命令参数脚本

构建步骤：\
1、编写一个dockerfile文件\
2、docker build 构建成为一个镜像\
3、docker run 运行镜像\
4、docker push 发布镜像（DockerHub，阿里云镜像仓库）

#### **3.1 Dockerfile的构建过程**
基础知识：\
1、每个保留关键字（指令）都必须是大写字母\
2、执行从上到下，顺序执行\
3、# 表示注释\
4、每一个指令都会创建提交一个新的镜像层，并提交

dockerfile是面向开发的，我们以后要发布项目，做镜像，就需要编写dockerfile文件

步骤：开发、部署、运维
- DockerFile：构建文件，定义了一切的步骤，源代码
- DockerImages：通过DockerFile构建生成的镜像，最终发布和运行的产品
- Docker容器：容器就是镜像运行起来提供服务

#### **3.2 DockerFile的指令**
以前我们用的是别人的镜像，现在我们知道了这些指令后，我们来自己写一个自己的镜像
```shell
FROM        # 基础镜像，一切从这里开始构建
MAINTAINER  # 镜像是谁写的，姓名+邮箱
RUN         # 镜像构建的时候需要运行的命令
ADD         # 步骤：搭建tomcat镜像，这个tomcat压缩包就是添加内容
WORKDIR     # 镜像工作目录
VOLUME      # 挂载的目录
EXPOSE      # 保留端口配置
CMD         # 指定这个容器启动的时候要运行的命令，只有最后一个会生效，可以被替代
ENTRYPOINT  # 指定这个容器启动的使用后要运行的命令，可以追加命令
ONBUILD     # 当构建一个被继承 DockerFIle 这个时候就会运行 ONBUILD 的指令。触发指令
COPY        # 类似ADD，将我们文件拷贝到镜像中
ENV         # 构建的时候设置环境变量

```

#### **3.3 实战测试**

**Dockerfile** 文件 

**1、编写Dockerfile的文件** 
```shell
FROM pytorch/pytorch:latest

# 设置工作目录
RUN mkdir -p /project
WORKDIR /project

# 复制算法代码和依赖包
COPY ./predict ./
COPY ./requirements.txt requirements.txt
RUN pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple

COPY run.sh /run.sh
RUN chmod +x /run.sh
ENTRYPOINT ["/run.sh"]

```
**2、通过这个文件构建镜像**
```shell
# 命令 docker build -f dockerfile文件路径 -t 镜像名:[tag] .
docker build -f mydockerfile-pytorch -t mypytorch:0.1 .
# 其中 . 构建的是上下文与环境
```
**3、测试运行**
```shell
docker run -it mypytorch:1.0
```


**run.sh**文件
```shell
#! /bin/bash

python httpserver.py &
python predict.py &
```

```shell
docker logs -f # 查看日志
```
如果要保存文件的话，input和output文件挂载到本地

#### **3.4 CMD和ENTRYPOINT的区别**
```shell
CMD         # 指定这个容器启动的时候要运行的命令，只有最后一个会生效，可以被替代
ENTRYPOINT  # 指定这个容器启动的使用后要运行的命令，可以追加命令
```
**3.4.1 测试CMD**
```shell
# 编写 dockerfile 文件
$ vim dockerfile-cmd-test
FROM python:3.7
CMD ["ls", "-a"]

# 构建镜像
$ docker build -f dockerfile-cmd-test -t cmdtest .

# run运行，发现我们的 ls -a 命令生效
$ docker run 420bd443c905
.
..
.dockerenv
bin
boot
dev
etc
home
lib
lib64

# 想追加一个命令 -l： ls -al
$ docker run 420bd443c905 -l
D:\docker toolbox\Docker Toolbox\docker.exe: Error response from daemon: OCI runtime create failed: container_linux.go:349: starting container process caused "exec: \"-l\": executable file not found in $PATH": unknown.

# CMD 的情况下， -l替换了 CMD["ls", "a"] 命令，-l 不是命令，所以报错
# 正确形式为：
$ docker run 420bd443c905 ls -al
```

**3.4.2 测试entrypoint**
```shell
# 编写 dockerfile 文件
$ vim dockerfile-entrypoint-test
FROM python:3.7
ENTRYPOINT ["ls", "-a"]

# 构建镜像
$ docker build -f dockerfile-cmd-test -t cmdtest .

# run运行，发现我们的 ls -a 命令生效
$ docker run 89acb05297f1

# 我们的追加命令，是直接拼接在我们的ENTRYPOINT命令后面
$ docker run 89acb05297f1 -l
total 72
drwxr-xr-x   1 root root 4096 Mar 27 02:58 .
drwxr-xr-x   1 root root 4096 Mar 27 02:58 ..
-rwxr-xr-x   1 root root    0 Mar 27 02:58 .dockerenv
```
dockerfile中很多命令都十分相似

#### **3.5 实战：Tomcat镜像**
1、准备镜像文件 tomcat 压缩包，jdk 压缩包\
2、编写dockerfile文件，官方命名`Dockerfile`，build会自动寻找这个文件，就不需要-f指定了