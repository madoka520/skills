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
- **个人 skill 全量备份在 `github.com/madoka520/skills`**(扁平顶层目录,一个 skill 一个目录):修改/新增任何个人 skill 后,同步把该 skill 目录提交推送到这个仓库 —— 它是唯一权威备份,本机 `.agents/.zcode/.claude` 里的软链或副本都依赖实体;漏推=下次只能恢复到旧版
