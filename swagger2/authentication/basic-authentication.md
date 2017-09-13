### 基本认证

[基本认证](https://en.wikipedia.org/wiki/Basic_access_authentication)是内置于HTTP协议中的非常简单的认证方案。客户端发送包含`Basic`字的Authorization头的HTTP请求，后跟一个空格和base64编码的 `username:password`字符串。例如，包含`demo` / `p@55w0rd`凭证的标题将被编码为：

```
Authorization: Basic ZGVtbzpwQDU1dzByZA==
```

**注意：** 因为base64容易解码，所以基本身份验证只能与其他安全机制（如HTTPS/SSL）一起使用。

基本认证易于定义。在全局`securityDefinitions`部分，添加一个包含`type：basic`的条目和一个任意的名字（在这个例子中是 *basicAuth*）。然后，通过使用`security`部分对整个API或特定操作应用安全性。

```
securityDefinitions:
  basicAuth:
    type: basic

# To apply Basic auth to the whole API:
security:
  - basicAuth: []

paths:
  /something:
    get:
      # To apply Basic auth to an individual operation:
      security:
        - basicAuth: []
      responses:
        200:
          description: OK (successfully authenticated)
```

#### 401回应

可以为缺少或不正确凭据的请求定义401“未经授权”的响应。此响应包括可能涉及的“WWW-Authenticate”标题。

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
    description: Authentication information is missing or invalid
    headers:
      WWW_Authenticate:
        type: string
```