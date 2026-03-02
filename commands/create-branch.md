开发前准备工作：拉取 jd-release 最新代码，并基于它创建新分支。

## 步骤

1. 询问用户本次开发的类型：
   - **fix**：修复 bug
   - **feature**：开发新功能

2. 询问用户对本次工作的简短描述（用英文小写字母和连字符，例如 `hotel-map-drag-fix`）

3. 获取今天的日期，格式为 `mmdd`（例如 2 月 11 日 → `0211`）

4. 执行以下 git 操作：
   ```bash
   # 切换到 jd-release 并拉取最新代码
   git checkout jd-release
   git pull origin jd-release

   # 基于 jd-release 创建新分支
   git checkout -b <新分支名>
   ```

5. 分支命名规则：
   - Bug 修复：`fix-[mmdd]-[描述]`，例如 `fix-0211-hotel-map-drag`
   - 功能开发：`feature-[mmdd]-[描述]`，例如 `feature-0211-ai-workflow`

6. 操作成功后，告知用户当前所在分支，并提示可以开始开发了。

## 注意事项

- 如果 `git pull` 失败（如有未提交的修改），请提示用户先处理本地变更
- 分支描述只使用英文小写字母和连字符（`-`），不要使用空格或中文
