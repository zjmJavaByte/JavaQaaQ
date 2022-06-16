- 第一步：打开[ipaddress.com](https://www.ipaddress.com),查询如下两个域名，并分别记录下其对应的ip：

> - github.com
> - github.global.ssl.fastly.net

- 第二步：更新`host`文件，[参考地址](https://blog.csdn.net/weixin_38629529/article/details/120788902)

> - 192.30.253.112 github.com
> - 151.101.185.194 github.global.ssl.fastly.net

- 第三步：清理下`DNS`，再试一下。

> ipconfig /flushdns

