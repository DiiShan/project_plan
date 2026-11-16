# 数据源与公开口径核验（v2）

- 规划入口：[../PLAN.md](../PLAN.md)。
- 查证日期：2026-11-17。公开网页与官方 PDF 于当日检索或抓取，链接如下。本地路径无法在线证实，一律待本机核实。
- 状态标签：`confirmed_by_user`（用户确认过路径或规则，不等于已核实现行内容）、`historical_record_unverified`（历史盘点或组员转述，必须重测）、`discovered_this_run`（本机运行中实测）、`missing`（尚未找到或确认不可用）。
- 下表已写出的本地路径来自既有记录。本次只公开路径字符串，不打开文件、不读取库内容。内容覆盖、表结构和日期范围待本机只读核实后，才可标为 `discovered_this_run`。QMT 具体库路径与 `datasink_caibao` 准确路径仍为 `missing`，必须从索引与本机配置发现，本文不编造。
- 本项目**先尽量只用本地已有库**。源库只读，派生只写 `D:\project\grok\market-regime`。允许查阅公开文档以确认代码、字段口径和制度规则；**不默许**为补库而批量下载，也**不默许**把私有数据、财报全文、日志或凭证上传到 GitHub 或第三方。
- 有权限才能取的数据（Tushare 积分、QMT 终端登录、付费接口）不得假设已开通 token 或积分。本规划不把这些接口当作将要执行的补数通道。

## 1. 本地路径（在线无法证实，待阶段 A 只读盘点）

| 对象 | 路径或名称 | 状态 | 说明 |
|---|---|---|---|
| 数据总索引 | `E:\金融数据\data.list` | 路径来自既有记录，`confirmed_by_user`；内容覆盖待本机只读核实，现标 `missing`，待 `discovered_this_run` | 本机盘点的第一入口。以运行时现行文件为准，不沿用旧摘录。 |
| 盘立方库 | `E:\金融数据\plf_data\database\plf.duckdb` | 路径来自既有记录，`confirmed_by_user`；库内表与覆盖待本机只读核实（`historical_record_unverified`） | 旧记曾有股指期权 IO/MO/HO、商品期货与商品期权日线等，**不得把旧覆盖写成当前事实**。 |
| 宏观库 | `E:\金融数据\macro_data\database\macro.duckdb` | 路径来自既有记录，`confirmed_by_user`；字段与发布日覆盖待本机只读核实（`historical_record_unverified`） | 旧记序列规模不写入本文件，以免把过时数量当现状。须核验是否有发布日/vintage。 |
| QMT / 迅投本地 A 股历史 | 具体库或 `userdata` 路径 `missing` | 待从 `data.list` 与本机 QMT 配置只读发现，**不得编造路径** | 用户称个股行情与数据本地较全，仍须盘点后再谈补数。官方接口说明见第 3 节，不代表本机已有全部字段。 |
| 财报全文库 | 名称 `datasink_caibao`，准确路径 `missing` | 待在索引和目录中核实 | 全文不等于结构化 PIT 财报。有无公告时间、修订版本、原文定位，一律实测。 |
| 结构化财报、交易日历、历史成分、ST/退市、涨跌停、资金面、行业分类 | 路径与是否存在均为 `historical_record_unverified` 或 `missing` | 不得把 2026-11-01 前后的「仍缺」清单当成现在仍缺 | 用户已提示本地尽量全，阶段 A 先查后分级。 |

旧盘点中曾出现、现一律待重测的线索（不作为缺口结论）：A 股退市/ST/涨跌停/指数历史成分、资金面、一致预期、部分宏观细项、港美股行情。

## 2. 目标指数官方代码、基日与发布日

以下为 2026-11-17 对编制方公开文件的核验。本地首尾日期、是否有开盘价、本地代码后缀（`.SH` / `.SZ` / 其他）只能本机发现。官方发布日前若编制方提供回溯行情，使用时必须标注「发布日前官方回溯」；**禁止自行向前插补或为对齐不同指数而伪造共同历史**。基日、发布日、本地首日是三个不同概念，须分列。

| 指数 | 官方代码 | 发布日 | 基日 | 基点 | 核验来源（2026-11-17） | 注意 |
|---|---|---|---|---|---|---|
| 上证指数 | 000001 | 1991-07-15 | 1990-12-19 | 100 | 编制方案（代码与基日、基点）：[上证综合指数编制方案](http://www.sse.com.cn/market/sseindex/indexlist/indexdetails/indexmethods/c1/000001_000001_CN.pdf)。发布日：国家统计局知识页《上证股价指数》（发布 2023-07-20）[页面](https://www.stats.gov.cn/zs/tjll/tjzs/202307/t20230719_1941399.html)。该编制方案写明自 2020-07-22 起实施，且样本不含 ST、*ST，与早期口径可能不同。 | 编制方案未写发布日，故发布日另引统计局页面。勿与股票代码 000001.SZ 混淆。 |
| 上证50 | 000016 | 2004-01-02 | 2003-12-31 | 1000 | 中证/上证事实页，页眉日期 2026-10-30，OSS：[000016factsheet.pdf](https://oss-ch.csindex.com.cn/static/html/csindex/public/uploads/indices/detail/files/zh_CN/000016factsheet.pdf)。上交所 2003-12-29 新闻亦载同一发布日、代码与基日：[链接](http://www.sse.com.cn/market/sseindex/diclosure/c/c_20150911_3984876.shtml)。 | |
| 沪深300 | 000300；深交所行情代码另有 399300 | 2005-04-08 | 2004-12-31 | 1000 | 事实页 2026-10-30：[000300factsheet.pdf](https://oss-ch.csindex.com.cn/static/html/csindex/public/uploads/indices/detail/files/zh_CN/000300factsheet.pdf)；编制方案：[000300_Index_Methodology_cn.pdf](https://oss-ch.csindex.com.cn/static/html/csindex/public/uploads/indices/detail/files/zh_CN/000300_Index_Methodology_cn.pdf)。 | 本地若两套代码并存，盘点时映射，分析时只选一条主序列，避免重复计算。 |
| 中证500 | 000905；另有 399905 | 2007-01-15 | 2004-12-31 | 1000 | 事实页 2026-10-30：[000905factsheet.pdf](https://oss-ch.csindex.com.cn/static/html/csindex/public/uploads/indices/detail/files/zh_CN/000905factsheet.pdf)。 | 同上，双代码只保留一条主序列。 |
| 中证1000 | 000852 | 2014-10-17 | 2004-12-31 | 1000 | 事实页 2026-10-30：[000852factsheet.pdf](https://oss-ch.csindex.com.cn/static/html/csindex/public/uploads/indices/detail/files/zh_CN/000852factsheet.pdf)。 | 基日早于发布日，若本地有发布日前数据，须标为官方回溯而非自造。 |
| 国证2000 | 399303 | 2014-03-28 | 2009-12-31 | 3326.4448 | 国证编制方案：[gz_399303.pdf](https://www.cnindex.com.cn/docs/gz_399303.pdf)；事实页：[jj_399303.pdf](https://www.cnindex.com.cn/html2pdf/preview/jj_399303.pdf)；指数详情入口：[index-detail 399303](https://www.cnindex.com.cn/module/index-detail.html?act_menu=1&indexCode=399303)。 | 与中证2000不是同一指数，必须并列分列，不得混用或拼接。 |
| 中证2000 | **932000** | 2023-08-11 | 2013-12-31 | 1000 | 编制方案：[932000_Index_Methodology_cn.pdf](https://oss-ch.csindex.com.cn/static/html/csindex/public/uploads/indices/detail/files/zh_CN/20231208180041-932000_Index_Methodology_cn.pdf)；事实页：[932000factsheet.pdf](https://oss-ch.csindex.com.cn/static/html/csindex/public/uploads/indices/detail/files/zh_CN/932000factsheet.pdf)。 | **不是 000932**。000932 为中证主要消费，事实页见 [000932factsheet.pdf](https://oss-ch.csindex.com.cn/static/html/csindex/public/uploads/indices/detail/files/zh_CN/000932factsheet.pdf)。代码用错会污染全部小盘结论。 |
| 科创50 | **000688** | 2020-07-23 | 2019-12-31 | 1000 | 上交所编制方案：[000688_000688hbook_CN.pdf](https://www.sse.com.cn/market/sseindex/indexlist/indexdetails/indexmethods/c1/000688_000688hbook_CN.pdf)；事实页 2026-10-30：[000688factsheet.pdf](https://oss-ch.csindex.com.cn/static/html/csindex/public/uploads/indices/detail/files/zh_CN/000688factsheet.pdf)。 | **588000 是跟踪该指数的 ETF 代码，不是指数本身。** |
| 创业板指 | 399006 | 2010-06-01 | 2010-05-31 | 1000 | 国证事实页（页内日期 2026-10）：[jj_399006.pdf](https://www.cnindex.com.cn/html2pdf/preview/jj_399006.pdf)。深交所 2010-05-31 公告页 URL 存在：[t20100531_500454.html](https://www.szse.cn/disclosure/notice/general/t20100531_500454.html)，但 2026-11-17 抓取返回 500，正文未在本次复核中打开，故发布日以国证事实页为准。 | 成交量口径可能与「全创业板成交」不同，见第 3 节 Tushare 说明。 |

本地代码如何写（如 `000300.SH`、`399006.SZ`、`932000.SH`）以本机数据字典为准，不在规划中臆造行情源后缀。

## 3. 字段口径的公开信息源（有权限才可取数，不假设本机已接）

这些来源只用于理解字段含义、权限门槛和常见陷阱，**不是授权补数**。

### 3.1 交易日历与交易时间

- 上交所：《上海证券交易所交易规则（2026年修订）》英译页载明交易日为周一至周五，法定节假日及本所公告休市日除外；竞价开盘 9:15–9:25，连续竞价 9:30–11:30、13:00–14:57，收盘集合竞价 14:57–15:00。[Trading Rules (2026 Revision)](https://english.sse.com.cn/start/sserules/stocks/trading/c/10825757/files/d263e3a87f37436ca2f8e5bcfc4ff001.pdf)。中文规则以交易所现行中文文本为准。
- 休市安排须用交易所公告，例如 [上证公告〔2025〕45号（2025-12-22）](https://www.sse.com.cn/disclosure/announcement/general/c/c_20251222_10802507.shtml) 及 [休市安排列表](https://www.sse.com.cn/disclosure/dealinstruc/closed/list/)。历史回测的交易日以**本地交易日历**为准，公开公告只作核验线索。
- 迅投 QMT `xtdata` 文档提供 `get_trading_dates`、`get_trading_calendar`、`get_holidays`，并说明数据来自已连接的 MiniQMT 本地，不足时文档中另有 `download_*`。本项目默认**只读本地已有数据**，未经另行批准不得用下载接口补源库。[XtQuant.XtData 行情模块](http://dict.thinktrader.net/nativeApi/xtdata.html)（文档含 2020–2024 修订记录）。
- Tushare 有交易日历接口目录，官方权限说明见 [积分与频次权限对应表](https://tushare.pro/document/1?doc_id=290)（2026-11-17 打开：积分门槛与独立付费权限分列）。**不得假设已有 token 或足够积分。** 本项目不把 Tushare 当默认数据源。

### 3.2 指数日线字段

- 迅投 K 线字段（文档）：`time, open, high, low, close, volume, amount, preClose, suspendFlag` 等。`suspendFlag`：0 正常，1 停牌，-1 当日起复牌。代码格式示例为 `000300.SH`。见同一 [xtdata 文档](http://dict.thinktrader.net/nativeApi/xtdata.html)。本机是否实际存有这些字段，待盘点。
- Tushare `index_daily`（2026-11-17 打开）：输出 `ts_code, trade_date, close, open, high, low, pre_close, change, pct_chg, vol, amount`。文档写明需累计 2000 积分方可调取；并提醒部分指数成交量与行情软件「全市场成交」不一致（例如创业板指成分成交 vs 创业板综指）。[index_daily](https://tushare.pro/document/2?doc_id=95)。
- AKShare 1.20.65 文档（2026-11-17 打开）：新浪指数历史接口现名为 `index_zh_daily_sina`（不要沿用过时函数名当本机事实），输出 `date, open, high, low, close, volume`。文档明确：「新浪的数据开始时间，不是该指数的上市时间」。[指数数据文档](https://akshare.akfamily.xyz/data/index/index.html)。公开文档可查；批量抓取补库不在授权内。

### 3.3 财报公告与修订

- 《上市公司信息披露管理办法》（证监会令，政府网转载）：年报在会计年度结束之日起四个月内披露，中期报告在上半年结束之日起两个月内披露。[政府网文本](https://www.gov.cn/zhengce/2021-12/16/content_5724570.htm)。后续修订施行日期以证监会最新文本为准；新闻报道称 2025 年修订自 2025-07-01 施行，规划只把「须用公告日而非报告期」作为 PIT 原则，不以新闻代替法条细节。
- 《公开发行证券的公司信息披露编报规则第19号——财务信息的更正及相关披露》（证监会公告〔2025〕5号，2025-03-27）：更正须单独以临时报告及时披露更正后财务信息；年报更正在特定情形下需审计或专项鉴证。本次打开的是法规汇编页 [relai 转载](https://relai.com.cn/securities/idcosp/cr/19.html)，正式引用应以证监会原文为准。
- 上交所信息披露办事指引列出年报/半年报/季报时限，以及业绩预告、快报的更正情形：[one.sse.com.cn 信息披露](https://one.sse.com.cn/onething/xxpl/)。
- 迅投财务接口可用 `report_type='announce_time'` 按披露日期筛选，表内常见 `m_anntime`（披露/公告日）与 `m_timetag`（截止日期）。见 [xtdata 文档](http://dict.thinktrader.net/nativeApi/xtdata.html)。本机是否有这些列待盘点。
- **全文库不是 PIT。** 无公告时间或无法区分修订版本的抽取，禁止进入历史拟合，仅可作当下参考，并标注口径。大模型训练知识可能含未来信息，不得当作当时可知特征。

### 3.4 税费：必须分工具、分时期，禁止把股票印花税套到指数上

- 《中华人民共和国印花税法》第三条：本法所称证券交易，指转让在依法设立的证券交易所、国务院批准的其他全国性证券交易场所交易的**股票和以股票为基础的存托凭证**；对出让方征收，不对受让方征收。2022-07-01 起施行。[中国人大网](http://www.npc.gov.cn/npc/c2/c30834/202106/t20210610_311898.html)。
- 财政部 税务总局公告 2023 年第 39 号：自 **2023-08-28** 起证券交易印花税减半征收。[中国政府网](https://www.gov.cn/zhengce/zhengceku/202308/content_6900443.htm)。
- 国家税务总局上海市税务局 2023-09-08 问答重申征税对象为股票和以股票为基础的存托凭证，计税依据为成交金额。[近期印花税相关热点问题](https://shanghai.chinatax.gov.cn/zcfw/rdwd/202309/t468612.html)。
- 指数点位本身不是应税证券。指数信号收益不得直接套用股票印花税。交易工具层（股票、ETF、其他基金等）须按标的类型和当时有效文件分别计费。上交所收费一览表（页面更新时间：2026年1月）列的是经手费等交易所费用，不是印花税：A 股竞价经手费与基金（含 ETF、LOF）竞价经手费费率不同，货币 ETF、债券 ETF 经手费有暂免情形。[上交所收费一览表](https://www.sse.com.cn/services/tradingservice/charge/ssecharge/)。
- 完整历史费率表不在本规划中拍板。阶段 C 若做交易成本敏感性，应建立「分工具、分时期」的配置，初值来自上述公开文件与本地券商常规佣金假设，并在汇报中列出；缺历史某段官方费率则标注不确定，做区间敏感性，不编造精度。

## 4. GitHub 与外发边界

本次仅交付并提交规划文档，位置是公开仓库中的 `market-regime/`（[规划入口](../PLAN.md) 与本目录文稿）。不运行 Codex 或研究程序，不访问本地数据库。用户随后在本机执行。

禁止上传：源数据库、派生 DuckDB/Parquet、财报全文或抽取语料、运行日志、机器调试材料、凭证、token、账户与仓位信息、个人资金规模或其他个人财务数字，以及附件 uploads 目录。本地项目根建议用忽略规则排除 `data/derived/`、`output/`、`logs/`，以及任何超出已公开路径字符串的敏感摘录。

已公开、且来自既有记录的路径只有：`D:\project\grok\market-regime`、`E:\金融数据\data.list`、`E:\金融数据\plf_data\database\plf.duckdb`、`E:\金融数据\macro_data\database\macro.duckdb`。内容覆盖待本机只读核实。QMT 具体库路径和 `datasink_caibao` 准确路径仍待本机索引与配置发现，本文不编造。本机 Codex 执行过程不得把数据库、财报全文、日志、凭证或账户信息提交到 GitHub 或第三方。
