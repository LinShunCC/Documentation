
---
title: 云端录制 RESTful API
description: Cloud recording restful api reference
platform: All Platforms
updatedAt: Thu Sep 24 2020 07:53:33 GMT+0800 (CST)
---
# 云端录制 RESTful API
该文提供云端录制 RESTful API 的详细信息。

> 除本文外，你也可以查看我们全新的交互式 API 文档[云端录制 RESTful API](https://docs.agora.io/cn/cloud-recording/restfulapi)。该互动式文档包含各方法和参数的详细介绍，并提供 Try it out 功能，使你在文档页内即能进行 RESTful API 的调试。

![](https://web-cdn.agora.io/docs-files/1584924717496)

## <a name="auth"></a>认证

云端录制 RESTful API 仅支持 HTTPS 协议。使用 RESTful API 前，你需要通过 [HTTP 基本认证](https://docs.agora.io/cn/faq/restful_authentication)。

## 数据格式

所有的请求都发送给域名：`api.agora.io`。

- 请求：请求的格式详见下面各个 API 中的示例
- 响应：响应内容的格式为 JSON

> 所有的请求 URL 和请求包体内容都是区分大小写的。

## 调用步骤

一般进行云端录制的步骤如下：

1. 通过 [`acquire`](#acquire) 请求获取一个 resource ID 用于开启云端录制。
2. 获取 resource ID 后在 5 分钟内调用 [`start`](#start) 开始云端录制。
3. 录制完成后调用 [`stop`](#stop) 停止录制。

在整个过程中可以通过 [`query`](#query) 请求查询云端录制的状态。完整示例代码请参考[云端录制快速开始示例代码](../../cn/cloud-recording/cloud_recording_rest.md)。

## <a name="acquire"></a>获取云端录制资源的 API

在开始云端录制之前，你需要调用该方法获取一个 resource ID。

>  一个 resource ID 只能用于一次云端录制服务。

- 方法：POST
- 接入点：/v1/apps/\<appid\>/cloud_recording/acquire

> 每个 App ID 每秒钟的请求数限制为 10 次。如需提高此限制，请[提交工单](https://agora-ticket.agora.io/)联系技术支持。

调用该方法成功后，你可以从 HTTP 响应包体中的 `resourceId` 字段得到一个 resource ID。这个 resource ID 的时效为 5 分钟，你需要在 5 分钟内用这个 resource ID 去调用[开始云端录制的 API](#start)。

### 参数

该 API 需要在 URL 中传入以下参数。

| 参数    | 类型   | 描述                                                         |
| :------ | :----- | :----------------------------------------------------------- |
| `appid` | String | 你的项目使用的 [App ID](https://docs.agora.io/cn/Agora%20Platform/terms?platform=All%20Platforms#appid)。必须使用和待录制的频道相同的 App ID。 |

该 API 需要在请求包体中传入以下参数。

| 参数            | 类型   | 描述                                                         |
| :-------------- | :----- | :----------------------------------------------------------- |
| `cname`         | String | 待录制的频道名。                                             |
| `uid`           | String | 字符串内容为云端录制服务在频道内使用的 UID，用于标识该录制服务，例如`"527841"`。需满足以下条件：<li>取值范围 1 到 (2<sup>32</sup>-1)，不可设置为 0。</li><li><b><em>不能与当前频道内的任何 UID 重复。</em></b></li><li>云端录制不支持 String 用户 ID（User Account），请确保该字段引号内为整型 UID，且频道内所有用户均使用整型 UID。</li> |
| `clientRequest` | JSON   | 客户请求参数，包含 `resourceExpiredHour` 字段。`resourceExpiredHour` 为 Number 类型，单位为小时，用于设置云端录制 RESTful API 的调用时效，从成功开启云端录制并获得 `sid` （录制 ID）后开始计算。超时后，你将无法调用 `query`，`updateLayout`，和 `stop` 方法。`resourceExpiredHour` 需大于等于 `1`， 且小于等于 `720`，默认值为 `72`。 |

### `acquire` 请求示例

- 请求 URL：

```
https://api.agora.io/v1/apps/<yourappid>/cloud_recording/acquire
```

- `Content-type` 为 `application/json;charset=utf-8`
- `Authorization` 为 Basic authorization，生成方法请参考 [RESTful API 认证](https://docs.agora.io/cn/faq/restful_authentication)。
- 请求包体内容：

 ```json
{
    "cname": "httpClient463224",
    "uid": "527841",
    "clientRequest":{
      "resourceExpiredHour":  24
   }
}
 ```

### `acquire` 响应示例

```json
"Code": 200,
"Body":
{
 "resourceId": "JyvK8nXHuV1BE64GDkAaBGEscvtHW7v8BrQoRPCHxmeVxwY22-x-kv4GdPcjZeMzoCBUCOr9q-k6wBWMC7SaAkZ_4nO3JLqYwM1bL1n6wKnnD9EC9waxJboci9KUz2WZ4YJrmcJmA7xWkzs_L3AnNwdtcI1kr_u1cWFmi9BWAWAlNd7S7gfoGuH0tGi6CNaOomvr7-ILjPXdCYwgty1hwT6tbAuaW1eqR0kOYTO0Z1SobpBxu1czSFh1GbzGvTZG"
}
```

- `code`: Number 类型，[响应状态码](#status)。
- `resourceId`: String 类型，云端录制资源 resource ID，使用这个 resource ID 可以开始一段云端录制。这个 resource ID 的有效期为 5 分钟，超时需要重新请求。

## <a name="start"></a>开始云端录制的 API

获取 resource ID 后，调用该方法开始云端录制。当你成功调用 `start` 方法后，云端录制会执行如下操作：


1. 根据你的[订阅](https://docs.agora.io/cn/Agora%20Platform/terms?platform=All%20Platforms#sub)设置（`recordingConfig`）订阅频道中的媒体流。
2. 对订阅的媒体流进行处理，如录制成指定的文件格式、截图、或上传至扩展服务。
3. 如果你设置了[第三方云存储](#storageConfig)（`storageConfig`），则云端录制还会将录制文件或截图上传到第三方云存储。

该 API 的请求方法和接入点为：

- 方法：POST
- 接入点：/v1/apps/\<appid\>/cloud_recording/resourceid/\<resourceid\>/mode/\<mode\>/start

> 每个 App ID 每秒钟的请求数限制为 10 次。如需提高此限制，请[提交工单](https://agora-ticket.agora.io/)联系技术支持。


### 参数

该 API 需要在 URL 中传入以下参数。

| 参数         | 类型   | 描述                                                         |
| :----------- | :----- | :----------------------------------------------------------- |
| `appid`      | String | 你的项目使用的 App ID。必须使用和待录制的频道相同的 App ID。 |
| `resourceid` | String | 通过 [`acquire`](#acquire) 请求获取的 resource ID。          |
| `mode`       | String | 录制模式，支持单流模式 `individual` 和合流模式 `mix` （默认模式）。单流模式下，分开录制频道内每个 UID 的音频流和视频流，每个 UID 均有其对应的音频文件和视频文件。合流模式下，频道内所有 UID 的音视频混合录制为一个音视频文件。 |

该 API 需要在请求包体中传入以下参数。

| 参数            | 类型   | 描述                                                         |
| :-------------- | :----- | :----------------------------------------------------------- |
| `cname`         | String | 待录制的频道名。                                             |
| `uid`           | String | 字符串内容为云端录制服务在频道内使用的 UID，用于标识该录制服务，需要和你在 [`acquire`](#acquire) 请求中输入的 UID 相同。 |
| `clientRequest` | JSON   | 特定的客户请求参数，对于该请求包含以下参数：<li>`token`：（选填） String 类型，用于鉴权的[动态密钥](https://docs.agora.io/cn/Agora%20Platform/terms?platform=All%20Platforms#token)。如果你的项目已启用 App 证书，则务必在该参数中传入你项目的动态秘钥。详见[校验用户权限](https://docs.agora.io/cn/cloud-recording/token?platform=All%20Platforms)。</li><li>[`recordingConfig`](#recordingConfig)：JSON 类型，订阅的详细设置。</li><li>[`recordingFileConfig`](#recordingFileConfig)：（选填）JSON 类型，录制文件的详细设置。</li><li>[`snapshotConfig`](#snapshotConfig)：（选填）JSON 类型，截图的详细设置。</li><li>[`storageConfig`](#storageConfig)：（选填）JSON 类型，第三方云存储的设置。</li><li>[`extensionServiceConfig`](#extensionServiceConfig)：（选填）JSON 类型，扩展服务的设置，目前仅支持阿里视频点播服务。</li> |

`clientRequest` 中各参数的关系及限制：

- 无论何种情况，都需要进行流订阅的设置。
- 音视频录制文件和截图只能择一设置。
- 每一种扩展服务都有特定的兼容性要求，详情可见下表：

| 参数                                                | 配置内容                                 | 互斥关系                                                    | 录制模式 |
| :-------------------------------------------------- | :--------------------------------------- | :---------------------------------------------------------- | :------- |
| [`recordingConfig`](#recordingConfig)               | 流订阅                                   | 无                                                          | 不限     |
| [`recordingFileConfig`](#recordingFileConfig)       | 音视频录制文件                           | 不可与 `snapshotConfig` 同时设置。                          | 不限     |
| [`snapshotConfig`](#snapshotConfig)                 | 截图文件                                 | 不可与 `recordingFileConfig` 或阿里视频点播服务同时设置。 | 仅限单流 |
| [`extensionServiceConfig`](#extensionServiceConfig) | 阿里视频点播服务（`aliyun_vod_service`） | 不可与 `snapshotConfig` 或 `storageConfig` 同时设置。     | 仅限合流 |
| [`storageConfig`](#storageConfig)                   | 第三方云存储                             | 不可与阿里视频点播服务同时设置。                            | 不限     |

#### <a name="recordingConfig"></a>**录制设置**

`recordingConfig` 是一个用于设置媒体流订阅的 JSON Object。云端录制会根据此设置订阅频道内的媒体流，并生成录制文件或截图。`recordingConfig` 包含以下字段：

- `channelType`：Number 类型，频道场景。频道场景必须与 Agora RTC Native/Web SDK 的设置一致，否则可能导致问题。
  - `0`：通信场景（默认）
  - `1`：直播场景
- `streamTypes`：（选填）Number 类型，订阅的媒体流类型。
  - `0`：仅订阅音频
  - `1`：仅订阅视频
  - `2`：（默认）订阅音频和视频
- `decryptionMode`：（选填）Number 类型，解密方案。如果频道设置了加密，该参数必须设置。解密方式必须与频道设置的加密方式一致。
  - `0`：无（默认）
  - `1`：设置 AES128XTS 解密方案
  - `2`：设置 AES128ECB 解密方案
  - `3`：设置 AES256XTS 解密方案
- `secret`：（选填）String 类型。启用解密模式后，设置的解密密码。
- `audioProfile`：（选填）设置输出音频的采样率、码率、编码模式和声道数。目前单流模式下不能设置该参数。
  - `0`：（默认）48 kHz 采样率，音乐编码，单声道，编码码率约 48 Kbps
  - `1`：48 kHz 采样率，音乐编码，单声道，编码码率约 128 Kbps
  - `2`：48 kHz 采样率，音乐编码，双声道，编码码率约 192 Kbps
- `videoStreamType`：（选填）Number 类型，设置订阅的视频流类型。如果频道中有用户开启了双流模式，你可以选择订阅视频大流或者小流。
  - `0`：视频大流（默认），即高分辨率高码率的视频流
  - `1`：视频小流，即低分辨率低码率的视频流
- `maxIdleTime`：（选填）Number 类型，最长空闲频道时间。默认值为 30 秒，该值需大于等于 5，且小于等于 (2<sup>32</sup>-1)。如果频道内无用户的状态持续超过该时间，录制程序会自动退出。退出后，再次调用 `start` 请求，会产生新的录制文件。

<div class="alert note"><ul><li>通信场景下，如果频道内有用户，但用户没有发流，不算作无用户状态。</li><li>直播场景下，如果频道内有观众但无主播，一旦无主播的状态超过 <code>maxIdleTime</code>，录制程序会自动退出。</li></div>

- `transcodingConfig`：（选填）JSON 类型，视频转码的详细设置。仅适用于合流模式，单流模式下不能设置该参数。如果不设置将使用默认值。如果设置该参数，必须填入 `width`、`height`、`fps` 和 `bitrate` 字段。请参考[如何设置录制视频的分辨率](../../cn/cloud-recording/recording_video_profile.md)设置该参数。
  - `width`：（必填）Number 类型，视频的宽度，单位为像素，默认值 360。`width` 不能超过 1920，且 `width` 和 `height` 的乘积不能超过 1920 * 1080，超过最大值会报错。
  - `height`：（必填）Number 类型，视频的高度，单位为像素，默认值 640。`height` 不能超过 1920，且 `width` 和 `height` 的乘积不能超过 1920 * 1080，超过最大值会报错。
  - `fps`：（必填）Number 类型，视频的帧率，单位 fps，默认值 15。
  - `bitrate`：（必填）Number 类型，视频的码率，单位 Kbps，默认值 500。
  - `maxResolutionUid`：（选填）String 类型，如果视频合流布局设为垂直布局，用该参数指定显示大视窗画面的用户 ID。
  - `mixedVideoLayout`：（选填）Number 类型，设置视频合流布局，0、1、2 为[预设的合流布局](../../cn/cloud-recording/cloud_recording_layout.md)，3 为自定义合流布局。该参数设为 3 时必须设置 `layoutConfig` 参数。
    - `0`：（默认）悬浮布局。第一个加入频道的用户在屏幕上会显示为大视窗，铺满整个画布，其他用户的视频画面会显示为小视窗，从下到上水平排列，最多 4 行，每行 4 个画面，最多支持共 17 个画面。
    - `1`：自适应布局。根据用户的数量自动调整每个画面的大小，每个用户的画面大小一致，最多支持 17 个画面。
    - `2`：垂直布局。指定一个用户在屏幕左侧显示大视窗画面，其他用户的小视窗画面在右侧垂直排列，最多两列，一列 8 个画面，最多支持共 17 个画面。
    - `3`：自定义布局。设置 `layoutConfig` 参数自定义合流布局。
  - `backgroundColor`：（选填）String 类型。屏幕（画布）的背景颜色。支持 RGB 颜色表，字符串格式为 # 号后 6 个十六进制数。默认值 `"#000000"` 黑色。
  - `layoutConfig`：（选填）JSONArray 类型。由每个用户对应的布局画面设置组成的数组，支持最多 17 个用户画面。当 mixedVideoLayout 设为 3 时，可以通过该参数自定义合流布局。一个用户画面设置包括以下参数：
    - `uid`：（选填）String 类型。字符串内容为待显示在该区域的用户的 UID，32 位无符号整数。如果不指定 UID，会按照用户加入频道的顺序自动匹配 `layoutConfig` 中的画面设置。
    - `x_axis`：（必填）Float 类型。屏幕里该画面左上角的横坐标的相对值，范围是  [0.0,1.0]，精确到小数点后六位。从左到右布局，0.0 在最左端，1.0 在最右端。
    - `y_axis`：（必填）Float 类型。屏幕里该画面左上角的纵坐标的相对值，范围是  [0.0,1.0]，精确到小数点后六位。从上到下布局，0.0 在最上端，1.0 在最下端。
    - `width`：（必填）Float 类型。该画面宽度的相对值，取值范围是 [0.0,1.0]，精确到小数点后六位。
    - `height`：（必填）Float 类型。该画面高度的相对值，取值范围是 [0.0,1.0]，精确到小数点后六位。
    - `alpha`：（选填）Float 类型。图像的透明度。取值范围是 [0.0,1.0] ，精确到小数点后六位。默认值 1.0。0.0 表示图像为透明的，1.0 表示图像为完全不透明的。
    - `render_mode`：（选填）Number 类型。画面显示模式：
      - `0`：（默认）裁剪模式。优先保证画面被填满。视频尺寸等比缩放，直至整个画面被视频填满。如果视频长宽与显示窗口不同，则视频流会按照画面设置的比例进行周边裁剪或图像拉伸后填满画面。
      - `1`：缩放模式。优先保证视频内容全部显示。视频尺寸等比缩放，直至视频窗口的一边与画面边框对齐。如果视频尺寸与画面尺寸不一致，在保持长宽比的前提下，将视频进行缩放后填满画面，缩放后的视频四周会有一圈黑边。
- `subscribeAudioUids`：（选填）JSONArray 类型，由 UID 组成的数组，指定订阅哪几个 UID 的音频流。数组长度不得超过 32，不推荐使用空数组。详见[设置订阅名单](https://docs.agora.io/cn/cloud-recording/cloud_recording_subscription)。
- `unSubscribeAudioUids`: （选填）JSONArray 类型，由 UID 组成的数组，指定不订阅哪几个 UID 的音频流。云端录制会订阅频道内除指定 UID 外所有 UID 的音频流。数组长度不得超过 32，不推荐使用空数组。详见[设置订阅名单](https://docs.agora.io/cn/cloud-recording/cloud_recording_subscription)。
- `subscribeVideoUids`:（选填）JSONArray 类型，由 UID 组成的数组，指定订阅哪几个 UID 的视频流。数组长度不得超过 32，不推荐使用空数组。详见[设置订阅名单](https://docs.agora.io/cn/cloud-recording/cloud_recording_subscription)。
- `unSubscribeVideoUids`: （选填）JSONArray 类型，由 UID 组成的数组，指定不订阅哪几个 UID 的视频流。云端录制会订阅频道内除指定 UID 外所有 UID 的视频流。数组长度不得超过 32，不推荐使用空数组。详见[设置订阅名单](https://docs.agora.io/cn/cloud-recording/cloud_recording_subscription)。

<div class="alert note">如果你设置了音频的订阅名单，但没有设置视频的订阅名单，云端录制服务不会订阅任何视频流。反之亦然。</div>

 - `subscribeUidGroup`: （选填）Number 类型，预估的订阅人数峰值。**在单流模式下，为必填参数。**举例来说，如果 `subscribeVideoUids` 为 `["100","101","102"]`，`subscribeAudioUids` 为 `["101","102","103"]`，则订阅人数为 4 人。
  - `0`: 1 到 2 个 UID
  - `1`: 3 到 7 个 UID
  - `2`: 8 到 12 个 UID
  - `3`: 13 到 17 个 UID

#### <a name="recordingFileConfig"></a>**录制文件设置**

`recordingFileConfig` 是一个用于设置录制文件的 JSON Object。`recordingFileConfig` 不可与 `snapshotConfig` 同时设置，否则会报错。`recordingFileConfig`包含以下字段：

- `avFileType`：（选填）JSONArray 类型，由多个字符串组成的数组，指定录制的视频文件类型。目前只支持默认值 `["hls"]`，即录制生成 M3U8 和 TS 文件。

#### <a name="snapshotConfig"></a>**截图设置**

`snapshotConfig` 是一个用于设置截图的 JSON Object。使用云端录制进行截图，需要注意以下参数的设置。设置错误会收到报错，或无法生成截图文件。

- 请求 URL 中的 `mode` 参数必须设为 `individual`。
- 不可设置 `recordingFileConfig`。
- `streamTypes` 必须设置为 1 或 2
- 如果设置了 `subscribeAudioUid`，则必须同时设置 `subscribeVideoUids`。

`snapshotConfig` 包含以下字段：

- `captureInterval`：（选填）Integer 类型，截图周期（s），云端录制会按此周期定期截图。取值范围是 [5, 3600]，默认值 `10`。
- `fileType`：JSONArray 类型，由多个字符串组成的数组，指定截图的文件格式。目前只支持 `["jpg"]`，即生成 JPG 截图文件。

#### <a name="storageConfig"></a>**云存储设置**

`storageConfig` 是一个用于设置第三方云存储的 JSON Object，包含以下字段：

- `vendor`：Number 类型，第三方云存储供应商。

  - `0`：[七牛云](https://www.qiniu.com/products/kodo)
  - `1`：[Amazon S3](https://aws.amazon.com/cn/s3/?nc2=h_m1)
  - `2`：[阿里云](https://www.aliyun.com/product/oss)
  - `3`：[腾讯云](https://cloud.tencent.com/product/cos)
  - `4`：[金山云](https://www.ksyun.com/post/product/KS3.html)

- `region`：Number 类型，第三方云存储指定的地区信息。
  当 `vendor` = 0，即第三方云存储为七牛云时：

  - `0`：华东
  - `1`：华北
  - `2`：华南
  - `3`：北美
  - `4`：东南亚

  当 `vendor` = 1，即第三方云存储为 Amazon S3 时：

  - `0`：US_EAST_1
  - `1`：US_EAST_2
  - `2`：US_WEST_1
  - `3`：US_WEST_2
  - `4`：EU_WEST_1
  - `5`：EU_WEST_2
  - `6`：EU_WEST_3
  - `7`：EU_CENTRAL_1
  - `8`：AP_SOUTHEAST_1
  - `9`：AP_SOUTHEAST_2
  - `10`：AP_NORTHEAST_1
  - `11`：AP_NORTHEAST_2
  - `12`：SA_EAST_1
  - `13`：CA_CENTRAL_1
  - `14`：AP_SOUTH_1
  - `15`：CN_NORTH_1
  - `17`：US_GOV_WEST_1

  当 `vendor` = 2，即第三方云存储为阿里云时：

  - `0`：CN_Hangzhou
  - `1`：CN_Shanghai
  - `2`：CN_Qingdao
  - `3`：CN_Beijin
  - `4`：CN_Zhangjiakou
  - `5`：CN_Huhehaote
  - `6`：CN_Shenzhen
  - `7`：CN_Hongkong
  - `8`：US_West_1
  - `9`：US_East_1
  - `10`：AP_Southeast_1
  - `11`：AP_Southeast_2
  - `12`：AP_Southeast_3
  - `13`：AP_Southeast_5
  - `14`：AP_Northeast_1
  - `15`：AP_South_1
  - `16`：EU_Central_1
  - `17`：EU_West_1
  - `18`：EU_East_1

  当 `vendor` = 3，即第三方云存储为腾讯云时：

  - `0`：AP_Beijing_1
  - `1`：AP_Beijing
  - `2`：AP_Shanghai
  - `3`：AP_Guangzhou
  - `4`：AP_Chengdu
  - `5`：AP_Chongqing
  - `6`：AP_Shenzhen_FSI
  - `7`：AP_Shanghai_FSI
  - `8`：AP_Beijing_FSI
  - `9`：AP_Hongkong
  - `10`：AP_Singapore
  - `11`：AP_Mumbai
  - `12`：AP_Seoul
  - `13`：AP_Bangkok
  - `14`：AP_Tokyo
  - `15`：NA_Siliconvalley
  - `16`：NA_Ashburn
  - `17`：NA_Toronto
  - `18`：EU_Frankfurt
  - `19`：EU_Moscow

  当 `vendor` = 4，即第三方云存储为金山云时：

  - `0`：CN_Hangzhou
  - `1`：CN_Shanghai
  - `2`：CN_Qingdao
  - `3`：CN_Beijing
  - `4`：CN_Guangzhou
  - `5`：CN_Hongkong
  - `6`：JR_Beijing
  - `7`：JR_Shanghai
  - `8`：NA_Russia_1
  - `9`：NA_Singapore_1

- `bucket`：String 类型，第三方云存储的 bucket。
- `accessKey`：String 类型，第三方云存储的 access key。建议提供只写权限的访问密钥。
- `secretKey`：String 类型，第三方云存储的 secret key。

- `fileNamePrefix`：（选填）JSONArray 类型，由多个字符串组成的数组，指定录制文件在第三方云存储中的存储位置。举个例子，`fileNamePrefix` = `["directory1","directory2"]`，将在录制文件名前加上前缀 "`directory1/directory2/`"，即 `directory1/directory2/xxx.m3u8`。前缀长度（包括斜杠）不得超过 128 个字符。字符串中不得出现斜杠、下划线、括号等符号字符。以下为支持的字符集范围：
  - 26 个小写英文字母 a-z
  - 26 个大写英文字母 A-Z
  - 10 个数字 0-9

#### <a name="extensionServiceConfig"></a>扩展服务设置

`extensionServiceConfig` 是一个用于设置扩展服务的 JSON Object。扩展服务是基于 Agora RTC SDK 的一系列应用服务，能够对 Agora RTC SDK 中产生的音视频流进行进一步处理，如视频点播服务。

`extensionServiceConfig` 包含以下字段：

- `errorHandlePolicy`：（选填）String 类型。错误处理策略。目前仅可设置为默认值 `"error_abort"`，表示当某一扩展服务发生错误后，订阅及其他非扩展服务均停止。

- `extensionServices`：JSONArray 类型，由每个扩展服务的设置组成的数组。一个扩展服务的设置包含以下字段：

  - `serviceName`：String 类型，扩展服务的名称。要使用阿里视频点播服务（VoD），你需要将其设置为 `"aliyun_vod_service"`。

    <div class="alert note">使用阿里视频点播服务，需要注意以下参数的设置：<ul><li>请求 URL 中的 <code>mode</code> 参数必须设为 <code>mix</code>。</li><li>不可设置 <code>snapshotConfig</code> 或 <code>storageConfig</code>。</li></ul>

  - `errorHandlePolicy`：（选填）String 类型。错误处理策略。目前仅可设置为默认值 `"error_abort"`，表示如果当前扩展服务发生错误，其他扩展服务均停止。

  - `serviceParam`：JSON 类型。扩展服务的具体参数设置。当使用阿里视频点播服务时，你需要设置以下参数：

    - `accessKey`：String 类型。阿里云访问秘钥 AccessKey 中的 `AccessKeyId`。详见[阿里云文档](https://help.aliyun.com/document_detail/53045.html)。
    - `secretKey`：String 类型。阿里云访问秘钥 AccessKey 中的 `AccessKeySecret`。详见[阿里云文档](https://help.aliyun.com/document_detail/53045.html)。
    - `regionId`：String 类型，接入区域标识。详见[阿里云文档](https://help.aliyun.com/document_detail/98194.html)。
    - `apiData`：JSON 类型。阿里视频点播服务的详细设置，包含以下参数：
      - `videoData`：JSON 类型。视频设置。你可以至[阿里云文档中心](https://help.aliyun.com/document_detail/55407.html)获取以下参数的详细解释。
        - `title`：String 类型。视频标题。
        - `description`：（选填）String 类型。视频描述。
        - `coverUrl`：（选填）String 类型。自定义视频封面的 URL 地址。
        - `cateId`：（选填）String 类型。字符串内容为视频分类 ID。
        - `tags`：（选填）String 类型。视频标签。
        - `templateGroupId`：（选填）String 类型。转码模板组 ID。
        - `userData`：（选填）String 类型。自定义设置。
        - `storageLocation`：（选填）String 类型。存储地址。
        - `workflowId`：（选填）String 类型。工作流 ID。
	
	
### `start` 请求示例

- 请求 URL：

```
https://api.agora.io/v1/apps/<yourappid>/cloud_recording/resourceid/<resourceid>/mode/<mode>/start
```

- `Content-type` 为 `application/json;charset=utf-8`
- `Authorization` 为 Basic authorization，生成方法请参考 [RESTful API 认证](https://docs.agora.io/cn/faq/restful_authentication)。
- 请求包体内容：

#### 录制	

```json
{
    "uid": "527841",
    "cname": "httpClient463224",
    "clientRequest": {
        "token": "<token if any>",
        "recordingConfig": {
            "maxIdleTime": 30,
            "streamTypes": 2,
            "channelType": 0, 
            "transcodingConfig": {
                "height": 640, 
                "width": 360,
                "bitrate": 500, 
                "fps": 15, 
                "mixedVideoLayout": 1,
                "backgroundColor": "#FF0000"
                        },
            "subscribeVideoUids": ["123","456"], 
            "subscribeAudioUids": ["123","456"],
            "subscribeUidGroup": 0
        }, 
        "recordingFileConfig": {
            "avFileType": ["hls"]
        },
        "storageConfig": {
            "accessKey": "xxxxxxf",
            "region": 3,
            "bucket": "xxxxx",
            "secretKey": "xxxxx",
            "vendor": 2,
            "fileNamePrefix": ["directory1","directory2"]
        }
    }
}
```

#### 截图

```json
{
    "uid": "527841",
    "cname": "httpClient463224",
    "clientRequest": {
        "token": "<token if any>",
        "recordingConfig": {
            "maxIdleTime": 30,
            "streamTypes": 2,
            "channelType": 0, 
            "subscribeUidGroup": 0
        }, 
        "snapshotConfig": {
            "captureInterval": 5,
            "fileType": ["jpg"]
        },
        "storageConfig": {
            "accessKey": "xxxxxxf",
            "region": 3,
            "bucket": "xxxxx",
            "secretKey": "xxxxx",
            "vendor": 2,
            "fileNamePrefix": ["directory1","directory2"]
        }
    }
}
```
#### 阿里视频点播
```json
{
    "uid": "527841",
    "cname": "httpClient463224",
    "clientRequest": {
        "token": "<token if any>",
        "recordingConfig": {
            "maxIdleTime": 30,
            "streamTypes": 2,
            "channelType": 0,
            "subscribeUidGroup": 0
       },
        "extensionServiceConfig": {
          "extensionServices": [{
            "serviceName": "aliyun_vod_service",
            "serviceParam": {
              "secretKey": "xxxxxx",
              "accessKey": "xxxxxx",
              "regionId": "cn-shanghai",
              "apiData": {
                "videoData": {
                  "title": "My Video",
                  "description": "This is my first video",
                  "coverUrl": "http://xxxxx"
               }
             }
           }
         }]
       }
   }
}
```			
### `start` 响应示例

```json
"Code": 200,
"Body":
{
  "sid": "38f8e3cfdc474cd56fc1ceba380d7e1a", 
  "resourceId": "JyvK8nXHuV1BE64GDkAaBGEscvtHW7v8BrQoRPCHxmeVxwY22-x-kv4GdPcjZeMzoCBUCOr9q-k6wBWMC7SaAkZ_4nO3JLqYwM1bL1n6wKnnD9EC9waxJboci9KUz2WZ4YJrmcJmA7xWkzs_L3AnNwdtcI1kr_u1cWFmi9BWAWAlNd7S7gfoGuH0tGi6CNaOomvr7-ILjPXdCYwgty1hwT6tbAuaW1eqR0kOYTO0Z1SobpBxu1czSFh1GbzGvTZG"
}
```

- `code`: Number 类型，[响应状态码](#status)。
- `resourceId`: String 类型，云端录制使用的 resource ID。
- `sid`: String 类型，录制 ID。成功开始云端录制后，你会得到一个 sid （录制 ID)。该 ID 是一次录制周期的唯一标识。

## <a name="updateUID"></a>更新订阅名单的 API

在录制过程中，可以随时调用该方法更新订阅的 UID 名单。每次调用该方法都会覆盖原来的订阅设置。

- 方法：POST
- 接入点：/v1/apps/\<appid\>/cloud_recording/resourceid/\<resourceid\>/sid/\<sid\>/mode/\<mode\>/update

> 每个 App ID 每秒钟的请求数限制为 10 次。如需提高此限制，请[提交工单](https://agora-ticket.agora.io/)联系技术支持。


### 参数

该 API 需要在 URL 中传入以下参数。

| 参数       | 类型   | 描述                                                         |
| :--------- | :----- | :----------------------------------------------------------- |
|`appid`      | String | 你的项目使用的 App ID |
| `resourceid` | String | 通过 [`acquire`](#acquire) 请求获取的 resource ID。 |
| `sid`        | String | 通过 [`start`](#start) 请求获取的录制 ID。 |
| `mode`       | String | 录制模式，支持单流模式 `individual` 和合流模式 `mix`（默认模式）。 |

该 API 需要在请求包体中传入以下参数。

| 参数            | 类型   | 描述                                                         |
| :-------------- | :----- | :----------------------------------------------------------- |
| `cname`         | String | 待录制的频道名。                                             |
| `uid`           | String | 字符串内容为云端录制服务在频道内使用的 UID，用于标识该录制服务，需要和你在 [`acquire`](#acquire) 请求中输入的 UID 相同。 |
| `clientRequest`  | JSON   | 客户请求参数，包含 `streamSubscribe` 字段。`streamSubscribe` 为 JSON 类型，用于更新订阅名单。 |

`streamSubscribe` 包含以下参数：

- `audioUidList`：（选填）JSON 类型。音频订阅名单。如果 `recordingConfig` 中的 `streamTypes` 为 `1` （只订阅视频），设置该参数会报错。
  - `subscribeAudioUids`：（选填）JSONArray 类型，由 UID 组成的数组，指定订阅哪几个 UID 的音频流。数组长度不得超过 32，数组不可为空。详见[设置订阅名单](https://docs.agora.io/cn/cloud-recording/cloud_recording_subscription)。
  - `unSubscribeAudioUids`: （选填）JSONArray 类型，由 UID 组成的数组，指定不订阅哪几个 UID 的音频流。云端录制会订阅频道内除指定 UID 外所有 UID 的音频流。数组长度不得超过 32，数组不可为空。详见[设置订阅名单](https://docs.agora.io/cn/cloud-recording/cloud_recording_subscription)。
- `videoUidList`：（选填）JSON 类型。视频订阅名单。如果 `recordingConfig` 中的 `streamTypes` 为 `0` （只订阅音频），设置该参数会报错。
  - `subscribeVideoUids`：（选填）JSONArray 类型，由 UID 组成的数组，指定订阅哪几个 UID 的视频流。数组长度不得超过 32，数组不可为空。详见[设置订阅名单](https://docs.agora.io/cn/cloud-recording/cloud_recording_subscription)。
  - `unSubscribeVideoUids`：（选填）JSONArray 类型，由 UID 组成的数组，指定不订阅哪几个 UID 的视频流。云端录制会订阅频道内除指定 UID 外所有 UID 的视频流。数组长度不得超过 32，数组不可为空。详见[设置订阅名单](https://docs.agora.io/cn/cloud-recording/cloud_recording_subscription)。

### update 请求示例

- 请求 URL：

```
https://api.agora.io/v1/apps/<appid>/cloud_recording/resourceid/<resourceid>/sid/<sid>/mode/<mode>/update
```

- `Content-type` 为 `application/json;charset=utf-8`
- `Authorization` 为 Basic authorization，生成方法请参考 [RESTful API 认证](https://docs.agora.io/cn/faq/restful_authentication)。
- 请求包体内容：

```
{
 "uid": "527841",
 "cname": "httpClient463224",
 "clientRequest": {
  "streamSubscribe": {
   "audioUidList": {
    "subscribeAudioUids": ["#allstream#"]
   },
   "videoUidList": {
    "unSubscribeVideoUids": ["444", "555", "666"]
   }
  }
 }
}                                                         
```

### update 响应示例

```
"Code": 200,
"Body":
{
  "sid": "38f8e3cfdc474cd56fc1ceba380d7e1a", 
  "resourceId": "JyvK8nXHuV1BE64GDkAaBGEscvtHW7v8BrQoRPCHxmeVxwY22-x-kv4GdPcjZeMzoCBUCOr9q-k6wBWMC7SaAkZ_4nO3JLqYwM1bL1n6wKnnD9EC9waxJboci9KUz2WZ4YJrmcJmA7xWkzs_L3AnNwdtcI1kr_u1cWFmi9BWAWAlNd7S7gfoGuH0tGi6CNaOomvr7-ILjPXdCYwgty1hwT6tbAuaW1eqR0kOYTO0Z1SobpBxu1czSFh1GbzGvTZG"
}
```

- `code`: Number 类型，[响应状态码](#status)。
- `resourceId`: String 类型，云端录制使用的 resource ID。
- `sid`: String 类型，录制 ID。成功开始云端录制后，你会得到一个 sid （录制 ID)。该 ID 是一次录制周期的唯一标识。

## <a name="update"></a>更新合流布局的 API

在录制过程中，可以随时调用该方法更新合流布局的设置。

每次调用该方法都会覆盖原来的布局设置，例如在开始录制时设置了 `backgroundColor` 为 `"#FF0000"`（红色），调用该方法更新合流布局时如果不设置 `backgroundColor` 参数，背景色会变为默认值黑色。

- 方法：POST
- 接入点：/v1/apps/\<appid\>/cloud_recording/resourceid/\<resourceid\>/sid/\<sid\>/mode/\<mode\>/updateLayout

> 每个 App ID 每秒钟的请求数限制为 10 次。如需提高此限制，请[提交工单](https://agora-ticket.agora.io/)联系技术支持。


### 参数

该 API 需要在 URL 中传入以下参数。

| 参数       | 类型   | 描述                                                         |
| :--------- | :----- | :----------------------------------------------------------- |
| appid      | String | 你的项目使用的 App ID。必须使用和待录制的频道相同的 App ID。                                   |
| resourceid | String | 通过 [`acquire`](https://docs.agora.io/cn/cloud-recording/cloud_recording_api_rest?platform=All%20Platforms#acquire) 请求获取的 resource ID。 |
| sid        | String | 通过 [`start`](https://docs.agora.io/cn/cloud-recording/cloud_recording_api_rest?platform=All%20Platforms#start) 请求获取的录制 ID。 |
| mode       | String | 录制模式，只支持合流模式 `mix` （默认模式）。 |

该 API 需要在请求包体中传入以下参数。

| 参数            | 类型   | 描述                                                         |
| :-------------- | :----- | :----------------------------------------------------------- |
| `cname`         | String | 待录制的频道名。                                             |
| `uid`           | String | 字符串内容为云端录制服务在频道内使用的 UID，用于标识该录制服务，需要和你在 [`acquire`](#acquire) 请求中输入的 UID 相同。 |
| `clientRequest` | JSON   | 特定的客户请求参数，对于该请求请参考下面的列表设置。         |

clientRequest 中需要填写的内容如下：

- `maxResolutionUid`：（选填）String 类型，如果 `layoutType` 设为 2（垂直布局），用该参数指定显示大视窗画面的用户 ID。
- `mixedVideoLayout`：（选填）Number 类型，设置视频合流布局，0、1、2 为[预设的合流布局](../../cn/cloud-recording/cloud_recording_layout.md)，3 为自定义合流布局。该参数设为 3 时必须设置 `layoutConfig` 参数。
  - `0`：（默认）悬浮布局。第一个加入频道的用户在屏幕上会显示为大视窗，铺满整个画布，其他用户的视频画面会显示为小视窗，从下到上水平排列，最多 4 行，每行 4 个画面，最多支持共 17 个画面。
  - `1`：自适应布局。根据用户的数量自动调整每个画面的大小，每个用户的画面大小一致，最多支持 17 个画面。
  - `2`：垂直布局。指定一个用户在屏幕左侧显示大视窗画面，其他用户的小视窗画面在右侧垂直排列，最多两列，一列 8 个画面，最多支持共 17 个画面。
  - `3`：自定义布局。设置 `layoutConfig` 参数自定义合流布局。
- `backgroundColor`：（选填）String 类型。屏幕（画布）的背景颜色。支持 RGB 颜色表，字符串格式为 # 号后 6 个十六进制数。默认值 `"#000000"` 黑色。
- `layoutConfig`：（选填）JSONArray 类型。由每个用户对应的布局画面设置组成的数组，支持最多 17 个用户画面。当 `layoutType` 设为 3 时，可以通过该参数自定义合流布局。一个用户画面设置包括以下参数：
  - `uid`：（选填）String 类型。字符串内容为待显示在该画面的用户的 UID，32 位无符号整数。如果不指定 UID，会按照用户加入频道的顺序自动匹配 `layoutConfig` 中的画面设置。
  - `x_axis`：（必填）Float 类型。屏幕里该画面左上角的横坐标的相对值，范围是  [0.0,1.0]。从左到右布局，0.0 在最左端，1.0 在最右端。`x_axi` 也可设置为整数 0 或 1。
  - `y_axis`：（必填）Float 类型。屏幕里该画面左上角的纵坐标的相对值，范围是  [0.0,1.0]。从上到下布局，0.0 在最上端，1.0 在最下端。`y_axi` 也可设置为整数 0 或 1。
  - `width`：（必填）Float 类型。该画面宽度的相对值，取值范围是 [0.0,1.0]。`width` 也可设置为整数 0 和 1。
  - `height`：（必填）Float 类型。该画面高度的相对值，取值范围是 [0.0,1.0]。`height` 也可设置为整数 0 和 1。
  - `alpha`：（选填）Float 类型。图像的透明度。取值范围是 [0.0,1.0] 。默认值 1.0。0.0 表示图像为透明的，1.0 表示图像为完全不透明的。
  - `render_mode`：（选填）Number 类型。画面显示模式：
    - 0：（默认）裁剪模式。优先保证画面被填满。视频尺寸等比缩放，直至整个画面被视频填满。如果视频长宽与显示窗口不同，则视频流会按照画面设置的比例进行周边裁剪或图像拉伸后填满画面。
    - 1：缩放模式。优先保证视频内容全部显示。视频尺寸等比缩放，直至视频窗口的一边与画面边框对齐。如果视频尺寸与画面尺寸不一致，在保持长宽比的前提下，将视频进行缩放后填满画面，缩放后的视频四周会有一圈黑边。

### updateLayout 请求示例

- 请求 URL：

```
https://api.agora.io/v1/apps/<appid>/cloud_recording/resourceid/<resourceid>/sid/<sid>/mode/<mode>/updateLayout
```

- `Content-type` 为 `application/json;charset=utf-8`
- `Authorization` 为 Basic authorization，生成方法请参考 [RESTful API 认证](https://docs.agora.io/cn/faq/restful_authentication)。
- 请求包体内容：

```json
{
    "uid": "527841",
    "cname": "httpClient463224",
    "clientRequest": {
        "mixedVideoLayout": 3,
        "backgroundColor": "#FF0000",
        "layoutConfig": [
        {
            "uid": "1",
             "x_axis": 0.1,
             "y_axis": 0.1,
             "width": 0.1,
             "height": 0.1,
             "alpha": 1.0,
            "render_mode": 1
         },
        {
            "uid": "2",
             "x_axis": 0.2,
            "y_axis": 0.2,
             "width": 0.1,
             "height": 0.1,
             "alpha": 1.0,
             "render_mode": 1
         }
         ]
     }
}
```

### updateLayout 响应示例

```json
"Code": 200,
"Body":
{
  "sid": "38f8e3cfdc474cd56fc1ceba380d7e1a", 
  "resourceId": "JyvK8nXHuV1BE64GDkAaBGEscvtHW7v8BrQoRPCHxmeVxwY22-x-kv4GdPcjZeMzoCBUCOr9q-k6wBWMC7SaAkZ_4nO3JLqYwM1bL1n6wKnnD9EC9waxJboci9KUz2WZ4YJrmcJmA7xWkzs_L3AnNwdtcI1kr_u1cWFmi9BWAWAlNd7S7gfoGuH0tGi6CNaOomvr7-ILjPXdCYwgty1hwT6tbAuaW1eqR0kOYTO0Z1SobpBxu1czSFh1GbzGvTZG"
}
```

- `code`：Number 类型，[响应状态码](#status)。
- `resourceId`：String 类型，云端录制使用的 resource ID。
- `sid`：String 类型，录制 ID。成功开始云端录制后，你会得到一个 sid （录制 ID)。该 ID 是一次录制周期的唯一标识。

## <a name="query"></a>查询云端录制状态的 API

开始录制后，你可以调用 query 查询录制状态。

<div class="note alert"><code>query</code> 请求仅在会话内有效。如果录制启动错误，或录制已结束，调用 <code>query</code> 将返回 404。</div>

- 方法：GET

- 接入点：/v1/apps/\<appid\>/cloud_recording/resourceid/\<resourceid\>/sid/\<sid\>/mode/\<mode\>/query

> 每个 App ID 每秒钟的请求数限制为 10 次。如需提高此限制，请[提交工单](https://agora-ticket.agora.io/)联系技术支持。


### 参数

该 API 需要在 URL 中传入以下参数。

| 参数       | 类型   | 描述                                                         |
| :--------- | :----- | :----------------------------------------------------------- |
| appid      | String | 你的项目使用的 App ID。必须使用和待录制的频道相同的 App ID。 |
| resourceid | String | 通过 [`acquire`](#acquire) 请求获取的 resource ID。          |
| sid        | String | 通过 [`start`](#start) 请求获取的录制 ID。                   |
| mode       | String | 录制模式，支持单流模式 `individual` 和合流模式 `mix` （默认模式）。 |

### `query` 请求示例

- 请求 URL：

```
https://api.agora.io/v1/apps/<yourappid>/cloud_recording/resourceid/<resourceid>/sid/<sid>/mode/<mode>/query
```

- `Content-type` 为 `application/json;charset=utf-8`
- `Authorization` 为 Basic authorization，生成方法请参考 [RESTful API 认证](https://docs.agora.io/cn/faq/restful_authentication)。

### `query` 响应示例
#### 录制或截图
 ```json
"Code": 200,
"Body":
{
  "resourceId":"JyvK8nXHuV1BE64GDkAaBGEscvtHW7v8BrQoRPCHxmeVxwY22-x-kv4GdPcjZeMzoCBUCOr9q-k6wBWMC7SaAkZ_4nO3JLqYwM1bL1n6wKnnD9EC9waxJboci9KUz2WZ4YJrmcJmA7xWkzs_L3AnNwdtcI1kr_u1cWFmi9BWAWAlNd7S7gfoGuH0tGi6CNaOomvr7-ILjPXdCYwgty1hwT6tbAuaW1eqR0kOYTO0Z1SobpBxu1czSFh1GbzGvTZG",
  "sid":"38f8e3cfdc474cd56fc1ceba380d7e1a",
  "serverResponse":{
    "fileListMode": "json",
    "fileList": [
   {
      "filename": "xxx.m3u8",
      "trackType": "audio_and_video",
      "uid": "123",
      "mixedAllUser": true,
      "isPlayable": true,
      "sliceStartTime": 1562724971626
   },    
   {
      "filename": "xxx.m3u8",
      "trackType": "audio_and_video",
      "uid": "456",
      "mixedAllUser": true,
      "isPlayable": true,
      "sliceStartTime": 1562724971626
   }
   ],
    "status": "5",
    "sliceStartTime": 1562724971626
   }       
}
 ```
#### 阿里视频点播
```json
"Code": 200,
"Body":
{
  "resourceId":"JyvK8nXHuV1BE64GDkAaBGEscvtHW7v8BrQoRPCHxmeVxwY22-x-kv4GdPcjZeMzoCBUCOr9q-k6wBWMC7SaAkZ_4nO3JLqYwM1bL1n6wKnnD9EC9waxJboci9KUz2WZ4YJrmcJmA7xWkzs_L3AnNwdtcI1kr_u1cWFmi9BWAWAlNd7S7gfoGuH0tGi6CNaOomvr7-ILjPXdCYwgty1hwT6tbAuaW1eqR0kOYTO0Z1SobpBxu1czSFh1GbzGvTZG",
  "sid":"38f8e3cfdc474cd56fc1ceba380d7e1a",
  "serverResponse":{
    "extensionServiceState":[
     {
        "payload":{
          "state": "inProgress",
          "videoInfo":[
           {
              "fileName":"38f8e3cfdc474cd56fc1ceba380d7e1a_httpClient463224.m3u8",
              "videoId":"3af7751d658a4381bbed893381708c92"
           }
         ]
       },
        "serviceName": "aliyun_vod_service"
     }
   ],
    "subServiceStatus":{
      "recordingService": "serviceInProgress"
   }
 }
}
```
- `code`：Number 类型，[响应状态码](#status)。
- `resourceId`: String 类型，云端录制使用的 resource ID。
- `sid`: String 类型，录制 ID。成功开始云端录制后，你会得到一个 sid （录制 ID)。该 ID 是一次录制周期的唯一标识。
- `serverResponse`：JSON 类型，服务器返回的具体信息。  
  <div class="alert note">该字段中的子元素与你在 start 请求中的设置有关。例如，如果你在 <code>start</code> 请求中设置了 <code>snapshotConfig</code> 或 <code>extensionServiceConfig</code>，则 <code>query</code> 不会返回 <code>fileListMode</code> 字段。</div>
			
  - `fileListMode`: String 类型，`fileList` 字段的数据格式。
    - `"string"`：`fileList` 为 String 类型。合流模式下，`fileListMode` 为 `"string"`。
    - `"json"`：`fileList` 为 JSONArray 类型。单流模式下，`fileListMode` 为 `"json"`。
  - `fileList`：当 `fileListMode` 为 `"string"` 时，`fileList` 为 String 类型，录制产生的 M3U8 文件的文件名。当 `fileListMode` 为 `"json"` 时, `fileList` 为 JSONArray 类型，由每个录制文件的具体信息组成的数组。如果你设置了 `snapshotConfig`，则不会返回该字段。一个录制文件的具体信息包括以下字段:
    - `filename`：String 类型，录制产生的 M3U8 文件的文件名。
    - `trackType`：String 类型，录制文件的类型。
      - `"audio"`：纯音频文件。
      - `"video"`：纯视频文件。
      - `"audio_and_video"`：音视频文件。
    - `uid`：String 类型，用户 UID，表示录制的是哪个用户的音频流或视频流。
    - `mixedAllUser`：Boolean 类型，用户是否是分开录制的。
      - `true`：所有用户合并在一个录制文件中。
      - `false`：每个用户分开录制。
    - `isPlayable`：Boolean 类型，是否可以在线播放。
      - `true`：可以在线播放。
      - `false`：无法在线播放。
    - `sliceStartTime`：Number 类型，该文件的录制开始时间，Unix 时间戳，单位为毫秒。
  - `status`：Number 类型，当前云服务的状态。
    - `0`：没有开始云服务。
    - `1`：云服务初始化完成。
    - `2`：云服务组件开始启动。
    - `3`：云服务部分组件启动完成。
    - `4`：云服务所有组件启动完成。
    - `5`：云服务正在进行中。
    - `6`：云服务收到停止请求。
    - `7`：云服务所有组件均停止。
    - `8`：云服务已退出。
    - `20`：云服务异常退出。
  - `sliceStartTime`: Number 类型，录制开始的时间，Unix 时间戳，单位为毫秒。
 - `extensionServiceState`: JSONArray 类型。由每个扩展服务的详细状态信息组成的数组。
    - `serviceName`：String 类型，扩展服务类型。 `"aliyun_vod_service"` 代表阿里视频点播服务。
    - `payload`：JSON 类型。该扩展服务的状态信息。
      - `state`：String 类型，将订阅内容上传至扩展服务的状态。
        - `"inProgress"`：正在将录制文件上传至扩展服务。
        - `"idle"`：无发流端，上传停止。
      - `videoInfo`：JSONArray 类型。M3U8 文件和视频 ID 的对应关系。每个 M3U8 文件上传至阿里视频点播服务后，都会生成一个视频 ID。
        - `fileName`：String 类型，M3U8 文件的文件名。
        - `videoId`：String 类型，视频 ID。
  - `subServiceStatus`：JSON 类型，云端录制子模块的状态。
    - `recordingService`：String 类型，订阅模块的状态，具体取值详见[服务状态](#service_status)。



## <a name="stop"></a>停止云端录制的 API

录制完成后，调用该方法离开频道，停止录制。录制停止后如需再次录制，必须再调用  [`acquire`](#acquire) 方法请求一个新的 resource ID。

- 方法：POST
- 接入点：/v1/apps/\<appid\>/cloud_recording/resourceid/\<resourceid\>/sid/\<sid\>/mode/\<mode\>/stop

> - 每个 App ID 每秒钟的请求数限制为 10 次。如需提高此限制，请[提交工单](https://agora-ticket.agora.io/)联系技术支持。
> - 当频道空闲（无用户）超过预设时间（默认为 30 秒） 后，云端录制也会自动退出频道停止录制。

### 参数

该 API 需要在 URL 中传入以下参数。

| 参数         | 类型   | 描述                                                         |
| :----------- | :----- | :----------------------------------------------------------- |
| `appid`      | String | 你的项目使用的 App ID。必须使用和待录制的频道相同的 App ID。 |
| `resourceid` | String | 通过 [`acquire`](#acquire) 请求获取的 resource ID。          |
| `sid`        | String | 通过 [`start`](#start) 请求获取的录制 ID。                   |
| `mode`       | String | 录制模式，支持单流模式 `individual` 和合流模式 `mix` （默认模式）。 |

该 API 需要在请求包体中传入以下参数。

| 参数            | 类型   | 描述                                                         |
| :-------------- | :----- | :----------------------------------------------------------- |
| `cname`         | String | 待录制的频道名。                                             |
| `uid`           | String | 字符串内容为云端录制服务在频道内使用的 UID，用于标识该录制服务，需要和你在 [`acquire`](#acquire) 请求中输入的 UID 相同。 |
| `clientRequest` | JSON   | 特定的客户请求参数，对于该方法无需填入任何内容，为一个空的 JSON。 |

### `stop` 请求示例

- 请求 URL：

```
https://api.agora.io/v1/apps/<yourappid>/cloud_recording/resourceid/<resourceid>/sid/<sid>/mode/<mode>/stop
```

- `Content-type` 为 `application/json;charset=utf-8`
- `Authorization` 为 Basic authorization，生成方法请参考 [RESTful API 认证](https://docs.agora.io/cn/faq/restful_authentication)。
- 请求包体内容：

 ```json
{
    "cname": "httpClient463224",
    "uid": "527841",  
    "clientRequest":{
    }
}
 ```

### `stop` 响应示例

```json
"Code": 200,
"Body":
{
  "resourceId":"JyvK8nXHuV1BE64GDkAaBGEscvtHW7v8BrQoRPCHxmeVxwY22-x-kv4GdPcjZeMzoCBUCOr9q-k6wBWMC7SaAkZ_4nO3JLqYwM1bL1n6wKnnD9EC9waxJboci9KUz2WZ4YJrmcJmA7xWkzs_L3AnNwdtcI1kr_u1cWFmi9BWAWAlNd7S7gfoGuH0tGi6CNaOomvr7-ILjPXdCYwgty1hwT6tbAuaW1eqR0kOYTO0Z1SobpBxu1czSFh1GbzGvTZG",
  "sid":"38f8e3cfdc474cd56fc1ceba380d7e1a",
  "serverResponse":{
    "fileListMode": "json",
    "fileList": [
    {
      "filename": "xxx.m3u8",
      "trackType": "audio_and_video",
      "uid": "123",
      "mixedAllUser": true,
      "isPlayable": true,
      "sliceStartTime": 1562724971626
    },
    {
      "filename": "xxx.m3u8",
      "trackType": "audio_and_video",
      "uid": "456",
      "mixedAllUser": true,
      "isPlayable": true,
      "sliceStartTime": 1562724971626
    }
    ],
    "uploadingStatus": "uploaded"
  }
}
```

- `code`: Number 类型，[响应状态码](#status)。
- `resourceId`: String 类型，云端录制使用的 resource ID。
- `sid`: String 类型，录制 ID。成功开始云端录制后，你会得到一个 sid （录制 ID)。该 ID 是一次录制周期的唯一标识。
- `serverResponse`: JSON 类型，服务器返回的具体信息。`serverResponse` 中包含如下字段：
  - `fileListMode`: String 类型，`fileList` 字段的数据格式。如果你设置了 `snapshotConfig`，则不会返回该字段。
    - `"string"`：`fileList` 为 String 类型。合流模式下，`fileListMode` 为 `"string"`。
    - `"json"`：`fileList` 为 JSONArray 类型。单流模式下，`fileListMode` 为 `"json"`。
  - `fileList`：当 `fileListMode` 为 `"string"` 时，`fileList` 为 String 类型，表示录制产生的 M3U8 文件的文件名。当 `fileListMode` 为 `"json"` 时, `fileList` 为 JSONArray 类型，由每个录制文件的具体信息组成的数组。如果你设置了 `snapshotConfig`，则不会返回该字段。一个录制文件的具体信息包括以下字段:
    - `filename`：String 类型，录制产生的 M3U8 文件的文件名。
    - `trackType`：String 类型，录制文件的类型。
      - `"audio"`：纯音频文件。
      - `"video"`：纯视频文件。
      - `"audio_and_video"`：音视频文件。
    - `uid`：String 类型，用户 UID，表示录制的是哪个用户的音频流或视频流。
    - `mixedAllUser`：Boolean 类型，用户是否是分开录制的。
      - `true`：所有用户合并在一个录制文件中。
      - `false`：每个用户分开录制。
    - `isPlayable`：Boolean 类型，是否可以在线播放。
      - `true`：可以在线播放。
      - `false`：无法在线播放。
    - `sliceStartTime`：Number 类型，该文件的录制开始时间，Unix 时间戳，单位为毫秒。
  - `uploadingStatus`：String 类型，当前录制上传的状态。
    - `"uploaded"`：本次录制的文件已经全部上传至指定的第三方云存储。
    - `"backuped"`：本次录制的文件已经全部上传完成，但是至少有一个 TS 文件上传到了 Agora 云备份。Agora 服务器会自动将这部分文件继续上传至指定的第三方云存储。
    - `"unknown"`：未知状态。


## <a name="status"></a>响应状态码

| 状态码 | 描述                                                         |
| :----- | :----------------------------------------------------------- |
| 200    | 请求成功。                                                   |
| 201    | 成功请求并创建了新的资源。                                   |
| 206    | 整个录制过程中没有用户发流，或部分录制文件没有上传到第三方云存储。 |
| 400    | 请求的语法错误（如参数错误），服务器无法理解。如果你填入的 App ID 没有[开通云端录制权限](https://docs.agora.io/cn/cloud-recording/cloud_recording_rest?platform=All%20Platforms#%E5%BC%80%E9%80%9A%E4%BA%91%E7%AB%AF%E5%BD%95%E5%88%B6%E6%9C%8D%E5%8A%A1)，也会返回 400。           |
| 401    | 未经授权的（客户 ID/客户密钥匹配错误）。          |
| 404    | 服务器无法根据请求找到资源（网页）。                         |
| 500    | 服务器内部错误，无法完成请求。                               |
| 504    | 服务器内部错误。充当网关或代理的服务器未从远端服务器获取请求。 |

## <a name="service_status"></a>服务状态

| 状态                      | 描述                                                         |
| :------------------------ | :----------------------------------------------------------- |
| "serviceIdle"             | 子模块服务未开始。                                           |
| "serviceStarted"          | 子模块服务已开始。                                           |
| "serviceReady"            | 子模块服务已就绪。                                           |
| "serviceInProgress"       | 子模块服务正在进行中。                                       |
| "serviceCompleted"        | 订阅内容已全部上传至扩展服务。                               |
| "servicePartialCompleted" | 订阅内容部分上传至扩展服务。                                 |
| "serviceValidationFailed" | 扩展服务验证失败。例如 `extensionServiceConfig` 中 `apiData` 填写错误。 |
| "serviceAbnormal"         | 子模块状态异常。                                             |

## 常见错误

下面仅列出使用云端录制 RESTful API 过程中常见的错误码或错误信息，如果遇到其他错误，请联系 Agora 技术支持。

- `2`：参数不合法，请确保参数类型、大小写和取值范围正确，且必填的参数均已填写。
- `7`：录制已经在进行中 ，请勿用同一个 resource ID 重复 `start` 请求。
- `8`：HTTP 请求头部字段错误，有以下几种情况：
  - `Content-type` 错误，请确保 `Content-type` 为 `application/json;charset=utf-8`。
  - 请求 URL 中缺少 `cloud_recording` 字段。
  - 使用了错误的 HTTP 方法。	
  - 请求包体不是合法的 JSON 格式。
- `49`：使用同一个 resource ID 和录制 ID（sid）重复 `stop` 请求。
- `53`：录制已经在进行中。当采用相同参数再次调用 `acquire` 获得新的 resource ID，并用于 `start` 请求时，会发生该错误。如需发起多路录制，需要在 `acquire` 方法中填入不同的 UID。
- `62`：调用 `Acquire` 请求时，如果出现该错误，表示你填入的 App ID 没有[开通云端录制权限](https://docs.agora.io/cn/cloud-recording/cloud_recording_rest?platform=All%20Platforms#%E5%BC%80%E9%80%9A%E4%BA%91%E7%AB%AF%E5%BD%95%E5%88%B6%E6%9C%8D%E5%8A%A1)。
- `65`：多为网络抖动引起。使用相同 resource ID 重试即可。
- `432`：请求参数错误。请求参数不合法，或请求中的 App ID，频道名或用户 ID 与 resource ID 不匹配。
- `433`：resource ID 过期。获得 resource ID 后必须在 5 分钟内开始云端录制。请重新调用 [`acquire`](#acquire) 获取新的 resource ID。
- `435`：没有录制文件产生。频道内没有用户发流，无录制对象。
- `501`：录制服务正在退出。该错误可能在调用了 [`stop`](#stop) 方法后再调用 [`query`](#query) 时发生。
- `1001`：resource ID 解密失败。请重新调用 [`acquire`](#acquire) 获取新的 resource ID。
- `1003`：App ID 或者录制 ID（sid）与 resource ID 不匹配。请确保在一个录制周期内 resource ID、App ID 和录制 ID 一一对应。
- `1013`：频道名不合法。频道名必须为长度在 64 字节以内的字符串。以下为支持的字符集范围（共 89 个字符）：
  - 26 个小写英文字母 a-z
  - 26 个大写英文字母 A-Z
  - 10 个数字 0-9
  - 空格
  - "!", "#", "$", "%", "&", "(", ")", "+", "-", ":", ";", "<", "=", ".", ">", "?", "@", "[", "]", "^", "_", " {", "}", "|", "~", ","

- `1028`：[`updateLayout`](#update) 方法的请求包体中参数错误。
- `"invalid appid"`：无效的 App ID。请确保 App ID 填写正确。如果你已经确认 App ID 填写正确，但仍出现该错误，请[提交工单](https://agora-ticket.agora.io/)。
- `"no Route matched with those values`": 该错误可能由 HTTP 方法填写错误导致，例如将 GET 方法填写为 POST；也可能由请求 URL 填写错误导致。
- `"Invalid authentication credentials"`: 该错误可能由以下原因导致。 如果你已经排除以下原因，但仍出现该错误，请联系 support@agora.io。
  - Customer ID 或 Customer Certificate 填写错误。
  - App ID 没有[开通云端录制权限](https://docs.agora.io/cn/cloud-recording/cloud_recording_rest?platform=All%20Platforms#%E5%BC%80%E9%80%9A%E4%BA%91%E7%AB%AF%E5%BD%95%E5%88%B6%E6%9C%8D%E5%8A%A1)。
  - HTTP 请求头的认证信息有误，如 `Authorization` 字段的值 `Basic <Authorization>` 缺少 `Basic`。
  - HTTP 请求头的格式不正确，如 `Content-type` 字段的值 `application/json;charset=utf-8` 大小写不正确或包含空格。
