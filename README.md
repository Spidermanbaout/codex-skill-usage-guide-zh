# Codex Skills 使用教程：创建、调用和改进可复用工作流

> 本文介绍 Codex Skills 的本地使用方式，适合希望把重复任务变成稳定流程的用户。菜单和入口会随 Codex 版本变化。  
> 最后核对：2026-10-13

## 目录

- [1. Skill 是什么，适合做什么](#1-skill-是什么适合做什么)
- [2. 用内置创建器快速上手](#2-用内置创建器快速上手)
- [3. 手动编写一个 SKILL.md](#3-手动编写一个-skillmd)
- [4. 安装到项目或个人目录](#4-安装到项目或个人目录)
- [5. 在 Codex 里调用 Skill](#5-在-codex-里调用-skill)
- [6. 测试 Skill 是否触发正确](#6-测试-skill-是否触发正确)
- [7. 检查安全和权限边界](#7-检查安全和权限边界)
- [8. 常见问题](#8-常见问题)
- [官方资料](#官方资料)

## 1. Skill 是什么，适合做什么

Skill 是一个包含 `SKILL.md` 的目录，可以为 Codex 保存一套可重复使用的任务说明，也可以附带参考资料、模板或脚本。Codex 会先读取 Skill 的名称和描述；判断当前任务符合时，再载入完整说明。

适合做成 Skill 的任务通常有明确的输入、固定步骤和可检查的交付结果，例如：

- 按团队格式整理会议记录；
- 根据仓库约定检查代码变更；
- 把 CSV 整理成固定格式的报告；
- 按项目流程生成发布说明。

每个 Skill 聚焦一个具体工作目标。像“帮我处理任何编程任务”这种范围太大的描述，难以稳定触发，也难以写清楚完成标准。

## 2. 用内置创建器快速上手

在 Codex 中输入 `$skill-creator`，让内置创建器根据你的目标生成 Skill。描述下面几项：

- 希望它完成什么任务；
- 哪些请求应该触发，哪些不应该；
- 输入材料是什么；
- 需要按哪些步骤处理；
- 最终交付什么；
- 哪些动作需要先停下来等你确认；
- 是否需要脚本（默认先做纯说明型 Skill）。

**可以直接这样问：**

```text
$skill-creator

创建一个 meeting-notes Skill。
当我提供会议录音转写或会议纪要，并要求整理行动项时使用。
提取事项、负责人、截止时间和原文依据；没有明确负责人的标成“待确认”，不要推测。
输出 Markdown 表格和未解决问题清单。不要创建日历事件或向任何人发送消息。
优先做成纯说明型，不需要脚本。
```

如果你只是想用已有 Skill，可以通过 Codex 的 Skills 列表查看已发现的 Skills；也可以在请求中用 `$skill-name` 明确点名。

## 3. 手动编写一个 SKILL.md

Skill 文件夹至少需要一个 `SKILL.md`。文件开头用 YAML front matter 提供 `name` 和 `description`，后面写工作流程。下面是一个简化示例：

```markdown
---
name: meeting-notes
description: Turn meeting notes into action items with owners and dates. Use when the user asks to organize a transcript or meeting notes.
---

# Meeting Notes

## Inputs
A meeting transcript or notes supplied by the user.

## Steps
1. Read the full material before summarizing.
2. Extract action items, owners, due dates, and supporting quotes.
3. Mark missing owners or dates as “Needs confirmation”; do not infer them.
4. Separate decisions from suggestions and open questions.

## Output
Return a Markdown table with:
- Action item
- Owner
- Due date
- Supporting evidence

End with a short list of unresolved questions.
Do not create calendar events or send messages.
```

字段用途：

- **name**：简短、可读、适合写成 `$meeting-notes` 的名称。
- **description**：告诉 Codex 这个 Skill 做什么、什么时候该触发。写清楚具体场景和边界。
- **正文**：说明输入、步骤、输出格式、验证方式和停止条件。

先让 `SKILL.md` 保持简洁。只有当详细资料确实需要按任务读取时，再添加辅助文件：

```text
meeting-notes/
├── SKILL.md
├── references/
│   └── team-style.md
├── assets/
│   └── action-items-template.md
└── scripts/
    └── validate_output.py
```

在 `SKILL.md` 中告诉 Codex 何时读取 `references/`、复制 `assets/` 或运行 `scripts/`。确定性计算或重复文件处理适合脚本；普通判断和已有工具能完成的步骤不必额外写脚本。

## 4. 安装到项目或个人目录

本地 Skill 用一个独立目录存放，并放在 Codex 会扫描的位置：

**只在当前项目或文件夹使用：**

```text
<项目根目录>/.agents/skills/meeting-notes/SKILL.md
```

**在自己的多个项目中使用：**

```text
$HOME/.agents/skills/meeting-notes/SKILL.md
```

把 Skill 放进仓库的 `.agents/skills`，方便团队成员通过 Git 一起维护；放进个人目录则用于自己的各个项目。Codex 会自动检测本地 Skill 的变化。如果更新后没有出现，重启 Codex 再检查。

安装社区或精选 Skill 时，可以运行内置的 `$skill-installer`，并按提示提供 Skill 名称或来源仓库。安装前先读一遍 `SKILL.md` 及它引用的脚本和资料。

如果要把 Skill 分享给其他用户、和连接器一起发布，或作为可安装体验分发，应考虑把它打包为 Plugin；单纯的本地 Skill 更适合个人试用和项目内工作流。

## 5. 在 Codex 里调用 Skill

Skills 有两种常见触发方式：

1. **自动触发**：你的请求符合 Skill 的 `description` 时，Codex 可以自动选择它。
2. **明确调用**：在 Codex 提示词中写 `$meeting-notes`，或通过 `/skills` 查看并选择 Skill。

例如：

```text
$meeting-notes
把这份会议转写整理成行动项。保留原文依据；不确定的负责人标为待确认。
```

如果 Codex 没有自动选择 Skill，先检查名称和描述是否具体，再用 `$skill-name` 明确调用一次，观察它执行的步骤是否符合预期。

## 6. 测试 Skill 是否触发正确

不要只测试“它运行成功了吗”。分别准备几种请求：

| 类型 | 示例 | 期望 |
| --- | --- | --- |
| 明确调用 | “用 `$meeting-notes` 整理这份转写” | 按 Skill 完整执行 |
| 自动触发 | “这是今天的会议记录，帮我列出行动项” | 自动选中 Skill |
| 缺少信息 | “整理一下” | 先确认材料或目标 |
| 相邻但不适用 | “帮我约一个会议时间” | 不应错误调用会议纪要 Skill |
| 边界情况 | 记录中没有负责人或日期 | 标为待确认，不编造 |

改进时分清问题类型：

- **没有触发或误触发**：调整 `description`，说清触发场景和不适用场景。
- **触发了但结果不一致**：补充步骤、示例、输出格式或验证要求。
- **流程做得对但少了信息**：明确必需输入；缺失时应提问或标注未提供。
- **步骤容易失败**：写明异常处理、停止条件和用户需要接手的节点。

OpenAI 的 Codex 技能评估指南建议用少量代表性提示词持续检查触发和结果，并把反复出现的失败案例加入测试集。

## 7. 检查安全和权限边界

Skill 内容会影响 Codex 的计划、工具选择和命令执行。安装来源不明的 Skill 前，先检查 `SKILL.md`、脚本和引用的文件；确认它没有要求泄露信息或执行不必要操作。

编写自己的 Skill 时：

- 不要把 API Key、密码、访问令牌写入 Skill 文件或提交到仓库。
- 对写文件、安装依赖、联网或调用外部服务的步骤写清楚范围。
- 删除、发布、发送消息或变更生产系统等动作，明确规定先提供草稿并等待用户批准。
- 不要把“没在资料里找到”的内容补写成事实。
- 使用权限较少的运行环境，并检查实际修改和命令记录。

官方文档也提醒，Skill 可能包含提示注入或不安全指令；应先审查再使用，并对敏感写入动作保留审批。

## 8. 常见问题

**Codex 没有发现 Skill**  
确认文件名是 `SKILL.md`，目录位于扫描路径中，并包含有效的 `name` 和 `description`。若刚刚新建或修改仍未出现，重启 Codex。

**自动触发不稳定**  
把 `description` 写得更具体，说明用户会提出什么任务；用几条相似请求和几条不应该触发的请求试验。需要稳定复现时，直接写 `$skill-name`。

**一个 Skill 越写越长**  
把按需查询的规则放到 `references/`，模板放到 `assets/`，确定性文件处理放到 `scripts/`。如果 Skill 同时处理多个目标，拆成几个职责清晰的 Skill。

**想把 Skill 发给团队**  
将项目内共享的 Skill 放到仓库的 `.agents/skills`，与代码一起评审；面向更广用户分发时，按需打包成 Plugin。

## 官方资料

- [Codex Skills：创建、调用、安装和本地发现](https://developers.openai.com/codex/skills)
- [Testing Agent Skills Systematically with Evals](https://developers.openai.com/blog/eval-skills)
- [OpenAI Developers：Build skills](https://developers.openai.com/plugins/build/skills)
- [OpenAI Developers：Skills 概念](https://developers.openai.com/plugins/concepts/skills)
