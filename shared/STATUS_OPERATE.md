# STATUS_OPERATE · A股指数框架

> 执行端(Claude Code / Jaseric)的私有层文档
> 最后更新：2026-05-18
> 配套文档：OPERATE_PLAN.md（详细执行计划）/ DATA_CAPABILITY.md（数据能力）

---

## 1. 当前实施阶段

- **Phase**: Phase 0 — 数据探索 + ATR PoC
- **架构**: OODA（Observe/Orient/Decide/Act），设计端已重构完成
- **存储选型**: DuckDB（热数据）+ Parquet（冷归档）+ JSON（切片状态）
- **当前任务**: 阶段 A 文档更新 → 阶段 B ATR 探索
- **卡点**: 期货数据未采集（影响风格 spread / 拔河残差）

---

## 2. 探索路径

详见 `OPERATE_PLAN.md` §四。

| Path | 切片 | 数据可行性 | 状态 |
|------|------|-----------|------|
| Path 0 | ATR + CUSUM | ✅ 指数日线已有 | 待探索 |
| Path 1a | HHI 缩圈 | ✅ 权重+日线已有 | 未开始 |
| Path 1b | 风格 spread | ⚠️ 需期货数据（可用合成基底近似） | 未开始 |
| Path 1c | 承接失三件套 | ✅ 全部已有 | 未开始 |
| Path 2 | 拔河残差/ABCD/三段式 | ⚠️ 部分需建 | 未开始 |

---

## 3. 已实装代码能力

| 脚本 | 功能 | 状态 |
|------|------|------|
| `stock_collector.py` | 个股数据采集（权重/日线/板块） | ✅ |
| `index_minutes_backfill.py` | 8 指数分钟线回填 | ✅ |
| `stock_data_backfill.py` | 1352 股批量回填 | ✅ |
| `synthetic_base_index.py` | 合成基底指数 | ✅ |
| `market_snapshot.py` | 期货/外盘快照 | ✅ |
| `etf_update_charts.py` | ETF 份额流图表 | ✅ |

---

## 4. 数据资产现状

详见 `DATA_CAPABILITY.md` §一。

| 数据类型 | 规模 | 日期范围 | 完整性 |
|---------|------|---------|--------|
| 指数分钟线（8 指数） | 3,487 CSV | 2024-01 ~ 2026-05 | ⚠️ 中证2000 仅 33 天 |
| 个股日线（1,352 股） | 1,352 CSV | 2024-01 ~ 2026-05 | ✅ |
| 个股分钟线（752 股） | 289,924 CSV（98.7% 有 389 天） | 2024-01 ~ 2026-05 | ✅ |
| 指数权重（7 指数） | 8 CSV | 月度快照 | ✅ |
| 合成基底 | 1,469 文件 | 2024-01 ~ 2026-05 | ✅ |

**总计**: ~298,000 CSV 文件

---

## 5. 与设计端的同步点

- **上次接收**: 2026-05-17 — STATUS_DESIGN.md（OODA 架构重构 + Orient 切片 + Phase 0-3 路线图）
- **上次反馈**: 2026-05-18 — OPERATE_PLAN.md + DATA_CAPABILITY.md（数据能力矩阵 + 存储选型 + 探索路径）
- **待对齐**: ATR 窗口/存放态/Phase 1 优先级 → 待 99 决策
- **需要设计端提供**: 各 Orient 切片的详细判断规则（阈值方向），Phase 0 验收后的优先级调整

---

## 6. 关键决策记录

| 日期 | 决策 | 理由 |
|------|------|------|
| 2026-05-18 | 存储选型 DuckDB + Parquet + JSON | 列式压缩、SQL window function、ACID、渐进迁移 |
| 2026-05-18 | 创建 OPERATE_PLAN.md / DATA_CAPABILITY.md / EXPLORE_LOG.md | 按 OODA 分离文档职责 |
| 2026-05-17 | 术语对齐 online/offline → design/operate | 配合 design-operator-bridge skill 升级 |
