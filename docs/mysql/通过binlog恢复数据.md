1、首先bin log日志会在服务每次重启都会生成一个，默认大小是1G，当超过这个大小的时候会重新生成 `-bin.000001`这样的格式自增

2、查看二进制文件

![image-20220209161114137](https://gitee.com/laoyouji1018/images/raw/master/img/202202091611161.png)

3、为了防止后续的操作影响需要恢复的那个日志文件，此处恢复前，我们需要先`flush logs;`

![image-20220209160953534](https://gitee.com/laoyouji1018/images/raw/master/img/202202091609583.png)

刷新之后会多出一个文件

![image-20220209161144637](https://gitee.com/laoyouji1018/images/raw/master/img/202202091611676.png)

后续的其他恢复操作都是在`000012`里面（恢复时建议对外停止更新，即禁止更新数据库）

4、根据命令可以查看到相应的操作和pos

```shell
mysql> show binlog events in "mall-mysql-bin.000011";
```

![image-20220209161329516](https://gitee.com/laoyouji1018/images/raw/master/img/202202091613546.png)

5、通过一下命令恢复你要恢复的数据

```shell
mysqlbinlog --start-position=1525 --stop-position=1747 --database=atguigudb1  /var/lib/mysql/mall-mysql-bin.000011 | mysql -uroot -proot -v atguigudb1;
```

`--start-position`:数据恢复的开始位置

`--stop-positio`:数据恢复的结束为止

比如恢复如下三条语句，红框分别代表开始位置和结束为止

![image-20220209161918077](https://gitee.com/laoyouji1018/images/raw/master/img/202202091619110.png)

`database`:数据库

`/var/lib/mysql/mall-mysql-bin.000011`:二进制文件路径

`mysql -uroot -proot`:自己的账号密码