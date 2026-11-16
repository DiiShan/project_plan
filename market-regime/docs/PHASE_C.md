# 阶段 C 规格：样本外策略验证

规划入口：[../PLAN.md](../PLAN.md)。在 B 通过后自动开始。本阶段回答：预注册的简单规则，在严格样本外是否优于预先声明的基准。若否，必须否决交易用途，但流水线继续 D、E 的描述性研究。最终 holdout 只封存一次。

## 1. 目标

封存最终 holdout，做内层调参与外层 walkforward，比较成本后收益、回撤与暴露，并给出明确的交易用途结论。

## 2. 输入与依赖

- 冻结的 `regime_daily`、`features_daily`、`scaler_params`、`model_version`。
- A 的 `exec_mode`：`next_open` 或 `close_only`。
- 交易日历。若做工具层映射，才需要停牌、涨跌停和品种费率；缺这些不阻塞指数信号研究。

## 3. 任务步骤

1. **只封存一次**最终留出区间（时间切分规则预先写入配置，例如最近一段固定长度或固定截止日之后全部留出）。写入 `split_plan` 后不得因结果修改，也不得因效果不好而重切。持有 `holdout_sealed=true`。
2. 在训练期内，从**固定小候选集**中选择规则参数（均线过滤、突破、回调、超跌反弹；状态过滤若使用，也只能用 B 的冻结在线状态）。内层验证不得扫到 holdout，也不得扫到外层测试折。
3. 外层按时间滚动 walkforward。标签重叠使用 purge/embargo。
4. 默认执行：信号日收盘后决策，下一交易日开盘成交。若 `exec_mode=close_only`，整个阶段只报告收盘口径（例如次日收盘或收盘对收盘），标题和字段不得出现「次日开盘回测」。
5. 收益分层：`signal_index_return`（指数点位）与 `tradable_return`（仅当存在工具映射时）。指数层**不扣股票印花税**。若尚无 ETF/股票映射，`tradable_return` 留空并声明「仅指数信号，不可当作可交易收益」。
6. 成本敏感性：对将要映射的工具，按时期、品种放三档摩擦（零、常规、偏保守）。费率初值来自公开制度与常规佣金假设，写入 `config` 并在台账列出。指数层可用很轻的点差敏感性，但不得冒充税费。
7. 基准：现金、同一指数买持、同风险暴露的简单均线。A 股只做多。
8. 预先声明指标：成本后收益、最大回撤、暴露（持有时间或平均仓位）、交易次数、相对基准的差异。不把未预先声明的指标选出来当主结论。
9. 若外层与 holdout 上相对基准无稳定改善：`trading_use=vetoed`，说明理由。不得改执行口径、扣费方式或切分来「做出」改善。然后自动进入 D。
10. 若有改善：`trading_use=provisional`，仍须强调样本有限、不是收益承诺，再进入 D。

## 4. 建议 schema

**split_plan**  
主键：`split_id`。字段：`train_start`、`train_end`、`wf_fold_id`、`wf_test_start`、`wf_test_end`、`holdout_start`、`holdout_end`、`sealed_at`、`rule_text`、`fingerprint`。

**experiment_ledger**  
主键：`experiment_id`。字段：`rule_family`、`param_json`、`candidate_set_id`、`fit_window`、`eval_window`、`metrics_json`、`selected`、`selection_stage`（inner/outer，禁止 holdout）、`notes`。

**strategy_trades**  
主键：`trade_id`。字段：`index_code`、`signal_date`、`exec_date`、`exec_mode`、`exec_price_type`（open/close/none）、`side`、`signal_index_return`、`tradable_return`、`cost_bucket`、`instrument_type`、`fee_schedule_version`、`asof_signal`、`model_version`、`split_id`。

**performance** 与 **walkforward**  
主键：`eval_id`（含 `index_code, rule_id, split_id, cost_bucket, layer`）。字段：`ret`、`drawdown`、`exposure`、`n_trades`、`vs_buyhold`、`vs_cash`、`vs_equal_exposure_ma`、`trading_use`、`input_fingerprint`。`layer` 为 `index_signal` 或 `tradable`。

**fee_schedule**  
主键：`fee_schedule_version, instrument_type, start_date, end_date, item`。字段：`rate`、`source_ref`、`applies_to`。股票印花税行不得应用到 `instrument_type=index`。

## 5. 命令接口建议（未实现）

```
python -m market_regime run --phase C
python -m market_regime test --phase C --suite leakage,holdout,fees,exec_mode
```

## 6. 测试清单

- holdout 在封存后没有任何参数选择或规则新增使用它。
- 外层测试折的信息未进入内层选择。
- `exec_mode=close_only` 时，输出中不存在声称 next_open 的成交。
- 缺 open 却使用 open 成交价 → 必须失败。
- 指数层 performance 未扣除股票印花税；若错误套用，测试失败。
- 交易工具层若启用，涨跌停或停牌日不得使用不可成交价。
- purge/embargo 后无非法重叠标签。
- 台账中的候选数不超过预注册上限。
- 无改善时 `trading_use` 必为否决，且指标未被改口径重算为改善。

## 7. 依赖与降级

- 无开盘价：收盘口径继续，不停止。
- 无工具行情或费率史：只做指数信号层，`tradable_return` 空，`degraded`。
- 无停牌/涨跌停：指数层继续；工具层若无法执行约束则不做工具层，而不是假装可成交。
- 样本太短难以 walkforward：减少折数但保留一次 holdout；若连 holdout 与一段训练都无法切开，则 `blocked`（无法做诚实的样本外）。
- 无改善：交易否决，阶段仍可 `passed`（研究目标已完成）并续行。

## 8. 结果样式

汇报必须把指数信号与可交易层分开制表；三档成本并排；holdout 与 walkforward 分开；明确写出 `trading_use`。禁止只展示最好的一条未预注册曲线。

## 9. 验收与自动续行

隔离与口径测试通过后，无论交易是否否决，自动进入 D。测试失败则 `failed` 停止。
