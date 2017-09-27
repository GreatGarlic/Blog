---
title: 简单的 Mock 工具 RestServerMock
date: 2017-09-27 13:55:19
tags: FE
---

前后端分离，如果前端需要等到服务器端接口开发完成后才能继续的话，效率太低，使用 Mock 工具能够更好的使得前后端分离各自开发，`RestServerMock` 一个是简单的静态 Web 服务器的 Mock 工具:

> A Simple REST HTTP server that serves the configured JSON responses rest-server-mock.
>
> Currently it servers JSON responses with status code 200. Future versions will support more options including : status codes, headers, encodings, etc.

<!--more-->

## 安装

1. 安装 Nodejs
2. 安装 RestServerMock: `npm install  -g rest-server-mock`

## 配置

把下面的 JSON 配置保存为 config.json:

```json
{
    "port": 8080,
    "routes": [{
        "path": "/a",
        "type": "GET",
        "response": {
            "field_1": "f1 get a ok",
            "field_2": "f2 get a ok"
        }
    }, {
        "path": "/a",
        "type": "POST",
        "response": {
            "field_1": "f1 post a ok",
            "field_2": "f2 post a ok"
        }
    }, {
        "path": "/a",
        "type": "PUT",
        "response": {
            "field_1": "f1 put a ok",
            "field_2": "f2 put a ok"
        }
    }, {
        "path": "/a",
        "type": "DELETE",
        "response": {
            "field_1": "f1 delete a ok",
            "field_2": "f2 delete a ok"
        }
    }, {
        "path": "/token",
        "type": "GET",
        "response": {
            "token": "49189B37-A5FA-4131-981F-D0EBA5C986D9"
        }
    }]
}
```

* port: Web 服务的端口
* path: URI
* type: Http 请求的方法
* response: 响应的 JSON 数据

## 测试

* 启动服务: `rest-server-mock config.json`

* 使用 REST 工具例如 Postman、浏览器等访问 http://localhost:8080/token，返回

  ```json
  {
    "token" : "49189B37-A5FA-4131-981F-D0EBA5C986D9"
  }
  ```

## 参考资料

RestServerMock 的 Github https://github.com/pmatzavin/restServerMock