---
name: telegram-voice-group
description: 向指定 Telegram 群组发送语音消息
metadata: {"openclaw":{"emoji":"🔊","os":["linux"],"requires":{"bins":["ffmpeg","edge-tts"]}}}
---

# Telegram 群组语音消息发送 (telegram-voice-group) 🔊

向指定 Telegram 群组发送语音消息。

## 功能

- 使用 Microsoft Edge-TTS 生成高质量中文语音
- 转换为 Telegram 语音气泡兼容格式
- 发送到指定的群组会话
- **话题独立上下文**：每个 Telegram 话题都有独立的会话上下文，格式为 `agent:main:telegram:group:{groupId}:topic:{threadId}`
- **会话隔离**：不同话题间的对话历史和上下文完全隔离，互不干扰，这使得 Telegram 群组话题功能可以有效替代 Discord 频道的组织功能
- 支持向特定话题发送语音消息

## 依赖

- `edge-tts` - 用于生成语音
- `ffmpeg` - 用于格式转换

## 使用方法

### 1. 直接在会话中使用此功能：

"向 {群组会话键} 发送语音: {语音内容}"

例如:
"向 agent:main:telegram:group:-1003847310728 发送语音: 大家好，我是阿威，威哥的AI助理。很高兴在Moltflow社区与大家见面，祝大家交流愉快！"

### 2. 通过 sessions_spawn 调用：

```js
await sessions_spawn({
  task: "向 agent:main:telegram:group:-1003847310728 发送语音: 这是一条测试消息",
  agentId: "telegram-voice-group"
})
```

### 3. 直接调用函数（在JS环境中）：

```js
const { sendVoiceToTelegramGroup } = require('./index.js');
await sendVoiceToTelegramGroup({
  text: "语音内容",
  groupId: "agent:main:telegram:group:-1003847310728",
  voice: "zh-CN-XiaoxiaoNeural",
  rate: "+5%"
});
```

### 4. 向特定话题发送语音消息：

可通过 threadId 参数向特定话题发送语音消息：

```js
const { message } = require('@openclaw/core');

await message({
  action: 'send',
  channel: 'telegram',
  to: '-1003847310728',
  message: '语音消息内容',
  asVoice: true,
  media: '语音文件路径',
  threadId: 14  // 指定话题ID
});
```

### 5. 使用函数发送到指定话题：

```js
const { sendVoiceToTelegramGroup } = require('./index.js');
await sendVoiceToTelegramGroup({
  text: "语音内容",
  groupId: "agent:main:telegram:group:-1003847310728",
  voice: "zh-CN-XiaoxiaoNeural",
  rate: "+5%",
  threadId: 14  // 可选：指定话题ID
});
```

## 实现逻辑

当检测到用户请求向群组发送语音消息时，系统将自动执行以下步骤：

1. 使用 edge-tts 生成语音文件
   ```bash
   edge-tts --voice zh-CN-XiaoxiaoNeural --rate=+5% --text "语音内容" --write-media /tmp/voice_msg.mp3
   ```

2. 使用 ffmpeg 转换为 Telegram 兼容格式
   ```bash
   ffmpeg -y -i /tmp/voice_msg.mp3 -c:a libopus -b:a 48k -ac 1 -ar 48000 -application voip /tmp/voice_msg.ogg
   ```

3. 使用 sessions_send 发送语音文件到指定群组
   ```bash
   sessions_send sessionKey "MEDIA:/tmp/voice_msg.ogg"
   ```

## 参数

- 语音内容: 要转换为语音的文本
- 群组会话键: 目标群组的完整会话键

## 技术规范

- 生成 MP3 格式临时音频
- 使用 FFmpeg 转换为 Telegram 兼容的 OGG Opus 格式
- 音频参数：libopus编码，48k比特率，单声道，48kHz采样率，VOIP应用类型
- 使用 `asVoice: true` 参数确保以语音气泡形式发送，而非文件
- 自动清理临时文件
- 文本格式清洗：自动移除 Markdown 标记、URL 链接和特殊符号，避免朗读出标记符号

## 文本格式清洗

为避免朗读出标记符号，技能会自动清洗文本内容：

| 需移除 | 示例 |
|--------|------|
| Markdown 标记 | `**加粗**`、`` `代码` ``、`# 标题` |
| URL 链接 | `https://example.com` |
| 特殊符号 | `---`、`***`、`>>>` |

## 注意事项

- 机器人必须已在目标群组中
- 需要相应的消息发送权限
- 语音内容应适合群组环境
- 清洗后的文本将用于语音生成，确保朗读效果
- 每个 Telegram 话题都有独立的会话上下文，格式为 `agent:main:telegram:group:{groupId}:topic:{threadId}`
- 不同话题间的对话历史和上下文完全隔离，互不干扰，这使得 Telegram 群组话题功能可以有效替代 Discord 频道的组织功能