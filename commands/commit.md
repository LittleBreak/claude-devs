---
description: 遵循 Conventional Commits 规范生成提交信息并提交代码
---

请按照以下步骤执行 git commit 操作：

## 第一步：分析变更

并行执行以下命令，了解当前工作区状态：

1. `git status` — 查看所有未跟踪和已修改的文件
2. `git diff --staged` 和 `git diff` — 查看暂存区和工作区的具体变更内容
3. `git log --oneline -10` — 查看最近的提交记录，了解项目的提交风格

## 第二步：暂存文件

- 根据变更内容，将相关文件添加到暂存区
- **禁止**暂存包含敏感信息的文件（如 `.env`、密钥文件、凭证等）
- 优先使用具体文件名添加，避免使用 `git add -A` 或 `git add .`

## 第三步：生成 Conventional Commit 消息

严格遵循 [Conventional Commits](https://www.conventionalcommits.org/) 规范：

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### 类型（type）必须为以下之一：

| 类型       | 说明                                       |
| ---------- | ------------------------------------------ |
| `feat`     | 新增功能                                   |
| `fix`      | 修复缺陷                                   |
| `docs`     | 仅文档变更                                 |
| `style`    | 不影响代码含义的变更（空格、格式、分号等） |
| `refactor` | 既非修复缺陷也非新增功能的代码重构         |
| `perf`     | 提升性能的变更                             |
| `test`     | 添加或修正测试                             |
| `build`    | 影响构建系统或外部依赖的变更               |
| `ci`       | CI 配置文件和脚本的变更                    |
| `chore`    | 不修改 src 或 test 的其他变更              |
| `revert`   | 撤销之前的提交                             |

### 规则：

- **description**（简短描述）：使用中文，不加句号，不超过 50 个字符
- **body**（正文）：可选，使用中文解释变更的动机和与之前行为的对比
- **footer**：可选，用于关联 issue（如 `Closes #123`）或标注 BREAKING CHANGE
- 如果存在破坏性变更，必须在 type 后加 `!` 或在 footer 中注明 `BREAKING CHANGE:`

### 示例：

```
feat(auth): 添加 OAuth2 登录支持

实现基于 PKCE 的 OAuth2 授权码流程，
替换原有的基于会话的身份认证方式。

Closes #42
```

## 第四步：执行提交

- 使用 HEREDOC 格式传递提交消息，确保格式正确
- **始终创建新提交**，禁止使用 `--amend` 修改上一次提交（除非用户明确要求）
- 禁止使用 `--no-verify` 跳过 hooks
- 提交完成后运行 `git status` 验证结果

## 注意事项

- 不要自动推送到远程仓库，除非用户明确要求
- 如果没有任何变更，告知用户无需提交
- 如果 pre-commit hook 失败，修复问题后创建**新的提交**，不要 amend
