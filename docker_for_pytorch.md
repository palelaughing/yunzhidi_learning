
问题：阿里云镜像无法配置完成
```shell
sed -i "s|EXTRA_ARGS='|EXTRA_ARGS='--registry-mirror=|g" /var/lib/boot2docker/profile
```

## docker quick start
### 基本操作

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

### 复制所需配置到容器中

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

### **DockerFile**
dockerfile是用来构建docker镜像的文件。命令参数脚本

构建步骤：\
1、编写一个dockerfile文件\
2、docker build 构建成为一个镜像\
3、docker run 运行镜像\
4、docker push 发布镜像（DockerHub，阿里云镜像仓库）

#### **Dockerfile的构建过程**
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

#### **DockerFile的指令**
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

#### **实战测试**

**Dockerfile** 文件  
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