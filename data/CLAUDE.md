[根目录](../../CLAUDE.md) > **data**

# 数据存储模块

> 更新时间：2025-12-10 12:19:50

## 模块职责

data 模块存储项目运行所需的各类数据文件，包括 MongoDB 的导出数据、配置文件和演示数据。主要负责：
- 存储 K 线历史数据
- 存储缠论分析结果
- 存储系统配置数据
- 提供数据备份和恢复支持

## 目录结构

```
data/
├── config/                                   # 配置数据
│   ├── replay_config.bson                    # 回放配置数据
│   └── replay_config.metadata.json           # 元数据信息
├── nlchan/                                   # 自然缠论数据
│   ├── essence_xd_000001.XSHG_1d.bson       # 本质线段数据
│   └── essence_xd_000001.XSHG_1d.metadata.json  # 元数据
└── stock/                                    # 股票数据
    ├── replay_config.bson                   # 股票回放配置
    ├── replay_config.metadata.json          # 元数据
    ├── stk_000001.XSHG_1d.bson             # 股票K线数据
    ├── stk_000001.XSHG_1d.metadata.json    # 元数据
    ├── stock_names.bson                    # 股票名称列表
    └── stock_names.metadata.json           # 元数据
```

## 数据类型说明

### 1. 配置数据 (config/)
- **replay_config.bson**: 回放功能的配置参数
- 用于控制 K 线回放的行为和设置

### 2. 缠论数据 (nlchan/)
- **essence_xd_*.bson**: 存储缠论本质线段数据
- 文件命名格式：`essence_xd_{股票代码}_{周期}.bson`
- 示例：`essence_xd_000001.XSHG_1d.bson`（平安银行日线数据）

### 3. 股票数据 (stock/)
- **stk_*.bson**: 股票 K 线数据
- **stock_names.bson**: 所有股票的基础信息
- 包含股票代码、名称、交易状态等信息

## 文件格式说明

### BSON 文件
- MongoDB 的二进制数据格式
- 用于快速数据导入和恢复
- 包含完整的文档结构和数据类型

### Metadata JSON 文件
- 记录对应 BSON 文件的元信息
- 包含：
  - 文件创建时间
  - 数据记录数
  - 数据版本
  - 导出配置

### 示例 Metadata 内容
```json
{
    "created_at": "2025-12-10T12:00:00Z",
    "record_count": 5000,
    "version": "1.0",
    "collection": "stock_kline"
}
```

## 数据恢复

### 使用恢复脚本
```bash
cd hetl/hmgo
./restore_chanvis_mongo.sh
```

### 手动恢复
```bash
# 恢复单个集合
mongorestore --db nlchan --collection essence_xd_000001.XSHG_1d data/nlchan/essence_xd_000001.XSHG_1d.bson

# 恢复整个数据库
mongorestore --db nlchan data/nlchan/
mongorestore --db stock data/stock/
mongorestore --db config data/config/
```

## 数据说明

### 演示数据
- 平安银行 (000001.XSHG) 的日线数据
- 包含完整的缠论分析结果
- 适合用于系统演示和测试

### 数据时间范围
- 根据实际导出时间确定
- 建议保留至少 1 年的历史数据
- 用于充分展示缠论分析效果

## 测试与质量

### 当前状态
- ✅ 演示数据已提供
- ✅ 数据结构完整
- ❌ 缺少数据更新机制
- ❌ 缺少数据验证

### 改进建议
1. 实现自动化数据更新
2. 添加数据完整性校验
3. 提供多时间周期的数据
4. 增加更多股票的示例数据

## 常见问题 (FAQ)

### Q: 如何更新数据？
A: 使用 hetl 模块的数据获取脚本，然后导出为 BSON

### Q: 可以直接修改 BSON 文件吗？
A: 不建议，应通过 MongoDB 操作或使用专门的 BSON 工具

### Q: 数据文件很大怎么办？
A: 可以考虑使用数据压缩或分批处理

### Q: 如何添加新的交易品种数据？
A: 通过 hetl/get_jqdata.py 获取数据，然后保存到相应目录

## 相关文件清单

```
data/
├── config/                    # 配置数据目录
├── nlchan/                    # 缠论数据目录
└── stock/                     # 股票数据目录
```

## 变更记录 (Changelog)

### 2025-12-10 12:19:50
- ✨ 创建 data 模块文档
- 📋 整理数据目录结构
- 🔧 说明文件格式和用途
- 💡 添加数据恢复方法

---

*返回[根目录](../../CLAUDE.md)*