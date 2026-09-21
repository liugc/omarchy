---
title: MacBook 合盖唤醒后黑屏约 85 秒、电源键无响应（雷雳控制器恢复失败）
slug: macbook-suspend-thunderbolt-resume-hang
category: fix
tags: [macbook, suspend, resume, thunderbolt, xhci, systemd, sleep-hook, mem_sleep]
created: 2026-09-20
updated: 2026-09-21
---

# MacBook 合盖唤醒后黑屏约 85 秒、电源键无响应（雷雳控制器恢复失败）

机器：MacBookPro14,1（Apple，无 T2），Alpine Ridge Thunderbolt 3，拓扑
`00:1c.4`(PCH 根端口) → `04:00.0` → `05:00.0`/`05:01.0`/`05:02.0`/`05:04.0` → `06:00.0`(NHI)
与 `07:00.0`(xHCI，就是两个 USB-C 口)，内核 `7.2.5-3-omarchy`，systemd 261.2-1。

## 场景 / 问题

合上盖子再掀开，屏幕不亮；按电源键没反应，要按好几次才「进去」。
偶发（约 3/13 次）彻底不回来，必须长按电源键断电重开机。

`deep`(S3) 时期每次恢复都出现同一组 9 行：

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

**`thunderbolt` 驱动自己的睡眠/恢复路径把交换机留在坏状态**，与电源（D3cold）无关：

- 恢复时 `tb_domain_resume_noirq` → `tb_switch_reset` / `tb_switch_configure` /
  `tb_switch_discover_tunnels` 全部 `tb_cfg_read/write: -108`，即根本联系不上交换机；
- 于是 `06:00.0` NHI 恢复不了，其后的 `07:00.0` xHCI 永远 ready 不了，内核在 xhci 的 resume
  回调里按 1+2+4+8+16+32+65 秒翻倍重试才放弃，最后由 pciehp 把 `07:00.0` 摘除
  （USB-C 口死到重启）；
- **这段等待期间整个系统冻结**（userspace 还没 thaw），所以黑屏、键盘死、电源键无效；
  黑屏时长 = 这段内核耗时。

实测 2026-09-20 22:38:50 合盖 → 22:40:30 恢复：

- 进入→退出之间内核单调时钟差（单调时钟在 S3 期间停走，因此即冻结时长）＝ **85.48 s**
- 同一段墙面时钟间隔 100.7 s ⇒ 实际睡着只有约 15 s，也就是**掀盖确实唤醒了机器**，
  剩下 85 秒全是冻结
- 全部 13 次挂起中的 10 次恢复、跨 6 个启动周期（09-03 起）都是上面那组日志
- 另外 3 次挂起完全没有回来（没有 `PM: suspend exit`，下次是冷启动）：
  `09-05 23:01:53`、`09-19 00:42:11`、`09-20 06:18:10`

> 这篇笔记 2026-09-20 版把原因写成「s2idle 会绕开这段等待，是唯一可行修法」——**已被实测推翻**：
> 纯 s2idle 下同样跑完 65 秒等待（实测冻结 79.34 s）。`xhci_pci` 没有 `*_noirq` 变体
> **并不**意味着该等待在 suspend-to-idle 下不执行。

## 解决

### 1. 最终修法：sleep hook 把 thunderbolt 驱动移出睡眠路径

```bash
# /usr/lib/systemd/system-sleep/60-macbook-tb-unbind   （chmod +x）
#!/bin/bash
NHI=0000:06:00.0
LOGF=/var/log/macbook-suspend-diag/hook.log
log() { printf '%s %s\n' "$(date -Is)" "$*" >> "$LOGF" 2>/dev/null || true; }

case $1 in
  pre)
    if [[ -e /sys/bus/pci/devices/$NHI/driver ]]; then
      log "pre: unbinding $NHI"
      echo "$NHI" > /sys/bus/pci/drivers/thunderbolt/unbind && log "pre: unbound ok" || log "pre: unbind FAILED"
    else
      log "pre: $NHI already unbound"
    fi
    ;;
  post)
    if [[ -e /sys/bus/pci/devices/$NHI && ! -e /sys/bus/pci/devices/$NHI/driver ]]; then
      log "post: rebinding $NHI"
      echo "$NHI" > /sys/bus/pci/drivers/thunderbolt/bind && log "post: rebind ok" || log "post: rebind FAILED"
    else
      log "post: nothing to do"
    fi
    ;;
esac
exit 0
```

不加 `MemorySleepMode=s2idle`、不改 `d3cold_allowed`、不动 cmdline：睡眠模式继续用默认的 `deep`，
hook 只在睡眠前后各改一次驱动绑定。睡眠安全网仍然有效（`pm_wakeup_pending()`），但驱动不再去碰交换机。

### 2. 只读定位命令

```bash
journalctl --no-pager -o short-iso -g 'PM: suspend (entry|exit)'
journalctl --no-pager -o short-iso -g 'not ready [0-9]+ms after resume|giving up'
journalctl --no-pager -o short-iso -g 'thunderbolt|pciehp|tb_cfg|D3cold to D0|D3hot to D0'
cat /sys/power/mem_sleep          # 方括号是当前选择
cat /proc/acpi/wakeup             # LID0 S4 *enabled / SPIT S3 *enabled
lspci -s 07:00.0                  # 恢复后被摘掉就是"USB-C 口失效"
ls -d /sys/bus/usb/devices/usb3 /sys/bus/usb/devices/usb4   # 07:00.0 的两个 root hub
```

**睡眠/恢复时长必须用 userspace 时钟量**（printk 时间戳分不开睡眠与恢复）：

```
freeze = ΔCLOCK_MONOTONIC          # 唤醒后无响应的死窗口（用户感知的黑屏时长）
slept  = ΔCLOCK_BOOTTIME − ΔMONOTONIC   # 真正睡着的时间
```

```bash
python3 -c 'import time; print(time.monotonic(), time.clock_gettime(time.CLOCK_BOOTTIME))'
# …睡眠…
python3 -c 'import time; print(time.monotonic(), time.clock_gettime(time.CLOCK_BOOTTIME))'
```

### 3. 走不通的路（有实测证据，别再试）

| 配置 | 结果 |
|---|---|
| 纯 `s2idle` | 冻结 79.34 s，65 秒等待照跑，xHCI 仍被摘 |
| `s2idle` + `d3cold_allowed=0` 只覆盖 `04:00.0..07:00.0` | 仍丢 —— 根端口 `00:1c.4` 的 D3cold 会把整棵子树断电（`Unable to change power state from D3cold to D0`） |
| `s2idle` + 覆盖整条路径（含 `00:1c.4`） | 冻结降到 5.5 s、65 秒等待消失，但桥报 `D3hot→D0 ... device inaccessible`，芯片内部隧道照样丢、xHCI 照样被摘 ⇒ 断电不是病根 |
| `deep` + `pm_async=off` | `resume_noirq` 阶段本来就同步，救不回来（omarchy 的 `pm_async=off mem_sleep_default=deep` 只对 T2 机型生效，见 `/usr/share/omarchy/install/hardware/apple/fix-t2.sh`） |

### 4. 诊断/落地脚本

```bash
sudo bash ~/.local/share/macbook-suspend-diag/diag.sh            # 默认测 H_deep_tb_unbind
sudo bash ~/.local/share/macbook-suspend-diag/diag.sh --all       # 再测 G/B/C/A/D/E
sudo bash ~/.local/share/macbook-suspend-diag/diag.sh --undo      # 清掉 hook / drop-in / unit
```

候选：`H_deep_tb_unbind`（deep + hook，默认）、`G_s2idle_tb_unbind`、`B_s2idle_no_d3cold`、
`C_s2idle_no_d3cold_nort`、`A_s2idle`、`D_deep_no_d3cold`、`E_deep_sync`。
脚本会：脏基线门禁 → 装 hook 并自检 → 用 hook 环绕的 rtcwake 循环量 `freeze`/`slept`/`usb_c_controller`
→ 通过则落盘 → 确认循环 → 真实合盖测试。日志在 `/var/log/macbook-suspend-diag/`。

## 验证

2026-09-21 23:30:39 真实合盖（logind → systemd-suspend，**deep**）实测：

```
23:30:39  PM: suspend entry (deep)
23:31:00  ACPI: PM: Waking up from system sleep state S3
23:31:00  xhci_hcd 0000:07:00.0: xHC error in resume, USBSTS 0x401, Reinit
23:31:00  usb usb3/usb4: root hub lost power or was reset
23:31:00  PM: suspend exit                     ← 恢复路径 ~6.9 s，原来 85 s
```

- 无 `not ready` 循环、无 `tb_cfg_*: -108`、没有 `Card not present`
- `07:00.0` 仍在且绑定 `xhci_hcd`；`usb3`/`usb4` root hub 存在；NHI `06:00.0` 配置空间可读；
  `/sys/bus/thunderbolt/devices/` 有 `0-0`、`domain0`
- **该 boot 内** `tb_cfg_.*: -108` / `not ready .* after resume` / `Card not present` /
  `D3cold to D0` / `D3hot to D0` 全部 **0 条**
- `hook.log` 显示该次睡眠 `pre`/`post` 都执行成功（unbind ok / rebind ok）
- 未验证：插一个 USB-C 设备实测端口数据功能（只确认了控制器与 root hub 活着）

## 备注

- **测量方法论的坑**（这次踩了三个，下次直接绕开）：
  1. printk 时间戳分不开睡眠与恢复 ⇒ 用 `CLOCK_BOOTTIME` − `CLOCK_MONOTONIC` 差值。
  2. **`rtcwake` 直接写 `/sys/power/state`，绕过 systemd-sleep，所以 sleep hook 不会执行** ——
     我第一轮 G 测试就是因此误判「hook 无效」。用 rtcwake 测 hook 类配置必须自己跑
     `$HOOK pre/post`，或者用 logind/`systemctl suspend`（合盖）。
  3. systemd 261 的 sleep hook **只扫 `/usr/lib/systemd/system-sleep/`**：
     `strings /usr/lib/systemd/systemd-sleep | grep system-sleep` 只输出这一个路径，
     `man systemd-sleep` 也只写这个目录。放在 `/etc/systemd/system-sleep/` 是**死文件**
     （我因此白跑了两轮）。
  4. 一次失败的恢复会把本 boot 的雷雳 fabric 弄坏（xHCI 被摘、之后每次挂起都超时）⇒
     每次测量前重启；脚本有脏基线门禁（检测 `07:00.0` 缺失或本 boot 已有 `tb_cfg_-108`）。
- **电源键为什么"没反应"**：`/etc/systemd/logind.conf.d/10-ignore-power-button.conf` 设了
  `HandlePowerKey=ignore`（omarchy 默认，防误触）。它不是病因（冻结期间 logind 本身也冻住），
  但短按永远没用，强制断电只能长按约 10 s。
- **06:18 那次是另一条路**：电池到 2%（`/etc/UPower/UPower.conf` 的 `PercentageAction=2.0`），
  upowerd 发起 `suspend-then-hibernate`（日志：`suspend-then-hibernate requested from client
  PID 1050 ('upowerd')`），机器在 2% 电量下睡死后到 20:44 才冷启动。整份日志里**从未有过真正的休眠**
  （`PM: hibernation entry/exit` 各 0 次），但 `resume=/dev/mapper/root resume_offset=1884480`
  已指向 btrfs swapfile（`btrfs inspect-internal map-swapfile -r` 与之相符）。这条临界电量路径
  未验证，可用 `--hibernate` 测；若休眠恢复不成立，就不该依赖它，应改 `CriticalPowerAction`。
- 现有落盘状态：只有那个 hook；`mem_sleep=deep`（默认）、无 drop-in/unit（s2idle 与 d3cold 方案
  最终都不需要）。`--undo` 会一并清理 `/etc/systemd/system-sleep/` 的旧残留路径。
- 之前遗留的 `/etc/systemd/system/omarchy-nvme-suspend-fix.service`（给 NVMe 设
  `d3cold_allowed=0`）与本病无关，NVMe 每次恢复都正常（`nvme nvme0: 4/0/0 default/read/poll queues`），
  未改动它。`/usr/lib/systemd/system-sleep/` 里另有 omarchy 的 `unmount-fuse`、`keyboard-backlight`。
