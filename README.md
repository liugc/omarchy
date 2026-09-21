# omarchy 笔记

个人笔记仓库：记录 omarchy（Arch + Hyprland）的技巧、配置与故障排查。仓库根目录下每个 `*.md` 是一篇笔记。

## 笔记导航

| 笔记 | 类型 | 更新 | 标签 | 内容 |
| --- | --- | --- | --- | --- |
| [MacBook 合盖唤醒后黑屏约 85 秒、电源键无响应（雷雳控制器恢复失败）](macbook-suspend-thunderbolt-resume-hang.md) | fix | 2026-09-21 | macbook, suspend, thunderbolt, xhci, systemd, sleep-hook | MacBookPro14,1 合盖唤醒黑屏、`xhci_hcd ... not ready ... giving up`；最终修法是 systemd sleep hook 在睡眠路径 unbind/rebind thunderbolt 驱动，恢复路径从约 85 秒降到约 6.9 秒，并记录了走不通的路与测量方法论 |
| [Omarchy 官方手册 51 章地图与速查入口](omarchy-manual-chapter-map.md) | tip | 2026-09-21 | manual, navigation, cli, keybinding | 手册 `manual/` 的定位（repo-only，不在 `$OMARCHY_PATH`）、三条最快入口、按 repo README 分组的章节地图，以及结构核查命令 |
| [Omarchy 命令与热键速查表](omarchy-hotkeys-commands-cheatsheet.md) | tip | 2026-09-21 | keybinding, cli, cheatsheet, tmux, neovim | 导航/窗口、系统面板、启动应用、剪贴板与捕获、通知与开关、Tmux、Ghostty、Neovim、CapsLock 表情与补全，外加 CLI 常用命令与 72 个分组全览 |
| [Hyprland 延长触摸板防误触（disable-while-typing）恢复延迟](hyprland-touchpad-dwt-delay.md) | tip | 2026-09-19 | hyprland, touchpad, libinput, lua, macbook, applespi | 把触摸板 DWT 恢复延迟从 libinput 写死的 500ms 延长到 1.5s：用 `hl.on("input.keyboard.key")` + `hl.timer` 在 `~/.config/hypr/input.lua` 里自行实现 |

## 约定

- 一篇笔记一个主题，文件名即 slug（ASCII kebab-case，如 `hyprland-touchpad-dwt-delay`）；同一主题再次记录时更新原文件，不新建近似重复的副本。
- frontmatter 字段固定为 `title` / `slug` / `category`（`fix` 或 `tip`）/ `tags` / `created` / `updated`，方便按关键词检索。
- 正文只写本机实际跑过、验证过的命令与输出；没验证的内容标注「未验证」，不混进正文冒充事实。
- 敏感信息（密钥、token、内网地址）脱敏为 `<redacted>` 后再落盘。

## 检索

```bash
grep -rl 触摸板 *.md          # 按中文关键词找笔记
grep -n '^tags:' *.md         # 看所有标签
grep -rn 'thunderbolt' *.md   # 按英文关键词找
```

## 新增笔记

说一句「记一下」/「记录」即会走 `omarchy-notes` skill（安装在 `~/.agents/skills/omarchy-notes/`）：先在本仓库查重，同主题则合并更新、新主题则新建 `<slug>.md`，然后 `git commit` + `git push`。
