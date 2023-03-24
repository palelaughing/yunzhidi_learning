
问题：阿里云镜像无法配置完成

## docker quick start
### 基本操作

拉取pytorch镜像

```
$ docker pull pytorch/pytorch
```

查看镜像
```
docker images 
```

在docker运行镜像
```
docker run -it pytorch/pytorch
```
进入的是一个类linux系统

```
docker ps // 看哪个容器是在运行的
docker stop
docker rm/rmi
```

### 复制所需配置到容器中

在所要复制的文件夹，右键使用`git bash here`\
将本地文件复制到容器当中
```
docker cp 本地文件夹 容器ID:文件夹
```

在使用httpserver后需要再次进入环境，可以再cmd中输入下面命令再次进入容器
```
docker exec -it containerID /bin/bash
```

### 下载配置好的项目

1、登录阿里云docker registry
```
docker login --username=username registry.cn-hangzhou.aliyuncs.com 
```

2、将镜像推动到Registry中
```
docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/命名空间/pytorch:[镜像版本号]

docker push registry.cn-hangzhou.aliyuncs.com/命名空间/pytorch:[镜像版本号]
```

3、从Registry拉取镜像
```
docker pull registry.cn-hangzhou.aliyuncs.com/命名空间/pytorch:[镜像版本号]
```

