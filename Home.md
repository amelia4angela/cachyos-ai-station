# 🖥️ CachyOS AI 工作站

> CachyOS + KDE Plasma 6 + Hermes 多代理 AI + 游戏串流，一台机器搞定工作、AI、游戏。

---

## 📋 部署清单

```
 ✅  阶段零    前置授权 ··················  paru / sudo 免密 / linger
 ↓
 ✅  阶段一    环境检测 ··················  系统 / Hermes / 网络 / 输入法
 ↓
 ✅  阶段二    Hermes 多代理系统 ·········  SkillHub / 4 Agent / soul.md
 ↓
 ✅  阶段三    网络代理 ··················  Clash Verge v4 / SOCKS5
 ↓
 ✅  阶段四    输入法 ····················  Fcitx5 / 雾凇拼音 / 29 主题
 ↓
 ✅  阶段五    游戏串流 ··················  Sunshine / Moonlight / 显示器切换
```

---

## 🎯 功能模块

| 模块 | 说明 | 链接 |
|:---:|:---|:---:|
| 🤖 | **Hermes 多代理 AI** — 4 个 Agent 协作（小涵/菲菲/薇薇/苏苏） | [[阶段二-Hermes多代理系统]] |
| 🌐 | **网络代理** — Clash Verge + SOCKS5 防 DNS 污染 | [[阶段三-网络代理]] |
| ⌨️ | **中文输入** — Fcitx5 + 雾凇拼音 + 29 个主题 | [[阶段四-输入法]] |
| 🎮 | **游戏串流** — Sunshine + Moonlight，自动切屏 | [[阶段五-Sunshine串流]] |
| 🖥️ | **游戏环境** — Steam / Wine / Proton / MangoHud | [[游戏环境]] |

---

## 🛠️ 部署指南

| 步骤 | 做什么 | 链接 |
|:---:|:---|:---:|
| 0 | 装 paru、配 sudo、开 linger | [[阶段零-前置授权]] |
| 1 | 检测系统状态 | [[阶段一-环境检测]] |
| 2 | 部署 Hermes + Agent | [[阶段二-Hermes多代理系统]] |
| 3 | 配代理 drop-in | [[阶段三-网络代理]] |
| 4 | 装输入法 + 主题 | [[阶段四-输入法]] |
| 5 | 配 Sunshine 串流 | [[阶段五-Sunshine串流]] |

---

## ⚙️ 系统配置

| 项目 | 规格 |
|:---|:---|
| 系统 | CachyOS Linux (Arch-based, 滚动更新) |
| 桌面 | KDE Plasma 6.7 · Wayland |
| 内核 | linux-cachyos 7.2.3 |
| GPU | NVIDIA RTX 4060 Max-Q + Intel UHD |
| AI | Hermes Agent v0.21.1 · 4 Agent |
| 代理 | Clash Verge Rev · 7897 端口 |
| 输入法 | Fcitx5 + 雾凇拼音 |
| 串流 | Sunshine + Moonlight |

---

## 📖 参考

| 页面 | 说明 |
|:---|:---|
| [[环境要求]] | 硬件/软件需求 |
| [[踩坑记录]] | 部署问题与解决方案 |

---

> 📅 最后更新：2026-09-10 · Hermes Agent 自动部署
