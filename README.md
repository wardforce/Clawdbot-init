# Clawdbot (Moltbot、Openclawd) 部署与配置全指南

clawdbot最近非常火爆，我去研究了大概2天才彻底搞懂应该怎么快、不折腾同时具有一定自由度的的部署，今天我就教大家如何部署这个项目。

## 1. 快速部署 (Zeabur)

如果你还没有搭建 Clawdbot，可以使用 Zeabur 进行快速部署，这个是我研究出来方便部署还能自己搞一搞插件skils的。

1. 访问 [Zeabur](https://zeabur.com/)，这个平台部署非常方便。
   ![Zeabur主页](%E5%A6%82%E4%BD%95%E6%90%9E%E5%AE%9Amoltbot%E6%A8%A1%E5%9E%8B.assets/image-20260131123352318.png)

2. 点击创建项目，选择你要的服务器。关于服务器方面，我推荐专用服务器2核4g内存的，最好位置在日本或者新加坡（2$/m对我来说还是很便宜的，可以接受的水平），不然会很卡。国内的服务器会影响搜索质量，不能使用telegram，网络方面建议run出去。~~当然你也可以使用shellcrash给国内服务器翻出去。但是不稳定，很折腾，不建议使用这个方法。~~
   ![创建项目](%E5%A6%82%E4%BD%95%E6%90%9E%E5%AE%9Amoltbot%E6%A8%A1%E5%9E%8B.assets/image-20260131123419584.png)

3. 在选择模板环节，直接搜索并选择 `clawdbot`或者`Moltbot` 模板即可。网页部分很简单的，进去一看就会，因为他帮你搞好了。**注意不要暴露网址和token！！！！！**

---

## 2. 基础配置方法

Moltbot 支持多种模型服务商（Alibaba Cloud, OpenAI, Anthropic, Google 等）。

### 2.1 交互式配置（推荐）
最简单的配置方式是使用向导：
```bash
moltbot configure
```

### 2.2 配置文件路径
如果需要手动修改，配置文件通常位于：`~/.clawdbot/config.json`

查看当前配置：
```bash
moltbot gateway config.get
```

---

## 3. 特定服务商实战指南（我最常用的服务提供商）

### 3.1 Minimax 配置
**海外版设置：**
```bash
clawdbot config set models.providers.minimax.baseUrl "https://api.minimaxi.com/anthropic"
```

**国内版设置：**
```bash
clawdbot config set models.providers.minimax.baseUrl "https://api.minimaxi.io/anthropic"
```

**设置 API Key：**
```bash
clawdbot config set models.providers.minimax.apiKey "你的MINIMAX_API_KEY"
```

### 3.2 Google Antigravity 配置
适用于 Gemini 系列及 Claude Opus/Sonnet Thinking 模型。

1. **启用插件**
   ```bash
   moltbot plugins enable google-antigravity-auth
   ```

2. **登录认证**
   ```bash
   moltbot models auth login --provider google-antigravity
   ```

3. **模型类型**
   支持的模型包括：`gemini-3-flash`, `gemini-3-pro-high`, `gemini-3-pro-low`, `claude-sonnet-4-5-thinking`, `gemini-2.5-flash`, `gemini-2.5-flash-thinking`

4. **设置默认模型**
   - 临时切换（telegram对话）：
     ```bash
     /model google-antigravity/claude-sonnet-4-5
     ```
   - 永久设置默认：
     ```bash
     moltbot models set google-antigravity/claude-opus-4-5-thinking
     ```

### 3.3 自定义 OpenAI 兼容服务商
适用于 Ollama、LM Studio 等本地或兼容服务。

在配置文件的 `models.providers` 中添加：
```json
{
  "models": {
    "providers": {
      "my-ollama": {
        "baseUrl": "http://localhost:11434/v1",
        "apiKey": "ollama",
        "auth": "api-key",
        "api": "openai-completions",
        "models": [
          {
            "id": "llama3",
            "name": "Llama 3",
            "api": "openai-completions",
            "input": ["text"],
            "contextWindow": 8192,
            "maxTokens": 2048
          }
        ]
      }
    }
  }
}
```

---

## 4. 应用集成

### Telegram 机器人
直接运行配置命令选择 Telegram 平台即可：（里面的指导堪称喂饭了，这都不会你是真完蛋了，建议重修9年义务教育。telegram你可以用telegramx来注册）
```bash
clawbot configure
```
![Telegram配置](%E5%A6%82%E4%BD%95%E6%90%9E%E5%AE%9Amoltbot%E6%A8%A1%E5%9E%8B.assets/image-20260131121750291.png)

---

## 5. 高级配置详解

### 5.1 自定义服务商通用模板
```json
"your-provider-name": {
  "baseUrl": "https://api.your-provider.com/v1",
  "apiKey": "your-api-key",
  "auth": "api-key",
  "api": "openai-completions",
  "models": [...]
}
```

**关键参数说明：**
- `baseUrl`: API 基础地址
- `api`: 接口类型（`openai-completions`, `anthropic-messages` 等）
- `contextWindow`: 上下文 token 长度
- `input`: 支持类型（`["text"]` 或 `["text", "image"]`）

### 5.2 多模型与备用策略
配置主模型故障时的备用模型：
```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "my-provider/powerful-model",
        "fallbacks": ["my-provider/fast-model", "alibaba-cloud/qwen-max"]
      }
    }
  }
}
```

---

## 6. 运维与故障排除

### 常用命令
- **应用配置**：`moltbot gateway config.apply`
- **重启服务**：`moltbot gateway restart`
- **查看状态**：`moltbot status` 或聊天中输入 `/status`
- **列出模型**：`moltbot models list`
- **查看日志**：`moltbot logs --follow`

### 常见问题
1. **模型未显示**：检查 `id` 和 `defaults.model.primary` 是否匹配。
2. **API 连接失败**：检查 `baseUrl` 和网络连通性。
3. **安全提示**：建议使用环境变量存储 API Key，不要直接写在配置文件中分享给他人。
