[根目录](../../CLAUDE.md) > **utils**

# 工具函数模块

> 更新时间：2025-12-10 12:19:50

## 模块职责

utils 模块提供项目的通用工具函数和辅助类，是项目的基础工具库。主要负责：
- 日期时间处理工具
- 缠论相关的算法辅助函数
- 通用的数据处理工具

## 入口与启动

### 主要文件
- **__init__.py**: 模块初始化
- **dtlib.py**: 日期时间处理库
- **nlchan.py**: 自然缠论相关工具

### 使用方式
```python
from utils.dtlib import time2int, int2time
from utils.nlchan import sym_float
```

## 核心功能

### 1. dtlib.py - 日期时间工具

提供 Unix 时间戳和日期字符串之间的转换功能。

#### 主要函数
```python
# 时间戳转换相关（具体实现需要查看文件内容）
# time2int() - 时间转时间戳
# int2time() - 时间戳转时间
```

### 2. nlchan.py - 缠论工具

提供缠论分析相关的辅助函数。

#### 主要函数
```python
def sym_float(minmov):
    """获取小数点的位数

    Args:
        minmov (str): 最小变动单位，如 '0.01', '0.001'

    Returns:
        tuple: (ps, bits) - 价格精度和小数位数
    """
    # 解析最小变动单位
    # 计算价格精度（用于 TradingView）
    # 返回精度和小数位数
```

#### 算法逻辑
1. 解析 minmov 字符串，找到小数点后的位数
2. 计算价格精度 ps（10的幂次）
3. 返回精度值和位数统计

#### 示例
```python
# minmov = '0.01' -> ps = 100, bits = 2
# minmov = '0.001' -> ps = 1000, bits = 3
# minmov = '1' -> ps = 1, bits = 0
```

## 数据模型

### 时间格式
- 支持多种时间格式之间的转换
- Unix 时间戳（秒级）
- 标准日期时间字符串
- TradingView 需要的时间格式

### 价格精度
- 基于最小变动单位计算
- 用于 TradingView 图表的显示精度
- 影响价格标签的显示格式

## 测试与质量

### 当前状态
- ✅ 基础工具函数已实现
- ❌ 缺少单元测试
- ❌ 缺少函数文档
- ❌ 需要补充更多工具函数

### 测试建议
```python
# 测试 sym_float 函数
def test_sym_float():
    assert sym_float('0.01') == (100, 2)
    assert sym_float('0.001') == (1000, 3)
    assert sym_float('1') == (1, 0)
```

### 扩展建议
1. 添加更多日期时间处理函数
2. 添加数值格式化工具
3. 添加文件操作工具
4. 添加数据验证工具

## 常见问题 (FAQ)

### Q: 如何处理不同时区的时间？
A: 需要在 dtlib 中添加时区转换功能

### Q: sym_float 函数为什么返回两个值？
A: ps 用于 TradingView 的 pricescale，bits 用于其他计算

### Q: 如何处理特殊的价格精度？
A: 可以扩展 sym_float 函数支持更多格式

## 相关文件清单

```
utils/
├── __init__.py    # 模块初始化
├── dtlib.py       # 日期时间处理工具
└── nlchan.py      # 缠论相关工具
```

## 变更记录 (Changelog)

### 2025-12-10 12:19:50
- ✨ 创建 utils 模块文档
- 📋 整理现有工具函数
- 🔧 说明函数用途和参数
- 💡 添加扩展建议

---

*返回[根目录](../../CLAUDE.md)*