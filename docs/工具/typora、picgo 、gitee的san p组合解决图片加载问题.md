typora、picgo 、gitee的san p组合解决图片加载问题

[toc]

> 备注说明：也可以使用GitHub代替gitee，但是鉴于GitHub服务器在国外，并且网络时而好时而坏的原因，此处选择gitee

#### 事出有因

- 问题1：你是否在使用typora编写文章时，在其中插入图片后，时间久了，文章中的图片不见了
- 问题2：当你更换电脑，或者分享md文件时，发现不仅要分享文章内容，还要连带图片一起分享
- 问题3：文章越来越多，图片越来越多，磁盘容量越来越小，Windows能忍，Mac不能忍

#### 解决方案

​		使用picgo 上传图片到gitee上，随时随地可以查看

​		免费！免费！免费！白嫖党！

##### 软件下载

- [typora下载链接](https://typora.io/)

- [typora的使用技巧](https://www.cnblogs.com/hider/p/11614688.html)

  ![image-20210618123907466](https://gitee.com/laoyouji1018/images/raw/master/img/20210618123907.png)

- [picgo下载链接](https://github.com/Molunerfinn/PicGo/releases)
- [picgo使用技巧](https://picgo.github.io/PicGo-Doc/zh/guide/config.html)

![image-20210618123742388](https://gitee.com/laoyouji1018/images/raw/master/img/20210618123742.png)

##### 具体操作

> 首先安装gitee插件，具体的步骤请看图片

![image-20210618125100334](https://gitee.com/laoyouji1018/images/raw/master/img/20210618125100.png)

> 安装成功之后，如图所示

![image-20210618125303646](https://gitee.com/laoyouji1018/images/raw/master/img/20210618125303.png)

> 其次在gitee上创建上传图片的仓库和用于身份验证的token

**创建仓库**

- 仓库名称`image`
- 选择`开源`
- 设置模板添加`Readme文件`

![image-20210618130019503](https://gitee.com/laoyouji1018/images/raw/master/img/20210618130019.png)

**获取token**

- 进入设置

![image-20210618130247825](https://gitee.com/laoyouji1018/images/raw/master/img/20210618130247.png)

- 选择私人令牌

![image-20210618130318282](https://gitee.com/laoyouji1018/images/raw/master/img/20210618130318.png)

- 选择生成新令牌

![image-20210618130409652](https://gitee.com/laoyouji1018/images/raw/master/img/20210618130409.png)

- 选择所有，然后提交

![image-20210618130506739](https://gitee.com/laoyouji1018/images/raw/master/img/20210618130506.png)

- 输入密码

![image-20210618130646389](https://gitee.com/laoyouji1018/images/raw/master/img/20210618130646.png)

- 复制token

![image-20210618130923976](https://gitee.com/laoyouji1018/images/raw/master/img/20210618130924.png)

> picgo设置gitee图床

![image-20210618125606287](https://gitee.com/laoyouji1018/images/raw/master/img/20210618125606.png)

- 修改picgo图片上传时间戳重命名

![image-20210618131113754](https://gitee.com/laoyouji1018/images/raw/master/img/20210618131113.png)

- 测试上传图片

![image-20210618131247195](https://gitee.com/laoyouji1018/images/raw/master/img/20210618131247.png)

- 查看上传的图片

![image-20210618131322429](https://gitee.com/laoyouji1018/images/raw/master/img/20210618131322.png)

> typora中配置picgo图片上传

- 在文件菜单栏目中找到偏好设置，配置如下三项

![image-20210618131518619](https://gitee.com/laoyouji1018/images/raw/master/img/20210618131518.png)

- 测试上传

将图片拉入typora中，会自动上传到gitee上，如果没有自动，可以右击手动上传

![image-20210618131707984](https://gitee.com/laoyouji1018/images/raw/master/img/20210618131708.png)

- 查看gitee上image仓库下img文件夹下的图片

![image-20210618132010003](https://gitee.com/laoyouji1018/images/raw/master/img/20210618132010.png)

备注：Mac上也适用以上同样的方式