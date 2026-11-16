# 阶段 D 规格：当下相似分布

规划入口：[../PLAN.md](../PLAN.md)。在 C 通过隔离测试后自动开始，**不依赖交易用途是否被否决**。本阶段只提供描述性对照：与「当时可知、且前瞻窗口已走完」的历史片段相比，后续收益与不利波动的分布是什么。不把 C 的否决改写为推荐，也不输出单一目标价。样本外无改善时，交易用途保持否决，本阶段仍可继续描述。

## 1. 目标

在冻结的在线状态与特征定义下，检索预测日起点之前的相似样本，报告 5/20/60 个交易日收益和最大不利波动（MAE）等分布，以及有效样本数、非重叠事件数、独立周期数与不确定性。

## 2. 输入与依赖

- B 冻结的 `online_state`、`features_daily`、`model_version`、`scaler_params`。
- 交易日历与各指数真实样本区间。
- C 的 `trading_use` 结论：仅作为上下文展示，不在本阶段重新优化规则。
- 不得使用事后分段标签来挑选「更漂亮」的对照样本。

## 3. 任务步骤

1. 定义相似度：只用预测日 `asof` 及之前的在线特征或状态路径（窗口长度预注册，不得按结果改）。归一化沿用已冻结的训练期参数，不得用含预测日之后的信息重估。
2. 候选事件的起点必须早于当前预测日，且其 5/20/60 日标签在**历史中已经成熟**（预测起点 + 60 个交易日 ≤ 该指数本地最后可用于研究的日期，且该段在形成对照库时已完全可知）。对「当下」最新一天，其未来 60 日尚未走完，故**不得**为当下这一点填前瞻实现值；只能展示它与历史样本的相似位置，前瞻分布来自已走完的历史对照。
3. 处理重叠：同一指数上时间重叠的事件须剔除或只保留预注册规则下的代表事件，避免一次行情被算成多个样本。
4. 统计每组对照的分位数、均值（若报告则同时报告样本量与离散度）、MAE 或最大不利波动、有效样本数 `n_eff`、非重叠事件数、粗略独立周期数。独立周期不足时，全文改为描述，不给交易含义。
5. 写确认条件与失效条件（例如在线状态改变、突破预注册阈值）。
6. 若 C 已否决交易用途，卡片顶部必须重申：以下分布不是可交易优势的证明。

## 4. 建议 schema

**analogue_queries**  
主键：`query_id`。字段：`index_code`、`query_date`、`asof_date`、`feature_set_version`、`model_version`、`window_spec`、`input_fingerprint`。

**analogue_events**  
主键：`query_id, event_id`。字段：`event_start`、`event_asof_end`、`horizon_end_60`、`mature_5/20/60`（布尔）、`overlap_group`、`kept`、`distance`、`ret_5`、`ret_20`、`ret_60`、`mae_5`、`mae_20`、`mae_60`、`knowledge_cutoff`。约束：`knowledge_cutoff` ≤ `query_date`，且 `horizon_end_60` 落在历史已实现区间（仅对计入分布的事件）。

**analogue_distributions**  
主键：`query_id, horizon, metric`。字段：`n_events`、`n_nonoverlap`、`n_independent_cycles`、`q10`、`q25`、`q50`、`q75`、`q90`、`descriptive_only`、`notes`。

**current_state**  
主键：`index_code, asof_date, model_version`。字段：`online_state`、`vol_bucket`、`nearest_query_id`、`confirm_rule`、`invalidate_rule`、`trading_use_context`、`forward_realized`（对当下必须为 false 或空）。

## 5. 命令接口建议（未实现）

```
python -m market_regime run --phase D
python -m market_regime test --phase D --suite pit,overlap,maturity
```

## 6. 测试清单

- 检索特征不含 query_date 之后的数据。
- 计入 60 日分布的事件，其第 60 个交易日已存在于行情且不超过本地研究截止日。
- 当下时点的 `forward_realized` 不得被填入尚未发生的收益。
- 重叠事件未在 `kept=true` 时重复计数。
- 未使用 `label_kind=expost` 作为相似度输入。
- 独立周期或非重叠样本低于预注册门槛时，`descriptive_only=true`。
- 不输出单一目标价字段，或该字段必须为空。

## 7. 依赖与降级

- 某指数历史太短，成熟 60 日样本极少：只做 5/20 或纯描述，`degraded`，不停止。
- 相似度特征缺失：退回只用在线状态匹配，并声明变粗。
- C 否决交易：必须在输出中展示，不停止。

## 8. 结果样式

以分布图或分位数表为主，标注 n 与「仅描述」标记。确认/失效条件用简短条文。不得用一条「最像」的后来走势代替分布。

## 9. 验收与自动续行

PIT、成熟窗口与重叠测试通过后进入 E。失败则停止。
