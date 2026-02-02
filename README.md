# Telegram Voice Group Skill for OpenClaw

向指定 Telegram 群组发送语音消息的 OpenClaw 技能。

## 功能

- 使用 Microsoft Edge-TTS 生成高质量中文语音
- 转换为 Telegram 语音气泡兼容格式
- 发送到指定的群组会话
- **话题独立上下文**：每个 Telegram 话题都有独立的会话上下文，格式为 `agent:main:telegram:group:{groupId}:topic:{threadId}`
- **会话隔离**：不同话题间的对话历史和上下文完全隔离，互不干扰，这使得 Telegram 群组话题功能可以有效替代 Discord 频道的组织功能
- 支持向特定话题发送语音消息

## 安装

```bash
openclaw skills install telegram-voice-group
```

## 依赖

- `edge-tts` - 用于生成语音
- `ffmpeg` - 用于格式转换

## 使用方法

### 1. 直接在会话中使用此功能：

"向 {群组会话键} 发送语音: {语音内容}"

例如:
"向 agent:main:telegram:group:[GROUP_ID] 发送语音: 大家好，我是[ASSISTANT_NAME]，[NICKNAME]的AI助理。很高兴在[MOLTBOT_COMMUNITY]社区与大家见面，祝大家交流愉快！"

### 2. 通过 sessions_spawn 调用：

```js
await sessions_spawn({
  task: "向 agent:main:telegram:group:[GROUP_ID] 发送语音: 这是一条测试消息",
  agentId: "telegram-voice-group"
})
```

### 3. 直接调用函数（在JS环境中）：

```js
const { sendVoiceToTelegramGroup } = require('./index.js');
await sendVoiceToTelegramGroup({
  text: "语音内容",
  groupId: "agent:main:telegram:group:[GROUP_ID]",
  voice: "zh-CN-XiaoxiaoNeural",
  rate: "+5%"
});
```

## 技术规范

- 生成 MP3 格式临时音频
- 使用 FFmpeg 转换为 Telegram 兼容的 OGG Opus 格式
- 音频参数：libopus编码，48k比特率，单声道，48kHz采样率，VOIP应用类型
- 使用 `asVoice: true` 参数确保以语音气泡形式发送，而非文件
- 自动清理临时文件
- 文本格式清洗：自动移除 Markdown 标记、URL 链接和特殊符号，避免朗读出标记符号

## 许可证

MIT License