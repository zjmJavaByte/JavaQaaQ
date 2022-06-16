# linux搭建elk并与springboot整合

## linux安装docker-compose

> [参考文章](https://blog.csdn.net/AlexanderRon/article/details/123412922)
>
> 本docker-compose安装教程适用于`CentOS，Ubuntu`等`Unix`系统。
> 其他版本可以在docker官网：[docker/compose release](https://github.com/docker/compose/releases)自行下载。

### 步骤

- 从git上拉取docker-compose，注意：这里版本我安装的是`1.28.2`，如果需要安装对应某个版本请更改。

```sh
sudo curl -L "https://github.com/docker/compose/releases/download/1.28.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

> 如果下载只有9个字节说明没找到你系统对应的版本，可以到[docker/compose release](https://github.com/docker/compose/releases)自行下载对应系统版本的docker-compose。
> **额外Tips**：查看系统类型：`echo $(uname -s)`，查看系统架构`echo $(uname -m)`

- 授权docker-compose

```sh
sudo chmod +x /usr/local/bin/docker-compose
```

- 建立软连接

```sh
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

- 验证

```sh
[root@iZuf67z0nmo18acuap4yhxZ ~]# docker-compose --version
docker-compose version 1.29.2, build 5becea4c
```

##  使用Docker Compose 搭建ELK环境

> [参考文章](https://www.macrozheng.com/mall/reference/mall_tiny_elk.html#%E5%AD%A6%E4%B9%A0%E5%89%8D%E9%9C%80%E8%A6%81%E4%BA%86%E8%A7%A3%E7%9A%84%E5%86%85%E5%AE%B9)

###  搭建前准备

```sh
#需要设置系统内核参数，否则会因为内存不足无法启动。
sysctl -w vm.max_map_count=262144
# 使之立即生效
sysctl -p
# 创建目录
mkdir -p /mydata/elasticsearch/data/
# 创建并改变该目录权限
chmod 777 /mydata/elasticsearch/data
# 创建目录
mkdir -p /mydata/logstash
# 创建目录
mkdir -p /mydata/sh
```

### 开始搭建

- 创建一个存放`logstash`配置的目录并上传配置文件到`/mydata/logstash`下

```config
input {
  tcp {
    mode => "server"
    host => "0.0.0.0"
    port => 4560
    codec => json_lines
  }
}
output {
  elasticsearch {
    hosts => "es:9200"
    index => "springboot-logstash-%{+YYYY.MM.dd}"
  }
}
```

- 使用`docker-compose.yml`脚本启动`ELK`服务

```yml
version: '3'
services:
  elasticsearch:
    image: elasticsearch:6.4.0
    container_name: elasticsearch
    environment:
      - "cluster.name=elasticsearch" #设置集群名称为elasticsearch
      - "discovery.type=single-node" #以单一节点模式启动
      - "http.cors.enabled=true" #以单一节点模式启动
      - "http.cors.allow-origin=*" #以单一节点模式启动
      - "ES_JAVA_OPTS=-Xms128m -Xmx128m" #设置使用jvm内存大小
    volumes:
      - /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins #插件文件挂载
      - /mydata/elasticsearch/data:/usr/share/elasticsearch/data #数据文件挂载
    ports:
      - 9200:9200
      - 9300:9300
  kibana:
    image: kibana:6.4.0
    container_name: kibana
    links:
      - elasticsearch:es #可以用es这个域名访问elasticsearch服务
    depends_on:
      - elasticsearch #kibana在elasticsearch启动之后再启动
    environment:
      - "elasticsearch.hosts=http://es:9200" #设置访问elasticsearch的地址
    ports:
      - 5601:5601
  logstash:
    image: logstash:6.4.0
    container_name: logstash
    volumes:
      - /mydata/logstash/logstash-springboot.conf:/usr/share/logstash/pipeline/logstash.conf #挂载logstash的配置文件
    depends_on:
      - elasticsearch #kibana在elasticsearch启动之后再启动
    links:
      - elasticsearch:es #可以用es这个域名访问elasticsearch服务
    ports:
      - 4560:4560
```

- 将`docker-compose.yml`上传到`linux`服务器`/mydata/sh`目录下并使用`docker-compose`命令运行

```sh
#以下命令在/mydata/sh文件夹下执行
docker-compose up -d
```

- 在`logstash`中安装`json_lines`插件

```sh
# 进入logstash容器
docker exec -it logstash /bin/bash
# 进入bin目录
cd /bin/
# 安装插件
logstash-plugin install logstash-codec-json_lines
# 退出容器
exit
# 重启logstash服务
docker restart logstash
```

- 去阿里云控制台下的安全组里面开启5601端口访问

![image-20220615151557573](https://zjm-picgo.oss-cn-shanghai.aliyuncs.com/202206151516887.png)

- 访问`ip:5601`

![](https://zjm-picgo.oss-cn-shanghai.aliyuncs.com/202206151518325.png)

## SpringBoot应用集成Logstash

- 在pom.xml中添加logstash-logback-encoder依赖

```xml
<!--集成logstash-->
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>5.3</version>
</dependency>
```

-  添加配置文件logback-spring.xml让logback的日志输出到logstash

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration>
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
    <include resource="org/springframework/boot/logging/logback/console-appender.xml"/>
    <!--应用名称-->
    <property name="APP_NAME" value="mall-admin"/>
    <!--日志文件保存路径-->
    <property name="LOG_FILE_PATH" value="${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}}/logs}"/>
    <contextName>${APP_NAME}</contextName>
    <!--每天记录日志到文件appender-->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_FILE_PATH}/${APP_NAME}-%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${FILE_LOG_PATTERN}</pattern>
        </encoder>
    </appender>
    <!--输出到logstash的appender-->
    <appender name="LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <!--可以访问的logstash日志收集端口-->
        <destination>192.168.3.101:4560</destination>
        <encoder charset="UTF-8" class="net.logstash.logback.encoder.LogstashEncoder"/>
    </appender>
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
        <appender-ref ref="LOGSTASH"/>
    </root>
</configuration>
```

- 运行Springboot应用

![](https://zjm-picgo.oss-cn-shanghai.aliyuncs.com/202206151520128.png)

## 在kibana中查看日志信息

- 创建index pattern

![](https://zjm-picgo.oss-cn-shanghai.aliyuncs.com/202206151520523.png)

![](https://zjm-picgo.oss-cn-shanghai.aliyuncs.com/202206151521863.png)

- 查看收集的日志

![](https://zjm-picgo.oss-cn-shanghai.aliyuncs.com/202206151521364.png)