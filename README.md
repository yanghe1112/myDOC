# 上报硬件设备信息

为实现设备用量管控及账单记录功能，你需要将设备信息上报给扣子。
该功能为扣子企业版的白名单功能，如果需要使用该功能，你可以提供扣子的用户 ID（UID），联系扣子商务经理为你的账号添加白名单。在扣子平台左下角单击个人头像，并选择**账号设置**，在账号页面中查看 UID。
![Image](https://p9-arcosite.byteimg.com/tos-cn-i-goo7wpa0wc/9cd7443f558440c09e366b82edddd26f~tplv-goo7wpa0wc-image.image)

## 方案对比
扣子提供两种设备信息传递方式，你可根据业务场景选择合适的接入方式。
| **方案** | **使用场景** | **说明** |
| --- | --- | --- |
| 在 JWT 授权时传递设备信息（推荐） | 适用于每个设备具有独立的密钥的场景。 | 将设备信息嵌入 JWT 的 Payload 中，设备获取 Token 时上报设备信息。 |
| API Header 中传递设备信息 | 适用于后端采用集中管理模式，且未为每个设备分配独立密钥的场景。 | 需要在所有涉及到资源点消耗的 API 的 Header 中添加设备字段，若未正确添加该字段，可能会导致设备用量统计不准确。 |

## 方式一：在 JWT 授权时传递（推荐）
适用于每个设备具有独立的密钥的场景。通过将设备信息嵌入 JWT 的 Payload 部分，设备在签署 JWT 并获取 Access Token 时，上报对应的设备信息，扣子将通过 Access Token 解析设备信息。具体参数及配置可参考[OAuth JWT 授权（渠道场景）](/developer_guides/oauth_jwt_channel)。
在 Payload 的 `session_context.device_info` 参数中配置设备相关信息，参数说明如下表所示。
| **参数** | **类型** | **是否必选** | **说明** |
| --- | --- | --- | --- |
| session_context | Object | 可选  | 会话上下文信息，包含设备相关信息等。 <br>  |
| session_context.device_info | Object | 可选  | 用于配置设备相关信息，扣子平台基于该部分信息对设备做用量管控以及账单记录。 |
| session_context.device_info.device_id | String | 可选  | IoT 等硬件设备 ID，一个设备对应一个唯一的设备号。 |
| session_context.device_info.custom_consumer <br>  | String | 可选  | 自定义维度的实体 ID，你可以根据业务需要进行设置，例如 APP 上的用户名等。 <br> * `device_id` 和 `custom_consumer` 建议选择其中一个即可。 <br> * `custom_consumer `参数用于设备用量管控，与对话等 API 传入的 `user_id` 无关，`user_id` 主要用于上下文、数据库隔离等场景。 <br> * 出于数据隐私及信息安全等方面的考虑，不建议使用业务系统中定义的用户敏感标识（如手机号等）作为 `custom_consumer` 的值。 <br>  |
Payload 示例代码如下：
```JSON
{
    "session_context": {
        "device_info": {
            "device_id": "1234567890" // IoT 等硬件设备的唯一标识 ID
        }
    }
}
```

## 方式二：API Header 传递
建议优先采用方式一。仅在以下场景中可考虑使用此接入方式：后端采用集中管理模式且未为每个设备分配独立密钥。
需要确保设备信息传递的准确性，扣子不验证该信息的真实性。

### API 列表
采用 Header 方式传递时，你需要在所有涉及到资源点消耗的 API 的 Header 中添加 `X-Coze-DeviceInfo` 字段。若未正确添加该字段，可能会导致设备用量统计不准确。具体需要添加该字段的 API 如下表所示。
| **分类** | **API**  |
| --- | --- |
| 音视频 | [语音合成](/developer_guides/text_to_speech) |
|  | [语音识别](/developer_guides/audio_transcriptions) |
|  | [创建房间](/developer_guides/create_room) |
| 对话 | [发起对话](/developer_guides/chat_v3) |
| 工作流 | [执行工作流](/developer_guides/workflow_run) |
|  | [执行工作流（流式响应）](/developer_guides/workflow_stream_run) |
|  | [执行对话流](/developer_guides/workflow_chat) |
|  | [恢复运行工作流](/developer_guides/workflow_resume) |
###  添加 Header 参数
在 API 的 Header 中添加如下参数。
| **参数** | **取值** | **说明** |
| --- | --- | --- |
|  X-Coze-DeviceInfo <br>  | JSON Marshal of [DeviceInfo](https://bytedance.larkoffice.com/docx/MZOWder6NoIX4HxJxEBclx6Onzh#share-Z0Sfd5rIMoHIncxgzDmcTikinjf) | 用于传递设备相关信息，扣子平台基于该部分信息对设备做用量管控以及账单记录。 |
#### DeviceInfo
| **参数** | **类型** | **示例** | **说明** |
| --- | --- | --- | --- |
| device_id | String | SN12345***** | IoT 等硬件设备 ID，一个设备对应一个唯一的设备号。 |
| custom_consumer <br>  | String <br>  | 1334567**** <br>  | 设备的自定义用户标识，你可以根据业务需要进行设置，例如用户名等。 <br> * `custom_consumer `参数用于用量管控，与对话等 API 传入的 `user_id` 无关，`user_id` 主要用于上下文、数据库隔离等场景。 <br> * 出于数据隐私及信息安全等方面的考虑，不建议使用业务系统中定义的用户敏感标识（如手机号等）作为 `custom_consumer` 的值。 <br>  |
#### 示例
以下是调用创建房间 API 时，使用 API Header 传递设备信息的示例：
```Shell
curl --location --request POST 'https://api.coze.cn/v1/audio/rooms' \ 
--header 'Authorization: Bearer pat_OYDacMzM3WyOWV3Dtj2bHRMymzxP****' \ 
--header 'Content-Type: application/json' \ 
--header 'X-Coze-DeviceInfo: {"device_id": "SN123456****", "custom_consumer": "1334567****"}' \
--data-raw '{ 
    "bot_id": "734829333445931****" 
}'
```

