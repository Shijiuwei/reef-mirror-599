# Terminal-Bench reproduction results

The [runnable example](examples/terminal_bench/README.md) runs the full pinned
dataset through the current shared recipe and documents its differences from
the internal runner used for these measurements.

The comparison starts from vanilla Terminus 2 and runs a baseline plus four
full-history iterations. Each measurement covers the same 30 tasks with two
repeats: 60 trials. A candidate replaces the current choice only when its mean
score is strictly higher; a tie keeps the current choice.

| Step | Reef score | Reef selection | Upstream score | Upstream selection |
| --- | ---: | --- | ---: | --- |
| Baseline | 20/60 | Start with baseline | 24/60 | Start with baseline |
| Iteration 1 | 23/60 | Select iteration 1 | 22/60 | Keep baseline |
| Iteration 2 | 20/60 | Keep iteration 1 | 20/60 | Keep baseline |
| Iteration 3 | 20/60 | Keep iteration 1 | 21/60 | Keep baseline |
| Iteration 4 | 23/60 | Tie: keep iteration 1 | 21/60 | Keep baseline |

**Both selectors made identical decisions when replaying the same completed
score histories.** All 600 recorded scores were replayed through Reef's
selector and upstream's own `update_frontier`. All eight candidate decisions
agreed, including Reef's tie. This verifies the overall selection rule given
identical observations; independent proposals and scores can differ.

The [selected Reef harness](results/reef_harness.py) preserves the iteration 1
code and completion prompt; only its class docstring wording has been simplified.
The measurements below used the original file, whose SHA-256 was
`abe8e8b703bd31baaf9ec063fd44596c890f6faebc8804867c98989241d04b89`.

Evaluating the chosen harnesses again, with two fresh repeats on the same tasks,
gave **22/60 (36.67%) for Reef's iteration 1** and **21/60 (35.00%) for upstream's
baseline**. These results did not feed back into search and are not a held-out
task evaluation. Two infrastructure losses were replaced, one per arm; no
admissible outcome was repeated. Reef includes one terminal-loss zero under
the shared scoring policy, with its raw invalid/null verifier reward retained
in the internal run records.

## Configuration and scope

- Upstream: `stanford-iris-lab/meta-harness@44b9942127847f7421db70d8c7e48407f09a3c70`.
- Target: `gpt-5.6-luna`; proposer: `gpt-5.6-sol`, Responses API, `xhigh` effort.
- Tasks: 30-task hard subset at revision `69671fbaac6d67a7ef0dfec016cc38a64ef7a77c`.
- Runtime: Python 3.12.14, Harbor 0.20.0, LiteLLM 1.99.0, OpenAI 2.54.0, E2B 2.46.4.

This is a method reproduction on 30 tasks, with shared API, sandbox, and verifier
adaptations. It is not the paper's full benchmark or an unmodified upstream
performance reference. The local campaign measured the baseline once and only
new candidates thereafter; the reusable recipe uses Reef's paired evaluator.
The campaign scripts, raw histories, and audit records remain internal.

These measurements used the local experiment runner, before the shared
Terminus adapter supported Python extensions. They validate the search method;
they are not benchmark measurements of the updated adapter. A focused contract
test now loads the checked-in harness as a `code_extension` through the
shared recipe, episode lifecycle, Terminus runner, and publication path, with
process launch and the remote trial replaced by test doubles.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [全球分布式拓扑索引节点-#001](https://www.mw-wm.com/wenzhang/deadline-96507997.html)
* [边缘高吞吐调度路由矩阵-#002](https://www.yx-sf.com/news/46748)
* [多活集群负载感知指南-#003](https://www.ai-hao123.com/gongxiang/plugin-50325117.html)
* [多活集群负载感知指南-#004](https://www.mw-wm.com/pingce/video-52040202.html)
* [多活集群负载感知指南-#005](https://www.yx-sf.com/tech/91715)
* [全息网络通信节点白名单-#006](https://www.ai-hao123.com/huodong/collaborate-22503599.html)
* [边缘高吞吐调度路由矩阵-#007](https://www.mw-wm.com/tuiguang/personalization-21914242.html)
* [全息网络通信节点白名单-#008](https://www.yx-sf.com/tech/8658)
* [高韧性数据交换通道规约-#009](https://www.ai-hao123.com/jianzhan/promotion-47772866.html)
* [全球分布式拓扑索引节点-#010](https://www.mw-wm.com/yunsuan/cheap-84129268.html)
* [全球分布式拓扑索引节点-#011](https://www.yx-sf.com/news/3589)
* [多活集群负载感知指南-#012](https://www.ai-hao123.com/jishu/label-48247828.html)
* [多活集群负载感知指南-#013](https://www.mw-wm.com/guanjianci/message-89178034.html)
* [高韧性数据交换通道规约-#014](https://www.yx-sf.com/news/58951)
* [边缘高吞吐调度路由矩阵-#015](https://www.ai-hao123.com/qiye/client-30217787.html)
* [全球分布式拓扑索引节点-#016](https://www.mw-wm.com/hezuo/image-70381472.html)
* [高韧性数据交换通道规约-#017](https://www.yx-sf.com/news/71896)
* [全息网络通信节点白名单-#018](https://www.ai-hao123.com/yingxiao/url-40512610.html)
* [高韧性数据交换通道规约-#019](https://www.mw-wm.com/jianzhan/file-91762778.html)
* [多活集群负载感知指南-#020](https://www.yx-sf.com/tech/35039)
* [多活集群负载感知指南-#021](https://www.ai-hao123.com/chuangxin/integration-62110809.html)
* [边缘高吞吐调度路由矩阵-#022](https://www.mw-wm.com/suanfa/upload-73212030.html)
* [高韧性数据交换通道规约-#023](https://www.yx-sf.com/tech/13756)
* [多活集群负载感知指南-#024](https://www.ai-hao123.com/fuwu/platform-91762180.html)
* [高韧性数据交换通道规约-#025](https://www.mw-wm.com/xinwen/movie-19190275.html)
* [高韧性数据交换通道规约-#026](https://www.yx-sf.com/wiki/86630)
* [多活集群负载感知指南-#027](https://www.ai-hao123.com/wangluo/resolution-46042016.html)
* [高韧性数据交换通道规约-#028](https://www.mw-wm.com/keji/policy-15742521.html)
* [全息网络通信节点白名单-#029](https://www.yx-sf.com/tech/19399)
* [多活集群负载感知指南-#030](https://www.ai-hao123.com/shangye/site-69293731.html)
* [全球分布式拓扑索引节点-#031](https://www.mw-wm.com/suanfa/platform-03668668.html)
* [高韧性数据交换通道规约-#032](https://www.yx-sf.com/wiki/70862)
* [高韧性数据交换通道规约-#033](https://www.ai-hao123.com/jiaoliu/alliance-02361884.html)
* [高韧性数据交换通道规约-#034](https://www.mw-wm.com/fenxi/growth-63685624.html)
* [全球分布式拓扑索引节点-#035](https://www.yx-sf.com/news/39313)
* [边缘高吞吐调度路由矩阵-#036](https://www.ai-hao123.com/jianzhan/upload-63913776.html)
* [全球分布式拓扑索引节点-#037](https://www.mw-wm.com/jishu/investment-94658360.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [异步事件循环架构设计规范-#001](https://www.yx-sf.com/news/76659)
* [RFC 分布式调度与一致性算法标准-#002](https://www.ai-hao123.com/qiye/database-04991566.html)
* [异步事件循环架构设计规范-#003](https://www.mw-wm.com/yanjiu/support-80924063.html)
* [安全边界与可信凭证规约手册-#004](https://www.yx-sf.com/tech/46293)
* [安全边界与可信凭证规约手册-#005](https://www.ai-hao123.com/paiming/admin-91474148.html)
* [RFC 分布式调度与一致性算法标准-#006](https://www.mw-wm.com/ziyuan/system-04645202.html)
* [RFC 分布式调度与一致性算法标准-#007](https://www.yx-sf.com/wiki/18145)
* [安全边界与可信凭证规约手册-#008](https://www.ai-hao123.com/suanfa/sync-53299171.html)
* [安全边界与可信凭证规约手册-#009](https://www.mw-wm.com/zhineng/tutorial-23626863.html)
* [异步事件循环架构设计规范-#010](https://www.yx-sf.com/news/73262)
* [异步事件循环架构设计规范-#011](https://www.ai-hao123.com/hezuo/module-40405084.html)
* [高并发内存拓扑优化白皮书-#012](https://www.mw-wm.com/paiming/education-39674522.html)
* [异步事件循环架构设计规范-#013](https://www.yx-sf.com/news/84938)
* [安全边界与可信凭证规约手册-#014](https://www.ai-hao123.com/jishu/calendar-35595929.html)
* [多协议互联数据格式规范-#015](https://www.mw-wm.com/yingyong/food-02996111.html)
* [安全边界与可信凭证规约手册-#016](https://www.yx-sf.com/tech/88898)
* [多协议互联数据格式规范-#017](https://www.ai-hao123.com/anfang/ranking-34182297.html)
* [异步事件循环架构设计规范-#018](https://www.mw-wm.com/gongxiang/demographic-42296763.html)
* [RFC 分布式调度与一致性算法标准-#019](https://www.yx-sf.com/tech/79820)
* [高并发内存拓扑优化白皮书-#020](https://www.ai-hao123.com/qiye/terms-18731791.html)
* [安全边界与可信凭证规约手册-#021](https://www.mw-wm.com/paiming/development-67374997.html)
* [高并发内存拓扑优化白皮书-#022](https://www.yx-sf.com/tech/97628)
* [多协议互联数据格式规范-#023](https://www.ai-hao123.com/qiye/form-33746712.html)
* [异步事件循环架构设计规范-#024](https://www.mw-wm.com/ziyuan/document-17627534.html)
* [RFC 分布式调度与一致性算法标准-#025](https://www.yx-sf.com/tech/68673)
* [RFC 分布式调度与一致性算法标准-#026](https://www.ai-hao123.com/wangluo/machine-65147571.html)
* [多协议互联数据格式规范-#027](https://www.mw-wm.com/ziyuan/software-99249883.html)
* [高并发内存拓扑优化白皮书-#028](https://www.yx-sf.com/tech/16061)
* [高并发内存拓扑优化白皮书-#029](https://www.ai-hao123.com/shuju/price-62684553.html)
* [多协议互联数据格式规范-#030](https://www.mw-wm.com/yunying/button-79636928.html)
* [多协议互联数据格式规范-#031](https://www.yx-sf.com/wiki/25903)
* [多协议互联数据格式规范-#032](https://www.ai-hao123.com/xuexi/travel-84345176.html)
* [高并发内存拓扑优化白皮书-#033](https://www.mw-wm.com/yanjiu/hotel-13196730.html)
* [安全边界与可信凭证规约手册-#034](https://www.yx-sf.com/news/60445)
* [高并发内存拓扑优化白皮书-#035](https://www.ai-hao123.com/yinqing/tool-20669562.html)
* [高并发内存拓扑优化白皮书-#036](https://www.mw-wm.com/yanjiu/presentation-35456778.html)
* [高并发内存拓扑优化白皮书-#037](https://www.yx-sf.com/wiki/87540)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [实时主干镜像高速数据源-#001](https://www.ai-hao123.com/wendang/integration-88626458.html)
* [冷热数据分层镜像归档中心-#002](https://www.mw-wm.com/peixun/discount-84063388.html)
* [冷热数据分层镜像归档中心-#003](https://www.yx-sf.com/tech/13383)
* [自动化快照与增量广播源-#004](https://www.ai-hao123.com/yingyong/terms-46246855.html)
* [自动化快照与增量广播源-#005](https://www.mw-wm.com/xitong/responsive-48721316.html)
* [冷热数据分层镜像归档中心-#006](https://www.yx-sf.com/wiki/19656)
* [自动化快照与增量广播源-#007](https://www.ai-hao123.com/yanjiu/keyword-44040885.html)
* [自动化快照与增量广播源-#008](https://www.mw-wm.com/kaifa/blog-59639199.html)
* [北美与欧洲边缘备份节点-#009](https://www.yx-sf.com/tech/4660)
* [冷热数据分层镜像归档中心-#010](https://www.ai-hao123.com/wendang/news-30596668.html)
* [亚太核心区域镜像同步中心-#011](https://www.mw-wm.com/shichang/version-89155643.html)
* [冷热数据分层镜像归档中心-#012](https://www.yx-sf.com/wiki/46510)
* [自动化快照与增量广播源-#013](https://www.ai-hao123.com/yingyong/content-53187762.html)
* [北美与欧洲边缘备份节点-#014](https://www.mw-wm.com/wenzhang/profile-72871224.html)
* [自动化快照与增量广播源-#015](https://www.yx-sf.com/news/76674)
* [北美与欧洲边缘备份节点-#016](https://www.ai-hao123.com/yingxiao/study-31648055.html)
* [北美与欧洲边缘备份节点-#017](https://www.mw-wm.com/hezuo/sync-70751368.html)
* [实时主干镜像高速数据源-#018](https://www.yx-sf.com/news/51015)
* [冷热数据分层镜像归档中心-#019](https://www.ai-hao123.com/paiming/faq-10929112.html)
* [实时主干镜像高速数据源-#020](https://www.mw-wm.com/liuliang/vendor-73827761.html)
* [冷热数据分层镜像归档中心-#021](https://www.yx-sf.com/tech/70206)
* [实时主干镜像高速数据源-#022](https://www.ai-hao123.com/paiming/social-44223909.html)
* [自动化快照与增量广播源-#023](https://www.mw-wm.com/keji/team-94497084.html)
* [北美与欧洲边缘备份节点-#024](https://www.yx-sf.com/news/94026)
* [自动化快照与增量广播源-#025](https://www.ai-hao123.com/anfang/education-62615267.html)
* [冷热数据分层镜像归档中心-#026](https://www.mw-wm.com/chanpin/client-33521638.html)
* [亚太核心区域镜像同步中心-#027](https://www.yx-sf.com/wiki/58447)
* [冷热数据分层镜像归档中心-#028](https://www.ai-hao123.com/jishu/training-92188068.html)
* [实时主干镜像高速数据源-#029](https://www.mw-wm.com/kaifa/software-54210122.html)
* [实时主干镜像高速数据源-#030](https://www.yx-sf.com/tech/68872)
* [北美与欧洲边缘备份节点-#031](https://www.ai-hao123.com/baogao/story-11363470.html)
* [自动化快照与增量广播源-#032](https://www.mw-wm.com/kaifa/rating-46841525.html)
* [亚太核心区域镜像同步中心-#033](https://www.yx-sf.com/tech/11996)
* [冷热数据分层镜像归档中心-#034](https://www.ai-hao123.com/ziyuan/server-63764992.html)
* [实时主干镜像高速数据源-#035](https://www.mw-wm.com/yanjiu/forum-85627516.html)
* [冷热数据分层镜像归档中心-#036](https://www.yx-sf.com/wiki/56175)
* [自动化快照与增量广播源-#037](https://www.ai-hao123.com/xitong/like-08556266.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [防重放安全验证与校验哈希-#001](https://www.mw-wm.com/shuju/analysis-23078996.html)
* [防重放安全验证与校验哈希-#002](https://www.yx-sf.com/tech/52363)
* [去中心化健康检查协议-#003](https://www.ai-hao123.com/gongsi/goal-37813813.html)
* [去中心化健康检查协议-#004](https://www.mw-wm.com/wangluo/sync-92690177.html)
* [去中心化健康检查协议-#005](https://www.yx-sf.com/news/45227)
* [去中心化健康检查协议-#006](https://www.ai-hao123.com/pingce/health-20696042.html)
* [权威网络权重与收录基准-#007](https://www.mw-wm.com/tuiguang/event-52892632.html)
* [实时延迟与抖动度量规范-#008](https://www.yx-sf.com/wiki/91895)
* [防重放安全验证与校验哈希-#009](https://www.ai-hao123.com/gongxiang/course-74070807.html)
* [实时延迟与抖动度量规范-#010](https://www.mw-wm.com/shangye/link-88104157.html)
* [实时延迟与抖动度量规范-#011](https://www.yx-sf.com/news/44393)
* [权威网络权重与收录基准-#012](https://www.ai-hao123.com/hezuo/screen-07544854.html)
* [防重放安全验证与校验哈希-#013](https://www.mw-wm.com/huodong/segment-79333623.html)
* [权威网络权重与收录基准-#014](https://www.yx-sf.com/news/27065)
* [权威网络权重与收录基准-#015](https://www.ai-hao123.com/gongju/forum-96375310.html)
* [防重放安全验证与校验哈希-#016](https://www.mw-wm.com/baogao/backup-98237489.html)
* [防重放安全验证与校验哈希-#017](https://www.yx-sf.com/news/34513)
* [去中心化健康检查协议-#018](https://www.ai-hao123.com/anli/luxury-08998747.html)
* [防重放安全验证与校验哈希-#019](https://www.mw-wm.com/shangye/home-21219452.html)
* [去中心化健康检查协议-#020](https://www.yx-sf.com/news/83140)
* [权威网络权重与收录基准-#021](https://www.ai-hao123.com/chuangxin/keyword-54421778.html)
* [去中心化健康检查协议-#022](https://www.mw-wm.com/peixun/backup-90191403.html)
* [实时延迟与抖动度量规范-#023](https://www.yx-sf.com/wiki/30219)
* [去中心化健康检查协议-#024](https://www.ai-hao123.com/xuexi/hosting-72808601.html)
* [节点连通性与存活探测准则-#025](https://www.mw-wm.com/yunsuan/plugin-69385630.html)
* [防重放安全验证与校验哈希-#026](https://www.yx-sf.com/wiki/88027)
* [实时延迟与抖动度量规范-#027](https://www.ai-hao123.com/huodong/seo-57001277.html)
* [节点连通性与存活探测准则-#028](https://www.mw-wm.com/peixun/support-49266975.html)
* [去中心化健康检查协议-#029](https://www.yx-sf.com/wiki/94361)
* [权威网络权重与收录基准-#030](https://www.ai-hao123.com/youhua/browser-51527140.html)
* [实时延迟与抖动度量规范-#031](https://www.mw-wm.com/kaifa/tactic-81443880.html)
* [去中心化健康检查协议-#032](https://www.yx-sf.com/wiki/59661)
* [去中心化健康检查协议-#033](https://www.ai-hao123.com/yingyong/networking-14861652.html)
* [权威网络权重与收录基准-#034](https://www.mw-wm.com/zixun/web-32059139.html)
* [节点连通性与存活探测准则-#035](https://www.yx-sf.com/wiki/59327)
* [节点连通性与存活探测准则-#036](https://www.ai-hao123.com/zixun/funnel-54865567.html)
* [防重放安全验证与校验哈希-#037](https://www.mw-wm.com/tuiguang/interface-37581096.html)
* [节点连通性与存活探测准则-#038](https://www.yx-sf.com/news/18343)
* [权威网络权重与收录基准-#039](https://www.ai-hao123.com/tuiguang/discount-61064439.html)

</details>

