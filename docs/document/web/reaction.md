# 消息表情回复 Reaction

<Toc />

环信即时通讯 IM 提供消息表情回复（下文统称 “Reaction”）功能。用户可以在单聊和群聊中对消息添加、删除表情。表情可以直观地表达情绪，利用 Reaction 可以提升用户的使用体验。同时在群组中，利用 Reaction 可以发起投票，根据不同表情的追加数量确认投票。

:::notice
1. 目前 Reaction 仅适用于单聊和群组。聊天室暂不支持 Reaction 功能。
2. 私有化版本不支持 Reaction 功能。
:::

## 技术原理

环信即时通讯 IM SDK 支持你通过调用 API 在项目中实现如下功能：

- `addReaction` 在消息上添加 Reaction；
- `deleteReaction` 删除消息的 Reaction；
- `getReactionList` 获取消息的 Reaction 列表；
- `getReactionDetail` 获取 Reaction 详情；
- `fetchHistoryMessages` 获取漫游消息中的 Reaction。

添加 Reaction：

![](@static/images/web/web_chat_reaction_add_reaction.png)
查看 Reaction：

![](@static/images/web/web_group_chat_reaction_detail_another_version.png)

## 前提条件

开始前，请确保满足以下条件：

1. 完成 `4.0.5 及以上版本` SDK 初始化，详见 [快速开始](quickstart.html)。
2. 了解环信即时通讯 IM API 的 [使用限制](/product/limitation.html)。
3. 已联系商务开通 Reaction 功能。

## 实现方法

### 在消息上添加 Reaction

调用 `addReaction` 在消息上添加 Reaction，在 `onReactionChange` 监听事件中会收到这条消息的最新 Reaction 概览。

示例代码如下：

```javascript
// 添加 Reaction。
conn.addReaction({ messageId: "messageId", reaction: "reaction" });

// 监听 Reaction 更新。
conn.addEventHandler("REACTION", {
  onReactionChange: (reactionMsg) => {
    console.log(reactionMsg);
  },
});
```

### 删除消息的 Reaction

调用 `deleteReaction` 删除消息的 Reaction，在 `onReactionChange` 监听事件中会收到这条消息的最新 Reaction 概览。

示例代码如下：

```javascript
// 删除 Reaction。
conn.deleteReaction({ messageId: "messageId", reaction: "reaction" });

// 监听 Reaction 更新。
conn.addEventHandler("REACTION", {
  onReactionChange: (reactionMsg) => {
    console.log(reactionMsg);
  },
});
```

### 获取消息的 Reaction 列表

调用 `getReactionList` 方法可以从服务器获取 Reaction 概览列表，列表内容包含 Reaction 内容，添加或移除 Reaction 的用户数量，以及添加或移除 Reaction 的前三个用户的用户 ID。示例代码如下：

```javascript
conn
  .getReactionList({ chatType: "singleChat", messageId: "messageId" })
  .then((res) => {
    console.log(res);
  });
```

### 获取 Reaction 详情

调用 `getReactionDetail` 方法可以从服务器获取 Reaction 详情，包括 Reaction 内容，添加或移除 Reaction 的用户数量以及添加或移除 Reaction 的全部用户列表。示例代码如下：

```javascript
conn
  .getReactionDetail({
    messageId: "messageId",
    reaction: "reaction",
    cursor: null,
    pageSize: 20,
  })
  .then((res) => {
    console.log(res);
  });
```

### 获取漫游消息中的 Reaction

调用 `fetchHistoryMessages` 可以获取漫游消息，如果一条消息已添加 Reaction，消息体中会包含 Reaction 概览。

示例代码如下：

```javascript
conn.fetchHistoryMessages({ queue: "user", count: 20 }).then((messages) => {
  console.log(messages);
});
```