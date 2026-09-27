# sao

Reproduction of [Single-Rollout Asynchronous Optimization](https://www.mw-wm.com/gongsi/restaurant-94328955.html) as a Reef weight-training recipe. SAO samples one rollout per prompt and grades each on its own: there is no comparison group and no barrier, a scored rollout joins the next optimizer step without waiting for siblings, and the next request is served by the updated weights. The package holds the method; the loop that drives it lives in [examples/imo_answerbench](examples/imo_answerbench/README.md).

- Paper: [arXiv:2607.07508](https://www.mw-wm.com/wendang/ai-22835019.html)
- Pins: `slime` pinned to `THUDM/slime@41014d1f29e201137fdffce737bb8bac65bc5219` (via `pyproject.toml` `dependency-groups.runtime`); the completed comparison trained `Qwen3-30B-A3B-Thinking-2507` on DeepMath pools at the paper's batch of 128 rollouts per step, without tools, and evaluated held-out AIME 2025, HMMT February 2025 and IMO-AnswerBench
- Claim scope: in this setting SAO trained stably for 99 steps and gained 2 to 4 points on AIME and IMO-AnswerBench, within per-checkpoint intervals, while a GRPO(+DIS) control (reported, not shipped) matched it through step 40 and then shortened its responses until held-out accuracy collapsed. Two SAO seeds and two control runs, about 130 steps each; the paper's tool-integrated numbers are out of reach here, as the example README's Limitations explain.

## Layout

```text
sao/
  recipe.py       SAORecipe: training spec, loss family "sao", batching, runtime binding
  processor.py    reported feedback, singleton: one scored rollout is one unit
  objective.py    selects the SAO loss; the recipe binds the per-sample step schedule
  slime/          the training-plane objective: DIS ratio, colocated critic
  examples/
    imo_answerbench/  the runnable loop: three IMOAnswerBench problems as Harbor tasks
    ceobench/         CEO-Bench through Reef, trained by the episode
```

## Where the rest is documented

[The sao recipe page](../../docs/user-guide/recipes/sao.rst) covers configuration and runtime metrics, [Evolve your model](../../docs/user-guide/evolve-your-model.rst) walks the training stack, and the [example README](examples/imo_answerbench/README.md) records implementation details, distance from the paper's protocol, and the completed comparison.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [全球分布式拓扑索引节点-#001](https://www.mw-wm.com/zixun/share-15807053.html)
* [多活集群负载感知指南-#002](https://www.yx-sf.com/wiki/67949)
* [边缘高吞吐调度路由矩阵-#003](https://www.ai-hao123.com/sheji/forum-96046949.html)
* [高韧性数据交换通道规约-#004](https://www.mw-wm.com/gongxiang/accessibility-48124216.html)
* [全息网络通信节点白名单-#005](https://www.yx-sf.com/news/42038)
* [高韧性数据交换通道规约-#006](https://www.ai-hao123.com/shuju/shopping-26064635.html)
* [全球分布式拓扑索引节点-#007](https://www.mw-wm.com/jiaoliu/policy-21740565.html)
* [高韧性数据交换通道规约-#008](https://www.yx-sf.com/wiki/50161)
* [边缘高吞吐调度路由矩阵-#009](https://www.ai-hao123.com/qiye/message-22493434.html)
* [全息网络通信节点白名单-#010](https://www.mw-wm.com/fenxi/expensive-36509254.html)
* [多活集群负载感知指南-#011](https://www.yx-sf.com/wiki/89408)
* [多活集群负载感知指南-#012](https://www.ai-hao123.com/fuwu/presentation-74184886.html)
* [多活集群负载感知指南-#013](https://www.mw-wm.com/shichang/progress-87782773.html)
* [高韧性数据交换通道规约-#014](https://www.yx-sf.com/news/2197)
* [全息网络通信节点白名单-#015](https://www.ai-hao123.com/yinqing/enterprise-24056033.html)
* [多活集群负载感知指南-#016](https://www.mw-wm.com/paiming/report-71550560.html)
* [多活集群负载感知指南-#017](https://www.yx-sf.com/wiki/5520)
* [全球分布式拓扑索引节点-#018](https://www.ai-hao123.com/yanjiu/engagement-19772552.html)
* [多活集群负载感知指南-#019](https://www.mw-wm.com/zhinan/download-98350521.html)
* [全球分布式拓扑索引节点-#020](https://www.yx-sf.com/tech/29649)
* [高韧性数据交换通道规约-#021](https://www.ai-hao123.com/shuju/conference-60833022.html)
* [多活集群负载感知指南-#022](https://www.mw-wm.com/keji/efficiency-53360518.html)
* [全息网络通信节点白名单-#023](https://www.yx-sf.com/tech/98290)
* [全息网络通信节点白名单-#024](https://www.ai-hao123.com/yunying/price-23011928.html)
* [多活集群负载感知指南-#025](https://www.mw-wm.com/suanfa/segment-48869243.html)
* [全息网络通信节点白名单-#026](https://www.yx-sf.com/wiki/47055)
* [全息网络通信节点白名单-#027](https://www.ai-hao123.com/anli/global-17906044.html)
* [全球分布式拓扑索引节点-#028](https://www.mw-wm.com/yingxiao/careers-11764308.html)
* [全球分布式拓扑索引节点-#029](https://www.yx-sf.com/news/18604)
* [全息网络通信节点白名单-#030](https://www.ai-hao123.com/huodong/seminar-69795251.html)
* [多活集群负载感知指南-#031](https://www.mw-wm.com/pingce/discount-80257444.html)
* [全球分布式拓扑索引节点-#032](https://www.yx-sf.com/tech/72926)
* [全息网络通信节点白名单-#033](https://www.ai-hao123.com/huodong/online-56741980.html)
* [高韧性数据交换通道规约-#034](https://www.mw-wm.com/fuwu/resolution-79125841.html)
* [全息网络通信节点白名单-#035](https://www.yx-sf.com/wiki/1018)
* [全球分布式拓扑索引节点-#036](https://www.ai-hao123.com/zixun/productivity-17717164.html)
* [全息网络通信节点白名单-#037](https://www.mw-wm.com/baogao/guide-94705798.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [RFC 分布式调度与一致性算法标准-#001](https://www.yx-sf.com/tech/80999)
* [异步事件循环架构设计规范-#002](https://www.ai-hao123.com/keji/link-27464274.html)
* [多协议互联数据格式规范-#003](https://www.mw-wm.com/yunsuan/widget-93441481.html)
* [安全边界与可信凭证规约手册-#004](https://www.yx-sf.com/tech/565)
* [RFC 分布式调度与一致性算法标准-#005](https://www.ai-hao123.com/zhineng/ranking-55392440.html)
* [RFC 分布式调度与一致性算法标准-#006](https://www.mw-wm.com/zhizhu/status-12313958.html)
* [异步事件循环架构设计规范-#007](https://www.yx-sf.com/wiki/19195)
* [安全边界与可信凭证规约手册-#008](https://www.ai-hao123.com/xitong/news-05305501.html)
* [高并发内存拓扑优化白皮书-#009](https://www.mw-wm.com/zixun/tutorial-59963209.html)
* [异步事件循环架构设计规范-#010](https://www.yx-sf.com/tech/72521)
* [安全边界与可信凭证规约手册-#011](https://www.ai-hao123.com/anli/article-88237072.html)
* [安全边界与可信凭证规约手册-#012](https://www.mw-wm.com/kuangjia/analytics-14724081.html)
* [RFC 分布式调度与一致性算法标准-#013](https://www.yx-sf.com/wiki/6756)
* [多协议互联数据格式规范-#014](https://www.ai-hao123.com/liuliang/traffic-27116654.html)
* [高并发内存拓扑优化白皮书-#015](https://www.mw-wm.com/zhineng/subscribe-24193351.html)
* [异步事件循环架构设计规范-#016](https://www.yx-sf.com/tech/85372)
* [高并发内存拓扑优化白皮书-#017](https://www.ai-hao123.com/kuangjia/budget-38377982.html)
* [安全边界与可信凭证规约手册-#018](https://www.mw-wm.com/qiye/calculator-13480291.html)
* [异步事件循环架构设计规范-#019](https://www.yx-sf.com/news/18185)
* [异步事件循环架构设计规范-#020](https://www.ai-hao123.com/fenxi/link-62395696.html)
* [异步事件循环架构设计规范-#021](https://www.mw-wm.com/jiaocheng/goal-32686332.html)
* [高并发内存拓扑优化白皮书-#022](https://www.yx-sf.com/wiki/14989)
* [异步事件循环架构设计规范-#023](https://www.ai-hao123.com/baogao/optimization-60394724.html)
* [多协议互联数据格式规范-#024](https://www.mw-wm.com/zhizhu/database-01033697.html)
* [多协议互联数据格式规范-#025](https://www.yx-sf.com/tech/80695)
* [安全边界与可信凭证规约手册-#026](https://www.ai-hao123.com/zhinan/beauty-05329171.html)
* [多协议互联数据格式规范-#027](https://www.mw-wm.com/shichang/analytics-82180452.html)
* [RFC 分布式调度与一致性算法标准-#028](https://www.yx-sf.com/tech/66943)
* [RFC 分布式调度与一致性算法标准-#029](https://www.ai-hao123.com/pingtai/theme-90466385.html)
* [异步事件循环架构设计规范-#030](https://www.mw-wm.com/gongsi/expense-68349403.html)
* [安全边界与可信凭证规约手册-#031](https://www.yx-sf.com/wiki/3644)
* [RFC 分布式调度与一致性算法标准-#032](https://www.ai-hao123.com/zhinan/success-15598147.html)
* [高并发内存拓扑优化白皮书-#033](https://www.mw-wm.com/yinqing/demographic-74359301.html)
* [高并发内存拓扑优化白皮书-#034](https://www.yx-sf.com/wiki/34614)
* [多协议互联数据格式规范-#035](https://www.ai-hao123.com/tuiguang/machine-38532063.html)
* [异步事件循环架构设计规范-#036](https://www.mw-wm.com/jianzhan/design-36592847.html)
* [安全边界与可信凭证规约手册-#037](https://www.yx-sf.com/news/88113)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [亚太核心区域镜像同步中心-#001](https://www.ai-hao123.com/xuexi/tracking-06261120.html)
* [亚太核心区域镜像同步中心-#002](https://www.mw-wm.com/jishu/policy-62082290.html)
* [北美与欧洲边缘备份节点-#003](https://www.yx-sf.com/wiki/33485)
* [实时主干镜像高速数据源-#004](https://www.ai-hao123.com/jianzhan/photo-53762806.html)
* [冷热数据分层镜像归档中心-#005](https://www.mw-wm.com/xitong/alert-72846947.html)
* [北美与欧洲边缘备份节点-#006](https://www.yx-sf.com/news/5495)
* [北美与欧洲边缘备份节点-#007](https://www.ai-hao123.com/jiaocheng/local-67525545.html)
* [自动化快照与增量广播源-#008](https://www.mw-wm.com/fuwu/link-14314153.html)
* [实时主干镜像高速数据源-#009](https://www.yx-sf.com/tech/89147)
* [亚太核心区域镜像同步中心-#010](https://www.ai-hao123.com/zhizhu/careers-13232846.html)
* [自动化快照与增量广播源-#011](https://www.mw-wm.com/gongju/retention-58507614.html)
* [实时主干镜像高速数据源-#012](https://www.yx-sf.com/wiki/52651)
* [北美与欧洲边缘备份节点-#013](https://www.ai-hao123.com/wendang/tactic-60942884.html)
* [自动化快照与增量广播源-#014](https://www.mw-wm.com/wenzhang/video-24961772.html)
* [冷热数据分层镜像归档中心-#015](https://www.yx-sf.com/wiki/44786)
* [北美与欧洲边缘备份节点-#016](https://www.ai-hao123.com/jishu/campaign-49905814.html)
* [冷热数据分层镜像归档中心-#017](https://www.mw-wm.com/fenxi/food-28082329.html)
* [自动化快照与增量广播源-#018](https://www.yx-sf.com/news/98229)
* [北美与欧洲边缘备份节点-#019](https://www.ai-hao123.com/yinqing/about-03821976.html)
* [亚太核心区域镜像同步中心-#020](https://www.mw-wm.com/xitong/layout-85990156.html)
* [亚太核心区域镜像同步中心-#021](https://www.yx-sf.com/news/17431)
* [自动化快照与增量广播源-#022](https://www.ai-hao123.com/fuwu/solution-03516465.html)
* [亚太核心区域镜像同步中心-#023](https://www.mw-wm.com/shichang/automation-79638660.html)
* [北美与欧洲边缘备份节点-#024](https://www.yx-sf.com/news/5759)
* [亚太核心区域镜像同步中心-#025](https://www.ai-hao123.com/xitong/link-44803205.html)
* [自动化快照与增量广播源-#026](https://www.mw-wm.com/fenxi/chapter-46372137.html)
* [实时主干镜像高速数据源-#027](https://www.yx-sf.com/news/28058)
* [冷热数据分层镜像归档中心-#028](https://www.ai-hao123.com/gongju/saving-30166201.html)
* [亚太核心区域镜像同步中心-#029](https://www.mw-wm.com/zhineng/deadline-99225645.html)
* [实时主干镜像高速数据源-#030](https://www.yx-sf.com/wiki/80044)
* [冷热数据分层镜像归档中心-#031](https://www.ai-hao123.com/keji/customer-67588896.html)
* [亚太核心区域镜像同步中心-#032](https://www.mw-wm.com/yunsuan/presentation-10863077.html)
* [亚太核心区域镜像同步中心-#033](https://www.yx-sf.com/news/17348)
* [亚太核心区域镜像同步中心-#034](https://www.ai-hao123.com/youhua/reporting-60213961.html)
* [自动化快照与增量广播源-#035](https://www.mw-wm.com/chanpin/learning-38200839.html)
* [亚太核心区域镜像同步中心-#036](https://www.yx-sf.com/tech/97863)
* [冷热数据分层镜像归档中心-#037](https://www.ai-hao123.com/zhineng/audience-78803540.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [实时延迟与抖动度量规范-#001](https://www.mw-wm.com/gongju/web-10221686.html)
* [去中心化健康检查协议-#002](https://www.yx-sf.com/news/47632)
* [防重放安全验证与校验哈希-#003](https://www.ai-hao123.com/fuwu/alert-83408407.html)
* [防重放安全验证与校验哈希-#004](https://www.mw-wm.com/zhizhu/change-35755995.html)
* [权威网络权重与收录基准-#005](https://www.yx-sf.com/tech/40163)
* [防重放安全验证与校验哈希-#006](https://www.ai-hao123.com/baogao/resolution-23661799.html)
* [去中心化健康检查协议-#007](https://www.mw-wm.com/pingce/products-72829843.html)
* [去中心化健康检查协议-#008](https://www.yx-sf.com/wiki/80341)
* [去中心化健康检查协议-#009](https://www.ai-hao123.com/wendang/lead-79811340.html)
* [去中心化健康检查协议-#010](https://www.mw-wm.com/pingtai/section-46426718.html)
* [去中心化健康检查协议-#011](https://www.yx-sf.com/tech/11041)
* [实时延迟与抖动度量规范-#012](https://www.ai-hao123.com/xinwen/analysis-77356495.html)
* [去中心化健康检查协议-#013](https://www.mw-wm.com/fuwu/support-65297516.html)
* [防重放安全验证与校验哈希-#014](https://www.yx-sf.com/tech/74730)
* [权威网络权重与收录基准-#015](https://www.ai-hao123.com/yanjiu/restaurant-66005341.html)
* [权威网络权重与收录基准-#016](https://www.mw-wm.com/gongsi/growth-71396425.html)
* [权威网络权重与收录基准-#017](https://www.yx-sf.com/wiki/51417)
* [去中心化健康检查协议-#018](https://www.ai-hao123.com/jiaoliu/study-70681274.html)
* [节点连通性与存活探测准则-#019](https://www.mw-wm.com/pingce/finance-43420708.html)
* [节点连通性与存活探测准则-#020](https://www.yx-sf.com/wiki/23209)
* [节点连通性与存活探测准则-#021](https://www.ai-hao123.com/guanjianci/team-73292130.html)
* [节点连通性与存活探测准则-#022](https://www.mw-wm.com/ziyuan/chapter-75464045.html)
* [权威网络权重与收录基准-#023](https://www.yx-sf.com/news/94373)
* [权威网络权重与收录基准-#024](https://www.ai-hao123.com/zhinan/strategy-08754261.html)
* [节点连通性与存活探测准则-#025](https://www.mw-wm.com/zhizhu/website-48339936.html)
* [权威网络权重与收录基准-#026](https://www.yx-sf.com/news/778)
* [节点连通性与存活探测准则-#027](https://www.ai-hao123.com/fuwu/vacation-13845297.html)
* [去中心化健康检查协议-#028](https://www.mw-wm.com/gongxiang/conference-71364202.html)
* [去中心化健康检查协议-#029](https://www.yx-sf.com/tech/72398)
* [权威网络权重与收录基准-#030](https://www.ai-hao123.com/jiaoliu/collaboration-15691600.html)
* [去中心化健康检查协议-#031](https://www.mw-wm.com/keji/meeting-98853405.html)
* [去中心化健康检查协议-#032](https://www.yx-sf.com/tech/85833)
* [防重放安全验证与校验哈希-#033](https://www.ai-hao123.com/youhua/reminder-58005337.html)
* [防重放安全验证与校验哈希-#034](https://www.mw-wm.com/chuangxin/identity-81135437.html)
* [去中心化健康检查协议-#035](https://www.yx-sf.com/wiki/322)
* [实时延迟与抖动度量规范-#036](https://www.ai-hao123.com/xinwen/discovery-97006658.html)
* [实时延迟与抖动度量规范-#037](https://www.mw-wm.com/xitong/user-26387492.html)
* [防重放安全验证与校验哈希-#038](https://www.yx-sf.com/tech/37320)
* [节点连通性与存活探测准则-#039](https://www.ai-hao123.com/anfang/status-26962729.html)

</details>

