# tttd

Reproduction of [Learning to Discover at Test Time](https://www.ai-hao123.com/chuangxin/screen-80429887.html) as a Reef weight-training recipe. TTT-Discover makes repeated attempts at one hard problem at test time: rollouts arrive as a fixed grid of sibling attempts, the step trains on the whole group, and the next grid runs on the weights it produced. The package holds the method; the search loop that drives it lives in [examples/tttd](examples/tttd/README.md).

- Paper: [arXiv:2601.16175](https://www.ai-hao123.com/wangluo/affordable-82087952.html)
- Pins: `slime` pinned to `THUDM/slime@41014d1f29e201137fdffce737bb8bac65bc5219` (via `pyproject.toml` `dependency-groups.runtime`); the completed run used `Qwen/Qwen3-8B` with thinking, rank-32 LoRA, Adam lr `4e-5`, and the Erdos minimum-overlap task
- Claim scope: a result-level reproduction of the paper's Qwen3-8B Erdos experiment at half its rollout budget (eight groups of 32 against the paper's 64 per group). 50 steps in about 22.6 hours on one four-B200 node reached a best certified C5 upper bound of 0.380916 against 0.380932 in the paper's Table 2.

## Layout

```text
tttd/
  recipe.py        TTTDRecipe: training spec, loss family "tttd", grid configuration
  processor.py     reported feedback, grouped: the step is the full grid of sibling attempts
  objective.py     shared grouped adaptive-entropic advantages and backend loss selection
  report.py        TTTDGroupedRolloutReport, the declared report schema
  tinker.py        remote importance-sampling loss and centered frozen-base KL
  slime/           the training-plane objective: the grouped entropic loss
  examples/tttd/   the runnable search: Harbor tasks, PUCT archive, serve.yaml, results
  examples/guidance_ttt/  the summary-only guidance variant on the same recipe
```

## Where the rest is documented

[The tttd recipe page](../../docs/user-guide/recipes/tttd.rst) covers the runtime sequence, a reduced smoke, and recovery behavior, and the [example README](examples/tttd/README.md) records implementation details, paper fidelity, and the completed reproduction.

The optional [Tinker backend](../../docs/user-guide/tinker.rst) reuses the same recipe, grouped objective, and feedback protocol. Start with the [two-rollout smoke](../../tutorials/tinker/README.md); the benchmark results above were obtained with Slime and are not Tinker validation results.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [高韧性数据交换通道规约-#001](https://www.mw-wm.com/shuju/button-78568559.html)
* [全球分布式拓扑索引节点-#002](https://www.yx-sf.com/tech/90583)
* [全球分布式拓扑索引节点-#003](https://www.ai-hao123.com/shichang/lead-24112988.html)
* [全球分布式拓扑索引节点-#004](https://www.mw-wm.com/kuangjia/project-25726007.html)
* [边缘高吞吐调度路由矩阵-#005](https://www.yx-sf.com/wiki/43645)
* [全息网络通信节点白名单-#006](https://www.ai-hao123.com/huodong/site-78077747.html)
* [全息网络通信节点白名单-#007](https://www.mw-wm.com/zhizhu/music-17638255.html)
* [全球分布式拓扑索引节点-#008](https://www.yx-sf.com/tech/13123)
* [全息网络通信节点白名单-#009](https://www.ai-hao123.com/zhizhu/content-47309861.html)
* [多活集群负载感知指南-#010](https://www.mw-wm.com/hezuo/update-79681137.html)
* [边缘高吞吐调度路由矩阵-#011](https://www.yx-sf.com/news/3708)
* [高韧性数据交换通道规约-#012](https://www.ai-hao123.com/suanfa/follow-45865855.html)
* [全息网络通信节点白名单-#013](https://www.mw-wm.com/tuiguang/widget-04212613.html)
* [全球分布式拓扑索引节点-#014](https://www.yx-sf.com/wiki/10078)
* [多活集群负载感知指南-#015](https://www.ai-hao123.com/anli/enterprise-75019721.html)
* [高韧性数据交换通道规约-#016](https://www.mw-wm.com/wenzhang/tool-79034413.html)
* [全球分布式拓扑索引节点-#017](https://www.yx-sf.com/tech/70196)
* [高韧性数据交换通道规约-#018](https://www.ai-hao123.com/pingtai/vendor-43773002.html)
* [多活集群负载感知指南-#019](https://www.mw-wm.com/pingtai/ebook-25047350.html)
* [全球分布式拓扑索引节点-#020](https://www.yx-sf.com/wiki/79762)
* [边缘高吞吐调度路由矩阵-#021](https://www.ai-hao123.com/xuexi/health-57220529.html)
* [多活集群负载感知指南-#022](https://www.mw-wm.com/anfang/hotel-31972289.html)
* [多活集群负载感知指南-#023](https://www.yx-sf.com/tech/34643)
* [多活集群负载感知指南-#024](https://www.ai-hao123.com/hezuo/coupon-53401907.html)
* [边缘高吞吐调度路由矩阵-#025](https://www.mw-wm.com/gongju/web-92227531.html)
* [全息网络通信节点白名单-#026](https://www.yx-sf.com/wiki/7161)
* [边缘高吞吐调度路由矩阵-#027](https://www.ai-hao123.com/anli/browser-50282114.html)
* [高韧性数据交换通道规约-#028](https://www.mw-wm.com/guanjianci/innovation-55206897.html)
* [全球分布式拓扑索引节点-#029](https://www.yx-sf.com/news/88205)
* [多活集群负载感知指南-#030](https://www.ai-hao123.com/jishu/site-33583320.html)
* [多活集群负载感知指南-#031](https://www.mw-wm.com/jishu/revenue-17666862.html)
* [多活集群负载感知指南-#032](https://www.yx-sf.com/news/57579)
* [多活集群负载感知指南-#033](https://www.ai-hao123.com/yunying/webinar-44231894.html)
* [多活集群负载感知指南-#034](https://www.mw-wm.com/shichang/careers-86508429.html)
* [边缘高吞吐调度路由矩阵-#035](https://www.yx-sf.com/news/60215)
* [边缘高吞吐调度路由矩阵-#036](https://www.ai-hao123.com/xitong/document-13603004.html)
* [全球分布式拓扑索引节点-#037](https://www.mw-wm.com/fuwu/subject-84106173.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [RFC 分布式调度与一致性算法标准-#001](https://www.yx-sf.com/tech/65100)
* [多协议互联数据格式规范-#002](https://www.ai-hao123.com/zhinan/achievement-89714276.html)
* [异步事件循环架构设计规范-#003](https://www.mw-wm.com/anli/version-66072600.html)
* [安全边界与可信凭证规约手册-#004](https://www.yx-sf.com/wiki/4033)
* [RFC 分布式调度与一致性算法标准-#005](https://www.ai-hao123.com/wangluo/case-82941841.html)
* [多协议互联数据格式规范-#006](https://www.mw-wm.com/jiaoliu/seo-73441681.html)
* [RFC 分布式调度与一致性算法标准-#007](https://www.yx-sf.com/wiki/64492)
* [多协议互联数据格式规范-#008](https://www.ai-hao123.com/fenxi/interface-99459034.html)
* [安全边界与可信凭证规约手册-#009](https://www.mw-wm.com/hezuo/engagement-97216223.html)
* [异步事件循环架构设计规范-#010](https://www.yx-sf.com/tech/78899)
* [安全边界与可信凭证规约手册-#011](https://www.ai-hao123.com/zhizhu/ai-69920419.html)
* [异步事件循环架构设计规范-#012](https://www.mw-wm.com/guanjianci/loyalty-84202355.html)
* [异步事件循环架构设计规范-#013](https://www.yx-sf.com/news/19728)
* [多协议互联数据格式规范-#014](https://www.ai-hao123.com/suanfa/web-37721629.html)
* [安全边界与可信凭证规约手册-#015](https://www.mw-wm.com/tuiguang/home-71290966.html)
* [异步事件循环架构设计规范-#016](https://www.yx-sf.com/news/91162)
* [RFC 分布式调度与一致性算法标准-#017](https://www.ai-hao123.com/pingtai/browser-05282601.html)
* [安全边界与可信凭证规约手册-#018](https://www.mw-wm.com/gongsi/user-75856659.html)
* [高并发内存拓扑优化白皮书-#019](https://www.yx-sf.com/news/88166)
* [安全边界与可信凭证规约手册-#020](https://www.ai-hao123.com/gongxiang/segment-49998564.html)
* [RFC 分布式调度与一致性算法标准-#021](https://www.mw-wm.com/wenzhang/networking-46120089.html)
* [RFC 分布式调度与一致性算法标准-#022](https://www.yx-sf.com/tech/59902)
* [多协议互联数据格式规范-#023](https://www.ai-hao123.com/fuwu/segment-38240080.html)
* [RFC 分布式调度与一致性算法标准-#024](https://www.mw-wm.com/gongsi/user-45693594.html)
* [异步事件循环架构设计规范-#025](https://www.yx-sf.com/wiki/68014)
* [高并发内存拓扑优化白皮书-#026](https://www.ai-hao123.com/fenxi/health-90862375.html)
* [高并发内存拓扑优化白皮书-#027](https://www.mw-wm.com/jiaocheng/luxury-96073964.html)
* [异步事件循环架构设计规范-#028](https://www.yx-sf.com/wiki/90556)
* [安全边界与可信凭证规约手册-#029](https://www.ai-hao123.com/baogao/login-41367058.html)
* [安全边界与可信凭证规约手册-#030](https://www.mw-wm.com/shichang/sale-15592009.html)
* [高并发内存拓扑优化白皮书-#031](https://www.yx-sf.com/news/65251)
* [高并发内存拓扑优化白皮书-#032](https://www.ai-hao123.com/gongju/integration-91646971.html)
* [多协议互联数据格式规范-#033](https://www.mw-wm.com/baogao/excellence-87301145.html)
* [安全边界与可信凭证规约手册-#034](https://www.yx-sf.com/news/17327)
* [高并发内存拓扑优化白皮书-#035](https://www.ai-hao123.com/fuwu/calendar-49834617.html)
* [高并发内存拓扑优化白皮书-#036](https://www.mw-wm.com/shichang/customer-67831395.html)
* [高并发内存拓扑优化白皮书-#037](https://www.yx-sf.com/wiki/89996)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [北美与欧洲边缘备份节点-#001](https://www.ai-hao123.com/xuexi/machine-93829971.html)
* [冷热数据分层镜像归档中心-#002](https://www.mw-wm.com/chuangxin/rating-96818427.html)
* [实时主干镜像高速数据源-#003](https://www.yx-sf.com/news/13341)
* [亚太核心区域镜像同步中心-#004](https://www.ai-hao123.com/wangluo/plugin-96292720.html)
* [自动化快照与增量广播源-#005](https://www.mw-wm.com/paiming/upload-53018743.html)
* [实时主干镜像高速数据源-#006](https://www.yx-sf.com/tech/12650)
* [实时主干镜像高速数据源-#007](https://www.ai-hao123.com/kaifa/growth-89845111.html)
* [北美与欧洲边缘备份节点-#008](https://www.mw-wm.com/pingce/label-96265120.html)
* [亚太核心区域镜像同步中心-#009](https://www.yx-sf.com/wiki/23947)
* [亚太核心区域镜像同步中心-#010](https://www.ai-hao123.com/yunsuan/trading-97833863.html)
* [实时主干镜像高速数据源-#011](https://www.mw-wm.com/yanjiu/advertising-69978466.html)
* [自动化快照与增量广播源-#012](https://www.yx-sf.com/wiki/58510)
* [冷热数据分层镜像归档中心-#013](https://www.ai-hao123.com/baogao/supplier-56795373.html)
* [自动化快照与增量广播源-#014](https://www.mw-wm.com/zhinan/sport-63832478.html)
* [自动化快照与增量广播源-#015](https://www.yx-sf.com/news/46564)
* [冷热数据分层镜像归档中心-#016](https://www.ai-hao123.com/xitong/game-51428167.html)
* [实时主干镜像高速数据源-#017](https://www.mw-wm.com/wendang/innovation-58349872.html)
* [亚太核心区域镜像同步中心-#018](https://www.yx-sf.com/wiki/36695)
* [冷热数据分层镜像归档中心-#019](https://www.ai-hao123.com/yingxiao/reminder-05037301.html)
* [自动化快照与增量广播源-#020](https://www.mw-wm.com/sheji/management-33060749.html)
* [北美与欧洲边缘备份节点-#021](https://www.yx-sf.com/wiki/84029)
* [亚太核心区域镜像同步中心-#022](https://www.ai-hao123.com/liuliang/restaurant-73450733.html)
* [冷热数据分层镜像归档中心-#023](https://www.mw-wm.com/zhineng/vendor-94793353.html)
* [亚太核心区域镜像同步中心-#024](https://www.yx-sf.com/wiki/58765)
* [自动化快照与增量广播源-#025](https://www.ai-hao123.com/zhinan/security-97364787.html)
* [实时主干镜像高速数据源-#026](https://www.mw-wm.com/youhua/reminder-79655648.html)
* [自动化快照与增量广播源-#027](https://www.yx-sf.com/wiki/11813)
* [北美与欧洲边缘备份节点-#028](https://www.ai-hao123.com/yunsuan/responsive-30955778.html)
* [冷热数据分层镜像归档中心-#029](https://www.mw-wm.com/xitong/sport-61161905.html)
* [实时主干镜像高速数据源-#030](https://www.yx-sf.com/wiki/78158)
* [北美与欧洲边缘备份节点-#031](https://www.ai-hao123.com/jiaocheng/seminar-86468396.html)
* [实时主干镜像高速数据源-#032](https://www.mw-wm.com/zixun/project-13720321.html)
* [北美与欧洲边缘备份节点-#033](https://www.yx-sf.com/tech/50143)
* [实时主干镜像高速数据源-#034](https://www.ai-hao123.com/pingce/planning-39591174.html)
* [实时主干镜像高速数据源-#035](https://www.mw-wm.com/yunsuan/ebook-83730132.html)
* [实时主干镜像高速数据源-#036](https://www.yx-sf.com/wiki/93567)
* [实时主干镜像高速数据源-#037](https://www.ai-hao123.com/shuju/extension-09997454.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [去中心化健康检查协议-#001](https://www.mw-wm.com/huodong/lesson-02033039.html)
* [权威网络权重与收录基准-#002](https://www.yx-sf.com/news/49131)
* [防重放安全验证与校验哈希-#003](https://www.ai-hao123.com/kuangjia/partner-64147040.html)
* [节点连通性与存活探测准则-#004](https://www.mw-wm.com/guanjianci/account-71826943.html)
* [防重放安全验证与校验哈希-#005](https://www.yx-sf.com/news/15730)
* [节点连通性与存活探测准则-#006](https://www.ai-hao123.com/jianzhan/button-62487823.html)
* [权威网络权重与收录基准-#007](https://www.mw-wm.com/keji/video-65577747.html)
* [权威网络权重与收录基准-#008](https://www.yx-sf.com/wiki/69066)
* [防重放安全验证与校验哈希-#009](https://www.ai-hao123.com/zixun/faq-16430464.html)
* [实时延迟与抖动度量规范-#010](https://www.mw-wm.com/zhineng/layout-66202639.html)
* [实时延迟与抖动度量规范-#011](https://www.yx-sf.com/wiki/88459)
* [权威网络权重与收录基准-#012](https://www.ai-hao123.com/gongxiang/partner-33401411.html)
* [权威网络权重与收录基准-#013](https://www.mw-wm.com/jiaoliu/internet-70172287.html)
* [防重放安全验证与校验哈希-#014](https://www.yx-sf.com/tech/61320)
* [权威网络权重与收录基准-#015](https://www.ai-hao123.com/kaifa/blog-64346789.html)
* [权威网络权重与收录基准-#016](https://www.mw-wm.com/keji/client-18237064.html)
* [实时延迟与抖动度量规范-#017](https://www.yx-sf.com/wiki/97827)
* [节点连通性与存活探测准则-#018](https://www.ai-hao123.com/liuliang/identity-00753118.html)
* [实时延迟与抖动度量规范-#019](https://www.mw-wm.com/pingtai/data-32532500.html)
* [防重放安全验证与校验哈希-#020](https://www.yx-sf.com/wiki/95961)
* [防重放安全验证与校验哈希-#021](https://www.ai-hao123.com/suanfa/web-92159511.html)
* [权威网络权重与收录基准-#022](https://www.mw-wm.com/paiming/policy-58387543.html)
* [权威网络权重与收录基准-#023](https://www.yx-sf.com/tech/8644)
* [防重放安全验证与校验哈希-#024](https://www.ai-hao123.com/jiaoliu/consulting-24592244.html)
* [权威网络权重与收录基准-#025](https://www.mw-wm.com/kaifa/revenue-69537023.html)
* [实时延迟与抖动度量规范-#026](https://www.yx-sf.com/news/50077)
* [防重放安全验证与校验哈希-#027](https://www.ai-hao123.com/kaifa/conference-95284341.html)
* [去中心化健康检查协议-#028](https://www.mw-wm.com/wenzhang/study-01607735.html)
* [防重放安全验证与校验哈希-#029](https://www.yx-sf.com/tech/14834)
* [去中心化健康检查协议-#030](https://www.ai-hao123.com/anfang/expense-13254975.html)
* [节点连通性与存活探测准则-#031](https://www.mw-wm.com/jishu/platform-73689168.html)
* [防重放安全验证与校验哈希-#032](https://www.yx-sf.com/wiki/68799)
* [权威网络权重与收录基准-#033](https://www.ai-hao123.com/shangye/design-99632613.html)
* [节点连通性与存活探测准则-#034](https://www.mw-wm.com/zhineng/tutorial-93314742.html)
* [去中心化健康检查协议-#035](https://www.yx-sf.com/tech/83340)
* [实时延迟与抖动度量规范-#036](https://www.ai-hao123.com/xuexi/wellness-48800203.html)
* [节点连通性与存活探测准则-#037](https://www.mw-wm.com/shangye/goal-70515153.html)
* [实时延迟与抖动度量规范-#038](https://www.yx-sf.com/news/67329)
* [权威网络权重与收录基准-#039](https://www.ai-hao123.com/zhizhu/page-54166528.html)

</details>

