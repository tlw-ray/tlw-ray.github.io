## 枚举

可以使用`enum`关键字来指定请求参数或模型属性的可能值。

例如，sort参数在：

```
GET /items?sort=[asc|desc]
```

可以被描述为：

```
paths:
  /items:
    get:
      parameters:
        - in: query
          name: sort
          description: 排序规则
          type: string
          enum: [asc, desc]
```

在YAML中，可以为每行指定一个枚举值：

```
          enum:
            - asc
            - desc
```

枚举中的所有值必须符合指定的类型。

如果需要指定枚举项目的描述，可以在参数或属性的`description`中执行此操作：

```
          type: string
          enum:
            - asc
            - desc
          description: >
            Sort order:
             * asc - Ascending, from A to Z.
             * desc - Descending, from Z to A.
```

### 可重用枚举

虽然Swagger 2.0没有对可重用枚举的内置支持，但是可以使用YAML锚点在YAML中定义它们 - 只要工具支持它们。

锚是YAML的一个方便的功能，可以使用`&anchor-name`标记一个键，然后进一步使用`*anchor-name`来引用该键的值。于是就可以轻松地通过YAML文件复制内容。

**注意：** 锚（`&`）必须在使用前定义。

```
definitions:
  Colors:
    type: string
    enum: &COLORS
      - black
      - white
      - red
      - green
      - blue
    # OR:
    # enum: &COLORS [black, white, red, green, blue]

paths:
  /products:
    get:
      parameters:
        - in: query
          name: color
          required: true
          type: string
          enum: *COLORS
      responses:
        200:
          description: OK
```

如果工具的YAML解析器支持YAML合并键（`<<`），可以使用此技巧来引用类型和枚举值。

```
definitions:
  Colors: &COLORS
    type: string
    enum: [black, white, red, green, blue]
paths:
  /products:
    get:
      parameters:
        - in: query
          name: color
          required: true
          <<: *COLORS
      responses:
        200:
          description: OK
```