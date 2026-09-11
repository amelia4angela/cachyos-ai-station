> [Home](README.md) | 部署指南：[0](阶段零-前置授权.md) → [1](阶段一-环境检测.md) → [2](阶段二-Hermes多代理系统.md) → [3](阶段三-网络代理.md) → [4](阶段四-输入法.md) → [5](阶段五-Sunshine串流.md) | 参考：[环境要求](环境要求.md) [游戏环境](游戏环境.md) [踩坑记录](踩坑记录.md)

---

# 阶段五：Sunshine 串流

## 最终方案（v9 实测确认）

| 组件 | 配置 |
|---|---|
| 捕获方式 | `capture = kms`（DRM framebuffer 直接捕获） |
| 编码器 | `encoder = nvenc`（NVIDIA 硬件编码） |
| 目标输出 | `output_name = HDMI-A-3`（欺骗器） |
| 显示器切换 | `sunshine-display-switch.sh` |
| 欺骗器 | HDMI Dummy Plug，内核强制 3840x2160@60 |

## Sunshine 配置

**`~/.config/sunshine/sunshine.conf`**：

```ini
address_family = both
locale = zh
upnp = enabled

# KMS 捕获，固定抓 HDMI-A-3（欺骗器）
capture = kms
output_name = HDMI-A-3

# NVENC 硬件编码
encoder = nvenc

# 串流时：开欺骗器+primary，关主屏
global_prep_cmd = [{"do":"/home/chao/.local/bin/sunshine-display-switch.sh do","undo":"/home/chao/.local/bin/sunshine-display-switch.sh undo"}]
```

## 显示器切换脚本

**`~/.local/bin/sunshine-display-switch.sh`**：

```bash
#!/bin/bash
# Sunshine 显示器切换脚本（v9）
#
# 串流时（do）：开欺骗器+primary → 关主屏 → 杀锁屏（KDE 在欺骗器上重建）
# 退出串流（undo）：开主屏+primary → 关欺骗器 → 杀锁屏（KDE 在主屏上重建）
#
# 不禁用锁屏服务——让 KDE 自己管理

export DISPLAY=:0
export WAYLAND_DISPLAY=wayland-0
export XDG_RUNTIME_DIR=/run/user/1000
export QT_QPA_PLATFORM=wayland

case "$1" in
    do)
        # 1. 确保欺骗器开启
        /usr/sbin/kscreen-doctor output.HDMI-A-3.enable 2>/dev/null
        # 2. 欺骗器设为主显示器
        /usr/sbin/kscreen-doctor output.HDMI-A-3.primary 2>/dev/null
        # 3. 关闭主屏
        /usr/sbin/kscreen-doctor output.eDP-1.disable 2>/dev/null
        # 4. 杀锁屏（KDE 会在欺骗器上重建）
        kill $(pgrep -f kscreenlocker_greet) 2>/dev/null
        kill $(pgrep -f kscreenlocker) 2>/dev/null
        ;;
    undo)
        # 1. 开启主屏
        /usr/sbin/kscreen-doctor output.eDP-1.enable 2>/dev/null
        # 2. 主屏设为 primary
        /usr/sbin/kscreen-doctor output.eDP-1.primary 2>/dev/null
        # 3. 关闭欺骗器
        /usr/sbin/kscreen-doctor output.HDMI-A-3.disable 2>/dev/null
        # 4. 杀锁屏（KDE 会在主屏上重建）
        kill $(pgrep -f kscreenlocker_greet) 2>/dev/null
        kill $(pgrep -f kscreenlocker) 2>/dev/null
        ;;
    status)
        /usr/sbin/kscreen-doctor -o 2>&1 | grep -E "Output:|enabled|disabled|primary"
        ;;
esac
```

## 显示器信息

| 输出名 | 类型 | 说明 |
|---|---|---|
| eDP-1 | 主屏 | 笔记本屏幕，串流时关闭 |
| HDMI-A-3 | 欺骗器 | HDMI Dummy Plug，Sunshine 抓这个 |

## 核心原则

- **主屏永远不 disable 时让锁屏有输出**：先切显示器，再杀锁屏，KDE 自动在正确的显示器上重建
- **不禁用锁屏服务**：让 KDE 自己管理锁屏生命周期
- **欺骗器永远在线**：内核参数 `video=HDMI-A-3:3840x2160@60` 强制分辨率

## 捕获方式选择

| 方案 | 结果 |
|---|---|
| `capture = kwin` | ❌ YUV 4:4:4 与 NVENC 不兼容 |
| `capture = gpu` | ❌ 找不到显示输出 |
| **`capture = kms`** | **✅ 正常工作** |

## 欺骗器内核参数

在 limine.conf 中添加：

```
video=HDMI-A-3:3840x2160@60
```

修改命令：
```bash
sudo sed -i '/cmdline:.*rootflags=subvol=\/@ root=UUID/s|$| video=HDMI-A-3:3840x2160@60|' /boot/limine.conf
sudo reboot
```

## 防火墙规则

```bash
sudo ufw allow 47984:47990/tcp comment "Sunshine TCP"
sudo ufw allow 48010/tcp comment "Sunshine TCP"
sudo ufw allow 47998:48002/udp comment "Sunshine UDP"
```

## 自启动

```bash
systemctl --user is-enabled app-dev.lizardbyte.app.Sunshine.service
# 输出：enabled
```

## 安装

```bash
paru -S sunshine
```

## 验证

```bash
# 检查欺骗器状态
cat /sys/class/drm/card1-HDMI-A-3/status
# 输出：connected

# 检查 Sunshine 端口
ss -tlnp | grep -E "47984|47989|47990|48010"

# 查看切换日志
~/.local/bin/sunshine-display-switch.sh log
```
