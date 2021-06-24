**使用crontab定时删除文件**

- 定义可执行文件

```shell
vim deletepdftemp.sh
```

```shell
#!/bin/bash

#find：linux的查找命令，用户查找指定条件的文件；
#/home/pdf/：想要进行清理的任意目录；
#-mtime：标准语句写法；
#+3：查找3天前的文件，这里用数字代表天数；
#"*.pdf"：希望查找的数据类型，"*.jpg"表示查找扩展名为jpg的所有文件，"*"表示查找所有文件，这个可以灵活运用，举一反三；
#-exec：固定写法；
#rm -rf：强制删除文件，包括目录；
# {} \; ：固定写法，一对大括号+空格+\+;

#添加删除日志到 /home/log/delete.log 文件夹下
echo "删除开始====`date "+%Y-%m-%d %H:%M:%S"`====删除开始" >> /home/log/delete.log
find /home/pdf/* -mtime +3 -name "*.pdf"  -exec rm -rf {} \;
echo "删除结束====`date "+%Y-%m-%d %H:%M:%S"`====删除结束" >> /home/log/delete.log
```

- 赋予文件权限

```shell
chmod 755 deletepdftemp.sh
```

- 编写crontab表达式

```shell
#命令说明
crontab -e：编辑当前用户的定时任务

crontab -l：查看当前用户的定时任务

crontab -r：删除当前用户的定时任务
```

> 编辑

```shell
crontab -e
```

> 每1分钟执行一次cmd；此处要写 deletepdftemp.sh 的全路径

```shell
* * * * * /home/deletepdftemp.sh
```

> 退出保存后命令行显示：install new crontab 即表示成功

- crontab表达式说明

```shell
#实例1：每1分钟执行一次cmd
* * * * * cmd

#实例2：每小时的第3和第15分钟执行
3,15 * * * * cmd
　　
#实例3：在上午8点到11点的第3和第15分钟执行
3,15 8-11 * * * cmd

#实例4：每隔两天的上午8点到11点的第3和第15分钟执行
3,15 8-11 */2  *  * cmd
　　
#实例5：每周一上午8点到11点的第3和第15分钟执行
3,15 8-11 * * 1 cmd
　　
#实例6：每晚的21:30执行
30 21 * * * cmd

#实例7：每月1、10、22日的4 : 45执行
45 4 1,10,22 * * cmd

#实例8：每周六、周日的1 : 10执行
10 1 * * 6,0 cmd

#实例9：每天18 : 00至23 : 00之间每隔30分钟执行
*/30 18-23 * * * cmd

#实例10：每星期六的晚上11 : 00 pm执行
0 23 * * 6 cmd

#实例11：每一小时执行
0 */1 * * * cmd

#实例12：晚上11点到早上7点之间，每隔一小时执行
0 23-7 * * * cmd
```

