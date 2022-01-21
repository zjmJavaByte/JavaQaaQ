在`main.go`目录下面执行如下语句

```shell
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build main.go
```

然后会在同级目录下的到一个可执行文件

![image-20211231165040532](https://gitee.com/laoyouji1018/images/raw/master/img/202112311650629.png)

上传到服务器的某个目录，我这里是上传到`/home/go/`目录下面

![image-20211231165943877](https://gitee.com/laoyouji1018/images/raw/master/img/202112311659916.png)

后台运行main

```shell
[root@iZuf67z0nmo18acuap4yhxZ go]# nohup ./main &
```

查看日志

```shell
[root@iZuf67z0nmo18acuap4yhxZ go]# tail -f nohup.out
```

