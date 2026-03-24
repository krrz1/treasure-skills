# Treasure Skills

一个个人收藏的实用 Claude Code skill 合集（也兼容其他 AI 编程助手）。

每个 skill 都是一个独立的流程，帮助你的 agent 更好地理解你真正想要什么。

## Skills

| Skill | 说明 |
|-------|------|
| [clarity-check](clarity-check/) | 在执行前评估模糊的技术请求。当 "测试一下" 或 "修一下 CI" 这类提示可能导向完全不同的工作时，这个 skill 会评估解读的可靠性，决定是直接执行、声明假设，还是先问一个澄清问题。 |

## 目录结构

```
treasure-skills/
├── <skill-name>/
│   └── SKILL.md        # Skill 定义（YAML frontmatter + 指令）
└── ...
```

每个 skill 独立放在自己的目录中，包含一个 `SKILL.md` 文件：

- **Frontmatter** — `name` 和 `description`（用于触发匹配）
- **正文** — agent 被调用时遵循的完整流程

## 使用方式

将本仓库 clone 或 symlink 到你的 Claude Code skills 目录，或在项目配置中引用单个 skill。
