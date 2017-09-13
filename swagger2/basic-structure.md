## 基本结构

Swagger可以用[JSON](https://en.wikipedia.org/wiki/JSON)或[YAML](https://en.wikipedia.org/wiki/YAML)编写。在本指南中，我们只使用YAML示例，但JSON工作效果同样好。

用YAML编写的Swagger示例如下所示：

```
swagger: "2.0"
info:
  title: Sample API
  description: API description in Markdown.
  version: 1.0.0

host: api.example.com
basePath: /v1
schemes:
  - https

paths:
  /users:
    get:
      summary: Returns a list of users.
      description: Optional extended description in Markdown.
      produces:
        - application/json
      responses:
        200:
          description: OK
```

### 元数据(Metadata)

每个Swagger规范以Swagger版本开始，2.0是最新版本。 Swagger版本定义了API规范的整体结构 - 决定了可以记录什么以及如何记录这些内容。

```
swagger: "2.0"
```

然后，需要指定API `info` -- `title`，`description`(可选)，`version`(API版本，不是文件版本或Swagger版本)。


```
info:
  title: Sample API
  description: API description in Markdown.
  version: 1.0.0
```

`version`可以是一个随机字符串。可以使用*major.minor.patch*(如[语义版本控制](http://semver.org/))，或任意格式，如*1.0-beta*或*2016.11.15*。

`description`可以是[多行的](http://stackoverflow.com/a/21699210)，并支持[Markdown格式](https://guides.github.com/features/mastering-markdown/)，用于富文本表示。

`info`还支持其他字段的联系信息，许可证和其他详细信息。

_参考:_ [Info Object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md#infoObject)。

### 基本URL

所有API调用的基本URL都使用`schemes`，`host`和`basePath`定义：

```
host: api.example.com
basePath: /v1
schemes:
  - https
```

所有API路径都是相对于基本URL。例如，*/users*实际上意味着*https://api.example.com/v1/users*。

_更多信息：_ [API主机和基本URL](api-host-and-base-path.html)。

### 消费，生产(Consumes, Produces)

`Consumes`和`Consumes`部分定义API支持的MIME类型。根级别定义可以在各个操作中被覆盖。

```
consumes:
  - application/json
  - application/xml
produces:
  - application/json
  - application/xml
```

_更多信息：_ [MIME类型](mime-types.html)。

### 路径(Paths)

`paths`部分定义了API中的各个端点(路径)以及这些端点支持的HTTP方法(操作)。例如，*GET /users*可以描述为：

```
paths:
  /users:
    get:
      summary: Returns a list of users.
      description: Optional extended description in Markdown.
      produces:
        - application/json
      responses:
        200:
          description: OK
```

_更多信息：_ [路径和操作](paths-and-operations.html)。

### 参数

操作可以具有可以通过URL路径(`/users/{userId}`)，查询字符串(`/ users?role=admin`)，头(`X-CustomHeader: Value`)和请求体传递的参数。可以定义参数类型，格式，是否需要或可选，以及其他详细信息：

```
paths:
  /users/{userId}:
    get:
      summary: Returns a user by ID.
      parameters:
        - in: path
          name: userId
          required: true
          type: integer
          minimum: 1
          description: Parameter description in Markdown.
      responses:
        200:
          description: OK
```

_更多信息：_ [描述参数](parameters.html)。

### 响应

对于每个操作，可以定义可能的状态代码，例如200 OK或404 Not Found，以及响应主体的`schema`。模式可以通过`$ref`内联定义或从外部定义引用。还可以为不同的内容类型提供示例响应。

```
paths:
  /users/{userId}:
    get:
      summary: Returns a user by ID.
      parameters:
        - in: path
          name: userId
          required: true
          type: integer
          minimum: 1
          description: The ID of the user to return.
      responses:
        200:
          description: A User object.
          schema:
            type: object
            properties:
              id:
                type: integer
                example: 4
              name:
                type: string
                example: Arthur Dent
        400:
          description: The specified user ID is invalid (e.g. not a number).
        404:
          description: A user with the specified ID was not found.
        default:
          description: Unexpected error
```

_更多信息：_ [描述响应](responses.html)。

### 输入和输出模型

全局`definitions`部分可以定义API中使用的常见数据结构。每当需要`schema`(无论是请求体还是响应体)，都可以通过`$ref`来引用它们。

例如，这个JSON对象：

```
{
  "id": 4,
  "name": "Arthur Dent"
}
```

可以表示为：

```
definitions:
  User:
    properties:
      id:
        type: integer
      name:
        type: string
    # Both properties are required
    required:
      - id
      - name
```

然后在请求主体模式和响应主体模式中引用如下：

```
paths:
  /users/{userId}:
    get:
      summary: Returns a user by ID.
      parameters:
        - in: path
          name: userId
          required: true
          type: integer
      responses:
        200:
          description: OK
          schema:
            $ref: '#/definitions/User'
  /users:
    post:
      summary: Creates a new user.
      parameters:
        - in: body
          name: user
          schema:
            $ref: '#/definitions/User'
      responses:
        200:
          description: OK
```

<!-- _More info:_ [Data Models](TODO). -->

### 认证

`securityDefinitions`和`security`关键字用于描述API中使用的身份验证方法。

```
securityDefinitions:
  BasicAuth:
    type: basic

security:
  - BasicAuth: []
```

支持的认证方法有：

* [基本认证](authentication/basic-authentication.html)
* [API密钥](authentication/api-keys.html)(作为标题或查询参数)
* [OAuth 2](authentication/oauth2.html)公共流(隐式，密码，应用和访问代码)

_更多信息：_ [验证](authentication/index.html)。