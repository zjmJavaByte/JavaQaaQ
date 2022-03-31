## picgo使用gitee做图床 出现 stateCode 403 Forbidden

报错信息如下：

```java
2022-03-24 17:49:25 [PicGo WARN] [PicGo Server] upload failed, see picgo.log for more detail ↑ 
2022-03-24 17:52:19 [PicGo INFO] [PicGo Server] get the request {"list":["\/Applications\/Typora.app\/Contents\/Resources\/TypeMark\/assets\/icon\/icon_512x512.png","\/Applications\/Typora.app\/Contents\/Resources\/TypeMark\/assets\/icon\/icon_256x256.png"]} 
2022-03-24 17:52:19 [PicGo INFO] [PicGo Server] upload files in list 
2022-03-24 17:52:19 [PicGo INFO] Before transform 
2022-03-24 17:52:19 [PicGo INFO] Transforming... Current transformer is [path] 
2022-03-24 17:52:19 [PicGo INFO] Before upload 
2022-03-24 17:52:19 [PicGo INFO] beforeUploadPlugins: renameFn running 
2022-03-24 17:52:19 [PicGo INFO] Uploading... Current uploader is [gitee] 
2022-03-24 17:52:19 [PicGo INFO] [上传操作]异常：403 - "<html>\r\n<head><title>403 Forbidden</title></head>\r\n<body>\r\n<center><h1>403 Forbidden</h1></center>\r\n<hr><center>nginx</center>\r\n</body>\r\n</html>\r\n" 
2022-03-24 17:52:19 [PicGo INFO] [上传操作]异常：403 - "<html>\r\n<head><title>403 Forbidden</title></head>\r\n<body>\r\n<center><h1>403 Forbidden</h1></center>\r\n<hr><center>nginx</center>\r\n</body>\r\n</html>\r\n" 
2022-03-24 17:52:19 [PicGo SUCCESS] 
```

主要原因是因为gitee官方发现太多的外部链接，所以禁止了你的账号上传图片。。。

[解决方案](https://github.com/zhanghuid/picgo-plugin-gitee/issues/11)