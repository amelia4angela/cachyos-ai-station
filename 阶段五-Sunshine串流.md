> [[Home]] | 部署指南：[[阶段零-前置授权|0]] → [[阶段一-环境检测|1]] → [[阶段二-Hermes多代理系统|2]] → [[阶段三-网络代理|3]] → [[阶段四-输入法|4]] → [[阶段五-Sunshine串流|5]] | 参考：[[环境要求]] [[游戏环境]] [[踩坑记录]]

---

# 阶段五：Sunshine 串流

## 架构

```
Moonlight 客户端（手机/平板/另一台电脑）
  → Sunshine 触发 global_prep_cmd.do
    → kscreen-doctor-env（注入 Wayland 环境变量）
      → kscreen-doctor output.eDP-1.disable
        → KWin 关闭内屏，只剩 HDMI-A-3

Moonlight 断开
  → Sunshine 触发 global_prep_cmd.undo
    → kscreen-doctor output.eDP-1.enable
      → 恢复内屏
```

## 显示器信息

| 输出名 | 类型 | 说明 |
|---|---|---|
| eDP-1 | 内屏 | 笔记本屏幕（第一屏幕）|
| HDMI-A-3 | 外接屏 | HDMI 外接显示器（第二屏幕）|

## kscreen-doctor-env Wrapper

Sunshine 从 systemd 启动，没有 Wayland 环境变量。需要 wrapper 从 plasmashell 进程注入：

```bash
#!/bin/bash
# ~/.local/bin/kscreen-doctor-env
PLASMA_PID=$(pgrep -u chao plasmashell | head -1)
if [[ -n "$PLASMA_PID" ]]; then
    while IFS='=' read -r key value; do
        case "$key" in
            DISPLAY|WAYLAND_DISPLAY|DBUS_SESSION_BUS_ADDRESS|XDG_RUNTIME_DIR|QT_QPA_PLATFORM)
                export "$key=$value" ;;
        esac
    done < /proc/$PLASMA_PID/environ
fi
exec /usr/sbin/kscreen-doctor "$@"
```

## Sunshine 配置

**`~/.config/sunshine/sunshine.conf`**：

```ini
address_family = both
locale = zh
upnp = enabled
global_prep_cmd = [{"do":"/home/chao/.local/bin/kscreen-doctor-env output.eDP-1.disable","undo":"/home/chao/.local/bin/kscreen-doctor-env output.eDP-1.enable"}]
```

## 防火墙规则

```bash
# TCP 控制/视频/音频
sudo ufw allow 47984:47990/tcp comment "Sunshine TCP"
sudo ufw allow 48010/tcp comment "Sunshine TCP"

# UDP 实时传输
sudo ufw allow 47998:48002/udp comment "Sunshine UDP"
```

## DPMS 与串流

| 关屏方式 | 能串流？ | 原因 |
|---|---|---|
| KDE 节能关屏（DPMS）| ✅ | GPU 仍在渲染 |
| kscreen-doctor disable | ❌ | 输出被禁用 |
| 物理关显示器电源 | ✅ | GPU 照常工作 |

**关键**：只要外接屏没被 disable，关屏也能串流。

## 重启 Sunshine

```bash
systemctl --user restart app-dev.lizardbyte.app.Sunshine.service
```
