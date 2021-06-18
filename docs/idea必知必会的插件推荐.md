[toc]

###  Alibaba Java Coding Guidelines

​		安装该插件后，代码超过 80 行、手动创建线程池等，这些和《手册》中的规约不符时，IDEA中会给出警告提示。

建议大家一定一定一定要安装该插件，它会帮助你检查出很多隐患，督促你写更规范的代码。

**效果演示**

自动提醒

![image-20210531181735576](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/image-20210531181735576.png)

全局检测

![image-20210531181841266](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/image-20210531181841266.png)

选择编码规约扫描

![image-20210531181947984](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/image-20210531181947984.png)

### Rainbow Brackets

效果如图所示，不同颜色的括号

![image-20210531182321854](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/image-20210531182321854.png)

### Maven Helper

效果图

![image-20210531182644747](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/image-20210531182644747.png)

如果有冲突可进行exclude排除

![image-20210531182759204](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210531182759204.png)

### QAPlug-FindBugs与QAPlug

![image-20210531182919414](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/image-20210531182919414.png)

使用

![image-20210531183104435](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/image-20210531183104435.png)



![image-20210531183156446](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/image-20210531183156446.png)

分析完之后给出优化建议

![image-20210531183242687](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/image-20210531183242687.png)



### Maven Helper

**功能一 自动生成SQL文件**

鼠标放在save上面，按住`Alt+enter`自动在mapper.xml里面生成相应的方法

![image-20210531183501977](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/1image-2021053118350197337.png)

![image-20210531183633957](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/22image-20210531183633957.png)

**功能二 跳转到相应的mapper.xml**

![image-20210531184202722](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/image-20210531184202722.png)



### SequenceDiagram

[SequenceDiagram ](https://link.zhihu.com/?target=https%3A//plugins.jetbrains.com/plugin/8286-sequencediagram/)可以根据代码调用链路自动生成时序图，超级赞，超级推荐！

这对研究源码，梳理工作中的业务代码有极大的帮助，堪称神器。

安装完成后，在某个类的某个函数中，右键 --> Sequence Diagaram 即可调出。

如下图是 Netty 的源码，可以通过该插件绘制出当前函数的调用链路。

**使用**

![image-20210531184422910](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/image-20210531184422910.png)



![image-20210531184608644](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/image-20210531184608644.png)

效果图

![image-20210531184546139](https://raw.githubusercontent.com/zjmJavaByte/images/master/images/image-20210531184546139.png)