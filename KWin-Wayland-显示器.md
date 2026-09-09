# KWin / Wayland / 显示器

## kscreen-doctor 环境要求

kscreen-doctor 是 Qt GUI 程序，必须在桌面会话环境下运行。从 systemd 服务或普通终端调用会崩溃。

必须的环境变量：
- `WAYLAND_DISPLAY=wayland-0`
- `DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus`
- `XDG_RUNTIME_DIR=/run/user/1000`
- `QT_QPA_PLATFORM=wayland`

## kwinoutputconfig.json 清理

热插拔显示器会留下 ghost 条目（connectorName=None），导致屏幕闪烁。

清理脚本：
```python
#!/usr/bin/env python3
import json, os
CONFIG = os.path.expanduser("~/.config/kwinoutputconfig.json")
with open(CONFIG) as f:
    data = json.load(f)
for group in data:
    group["data"] = [d for d in group.get("data", []) if d.get("connectorName") is not None]
with open(CONFIG, "w") as f:
    json.dump(data, f, indent=4, ensure_ascii=False)
```

## D-Bus 激活文件命名

文件名必须与 D-Bus 名完全匹配：
- ❌ `org.kde.kscreen.service` (全小写)
- ✅ `org.kde.KScreen.service` (大写 S)

创建用户级覆盖：
```ini
# ~/.local/share/dbus-1/services/org.kde.KScreen.service
[D-BUS Service]
Name=org.kde.KScreen
Exec=/usr/lib/kf6/kscreen_backend_launcher
SystemdService=plasma-kscreen.service
```
