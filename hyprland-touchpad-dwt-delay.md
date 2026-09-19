---
title: Hyprland 延长触摸板防误触（disable-while-typing）恢复延迟
slug: hyprland-touchpad-dwt-delay
category: tip
tags: [hyprland, touchpad, libinput, lua, macbook, applespi]
created: 2026-09-19
updated: 2026-09-19
---

# Hyprland 延长触摸板防误触（disable-while-typing）恢复延迟

## 场景 / 问题

打字时触摸板会误触导致鼠标移动。`input.touchpad.disable_while_typing = true` 只在按键期间生效，
停止按键约 500ms 后触摸板就恢复，打字间隙仍会碰到触摸板。

## 原因

libinput 的 DWT（disable-while-typing）超时被写死为 500ms。libinput 1.31 新增了
`libinput_device_config_dwt_set_timeout()` API，但 Hyprland 0.56.2 只暴露了
`disable_while_typing` 开关，没有超时配置项，所以纯配置项调不了这个延迟。

Hyprland 0.56 的 Lua 配置提供了 `hl.on("input.keyboard.key")` 事件、`hl.timer` 和 `hl.device`，
可以在配置里自己实现：按键即禁用触摸板，停止打字 N 毫秒后再启用。

## 解决

在 `~/.config/hypr/input.lua` 末尾追加（在 `hl.config({...})` 之后）：

```lua
local dwt_delay_ms = 1500
local touchpad_name = "apple-spi-touchpad"
local touchpad_disabled = false
local dwt_timer

dwt_timer = hl.timer(function()
  if touchpad_disabled then
    hl.device({ name = touchpad_name, enabled = true })
    touchpad_disabled = false
  end

  dwt_timer:set_enabled(false)
end, { timeout = dwt_delay_ms, type = "repeat" })

hl.on("input.keyboard.key", function(_, _, state)
  if state ~= 1 then -- 1 = 按下
    return
  end

  if not touchpad_disabled then
    hl.device({ name = touchpad_name, enabled = false })
    touchpad_disabled = true
  end

  dwt_timer:set_enabled(true)
end)
```

- `dwt_delay_ms` 就是停止打字后恢复触摸板的延迟，按需调整（默认 1500ms）。
- `touchpad_name` 用 `hyprctl devices` 里的名字（这台 MacBook 是 `apple-spi-touchpad`）。
- `hl.timer` 用 `repeat` 类型创建但初始未启用，每次按键通过 `set_enabled(true)` 重置倒计时；
  触发一次后自行 `set_enabled(false)`。
- 事件参数为 `(keycode, timeMs, state)`，state 1 = 按下、0 = 抬起，按键抬起不重置计时器。

## 验证

```bash
hyprctl reload
hyprctl configerrors   # 无输出即无错误
```

用 `wtype` 模拟按键，并临时挂一个把状态写到 `/tmp` 的监听器观察：

```bash
hyprctl eval '
local function log(s) local f=io.open("/tmp/opencode/dwtlog","a") if f then f:write(s.."\n") f:close() end end
local disabled=false local t
t = hl.timer(function()
  log("timer fired, disabled="..tostring(disabled))
  if disabled then hl.device({name="apple-spi-touchpad",enabled=true}) disabled=false log("re-enabled") end
  t:set_enabled(false)
end, {timeout=1500, type="repeat"})
hl.on("input.keyboard.key", function(_,_,s)
  if s==1 then log("key press")
    if not disabled then hl.device({name="apple-spi-touchpad",enabled=false}) disabled=true log("disabled") end
    t:set_enabled(true) log("timer restart")
  end
end)'

wtype h; sleep 0.4; wtype i; sleep 2.2; cat /tmp/opencode/dwtlog
```

输出确认：每次按键打印 `key press` / `timer restart`，最后一次按键 1.5s 后打印
`timer fired, disabled=true` / `re-enabled`，说明计时器被按键重置、延迟后恢复触摸板。

## 备注

- `input.lua` 里那条「DWT 需要 `/etc/libinput/local-overrides.quirks`，因为 applespi 键盘不上报
  vendor/product ID」的注释已过时。实测内核已设置 `LIBINPUT_DEVICE_GROUP` 和
  `ID_INPUT_TOUCHPAD_INTEGRATION=internal`，键盘/触摸板 DWT 配对本来就正常。
- 本机设备名（`udevadm info -q property`）：
  - 键盘 `/dev/input/event4`：`ID_BUS=spi`，`LIBINPUT_DEVICE_GROUP=1c/0/0:applespi`
  - 触摸板 `/dev/input/event5`：`ID_INPUT_TOUCHPAD_INTEGRATION=internal`，同 group
- Hyprland 事件定义（v0.56.2）见源码 `src/config/lua/LuaEventHandler.cpp` 的
  `input.keyboard.key` 监听；`hl.device` 支持 `enabled` 字段可运行时切换设备。
- 未验证：换用 libinput 1.31 的 DWT 超时 API 需要 Hyprland 或自定义工具直接调用 C API，
  当前未做，仍以上述 Lua 方案为准。
