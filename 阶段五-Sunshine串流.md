> [Home](README.md) | 部署指南：[0](阶段零-前置授权.md) → [1](阶段一-环境检测.md) → [2](阶段二-Hermes多代理系统.md) → [3](阶段三-网络代理.md) → [4](阶段四-输入法.md) → [5](阶段五-Sunshine串流.md) | 参考：[环境要求](环境要求.md) [游戏环境](游戏环境.md) [踩坑记录](踩坑记录.md)

---

# 阶段五：Sunshine 串流

## 最终方案（v7 实测确认）

| 组件 | 配置 |
|---|---|
| 捕获方式 | `capture = kms`（DRM framebuffer 直接捕获） |
| 编码器 | `encoder = nvenc`（NVIDIA 硬件编码） |
| 显示器切换 | `global_prep_cmd` + `sunshine-display-switch.sh` |
| 欺骗器 | HDMI-A-3，内核强制 3840x2160@60 |
| 关屏串流 | ✅ 欺骗器永远在线，不受 DPMS 影响 |

## Sunshine 配置

**`~/.config/sunshine/sunshine.conf`**：

```ini
address_family = both
locale = zh
upnp = enabled

# KMS 捕获（直接从 DRM framebuffer）
capture = kms

# NVENC 硬件编码
encoder = nvenc

# 串流时：关闭内屏，只用外接屏
# 退出串流：恢复内屏
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

## 欺骗器配置（关键）

### 硬件

HDMI-A-3 接了一个**欺骗器**（HDMI Dummy Plug），欺骗器永远报告 `connected`，不会被 DPMS 关掉。

### 内核强制分辨率

在 limine.conf 中添加内核参数，开机强制 dummy 显示器以最高分辨率运行：

```
video=HDMI-A-3:3840x2160@60
```

修改方法：
```bash
sudo sed -i '/cmdline:.*rootflags=subvol=\/@ root=UUID/s|$| video=HDMI-A-3:3840x2160@60|' /boot/limine.conf
sudo reboot
```

### 为什么需要欺骗器

| 没有欺骗器 | 有欺骗器 |
|---|---|
| Sunshine 找不到显示输出 | 欺骗器永远在线 |
| 关屏后 KMS 断连 | 关屏不影响欺骗器 |
| 需要虚拟显示模块 | 即插即用，零配置 |

## 显示器信息

| 输出名 | 类型 | 说明 |
|---|---|---|
| eDP-1 | 内屏 | 笔记本屏幕，串流时关闭 |
| HDMI-A-3 | 欺骗器 | HDMI Dummy Plug，永远在线，Sunshine 抓这个 |

## 捕获方式选择

| 方案 | 结果 |
|---|---|
| `capture = kwin` | ❌ YUV 4:4:4 与 NVENC 不兼容，黑屏 |
| `capture = gpu` | ❌ 找不到显示输出 |
| **`capture = kms`** | **✅ 正常工作** |

KMS 直接从 DRM framebuffer 捕获，不经过 PipeWire/KWin，格式兼容性最好。

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

## 验证

```bash
# 检查欺骗器状态
cat /sys/class/drm/card1-HDMI-A-3/status
# 输出：connected

# 检查 Sunshine 端口
ss -tlnp | grep -E "47984|47989|47990|48010"
```
