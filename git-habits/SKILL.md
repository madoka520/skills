---
name: git-habits
description: 我的 Git 工作流习惯(所有仓库、所有项目)。完成任何代码改动后使用:立即自动 commit 并 push 到 GitHub,无需询问;只带本次相关文件;提交信息中文、conventional 前缀、写清楚为什么。
---

# Git 习惯

- 每完成一次功能/修复/调整,立即 `git commit` 并 `git push` 到 GitHub,**不需要询问** —— 用户已持久授权(所有仓库通用)
- 只提交本次改动相关的文件;用户工作区里他自己未完成的修改绝不捎带
- 提交信息沿用仓库既有风格:中文,`feat/fix/refactor/style/chore` 前缀,正文写清楚为什么(动机/根因),不只是改了什么
- 新分支首次推送用 `git push -u origin <branch>`
- 多仓库联动时(如组件库发版 + 使用方升级),各自仓库独立提交,信息里互相引用版本号
- **个人 skill 全量备份在 `github.com/madoka520/skills`**:`~/.agents/skills` 就是该仓库的**本地克隆**(实体层)——修改任何个人 skill 后,直接在 `~/.agents/skills` 里 `git add/commit/push` 即完成备份,**不要用临时 clone 绕路**。`.zcode/.claude` 里是指向实体的软链/junction,删目录前先 `ls -la` 确认不是实体
