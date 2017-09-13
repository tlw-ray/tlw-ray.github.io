## 使用标签分组操作

可以为每个API操作分配一个`tags`列表。标记操作可以由工具和库处理不同。例如，Swagger UI使用`tags`对显示的操作进行分组。

```
paths:
  /pet/findByStatus:
    get:
      summary: Finds pets by Status
      tags:
        - pets
      ...
  /pet:
    post:
      summary: Adds a new pet to the store
      tags:
        - pets
      ...
  /store/inventory:
    get:
      summary: Returns pet inventories
      tags:
        - store
      ...
```

![Swagger UI中的标签分组](images/swagger-ui-tags.png)

或者，可以通过使用根级别上的全局`tags`部分为每个标签指定`description`和`externalDocs`。这里的标签名称应与操作中使用的标签名称匹配。

```
tags:
  - name: pets
    description: Everything about your Pets
    externalDocs:
      url: http://docs.my-api.com/pet-operations.htm
  - name: store
    description: Access to Petstore orders
    externalDocs:
      url: http://docs.my-api.com/store-orders.htm
```

全局标签部分中的标签顺序也控制Swagger UI中的默认排序。

请注意，即使在根级别上没有定义，也可以在操作中使用标签。