> [Home](README.md) | 部署指南：[0](阶段零-前置授权.md) → [1](阶段一-环境检测.md) → [2](阶段二-Hermes多代理系统.md) → [3](阶段三-网络代理.md) → [4](阶段四-输入法.md) → [5](阶段五-Sunshine串流.md) | 参考：[环境要求](环境要求.md) [游戏环境](游戏环境.md) [踩坑记录](踩坑记录.md)

---

# AI 执行边界

Hermes Agent 在本机执行操作时的权限分级。

## 权限分级表

| 级别 | 操作 | AI 能做？ | 说明 |
|:---:|:---|:---:|:---|
| 🔵 用户级 | `hermes config set/get` | ✅ | Hermes 自己的配置 |
| 🔵 用户级 | `systemctl --user start/stop/enable/disable` | ✅ | user 服务管理 |
| 🔵 用户级 | `skillhub install/search` | ✅ | 技能安装 |
| 🔵 用户级 | `fcitx5-configtool` | ✅ | 输入法配置 |
| 🔵 用户级 | 改 `~/.config/...` | ✅ | 用户目录文件 |
| 🟡 普通 sudo | `sudo systemctl start clash-verge-service` | ✅ | 启动系统服务（低风险） |
| 🟡 普通 sudo | `paru -S package` | ⚠️ 先问 | AUR 装包需用户确认 |
| 🟠 敏感 sudo | `sudo pacman -S package` | ⚠️ 先问 | 装包需用户确认 |
| 🟠 敏感 sudo | `sudo pacman -R package` | ⚠️ 先问 | 删包需用户确认 |
| 🟠 敏感 sudo | 改 `/etc/...` | ⚠️ 先问 | 系统配置需用户确认 |
| 🟠 敏感 sudo | `sudo loginctl enable-linger` | ⚠️ 先问 | 一次性但需确认 |
| 🔴 禁止 | `systemctl restart hermes-gateway` | ❌ | Hermes 护栏拦截（自杀保护） |
| 🔴 禁止 | `systemctl stop hermes-gateway` | ❌ | 同上 |
| 🔴 禁止 | `systemctl disable hermes-gateway` | ❌ | 同上 |
| 🔴 极高危 | 改 `/etc/sudoers.d/...` | ❌ 必须二次确认 | 安全敏感 |
| 🔴 极高危 | `rm -rf /...` | ❌ 必须二次确认 | 不可逆操作 |

## 判定流程

```
操作涉及 sudo?
├─ 否 → 直接执行
└─ 是 → 命令危险?
       ├─ 装包/改系统配置 → ⚠️ clarify 询问用户
       ├─ 启动/重启 service → ✅ 直接执行（低危）
       └─ 改 sudoers / 防火墙 / 删数据 → ❌ 必须二次确认
```

## 关键原则

1. **先检查，再执行** — 不盲目修改系统
2. **不覆盖已有正确配置** — 备份后再改
3. **不删除用户数据** — 除非用户明确要求
4. **保留执行记录** — 每次操作可追溯
5. **危险操作必须确认** — rm / 格式化 / 磁盘 / 防火墙 / 核心配置

## Hermes 护栏（不可绕过）

Hermes 内置 tirith 安全系统，以下操作**即使 sudo 也无法执行**：

- `systemctl restart hermes-gateway` — 会杀父进程链（自杀）
- `systemctl stop hermes-gateway` — 同上
- `sudo -S` 密码管道 — 识别为密码爆破

**唯一解决办法**：用户在另一个终端手动执行。

## 如何让执行边界每次会话都生效

通过两层机制确保 AI 每次启动都遵守执行边界：

### 1. Memory（自动注入）

执行边界规则写入 Hermes 的 **memory**（`~/.hermes/memories/`），每次会话启动时**自动注入**到上下文：

```
AI 执行边界（每次会话必须遵守）：
- 用户级操作直接执行
- 装包/改系统配置先 clarify 问用户
- 禁止 restart/stop/disable hermes-gateway
- 禁止改 /etc/sudoers.d/ 未经二次确认
- 危险操作（rm/格式化/磁盘/防火墙）必须确认
```

**特点**：不限于某个 Agent 人格，所有会话都生效。

### 2. soul.md（人格加载时）

小涵的 `~/.hermes/personalities/xiaohan/soul.md` 包含完整的安全原则、sudo 原则、命令原则，作为人格定义的一部分加载。

**特点**：仅小涵人格生效，其他 Agent 需要各自的 soul.md 也写入。

### 3. 补充：用户偏好也写入 memory

用户的个性化偏好（如"装软件前必须问"）同样写入 memory，跨会话持久生效。

> 💡 **建议**：核心规则放 memory（全局生效），详细原则放 soul.md（人格相关）。
