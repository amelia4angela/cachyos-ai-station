> [Home](README.md) | 部署指南：[0](阶段零-前置授权.md) → [1](阶段一-环境检测.md) → [2](阶段二-Hermes多代理系统.md) → [3](阶段三-网络代理.md) → [4](阶段四-输入法.md) → [5](阶段五-Sunshine串流.md) | 参考：[环境要求](环境要求.md) [游戏环境](游戏环境.md) [踩坑记录](踩坑记录.md)

---

# 阶段五：Sunshine 串流

## 最终方案（v7 实测确认）

| 组件 | 配置 |
|---|---|
| 捕获方式 | `capture = kms`（DRM framebuffer 直接捕获） |
| 编码器 | `encoder = nvenc`（NVIDIA 硬件编码） |
| 显示器切换 | `global_prep_cmd` + `sunshine-display-switch.sh` |
| 关屏串流 | ✅ KMS 捕获不受 DPMS 影响 |

## Sunshine 配置

**`~/.config/sunshine/sunshine.conf`**：

```ini
address_family = both
locale = zh
upnp = enabled

# KMS 捕获（直接从 DRM framebuffer，关屏也能串流）
capture = kms

# NVENC 硬件编码
encoder = nvenc

# 串流时：关闭内屏，只用外接屏
global_prep_cmd = [{"do":"/home/chao/.local/bin/sunshine-display-switch.sh do","undo":"/home/chao/.local/bin/sunshine-display-switch.sh undo"}]
```

## 显示器切换脚本

**`~/.local/bin/sunshine-display-switch.sh`**：

```bash
#!/bin/bash
export DISPLAY=:0
export WAYLAND_DISPLAY=wayland-0
export XDG_RUNTIME_DIR=/run/user/1000
export QT_QPA_PLATFORM=wayland

case "$1" in
    do)
        /usr/sbin/kscreen-doctor output.eDP-1.disable 2>/dev/null
        ;;
    undo)
        /usr/sbin/kscreen-doctor output.eDP-1.enable 2>/dev/null
        ;;
    status)
        /usr/sbin/kscreen-doctor -o 2>&1 | grep -E "Output:|enabled|disabled"
        ;;
esac
```

## 显示器信息

| 输出名 | 类型 | 说明 |
|---|---|---|
| eDP-1 | 内屏 | 笔记本屏幕 |
| HDMI-A-3 | 外接屏 | HDMI 外接显示器 |

## 为什么选 KMS

| 捕获方式 | 结果 |
|---|---|
| `capture = kwin` | ❌ YUV 4:4:4 与 NVENC 不兼容，黑屏 |
| `capture = gpu` | ❌ 找不到显示输出 |
| **`capture = kms`** | **✅ 正常工作** |

KMS 直接从 GPU framebuffer 捕获，不经过 PipeWire/KWin，格式兼容性最好。

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
# AUR 版本（推荐，最新）
paru -S sunshine
```
