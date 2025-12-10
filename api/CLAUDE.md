[根目录](../../CLAUDE.md) > **api**

# API 服务模块

> 更新时间：2025-12-10 12:19:50

## 模块职责

API 模块是 ChanVis 项目的后端核心，基于 Flask 框架提供 RESTful API 服务。主要负责：
- 提供 TradingView 图表所需的数据接口
- 处理 K 线历史数据查询
- 管理交易品种（股票、加密货币）信息
- 提供缠论结构数据的查询接口

## 入口与启动

### 主程序入口
- **文件**: `chanapi.py`
- **启动方式**: `python chanapi.py`
- **默认端口**: 8421
- **服务地址**: http://127.0.0.1:8421

### 依赖安装
```bash
pip install -r requirements.txt
```

### 依赖包清单
- flask: Web框架
- flask_cors: 跨域支持
- arrow: 时间处理
- pymongo: MongoDB连接
- pandas: 数据处理
- pynput: 键盘监听（用于调试）
- termcolor: 终端颜色输出

## 对外接口

### 1. 配置接口 `/api/config`
- **方法**: GET
- **功能**: 返回 TradingView 图表配置信息
- **返回**: 支持的时间周期、功能开关等

### 2. 搜索接口 `/api/search`
- **方法**: GET
- **参数**: query（搜索关键词）
- **功能**: 搜索支持的交易品种
- **返回**: 匹配的品种列表

### 3. 品种信息 `/api/symbols`
- **方法**: GET
- **参数**: symbol（品种代码）
- **功能**: 获取指定品种的详细信息
- **返回**: 品种名称、交易所、精度等信息

### 4. 历史数据 `/api/history`
- **方法**: GET
- **参数**:
  - symbol: 交易品种
  - from: 起始时间戳
  - to: 结束时间戳
  - resolution: 时间周期
- **功能**: 获取K线历史数据
- **返回**: OHLCV数据

### 5. 标记数据 `/api/marks`
- **方法**: GET
- **参数**:
  - symbol: 交易品种
  - from: 起始时间戳
  - to: 结束时间戳
  - resolution: 时间周期
- **功能**: 获取缠论标记数据（线段、中枢等）
- **返回**: 图形标记信息

### 6. 时间刻度 `/api/timescale_marks`
- **方法**: GET
- **功能**: 获取时间刻度标记
- **返回**: 特殊时间点标记

### 7. 实时数据 `/api/quote`
- **方法**: GET
- **功能**: 获取实时报价数据（模拟）
- **返回**: 当前价格信息

## 关键依赖与配置

### 数据库连接
- **MongoDB主机**: localhost:27017
- **数据库**:
  - `nlchan`: 缠论结构数据
  - `ohlcv`: K线历史数据
  - `stock`: 股票基础信息
  - `config`: 配置数据

### 配置文件
主要配置项来自 `comm/conf.py`：
- `DATA_PATH`: 数据文件路径
- `RESOU_DICT`: 时间周期映射
- `TF_SEC_MAP`: 时间周期秒数映射
- `DATE_START_TS`: 各周期数据起始时间
- `MAX_XD_LEN`: 线段最大条数限制

### 支持的品种
- **加密货币**: 从 `hetl/selcoin/binance_syms.txt` 加载
- **股票**: 从 MongoDB stock_names 集合加载

## 数据模型

### K线数据格式
```python
{
    "time": int,      # Unix时间戳
    "open": float,    # 开盘价
    "high": float,    # 最高价
    "low": float,     # 最低价
    "close": float,   # 收盘价
    "volume": float   # 成交量
}
```

### 品种信息格式
```python
{
    "name": str,              # 品种名称
    "symbol": str,            # 品种代码
    "description": str,       # 描述
    "exchange": str,          # 交易所
    "minmov": int,            # 最小变动单位
    "pricescale": int,        # 价格精度
    "has_intraday": bool,     # 是否支持日内数据
    "type": str,              # 类型（stock/bitcoin）
    "session": str,           # 交易时段
    "timezone": str,          # 时区
    "intraday_multipliers": list  # 支持的周期
}
```

### 缠论标记格式
```python
{
    "id": str,           # 标记唯一ID
    "time": int,         # 时间戳
    "color": str,        # 颜色
    "text": str,         # 文本
    "label": str,        # 标签
    "labelFontColor": str,
    "fixedSize": bool    # 是否固定大小
}
```

## 测试与质量

### 当前状态
- ✅ API接口已实现
- ❌ 缺少单元测试
- ❌ 缺少API文档
- ❌ 缺少错误处理优化

### 测试建议
1. 使用 pytest 框架编写单元测试
2. 测试各API端点的响应格式
3. 测试数据库查询的正确性
4. 测试错误场景的处理

### 性能优化
- 为频繁查询的字段建立索引
- 实现数据缓存机制
- 优化大数据量的分页查询

## 常见问题 (FAQ)

### Q: 如何添加新的交易品种？
A:
- 加密货币：编辑 `hetl/selcoin/binance_syms.txt`
- 股票：通过 `hetl/stock/get_jqdata.py` 导入

### Q: 如何修改数据存储路径？
A: 修改 `comm/conf.py` 中的 `DATA_PATH` 配置

### Q: API响应慢怎么办？
A:
1. 检查 MongoDB 索引
2. 优化查询条件
3. 考虑添加 Redis 缓存

### Q: 如何支持实时数据？
A: 需要集成实时数据源（如WebSocket），并在 `/api/quote` 接口中实现

## 相关文件清单

```
api/
├── chanapi.py         # 主程序，所有API接口实现
├── requirements.txt   # Python依赖包列表
└── symbol_info.py     # 交易品种信息定义
```

## 变更记录 (Changelog)

### 2025-12-10 12:19:50
- ✨ 创建 API 模块文档
- 📋 整理所有API接口说明
- 🔧 明确数据模型和配置项
- 💡 添加测试建议和优化方向

---

*返回[根目录](../../CLAUDE.md)*