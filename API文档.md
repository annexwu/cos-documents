#### 功能说明

请求发往用户配置的 API 网关，并由 API 网关将请求转发给用户配置的后端 SCF 云函数服务，云函数解析请求正文中的 JSON 参数，将指定 url 中的内容加入 ZIP 包并上传到指定存储桶中。

>- 将**请求示例**中的 `<api-gateway-domain>` 和 `<api-gateway-pathname>` 替换为用户配置的 API 网关域名和 API 路径
>- 请求 API 网关时，需要进行**密钥对认证**，详情可参考 [密钥对认证](https://cloud.tencent.com/document/product/628/11819)，各语言的签名算法可参考 [多种语言生成签名](https://cloud.tencent.com/document/product/628/42189)，请将**请求示例**中的 `<source>`， `<datetime>`，`<authorization>` 替换为鉴权计算的值
>- 请求参数通过请求正文携带，格式为 JSON

#### 使用示例
[//]: # (.cssg-snippet-transfer-upload-file)
```
POST /<api-gateway-pathname> HTTP/1.1
Host: <api-gateway-domain>
Date: GMT Date
Content-Type: application/json
Content-Length: Content Length
Source: <source>
X-Date: <datetime>
Authorization: <authorization>

[Request JSON Body]
```

#### 请求参数示例

```
{
    bucket: 'bucket-1250000000',
    region: 'ap-guangzhou',
    key: 'mypack.zip',
    flatten: false,
    /**
     * sourceList 和 sourceConfigList 参数仅需指定一种即可
     */
    sourceList: [
        {
            url: 'https://bucket-1250000000.cos.ap-guangzhou.myqcloud.com/file1.jpg',
            renamePath: '重命名_file1.jpg',
        },
        {
            url: 'https://bucket-1250000000.cos.ap-guangzhou.myqcloud.com/file2.mp4',
            renamePath: '新路径/重命名_file2.mp4',
        },
        {
            url: 'https://bucket-1250000000.cos.ap-guangzhou.myqcloud.com/file3.txt',
        }
    ],
    sourceConfigList: [
        {
             url: 'https://bucket-1250000000.cos.ap-guangzhou.myqcloud.com/sourceList.json',
        }
    ],
}
```

#### 参数说明

| 参数名                                 | 参数描述                                                     | 类型      | 是否必填 |
| ------------------------------------------------------------ | ------------------------------------------------------------ | --------- | ---- |
| bucket                                                       | ZIP 包投递的存储桶，命名格式为 BucketName-APPID，此处填写的存储桶名称必须为此格式，例如：bucket-1250000000 | String    | 是   |
| region                                                       | ZIP 包投递的存储桶所在地域，枚举值请参见 [地域和访问域名](https://cloud.tencent.com/document/product/436/6224) | String    | 是   |
| key                                                          | ZIP 包的名称（Object 的名称），对象在存储桶中的唯一标识，详情请参见 [对象概述](https://cloud.tencent.com/document/product/436/13324) | String    | 是   |
| flatten                                                         | 是否需要路径扁平化（去除源目录结构），例如：https://domain/source/test.mp4 的源文件路径为 source/test.mp4，如果为 true，则ZIP包中该文件路径为 test.mp4，否则为 source/test.mp4。默认为 false          | Boolean | 否   |
| sourceList                                                    | 源文件列表，**sourceList 和 sourceConfigList 不能同时为空**                                                     | Array    | 是   |
| sourceList[].url                                                   | 源文件的 URL                                                 | String    | 是   |
| sourceList[].renamePath                                                 | 带路径的重命名，该路径就是在 ZIP 包中的文件路径，例如将 1.mp4 重命名为 source/test.mp4       | String    | 否   |
| sourceConfigList                                                  | sourceList 的配置文件列表，如果您不希望在请求时携带整个 sourceList，可以将 sourceList 参数 JSON 字符串处理，生成 json 配置文件，上传到 COS，并在 sourceConfigList 中指定该文件的 url，支持指定多个配置文件，**sourceList 和 sourceConfigList 不能同时为空** | Array  | 是   |
| sourceConfigList[].url                                                  | sourceList 配置文件的 url | String  | 是   |

#### 响应正文
```
{
  code: 0,
  data: {
    Bucket: "mybucket-1250000000",
    ETag: "\"35bb5e5f050e22bed8f443d8da5dbfb8-1\"",
    Key: "mypack.zip",
    Location: "mybucket-1250000000.cos.ap-guangzhou.myqcloud.com/mypack.zip"
  },
  error: null,
  message: "cos zip file success"
}
```

| 参数名       | 参数描述                                                     | 类型   |
| ------------ | ------------------------------------------------------------ | ------ |
| code          | 业务错误码，如果为 0 则说明执行成功，否则为执行失败 | Number |
| message | 执行结果的文字说明                  | String |
| data | 执行成功的信息，包含 ZIP 包的 url                  | Object |
| error    | 执行的错误信息，如执行成功则为 null                                           | Object or String |