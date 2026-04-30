---
description: 解析用户的自然语言指令，调用 glab CLI 执行对应的代码协作操作（MR、Issue、Repo 等），并输出关键结果与链接
---

用户输入的指令为：`$ARGUMENTS`

你的任务是把用户的自然语言指令翻译成正确的 `glab` 命令，执行后将关键结果（状态、ID、URL）回显给用户。

---

## 工作范围

只处理与**代码协作**相关的操作，包括但不限于：

- **MR（Merge Request）**：创建、查看、列出、检出、评论、批准、合并、关闭
- **Issue**：创建、查看、列出、关闭、评论
- **Repo**：查看、克隆、搜索、贡献者
- **Release / Tag**：列表、创建
- **CI / Pipeline**：状态查看
- **API**：当现成子命令满足不了用户需求时，回退到 `glab api` 调用

如果用户请求超出代码协作范畴（如改本地代码、写文档），告知用户该命令仅处理 glab 操作。

---

## 执行流程

### 第一步：环境检查

并行执行：

1. `glab auth status` — 确认已登录，否则提示用户先 `glab auth login`
2. `git remote -v`（仅当用户未通过 `-R` 指定仓库时）— 确认当前在 git 仓库内、glab 能识别默认仓库

如果不在 git 仓库且用户未指定 `-R <namespace>/<repo>`，向用户确认目标仓库后再执行。

### 第二步：解析意图

仔细阅读 `$ARGUMENTS`，识别：

- **动作**（create / list / view / merge / approve / close / checkout / diff …）
- **对象**（mr / issue / repo / release / pipeline …）
- **关键参数**（ID、分支名、标题、描述、reviewer、label、target-branch 等）

#### 环境 → 目标分支映射（创建 MR 时务必遵守）

| 用户说法 | 含义 | `--target-branch` |
| --- | --- | --- |
| "测试环境" / "提到测试" / "test" | 测试环境 | `test` |
| "预发环境" / "预发" / "提到预发" / "release" | 预发环境 | `release` |
| "线上" / "生产" / "上线" / "main" / "master" | 线上环境 | `main` |

**规则**：

- 用户提到上述任一环境关键词时，**必须**用对应分支作为 `--target-branch`，不要默认到 `main`
- 如果用户只说"提个 MR"没指明环境，**反问用户目标环境**（test / release / main），不要擅自默认
- 一句指令中可能涉及多次提交流程（如"先提到测试再提到预发"），按顺序逐个创建并向用户确认

常见意图映射示例：

| 用户说法 | 对应命令 |
| --- | --- |
| "创建一个 MR" / "提一个合并请求" | `glab mr create --fill --yes --target-branch <env-branch>`（env 由下方环境映射决定，未明示则反问） |
| "提到测试环境" / "提到 test" | `glab mr create --fill --yes --target-branch test` |
| "提到预发" / "提到 release" | `glab mr create --fill --yes --target-branch release` |
| "提到线上" / "提到 main" | `glab mr create --fill --yes --target-branch main` |
| "提个草稿 MR" | `glab mr create --fill --yes --draft --target-branch <env-branch>` |
| "列一下我的 MR" / "我开的 MR" | `glab mr list --author=@me` |
| "需要我审的 MR" | `glab mr list --reviewer=@me` |
| "看 MR 271" / "MR !271 详情" | `glab mr view 271` |
| "看 MR 271 的 diff" | `glab mr diff 271` |
| "把 MR 271 拉下来" | `glab mr checkout 271` |
| "批准 MR 271" / "approve 271" | `glab mr approve 271` |
| "合并 MR 271" | `glab mr merge 271`（需用户二次确认） |
| "关闭 MR 271" | `glab mr close 271`（需用户二次确认） |
| "评论 MR 271 'LGTM'" | `glab mr note 271 -m "LGTM"` |
| "建一个 issue" | `glab issue create -t "..." -d "..."` |
| "看 issue 42" | `glab issue view 42 --comments` |
| "在浏览器打开当前仓库" | `glab repo view --web` |
| "看仓库信息" | `glab repo view` |

### 第三步：参数补全

如果用户未提供必要参数（如创建 MR 时缺少标题/目标分支），按下列优先级处理：

1. **能从上下文推断**：用 `--fill` 让 glab 自动从 commit message 取标题/描述；目标分支默认 `main`
2. **可用合理默认值**：如 reviewer / assignee 为空就不传
3. **必须用户给**（如自定义标题、自定义描述）：向用户提问后再执行

### 第四步：执行命令

- **只读操作**（list / view / diff / status …）直接执行
- **写操作**（create / merge / close / approve / note …）：
  - 在执行前用一句话告诉用户即将执行的完整命令
  - **合并 / 关闭 / 强制类操作**（`merge`、`close`、`revoke`、`token rotate` 等）必须先向用户二次确认；用户确认后再执行命令本身（命令可加 `--yes` 跳过 glab 自身的 TTY 弹窗，避免输出被终端控制码污染）
  - **非破坏性写操作**（如 `mr create`、`mr note`、`mr approve`、`issue create`）默认加 `--yes`，避免 glab 进入 TTY 交互模式输出 ANSI 控制码（`[?2026$p`、`[?25l` 等）淹没结果
  - 创建 MR 时如未指定 `--remove-source-branch`，按用户偏好处理（默认不加，除非用户说"合并后删源分支"）

### 第五步：输出结果

向用户回报时遵循以下格式：

#### 成功时

简短一句话说明做了什么 + 关键信息：

- **MR/Issue 创建**：标题、IID、URL
- **MR/Issue 查看**：状态、作者、目标分支、URL（用 `--output json` 取关键字段，避免长输出占满屏幕）
- **list 类**：以 markdown 表格输出 IID、标题、作者、状态、URL；每行 URL 用 markdown 链接形式
- **merge / approve / close**：操作结果 + 当前 MR 状态 + URL
- **diff**：输出在终端（用户可直接看），不要复述全部内容

URL 一律用 markdown 链接渲染，例如 `[!271](https://coding.jd.com/.../merge_requests/271)`。

#### 失败时

- 显示 glab 返回的错误信息
- 给出可能的原因（未登录 / 仓库未匹配 / 权限不足 / ID 不存在）
- 给出下一步建议

---

## 注意事项

- **仓库识别**：默认使用当前目录的 `git remote`；用户明确指定其它仓库时加 `-R <namespace>/<repo>`
- **不要盲目使用 `--web`**：除非用户明确说"在浏览器打开"
- **不要自动 push 代码**：`glab mr create` 不会推分支，若本地分支未推送到远端，先提示用户 `git push -u origin <branch>` 再创建 MR
- **JSON 输出**：当输出结果会很长（如 list、view），优先用 `--output json` + `jq` 提取关键字段，避免污染上下文
- **避免 TTY 控制码污染**：在非交互执行环境下，glab 的写操作（如 `mr create`）若检测到伪 TTY，会输出大量 ANSI 控制序列（`[?2026$p`、`[?25l`、`[J` 等）淹没真正结果。统一对写操作传 `--yes` 跳过确认；如仍出现污染，可改为 `glab ... 2>&1 | cat`（强制 stdout 非 TTY）或在命令前加 `GLAB_PAGER=cat NO_COLOR=1` 环境变量
- **敏感操作**：`merge`、`close`、`revoke`、`reopen`、`release create`、`token rotate`、写类 `glab api -X POST/PUT/DELETE` 必须二次确认
- **参考文档**：完整命令参考见 [data/glab.md](data/glab.md)，有疑问时优先查阅
