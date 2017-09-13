## MIME类型

API可以接受和返回不同格式的数据，最常见的是JSON和XML。可以使用`consumes`和`produces`关键字来指定API理解的MIME类型。 `consumes`和`produces`的值是一个MIME类型的数组。

全局MIME类型可以在API规范的根级别定义，并由所有API操作继承。在这里，API使用JSON和XML：

```
consumes:
  - application/json
  - application/xml
produces:
  - application/json
  - application/xml
```

请注意，`consumes`仅影响具有请求体的操作，例如POST，PUT和PATCH。对于诸如GET的无身份操作，它被忽略。

当在操作级别使用时，`consumes`和`produces`覆盖（不扩展）全局定义。在下面的例子中，`GET /logo`操作重新定义`produce`数组以返回一个图像：

```
paths:
  /logo:
    get:
      summary: Returns the logo image
      produces:
        - image/png
        - image/gif
        - image/jpeg
      responses:
        200:
          description: OK
          schema:
            type: file
```

`consumes`和`produces`中列出的MIME类型应符合[RFC 6838](http://tools.ietf.org/html/rfc6838)。例如，可以使用标准MIME类型，例如：

```
application/json
application/xml
application/x-www-form-urlencoded
multipart/form-data
text/plain; charset=utf-8
text/html
application/pdf
image/png
```

以及供应商特定的MIME类型（由`vnd.`表示））：

```
application/vnd.mycompany.myapp.v2+json
application/vnd.ms-excel
application/vnd.openstreetmap.data+xml
application/vnd.github-issue.text+json
application/vnd.github.v3.diff
image/vnd.djvu
```