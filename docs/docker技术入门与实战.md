基础入门

## 安装配置

### 安装docker引擎

#### CentOS环境下安装Docker

```shell
#首先，为了方便添加软件源头，以及支持devicemapper存储类型，安装如下软件包
$  yum update
$  yum install -y yum-utils device-mapper-persistent-data lvm2

#添加Docker稳定版本的yum软件园
$  yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

#之后更新yum软件缓存，并安装Docker
$  yum update
$  yum install -y docker-ce

#最后确认Docker服务启动
$  systemctl start docker

```



```shell
#docker配置阿里云镜像源(如果自己的服务器是阿里云服务器)
#下方中的https://f9dk003m.mirror.aliyuncs.com需要自己去阿里云平台搜索

$  mkdir -p /etc/docker
$  tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://f9dk003m.mirror.aliyuncs.com"]
}
EOF
$  systemctl daemon-reload
$  systemctl restart docker
```

<img src="https://raw.githubusercontent.com/zjmJavaByte/images/master/images/18259896-15d5cd4f39a3c862.png" style="zoom:200%;" />

## 使用Docker镜像

### 搜索镜像

```shell
#docker search 镜像名称     （实际上可以通过官网https://hub.docker.com/搜索）
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker search tensorflow
NAME                                       DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
tensorflow/tensorflow                      Official Docker images for the machine learn…   1905                 
jupyter/tensorflow-notebook                Jupyter Notebook Scientific Python Stack w/ …   269                  
tensorflow/serving                         Official images for TensorFlow Serving (http…   110                  
rocm/tensorflow                            Tensorflow with ROCm backend support            62                   
xblaster/tensorflow-jupyter                Dockerized Jupyter with tensorflow              56                   [OK]
floydhub/tensorflow                        tensorflow                                      29                   [OK]
bitnami/tensorflow-serving                 Bitnami Docker Image for TensorFlow Serving     14                   [OK]
opensciencegrid/tensorflow-gpu             TensorFlow GPU set up for OSG                   12                   
emacski/tensorflow-serving                 Project images from https://github.com/emacs…   9                    
ibmcom/tensorflow-ppc64le                  Community supported ppc64le docker images fo…   5                    
tokunagaken/tensorflow-keras-jupyter-py3   TensorFlow-gpu 1.13.1 Keras 2.2.4 python 3.5…   5                    
tensorflow/tf_grpc_test_server             Testing server for GRPC-based distributed ru…   4                    
rocm/tensorflow-autobuilds                 The repo for building latest tensorflow dock…   3                    
bitnami/tensorflow-inception               Bitnami Docker Image for TensorFlow Inception   3                    [OK]
spellrun/tensorflow-cpu                                                                    2                    
mediadesignpractices/tensorflow            Tensorflow w/ CUDA (GPU) + extras               1                    [OK]
clearlinux/tensorflow                      Tensorflow machine learning framework with t…   1                    
uodcvip/tensorflow                         Rebuild of Tensorflow base GPU image for Jup…   1                    
spellrun/tensorflow-cpu-jupyter                                                            1                    
mpioperator/tensorflow-benchmarks          TensorFlow benchmarks using MPI.                1                    [OK]
spellrun/tensorflow2-cpu                                                                   1                    
clearlinux/tensorflow-serving              Tensorflow serving a serving system for mach…   1                    
bitnami/tensorflow-resnet                  Bitnami Docker Image for TensorFlow ResNet      0                    [OK]
kuberlab/tensorflow                                                                        0                    
opensciencegrid/tensorflow                 TensorFlow image with some OSG additions        0

#搜索官方提供的  带有NGINX关键字的镜像
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker search --filter=is-official=true nginx
NAME      DESCRIPTION                STARS     OFFICIAL   AUTOMATED
nginx     Official build of Nginx.   14896     [OK] 

#搜索所有收藏数量超过100的  关键字包括Redis的镜像
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker search --filter=stars=100 redis
NAME            DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
redis           Redis is an open source key-value store that…   9466      [OK]       
bitnami/redis   Bitnami Redis Docker Image                      181                  [OK]
```

### 获取镜像

```shell
# 1、docker pull 名称:标签
# 2、docker pull 名称         (如果不指定标签那拉取的就是最新的版本，建议最好不要用这种方式)
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker pull ubuntu:18.04
18.04: Pulling from library/ubuntu
4bbfd2c87b75: Pull complete 
d2e110be24e1: Pull complete 
889a7173dcfe: Pull complete 
Digest: sha256:04919776d30640ce4ed24442d5f7c1a8e4bd0e4793ed9469843cedaecb0d72fb
Status: Downloaded newer image for ubuntu:18.04
docker.io/library/ubuntu:18.04
```

`提问：当我们拉取一个ubuntu:18.04镜像时，会发现出现了三个镜像的拉取，这是为什么？ `

### 查看镜像信息

#### 使用images命令列出镜像信息

```shell
#docker images
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED      SIZE
ubuntu       18.04     81bcf752ac3d   2 days ago   63.1MB

#查看images能携带哪些命令
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker images --help
Usage:  docker images [OPTIONS] [REPOSITORY[:TAG]]
List images
Options:
  -a, --all             Show all images (default hides intermediate images)    #列出所有（包括临时文件）镜像文件
      --digests         Show digests										   #列出镜像的数字摘要值
  -f, --filter filter   Filter output based on conditions provided			   #过滤列出镜像列表
      --format string   Pretty-print images using a Go template
      --no-trunc        Don't truncate output
  -q, --quiet           Only show image IDs									   #仅仅输出镜像ID信息

```

#### 使用tag命令添加镜像标签

```shell
#为本地镜像任意添加新的标签
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker tag ubuntu:18.04 myubuntu

#查看镜像信息
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED      SIZE
myubuntu     latest    81bcf752ac3d   2 days ago   63.1MB
ubuntu       18.04     81bcf752ac3d   2 days ago   63.1MB
#可以看到这两个的镜像ID是一样的，实际上指向的都是同一个文件
```

#### 使用inspect命令查看详细信息

```shell
#docker inspect  镜像的名称
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker inspect  myubuntu
[
    {
        "Id": "sha256:81bcf752ac3dc8a12d54908ecdfe98a857c84285e5d50bed1d10f9812377abd6",
        "RepoTags": [
            "myubuntu:latest",
            "ubuntu:18.04"
        ],
        "RepoDigests": [
            "ubuntu@sha256:04919776d30640ce4ed24442d5f7c1a8e4bd0e4793ed9469843cedaecb0d72fb"
        ],
        "Parent": "",
        "Comment": "",
        "Created": "2021-05-19T19:44:33.928995967Z",
        "Container": "8cb609e6c7b4e196f45e433fd0e3f19ca56f68b7f3c8b4c347b162ab600cdc3e",
        "ContainerConfig": {
            "Hostname": "8cb609e6c7b4",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "CMD [\"/bin/bash\"]"
            ],
            "Image": "sha256:634a292fad8dc40a8a106c6f1fe72aab3c6d4382dc888118003746066395f01c",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {}
        },
        "DockerVersion": "19.03.12",
        "Author": "",
        "Config": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/bash"
            ],
            "Image": "sha256:634a292fad8dc40a8a106c6f1fe72aab3c6d4382dc888118003746066395f01c",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": null
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 63077987,
        "VirtualSize": 63077987,
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/d9555e6871f5ab41df2c197794cebb59c123a14fea57205a5c6e303de1ae4f4f/diff:/var/lib/docker/overlay2/b1da3b224fa5b25ffdb074e2d7047e824bb39422cdef960bf69f2d5453ec3b16/diff",
                "MergedDir": "/var/lib/docker/overlay2/a53d4228965ab413c2c90adf4acccbc695a6d5cb4bdfa5fe7adf2ca26a2e3a18/merged",
                "UpperDir": "/var/lib/docker/overlay2/a53d4228965ab413c2c90adf4acccbc695a6d5cb4bdfa5fe7adf2ca26a2e3a18/diff",
                "WorkDir": "/var/lib/docker/overlay2/a53d4228965ab413c2c90adf4acccbc695a6d5cb4bdfa5fe7adf2ca26a2e3a18/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:50858308da3d192ec20027838c7aaf983333731dc2dcc0cd03c4522495a4cee8",
                "sha256:c7bb31fc0e0892ba47b0059aab580a5c7575289320d47890fd7110e82e31fd58",
                "sha256:5f08512fd434ebecfa88e2e5321503762258950217478d5ba51b7a3876a036b7"
            ]
        },
        "Metadata": {
            "LastTagTime": "2021-05-22T17:38:05.338618925+08:00"
        }
    }
]

```

#### 使用history查看镜像历史

```shell
#docker history  镜像名称
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker history  myubuntu
IMAGE          CREATED      CREATED BY                                      SIZE      COMMENT
81bcf752ac3d   2 days ago   /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B        
<missing>      2 days ago   /bin/sh -c mkdir -p /run/systemd && echo 'do…   7B        
<missing>      2 days ago   /bin/sh -c [ -z "$(apt-get indextargets)" ]     0B        
<missing>      2 days ago   /bin/sh -c set -xe   && echo '#!/bin/sh' > /…   745B      
<missing>      2 days ago   /bin/sh -c #(nop) ADD file:e05689b5b0d51a231…   63.1MB
```

使用Dockerhub搜索ubuntu镜像发现，其中的每一个镜像历史对应一条Dockerfile命令

`https://github.com/tianon/docker-brew-ubuntu-core/blob/c5bc8f61f0e0a8aa3780a8dc3a09ae6558693117/bionic/Dockerfile`

![7ca18ff890a85a80f793b68be07847d](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/7ca18ff890a85a80f793b68be07847d.png)

### 删除和清理镜像

#### 使用标签来删除镜像

```shell
#docker rmi IMAGE [IMAGE IMAGE]						 IMAGE可以使标签或者ID
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker rmi myubuntu
Untagged: myubuntu:latest

#查看rmi能携带哪些命令
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker rmi --help
Usage:  docker rmi [OPTIONS] IMAGE [IMAGE...]
Remove one or more images
Options:
  -f, --force      Force removal of the image    			#强制删除镜像，即使有容器依赖他
      --no-prune   Do not delete untagged parents			#不要清理未带标签的父镜像
      
#当同时一个镜像通过tag命令添加的多个标签时，删除的只会是删除这个标签，否则连镜像文件一起删除
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker rmi myubuntu
Untagged: myubuntu:latest
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker images    		#此时可以看到我们之前为这个镜像添加的myubuntu标签的不见了
REPOSITORY   TAG       IMAGE ID       CREATED      SIZE
ubuntu       18.04     81bcf752ac3d   2 days ago   63.1MB
```

#### 使用镜像ID删除

```shell
#当使用 docker rmi 镜像ID 命令删除时，会去删除所有指向该镜像的标签，然后再去删除镜像
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED      SIZE
myubuntu     latest    81bcf752ac3d   2 days ago   63.1MB
ubuntu       18.04     81bcf752ac3d   2 days ago   63.1MB
redis        latest    bc8d70f9ef6c   9 days ago   105MB
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker rmi bc8d70f9ef6c
Untagged: redis:latest
Untagged: redis@sha256:365eddf64356169aa0cbfbeaf928eb80762de3cc364402e7653532bcec912973
Deleted: sha256:bc8d70f9ef6cae366eb6423fcb4699597c5a6b99126f3ced35b6b9d6d134375b
Deleted: sha256:a428dd1930bdc3ac7d127a745e384e634e9de5d09d48bf124e5a40409843d6f8
Deleted: sha256:2ab24e48ced3713450e4a4e2e9777a7255a5af08ac23010ea4d90d2a576f3120
Deleted: sha256:f8ac5137af2c2250b3f2cf67883883aa5e7ce915a329d0ee4ce8b4cff97965b5
Deleted: sha256:d6f89a74ae79d19535904575f2ee6d08881742af9f33d1f78e2a000b0d3fa26b
Deleted: sha256:ac98421641e727deccd15eed0f50bbe33f32689311fd387ad452d33731ba1f9f
Deleted: sha256:02c055ef67f5904019f43a41ea5f099996d8e7633749b6e606c400526b2c4b33
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED      SIZE
myubuntu     latest    81bcf752ac3d   2 days ago   63.1MB
ubuntu       18.04     81bcf752ac3d   2 days ago   63.1MB

#当有该镜像创建的容器时，删除次镜像将会报错
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker images -a
REPOSITORY   TAG       IMAGE ID       CREATED      SIZE
myubuntu     latest    81bcf752ac3d   2 days ago   63.1MB
ubuntu       18.04     81bcf752ac3d   2 days ago   63.1MB
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker rmi 81bcf752ac3d
Error response from daemon: conflict: unable to delete 81bcf752ac3d (must be forced) - image is referenced in multiple repositories

#强制删除镜像，即使有依赖该镜像的容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker images -a
REPOSITORY   TAG       IMAGE ID       CREATED      SIZE
myubuntu     latest    81bcf752ac3d   2 days ago   63.1MB
ubuntu       18.04     81bcf752ac3d   2 days ago   63.1MB
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker rmi -f 81bcf752ac3d
Untagged: myubuntu:latest
Untagged: ubuntu:18.04
Untagged: ubuntu@sha256:04919776d30640ce4ed24442d5f7c1a8e4bd0e4793ed9469843cedaecb0d72fb
Deleted: sha256:81bcf752ac3dc8a12d54908ecdfe98a857c84285e5d50bed1d10f9812377abd6
Deleted: sha256:c6ba5a1f46bb1847ee5523c97c9811f116d5ec5ab9c4671e73f228fe48b35fd7
Deleted: sha256:4f56df2e02e984a9a18f667011ae7abe7262a57118633a1a5a476e5be8e62ac8
Deleted: sha256:50858308da3d192ec20027838c7aaf983333731dc2dcc0cd03c4522495a4cee8
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE

#同同时删除所有镜像
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker rmi -f $(docker images -aq)
#其中`docker rmi -f`表示强制删除，`$(docker images -aq)`表示获取所有镜像的ID
```

#### 清理镜像

```shell
#使用Docker一段时间后，系统中可能遗留一些临时镜像文件
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker image -a               #查看所有的镜像，包括临时文件
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker image prune			#删除临时镜像
WARNING! This will remove all dangling images.
Are you sure you want to continue? [y/N] y
Total reclaimed space: 0B

[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker image prune -f         #强制删除
```

### 创建镜像

#### 基于已有容器创建

```shell
#查看已有镜像
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
ubuntu       18.04     81bcf752ac3d   2 days ago    63.1MB
ubuntu       latest    7e0aa2d69a15   4 weeks ago   72.7MB

#以交互的 (-it) 并且可执行shell (/bin/bash) 命令的形式启动容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker run -it ubuntu:18.04 /bin/bash
root@e3606c5cb5c2:/# touch test											 #创建文件夹
root@e3606c5cb5c2:/# exit										 		 #退出容器

#查看刚刚创建的容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker ps									#显示正在运行的容器
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker ps -a								#显示所有创建的容器
CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS                      PORTS     NAMES
e3606c5cb5c2   ubuntu:18.04   "/bin/bash"   33 seconds ago   Exited (0) 11 seconds ago             unruffled_austin

#根据刚刚创建的容器ID创建镜像
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
ubuntu       18.04     81bcf752ac3d   2 days ago    63.1MB
ubuntu       latest    7e0aa2d69a15   4 weeks ago   72.7MB
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker commit -m "new container" -a "zjm" e3606c5cb5c2 test:0.1
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
test         0.1       43bd406c4cab   4 seconds ago   63.1MB
ubuntu       18.04     81bcf752ac3d   2 days ago      63.1MB
ubuntu       latest    7e0aa2d69a15   4 weeks ago     72.7MB


#查看commit命令所能携带的参数
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker commit --help
Usage:  docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
Create a new image from a container's changes
Options:
  -a, --author string    Author (e.g., "John Hannibal Smith <hannibal@a-team.com>")	#作者信息
  -c, --change list      Apply Dockerfile instruction to the created image			#提交的时候执行Dockerfile
  -m, --message string   Commit message												#提交的信息
  -p, --pause            Pause container during commit (default true)				#提交时暂停容器
```

#### 基于本地模板导入

```shell
#通过OpenVZ下载模板
https://download.openvz.org/template/precreated/

#cat 下载的文件 | docker import - 名称:标签
[root@iZuf6jcqwirs6izt7krfcmZ /]# cat ubuntu-16.04-x86_64.tar.gz | docker import - ubuntu:16.04
sha256:3a07811e4fbb87a226455e0d25dae5bf1b13356f6d804caa100f39e6667061a9
[root@iZuf6jcqwirs6izt7krfcmZ /]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
ubuntu       16.04     3a07811e4fbb   5 seconds ago    505MB
test         0.1       43bd406c4cab   30 minutes ago   63.1MB
ubuntu       18.04     81bcf752ac3d   2 days ago       63.1MB
ubuntu       latest    7e0aa2d69a15   4 weeks ago      72.7MB
```

#### 基于Dockerfile创建

```shell
#Dockerfile是一个文本文件，利用指定的指令描述基于父镜像创建新镜像的过程
#基于debian:stretch-slim镜像安装Python 3 环境，构成一个新的Python 3镜像

#创建并且进入dockerfile文件
[root@iZuf6jcqwirs6izt7krfcmZ home]# vim dockerfile 											#创建文本文件

#往Dockerfile添加如下命令内容
FROM debian:stretch-slim
LABEL version="1.0" maintainer="docker user <docker_user@github>"
RUN apt-get update && apt-get install -y python3 && apt-get clean && rm -rf /var/lib/apt/lists/*

#查看dockerfile文件
[root@iZuf6jcqwirs6izt7krfcmZ home]# cat dockerfile 											#查看内容
FROM debian:stretch-slim
LABEL version="1.0" maintainer="docker user <docker_user@github>"
RUN apt-get update && apt-get install -y python3 && apt-get clean && rm -rf /var/lib/apt/lists/*

#docker build 构建镜像 （-f指定创建的Dockerfile所在的pwd路径，建议后面创建该文件的文件名陈改为官方指定的Dockerfile名字）
[root@iZuf6jcqwirs6izt7krfcmZ home]# docker build -f /home/dockerfile -t python:3 .      		#构建镜像
Sending build context to Docker daemon  11.78kB
Step 1/3 : FROM debian:stretch-slim
stretch-slim: Pulling from library/debian
fa1690ae9228: Pull complete 
Digest: sha256:828b649e84b713d1b11a1bcf88e832625e99d74aecfeeb5bec1b0846bb40dce1
Status: Downloaded newer image for debian:stretch-slim
 ---> e59fcdf363c2
Step 2/3 : LABEL version="1.0" maintainer="docker user <docker_user@github>"
 ---> Running in 3171fb34c0fe
Removing intermediate container 3171fb34c0fe
 ---> 8470bf3c3e72
Step 3/3 : RUN apt-get update && apt-get install -y python3 && apt-get clean && rm -rf /var/lib/apt/lists/*
 ---> Running in 319328f7ab75
 
。。。。。。中间省略

Removing intermediate container 319328f7ab75
 ---> d442b8f50956
Successfully built d442b8f50956
Successfully tagged python:3

#查看所有镜像
[root@iZuf6jcqwirs6izt7krfcmZ home]# docker images											#查看镜像
REPOSITORY   TAG            IMAGE ID       CREATED          SIZE
python       3              d442b8f50956   54 seconds ago   95.2MB							#刚刚创建的镜像
debian       stretch-slim   e59fcdf363c2   13 days ago     55.3MB

#查看父级容器镜像（即dockerfile文件中FROM 后面的镜像）的历史操作记录
[root@iZuf6jcqwirs6izt7krfcmZ docker_file]# docker history debian:stretch-slim
IMAGE          CREATED       CREATED BY                                      SIZE      COMMENT
e59fcdf363c2   13 days ago   /bin/sh -c #(nop)  CMD ["bash"]                 0B        
<missing>      13 days ago   /bin/sh -c #(nop) ADD file:a88164546613d1850…   55.3MB

#查看我们自己写的容器镜像的历史操作记录
[root@iZuf6jcqwirs6izt7krfcmZ docker_file]# docker history python:3
IMAGE          CREATED       CREATED BY                                      SIZE      COMMENT
d442b8f50956   2 days ago    /bin/sh -c apt-get update && apt-get install…   39.9MB    
8470bf3c3e72   2 days ago    /bin/sh -c #(nop)  LABEL version=1.0 maintai…   0B        
e59fcdf363c2   13 days ago   /bin/sh -c #(nop)  CMD ["bash"]                 0B        
<missing>      13 days ago   /bin/sh -c #(nop) ADD file:a88164546613d1850…   55.3MB

#备注：可以很显然的发现我们自己创建的镜像是建立在父级镜像的基础上
#回答了`使用history查看镜像历史`小节：从镜像history中可以看到一个镜像中包含很多镜像，并且似乎每条操作指令都会生成一个镜像
```

### 存出和载入镜像

#### 存出镜像

```shell
#查看镜像
[root@iZuf6jcqwirs6izt7krfcmZ home]# docker images
REPOSITORY   TAG            IMAGE ID       CREATED          SIZE
python       3              d442b8f50956   54 seconds ago   95.2MB
ubuntu       16.04          3a07811e4fbb   26 minutes ago   505MB
test         0.1            43bd406c4cab   56 minutes ago   63.1MB
ubuntu       18.04          81bcf752ac3d   2 days ago       63.1MB
debian       stretch-slim   e59fcdf363c2   10 days ago      55.3MB
ubuntu       latest         7e0aa2d69a15   4 weeks ago      72.7MB

#导出镜像   docker save -o  指定的位置和文件名称  镜像名称:标签 
[root@iZuf6jcqwirs6izt7krfcmZ home]# docker save -o ubuntu_18.04.tar ubuntu:18.04

#查看服务器上的镜像
[root@iZuf6jcqwirs6izt7krfcmZ home]# ll
total 63952
-rw-r--r-- 1 root root      190 May 22 19:54 dockerfile
drwx------ 2 es   es       4096 Mar 24 18:16 es
-rw------- 1 root root 65477120 May 22 20:02 ubuntu_18.04.tar

#docker save所能携带的参数
[root@iZuf6jcqwirs6izt7krfcmZ home]# docker save --help
Usage:  docker save [OPTIONS] IMAGE [IMAGE...]
Save one or more images to a tar archive (streamed to STDOUT by default)
Options:
  -o, --output string   Write to a file, instead of STDOUT  #导出的指定的位置和文件名称
```

#### 载入镜像

```shell
#将指定位置的镜像文件导入

#方式一
[root@iZuf6jcqwirs6izt7krfcmZ home]# docker load -i ubuntu_18.04.tar

#方式二
[root@iZuf6jcqwirs6izt7krfcmZ home]# docker load < ubuntu_18.04.tar
```

### 发布自己的镜像

#### 上传镜像到Dockerhub

1. 去https://hub.docker.com/先注册自己的账号

2. 通过自己的服务器登录Dockerhub

   ```shell
   [root@iZuf6jcqwirs6izt7krfcmZ ~]# docker login --help
   Usage:  docker login [OPTIONS] [SERVER]
   Log in to a Docker registry.
   If no server is specified, the default is defined by the daemon.
   Options:
     -p, --password string   Password
         --password-stdin    Take the password from stdin
     -u, --username string   Username
    
   [root@iZuf6jcqwirs6izt7krfcmZ ~]# docker login -u zjmqaaq
   Password: 
   WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
   Configure a credential helper to remove this warning. See
   https://docs.docker.com/engine/reference/commandline/login/#credentials-store
   
   Login Succeeded
   ```

3. 提交镜像

```shell
[root@iZuf6jcqwirs6izt7krfcmZ home]# docker images
REPOSITORY   TAG            IMAGE ID       CREATED             SIZE
python       3              d442b8f50956   12 minutes ago      95.2MB
ubuntu       16.04          3a07811e4fbb   37 minutes ago      505MB
test         0.1            43bd406c4cab   About an hour ago   63.1MB
ubuntu       18.04          81bcf752ac3d   2 days ago          63.1MB
debian       stretch-slim   e59fcdf363c2   10 days ago         55.3MB
ubuntu       latest         7e0aa2d69a15   4 weeks ago         72.7MB
[root@iZuf6jcqwirs6izt7krfcmZ home]# docker push test:0.1
```

#### 上传到阿里云镜像仓库

1. 登录阿里云平台

2. 找到容器服务

3. 创建命名空间

   ![ab029a980b7ad9ccec457f77594f932](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/ab029a980b7ad9ccec457f77594f932.png)

4. 创建容器镜像

   ![6e968d18d3c4eb9192ed02fddfa8a97](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/6e968d18d3c4eb9192ed02fddfa8a97.png)

   ![e4e40038b4335a921b79d3f612d77bb](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/e4e40038b4335a921b79d3f612d77bb.png)

   ![59699a928c63506fe1af9fde02e0c5a](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/59699a928c63506fe1af9fde02e0c5a.png)

5. 操作指南

   1. 登录阿里云Docker Registry

   ```
   $ docker login --username=laoy****i1018 registry.cn-shanghai.aliyuncs.com
   ```

   用于登录的用户名为阿里云账号全名，密码为开通服务时设置的密码。

   您可以在访问凭证页面修改凭证密码。

   2. 从Registry中拉取镜像

   ```
   $ docker pull registry.cn-shanghai.aliyuncs.com/laoyouji/zjmck:[镜像版本号]
   ```

   3. 将镜像推送到Registry

   ```
   $ docker login --username=laoy****i1018 registry.cn-shanghai.aliyuncs.com$ docker tag [ImageId] registry.cn-shanghai.aliyuncs.com/laoyouji/zjmck:[镜像版本号]$ docker push registry.cn-shanghai.aliyuncs.com/laoyouji/zjmck:[镜像版本号]
   ```

   请根据实际镜像信息替换示例中的[ImageId]和[镜像版本号]参数。

   4. 选择合适的镜像仓库地址

   从ECS推送镜像时，可以选择使用镜像仓库内网地址。推送速度将得到提升并且将不会损耗您的公网流量。

   如果您使用的机器位于VPC网络，请使用 registry-vpc.cn-shanghai.aliyuncs.com 作为Registry的域名登录。

   5. 示例

   使用"docker tag"命令重命名镜像，并将它通过专有网络地址推送至Registry。

   ```
   $ docker imagesREPOSITORY                                                         TAG                 IMAGE ID            CREATED             VIRTUAL SIZEregistry.aliyuncs.com/acs/agent                                    0.7-dfb6816         37bb9c63c8b2        7 days ago          37.89 MB$ docker tag 37bb9c63c8b2 registry-vpc.cn-shanghai.aliyuncs.com/acs/agent:0.7-dfb6816
   ```

   使用 "docker push" 命令将该镜像推送至远程。

   ```
   $ docker push registry-vpc.cn-shanghai.aliyuncs.com/acs/agent:0.7-dfb6816
   ```

## 操作Docker容器

### 创建容器

#### 新建容器

```shell
#docker create -it 镜像名称:镜像标签   									#此时创建的容器时未启动状态的
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker create -it ubuntu:latest
e984eccd51b999b112cde9c75142a0720e4da301cbd9014816a0855df10f4c9a
#查看所有的容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker ps -a
CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS                   PORTS     NAMES
e984eccd51b9   ubuntu         "/bin/bash"   16 seconds ago   Created                            lucid_ellis
e3606c5cb5c2   ubuntu:18.04   "/bin/bash"   4 hours ago      Exited (0) 4 hours ago             unruffled_austin
```

#### 启动容器

```shell
#docker start 镜像名称或者镜像ID

#查看所有的容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker ps -a
CONTAINER ID   IMAGE          COMMAND       CREATED         STATUS                   PORTS     NAMES
e984eccd51b9   ubuntu         "/bin/bash"   6 minutes ago   Created                            lucid_ellis
e3606c5cb5c2   ubuntu:18.04   "/bin/bash"   4 hours ago     Exited (0) 4 hours ago             unruffled_austin
#启动容器   docker start  容器ID
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker start e984eccd51b9
e984eccd51b9
#查看已经启动的容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker ps 
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS         PORTS     NAMES
e984eccd51b9   ubuntu    "/bin/bash"   6 minutes ago   Up 4 seconds             lucid_ellis
```

#### 新建并启动容器

```shell
#docker run 镜像名称或者镜像ID

#查看所有镜像
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker images
REPOSITORY   TAG            IMAGE ID       CREATED       SIZE
python       3              d442b8f50956   3 hours ago   95.2MB
ubuntu       16.04          3a07811e4fbb   3 hours ago   505MB
test         0.1            43bd406c4cab   4 hours ago   63.1MB
ubuntu       18.04          81bcf752ac3d   2 days ago    63.1MB
debian       stretch-slim   e59fcdf363c2   10 days ago   55.3MB

#启动ubuntu容器并且执行命令通过命令`/bin/echo`执行`Hello Word`
# docker run 镜像名称:标签     											#相当于pull create start 三个命令
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker run ubuntu /bin/echo 'Hello Word'
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
345e3491a907: Already exists 
57671312ef6f: Already exists 
5e9250ddb7d0: Already exists 
Digest: sha256:cf31af331f38d1d7158470e095b132acd126a7180a54f263d386da88eb681d93
Status: Downloaded newer image for ubuntu:latest
Hello Word

#命令包括
1、检查本地是否存在制定的镜像，不存在就从共有仓库下载
2、利用镜像创建一个容器，并启动该容器
3、分配一个文件系统给容器，并在只读的镜像层外面挂载一层可读可写层
4、从宿主主机配置的网桥接口中桥接一个虚拟接口倒容器中去
5、执行用户指定的应用程序
6、执行完毕后容器被自动终止

#docker run --help  所能携带参数
-t 			#Run container in background and print container ID 让Docker分配一个伪终端，并绑定到容器的标准输入上
-i			#Keep STDIN open even if not attached				让容器的标准输入保持打开
-d			#Allocate a pseudo-TTY								让容器处于后台运行
```

#### 守护态运行

```shell
#通过-d参数让容器以后台方式运行
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker run -d  ubuntu  /bin/sh "while true; do echo holle word; sleep 5; done"
07fdf56796f4c0759f5129102768e53dee8c07fff8fa406b9d8aa651eb56ae5e
```

#### 查看容器的输出日志

```shell
#docker logs 容器名称或者容器ID

#查看运行中的容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
4d464b68e6b0   ubuntu    "/bin/sh -c 'while t…"   4 seconds ago   Up 4 seconds             competent_solomon

#docker logs 容器ID
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker logs 4d464b68e6b0
holle word
holle word
holle word
holle word
holle word
holle word
holle word
holle word
holle word
holle word
holle word
holle word
holle word
holle word
holle word
holle word
holle word
holle word
holle word

#所能携带参数
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker logs --help
Usage:  docker logs [OPTIONS] CONTAINER
Fetch the logs of a container
Options:
      --details        Show extra details provided to logs					#打印详细信息
  -f, --follow         Follow log output									#持续保持输出					
      --since string   Show logs since timestamp (e.g. 2013-01-02T13:23:37Z) or relative (e.g. 42m for 42 minutes)																	  #输出从某个时间点开始的日志
  -n, --tail string    Number of lines to show from the end of the logs (default "all")  #输出最近的若干日志
  -t, --timestamps     Show timestamps										#显示时间戳信息
      --until string   Show logs before a timestamp (e.g. 2013-01-02T13:23:37Z) or relative (e.g. 42m for 42 minutes)																	 #输出某个时间之前的日志
```

### 停止容器

#### 暂停容器

```shell
#docker pause 容器名称或者容器ID

#查看运行的容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

#以交互的方式运行容器
#docker run  --name 自己取的运行容器的名称 --rm -it ubuntu bash
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker run  --name test --rm -it ubuntu bash

#查看运行的容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS          PORTS     NAMES
1c66cc9166e4   ubuntu    "bash"    41 seconds ago   Up 40 seconds             test

#暂停运行的容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker pause test
test

#查看运行的容器，发现容器已经暂停
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS                   PORTS     NAMES
1c66cc9166e4   ubuntu    "bash"    57 seconds ago   Up 57 seconds (Paused)             test
```

#### 终止容器

```shell
#docker stop 容器ID
#docker kill 容器ID

#查看运行的容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS                   PORTS     NAMES
1c66cc9166e4   ubuntu    "bash"    57 seconds ago   Up 57 seconds (Paused) 
test

#停止运行的容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker stop 1c66cc9166e4
1c66cc9166e4

#查看运行的容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

### 进入容器

1、attach命令

```shell
#docker attach 容器ID

#查看运行的容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

#以交互的形式后台运行容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker run -itd ubuntu
0520503cad4025e5f76199c8528c69baf58e8de51368866c412b63a098d04151

#查看运行的容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS         PORTS     NAMES
0520503cad40   ubuntu    "/bin/bash"   6 seconds ago   Up 5 seconds             affectionate_rosalind

#进入运行的容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker attach 0520503cad40
root@0520503cad40:/# 
```

2、exec命令

```shell
#docker exec 【参数】 容器ID

#以交互的形式后台运行容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker run -itd ubuntu
5f03ec48cdc3291c620e3e8a273224c31c7cee361f11b8814f4bd703689616a9

#查看运行的容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
5f03ec48cdc3   ubuntu    "/bin/bash"   23 seconds ago   Up 22 seconds             gallant_hertz

#进入运行的容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker exec -it 5f03ec48cdc3 /bin/bash
root@5f03ec48cdc3:/# 

#docker exec所能携带的参数
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker exec --help
Usage:  docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
Run a command in a running container
Options:
  -d, --detach               Detached mode: run command in the background        #在容器中后台执行命令
      --detach-keys string   Override the key sequence for detaching a container #指定的容器切回后台的按键
  -e, --env list             Set environment variables							 #指定环境变量
      --env-file list        Read in a file of environment variables			 #打开标准输入接受用户输入命令
  -i, --interactive          Keep STDIN open even if not attached
      --privileged           Give extended privileges to the command			 #是否执行命令，以最高权限
  -t, --tty                  Allocate a pseudo-TTY								 #分配伪终端
  -u, --user string          Username or UID (format: <name|uid>[:<group|gid>])	 #指定命令的用户名或ID
  -w, --workdir string       Working directory inside the container				 #

```

### 删除容器

```shell
#docker rm 容器ID

#查看所有容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS                      PORTS     NAMES
5f03ec48cdc3   ubuntu    "/bin/bash"   11 minutes ago   Up 11 minutes                         gallant_hertz
0520503cad40   ubuntu    "/bin/bash"   16 minutes ago   Exited (0) 11 minutes ago             affectionate_rosalind

#删除指定容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker rm 0520503cad40
0520503cad40

#查看所有容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
5f03ec48cdc3   ubuntu    "/bin/bash"   11 minutes ago   Up 11 minutes             gallant_hertz

#删除指定的运行中的容器，会报错
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker rm 5f03ec48cdc3
Error response from daemon: You cannot remove a running container 5f03ec48cdc3291c620e3e8a273224c31c7cee361f11b8814f4bd703689616a9. Stop the container before attempting removal or force remove

#强制删除指定的运行中的容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker rm -f 5f03ec48cdc3
5f03ec48cdc3

#查看所有容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

#docker rm 所能携带的参数
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker rm --help
Usage:  docker rm [OPTIONS] CONTAINER [CONTAINER...]
Remove one or more containers
Options:
  -f, --force     Force the removal of a running container (uses SIGKILL)  #是否终止并删除一个运行中的容器
  -l, --link      Remove the specified link								   #删除容器中的连接，单保留容器
  -v, --volumes   Remove anonymous volumes associated with the container   #删除容器挂在的卷
```

### 导入和导出容器

### 查看容器

#### 查看容器详情

```shell
#docker inspect 容器ID或者名称

#查看所有容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS                      PORTS     NAMES
16bea1896f0d   ubuntu    "/bin/bash"   22 seconds ago   Up 21 seconds                         amazing_buck
16101ed7b9d8   ubuntu    "/bin/bash"   38 seconds ago   Exited (0) 37 seconds ago             frosty_antonelli

#查看容器详情
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker inspect 16bea1896f0d
```

#### 查看容器内进程

```shell
#docker top 容器名称或者容器ID

#查看所有容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker ps 
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS         PORTS     NAMES
16bea1896f0d   ubuntu    "/bin/bash"   2 minutes ago   Up 2 minutes             amazing_buck

#查看容器内进程
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker top 16bea1896f0d
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                32246               32226               0                   May22               pts/0               00:00:00            /bin/bash
```

#### 查看统计信息

```shell
#docker stats 容器ID或者名称

[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker stats 16bea1896f0d
CONTAINER ID   NAME           CPU %     MEM USAGE / LIMIT   MEM %     NET I/O     BLOCK I/O   PIDS
16bea1896f0d   amazing_buck   0.00%     540KiB / 1.795GiB   0.03%     656B / 0B   0B / 0B     1

#docker stats 所能携带的参数
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker stats --help
Usage:  docker stats [OPTIONS] [CONTAINER...]
Display a live stream of container(s) resource usage statistics
Options:
  -a, --all             Show all containers (default shows just running)    		#输出所有容器统计信息
      --format string   Pretty-print images using a Go template						#格式化输出信息
      --no-stream       Disable streaming stats and only pull the first result		#不持续输出
      --no-trunc        Do not truncate output										#不截断输出信息
```

### 其他容器命令

#### 复制文件

```shell
#docker cp 源文件 容器名称:容器下的路径

#将本地的路径data复制到test容器的/temp/路径下
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker cp data test:/temp/
```

#### 查看变更

```shell
#docker diff 容器ID
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker diff 16bea1896f0d
```

#### 查看端口映射

```shell
#docker port 容器ID
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker port 16bea1896f0d
```

#### 更新配置

```shell
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker update --help
Usage:  docker update [OPTIONS] CONTAINER [CONTAINER...]
Update configuration of one or more containers
Options:
      --blkio-weight uint16        Block IO (relative weight), between 10 and 1000, or 0 to disable (default 0)																		#更新块IO限制，10~100，默认0，代表无限制
      --cpu-period int             Limit CPU CFS (Completely Fair Scheduler) period #限制CPU调度器CFS使用时间
      --cpu-quota int              Limit CPU CFS (Completely Fair Scheduler) quota  #限制CPU调度器CFS配额
      --cpu-rt-period int          Limit the CPU real-time period in microseconds	#限制CPU调度器的实时周期
      --cpu-rt-runtime int         Limit the CPU real-time runtime in microseconds	#限制CPU调度器的实时运行时间
  -c, --cpu-shares int             CPU shares (relative weight)						#限制CPU使用份额
      --cpus decimal               Number of CPUs									#限制CPU个数
      --cpuset-cpus string         CPUs in which to allow execution (0-3, 0,1)		#允许使用的CPU核
      --cpuset-mems string         MEMs in which to allow execution (0-3, 0,1)		#允许使用的内存块
      --kernel-memory bytes        Kernel memory limit								#限制使用的内核内存
  -m, --memory bytes               Memory limit										#限制使用的内存
      --memory-reservation bytes   Memory soft limit								#内存软限制
      --memory-swap bytes          Swap limit equal to memory plus swap: '-1' to enable unlimited swap #内存																			 #加上缓存区的限制，-1代表对缓冲区无限制
      --pids-limit int             Tune container pids limit (set -1 for unlimited) #
      --restart string             Restart policy to apply when a container exits	#内存退出后的重启策略
```

## Docker数据管理

### 数据卷

#### 创建数据卷

```shell
#docker volume create -d local 卷名称

[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker volume create -d local test
test
#docker volume 所能携带的参数
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker volume --help
Usage:  docker volume COMMAND
Manage volumes
Commands:
  create      Create a volume											#创建
  inspect     Display detailed information on one or more volumes		#查看详细信息
  ls          List volumes												#列出已有卷
  prune       Remove all unused local volumes							#清理无用的卷
  rm          Remove one or more volumes								#删除数据卷
Run 'docker volume COMMAND --help' for more information on a command.
```

#### 绑定数据卷

```shell
#方式一
#docker run 使用参数--mount
#-d 后台运行
#-p 映射端口号
#--name 容器名称
#--mount 将文件系统挂载附加到容器
	#type=volume:普通数据卷，映射到主机/var/lib/docker/volumes路径下
	#type=bind:绑定数据卷，映射到主机指定路径下
	#type=tmpfs:临时数据卷，只存在于内存中
#source 本地绝对路径
#destination 容器目标路径

[root@iZuf6jcqwirs6izt7krfcmZ home]# docker run -d -p 8081:8081 --name web --mount type=bind,source=/home/webapp,destination=/opt/webapp training/webapp python app.py
Unable to find image 'training/webapp:latest' locally
latest: Pulling from training/webapp
Image docker.io/training/webapp:latest uses outdated schema1 manifest format. Please upgrade to a schema2 image for better future compatibility. More information at https://docs.docker.com/registry/spec/deprecated-schema-v1/
e190868d63f8: Pull complete 
909cd34c6fd7: Pull complete 
0b9bfabab7c1: Pull complete 
a3ed95caeb02: Pull complete 
10bbbc0fc0ff: Pull complete 
fca59b508e9f: Pull complete 
e7ae2541b15b: Pull complete 
9dd97ef58ce9: Pull complete 
a4c1b0cb7af7: Pull complete 
Digest: sha256:06e9c1983bd6d5db5fba376ccd63bfa529e8d02f23d5079b8f74a616308fb11d
Status: Downloaded newer image for training/webapp:latest
b5d1f689297504301fea8b4d96efefce50e4e30265572a1cb6f84a82534c5c2e
[root@iZuf6jcqwirs6izt7krfcmZ home]# docker images
REPOSITORY        TAG            IMAGE ID       CREATED        SIZE
ubuntu            latest         7e0aa2d69a15   4 weeks ago    72.7MB
training/webapp   latest         6fae60ef3446   6 years ago    349MB
[root@iZuf6jcqwirs6izt7krfcmZ home]# docker ps -a
CONTAINER ID   IMAGE             COMMAND           CREATED         STATUS                      PORTS     NAMES
b5d1f6892975   training/webapp   "python app.py"   2 minutes ago   Exited (2) 2 minutes ago              web
16bea1896f0d   ubuntu            "/bin/bash"       17 hours ago    Exited (0) 20 minutes ago             amazing_buck
16101ed7b9d8   ubuntu            "/bin/bash"       17 hours ago    Exited (0) 17 hours ago               frosty_antonelli

#查看挂载信息
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker inspect --format {{.Mounts}} web
[{bind  /home/webapp /opt/webapp   true rprivate}] 

#方式二
#使用旧的-v标记可以再容器内创建一个数据卷

#指定路径挂载（ /home/webapp:/opt/webapp  ---》》》 宿主机路径:容器内路径 ）
[root@iZuf6jcqwirs6izt7krfcmZ home]# docker run -d -p 8081:8081 --name web -v /home/webapp:/opt/webapp training/webapp python app.py
996bb9c313f53f180fb572b502a250f59a831899516942bedf4e2b15ab3e0df1
[root@iZuf6jcqwirs6izt7krfcmZ home]# ll
total 63956
-rw-r--r-- 1 root root      190 May 22 19:54 dockerfile
drwx------ 2 es   es       4096 Mar 24 18:16 es
-rw------- 1 root root 65477120 May 22 20:02 ubuntu_18.04.tar
drwxr-xr-x 2 root root     4096 May 24 21:50 webapp

#匿名挂载（ /opt/webapp  ---》》》 容器内路径 ）
[root@iZuf6jcqwirs6izt7krfcmZ home]# docker run -d -p 8081:8081 --name web -v /opt/webapp training/webapp python app.py
#查看所有容器
[root@iZuf6jcqwirs6izt7krfcmZ home]# docker ps
CONTAINER ID   IMAGE               COMMAND                  CREATED         STATUS         PORTS                                                 NAMES
1605cb8a4dfc   training/webapp     "python app.py"          5 seconds ago   Up 4 seconds   5000/tcp, 0.0.0.0:8081->8081/tcp, :::8081->8081/tcp   web
aae32deafbbc   training/postgres   "su postgres -c '/us…"   25 hours ago    Up 25 hours    5432/tcp                                              db
#查看刚刚启动的容器具体信息中有一个Mounts
[root@iZuf6jcqwirs6izt7krfcmZ home]# docker inspect 1605cb8a4dfc

"Mounts": [
            {
                "Type": "volume",
                "Name": "4664d31e43adff75868c7ba397e9a0956f5289bcfb0c60c8da3d0806a437084a",
                "Source": "/var/lib/docker/volumes/4664d31e43adff75868c7ba397e9a0956f5289bcfb0c60c8da3d0806a437084a/_data",
                "Destination": "/opt/webapp",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],



#具名挂载
[root@iZuf6jcqwirs6izt7krfcmZ /]# docker run -d -p 8081:8081 --name web -v jumingguazai:/opt/webapp training/webapp python app.py
f3344dff6ba1f01f7d40550516a0a14bdf54abb4db4e829c69916407afea6a8f
#查看volumes中多了一个jumingguazai的卷
[root@iZuf6jcqwirs6izt7krfcmZ /]# ls -l /var/lib/docker/volumes/
total 72
drwx-----x 3 root root   4096 May 24 21:46 jumingguazai

#Docker挂载数据卷的默认权限是读写（rw），用户也可以通过ro指定只读
[root@iZuf6jcqwirs6izt7krfcmZ home]# docker run -d -p 8081:8081 --name web --v /home/webapp:/opt/webapp:ro training/webapp python app.py
```

### 数据卷容器

```shell
#创建数据卷容器dbdata

[root@iZuf6jcqwirs6izt7krfcmZ home]# docker run -it -v  /dbdata --name dbdata ubuntu
root@4714f0a4e207:/# ls
bin  boot  dbdata  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
#创建test目录
root@4714f0a4e207:/# cd dbdata/
root@4714f0a4e207:/dbdata# touch test

#创建db2容器挂在到dbdata下
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker run -it --volumes-from dbdata --name db2 ubuntu
root@c4fb1a5912ef:/# ls
bin  boot  dbdata  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

#此时可以看到dbdata创建的文件夹
root@c4fb1a5912ef:/# cd dbdata/
root@c4fb1a5912ef:/dbdata# ls
test

#可以多次使用--volumes-from参数来从多个容器挂载多个数据卷，还可以从其他已经挂载了容器卷的容器来挂载数据卷
[root@iZuf6jcqwirs6izt7krfcmZ home]# docker run -d --name db3 --volumes-from db1 training/postgres

#注意：如果删除了挂载的容器，数据卷并不会被自动删除。如果要删除一个数据卷，必须在删除最后一个挂载着它的容器时显示使用docker rm -v命令来指定同事删除关联的容器
```

### 利用数据卷容器来迁移数据

#### 备份

```shell
1、--name worker ubuntu					利用ubuntu创建一个名为worker的容器
2、--volumes-from dbdata  				让worker挂载到dbdata容器上
3、-v $(pwd):/backup						挂载本地的当前目录到worker容器的backup目录
4、tar cvf /backup/backup.tar /dbdata	worker容器启动后将/dbdata下的内容备份为容器内的/backup/backup.tar即宿主机当										  前目录下的backup.tar
[root@iZuf6jcqwirs6izt7krfcmZ /]# docker run --volumes-from dbdata -v $(pwd):/backup --name worker ubuntu tar cvf /backup/backup.tar /dbdata
tar: Removing leading `/' from member names
/dbdata/
/dbdata/test3
/dbdata/test2
/dbdata/tes
[root@iZuf6jcqwirs6izt7krfcmZ /]# ls
backup.tar  bin  boot  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  ubuntu-16.04-x86_64.tar.gz  usr  var
```

#### 恢复

```shell
#首先创建一个带有数据卷容器的容器dbdata2
[root@iZuf6jcqwirs6izt7krfcmZ /]# docker run -v /dndata --name dbdata2 ubuntu /bin/bash

#然后创建另外一个新的容器，挂载dbdata2的容器，并使用untar解压备份文件到所挂载的容器中
[root@iZuf6jcqwirs6izt7krfcmZ /]# docker run --volumes-from dbdata2 -v $(pwd):/backup busybox tar xvf /backup/backup.tar 
dbdata/
dbdata/test3
dbdata/test2
dbdata/test

```

## 端口映射与容器互联

### 端口映射实现容器访问

#### 从内部访问容器应用

```shell
#docker run -d -P
#-d		后台运行
#-P		Docker会随机映射一个49000~49900的端口到内部容器开放的网络（此处为大写的P）

[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker run -d -P training/webapp python app.py
3c830791e1019778bd9c268529f8cc4b16cdb17db9b983c7559f02111eb02264
#查看容器信息，可以看到training/webapp镜像生成的容器端口5000映射到宿主机的49153端口
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker ps
CONTAINER ID   IMAGE               COMMAND                  CREATED              STATUS              PORTS                                         NAMES
3c830791e101   training/webapp     "python app.py"          About a minute ago   Up About a minute   0.0.0.0:49153->5000/tcp, :::49153->5000/tcp   elegant_kilby
f5c14a54d1b0   training/postgres   "su postgres -c '/us…"   3 hours ago          Up 3 hours          5432/tcp                                      db3
c1bc508cd142   ubuntu              "/bin/bash"              3 hours ago          Up 3 hours                                                        db1
4714f0a4e207   ubuntu              "/bin/bash"              3 hours ago          Up 2 hours                                                        dbdata
#如下图所示进行访问（注意如果你用的阿里云服务器，需要去安全组打开响应的接口）
```

![image-20210523200528457](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/image-20210523200528457.png)

#### 映射所有的接口

```shell
#HostPort:ContainerPort
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker run -d -p 5000:5000 training/webapp python app.py
```

#### 映射到指定地址的指定端口

```shell
#IP:HostPort:ContainerPort
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker run -d -p 127.0.0.1:5000:5000 training/webapp python app.py
```

#### 映射到指定地址的任意端口

```shell
#IP:ContainerPort
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker run -d -p 127.0.0.1::5000 training/webapp python app.py

#还可以使用udp标记来指定udp端口
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker run -d -p 127.0.0.1:5000:5000/udp training/webapp python app.py
```

#### 查看端口映射

```shell
#docker port 

#查看该容器所有的端口映射
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker port 3c830791e101
5000/tcp -> 0.0.0.0:49153
5000/tcp -> :::49153

#查看该容器指定的端口映射
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker port 3c830791e101 5000
0.0.0.0:49153
:::49153

```

### 互联机制实现便捷访问

```shell
1、自定义容器名称
#docker run -d -P --name 容器的名称
#--name		指定容器的名称
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker run -d -P --name web training/webapp python app.py
e286d19362c3ad77c3af4b2eef0380b398664a0d3492ee2ba4c0440b1af020b8
#查看最后一次启动的容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker ps -l
CONTAINER ID   IMAGE             COMMAND           CREATED          STATUS          PORTS                                         NAMES
e286d19362c3   training/webapp   "python app.py"   33 seconds ago   Up 32 seconds   0.0.0.0:49154->5000/tcp, :::49154->5000/tcp   web

#使用inspect查看容器的名称
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker inspect -f {{'.Name'}} e286d19362c3
/web

2、容器互联
#创建一个数据库容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker run -d --name db training/postgres
8344a5653e905f95629a11c549170038a0621416eaa5cb9bdc2b5a719bb65ccd

#删除之前创建的web容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker rm -f web
web

#创建一个新的web容器并且连接数据库容器
#--link name:alias    其中name是要连接的容器名称，alias是别名
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker run -d -P --name web --link db:db training/webapp python app.py
cc1a27f7c41239f22ec359175788a296e4d2b38d1f51037a46283854bd3707a0

#可以看到db的names列有web/db，表示web容器可以访问db容器了
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker ps --no-trunc
CONTAINER ID                                                       IMAGE               COMMAND                                                                                                                                                                    CREATED              STATUS              PORTS                                         NAMES
9bcfcbb818cc65c1e9d51bf1709c155d06abdb43ce80467b924a39746fd36a50   training/webapp     "python app.py"                                                                                                                                                            About a minute ago   Up About a minute   0.0.0.0:49157->5000/tcp, :::49157->5000/tcp   web
aae32deafbbceb6554fbf0ed50e3dc81264c24bd92cb3689759e05a5805d008b   training/postgres   "su postgres -c '/usr/lib/postgresql/$PG_VERSION/bin/postgres -D /var/lib/postgresql/$PG_VERSION/main/ -c config_file=/etc/postgresql/$PG_VERSION/main/postgresql.conf'"   About a minute ago   Up About a minute   5432/tcp                                      db,web/db
```

## 使用Dockerfile创建镜像

### 常用指令及说明

<table>
	<tr>
	    <th>分类</th>
	    <th>指令</th>
	    <th>说明</th>  
	</tr >
	<tr >
	    <td rowspan="13">配置指令</td>
	    <td>ARG</td>
	    <td>定义创建镜像过程中使用的变量</td>
	</tr>
	<tr>
	    <td>FROM</td>
	    <td>指定创建镜像的基础镜像</td>
	</tr>
    <tr>
	    <td>LABLE</td>
	    <td>为生成的镜像添加数据标签信息</td>
	</tr>
    <tr>
	    <td>EXPOSE</td>
	    <td>声明镜像内服务监听的端口</td>
	</tr>
	<tr>
	    <td>ENV</td>
	    <td>指定环境变量</td>
	</tr>
	<tr>
	    <td>ENTRYPOINT</td>
	    <td>指定镜像的默认入口命令</td>
	</tr>
	<tr><td>VOLUME</td>
	    <td>创建一个数据点挂载点</td>
	</tr>
	<tr>
	    <td>USER</td>
	    <td>指定容器运行时的用户名或UID</td>
	</tr>
	<tr>
	    <td>WORKDIR</td>
	    <td>配置工作目录</td>
	</tr>
    <tr>
	    <td>ONBUILD</td>
	    <td>创建子镜像时指定自动执行的操作指令</td>
	</tr>
    <tr>
	    <td>STOPSIGNAL</td>
	    <td>指定退出新号</td>
	</tr>
	<tr>
	    <td>HEALTHCHECK</td>
	    <td>配置所启动容器时指定自动执行的操作指令</td>
	</tr>
	<tr>
	    <td >SHELL</td>
	    <td>指定默认shell类型</td>
	</tr>
	<tr>
	    <td rowspan="4">操作指令</td>
	    <td>RUN</td>
	    <td>运行指定命令</td>
	</tr>
	<tr>
	    <td >CMD</td>
	    <td >启动容器时指定默认执行命令</td>
	</tr>
	<tr>
	    <td >ADD</td>
	    <td >添加内容到镜像</td>
	</tr>
	<tr>
	    <td >COPY</td>
	    <td >复制内容到镜像</td>
	</tr>
</table>

### 指令使用说明

#### ARG 构建参数

**定义**

​	定义创建镜像的过程使用的变量

**格式**

```shell
ARG <name>[=<default value>]
```

**说明**

​	`Dockerfile` 中的 `ARG` 指令是定义参数名称，以及定义其默认值。在docker build创建镜像的时候，使用 `--build-arg <varname>=<value>`来为变量赋值

**案例**

​	ARG 指令有生效范围，如果在 `FROM` 指令之前指定，那么只能用于 `FROM` 指令中。

```shell
ARG DOCKER_USERNAME=library

FROM ${DOCKER_USERNAME}/alpine

RUN set -x ; echo ${DOCKER_USERNAME}
```

​	使用上述 Dockerfile 会发现无法输出 `${DOCKER_USERNAME}` 变量的值，要想正常输出，你必须在 `FROM` 之后再次指定 `ARG`

```shell
# 只在 FROM 中生效
ARG DOCKER_USERNAME=library

FROM ${DOCKER_USERNAME}/alpine

# 要想在 FROM 之后使用，必须再次指定
ARG DOCKER_USERNAME=library

RUN set -x ; echo ${DOCKER_USERNAME}
```

​	对于多阶段构建，尤其要注意这个问题

```shell
# 这个变量在每个 FROM 中都生效
ARG DOCKER_USERNAME=library

FROM ${DOCKER_USERNAME}/alpine

RUN set -x ; echo 1

FROM ${DOCKER_USERNAME}/alpine

RUN set -x ; echo 2
```

​	对于上述 Dockerfile 两个 `FROM` 指令都可以使用 `${DOCKER_USERNAME}`，对于在各个阶段中使用的变量都必须在每个阶段分别指定：

```shell
ARG DOCKER_USERNAME=library

FROM ${DOCKER_USERNAME}/alpine

# 在FROM 之后使用变量，必须在每个阶段分别指定
ARG DOCKER_USERNAME=library

RUN set -x ; echo ${DOCKER_USERNAME}

FROM ${DOCKER_USERNAME}/alpine

# 在FROM 之后使用变量，必须在每个阶段分别指定
ARG DOCKER_USERNAME=library

RUN set -x ; echo ${DOCKER_USERNAME}
```

#### FROM 基础镜像

**定义**

​	功能为指定基础镜像，并且必须是第一条指令

**格式**

```shell
格式：FROM <image>
	 FROM <image>:<tag>
	 FROM <image>:<digest> 
```

**说明**

​	如果在同一个Dockerfile中创建多个镜像时，可以使用多个FROM指令

#### LABEL 

**定义**

​	为生成的镜像添加标签

**格式**

```shell
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```

**说明**

​	LABEL会继承基础镜像中的LABEL，如遇到key相同，则值覆盖

**案例**

​	我们还可以用一些标签来申明镜像的作者、文档地址等：

```shell
LABEL org.opencontainers.image.authors="yeasy"

LABEL org.opencontainers.image.documentation="https://yeasy.gitbooks.io"
```

#### EXPOSE 声明端口

**定义**

​	暴漏容器运行时的监听端口给外部

**格式**

```shell
格式：EXPOSE <port> [<port>/<protocol>...]
列如：EXPOSE 22 80 8888
```

**说明**

​	该指令只是起到声明的作用，并不会自动完成端口映射 

**优点**

​	一个是帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射；

​	另一个用处则是在运行时使用随机端口映射时，也就是 `docker run -P` 时，会自动随机映射 `EXPOSE` 的端口

**注意**

​	要将 `EXPOSE` 和在运行时使用 `-p <宿主端口>:<容器端口>` 区分开来。`-p`，是映射宿主端口和容器端口，换句话说，就是将容器的对应端口服务公开给外界访问，而 `EXPOSE` 仅仅是声明容器打算使用什么端口而已，并不会自动在宿主进行端口映射。

#### ENV 设置环境变量

**定义**

​	指定环境变量

**格式**

```shell
格式：ENV <key> <value>
   	 ENV <key>=<value> ...
列如：ENV APP_VERSION=1.0.0
	 ENV APP_HOME=/use/local/app
	 ENV PATH $PATH:/usr/local/bin
```

**说明**

​	1、镜像生成的过程中会被后续RUN指令使用
​	2 、在镜像启动过程中也会被使用 `docker run --env <key>=<value> centos`
​	 3、当一条ENV指令同时为多个环境变量赋值并且值也是从环境变量读取，会为变量都赋值后在更新
​	   			```ENV key1=value2  ```
​	   			```ENV key1=value1 key2=${key1}```
​	   		最终的结果为 `key1=value1  key2=value2`

**案例**

​	官方 `node` 镜像 `Dockerfile` 中，就有类似这样的代码：

```shell
ENV NODE_VERSION 7.2.0

RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" \
  && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
  && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
  && grep " node-v$NODE_VERSION-linux-x64.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
  && tar -xJf "node-v$NODE_VERSION-linux-x64.tar.xz" -C /usr/local --strip-components=1 \
  && rm "node-v$NODE_VERSION-linux-x64.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt \
  && ln -s /usr/local/bin/node /usr/local/bin/nodejs
```

​	在这里先定义了环境变量 `NODE_VERSION`，其后的 `RUN` 这层里，多次使用 `$NODE_VERSION` 来进行操作定制。可以看到，将来升级镜像构建版本的时候，只需要更新 `7.2.0` 即可，`Dockerfile` 构建维护变得更轻松了。

#### ENTRYPOINT 入口点

**定义**

​	指定镜像的默认入口命令，该命令会在启动容器时作为根命令执行

**格式**

```shell
格式：ENTRYPOINT ["executable", "param1", "param2"]  exec调用执行
     ENTRYPOINT command param1 param2				shell中执行
```

**说明**

​	1、`ENTRYPOINT` 的格式和 `RUN` 指令格式一样，分为 `exec` 格式和 `shell` 格式。

​	2、每个Dockerfile只能有一个ENTRYPOINT，当指定多个时，只有最后一个起效
​	3、ENTRYPOINT指令指定的命令能不能被覆盖，取决于`--entrypoint`：

​	  `ENTRYPOINT` 在运行时不可以替代，将docker run指定的参数加到ENTRYPOINT指定命令的参数	

```shell
docker run -i 
```

​      `ENTRYPOINT` 在运行时也可以替代，需要通过 `docker run` 的参数 `--entrypoint` 来指定。

```shell
docker run --entrypoint
```

​	4、当指定了 `ENTRYPOINT` 后，`CMD` 的含义就发生了改变，不再是直接的运行其命令，而是将 `CMD` 的内容作为参数传给 `ENTRYPOINT` 指令，换句话说实际执行时

```shell
<ENTRYPOINT> "<CMD>"
```

**案例**

 场景一：让镜像变成像命令一样使用

​	假设我们需要一个得知自己当前公网 IP 的镜像，那么可以先用 `CMD` 来实现：

```shell
[root@iZuf6jcqwirs6izt7krfcmZ docker_file]# cat Dockerfile 
FROM ubuntu:18.04
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
CMD ["curl","-s","http://myip.ipip.net"]
```

​	通过Dockerfile我们来创建一个myip镜像

```shell
[root@iZuf6jcqwirs6izt7krfcmZ docker_file]# docker build -t myip .
```

​	根据这个镜像我们运行一个容器

```shell
[root@iZuf6jcqwirs6izt7krfcmZ docker_file]# docker run myip 
当前 IP：139.196.216.183  来自于：中国 上海 上海  阿里云/电信/联通/移动/教育网
```

​	嗯，这么看起来好像可以直接把镜像当做命令使用了，不过命令总有参数，如果我们希望加参数呢？比如从上面的 `CMD` 中可以看到实质的命令是 `curl`，那么如果我们希望显示 HTTP 头信息，就需要加上 `-i` 参数。那么我们可以直接加 `-i` 参数给 `docker run myip` 么？

```shell
[root@iZuf6jcqwirs6izt7krfcmZ docker_file]# docker run myip -i
docker: Error response from daemon: OCI runtime create failed: container_linux.go:367: starting container process caused: exec: "-i": executable file not found in $PATH: unknown.
```

​	我们可以看到可执行文件找不到的报错，`executable file not found`。之前我们说过，跟在镜像名后面的是 `command`，运行时会替换 `CMD` 的默认值。因此这里的 `-i` 替换了原来的 `CMD`，而不是添加在原来的 `curl -s http://myip.ipip.net` 后面。而 `-i` 根本不是命令，所以自然找不到。

​	那么如果我们希望加入 `-i` 这参数，我们就必须重新完整的输入这个命令：

```shell
[root@iZuf6jcqwirs6izt7krfcmZ docker_file]# docker run myip curl -s http://myip.ipip.net -i
```

​	这显然不是很好的解决方案，而使用 `ENTRYPOINT` 就可以解决这个问题。现在我们重新用 `ENTRYPOINT` 来实现这个镜像：

```shell
[root@iZuf6jcqwirs6izt7krfcmZ docker_file]# cat Dockerfile 
FROM ubuntu:18.04
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
ENTRYPOINT ["curl","-s","http://myip.ipip.net"]
```

​	这次我们再来尝试直接使用 `docker run myip -i`：

```shell
[root@iZuf6jcqwirs6izt7krfcmZ docker_file]# docker run myip
当前 IP：139.196.216.183  来自于：中国 上海 上海  阿里云/电信/联通/移动/教育网

[root@iZuf6jcqwirs6izt7krfcmZ docker_file]# docker run myip -i
HTTP/1.1 200 OK
Date: Wed, 26 May 2021 08:47:34 GMT
Content-Type: text/plain; charset=utf-8
Content-Length: 104
Connection: keep-alive
X-Via-JSL: 1d9bd9a,-
Set-Cookie: __jsluid_h=baa792983939457865a4eec714b9e066; max-age=31536000; path=/; HttpOnly
X-Cache: bypass

当前 IP：139.196.216.183  来自于：中国 上海 上海  阿里云/电信/联通/移动/教育网
```

​	可以看到，这次成功了。这是因为当存在 `ENTRYPOINT` 后，`CMD` 的内容将会作为参数传给 `ENTRYPOINT`，而这里 `-i` 就是新的 `CMD`，因此会作为参数传给 `curl`，从而达到了我们预期的效果。

 场景二：应用运行前的准备工作

​	启动容器就是启动主进程，但有些时候，启动主进程前，需要一些准备工作。

​	比如 `mysql` 类的数据库，可能需要一些数据库配置、初始化的工作，这些工作要在最终的 mysql 服务器运行之前解决。

​	此外，可能希望避免使用 `root` 用户去启动服务，从而提高安全性，而在启动服务前还需要以 `root` 身份执行一些必要的准备工作，最后切换到服务用户身份启动服务。或者除了服务外，其它命令依旧可以使用 `root` 身份执行，方便调试等。

​	这些准备工作是和容器 `CMD` 无关的，无论 `CMD` 为什么，都需要事先进行一个预处理的工作。这种情况下，可以写一个脚本，然后放入 `ENTRYPOINT` 中去执行，而这个脚本会将接到的参数（也就是 `<CMD>`）作为命令，在脚本最后执行。比如官方镜像 `redis` 中就是这么做的：

```shell
FROM alpine:3.4
...
RUN addgroup -S redis && adduser -S -G redis redis
...
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 6379
CMD [ "redis-server" ]
```

​	可以看到其中为了 redis 服务创建了 redis 用户，并在最后指定了 `ENTRYPOINT` 为 `docker-entrypoint.sh` 脚本。

```shell
#!/bin/sh
...
# allow the container to be started with `--user`
if [ "$1" = 'redis-server' -a "$(id -u)" = '0' ]; then
	find . \! -user redis -exec chown redis '{}' +
	exec gosu redis "$0" "$@"
fi

exec "$@"
```

​	该脚本的内容就是根据 `CMD` 的内容来判断，如果是 `redis-server` 的话，则切换到 `redis` 用户身份启动服务器，否则依旧使用 `root` 身份执行。比如：

```shell
$ docker run -it redis id
uid=0(root) gid=0(root) groups=0(root)
```

#### VOLUME 定义匿名卷

**定义**

​	创建一个数据卷挂载点

**格式**

```shell
VOLUME ["<路径1>", "<路径2>"...]
VOLUME <路径>
例如
	VOLUME /data
```

​	这里的 `/data` 目录就会在容器运行时自动挂载为匿名卷，任何向 `/data` 中写入的信息都不会记录进容器存储层，从而保证了容器存储层的无状态化。当然，运行容器时可以覆盖这个挂载设置。比如：

```shell
$ docker run -d -v mydata:/data xxxx
```

​	在这行命令中，就使用了 `mydata` 这个命名卷挂载到了 `/data` 这个位置，替代了 `Dockerfile` 中定义的匿名卷的挂载配置。

**说明**

​	1、`["/data"]`可以是一个JsonArray ，也可以是多个值。所以如下几种写法都是正确的

```shell
VOLUME ["/var/log/"]
VOLUME /var/log
VOLUME /var/log /var/db
```

​	2、一般使用场景为需要持久化存储数据时，容器使用的是AUFS，这种文件系统不能持久化数据，当容器关闭后，所有的更改都会丢失

#### USER

**定义**

​	设置启动容器的用户，可以是用户名或UID

**格式**

```shell
USER daemo
USER UID
```

**说明**

​	当服务不需要管理员权限时，可以通过该命令指定运行用户，并且可以再Dockerfile中创建所需要的用户

```shell
RUN groupadd -r postgres && useradd --no--log--init -r -g postgres postgres
```

​	我们可以看到Redis的镜像中添加我们的用户和组，以确保他们的id被一致分配，不管添加了什么依赖

![](E:\JavaQaaQ\JavaQaaQ\images\9763561458243b146d6a177b6c8b1b4.png)

#### WORKDIR 指定工作目录

**定义**

​	使用 `WORKDIR` 指令可以来指定工作目录（或者称为当前目录），以后各层的当前目录就被改为指定的目录，如该目录不存在，`WORKDIR` 会帮你建立目录。

**格式**

```shell
WORKDIR /path/to/workdir
```

**误区**

​	在Dockerfile中添加以下两行命令

```shell
RUN cd /app
RUN echo "hello" > world.txt
```

​	如果将这个 `Dockerfile` 进行构建镜像运行后，会发现找不到 `/app/world.txt` 文件，或者其内容不是 `hello`。原因其实很简单，在 Shell 中，连续两行是同一个进程执行环境，因此前一个命令修改的内存状态，会直接影响后一个命令；而在 `Dockerfile` 中，这两行 `RUN` 命令的执行环境根本不同，是两个完全不同的容器。这就是对 `Dockerfile` 构建分层存储的概念不了解所导致的错误。

​	之前说过每一个 `RUN` 都是启动一个容器、执行命令、然后提交存储层文件变更。第一层 `RUN cd /app` 的执行仅仅是当前进程的工作目录变更，一个内存上的变化而已，其结果不会造成任何文件变更。而到第二层的时候，启动的是一个全新的容器，跟第一层的容器更完全没关系，自然不可能继承前一层构建过程中的内存变化。

**说明**

​	1、如果需要改变以后各层的工作目录的位置，那么应该使用 `WORKDIR` 指令

```shell
WORKDIR /app

RUN echo "hello" > world.txt
```

​	2、如果你的 `WORKDIR` 指令使用的相对路径，那么所切换的路径与之前的 `WORKDIR` 有关

```shell
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```

​	pwd执行的结果是/a/b/c

​	3、WORKDIR也可以解析环境变量

```shell
ENV DIRPATH /path
WORKDIR $DIRPATH/$DIRNAME
RUN pwd
```

​	pwd的执行结果是/path/$DIRNAME

#### ONBUILD 为他人做嫁衣裳

**定义**

是一个特殊的指令，它后面跟的是其它指令，比如 RUN, COPY 等，而这些指令，在当前镜像构建时并不会被执行。只有当以当前镜像为基础镜像，去构建下一级镜像的时候才会被执行

**格式**

```shell
ONBUILD <其它指令>
```

**说明**

​	1、Dockerfile 中的其它指令都是为了定制当前镜像而准备的，唯有 ONBUILD 是为了帮助别人定制自己而准备的
​	2、当我们编写一个新的Dockerfile文件来基于A镜像构建一个镜像（比如为B镜像）时，这时构造A镜像的Dockerfile文件中的ONBUILD指令就生效了，在构建B镜像的过程中，首先会执行ONBUILD指令指定的指令，然后才会执行其它指令。

**案列**

​	假设我们要制作 Node.js 所写的应用的镜像。我们都知道 Node.js 使用 `npm` 进行包管理，所有依赖、配置、启动信息等会放到 `package.json` 文件里。在拿到程序代码后，需要先进行 `npm install` 才可以获得所有需要的依赖。然后就可以通过 `npm start` 来启动应用。因此，一般来说会这样写 `Dockerfile`：

```shell
FROM node:slim
RUN mkdir /app
WORKDIR /app
COPY ./package.json /app
RUN [ "npm", "install" ]
COPY . /app/
CMD [ "npm", "start" ]
```

​	把这个 `Dockerfile` 放到 Node.js 项目的根目录，构建好镜像后，就可以直接拿来启动容器运行。但是如果我们还有第二个 Node.js 项目也差不多呢？好吧，那就再把这个 `Dockerfile` 复制到第二个项目里。那如果有第三个项目呢？再复制么？文件的副本越多，版本控制就越困难，让我们继续看这样的场景维护的问题。

​	如果第一个 Node.js 项目在开发过程中，发现这个 `Dockerfile` 里存在问题，比如敲错字了、或者需要安装额外的包，然后开发人员修复了这个 `Dockerfile`，再次构建，问题解决。第一个项目没问题了，但是第二个项目呢？虽然最初 `Dockerfile` 是复制、粘贴自第一个项目的，但是并不会因为第一个项目修复了他们的 `Dockerfile`，而第二个项目的 `Dockerfile` 就会被自动修复。

​	那么我们可不可以做一个基础镜像，然后各个项目使用这个基础镜像呢？这样基础镜像更新，各个项目不用同步 `Dockerfile` 的变化，重新构建后就继承了基础镜像的更新？好吧，可以，让我们看看这样的结果。那么上面的这个 `Dockerfile` 就会变为：

```shell
FROM node:slim
RUN mkdir /app
WORKDIR /app
CMD [ "npm", "start" ]
```

​	这里我们把项目相关的构建指令拿出来，放到子项目里去。假设这个基础镜像的名字为 `my-node` 的话，各个项目内的自己的 `Dockerfile` 就变为：

```shell
FROM my-node
COPY ./package.json /app
RUN [ "npm", "install" ]
COPY . /app/
```

​	基础镜像变化后，各个项目都用这个 `Dockerfile` 重新构建镜像，会继承基础镜像的更新。

​	那么，问题解决了么？没有。准确说，只解决了一半。如果这个 `Dockerfile` 里面有些东西需要调整呢？比如 `npm install` 都需要加一些参数，那怎么办？这一行 `RUN` 是不可能放入基础镜像的，因为涉及到了当前项目的 `./package.json`，难道又要一个个修改么？所以说，这样制作基础镜像，只解决了原来的 `Dockerfile` 的前4条指令的变化问题，而后面三条指令的变化则完全没办法处理。

`	ONBUILD` 可以解决这个问题。让我们用 `ONBUILD` 重新写一下基础镜像的 `Dockerfile`:

```shell
FROM node:slim
RUN mkdir /app
WORKDIR /app
ONBUILD COPY ./package.json /app
ONBUILD RUN [ "npm", "install" ]
ONBUILD COPY . /app/
CMD [ "npm", "start" ]
```

​	这次我们回到原始的 `Dockerfile`，但是这次将项目相关的指令加上 `ONBUILD`，这样在构建基础镜像的时候，这三行并不会被执行。然后各个项目的 `Dockerfile` 就变成了简单地：

```shell
FROM my-node
```

​	是的，只有这么一行。当在各个项目目录中，用这个只有一行的 `Dockerfile` 构建镜像时，之前基础镜像的那三行 `ONBUILD` 就会开始执行，成功的将当前项目的代码复制进镜像、并且针对本项目执行 `npm install`，生成应用镜像。

#### HEALTHCHECK 健康检查

**定义**

​	指令是告诉 Docker 应该如何进行判断容器的状态是否正常

**格式**

```shell
HEALTHCHECK [选项] CMD <命令>：设置检查容器健康状况的命令
HEALTHCHECK NONE：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令
```

**HEALTHCHECK 支持的选项**

```shell
--interval=<间隔>：两次健康检查的间隔，默认为 30 秒；
--timeout=<时长>：健康检查命令运行超时时间，如果超过这个时间，本次健康检查就被视为失败，默认 30 秒；
--retries=<次数>：当连续失败指定次数后，则将容器状态视为 unhealthy，默认 3 次。
```

**状态**

​	当在一个镜像指定了 `HEALTHCHECK` 指令后，用其启动容器，初始状态会为 `starting`，在 `HEALTHCHECK` 指令检查成功后变为 `healthy`，如果连续一定次数失败，则会变为 `unhealthy`

**案列**

​	假设我们有个镜像是个最简单的 Web 服务，我们希望增加健康检查来判断其 Web 服务是否在正常工作，我们可以用 `curl` 来帮助判断，其 `Dockerfile` 的 `HEALTHCHECK` 可以这么写：

```shell
FROM nginx
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
HEALTHCHECK --interval=5s --timeout=3s CMD curl -fs http://localhost/ || exit 1
```

​	这里我们设置了每 5 秒检查一次（这里为了试验所以间隔非常短，实际应该相对较长），如果健康检查命令超过 3 秒没响应就视为失败，并且使用 `curl -fs http://localhost/ || exit 1` 作为健康检查命令。

​	使用 `docker build` 来构建这个镜像：

```shell
$ docker build -t myweb:v1 .
```

​	构建好了后，我们启动一个容器：

```shell
$ docker run -d --name web -p 80:80 myweb:v1
```

​	当运行该镜像后，可以通过 `docker container ls` 看到最初的状态为 `(health: starting)`：

```shell
$ docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                            PORTS               NAMES
03e28eb00bd0        myweb:v1            "nginx -g 'daemon off"   3 seconds ago       Up 2 seconds (health: starting)   80/tcp, 443/tcp     web
```

​	在等待几秒钟后，再次 `docker container ls`，就会看到健康状态变化为了 `(healthy)`：

```shell
$ docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                    PORTS               NAMES
03e28eb00bd0        myweb:v1            "nginx -g 'daemon off"   18 seconds ago      Up 16 seconds (healthy)   80/tcp, 443/tcp     web
```

#### RUN

**定义**

​	运行指定指令

**格式**

```shell
RUN <command>
RUN ["executable","param1","param2"]
```

​	注意：后者指令会被解析为JSON数组，因此必须使用双引号。前者默认将在shell终端运行命令，即/bin/sh -c；后者则使用exec执行

**说明**

​	1、每条RUN指令将在当前镜像基础上通过第二种方式实现，并提交为新的镜像层

#### CMD 

**定义**

用来指定启动容器时默认执行的命令

**格式**

```shell
CMD ["executable","param1","param2"]  相当于执行 executable  param1 param2
CMD command param1 param2  			  默认的shell中执行，提供给需要交互的应用
CMD ["param1","param2"]				  提供给ENTRYPOINT的参数
```

**说明**

​	1、每个Dockerfile文件只能有一个CMD命令，如果指定了多条CMD命令只会执行最后一条

```shell
FROM ubuntu:18.04
CMD ["ls","-al"]
CMD ["echo","hello"]
```

```shell
[root@iZuf6jcqwirs6izt7krfcmZ docker_file]# cat Dockerfile 
FROM ubuntu:18.04
CMD ["ls","-al"]
CMD ["echo","hello"]
[root@iZuf6jcqwirs6izt7krfcmZ docker_file]# docker build -t mycmd:1.0 .		构造镜像（如果不通过-f指定文件，默认当前目录下的名称为Dockerfile的文件）
[root@iZuf6jcqwirs6izt7krfcmZ docker_file]# docker run c7a36110a603			运行容器
hello
```

​	2、如果用户启动容器时候手动指定运行的命令（作为润命令的参数），则会覆盖掉CMD指令

编写的Dockerfile文件

```shell
FROM ubuntu:18.04
CMD ["ls","-al"]
```

```shell
[root@iZuf6jcqwirs6izt7krfcmZ docker_file]# docker build -t cmdimages:1.0 构造镜像（如果不通过-f指定文件，默认当前目录下的名称为Dockerfile的文件）
[root@iZuf6jcqwirs6izt7krfcmZ docker_file]# docker run 6986db2d5708 /bin/echo 'Hello Word' 运行容器
Hello Word
```

**案列**

​	编写Dockerfile文件，包含以下指令

```shell
[root@iZuf6jcqwirs6izt7krfcmZ docker_file]# cat Dockerfile 
FROM ubuntu:18.04
CMD ["ls","-al"]
```

​	构建镜像

```shell
[root@iZuf6jcqwirs6izt7krfcmZ docker_file]# docker build -t  cmdimages:1.0 .
Sending build context to Docker daemon  4.096kB
Step 1/2 : FROM ubuntu:18.04
 ---> 81bcf752ac3d
Step 2/2 : CMD ["ls","-al"]
 ---> Running in 13710b105056
Removing intermediate container 13710b105056
 ---> eeaeeb973b1a
Successfully built eeaeeb973b1a
Successfully tagged cmdimages:1.0
```

运行容器，可以看到容器运行后执行了`ls -a`

```shell
[root@iZuf6jcqwirs6izt7krfcmZ docker_file]# docker images
REPOSITORY            TAG            IMAGE ID       CREATED         SIZE
cmdimages             1.0            eeaeeb973b1a   8 seconds ago   63.1MB
[root@iZuf6jcqwirs6izt7krfcmZ docker_file]# docker run eeaeeb973b1a
total 72
drwxr-xr-x  1 root root 4096 May 26 14:09 .
drwxr-xr-x  1 root root 4096 May 26 14:09 ..
-rwxr-xr-x  1 root root    0 May 26 14:09 .dockerenv
drwxr-xr-x  2 root root 4096 May 12 23:09 bin
drwxr-xr-x  2 root root 4096 Apr 24  2018 boot
drwxr-xr-x  5 root root  340 May 26 14:09 dev
drwxr-xr-x  1 root root 4096 May 26 14:09 etc
drwxr-xr-x  2 root root 4096 Apr 24  2018 home
drwxr-xr-x  8 root root 4096 May 23  2017 lib
drwxr-xr-x  2 root root 4096 May 12 23:08 lib64
drwxr-xr-x  2 root root 4096 May 12 23:05 media
drwxr-xr-x  2 root root 4096 May 12 23:05 mnt
drwxr-xr-x  2 root root 4096 May 12 23:05 opt
dr-xr-xr-x 99 root root    0 May 26 14:09 proc
drwx------  2 root root 4096 May 12 23:09 root
drwxr-xr-x  1 root root 4096 May 19 19:44 run
drwxr-xr-x  1 root root 4096 May 19 19:44 sbin
drwxr-xr-x  2 root root 4096 May 12 23:05 srv
dr-xr-xr-x 13 root root    0 May 22 10:57 sys
drwxrwxrwt  2 root root 4096 May 12 23:09 tmp
drwxr-xr-x  1 root root 4096 May 12 23:05 usr
drwxr-xr-x  1 root root 4096 May 12 23:09 var
```

#### ADD

**定义**

​	添加内容到镜像

**格式**

```shell
ADD <SRC> <dest>
```

​	将复制<SRC> 路径下的内容到容器中的<dest>路径下

**说明**

​	1、其中`<SRC>` 可以使Dockerfile所在目录的一个相对路径（文件或目录）；也可以是一个URL；还可以是一个tar文件（）自动解压为目录。   ` <dest>`可以使镜像内绝对路径，或者相对于工作目录（WOOKDIR）的相对路径。

2、路径支持正则表达式

```shell
ADD *.c /code/
```

**案列**

```shell
#构建镜像文件
[root@iZuf6jcqwirs6izt7krfcmZ docker_file]# cat Dockerfile 
FROM ubuntu:18.04
ADD ubuntu_18.04.tar /usr/local/
CMD ["ls","/usr/local/"]
#构建镜像
[root@iZuf6jcqwirs6izt7krfcmZ docker_file]# docker build -t  myadd:1.0 .
[root@iZuf6jcqwirs6izt7krfcmZ docker_file]# docker images
REPOSITORY            TAG            IMAGE ID       CREATED         SIZE
myadd                 1.0            3dcabe44a365   7 seconds ago   129MB
#运行容器，发现当前目录下的ubuntu_18.04.tar添加到容器中的/usr/local/并且解压了
[root@iZuf6jcqwirs6izt7krfcmZ docker_file]# docker run 3dcabe44a365
0a7c55f1a00564ffe36bfd4bf649b408bed682dd92c41679e43545b71b06a2f8
631934f4c1338cbb0e84ae53d6eca14c98deefee23247488719054a6c07c87b1
81bcf752ac3dc8a12d54908ecdfe98a857c84285e5d50bed1d10f9812377abd6.json
a7288c5e73f896a7b80409dd46f63c4a3c697da917714f656216fe668e58e873
bin
etc
games
include
lib
man
manifest.json
repositories
sbin
share
src
```

#### COPY

**定义**

复制内容到镜像

**格式**

```shell
COPY <src> dest<>
```

**说明**

复制本地主机的`<src>`（为Dockerfile所在目录的相对路径，文件或目录）下的内容到镜像中的 `<dest>`。目标路径不存在，会自动创建

### 创建镜像

**基本格式**

```shell
docker build -t 镜像标签 -f 编写的Dockerfile文件路径
docker build -t builder/first_images:1.0 /tmp/docker_builder/
```

**创建命令的选项及说明**

```shell
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker build --help
Usage:  docker build [OPTIONS] PATH | URL | -
Build an image from a Dockerfile
	  --add-host list           Add a custom host-to-IP mapping (host:ip)	添加自定义的主机名到IP的映射
      --build-arg list          Set build-time variables					添加创建时的变量
      --cache-from strings      Images to consider as cache sources			使用指定镜像作为缓存源
      --cgroup-parent string    Optional parent cgroup for the container	继承上层的cgroup
      --compress                Compress the build context using gzip
      --cpu-period int          Limit the CPU CFS (Completely Fair Scheduler) period
      --cpu-quota int           Limit the CPU CFS (Completely Fair Scheduler) quota
  -c, --cpu-shares int          CPU shares (relative weight)
      --cpuset-cpus string      CPUs in which to allow execution (0-3, 0,1)
      --cpuset-mems string      MEMs in which to allow execution (0-3, 0,1)
      --disable-content-trust   Skip image verification (default true)
  -f, --file string             Name of the Dockerfile (Default is 'PATH/Dockerfile')
      --force-rm                Always remove intermediate containers
      --iidfile string          Write the image ID to the file
      --isolation string        Container isolation technology
      --label list              Set metadata for an image
  -m, --memory bytes            Memory limit
      --memory-swap bytes       Swap limit equal to memory plus swap: '-1' to enable unlimited swap
      --network string          Set the networking mode for the RUN instructions during build (default "default")
      --no-cache                Do not use cache when building the image
      --pull                    Always attempt to pull a newer version of the image
  -q, --quiet                   Suppress the build output and print image ID on success
      --rm                      Remove intermediate containers after a successful build (default true)
      --security-opt strings    Security options
      --shm-size bytes          Size of /dev/shm
  -t, --tag list                Name and optionally a tag in the 'name:tag' format
      --target string           Set the target build stage to build.
      --ulimit ulimit           Ulimit options (default [])

```

##### **多步创建**

 之前的做法,在 Docker 17.05 版本之前，我们构建 Docker 镜像时，通常会采用两种方式：

1、全部放入一个-dockerfile

一种方式是将所有的构建过程编包含在一个 `Dockerfile` 中，包括项目及其依赖库的编译、测试、打包等流程，这里可能会带来的一些问题：

- 镜像层次多，镜像体积较大，部署时间变长
- 源代码存在泄露的风险

例如，编写 `app.go` 文件，该程序输出 `Hello World!`

```go
package main

import "fmt"

func main(){
    fmt.Printf("Hello World!");
}
```

编写 `Dockerfile.one` 文件

```shell
FROM golang:alpine

RUN apk --no-cache add git ca-certificates

WORKDIR /go/src/github.com/go/helloworld/

COPY app.go .

RUN go get -d -v github.com/go-sql-driver/mysql \
  && CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app . \
  && cp /go/src/github.com/go/helloworld/app /root

WORKDIR /root/

CMD ["./app"]
```

构建镜像

```shell
$ docker build -t go/helloworld:1 -f Dockerfile.one .
```

2、分散到多个 Dockerfile

另一种方式，就是我们事先在一个 `Dockerfile` 将项目及其依赖库编译测试打包好后，再将其拷贝到运行环境中，这种方式需要我们编写两个 `Dockerfile` 和一些编译脚本才能将其两个阶段自动整合起来，这种方式虽然可以很好地规避第一种方式存在的风险，但明显部署过程较复杂。

例如，编写 `Dockerfile.build` 文件

```shell
FROM golang:alpine

RUN apk --no-cache add git

WORKDIR /go/src/github.com/go/helloworld

COPY app.go .

RUN go get -d -v github.com/go-sql-driver/mysql \
  && CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .
```

编写 `Dockerfile.copy` 文件

```shell
FROM alpine:latest

RUN apk --no-cache add ca-certificates

WORKDIR /root/

COPY app .

CMD ["./app"]
```

新建 `build.sh`

```shell
#!/bin/sh
echo Building go/helloworld:build

docker build -t go/helloworld:build . -f Dockerfile.build

docker create --name extract go/helloworld:build
docker cp extract:/go/src/github.com/go/helloworld/app ./app
docker rm -f extract

echo Building go/helloworld:2

docker build --no-cache -t go/helloworld:2 . -f Dockerfile.copy
rm ./app
```

现在运行脚本即可构建镜像

```shell
$ chmod +x build.sh

$ ./build.sh
```

对比两种方式生成的镜像大小

```shell
$ docker image ls

REPOSITORY      TAG    IMAGE ID        CREATED         SIZE
go/helloworld   2      f7cf3465432c    22 seconds ago  6.47MB
go/helloworld   1      f55d3e16affc    2 minutes ago   295MB
```

2、使用多阶段构建

为解决以上问题，Docker v17.05 开始支持多阶段构建 (`multistage builds`)。使用多阶段构建我们就可以很容易解决前面提到的问题，并且只需要编写一个 `Dockerfile`：

例如，编写 `Dockerfile` 文件

```shell
FROM golang:alpine as builder

RUN apk --no-cache add git

WORKDIR /go/src/github.com/go/helloworld/

RUN go get -d -v github.com/go-sql-driver/mysql

COPY app.go .

RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

FROM alpine:latest as prod

RUN apk --no-cache add ca-certificates

WORKDIR /root/

COPY --from=0 /go/src/github.com/go/helloworld/app .

CMD ["./app"]
```

构建镜像

```shell
$ docker build -t go/helloworld:3 .
```

对比三个镜像大小

```shell
$ docker image ls

REPOSITORY        TAG   IMAGE ID         CREATED            SIZE
go/helloworld     3     d6911ed9c846     7 seconds ago      6.47MB
go/helloworld     2     f7cf3465432c     22 seconds ago     6.47MB
go/helloworld     1     f55d3e16affc     2 minutes ago      295MB
```

很明显使用多阶段构建的镜像体积小，同时也完美解决了上边提到的问题。

**只构建某一阶段的镜像**

我们可以使用 `as` 来为某一阶段命名，例如

```shell
FROM golang:alpine as builder
```

如当我们只想构建 `builder` 阶段的镜像时，增加 `--target=builder` 参数即可

```shell
$ docker build --target builder -t username/imagename:tag .
```

**构建时从其他镜像复制文件**

上面例子中我们使用 `COPY --from=0 /go/src/github.com/go/helloworld/app .` 从上一阶段的镜像中复制文件，我们也可以复制任意镜像中的文件。

```shell
$ COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf
```

# 实战案例

## 为容器添加SSH服务

### 基于commit创建

**准备工作**

```shell
#创建容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker pull ubuntu:18.04
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker run -it ubuntu:18.04 bash
root@e4c0007d9035:/# apt-get update
```

**配置软件源**

```shell
root@e4c0007d9035:/# apt-get update
```

**安装和配置**SSH**服务**

```shell
#安装vim工具包
root@e4c0007d9035:/# apt-get install -y vim

#安装SSH服务
root@e4c0007d9035:/# apt-get install openssh-server

#创建文件夹
root@e4c0007d9035:/# mkdir -p /var/run/sshd

#启动服务
root@e4c0007d9035:/# /usr/sbin/sshd -D &

#安装网络工具
root@e4c0007d9035:/# apt-get install net-tools

#查看SSH服务服务端口是否处于监听状态
root@e4c0007d9035:/# netstat -tunlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      3896/sshd           
tcp6       0      0 :::22                   :::*                    LISTEN      3896/sshd

#修改SSH的安全登录配置，取消pam登陆限制
root@e4c0007d9035:/# sed -ri 's/session required pam_loginuid.so/#session required pam_loginuid.so/g' /etc/pam.d/sshd

#去宿主机目录下执行以下命令，一路回车获取key
[root@iZuf6jcqwirs6izt7krfcmZ ~]# ssh-keygen -t rsa
The key fingerprint is:
SHA256:I4GixA/zoZssmCwgcDb5y4VJFIQIabFVlW3oFRzwfxo root@iZuf6jcqwirs6izt7krfcmZ
The key's randomart image is:
+---[RSA 2048]----+
|o+.+=o.o*oo      |
|+.+o . o.=       |
|o**.o o o.       |
|o+*=.o o  .      |
|+. o+ o S  E .   |
|*.o. o . .  +    |
|==  o      .     |
|o                |
|                 |
+----[SHA256]-----+

#生成的key
[root@iZuf6jcqwirs6izt7krfcmZ ~]# cat .ssh/id_rsa.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDk3fAgh1i5A3Y5N8dWdzTtS1eo/NhB2M0+SJhOhg3y70wytkWABV4t7fUgFToH15OTuTSwi7ninQ2CUGgCq9poKLLDzlC0Xqj0Ys+oAG0DtcUw5p4MRcVu+y3enwLl/9PwCB7iGwEIZ1/dXlhwQp32n09Wum57GP7MTjugbWuokJPvkRGuBa2zmyLrScAjqFAGfwRbUHW0w+DA0tno979JwLy/KP2jvOuFBo5iKRmyrnTBsTlHTInIGjOjsovDQ0WSTrjREDvB+XxW4Xz6x9GEf9nuKi/RJqLydAz9Rn+0R05Kd5Rh4GVijjlFC5+Sfab8UCnEhfclxpCHORzsjbkp root@iZuf6jcqwirs6izt7krfcmZ

#创建文件夹
root@e4c0007d9035:/# mkdir root/.ssh

#并将生成的key放入容器中的该目录文件下
root@e4c0007d9035:/# vi /root/.ssh/authorized_keys

#创建自动启动的SSH服务的可执行文件run.sh，并且添加权限
root@e4c0007d9035:/# vim /run.sh
root@e4c0007d9035:/# cat /run.sh 
#! /bin/bash
/usr/sbin/sshd -D 
root@6b59e1de5111:/# chmod +x /run.sh

#查看刚刚修改的容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker ps -a
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS                          PORTS                                       NAMES
e4c0007d9035   ubuntu:18.04          "bash"                   14 minutes ago   Exited (0) About a minute ago                                               upbeat_leavitt
                      0.0.0.0:9000->9000/tcp, :::9000->9000/tcp   naughty_kirch
                      
#提交容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker commit e4c0007d9035 sshd:ubuntu

#查看生成的镜像
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker images
REPOSITORY            TAG       IMAGE ID       CREATED         SIZE
sshd                  ubuntu    822b185733f4   5 seconds ago   251MB

#启动容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker run -p 10022:22 -d sshd:ubuntu /run.sh
97a2c953b2e27fcc5f8a71dbf5d7acc577343e4e6706e9b3b4472baa96170b6d

#查看容器
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker ps
CONTAINER ID   IMAGE                 COMMAND        CREATED         STATUS         PORTS                                       NAMES
97a2c953b2e2   sshd:ubuntu           "/run.sh"      5 seconds ago   Up 4 seconds   0.0.0.0:10022->22/tcp, :::10022->22/tcp     upbeat_roentgen

#在宿主机或者其他主机上，可以通过SSH访问1002 端口来登陆了
[root@iZuf6jcqwirs6izt7krfcmZ ~]# ssh 172.27.240.55 -p 10022
The authenticity of host '[172.27.240.55]:10022 ([172.27.240.55]:10022)' can't be established.
ECDSA key fingerprint is SHA256:pr7bZNwS/YsPVYa5UN16AK7v5R7Q6sbchR5s+Ct8miI.
ECDSA key fingerprint is MD5:83:e9:d6:00:4e:84:a7:68:c4:de:f3:aa:87:4d:a4:9b.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[172.27.240.55]:10022' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 3.10.0-1160.11.1.el7.x86_64 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

root@97a2c953b2e2:~# 

```

### 使用Dockerfile创建

```shell
#创建工作目录
[root@iZuf6jcqwirs6izt7krfcmZ home]# mkdir sshd_ubuntu

#创建Dockerfile和run.sh文件
[root@iZuf6jcqwirs6izt7krfcmZ home]#  cd /home/sshd_ubuntu/
[root@iZuf6jcqwirs6izt7krfcmZ sshd_ubuntu]# touch Dockerfile run.sh
#并在 run.sh文加下添加一下内容
[root@iZuf6jcqwirs6izt7krfcmZ sshd_ubuntu]# cat run.sh 
#!/bin/bash
/usr/sbin/sshd  -D

#使用上一步基于commit创建的authorized_keys
[root@iZuf6jcqwirs6izt7krfcmZ sshd_ubuntu]# cat /root/.ssh/id_rsa.pub > authorized_keys
[root@iZuf6jcqwirs6izt7krfcmZ sshd_ubuntu]# ll
total 8
-rw-r--r-- 1 root root 410 May 29 18:04 authorized_keys
-rw-r--r-- 1 root root   0 May 29 18:03 Dockerfile
-rw-r--r-- 1 root root  31 May 29 18:04 run.sh

#编写Dockerfile
[root@iZuf6jcqwirs6izt7krfcmZ sshd_ubuntu]# cat Dockerfile    
#设置继承镜像
FROM ubuntu:18.04
LABEL author="zhonghao"
#下面开始运行命令,此处更改ubuntu的源为国内163的源
RUN echo "deb http://mirrors.163.com/ubuntu/ bionic main restricted universe multiverse" > /etc/apt/sources.list
RUN echo "deb http://mirrors.163.com/ubuntu/ bionic-security main restricted universe multiverse" >> /etc/apt/sources.list
RUN echo "deb http://mirrors.163.com/ubuntu/ bionic-updates main restricted universe multiverse" >> /etc/apt/sources.list
RUN echo "deb http://mirrors.163.com/ubuntu/ bionic-proposed main restricted universe multiverse" >> /etc/apt/sources.list
RUN echo "deb http://mirrors.163.com/ubuntu/ bionic-backports main restricted universe multiverse" >> /etc/apt/sources.list
RUN apt-get update
#安装ssh服务
RUN apt-get install -y openssh-server
RUN mkdir -p /var/run/sshd
RUN mkdir -p /root/.ssh
#取消pam限制
RUN sed -ri  's/session    required     pam_loginuid.so/#session    required     pam_loginuid.so/g'  /etc/pam.d/sshd
#复制配置文件到相应位置，并赋予脚本可执行权限
COPY authorized_keys /root/.ssh/authorized_keys
COPY run.sh /run.sh
RUN chmod 755 /run.sh
#开放端口
EXPOSE 22
#设置自启动命令
CMD ["/run.sh"]

#构建镜像
[root@iZuf6jcqwirs6izt7krfcmZ sshd_ubuntu]# docker build -t sshd2:ubuntu .

#查看镜像信息
[root@iZuf6jcqwirs6izt7krfcmZ sshd_ubuntu]# docker images
REPOSITORY            TAG       IMAGE ID       CREATED          SIZE
sshd2                 ubuntu    73a97df110b6   11 seconds ago   215MB
sshd                  ubuntu    822b185733f4   27 minutes ago   251MB

#运行容器
[root@iZuf6jcqwirs6izt7krfcmZ sshd_ubuntu]# docker run -p 10033:22 -d sshd2:ubuntu 
c0a64a25198866b3a83cb610a7e6b1c62a195d927cf5044745bcae8857779a55
[root@iZuf6jcqwirs6izt7krfcmZ sshd_ubuntu]# docker ps
CONTAINER ID   IMAGE                 COMMAND        CREATED          STATUS          PORTS                                       NAMES
c0a64a251988   sshd2:ubuntu          "/run.sh"      5 seconds ago    Up 4 seconds    0.0.0.0:10033->22/tcp, :::10033->22/tcp     crazy_franklin
97a2c953b2e2   sshd:ubuntu           "/run.sh"      28 minutes ago   Up 28 minutes   0.0.0.0:10022->22/tcp, :::10022->22/tcp     upbeat_roentgen

#在宿主机或者其他主机上，可以通过SSH访问1002 端口来登陆了
[root@iZuf6jcqwirs6izt7krfcmZ sshd_ubuntu]# ssh 172.27.240.55 -p 10033
The authenticity of host '[172.27.240.55]:10033 ([172.27.240.55]:10033)' can't be established.
ECDSA key fingerprint is SHA256:4NG9xeu8Tz6eG/sb3GLcQwZFk9OywQUFA1SVY9+boN0.
ECDSA key fingerprint is MD5:9e:99:73:54:b0:92:7c:42:19:9e:35:c2:34:d2:7b:3a.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[172.27.240.55]:10033' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 3.10.0-1160.11.1.el7.x86_64 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

root@c0a64a251988:~# 
```

## web应用与服务

### Apache

### Nginx

### Tomcat

## 数据库应用

### Redis集群搭建

参考文章：https://my.oschina.net/dslcode/blog/1936656

#### **所有redis实例运行在同一台宿主机上**

1. 拉取Redis镜像

   ```shell
   #选择5.0.12版本的
   [root@iZuf67z0nmo18acuap4yhxZ data]# docker pull redis:5.0.12
   ```

2. 创建文件夹

   ```shell
   [root@iZuf67z0nmo18acuap4yhxZ /]# mkdir/home/redis
   [root@iZuf67z0nmo18acuap4yhxZ redis]# ll
   total 72
   ```

3. 找到一份原始的redis.conf文件，将其重命名为：redis-cluster.tmpl，并配置如下几个参数，此文件的目的是生成每一个redis实例的redis.conf

   ```shell
   # Redis configuration file example.
   #
   # Note that in order to read the configuration file, Redis must be
   # started with the file path as first argument:
   #
   # ./redis-server /path/to/redis.conf
   
   # Note on units: when memory size is needed, it is possible to specify
   # it in the usual form of 1k 5GB 4M and so forth:
   #
   # 1k => 1000 bytes
   # 1kb => 1024 bytes
   # 1m => 1000000 bytes
   # 1mb => 1024*1024 bytes
   # 1g => 1000000000 bytes
   # 1gb => 1024*1024*1024 bytes
   #
   # units are case insensitive so 1GB 1Gb 1gB are all the same.
   
   ################################## INCLUDES ###################################
   
   # Include one or more other config files here.  This is useful if you
   # have a standard template that goes to all Redis servers but also need
   # to customize a few per-server settings.  Include files can include
   # other files, so use this wisely.
   #
   # Notice option "include" won't be rewritten by command "CONFIG REWRITE"
   # from admin or Redis Sentinel. Since Redis always uses the last processed
   # line as value of a configuration directive, you'd better put includes
   # at the beginning of this file to avoid overwriting config change at runtime.
   #
   # If instead you are interested in using includes to override configuration
   # options, it is better to use include as the last line.
   #
   # include /path/to/local.conf
   # include /path/to/other.conf
   
   ################################## MODULES #####################################
   
   # Load modules at startup. If the server is not able to load modules
   # it will abort. It is possible to use multiple loadmodule directives.
   #
   # loadmodule /path/to/my_module.so
   # loadmodule /path/to/other_module.so
   
   ################################## NETWORK #####################################
   
   # By default, if no "bind" configuration directive is specified, Redis listens
   # for connections from all the network interfaces available on the server.
   # It is possible to listen to just one or multiple selected interfaces using
   # the "bind" configuration directive, followed by one or more IP addresses.
   #
   # Examples:
   #
   # bind 192.168.1.100 10.0.0.1
   # bind 127.0.0.1 ::1
   #
   # ~~~ WARNING ~~~ If the computer running Redis is directly exposed to the
   # internet, binding to all the interfaces is dangerous and will expose the
   # instance to everybody on the internet. So by default we uncomment the
   # following bind directive, that will force Redis to listen only into
   # the IPv4 loopback interface address (this means Redis will be able to
   # accept connections only from clients running into the same computer it
   # is running).
   #
   # IF YOU ARE SURE YOU WANT YOUR INSTANCE TO LISTEN TO ALL THE INTERFACES
   # JUST COMMENT THE FOLLOWING LINE.
   # ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
   #bind 127.0.0.1
   
   # Protected mode is a layer of security protection, in order to avoid that
   # Redis instances left open on the internet are accessed and exploited.
   #
   # When protected mode is on and if:
   #
   # 1) The server is not binding explicitly to a set of addresses using the
   #    "bind" directive.
   # 2) No password is configured.
   #
   # The server only accepts connections from clients connecting from the
   # IPv4 and IPv6 loopback addresses 127.0.0.1 and ::1, and from Unix domain
   # sockets.
   #
   # By default protected mode is enabled. You should disable it only if
   # you are sure you want clients from other hosts to connect to Redis
   # even if no authentication is configured, nor a specific set of interfaces
   # are explicitly listed using the "bind" directive.
   protected-mode no
   
   # Accept connections on the specified port, default is 6379 (IANA #815344).
   # If port 0 is specified Redis will not listen on a TCP socket.
   port ${PORT}
   
   # TCP listen() backlog.
   #
   # In high requests-per-second environments you need an high backlog in order
   # to avoid slow clients connections issues. Note that the Linux kernel
   # will silently truncate it to the value of /proc/sys/net/core/somaxconn so
   # make sure to raise both the value of somaxconn and tcp_max_syn_backlog
   # in order to get the desired effect.
   tcp-backlog 511
   
   # Unix socket.
   #
   # Specify the path for the Unix socket that will be used to listen for
   # incoming connections. There is no default, so Redis will not listen
   # on a unix socket when not specified.
   #
   # unixsocket /tmp/redis.sock
   # unixsocketperm 700
   
   # Close the connection after a client is idle for N seconds (0 to disable)
   timeout 0
   
   # TCP keepalive.
   #
   # If non-zero, use SO_KEEPALIVE to send TCP ACKs to clients in absence
   # of communication. This is useful for two reasons:
   #
   # 1) Detect dead peers.
   # 2) Take the connection alive from the point of view of network
   #    equipment in the middle.
   #
   # On Linux, the specified value (in seconds) is the period used to send ACKs.
   # Note that to close the connection the double of the time is needed.
   # On other kernels the period depends on the kernel configuration.
   #
   # A reasonable value for this option is 300 seconds, which is the new
   # Redis default starting with Redis 3.2.1.
   tcp-keepalive 300
   
   ################################# GENERAL #####################################
   
   # By default Redis does not run as a daemon. Use 'yes' if you need it.
   # Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
   daemonize no
   
   # If you run Redis from upstart or systemd, Redis can interact with your
   # supervision tree. Options:
   #   supervised no      - no supervision interaction
   #   supervised upstart - signal upstart by putting Redis into SIGSTOP mode
   #   supervised systemd - signal systemd by writing READY=1 to $NOTIFY_SOCKET
   #   supervised auto    - detect upstart or systemd method based on
   #                        UPSTART_JOB or NOTIFY_SOCKET environment variables
   # Note: these supervision methods only signal "process is ready."
   #       They do not enable continuous liveness pings back to your supervisor.
   supervised no
   
   # If a pid file is specified, Redis writes it where specified at startup
   # and removes it at exit.
   #
   # When the server runs non daemonized, no pid file is created if none is
   # specified in the configuration. When the server is daemonized, the pid file
   # is used even if not specified, defaulting to "/var/run/redis.pid".
   #
   # Creating a pid file is best effort: if Redis is not able to create it
   # nothing bad happens, the server will start and run normally.
   pidfile /var/run/redis_6379.pid
   
   # Specify the server verbosity level.
   # This can be one of:
   # debug (a lot of information, useful for development/testing)
   # verbose (many rarely useful info, but not a mess like the debug level)
   # notice (moderately verbose, what you want in production probably)
   # warning (only very important / critical messages are logged)
   loglevel notice
   
   # Specify the log file name. Also the empty string can be used to force
   # Redis to log on the standard output. Note that if you use standard
   # output for logging but daemonize, logs will be sent to /dev/null
   logfile ""
   
   # To enable logging to the system logger, just set 'syslog-enabled' to yes,
   # and optionally update the other syslog parameters to suit your needs.
   # syslog-enabled no
   
   # Specify the syslog identity.
   # syslog-ident redis
   
   # Specify the syslog facility. Must be USER or between LOCAL0-LOCAL7.
   # syslog-facility local0
   
   # Set the number of databases. The default database is DB 0, you can select
   # a different one on a per-connection basis using SELECT <dbid> where
   # dbid is a number between 0 and 'databases'-1
   databases 16
   
   # By default Redis shows an ASCII art logo only when started to log to the
   # standard output and if the standard output is a TTY. Basically this means
   # that normally a logo is displayed only in interactive sessions.
   #
   # However it is possible to force the pre-4.0 behavior and always show a
   # ASCII art logo in startup logs by setting the following option to yes.
   always-show-logo yes
   
   ################################ SNAPSHOTTING  ################################
   #
   # Save the DB on disk:
   #
   #   save <seconds> <changes>
   #
   #   Will save the DB if both the given number of seconds and the given
   #   number of write operations against the DB occurred.
   #
   #   In the example below the behaviour will be to save:
   #   after 900 sec (15 min) if at least 1 key changed
   #   after 300 sec (5 min) if at least 10 keys changed
   #   after 60 sec if at least 10000 keys changed
   #
   #   Note: you can disable saving completely by commenting out all "save" lines.
   #
   #   It is also possible to remove all the previously configured save
   #   points by adding a save directive with a single empty string argument
   #   like in the following example:
   #
   #   save ""
   
   save 900 1
   save 300 10
   save 60 10000
   
   # By default Redis will stop accepting writes if RDB snapshots are enabled
   # (at least one save point) and the latest background save failed.
   # This will make the user aware (in a hard way) that data is not persisting
   # on disk properly, otherwise chances are that no one will notice and some
   # disaster will happen.
   #
   # If the background saving process will start working again Redis will
   # automatically allow writes again.
   #
   # However if you have setup your proper monitoring of the Redis server
   # and persistence, you may want to disable this feature so that Redis will
   # continue to work as usual even if there are problems with disk,
   # permissions, and so forth.
   stop-writes-on-bgsave-error yes
   
   # Compress string objects using LZF when dump .rdb databases?
   # For default that's set to 'yes' as it's almost always a win.
   # If you want to save some CPU in the saving child set it to 'no' but
   # the dataset will likely be bigger if you have compressible values or keys.
   rdbcompression yes
   
   # Since version 5 of RDB a CRC64 checksum is placed at the end of the file.
   # This makes the format more resistant to corruption but there is a performance
   # hit to pay (around 10%) when saving and loading RDB files, so you can disable it
   # for maximum performances.
   #
   # RDB files created with checksum disabled have a checksum of zero that will
   # tell the loading code to skip the check.
   rdbchecksum yes
   
   # The filename where to dump the DB
   dbfilename dump.rdb
   
   # The working directory.
   #
   # The DB will be written inside this directory, with the filename specified
   # above using the 'dbfilename' configuration directive.
   #
   # The Append Only File will also be created inside this directory.
   #
   # Note that you must specify a directory here, not a file name.
   dir /data/redis
   
   ################################# REPLICATION #################################
   
   # Master-Replica replication. Use replicaof to make a Redis instance a copy of
   # another Redis server. A few things to understand ASAP about Redis replication.
   #
   #   +------------------+      +---------------+
   #   |      Master      | ---> |    Replica    |
   #   | (receive writes) |      |  (exact copy) |
   #   +------------------+      +---------------+
   #
   # 1) Redis replication is asynchronous, but you can configure a master to
   #    stop accepting writes if it appears to be not connected with at least
   #    a given number of replicas.
   # 2) Redis replicas are able to perform a partial resynchronization with the
   #    master if the replication link is lost for a relatively small amount of
   #    time. You may want to configure the replication backlog size (see the next
   #    sections of this file) with a sensible value depending on your needs.
   # 3) Replication is automatic and does not need user intervention. After a
   #    network partition replicas automatically try to reconnect to masters
   #    and resynchronize with them.
   #
   # replicaof <masterip> <masterport>
   
   # If the master is password protected (using the "requirepass" configuration
   # directive below) it is possible to tell the replica to authenticate before
   # starting the replication synchronization process, otherwise the master will
   # refuse the replica request.
   #
   # masterauth <master-password>
   
   # When a replica loses its connection with the master, or when the replication
   # is still in progress, the replica can act in two different ways:
   #
   # 1) if replica-serve-stale-data is set to 'yes' (the default) the replica will
   #    still reply to client requests, possibly with out of date data, or the
   #    data set may just be empty if this is the first synchronization.
   #
   # 2) if replica-serve-stale-data is set to 'no' the replica will reply with
   #    an error "SYNC with master in progress" to all the kind of commands
   #    but to INFO, replicaOF, AUTH, PING, SHUTDOWN, REPLCONF, ROLE, CONFIG,
   #    SUBSCRIBE, UNSUBSCRIBE, PSUBSCRIBE, PUNSUBSCRIBE, PUBLISH, PUBSUB,
   #    COMMAND, POST, HOST: and LATENCY.
   #
   replica-serve-stale-data yes
   
   # You can configure a replica instance to accept writes or not. Writing against
   # a replica instance may be useful to store some ephemeral data (because data
   # written on a replica will be easily deleted after resync with the master) but
   # may also cause problems if clients are writing to it because of a
   # misconfiguration.
   #
   # Since Redis 2.6 by default replicas are read-only.
   #
   # Note: read only replicas are not designed to be exposed to untrusted clients
   # on the internet. It's just a protection layer against misuse of the instance.
   # Still a read only replica exports by default all the administrative commands
   # such as CONFIG, DEBUG, and so forth. To a limited extent you can improve
   # security of read only replicas using 'rename-command' to shadow all the
   # administrative / dangerous commands.
   replica-read-only yes
   
   # Replication SYNC strategy: disk or socket.
   #
   # -------------------------------------------------------
   # WARNING: DISKLESS REPLICATION IS EXPERIMENTAL CURRENTLY
   # -------------------------------------------------------
   #
   # New replicas and reconnecting replicas that are not able to continue the replication
   # process just receiving differences, need to do what is called a "full
   # synchronization". An RDB file is transmitted from the master to the replicas.
   # The transmission can happen in two different ways:
   #
   # 1) Disk-backed: The Redis master creates a new process that writes the RDB
   #                 file on disk. Later the file is transferred by the parent
   #                 process to the replicas incrementally.
   # 2) Diskless: The Redis master creates a new process that directly writes the
   #              RDB file to replica sockets, without touching the disk at all.
   #
   # With disk-backed replication, while the RDB file is generated, more replicas
   # can be queued and served with the RDB file as soon as the current child producing
   # the RDB file finishes its work. With diskless replication instead once
   # the transfer starts, new replicas arriving will be queued and a new transfer
   # will start when the current one terminates.
   #
   # When diskless replication is used, the master waits a configurable amount of
   # time (in seconds) before starting the transfer in the hope that multiple replicas
   # will arrive and the transfer can be parallelized.
   #
   # With slow disks and fast (large bandwidth) networks, diskless replication
   # works better.
   repl-diskless-sync no
   
   # When diskless replication is enabled, it is possible to configure the delay
   # the server waits in order to spawn the child that transfers the RDB via socket
   # to the replicas.
   #
   # This is important since once the transfer starts, it is not possible to serve
   # new replicas arriving, that will be queued for the next RDB transfer, so the server
   # waits a delay in order to let more replicas arrive.
   #
   # The delay is specified in seconds, and by default is 5 seconds. To disable
   # it entirely just set it to 0 seconds and the transfer will start ASAP.
   repl-diskless-sync-delay 5
   
   # Replicas send PINGs to server in a predefined interval. It's possible to change
   # this interval with the repl_ping_replica_period option. The default value is 10
   # seconds.
   #
   # repl-ping-replica-period 10
   
   # The following option sets the replication timeout for:
   #
   # 1) Bulk transfer I/O during SYNC, from the point of view of replica.
   # 2) Master timeout from the point of view of replicas (data, pings).
   # 3) Replica timeout from the point of view of masters (REPLCONF ACK pings).
   #
   # It is important to make sure that this value is greater than the value
   # specified for repl-ping-replica-period otherwise a timeout will be detected
   # every time there is low traffic between the master and the replica.
   #
   # repl-timeout 60
   
   # Disable TCP_NODELAY on the replica socket after SYNC?
   #
   # If you select "yes" Redis will use a smaller number of TCP packets and
   # less bandwidth to send data to replicas. But this can add a delay for
   # the data to appear on the replica side, up to 40 milliseconds with
   # Linux kernels using a default configuration.
   #
   # If you select "no" the delay for data to appear on the replica side will
   # be reduced but more bandwidth will be used for replication.
   #
   # By default we optimize for low latency, but in very high traffic conditions
   # or when the master and replicas are many hops away, turning this to "yes" may
   # be a good idea.
   repl-disable-tcp-nodelay no
   
   # Set the replication backlog size. The backlog is a buffer that accumulates
   # replica data when replicas are disconnected for some time, so that when a replica
   # wants to reconnect again, often a full resync is not needed, but a partial
   # resync is enough, just passing the portion of data the replica missed while
   # disconnected.
   #
   # The bigger the replication backlog, the longer the time the replica can be
   # disconnected and later be able to perform a partial resynchronization.
   #
   # The backlog is only allocated once there is at least a replica connected.
   #
   # repl-backlog-size 1mb
   
   # After a master has no longer connected replicas for some time, the backlog
   # will be freed. The following option configures the amount of seconds that
   # need to elapse, starting from the time the last replica disconnected, for
   # the backlog buffer to be freed.
   #
   # Note that replicas never free the backlog for timeout, since they may be
   # promoted to masters later, and should be able to correctly "partially
   # resynchronize" with the replicas: hence they should always accumulate backlog.
   #
   # A value of 0 means to never release the backlog.
   #
   # repl-backlog-ttl 3600
   
   # The replica priority is an integer number published by Redis in the INFO output.
   # It is used by Redis Sentinel in order to select a replica to promote into a
   # master if the master is no longer working correctly.
   #
   # A replica with a low priority number is considered better for promotion, so
   # for instance if there are three replicas with priority 10, 100, 25 Sentinel will
   # pick the one with priority 10, that is the lowest.
   #
   # However a special priority of 0 marks the replica as not able to perform the
   # role of master, so a replica with priority of 0 will never be selected by
   # Redis Sentinel for promotion.
   #
   # By default the priority is 100.
   replica-priority 100
   
   # It is possible for a master to stop accepting writes if there are less than
   # N replicas connected, having a lag less or equal than M seconds.
   #
   # The N replicas need to be in "online" state.
   #
   # The lag in seconds, that must be <= the specified value, is calculated from
   # the last ping received from the replica, that is usually sent every second.
   #
   # This option does not GUARANTEE that N replicas will accept the write, but
   # will limit the window of exposure for lost writes in case not enough replicas
   # are available, to the specified number of seconds.
   #
   # For example to require at least 3 replicas with a lag <= 10 seconds use:
   #
   # min-replicas-to-write 3
   # min-replicas-max-lag 10
   #
   # Setting one or the other to 0 disables the feature.
   #
   # By default min-replicas-to-write is set to 0 (feature disabled) and
   # min-replicas-max-lag is set to 10.
   
   # A Redis master is able to list the address and port of the attached
   # replicas in different ways. For example the "INFO replication" section
   # offers this information, which is used, among other tools, by
   # Redis Sentinel in order to discover replica instances.
   # Another place where this info is available is in the output of the
   # "ROLE" command of a master.
   #
   # The listed IP and address normally reported by a replica is obtained
   # in the following way:
   #
   #   IP: The address is auto detected by checking the peer address
   #   of the socket used by the replica to connect with the master.
   #
   #   Port: The port is communicated by the replica during the replication
   #   handshake, and is normally the port that the replica is using to
   #   listen for connections.
   #
   # However when port forwarding or Network Address Translation (NAT) is
   # used, the replica may be actually reachable via different IP and port
   # pairs. The following two options can be used by a replica in order to
   # report to its master a specific set of IP and port, so that both INFO
   # and ROLE will report those values.
   #
   # There is no need to use both the options if you need to override just
   # the port or the IP address.
   #
   # replica-announce-ip 5.5.5.5
   # replica-announce-port 1234
   
   ################################## SECURITY ###################################
   
   # Require clients to issue AUTH <PASSWORD> before processing any other
   # commands.  This might be useful in environments in which you do not trust
   # others with access to the host running redis-server.
   #
   # This should stay commented out for backward compatibility and because most
   # people do not need auth (e.g. they run their own servers).
   #
   # Warning: since Redis is pretty fast an outside user can try up to
   # 150k passwords per second against a good box. This means that you should
   # use a very strong password otherwise it will be very easy to break.
   #
   # requirepass foobared
   
   # Command renaming.
   #
   # It is possible to change the name of dangerous commands in a shared
   # environment. For instance the CONFIG command may be renamed into something
   # hard to guess so that it will still be available for internal-use tools
   # but not available for general clients.
   #
   # Example:
   #
   # rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
   #
   # It is also possible to completely kill a command by renaming it into
   # an empty string:
   #
   # rename-command CONFIG ""
   #
   # Please note that changing the name of commands that are logged into the
   # AOF file or transmitted to replicas may cause problems.
   
   ################################### CLIENTS ####################################
   
   # Set the max number of connected clients at the same time. By default
   # this limit is set to 10000 clients, however if the Redis server is not
   # able to configure the process file limit to allow for the specified limit
   # the max number of allowed clients is set to the current file limit
   # minus 32 (as Redis reserves a few file descriptors for internal uses).
   #
   # Once the limit is reached Redis will close all the new connections sending
   # an error 'max number of clients reached'.
   #
   # maxclients 10000
   
   ############################## MEMORY MANAGEMENT ################################
   
   # Set a memory usage limit to the specified amount of bytes.
   # When the memory limit is reached Redis will try to remove keys
   # according to the eviction policy selected (see maxmemory-policy).
   #
   # If Redis can't remove keys according to the policy, or if the policy is
   # set to 'noeviction', Redis will start to reply with errors to commands
   # that would use more memory, like SET, LPUSH, and so on, and will continue
   # to reply to read-only commands like GET.
   #
   # This option is usually useful when using Redis as an LRU or LFU cache, or to
   # set a hard memory limit for an instance (using the 'noeviction' policy).
   #
   # WARNING: If you have replicas attached to an instance with maxmemory on,
   # the size of the output buffers needed to feed the replicas are subtracted
   # from the used memory count, so that network problems / resyncs will
   # not trigger a loop where keys are evicted, and in turn the output
   # buffer of replicas is full with DELs of keys evicted triggering the deletion
   # of more keys, and so forth until the database is completely emptied.
   #
   # In short... if you have replicas attached it is suggested that you set a lower
   # limit for maxmemory so that there is some free RAM on the system for replica
   # output buffers (but this is not needed if the policy is 'noeviction').
   #
   # maxmemory <bytes>
   
   # MAXMEMORY POLICY: how Redis will select what to remove when maxmemory
   # is reached. You can select among five behaviors:
   #
   # volatile-lru -> Evict using approximated LRU among the keys with an expire set.
   # allkeys-lru -> Evict any key using approximated LRU.
   # volatile-lfu -> Evict using approximated LFU among the keys with an expire set.
   # allkeys-lfu -> Evict any key using approximated LFU.
   # volatile-random -> Remove a random key among the ones with an expire set.
   # allkeys-random -> Remove a random key, any key.
   # volatile-ttl -> Remove the key with the nearest expire time (minor TTL)
   # noeviction -> Don't evict anything, just return an error on write operations.
   #
   # LRU means Least Recently Used
   # LFU means Least Frequently Used
   #
   # Both LRU, LFU and volatile-ttl are implemented using approximated
   # randomized algorithms.
   #
   # Note: with any of the above policies, Redis will return an error on write
   #       operations, when there are no suitable keys for eviction.
   #
   #       At the date of writing these commands are: set setnx setex append
   #       incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd
   #       sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby
   #       zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby
   #       getset mset msetnx exec sort
   #
   # The default is:
   #
   # maxmemory-policy noeviction
   
   # LRU, LFU and minimal TTL algorithms are not precise algorithms but approximated
   # algorithms (in order to save memory), so you can tune it for speed or
   # accuracy. For default Redis will check five keys and pick the one that was
   # used less recently, you can change the sample size using the following
   # configuration directive.
   #
   # The default of 5 produces good enough results. 10 Approximates very closely
   # true LRU but costs more CPU. 3 is faster but not very accurate.
   #
   # maxmemory-samples 5
   
   # Starting from Redis 5, by default a replica will ignore its maxmemory setting
   # (unless it is promoted to master after a failover or manually). It means
   # that the eviction of keys will be just handled by the master, sending the
   # DEL commands to the replica as keys evict in the master side.
   #
   # This behavior ensures that masters and replicas stay consistent, and is usually
   # what you want, however if your replica is writable, or you want the replica to have
   # a different memory setting, and you are sure all the writes performed to the
   # replica are idempotent, then you may change this default (but be sure to understand
   # what you are doing).
   #
   # Note that since the replica by default does not evict, it may end using more
   # memory than the one set via maxmemory (there are certain buffers that may
   # be larger on the replica, or data structures may sometimes take more memory and so
   # forth). So make sure you monitor your replicas and make sure they have enough
   # memory to never hit a real out-of-memory condition before the master hits
   # the configured maxmemory setting.
   #
   # replica-ignore-maxmemory yes
   
   ############################# LAZY FREEING ####################################
   
   # Redis has two primitives to delete keys. One is called DEL and is a blocking
   # deletion of the object. It means that the server stops processing new commands
   # in order to reclaim all the memory associated with an object in a synchronous
   # way. If the key deleted is associated with a small object, the time needed
   # in order to execute the DEL command is very small and comparable to most other
   # O(1) or O(log_N) commands in Redis. However if the key is associated with an
   # aggregated value containing millions of elements, the server can block for
   # a long time (even seconds) in order to complete the operation.
   #
   # For the above reasons Redis also offers non blocking deletion primitives
   # such as UNLINK (non blocking DEL) and the ASYNC option of FLUSHALL and
   # FLUSHDB commands, in order to reclaim memory in background. Those commands
   # are executed in constant time. Another thread will incrementally free the
   # object in the background as fast as possible.
   #
   # DEL, UNLINK and ASYNC option of FLUSHALL and FLUSHDB are user-controlled.
   # It's up to the design of the application to understand when it is a good
   # idea to use one or the other. However the Redis server sometimes has to
   # delete keys or flush the whole database as a side effect of other operations.
   # Specifically Redis deletes objects independently of a user call in the
   # following scenarios:
   #
   # 1) On eviction, because of the maxmemory and maxmemory policy configurations,
   #    in order to make room for new data, without going over the specified
   #    memory limit.
   # 2) Because of expire: when a key with an associated time to live (see the
   #    EXPIRE command) must be deleted from memory.
   # 3) Because of a side effect of a command that stores data on a key that may
   #    already exist. For example the RENAME command may delete the old key
   #    content when it is replaced with another one. Similarly SUNIONSTORE
   #    or SORT with STORE option may delete existing keys. The SET command
   #    itself removes any old content of the specified key in order to replace
   #    it with the specified string.
   # 4) During replication, when a replica performs a full resynchronization with
   #    its master, the content of the whole database is removed in order to
   #    load the RDB file just transferred.
   #
   # In all the above cases the default is to delete objects in a blocking way,
   # like if DEL was called. However you can configure each case specifically
   # in order to instead release memory in a non-blocking way like if UNLINK
   # was called, using the following configuration directives:
   
   lazyfree-lazy-eviction no
   lazyfree-lazy-expire no
   lazyfree-lazy-server-del no
   replica-lazy-flush no
   
   ############################## APPEND ONLY MODE ###############################
   
   # By default Redis asynchronously dumps the dataset on disk. This mode is
   # good enough in many applications, but an issue with the Redis process or
   # a power outage may result into a few minutes of writes lost (depending on
   # the configured save points).
   #
   # The Append Only File is an alternative persistence mode that provides
   # much better durability. For instance using the default data fsync policy
   # (see later in the config file) Redis can lose just one second of writes in a
   # dramatic event like a server power outage, or a single write if something
   # wrong with the Redis process itself happens, but the operating system is
   # still running correctly.
   #
   # AOF and RDB persistence can be enabled at the same time without problems.
   # If the AOF is enabled on startup Redis will load the AOF, that is the file
   # with the better durability guarantees.
   #
   # Please check http://redis.io/topics/persistence for more information.
   
   appendonly yes
   
   # The name of the append only file (default: "appendonly.aof")
   
   appendfilename "appendonly.aof"
   
   # The fsync() call tells the Operating System to actually write data on disk
   # instead of waiting for more data in the output buffer. Some OS will really flush
   # data on disk, some other OS will just try to do it ASAP.
   #
   # Redis supports three different modes:
   #
   # no: don't fsync, just let the OS flush the data when it wants. Faster.
   # always: fsync after every write to the append only log. Slow, Safest.
   # everysec: fsync only one time every second. Compromise.
   #
   # The default is "everysec", as that's usually the right compromise between
   # speed and data safety. It's up to you to understand if you can relax this to
   # "no" that will let the operating system flush the output buffer when
   # it wants, for better performances (but if you can live with the idea of
   # some data loss consider the default persistence mode that's snapshotting),
   # or on the contrary, use "always" that's very slow but a bit safer than
   # everysec.
   #
   # More details please check the following article:
   # http://antirez.com/post/redis-persistence-demystified.html
   #
   # If unsure, use "everysec".
   
   # appendfsync always
   appendfsync everysec
   # appendfsync no
   
   # When the AOF fsync policy is set to always or everysec, and a background
   # saving process (a background save or AOF log background rewriting) is
   # performing a lot of I/O against the disk, in some Linux configurations
   # Redis may block too long on the fsync() call. Note that there is no fix for
   # this currently, as even performing fsync in a different thread will block
   # our synchronous write(2) call.
   #
   # In order to mitigate this problem it's possible to use the following option
   # that will prevent fsync() from being called in the main process while a
   # BGSAVE or BGREWRITEAOF is in progress.
   #
   # This means that while another child is saving, the durability of Redis is
   # the same as "appendfsync none". In practical terms, this means that it is
   # possible to lose up to 30 seconds of log in the worst scenario (with the
   # default Linux settings).
   #
   # If you have latency problems turn this to "yes". Otherwise leave it as
   # "no" that is the safest pick from the point of view of durability.
   
   no-appendfsync-on-rewrite no
   
   # Automatic rewrite of the append only file.
   # Redis is able to automatically rewrite the log file implicitly calling
   # BGREWRITEAOF when the AOF log size grows by the specified percentage.
   #
   # This is how it works: Redis remembers the size of the AOF file after the
   # latest rewrite (if no rewrite has happened since the restart, the size of
   # the AOF at startup is used).
   #
   # This base size is compared to the current size. If the current size is
   # bigger than the specified percentage, the rewrite is triggered. Also
   # you need to specify a minimal size for the AOF file to be rewritten, this
   # is useful to avoid rewriting the AOF file even if the percentage increase
   # is reached but it is still pretty small.
   #
   # Specify a percentage of zero in order to disable the automatic AOF
   # rewrite feature.
   
   auto-aof-rewrite-percentage 100
   auto-aof-rewrite-min-size 64mb
   
   # An AOF file may be found to be truncated at the end during the Redis
   # startup process, when the AOF data gets loaded back into memory.
   # This may happen when the system where Redis is running
   # crashes, especially when an ext4 filesystem is mounted without the
   # data=ordered option (however this can't happen when Redis itself
   # crashes or aborts but the operating system still works correctly).
   #
   # Redis can either exit with an error when this happens, or load as much
   # data as possible (the default now) and start if the AOF file is found
   # to be truncated at the end. The following option controls this behavior.
   #
   # If aof-load-truncated is set to yes, a truncated AOF file is loaded and
   # the Redis server starts emitting a log to inform the user of the event.
   # Otherwise if the option is set to no, the server aborts with an error
   # and refuses to start. When the option is set to no, the user requires
   # to fix the AOF file using the "redis-check-aof" utility before to restart
   # the server.
   #
   # Note that if the AOF file will be found to be corrupted in the middle
   # the server will still exit with an error. This option only applies when
   # Redis will try to read more data from the AOF file but not enough bytes
   # will be found.
   aof-load-truncated yes
   
   # When rewriting the AOF file, Redis is able to use an RDB preamble in the
   # AOF file for faster rewrites and recoveries. When this option is turned
   # on the rewritten AOF file is composed of two different stanzas:
   #
   #   [RDB file][AOF tail]
   #
   # When loading Redis recognizes that the AOF file starts with the "REDIS"
   # string and loads the prefixed RDB file, and continues loading the AOF
   # tail.
   aof-use-rdb-preamble yes
   
   ################################ LUA SCRIPTING  ###############################
   
   # Max execution time of a Lua script in milliseconds.
   #
   # If the maximum execution time is reached Redis will log that a script is
   # still in execution after the maximum allowed time and will start to
   # reply to queries with an error.
   #
   # When a long running script exceeds the maximum execution time only the
   # SCRIPT KILL and SHUTDOWN NOSAVE commands are available. The first can be
   # used to stop a script that did not yet called write commands. The second
   # is the only way to shut down the server in the case a write command was
   # already issued by the script but the user doesn't want to wait for the natural
   # termination of the script.
   #
   # Set it to 0 or a negative value for unlimited execution without warnings.
   lua-time-limit 5000
   
   ################################ REDIS CLUSTER  ###############################
   
   # Normal Redis instances can't be part of a Redis Cluster; only nodes that are
   # started as cluster nodes can. In order to start a Redis instance as a
   # cluster node enable the cluster support uncommenting the following:
   #
   cluster-enabled yes
   
   # Every cluster node has a cluster configuration file. This file is not
   # intended to be edited by hand. It is created and updated by Redis nodes.
   # Every Redis Cluster node requires a different cluster configuration file.
   # Make sure that instances running in the same system do not have
   # overlapping cluster configuration file names.
   #
   cluster-config-file nodes.conf
   
   # Cluster node timeout is the amount of milliseconds a node must be unreachable
   # for it to be considered in failure state.
   # Most other internal time limits are multiple of the node timeout.
   #
   cluster-node-timeout 15000
   
   # A replica of a failing master will avoid to start a failover if its data
   # looks too old.
   #
   # There is no simple way for a replica to actually have an exact measure of
   # its "data age", so the following two checks are performed:
   #
   # 1) If there are multiple replicas able to failover, they exchange messages
   #    in order to try to give an advantage to the replica with the best
   #    replication offset (more data from the master processed).
   #    Replicas will try to get their rank by offset, and apply to the start
   #    of the failover a delay proportional to their rank.
   #
   # 2) Every single replica computes the time of the last interaction with
   #    its master. This can be the last ping or command received (if the master
   #    is still in the "connected" state), or the time that elapsed since the
   #    disconnection with the master (if the replication link is currently down).
   #    If the last interaction is too old, the replica will not try to failover
   #    at all.
   #
   # The point "2" can be tuned by user. Specifically a replica will not perform
   # the failover if, since the last interaction with the master, the time
   # elapsed is greater than:
   #
   #   (node-timeout * replica-validity-factor) + repl-ping-replica-period
   #
   # So for example if node-timeout is 30 seconds, and the replica-validity-factor
   # is 10, and assuming a default repl-ping-replica-period of 10 seconds, the
   # replica will not try to failover if it was not able to talk with the master
   # for longer than 310 seconds.
   #
   # A large replica-validity-factor may allow replicas with too old data to failover
   # a master, while a too small value may prevent the cluster from being able to
   # elect a replica at all.
   #
   # For maximum availability, it is possible to set the replica-validity-factor
   # to a value of 0, which means, that replicas will always try to failover the
   # master regardless of the last time they interacted with the master.
   # (However they'll always try to apply a delay proportional to their
   # offset rank).
   #
   # Zero is the only value able to guarantee that when all the partitions heal
   # the cluster will always be able to continue.
   #
   # cluster-replica-validity-factor 10
   
   # Cluster replicas are able to migrate to orphaned masters, that are masters
   # that are left without working replicas. This improves the cluster ability
   # to resist to failures as otherwise an orphaned master can't be failed over
   # in case of failure if it has no working replicas.
   #
   # Replicas migrate to orphaned masters only if there are still at least a
   # given number of other working replicas for their old master. This number
   # is the "migration barrier". A migration barrier of 1 means that a replica
   # will migrate only if there is at least 1 other working replica for its master
   # and so forth. It usually reflects the number of replicas you want for every
   # master in your cluster.
   #
   # Default is 1 (replicas migrate only if their masters remain with at least
   # one replica). To disable migration just set it to a very large value.
   # A value of 0 can be set but is useful only for debugging and dangerous
   # in production.
   #
   # cluster-migration-barrier 1
   
   # By default Redis Cluster nodes stop accepting queries if they detect there
   # is at least an hash slot uncovered (no available node is serving it).
   # This way if the cluster is partially down (for example a range of hash slots
   # are no longer covered) all the cluster becomes, eventually, unavailable.
   # It automatically returns available as soon as all the slots are covered again.
   #
   # However sometimes you want the subset of the cluster which is working,
   # to continue to accept queries for the part of the key space that is still
   # covered. In order to do so, just set the cluster-require-full-coverage
   # option to no.
   #
   # cluster-require-full-coverage yes
   
   # This option, when set to yes, prevents replicas from trying to failover its
   # master during master failures. However the master can still perform a
   # manual failover, if forced to do so.
   #
   # This is useful in different scenarios, especially in the case of multiple
   # data center operations, where we want one side to never be promoted if not
   # in the case of a total DC failure.
   #
   # cluster-replica-no-failover no
   
   # In order to setup your cluster make sure to read the documentation
   # available at http://redis.io web site.
   
   ########################## CLUSTER DOCKER/NAT support  ########################
   
   # In certain deployments, Redis Cluster nodes address discovery fails, because
   # addresses are NAT-ted or because ports are forwarded (the typical case is
   # Docker and other containers).
   #
   # In order to make Redis Cluster working in such environments, a static
   # configuration where each node knows its public address is needed. The
   # following two options are used for this scope, and are:
   #
   # * cluster-announce-ip
   # * cluster-announce-port
   # * cluster-announce-bus-port
   #
   # Each instruct the node about its address, client port, and cluster message
   # bus port. The information is then published in the header of the bus packets
   # so that other nodes will be able to correctly map the address of the node
   # publishing the information.
   #
   # If the above options are not used, the normal Redis Cluster auto-detection
   # will be used instead.
   #
   # Note that when remapped, the bus port may not be at the fixed offset of
   # clients port + 10000, so you can specify any port and bus-port depending
   # on how they get remapped. If the bus-port is not set, a fixed offset of
   # 10000 will be used as usually.
   #
   # Example:
   #
   # cluster-announce-ip 10.1.1.5
   # cluster-announce-port 6379
   # cluster-announce-bus-port 6380
   
   ################################## SLOW LOG ###################################
   
   # The Redis Slow Log is a system to log queries that exceeded a specified
   # execution time. The execution time does not include the I/O operations
   # like talking with the client, sending the reply and so forth,
   # but just the time needed to actually execute the command (this is the only
   # stage of command execution where the thread is blocked and can not serve
   # other requests in the meantime).
   #
   # You can configure the slow log with two parameters: one tells Redis
   # what is the execution time, in microseconds, to exceed in order for the
   # command to get logged, and the other parameter is the length of the
   # slow log. When a new command is logged the oldest one is removed from the
   # queue of logged commands.
   
   # The following time is expressed in microseconds, so 1000000 is equivalent
   # to one second. Note that a negative number disables the slow log, while
   # a value of zero forces the logging of every command.
   slowlog-log-slower-than 10000
   
   # There is no limit to this length. Just be aware that it will consume memory.
   # You can reclaim memory used by the slow log with SLOWLOG RESET.
   slowlog-max-len 128
   
   ################################ LATENCY MONITOR ##############################
   
   # The Redis latency monitoring subsystem samples different operations
   # at runtime in order to collect data related to possible sources of
   # latency of a Redis instance.
   #
   # Via the LATENCY command this information is available to the user that can
   # print graphs and obtain reports.
   #
   # The system only logs operations that were performed in a time equal or
   # greater than the amount of milliseconds specified via the
   # latency-monitor-threshold configuration directive. When its value is set
   # to zero, the latency monitor is turned off.
   #
   # By default latency monitoring is disabled since it is mostly not needed
   # if you don't have latency issues, and collecting data has a performance
   # impact, that while very small, can be measured under big load. Latency
   # monitoring can easily be enabled at runtime using the command
   # "CONFIG SET latency-monitor-threshold <milliseconds>" if needed.
   latency-monitor-threshold 0
   
   ############################# EVENT NOTIFICATION ##############################
   
   # Redis can notify Pub/Sub clients about events happening in the key space.
   # This feature is documented at http://redis.io/topics/notifications
   #
   # For instance if keyspace events notification is enabled, and a client
   # performs a DEL operation on key "foo" stored in the Database 0, two
   # messages will be published via Pub/Sub:
   #
   # PUBLISH __keyspace@0__:foo del
   # PUBLISH __keyevent@0__:del foo
   #
   # It is possible to select the events that Redis will notify among a set
   # of classes. Every class is identified by a single character:
   #
   #  K     Keyspace events, published with __keyspace@<db>__ prefix.
   #  E     Keyevent events, published with __keyevent@<db>__ prefix.
   #  g     Generic commands (non-type specific) like DEL, EXPIRE, RENAME, ...
   #  $     String commands
   #  l     List commands
   #  s     Set commands
   #  h     Hash commands
   #  z     Sorted set commands
   #  x     Expired events (events generated every time a key expires)
   #  e     Evicted events (events generated when a key is evicted for maxmemory)
   #  A     Alias for g$lshzxe, so that the "AKE" string means all the events.
   #
   #  The "notify-keyspace-events" takes as argument a string that is composed
   #  of zero or multiple characters. The empty string means that notifications
   #  are disabled.
   #
   #  Example: to enable list and generic events, from the point of view of the
   #           event name, use:
   #
   #  notify-keyspace-events Elg
   #
   #  Example 2: to get the stream of the expired keys subscribing to channel
   #             name __keyevent@0__:expired use:
   #
   #  notify-keyspace-events Ex
   #
   #  By default all notifications are disabled because most users don't need
   #  this feature and the feature has some overhead. Note that if you don't
   #  specify at least one of K or E, no events will be delivered.
   notify-keyspace-events ""
   
   ############################### ADVANCED CONFIG ###############################
   
   # Hashes are encoded using a memory efficient data structure when they have a
   # small number of entries, and the biggest entry does not exceed a given
   # threshold. These thresholds can be configured using the following directives.
   hash-max-ziplist-entries 512
   hash-max-ziplist-value 64
   
   # Lists are also encoded in a special way to save a lot of space.
   # The number of entries allowed per internal list node can be specified
   # as a fixed maximum size or a maximum number of elements.
   # For a fixed maximum size, use -5 through -1, meaning:
   # -5: max size: 64 Kb  <-- not recommended for normal workloads
   # -4: max size: 32 Kb  <-- not recommended
   # -3: max size: 16 Kb  <-- probably not recommended
   # -2: max size: 8 Kb   <-- good
   # -1: max size: 4 Kb   <-- good
   # Positive numbers mean store up to _exactly_ that number of elements
   # per list node.
   # The highest performing option is usually -2 (8 Kb size) or -1 (4 Kb size),
   # but if your use case is unique, adjust the settings as necessary.
   list-max-ziplist-size -2
   
   # Lists may also be compressed.
   # Compress depth is the number of quicklist ziplist nodes from *each* side of
   # the list to *exclude* from compression.  The head and tail of the list
   # are always uncompressed for fast push/pop operations.  Settings are:
   # 0: disable all list compression
   # 1: depth 1 means "don't start compressing until after 1 node into the list,
   #    going from either the head or tail"
   #    So: [head]->node->node->...->node->[tail]
   #    [head], [tail] will always be uncompressed; inner nodes will compress.
   # 2: [head]->[next]->node->node->...->node->[prev]->[tail]
   #    2 here means: don't compress head or head->next or tail->prev or tail,
   #    but compress all nodes between them.
   # 3: [head]->[next]->[next]->node->node->...->node->[prev]->[prev]->[tail]
   # etc.
   list-compress-depth 0
   
   # Sets have a special encoding in just one case: when a set is composed
   # of just strings that happen to be integers in radix 10 in the range
   # of 64 bit signed integers.
   # The following configuration setting sets the limit in the size of the
   # set in order to use this special memory saving encoding.
   set-max-intset-entries 512
   
   # Similarly to hashes and lists, sorted sets are also specially encoded in
   # order to save a lot of space. This encoding is only used when the length and
   # elements of a sorted set are below the following limits:
   zset-max-ziplist-entries 128
   zset-max-ziplist-value 64
   
   # HyperLogLog sparse representation bytes limit. The limit includes the
   # 16 bytes header. When an HyperLogLog using the sparse representation crosses
   # this limit, it is converted into the dense representation.
   #
   # A value greater than 16000 is totally useless, since at that point the
   # dense representation is more memory efficient.
   #
   # The suggested value is ~ 3000 in order to have the benefits of
   # the space efficient encoding without slowing down too much PFADD,
   # which is O(N) with the sparse encoding. The value can be raised to
   # ~ 10000 when CPU is not a concern, but space is, and the data set is
   # composed of many HyperLogLogs with cardinality in the 0 - 15000 range.
   hll-sparse-max-bytes 3000
   
   # Streams macro node max size / items. The stream data structure is a radix
   # tree of big nodes that encode multiple items inside. Using this configuration
   # it is possible to configure how big a single node can be in bytes, and the
   # maximum number of items it may contain before switching to a new node when
   # appending new stream entries. If any of the following settings are set to
   # zero, the limit is ignored, so for instance it is possible to set just a
   # max entires limit by setting max-bytes to 0 and max-entries to the desired
   # value.
   stream-node-max-bytes 4096
   stream-node-max-entries 100
   
   # Active rehashing uses 1 millisecond every 100 milliseconds of CPU time in
   # order to help rehashing the main Redis hash table (the one mapping top-level
   # keys to values). The hash table implementation Redis uses (see dict.c)
   # performs a lazy rehashing: the more operation you run into a hash table
   # that is rehashing, the more rehashing "steps" are performed, so if the
   # server is idle the rehashing is never complete and some more memory is used
   # by the hash table.
   #
   # The default is to use this millisecond 10 times every second in order to
   # actively rehash the main dictionaries, freeing memory when possible.
   #
   # If unsure:
   # use "activerehashing no" if you have hard latency requirements and it is
   # not a good thing in your environment that Redis can reply from time to time
   # to queries with 2 milliseconds delay.
   #
   # use "activerehashing yes" if you don't have such hard requirements but
   # want to free memory asap when possible.
   activerehashing yes
   
   # The client output buffer limits can be used to force disconnection of clients
   # that are not reading data from the server fast enough for some reason (a
   # common reason is that a Pub/Sub client can't consume messages as fast as the
   # publisher can produce them).
   #
   # The limit can be set differently for the three different classes of clients:
   #
   # normal -> normal clients including MONITOR clients
   # replica  -> replica clients
   # pubsub -> clients subscribed to at least one pubsub channel or pattern
   #
   # The syntax of every client-output-buffer-limit directive is the following:
   #
   # client-output-buffer-limit <class> <hard limit> <soft limit> <soft seconds>
   #
   # A client is immediately disconnected once the hard limit is reached, or if
   # the soft limit is reached and remains reached for the specified number of
   # seconds (continuously).
   # So for instance if the hard limit is 32 megabytes and the soft limit is
   # 16 megabytes / 10 seconds, the client will get disconnected immediately
   # if the size of the output buffers reach 32 megabytes, but will also get
   # disconnected if the client reaches 16 megabytes and continuously overcomes
   # the limit for 10 seconds.
   #
   # By default normal clients are not limited because they don't receive data
   # without asking (in a push way), but just after a request, so only
   # asynchronous clients may create a scenario where data is requested faster
   # than it can read.
   #
   # Instead there is a default limit for pubsub and replica clients, since
   # subscribers and replicas receive data in a push fashion.
   #
   # Both the hard or the soft limit can be disabled by setting them to zero.
   client-output-buffer-limit normal 0 0 0
   client-output-buffer-limit replica 256mb 64mb 60
   client-output-buffer-limit pubsub 32mb 8mb 60
   
   # Client query buffers accumulate new commands. They are limited to a fixed
   # amount by default in order to avoid that a protocol desynchronization (for
   # instance due to a bug in the client) will lead to unbound memory usage in
   # the query buffer. However you can configure it here if you have very special
   # needs, such us huge multi/exec requests or alike.
   #
   # client-query-buffer-limit 1gb
   
   # In the Redis protocol, bulk requests, that are, elements representing single
   # strings, are normally limited ot 512 mb. However you can change this limit
   # here.
   #
   # proto-max-bulk-len 512mb
   
   # Redis calls an internal function to perform many background tasks, like
   # closing connections of clients in timeout, purging expired keys that are
   # never requested, and so forth.
   #
   # Not all tasks are performed with the same frequency, but Redis checks for
   # tasks to perform according to the specified "hz" value.
   #
   # By default "hz" is set to 10. Raising the value will use more CPU when
   # Redis is idle, but at the same time will make Redis more responsive when
   # there are many keys expiring at the same time, and timeouts may be
   # handled with more precision.
   #
   # The range is between 1 and 500, however a value over 100 is usually not
   # a good idea. Most users should use the default of 10 and raise this up to
   # 100 only in environments where very low latency is required.
   hz 10
   
   # Normally it is useful to have an HZ value which is proportional to the
   # number of clients connected. This is useful in order, for instance, to
   # avoid too many clients are processed for each background task invocation
   # in order to avoid latency spikes.
   #
   # Since the default HZ value by default is conservatively set to 10, Redis
   # offers, and enables by default, the ability to use an adaptive HZ value
   # which will temporary raise when there are many connected clients.
   #
   # When dynamic HZ is enabled, the actual configured HZ will be used as
   # as a baseline, but multiples of the configured HZ value will be actually
   # used as needed once more clients are connected. In this way an idle
   # instance will use very little CPU time while a busy instance will be
   # more responsive.
   dynamic-hz yes
   
   # When a child rewrites the AOF file, if the following option is enabled
   # the file will be fsync-ed every 32 MB of data generated. This is useful
   # in order to commit the file to the disk more incrementally and avoid
   # big latency spikes.
   aof-rewrite-incremental-fsync yes
   
   # When redis saves RDB file, if the following option is enabled
   # the file will be fsync-ed every 32 MB of data generated. This is useful
   # in order to commit the file to the disk more incrementally and avoid
   # big latency spikes.
   rdb-save-incremental-fsync yes
   
   # Redis LFU eviction (see maxmemory setting) can be tuned. However it is a good
   # idea to start with the default settings and only change them after investigating
   # how to improve the performances and how the keys LFU change over time, which
   # is possible to inspect via the OBJECT FREQ command.
   #
   # There are two tunable parameters in the Redis LFU implementation: the
   # counter logarithm factor and the counter decay time. It is important to
   # understand what the two parameters mean before changing them.
   #
   # The LFU counter is just 8 bits per key, it's maximum value is 255, so Redis
   # uses a probabilistic increment with logarithmic behavior. Given the value
   # of the old counter, when a key is accessed, the counter is incremented in
   # this way:
   #
   # 1. A random number R between 0 and 1 is extracted.
   # 2. A probability P is calculated as 1/(old_value*lfu_log_factor+1).
   # 3. The counter is incremented only if R < P.
   #
   # The default lfu-log-factor is 10. This is a table of how the frequency
   # counter changes with a different number of accesses with different
   # logarithmic factors:
   #
   # +--------+------------+------------+------------+------------+------------+
   # | factor | 100 hits   | 1000 hits  | 100K hits  | 1M hits    | 10M hits   |
   # +--------+------------+------------+------------+------------+------------+
   # | 0      | 104        | 255        | 255        | 255        | 255        |
   # +--------+------------+------------+------------+------------+------------+
   # | 1      | 18         | 49         | 255        | 255        | 255        |
   # +--------+------------+------------+------------+------------+------------+
   # | 10     | 10         | 18         | 142        | 255        | 255        |
   # +--------+------------+------------+------------+------------+------------+
   # | 100    | 8          | 11         | 49         | 143        | 255        |
   # +--------+------------+------------+------------+------------+------------+
   #
   # NOTE: The above table was obtained by running the following commands:
   #
   #   redis-benchmark -n 1000000 incr foo
   #   redis-cli object freq foo
   #
   # NOTE 2: The counter initial value is 5 in order to give new objects a chance
   # to accumulate hits.
   #
   # The counter decay time is the time, in minutes, that must elapse in order
   # for the key counter to be divided by two (or decremented if it has a value
   # less <= 10).
   #
   # The default value for the lfu-decay-time is 1. A Special value of 0 means to
   # decay the counter every time it happens to be scanned.
   #
   # lfu-log-factor 10
   # lfu-decay-time 1
   
   ########################### ACTIVE DEFRAGMENTATION #######################
   #
   # WARNING THIS FEATURE IS EXPERIMENTAL. However it was stress tested
   # even in production and manually tested by multiple engineers for some
   # time.
   #
   # What is active defragmentation?
   # -------------------------------
   #
   # Active (online) defragmentation allows a Redis server to compact the
   # spaces left between small allocations and deallocations of data in memory,
   # thus allowing to reclaim back memory.
   #
   # Fragmentation is a natural process that happens with every allocator (but
   # less so with Jemalloc, fortunately) and certain workloads. Normally a server
   # restart is needed in order to lower the fragmentation, or at least to flush
   # away all the data and create it again. However thanks to this feature
   # implemented by Oran Agra for Redis 4.0 this process can happen at runtime
   # in an "hot" way, while the server is running.
   #
   # Basically when the fragmentation is over a certain level (see the
   # configuration options below) Redis will start to create new copies of the
   # values in contiguous memory regions by exploiting certain specific Jemalloc
   # features (in order to understand if an allocation is causing fragmentation
   # and to allocate it in a better place), and at the same time, will release the
   # old copies of the data. This process, repeated incrementally for all the keys
   # will cause the fragmentation to drop back to normal values.
   #
   # Important things to understand:
   #
   # 1. This feature is disabled by default, and only works if you compiled Redis
   #    to use the copy of Jemalloc we ship with the source code of Redis.
   #    This is the default with Linux builds.
   #
   # 2. You never need to enable this feature if you don't have fragmentation
   #    issues.
   #
   # 3. Once you experience fragmentation, you can enable this feature when
   #    needed with the command "CONFIG SET activedefrag yes".
   #
   # The configuration parameters are able to fine tune the behavior of the
   # defragmentation process. If you are not sure about what they mean it is
   # a good idea to leave the defaults untouched.
   
   # Enabled active defragmentation
   # activedefrag yes
   
   # Minimum amount of fragmentation waste to start active defrag
   # active-defrag-ignore-bytes 100mb
   
   # Minimum percentage of fragmentation to start active defrag
   # active-defrag-threshold-lower 10
   
   # Maximum percentage of fragmentation at which we use maximum effort
   # active-defrag-threshold-upper 100
   
   # Minimal effort for defrag in CPU percentage
   # active-defrag-cycle-min 5
   
   # Maximal effort for defrag in CPU percentage
   # active-defrag-cycle-max 75
   
   # Maximum number of set/hash/zset/list fields that will be processed from
   # the main dictionary scan
   # active-defrag-max-scan-fields 1000
   
   # It is possible to pin different threads and processes of Redis to specific
   # CPUs in your system, in order to maximize the performances of the server.
   # This is useful both in order to pin different Redis threads in different
   # CPUs, but also in order to make sure that multiple Redis instances running
   # in the same host will be pinned to different CPUs.
   #
   # Normally you can do this using the "taskset" command, however it is also
   # possible to this via Redis configuration directly, both in Linux and FreeBSD.
   #
   # You can pin the server/IO threads, bio threads, aof rewrite child process, and
   # the bgsave child process. The syntax to specify the cpu list is the same as
   # the taskset command:
   #
   # Set redis server/io threads to cpu affinity 0,2,4,6:
   # server_cpulist 0-7:2
   #
   # Set bio threads to cpu affinity 1,3:
   # bio_cpulist 1,3
   #
   # Set aof rewrite child process to cpu affinity 8,9,10,11:
   # aof_rewrite_cpulist 8-11
   #
   # Set bgsave child process to cpu affinity 1,10,11
   # bgsave_cpulist 1,10-11
   
   # In some cases redis will emit warnings and even refuse to start if it detects
   # that the system is in bad state, it is possible to suppress these warnings
   # by setting the following config which takes a space delimited list of warnings
   # to suppress
   #
   # ignore-warnings ARM64-COW-BUG
   ```

   将其上传至刚刚创建的目录下

   ```shell
   [root@iZuf67z0nmo18acuap4yhxZ redis]# ll
   -rw-r--r-- 1 root root 64491 May 29 23:21 redis-cluster.tmpl
   ```

4. 由于此处所有redis实例运行在同一台宿主机，而redis-cluster之间需要通信，所以需要创建docker network

   ```shell
   # 创建docker内部网络
   docker network create redis-cluster-net
   
   #查看docker网络列表
   [root@iZuf67z0nmo18acuap4yhxZ redis]# docker network ls
   NETWORK ID          NAME                DRIVER              SCOPE
   71b8734768e3        bridge              bridge              local
   68e5a13b40d7        host                host                local
   67e3a52c6c66        none                null                local
   a1caa472608f        redis-cluster-net   bridge              local      #我们刚刚创建的网络
   
   #查看redis-cluster-net网络信息，可以发现Containers字段中并没有容器连接进来
   [root@iZuf67z0nmo18acuap4yhxZ redis]# docker inspect redis-cluster-net
   [
       {
           "Name": "redis-cluster-net",
           "Id": "a1caa472608f447164d20744e5253a1346547e8f13826d837a885669afddd8e5",
           "Created": "2021-05-29T21:00:22.83740273+08:00",
           "Scope": "local",
           "Driver": "bridge",
           "EnableIPv6": false,
           "IPAM": {
               "Driver": "default",
               "Options": {},
               "Config": [
                   {
                       "Subnet": "172.18.0.0/16",
                       "Gateway": "172.18.0.1"
                   }
               ]
           },
           "Internal": false,
           "Attachable": false,
           "Containers": {},
           "Options": {},
           "Labels": {}
       }
   ]
   
   ```

5. 创建 master 和 slave 文件夹

```shell
[root@iZuf67z0nmo18acuap4yhxZ redis]# 
for port in `seq 7000 7005`; do
    ms="master"
    if [ $port -ge 7003 ]; then
        ms="slave"
    fi
    mkdir -p ./$ms/$port/ && mkdir -p ./$ms/$port/data \
    && PORT=$port envsubst < ./redis-cluster.tmpl > ./$ms/$port/redis.conf;
done
```

6. 运行docker redis 的 master 和 slave 实例

```shell
[root@iZuf67z0nmo18acuap4yhxZ redis]# 
for port in `seq 7000 7005`; do
    ms="master"
    if [ $port -ge 7003 ]; then
        ms="slave"
    fi
    docker run -d -p $port:$port -p 1$port:1$port \
    -v $PWD/$ms/$port/redis.conf:/data/redis.conf \
    -v $PWD/$ms/$port/data:/data/redis \
    --restart always --name redis-$ms-$port --net redis-cluster-net \
    redis:5.0.12 redis-server /data/redis.conf;
done
```

7. 查看网络映射关系

```shell
#发现Containers中已经多出来6个容器了
[root@iZuf67z0nmo18acuap4yhxZ redis]# docker inspect redis-cluster-net
[
    {
        "Name": "redis-cluster-net",
        "Id": "a1caa472608f447164d20744e5253a1346547e8f13826d837a885669afddd8e5",
        "Created": "2021-05-29T21:00:22.83740273+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Containers": {
            "22fc38f227d8db3fe13dcd7ea6a21b0d7b8aaf5e1171e04d96e8b42ad83dc806": {
                "Name": "redis-slave-7004",
                "EndpointID": "3d63aedfd3b58aebcaafcbd0ecaeb5366d781fd585f2dccc1ee4178120e00e1b",
                "MacAddress": "02:42:ac:12:00:06",
                "IPv4Address": "172.18.0.6/16",
                "IPv6Address": ""
            },
            "31ea375f100ea26aa55d9db60b1ed81fa25cd2e5b5a8883eb36dee98f3c7cb3d": {
                "Name": "redis-slave-7005",
                "EndpointID": "0f78236c23b399b71ed9bd8324121b73638f262185bdbc33bb4937080ccb6485",
                "MacAddress": "02:42:ac:12:00:07",
                "IPv4Address": "172.18.0.7/16",
                "IPv6Address": ""
            },
            "84d106937cb56e6e3edbccf44c242b179b6e984c7d59ef1d4c556e80513d321d": {
                "Name": "redis-master-7001",
                "EndpointID": "e0925d49ce2cdd913ae278454e8895d580e25b34412c6ce1bc3ef55534266017",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            },
            "9f5fa71394676377fccdd2fb942acd611600c0f71d087c525a2b150c53f14a21": {
                "Name": "redis-slave-7003",
                "EndpointID": "b344ff8cb5794d1f8747a95fd0d7c9a16850b4d106f15205b051006df6a5fac8",
                "MacAddress": "02:42:ac:12:00:05",
                "IPv4Address": "172.18.0.5/16",
                "IPv6Address": ""
            },
            "b809a322e40e904556455bdf5dd87817a5e9f3ef54b5250abe4e6e5d46c1c921": {
                "Name": "redis-master-7000",
                "EndpointID": "60935731c74814e8d67dac192d38015ac29aa84269dbfdb32effe86971724676",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            },
            "ba25d69373bcdb52359539b2b1733152261d87015912283afff1f9279f338574": {
                "Name": "redis-master-7002",
                "EndpointID": "1a13f374dadb608d887aa6b877c375b5095e4662a6e8dec0a8cfe6907e2e29c2",
                "MacAddress": "02:42:ac:12:00:04",
                "IPv4Address": "172.18.0.4/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

8. 进入任意一个容器内

   ```shell
   #查看运行的容器
   [root@iZuf67z0nmo18acuap4yhxZ redis]# docker ps
   CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                        NAMES
   31ea375f100e        redis:5.0.12        "docker-entrypoint..."   52 minutes ago      Up 52 minutes       0.0.0.0:7005->7005/tcp, 6379/tcp, 0.0.0.0:17005->17005/tcp   redis-slave-7005
   22fc38f227d8        redis:5.0.12        "docker-entrypoint..."   52 minutes ago      Up 52 minutes       0.0.0.0:7004->7004/tcp, 6379/tcp, 0.0.0.0:17004->17004/tcp   redis-slave-7004
   9f5fa7139467        redis:5.0.12        "docker-entrypoint..."   52 minutes ago      Up 52 minutes       0.0.0.0:7003->7003/tcp, 6379/tcp, 0.0.0.0:17003->17003/tcp   redis-slave-7003
   ba25d69373bc        redis:5.0.12        "docker-entrypoint..."   52 minutes ago      Up 52 minutes       0.0.0.0:7002->7002/tcp, 6379/tcp, 0.0.0.0:17002->17002/tcp   redis-master-7002
   84d106937cb5        redis:5.0.12        "docker-entrypoint..."   52 minutes ago      Up 52 minutes       0.0.0.0:7001->7001/tcp, 6379/tcp, 0.0.0.0:17001->17001/tcp   redis-master-7001
   b809a322e40e        redis:5.0.12        "docker-entrypoint..."   52 minutes ago      Up 52 minutes       0.0.0.0:7000->7000/tcp, 6379/tcp, 0.0.0.0:17000->17000/tcp   redis-master-7000
   
   #进入任意一个容器,并且创建集群
   [root@iZuf67z0nmo18acuap4yhxZ redis]# docker exec -it redis-slave-7005 /bin/sh
   # ls
   redis  redis.conf
   # redis-cli --cluster create 172.18.0.6:7004 172.18.0.2:7000 172.18.0.3:7001 172.18.0.4:7002 172.18.0.7:7005 172.18.0.5:7003 --cluster-replicas 1
   
   ###备注：此处的  172.18.0.6:7004 172.18.0.2:7000 172.18.0.3:7001 172.18.0.4:7002 172.18.0.7:7005 172.18.0.5:7003  来自于  docker inspect redis-cluster-net  中的Containers中
   
   ###通过以下任意一条命令进入集群中，其中IP和port来自于上面的命令，---》》》注意修改《《《---
   ###格式如下redis-cli -c -h IP -p port -a password     如果有密码需要输入密码
   
   # redis-cli -c -h 172.18.0.2 -p 7000  
   172.18.0.2:7000> cluster nodes              		#查看集群信息
   4357037dc4760134a25d884d77c009f0309c50da 172.18.0.5:7003@17003 slave 2d4ad4b867bf2ce387af7080b8273ee6d2deb7c4 0 1622305162139 6 connected
   ac5973532bed8eda17e1b72da4f9bdd50c7750fb 172.18.0.6:7004@17004 master - 0 1622305161000 1 connected 0-5460
   2d4ad4b867bf2ce387af7080b8273ee6d2deb7c4 172.18.0.2:7000@17000 myself,master - 0 1622305160000 2 connected 5461-10922
   14620b48ab252860550e5b0eb2e5a40207d024b0 172.18.0.3:7001@17001 master - 0 1622305163140 3 connected 10923-16383
   fe9039b5f57fc5cfc62c6978baae5b4cb6fb281e 172.18.0.7:7005@17005 slave ac5973532bed8eda17e1b72da4f9bdd50c7750fb 0 1622305162000 5 connected
   f69657190a3b479543777f84fa88cdae1392a4ad 172.18.0.4:7002@17002 slave 14620b48ab252860550e5b0eb2e5a40207d024b0 0 1622305161000 4 connected
   ```

9. 进行操作集群

   ```shell
   172.18.0.2:7000> set key2 555
   -> Redirected to slot [4998] located at 172.18.0.6:7004
   OK
   172.18.0.6:7004> get key2
   "555"
   ```

#### **redis实例运行在不同的宿主机上**

 这里我将3个master实例运行在一台机(10.82.12.95)上，3个slave实例运行在另一台机器(10.82.12.98)上

1. 在两台机器上分别创建文件夹

   ```shell
   # 创建文件夹
   for port in `seq 7000 7002`; do
       mkdir -p ./$port/ && mkdir -p ./$port/data \
       && PORT=$port envsubst < ./redis-cluster.tmpl > ./$port/redis.conf;
   done
   ```

2. 在两台机器上分别运行docker redis 实例，注意这里就没有使用docker network了，直接使用的是宿主机的host，原因是要在不同机器的docker容器中通信是很麻烦的，有兴趣的朋友可以看下相关文章

   ```shell
   # 运行docker redis 实例
   for port in `seq 7000 7002`; do
       docker run -d \
       -v $PWD/$port/redis.conf:/data/redis.conf \
       -v $PWD/$port/data:/data/redis \
       --restart always --name redis-$port --net host \
       redis redis-server /data/redis.conf;
   done
   ```

3. 在任意一台机器上创建cluster

   ```shell
   # 创建cluster
   [root@iZuf67z0nmo18acuap4yhxZ redis]# docker exec -it redis-7000 /bin/sh
   # ls
   redis  redis.conf
   # redis-cli --cluster create 10.82.12.95:7000 10.82.12.95:7001 10.82.12.95:7002 10.82.12.98:7000 10.82.12.98:7001 10.82.12.98:7002 --cluster-replicas 1
   ```

4. 后面的操作和**所有redis实例运行在同一台宿主机上**节是一样的了

### MySQL主从复制搭建

#### 主数据库搭建

**拉取镜像**

```shell
[root@iZuf67z0nmo18acuap4yhxZ ~]# docker pull mysql:5.7
```

**运行mysql主实例**

```shell
[root@iZuf67z0nmo18acuap4yhxZ ~]# docker run -p 3307:3306 --name mysql-master \
-v /home/mysql/mysql-master/log:/var/log/mysql \
-v /home/mysql/mysql-master/data:/var/lib/mysql \
-v /home/mysql/mysql-master/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root  \
-d mysql:5.7
```

**在mysql的配置文件夹`/home/mysql/mysql-master/conf`中创建一个配置文件`my.cnf`**

```shell
[root@iZuf67z0nmo18acuap4yhxZ ~]# cd /home/mysql/mysql-master/conf

[root@iZuf67z0nmo18acuap4yhxZ ~]# touch my.cnf
[mysqld]
## 设置server_id，同一局域网中需要唯一
server_id=101 
## 指定不需要同步的数据库名称
binlog-ignore-db=mysql  
## 开启二进制日志功能
log-bin=mall-mysql-bin  
## 设置二进制日志使用内存大小（事务）
binlog_cache_size=1M  
## 设置使用的二进制日志格式（mixed,statement,row）
binlog_format=mixed  
## 二进制日志过期清理时间。默认值为0，表示不自动清理。
expire_logs_days=7  
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062
```

**修改完配置后重启实例**

```shell
[root@iZuf67z0nmo18acuap4yhxZ conf]# docker restart mysql-master
mysql-master
```

**进入`mysql-master`容器中**

```shell
[root@iZuf67z0nmo18acuap4yhxZ conf]# docker exec -it mysql-master /bin/bash
```

**在容器中使用mysql的登录命令连接到客户端**

```shell
oot@b49a8eca34fa:/# mysql -uroot -proot
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.34-log MySQL Community Server (GPL)

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

**创建数据同步用户**

```shell
mysql> CREATE USER 'slave'@'%' IDENTIFIED BY '123456';
Query OK, 0 rows affected (0.01 sec)

mysql> GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';
Query OK, 0 rows affected (0.00 sec)
```

#### 从数据库搭建

**运行mysql从实例**

```shell
[root@iZuf67z0nmo18acuap4yhxZ mysql]# docker run -p 3308:3306 --name mysql-slave \
-v /home/mysql/mysql-slave/log:/var/log/mysql \
-v /home/mysql/mysql-slave/data:/var/lib/mysql \
-v /home/mysql/mysql-slave/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root  \
-d mysql:5.7
```

**在mysql的配置文件夹`/home/mysql/mysql-slave/conf`中创建一个配置文件`my.cnf`**

```shell
[root@iZuf67z0nmo18acuap4yhxZ conf]# touch my.cnf

[root@iZuf67z0nmo18acuap4yhxZ conf]# cat my.cnf 
[mysqld]
## 设置server_id，同一局域网中需要唯一
server_id=102
## 指定不需要同步的数据库名称
binlog-ignore-db=mysql  
## 开启二进制日志功能，以备Slave作为其它数据库实例的Master时使用
log-bin=mall-mysql-slave1-bin  
## 设置二进制日志使用内存大小（事务）
binlog_cache_size=1M  
## 设置使用的二进制日志格式（mixed,statement,row）
binlog_format=mixed  
## 二进制日志过期清理时间。默认值为0，表示不自动清理。
expire_logs_days=7  
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062  
## relay_log配置中继日志
relay_log=mall-mysql-relay-bin  
## log_slave_updates表示slave将复制事件写进自己的二进制日志
log_slave_updates=1  
## slave设置为只读（具有super权限的用户除外）
read_only=1
```

**修改完配置后重启实例**

```shell
[root@iZuf67z0nmo18acuap4yhxZ conf]# docker restart mysql-slave
mysql-slave
```

**进入主数据库容器**

```shell
#进入容器
[root@iZuf67z0nmo18acuap4yhxZ conf]# docker exec -it mysql-master /bin/bash
root@b49a8eca34fa:/# mysql -uroot -proot
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.34-log MySQL Community Server (GPL)

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 

#查看master状态信息
mysql> show master status;
+-----------------------+----------+--------------+------------------+-------------------+
| File                  | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+-----------------------+----------+--------------+------------------+-------------------+
| mall-mysql-bin.000001 |      617 |              | mysql            |                   |
+-----------------------+----------+--------------+------------------+-------------------+

```

![21a5dfa8363af047eda78df9db0c4cd](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/21a5dfa8363af047eda78df9db0c4cd.png)

**进入从数据库容器**

```shell
[root@iZuf67z0nmo18acuap4yhxZ conf]# docker exec -it mysql-slave /bin/bash

#在容器中使用mysql的登录命令连接到客户端
root@68618cf875e0:/# mysql -uroot -proot
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.34-log MySQL Community Server (GPL)

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>

#在从数据库中配置主从复制
mysql> change master to master_host='172.28.128.216', master_user='slave', master_password='123456', master_port=3307, master_log_file='mall-mysql-bin.000001', master_log_pos=617, master_connect_retry=30;
```

**主从复制命令参数说明：**

master_host：主数据库的IP地址；
master_port：主数据库的运行端口；
master_user：在主数据库创建的用于同步数据的用户账号；
master_password：在主数据库创建的用于同步数据的用户密码；
master_log_file：指定从数据库要复制数据的日志文件，通过查看主数据的状态，获取File参数；
master_log_pos：指定从数据库从哪个位置开始复制数据，通过查看主数据的状态，获取Position参数；
master_connect_retry：连接失败重试的时间间隔，单位为秒。

**查看主从同步状态**

```shell
mysql> show slave status \G;
*************************** 1. row ***************************
               Slave_IO_State: 
                  Master_Host: 172.28.128.216
                  Master_User: slave
                  Master_Port: 3307
                Connect_Retry: 30
              Master_Log_File: mall-mysql-bin.000001
          Read_Master_Log_Pos: 617
               Relay_Log_File: mall-mysql-relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mall-mysql-bin.000001
             Slave_IO_Running: No
            Slave_SQL_Running: No
.... 省略
```

![b4b2e475797be4f3d84c523189de650](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/b4b2e475797be4f3d84c523189de650.png)

**开启主从同步**

```shell
mysql> start slave;
Query OK, 0 rows affected (0.00 sec)
```

**再次查看从数据库状态**

```shell
mysql> show slave status \G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 172.28.128.216
                  Master_User: slave
                  Master_Port: 3307
                Connect_Retry: 30
              Master_Log_File: mall-mysql-bin.000001
          Read_Master_Log_Pos: 617
               Relay_Log_File: mall-mysql-relay-bin.000002
                Relay_Log_Pos: 325
        Relay_Master_Log_File: mall-mysql-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
...省略
```

![711312b8ad3867cd49be148ddf8e964](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/711312b8ad3867cd49be148ddf8e964.png)

**测试**

```shell
#进入主容器数据库新增blog数据库
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)

mysql> create database blog;
Query OK, 1 row affected (0.00 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| blog               |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

#查看从库数据库
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| blog               |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
```

### Minio文件服务器搭建

**搜索镜像的命令**

```shell
[root@iZuf67z0nmo18acuap4yhxZ conf]# docker search minio
```

**拉取镜像的命令**

```shell
[root@iZuf67z0nmo18acuap4yhxZ conf]# docker pull minio/minio
```

**启动容器**

```shell
[root@iZuf67z0nmo18acuap4yhxZ conf]# docker run -p 9000:9000 --name minio  -d --restart=always  -e "MINIO_ACCESS_KEY=root23"  -e "MINIO_SECRET_KEY=root23"  -v /home/minio/data:/data  -v /home/minio/config:/root/.minio  minio/minio server /data
```

**名词解释**

 -p ：内部端口:外部端口

--name ：容器名称

-d：后台运行

--restart=always：总是重启

 -e：配置环境

​		"MINIO_ACCESS_KEY=root123"  账号

​		"MINIO_SECRET_KEY=root123"  密码

-v ： 文件挂载到宿主机上

**查看容器运行状态**

```shell
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                        NAMES
5317d797e59c        minio/minio         "/usr/bin/docker-e..."   17 hours ago        Up 17 hours         0.0.0.0:9000->9000/tcp                                       minio
```

**浏览器访问服务**

`http://136.14.69.83:9000/minio/login`

![9b6cce18423b8ecc0f20d5fd3c4634e](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/9b6cce18423b8ecc0f20d5fd3c4634e.png)

至于使用，整合springboot等，可以去百度

### MongoDB搭建

**拉取镜像**

```shel
[root@iZuf67z0nmo18acuap4yhxZ /]# docker pull mongo
```

**创建文件夹**

```shell
#分别创建两个文件夹用于数据持久化
/home/mongodb/data
/home/mongodb/backup
```

**启动容器**

```shell
[root@iZuf67z0nmo18acuap4yhxZ /]# docker run --name mongo -p 27017:27017 -v /home/mongodb/data:/data/db -v /home/mongodb/backup:/data/backup -d mongo --auth

#备注：--auth代表是否进行账号密码验证
```

**进入容器**

```shell
[root@iZuf67z0nmo18acuap4yhxZ data]# docker exec -it mongo mongo admin
MongoDB shell version v4.4.6
connecting to: mongodb://127.0.0.1:27017/admin?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("c766e82b-2e6f-45b0-996a-232b19708f5d") }
MongoDB server version: 4.4.6
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
	https://docs.mongodb.com/
Questions? Try the MongoDB Developer Community Forums
	https://community.mongodb.com
>
```

**创建用户并且构建数据库**

```shell

> db.createUser({ user: 'zjm', pwd: 'zjm123', roles: [ { role: "userAdminAnyDatabase", db: "admin" } ] });
Successfully added user: {
	"user" : "zjm",
	"roles" : [
		{
			"role" : "userAdminAnyDatabase",
			"db" : "admin"
		}
	]
}

#进入管理员数据库
> use admin
switched to db admin

#进行验证
> db.auth("zjm","zjm123")
1

#使用测试dbtest
> use dbtest
switched to db dbtest

#创建用户
> db.createUser({user:"testuser",pwd:"testpass",roles:["readWrite"]});
Successfully added user: { "user" : "testuser", "roles" : [ "readWrite" ] }
```

**使用Navicat for MongoDB连接**

![b643897b8391f97cf577d90c0dedbe8](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/b643897b8391f97cf577d90c0dedbe8.png)

## springboot打包docker镜像

**构建springboot项目**

```java
//新建一个TestController
package com.example.demo;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @Author zjm
 * @Description:
 * @Date: Created in 19:48 2021/5/30
 * @Modified By:
 */
@RestController
public class TestController {


    @RequestMapping("/hello")
    public String test(){
        return "hello word!";
    }

}
```

**打jar包**

![202455fd47a7943c4b3bc7f2e8bf2a5](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/202455fd47a7943c4b3bc7f2e8bf2a5.png)

**安装docker插件**

![ce2866956898a2d266d1d527c220560](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/ce2866956898a2d266d1d527c220560.png)

**编写Dockerfile文件，内容如下**

```shell
FROM java:8

COPY *.jar /app.jar

CMD ["--server.port=8080"]

EXPOSE 8080

ENTRYPOINT ["java","-jar","/app.jar"]
```

![c10df63c6f19cecfbf10e1b9e4a5129](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/c10df63c6f19cecfbf10e1b9e4a5129.png)

**上传到指定文件夹下**

```shell
[root@iZuf67z0nmo18acuap4yhxZ project]# ll
total 16332
-rw-r--r-- 1 root root 16716884 May 30 20:06 demo-0.0.1-SNAPSHOT.jar
-rw-r--r-- 1 root root      120 May 30 20:08 Dockerfile
```

**构建镜像**

```shell
[root@iZuf67z0nmo18acuap4yhxZ project]# docker build -t demoweb .

[root@iZuf67z0nmo18acuap4yhxZ project]# docker images
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
demoweb                 latest              118d4ca976f8        8 seconds ago       660 MB
```

**运行容器**

```shell
[root@iZuf67z0nmo18acuap4yhxZ project]# docker run -d -P --name domeweb demoweb

[root@iZuf67z0nmo18acuap4yhxZ project]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                        NAMES
588d8dbc9d3b        demoweb             "java -jar /app.ja..."   26 seconds ago      Up 26 seconds       0.0.0.0:32768->8080/tcp                                      domeweb
```

**访问项目**

```shell
[root@iZuf67z0nmo18acuap4yhxZ project]# curl localhost:32768/hello
hello word!
```

# 附录

## 整体架构图

![](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/src=http___www.51wendang.com_pic_147bc150ad876beb1a56da5c_2-485-png_6_0_0_0_0_0_0_892.979_1262.879-893-0-272-893.jpg&refer=http___www.51wendang.jpg)

## docker命令图

![](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/src=http___www.jqhtml.com_wp-content_uploads_2019_5_zUfQre.png&refer=http___www.jqhtml.jpg)

## 参考书籍



![](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20210530202215.jpg)

## 参考视频

【【狂神说Java】Docker最新超详细版教程通俗易懂-哔哩哔哩】https://b23.tv/a2jm1b

## 参考资料

[]: https://vuepress.mirror.docker-practice.com/	"Docker 从入门到实践"

