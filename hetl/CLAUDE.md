[根目录](../../CLAUDE.md) > **hetl**

# 数据ETL模块

> 更新时间：2025-12-10 12:19:50

## 模块职责

hetl (History ETL) 模块负责历史数据的抽取、转换和加载工作，是项目数据获取和处理的核心。主要负责：
- 获取股票历史数据
- 获取加密货币数据
- 数据格式转换和清洗
- 数据导入 MongoDB
- 提供数据恢复脚本

## 入口与启动

### 目录结构
```
hetl/
├── hmgo/                    # MongoDB 相关
│   └── restore_chanvis_mongo.sh  # 数据恢复脚本
├── selcoin/                 # 加密货币选择
│   ├── __init__.py
│   └── binance_syms.txt    # 币安交易对列表
└── stock/                   # 股票数据处理
    └── get_jqdata.py       # 聚宽数据获取脚本
```

### 数据恢复
```bash
cd hetl/hmgo
./restore_chanvis_mongo.sh
```

## 核心功能

### 1. 股票数据获取 (get_jqdata.py)

#### 依赖
- **jqdatasdk**: 聚宽数据SDK
- **jqauth**: 自定义认证模块（需自行实现）

#### 主要功能
```python
# 获取所有股票、指数、基金
def get_all_secs():
    """获取所有的股票，指数，基金数据"""

# 获取历史K线数据
def get_day_hist(symbol, tf='1d', start_date="2005-01-01", end_date="2022-12-31"):
    """获取日线数据"""
```

#### 数据处理流程
1. 从聚宽获取原始数据
2. 转换时间格式（调整为交易时间）
3. 添加时间戳字段
4. 过滤无成交量的数据
5. 存储到 MongoDB

### 2. 加密货币数据 (selcoin/)

#### binance_syms.txt 格式
```
BTCUSDT 0.01
ETHUSDT 0.01
...
```
每行包含：交易对名称 和 最小变动单位

#### 特殊处理
- 过滤稳定币（DAI, TUSD）
- 支持调试模式（使用 binance_syms.debug）

### 3. MongoDB 数据恢复

#### restore_chanvis_mongo.sh
- 恢复预处理的 BSON 数据
- 包含：
  - 配置数据 (config/)
  - 缠论数据 (nlchan/)
  - 股票数据 (stock/)

## 数据模型

### 股票数据格式
```python
{
    "datetime": str,      # 日期时间字符串
    "ts": int,           # Unix时间戳
    "open": float,       # 开盘价
    "high": float,       # 最高价
    "low": float,        # 最低价
    "close": float,      # 收盘价
    "volume": float      # 成交量
}
```

### 股票基础信息
```python
{
    "code": str,         # 股票代码
    "display_name": str, # 显示名称
    "name": str          # 股票名称
}
```

### 加密货币信息
```python
{
    "symbol": str,       # 交易对
    "minmov": str        # 最小变动单位
}
```

## 测试与质量

### 当前状态
- ✅ 股票数据获取已实现
- ✅ 加密货币列表已配置
- ✅ 数据恢复脚本已提供
- ❌ 缺少数据质量检查
- ❌ 缺少增量更新机制
- ❌ 缺少错误处理和重试

### 改进建议
1. 添加数据验证逻辑
2. 实现增量数据更新
3. 添加多数据源支持
4. 实现数据备份和恢复策略
5. 添加数据清洗规则

## 常见问题 (FAQ)

### Q: 如何配置聚宽数据认证？
A: 需要创建 jqauth.py 文件，实现 auth() 函数

### Q: 如何添加新的加密货币？
A: 编辑 binance_syms.txt，添加新的交易对

### Q: 数据恢复失败怎么办？
A: 检查 MongoDB 是否正常运行，确认数据文件路径

### Q: 如何获取实时数据？
A: 需要集成实时数据源的API

## 相关文件清单

```
hetl/
├── hmgo/
│   └── restore_chanvis_mongo.sh    # MongoDB数据恢复脚本
├── selcoin/
│   ├── __init__.py                # 模块初始化
│   └── binance_syms.txt          # 币安交易对列表
└── stock/
    └── get_jqdata.py             # 聚宽数据获取
```

## 变更记录 (Changelog)

### 2025-12-10 12:19:50
- ✨ 创建 hetl 模块文档
- 📋 整理数据获取和处理流程
- 🔧 说明各组件的用途
- 💡 添加改进建议

---

*返回[根目录](../../CLAUDE.md)*