---
title: MacBook 合盖唤醒后黑屏约 85 秒、电源键无响应（雷雳控制器恢复失败）
slug: macbook-suspend-thunderbolt-resume-hang
category: fix
tags: [macbook, suspend, resume, thunderbolt, xhci, systemd, s2idle, mem_sleep]
created: 2026-09-20
updated: 2026-09-20
---

# MacBook 合盖唤醒后黑屏约 85 秒、电源键无响应（雷雳控制器恢复失败）

机器：MacBookPro14,1（Apple，无 T2），Alpine Ridge Thunderbolt 3（`04:00.0` / `05:0x` / `06:00.0` / `07:00.0`），
内核 `7.2.5-3-omarchy`，systemd 261.2-1。

## 场景 / 问题

合上盖子再掀开，屏幕不亮；按电源键没反应，要按好几次才「进去」。
偶发（约 3/13 次）彻底不回来，必须长按电源键断电重开机。

内核日志里每次恢复都出现同一组 9 行：

```
xhci_hcd 0000:07:00.0: not ready 1023ms after resume; waiting
xhci_hcd 0000:07:00.0: not ready 2047ms after resume; waiting
xhci_hcd 0000:07:00.0: not ready 4095ms after resume; waiting
xhci_hcd 0000:07:00.0: not ready 8191ms after resume; waiting
xhci_hcd 0000:07:00.0: not ready 16383ms after resume; waiting
xhci_hcd 0000:07:00.0: not ready 32767ms after resume; waiting
xhci_hcd 0000:07:00.0: not ready 65535ms after resume; giving up
thunderbolt 0000:06:00.0: not ready 1023ms after resume; giving up
pcieport 0000:05:02.0: pciehp: Slot(0): Card not present
```

伴随 `tb_cfg_read: -108` / `tb_cfg_write: -108`（ETIMEDOUT，`tb_switch_reset` 也失败）和
`pci_disable_device+0xbc/0xd0` 的 WARNING（`irq/31-pciehp` → `xhci_pci_remove`）。

## 原因

`deep`(S3) 唤醒时，Alpine Ridge 雷雳控制器 `0000:06:00.0` 和它后面的 xHCI `0000:07:00.0`
都没能恢复（noirq 阶段就已经 `-108` 超时不响应）。内核在 `xhci_hcd` 的 resume 回调里把
「等控制器 ready」的超时翻倍重试，1+2+4+8+16+32+65 秒后才放弃，然后由 pciehp 把
`0000:07:00.0` 摘除。**这段等待期间整个系统冻结**（userspace 还没 thaw），所以屏幕黑、
键盘死、电源键无效；黑屏时长等于这段内核耗时。

实测某个周期（22:38:50 合盖 → 22:40:30 恢复）：

- 进入→退出之间内核单调时钟差（单调时钟在 S3 期间停走，因此即冻结时长）＝ **85.48 s**
- 同一段墙面时钟间隔 100.7 s ⇒ 实际睡着只有约 15 s，也就是**掀盖确实唤醒了机器**，
  剩下 85 秒全是冻结
- 全部 13 次挂起中的 10 次恢复、跨 6 个启动周期（09-03 起）都是上面那组日志

另外 3 次挂起完全没有回来（没有 `PM: suspend exit`，下次是冷启动）：
`09-05 23:01:53`、`09-19 00:42:11`、`09-20 06:18:10`。

## 解决

### 1. 量化与定位（只读）

```bash
# 每次挂起/恢复的成对记录
journalctl --no-pager -o short-iso -g 'PM: suspend (entry|exit)'
# 恢复期到底在等谁
journalctl --no-pager -o short-iso -g 'not ready [0-9]+ms after resume|giving up'
journalctl --no-pager -o short-iso -g 'thunderbolt|pciehp|tb_cfg'
# 当前用的是 deep 还是 s2idle（方括号是当前选择）
cat /sys/power/mem_sleep          # s2idle [deep]
# 唤醒源
cat /proc/acpi/wakeup             # LID0 S4 *enabled / SPIT S3 *enabled
# 恢复后 USB-C 口的控制器还在不在
lspci -s 07:00.0                  # 恢复后消失
```

冻结时长用内核单调时间戳精确量（`_SOURCE_MONOTONIC_TIMESTAMP` 是内核侧时间戳，
`journalctl` 显示的行首时间是 thaw 后才写入的收报时间，不能用）：

```bash
journalctl -k -o json --no-pager --since '2026-09-20 22:38' | \
  jq -r 'select(.MESSAGE|test("PM: suspend")) | "\(._SOURCE_MONOTONIC_TIMESTAMP) \(.MESSAGE)"'
# 118918727 PM: suspend entry (deep)
# 204401967 PM: suspend exit          → (204401967-118918727)/1e6 = 85.48 s
```

### 2. 确认 s2idle 会绕开这段等待（本机内核符号表）

```bash
awk '/xhci_pci_(resume|suspend)|xhci_pci_pm_ops/ {print $1, $3}' \
  /usr/lib/modules/$(uname -r)/build/System.map
# ffffffff8220c8d0 xhci_pci_suspend
# ffffffff8220e100 xhci_pci_resume      ← 只有这两个，没有 *_noirq 变体
# （输出里另有两行对应的 __pfx_ 前缀符号，略）
```

`xhci_pci` 是内置的（不是模块），systemd 261 支持 `MemorySleepMode=`（见 `man systemd-sleep.conf`）。
suspend-to-idle 只调用 `*_noirq` 回调、且不会给设备断电，所以 `xhci_pci_resume()` 里那段
65 秒等待不会执行，雷雳链也不会被断电 → 应是本机唯一可行的修法。

### 3. 落盘

```ini
# /etc/systemd/sleep.conf.d/20-macbook-mem-sleep.conf
[Sleep]
MemorySleepMode=s2idle
```

`deep` 路线的两个补救（`pm_async=off`、雷雳链 `d3cold_allowed=0`）在原理上是死路：
`resume_noirq` 阶段本来就是同步的，而控制器在 noirq 阶段就 `-108` 不响应，调顺序救不回来。
（顺带：omarchy 的 Mac 专属参数 `pm_async=off mem_sleep_default=deep` 只在 **T2 机型**生效，
见 `/usr/share/omarchy/install/hardware/apple/fix-t2.sh`：检测 `106b:1801/1802`；
这台无 T2，所以是裸 `deep` + `pm_async=1`。）

### 4. 诊断/落地脚本

```bash
sudo bash ~/.local/share/macbook-suspend-diag/diag.sh
```

脚本用 RTC 定时唤醒自动跑 20 s 的 s2idle 循环（不需要人按任何键）→ 量出 `resume_path`
→ 落盘最优配置 → 再跑一次确认循环 → 最后做真实合盖/掀盖测试。
可选：`--all`（也测 deep 两套变体）、`--drain 600`（10 分钟睡眠耗电对比）、
`--hibernate`（验证休眠是否真能恢复）、`--undo`（一键还原）。
日志在 `/var/log/macbook-suspend-diag/`。

> 注意：脚本可以测 `deep` 变体，但历史数据显示每次 `deep` 恢复都要冻结 85 秒且有约 1/4 概率
> 彻底不回来，所以默认只测 s2idle。

## 验证

`resume_path` 从 85 s 降到个位数秒（脚本每次循环都会打印该值），并且恢复后
`lspci -s 07:00.0` 仍在、USB-C 口可用，即为修好。真实场景的验收是脚本最后的合盖/掀盖测试：
掀盖后自己亮屏、不需要按电源键。

截至写这篇笔记时：诊断、根因、修复配置和脚本都已就绪，**脚本本身还没在 sudo 下跑过**
（当前 `mem_sleep` 仍是 `[deep]`），修复效果属于未验证，见「备注」。

## 备注

- **电源键为什么"没反应"**：`/etc/systemd/logind.conf.d/10-ignore-power-button.conf` 设了
  `HandlePowerKey=ignore`（omarchy 默认，防误触）。它不是病因（冻结期间 logind 本身也冻住），
  但意味着短按永远没用，强制断电只能长按约 10 s。
- **恢复后 USB-C 口会死**：`0000:07:00.0` 被 pciehp 摘除后不会自动回来，要重启才恢复。
  s2idle 下设备不断电，这个问题应一并消失（未验证）。
- **06:18 那次是另一条路**：电池到 2%（`/etc/UPower/UPower.conf` 里 `PercentageAction=2.0`），
  upowerd 发起 `suspend-then-hibernate`（日志：`suspend-then-hibernate requested from client
  PID 1050 ('upowerd')`），机器在 2% 电量下睡死后到 20:44 才冷启动。整份日志里**从未有过真正的休眠**
  （`PM: hibernation entry/exit` 各 0 次），但 `resume=/dev/mapper/root resume_offset=1884480`
  已经指向 btrfs swapfile。这条临界电量路径没验证过，可用 `--hibernate` 测；若休眠恢复不成立，
  就不该依赖它，应改 upower 的 `CriticalPowerAction`。
- **未验证**：s2idle 的实际效果、睡眠耗电（s2idle 不断电，理论上比 deep 费电，用 `--drain` 量）、
  以及 s2idle 下掀盖唤醒是否仍然有效（`/proc/acpi/wakeup` 里 `LID0 S4 *enabled`，
  且实测 deep 下掀盖能唤醒机器，推测 s2idle 下也可以）。
- 之前遗留的 `/etc/systemd/system/omarchy-nvme-suspend-fix.service`（给 NVMe 设
  `d3cold_allowed=0`）与本病无关，NVMe 每次恢复都正常（`nvme nvme0: 4/0/0 default/read/poll queues`），
  未改动它。
- `/usr/lib/systemd/system-sleep/` 下的 `unmount-fuse`、`keyboard-backlight` 两个钩子与本病无关。
