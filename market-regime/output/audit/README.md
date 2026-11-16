# 研究审计

运行：`20261117T014316+0800-a687aacf07`

配置：`a687aacf071e2b7d867168825731217c647e2520061afa54ec1025d4f69b7373`

代码：`d5741f5b32c21ddef52de8120f7e582c67632ba275e8c89b873920acdefca4a6`

| phase | status | completed_at |
| --- | --- | --- |
| A | degraded | 2026-11-17T01:44:08+08:00 |
| B | passed | 2026-11-17T01:44:40+08:00 |
| C | degraded | 2026-11-17T01:44:40+08:00 |
| D | degraded | 2026-11-17T01:44:41+08:00 |
| E | degraded | 2026-11-17T01:45:19+08:00 |

| phase | artifact | source_tables | asof_date |
| --- | --- | --- | --- |
| A | canonical_bars | data.list / 本机只读文件 | 2026-11-13 00:00:00 |
| A | canonical_calendar | data.list / 本机只读文件 | 2026-11-13 00:00:00 |
| A | catalog_fields | data.list / 本机只读文件 | 2026-11-13 00:00:00 |
| A | catalog_sources | data.list / 本机只读文件 | 2026-11-13 00:00:00 |
| A | catalog_tables | data.list / 本机只读文件 | 2026-11-13 00:00:00 |
| A | index_availability | data.list / 本机只读文件 | 2026-11-13 00:00:00 |
| A | missing_data | data.list / 本机只读文件 | 2026-11-13 00:00:00 |
| A | quality_issues | data.list / 本机只读文件 | 2026-11-13 00:00:00 |
| A | source_issues | data.list / 本机只读文件 | 2026-11-13 00:00:00 |
| A | source_locators | data.list / 本机只读文件 | 2026-11-13 00:00:00 |
| A | source_snapshot | data.list / 本机只读文件 | 2026-11-13 00:00:00 |
| A | table_quality | data.list / 本机只读文件 | 2026-11-13 00:00:00 |
| B | features_daily | canonical_bars / canonical_calendar | 2026-11-13 00:00:00 |
| B | regime_daily | canonical_bars / canonical_calendar | 2026-11-13 00:00:00 |
| B | research_schedule | canonical_bars / canonical_calendar | 2026-11-13 00:00:00 |
| B | scaler_params | canonical_bars / canonical_calendar | 2026-11-13 00:00:00 |
| B | segments_expost | canonical_bars / canonical_calendar | 2026-11-13 00:00:00 |
| B | segments_online | canonical_bars / canonical_calendar | 2026-11-13 00:00:00 |
| B | weekly_bars | canonical_bars / canonical_calendar | 2026-11-13 00:00:00 |
| C | experiment_ledger | canonical_bars / research_schedule / 冻结候选 | 2026-11-13 00:00:00 |
| C | fee_schedule | canonical_bars / research_schedule / 冻结候选 | 2026-11-13 00:00:00 |
| C | performance | canonical_bars / research_schedule / 冻结候选 | 2026-11-13 00:00:00 |
| C | split_plan | canonical_bars / research_schedule / 冻结候选 | 2026-11-13 00:00:00 |
| C | strategy_trades | canonical_bars / research_schedule / 冻结候选 | 2026-11-13 00:00:00 |
| C | trading_use | canonical_bars / research_schedule / 冻结候选 | 2026-11-13 00:00:00 |
| C | walkforward | canonical_bars / research_schedule / 冻结候选 | 2026-11-13 00:00:00 |
| D | analogue_distributions | canonical_bars / regime_daily / scaler_params / trading_use | 2026-11-13 00:00:00 |
| D | analogue_events | canonical_bars / regime_daily / scaler_params / trading_use | 2026-11-13 00:00:00 |
| D | analogue_queries | canonical_bars / regime_daily / scaler_params / trading_use | 2026-11-13 00:00:00 |
| D | current_state | canonical_bars / regime_daily / scaler_params / trading_use | 2026-11-13 00:00:00 |
| E | breadth_daily | QMT daily_bar and current_stock_list | 2026-11-13 00:00:00 |
| E | breadth_description | breadth_daily and regime_daily | 2026-11-13 00:00:00 |
| E | filing_trace_samples | DataSinking documents | 2026-11-13 00:00:00 |
| E | filings_extracted | QMT financial_pit | 2026-11-13 00:00:00 |
| E | filings_index | QMT financial_pit | 2026-11-13 00:00:00 |
| E | financial_industry_description | filings_index, filings_extracted and sector_membership | 2026-11-13 00:00:00 |
| E | index_membership_coverage | QMT index_membership_history_weekly | 2026-11-13 00:00:00 |
| E | industry_membership_hist | QMT sector_membership | 2026-11-13 00:00:00 |
| E | macro_reference | macro_versions (latest anchor reference) | 2026-11-13 00:00:00 |
| E | macro_version_coverage | macro_versions | 2026-11-13 00:00:00 |
| E | macro_vintage | macro_versions (validated projection of macro_all_versions) | 2026-11-13 00:00:00 |
| E | research_eval | stage E research only; no C holdout reads | 2026-11-13 00:00:00 |
| E | universe_status_hist | QMT security_master | 2026-11-13 00:00:00 |
| E | universe_stock_coverage | QMT delisted_stock_list and daily_bar | 2026-11-13 00:00:00 |
