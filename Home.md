# Hermes Agent 部署经验与踩坑记录

**环境**: CachyOS Linux + KDE Plasma 6 (Wayland) + ASUS ROG Strix G814JV
**记录时间**: 2026-09-09 (持续更新)
**总计**: 50+ 条踩坑

## 快速导航

| 章节 | 内容 |
|------|------|
| [[代理与网络]] | NTP 污染、SOCKS5、Thunderbird 代理 |
| [[Thunderbird-Gmail]] | IMAP 连接、DNS 污染、Profile 冲突 |
| [[KWin-Wayland-显示器]] | kscreen-doctor 崩溃、显示配置 |
| [[Sunshine-串流]] | 串流配置、自动切换显示器、capture 方式 |
| [[Portal-DBus]] | ScreenCast 接口、多 portal 冲突 |
| [[系统-systemd]] | logind、Linger、D-Bus 激活 |
| [[AUR-包管理]] | sunshine AUR 版、makepkg 依赖 |
| [[字体与音频]] | 中文字体、EasyEffects 预设 |
| [[fcitx5-输入法]] | Wayland 启动、主题配置 |
| [[NVIDIA-独显]] | 独显直连、亮度、NVENC |
| [[Hermes-AI-Agent]] | gateway 重启、drop-in 验证 |
| [[调试技巧]] | 常用排查命令 |
| [[设计决策]] | 技术选型原因 |
| [[关键经验]] | 最重要的教训 |
