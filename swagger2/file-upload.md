## 上传文件

Swagger 2.0支持使用`Content-Type: multipart/form-data`发送的文件上传。也就是说，API服务器必须为此操作消耗`multipart/form-data`：

```
consumes:
   - multipart/form-data
```

操作有效负载使用`formData`参数（而不是body参数）。 file参数必须具有`type: file`：

```
paths:
   /upload:
     post:
       summary: Uploads a file.
       consumes:
         - multipart/form-data
       parameters:
         - in: formData
           name: upfile
           type: file
           description: The file to upload.
```

此定义对应于以下HTTP请求：

```
POST /upload
Host: example.com
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryqzByvokjOTfF9UwD
Content-Length: 204

------WebKitFormBoundaryqzByvokjOTfF9UwD
Content-Disposition: form-data; name="upfile"; filename="example.txt"
Content-Type: text/plain

File contents go here.

------WebKitFormBoundaryqzByvokjOTfF9UwD--
```

Swagger UI使用文件输入控件显示文件参数，允许用户浏览本地文件进行上传。
![Swagger UI中的文件上传](../../images/docs/swagger-ui-file-upload.png)

### 上传文件+其他数据

文件参数可以与其他表单数据一起发送：

```
      parameters:
        - in: formData
          name: upfile
          type: file
          required: true
          description: The file to upload.
        - in: formData
          name: note
          type: string
          required: false
          description: Description of file contents.
```


相应的HTTP请求有效载荷将包括多个部分：

```
POST /upload
Host: example.com
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryqzByvokjOTfF9UwD
Content-Length: 332

------WebKitFormBoundaryqzByvokjOTfF9UwD
Content-Disposition: form-data; name="upfile"; filename="example.txt"
Content-Type: text/plain

File contents go here.

------WebKitFormBoundarysKk4Z8KcYfU3u6Cs
Content-Disposition: form-data; name="note"

Uploading a file named "example.txt"

------WebKitFormBoundaryqzByvokjOTfF9UwD--
```

### 多文件上传

可以有多个命名的文件参数，每个参数分别定义：

```
      parameters:
        - in: formData
          name: upfile1
          type: file
          required: true
        - in: formData
          name: upfile2
          type: file
          required: false
        - in: formData
          name: upfile3
          type: file
          required: false
```

但是，不支持上传任意数量的文件（一组文件）。在[https://github.com/OAI/OpenAPI-Specification/issues/254](https://github.com/OAI/OpenAPI-Specification/issues/254)上有一个开放的功能要求。现在，不过可以使用二进制字符串数组作为上传任意数量文件的解决方法：

```
type: array
items:
  type: string
  format: binary
```

请注意，这不会在Swagger UI中生成文件上传界面。

### FAQ

**可以通过PUT上传文件吗？**

Swagger支持使用`Content-Type：multipart/form-data`的文件上传请求，但不关心HTTP方法。可以使用POST，PUT或任何其他方法，只要该操作消耗`multipart/form-data`即可。

不支持有效载荷只是原始文件内容的上传，因为它们不是`multipart/form-data`。也就是说，Swagger不支持以下内容：

```
curl --upload-file archive.zip http://example.com/upload
```

请注意，Swagger UI中的文件上传仅适用于POST请求，因为浏览器中的HTML表单仅支持GET和POST方法。

**可以为上传的文件定义Content-Type吗？**

目前不支持。可以说一个操作接受一个文件，但不能说这个文件是一个特定的类型或结构。在https://github.com/OAI/OpenAPI-Specification/issues/222](https://github.com/OAI/OpenAPI-Specification/issues/222)上有一个开放的功能要求。

现在，[供应商扩展](swagger-extensions.md)可用于扩展此功能，例如：

```
- in: formData
  name: zipfile
  type: file
  description: Contents of the ZIP file.
  x-mimetype: application/zip
```