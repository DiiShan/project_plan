# 阶段 E 规格：财报、行业与宏观增强

规划入口：[../PLAN.md](../PLAN.md)。在 D 通过后自动开始。本阶段是研究增强，不是回头把 C、D 的已见样本做成更好看的交易成绩。若 C 已否决交易用途，E 可以继续描述性研究，但必须保持 holdout 封存，并使用新的 `research_version`。新增研究不得重用已经看过的最终 holdout。

## 1. 目标

在可知时间完整的前提下，把结构化财报、宏观 vintage、历史行业成分和市场宽度接入研究，评估它们是否对状态描述或规则解释有增量。全文财报只作为可追溯原文，不等于 PIT 表。

## 2. 输入与依赖

- A 盘点结论：财报全文路径、结构化表、宏观库、成分与行业是否存在。
- 冻结的 B/C/D 版本与已封存 `split_plan`。
- 公开口径：公告日与更正披露制度见 [DATA_SOURCES.md](DATA_SOURCES.md)。大模型知识视为可能含未来信息，不得当特征。

## 3. 任务步骤

1. 若存在结构化财报：建立 PIT 表，键包括证券、报告期、**公告时间**、**修订版次**。同一报告期多版本时，在线研究只能使用不晚于 asof 的最新已公告版本。
2. 若仅有全文：建解析数据字典与抽样原文追溯（抽取字段 ← 原文位置）。记录抽取质量。本阶段默认用规则或结构化字段，不启用 MiMo 或大模型全量抽取（权限未确认）。抽样规模保持很小，只为评估能否做 PIT，不在本阶段承诺建成全市场精标语料。
3. 宏观：建立 `period_date` 与 `release_date`（或 vintage）。无发布日的序列打标，禁止进入历史拟合，仅可出现在「当下参考」附录。
4. 行业与宽度：尽量使用**历史成分**和当时行业分类，而不是当前成分回填。处理 ST、退市，避免幸存者偏差。缺历史成分则不做「全市场宽度」结论，或明确写成「存活样本宽度，有偏」。
5. 跨公司、跨行业的稳健性检查：新特征若只在单一行业或单一时期有效，必须写明，不得推广为普遍规律。
6. 如需评估新特征对规则的增量：开 `research_version`（新 feature_set、新 model_version）。训练与选择仍只能用原训练时间墙之前、且此前评估未见过的数据。**禁止**在已看过的最终 holdout 上再选择或微调。若没有真正未见的新区间，则只做描述性相关或机制分析，不做「新策略样本外改善」声明。
7. 交易工具映射（可选）：指数到 ETF 等，此时才启用停牌、涨跌停和分品种税费。仍不含 A 股 ETF 期权。
8. 写审计记录与阶段汇报，结束流水线。

## 4. 建议 schema

**filings_index**  
主键：`security_code, report_period, revision_no`。字段：`announce_datetime`、`doc_type`、`source_path_or_locator`、`has_full_text`、`knowledge_date`、`input_fingerprint`。

**filings_extracted**  
主键：`security_code, report_period, revision_no, field_name, extract_version`。字段：`value`、`quality_flag`、`trace_locator`、`extractor`（rule/manual/llm，默认不得为未批准 llm）、`asof_ok`。

**macro_vintage**  
主键：`series_id, period_date, release_date, vintage_id`。字段：`value`、`knowledge_date`、`usable_in_fit`（无发布日则为 false）。

**industry_membership_hist**  
主键：`security_code, industry_scheme, start_date, end_date`。字段：`industry_code`、`source_id`、`asof_date`。

**universe_status_hist**  
主键：`security_code, start_date, end_date, status_type`（ST/delist/suspend 等）。字段：`source_id`。

**breadth_daily**  
主键：`universe_id, trade_date, research_version`。字段：`n_members`、`advance_decline`、`other_metrics`、`survivorship_bias`（none/likely/unknown）、`asof_date`。

**research_eval**  
主键：`research_version, eval_window_id`。字段：`unseen_data`（必须 true 才可声称增量）、`used_sealed_holdout`（必须 false 用于选择）、`metrics_json`、`claim_allowed`。

## 5. 命令接口建议（未实现）

```
python -m market_regime run --phase E
python -m market_regime test --phase E --suite pit,revision,vintage,survivorship,holdout_isolation
```

## 6. 测试清单

- 无公告日的财报行 `usable_in_fit` 或等价标记为否，且未进入拟合特征。
- 修订版本：asof 之前未公告的修订不得出现在该 asof 的在线特征里。
- 宏观无发布日不得进入拟合。
- 宽度或行业若使用当前成分回填，必须被检测为 `survivorship_bias=likely` 或测试失败（若声称 none）。
- `research_eval.used_sealed_holdout` 在任何选择步骤中为 true → 失败。
- 未把全文或抽取语料写入公开仓库、本规划目录或任何将同步出去的路径。
- 未启用未批准的大模型全量抽取。
- 新研究版本未覆盖旧 `model_version` 的冻结输出。

## 7. 依赖与降级

- 无财报、无宏观、无行业：相应分支 skip，写明缺失，阶段仍可 `degraded` 结束，不因非核心缺失停止。
- 有全文无公告日：只做当下参考和抽样追溯，不拟合。
- 无历史成分：不做无偏宽度结论。
- 无新的未见区间：禁止声称样本外增量。
- 核心仍是不得改源库、不得外发私有全文。

## 8. 结果样式

汇报分三块：① 数据 PIT 能力（有什么、缺什么）；② 描述性发现（带样本与偏差声明）；③ 若且仅若存在真正未见数据上的评估，给出增量结果。每条声称链接 `output/audit`。若 C 已否决交易，不得在 E 用新故事推翻否决，除非新版本在未见数据上按同一预注册协议评估——即便如此，也是新 research_version 的另报，不是改写 C 档案。

## 9. 验收与结束

测试通过则 `passed` 或 `degraded`，结束本次自动流水线。触发停止条件或测试失败则 `blocked`/`failed`。不进入未规划的阶段。
