# 阶段 A 规格：本地盘点

规划入口：[../PLAN.md](../PLAN.md)。本文件供本机 Codex 连续执行时使用。进入本阶段不需要额外人工批准。只做只读盘点、质量检查、接口与数据字典草案、manifest 与指纹；不识别状态、不回测、不补库、不改源库、不写项目目录以外的路径。本次提交不运行本阶段。

## 1. 目标

摸清本地到底有什么，能支持哪种执行口径，并把结果写成可复核的目录、质量和缺口文件。用户已提示个股财报和数据本地较全，因此**先盘点、后补数**。旧盘点只作线索。

## 2. 输入与依赖

- `E:\金融数据\data.list`（路径来自既有记录，`confirmed_by_user`；内容覆盖待本机只读核实）。
- 文档中已列路径：`E:\金融数据\plf_data\database\plf.duckdb`，`E:\金融数据\macro_data\database\macro.duckdb`。`datasink_caibao` 与 QMT 本地库路径待从索引和本机配置发现。
- [DATA_SOURCES.md](DATA_SOURCES.md) 中的官方指数代码与口径。
- 依赖：只读文件系统、DuckDB 只读打开能力、如存在则只读调用 QMT `get_local_data` / `get_trading_dates` 一类本地接口。禁止调用文档中的 `download_*` 或任何外网批量取数来填源库。

## 3. 任务步骤

1. 读取 `data.list`，展开全部库、文件和说明；记录索引本身的修改时间与指纹。
2. 对每个条目检查：路径是否存在、格式或引擎、表/文件清单、关键字段、规模、最小和最大日期、更新时间、重复、断档、明显空值。打上 `discovered_this_run`。与 `historical_record_unverified` 线索比对时，只记录异同，不把旧结论当现状。
3. 定位目标指数九条主序列（见数据源文档），建立 `code_map`（官方代码、本地代码、市场后缀、是否双代码、选用的主序列）。确认本地首尾日，并与官方基日、发布日并列，不修改原始行情。
4. 检查每条指数是否有 open/high/low/close/volume/amount。无 open 则标记：后续不得称为次日开盘回测。
5. 检查交易日历：市场、日期、是否开市、来源。若只能从行情日期反推，须声明限制（无法预知未来休市或补齐历史缺日原因）。
6. 财报：区分全文与结构化表；查公告日、报告期、修订版次、原文路径。抽样若干条看能否从抽取结果回到原文，不在本阶段用模型做全量抽取。
7. 宏观：查报告期、发布日或 vintage。无发布日则标记「禁止用于历史拟合」。
8. 历史成分、ST/退市、停牌、涨跌停、行业分类、资金面：有则记录字段与覆盖，无则列入缺口并说明不影响指数信号 MVP。
9. 起草只读适配器说明与本地项目根下的 `docs/DATA_DICTIONARY.md`（只写已看见的字段，不编造未见面的列；本次不预填）。
10. 写 manifest、质量报告、缺口分级、阶段汇报，更新 `run_state`。核心满足则 `passed` 或 `degraded` 并自动进入 B。

## 4. 建议 schema（规划，不是已建表）

公共列建议在所有派生表中出现：`schema_version`、`source_id`、`input_fingerprint`、`run_id`、`asof_date`（本表可知时间）、`row_status`（`discovered_this_run` / `missing` 等）。

**catalog_sources**  
主键：`source_id`。字段：`path`、`path_status`（四类状态标签）、`engine`、`readonly_ok`、`index_mtime`、`file_hash_or_equiv`、`notes`。

**catalog_tables**  
主键：`source_id, table_or_file`。字段：`n_rows`、`n_cols`、`min_date`、`max_date`、`date_field`、`update_time`、`issues`。

**catalog_fields**  
主键：`source_id, table_or_file, field_name`。字段：`dtype`、`nullable`、`role`（code/date/open/high/low/close/volume/amount/announce_time/report_period/revision/release_date/other）、`example_ok`（不得写入真实敏感文本全文）。

**index_availability**  
主键：`official_code, local_code`。字段：`index_name`、`official_launch_date`、`official_base_date`、`local_first_date`、`local_last_date`、`has_open`、`has_ohlc`、`has_volume`、`calendar_aligned`、`series_role`（primary/duplicate）、`history_label`（live_after_launch / official_backcalc / unknown）、`exec_mode`（next_open / close_only）。

**missing_data**  
主键：`item_id`。字段：`gap_class`（P0/P1/P2）、`blocks_phase`、`can_degrade`、`degrade_plan`、`status`。

**quality_issues**  
主键：`issue_id`。字段：`object`、`severity`、`description`、`evidence_ref`。

## 5. 命令接口建议（未实现）

```
python -m market_regime run --phase A
python -m market_regime inventory --index "E:\金融数据\data.list" --out output/manifest
```

写路径仅限项目根。打开 DuckDB 必须只读。

## 6. 测试清单

- 源库打开前后的路径、大小或修改时间不变（只读承诺）。
- 派生输出不越出项目根。
- 九个目标指数：有则记录完整首尾日，无则进入缺口，不编造行。
- 官方代码映射无 000932 冒充中证2000、无 588000 冒充科创50、无 000001.SZ 冒充上证指数。
- `has_open=false` 的序列 `exec_mode` 不得为 `next_open`。
- manifest 指纹可重复计算并一致。
- 未调用外网批量下载，未写源库。

## 7. 依赖与降级

- P0：至少若干（须覆盖用户指定的主指数集合中「本地实际找得到」的部分）具有日频收盘价，且有可用交易日历或可安全声明的行情日期日历；只读接口能按代码+日期取数。若**全部**目标指数收盘价都不可用 → `blocked`。
- 个别指数缺失：skip 该指数，其余继续，状态 `degraded`。
- 无开盘价：`degraded`，C 只做收盘口径。
- 无财报/宏观/成分：不阻塞 A 和 B，记 P1/P2。
- 日历只能从行情反推：`degraded`，在质量报告写明。

## 8. 结果样式

阶段汇报（Markdown，放 `output/manifest/phase_A_report.md`）应包含：库清单、指数可用表、执行口径、财报与宏观可知时间结论、缺口分级、将自动进入 B 的声明。不要向用户索要本可本地查到的路径，也不预填未看见的字段。

## 9. 验收与自动续行

满足测试且未触发停止条件 → `passed` 或 `degraded`，自动进入阶段 B。否则 `blocked` 或 `failed` 并停止。
