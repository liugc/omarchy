---
name: omarchy-notes
description: >
  把会话里已经讲清楚或已经修好的 omarchy 技巧、配置、故障排查沉淀成一篇中文 markdown 笔记，
  写进笔记仓库（默认 /home/godson/Projects/omarchy），然后 git commit + push。
  只要用户说「记录」「记一下」「记下来」「帮我记」「保存这个」「写个笔记」「记到笔记里」
  「这个存一下」，或英文 request "record this / save this as a note"，就必须使用本 skill ——
  哪怕用户只说这两个字、没给任何内容，也要按会话上下文总结后落盘。
  不适用于：改配置、写代码、修 bug 本身（这些是记录的对象，不是记录本身）。
---

# omarchy 笔记记录

笔记仓库：`/home/godson/Projects/omarchy`（GitHub `liugc/omarchy`，public，默认分支 `main`）。

一篇笔记 = 一个主题 = 仓库根目录下的一个 `<slug>.md`。同一主题再次记录时**更新原文件**（upsert），
不新建近似重复的副本 —— 否则几个月后搜「截图快捷键」会命中好几篇互相矛盾的文件，笔记就失效了。

如果用户或环境变量 `OMARCHY_NOTES_DIR` 指了别的目录，以它为准（换仓库或跑测试时会用到），
此时仓库地址、远程名不一定是 origin，按 `git -C <notes> remote -v` 实际结果来。

## 一、确定记什么

- 用户消息里已经写明要记的内容 → 直接用，别再自行扩写。
- 用户只说了「记录」「记一下」这类词 → 总结本会话中**最近一个已经收尾的主题**：
  遇到的症状、根因、最终做法、验证方式。
- 会话里同时有好几个候选主题（刚聊完 A 又聊 B）→ 用 `ask` 让用户挑，别猜。
- 内容只能来自本会话：实际跑过的命令、真实路径、真实报错文本、验证过的结论。
  凭记忆补一条没验证过的参数或路径，是这个 skill 最容易犯也最有害的错 —— 笔记是给人以后照着做的，
  一条假命令比少写一段更糟。确实需要但没验证的信息，写进 `## 备注` 并标注「未验证」，
  不要混进正文冒充事实。

主题必须能独立成篇：一句中文标题 + 一个 ASCII kebab-case slug，
例如 `hyprland-screenshot-to-clipboard`、`waybar-battery-icon-missing`。
slug 定下就固定，同一主题以后继续用它。

## 二、先查重，再决定新建还是更新

在笔记仓库根目录的 `*.md` 里搜标题关键词和 slug 词根（中英文都搜一遍），
命中看起来是同一主题的文件就用 `read` 读全文确认。

- **命中同主题** → 合并，别覆盖：
  - 新信息是补充 → 追加 `## 更新 YYYY-MM-DD` 小节，写清这次新增了什么。
  - 新结论推翻了旧做法 → 就地改写旧段落，并补一句为什么改（旧做法在哪失效/被弃用）。
    留着过期建议比没有笔记更坑人。
  - frontmatter 的 `updated:` 改成今天，`created:` 保持不动。
- **没命中** → 新建 `<slug>.md`。

只动这一篇，不要顺手重排、改名或"整理"别的笔记。

## 三、写文件

日期不要凭记忆写，用 `date +%F` 取。模板结构照抄（frontmatter 字段名不要改，方便以后检索）：

```markdown
---
title: <中文标题>
slug: <slug>
category: fix | tip
tags: [hyprland, keybinding]
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# <中文标题>

## 场景 / 问题

用户视角的现象，含真实报错文本（原样粘贴，不要翻译）。

## 原因

根因，一两句。纯技巧型笔记（category: tip）可省略本节。

## 解决

按顺序写实际生效的步骤。命令、路径、配置片段保持原文，代码块标语言：

```bash
<实际跑过的命令>
```

```ini
<实际改过的配置片段>
```

## 验证

怎么确认解决了的（命令输出、界面现象）。

## 备注

坑、限制、未验证的猜测、上游 issue / 文档链接。
```

正文用中文，命令、路径、配置键名保持英文原样。不要写空壳小节 —— 某节没有内容就整节删掉，
不要留 `TODO`。

## 四、提交并推送

```bash
git -C <notes> add <file>
git -C <notes> commit -m "docs(<slug>): <中文标题>"
git -C <notes> push
```

- 提交前确认身份已配置：`git -C <notes> config user.email` 为空就先补
  `git config --local user.email "$(gh api user --jq '.id')+$(gh api user --jq .login)@users.noreply.github.com"`
  和 `user.name "$(gh api user --jq .login)"`。
- 首次推送新分支用 `git push -u origin main`（该机全局开了 `push.autosetupremote`，通常直接 `git push` 就会自动建上游）。
- push 失败（断网、token 失效、远端拒绝）→ 本地 commit 保留住，明确告诉用户「已提交本地、未推送」和失败原因。
  不要 force push，不要反复重试，也不要改用别的方式硬推。

## 五、回报用户

三行以内：

1. 文件路径（相对仓库根）
2. 一句话摘要（这篇记了什么）
3. 新建还是更新了已有笔记 + commit hash / 是否已推送

然后把文件内容要点贴 1-2 条给用户核对，方便他立刻纠正。

## 红线

- 不修改笔记仓库里主题之外的文件（含 .git 配置，除上面补身份那一步）。
- 不把会话里的密钥、token、内网地址写进笔记；遇到就脱敏成 `<redacted>` 并提一句。
- 系统路径、命令一律以本会话实际输出为准；没跑过就不要写成"已验证"。
