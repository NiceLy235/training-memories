# OpenClaw 飞书集成完整指南

本文档记录了将 OpenClaw 集成到飞书群聊的完整过程，包括遇到的问题和解决方案。

---

## 目录

- [环境信息](#环境信息)
- [第一步：安装飞书插件](#第一步安装飞书插件)
- [第二步：创建飞书应用](#第二步创建飞书应用)
- [第三步：配置 OpenClaw](#第三步配置-openclaw)
- [第四步：启动 Gateway](#第四步启动-gateway)
- [第五步：配置事件订阅](#第五步配置事件订阅)
- [第六步：测试和使用](#第六步测试和使用)
- [常见问题排查](#常见问题排查)
- [配置参考](#配置参考)

---

## 环境信息

- **操作系统**: Linux
- **OpenClaw 版本**: 2026.3.2
- **飞书版本**: 国内版 (feishu.cn)
- **Node.js**: v22.22.0
- **工具权限**: full (完整工具集)

---

## 第一步：安装飞书插件

### 1.1 安装插件

```bash
openclaw plugins install @openclaw/feishu
```

**预期输出:**
```
Downloading @openclaw/feishu…
Installing plugin dependencies…
Installed plugin: feishu
```

### 1.2 验证安装

```bash
openclaw plugins list
```

应该看到 `feishu` 插件已启用。

---

## 第二步：创建飞书应用

### 2.1 访问飞书开放平台

访问：https://open.feishu.cn/app

### 2.2 创建企业自建应用

1. 点击"创建企业自建应用"
2. 填写应用信息：
   - **应用名称**: LeRobot 训练助手（或你喜欢的名字）
   - **应用描述**: 分布式 GPU 训练助手
   - **应用图标**: 上传一个图标

### 2.3 获取凭证

在"凭证与基础信息"页面，复制：
- **App ID**: `cli_xxxxxxxxxxxxxxxx`
- **App Secret**: `xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

⚠️ **重要**: 妥善保管 App Secret，不要泄露！

### 2.4 配置权限

在"权限管理"页面，点击"批量导入"，粘贴以下权限：

```json
{
  "scopes": {
    "tenant": [
      "im:message",
      "im:message.group_at_msg:readonly",
      "im:message.p2p_msg:readonly",
      "im:message:readonly",
      "im:message:send_as_bot",
      "im:resource"
    ]
  }
}
```

### 2.5 启用机器人能力

1. 进入"应用能力" → "机器人"
2. 启用机器人能力
3. 设置机器人名称

---

## 第三步：配置 OpenClaw

### 3.1 编辑配置文件

编辑 `~/.openclaw/openclaw.json`，添加飞书配置：

```json
{
  "channels": {
    "feishu": {
      "enabled": true,
      "connectionMode": "websocket",
      "dmPolicy": "pairing",
      "groupPolicy": "open",
      "accounts": {
        "main": {
          "appId": "cli_a9238f9673b99cc1",
          "appSecret": "JIbTScQNcCntGZXC226mOfiMbMJlbinf",
          "domain": "feishu",
          "enabled": true
        }
      }
    }
  }
}
```

**重要配置说明:**
- `domain`: 必须设置为 `"feishu"`（国内版）或 `"lark"`（国际版）
- `connectionMode`: 使用 `"websocket"` 长连接模式
- `dmPolicy`: `"pairing"` 表示需要配对批准
- `groupPolicy`: `"open"` 表示在所有群聊中响应（需要 @机器人）

### 3.2 环境变量（可选）

也可以通过环境变量配置：

```bash
export FEISHU_APP_ID="cli_a9238f9673b99cc1"
export FEISHU_APP_SECRET="JIbTScQNcCntGZXC226mOfiMbMJlbinf"
```

---

## 第四步：启动 Gateway

### 4.1 检查代理设置（重要！）

**问题**: 如果系统设置了 HTTP 代理，会导致飞书 API 请求失败。

**检查代理:**
```bash
env | grep -i proxy
```

**如果有代理输出，临时取消:**
```bash
unset https_proxy
unset http_proxy
unset HTTPS_PROXY
unset HTTP_PROXY
```

### 4.2 启动 Gateway

```bash
openclaw gateway run
```

**预期输出:**
```
✅ [feishu] client ready
✅ Gateway listening on ws://127.0.0.1:18789
```

### 4.3 验证连接

```bash
openclaw gateway status
```

应该看到 gateway 正在运行。

---

## 第五步：配置事件订阅

### 5.1 配置长连接

1. 回到飞书开放平台
2. 进入"事件订阅"页面
3. 选择"使用长连接接收事件"
4. 添加事件：`im.message.receive_v1`
5. 保存配置

**预期结果:**
- 连接状态显示为"已连接"或"正常"
- 如果显示"未检测到应用连接信息"，说明 gateway 未启动

### 5.2 发布应用

1. 进入"版本管理与发布"
2. 点击"创建版本"
3. 填写版本信息（如：1.0.0）
4. 提交审核
5. 等待审核通过（企业自建应用通常自动审批）

---

## 第六步：测试和使用

### 6.1 私聊测试

1. 在飞书中找到你的机器人
2. 发送消息："你好"
3. 机器人会回复配对码

**示例回复:**
```
你好
OpenClaw: access not configured.
Your Feishu user id: ou_99bd70c78ebc25a2120f5130c9db4882
Pairing code: K35WGXA8
Ask the bot owner to approve with:
openclaw pairing approve feishu K35WGXA8
```

### 6.2 批准配对

```bash
# 查看配对请求
openclaw pairing list feishu

# 批准配对
openclaw pairing approve feishu K35WGXA8
```

**预期输出:**
```
✅ Approved feishu sender ou_99bd70c78ebc25a2120f5130c9db4882
```

### 6.3 再次测试

批准配对后，再次发送消息：

```
👤 你: 你好
🤖 机器人: 你好！我是 OpenClaw 助手，有什么可以帮助你的吗？
```

### 6.4 群聊测试

1. 创建飞书群聊
2. 添加机器人到群
3. @机器人 发送消息
4. 机器人应该会回复

---

## 常见问题排查

### 问题 1: Invalid URL 错误

**错误信息:**
```
TypeError: Invalid URL
code: 'ERR_INVALID_URL',
input: 'feishu.cn/open-apis/auth/v3/tenant_access_token/internal'
```

**原因**: `domain` 配置错误

**解决方案:**
```json
// ❌ 错误
"domain": "feishu.cn"
"domain": "https://open.feishu.cn"

// ✅ 正确
"domain": "feishu"  // 国内版
"domain": "lark"    // 国际版
```

---

### 问题 2: 400 Bad Request - HTTP sent to HTTPS

**错误信息:**
```
400 The plain HTTP request was sent to HTTPS port
```

**原因**: 系统代理配置导致请求被错误处理

**解决方案:**
```bash
# 临时取消代理
unset https_proxy http_proxy HTTPS_PROXY HTTP_PROXY

# 重启 gateway
openclaw gateway run
```

---

### 问题 3: 机器人不回复

**可能原因:**
1. Gateway 未运行
2. 事件订阅未配置
3. 应用未发布
4. 配对未批准（私聊）
5. 未 @机器人（群聊）

**排查步骤:**
```bash
# 1. 检查 gateway 状态
openclaw gateway status

# 2. 查看日志
openclaw logs --follow

# 3. 检查配对状态
openclaw pairing list feishu

# 4. 检查配置
cat ~/.openclaw/openclaw.json | grep -A 20 "feishu"
```

---

### 问题 4: Gateway 无法启动 - 端口占用

**错误信息:**
```
Port 18789 is already in use.
Gateway already running locally.
```

**解决方案:**
```bash
# 查找并杀死进程
ps aux | grep openclaw
kill <PID>

# 或者直接杀死所有 openclaw gateway 进程
pkill -f "openclaw gateway"

# 重新启动
openclaw gateway run
```

---

### 问题 5: 飞书事件订阅显示"未检测到应用连接信息"

**原因**: Gateway 未启动或配置错误

**解决方案:**
1. 确认 gateway 正在运行
2. 确认配置中 `connectionMode: "websocket"`
3. 确认 `domain` 设置正确
4. 重启 gateway 并等待 10 秒后再保存事件订阅

---

## 配置参考

### 完整配置示例

```json
{
  "channels": {
    "feishu": {
      "enabled": true,
      "connectionMode": "websocket",
      "dmPolicy": "pairing",
      "groupPolicy": "open",
      "accounts": {
        "main": {
          "appId": "cli_xxxxxxxxxxxxxxxx",
          "appSecret": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
          "domain": "feishu",
          "enabled": true
        }
      }
    }
  },
  "plugins": {
    "entries": {
      "feishu": {
        "enabled": true
      }
    }
  }
}
```

### 配置参数说明

| 参数 | 类型 | 说明 | 默认值 |
|------|------|------|--------|
| `enabled` | boolean | 是否启用飞书通道 | `true` |
| `connectionMode` | string | 连接模式：`websocket` 或 `webhook` | `websocket` |
| `dmPolicy` | string | 私聊策略：`pairing`, `open`, `allowlist`, `disabled` | `pairing` |
| `groupPolicy` | string | 群聊策略：`open`, `allowlist`, `disabled` | `open` |
| `appId` | string | 飞书应用 ID | - |
| `appSecret` | string | 飞书应用密钥 | - |
| `domain` | string | 域名：`feishu` (国内) 或 `lark` (国际) | `feishu` |

---

## 高级配置

### 多账户配置

```json
{
  "channels": {
    "feishu": {
      "defaultAccount": "main",
      "accounts": {
        "main": {
          "appId": "cli_xxx",
          "appSecret": "xxx",
          "domain": "feishu"
        },
        "backup": {
          "appId": "cli_yyy",
          "appSecret": "yyy",
          "domain": "feishu",
          "enabled": false
        }
      }
    }
  }
}
```

### 群聊白名单

```json
{
  "channels": {
    "feishu": {
      "groupPolicy": "allowlist",
      "groupAllowFrom": ["oc_xxx", "oc_yyy"],
      "accounts": {
        "main": {
          "appId": "cli_xxx",
          "appSecret": "xxx",
          "domain": "feishu"
        }
      }
    }
  }
}
```

### 免提及群聊

```json
{
  "channels": {
    "feishu": {
      "groups": {
        "oc_xxx": {
          "requireMention": false
        }
      }
    }
  }
}
```

---

## 管理命令

### Gateway 管理

```bash
# 启动 gateway（前台运行）
openclaw gateway run

# 查看 gateway 状态
openclaw gateway status

# 停止 gateway
openclaw gateway stop

# 重启 gateway
openclaw gateway restart

# 查看日志
openclaw logs --follow
```

### 配对管理

```bash
# 查看配对请求
openclaw pairing list feishu

# 批准配对
openclaw pairing approve feishu <CODE>

# 拒绝配对
openclaw pairing reject feishu <CODE>
```

### 配置管理

```bash
# 查看配置
cat ~/.openclaw/openclaw.json

# 编辑配置
nano ~/.openclaw/openclaw.json

# 验证配置
openclaw doctor --fix
```

---

## 使用技巧

### 1. 保持 Gateway 运行

使用 `screen` 或 `tmux` 保持 gateway 后台运行：

```bash
# 使用 screen
screen -S openclaw
openclaw gateway run
# 按 Ctrl+A+D 分离

# 重新连接
screen -r openclaw
```

### 2. 开机自启动

创建 systemd 服务：

```bash
# 创建服务文件
sudo nano /etc/systemd/system/openclaw-gateway.service

# 内容：
[Unit]
Description=OpenClaw Gateway
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/bin/openclaw gateway run
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target

# 启用服务
sudo systemctl daemon-reload
sudo systemctl enable openclaw-gateway
sudo systemctl start openclaw-gateway
```

### 3. 监控脚本

创建健康检查脚本：

```bash
#!/bin/bash
# health-check.sh

if ! pgrep -f "openclaw gateway" > /dev/null; then
    echo "Gateway not running, restarting..."
    openclaw gateway run &
fi
```

---

## 总结

通过以上步骤，你已经成功将 OpenClaw 集成到飞书中。现在你可以：

- ✅ 在私聊中使用机器人
- ✅ 在群聊中 @机器人 获得帮助
- ✅ 使用 LeRobot 训练助手功能
- ✅ 扫描局域网 GPU 节点
- ✅ 配置和部署训练任务
- ✅ 查看历史训练经验

---

## 相关链接

- [OpenClaw 官方文档](https://docs.openclaw.ai)
- [飞书开放平台](https://open.feishu.cn)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
- [LeRobot 训练助手技能](./lerobot-training-assistant/)

---

## 更新日志

- **2026-03-05**: 初版发布，完成飞书集成
- **2026-03-05**: 添加常见问题排查
- **2026-03-05**: 添加高级配置说明

---

**文档维护**: OpenClaw 自动生成  
**最后更新**: 2026-03-05 19:12 GMT+8
