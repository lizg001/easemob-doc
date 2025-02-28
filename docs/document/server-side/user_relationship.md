# 管理用户关系

<Toc />

环信即时通讯 IM 支持通过 RESTful API 管理用户之间的关系，包括添加和移除联系人以及将用户添加至或移除黑名单。

## 前提条件

要调用环信即时通讯 RESTful API，请确保满足以下要求：

- 已在环信即时通讯控制台 [开通配置环信即时通讯 IM 服务](enable_and_configure_IM.html)。
- 了解环信 IM REST API 的调用频率限制，详见 [接口频率限制](limitationapi.html)。

## 认证方式

环信即时通讯 IM RESTful API 要求 Bearer HTTP 认证。每次发送 HTTP 请求时，都必须在请求头部填入如下 `Authorization` 字段：

Authorization：`Bearer ${YourAppToken}`

为提高项目的安全性，环信使用 Token（动态密钥）对即将登录即时通讯系统的用户进行鉴权。即时通讯 RESTful API 推荐使用 app token 的鉴权方式，详见 [使用 App Token 鉴权](easemob_app_token.html)。

## 公共参数

### 请求参数

| 参数    | 类型   | 是否必需 | 描述         |
| :------------ | :----- | :------ | :---------------- |
| `host`| String | 是    | 环信即时通讯 IM 分配的用于访问 RESTful API 的域名。详见 [获取环信即时通讯 IM 的信息](enable_and_configure_IM.html#获取环信即时通讯-im-的信息)。|
| `org_name` | String | 是     | 环信即时通讯 IM 为每个公司（组织）分配的唯一标识。详见 [获取环信即时通讯 IM 的信息](enable_and_configure_IM.html#获取环信即时通讯-im-的信息)。  |
| `app_name` | String | 是    | 你在环信即时通讯云控制台创建应用时填入的应用名称。详见 [获取环信即时通讯 IM 的信息](enable_and_configure_IM.html#获取环信即时通讯-im-的信息)。|
| `username` | String | 是    | 用户 ID。         |

### 响应参数

| 参数        | 类型      | 描述                                                         |
| :----------| :------- | :------------------------------------------- |
| `entities`  | Object    | 响应实体。                     |
| `data`      | Object  | 实际获取的数据详情。                                         |
| `uuid`      | String  | 用户在系统内的唯一标识。该标识由系统生成，开发者无需关心。   |
| `username`  | String| 用户 ID。                                                     |
| `action`   | String  | 请求方法。                                     |
| `organization`   | String   | 环信即时通讯 IM 为每个公司（组织）分配的唯一标识，与请求参数 `org_name` 相同。 |
| `application`  | String | 应用在系统内的唯一标识。该标识由系统生成，开发者无需关心。   |
| `applicationName` | String | 你在环信即时通讯云控制台创建应用时填入的应用名称，与请求参数 `app_name` 相同。 |
| `uri`        | String     | 请求 URL。                                                   |
| `path`        | String  | 请求路径，属于请求 URL 的一部分，开发者无需关注。            |
| `username`  | String | 用户 ID。                                                     |
| `nickname`   | String | 用户昵称。                                                   |
| `timestamp`   | Long | Unix 时间戳，单位为毫秒。                                    |
| `duration`  | String  | 请求响应时间，单位为毫秒。                                   |

用户关系管理的主要接口如下：

## 添加好友

添加好友，好友必须是和当前用户在一个 App Key 下的用户。

对于免费版即时通讯服务，单个 App Key 下的每个用户的好友数量上限为 1000，不同服务版本的 App Key 的该数量上限不同，具体可参考[版本功能介绍](https://www.easemob.com/pricing/im)。

### HTTP 请求

```http
POST https://{host}/{org_name}/{app_name}/users/{owner_username}/contacts/users/{friend_username}
```

#### 路径参数

| 参数              | 类型   | 是否必需 | 描述                                                         |
| :---------------- | :----- | :------- | :----------------------------------------------------------- |
| `owner_username`  | String | 是    | 当前用户的用户 ID。       |
| `friend_username` | String | 是    | 要添加的用户 ID。                                             |

其他参数及描述详见 [公共参数](#公共参数)。

#### 请求 header

| 参数    | 类型   |是否必需<div style="width: 80px;"></div> | 描述      |
| :-------------- | :----- | :---------------- | :------- |
| `Content-Type`  | String | 是    | 内容类型。请填 `application/json`。 |
| `Accept`   | String | 是    | 内容类型。请填 `application/json`。 |
|`Authorization`| String | 是    | 该用户或管理员的鉴权 token，格式为 `Bearer ${YourAppToken}`，其中 `Bearer` 是固定字符，后面加英文空格，再加获取到的 token 值。|

### HTTP 响应

#### 响应 body

如果返回的 HTTP 状态码为 `200`，表示请求成功，响应包体中包含以下字段：

| 字段       | 类型      | 描述                                                         |
| :---------- | :---------------- | :----------------------------------------------------------- |
| `entities`     | JSON Array    | 添加的好友的详情。                                                   |
| `entities.uuid`       | String      | 系统内为好友生成的系统内唯一标识，开发者无需关心。           |
| `entities.type`        | String     | 对象类型，值为 `user` 或 `group`。                          |
| `entities.created`       | Long   | 用户创建时间，Unix 时间戳，单位为毫秒。                      |
| `entities.modified`      | Long  | 好友的用户信息如密码或者昵称等最新修改时间，Unix 时间戳，单位为毫秒。 |
| `entities.username`   | String   | 添加的好友的用户 ID。                                             |
| `entities.activated`   | Bool  | 好友是否为正常状态：<br/> • `true` 正常状态。<br/> • `false` 已被封禁。 |
| `entities.nickname`      | String   | 好友的用户昵称。                                                   |

其他字段及描述详见 [公共参数](#公共参数)。

如果返回的 HTTP 状态码非 `200`，表示请求失败。你可以参考 [响应状态码](error.html) 了解可能的原因。

### 示例

#### 请求示例

```shell
# 将 <YourAppToken> 替换为你在服务端生成的 App Token

curl -X POST -H 'Content-Type: application/json' -H 'Accept: application/json' -H 'Authorization: Bearer <YourAppToken> ' 'http://XXXX/XXXX/XXXX/users/user1/contacts/users/user2'
```

#### 响应示例

```json
{
  "action": "post",
  "application": "8bXXXX402",
  "path": "/users/475XXXXba/contacts",
  "uri": "https://XXXX/XXXX/XXXX/users/475XXXXba/contacts",
  "entities": [
    {
      "uuid": "b2aXXXXf1",
      "type": "user",
      "created": 1542356523769,
      "modified": 1542597334500,
      "username": "user2",
      "activated": true,
      "nickname": "testuser"
    }
  ],
  "timestamp": 1542598913819,
  "duration": 63,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

## 移除好友

从用户的好友列表中移除一个用户。

### HTTP 请求

```http
DELETE https://{host}/{org_name}/{app_name}/users/{owner_username}/contacts/users/{friend_username}
```

#### 路径参数

| 参数              | 类型   | 是否必需 | 描述                                                         |
| :---------------- | :----- | :------- | :----------------------------------------------------------- |
| `owner_username`  | String | 是    | 发起操作的用户 ID。                                           |
| `friend_username` | String | 是    | 被移除好友的用户 ID。                                         |

其他参数及描述详见 [公共参数](#公共参数)。

#### 请求 header

| 参数    | 类型   |是否必需<div style="width: 80px;"></div> | 描述      |
| :-------------- | :----- | :---------------- | :------- |
| `Accept`   | String | 是    |内容类型。请填 `application/json`。 |
| `Authorization`| String | 是    |该用户或管理员的鉴权 token，格式为 `Bearer ${YourAppToken}`，其中 `Bearer` 是固定字符，后面加英文空格，再加获取到的 token 值。|

### HTTP 响应

#### 响应 body

如果返回的 HTTP 状态码为 `200`，表示请求成功，响应包体中包含以下字段：

| 字段        | 类型   | 描述                                           |
| :---------- | :------------ | :--------------------------------------------- |
| `entities` | JSON Array   | 被移除的好友的详情。                                                   |
| `entities.uuid`    | String   | 系统内为好友生成的系统内唯一标识，开发者无需关心。           |
| `entities.type`         | String    | 对象类型，值为 `user` 或 `group`。                          |
| `entities.created`     | Long    | 用户创建时间，Unix 时间戳，单位为毫秒。                      |
| `entities.modified`      | Long   | 好友的用户信息如密码或者昵称等最近一次修改时间，Unix 时间戳，单位为毫秒。 |
| `entities.username`     | String    | 被移除好友的用户 ID。                                             |
| `entities.activated`      | Bool  | 好友是否为正常状态：<ul><li>`true` 正常状态。</li><li>`false` 已被封禁。</li></ul> |
| `entities.nickname`      | String   | 好友的用户昵称。                                                   |

其他字段及描述详见 [公共参数](#公共参数)。

如果返回的 HTTP 状态码非 `200`，表示请求失败。你可以参考 [响应状态码](error.html) 了解可能的原因。

### 示例

#### 请求示例

```shell
# 将 <YourAppToken> 替换为你在服务端生成的 App Token

curl -X DELETE -H 'Accept: application/json' -H 'Authorization: Bearer <YourAppToken> ' 'http://XXXX/XXXX/XXXX/users/user1/contacts/users/user2'
```

#### 响应示例

```json
{
  "action": "delete",
  "application": "8bXXXX402",
  "path": "/users/475XXXXba/contacts",
  "uri": "https://XXXX/XXXX/XXXX/users/475XXXXba/contacts",
  "entities": [
    {
      "uuid": "b2aXXXXf1",
      "type": "user",
      "created": 1542356523769,
      "modified": 1542597334500,
      "username": "user2",
      "activated": true,
      "nickname": "testuser"
    }
  ],
  "timestamp": 1542599266616,
  "duration": 350,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

## 获取好友列表

获取指定用户的好友列表。

### HTTP 请求

```http
GET https://{host}/{org_name}/{app_name}/users/{owner_username}/contacts/users
```

#### 路径参数

| 参数             | 类型   | 是否必需 | 描述                                                         |
| :--------------- | :----- | :------- | :----------------------------------------------------------- |
| `owner_username` | String | 是    | 好友列表所有者的用户 ID。                                     |

其他参数及描述详见 [公共参数](#公共参数)。

#### 请求 header

| 参数    | 类型   |是否必需 | 描述      |
| :-------------- | :----- | :---------------- | :------- |
| `Content-Type`  | String | 是    | 内容类型。请填 `application/json`。 |
| `Accept`   | String | 是    |内容类型。请填 `application/json`。 |
| `Authorization`| String | 是    |该用户或管理员的鉴权 token，格式为 `Bearer ${YourAppToken}`，其中 `Bearer` 是固定字符，后面加英文空格，再加获取到的 token 值。|

### HTTP 响应

#### 响应 body

如果返回的 HTTP 状态码为 `200`，表示请求成功，响应包体中包含以下字段：

| 字段      | 类型   | 描述                                 |
| :-------| :-----| :----------------------------------- |
| `data`  | Array | 获取的好友列表，例如 "user1", "user2"。 |
| `count`  | Int   | 好友数量。                     |

其他字段及描述详见 [公共参数](#公共参数)。

如果返回的 HTTP 状态码非 `200`，表示请求失败。你可以参考 [响应状态码](error.html) 了解可能的原因。

### 示例

#### 请求示例

```shell
# 将 <YourAppToken> 替换为你在服务端生成的 App Token

curl -X GET 'http://XXXX/XXXX/XXXX/users/user1/contacts/users' \
-H 'Authorization: Bearer <YourAppToken> '
```

#### 响应示例

```json
{
  "action": "get",
  "uri": "http://XXXX/XXXX/XXXX/users/user1/contacts/users",
  "entities": [],
  "data": [
    "user3",
    "user2"
  ],
  "timestamp": 1543819826513,
  "duration": 12,
  "count": 2
}
```

## 添加用户至黑名单

向用户的黑名单列表中添加一个或者多个用户，黑名单中的用户无法对该用户发送消息，每个用户的黑名单人数上限为 500。

### HTTP 请求

```http
POST https://{host}/{org_name}/{app_name}/users/{owner_username}/blocks/users
```

#### 路径参数

| 参数             | 类型   | 是否必需 | 描述              |
| :--------------- | :----- | :------- | :------------------------------ |
| `owner_username` | String | 是    | 当前用户的用户 ID。           |

其他参数及描述详见[公共参数](#公共参数)。

#### 请求 header

| 参数    | 类型   | 是否必需<div style="width: 80px;"></div> | 描述      |
| :-------------- | :----- | :---------------- | :------- |
| `Content-Type`  | String | 是    | 内容类型。请填 `application/json`。 |
| `Accept`   | String | 是    |内容类型。请填 `application/json`。 |
| `Authorization`| String | 是    |该用户或管理员的鉴权 token，格式为 `Bearer ${YourAppToken}`，其中 `Bearer` 是固定字符，后面加英文空格，再加获取到的 token 值。|

#### 请求 body

| 参数        | 类型       | 是否必需 | 描述                 |
| :---------- | :--------- | :------- | :----------------------------- |
| `usernames` | Array | 是    | 要加入黑名单的用户 ID，例如 ["user1", "user2"]。 |

### HTTP 响应

#### 响应 body

如果返回的 HTTP 状态码为 `200`，表示请求成功，响应包体中包含以下字段：

| 字段    | 类型      | 描述                                                         |
| :------------ | :------------- | :---------------------------------- |
| `data`        | Array   | 添加至黑名单的用户 ID。         |

其他字段及描述详见 [公共参数](#公共参数)。

如果返回的 HTTP 状态码非 `200`，表示请求失败。你可以参考 [响应状态码](error.html) 了解可能的原因。

### 示例

#### 请求示例

```shell
# 将 <YourAppToken> 替换为你在服务端生成的 App Token

curl -X POST -H 'Content-Type: application/json' -H 'Accept: application/json' -H 'Authorization: Bearer <YourAppToken> ' -d '{
   "usernames": [
     "user2"
   ]
 }' 'http://XXXX/XXXX/XXXX/users/user1/blocks/users'
```

#### 响应示例

```json
{
  "action": "post",
  "application": "8bXXXX402",
  "uri": "https://XXXX.com/XXXX/testapp",
  "entities": [],
  "data": [
    "user2"
  ],
  "timestamp": 1542600372046,
  "duration": 11,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```

## 获取黑名单列表

获取加入黑名单的用户列表。

### HTTP 请求

```http
GET https://{host}/{org_name}/{app_name}/users/{owner_username}/blocks/users
```

#### 路径参数

| 参数             | 类型   | 是否必需 | 描述      |
| :--------------- | :----- | :------- | :---------------- |
| `owner_username` | String | 是    | 当前用户的用户 ID。                     |

其他参数及描述详见[公共参数](#公共参数)。

#### 请求 header

| 参数    | 类型   |是否必需<div style="width: 80px;"></div> | 描述      |
| :-------------- | :----- | :---------------- | :------- |
| `Accept`   | String | 是    |内容类型。请填 `application/json`。 |
|`Authorization`| String | 是    |该用户或管理员的鉴权 token，格式为 `Bearer ${YourAppToken}`，其中 `Bearer` 是固定字符，后面加英文空格，再加获取到的 token 值。|

### HTTP 响应

#### 响应 body

如果返回的 HTTP 状态码为 `200`，表示请求成功，响应包体中包含以下字段：

| 字段       | 类型    | 描述                                   |
| :---------| :------ | :------------------------------------- |
| `data` | Array | 获取的黑名单列表，例如 ["user1", "user2"]。 |
| `count`    | Int | 黑名单上用户的数量。                       |

其他字段及描述详见[公共参数](#公共参数)。

如果返回的 HTTP 状态码非 `200`，表示请求失败。你可以参考 [响应状态码](error.html) 了解可能的原因。

### 示例

#### 请求示例

```shell
# 将 <YourAppToken> 替换为你在服务端生成的 App Token

curl -X GET -H 'Accept: application/json' -H 'Authorization: Bearer <YourAppToken> ' 'http://XXXX/XXXX/XXXX/users/user1/blocks/users'
```

#### 响应示例

```json
{
  "action": "get",
  "uri": "http://XXXX/XXXX/XXXX/users/user1/blocks/users",
  "entities": [],
  "data": [
    "user2"
  ],
  "timestamp": 1542599978751,
  "duration": 4,
  "count": 1
}
```

## 从黑名单中移除用户

从用户的黑名单中移除用户：
- 将好友从黑名单中移除后，恢复好友关系，可以正常收发消息；
- 将非好友从黑名单中移除后，恢复到未添加好友的状态。
### HTTP 请求

```http
DELETE https://{host}/{org_name}/{app_name}/users/{owner_username}/blocks/users/{blocked_username}
```

#### 路径参数

| 参数               | 类型   | 是否必需 | 描述              |
| :----------------- | :----- | :------- | :------------------------ |
| `owner_username`   | String | 是    | 当前用户的用户 ID。                 |
| `blocked_username` | String | 是    | 要移出黑名单的用户 ID。                |

其他参数及描述详见 [公共参数](#公共参数)。

#### 请求 header

| 参数    | 类型   |是否必需<div style="width: 80px;"></div> | 描述      |
| :-------------- | :----- | :---------------- | :------- |
| `Accept`   | String | 是    |内容类型。请填 `application/json`。 |
|`Authorization`| String | 是    |该用户或管理员的鉴权 token，格式为 `Bearer ${YourAppToken}`，其中 `Bearer` 是固定字符，后面加英文空格，再加获取到的 token 值。|

### HTTP 响应

#### 响应 body

如果返回的 HTTP 状态码为 `200`，表示请求成功，响应包体中包含以下字段：

| 参数           | 类型   | 描述                                                         |
| :----------- | :------ | :---------------------------------------------- |
| `entities`       | JSON Array  | 从黑名单中移除的用户的详细信息。                             |
| `entities.uuid`        | String     | 用户在系统内的唯一标识。系统自动生成，开发者无需关心。       |
| `entities.type`          | String   | 对象类型，值为 `user`。         |
| `entities.created`         | Long  | 用户创建时间，Unix 时间戳，单位为毫秒。                      |
| `entities.modified`       | Long     | 用户信息如密码或昵称等的最新修改时间，Unix 时间戳，单位为毫秒。 |
| `entities.username`       | String  | 被移出黑名单的用户 ID。         |
| `entities.activated`      | Bool   | 用户是否为正常状态：<br/> • `true` 该用户为正常状态。<br/> • `false` 该用户为封禁状态。 |
| `entities.nickname`      | String   | 被移出黑名单的用户的昵称。               |

其他字段及描述详见[公共参数](#公共参数)。

如果返回的 HTTP 状态码非 `200`，表示请求失败。你可以参考 [响应状态码](error.html) 了解可能的原因。

### 示例

#### 请求示例

```shell
# 将 <YourAppToken> 替换为你在服务端生成的 App Token

curl -X DELETE -H 'Accept: application/json' -H 'Authorization: Bearer <YourAppToken> ' 'http://XXXX/XXXX/XXXX/users/user1/blocks/users/user2'
```

#### 响应示例

```json
{
  "action": "delete",
  "application": "8bXXXX402",
  "path": "/users/475XXXXba/blocks",
  "uri": "https://XXXX/XXXX/XXXX/users/475XXXXba/blocks",
  "entities": [
    {
      "uuid": "b2aXXXXf1",
      "type": "user",
      "created": 1542356523769,
      "modified": 1542597334500,
      "username": "user2",
      "activated": true,
      "nickname": "testuser"
    }
  ],
  "timestamp": 1542600712985,
  "duration": 20,
  "organization": "XXXX",
  "applicationName": "testapp"
}
```