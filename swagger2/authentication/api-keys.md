### API密钥

一些API使用API​​密钥进行授权。 API密钥是客户端在进行API调用时需要提供的特殊令牌。密钥通常作为请求头发送：

```
GET /something HTTP/1.1
X-API-Key: abcdef12345
```

或作为查询参数：

```
GET /something?api_key=abcdef12345
```

API密钥应该是只有客户端和服务器知道的秘密。如果不与其他安全机制（如HTTPS/SSL）一起使用，基本密钥验证就不被认为是安全的。

定义API`key-base`安全：

* 在全局`securityDefinitions`部分添加一个`type: apiKey`的条目。条目名称可以是任意的（例如下面的示例中的*APIKeyHeader*），用于在其它部分引用该规范的安全定义。

* 指定API密钥是否会传入`in: header`或`in: query`。

* 指定一个`name`在参数或头信息中。

```
securityDefinitions:
   # X-API-Key: abcdef12345
   APIKeyHeader:
     type: apiKey
     in: header
     name: X-API-Key
   # /path?api_key=abcdef12345
   APIKeyQueryParam:
     type: apiKey
     in: query
     name: api_key
```

然后，使用根级别或操作级别的`security`部分将API密钥应用于整个API或特定操作。

```
# Global security (applies to all operations):
security:
  - APIKeyHeader: []
paths:
  /something:
    get:
      # Operation-specific security:
      security:
        - APIKeyQueryParam: []
      responses:
        200:
          description: OK (successfully authenticated)
```

请注意，可以在API中支持多种授权类型。请参阅[使用多个身份验证类型](index.html#使用多个身份验证类型)。

#### 成对API密钥

一些API使用成对安全密钥，例如API密钥和应用程序ID。要指定密钥一起使用（如逻辑AND），将它们列在`security`数组中的相同数组项中：

```
securityDefinitions:
  apiKey:
    type: apiKey
    in: header
    name: X-API-KEY
  appId:
    type: apiKey
    in: header
    name: X-APP-ID
security:
  - apiKey: []
    appId: []
```

注意与以下不同之处：

```
security:
  - apiKey: []
  - appId: []
```

这意味着可以使用任何一个密钥（如逻辑或）。有关更多示例，请参阅[使用多个身份验证类型](index.html#使用多个身份验证类型)。

#### 401回应

可以为缺少或无效API密钥的请求定义401“未经授权”的响应。此响应包括可能想提及的`WWW-Authenticate`标题。

与其他常见响应一样，401响应可以在全局`responses`部分中定义，并从多个操作中引用。

```
paths:
  /something:
    get:
      ...
      responses:
        ...
        401:
           $ref: "#/responses/UnauthorizedError"
    post:
      ...
      responses:
        ...
        401:
          $ref: "#/responses/UnauthorizedError"
responses:
  UnauthorizedError:
    description: API key is missing or invalid
    headers:
      WWW_Authenticate:
        type: string
```