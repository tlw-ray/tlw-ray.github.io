## 描述请求体
(Request Body)

POST，PUT和PATCH请求可以具有请求正文（载荷），例如JSON或XML数据。在Swagger术语中，请求体称为 **body参数** 。只能有一个body参数，尽管操作可能有其他参数（path，query，header）。

**注意：** `application/x-www-form-urlencoded`和`multipart/form-data`请求的有效内容使用[表单参数](parameters.html#表单参数)描述，而不是body参数。

body参数在操作参数部分定义，包括以下内容：

* `in: body`

* `schema`，描述body数据类型和结构。数据类型通常是一个对象，但也可以是一个基元（如字符串或数字）或数组。

* 可选`description`。

* 有效载荷名称。它是必需的但被忽略（仅用于文档目的）。

### 对象有效载荷（JSON等）

许多API将数据作为对象传输，如JSON。对于一个对象，`schema`应该指定`type: object`和该对象的属性。

例如，使用这个JSON体的`POST /users`操作：

```
{
  "userName":  "Trillian",
  "firstName": "Tricia",
  "lastName":  "McMillan"
}
```

可以描述为：

```
paths:
  /users:
    post:
      summary: Creates a new user.
      consumes:
        - application/json
      parameters:
        - in: body
          name: user
          description: The user to create.
          schema:
            type: object
            required:
              - userName
            properties:
              userName:
                type: string
              firstName:
                type: string
              lastName:
                type: string
      responses:
        201:
          description: Created
```

**注意：** body参数的名称被忽略。它仅用于文档目的。

对于更模块化的样式，可以将模式定义移动到全局`definitions`部分，并使用`$ref`引用它们：

```
paths:
  /users:
    post:
      summary: Creates a new user.
      consumes:
        - application/json
      parameters:
        - in: body
          name: user
          description: The user to create.
          schema:
            $ref: "#/definitions/User"     # <----------
     responses:
         200:
           description: OK
definitions:
  User:           # <----------
    type: object
    required:
      - userName
    properties:
      userName:
        type: string
      firstName:
        type: string
      lastName:
        type: string
```

### 主体(Primitive Body)

想要POST/PUT只是一个值？没有问题，可以将主体模式类型定义为基元，例如字符串或数字。

原始请求：

```
POST /status HTTP/1.1
Host: api.example.com
Content-Type: text/plain
Content-Length: 42

Time is an illusion. Lunchtime doubly so.
```

Swagger定义：

```
paths:
  /status:
    post:
      summary: Updates the status message.
      consumes:
        - text/plain      # <----------
    parameters:
      - in: body
        name: status
        required: true
        schema:
          type: string  # <----------
    responses:
      200:
        description: Success!
```