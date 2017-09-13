## 什么是Swagger？

Swagger用来描述API的结构，以便机器可以读取它们。

API描述自己的结构的能力是Swagger的出发点。通过阅读API的结构，就可以自动构建美观并具有交互式的API文档。还可以为API以多种语言自动生成客户端库，以及自动测试等。

Swagger通过要求API返回包含整个API的详细自描述的YAML或JSON。该文件本质上是API的资源列表，它遵守[OpenAPI Specification](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md)。Swagger规范要求API自描述文档包括以下信息：

* API支持的所有操作是什么？

* API参数是什么，它返回什么？

* API是否需要一些授权？

* 其它有关内容，如术语，联系信息和使用API​​的许可证。

借助Swagger规范可以手动为API编写接口规范，或者从源代码中的注释自动生成。迁出[swagger.io/open-source-integrations](https://swagger.io/open-source-integrations/)以获取可以从代码生成Swagger的工具列表。

### 所以，要API制订了一个Swagger规范，接下来该怎么办？

Swagger有几种方法可以进一步推动API开发：

* 设计优先：使用[Swagger Codegen](https://swagger.io/swagger-codegen/)为API生成**服务器存根**。唯一剩下的就是实现服务器逻辑 - 因为API已经准备好了！

* 使用[Swagger Codegen](https://swagger.io/swagger-codegen/)以超过40种语言为API生成**客户端库**。

* 使用[Swagger UI](https://swagger.io/swagger-ui/)生成**交互式API文档**，让用户直接在浏览器中尝试API调用。

* 使用规范将API相关工具连接到API。例如，将规范导入[SoapUI](https://soapui.org/)或[PostMan](https://www.getpostman.com/)，为API创建自动测试。

* 以及更多！查看与Swagger集成的[开源](https://swagger.io/open-source-integrations/)和[商业工具](https://swagger.io/commercial-tools/)。