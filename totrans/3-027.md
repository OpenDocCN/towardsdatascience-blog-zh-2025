# Shape-First Tune-Up 为组织提供了一种减少 MongoDB 成本 79% 的方法

> 原文：[`towardsdatascience.com/the-shape%e2%80%91first-tune%e2%80%91up-provides-organizations-with-a-means-to-reduce-mongodb-expenses-by-79/`](https://towardsdatascience.com/the-shape%e2%80%91first-tune%e2%80%91up-provides-organizations-with-a-means-to-reduce-mongodb-expenses-by-79/)

**TL;DR**

<mdspan datatext="el1746054440266" class="mdspan-comment">快速增长的</mdspan> SaaS 在一夜之间从 **M20 → M60** 的静默自动扩展中醒来，云费用一夜之间增加了 20%。在疯狂的 48 小时冲刺中我们：

+   使用 **`$lookup`** 平滑 N + 1 级联，

+   使用投影、**`limit()`** 和 TTL 来驯服无界游标，

+   将 16 MB 的“巨型”文档拆分为精简的元数据 + GridFS 块，

+   重新排列了一些沉睡的索引

同时，$15 284 → $3 210/mo (‑79 %) 的账单被削减，而 p95 延迟从 **1.9 s → 140 ms**。

所有这些都在一个普通的副本集中。

* * *

## 第一步：发票爆炸的那一天

**02:17 a.m.** — 值班电话像弹球机一样亮了起来。Atlas 静悄悄地将我们可靠的 **M20** 热插拔为满载的 M60。Slack 中充满了 🟥 **账单惊吓** 提醒，而 Grafana 的红线图表实时地描绘了一部恐怖电影。

> “财务部门表示，新的支出耗尽了九个月的运营资金。我们需要在站立会议之前找到解决方案。”
> 
> — 首席运营官，02:38

半梦半醒之间，工程师打开了分析器。三个罪魁祸首从屏幕上跳了出来：

+   **查询级联** — 每个订单 API 调用都会触发对其行额外的一次获取。**1 000** 个订单？**1 001** 次往返。

+   **数据洪流游标** — 一个点击流端点在每次页面加载时流式传输 **30 个月** 的事件。

+   **大型文档** — 16 MB 的发票（附带 PDF）将缓存炸得粉碎。

Atlas 试图通过向火中投入硬件来帮助——从 64 GB RAM 升级到 320 GB，提高 IOPS，当然，也提高了账单。

早餐时，战情室的规定已经明确：**在 48 小时内削减 70% 的支出，零停机时间，不破坏模式**。详细的行动开始如下。

* * *

## 第二步：三种 Shape 犯罪及其补救方法

#### 2.1 N + 1 查询海啸

**症状**：对于每个订单，API 都会为它的行项目发起第二个查询。1 000 个订单 ⇒ 1 001 次往返。

```py
// Old (painful)
const orders = await db.orders.find({ userId }).toArray();
for (const o of orders) {
  o.lines = await db.orderLines.find({ orderId: o._id }).toArray();
}
```

**隐藏费用**：1 000 次索引遍历，1 000 次 TLS 握手，1 000 次上下文切换。

**补救措施 (4 行)：**

```py
// New (single pass)
db.orders.aggregate([
 { $match: { userId } },
 { $lookup: {
   from: 'orderLines',
   localField: '_id',
   foreignField: 'orderId',
   as: 'lines'
 } },
 { $project: { lines: 1, total: 1, ts: 1 } }
]);
```

延迟 p95：**2 300 ms → 160 ms**。读操作：**101 → 1 (‑99 %)**。

**2.2 无界查询数据洪流**

**症状**：一个端点在一个游标中流式传输了 30 个月的点击历史。

```py
// Before
const events = db.events.find({ userId }).toArray();
```

**解决方案**：限制窗口并仅投影渲染字段。

```py
const events = db.events.find(
  {
   userId,
   ts: { $gte: new Date(Date.now() - 30*24*3600*1000) }
  },
  { _id: 0, ts: 1, page: 1, ref: 1 }
).sort({ ts: -1 }).limit(1_000);
```

然后让 Mongo 为您修剪：

```py
// 90‑day TTL
db.events.createIndex({ ts: 1 }, { expireAfterSeconds: 90*24*3600 });
```

一家金融科技客户仅使用 TTL 就在一夜之间将他们的存储成本削减了 **72 %**。

**2.3 巨型文档金钱陷阱**

任何超过 **256 KB** 的内容都会使缓存行紧张；一个存储了多 MB 发票（附带 PDF）和 1 200 行历史的集合。

**解决方案**：根据访问模式拆分—发票中的热元数据，S3/GridFS 中的冷 BLOB。

```py
graph TD
  Invoice[(invoices <2 kB)] -->|ref| Hist[history <1 kB * N]
  Invoice -->|ref| Bin[pdf‑store (S3/GridFS)]
```

SSD 支出像雪一样融化；缓存命中率提高了 22 p.p。

* * *

## 步骤 3：显而易见的四个形状罪过

形状不仅仅是关于文档大小——它是查询、索引和访问模式交织的方式。

这四种反模式潜伏在大多数生产集群中，并默默地消耗资金。

**3.1 低基数前导索引键**

**症状** 索引以具有 < 10% 独特值的字段开始，例如 **`{ type: 1, ts: -1 }`**。计划者必须遍历大量数据，然后应用选择部分。

**成本** 高 B 树分支因子，缓存局部性差，额外的磁盘查找。

**修复** 首先将选择键（**userId, orgId, tenantId**）移动：**{ userId: 1, ts: -1 }**。在线重建后，删除旧索引。

**3.2 盲 `$regex` 扫描**

**症状** **`$regex: /foo/i`** 在非索引字段上强制全集合扫描；CPU 激增，缓存翻滚。

每个模式匹配都会遍历每个文档并解码热路径中的 BSON。

修复优先使用锚定模式（**`/^foo/`**）并带有支持索引，或者添加可搜索的缩略字段（**`lower(name)`**）并对其索引。

**3.3 findOneAndUpdate 作为消息队列**

**症状** 工作者使用 **`findOneAndUpdate({ status: 'new' }, { $set: { status: 'taken' } }).`** 进行轮询。

**成本** 文档级锁序列化写入者；吞吐量在几千 ops/s 以下崩溃。

**修复** 使用专用队列（Redis Streams、Kafka、SQS）或 MongoDB 的本地更改流来推送事件，保持写入为追加。

**3.4 偏移分页陷阱**

**症状** **`find().skip(N).limit(20)`** 其中 N 可以达到六位数的偏移量。

**成本** Mongo 仍然会计算并丢弃所有跳过的文档——线性时间。延迟激增，计费统计每个读取。

**修复** 使用复合索引 **`(ts, _id)`** 切换到 **范围游标**：

```py
// page after the last item of previous page
find({ ts: { $lt: lastTs } })
 .sort({ ts: -1, _id: -1 })
 .limit(20);
```

掌握这四个，你将回收 RAM，降低读取单元，并推迟分片数个季度。

* * *

## 步骤 4：成本解剖 101

| 度量 | 前 | 单位 $ | 成本 | 后 | Δ % |
| --- | --- | --- | --- | --- | --- |
| 读取（3 k/s） | 7.8 B | 0.09/M | $702 | 2.3 B | -70 |
| 写入（150/s） | 380 M | 0.225/M | $86 | 380 M | 0 |
| 转移 | 1.5 TB | 0.25/GB | $375 | 300 GB | -80 |
| 存储 | 2 TB | 0.24/GB | $480 | 800 GB | -60 |
| **总计** |  | **$1,643** |  | **-66** |

## 步骤 5：48 小时救援时间表

| 小时 | 行动 | 工具 | 胜利 |
| --- | --- | --- | --- |
| 0-2 | 启用分析器（slowms = 50） | mongo shell | 定位前 10 个慢操作 |
| 2-6 | 用 $lookup 替换 N + 1 | VS Code + 测试 | 读取量减少 90% |
| 6-10 | 添加投影和 limit() | API 层 | RAM 稳定，API 快 4 倍 |
| 10-16 | 分割巨型文档 | 脚本 ETL | 工作集适合在 RAM 中 |
| 16-22 | 删除/重新排序弱索引 | Compass | 磁盘缩小，缓存命中↑ |
| 22-30 | 创建 TTLs/在线存档 | Atlas UI | 存储 -60% |
| 30-36 | 连接 Grafana 面板 | Prometheus | 提前警告实时 |
| 36-48 | 使用 k6 进行负载测试 | k6 + Atlas | 2 倍负载下 p95 < 150 毫秒 |

* * *

## 步骤 6：自我审计清单

+   最大文档除以中位数大于 10？→ 重新设计。

+   任何游标 > 1,000 个文档？→ 分页。

+   每个事件集合的 TTL？（是/否）

+   索引基数 < 10%？→ 删除或重新排序。

+   分析器“慢”操作 > 1%？→ 优化或缓存。

在周五部署之前将此贴到你的显示器上。

* * *

## 第 7 步：为什么形状 > 索引（大多数日子）

添加索引就像为仓库购买一台更快的叉车：它能加快拣选，但如果通道被超大的箱子堵塞，则毫无作用。在 MongoDB 术语中，规划者的成本公式大致为：

```py
workUnits = ixScans + fetches + sorts + returnedDocs
```

索引修剪 **, yet** 和 **“** 在文档膨胀、访问稀疏或分组不佳时仍然可能占主导地位。

**两个查询的故事**

|  | 瘦文档（2 kB） | 超大文档（16 MB） |
| --- | --- | --- |
| ixScans | 1 000 | 1 000 |
| 获取 | 1 000×2 kB = 2 MB | 1 000×16 MB = 16 GB |
| 净时间 | 80 ms | 48 s + 清除风暴 |

相同的索引，相同的查询模式——唯一的区别是**形状**。

**经验法则**

> 首先固定形状，然后一次性索引。
> 
> – 每个重塑的文档都会缩小未来的获取、缓存行和复制数据包。

三个形状的胜利轻易胜过十二个额外的 B 树。

* * *

## 第 8 步：你应该警告的实时指标（PromQL）

```py
# Cache miss ratio (>10 % for 5 m triggers alert)
 (rate(wiredtiger_blockmanager_blocks_read[1m]) /
 (rate(wiredtiger_blockmanager_blocks_read[1m]) +
 rate(wiredtiger_blockmanager_blocks_read_from_cache[1m]))) > 0.10

# Docs scanned vs returned (>100 triggers alert)
 rate(mongodb_ssm_metrics_documents{state="scanned"}[1m]) /
 rate(mongodb_ssm_metrics_documents{state="returned"}[1m]) > 100
```

* * *

## 第 9 步：薄片迁移脚本

需要在不停机的情况下将 1 TB 的 **`events`** 集合拆分为子集合？使用双重写入 + 补充：

```py
// 1) Forward writes
const cs = db.events.watch([], { fullDocument: 'updateLookup' });
cs.on('change', ev => {
 db[`${ev.fullDocument.type}s`].insertOne(ev.fullDocument);
});

// 2) Backfill history
let lastId = ObjectId("000000000000000000000000");
while (true) {
 const batch = db.events
  .find({ _id: { $gt: lastId } })
  .sort({ _id: 1 })
  .limit(10_000)
  .toArray();
 if (!batch.length) break;
 db[batch[0].type + 's'].insertMany(batch);
 lastId = batch[batch.length - 1]._id;
}
```

* * *

## 第 11 步：当实际需要分片时

分片是一种**最后一英里策略，而不是一线解决方案**。它分割数据，增加故障模式，并使每次迁移都变得复杂。首先耗尽垂直升级和基于形状的优化。只有在以下阈值之一在真实负载下**持续**且无法以更低成本解决时，才考虑使用分片键。

**硬容量上限**

| 症状 | 经验法则 | 为什么水平拆分有帮助 |
| --- | --- | --- |
| 工作集在 24 h+ 内位于物理 RAM 的 80% 以上 | < 60% 是健康的；60-80% 可以通过更大的盒子来掩盖；> 80% 的页面会不断出现 | 分割将热分区放在不同的节点上，恢复缓存命中率 |
| 主要写入吞吐量在索引调整后 > 15 000 ops/s | 低于 10 000 ops/s，您通常可以通过批处理或批量更新来生存 | 隔离高速块可以减少日志延迟和锁竞争 |
| 多区域产品需要 < 70 ms p95 读取延迟 | 光速设定 ~80 ms US↔EU 楼层 | 区域分片将数据靠近用户，而不需要求助于边缘缓存 |

**软信号表明分片即将到来**

+   即使使用在线索引，索引构建也会超过维护窗口。

+   压缩时间会侵蚀灾难恢复服务等级协议。

+   单个租户拥有 > 25% 的集群容量。

+   分析器显示来自长事务的 > 500 ms 锁峰值。

**切割前的清单**

+   **重塑文档**：如果最大文档是中位数的 20 倍，则首先进行重构。

+   **启用压缩**（**zstd** 或 **snappy**）通常可以节省 30% 的存储空间。

+   **通过在线归档或分层 S3 存储归档冷数据**。

+   如果 JSON 解析主导 CPU，则用 Go/Rust 重写最热端点。

+   运行**`mongo-perf`**；如果工作负载适合单个副本集后修复，则中止分片计划。

**选择分片键**

+   使用**高基数、单调递增**的字段（**`ObjectId`**，时间戳前缀）。

+   避免低熵键（**`status`**，**`country`**）将写入引导到几个块中。

+   将最常见的查询谓词放在第一位，以避免分散-聚集。

> 分片就像手术；一旦你切了，就得忍受疤痕。确保病人真正需要这个手术。

* * *

## 结论 — 在账单到期前做好准备

当**M60**升级以无声的轰鸣声落地时，这不是硬件的错，这是一个警钟。这不仅仅关乎 CPU、内存或磁盘，这是关于形状。文档的形状。查询的形状。在“只是发货”的冲刺中默默膨胀了几个月的假设的形状。

修复它不需要新的数据库、周末迁移或一队顾问。只需要一个愿意内省、用分析代替恐慌、重塑现有事物的团队。

结果是无可否认的：延迟降低了**92**%，成本减少了近**80**%，代码库现在足够精简，可以呼吸。

但真正的收获是：形状上的技术债务不仅仅是一个性能问题，它还是一个财务问题。而且与索引或缓存技巧不同，正确塑造事物在每次查询运行时、每次数据复制时、每次你扩展时都会带来回报。

所以在你下一个账单周期激增之前，问问自己：

+   每个端点都需要完整的文档吗？

+   我们是在为读取设计，还是只是快速写入？

+   我们的索引是有效工作，还是只是努力工作？

以形状为先并不是一种技术——它是一种心态。一种习惯。而且你越早采用它，你的系统——以及你的发展空间——就会持续更久。

* * *

## 来源与进一步阅读

+   MongoDB 工程博客 — [理解 WiredTiger 缓存](https://www.mongodb.com/docs/manual/core/wiredtiger/)

+   AWS 架构博客 — [优化文档存储成本](https://aws.amazon.com/ru/blogs/architecture/top-architecture-blog-posts-of-2024/)

+   Prisma – [避免 N+1 查询](https://www.youtube.com/watch?v=7oMfBGEdwsc&ab_channel=Prisma)

+   MongoDB 工程博客 — [查询优化技术](https://www.mongodb.com/docs/manual/tutorial/optimize-query-performance-with-indexes-and-projections/)

## 关于作者

**海克·古卡西亚安**是 Hexact 的首席工程师，在那里他帮助构建自动化平台，如 Hexomatic 和 Hexospark。他在大规模系统架构、实时数据库和优化工程方面拥有超过 20 年的经验。
