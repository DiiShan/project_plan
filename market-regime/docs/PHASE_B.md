# 阶段 B 规格：在线状态 vs 事后分段

规划入口：[../PLAN.md](../PLAN.md)。在阶段 A 为 `passed` 或 `degraded` 后自动开始。本阶段建立两套并列对象：当时可识别的在线状态，以及事后复盘分段。二者不得混写进同一个不可区分的标签列而不加前缀。不在本阶段做策略收益排名。最终 holdout 在阶段 C 只封存一次；本阶段不使用该留出段调参或挑选规则。

为避免与 C 冲突，本文件规定：**状态规则的候选集在进入 B 前预注册且数量很小；任何需估计的参数只允许在「参数估计截止日」之前的样本上确定；该截止日不得晚于随后 C 封存的 holdout 起点。** 阶段 A 汇报后即可在配置中写下参数估计截止日的确定规则（例如：全部可用交易日的较早大部分，具体比例写入配置并固定，不因结果改切）。无参且已预注册的规则可以在全样本上生成在线状态，但每个交易日仍只用当时可知数据；这不等于在留出段上选参数。正式规则选择只在 C 的训练期内进行。若 B 必须拟合阈值，则参数估计截止日及之后的样本不参与选择，并在 C 启动时与封存持一致。

## 1. 目标

为每条可用指数生成日频在线状态、事后分段、以及当时可知的特征，并保证可回放、无泄漏。

## 2. 输入与依赖

- A 的 `index_availability`、只读行情接口、交易日历、指纹。
- 仅使用 A 声明可用的字段。无成交量则相关特征 skip。
- 官方起始日与本地首日来自 A 与 [DATA_SOURCES.md](DATA_SOURCES.md)，禁止向前填充缺失年份。

## 3. 任务步骤

1. 按指数分别截取真实可获得区间，周线只由日线按交易日历合成，不另造周线历史。
2. 预注册很小的在线规则候选（例如：趋势用少数均线对与滞回天数；波动用训练期内的历史波动分位；状态机避免单日翻转）。候选名单写入台账，不得在看过未来段后再加规则。
3. 计算在线特征：全部输入的 `asof_date` ≤ 当日收盘可知时间。归一化只在参数估计期内 fit，冻结 `scaler_params`。
4. 生成 `online_state`（上涨/下跌/震荡）与 `vol_bucket`。
5. 事后分段单独生成（例如用事后高低点或全程可知的波段规则），字段与在线状态分开，并标明 `label_kind=expost`。事后标签不得进入在线特征。
6. 导出分段摘要：涨跌幅、持续期、最大回撤、量能（若有）。
7. 运行泄漏与回放测试。通过则 `passed`/`degraded` 并自动进 C。

## 4. 建议 schema

**features_daily**  
主键：`index_code, trade_date, feature_set_version`。字段：`asof_date`、`feat_*`、`scaler_version`、`source_id`、`input_fingerprint`。`asof_date` 必须 ≤ 该行用于决策的时间。

**scaler_params**  
主键：`scaler_version, feature_name`。字段：`fit_start`、`fit_end`、`method`、`param_json`、`fingerprint`。

**regime_daily**  
主键：`index_code, trade_date, model_version`。字段：`online_state`、`vol_bucket`、`rule_id`、`confirm_delay`、`asof_date`、`feature_set_version`、`input_fingerprint`。

**segments_online** 与 **segments_expost**  
主键：`segment_id`。字段：`index_code`、`label_kind`（online/expost）、`state`、`start_date`、`end_date`、`ret`、`max_drawdown`、`n_days`、`volume_summary`、`asof_end`、`model_version`。事后表的 `asof_end` 可以等于段结束日（因为事后可知），但不得回写到在线表。

## 5. 命令接口建议（未实现）

```
python -m market_regime run --phase B
python -m market_regime test --phase B --suite leakage,replay,alignment
```

## 6. 测试清单

- 在线特征与在线状态不得引用 t 日之后的价格或成交。
- 事后标签列若被错误并入在线特征，测试必须失败。
- 参数估计截止日之后的样本未参与阈值或 scaler 的 fit。
- 修改代码后，对已存档的历史在线输出做回归：默认不得改变旧 `model_version` 的字节级或数值容差结果；若算法变更必须新版本号。
- 每个指数的第一行日期 ≥ 本地首日，且没有官方基日之前的自造点。不同指数不因对齐而补假行。
- 单日翻转受到滞回或确认延迟约束（在预注册规则下可测试）。
- 双代码指数只有一条 primary 序列进入状态表。

## 7. 依赖与降级

- 无收盘价或日历：该指数 skip。若全部指数都不可用 → `blocked`（通常 A 已挡住）。
- 无成交量：量能特征 skip，状态规则改用纯价格预注册子集，`degraded`。
- 序列过短（例如中证2000本地起始很晚）：只在短样本上分段并在报告中声明不确定性，不与长历史指数强行对齐。
- 开盘价缺失不影响 B（B 用收盘状态）。

## 8. 结果样式

`output/segments/` 与 `output/regime_daily/` 给出表和简图说明（若作图：线条加粗，靠线型区分，不依赖难辨颜色）。汇报须展示：规则候选清单、参数估计截止日、各指数样本区间、泄漏测试结果。不给出交易表现排名。

## 9. 验收与自动续行

泄漏、回放、对齐测试全部通过 → 进入 C。任一泄漏测试失败 → `failed` 停止。
