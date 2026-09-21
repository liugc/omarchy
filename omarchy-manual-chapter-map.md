---
title: Omarchy 官方手册 51 章地图与速查入口
slug: omarchy-manual-chapter-map
category: tip
tags: [manual, navigation, cli, keybinding]
created: 2026-09-21
updated: 2026-09-21
---

# Omarchy 官方手册 51 章地图与速查入口

## 场景 / 问题

想改某项设置、查某个快捷键时，不知道官方手册里该翻哪一章。更麻烦的是手册**不在**已安装的 `$OMARCHY_PATH` 里：本机 `OMARCHY_PATH=/usr/share/omarchy`，该目录下没有 `manual/`，也没找到 `omarchy manual` 之类的命令，只有 repo 检出（`/home/godson/omarchy/manual`）和线上镜像两处可读。

## 解决

### 手册在哪

- 权威来源：repo 的 `manual/`（本机 `/home/godson/omarchy/manual`），repo README 的 “The Omarchy Manual” 一节逐章链接了全部 51 章。
- 线上镜像：[learn.omacom.io](https://learn.omacom.io/2/the-omarchy-manual)（README 说明截图也托管在这里）。
- 菜单入口：`default/omarchy/omarchy-menu.jsonc` 的 `learn.omarchy` → `https://omarchy.org/manual/`。
- 打包情况：`manual/` 不打进任何包，只存在于 repo（`docs/file-layout.md:29`：`manual/`、`agents/skills/`、`docs/`、`test/`、`plans/` 均为 repo-only）。

### 三条最快入口

1. `Super + Space` 打开 Omarchy 菜单 —— 文档里几乎所有“去哪设置”都由菜单项 + 热键双路径给出。
2. `Super + K` 看全部键位（`07-hotkeys.md` 是手册里最长的一章，374 行）。
3. 裸跑 `omarchy` 看 CLI 命令中心，再 `omarchy <group> --help` 深入（`14-omarchy-cli.md`）。

### 章节地图（与 repo README 分组一致）

| 分组 | 章节 | 内容 |
| --- | --- | --- |
| 开篇 | 01 | 欢迎；Arch + Hyprland + Quickshell 的定位与预装软件 |
| The Basics | 02–04 | ISO 安装（含双系统与不加密路径）；从 mac/Windows 迁移的键位对照；导航、布局、分组、scratchpad |
| The Basics | 05–07 | 顶栏（quickshell 单进程，各面板热键与 `shell.json` 配置）；主题；完整键位表（含 Tmux/Ghostty/Neovim 键位与 CapsLock 表情输入） |
| The Basics | 08–14 | 统一剪贴板与历史；提醒；notices（时间/电池/天气）；OCR 与听写；截图与录屏；toggle / idle / 屏保；`omarchy` CLI |
| The Applications | 15–20 | Terminal 与 Tmux 布局函数；Neovim(LazyVim)；AI agent（mise 惰性 stub、默认 agent、崩溃诊断、用量面板）；开发工具与 Mise/Docker；shell 工具（fzf/zoxide/rg/eza/fd/bat）；shell 函数（compress、iso2sd、rsync 监视、SSH 转发） |
| The Applications | 21–29 | TUI 集合；GUI 集合；浏览器与默认浏览器设置；商业应用安装通道；自定义 web app；游戏；填 PDF；Windows VM；Arch/AUR 包管理 |
| Configuration | 30–43 | 更新与四通道/快照回滚；dotfiles 与 hooks；shell 插件体系；显示器与缩放；输入设备；网络与防火墙；挂起/休眠与电源模式；指纹与 Fido2；字体；背景（含视频壁纸）；Starship 提示符；品牌（Plymouth/屏保/About）；常见定制；自制主题与模板 |
| The Rest | 44–51 | Intel Mac 支持；故障排查；FAQ；系统快照（仅 Limine）；安全设计（LUKS/ufw/改密码/转交机器）；非默认平台（Asahi/Steam Deck/NixOS）；双系统安装；无人值守安装 |

## 验证

结构核查（在 `/home/godson/omarchy/manual` 下实跑）：

```bash
ls *.md | wc -l                      # 51，编号 01–51 连续无缺
wc -l *.md                           # 合计 2855 行
grep -ohE '\(images/[^)]+\)' *.md | tr -d '()' | sort -u > /tmp/ref.txt
ls images | sed 's|^|images/|' | sort > /tmp/have.txt
comm -3 /tmp/ref.txt /tmp/have.txt   # 无输出 → 44 张图引用与文件一一对应，无缺失无冗余
```

章节内部链接检查（`]\(NN-slug.md)` 逐个 `[[ -f ]]`）无任何 BROKEN；`README.md` 对 `manual/*.md` 的覆盖检查也无缺失（51/51）。

热点入口抽查（`grep -rlF` 确认这些字符串确实出现在手册正文里）：`Super + Ctrl + Shift + Space`（06/07）、`Super + K`（03/07）、`Super + Ctrl + V`（03/07/08）、`omarchy menu summon`（14）、`omarchy reminder`（09）、`omarchy plugin`（05/32）。

## 备注

- 手册文件名自带章节号（`07-hotkeys.md`），README 与章间链接都用文件名，所以重编号或改名会同时打断 repo README 与站内互链；新增内容应续在末尾并更新 README。
- 章节结构会随版本演进（当前 51 章，2026-09-21 检出状态）；离线判断内容是否最新，以本机 repo 的 `manual/` 与 `git log` 为准，不要以线上镜像为准。
- 手册正文里的键位/命令与实现是两套文本，改配置前若涉及行为差异，直接查 `bin/` 与 `shell/` 更可靠。
