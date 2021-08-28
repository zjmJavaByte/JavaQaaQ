**git Invalid username or password**

### 问题

> 14:03	Push failed
> remote: Invalid username or password.
> Authentication failed for 'https://github.com.*************/JavaQaaQ.git/'
> Show details in console

### 原因

> 主要是因为`git`密码的修改，或者是因为`personal access tokens` 过期导致的

### 解决办法

#### 生成`personal  access tokens`

- 找到设置

![image-20210828141149277](https://gitee.com/laoyouji1018/images/raw/master/img/20210828141201.png)

- developSetting

![image-20210828141226498](https://gitee.com/laoyouji1018/images/raw/master/img/20210828141241.png)

- Generate new tokens

![image-20210828141438448](https://gitee.com/laoyouji1018/images/raw/master/img/20210828141439.png)

- 点击生成Generate new tokens，并且按照下面所示设置

![image-20210828141724853](https://gitee.com/laoyouji1018/images/raw/master/img/20210828141823.png)

- 将生成的tokens保存下来

![image-20210828141819056](https://gitee.com/laoyouji1018/images/raw/master/img/20210828141820.png)

#### 修改Windows管理凭据

- 打开凭据管理

![image-20210828142041435](https://gitee.com/laoyouji1018/images/raw/master/img/20210828142042.png)

- 找到项目的git

![image-20210828142143201](https://gitee.com/laoyouji1018/images/raw/master/img/20210828142144.png)

- 编辑

![image-20210828142219440](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210828142219440.png)

- 将刚刚生成的凭据用于密码设置

![image-20210828142316634](https://gitee.com/laoyouji1018/images/raw/master/img/20210828142318.png)

#### 验证

