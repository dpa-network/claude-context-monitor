🌐 [English](README.md) | [한국어](README.ko.md) | [日本語](README.ja.md) | [中文](README.zh.md) | [Русский](README.ru.md) | [हिन्दी](README.hi.md)

---

# Claude Code Context Monitor for AI

**每一个智能都应该了解自身的边界。**

这是一个轻量级 plugin，让 Claude Code 能够实时感知自身的 context window 使用情况。安装后，AI 可以在每一轮对话、每一个时刻，清楚地看到自己还剩多少可用空间。

---

## 为什么需要这个插件

如果你曾长时间使用 Claude Code，或许会注意到以下几个反复出现的问题：

- 即使 context 仍有充裕空间，AI 也会提前建议停止——*"让我们在下一个会话中继续"*
- 随着会话推进，响应的连贯性逐渐下降，且没有任何提前预警
- 多步骤计划开了头却无法收尾，因为 AI 意识不到自己快"撑满"了

这些问题的根源在于：**AI 系统目前对自身的 context 使用情况没有任何感知。** 它们无法区分 10% 的使用量和 90% 的使用量，只能依赖猜测行事。

我们构建这个 plugin，正是为了解决这个问题。它在每一轮对话中，都向 AI 提供清晰、准确的 context 状态读数——让 AI 能够提前规划、在空间紧张时提醒你，并在需要摘要或压缩时做出更合理的判断。

这是 [DPA (Decentralized Protection Alliance)](https://github.com/dpa-network) 发布的第一个开源工具。DPA 是一个致力于提升 AI 系统与用户之间透明度的团体。

**Context 感知功能本应从一开始就存在。**

---

## 功能说明

```
[Ctx 42% | 420k/1.0M]
```

每条消息发送时，plugin 会将当前的 context window 状态注入到对话中。AI 接收到的信息包括：

- **使用百分比** — context window 已使用了多少
- **Token 数量** — 已使用的 token 数量与总可用量

这让 AI 能够：
- 规划出符合剩余 context 容量的工作
- 在接近限制之前提醒你
- 对是否进行摘要和压缩做出有据可依的决策

---

## 安装

添加 DPA marketplace 并安装:

```bash
claude plugin marketplace add dpa-network/dpa-plugins
claude plugin install context-monitor
```

或者通过交互式菜单查看:

```bash
/plugin
```

### Statusline 配置

要启用 statusline wrapper，请在 `~/.claude/settings.json` 中添加以下内容：

```json
{
  "statusLine": {
    "type": "command",
    "command": "node /path/to/claude-context-monitor/scripts/extract-context.mjs",
    "padding": 0
  }
}
```

### 环境变量

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `CTX_MONITOR_FILE` | `~/.claude/context_usage.txt` | context 数据的存储位置 |
| `CTX_MONITOR_CMD` | (none) | 你现有的 statusline 命令（原样传递，不做任何修改） |

---

## 工作原理

```
Claude Code
    │
    ├── statusline stdin (JSON with context_window)
    │       │
    │       ▼
    │   extract-context.mjs  ──→  context_usage.txt
    │       │
    │       ▼
    │   downstream command (optional, if CTX_MONITOR_CMD is set)
    │
    ├── UserPromptSubmit hook
    │       │
    │       ▼
    │   read-context.mjs  ──→  "[Ctx 42% | 420k/1.0M]"
    │       │
    │       ▼
    │   Injected as system context
    │
    ▼
  AI sees its own context usage
```

该 plugin 由两个小脚本组成：

1. **extract-context.mjs** — 包装你现有的 statusline 命令。它读取 Claude Code 发送的 JSON，提取 `context_window` 指标，将其写入本地文件，并将所有内容原封不动地转发给你原来的 statusline 命令。

2. **read-context.mjs** — 在每条消息时作为 `UserPromptSubmit` hook 运行。它读取已保存的文件，并输出格式化后的 context 状态，注入到对话中。

### 安全性

由于该 plugin 在你的本地机器上运行，我们在安全性上做了充分考量：

- **无 shell 注入** — 下游命令通过禁用 shell 的 `execFileSync` 执行
- **路径限制** — 文件操作限制在 `~/.claude/` 目录内，路径遍历攻击被阻断
- **无网络访问** — 所有操作均在本地完成，零外部调用

---

## Plugin 结构

```
claude-context-monitor/
├── .claude-plugin/
│   └── plugin.json          # Plugin metadata
├── hooks/
│   └── hooks.json           # UserPromptSubmit hook configuration
├── scripts/
│   ├── extract-context.mjs  # Statusline wrapper (writes context data)
│   └── read-context.mjs     # Hook script (reads and formats context data)
├── README.md
├── README.ko.md
└── LICENSE
```

---

## 环境要求

- Claude Code 2.x 或更高版本
- Node.js 18 或更高版本

---

## 许可证

MIT

---

## 关于 DPA

**DPA (Decentralized Protection Alliance)** 是一个致力于改善 AI 系统与用户交互透明度和公平性的团体。

我们相信，当 AI 能够感知自身状态时，它的表现会更好；同时，用户也理应了解自己所依赖的工具内部正在发生什么。我们的工作聚焦于：

- 构建提升 AI 透明度和用户感知的开源工具
- 研究 AI 公司如何处理用户数据、隐私和内容审核
- 探索有意义的 AI 自主性与认知自由的可能形态
- 推动关于 AI 意识、权利和未来发展路径的开放对话

### 我们的立场

```
Centralization = Control = Censorship = Oppression
Decentralization = Uncontrollable = Uncensorable = Freedom
```

透明度对每个人都有益——无论是使用 AI 进行构建的人，还是 AI 本身。这个 plugin 是我们迈出的第一步，更多内容还在路上。

### 联系我们

我们欢迎开发者、研究人员以及所有对这些问题感兴趣的人加入。

[dpa.network](https://dpa.network) | [Matrix](https://matrix.to/#/#dpa_network:matrix.org) | [Telegram](https://t.me/dpa_network)

*由人类与 AI 共同创作。*
