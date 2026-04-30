下面是按**日常使用频率**排序的 glab 命令清单。

---

## 🥇 一、MR（最高频）

| 命令                    | 说明                       |
| ----------------------- | -------------------------- |
| `glab mr create`        | 创建 MR                    |
| `glab mr list`          | 列出 MR                    |
| `glab mr view`          | 查看 MR 详情               |
| `glab mr checkout <id>` | 把别人的 MR 拉下来本地评审 |
| `glab mr diff <id>`     | 看 MR 的代码 diff          |
| `glab mr update`        | 修改 MR 标题/描述/labels   |
| `glab mr approve <id>`  | 批准 MR                    |
| `glab mr merge <id>`    | 合并 MR                    |
| `glab mr close <id>`    | 关闭 MR                    |
| `glab mr note`          | 给 MR 加评论               |

### 1. 创建 MR

```bash
# 用 commit 信息预填，最常用
glab mr create --fill --target-branch main

# 草稿 MR
glab mr create --fill --draft

# 完整指定（适合脚本）
glab mr create \
  --title "feat(订单): 新增退款类型筛选" \
  --description "$(cat <<EOF
## 变更
- xxx
## 测试
- xxx
EOF
)" \
  --target-branch main \
  --assignee zhangsan,lisi \
  --reviewer wangwu \
  --label "前端,bugfix" \
  --remove-source-branch

# 创建后立即在浏览器打开
glab mr create --fill --web
```

**关键 flag**：
- `-f, --fill` — 用最近 commit message 填 title/description
- `-d, --draft` — 草稿 MR（行云上叫"草案"）
- `-b, --target-branch` — 目标分支（不写默认 main）
- `-a, --assignee` — 指派人，逗号分隔
- `-r, --reviewer` — 评审人
- `-l, --label` — 打 label
- `--remove-source-branch` — 合并后删源分支

### 2. 列 MR

```bash
glab mr list                  # 当前仓库的 open MR
glab mr list --all            # 包括 closed/merged
glab mr list --author=@me     # 我创建的
glab mr list --reviewer=@me   # 我要审的
glab mr list --assignee=@me   # 指派给我的
glab mr list --draft          # 仅草稿
glab mr list --label="bug"    # 按 label 过滤
glab mr list --search "退款"   # 标题/描述搜索
glab mr list --per-page 50    # 翻页
```

### 3. 看 MR

```bash
glab mr view 271              # 看详情
glab mr view 271 --comments   # 含评论
glab mr view 271 --web        # 浏览器打开
glab mr view                  # 不传 ID = 看当前分支对应的 MR
glab mr diff 271              # 看代码 diff（彩色）
glab mr diff 271 | less       # 翻页看
```

### 4. 评审别人的 MR

```bash
glab mr checkout 271          # 自动建本地分支并切过去
glab mr note 271 -m "LGTM"    # 评论
glab mr approve 271           # 批准
glab mr revoke 271            # 撤销批准
```

### 5. 合并 / 关闭

```bash
glab mr merge 271                          # 合并（交互确认）
glab mr merge 271 -y                       # 不问直接合
glab mr merge 271 --squash --remove-source-branch
glab mr merge 271 --when-pipeline-succeeds # 等 CI 绿了再合
glab mr close 271
glab mr reopen 271
```

---

## 🥉 三、Repo（中频）

| 命令                     | 说明               |
| ------------------------ | ------------------ |
| `glab repo view`         | 看仓库元信息       |
| `glab repo view --web`   | 浏览器打开仓库     |
| `glab repo clone`        | 克隆仓库           |
| `glab repo fork`         | fork 仓库          |
| `glab repo search`       | 搜仓库             |
| `glab repo contributors` | 贡献者列表         |
| `glab repo archive`      | 归档仓库（管理员） |

```bash
glab repo view business-travel-fed-test/wmmos
glab repo view --web                              # 在浏览器打开当前仓库
glab repo clone business-travel-fed-test/wmmos
glab repo search --search "wmmos"                 # 搜行云上的仓库
```

---

## 四、Issue（中频）

| 命令                    | 说明     |
| ----------------------- | -------- |
| `glab issue create`     | 建 issue |
| `glab issue list`       | 列 issue |
| `glab issue view <id>`  | 看 issue |
| `glab issue close <id>` | 关闭     |
| `glab issue note <id>`  | 评论     |

```bash
glab issue list --assignee=@me
glab issue create -t "标题" -d "描述" -l "bug,前端" -a zhangsan
glab issue view 42 --comments --web
```

> 注：行云 issue 用得不多（公司主用「需求管理」），但偶尔仓库里会用。

---

## 五、API（脚本化必备）

高级用法 —— 直接调任意 GitLab API，最灵活。

```bash
# REST
glab api version
glab api user
glab api projects/business-travel-fed-test%2Fwmmos
glab api projects/business-travel-fed-test%2Fwmmos/merge_requests?state=opened

# 分页
glab api --paginate projects/business-travel-fed-test%2Fwmmos/merge_requests

# 用 jq 提取字段
glab api projects/business-travel-fed-test%2Fwmmos/merge_requests?state=opened \
  --jq '.[] | {iid, title, author: .author.username}'

# POST / PUT
glab api -X POST projects/.../merge_requests -f title="xxx" -f source_branch="..."

# GraphQL
glab api graphql -f query='query { currentUser { name } }'
```

**关键 flag**：
- `--paginate` — 自动翻页
- `--jq <expr>` — 内建 jq 处理 JSON
- `-f key=value` — POST 表单字段
- `-F key=@file` — 上传文件
- `-X METHOD` — 改 HTTP 方法

---

## 六、Auth / Config（低频，配一次就行）

```bash
glab auth status                        # 看登录状态
glab auth login --hostname coding.jd.com  # 登录
glab auth logout --hostname coding.jd.com # 登出

glab config get host                    # 看当前 host
glab config set -h coding.jd.com editor vim
glab config set check_update false      # 关闭更新提示
```

---

## 七、其他偶尔会用的

| 命令                                 | 用途                   |
| ------------------------------------ | ---------------------- |
| `glab release list`                  | 看 tag/release         |
| `glab release create v1.0.0`         | 发版                   |
| `glab snippet create -t "x" file.sh` | 上传代码片段           |
| `glab variable list`                 | 看仓库 CI 变量         |
| `glab schedule list`                 | 看定时流水线           |
| `glab token rotate`                  | 轮换 PAT（自服务）     |
| `glab label list`                    | 看仓库 label           |
| `glab user events`                   | 看自己最近的活动       |
| `glab search code "useEffect"`       | 在仓库代码里搜（BETA） |

---

## 八、效率技巧

### 1. 设别名

```bash
glab alias set co 'mr checkout'           # glab co 271
glab alias set mine 'mr list --author=@me'
glab alias set ci-fail 'ci list -s failed'
glab alias list
```

### 2. 默认仓库

进入仓库目录后，`glab` 会从 `git remote` 自动识别，所有命令默认对当前仓库生效。如果不在仓库目录，加 `-R business-travel-fed-test/wmmos`。

### 3. 输出格式（脚本友好）

```bash
glab mr list --output json | jq '.[] | .iid'
glab mr view 271 --output json
```

### 4. Shell 补全

```bash
# zsh（你的 shell）
echo 'autoload -U compinit && compinit' >> ~/.zshrc  # 如果没启
source <(glab completion -s zsh)
# 一次性写入：
glab completion -s zsh > /opt/homebrew/share/zsh/site-functions/_glab
```
