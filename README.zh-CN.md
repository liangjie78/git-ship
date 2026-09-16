<p align="center">
  <img src="./assets/readme/hero.svg" width="100%" alt="git-ship 通过带边界检查的 Git 工作流，把明确范围的改动交付到默认分支">
</p>

<p align="center">
  <a href="./README.md">English</a>
</p>

Git Ship 用目标仓库已有的 GitHub 流程处理明确范围的工作区改动。默认模式保持全自动：

```text
工作区改动 → 范围检查 → 最新默认分支 → 任务分支 → commit
→ 本地验证 → PR → CI 修复 → 合并 → 清理
```

需要人工审查时，明确使用 `ship --review`，流程会在 PR 创建或更新后停止。

<p align="center">
  <img src="./assets/readme/workflow.svg" width="100%" alt="git-ship 带边界检查的工作阶段">
</p>

> 本仓库是 [oil-oil/git-ship](https://github.com/oil-oil/git-ship) 的 fork；下面的安装示例明确指向当前的 `liangjie78/git-ship`，以本仓库的 `SKILL.md` 为准。

## 它解决什么问题

- 默认一路推进到实际合并，而不是只创建 PR 就结束。
- 在写入 Git 状态前识别任务范围，只暂存明确路径。
- 使用仓库真实默认分支和已有验证命令。
- 读取 CI 失败并修复范围内的安全问题，提交后重新等待检查。
- 保留已明确且能安全排除的无关改动、已有分支、本地独有提交和用户已有 stash；归属或安全排除方式不明确时停止。
- 仅在范围歧义、认证、权限、人工审批或外部服务等阻塞无法自行解决时暂停。
- 提供明确的审查模式，但不把审查模式设为默认。

## 安装

将 fork 版本安装为用户级 Codex Skill：

```bash
npx skills add liangjie78/git-ship -g -a codex
```

也可以克隆到 Agent 能发现 Skill 的目录：

```bash
git clone https://github.com/liangjie78/git-ship.git ~/.agents/skills/git-ship
```

入口文件是 `SKILL.md`。如果宿主会缓存 Skill，安装后重新加载宿主。

## 使用

完成改动并保留在工作区，然后告诉 Agent：

```text
ship
```

这会走完整流程：commit、push、PR、CI 修复、合并和清理。也可以在调用时指定分支或 commit：

```text
把这次改动 ship 为 feat/add-export，commit 用 "feat(export): add markdown export"
```

需要创建 PR 并交给人工审查时：

```text
ship --review
```

之后再次执行不带 `--review` 的 `ship`，即可继续现有任务 PR 的完整交付。只要求 commit、push 或创建 PR 的普通请求，
不会触发完整 Ship 流程。

## 工作流程

| 阶段 | 默认 `ship` | `ship --review` | 边界检查 |
| --- | --- | --- | --- |
| 检查 | 建立任务范围并读取仓库状态 | 相同 | 范围不明或归属混合时停止 |
| 同步 | 获取真实默认分支并保留范围内改动 | 相同 | 不覆盖本地独有提交 |
| 分支 | 创建或复用任务分支 | 相同 | 不复用无关分支 |
| 提交 | 暂存明确路径并运行钩子 | 相同 | 不使用 `git add -A` |
| 验证 | 运行已有检查并修复范围内失败 | 相同的本地检查 | 不编造或弱化检查 |
| 发布 | push 并创建或更新 PR | 相同 | 需要正常 GitHub 写权限 |
| CI | 等待、诊断、修复、push、再次等待 | 在 PR 处停止，不等待或修复 | 不绕过或伪造通过 |
| 合并 | 遵循仓库策略并合并 | 不合并 | 不使用管理员权限或绕过保护 |
| 清理 | 只删除本次创建且已合并的分支并回到默认分支 | 不清理 | 保留预先存在的分支 |

## 运行要求

- Git
- 已通过 `gh auth login` 登录的 [GitHub CLI](https://cli.github.com/)
- 能发现真实默认分支并具备正常 push/PR 权限的仓库
- 仓库已有的验证命令（如果项目提供）

找不到可信验证命令时，Git Ship 会明确报告，不会自行猜测。必需人工审批、权限不足、服务不可用、范围无法判断，
或修复后同一失败没有新证据地重复出现，属于外部停止条件。

## 安全边界

明确调用 `ship` 授权当前任务范围内的 commit、正常 push、PR、修复、合并和清理，不授权无关文件或用户已有分支。
`ship --review` 只改变停止位置，不会默默改变默认模式。

Git Ship 不使用 force push、硬重置、清理工作区、跳过钩子或测试、弱化有效断言、管理员合并或绕过分支保护；
需要停止时保留现场，确保后续可以继续。

## 自定义

可以 fork 本仓库，修改 `SKILL.md` 以适配团队的分支命名、合并策略、PR 模板或必需检查。请保留范围检查、显式暂存、
非破坏性恢复、审查开关和外部权限边界。

## 参与贡献

保持改动聚焦，说明行为与失败路径，并同时验证默认全自动交付和 `--review` 行为。

## 开源协议

[MIT](./LICENSE)
