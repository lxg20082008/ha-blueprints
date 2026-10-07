# HA 蓝图（Home Assistant Blueprints）

个人的 Home Assistant 自动化蓝图集合，通过「导入蓝图」即可使用。

## 蓝图列表

### 电池低电量检测和通知（含事件型）

检测家里所有低电量设备并发送通知，覆盖三种类型：

| 类型 | 检测方式 | 设备示例 |
|---|---|---|
| 百分比型 | `sensor.*`，`device_class: battery`，值低于阈值 | 小米温湿度计 2（BLE） |
| 二进制型 | `binary_sensor.*`，`device_class: battery`，值为 `on` | — |
| 事件型 | `event.*_low_battery`，勾选的实体状态非空 | 小米 Zigbee 传感器 |

#### 导入方法

设置 → 自动化与场景 → **蓝图** → **导入蓝图**，粘贴下面的**文件地址**：

```
https://raw.githubusercontent.com/lxg20082008/ha-blueprints/main/low_battery.yaml
```

也可以贴 GitHub 文件页地址（HA 会自动转 raw）：

```
https://github.com/lxg20082008/ha-blueprints/blob/main/low_battery.yaml
```

> ⚠️ 结尾必须是 `/low_battery.yaml`。贴仓库主页 `https://github.com/lxg20082008/ha-blueprints` 会报「mapping values are not allowed here」——因为主页是 HTML，不是 YAML。

#### 配置项

| 输入 | 说明 | 默认值 |
|---|---|---|
| 电池警告阈值 | 百分比低于此值视为低电量 | 20% |
| 检测时间 | 每天定时全量检测一次 | 10:00 |
| 检测星期 | 0=每天，1~7=周一~周日 | 0 |
| 排除的传感器 | 不参与提醒的设备（手机等） | 空 |
| 事件型低电量实体 | 只报 `event.*_low_battery` 的设备，勾选（可多选） | 空 |
| 通知动作 | 低电量时执行的动作 | — |

#### 通知动作怎么填

「通知动作」选 **持久通知**（或 `notify.mobile_app_xxx` 手机推送），字段这样填：

| 字段 | 填写 |
|---|---|
| 消息 | `低电量设备：{{sensors}}` |
| 标题 | `🔋 低电量提醒` |
| 通知标识符 | `low_battery`（固定 ID，新通知覆盖旧的；想保留多条就留空） |

`{{sensors}}` 运行时会替换成实际低电量设备列表，例如：

> 低电量设备：书房 温湿度计2 (15%)、弱电箱 温湿度传感器

#### 触发逻辑

- **每天定时**：全量扫描，一次通知列出所有低电量设备。
- **即时触发**：勾选的事件型设备一变低电量，立即单独通知（逐实体 `state` 触发，每个设备各自提醒）。

#### 已知局限

事件型设备**不会自动识别**，必须手动勾选进「事件型低电量实体」。新加一个事件型设备时记得回来勾上，否则它既不会即时触发，也**不会被每天定时扫到**（定时扫的就是你勾选的那份列表）。
