## Swagger扩展

扩展或供应商扩展是以`x-`开头的自定义属性，例如`x-logo`。它们可用于描述标准Swagger规范未涵盖的额外功能。支持Swagger的许多API相关产品使用扩展来记录自己的属性，如Amazon API Gateway，ReDoc，APIMatic等。

在API规范的根级别和以下位置支持扩展：

* `info`部分
* `paths`部分，单独的路径和操作
* 操作参数
* `responses`答复
* `tags`标签
* 安全架构

扩展值可以是原语，数组，对象或`null`。如果该值是对象或数组的对象，则该对象的属性名称不需要以`x-`开头。

### 示例

使用Amazon API Gateway自定义授权器的API将包括类似于以下的扩展：

```
securityDefinitions:
  APIGatewayAuthorizer:
    type: apiKey
    name: Authorization
    in: header
    x-amazon-apigateway-authtype: oauth2
    x-amazon-apigateway-authorizer:
      type: token
      authorizerUri: arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/arn:aws:lambda:us-east-1:account-id:function:function-name/invocations
      authorizerCredentials: arn:aws:iam::account-id:role
      identityValidationExpression: "^x-[a-z]+"
      authorizerResultTtlInSeconds: 60
```