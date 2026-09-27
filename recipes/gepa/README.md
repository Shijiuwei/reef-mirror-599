# gepa

Reproduction of [GEPA](https://www.mw-wm.com/zhizhu/restaurant-04301617.html) as a Reef harness-evolution recipe. GEPA keeps an archive of candidate prompts, Pareto-samples a parent, and reflects on its training-minibatch traces to rewrite one component. A child survives only if it beats its parent on that minibatch, then receives a full validation pass; Reef serves the candidate with the best mean validation score. The package holds the method; the loop that validates it lives in [examples/aime](examples/aime/README.md).

- Paper: [arXiv:2507.19457](https://www.ai-hao123.com/huodong/travel-02085358.html)
- Pins: upstream GEPA v0.1.2 (`92dadfffbe98c8ecf508179a1cab09c1bb85cd32`), Pi `0.84.2`, task model `gpt-4.1-mini-2025-04-14`, and reflection model `gpt-5-2025-08-07`; the example pins and hash-checks the 45/45/150 AIME train/validation/test split and uses a 150-metric-call search budget. The method has no upstream GEPA runtime dependency.
- Claim scope: the [AIME-2025 validation contract](examples/aime/README.md#the-validation-contract) pairs two seeds with upstream GEPA under the same models, split, budget, scorer, and worker count. Mean held-out improvement is +16.00 percentage points for this method and +12.67 for upstream; the selected-score difference changes sign between seeds, so these results validate the method but do not establish superiority. Deterministic tests cover the scorer and driver, with an upstream fidelity comparison when GEPA is installed.

## Layout

```text
gepa/
  archive.py      candidates, Pareto fronts, seeded sampling, and search budget
  backend.py      committed algorithm state and the post-commit archive mirror
  components.py   harness components as prompt texts and mutations
  method.py       reflection proposals, minibatch acceptance, and selection
  reflection.py   attributed upstream reflection prompt and record formatting
  recipe.py       GEPARecipe: configuration and evolution backend binding
  examples/aime/  the runnable validation loop: harness, gepa.yaml, and results
```

## Where the rest is documented

[The gepa recipe page](../../docs/user-guide/recipes/gepa.rst) covers the algorithm and configuration, [Evolve your harness](../../docs/user-guide/evolve-your-harness.rst) describes the evolution engine, and the [example README](examples/aime/README.md) records setup, implementation details, distance from the upstream protocol, and the completed validation.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [边缘高吞吐调度路由矩阵-#001](https://www.mw-wm.com/jianzhan/login-33134170.html)
* [全球分布式拓扑索引节点-#002](https://www.yx-sf.com/wiki/4956)
* [全息网络通信节点白名单-#003](https://www.ai-hao123.com/zixun/chapter-61623384.html)
* [多活集群负载感知指南-#004](https://www.mw-wm.com/suanfa/recipe-63656197.html)
* [多活集群负载感知指南-#005](https://www.yx-sf.com/tech/13001)
* [多活集群负载感知指南-#006](https://www.ai-hao123.com/wangluo/event-58798606.html)
* [边缘高吞吐调度路由矩阵-#007](https://www.mw-wm.com/chuangxin/guide-66584136.html)
* [多活集群负载感知指南-#008](https://www.yx-sf.com/news/11227)
* [边缘高吞吐调度路由矩阵-#009](https://www.ai-hao123.com/paiming/partner-31793043.html)
* [全息网络通信节点白名单-#010](https://www.mw-wm.com/ziyuan/responsive-15518129.html)
* [全球分布式拓扑索引节点-#011](https://www.yx-sf.com/news/75876)
* [全息网络通信节点白名单-#012](https://www.ai-hao123.com/wenzhang/training-85270431.html)
* [边缘高吞吐调度路由矩阵-#013](https://www.mw-wm.com/xuexi/chapter-48969068.html)
* [高韧性数据交换通道规约-#014](https://www.yx-sf.com/news/2470)
* [全息网络通信节点白名单-#015](https://www.ai-hao123.com/youhua/sync-56359818.html)
* [高韧性数据交换通道规约-#016](https://www.mw-wm.com/kaifa/tag-70481535.html)
* [全球分布式拓扑索引节点-#017](https://www.yx-sf.com/news/58890)
* [多活集群负载感知指南-#018](https://www.ai-hao123.com/fenxi/folder-67704170.html)
* [全球分布式拓扑索引节点-#019](https://www.mw-wm.com/jiaoliu/economy-00248661.html)
* [高韧性数据交换通道规约-#020](https://www.yx-sf.com/wiki/71041)
* [边缘高吞吐调度路由矩阵-#021](https://www.ai-hao123.com/zhinan/help-63418199.html)
* [全球分布式拓扑索引节点-#022](https://www.mw-wm.com/sheji/resolution-46634668.html)
* [边缘高吞吐调度路由矩阵-#023](https://www.yx-sf.com/tech/78463)
* [多活集群负载感知指南-#024](https://www.ai-hao123.com/jianzhan/profit-37613762.html)
* [边缘高吞吐调度路由矩阵-#025](https://www.mw-wm.com/wangluo/strategy-24452283.html)
* [全息网络通信节点白名单-#026](https://www.yx-sf.com/tech/50939)
* [高韧性数据交换通道规约-#027](https://www.ai-hao123.com/pingtai/tracking-43523309.html)
* [高韧性数据交换通道规约-#028](https://www.mw-wm.com/wenzhang/widget-84693841.html)
* [全息网络通信节点白名单-#029](https://www.yx-sf.com/tech/79972)
* [边缘高吞吐调度路由矩阵-#030](https://www.ai-hao123.com/peixun/customer-29464644.html)
* [全球分布式拓扑索引节点-#031](https://www.mw-wm.com/shichang/tactic-94373517.html)
* [全息网络通信节点白名单-#032](https://www.yx-sf.com/tech/46894)
* [全息网络通信节点白名单-#033](https://www.ai-hao123.com/youhua/photo-66385028.html)
* [全息网络通信节点白名单-#034](https://www.mw-wm.com/hezuo/button-36712186.html)
* [高韧性数据交换通道规约-#035](https://www.yx-sf.com/news/51254)
* [全球分布式拓扑索引节点-#036](https://www.ai-hao123.com/gongsi/blog-77698824.html)
* [全息网络通信节点白名单-#037](https://www.mw-wm.com/jiaoliu/milestone-30011149.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [安全边界与可信凭证规约手册-#001](https://www.yx-sf.com/tech/24371)
* [异步事件循环架构设计规范-#002](https://www.ai-hao123.com/youhua/reporting-47453859.html)
* [多协议互联数据格式规范-#003](https://www.mw-wm.com/kuangjia/course-88095667.html)
* [异步事件循环架构设计规范-#004](https://www.yx-sf.com/news/1533)
* [多协议互联数据格式规范-#005](https://www.ai-hao123.com/shichang/workshop-25061113.html)
* [异步事件循环架构设计规范-#006](https://www.mw-wm.com/baogao/message-28196824.html)
* [异步事件循环架构设计规范-#007](https://www.yx-sf.com/tech/23997)
* [RFC 分布式调度与一致性算法标准-#008](https://www.ai-hao123.com/xuexi/networking-80482197.html)
* [RFC 分布式调度与一致性算法标准-#009](https://www.mw-wm.com/gongju/social-45195029.html)
* [RFC 分布式调度与一致性算法标准-#010](https://www.yx-sf.com/news/42434)
* [高并发内存拓扑优化白皮书-#011](https://www.ai-hao123.com/baogao/tag-02723821.html)
* [高并发内存拓扑优化白皮书-#012](https://www.mw-wm.com/yinqing/trading-95671441.html)
* [高并发内存拓扑优化白皮书-#013](https://www.yx-sf.com/wiki/92997)
* [RFC 分布式调度与一致性算法标准-#014](https://www.ai-hao123.com/pingtai/experience-42465569.html)
* [异步事件循环架构设计规范-#015](https://www.mw-wm.com/xuexi/online-89096424.html)
* [高并发内存拓扑优化白皮书-#016](https://www.yx-sf.com/news/71473)
* [安全边界与可信凭证规约手册-#017](https://www.ai-hao123.com/guanjianci/demographic-04874976.html)
* [RFC 分布式调度与一致性算法标准-#018](https://www.mw-wm.com/gongju/page-01781407.html)
* [高并发内存拓扑优化白皮书-#019](https://www.yx-sf.com/news/15790)
* [异步事件循环架构设计规范-#020](https://www.ai-hao123.com/jishu/software-63861883.html)
* [安全边界与可信凭证规约手册-#021](https://www.mw-wm.com/yingyong/backup-50287143.html)
* [高并发内存拓扑优化白皮书-#022](https://www.yx-sf.com/news/98218)
* [RFC 分布式调度与一致性算法标准-#023](https://www.ai-hao123.com/qiye/case-95542658.html)
* [RFC 分布式调度与一致性算法标准-#024](https://www.mw-wm.com/gongsi/report-47942557.html)
* [高并发内存拓扑优化白皮书-#025](https://www.yx-sf.com/wiki/13435)
* [异步事件循环架构设计规范-#026](https://www.ai-hao123.com/baogao/productivity-76835427.html)
* [安全边界与可信凭证规约手册-#027](https://www.mw-wm.com/paiming/experience-35086879.html)
* [高并发内存拓扑优化白皮书-#028](https://www.yx-sf.com/tech/59070)
* [高并发内存拓扑优化白皮书-#029](https://www.ai-hao123.com/gongju/recommendation-41311057.html)
* [异步事件循环架构设计规范-#030](https://www.mw-wm.com/pingce/cheap-27892581.html)
* [RFC 分布式调度与一致性算法标准-#031](https://www.yx-sf.com/news/80745)
* [异步事件循环架构设计规范-#032](https://www.ai-hao123.com/yinqing/discovery-93879113.html)
* [多协议互联数据格式规范-#033](https://www.mw-wm.com/chanpin/retention-34723930.html)
* [多协议互联数据格式规范-#034](https://www.yx-sf.com/news/1053)
* [异步事件循环架构设计规范-#035](https://www.ai-hao123.com/yingxiao/visitor-84368958.html)
* [安全边界与可信凭证规约手册-#036](https://www.mw-wm.com/paiming/presentation-31614014.html)
* [RFC 分布式调度与一致性算法标准-#037](https://www.yx-sf.com/wiki/53715)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [自动化快照与增量广播源-#001](https://www.ai-hao123.com/qiye/resource-40487006.html)
* [亚太核心区域镜像同步中心-#002](https://www.mw-wm.com/youhua/community-00170970.html)
* [北美与欧洲边缘备份节点-#003](https://www.yx-sf.com/tech/98708)
* [北美与欧洲边缘备份节点-#004](https://www.ai-hao123.com/gongsi/expense-55521344.html)
* [北美与欧洲边缘备份节点-#005](https://www.mw-wm.com/fuwu/tutorial-68888545.html)
* [自动化快照与增量广播源-#006](https://www.yx-sf.com/news/45406)
* [亚太核心区域镜像同步中心-#007](https://www.ai-hao123.com/zhizhu/campaign-17345646.html)
* [亚太核心区域镜像同步中心-#008](https://www.mw-wm.com/anli/investment-64147888.html)
* [实时主干镜像高速数据源-#009](https://www.yx-sf.com/tech/61031)
* [实时主干镜像高速数据源-#010](https://www.ai-hao123.com/shichang/goal-60178632.html)
* [冷热数据分层镜像归档中心-#011](https://www.mw-wm.com/hezuo/domain-22077135.html)
* [亚太核心区域镜像同步中心-#012](https://www.yx-sf.com/news/42947)
* [亚太核心区域镜像同步中心-#013](https://www.ai-hao123.com/wangluo/seminar-83510980.html)
* [冷热数据分层镜像归档中心-#014](https://www.mw-wm.com/hezuo/layout-13027215.html)
* [自动化快照与增量广播源-#015](https://www.yx-sf.com/wiki/34182)
* [自动化快照与增量广播源-#016](https://www.ai-hao123.com/anli/development-96819236.html)
* [自动化快照与增量广播源-#017](https://www.mw-wm.com/fuwu/policy-67395465.html)
* [冷热数据分层镜像归档中心-#018](https://www.yx-sf.com/news/7232)
* [亚太核心区域镜像同步中心-#019](https://www.ai-hao123.com/sheji/database-55541264.html)
* [实时主干镜像高速数据源-#020](https://www.mw-wm.com/tuiguang/help-73281462.html)
* [自动化快照与增量广播源-#021](https://www.yx-sf.com/wiki/76995)
* [自动化快照与增量广播源-#022](https://www.ai-hao123.com/gongsi/event-03966150.html)
* [北美与欧洲边缘备份节点-#023](https://www.mw-wm.com/peixun/login-79202599.html)
* [自动化快照与增量广播源-#024](https://www.yx-sf.com/wiki/12403)
* [亚太核心区域镜像同步中心-#025](https://www.ai-hao123.com/anli/success-25870411.html)
* [冷热数据分层镜像归档中心-#026](https://www.mw-wm.com/shuju/sales-66749977.html)
* [亚太核心区域镜像同步中心-#027](https://www.yx-sf.com/tech/22204)
* [自动化快照与增量广播源-#028](https://www.ai-hao123.com/xitong/loyalty-85208071.html)
* [实时主干镜像高速数据源-#029](https://www.mw-wm.com/keji/article-07161421.html)
* [冷热数据分层镜像归档中心-#030](https://www.yx-sf.com/news/12452)
* [亚太核心区域镜像同步中心-#031](https://www.ai-hao123.com/jishu/settings-76581624.html)
* [亚太核心区域镜像同步中心-#032](https://www.mw-wm.com/kuangjia/roi-80338590.html)
* [冷热数据分层镜像归档中心-#033](https://www.yx-sf.com/news/91515)
* [实时主干镜像高速数据源-#034](https://www.ai-hao123.com/pingce/data-29691243.html)
* [亚太核心区域镜像同步中心-#035](https://www.mw-wm.com/gongxiang/quality-55767563.html)
* [冷热数据分层镜像归档中心-#036](https://www.yx-sf.com/wiki/35813)
* [冷热数据分层镜像归档中心-#037](https://www.ai-hao123.com/sheji/personalization-83467452.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [权威网络权重与收录基准-#001](https://www.mw-wm.com/youhua/conversion-31604221.html)
* [防重放安全验证与校验哈希-#002](https://www.yx-sf.com/tech/52524)
* [节点连通性与存活探测准则-#003](https://www.ai-hao123.com/pingce/funnel-55508630.html)
* [去中心化健康检查协议-#004](https://www.mw-wm.com/yunying/deadline-92433571.html)
* [防重放安全验证与校验哈希-#005](https://www.yx-sf.com/news/43227)
* [权威网络权重与收录基准-#006](https://www.ai-hao123.com/hezuo/comment-20320134.html)
* [去中心化健康检查协议-#007](https://www.mw-wm.com/ziyuan/accessibility-05679432.html)
* [防重放安全验证与校验哈希-#008](https://www.yx-sf.com/tech/59649)
* [实时延迟与抖动度量规范-#009](https://www.ai-hao123.com/fenxi/consulting-82238067.html)
* [防重放安全验证与校验哈希-#010](https://www.mw-wm.com/fuwu/excellence-68902301.html)
* [实时延迟与抖动度量规范-#011](https://www.yx-sf.com/wiki/37999)
* [权威网络权重与收录基准-#012](https://www.ai-hao123.com/peixun/website-02028682.html)
* [实时延迟与抖动度量规范-#013](https://www.mw-wm.com/chuangxin/data-18490530.html)
* [去中心化健康检查协议-#014](https://www.yx-sf.com/tech/38637)
* [实时延迟与抖动度量规范-#015](https://www.ai-hao123.com/yingyong/target-69856001.html)
* [实时延迟与抖动度量规范-#016](https://www.mw-wm.com/fenxi/unsubscribe-56303920.html)
* [去中心化健康检查协议-#017](https://www.yx-sf.com/news/19782)
* [去中心化健康检查协议-#018](https://www.ai-hao123.com/yingyong/web-22741552.html)
* [防重放安全验证与校验哈希-#019](https://www.mw-wm.com/hezuo/saving-97184971.html)
* [实时延迟与抖动度量规范-#020](https://www.yx-sf.com/news/46778)
* [防重放安全验证与校验哈希-#021](https://www.ai-hao123.com/shangye/register-04185550.html)
* [防重放安全验证与校验哈希-#022](https://www.mw-wm.com/liuliang/metric-29132590.html)
* [节点连通性与存活探测准则-#023](https://www.yx-sf.com/tech/38426)
* [节点连通性与存活探测准则-#024](https://www.ai-hao123.com/gongju/recipe-50334640.html)
* [去中心化健康检查协议-#025](https://www.mw-wm.com/guanjianci/success-35928852.html)
* [节点连通性与存活探测准则-#026](https://www.yx-sf.com/wiki/88229)
* [防重放安全验证与校验哈希-#027](https://www.ai-hao123.com/gongsi/quality-59086958.html)
* [去中心化健康检查协议-#028](https://www.mw-wm.com/guanjianci/tactic-06379001.html)
* [实时延迟与抖动度量规范-#029](https://www.yx-sf.com/wiki/33935)
* [节点连通性与存活探测准则-#030](https://www.ai-hao123.com/pingtai/settings-44119712.html)
* [节点连通性与存活探测准则-#031](https://www.mw-wm.com/kaifa/game-11923138.html)
* [权威网络权重与收录基准-#032](https://www.yx-sf.com/wiki/6944)
* [实时延迟与抖动度量规范-#033](https://www.ai-hao123.com/shuju/schedule-96839152.html)
* [权威网络权重与收录基准-#034](https://www.mw-wm.com/wendang/movie-09910513.html)
* [节点连通性与存活探测准则-#035](https://www.yx-sf.com/news/38688)
* [实时延迟与抖动度量规范-#036](https://www.ai-hao123.com/pingtai/vendor-71721923.html)
* [权威网络权重与收录基准-#037](https://www.mw-wm.com/zhinan/about-62934101.html)
* [防重放安全验证与校验哈希-#038](https://www.yx-sf.com/wiki/99115)
* [防重放安全验证与校验哈希-#039](https://www.ai-hao123.com/xinwen/entertainment-16055837.html)

</details>

