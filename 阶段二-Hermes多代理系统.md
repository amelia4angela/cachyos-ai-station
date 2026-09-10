> [Home](README.md) | 部署指南：[0](阶段零-前置授权.md) → [1](阶段一-环境检测.md) → [2](阶段二-Hermes多代理系统.md) → [3](阶段三-网络代理.md) → [4](阶段四-输入法.md) → [5](阶段五-Sunshine串流.md) | 参考：[环境要求](环境要求.md) [游戏环境](游戏环境.md) [踩坑记录](踩坑记录.md)

---

# 阶段二：Hermes 多代理系统

## 2.1 安装 SkillHub

SkillHub 是技能统一来源（国内优先）。

```bash
# 安装 CLI + 默认 Skill
curl -fsSL https://skillhub-1388575217.cos.ap-guangzhou.myqcloud.com/install/install.sh | bash

# 验证
skillhub --version
```

> 来源是腾讯云 COS（国内直连），无需代理。

## 2.2 用 SkillHub 安装技能

> ⚠️ `--dir` 是**顶层参数**，必须在子命令前！

### 菲菲（信息与文档专员）

```bash
skillhub --dir ~/.hermes/skills install agent-browser --namespace clawhub_rez0
skillhub --dir ~/.hermes/skills install weather --namespace clawhub_steipete
skillhub --dir ~/.hermes/skills install github --namespace clawhub_steipete
skillhub --dir ~/.hermes/skills install summarize --namespace user_814dbe54
```

### 薇薇（文档处理）

```bash
skillhub --dir ~/.hermes/skills install word-docx --namespace zcwl
skillhub --dir ~/.hermes/skills install excel-xlsx --namespace clawhub_ivangdavila
skillhub --dir ~/.hermes/skills install diagram-generator --namespace clawhub_matthewyin
skillhub --dir ~/.hermes/skills install pptx --namespace zcwl
# pdf：Hermes 自带，跳过
```

### 苏苏（系统与研发）

```bash
skillhub --dir ~/.hermes/skills install skill-creator --namespace clawhub_chindden
skillhub --dir ~/.hermes/skills install skill-vetter --namespace clawhub_spclaudehome
skillhub --dir ~/.hermes/skills install find-skills-skill --namespace clawhub_fangkelvin
skillhub --dir ~/.hermes/skills install skillhub-preference --namespace clawhub_pcicq
skillhub --dir ~/.hermes/skills install disk-watch --namespace clawhub_theshadowrose
skillhub --dir ~/.hermes/skills install api-gateway --namespace clawhub_byungkyu
skillhub --dir ~/.hermes/skills install auto-updater --namespace zcwl
```

## 2.3 Agent Profiles

用 `hermes config set` 注入（不要直接编辑 config.yaml）：

```bash
hermes config set agent.active_profile xiaohan --force

# 小涵（主控）
hermes config set agent.personalities.xiaohan.name '小涵' --force
hermes config set agent.personalities.xiaohan.soul '~/.hermes/personalities/xiaohan/soul.md' --force
hermes config set agent.personalities.xiaohan.role '主控' --force
hermes config set agent.personalities.xiaohan.skills '["delegate_task","cronjob","memory","todo"]' --force
hermes config set agent.personalities.xiaohan.description '本机AI总指挥' --force

# 菲菲、薇薇、苏苏 同理
```

## 2.4 soul.md 人格文件

4 个 Agent 各有独立的 soul.md：

| 文件 | Agent | 角色 |
|---|---|---|
| `~/.hermes/personalities/xiaohan/soul.md` | 小涵 | 主控，调度 Agent |
| `~/.hermes/personalities/feifei/soul.md` | 菲菲 | 信息与文档专员 |
| `~/.hermes/personalities/weiwei/soul.md` | 薇薇 | 文档处理专员 |
| `~/.hermes/personalities/susu/soul.md` | 苏苏 | 系统与研发专家 |

> 用户可以随时覆盖 soul.md，这是对 Agent 人格的最终控制权。

## 2.5 Gateway 重启

**AI 不能从内部重启 gateway**（会被拦截）。用户单独执行：

```bash
systemctl --user daemon-reload && \
  systemctl --user restart hermes-gateway.service && \
  sleep 3 && \
  systemctl --user show hermes-gateway.service --property=Environment
```

看到 `HTTP_PROXY=http://127.0.0.1:7897` 即生效。
