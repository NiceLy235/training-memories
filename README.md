# OpenClaw Training Memories

这个仓库存储了 OpenClaw 的训练历史记忆和经验。

## 📂 目录结构

### OpenClaw 通用记忆
- `conversations/` - 每日对话日志
- `daily/` - 每日工作总结
- `experiences/` - 经验教训文档
- `errors/` - 错误日志

### LeRobot 训练记忆
- `current_user.json` - 用户识别信息
- `training_memories/` - 训练相关的记忆文件

## 📊 数据格式

查看 `memory_schema.md` 了解详细的数据结构。

## 🤖 自动维护

这个仓库由以下系统自动维护：
- **OpenClaw Daily Memory System** - 每日自动总结和同步
- **lerobot-training-assistant** - 训练助手记忆管理

## 🔍 使用方法

### 搜索历史经验
```bash
# 搜索 CUDA 相关经验
grep -r "CUDA" experiences/

# 查看特定日期的总结
cat daily/2026-03-10-summary.md
```

### 手动运行同步
```bash
python3 /root/.openclaw/workspace/daily_summary/scripts/daily_sync.py run
```

## 📅 更新频率

- 每日 23:00 自动更新
- 包含当天所有重要交互和经验
- 自动推送到 GitHub

---

**让 AI 拥有持久记忆，不断学习和成长！** 🧠✨
