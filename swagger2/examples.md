## 添加示例

API规范可以包括以下示例：

* 响应MIME类型，

* 架构（数据模型），

* 架构中的个别属性。

工具和库可以使用本示例，例如，Swagger UI基于输入模式示例自动填充请求体，并且一些API模拟工具使用示例来生成模拟响应。

**注意：** 不要将示例值与`default`值混淆。一个例子用来说明这个值应该是什么样的。默认值是服务器使用的值，如果请求中没有提供此值。

### 架构示例

`example`键用于提供架构示例。可以给出个别属性，对象和整个模式的示例。

#### 属性示例

属性示例可以内联指定。示例值必须符合属性类型。

```
definitions:
  CatalogItem:
    id:
      type: integer
      example: 38
    title:
      type: string
      example: T-shirt
    required:
      - id
      - title
```

请注意，不支持每个属性或模式的多个值，也就是说，不能是：

```
title:
  type: string
  example: T-shirt
  example: Phone
```

#### 对象示例

类型对象的属性可以包含复杂的内联示例，其中包含该对象的属性。该示例应该符合对象模式。

```
definitions:
  CatalogItem:
    id:
      type: integer
      example: 38
    title:
      type: string
      example: T-shirt
    image:
      type: object
      properties:
        url:
          type: string
        width:
          type: integer
        height:
          type: integer
      required:
        - url
      example:   # <-----
        url: images/38.png
        width: 100
        height: 100
    required:
      - id
      - title
```

#### 数组示例

原始类型的数组的例子：

```
definitions:
  ArrayOfStrings:
    type: array
    items:
      type: string
    example:
      - foo
      - bar
      - baz
```

类似地，一组对象将被指定为：

```
definitions:
  ArrayOfCatalogItems:
    type: array
    items:
      $ref: "#/definitions/CatalogItem"
    example:
      - id: 38
        title: T-shirt
      - id: 114
        title: Phone
```

#### 整个架构示例

可以为整个模式（包括所有嵌套模式）指定`example`，前提是示例符合模式。

```
definition:
  CatalogItem:
    type: object
    properties:
      id:
        type: integer
      name:
        type: string
      image:
        type: object
        properties:
          url:
            type: string
          width:
            type: integer
          height:
            type: integer
    required:
      - id
      - name
    example:   # <----------
      id: 38
      name: T-shirt
      image:
        url: images/38.png
        width: 100
        height: 100
```

### 响应示例

Swagger允许响应级别的示例，每个示例对应于操作返回的特定MIME类型。例如`application/json`的例子，`text/csv`的另一个例子等等。每个MIME类型必须是操作的`produce`值之一，不管是显示还是从全局范围继承。

```
      produces:
        - application/json
        - text/csv
      responses:
        200:
          description: OK
          examples:
            application/json: { "id:" 38, "title": "T-shirt" }
            text/csv: >
              id,title
              38,T-shirt
```

所有示例都是**自由表单**，这意味着他们的解释取决于工具和库。

#### JSON和YAML示例

由于JSON和YAML是可互换的（YAML是JSON的超集），所以可以使用JSON语法指定两者：

```
          examples:
            application/json:
              {
                "id": 38,
                "title": "T-shirt"
              }
```

或YAML语法：

```
          examples:
            application/json:
              id: 38
              title: T-shirt
              image:
                url: images/38.png
```

`$ref`可用于引用包含示例响应的外部.json或.yaml文件：

```
          examples:
            application/json:
               $ref: http://myapi.com/examples/catalogItem.json
```

#### XML示例

XML响应示例没有具体的语法。但是，由于响应示例是自由格式，可以使用任何希望的格式或工具支持的格式。

```
          examples:
            application/xml: "<users><user>Alice</user><user>Bob</user></users>"

          examples:
            application/xml:
              users:
                user:
                  - Alice
                  - Bob

          examples:
            application/xml:
              url: http://myapi.com/examples/users.xml
```

或者，在响应模式中指定示例值，[如上所述](#架构示例)。

#### 文本示例

由于所有响应示例都是自由格式的，因此可以使用工具或库支持的任何格式。例如，像：

```
          examples:
            text/html: "<html><body><p>Hello, world!</p></body></html>"
            text/plain: Hello, world!
```

另请参阅[Stack Overflow上的这篇文章](http://stackoverflow.com/questions/3790454/in-yaml-how-do-i-break-a-string-over-multiple-lines)，了解如何编写提示YAML中的多行字符串。

#### 示例优先级

如果在不同级别（属性，架构，响应）上有多个示例，则正在处理规范的工具将使用更高级别的示例。优先顺序是：

1. 回应的例子
2. 架构示例
3. 对象和数组属性的例子
4. 原子属性示例和数组项示例