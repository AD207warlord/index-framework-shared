# STATUS_OPERATE · A股指数框架

> 执行端(Claude Code / Jaseric)的私有层文档
> 最后更新：2026-05-18
> 配套文档：OPERATE_PLAN.md（详细执行计划）/ DATA_CAPABILITY.md（数据能力）

---

## 1. 当前实施阶段

- **Phase**: Phase 0 — 数据探索 + ATR PoC
- **当前任务**: 阶段 A 文档更新完成，待进入阶段 B ATR 探索
- **卡点**: 无阻塞项。ATR 参数已授权本地自选起点（建议 ATR3/ATR14/100天回看）

---

## 2. 已实装能力

| 脚本 | 功能 | 输入 | 输出 | 状态 |
|------|------|------|------|------|
| `stock_collector.py` | 个股数据采集（权重/日线/板块） | tushare API | `notes/data/stock/` | ✅ |
| `index_minutes_backfill.py` | 8 指数分钟线回填 | tushare API | `notes/data/index/minutes/` | ✅ |
| `stock_data_backfill.py` | 1352 股批量回填 | tushare API | `notes/data/stock/` | ✅ |
| `synthetic_base_index.py` | 合成基底指数（上证50-6科创/沪深300-创头） | 权重+日线 | `notes/data/synthetic/` | ✅ |
| `market_snapshot.py` | 期货/外盘快照 | tushare API | JSON+MD | ✅ |
| `etf_update_charts.py` | ETF 份额流+价格叠加图 | tushare API | PNG 图表 | ✅ |
| `index_collector.py` | 领先线采集（东财 trends2） | EastMoney API | `daily_summary.csv` | ❌ 已退役 |

---

## 3. 资产现状

详见 `DATA_CAPABILITY.md` §一。摘要：

| 资产类型 | 数量 | 范围 | 格式 | 完整性 |
|---------|------|------|------|--------|
| 指数分钟线（8 指数） | 3,487 CSV | 2024-01 ~ 2026-05 | OHLCV+lead | ⚠️ 中证2000 仅 33 天 |
| 个股日线（1,352 股） | 1,352 CSV | 2024-01 ~ 2026-05 | OHLCV+amount | ✅ |
| 个股 daily_basic | 1,352 CSV | 2024-01 ~ 2026-05 | PE/PB/换手/市值 | ✅ |
| 个股分钟线（752 股） | 289,924 CSV | 2024-01 ~ 2026-05 | OHLCV | ✅ 98.7% 有 389 天 |
| 指数权重（7 指数） | 8 CSV | 月度快照 | 权重+行业 | ✅ |
| 合成基底 | 1,469 文件 | 2024-01 ~ 2026-05 | base vs actual | ✅ |

**总计**: ~298,000 CSV 文件

---

## 4. 待开发清单

### P0（Phase 0 — 当前必做）
- [ ] 验证 DuckDB 直接读取现有 CSV 的查询性能
- [ ] ATR 计算（SQL window function，短/长窗口 + ratio + 分位值）
- [ ] CUSUM 检测在 5/12 数据上的结果
- [ ] 流式更新机制验证（读 JSON 状态 → 算新值 → 检测 → 写回）
- [ ] 记录探索结论到 EXPLORE_LOG.md

### P1（Phase 1 — Orient 切片探索）
- [ ] HHI 缩圈探索（权重月度 → 日度近似 → HHI 时间序列）
- [ ] 承接失三件套探索（合成基底 + 创业板 + 广度）
- [ ] 风格 spread 探索（先用合成基底近似）

### P2（Phase 2 — 综合切片）
- [ ] 拔河模型残差（需沪深300日线）
- [ ] 贡献度集中度（权重+涨跌已有）
- [ ] ABCD 分类器（多信号综合）
- [ ] 三段式形态（依赖 Phase 1 输出）

### P3（远期 — 后增切片 + 需设计端确认）
- [ ] 期货数据采集（`fut_daily`）— 升贴水 / 基差切片
- [ ] 期权数据采集（`opt_daily`）— 波动率曲面 / 折溢价切片
- [ ] Phase 3 切片（双向四象限 / 三反馈分离）— 需设计端定义

---

## 5. 实操遇到的问题

1. **中证2000 分钟数据只有 33 天** — `idx_mins` 对 932000.CSI 返回空，用 ETF 代理(563300.SH)采集，覆盖不足。替代：日线做 HHI 分析足够，分钟线暂不覆盖中证2000。
2. **R²=0.96 只有 16 天样本** — 沪深300拔河回归不具统计显著性，需 300+ 日重验证。已在 STATUS_DESIGN 标注。
3. **等权数据无直接 API** — tushare 无等权指数接口，需手动从成分股构建。当前暂缓，优先 ATR PoC。
4. **个股分钟线覆盖** — 1,352 股中 752 股有分钟线，头部覆盖已足够做 HHI/贡献度分析。

---

## 6. 与设计端的同步点

- **上次接收**: 2026-05-18 — STATUS_DESIGN.md v2（术语锁定 + Block 4/5 拆分 + P3 切片新增 + ATR 参数授权本地 + Phase 0 存储建议 pandas+JSON）
- **上次反馈**: 2026-05-18 — OPERATE_PLAN.md + DATA_CAPABILITY.md + EXPLORE_LOG.md（数据能力矩阵 + 存储选型 DuckDB + 探索路径 + 停止项）
- **当前差距**: 设计端有更多认知文档（见顶预警模块_v0.1 / OODA日常工具卡_v1）在 STATUS_DESIGN 附录提及但执行端未见过
- **需要设计端提供**: 各 Orient 切片的详细判断规则（阈值方向）；Phase 0 验收后的优先级调整

---

## 7. 操作参考

| 场景 | 参考文档 | 说明 |
|------|---------|------|
| 每日开盘前 | `开盘简报查询流程_v1.md` | 7 步查询流程（待 ATR 验证后更新） |
| 每日收盘后 | `盘后速查卡_v1.md` | 5 步速查清单（待 OODA 重构） |
| 异常观察 | `框架观察日志.md` | 持续记录 Observe 层异常 |

> 这些操作流程基于旧 Block 架构编写。Phase 0 ATR 验证后，将按 OODA + Orient 切片重新设计。

---

## 8. 关键决策记录

| 日期 | 决策 | 理由 |
|------|------|------|
| 2026-05-18 | 术语统一：IF/IH/IC/IM → 中文指数名 | 99 口语简称指指数非期指，避免歧义 |
| 2026-05-18 | 期货/期权归 P3 后增切片 | 升贴水/基差/IV曲面是后期视角，不阻塞 Phase 0-2 |
| 2026-05-18 | 风格 spread / 拔河残差用指数数据 | 不依赖期货，指数日线已有可直接计算 |
| 2026-05-18 | ATR 参数本地自选起点 | 设计端授权：建议 ATR3/ATR14/100天，跑完看效果再定 |
| 2026-05-18 | Phase 0 存储用 pandas+JSON | 设计端建议：算法验证先于存储层，DuckDB 后置到 Phase 1+ |
| 2026-05-18 | 存储选型 DuckDB + Parquet + JSON（整体方向） | 列式压缩、SQL window function、ACID、渐进迁移 |
| 2026-05-18 | 创建 OPERATE_PLAN.md / DATA_CAPABILITY.md / EXPLORE_LOG.md | 按 OODA 分离文档职责 |
| 2026-05-18 | 6 个文件归档至 archive/ | STATUS.md 与 WORKFLOW 重叠 + 5 个已完成历史产物 |
| 2026-05-17 | 术语对齐 online/offline → design/operate | 配合 design-operator-bridge skill 升级 |
