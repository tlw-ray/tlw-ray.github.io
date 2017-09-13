## API主机和基本URL

REST API具有添加端点路径的基本URL。基本URL由API规范的根级别由`scheme`，`host`和`basePath`定义。

```
host: petstore.swagger.io
basePath: /v2
schemes:
  - https
```

所有API路径都是相对于这个基本URL，例如`/users`实际上是指`<scheme>://<host>/<basePath>/users`。

![URL结构](images/url-structure.png)

### Schemes

`scheme`是API使用的传输协议。 Swagger支持`http`，`https`和`WebSocket`(https://en.wikipedia.org/wiki/WebSocket)方案 - `ws`和`wss`。与YAML中的任何列表一样，可以使用列表语法指定方案：

```
schemes:
  - http
  - https
```

或数组：

```
schemes: [http, https]
```

如果未指定`scheme`，则用于提供API规范的方案将用于API调用。

### 主机

`host`是为API服务的主机的域名或IP地址(IPv4)。如果不同于方案的默认端口(HTTP为HTTP 80，HTTPS为443)，则需要包含端口号。请注意，必须仅仅是主机，不带 *http(s)://* 或子路径。

有效主机：

```
api.example.com
example.com:8089
93.184.216.34
93.184.216.34:8089
```

不正确的主机：

```
http://api.example.com
example.com/api/v1
```

如果没有指定`host`，那么它被认为是与正在提供API文档的主机相同的主机。

### basePath

`basePath`是相对于主机根的所有API路径的URL前缀。它必须以一个主要的斜杠开头`/`。如果没有指定`basePath`，它将默认为`/`，即所有路径都从主机根开始。

有效的基本路径：

```
/V2
/API/V2
/
```

不正确的基本路径：

```
V2
```

### 省略主机和方案

`host`和`scheme`可以省略一个更加动态的关联。在这种情况下，用于提供API文档的主机和方案将用于API调用。例如，如果Swagger基于UI的文档托管在 *https://api.example.com/apidocs/index.html* ，则“尝试”API调用将被定向到 *https://api.example.com*。

### FAQ

##### **能否指定多个主机，分别用于开发，测试和生产？**

Swagger 2.0仅支持一个`host`在每个API规范(如果要将HTTP和HTTPS计为不同的主机，则只有两个)。

定位多个主机的一种可能的方法是从规范中省略`host`和`schema`，并从每个主机提供服务。在这种情况下，规范的每个副本将针对相应的主机。

规范中3.0版将支持多台主机。

##### **主机和basePath支持模板吗？如：**

```
https://{customer_id}.saas-app.com/api/v1
https://api.saas-app.com/v1/{customer_id}/apis
```

在Swagger 2.0中还不行，但是在3.0版本中将是可能的。

有关主机模板的解决方法，请参阅上一个问题。

##### **可以为HTTP和HTTPS指定不同的端口吗？如：**

```
http://example.com:8080
https://example.com:8443
```

Swagger 2.0不支持这一点，但是在下一个版本3.0中可能会出现这种情况。

现在，就可以省略“host”和“scheme”，并从两个主机提供规范。这样，规范的每个副本将针对用于访问该规范的主机和端口。

### 参考

[https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md#swagger-object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md#swagger-object)