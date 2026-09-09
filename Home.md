# 🏠 CachyOS Hermes 部署指南

> 本机 AI 总指挥 + 多代理协作 + 游戏串流，一站式 CachyOS 部署手册。

## 系统概览

| 项目 | 规格 |
|---|---|
| 系统 | CachyOS Linux (Arch-based, 滚动更新) |
| 桌面 | KDE Plasma 6.7 · Wayland |
| 内核 | linux-cachyos 7.2.3 |
| GPU | NVIDIA RTX 4060 Max-Q + Intel UHD |
| AI | Hermes Agent v0.21.1 · 4 Agent 协作 |

## 📖 部署步骤

| 步骤 | 内容 | 状态 |
|---|---|---|
| [[阶段零-前置授权]] | 安装 paru、配置 sudo 免密、开启 linger | ✅ |
| [[阶段一-环境检测]] | 检测系统/Hermes/网络/输入法状态 | ✅ |
| [[阶段二-Hermes多代理系统]] | SkillHub 安装、Agent 配置、soul.md | ✅ |
| [[阶段三-网络代理]] | Clash Verge v4 代理方案 | ✅ |
| [[阶段四-输入法]] | Fcitx5 + 雾凇拼音 + 29 个主题 | ✅ |
| [[阶段五-Sunshine串流]] | Moonlight 串流 + 显示器自动切换 | ✅ |

## 🎮 环境配置

| 内容 | 页面 |
|---|---|
| [[环境要求]] | 系统需求与推荐配置 |
| [[游戏环境]] | Steam/Wine/Proton/MangoHud/Gamemode |
| [[踩坑记录]] | 部署问题与解决方案 |

## 快速开始

```bash
# 1. 安装 AUR 助手
sudo pacman -S --needed base-devel git
git clone https://aur.archlinux.org/paru.git /tmp/paru
(cd /tmp/paru && makepkg -si --noconfirm)

# 2. 配置免密 sudo
echo 'chao ALL=(ALL) NOPASSWD: ALL' | sudo tee /etc/sudoers.d/99-chao-nopasswd
sudo chmod 440 /etc/sudoers.d/99-chao-nopasswd

# 3. 然后按阶段一 → 阶段五 顺序执行
```
