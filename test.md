# 目录扫描dirb和dirsearch（二选一）安装和使用

下载链接：[https://github.com/lemonlove7/dirsearch_bypass403/releases/tag/v0.2](https://github.com/lemonlove7/dirsearch_bypass403/releases/tag/v0.2)

在本地或者虚拟机或者服务器上进行下载（两种方法）

  git clone https://xxxx.xxx/xxx

  直接下载 zip 压缩包进行解压

下载完成

```Python
dwda
```


![image.png](https://tc-cdn.flowus.cn/oss/d5099caf-9fda-4525-8b92-99b6620a2514/image.png?time=1744386300&token=b88e76522018679532a9dc3c11dbf1385944bed7e1a31ea8183178bf10520d8d&role=free)

执行命令

![image.png](https://tc-cdn.flowus.cn/oss/90012749-2482-4a20-b81e-0f551aa4de52/image.png?time=1744386300&token=52b0c80a7529e156175723ff2a8c1b6c55bd5d1dbaea6c937ed3e9fcf9741b92&role=free)

# 钓鱼网站制作

注：我使用的是在本地搭建的 gophish

下载链接：[https://github.com/gophish/gophish](https://github.com/gophish/gophish)

启动gophish

![image.png](https://tc-cdn.flowus.cn/oss/d2d3820b-1bf9-4503-8238-e90b631d0aed/image.png?time=1744386300&token=b92db692fbede0c8b39abf141da7251fe876f39911ba8221c3015e398b8bb7a3&role=free)

输入 http://127.0.0.1:3333

成功进入后台管理系统

![image.png](https://tc-cdn.flowus.cn/oss/97ba7444-0862-4a04-b5cd-edacd5eec7d1/image.png?time=1744386300&token=7cae06572c325880c6e8ff3235af761ff2121ff7990e2c518858fbd452e34dab&role=free)

进行一系列的配置

  1. Sending Profiles 发件策略

  ![image.png](https://tc-cdn.flowus.cn/oss/5287e45f-3aee-488f-b889-c9d3f626f3f2/image.png?time=1744386300&token=6c165ca8e88b079c9c1ffa5220d62065e81d0a649b1ff671eb91c60574584dbe&role=free)

  2. Landing Pages 钓鱼页面

  ![image.png](https://tc-cdn.flowus.cn/oss/3e7f0ed6-82b4-4191-a7b4-05847d7f7195/image.png?time=1744386300&token=d355f98df2a0174fd72900ba40c3364e592c9a02117627f371a4a60f571c11cd&role=free)

  3. Email Templates 钓鱼邮件模板

  ![image.png](https://tc-cdn.flowus.cn/oss/83f765c8-512e-46f8-a814-f59361267730/image.png?time=1744386300&token=77146e2ba2c436e0079307ce4c1cb96156f8a768c6a1a02907a8bcf823175c50&role=free)

  4. Users & Groups 用户和组

  ![image.png](https://tc-cdn.flowus.cn/oss/b12d3d06-4159-4f9e-a5fd-d60609875997/image.png?time=1744386300&token=734df1eebcdd1d4d1e19cc65ae4da13be2338975d878dab9f52b28411346bd60&role=free)

  5. Campaigns 钓鱼事件

  ![image.png](https://tc-cdn.flowus.cn/oss/04cc17e1-6c5c-4a19-9de2-ccf5ef9a30bc/image.png?time=1744386300&token=288248cafdc0d70a43916cd27e1f1ca390ef81daa2c2dfa08c40f63bba7f224d&role=free)

  开始自己给自己发送钓鱼邮件

  我搞的很粗糙，用的gitee的异常登录警告，但是跳转的是学校登录界面，不过效果倒是达到了，成功获取被调用户在登录框所输入的密码

  ![image.png](https://tc-cdn.flowus.cn/oss/c97b9a9d-bab8-4d47-94c0-e9ec91cd4204/image.png?time=1744386300&token=59aa1b7aa2207cf1bb44dc5d5d7e1635c1518ce453c1442867643b4f6ab44d39&role=free)



# hydra爆破

使用hydra爆破 ftp 服务

![image.png](https://tc-cdn.flowus.cn/oss/2464d82e-a78f-46dc-ba9a-17bdc7130044/image.png?time=1744386300&token=3766e817694fc5959fd2e76410d58850ec9af3b04be73cf145631267094cd143&role=free)

# BP爆破后台登陆用户名密码

可以看到测试网站为明文传输

![image.png](https://tc-cdn.flowus.cn/oss/b5595a9a-7e56-4175-95e8-09c7394781d0/image.png?time=1744386300&token=f0f34b2d5572d17899828430a04a5f48b500e6aea6318a2a856c699c014b5db5&role=free)



直接爆破

![image.png](https://tc-cdn.flowus.cn/oss/a9e3516e-88fe-4a42-b922-6f9a0cad98a9/image.png?time=1744386300&token=e6a1b76a6b10786fcb8a03066a7bc45ed13143350848f7623847bb3899e896e2&role=free)

看见这几个响应包不太一样，直接手动测试

经过测试，发现有这三个都成功登入后台

admin/123456

Admin/123456

/123456（前端校验账号密码是否输入，抓包删除用户名，进行登录，一样可以绕过前端校验进行登录）

![image.png](https://tc-cdn.flowus.cn/oss/f7806df8-7528-4630-a3b1-ae29db3e526f/image.png?time=1744386300&token=7953453e21104062cb29f1efb7f2d99ff1c527cd8196eaad17b0d2ca09b63b93&role=free)



