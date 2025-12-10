[根目录](../../CLAUDE.md) > **comm**

# 公共配置模块

> 更新时间：2025-12-10 12:19:50

## 模块职责

comm 模块存放项目的公共配置和常量定义，是整个项目的基础配置中心。主要负责：
- 定义全局配置参数
- 管理数据库连接
- 定义时间周期映射
- 存储交易品种信息
- 提供日志配置

## 入口与启动

### 主要文件
- **conf.py**: 核心配置文件
- **__init__.py**: 模块初始化（空文件）

### 使用方式
```python
from comm.conf import (
    DATA_PATH,      # 数据路径
    STOCK_DB,       # 股票数据库
    CHAN_DB,        # 缠论数据库
    TF_SEC_MAP,     # 时间周期映射
    ALL_SYMBOLS     # 所有交易品种
)
```

## 关键配置项

### 1. 路径配置
```python
ROOT_PATH = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
DATA_PATH = os.path.join(ROOT_PATH, 'data')
```

### 2. 时间周期映射
```python
# TradingView 周期到内部周期映射
RESOU_DICT = {
    "1": "1m",
    "5": "5m",
    "15": "15m",
    "30": "30m",
    "60": "1h",
    "240": "4h",
    "1D": "1d",
    "1W": "1w"
}

# 时间周期到秒数映射
TF_SEC_MAP = {
    '1m': 1 * 60,
    '5m': 5 * 60,
    '30m': 30 * 60,
    '1h': 1 * 60 * 60,
    '4h': 4 * 60 * 60,
    '1d': 24 * 60 * 60,
    '1w': 7 * 24 * 60 * 60
}
```

### 3. 数据库连接
```python
client = MongoClient('localhost', 27017)
CHAN_DB = client.nlchan      # 缠论数据
HIST_DB = client.ohlcv       # K线历史数据
STOCK_DB = client.stock      # 股票信息
CONF_DB = client.config      # 配置数据
```

### 4. 交易品种配置
- 加载自 `hetl/selcoin/binance_syms.txt`（调试模式用 binance_syms.debug）
- 过滤特殊品种（DAI, TUSD）
- 支持的周期根据 DEBUG 模式不同而变化

### 5. 时间范围配置
```python
# 各周期数据起始时间戳
DATE_START_TS = {
    '1m': 1635724800,  # 2021-11-01 08:00:00
    '5m': now.shift(days=-base_days).int_timestamp,
    # ... 其他周期
}
```

## 数据模型

### K线数据列定义
```python
# 各数据源的不同列名
HB_KDATA_COLUMNS = ['id', 'datetime', 'open', 'high', 'low', 'close', 'count', 'vol', 'amount']
OK_KDATA_COLUMNS = ['ts', 'datetime', 'open', 'high', 'low', 'close', 'cnt', 'volume', 'currency_volume']
STAND_KDATA_COLUMNS = ['id', 'datetime', 'open', 'high', 'low', 'close', 'volume', 'amount']
GP_KDATA_COLUMNS = ["ts", "datetime", "open", "high", "low", "close", "volume"]
```

### MongoDB 集合命名规则
```python
# 缠论本质线段
ESSENCE_XD_COL = 'essence_xd_{sym}_{tf}'
# 缠论本质中枢
ESSENCE_ZS_COL = 'essence_zs_{sym}_{tf}'
# 自然缠论线段
LNCHAN_XD_COL = 'lnchan_xd_{sym}_{tf}'
```

## 常量说明

### 时间相关
- `MAX_XD_LEN`: 线段最大条数（500）
- `TF_DAY_KLINE_CNT`: 一天中各周期的K线数量
- `ALL_TIMEFRAMES`: 所有支持的时间周期

### 价格相关
- `BIGEST_PRICE`: 最大价格（999999999）
- `minmov`: 最小变动单位（从配置文件读取）
- `pricescale`: 价格精度（根据最小变动单位计算）

### 交易品种特殊处理
- `SPECILS`: 需要特殊处理的品种列表
- `DEBUG`: 调试模式开关，影响数据源和周期选择

## 测试与质量

### 当前状态
- ✅ 配置项定义完整
- ✅ 支持调试和生产模式切换
- ❌ 缺少配置验证
- ❌ 缺少配置文档说明

### 改进建议
1. 添加配置项验证，确保值的有效性
2. 使用环境变量管理敏感配置
3. 提供配置模板和示例
4. 添加配置项的详细注释

## 常见问题 (FAQ)

### Q: 如何修改数据库连接？
A: 修改 MongoClient 的连接参数，支持远程数据库

### Q: 如何添加新的时间周期？
A: 同时更新 RESOU_DICT 和 TF_SEC_MAP

### Q: DEBUG 模式有什么区别？
A: DEBUG 模式使用更少的数据和不同的配置文件

### Q: 如何调整数据起始时间？
A: 修改 DATE_START_TS 或调整 base_days 值

## 相关文件清单

```
comm/
├── __init__.py    # 模块初始化文件
└── conf.py        # 核心配置文件
```

## 变更记录 (Changelog)

### 2025-12-10 12:19:50
- ✨ 创建 comm 模块文档
- 📋 整理所有配置项说明
- 🔧 明确配置项的用途和关系
- 💡 添加改进建议和使用说明

---

*返回[根目录](../../CLAUDE.md)*