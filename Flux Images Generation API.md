# Flux Images Generation API Integration Instructions

This document introduces the integration instructions for the Flux Images Generation API, which allows users to generate images from Flux by inputting custom parameters.

Next, let's discuss the integration instructions for the Flux Images Generation API.

## Application Process

To use the API, you first need to go to the corresponding page of the [Flux Images Generation API](https://surl.id/1uKeB10fzz) to apply for the service. After entering the page, click the "Acquire" button, as shown in the image:

![](https://cdn.acedata.cloud/q6ytrc.png)

If you are not logged in or registered, you will be automatically redirected to the login page inviting you to register and log in. After logging in or registering, you will automatically return to the current page.

In the first application, there will be complimentary credits, allowing you to use the API for free.

## Basic Usage

First, understand the basic usage method, which is to input the prompt `prompt`, the action `action`, and the image size `size`, to obtain the processed result. First, you need to simply pass an `action` field, with its value being `generate`, and then we also need to input the prompt; the specifics are as follows:

<p><img src="https://cdn.acedata.cloud/svluah.png" width="500" class="m-auto"></p>

Here, we can see that we have set the Request Headers, which include:

- `accept`: The desired format for the response, here it is filled as `application/json`, i.e., JSON format.
- `authorization`: The key for calling the API, selectable directly after application.

Additionally, we have set the Request Body, which includes:

- `action`: The behavior of this image generation task.
- `size`: The dimensions of the generated image result.
- `prompt`: The prompt.
- `callback_url`: The URL for the callback results.

After selection, you will find that the corresponding code has been generated on the right side, as shown in the image:

<p><img src="https://cdn.acedata.cloud/uil6f8.png" width="500" class="m-auto"></p>

By clicking the "Try" button, you can perform the test, as shown in the above image, here we received the following result:

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

The returned result contains multiple fields, described as follows:

- `success`: The status of the video generation task.
- `task_id`: The ID of the video generation task.
- `trace_id`: The trace ID of the video generation.
- `data`: The result list of the image generation task.
  - `image_url`: The link for the image generation task.
  - `prompt`: The prompt.

We can see that we have received satisfactory image information, and we only need to obtain the generated Flux image from the image link address in `data`.

Additionally, if you want to generate the corresponding integration code, you can directly copy the generated code, for example, the CURL code is as follows:

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

## Asynchronous Callback

Since the generation time for the Flux Images Generation API is relatively long, usually taking 1-2 minutes, if the API does not respond for a long time, the HTTP request will keep the connection open, leading to extra system resource consumption. Therefore, this API also provides support for asynchronous callbacks.

The overall process is: when the client initiates a request, it additionally specifies a `callback_url` field. After the client initiates the API request, the API will immediately return a result containing a `task_id` field, representing the current task ID. When the task is completed, the result of the generated image will be sent to the client's specified `callback_url` in the form of POST JSON, which will also include the `task_id` field, allowing the task result to be associated using the ID.

Let’s understand the specific operations through an example.

First, the Webhook callback is a service that can receive HTTP requests; developers should replace it with the URL of the HTTP server they set up. For demonstration purposes, a public Webhook sample site https://webhook.site/ is used, where you can obtain a Webhook URL, as shown in the image:

![](https://cdn.acedata.cloud/cjjfly.png)

Copy this URL, and it can be used as the Webhook. The sample here is `https://webhook.site/3d32690d-6780-4187-a65c-870061e8c8ab`.

Next, we can set the `callback_url` field to the above Webhook URL, while filling in the corresponding parameters. The specific content is shown in the image:

<p><img src="https://cdn.acedata.cloud/wm6caw.png" width="500" class="m-auto"></p>

Click on run, and you will immediately receive a result, as follows:

```
{
  "task_id": "6a97bf49-df50-4129-9e46-119aa9fca73c"
}
```

After a short wait, we can observe the generated image result at `https://webhook.site/3d32690d-6780-4187-a65c-870061e8c8ab`, as shown in the image:

![](https://cdn.acedata.cloud/v23lot.png)

The content is as follows:

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

We can see that the result contains a `task_id` field, and the other fields are similar to the previous ones, allowing task association through this field.

## Error Handling

When calling the API, if errors occur, the API will return the corresponding error codes and messages. For example:

- `400 token_mismatched`: Bad request, possibly due to missing or invalid parameters.
- `400 api_not_implemented`: Bad request, possibly due to missing or invalid parameters.
- `401 invalid_token`: Unauthorized, invalid or missing authorization token.
- `429 too_many_requests`: Too many requests, you have exceeded the rate limit.
- `500 api_error`: Internal server error, something went wrong on the server.

### Error Response Example
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

## Conclusion

Through this document, you have learned how to use the Flux Images Generation API to generate images by inputting prompt words. We hope this document helps you better connect and use the API. If you have any questions, please feel free to contact our technical support team.