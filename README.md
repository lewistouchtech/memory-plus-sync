# Memory Plus - 跨渠道记忆同步工具

> 实现多渠道记忆采集、统一存储、实时监控和异常告警

---

## 快速开始

### 1. 演示模式

```bash
python3 main.py demo
```

### 2. 同步消息

```bash
# 同步最近 24 小时的所有渠道
python3 main.py sync

# 同步最近 2 小时的飞书和语音消息
python3 main.py sync --channels feishu,voice --hours 2
```

### 3. 监控状态

```bash
# 单次检查
python3 main.py monitor --once

# 持续监控
python3 main.py monitor
```

### 4. 健康检查

```bash
python3 main.py health
```

---

## 功能特性

✅ **跨渠道同步**
- 飞书消息采集
- 微信消息采集（框架就绪）
- Telegram 消息采集（框架就绪）
- 语音对话记录采集
- 统一存储到官方 SQLite

✅ **实时监控**
- 数据库连通性
- 记忆写入新鲜度
- FTS 索引一致性
- 数据库完整性
- 数据库大小

✅ **异常告警**
- 停滞检测（>1 小时）
- 严重停滞（>2 小时）
- 数据库损坏
- 索引不一致
- 告警日志（JSONL）

✅ **自动恢复**
- 数据库备份
- VACUUM 修复
- 索引重建

---

## 核心模块

### memory_plus.py
核心功能模块
- `MemoryPlus` 类
- 数据库操作
- 监控功能
- 健康检查

### monitor.py
监控守护进程
- 实时监控循环
- 告警处理
- 自动恢复

### collector.py
多渠道采集器
- `FeishuCollector` - 飞书采集
- `WechatCollector` - 微信采集
- `TelegramCollector` - Telegram 采集
- `VoiceCollector` - 语音采集
- `MultiChannelCollector` - 统一管理

---

## 数据库位置

- **主数据库**: `~/.openclaw/memory/main.sqlite`
- **记忆文件**: `~/.openclaw/memory/*.md`
- **渠道记忆**: `~/.openclaw/memory/{channel}/YYYY-MM-DD.md`

---

## 日志文件

- **监控日志**: `~/.openclaw/workspace/logs/memory_plus_monitor.log`
- **告警日志**: `~/.openclaw/workspace/logs/memory_plus_alerts.jsonl`
- **统计日志**: `~/.openclaw/workspace/logs/memory_plus_stats.json`

---

## Python API

```python
from memory_plus import MemoryPlus
from collector import MultiChannelCollector
from datetime import datetime, timedelta

# 同步示例
mp = MemoryPlus()
if mp.connect():
    # 采集消息
    mcc = MultiChannelCollector()
    messages = mcc.collect_and_merge(
        channels=['feishu'],
        start_time=datetime.now() - timedelta(hours=2)
    )
    
    # 同步到数据库
    mp.sync_from_channel('feishu', messages)
    
    # 监控状态
    result = mp.monitor_official_system()
    print(f"状态：{result['status']}")
    
    mp.close()
```

---

## 定时任务

```bash
# 每小时健康检查
0 * * * * python3 /path/to/main.py health

# 每 5 分钟监控
*/5 * * * * python3 /path/to/main.py monitor --once

# 每天同步
0 2 * * * python3 /path/to/main.py sync --hours 24
```

---

## 状态说明

### 正常 (normal)
- 记忆系统正常工作
- 最近 2 小时内有写入
- 数据库完整性 ok

### 警戒 (warning)
- 1-2 小时未写入
- FTS 索引轻微不一致
- 数据库大小 50-100MB

### 严重 (critical)
- >2 小时未写入
- 数据库损坏
- FTS 索引严重不一致

---

## 故障排查

### 问题 1：数据库连接失败
```bash
# 检查数据库文件
ls -lh ~/.openclaw/memory/main.sqlite

# 检查权限
chmod 644 ~/.openclaw/memory/main.sqlite
```

### 问题 2：FTS 索引不一致
```bash
# 查看详细信息
python3 main.py monitor --once

# 系统会自动尝试修复
```

### 问题 3：记忆停滞
```bash
# 检查最新写入时间
python3 -c "
import sqlite3
conn = sqlite3.connect('~/.openclaw/memory/main.sqlite')
cursor = conn.cursor()
cursor.execute('SELECT MAX(updated_at) FROM chunks')
ts = cursor.fetchone()[0]
from datetime import datetime
print('最新写入:', datetime.fromtimestamp(ts/1000))
"
```

---

## 贡献与改进

欢迎提交 Issue 和 Pull Request！

### 待办事项
- [ ] 集成真实 Embedding API
- [ ] 完善微信消息采集
- [ ] 完善 Telegram 消息采集
- [ ] 添加语义去重
- [ ] Web 管理界面

---

## 许可证

MIT License

---

## 联系方式

作者：伊娃 (Eva)  
公司：伊娃人工智能有限公司  
日期：2026-04-07
