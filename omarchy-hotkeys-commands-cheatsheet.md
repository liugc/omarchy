---
title: Omarchy 命令与热键速查表
slug: omarchy-hotkeys-commands-cheatsheet
category: tip
tags: [keybinding, cli, cheatsheet, tmux, neovim]
created: 2026-09-21
updated: 2026-09-21
---

# Omarchy 命令与热键速查表

## 场景 / 问题

键位分散在手册 51 章里（权威表是 `07-hotkeys.md`，374 行），CLI 又有 70 多个分组，名称容易记混。需要一张能直接抄的表。

## 解决

### 权威来源与自证命令

- 屏幕上看全部主要绑定：`Super + K`（Tmux 绑定 `Super + Alt + K`，Herdr 绑定 `Super + Ctrl + K`）。
- 改绑定：`~/.config/hypr/bindings.lua`；Tmux 键位：`~/.config/tmux/tmux.conf`。
- CLI 总入口：裸跑 `omarchy` 打命令中心；`omarchy commands [--all|--json|--check]`；任意层级 `omarchy <group> [<command>] --help`。
- 本机实测：`omarchy commands --check` → `Command metadata check passed (440 commands)`；`omarchy commands --all` → 452 行（含 `# omarchy:hidden=true` 的隐藏命令）；分组 72 个。

### 导航与窗口

| 热键 | 作用 |
| --- | --- |
| `Super + Space` / `Super + Alt + Space` / `Super + Escape` | Omarchy 菜单 / 应用菜单 / 系统菜单 |
| `Super + Ctrl + L` | 锁屏 |
| `Super + W` 或 `Super + Q` / `Ctrl + Alt + Del` | 关当前窗口 / 关全部窗口 |
| `Super + T` / `Super + J` / `Super + P` | 平铺↔浮动 / 横竖位置互换 / 伪窗口样式 |
| `Super + O` | 弹出窗口（sticky + floating） |
| `Super + L` | dwindle ↔ scrolling 布局 |
| `Super + F` / `Super + Alt + F` / `Super + Ctrl + F` | 全屏 / 全宽 / 窗口内全屏 |
| `Super + 1/2/3/4`，`Super + Tab`，`Super + Shift + Tab`，`Super + Ctrl + Tab` | 跳工作区：指定 / 下一个 / 上一个 / 前一个 |
| `Super + Shift + 1/2/3/4` / `Super + Shift + Alt + 1/2/3/4` | 移动窗口到工作区 / 移动但不跟随 |
| `Super + S` 或 `Super + Grave` / `Super + Alt + S` 或 `Super + Shift + Grave` | 切换 scratchpad / 把窗口放进 scratchpad |
| `Super + 方向键` / `Super + Shift + 方向键` | 移动焦点 / 交换窗口位置 |
| `Super + Alt + 方向键` / `Super + Ctrl + 方向键` | 分组内移动窗口 / 分组内切换窗口 |
| `Super + Minus`/`Equal`（加 `Alt` 更小步、加 `Ctrl` 更大步） | 调整窗口尺寸 |
| `Super + Alt + Home` / `Super + Home` | 保存窗口宽度 / 恢复 |
| `Super + 左键` / `Super + 右键` / `Super + 滚轮` | 拖动窗口 / 缩放窗口 / 滚轮切工作区 |
| `Super + G` / `Super + Alt + G` / `Super + Alt + Tab` / `Super + Alt + 1-5` | 建组或解组 / 移出组 / 组内循环 / 跳到组内第 n 个 |
| `Super + /` / `Super + Alt + /` | 显示器缩放档位前进 / 后退 |
| `Alt + Tab` / `Alt + Shift + Tab` | 当前工作区窗口前后循环 |
| `Ctrl + Alt + Tab` / `Ctrl + Alt + Shift + Tab` | 显示器间焦点前后切换 |

### 系统面板与即时操作

| 热键 | 作用 |
| --- | --- |
| `Super + Ctrl + A/B/W/D/P` | 音频 / 蓝牙 / Wi-Fi / 显示 / 电源面板 |
| `Super + Ctrl + Alt + D` | 日历面板 |
| `Super + Ctrl + 1-9` | 按位置切换顶栏面板 |
| `Super + Ctrl + S` | Share 菜单（LocalSend） |
| `Super + Ctrl + T` | Activity（btop） |
| `Super + Ctrl + C` | 捕获控制（截图/录屏/选择器） |
| `Super + Ctrl + O` / `Super + Ctrl + H` | Toggle 菜单 / Hardware 菜单 |
| `Super + Ctrl + Q` / `Super + Ctrl + E` | 计算器 / 表情选择器 |
| `Super + Ctrl + .` | 媒体转码 |
| `Super + Shift + Ctrl + A` | 选择 AI agent |
| `Shift + 亮度±` / `Alt + 亮度±` | 亮度到最大最小 / 1% 精细调整 |
| `Alt + 音量±` | 音量 1% 精细调整 |
| `Alt + Play` / `Alt + Shift + Play` | 下一曲 / 上一曲 |

### 启动应用

| 热键 | 作用 |
| --- | --- |
| `Super + Return` / `Super + Alt + Return` / `Super + Ctrl + Return` | 终端 / Tmux 终端 / Herdr |
| `Super + Shift + Return` / `Super + Shift + Alt + B` | 浏览器 / 隐私窗口 |
| `Super + Shift + F` / `Super + Shift + Alt + F` | 文件管理器 / 在终端当前目录打开 |
| `Super + Shift + N` | 编辑器（Neovim） |
| `Super + Shift + M` / `Super + Shift + Alt + M` | Spotify / cliamp |
| `Super + Shift + /` | 1Password |
| `Super + Shift + C` / `Super + Shift + E` / `Super + Shift + Alt + E` | HEY 日历 / HEY 邮件 / 新邮件 |
| `Super + Shift + A` / `Super + Shift + Alt + A` | ChatGPT / Grok |
| `Super + Shift + G` / `Super + Shift + Alt + G` / `Super + Shift + Ctrl + G` | Signal / WhatsApp / Google Messages |
| `Super + Shift + P` / `Super + Shift + S` | Google Photos / Google Maps |
| `Super + Shift + D` / `Super + Shift + O` / `Super + Shift + W` | LazyDocker / Obsidian / Omawrite |
| `Super + Shift + X` / `Super + Shift + Alt + X` / `Super + Shift + Y` | X / X Compose / YouTube |

### 剪贴板、捕获、通知

| 热键 | 作用 |
| --- | --- |
| `Super + C` / `Super + X` / `Super + V` / `Super + Ctrl + V` | 复制 / 剪切（终端内无效）/ 粘贴 / 剪贴板历史 |
| `Print Screen` / `Alt + Print Screen` | 截图 / 录屏（再按一次停止） |
| `Super + Print Screen` / `Super + Ctrl + Print Screen` | 取色 / 区域 OCR 到剪贴板 |
| `Super + Alt + [` / `Super + Alt + ]` | 录屏画中画摄像头变小 / 变大 |
| `Alt + Shift + L` / `Alt + Shift + D` | 复制当前 URL / 下载页面视频到 `~/Videos` |
| `Super + Ctrl + X` / `F9` | 听写开关 / 按住说话（需先装 Dictation） |
| `Super + ,` / `Super + Shift + ,` / `Super + Ctrl + ,` / `Super + Alt + ,` / `Super + Shift + Alt + ,` | 关最新通知 / 关全部 / 静音开关 / 呼出最近一条 / 通知历史 |

### 样式、开关、提醒、notices

| 热键 | 作用 |
| --- | --- |
| `Super + Ctrl + Shift + Space` / `Super + Ctrl + Space` | 换主题 / 换主题背景 |
| `Super + Backspace` / `Super + Ctrl + Backspace` | 窗口透明度开关 / 单窗口正方形比例 |
| `Super + Ctrl + I` / `Super + Ctrl + N` | 空闲锁定开关 / 夜灯开关 |
| `Super + Ctrl + Delete` / `Super + Ctrl + Alt + Delete` | 笔记本内屏开关 / 内屏镜像 |
| `Super + Shift + Space` / `Super + Shift + Backspace` | 顶栏显隐 / 窗口间隙开关 |
| `Super + Ctrl + Alt + F` | 全屏桌面（顶栏 + 间隙一起隐藏） |
| `Shift + Mute` / `Shift + Play` | 切下一个音频输出 / 下一个媒体源 |
| `Super + Ctrl + R` / `Super + Ctrl + Alt + R` / `Super + Ctrl + Shift + R` | 设提醒 / 看全部 / 清空 |
| `Super + Ctrl + Alt + T` / `Super + Ctrl + Alt + B` / `Super + Ctrl + Alt + W` | 时间 / 电池 / 天气通知 |

### Tmux（prefix `Ctrl + Space`，`Ctrl + B` 亦可）

| 键位 | 作用 |
| --- | --- |
| `Prefix + v` / `Prefix + h` | 竖切 / 横切 pane（无 prefix：`Alt + Enter` 横切、`Alt + Shift + Enter` 竖切） |
| `Prefix + x` / `Prefix + z` | 关 pane / pane 全屏切换（无 prefix：`Alt + Escape` 关 pane） |
| `Ctrl + Alt + 方向` / `Ctrl + Alt + Shift + 方向` | pane 间移动 / 调整 pane 尺寸 |
| `Prefix + c` / `Prefix + k` / `Prefix + r` | 新窗口 / 关窗口 / 重命名 |
| `Alt + 1-9` / `Alt + 方向左右` / `Alt + Shift + 方向左右` | 跳窗口 / 换窗口 / 移动窗口 |
| `Prefix + C` / `Prefix + K` / `Prefix + R` / `Prefix + N` / `Prefix + P` | 新/关/重命名/下一个/上一个会话 |
| `Prefix + s` / `Prefix + d` / `Alt + 方向上下` | 列会话 / detach / 会话间移动 |
| `Prefix + [` 后 `v` 选、`y` 复制 | vi 风格 copy mode |
| `Prefix + q` / `Prefix + ?` / `Prefix + :` | 重载配置 / 键位表 / 命令提示符 |
| `tdl <ai> [<second_ai>]` / `tdlm <ai>` / `tsl <count> <command>` | 开发布局 / 按子目录开布局 / 多 pane swarm（须在 Tmux 会话内运行） |

### Ghostty（`Install > Terminal` 安装）

| 键位 | 作用 |
| --- | --- |
| `Ctrl + Shift + E` / `Ctrl + Shift + O` | 下方 / 旁边分屏 |
| `Ctrl + Alt + 方向` / `Super + Ctrl + Shift + 方向` / `Super + Ctrl + Shift + Alt + 方向` | 分屏间移动 / 调整 10 行 / 调整 100 行 |
| `Ctrl + Shift + T` / `Ctrl + Shift + 方向` / `Alt + 数字` | 新标签 / 换标签 / 跳到指定标签 |
| `Shift + PgUp/PgDn` / `Ctrl + 左键` | 翻历史 / 用浏览器打开链接 |

### Neovim（LazyVim）

| 键位 | 作用 |
| --- | --- |
| `Space` / `Space Space` | 显示命令提示 / 模糊搜索文件 |
| `Space E` / `Ctrl + W W` / `Ctrl + 左右方向` | 侧栏开关 / 侧栏与编辑区切换 / 调整侧栏宽度 |
| `Space G G` / `Space S G` | LazyGit / 搜索文件内容 |
| `Shift + H` / `Shift + L` / `Space B D` | 左文件标签 / 右文件标签 / 关标签 |
| 侧栏内 `A` / `Shift + A` / `D` / `M` / `R` / `?` | 新建文件 / 新建目录 / 删除 / 移动 / 重命名 / 帮助 |

### 表情与快捷补全（CapsLock 是 compose 键）

- 完整表情选择器：`Super + Ctrl + E`。
- `CapsLock M <字母>`：`S`😄 `C`😂 `L`😍 `V`✌️ `H`❤️ `Y`👍 `N`👎 `F`🖕 `W`🤞 `R`🤘 `K`😘 `E`🙄 `I`😉 `P`🙏 `D`🤤 `M`💰 `X`🎉 `1`💯 `T`🥂 `O`👌 `G`👋 `A`💪 `B`🤯。
- 补全：`CapsLock Space Space`→em dash、`CapsLock Space N`→姓名、`CapsLock Space E`→邮箱；扩展改 `~/.XCompose` 后跑 `omarchy-restart-xcompose`。

### CLI 常用命令

```bash
omarchy                      # 命令中心：常用命令 + 分组列表
omarchy commands --all       # 含隐藏命令的完整清单（本机 452 行）
omarchy commands --json      # 机器可读
omarchy commands --check     # 校验命令元数据与路由
omarchy <group> --help       # 分组内命令
omarchy update               # 更新 Omarchy 与系统包
omarchy theme list / theme set <name>
omarchy font list
omarchy screenshot
omarchy debug                # 排障信息
omarchy menu summon style.theme   # 直接打开主题选择器
omarchy menu toggle system / omarchy menu close
```

### CLI 分组全览（本机 `omarchy` 输出，72 个）

```
agent audio bar battery bluetooth branch branding brightness capture channel clipboard
cmd config debug default dev disk display dns drive file finalize font games hibernation
hook hw hyprland install installed launch menu migrate mise monitor network notification
openclaw osd pkg plugin plymouth power powerprofiles refresh reinstall reminder remove
restart screensaver shell setup snapshot sudo system tailscale theme toggle transcode tui
update version voxtype weather webapp wifi windows
```

## 验证

- 热键表逐条来自本机手册 `manual/07-hotkeys.md`（含 Tmux / Ghostty / Neovim / 表情小节），原文照抄。
- CLI 事实来自本机实跑：`omarchy`（分组与常用命令输出）、`omarchy commands --check` → `Command metadata check passed (440 commands)`、`omarchy commands --all | wc -l` → `452`、分组计数 → `72`。

## 备注

- 手册里的键位与 `~/.config/hypr/bindings.lua` 的实现是两套文本：自己改过绑定后，以 `Super + K` 的实际显示为准。
- `omarchy commands --check` 会在本地校验元数据，怀疑手册/CLI 不一致时先跑它。
- 上游给脚本用的稳定入口是 `omarchy menu summon <点分 id>`（如 `style.theme`）与 `omarchy bar set omarchy.clock format "dddd h:mm AP"` 之类的分层命令，而不是模拟按键。
- 手册 07 章未覆盖的命令（如 `omarchy windows vm launch --keep-alive`、`omarchy plugin clone`）以 `omarchy <group> --help` 为准，本表不重复罗列。
