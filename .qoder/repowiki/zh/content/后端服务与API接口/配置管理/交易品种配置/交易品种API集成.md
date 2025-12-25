# 交易品种API集成

<cite>
**本文档引用的文件**   
- [chanapi.py](file://api/chanapi.py#L61-L93)
- [symbol_info.py](file://api/symbol_info.py#L4-L73)
- [ChanContainer.vue](file://ui/src/components/ChanContainer.vue#L75-L1576)
</cite>

## 目录
1. [API端点说明](#api端点说明)
2. [API响应结构](#api响应结构)
3. [SUPPORT_SYMBOLS索引机制](#support_symbols索引机制)
4. [前端调用场景](#前端调用场景)
5. [客户端集成示例](#客户端集成示例)
6. [性能考量](#性能考量)

## API端点说明

`/api/search` 和 `/api/symbols` 是两个用于交易品种搜索和查询的核心API端点，均通过HTTP GET方法提供服务。

`/api/search` 端点用于支持前端搜索功能，允许用户通过前缀匹配来查找交易品种。该端点接收一个名为 `query` 的查询参数。当 `query` 参数为 "all" 时，返回所有支持的交易品种列表；否则，使用正则表达式对 `query` 字符串进行处理（在每个字符间插入 `.*`），然后在 `SUPPORT_SYMBOLS` 列表中搜索名称（`name` 字段）与该正则表达式匹配的所有品种。

`/api/symbols` 端点用于根据指定的符号（symbol）获取完整的品种信息。该端点接收一个名为 `symbol` 的查询参数。它会遍历 `SUPPORT_SYMBOLS` 列表，使用与 `/api/search` 相同的正则表达式匹配逻辑，优先匹配品种的 `name` 字段，其次匹配 `symbol` 字段。一旦找到匹配项，立即返回该品种的完整信息对象；如果未找到，则返回一个空对象。

**Section sources**
- [chanapi.py](file://api/chanapi.py#L61-L93)

## API响应结构

### `/api/search` 响应
当请求 `/api/search?query=all` 时，响应为一个包含所有支持品种的JSON数组，每个品种对象包含以下字段：
- **name**: 品种名称（字符串）
- **symbol**: 品种代码（字符串）
- **description**: 描述信息（字符串）
- **exchange**: 交易所名称（字符串）
- **minmov**: 最小变动价位（整数）
- **minmov2**: 第二最小变动价位（整数）
- **pricescale**: 价格缩放因子（整数）
- **has_intraday**: 是否支持日内数据（布尔值）
- **type**: 品种类型（如 "bitcoin", "stock"）
- **ticker**: 交易代码（字符串）
- **session**: 交易时段（字符串）
- **timezone**: 时区（字符串）
- **intraday_multipliers**: 支持的时间周期列表（字符串数组）

当请求 `/api/search?query=<prefix>` 时，响应为一个包含所有匹配品种的JSON数组，结构同上。

### `/api/symbols` 响应
当请求 `/api/symbols?symbol=<symbol>` 且找到匹配项时，响应为一个包含完整品种信息的JSON对象，其字段与 `/api/search` 响应中的单个品种对象完全相同。

当未找到匹配项时，响应为一个空的JSON对象 `{}`。

**Section sources**
- [chanapi.py](file://api/chanapi.py#L61-L93)

## SUPPORT_SYMBOLS索引机制

`SUPPORT_SYMBOLS` 是一个在 `symbol_info.py` 文件中定义的全局列表，它作为两个API端点的核心数据源和搜索索引，极大地提升了查询效率。

该列表的构建过程如下：
1.  **初始化**: 从 `comm.conf` 模块导入 `ALL_SYMBOLS`（包含加密货币信息）和 `STOCK_DB`（MongoDB数据库连接）。
2.  **加密货币数据加载**: 遍历 `ALL_SYMBOLS`，为每个加密货币创建一个包含详细信息（如 `name`, `symbol`, `minmov`, `pricescale` 等）的字典，并将其添加到 `SUPPORT_SYMBOLS` 列表中。
3.  **股票数据加载**: 从 `STOCK_DB` 的 `stock_names` 集合中查询所有股票信息。遍历查询结果，为每只股票创建一个信息字典（其中 `symbol` 字段使用股票代码，`ticker` 字段也使用股票代码），并将其添加到 `SUPPORT_SYMBOLS` 列表中。

通过将所有品种信息预先加载到内存中的 `SUPPORT_SYMBOLS` 列表，`/api/search` 和 `/api/symbols` 端点避免了每次请求时都进行数据库查询的开销。API的搜索操作直接在内存列表上进行，虽然使用了正则表达式匹配，但由于数据量相对可控，这种设计在响应速度和实现复杂度之间取得了良好的平衡。

**Section sources**
- [symbol_info.py](file://api/symbol_info.py#L4-L73)

## 前端调用场景

前端通过 `ui/src/components/ChanContainer.vue` 文件中的 TradingView 小部件（widget）配置来集成这些API。

在 `ChanContainer.vue` 的 `mounted` 生命周期钩子中，`TradingView` 小部件被初始化。其配置项 `datafeedUrl` 被设置为 `'http://127.0.0.1:8421/api'`，这正是后端API服务的地址。TradingView SDK 会自动调用 `/api/config` 端点来获取配置信息（如支持的分辨率），并调用 `/api/search` 端点来填充搜索框的自动补全功能。

当用户在TradingView界面的搜索框中输入字符时，SDK会向 `/api/search` 发送带有 `query` 参数的请求。后端返回匹配的品种列表，SDK随即在搜索框下方显示下拉建议，实现品种搜索的自动补全。当用户选择一个品种后，图表会加载该品种的数据。

**Section sources**
- [ChanContainer.vue](file://ui/src/components/ChanContainer.vue#L75-L1576)

## 客户端集成示例

### 使用 curl 命令
```bash
# 获取所有支持的交易品种
curl "http://127.0.0.1:8421/api/search?query=all"

# 搜索名称以 "BTC" 开头的品种
curl "http://127.0.0.1:8421/api/search?query=BTC"

# 获取 BTC 品种的详细信息
curl "http://127.0.0.1:8421/api/symbols?symbol=BTC"
```

### 使用 JavaScript fetch
```javascript
// 搜索品种
async function searchSymbols(query) {
    const response = await fetch(`http://127.0.0.1:8421/api/search?query=${query}`);
    const data = await response.json();
    return data;
}

// 获取特定品种信息
async function getSymbolInfo(symbol) {
    const response = await fetch(`http://127.0.0.1:8421/api/symbols?symbol=${symbol}`);
    const data = await response.json();
    return data;
}

// 使用示例
searchSymbols('BTC').then(symbols => console.log(symbols));
getSymbolInfo('BTC').then(info => console.log(info));
```

## 性能考量

尽管 `SUPPORT_SYMBOLS` 索引机制提供了快速的内存内查询，但仍有一些性能考量需要注意：

1.  **响应缓存**: 对于 `/api/search` 端点，特别是当 `query="all"` 时，返回的数据是静态的。可以考虑在前端或使用HTTP缓存头（如 `Cache-Control`）对这些响应进行缓存，以减少服务器负载和提高前端加载速度。

2.  **大数据量下的分页**: 当前的 `/api/search` 端点在返回所有匹配结果时没有分页机制。如果 `SUPPORT_SYMBOLS` 列表变得非常庞大，返回的JSON数据量可能会很大，影响网络传输和前端解析性能。建议在未来的版本中为 `/api/search` 添加分页支持，例如通过 `limit` 和 `offset` 查询参数来控制返回结果的数量和起始位置。

**Section sources**
- [chanapi.py](file://api/chanapi.py#L61-L93)
- [symbol_info.py](file://api/symbol_info.py#L4-L73)