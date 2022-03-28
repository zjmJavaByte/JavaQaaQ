## idea使用maven插件构建springboot项目镜像（基于阿里云服务器）

### 搭建私有仓库Docker Registry

```sh
docker run -d -p 5000:5000 --restart=always --name registry2 registry:2
```

如果遇到镜像下载不下来的情况，需要修改 /etc/docker/daemon.json 文件并添加上 registry-mirrors 键值，然后重启docker服务：

```json
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}

```

### Docker开启远程API

#### 用vim编辑器修改docker.service文件

```sh
vi /usr/lib/systemd/system/docker.service
```

找到`ExecStart`这一行

1、如果你的内容如下

```json
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

需要修改成如下：

```sh
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2275 -H unix://var/run/docker.sock
```

2、如果你的内容如下

```json
ExecStart=/usr/bin/dockerd-current  \
          --add-runtime docker-runc=/usr/libexec/docker/docker-runc-current \
          --default-runtime=docker-runc \
          --exec-opt native.cgroupdriver=systemd \
          --userland-proxy-path=/usr/libexec/docker/docker-proxy-current \
          --init-path=/usr/libexec/docker/docker-init-current \
          --seccomp-profile=/etc/docker/seccomp.json \
          $OPTIONS \
          $DOCKER_STORAGE_OPTIONS \
          $DOCKER_NETWORK_OPTIONS \
          $ADD_REGISTRY \
          $BLOCK_REGISTRY \
          $INSECURE_REGISTRY \
          $REGISTRIES
```

需要修改成如下：在`/usr/bin/dockerd-current`后面添加`-H tcp://0.0.0.0:2275 -H unix://var/run/docker.sock`

```sh
ExecStart=/usr/bin/dockerd-current -H tcp://0.0.0.0:2275 -H unix://var/run/docker.sock \
          --add-runtime docker-runc=/usr/libexec/docker/docker-runc-current \
          --default-runtime=docker-runc \
          --exec-opt native.cgroupdriver=systemd \
          --userland-proxy-path=/usr/libexec/docker/docker-proxy-current \
          --init-path=/usr/libexec/docker/docker-init-current \
          --seccomp-profile=/etc/docker/seccomp.json \
          $OPTIONS \
          $DOCKER_STORAGE_OPTIONS \
          $DOCKER_NETWORK_OPTIONS \
          $ADD_REGISTRY \
          $BLOCK_REGISTRY \
          $INSECURE_REGISTRY \
          $REGISTRIES
```

开启阿里云安全规则组 `2275 `端口

![image-20220325152639296](https://s2.loli.net/2022/03/25/JuWCIodjFyGSimV.png)

#### 让Docker支持http上传镜像

```sh
echo '{ "insecure-registries":["ip:5000"] }' > /etc/docker/daemon.json
```

开启阿里云安全规则组 `5000 `端口

![image-20220325104827761](https://s2.loli.net/2022/03/25/qnSfWMNmYl7415u.png)

#### 修改配置后需要使用如下命令使配置生效

```sh
systemctl daemon-reload
```

#### 重新启动Docker服务

```sh
systemctl stop docker
systemctl start docker
```

### 使用docker插件

安装docker插件

![image-20220325151924693](https://s2.loli.net/2022/03/25/GYoXWlfZai2y9e1.png)

配置docker连接

![image-20220325112505304](https://s2.loli.net/2022/03/25/DwQlE2XBF3ob1Z8.png)

连接docker服务

![image-20220325110902593](https://s2.loli.net/2022/03/25/je8FrCRUtEd5Abf.png)

### 使用Maven构建Docker镜像

#### 在应用的pom.xml文件中添加docker-maven-plugin的依赖

```xml
<plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <!--这里的版本不能太高,比如1.0.0版本，会出现如下的错误
                Failed to execute goal com.spotify:docker-maven-plugin:1.0.0:build (build-image) on project photo: Exception caught
具体的原因未知，有知道的麻烦告知下
                -->
                <version>0.4.13</version>
                <executions>
                    <execution>
                        <id>build-image</id>
                        <phase>package</phase>
                        <goals>
                            <goal>build</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <dockerHost>http://ip:2275</dockerHost>
                    <imageName>${project.build.finalName}:${project.version}</imageName>
                    <baseImage>java</baseImage>
                    <entryPoint>["java","-jar","/${project.build.finalName}.jar"]</entryPoint>
                    <exposes>1111</exposes>
                    <resources>
                        <resource>
                            <targetPath>/</targetPath>
                            <directory>${project.build.directory}</directory>
                            <include>${project.build.finalName}.jar</include>
                        </resource>
                    </resources>
                    <forceTags> true</forceTags>
                </configuration>
            </plugin>
```

相关配置说明：

- executions.execution.phase:此处配置了在maven打包应用时构建docker镜像；
- imageName：用于指定镜像名称dockerHost：打包后上传到的docker服务器地址；
- baseImage：该应用所依赖的基础镜像，此处为java；
- entryPoint：docker容器启动时执行的命令；
- resources.resource.targetPath：将打包后的资源文件复制到该目录；
- resources.resource.directory：需要复制的文件所在目录，maven打包的应用jar包保存在target目录下面；
- resources.resource.include：需要复制的文件，打包好的应用jar包。

#### 使用IDEA打包项目并构建镜像

执行maven的package命令:

![image-20220325105615108](https://s2.loli.net/2022/03/25/Wecf4aBJL2nC1pZ.png)

构建成功:

![image-20220325105713044](https://s2.loli.net/2022/03/25/MNskKmzZoelCrpU.png)

构建成功后可以在docker插件看到相应的镜像文件：

![image-20220325111050408](https://s2.loli.net/2022/03/25/TWmpcB4IE1Z6RK8.png)

或者去阿里云服务器上查看：

![image-20220325111202667](https://s2.loli.net/2022/03/25/SGY5cAWZ2vl9gnj.png)