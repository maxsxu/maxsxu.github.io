+++ 
draft = false
date = 2020-11-06T21:02:26+08:00
title = "基于 Serverless 的车牌识别云服务"
description = ""
slug = "" 
tags = ["Serverless"]
categories = ["CloudNative"]
externalLink = ""
series = []
+++

Serverless 被誉为“云计算的下个十年”。本文从实际开发一个车牌识别云服务出发，让开发者直观感受到 Serverless 带来的变革。

# 需求
产品经理提了个新需求，开发一个车牌识别云服务。功能如下:

- 服务输入为车牌图片 base64, 输出为车牌号码
- 服务形式为 RESTful API

而且开发速度要快，运行要帅！下面让我们采用[阿里云提供的 Serverless 函数计算](https://www.aliyun.com/product/fc)来满足产品经理的合理要求！

# 基于阿里云 Serverless 函数计算
使用阿里云的 `Funcraft` 工具来进行开发、构建和部署操作.

执行 `npm install @alicloud/fun -g` 安装 Funcraft，然后执行 `fun init http-trigger-python3` 初始化项目. 最终的项目结构如下:

```shell
serverless/
├── Funfile
├── index.py
├── template.yml
└── test.jpg
```

其中 `Funfile` 用于管理依赖, `template.yml` 用于配置服务和函数, `index.py` 为核心代码，`test.jpg` 为测试用车牌照片.

## 1. 编写核心业务逻辑
首先使用 `Funcraft` 安装 `hyperlpr` 和 `opencv-python` 依赖，`hyperlpr` 是一个高性能的车牌识别 Python 库.

```shell
fun install --runtime python3 --package-type pip --save --index-url https://mirrors.aliyun.com/pypi/simple/ hyperlpr opencv-python==3.4.8.29
```

其中 `--save` 参数用于写入依赖到 `Funfile` 文件。

车牌识别核心函数如下，输入为车牌图片的 base64 字符串，输出识别结果:

```python
def licence_plate_parser(img_base64: str):
    base64data = base64.b64decode(img_base64.encode("utf8"))
    nparr = np.fromstring(base64data, np.uint8)
    img = cv2.imdecode(nparr, cv2.IMREAD_COLOR)
    return {
        "result": hyperlpr.HyperLPR_plate_recognition(img)
    }
```

核心业务逻辑依托强大的 Python 库，代码十分简洁，仅需四行即可实现车牌识别的功能。简单解释下逻辑: opencv 将图片的 base64 字符串读取为内存图片，传递给 hyperlpr 识别。其中，

- `np.fromstring` 函数用于将数组的字符流转换成结构化的 `ndarray` 数组
- `cv2.imdecode` 函数从内存读取图片数组并转换成 RBG image
- `hyperlpr.HyperLPR_plate_recognition` 函数读取 RBG image 并返回解析车牌解析结果

以上所有操作都在内存中进行，无须申请外部存储。

## 2. 编写触发器
触发器就是一个接收到某类事件会自动执行的函数，HTTP 触发器就是收到 HTTP 请求时会自动执行的函数，这里使用的就是 HTTP 触发器。

```python
def handler(environ, start_response):
    try:
        query_string = environ['QUERY_STRING']
        img_base64 = query_string.split("=")[1]
        result = licence_plate_parser(img_base64)
    except (KeyError):
        result = {"message": "image base64 required."}

    status = '200 OK'
    response_headers = [('Content-type', 'application/json')]
    start_response(status, response_headers)
    return [json.dumps(result).encode("utf8")]
```

> Python HTTP 触发器遵循 `WSGI` 规范，输入参数为:
> 
> - `environ`：是一个 Python `dict`，里面存放了所有和客户端相关的信息
> - `start_response`：是一个可调用函数, 在触发器将 body 作为返回值返回前需要先调用 start_response() 
> 
> 返回值为 HTTP 响应的具体 body 数据, 必须为 bytestring 的可迭代对象.

同时为了便于测试核心算法，写了一个测试用的触发器:

```python
def handler_test(environ, start_response):
    with open("test.jpg", "rb") as f:
        base64data = base64.b64encode(f.read())
    img_base64 = base64data.decode("utf8")

    result = licence_plate_parser(img_base64)
    status = '200 OK'
    response_headers = [('Content-type', 'application/json')]
    start_response(status, response_headers)
    return [json.dumps(result).encode("utf8")]
```

## 3. 部署服务
部署前我们看下 `template.yml` 文件内容如下，都是由工具自动生成，描述了服务和两个函数的配置详情，包括运行环境、触发方式、入口函数、资源规格等:

```yaml
ROSTemplateFormatVersion: '2015-09-01'
Transform: 'Aliyun::Serverless-2018-04-03'
Resources:
  lpr:
    Type: 'Aliyun::Serverless::Service'
    Properties:
      Description: LPR Service
    lpr:
      Type: 'Aliyun::Serverless::Function'
      Properties:
        Handler: index.handler
        Runtime: python3
        Timeout: 60
        MemorySize: 512
        CodeUri: ./
      Events:
        httpTrigger:
          Type: HTTP
          Properties:
            AuthType: ANONYMOUS
            Methods:
              - GET
    test:
      Type: 'Aliyun::Serverless::Function'
      Properties:
        Handler: index.handler_test
        Runtime: python3
        Timeout: 60
        MemorySize: 512
        CodeUri: ./ 
      Events:
        httpTrigger:
          Type: HTTP
          Properties:
            AuthType: ANONYMOUS
            Methods:
              - GET
```

现在，我们执行 `fun deploy` 将服务部署到线上，结果如下:

![](/images/lpr-based-on-serverless_2020-11-10-23-35-08.png)

可以看到部署完成后输出了两个触发器对应的 HTTP 端点，这就是提供 RESTful 车牌识别服务的 URL. 接下来就可以在应用中请求 `https://1812616409925047.cn-shenzhen.fc.aliyuncs.com/2016-08-15/proxy/lpr/lpr/?image={image_base64}` 来获取车牌识别结果。我们来请求下测试 URL 得到响应如下:

![](/images/lpr-based-on-serverless_2020-11-10-23-45-23.png)

可以看到，成功的输出了车牌识别结果.

同时可以在阿里云的控制台看到刚部署成功的两个函数:

![](/images/lpr-based-on-serverless_2020-11-10-23-42-46.png)




在部署到线上之前，可以先执行 `fun local start` 进行本地运行调试，输出如下:

```shell
➜ fun local start
using template: template.yml
HttpTrigger httpTrigger of lpr/lpr was registered
        url: http://localhost:8000/2016-08-15/proxy/lpr/lpr
        methods: [ 'GET' ]
        authType: ANONYMOUS
HttpTrigger httpTrigger of lpr/test was registered
        url: http://localhost:8000/2016-08-15/proxy/lpr/test
        methods: [ 'GET' ]
        authType: ANONYMOUS


function compute app listening on port 8000!
```

本地调试通过之后再执行 `fun deploy` 部署到线上。

# 总结
可以看到，基于 Serverless 来开发并上线一个车牌识别云服务已经变得非常简单，流程只需三步走:

1. 编写核心业务逻辑代码
2. 编写触发器代码
3. 部署服务

而传统基于 Serverful 的开发方式至少需要六步走:

1. 编写核心业务逻辑代码
2. 编写 HTTP 请求响应代码
3. 编写 Dockerfile 构建容器
4. 编写 Makefile 打包镜像
5. 登录服务器配置环境
6. 部署服务

Serverless 的高效可以让我们更快速的上云！而且所有与核心业务无关的操作如运维等都不用我们操心，以函数为粒度运行，这才是真正的弹性！s

最后作为对比，本文提供基于 Serverless 实现和 Serverful 实现的两种方式的完整代码，Demo 地址: https://github.com/serverless-lab/license-plate-recognition

# Q&A
## `NoneType` object has no attribute `split`
遇到请求服务返回如下错误信息:

![](/images/aliyun-serverless_2020-11-06-23-54-42.png)

错误原因: Python Runtime 中函数的签名遵循 `WSGI` 规范

解决方案: handler 触发器的返回值必须为 bytestring 的可迭代对象, 如返回元素为 `bytes` 的 `list`

```python
result = { "result": "" }

# 不合格返回值

return result

return [ result ]

return [ json.dumps(result) ]

# 合格返回值. bytes list

return [ json.dumps(result).encode("utf8") ]
```


# 参考
- 什么是 Serverless? https://docs.google.com/presentation/d/1bCxqoblAkl9IwxgLWkVIfWoUF-Oh1R0DTwz8Gysy6Dc/edit?usp=sharing
- 阿里云文档 - 函数计算 - Python HTTP 函数, http://help.aliyun.com/document_detail/74756.html
- PEP 3333 - Python Web Server Gateway Interface v1.0.1, https://www.python.org/dev/peps/pep-3333
- HyperLPR, https://github.com/szad670401/HyperLPR