# 基础入门

## 安装配置

### 安装docker引擎

#### CentOS环境下安装Docker

```shell
#首先，为了方便添加软件园，以及支持devicemapper存储类型，安装如下软件包
$ sudo yum update
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2

#添加Docker稳定版本的yum软件园
$ sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

#之后更新yum软件缓存，并安装Docker
$ sudo yum update
$ sudo yum install -y docker-ce

#最后确认Docker服务启动
$ sudo systemctl start docker

```



```shell
#docker配置阿里云镜像源(如果自己的服务器是阿里云服务器)
#下方中的https://f9dk003m.mirror.aliyuncs.com需要自己去阿里云平台搜索

$ sudo mkdir -p /etc/docker
$ sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://f9dk003m.mirror.aliyuncs.com"]
}
EOF
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

<img src="https://upload-images.jianshu.io/upload_images/18259896-15d5cd4f39a3c862.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200" style="zoom:200%;" />

## 使用Docker镜像

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
#可以看到另两个的镜像ID是一样的，实际上指向的都是同一个文件
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
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker images    				#此时可以看到myubuntu标签的不见了
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

#以交互的形式启动容器
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
#Dockerfile是一个文本文件，利用制定的指令描述基于父镜像创建新镜像的过程
#基于debian:stretch-slim镜像安装Python 3 环境，构成一个新的Python 3镜像

[root@iZuf6jcqwirs6izt7krfcmZ home]# vim dockerfile 											#创建文本文件

[root@iZuf6jcqwirs6izt7krfcmZ home]# cat dockerfile 											#查看内容
FROM debian:stretch-slim
LABEL version="1.0" maintainer="docker user <docker_user@github>"
RUN apt-get update && apt-get install -y python3 && apt-get clean && rm -rf /var/lib/apt/lists/*

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
Ign:1 http://deb.debian.org/debian stretch InRelease
Get:2 http://security.debian.org/debian-security stretch/updates InRelease [53.0 kB]
Get:3 http://deb.debian.org/debian stretch-updates InRelease [93.6 kB]
Get:4 http://security.debian.org/debian-security stretch/updates/main amd64 Packages [686 kB]
Get:5 http://deb.debian.org/debian stretch Release [118 kB]
Get:6 http://deb.debian.org/debian stretch Release.gpg [2410 B]
Get:7 http://deb.debian.org/debian stretch/main amd64 Packages [7080 kB]
Fetched 8032 kB in 13s (579 kB/s)
Reading package lists...
Reading package lists...
Building dependency tree...
Reading state information...
The following additional packages will be installed:
  bzip2 dh-python file libexpat1 libmagic-mgc libmagic1 libmpdec2
  libpython3-stdlib libpython3.5-minimal libpython3.5-stdlib libreadline7
  libsqlite3-0 libssl1.1 mime-support python3-minimal python3.5
  python3.5-minimal readline-common xz-utils
Suggested packages:
  bzip2-doc libdpkg-perl python3-doc python3-tk python3-venv python3.5-venv
  python3.5-doc binutils binfmt-support readline-doc
The following NEW packages will be installed:
  bzip2 dh-python file libexpat1 libmagic-mgc libmagic1 libmpdec2
  libpython3-stdlib libpython3.5-minimal libpython3.5-stdlib libreadline7
  libsqlite3-0 libssl1.1 mime-support python3 python3-minimal python3.5
  python3.5-minimal readline-common xz-utils
0 upgraded, 20 newly installed, 0 to remove and 1 not upgraded.
Need to get 7897 kB of archives.
After this operation, 36.7 MB of additional disk space will be used.
Get:1 http://security.debian.org/debian-security stretch/updates/main amd64 libssl1.1 amd64 1.1.0l-1~deb9u3 [1359 kB]
Get:2 http://deb.debian.org/debian stretch/main amd64 libexpat1 amd64 2.2.0-2+deb9u3 [83.7 kB]
Get:3 http://deb.debian.org/debian stretch/main amd64 python3-minimal amd64 3.5.3-1 [35.3 kB]
Get:4 http://deb.debian.org/debian stretch/main amd64 mime-support all 3.60 [36.7 kB]
Get:5 http://deb.debian.org/debian stretch/main amd64 libmpdec2 amd64 2.4.2-1 [85.2 kB]
Get:6 http://deb.debian.org/debian stretch/main amd64 readline-common all 7.0-3 [70.4 kB]
Get:7 http://deb.debian.org/debian stretch/main amd64 libreadline7 amd64 7.0-3 [151 kB]
Get:8 http://security.debian.org/debian-security stretch/updates/main amd64 libpython3.5-minimal amd64 3.5.3-1+deb9u4 [574 kB]
Get:9 http://deb.debian.org/debian stretch/main amd64 libpython3-stdlib amd64 3.5.3-1 [18.6 kB]
Get:10 http://deb.debian.org/debian stretch/main amd64 dh-python all 2.20170125 [86.8 kB]
Get:11 http://deb.debian.org/debian stretch/main amd64 python3 amd64 3.5.3-1 [21.6 kB]
Get:12 http://deb.debian.org/debian stretch/main amd64 bzip2 amd64 1.0.6-8.1 [47.5 kB]
Get:13 http://deb.debian.org/debian stretch/main amd64 libmagic-mgc amd64 1:5.30-1+deb9u3 [222 kB]
Get:14 http://deb.debian.org/debian stretch/main amd64 libmagic1 amd64 1:5.30-1+deb9u3 [111 kB]
Get:15 http://security.debian.org/debian-security stretch/updates/main amd64 python3.5-minimal amd64 3.5.3-1+deb9u4 [1692 kB]
Get:16 http://deb.debian.org/debian stretch/main amd64 file amd64 1:5.30-1+deb9u3 [64.2 kB]
Get:17 http://deb.debian.org/debian stretch/main amd64 xz-utils amd64 5.2.2-1.2+b1 [266 kB]
Get:18 http://security.debian.org/debian-security stretch/updates/main amd64 libsqlite3-0 amd64 3.16.2-5+deb9u3 [574 kB]
Get:19 http://security.debian.org/debian-security stretch/updates/main amd64 libpython3.5-stdlib amd64 3.5.3-1+deb9u4 [2166 kB]
Get:20 http://security.debian.org/debian-security stretch/updates/main amd64 python3.5 amd64 3.5.3-1+deb9u4 [231 kB]
debconf: delaying package configuration, since apt-utils is not installed
Fetched 7897 kB in 4s (1800 kB/s)
Selecting previously unselected package libssl1.1:amd64.
(Reading database ... 6320 files and directories currently installed.)
Preparing to unpack .../00-libssl1.1_1.1.0l-1~deb9u3_amd64.deb ...
Unpacking libssl1.1:amd64 (1.1.0l-1~deb9u3) ...
Selecting previously unselected package libpython3.5-minimal:amd64.
Preparing to unpack .../01-libpython3.5-minimal_3.5.3-1+deb9u4_amd64.deb ...
Unpacking libpython3.5-minimal:amd64 (3.5.3-1+deb9u4) ...
Selecting previously unselected package libexpat1:amd64.
Preparing to unpack .../02-libexpat1_2.2.0-2+deb9u3_amd64.deb ...
Unpacking libexpat1:amd64 (2.2.0-2+deb9u3) ...
Selecting previously unselected package python3.5-minimal.
Preparing to unpack .../03-python3.5-minimal_3.5.3-1+deb9u4_amd64.deb ...
Unpacking python3.5-minimal (3.5.3-1+deb9u4) ...
Selecting previously unselected package python3-minimal.
Preparing to unpack .../04-python3-minimal_3.5.3-1_amd64.deb ...
Unpacking python3-minimal (3.5.3-1) ...
Selecting previously unselected package mime-support.
Preparing to unpack .../05-mime-support_3.60_all.deb ...
Unpacking mime-support (3.60) ...
Selecting previously unselected package libmpdec2:amd64.
Preparing to unpack .../06-libmpdec2_2.4.2-1_amd64.deb ...
Unpacking libmpdec2:amd64 (2.4.2-1) ...
Selecting previously unselected package readline-common.
Preparing to unpack .../07-readline-common_7.0-3_all.deb ...
Unpacking readline-common (7.0-3) ...
Selecting previously unselected package libreadline7:amd64.
Preparing to unpack .../08-libreadline7_7.0-3_amd64.deb ...
Unpacking libreadline7:amd64 (7.0-3) ...
Selecting previously unselected package libsqlite3-0:amd64.
Preparing to unpack .../09-libsqlite3-0_3.16.2-5+deb9u3_amd64.deb ...
Unpacking libsqlite3-0:amd64 (3.16.2-5+deb9u3) ...
Selecting previously unselected package libpython3.5-stdlib:amd64.
Preparing to unpack .../10-libpython3.5-stdlib_3.5.3-1+deb9u4_amd64.deb ...
Unpacking libpython3.5-stdlib:amd64 (3.5.3-1+deb9u4) ...
Selecting previously unselected package python3.5.
Preparing to unpack .../11-python3.5_3.5.3-1+deb9u4_amd64.deb ...
Unpacking python3.5 (3.5.3-1+deb9u4) ...
Selecting previously unselected package libpython3-stdlib:amd64.
Preparing to unpack .../12-libpython3-stdlib_3.5.3-1_amd64.deb ...
Unpacking libpython3-stdlib:amd64 (3.5.3-1) ...
Selecting previously unselected package dh-python.
Preparing to unpack .../13-dh-python_2.20170125_all.deb ...
Unpacking dh-python (2.20170125) ...
Setting up libssl1.1:amd64 (1.1.0l-1~deb9u3) ...
debconf: unable to initialize frontend: Dialog
debconf: (TERM is not set, so the dialog frontend is not usable.)
debconf: falling back to frontend: Readline
debconf: unable to initialize frontend: Readline
debconf: (Can't locate Term/ReadLine.pm in @INC (you may need to install the Term::ReadLine module) (@INC contains: /etc/perl /usr/local/lib/x86_64-linux-gnu/perl/5.24.1 /usr/local/share/perl/5.24.1 /usr/lib/x86_64-linux-gnu/perl5/5.24 /usr/share/perl5 /usr/lib/x86_64-linux-gnu/perl/5.24 /usr/share/perl/5.24 /usr/local/lib/site_perl /usr/lib/x86_64-linux-gnu/perl-base .) at /usr/share/perl5/Debconf/FrontEnd/Readline.pm line 7.)
debconf: falling back to frontend: Teletype
Setting up libpython3.5-minimal:amd64 (3.5.3-1+deb9u4) ...
Setting up libexpat1:amd64 (2.2.0-2+deb9u3) ...
Setting up python3.5-minimal (3.5.3-1+deb9u4) ...
Setting up python3-minimal (3.5.3-1) ...
Selecting previously unselected package python3.
(Reading database ... 7313 files and directories currently installed.)
Preparing to unpack .../0-python3_3.5.3-1_amd64.deb ...
Unpacking python3 (3.5.3-1) ...
Selecting previously unselected package bzip2.
Preparing to unpack .../1-bzip2_1.0.6-8.1_amd64.deb ...
Unpacking bzip2 (1.0.6-8.1) ...
Selecting previously unselected package libmagic-mgc.
Preparing to unpack .../2-libmagic-mgc_1%3a5.30-1+deb9u3_amd64.deb ...
Unpacking libmagic-mgc (1:5.30-1+deb9u3) ...
Selecting previously unselected package libmagic1:amd64.
Preparing to unpack .../3-libmagic1_1%3a5.30-1+deb9u3_amd64.deb ...
Unpacking libmagic1:amd64 (1:5.30-1+deb9u3) ...
Selecting previously unselected package file.
Preparing to unpack .../4-file_1%3a5.30-1+deb9u3_amd64.deb ...
Unpacking file (1:5.30-1+deb9u3) ...
Selecting previously unselected package xz-utils.
Preparing to unpack .../5-xz-utils_5.2.2-1.2+b1_amd64.deb ...
Unpacking xz-utils (5.2.2-1.2+b1) ...
Setting up readline-common (7.0-3) ...
Setting up mime-support (3.60) ...
Setting up libreadline7:amd64 (7.0-3) ...
Setting up libmagic-mgc (1:5.30-1+deb9u3) ...
Setting up bzip2 (1.0.6-8.1) ...
Setting up libmagic1:amd64 (1:5.30-1+deb9u3) ...
Processing triggers for libc-bin (2.24-11+deb9u4) ...
Setting up xz-utils (5.2.2-1.2+b1) ...
update-alternatives: using /usr/bin/xz to provide /usr/bin/lzma (lzma) in auto mode
update-alternatives: warning: skip creation of /usr/share/man/man1/lzma.1.gz because associated file /usr/share/man/man1/xz.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/unlzma.1.gz because associated file /usr/share/man/man1/unxz.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzcat.1.gz because associated file /usr/share/man/man1/xzcat.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzmore.1.gz because associated file /usr/share/man/man1/xzmore.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzless.1.gz because associated file /usr/share/man/man1/xzless.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzdiff.1.gz because associated file /usr/share/man/man1/xzdiff.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzcmp.1.gz because associated file /usr/share/man/man1/xzcmp.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzgrep.1.gz because associated file /usr/share/man/man1/xzgrep.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzegrep.1.gz because associated file /usr/share/man/man1/xzegrep.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzfgrep.1.gz because associated file /usr/share/man/man1/xzfgrep.1.gz (of link group lzma) doesn't exist
Setting up libsqlite3-0:amd64 (3.16.2-5+deb9u3) ...
Setting up libmpdec2:amd64 (2.4.2-1) ...
Setting up libpython3.5-stdlib:amd64 (3.5.3-1+deb9u4) ...
Setting up file (1:5.30-1+deb9u3) ...
Setting up python3.5 (3.5.3-1+deb9u4) ...
Setting up libpython3-stdlib:amd64 (3.5.3-1) ...
Setting up dh-python (2.20170125) ...
Setting up python3 (3.5.3-1) ...
running python rtupdate hooks for python3.5...
running python post-rtupdate hooks for python3.5...
Processing triggers for libc-bin (2.24-11+deb9u4) ...
Removing intermediate container 319328f7ab75
 ---> d442b8f50956
Successfully built d442b8f50956
Successfully tagged python:3

[root@iZuf6jcqwirs6izt7krfcmZ home]# docker images											#查看镜像
REPOSITORY   TAG            IMAGE ID       CREATED          SIZE
python       3              d442b8f50956   54 seconds ago   95.2MB							#刚刚创建的镜像
ubuntu       16.04          3a07811e4fbb   26 minutes ago   505MB
test         0.1            43bd406c4cab   56 minutes ago   63.1MB
ubuntu       18.04          81bcf752ac3d   2 days ago       63.1MB
debian       stretch-slim   e59fcdf363c2   10 days ago      55.3MB
ubuntu       latest         7e0aa2d69a15   4 weeks ago      72.7MB
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

### 上传镜像

```shell
#需要登录
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

## 操作Docker容器

### 创建容器

#### 新建容器

```shell
#docker create -it 容器名称:标签   									#此时创建的容器时为启动的

[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker create -it ubuntu
e984eccd51b999b112cde9c75142a0720e4da301cbd9014816a0855df10f4c9a
[root@iZuf6jcqwirs6izt7krfcmZ ~]# docker ps -a
CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS                   PORTS     NAMES
e984eccd51b9   ubuntu         "/bin/bash"   16 seconds ago   Created                            lucid_ellis
e3606c5cb5c2   ubuntu:18.04   "/bin/bash"   4 hours ago      Exited (0) 4 hours ago             unruffled_austin
```

#### 启动容器

```shell
#ocker start 镜像名称或者镜像ID

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

# docker run 名称:标签     											#相当于pull create start 三个命令
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
#通过-d参数让容器后台以寿虎泰运行
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

## 访问Docker仓库

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

#方式二
#使用旧的-v标记可以再容器内创建一个数据卷
[root@iZuf6jcqwirs6izt7krfcmZ home]# docker run -d -p 8081:8081 --name web --v /home/webapp:/opt/webapp training/webapp python app.py

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

![image-20210523200528457](D:\javaSoft\typora\doc\data\images\image-20210523200528457.png)

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
#创建啊一个新的web容器并且连接数据库容器
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



# 实战案例







# 进阶技能