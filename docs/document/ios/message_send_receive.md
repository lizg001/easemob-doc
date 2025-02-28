# 发送和接收消息

<Toc />

登录 Chat app 后，用户可以在单聊、群聊、聊天室中发送如下类型的消息：

- 文字消息，包含超链接和表情消息。
- 附件消息，包含图片、语音、视频及文件消息。
- 位置消息。
- 透传消息。
- 自定义消息。

:::tip
对于聊天室消息，环信即时通讯提供消息分级功能，将消息的优先级划分为高、普通和低三种级别，高优先级的消息会优先送达。你可以在创建消息时对指定聊天室消息类型或指定成员的消息设置为高优先级，确保这些消息优先送达。这种方式确保在聊天室内消息并发量很大或消息发送频率过高时，重要消息能够优先送达，从而提升重要消息的可靠性。
当服务器的负载较高时，会优先丢弃低优先级的消息，将资源留给高优先级的消息。不过，消息分级功能只确保消息优先到达，并不保证必达。服务器负载过高的情况下，即使是高优先级消息依然会被丢弃。
:::

本文介绍如何使用即时通讯 IM SDK 实现发送和接收这些类型的消息。

## 技术原理

环信即时通讯 IM iOS SDK 提供 `ChatManager` 类和 `EMChatMessage` 类，支持发送、接收消息，发送、接收消息已读回执，并管理用户设备上存储的消息会话数据，其中包含如下主要方法：

- `sendMessage` 发送消息给某个用户，群组或者聊天室；
- `recallMessage` 撤回自己发出的消息；
- `addMessageListener` 添加消息接收的回调通知；
- `ackConversationRead` 发送会话已读通知；
- `ackMessageRead` 发送指定消息已读的通知；

消息的收发流程如下：

1. 用户 A 发送一条消息到环信即时通讯 IM 消息服务器。
2. 消息服务器投递消息给用户 B，用户 B 收到该消息。

![img](@static/images/android/sendandreceivemsg.png)

## 前提条件

开始前，请确保满足以下条件：

- 完成 SDK 初始化，详见 [快速开始](quickstart.html)。
- 了解环信即时通讯 IM 的使用限制，详见 [使用限制](/product/limitation.html)。

## 实现方法

### 发送文本消息

你可以利用 `EMChatMessage` 类构造一条消息，然后通过 `ChatManager` 将该消息发出。

示例代码：

```objectivec
// 创建一条文本消息，`content` 为消息文字内容，`toChatUsername` 为对方用户或者群聊的 ID，`fromChatUsername` 为发送方用户或群聊的 ID，`textMessageBody` 为消息体，`messageExt` 为消息扩展，后文皆是如此。
EMTextMessageBody *textMessageBody = [[TextMessageBody alloc] initWithText:content];
EMChatMessage *message = [[EMChatMessage alloc] initWithConversationID:toChatUsername from:fromChatUsername to:toChatUsername body:textMessageBody ext:messageExt];
// 构造消息时需设置 `EMChatMessage` 类的 `ChatType` 属性，可设置为 `EMChatTypeChat`、`EMChatTypeGroupChat` 和 `EMChatTypeChatRoom`，即单聊、群聊或聊天室消息，默认为单聊。
message.chatType = EMChatTypeChat;
// 发送消息，异步方法。
[[EMClient sharedClient].chatManager sendMessage:message progress:nil completion:nil];
// 发送消息时可以设置发送回调，获得消息发送状态。可以在该回调中更新消息的显示状态。例如消息发送失败后的提示等等。
[[EMClient sharedClient].chatManager sendMessage:message progress:nil completion:^(EMChatMessage *message, EMError *error) {
    if (!error) {
        // 发送消息成功！
    } else {
        // 发送消息失败！
    }
}];
```

对于聊天室消息，可设置消息优先级。示例代码如下：

```Objectivec
EMTextMessageBody* textBody = [[EMTextMessageBody alloc] initWithText:@"Hi"];
EMChatMessage* message = [[EMChatMessage alloc] initWithConversationID:@"roomId" body:textBody ext:nil];
message.chatType = EMChatTypeChatRoom;
// 聊天室消息的优先级。如果不设置，默认值为 `Normal`，即“普通”优先级。
message.priority = EMChatRoomMessagePriorityHigh;
[[EMClient sharedClient].chatManager sendMessage:message progress:nil completion:nil];
```

### 接收消息

你可以用注册监听 `EMChatManagerDelegate` 接收消息。

该 `EMChatManagerDelegate` 可以多次添加，请记得在不需要的时候移除 `Delegate`，如在`ViewController` `dealloc()` 时。

在新消息到来时，你会收到 `messagesDidReceive` 的回调，消息接收时可能是一条，也可能是多条。你可以在该回调里遍历消息队列，解析并显示收到的消息。

```objectivec
// 添加代理。
[[EMClient sharedClient].chatManager addDelegate:self delegateQueue:nil];

// 收到消息回调。

- (void)messagesDidReceive:(NSArray *)aMessages
  {
  // 收到消息，遍历消息列表。
  for (EMChatMessage *message in aMessages) {
    // 消息解析和展示。
  }
  }

// 移除代理。

- (void)dealloc
  {
  [[EMClient sharedClient].chatManager removeDelegate:self];
  }

```

### 撤回消息

消息撤回功能指用户可以撤回一定时间内自己发送出去的消息，消息撤回时限默认 2 分钟。如需调整，可联系商务。

```objectivec
// 异步方法
[[EMClient sharedClient].chatManager recallMessageWithMessageId:messageId completion:^(EMError *aError) {
    if (!aError) {
        NSLog(@"撤回消息成功");
    } else {
        NSLog(@"撤回消息失败的原因 --- %@", aError.errorDescription);
    }
}];
```

#### 设置消息撤回回执

```objectivec
- (void)messagesDidRecall:(NSArray *)aMessages
{
    // `aMessages` 为被撤回的消息列表
}
```

### 发送附件类型的消息

除文本消息外，还有几种其他类型的消息，其中语音，图片，短视频，文件等消息，是通过先将附件上传到消息服务器的方式实现。收到语音时，会自动下载，而图片和视频会自动下载缩略图。文件消息不会自动下载附件，接收方需调用下载附件的 API，具体实现参考下文。

#### 发送语音消息

发送语音消息时，应用层需要完成语音文件录制的功能，并给出语音文件的 URI 和附件的显示名称。

```objectivec
// `localPath` 为语音文件本地资源路径，`displayName` 为附件的显示名称。
EMVoiceMessageBody *body = [[EMVoiceMessageBody alloc] initWithLocalPath:localPath displayName:displayName];
EMChatMessage *message = [[EMChatMessage alloc] initWithConversationID:toChatUsername from:fromChatUsername to:toChatUsername body:body ext:messageExt];
message.chatType = EMChatTypeChat;
// 如果是群聊，设置 chatType，默认是单聊。
message.chatType = EMChatTypeGroupChat;
// 发送消息。
[[EMClient sharedClient].chatManager sendMessage:message progress:nil completion:nil];
```

接收方收到语音消息后，参考如下示例代码获取语音消息的附件：

```objectivec
EMVoiceMessageBody *voiceBody = (EMVoiceMessageBody *)message.body;
// 获取语音文件在服务器的地址。
NSString *voiceRemotePath = voiceBody.remotePath;
// 本地语音文件的资源路径。
NSString *voiceLocalPath = voiceBody.localPath;
```

#### 发送图片消息

```objectivec
// `imageData` 为图片本地资源，`displayName` 为附件的显示名称。
EMImageMessageBody *body = [[EMImageMessageBody alloc] initWithData:imageData displayName:displayName];
EMChatMessage *message = [[EMChatMessage alloc] initWithConversationID:toChatUsername from:fromChatUsername to:toChatUsername body:body ext:messageExt];

message.chatType = EMChatTypeChat;
// 设置消息类型，即设置 `Message` 类的 `MessageType` 属性。
// 设置该属性的值为 `Chat`、`Group` 和 `Room`，分别代表该消息是单聊、群聊或聊天室消息，默认为单聊。
message.chatType = EMChatTypeGroupChat;
// 发送消息。
[[EMClient sharedClient].chatManager sendMessage:message progress:nil completion:nil];
```

```objectivec
// 发送成功后，获取图片消息缩略图及附件。
EMImageMessageBody *body = (EMImageMessageBody *)message.body;
// 从服务器端获取图片文件。
NSString *remotePath = body.remotePath;
// 从服务器端获取图片缩略图。
NSString *thumbnailPath = body.thumbnailRemotePath;
// 从本地获取图片文件。
NSString *localPath = body.localPath;
// 从本地获取图片缩略图。
NSString *thumbnailLocalPath = body.thumbnailLocalPath;
```

接收方如果设置了自动下载，即 `[EMClient sharedClient].options.isAutoDownloadThumbnail;` 为 `true`，SDK 接收到消息后会下载缩略图；如果未设置自动下载，需主动调用 `[[EMClient sharedClient].chatManager downloadMessageThumbnail:message progress:nil completion:nil];` 下载。

下载完成后，在回调里调用相应消息 `body` 的 `thumbnailLocalPath` 获取缩略图路径。

#### 发送短视频消息

发送短视频消息时，应用层需要完成视频文件的选取或者录制。视频消息支持输入视频的首帧作为缩略图，也支持给出视频的时长作为参数，发送给接收方。

```objectivec
// `localPath` 为本地资源路径，`displayName` 为视频的显示名称。
EMVideoMessageBody *body = [[EMVideoMessageBody alloc] initWithLocalPath:localPath displayName:@"displayName"];
body.duration = duration;// 视频时长。

EMChatMessage *message = [[EMChatMessage alloc] initWithConversationID:toChatUsername from:fromChatUsername to:toChatUsername body:body ext:messageExt];
message.chatType = EMChatTypeChat;
// 如果是群聊，设置 chatType，默认是单聊。
message.chatType = EMChatTypeGroupChat;
// 发送消息。
[[EMClient sharedClient].chatManager sendMessage:message progress:nil completion:nil];
```

```objectivec
// 发送成功后，获取视频消息缩略图及附件。
EMVideoMessageBody *body = (EMVideoMessageBody *)message.body;
// 从服务器端获取视频文件的地址。
NSString *remotePath = body.remotePath;
// 从服务器端获取视频缩略图。
NSString *thumbnailPath = body.thumbnailRemotePath;
// 从本地获取视频文件。
NSString *localPath = body.localPath;
// 从本地获取视频缩略图。
NSString *thumbnailLocalPath = body.thumbnailLocalPath;
```

接收方如果设置了自动下载，即 `[EMClient sharedClient].options.isAutoDownloadThumbnail;` 为 `YES`，SDK 接收到消息后会下载缩略图；如果未设置自动下载，需主动调用 `[[EMClient sharedClient].chatManager downloadMessageThumbnail:message progress:nil completion:nil];` 下载。

下载完成后，在回调里调用相应消息 `body` 的 `thumbnailLocalPath` 获取视频缩略图路径。

#### 发送文件消息

```objectivec
// `fileData` 为本地资源，`fileName` 为附件的显示名称。
EMFileMessageBody *body = [[EMFileMessageBody alloc]initWithData:fileData displayName:fileName];
EMChatMessage *message = [[EMChatMessage alloc] initWithConversationID:toChatUsername from:fromChatUsername to:toChatUsername body:body ext:messageExt];
message.chatType = EMChatTypeChat;
// 如果是群聊，设置 `ChatType` 为 `GroupChat`，该参数默认是单聊（`Chat`）。
message.chatType = EMChatTypeGroupChat;
// 发送消息。
[[EMClient sharedClient].chatManager sendMessage:message progress:nil completion:nil];
```

发送成功后，获取文件消息附件：

```objectivec
EMFileMessageBody *body = (EMFileMessageBody *)message.body;
// 从服务器端获取文件路径。
NSString *remotePath = body.remotePath;
// 从本地获取文件路径。
NSString *localPath = body.localPath;
```

发送附件类型消息时，可以在 progress 回调中获取附件上传的进度，以百分比表示，示例代码如下：

```objectivec
// 发送消息时可以设置 `EMCallBack` 的实例，获得消息发送的状态。可以在该回调中更新消息的显示状态。例如消息发送失败后的提示等等。
[[EMClient sharedClient].chatManager sendMessage:message progress:^(int progress) {
        // progress 附件上传进度块的百分比。
} completion:^(EMChatMessage *message, EMError *error) {
    // error 发送结果，message 发送的消息。
}];
```

#### 下载缩略图及附件

图片消息和视频消息默认会生成缩略图，接收到这两类消息时默认自动下载缩略图。语音消息接收到后会自动下载。如果设置为不自动下载附件，可修改 `[[EMClient sharedClient].options setIsAutoDownloadThumbnail:NO];`， 需主动调用 `[[EMClient sharedClient].chatManager downloadMessageThumbnail:message progress:nil completion:nil];` 下载。 下载完成后，调用相应消息体的 `thumbnailLocalPath` 获取缩略图路径。

```objectivec
EMImageMessageBody *body = (EMImageMessageBody *)message.body;
// 从服务器端获取图片缩略图。
NSString *thumbnailPath = body.thumbnailRemotePath;
// 从本地获取图片缩略图。
NSString *thumbnailLocalPath = body.thumbnailLocalPath;
```

下载附件

下载附件的方法为：`[[EMClient sharedClient].chatManager downloadMessageThumbnail:message progress:nil completion:nil]; 下载完成后，调用相应消息 body 的`LocalPath()` 去获取附件路径。例如：

```objectivec
EMImageMessageBody *body = (EMImageMessageBody *)message.body;
// 从本地获取图片文件。
NSString *localPath = body.localPath;
```

### 发送位置消息

当你需要发送位置时，需要集成第三方的地图服务，获取到位置点的经纬度信息。接收方接收到位置消息时，需要将该位置的经纬度，借由第三方的地图服务，将位置在地图上显示出来。

```objectivec
// `latitude` 为纬度，`longitude` 为经度，`address` 为具体位置内容。
EMLocationMessageBody *body = [[EMLocationMessageBody alloc] initWithLatitude:latitude longitude:longitude address:aAddress];
EMChatMessage *message = [[EMChatMessage alloc] initWithConversationID:toChatUsername from:fromChatUsername to:toChatUsername body:body ext:messageExt];
message.chatType = EMChatTypeChat;
// 如果是群聊，设置 chatType，默认是单聊。
message.chatType = EMChatTypeGroupChat;
// 发送消息。
[[EMClient sharedClient].chatManager sendMessage:message progress:nil completion:nil];
```

### 发送透传消息

可以把透传消息理解为一条指令，通过发送这条指令给对方，通知对方要执行的操作，收到消息可以自定义处理。（透传消息不会存入本地数据库中，所以在 UI 上不会显示）。另外，以 “em\_” 和 “easemob::” 开头的 `action` 为内部保留字段，注意不要使用。

透传消息适用于更新头像、更新昵称等场景。

```objectivec
// `action` 自定义 `NSString` 类型的命令内容。
EMCmdMessageBody *body = [[EMCmdMessageBody alloc] initWithAction:action];
    EMChatMessage *message = [[EMChatMessage alloc] initWithConversationID:toChatUsername from:fromChatUsername to:toChatUsername body:body ext:messageExt];
    // 支持单聊，群聊和聊天室，默认为单聊。
    // 若为群聊，添加下行代码。
    message.chatType = EMChatTypeGroupChat;
    // 若为聊天室，添加下行代码。
    //message.chatType = EMChatTypeChatRoom;
    // 发送消息。
    [[EMClient sharedClient].chatManager sendMessage:message progress:nil completion:nil];
```

请注意透传消息的接收方，也是由单独的回调进行通知，方便用户进行不同的处理。

```objectivec
// 收到透传消息。
- (void)cmdMessagesDidReceive:(NSArray *)aCmdMessages{
  for (EMChatMessage *message in aCmdMessages) {
        EMCmdMessageBody *body = (EMCmdMessageBody *)message.body;
        // 进行透传消息 body 解析。
    }
  }
```

#### 通过透传消息实现输入指示器

输入指示器显示其他用户何时输入消息。通过该功能，用户之间可进行有效沟通，增加了用户对聊天应用中交互的期待感。

你可以通过透传消息实现输入指示器。

下图为输入指示器的工作原理。

![img](@static/images/common/typing_indicator.png)

监听用户 A 的输入状态。一旦有文本输入，通过透传消息将输入状态发送给用户 B，用户 B 收到该消息，了解到用户 A 正在输入文本。

- 用户 A 向用户 B 发送消息，通知其开始输入文本。
- 收到消息后，如果用户 B 与用户 A 的聊天页面处于打开状态，则显示用户 A 的输入指示器。
- 如果用户 B 在几秒后未收到用户 A 的输入，则自动取消输入指示器。

:::notice
用户 A 可根据需要设置透传消息发送间隔。
:::

以下示例代码展示如何发送输入状态的透传消息。

```objectivec
//发送表示正在输入的透传消息
#define MSG_TYPING_BEGIN @"TypingBegin"

- (void)textViewDidChange:(UITextView *)textView
{
    long long currentTimestamp = [self getCurrentTimestamp];
    // 5 秒内不能重复发送消息
    if ((currentTimestamp - _previousChangedTimeStamp) > 5) {
        // 发送开始输入的透传消息
        [self _sendBeginTyping];
        _previousChangedTimeStamp = currentTimestamp;
    }
}

- (void)_sendBeginTyping
{
    EMCmdMessageBody *body = [[EMCmdMessageBody alloc] initWithAction:MSG_TYPING_BEGIN];
    body.isDeliverOnlineOnly = YES;
    EMChatMessage *message = [[EMChatMessage alloc] initWithConversationID:conversationId body:body ext:nil];
    [[EMClient sharedClient].chatManager sendMessage:message progress:nil completion:nil];
}

```

以下示例代码展示如何接受和解析输入状态的透传消息。

```objectivec
#define TypingTimerCountNum 10
- (void)cmdMessagesDidReceive:(NSArray *)aCmdMessages
{
    NSString *conId = self.currentConversation.conversationId;
    for (EMChatMessage *message in aCmdMessages) {
        if (![conId isEqualToString:message.conversationId]) {
            continue;
        }
        EMCmdMessageBody *body = (EMCmdMessageBody *)message.body;
        // 收到正在输入的透传消息
        if ([body.action isEqualToString:MSG_TYPING_BEGIN]) {
            if (_receiveTypingCountDownNum == 0) {
                [self startReceiveTypingTimer];
            }else {
                _receiveTypingCountDownNum = TypingTimerCountNum;
            }
        }

    }
}

- (void)startReceiveTypingTimer {
    [self stopReceiveTypingTimer];
    _receiveTypingCountDownNum = TypingTimerCountNum;
    _receiveTypingTimer = [NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(startReceiveCountDown) userInfo:nil repeats:YES];

    [[NSRunLoop currentRunLoop] addTimer:_receiveTypingTimer forMode:UITrackingRunLoopMode];
    [_receiveTypingTimer fire];
    // 这里需更新 UI，显示“对方正在输入”
}

- (void)startReceiveCountDown
{
    if (_receiveTypingCountDownNum == 0) {
        [self stopReceiveTypingTimer];
        // 这里需更新 UI，不再显示“对方正在输入”

        return;
    }
    _receiveTypingCountDownNum--;
}

- (void)stopReceiveTypingTimer {
    _receiveTypingCountDownNum = 0;
    if (_receiveTypingTimer) {
        [_receiveTypingTimer invalidate];
        _receiveTypingTimer = nil;
    }
}
```

### 发送自定义类型消息

除了几种消息之外，你可以自己定义消息类型，方便业务处理，即首先设置一个消息类型名称，然后可添加多种自定义消息。自定义消息内容是 key，value 格式，你需要自己添加并解析该内容。

```objectivec
// event 为需要传递的自定义消息事件，比如名片消息，可以设置 "userCard"；`ext` 为事件扩展字段，比如可以设置 `uid`，`nickname`，`avatar`。
EMCustomMessageBody* body = [[EMCustomMessageBody alloc] initWithEvent:@"userCard" ext:@{@"uid":aUid ,@"nickname":aNickName,@"avatar":aUrl}];
EMChatMessage *message = [[EMChatMessage alloc] initWithConversationID:toChatUsername from:fromChatUsername to:toChatUsername body:body ext:messageExt];
message.chatType = EMChatTypeChat;
// 如果是群聊，设置 chatType，默认是单聊。
message.chatType = EMChatTypeGroupChat;
// 发送消息。
[[EMClient sharedClient].chatManager sendMessage:message progress:nil completion:nil];
```

### 使用消息的扩展字段

当 SDK 提供的消息类型不满足需求时，你可以通过消息扩展字段来传递自定义的内容，从而生成自己需要的消息类型。

当目前消息类型不满足用户需求时，可以在扩展部分保存更多信息，例如消息中需要携带被回复的消息内容或者是图文消息等场景。

```objectivec
EMTextMessageBody *textMessageBody = [[EMTextMessageBody alloc] initWithText:content];
// 增加自定义属性。
NSDictionary *messageExt = @{@"attribute":@"value"};
EMChatMessage *message = [[EMChatMessage alloc] initWithConversationID:toChatUsername from:fromChatUsername to:toChatUsername body:textMessageBody ext:messageExt];
message.chatType = EMChatTypeChat;
// 发送消息。
[[EMClient sharedClient].chatManager sendMessage:message progress:nil completion:nil];

// 接收消息的时候获取扩展属性。
- (void)messagesDidReceive:(NSArray *)aMessages
  {
  // 收到消息，遍历消息列表。
  for (EMChatMessage *message in aMessages) {
     // value 为消息扩展里 attribute 字段的值。
     NSString *value = [message.ext objectForKey:@"attribute"];  }
  }
```
