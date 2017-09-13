## 描述参数

在Swagger中，API操作参数在操作定义的`parameters`部分定义。每个参数都有名称`name`，值类型`type`（用于原始值类型参数）或`schema`（对于请求体）和可选的`description`。例如：

```
paths:
  /users/{userId}:
    get:
      summary: Gets a user by ID.
      parameters:
        - in: path
          name: userId
          type: integer
          required: true
          description: Numeric ID of the user to get.
```

请注意，`parameters`是一个数组，所以在YAML中，每个参数定义都必须在前面加上一个破折号（`-`）。

### 参数类型

Swagger根据参数位置区分以下参数类型。该位置由参数的`in`键确定，例如`in: query`或`in: path`。

* [查询参数](#查询参数)，如`/users?role=admin`

* [路径参数](#路径参数)，如`/ users/{id}`

* [头参数](#头参数)，如`X-MyHeader: Value`

* body参数描述POST，PUT和PATCH请求的主体（请参阅[描述请求体](request-body.html)）

* [表单参数](#表单参数) - 各种body参数用于描述带有`application/x-www-form-urlencoded`和`multipart/form-data`这样的`Content-Type`的请求的有效载荷（后者通常用于[文件上传](file-upload.html)）

### 查询参数

查询参数是最常见的参数类型。请求URL的尾部跟随一个问号（`?`），问号后面跟随不同的`name=value`对，其间由&符号分隔（`&`）。查询参数可以是必要或可选的。例如：

```
GET /pets/findByStatus?status=available
GET /notes?offset=100&limit=50
```

使用`in: query`来表示查询参数：

```
     parameters:
        - in: query
          name: offset
          type: integer
          description: The number of items to skip before starting to collect the result set.
        - in: query
          name: limit
          type: integer
          description: The numbers of items to return.
```

查询参数仅支持基本类型。可以包含一个`array`，但`items`必须是原始值类型。不支持对象。

**注意：** 要描述作为查询参数传递的API键，请改用安全性定义。请参阅[API密钥](authentication/api-keys.html)。

### 路径参数

路径参数是可以变化的URL路径的组件。它们通常用于指向集合中的特定资源，例如由ID标识的用户。 URL可以有几个路径参数，每个路径参数都用大括号`{}`表示。

```
GET /users/{id}
GET /cars/{carId}/drivers/{driverId}
```

当客户端进行API调用时，每个路径参数都必须用实际值代替。

在Swagger中，根据需要使用`in：path`或其他属性定义路径参数。参数名称必须与路径中指定的相同。此外，请记住添加`required: true`，因为路径参数总是必须的。

以下是`GET /users/{id}`的示例：

```
paths:
  /users/{id}:
    get:
      parameters:
        - in: path
          name: id   # Note the name is the same as in the path
          required: true
          type: integer
          minimum: 1
          description: The user ID.
       responses:
         200:
           description: OK
```

路径参数可以是多值的，例如`GET /users/12,34,56`。这是通过将参数类型指定为“array”来实现的。请参见下面的[数组和多值参数](#array-and-multi-value-parameters)。

### 头参数

API调用可能需要使用HTTP请求发送自定义标头(Head Parameter)。Swagger可以将自定义请求标头定义为`in: header`参数。

例如，假设对`GET /ping`的调用需要`X-Request-ID`头：

```
GET /ping HTTP/1.1
Host: example.com
X-Request-ID: 77e1c83b-7bb0-437b-bc50-a7a58e5660ac
```

在Swagger中，将定义此操作如下：

```
paths:
  /ping:
    get:
      summary: Checks if the server is alive.
      parameters:
        - in: header
          name: X-Request-ID
          type: string
          required: true
```

以类似的方式，可以定义[自定义响应标头](#头参数)。

**注意：** Swagger规范对某些标题有特殊关键字：

|标题| Swagger关键字|有关更多信息，请参阅... |
| --------------- | ---------------------- | ---------------------- |
| `Content-Type`  | `consumes`, `produces` | [MIME Types](mime-types.html) |
| `Authorization` | `securityDefinitions`, `security` | [Authentication](authentication/index.html) |


### 表单参数

表单(Form Parameters)参数用于描述具有以下内容的`Content-Type`的请求的有效内容：

* `application/x-www-form-urlencoded`（用于POST原始值和原始值的数组）。

* `multipart/form-data`（用于上传文件或文件和原始数据的组合）。

也就是说，操作的`consumes`属性必须指定这些内容类型之一。

表单参数定义为`in: formData`。它们只能是基元（字符串，数字，布尔值）或基元数组（意味着不能使用`$ref`作为`items`值）。此外，表单参数不能与`in: body`参数共存，因为`formData`是描述正文的一种特定方式。

为了说明表单参数，请考虑使用HTML POST表单：

```
<form action="http://example.com/survey" method="post">
  <input type="text"   name="name" />
  <input type="number" name="fav_number" />
  <input type="submit"/>
 </form>
```

此表单将数据POST到表单的端点：

```
POST /survey HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 29

name=Amy+Smith&fav_number=321
```

在Swagger中，可以如下描述端点(endpoint)：

```
paths:
  /survey:
    post:
      summary: A sample survey.
      consumes:
        - application/x-www-form-urlencoded
      parameters:
        - in: formData
          name: name
          type: string
          description: A person's name.
        - in: formData
          name: fav_number
          type: number
          description: A person's favorite number.
      responses:
        200:
          description: OK
```

要了解如何定义文件上传的表单参数，请参阅[文件上传](file-upload.html)。

### 必需和可选参数

默认情况下，Swagger将所有请求参数视为可选的。可以根据需要添加`required: true`来标记必选参数。请注意，路径参数必须具有`required: true`，因为它们始终是必需的。

```
      parameters:
        - in: path
          name: userId
          type: integer
          required: true    # <----------
          description: Numeric ID of the user to get.
```

### 默认参数值

可以使用`default`键指定可选参数的默认值。如果客户端没有在请求中提供参数值，则默认值是服务器使用的值。值类型必须与参数的数据类型相同。

一个典型的例子是分页参数，如偏移和限量：

```
GET /users
GET /users?offset=30&limit=10
```

假设offset默认为0，默认值为20，范围为0到100，您可以将这些参数定义为：

```
      parameters:
        - in: query
          name: offset
          type: integer
          required: false
          default: 0
          minimum: 0
          description: The number of items to skip before starting to collect the result set.
        - in: query
          name: limit
          type: integer
          required: false
          default: 20
          minimum: 1
          maximum: 100
          description: The numbers of items to return.
```

#### 常见错误

使用`default`关键字时有两个常见的错误：

* 使用`default`与`required`参数或属性，例如，使用路径参数。这没有意义 - 如果需要值，客户端必须始终发送它，并且不会使用默认值。

* 使用`default`指定一个样本值。这不是默认的使用，可能会导致某些Swagger工具中的意外行为。该规范的某些元素支持`example`或`examples`关键字用于此目的。请参阅[提供示例值](provide-example-values.html)。

### 枚举参数

`enum`关键字可以将参数值限制为一组固定值。枚举值必须与参数`type`具有相同的类型。

```
        - in: query
          name: status
          type: string
          enum: [available, pending, sold]
```

更多信息：[定义枚举](enums.html)。

### 数组和多值参数

路径，查询，标题和表单参数可以接受值列表，例如：

```
GET /users/12,34,56,78
GET /resource?param=value1,value2,value3
GET /resource?param=value1&param=value2&param=value3

POST /resource
param=value1&param=value2
```

多值参数必须用`type：array`和适当的`collectionFormat`定义。

```
      # color=red,black,white
      parameters:
        - in: query
          name: color
          type: array
          collectionFormat: csv
          items:
            type: string
```

`collectionFormat`指定数组格式（具有多个参数的单个参数或具有相同名称的多个参数）和数组项的分隔符。


| collectionFormat |说明|示例|
| ---------------- | ----------------------- | -------------- |
| `csv`（默认）|逗号分隔值。 |`foo,bar,baz` |
| `ssv` |空格分隔值。 | `foo bar baz` |
| `tsv`|制表符分隔值。 | `"foo\tbar\tbaz"`|
| `pipes` |管道分隔值。 | `foo|bar|baz` |
| `multi` |多个参数实例，而不是多个值。这仅适用于`in: query`和`in: formData`参数。 | `foo=value&foo=another_value` |

此外，还可以：

* 使用`minItems`和`maxItems`来控制数组的大小，
* 强制执行`uniqueItems`，
* 将数组项限制为一组固定的`enum`。

例如：
```
        - in: query
          name: color
          required: false
          type: array
          minItems: 1
          maxItems: 5
          uniqueItems: true
          items:
            type: string
            enum: [black, white, gray, red, pink, orange, yellow, green, blue, purple, brown]
```

如果省略此参数，还可以指定服务器将使用的默认数组：

```
        - in: query
          name: sort
          required: false
          type: array
          items:
            type: string
          default: ["-modified", "+id"]
```

### 常量参数

可以将常量参数定义为只有一个可能值的必需参数：

```
- required: true
  enum: [value]
```

`enum`属性指定可能的值。在此示例中，只能使用一个值，这将是Swagger UI中唯一可供用户选择的值。

**注意：** 常数参数与[默认参数值](#default-parameter-values)不同。常量参数始终由客户端发送，而如果参数不是由客户端发送，则默认值是服务器使用的值。

### 没有值的参数

查询字符串和表单数据参数可能只能有一个名字，没有值：

```
GET /foo?metadata

POST /something
foo&bar&baz
```

使用`allowEmptyValue`来描述这样的参数：

```
        - in: query
          name: metadata
          required: true
          type: boolean
          allowEmptyValue: true  # <-----
```

### 常用参数

#### 路径的所有方法的通用参数

参数可以在路径本身下定义，在这种情况下，参数存在于此路径下描述的所有操作中。一个典型的例子是操作通过路径参数访问的相同资源的GET/PUT/PATCH/DELETE操作。

```
paths:
  /user/{id}:
    parameters:
      - in: path
        name: id
        type: integer
        required: true
        description: The user ID.

    get:
      summary: Gets a user by ID.
      ...
    patch:
      summary: Updates an existing user with the specified ID.
      ...
    delete:
      summary: Deletes the user with the specified ID.
      ...
```

在操作级别定义的任何额外参数与路径级参数一起使用：

```
paths:
  /users/{id}:
    parameters:
      - in: path
        name: id
        type: integer
        required: true
        description: The user ID.

    # GET/users/{id}?metadata=true
    get:
      summary: Gets a user by ID.
      # Note we only define the query parameter, because the {id} is defined at the path level.
      parameters:
        - in: query
          name: metadata
          type: boolean
          required: false
          description: If true, the endpoint returns only the user metadata.
      responses:
        200:
          description: OK
```

可以在操作级别覆盖特定路径级别的参数，但无法将其删除。

```
paths:
  /users/{id}:
    parameters:
      - in: path
        name: id
        type: integer
        required: true
        description: The user ID.

    # DELETE /users/{id} - uses a single ID.
    # Reuses the {id} parameter definition from the path level.
    delete:
      summary: Deletes the user with the specified ID.
      responses:
        204:
          description: User was deleted.

    # GET /users/id1,id2,id3 - uses one or more user IDs.
    # Overrides the path-level {id} parameter.
    get:
      summary: Gets one or more users by ID.
      parameters:
        - in: path
          name: id
          required: true
          description: A comma-separated list of user IDs.
          type: array
          items:
            type: integer
          collectionFormat: csv
          minItems: 1
      responses:
        200:
          description: OK
```

#### 不同路径中的公共参数

不同的API路径可能具有一些常见参数，例如分页参数。可以在全局`parameters`部分定义公共参数，并通过`$ref`在各个操作中引用它们。

```
parameters:
  offsetParam:  # <-- Arbitrary name for the definition that will be used to refer to it.
                # Not necessarily the same as the parameter name.
    in: query
    name: offset
    required: false
    type: integer
    minimum: 0
    description: The number of items to skip before starting to collect the result set.
  limitParam:
    in: query
    name: limit
    required: false
    type: integer
    minimum: 1
    maximum: 50
    default: 20
    description: The numbers of items to return.
paths:
  /users:
    get:
      summary: Gets a list of users.
      parameters:
        - $ref: '#/parameters/offsetParam'
        - $ref: '#/parameters/limitParam'
      responses:
        200:
          description: OK
  /teams:
    get:
      summary: Gets a list of teams.
      parameters:
        - $ref: '#/parameters/offsetParam'
        - $ref: '#/parameters/limitParam'
      responses:
        200:
          description: OK
```

请注意，全局`parameters`不适用于所有操作的参数 - 它们只是可以轻松重用的全局定义。

### 参数依赖关系

Swagger不支持参数依赖和互斥参数。在https://github.com/OAI/OpenAPI-Specification/issues/256](https://github.com/OAI/OpenAPI-Specification/issues/256)上有一个开放的功能要求。

可以做的是记录参数描述中的限制，并定义400错误请求响应中的逻辑。

例如，考虑接受相对日期范围（`rdate`）或确切范围（`start_date` +`end_date`）的`/report`端点：

```
GET /report?rdate=Today
GET /report?start_date=2016-11-15&end_date=2016-11-20
```

可以如下描述此端点：

```
paths:
  /report:
    get:
      parameters:
        - name: rdate
          in: query
          type: string
          description: >
             A relative date range for the report, such as `Today` or `LastWeek`.
             For an exact range, use `start_date` and `end_date` instead.
        - name: start_date
          in: query
          type: string
          format: date
          description: >
            The start date for the report. Must be used together with `end_date`.
            This parameter is incompatible with `rdate`.
        - name: end_date
          in: query
          type: string
          format: date
          description: >
            The end date for the report. Must be used together with `start_date`.
            This parameter is incompatible with `rdate`.
      responses:
        400:
          description: Either `rdate` or `start_date`+`end_date` are required.
```

### FAQ

**什么时候应该使用“type”与“schema”？**

模式仅与`in: body`参数一起使用。任何其他参数都需要一个原始类型，例如`type: string`或'array'的基元。

**可以将对象作为查询参数吗？**

否。查询参数仅支持基元类型和数组。