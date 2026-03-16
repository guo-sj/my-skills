---
name: gh-issues
description: "查看当前目录对应的 GitHub 仓库的 issues。当用户提到 'issues'、'查看 issue'、'github issues'、'当前仓库的 issue'、'列出 issue' 时触发。"
argument-hint: ""
allowed-tools: Bash(gh --version*), Bash(gh auth status*), Bash(gh repo view*), Bash(gh issue list*), Bash(gh issue view*)
---

# GitHub Issues 查看 Skill

## 1. Pre-flight 检查

按顺序执行，遇到失败立即停止：

1. `gh --version` — 失败则提示：
   > "未检测到 gh CLI，请先安装：https://cli.github.com"

2. `gh auth status` — 失败则提示：
   > "gh 未登录，请先运行：`gh auth login`"

3. `gh repo view --json nameWithOwner` — 失败则：
   - stderr 包含 "not a git repository" → 提示 "当前目录不是 git 仓库"
   - 其他原因 → 提示 "当前仓库未托管在 GitHub 或未配置 GitHub remote，此 skill 仅支持 GitHub"

## 2. 列出 Issues

运行：

```bash
gh issue list --limit 50
```

- 若 stderr 包含 "Issues are disabled" → 提示 "该仓库未启用 Issues 功能"，停止
- 其他失败 → 输出原始错误，停止
- 返回空列表 → 提示 "当前仓库没有 open issues"，直接退出，**不进入步骤 3**
- 成功 → 展示列表，并在末尾附注：
  > "（showing first 50 open issues；如有更多请通过 `gh issue list` 查看）"

## 3. 交互循环

展示列表后，维护一个**共享重试计数器**（每次回到此步骤时重置为 0）。

提示用户：
> "要查看某个 issue 的详情吗？请输入 issue 编号，或输入 n 退出。"

根据用户输入处理：

**有效整数：**
运行 `gh issue view <number> --comments`
- 若 stderr 包含 "Could not resolve" → 提示 "Issue #N 不存在，请确认编号后重试"，计数器 +1，重新提示
- 其他失败 → 输出原始错误，停止
- 成功 → 展示详情，转到步骤 4

**输入 "n"：**
退出，输出 "已退出。"

**其他输入：**
提示 "请输入有效的 issue 编号或 n 退出"，计数器 +1，重新提示

**计数器达到 3：**
退出，输出 "输入无效次数过多，已退出。"

## 4. 展示详情后

展示 issue 详情后，提示：
> "要基于此 issue 开始开发吗？如需创建本地分支，可运行：
> `git checkout -b fix/<number>-<slug>`"
>
> （slug 生成规则：去除非 ASCII 字符，转小写，非字母数字替换为连字符，首尾去连字符，截断至 40 字符；若 slug 为空则省略，仅用 `fix/<number>`）

提示后回到步骤 3（计数器重置为 0）。
