# Codex 连续执行简报（阶段 A–E）

规划入口：[../PLAN.md](../PLAN.md)。本 brief 给本机 Codex 一次性执行用，不是只跑阶段 A。**本次仅交付并提交规划文档，不运行 Codex/研究程序、不访问本地数据库；用户随后在本机执行。** 当用户在本机另行启动执行时，按本 brief 和 [规划入口](../PLAN.md)、[PHASE_A.md](PHASE_A.md)、[PHASE_B.md](PHASE_B.md)、[PHASE_C.md](PHASE_C.md)、[PHASE_D.md](PHASE_D.md)、[PHASE_E.md](PHASE_E.md)、[DATA_SOURCES.md](DATA_SOURCES.md) 连续工作。这些文件在本地项目根下的路径是 `PLAN.md`、`docs/PHASE_A.md` 至 `docs/PHASE_E.md`、`docs/DATA_SOURCES.md`。阶段验收为 `passed` 或 `degraded` 后自动进入下一阶段，**不要停下来等待每阶段人工批准**。

## 1. 项目与权限

- 项目根：`D:\project\grok\market-regime`。所有派生库、输出、日志只写这里。
- 源只读。总索引 `E:\金融数据\data.list`。四条已公开的本地路径见 [DATA_SOURCES.md](DATA_SOURCES.md)，来自既有记录，内容覆盖待本机只读核实。QMT 库路径和 `datasink_caibao` 准确路径必须本地发现，不得编造。
- 先用本地已有数据和财报。复用现库，不重新下载，不重建源库。公开网页只可查口径，不可批量补库，不可上传私有数据。
- 不要改源库，不要写到项目外，不要产生额外费用或新账号，不要把数据库、财报全文、日志、机器调试材料、凭证、账户仓位或个人资金规模推到 GitHub 或任何第三方。规划 Markdown 的公开范围只限已提交的 `market-regime/` 文稿；本机执行过程不得再提交数据或研究结果。
- 不要指导关闭安全控制或提权。本机 Codex 执行与其他机器人工具权限是分开的；不能用本 brief 要求绕过既有拒绝。

## 2. 模型与用量

启动时在本机查验将要调用的准确模型 ID。用户口头称为「6.1 sol」；组员曾报告 `gpt-6.1-sol`，此为 `historical_record_unverified`，须本机确认。未确认前不要臆造其他 ID。effort 用 `high`，不用 `xhigh`。长文本 MiMo 默认不可用。实现用 Python + DuckDB/Parquet 离线流水线，财务抽取优先结构化字段，不要用模型抽全量正文。

## 3. 状态机与停止条件

每阶段结束写入本地项目根下的 `output/run_state.json`：`passed` / `degraded` / `blocked` / `failed`。`passed` 或 `degraded` 立即进入下一阶段。下列情形停止并报告：核心数据不可用且无法按阶段文档安全降级；权限限制或安全控制（含提权、绕过既有拒绝）；将写源库或项目外；额外费用、授权或批量外发；实质改变范围；测试失败或不可恢复错误。非核心缺失：记录 skip，降级继续。样本外无改善：否决交易用途，仍继续 D 与 E 的描述性研究，不得伪造胜绩，不得解开已封存 holdout。E 的新增研究不得重用已经看过的最终 holdout。

## 4. 执行顺序

1. 阅读 [规划入口](../PLAN.md)、[DATA_SOURCES.md](DATA_SOURCES.md)、[PHASE_A.md](PHASE_A.md)、[PHASE_B.md](PHASE_B.md)、[PHASE_C.md](PHASE_C.md)、[PHASE_D.md](PHASE_D.md)、[PHASE_E.md](PHASE_E.md)，确认官方指数代码（尤其 中证2000=932000 而非 000932，科创50=000688 而非 588000，国证2000 与中证2000 分列）。
2. **[A](PHASE_A.md)** 只读盘点，写 manifest、quality、missing_data、数据字典草案、指数可用性和执行口径。
3. **[B](PHASE_B.md)** 在线状态与事后分段分列，泄漏测试通过后再走。
4. **[C](PHASE_C.md)** 只封存一次 holdout，内层调参、外层 walkforward，分层收益与分时期税费，给出交易采用或否决。无改善则否决交易用途，仍继续 D 与 E。
5. **[D](PHASE_D.md)** 只用预测日前窗口和已成熟 5/20/60 日标签，报告非重叠分布与不确定性。
6. **[E](PHASE_E.md)** 财报 PIT、宏观 vintage、历史行业与幸存者处理；新研究版本不得重用已看过的 holdout。结束。

中断后从 `run_state` 已完成阶段之后恢复，不重写已通过阶段的冻结版本，除非修复错误并升版本。

## 5. 验收总原则

先过泄漏、PIT、holdout 隔离、执行口径和税费分层测试，再谈表现。无开盘价不得冒充次日开盘回测。不向用户索要一堆参数或本可本地查到的路径。不承诺工期或通过率。每个对外结论写入 `output/audit`，能追溯指纹与 asof。
