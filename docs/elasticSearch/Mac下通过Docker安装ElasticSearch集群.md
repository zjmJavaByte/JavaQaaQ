# Mac下通过Docker安装ElasticSearch-7.9.3集群

[参考文章](https://www.jianshu.com/p/e564acf12319)

## 创建集群网络

```sh
#通过help查看有哪些参数
zhujianmin@zhujianmindeMacBook-Pro ~ % docker network help
#查看 network
zhujianmin@zhujianmindeMacBook-Pro ~ % docker network ls
NETWORK ID     NAME             DRIVER    SCOPE
e343d8468b8b   bridge           bridge    local
7138c53e1c21   host             host      local
e6165defbadf   none             null      local
#创建自定义网络
zhujianmin@zhujianmindeMacBook-Pro ~ % docker network create --subnet=172.17.0.0/16 es-cluster-net
#查看创建的自定义网络(可以发现Containers字段中并没有容器连接进来)
zhujianmin@zhujianmindeMacBook-Pro ~ % docker inspect es-cluster-net
[
    {
        "Name": "es-cluster-net",
        "Id": "889e1ec6b184373e5abb560ae24d12c511036f0af98f3a452",
        "Created": "2022-06-13T07:43:18.93927655Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]

```

## 创建配置文件

在你自己的文件夹下分别创建4个文件夹如下：

![image-20220613161952563](http://106.14.69.81:9000/picgo/202206162033512_repeat_1655382822557__504816.png)

config里面新增三个配置文件：

- es1.yml

```yml
#一个集群名称，在该集群下的所有节点都会自动分配数据共享数据
cluster.name: elasticsearch-cluster
每一个节点都有自己的一个名称，仅此而已
node.name: es-node1
network.bind_host: 0.0.0.0
# network.publish_host: 127.0.0.1
##WebUI 默认9200
http.port: 9200
transport.host: 0.0.0.0
transport.tcp.port: 9300
# head 插件需要这打开这两个配置
#是否支持跨域
http.cors.enabled: true
#*表示支持所有域名#是否支持跨域
http.cors.allow-origin: "*"
#指定该节点是否有资格被选举成为node,默认是true，es是默认集群中的第一台机器为master，如果这台机挂了就会重新选举master。
node.master: true 
#指定该节点是否存储索引数据，默认为true。
node.data: true 
transport.tcp.compress: true
http.max_content_length: 100m
##es7.x 之后新增的配置，初始化一个新的集群时需要此配置来选举 master
cluster.initial_master_nodes: ["es-node1"] 
# ip:port = es-cluster-net 172.17.0.0/16 范围的IP:每个节点对应的 transport.tcp.port
discovery.seed_hosts: ["172.18.0.3:9300","172.18.0.4:9301","172.18.0.5:9302"]
gateway.recover_after_nodes: 2
```

- es2.yml

```yml
cluster.name: elasticsearch-cluster
node.name: es-node2
network.bind_host: 0.0.0.0
# network.publish_host: 127.0.0.1
http.port: 9201
transport.host: 0.0.0.0
transport.tcp.port: 9301
http.cors.enabled: true
http.cors.allow-origin: "*"
node.master: false 
node.data: true 
transport.tcp.compress: true
http.max_content_length: 100m
#此处为master节点的名称
cluster.initial_master_nodes: ["es-node1"] 
# ip:port = es-cluster-net 172.17.0.0/16 范围的IP:每个节点对应的 transport.tcp.port
discovery.seed_hosts: ["172.18.0.3:9300","172.18.0.4:9301","172.18.0.5:9302"]
gateway.recover_after_nodes: 2
```

es3.yml

```yml
cluster.name: elasticsearch-cluster
node.name: es-node3
network.bind_host: 0.0.0.0
# network.publish_host: 127.0.0.1
http.port: 9202
transport.host: 0.0.0.0
transport.tcp.port: 9302
http.cors.enabled: true
http.cors.allow-origin: "*"
node.master: false 
node.data: true 
transport.tcp.compress: true
http.max_content_length: 100m
#此处为master节点的名称
cluster.initial_master_nodes: ["es-node1"] 
# ip:port = es-cluster-net 172.17.0.0/16 范围的IP:每个节点对应的 transport.tcp.port
discovery.seed_hosts: ["172.18.0.3:9300","172.18.0.4:9301","172.18.0.5:9302"]
gateway.recover_after_nodes: 2
```

## docker启动三个节点

> 建议先启动完es01，再去启动剩下的两个

```sh
#节点es-node1
docker run --name es01 --network es-cluster-net --ip 172.18.0.3 \
-p 9200:9200 -p 9300:9300 \
-e ES_JAVA_OPTS="-Xms256m -Xmx256m" \
-v /Users/zhujianmin/Documents/NoiICloud.nosync/doc/docker/elasticsearch-7.9.3/config/es1.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /Users/zhujianmin/Documents/NoiICloud.nosync/doc/docker/elasticsearch-7.9.3/data1:/usr/share/elasticsearch/data \
-d elasticsearch:7.9.3
#节点es-node2
docker run --name es02 --network es-cluster-net --ip 172.18.0.4 \
-p 9201:9201 -p 9301:9301 \
-e ES_JAVA_OPTS="-Xms256m -Xmx256m" \
-v /Users/zhujianmin/Documents/NoiICloud.nosync/doc/docker/elasticsearch-7.9.3/config/es2.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /Users/zhujianmin/Documents/NoiICloud.nosync/doc/docker/elasticsearch-7.9.3/data2:/usr/share/elasticsearch/data \
-d elasticsearch:7.9.3

#节点es-node3
docker run --name es03 --network es-cluster-net --ip 172.18.0.5 \
-p 9202:9202 -p 9302:9302 \
-e ES_JAVA_OPTS="-Xms256m -Xmx256m" \
-v /Users/zhujianmin/Documents/NoiICloud.nosync/doc/docker/elasticsearch-7.9.3/config/es3.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /Users/zhujianmin/Documents/NoiICloud.nosync/doc/docker/elasticsearch-7.9.3/data3:/usr/share/elasticsearch/data \
-d elasticsearch:7.9.3
```

## 安装ik分词插件

```sh
#节点1
docker exec -it es01 /bin/bash ./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.9.3/elasticsearch-analysis-ik-7.9.3.zip
#节点2
docker exec -it es02 /bin/bash ./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.9.3/elasticsearch-analysis-ik-7.9.3.zip
#节点3
docker exec -it es03 /bin/bash ./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.9.3/elasticsearch-analysis-ik-7.9.3.zip
```

