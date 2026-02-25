# OpenClaw WeCom Channel 插件

企业微信 (WeCom/WxWork) 消息通道插件，让 [OpenClaw](https://github.com/openclaw/openclaw) AI Agent 通过企业微信收发消息。

## ✨ 功能

- 📩 **接收消息** — 企业微信用户发送文本消息，AI Agent 自动回复
- 📤 **主动推送** — Agent 通过企业微信 API 主动发送消息给用户
- 🔐 **消息加解密** — 完整实现企业微信 AES-256-CBC 消息加解密
- 🔑 **Token 管理** — access_token 自动缓存 + 过期刷新
- 🛡️ **安全策略** — 支持 open / pairing / allowlist 三种访问控制模式

## 📦 安装

### 方式一：本地安装（推荐）

```bash
# 克隆到 OpenClaw extensions 目录
git clone git@github.com:darrryZ/openclaw-wecom-channel.git ~/.openclaw/extensions/wecom
```

### 方式二：复制文件

```bash
mkdir -p ~/.openclaw/extensions/wecom
cp -r . ~/.openclaw/extensions/wecom/
```

## ⚙️ 配置

在 `~/.openclaw/openclaw.json` 中添加：

```json
{
  "channels": {
    "wecom": {
      "enabled": true,
      "corpId": "你的企业ID",
      "agentId": 1000003,
      "secret": "应用Secret",
      "token": "回调Token",
      "encodingAESKey": "回调EncodingAESKey",
      "port": 18800,
      "dmPolicy": "open"
    }
  },
  "plugins": {
    "entries": {
      "wecom": {
        "enabled": true
      }
    }
  }
}
```

### 配置项说明

| 参数 | 必填 | 说明 |
|------|------|------|
| `corpId` | ✅ | 企业微信企业 ID |
| `agentId` | ✅ | 自建应用 Agent ID |
| `secret` | ✅ | 自建应用 Secret |
| `token` | ✅ | 接收消息回调 Token |
| `encodingAESKey` | ✅ | 接收消息回调 EncodingAESKey |
| `port` | ✅ | 回调监听端口 |
| `dmPolicy` | ❌ | 访问策略：`open`（默认）/ `pairing` / `allowlist` |
| `allowFrom` | ❌ | 白名单用户列表（`dmPolicy: "allowlist"` 时生效） |

### 获取企业微信配置

1. 登录 [企业微信管理后台](https://work.weixin.qq.com/wework_admin/frame)
2. **企业ID**：我的企业 → 企业信息 → 企业ID
3. **创建应用**：应用管理 → 创建应用
4. **Secret**：应用详情页 → Secret
5. **回调配置**：应用详情页 → 接收消息 → 设置API接收
   - URL: `http://你的域名:端口/wecom/callback`
   - Token & EncodingAESKey: 随机生成

## 🌐 公网访问

企业微信回调需要公网可访问的 URL。推荐使用 **Cloudflare Tunnel**：

```bash
# 安装 cloudflared
brew install cloudflared

# 创建 tunnel
cloudflared tunnel create wecom-tunnel
cloudflared tunnel route dns wecom-tunnel wecom.yourdomain.com

# 启动（如使用 ClashX 等代理，需加 edge-ip-version 参数）
cloudflared tunnel run --edge-ip-version 4 --edge-bind-address 0.0.0.0 \
  --url http://localhost:18800 wecom-tunnel
```

然后在企业微信后台设置回调 URL 为 `https://wecom.yourdomain.com/wecom/callback`。

## 🏗️ 项目结构

```
├── index.ts              # 插件入口
├── package.json
├── openclaw.plugin.json
└── src/
    ├── channel.ts        # ChannelPlugin 定义（配置、路由、能力声明）
    ├── bot.ts            # HTTP 回调服务（签名验证、消息解析、分发）
    ├── crypto.ts         # AES-256-CBC 消息加解密
    ├── token.ts          # access_token 获取与缓存
    ├── send.ts           # 主动发送消息
    ├── outbound.ts       # 出站消息处理
    ├── reply-dispatcher.ts # 被动回复调度（5秒超时降级为主动推送）
    ├── accounts.ts       # 账户解析
    ├── monitor.ts        # 服务生命周期管理
    ├── runtime.ts        # 运行时上下文
    └── types.ts          # 类型定义
```

## 🔧 技术细节

- **被动回复 vs 主动推送**：企业微信要求 5 秒内被动回复，超时自动降级为主动推送
- **消息加解密**：完整实现 `WXBizMsgCrypt` 标准，AES-256-CBC + PKCS7 填充
- **Token 缓存**：access_token 缓存在内存，提前 5 分钟刷新，避免频繁请求
- **TypeScript**：全 TypeScript 编写，类型安全

## 📋 当前限制

- 仅支持**文本消息**，图片/文件/语音等待后续支持
- 仅支持**单聊**（direct），群聊待后续支持
- 不支持 reactions、threads、polls

## 📝 License

MIT

## 🔗 相关链接

- [OpenClaw](https://github.com/openclaw/openclaw) — AI Agent 框架
- [企业微信开发文档](https://developer.work.weixin.qq.com/document/)
- [OpenClaw 文档](https://docs.openclaw.ai)
