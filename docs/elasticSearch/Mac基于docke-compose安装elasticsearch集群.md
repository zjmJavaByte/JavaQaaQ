# Mac基于docke-compose安装elasticsearch集群

[参考文章](https://cloud.tencent.com/developer/article/1906835)

## 创建文件夹

- 在任意文件夹下创建如下结构的文件夹

![image-20220613173116875](http://106.14.69.81:9000/picgo/202206162024998_repeat_1655382274045__359504.png)

- 

## 构建镜像

编写Dockerfile文件（在dockerFile文件夹下创建）

```sh
#用于创建elasticsearch容器的同时安装ik分词器
FROM docker.elastic.co/elasticsearch/elasticsearch:7.9.3
#作者
MAINTAINER zjm 
RUN sh -c '/bin/echo -e "y" | /usr/share/elasticsearch/bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.9.3/elasticsearch-analysis-ik-7.9.3.zip'
```

构建image

```sh
#在dockerFile文件夹下执行
docker build --tag=elasticsearch-ik-custom:7.9.3 .
```

成功的结果：

![image-20220613173252676](http://106.14.69.81:9000/picgo/202206162025979_repeat_1655382308024__951851.png)

## 编写docker-compose-custom.yml文件

```yml
version: '2.2'
services:
  es01:
    image: elasticsearch-ik-custom:7.9.3
    container_name: es01
    environment:
      - node.name=es01
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es02,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - http.cors.enabled=true
      - http.cors.allow-origin=*
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      #更换成你自己的文件路径
      - /Users/zhujianmin/Documents/NoiICloud.nosync/doc/docker/elasticsearch-7.9.3/data/node0:/usr/share/elasticsearch/data
      #更换成你自己的文件路径
      - /Users/zhujianmin/Documents/NoiICloud.nosync/doc/docker/elasticsearch-7.9.3/logs/node0:/usr/share/elasticsearch/logs
    ports:
      - 9200:9200
    networks:
      - elastic
  es02:
    image: elasticsearch-ik-custom:7.9.3
    container_name: es02
    environment:
      - node.name=es02
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - http.cors.enabled=true
      - http.cors.allow-origin=*
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      #更换成你自己的文件路径
      - /Users/zhujianmin/Documents/NoiICloud.nosync/doc/docker/elasticsearch-7.9.3/data/node1:/usr/share/elasticsearch/data
      #更换成你自己的文件路径
      - /Users/zhujianmin/Documents/NoiICloud.nosync/doc/docker/elasticsearch-7.9.3/logs/node1:/usr/share/elasticsearch/logs
    networks:
      - elastic
  es03:
    image: elasticsearch-ik-custom:7.9.3
    container_name: es03
    environment:
      - node.name=es03
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es02
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - http.cors.enabled=true
      - http.cors.allow-origin=*
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      #更换成你自己的文件路径
      - /Users/zhujianmin/Documents/NoiICloud.nosync/doc/docker/elasticsearch-7.9.3/data/node2:/usr/share/elasticsearch/data
      #更换成你自己的文件路径
      - /Users/zhujianmin/Documents/NoiICloud.nosync/doc/docker/elasticsearch-7.9.3/logs/node3:/usr/share/elasticsearch/logs
    networks:
      - elastic

networks:
  elastic:
    driver: bridge
```

启动服务

```sh
#在dockerFile文件夹下执行
docker-compose -f docker-compose-custom.yml up -d
```

测试：访问`http://127.0.0.1:9200/_cat/nodes?pretty`

![image-20220613173704415](http://106.14.69.81:9000/picgo/202206162025748_repeat_1655382320788__963586.png)

