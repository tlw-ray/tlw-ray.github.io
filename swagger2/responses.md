## 描述回应

API规范需要为所有API操作指定`responses`。每个操作必须至少定义一个响应，通常是成功的响应。响应由其HTTP状态代码和响应主体和/或标题中返回的数据定义。

这是一个最小的例子：

```
paths:
  /ping:
    get:
      produces:
        - application/json
      responses:
        200:
          description: OK
```

### 响应媒体类型

Response Media Types

API可以响应各种媒体类型。 JSON是数据交换最常用的格式，但不是唯一可能的格式。

要指定响应媒体类型，请在根级别或操作级别使用`produce`关键字。可以在操作级别覆盖全局列表。

```
produces:
  - application/json

paths:
  # This operation returns JSON - as defined globally above
  /users:
    get:
      summary: Gets a list of users.
      responses:
        200:
          description: OK
  # Here, we override the "produces" array to specify other media types
  /logo:
    get:
      summary: Gets the logo image.
      produces:
        - image/png
        - image/gif
        - image/jpeg
      responses:
        200:
          description: OK
```

_更多信息：_ [MIME类型](mime-types.md)。

### HTTP状态码

在`responses`下，每个响应定义以状态代码开始，例如200或404.操作通常返回一个成功的状态代码和一个或多个错误状态。

每个响应状态都需要一个`description`。例如，可以描述错误响应的条件。 [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/)可用于富文本表示。

```
      responses:
        200:
          description: OK
        400:
          description: Bad request. User ID must be an integer and bigger than 0.
        401:
          description: Authorization information is missing or invalid.
        404:
          description: A user with the specified ID was not found.
```

请注意，API规范不一定需要覆盖 *所有可能的* HTTP响应代码，因为它们可能不是提前知道的。然而，预计会覆盖成功的回应和任何 *已知的* 错误。通过“已知错误”，我们意味着例如对于通过ID返回资源的操作的404 Not Found响应，或者在无效操作参数的情况下的400错误请求响应。

### 响应体

`schema`关键字用于描述响应体。模式可以定义：

* `object`或`array` - 通常与JSON和XML API一起使用，
* 一个原始的数字或字符串 - 用于纯文本响应，
* `file`（参见[下面](#响应返回文件)）。

模式可以在操作中内联定义：

```
      responses:
        200:
          description: A User object
          schema:
            type: object
            properties:
              id:
                type: integer
                description: The user ID.
              username:
                type: string
                description: The user name.
```

或在根级别定义并通过`$ref`引用。如果多个响应使用相同的模式，这会很有用。

```
      responses:
        200:
          description: A User object
          schema:
            $ref: "#/definitions/User"
definitions:
  User:
    type: object
    properties:
      id:
        type: integer
        description: The user ID.
      username:
        type: string
        description: The user name.
```

### 响应返回文件

API操作可以返回文件，例如图像或PDF。在这种情况下，使用`type：file`定义响应`schema`，并在`produce`部分指定适当的MIME类型。

```
paths:
  /report:
    get:
      summary: Returns the report in the PDF format.
      produces:
        - application/pdf
      responses:
        200:
          description: A PDF file.
          schema:
            type: file
```

### 空返回体
Empty Response Body

一些回应没有body，如​​：

* 204无内容
* 201已创建（创建的资源的URL在“位置”标题中而不是正文中返回）

要指示响应正文是空的，请不要为响应指定`schema`。 Swagger将没有模式视为没有身体的响应。

```
      responses:
        204:
          description: The resource was deleted successfully.
```

### 响应头
Response Headers

来自API的响应可以包括自定义标题，以提供有关API调用结果的其他信息。例如，速率限制API可以通过响应头提供速率限制状态，如下所示：

```
HTTP 1/1 200 OK
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 99
X-RateLimit-Reset: 2016-10-12T11:00:00Z

{ ... }
```

可以为每个响应定义自定义的`headers`，如下所示：

```
paths:
  /ping:
    get:
      summary: Checks if the server is alive.
      responses:
        200:
          description: OK
          headers:
            X-RateLimit-Limit:
              type: integer
              description: Request limit per hour.
            X-RateLimit-Remaining:
              type: integer
              description: The number of requests left for the time window.
            X-RateLimit-Reset:
              type: string
              format: date-time
              description: The UTC date/time at which the current rate limit window resets.
```

请注意，目前，Swagger无法为不同的响应代码或不同的API操作定义常见的响应头。需要单独定义每个响应的标题。

### 默认响应

有时，操作可以使用不同的HTTP状态代码返回多个错误，但都具有相同的响应结构：

```
      responses:
        200:
          description: Success
          schema:
            $ref: "#/definitions/User"
        400:
          description: Bad request
          schema:
            $ref: "#/definitions/Error"    # <-----
        404:
          description: Not found
          schema:
            $ref: "#/definitions/Error"    # <-----
```

可以使用`default`响应来共同描述这些错误，而不是单独描述。 “默认值”表示此响应用于此操作未单独涵盖的所有HTTP代码。

```
      responses:
        200:
          description: Success
          schema:
            $ref: '#/definitions/User'
        # Definition of all error statuses
        default:
          description: Unexpected error
          schema:
            $ref: '#/definitions/Error'
```

### 重复使用回应

如果多个操作返回相同的响应（状态代码和数据），可以在全局`responses`部分定义它，并通过`$ref`在操作级别引用该定义。这对于具有相同状态代码和响应正文的错误响应非常有用。

```
paths:
  /users:
    get:
      summary: Gets a list of users.
      response:
        200:
          description: OK
          schema:
            $ref: "#/definitions/ArrayOfUsers"
        401:
          $ref: "#/responses/Unauthorized"   # <-----
  /users/{id}:
    get:
      summary: Gets a user by ID.
      response:
        200:
          description: OK
          schema:
            $ref: "#/definitions/User"
        401:
          $ref: "#/responses/Unauthorized"   # <-----
        404:
          $ref: "#/responses/NotFound"       # <-----
# Descriptions of common responses
responses:
  NotFound:
    description: The specified resource was not found
    schema:
      $ref: "#/definitions/Error"
  Unauthorized:
    description: Unauthorized
    schema:
      $ref: "#/definitions/Error"
definitions:
  # Schema for error response body
  Error:
    type: object
    properties:
      code:
        type: string
      message:
        type: string
    required:
      - code
      - message
```

请注意，根级别定义的响应不会自动应用于所有操作。这些只是可以被多个操作引用和重用的定义。

### 示例响应

Swagger允许在API规范中包含响应示例，如[提供示例值](examples.html)中所述。一些API模拟工具使用这些示例来生成模拟响应。

### FAQ

**可以根据请求参数有不同的答复吗？如：**

```
GET /something -> {200, schema_1}
GET /something?foo=bar -> {200, schema_2}
```

不，不支持。

### 参考

Swagger的响应(Response)规范参考了如下内容：

[https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md#responsesDefinitionsObject](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md#responsesDefinitionsObject)

[https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md#responsesObject](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md#responsesObject)

[https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md#responseObject](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md#responseObject)