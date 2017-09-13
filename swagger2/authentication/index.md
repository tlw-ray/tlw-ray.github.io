# 验证

Swagger 2.0允许为API定义以下身份验证类型：

* [基本认证](basic-authentication.html)

* [API键](api-keys.html)（作为标题或查询字符串参数）

* [OAuth 2](oauth2.html)公用流（授权码，隐式资源所有者密码凭证，客户端凭证）

按照上面的链接，了解这些身份验证类型的具体示例，或者继续阅读以了解如何一般地描述身份验证。

通过使用`securityDefinitions`和`security`关键字描述认证。使用`securityDefinitions`来定义API支持的所有验证类型，然后使用`security`将特定的身份验证类型应用于整个API或各个操作。

## 定义安全方案

`securityDefinitions`部分用于定义API支持的所有安全方案（认证类型）。它是一个名称->定义映射，将任意名称映射到安全方案定义。

在这里，API支持三种名为_BasicAuth_，_ApiKeyAuth_和_OAuth2_的安全方案，这些名称将用于从其他地方引用这些安全方案：

```
securityDefinitions:
  BasicAuth:
    type: basic
  ApiKeyAuth:
    type: apiKey
    in: header
    name: X-API-Key
  OAuth2:
    type: oauth2
    flow: accessCode
    authorizationUrl: https://example.com/oauth/authorize
    tokenUrl: https://example.com/oauth/token
    scopes:
      read: Grants read access
      write: Grants write access
      admin: Grants read and write access to administrative information
```

每种安全方案都可以是`type`：

* `basic`用于基本认证

* `apiKey`用于API密钥

* `oauth2`为OAuth 2

其他必需属性取决于安全类型。有关详细信息，请查看[Swagger规范](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md#securitySchemeObject)或[Basic auth](basic-authentication.html)的示例），[API键](api-keys.html)和[OAuth 2](oauth2.html)。

## 应用安全方案

在`securityDefinitions`中定义了安全方案之后，可以通过在根级别或操作级别分别添加`security`部分来将其应用于整个API或各个操作。

当在根级别使用时，`security`将全局指定的安全方案应用于所有API操作，除非在操作级别被覆盖。在以下示例中，API调用可以使用API​​密钥或OAuth 2进行身份验证。 _ApiKeyAuth_ 和 _OAuth2_ 名称是指先前在`securityDefinitions`中定义的安全方案。

```
security:
  - ApiKeyAuth: []
  - OAuth2: [read, write]
```

全局`security`可以在各个操作中被覆盖，以使用不同的身份验证类型，不同的OAuth 2范围，或者根本没有认证：

```
 paths:
  /billing_info:
    get:
      summary: Gets the account billing info
      security:
        - OAuth2: [admin]   # Use OAuth with a different scope
      responses:
        200:
          description: OK
        401:
          description: Not authenticated
        403:
          description: Access token does not have the required scope

  /ping:
    get:
      summary: Checks if the server is running
      security: []   # No security
      responses:
        200:
          description: Server is up and running
        default:
          description: Something is wrong
```

## 使用多种身份验证类型

一些REST API支持多种身份验证类型。 `security`部分可以使用逻辑OR和AND组合安全性要求，以达到所需的结果。

`security`使用以下逻辑：

```
security:
  - A
  - B

 # A OR B
```

```
security:
  - A
    B

 # A AND B
```

```
security:
  - A
    B
  - C
    D

 # (A AND B) OR (C AND D)
```

也就是说，`security`是一个hashmaps数组，每个hashmap包含一个或多个命名的安全性方案。使用逻辑“与AND”组合哈希函数中的项，并且使用逻辑“或OR”组合数组项。通过OR组合的安全方案是替代方案 - 任何一种可以在给定的上下文中使用。通过AND组合的安全方案必须在同一请求中同时使用。

在这里，我们可以使用基本身份验证或API密钥：

```
security:
  - basicAuth: []
  - apiKey: []
```

在这里，API需要在请求中包含一对API密钥：

```
security:
  - apiKey1: []
    apiKey2: []
```

在这里，我们可以使用OAuth 2或一对API密钥：

```
security:
  - oauth2: [scope1, scope2]
  - apiKey1: []
    apiKey2: []
```

[OAuth 2](oauth2.html)页面具有将不同作用域分配给不同操作的示例。

## FAQ

#### `securitySchemeName: []`中的[]是什么意思？

`[]`是空数组的YAML/JSON语法。 Swagger规范要求`security`数组中的项目指定所需范围的列表，如：

```
security:
  - securityA: [scopeA1, scopeA2]
  - securityB: [scopeB1, scopeB2]
```

范围仅用于OAuth 2，所以Basic和API密钥`security`项目改为使用空数组。

```
security:
  - oauth2: [scope1, scope2]
  - BasicAuth: []
  - ApiKeyAuth: []

```

##参考

`securityDefinitions`: [https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md#securityDefinitionsObject](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md#securityDefinitionsObject)

`security`: [https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md#securityRequirementObject](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md#securityRequirementObject)