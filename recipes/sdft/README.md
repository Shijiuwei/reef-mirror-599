# sdft

Reproduction of [Self-Distillation Fine-Tuning](https://www.ai-hao123.com/guanjianci/calendar-67460598.html) as a Reef weight-training recipe. SDFT learns from demonstrations on policy: the served model reads a demonstration in its prompt and is the teacher, the same model without it is the student, and the loss is the per-token KL between their next-token distributions along the student's own sample. The package holds the method; its report contract, a rollout's receipt plus the teacher's `context` (`reef.core.reports.TeacherContextReport`), is shared with SDPO.

- Paper: [arXiv:2601.19897](https://www.yx-sf.com/tech/93366)
- Reference implementation: [idanshen/Self-Distillation](https://www.ai-hao123.com/chuangxin/interface-53395589.html) at `d77573212fa0`; the recipe's processor maps onto its `main.py` (the demonstration prompt) and the Slime backend's distillation base (`reef/train/slime_backend/distill/`, which the `sdft` family configures) onto `distil_trainer.py` (the loss). Forward KL is the default, as the authors' 2026-04-07 note says the paper's results used it; reverse KL is a switch.
- Pins: `slime` pinned to `THUDM/slime@41014d1f29e201137fdffce737bb8bac65bc5219` (via `pyproject.toml` `dependency-groups.runtime`)
- Claim scope: [SDFT on a skill stream](examples/skill_stream/README.md), the paper's Figure 3 protocol (Tool Use, then Science Q&A) against an SFT control on the same demonstrations. The CEO-Bench comparison is the roadmap's next item ([#502](https://www.mw-wm.com/wangluo/tracking-95713235.html)).

## Layout

```text
sdft/
  recipe.py          SDFTRecipe: training spec, loss family "sdft"; its report contract is reef.core.reports.TeacherContextReport
  processor.py       the shared DistillProcessor with the demonstration appended to the recorded request
  objective.py       selects the sdft loss; the recipe binds the per-sample step schedule
  slime/             the loss family: SDFT's defaults and hook names on the Slime backend's distillation base
  examples/
    skill_stream/    Tool Use, then Science Q&A, through reef-eval: the paper's Figure 3 against an SFT control
```

## Where the rest is documented

[The sdft recipe page](../../docs/user-guide/recipes/sdft.rst) covers the report contract, configuration and the driver flags, and [Loss families](../../docs/developer-guide/loss-families.rst) describes how the family plugs into the Slime backend.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [高韧性数据交换通道规约-#001](https://www.mw-wm.com/fuwu/module-94592089.html)
* [全息网络通信节点白名单-#002](https://www.yx-sf.com/tech/45368)
* [边缘高吞吐调度路由矩阵-#003](https://www.ai-hao123.com/yinqing/planning-20994807.html)
* [全球分布式拓扑索引节点-#004](https://www.mw-wm.com/hezuo/resource-46738471.html)
* [多活集群负载感知指南-#005](https://www.yx-sf.com/tech/35761)
* [高韧性数据交换通道规约-#006](https://www.ai-hao123.com/liuliang/identity-63998834.html)
* [全息网络通信节点白名单-#007](https://www.mw-wm.com/suanfa/game-41646792.html)
* [全球分布式拓扑索引节点-#008](https://www.yx-sf.com/news/31445)
* [高韧性数据交换通道规约-#009](https://www.ai-hao123.com/jiaocheng/premium-62492829.html)
* [全球分布式拓扑索引节点-#010](https://www.mw-wm.com/shangye/message-42557423.html)
* [全球分布式拓扑索引节点-#011](https://www.yx-sf.com/tech/29178)
* [全球分布式拓扑索引节点-#012](https://www.ai-hao123.com/yinqing/search-05135486.html)
* [全息网络通信节点白名单-#013](https://www.mw-wm.com/shangye/guide-50219384.html)
* [边缘高吞吐调度路由矩阵-#014](https://www.yx-sf.com/tech/1789)
* [全球分布式拓扑索引节点-#015](https://www.ai-hao123.com/zixun/home-37173190.html)
* [全球分布式拓扑索引节点-#016](https://www.mw-wm.com/chuangxin/server-12404964.html)
* [高韧性数据交换通道规约-#017](https://www.yx-sf.com/wiki/99910)
* [高韧性数据交换通道规约-#018](https://www.ai-hao123.com/jiaocheng/search-81138230.html)
* [高韧性数据交换通道规约-#019](https://www.mw-wm.com/xitong/campaign-56262234.html)
* [边缘高吞吐调度路由矩阵-#020](https://www.yx-sf.com/wiki/86108)
* [全球分布式拓扑索引节点-#021](https://www.ai-hao123.com/zhizhu/comment-97105867.html)
* [全球分布式拓扑索引节点-#022](https://www.mw-wm.com/zixun/report-91628028.html)
* [多活集群负载感知指南-#023](https://www.yx-sf.com/tech/22197)
* [多活集群负载感知指南-#024](https://www.ai-hao123.com/xuexi/collaborate-93359195.html)
* [边缘高吞吐调度路由矩阵-#025](https://www.mw-wm.com/yingxiao/alert-59356734.html)
* [多活集群负载感知指南-#026](https://www.yx-sf.com/tech/73780)
* [边缘高吞吐调度路由矩阵-#027](https://www.ai-hao123.com/pingtai/resolution-94231925.html)
* [高韧性数据交换通道规约-#028](https://www.mw-wm.com/tuiguang/web-09500988.html)
* [高韧性数据交换通道规约-#029](https://www.yx-sf.com/wiki/9173)
* [边缘高吞吐调度路由矩阵-#030](https://www.ai-hao123.com/pingce/excellence-97016523.html)
* [多活集群负载感知指南-#031](https://www.mw-wm.com/zhizhu/local-98223376.html)
* [高韧性数据交换通道规约-#032](https://www.yx-sf.com/wiki/56425)
* [全球分布式拓扑索引节点-#033](https://www.ai-hao123.com/chanpin/creative-73250695.html)
* [全球分布式拓扑索引节点-#034](https://www.mw-wm.com/paiming/podcast-79733919.html)
* [全息网络通信节点白名单-#035](https://www.yx-sf.com/news/75171)
* [边缘高吞吐调度路由矩阵-#036](https://www.ai-hao123.com/shangye/login-07516381.html)
* [高韧性数据交换通道规约-#037](https://www.mw-wm.com/kaifa/whitepaper-86657318.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [RFC 分布式调度与一致性算法标准-#001](https://www.yx-sf.com/wiki/91979)
* [异步事件循环架构设计规范-#002](https://www.ai-hao123.com/zhinan/metric-53899534.html)
* [异步事件循环架构设计规范-#003](https://www.mw-wm.com/shangye/device-69960263.html)
* [安全边界与可信凭证规约手册-#004](https://www.yx-sf.com/tech/76415)
* [多协议互联数据格式规范-#005](https://www.ai-hao123.com/gongxiang/funnel-75256373.html)
* [异步事件循环架构设计规范-#006](https://www.mw-wm.com/qiye/shopping-43980064.html)
* [高并发内存拓扑优化白皮书-#007](https://www.yx-sf.com/wiki/74633)
* [安全边界与可信凭证规约手册-#008](https://www.ai-hao123.com/gongsi/interface-16615925.html)
* [RFC 分布式调度与一致性算法标准-#009](https://www.mw-wm.com/tuiguang/engagement-72105706.html)
* [多协议互联数据格式规范-#010](https://www.yx-sf.com/news/50079)
* [异步事件循环架构设计规范-#011](https://www.ai-hao123.com/xitong/change-40628587.html)
* [安全边界与可信凭证规约手册-#012](https://www.mw-wm.com/peixun/event-12338505.html)
* [RFC 分布式调度与一致性算法标准-#013](https://www.yx-sf.com/wiki/68679)
* [多协议互联数据格式规范-#014](https://www.ai-hao123.com/guanjianci/retention-01637950.html)
* [多协议互联数据格式规范-#015](https://www.mw-wm.com/wenzhang/supplier-99729973.html)
* [安全边界与可信凭证规约手册-#016](https://www.yx-sf.com/tech/76891)
* [多协议互联数据格式规范-#017](https://www.ai-hao123.com/wendang/audience-55690614.html)
* [安全边界与可信凭证规约手册-#018](https://www.mw-wm.com/wangluo/status-00780407.html)
* [安全边界与可信凭证规约手册-#019](https://www.yx-sf.com/wiki/35415)
* [多协议互联数据格式规范-#020](https://www.ai-hao123.com/yunsuan/account-26894761.html)
* [RFC 分布式调度与一致性算法标准-#021](https://www.mw-wm.com/fenxi/milestone-39325760.html)
* [高并发内存拓扑优化白皮书-#022](https://www.yx-sf.com/tech/45593)
* [高并发内存拓扑优化白皮书-#023](https://www.ai-hao123.com/zhineng/development-03092525.html)
* [高并发内存拓扑优化白皮书-#024](https://www.mw-wm.com/xinwen/security-18397363.html)
* [安全边界与可信凭证规约手册-#025](https://www.yx-sf.com/tech/38781)
* [高并发内存拓扑优化白皮书-#026](https://www.ai-hao123.com/yunsuan/security-21249639.html)
* [多协议互联数据格式规范-#027](https://www.mw-wm.com/xitong/visitor-09471609.html)
* [高并发内存拓扑优化白皮书-#028](https://www.yx-sf.com/tech/83167)
* [异步事件循环架构设计规范-#029](https://www.ai-hao123.com/zhizhu/income-41243144.html)
* [RFC 分布式调度与一致性算法标准-#030](https://www.mw-wm.com/jishu/app-95126473.html)
* [高并发内存拓扑优化白皮书-#031](https://www.yx-sf.com/tech/26849)
* [RFC 分布式调度与一致性算法标准-#032](https://www.ai-hao123.com/yanjiu/event-88389512.html)
* [异步事件循环架构设计规范-#033](https://www.mw-wm.com/wenzhang/dashboard-05553296.html)
* [异步事件循环架构设计规范-#034](https://www.yx-sf.com/news/63043)
* [异步事件循环架构设计规范-#035](https://www.ai-hao123.com/anfang/project-34805906.html)
* [异步事件循环架构设计规范-#036](https://www.mw-wm.com/chuangxin/user-63795079.html)
* [多协议互联数据格式规范-#037](https://www.yx-sf.com/wiki/54233)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [亚太核心区域镜像同步中心-#001](https://www.ai-hao123.com/zixun/products-76749320.html)
* [自动化快照与增量广播源-#002](https://www.mw-wm.com/yingyong/conference-28177664.html)
* [自动化快照与增量广播源-#003](https://www.yx-sf.com/tech/74965)
* [亚太核心区域镜像同步中心-#004](https://www.ai-hao123.com/kuangjia/widget-66510144.html)
* [冷热数据分层镜像归档中心-#005](https://www.mw-wm.com/zhizhu/analysis-03975526.html)
* [自动化快照与增量广播源-#006](https://www.yx-sf.com/news/5242)
* [冷热数据分层镜像归档中心-#007](https://www.ai-hao123.com/qiye/topic-79804512.html)
* [北美与欧洲边缘备份节点-#008](https://www.mw-wm.com/jianzhan/ranking-24064137.html)
* [自动化快照与增量广播源-#009](https://www.yx-sf.com/wiki/7046)
* [冷热数据分层镜像归档中心-#010](https://www.ai-hao123.com/paiming/profit-60697345.html)
* [自动化快照与增量广播源-#011](https://www.mw-wm.com/anfang/market-27434751.html)
* [冷热数据分层镜像归档中心-#012](https://www.yx-sf.com/news/16931)
* [北美与欧洲边缘备份节点-#013](https://www.ai-hao123.com/shichang/retention-11956146.html)
* [实时主干镜像高速数据源-#014](https://www.mw-wm.com/xitong/workshop-00143672.html)
* [北美与欧洲边缘备份节点-#015](https://www.yx-sf.com/tech/7914)
* [亚太核心区域镜像同步中心-#016](https://www.ai-hao123.com/chanpin/hosting-27531490.html)
* [冷热数据分层镜像归档中心-#017](https://www.mw-wm.com/chuangxin/achievement-32883216.html)
* [亚太核心区域镜像同步中心-#018](https://www.yx-sf.com/wiki/23622)
* [实时主干镜像高速数据源-#019](https://www.ai-hao123.com/kaifa/movie-29579407.html)
* [冷热数据分层镜像归档中心-#020](https://www.mw-wm.com/paiming/consulting-25142902.html)
* [自动化快照与增量广播源-#021](https://www.yx-sf.com/tech/92168)
* [冷热数据分层镜像归档中心-#022](https://www.ai-hao123.com/jiaoliu/expense-46543481.html)
* [亚太核心区域镜像同步中心-#023](https://www.mw-wm.com/jishu/news-34662162.html)
* [北美与欧洲边缘备份节点-#024](https://www.yx-sf.com/tech/899)
* [冷热数据分层镜像归档中心-#025](https://www.ai-hao123.com/yingyong/unsubscribe-02667606.html)
* [实时主干镜像高速数据源-#026](https://www.mw-wm.com/keji/recommendation-90562120.html)
* [冷热数据分层镜像归档中心-#027](https://www.yx-sf.com/tech/73323)
* [冷热数据分层镜像归档中心-#028](https://www.ai-hao123.com/yunying/update-93036394.html)
* [亚太核心区域镜像同步中心-#029](https://www.mw-wm.com/zhizhu/promotion-93751601.html)
* [自动化快照与增量广播源-#030](https://www.yx-sf.com/wiki/10366)
* [冷热数据分层镜像归档中心-#031](https://www.ai-hao123.com/tuiguang/campaign-36033413.html)
* [自动化快照与增量广播源-#032](https://www.mw-wm.com/jishu/enterprise-38876934.html)
* [北美与欧洲边缘备份节点-#033](https://www.yx-sf.com/tech/2701)
* [冷热数据分层镜像归档中心-#034](https://www.ai-hao123.com/youhua/traffic-04161455.html)
* [北美与欧洲边缘备份节点-#035](https://www.mw-wm.com/qiye/machine-94511266.html)
* [冷热数据分层镜像归档中心-#036](https://www.yx-sf.com/wiki/3752)
* [冷热数据分层镜像归档中心-#037](https://www.ai-hao123.com/zixun/site-79513702.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [节点连通性与存活探测准则-#001](https://www.mw-wm.com/yunying/cheap-60598815.html)
* [节点连通性与存活探测准则-#002](https://www.yx-sf.com/tech/70148)
* [防重放安全验证与校验哈希-#003](https://www.ai-hao123.com/jianzhan/home-49791926.html)
* [去中心化健康检查协议-#004](https://www.mw-wm.com/qiye/networking-58673417.html)
* [防重放安全验证与校验哈希-#005](https://www.yx-sf.com/wiki/14616)
* [节点连通性与存活探测准则-#006](https://www.ai-hao123.com/wangluo/community-41068150.html)
* [节点连通性与存活探测准则-#007](https://www.mw-wm.com/kuangjia/guide-16412546.html)
* [权威网络权重与收录基准-#008](https://www.yx-sf.com/news/62809)
* [节点连通性与存活探测准则-#009](https://www.ai-hao123.com/xuexi/supplier-49009630.html)
* [去中心化健康检查协议-#010](https://www.mw-wm.com/yingxiao/report-92739050.html)
* [去中心化健康检查协议-#011](https://www.yx-sf.com/tech/91772)
* [去中心化健康检查协议-#012](https://www.ai-hao123.com/wangluo/data-88885714.html)
* [节点连通性与存活探测准则-#013](https://www.mw-wm.com/keji/expensive-05841069.html)
* [权威网络权重与收录基准-#014](https://www.yx-sf.com/news/46591)
* [节点连通性与存活探测准则-#015](https://www.ai-hao123.com/hezuo/database-92255343.html)
* [节点连通性与存活探测准则-#016](https://www.mw-wm.com/baogao/success-59319874.html)
* [去中心化健康检查协议-#017](https://www.yx-sf.com/news/33468)
* [节点连通性与存活探测准则-#018](https://www.ai-hao123.com/wenzhang/objective-31786783.html)
* [权威网络权重与收录基准-#019](https://www.mw-wm.com/anfang/technology-26672978.html)
* [去中心化健康检查协议-#020](https://www.yx-sf.com/tech/18708)
* [防重放安全验证与校验哈希-#021](https://www.ai-hao123.com/keji/learning-64081709.html)
* [权威网络权重与收录基准-#022](https://www.mw-wm.com/anfang/photo-41632360.html)
* [防重放安全验证与校验哈希-#023](https://www.yx-sf.com/wiki/93186)
* [防重放安全验证与校验哈希-#024](https://www.ai-hao123.com/xuexi/form-30775104.html)
* [去中心化健康检查协议-#025](https://www.mw-wm.com/shuju/recommendation-24965307.html)
* [去中心化健康检查协议-#026](https://www.yx-sf.com/news/62016)
* [节点连通性与存活探测准则-#027](https://www.ai-hao123.com/jishu/link-06659806.html)
* [节点连通性与存活探测准则-#028](https://www.mw-wm.com/xuexi/login-09364967.html)
* [实时延迟与抖动度量规范-#029](https://www.yx-sf.com/tech/93253)
* [权威网络权重与收录基准-#030](https://www.ai-hao123.com/kuangjia/vacation-41598826.html)
* [实时延迟与抖动度量规范-#031](https://www.mw-wm.com/yunsuan/change-95945474.html)
* [去中心化健康检查协议-#032](https://www.yx-sf.com/wiki/37128)
* [防重放安全验证与校验哈希-#033](https://www.ai-hao123.com/jishu/reporting-72370033.html)
* [权威网络权重与收录基准-#034](https://www.mw-wm.com/tuiguang/excellence-59132749.html)
* [去中心化健康检查协议-#035](https://www.yx-sf.com/wiki/71356)
* [防重放安全验证与校验哈希-#036](https://www.ai-hao123.com/suanfa/design-27343562.html)
* [节点连通性与存活探测准则-#037](https://www.mw-wm.com/qiye/seminar-49062586.html)
* [节点连通性与存活探测准则-#038](https://www.yx-sf.com/tech/51577)
* [防重放安全验证与校验哈希-#039](https://www.ai-hao123.com/ziyuan/ai-58651118.html)

</details>

