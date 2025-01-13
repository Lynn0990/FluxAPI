# Flux Images Generation API 对接说明

本文将介绍一种 Flux Images Generation API 对接说明，它是可以通过输入自定义参数来生成Flux官方的图片。

接下来介绍下 Flux Images Generation API 的对接说明。

## 申请流程

要使用 API，需要先到 [Flux Images Generation API](https://surl.id/1uKeB10xrh) 对应页面申请对应的服务，进入页面之后，点击「Acquire」按钮，如图所示：

![](https://cdn.acedata.cloud/q6ytrc.png)

如果你尚未登录或注册，会自动跳转到登录页面邀请您来注册和登录，登录注册之后会自动返回当前页面。

在首次申请时会有免费额度赠送，可以免费使用该 API。

## 基本使用

首先先了解下基本的使用方式，就是输入提示词 `prompt`、 生成行为 `action`、图片尺寸 `size`，便可获得处理后的结果，首先需要简单地传递一个 `action` 字段，它的值为 `generate`，然后我们还需要输入提示词，具体的内容如下：

<p><img src="https://cdn.acedata.cloud/svluah.png" width="500" class="m-auto"></p>

可以看到这里我们设置了 Request Headers，包括：

- `accept`：想要接收怎样格式的响应结果，这里填写为 `application/json`，即 JSON 格式。
- `authorization`：调用 API 的密钥，申请之后可以直接下拉选择。

另外设置了 Request Body，包括：

- `action`：此次图片生成任务的行为。
- `size`：图片生成结果的尺寸大小。
- `prompt`：提示词。
- `callback_url`：需要回调结果的URL。

选择之后，可以发现右侧也生成了对应代码，如图所示：

<p><img src="https://cdn.acedata.cloud/uil6f8.png" width="500" class="m-auto"></p>

点击「Try」按钮即可进行测试，如上图所示，这里我们就得到了如下结果：

```json
{
  "success": true,
  "task_id": "d717a498-7efe-4926-a7ec-0eadb958bb7a",
  "trace_id": "ff7f7b65-2a77-416d-94d7-048f1193ba2d",
  "data": [
    {
      "prompt": "a white siamese cat",
      "image_url": "https://sf-maas-uat-prod.oss-cn-shanghai.aliyuncs.com/outputs/438978f3-d14d-41f7-9a3a-8f017820ca2b_0.png",
      "seed": 1782833889,
      "timings": {
        "inference": 3.179
      }
    }
  ]
}
```

返回结果一共有多个字段，介绍如下：

- `success`，此时视频生成任务的状态情况。
- `task_id`，此时视频生成任务ID。
- `trace_id`，此时视频生成跟踪ID。
- `data`，此时图像生成任务的结果列表。
	- `image_url`，此时图片生成任务的链接。
	- `prompt`，提示词。
	

可以看到我们得到了满意的图片信息，我们只需要根据结果中 `data` 的图片链接地址获取生成的Flux图片即可。

另外如果想生成对应的对接代码，可以直接复制生成，例如 CURL 的代码如下：

```shell
curl -X POST 'https://api.acedata.cloud/flux/images' \
-H 'accept: application/json' \
-H 'authorization: Bearer da33638ce63942d7bce97c09cd710280' \
-H 'content-type: application/json' \
-d '{
  "action": "generate",
  "size": "1024x1024",
  "prompt": "a white siamese cat"
}'
```

## 异步回调

由于 Flux Images Generation API 生成的时间相对较长，大约需要 1-2 分钟，如果 API 长时间无响应，HTTP 请求会一直保持连接，导致额外的系统资源消耗，所以本 API 也提供了异步回调的支持。

整体流程是：客户端发起请求的时候，额外指定一个 `callback_url` 字段，客户端发起 API 请求之后，API 会立马返回一个结果，包含一个 `task_id` 的字段信息，代表当前的任务 ID。当任务完成之后，生成图片的结果会通过 POST JSON 的形式发送到客户端指定的 `callback_url`，其中也包括了 `task_id` 字段，这样任务结果就可以通过 ID 关联起来了。

下面我们通过示例来了解下具体怎样操作。

首先，Webhook 回调是一个可以接收 HTTP 请求的服务，开发者应该替换为自己搭建的 HTTP 服务器的 URL。此处为了方便演示，使用一个公开的 Webhook 样例网站 https://webhook.site/，打开该网站即可得到一个 Webhook URL，如图所示：

![](https://cdn.acedata.cloud/cjjfly.png)

将此 URL 复制下来，就可以作为 Webhook 来使用，此处的样例为 `https://webhook.site/3d32690d-6780-4187-a65c-870061e8c8ab`。

接下来，我们可以设置字段 `callback_url` 为上述 Webhook URL，同时填入相应的参数，具体的内容如图所示：

<p><img src="https://cdn.acedata.cloud/wm6caw.png" width="500" class="m-auto"></p>

点击运行，可以发现会立即得到一个结果，如下：

```
{
  "task_id": "6a97bf49-df50-4129-9e46-119aa9fca73c"
}
```

稍等片刻，我们可以在 `https://webhook.site/3d32690d-6780-4187-a65c-870061e8c8ab` 上观察到生成图片的结果，如图所示：

![](https://cdn.acedata.cloud/v23lot.png)

内容如下：

```json
{
    "success": true,
    "task_id": "6a97bf49-df50-4129-9e46-119aa9fca73c",
    "trace_id": "9b4b1ff3-90f2-470f-b082-1061ec2948cc",
    "data": [
        {
            "prompt": "a white siamese cat",
            "image_url": "https://sf-maas-uat-prod.oss-cn-shanghai.aliyuncs.com/outputs/f4f8d407-377a-408a-82d0-427a5a836f09_0.png",
            "seed": 1698551532,
            "timings": {
                "inference": 3.328
            }
        }
    ]
}
```

可以看到结果中有一个 `task_id` 字段，其他的字段都和上文类似，通过该字段即可实现任务的关联。


## 错误处理

在调用 API 时，如果遇到错误，API 会返回相应的错误代码和信息。例如：

- `400 token_mismatched`：Bad request, possibly due to missing or invalid parameters.
- `400 api_not_implemented`：Bad request, possibly due to missing or invalid parameters.
- `401 invalid_token`：Unauthorized, invalid or missing authorization token.
- `429 too_many_requests`：Too many requests, you have exceeded the rate limit.
- `500 api_error`：Internal server error, something went wrong on the server.

### 错误响应示例

```json
{
  "success": false,
  "error": {
    "code": "api_error",
    "message": "fetch failed"
  },
  "trace_id": "2cf86e86-22a4-46e1-ac2f-032c0f2a4e89"
}
```

## 结论

通过本文档，您已经了解了如何使用 Flux Images Generation API 可通过输入提示词来生成图片。希望本文档能帮助您更好地对接和使用该 API。如有任何问题，请随时联系我们的技术支持团队。