# Sunshine / 串流

## 架构

```
Moonlight 客户端
  → Sunshine 触发 global_prep_cmd.do
    → kscreen-doctor-env（注入环境变量）
      → kscreen-doctor output.eDP-1.disable
        → KWin 关闭内屏，只剩 HDMI-A-3

Moonlight 断开
  → Sunshine 触发 global_prep_cmd.undo
    → 恢复内屏
```

## 核心配置

### sunshine.conf
```ini
address_family = both
capture = kwin
locale = zh
upnp = enabled
global_prep_cmd = [{"do":"/home/chao/.local/bin/kscreen-doctor-env output.eDP-1.disable","undo":"/home/chao/.local/bin/kscreen-doctor-env output.eDP-1.enable"}]
```

### kscreen-doctor-env (环境注入 wrapper)
```bash
#!/bin/bash
PLASMA_PID=$(pgrep -u chao plasmashell | head -1)
while IFS='=' read -r key value; do
    case "$key" in
        DISPLAY|WAYLAND_DISPLAY|DBUS_SESSION_BUS_ADDRESS|XDG_RUNTIME_DIR|QT_QPA_PLATFORM)
            export "$key=$value" ;;
    esac
done < /proc/$PLASMA_PID/environ
exec /usr/sbin/kscreen-doctor "$@"
```

### systemd drop-in
```ini
# ~/.config/systemd/user/app-dev.lizardbyte.app.Sunshine.service.d/10-sunshine-env.conf
[Service]
ExecStart=
ExecStart=/usr/bin/sunshine
Environment=WAYLAND_DISPLAY=wayland-0
Environment=LIBVA_DRIVER_NAME=nvidia
```

## 踩坑记录

### kscreen-doctor 崩溃 (SIGABRT)
**问题**: 从 systemd 服务调用时崩溃 (exit 134)
**原因**: 缺少 WAYLAND_DISPLAY、DBUS_SESSION_BUS_ADDRESS 等变量
**排查**: `coredumpctl info <pid>` 看到崩溃在 `QGuiApplicationPrivate::createEventDispatcher`
**解决**: wrapper 脚本从 plasmashell 进程注入环境变量

### capture = auto 找不到编码器
**问题**: 混合 GPU (Intel + NVIDIA) 系统上 auto 方式找不到编码器
**解决**: 用 `capture = kwin`

### output_name 启动失败
**问题**: `output_name = Virtual-sunshine-vm` 在启动时找不到虚拟显示器
**解决**: 去掉 output_name，让 Sunshine 自动选择

### global_prep_cmd 格式
**注意**: 是 JSON 数组，不是简单命令字符串
- `do` = 客户端连接时执行
- `undo` = 客户端断开时执行

### kscreen-doctor -o 崩溃
**问题**: 列出所有输出时崩溃，但 disable/enable 正常
**影响**: 不影响使用，只是不能用 -o 参数
