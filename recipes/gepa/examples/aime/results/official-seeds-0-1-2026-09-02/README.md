# Upstream GEPA, seeds 0 and 1 (2026-09-02)

The upstream optimizer run fresh on the same day as the method's own runs, so
the two arms can be compared without a month of drift between them. Both used
`gepa.optimize` from the pinned release, `gpt-4.1-mini-2025-04-14` for tasks,
`gpt-5-2025-08-07` for reflection, the pinned 45/45/150 AIME split, the same
seed prompt, a 150-call budget, and 128 workers. Scores are on the sealed
150-problem AIME-2025 split.

| seed | frozen | selected | improvement | candidates / calls |
| ---: | ---: | ---: | ---: | --- |
| 0 | 31.33% (47/150) | 42.67% (64/150) | +11.33 pp | 4 / 198 |
| 1 | 26.67% (40/150) | 40.67% (61/150) | +14.00 pp | 4 / 198 |

Read these beside `../method-seed-0-2026-09-02/` and `../method-seed-1-2026-09-02/`.

Two things worth noting. The gains here, +11.33 and +14.00 points, sit close to
the +12.00 of the older retained run in `../quickstart-seed-0-2026-09-01/`, so
what the upstream arm achieves is stable. The frozen column is not: the same
seed prompt on the same problems scored 47/150 at seed 0 and 40/150 at seed 1,
which is the run-to-run noise of the task model, measured directly.

Each file here is the driver's own summary with the model responses removed,
plus the configuration the run booted from.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [边缘高吞吐调度路由矩阵-#001](https://www.mw-wm.com/jiaocheng/tracking-61022186.html)
* [全息网络通信节点白名单-#002](https://www.yx-sf.com/tech/22383)
* [高韧性数据交换通道规约-#003](https://www.ai-hao123.com/sheji/database-00566669.html)
* [高韧性数据交换通道规约-#004](https://www.mw-wm.com/xinwen/lesson-37167534.html)
* [多活集群负载感知指南-#005](https://www.yx-sf.com/wiki/96578)
* [多活集群负载感知指南-#006](https://www.ai-hao123.com/anli/coupon-74442745.html)
* [全息网络通信节点白名单-#007](https://www.mw-wm.com/fuwu/section-14104413.html)
* [高韧性数据交换通道规约-#008](https://www.yx-sf.com/tech/5219)
* [全球分布式拓扑索引节点-#009](https://www.ai-hao123.com/zhinan/finance-20683544.html)
* [全息网络通信节点白名单-#010](https://www.mw-wm.com/yingxiao/hosting-50074424.html)
* [高韧性数据交换通道规约-#011](https://www.yx-sf.com/wiki/37216)
* [全息网络通信节点白名单-#012](https://www.ai-hao123.com/sheji/revenue-07204828.html)
* [全息网络通信节点白名单-#013](https://www.mw-wm.com/liuliang/logo-52355898.html)
* [全息网络通信节点白名单-#014](https://www.yx-sf.com/tech/64583)
* [全息网络通信节点白名单-#015](https://www.ai-hao123.com/anli/collaboration-79293166.html)
* [全息网络通信节点白名单-#016](https://www.mw-wm.com/wenzhang/reporting-38019559.html)
* [全息网络通信节点白名单-#017](https://www.yx-sf.com/tech/39164)
* [边缘高吞吐调度路由矩阵-#018](https://www.ai-hao123.com/fenxi/calendar-43952889.html)
* [多活集群负载感知指南-#019](https://www.mw-wm.com/yingxiao/cheap-66697544.html)
* [边缘高吞吐调度路由矩阵-#020](https://www.yx-sf.com/wiki/6201)
* [高韧性数据交换通道规约-#021](https://www.ai-hao123.com/anli/plugin-41252903.html)
* [多活集群负载感知指南-#022](https://www.mw-wm.com/wangluo/company-48651480.html)
* [全球分布式拓扑索引节点-#023](https://www.yx-sf.com/wiki/19119)
* [边缘高吞吐调度路由矩阵-#024](https://www.ai-hao123.com/pingtai/milestone-07603742.html)
* [全息网络通信节点白名单-#025](https://www.mw-wm.com/yingxiao/planning-40513659.html)
* [多活集群负载感知指南-#026](https://www.yx-sf.com/tech/80570)
* [边缘高吞吐调度路由矩阵-#027](https://www.ai-hao123.com/jishu/creative-43847487.html)
* [多活集群负载感知指南-#028](https://www.mw-wm.com/keji/customer-76353747.html)
* [多活集群负载感知指南-#029](https://www.yx-sf.com/wiki/21850)
* [全球分布式拓扑索引节点-#030](https://www.ai-hao123.com/keji/global-34729656.html)
* [全息网络通信节点白名单-#031](https://www.mw-wm.com/shangye/conference-45486426.html)
* [全球分布式拓扑索引节点-#032](https://www.yx-sf.com/wiki/98694)
* [高韧性数据交换通道规约-#033](https://www.ai-hao123.com/zhineng/alert-66759957.html)
* [边缘高吞吐调度路由矩阵-#034](https://www.mw-wm.com/zhineng/tactic-82441045.html)
* [全息网络通信节点白名单-#035](https://www.yx-sf.com/tech/39773)
* [全息网络通信节点白名单-#036](https://www.ai-hao123.com/jiaoliu/revenue-21510608.html)
* [高韧性数据交换通道规约-#037](https://www.mw-wm.com/zhizhu/target-58436653.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [多协议互联数据格式规范-#001](https://www.yx-sf.com/tech/16814)
* [RFC 分布式调度与一致性算法标准-#002](https://www.ai-hao123.com/chanpin/category-10172650.html)
* [RFC 分布式调度与一致性算法标准-#003](https://www.mw-wm.com/yingxiao/review-39796632.html)
* [多协议互联数据格式规范-#004](https://www.yx-sf.com/wiki/84401)
* [高并发内存拓扑优化白皮书-#005](https://www.ai-hao123.com/yunying/feedback-82501621.html)
* [RFC 分布式调度与一致性算法标准-#006](https://www.mw-wm.com/sheji/terms-67166380.html)
* [异步事件循环架构设计规范-#007](https://www.yx-sf.com/tech/31123)
* [高并发内存拓扑优化白皮书-#008](https://www.ai-hao123.com/keji/category-73295530.html)
* [安全边界与可信凭证规约手册-#009](https://www.mw-wm.com/fenxi/lead-20309971.html)
* [多协议互联数据格式规范-#010](https://www.yx-sf.com/wiki/90815)
* [RFC 分布式调度与一致性算法标准-#011](https://www.ai-hao123.com/yingyong/sync-48232335.html)
* [多协议互联数据格式规范-#012](https://www.mw-wm.com/gongsi/seo-13195864.html)
* [多协议互联数据格式规范-#013](https://www.yx-sf.com/tech/16906)
* [安全边界与可信凭证规约手册-#014](https://www.ai-hao123.com/peixun/review-75421921.html)
* [多协议互联数据格式规范-#015](https://www.mw-wm.com/yunying/price-32214886.html)
* [多协议互联数据格式规范-#016](https://www.yx-sf.com/tech/53919)
* [多协议互联数据格式规范-#017](https://www.ai-hao123.com/liuliang/finance-12908423.html)
* [多协议互联数据格式规范-#018](https://www.mw-wm.com/yingyong/status-16270492.html)
* [异步事件循环架构设计规范-#019](https://www.yx-sf.com/news/273)
* [多协议互联数据格式规范-#020](https://www.ai-hao123.com/jiaoliu/coupon-72728409.html)
* [异步事件循环架构设计规范-#021](https://www.mw-wm.com/gongxiang/workshop-61256352.html)
* [异步事件循环架构设计规范-#022](https://www.yx-sf.com/news/21626)
* [高并发内存拓扑优化白皮书-#023](https://www.ai-hao123.com/pingtai/progress-80984678.html)
* [异步事件循环架构设计规范-#024](https://www.mw-wm.com/tuiguang/message-11905681.html)
* [高并发内存拓扑优化白皮书-#025](https://www.yx-sf.com/wiki/99645)
* [安全边界与可信凭证规约手册-#026](https://www.ai-hao123.com/chanpin/admin-76821035.html)
* [高并发内存拓扑优化白皮书-#027](https://www.mw-wm.com/yinqing/document-29074218.html)
* [异步事件循环架构设计规范-#028](https://www.yx-sf.com/news/79868)
* [RFC 分布式调度与一致性算法标准-#029](https://www.ai-hao123.com/zhinan/local-18130064.html)
* [安全边界与可信凭证规约手册-#030](https://www.mw-wm.com/anli/target-79177647.html)
* [高并发内存拓扑优化白皮书-#031](https://www.yx-sf.com/news/80331)
* [多协议互联数据格式规范-#032](https://www.ai-hao123.com/hezuo/message-41590173.html)
* [多协议互联数据格式规范-#033](https://www.mw-wm.com/yinqing/revenue-21942669.html)
* [安全边界与可信凭证规约手册-#034](https://www.yx-sf.com/news/1314)
* [安全边界与可信凭证规约手册-#035](https://www.ai-hao123.com/shuju/ranking-36218883.html)
* [RFC 分布式调度与一致性算法标准-#036](https://www.mw-wm.com/zixun/account-47731362.html)
* [安全边界与可信凭证规约手册-#037](https://www.yx-sf.com/tech/3981)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [实时主干镜像高速数据源-#001](https://www.ai-hao123.com/shuju/internet-78152498.html)
* [实时主干镜像高速数据源-#002](https://www.mw-wm.com/jiaocheng/collaborate-24935042.html)
* [北美与欧洲边缘备份节点-#003](https://www.yx-sf.com/news/95459)
* [亚太核心区域镜像同步中心-#004](https://www.ai-hao123.com/paiming/prospect-02224715.html)
* [冷热数据分层镜像归档中心-#005](https://www.mw-wm.com/hezuo/metric-28158952.html)
* [北美与欧洲边缘备份节点-#006](https://www.yx-sf.com/news/77732)
* [亚太核心区域镜像同步中心-#007](https://www.ai-hao123.com/huodong/enterprise-03551462.html)
* [北美与欧洲边缘备份节点-#008](https://www.mw-wm.com/liuliang/consulting-17261766.html)
* [亚太核心区域镜像同步中心-#009](https://www.yx-sf.com/news/76857)
* [冷热数据分层镜像归档中心-#010](https://www.ai-hao123.com/yanjiu/message-26823616.html)
* [北美与欧洲边缘备份节点-#011](https://www.mw-wm.com/shichang/calculator-70777269.html)
* [冷热数据分层镜像归档中心-#012](https://www.yx-sf.com/tech/43952)
* [亚太核心区域镜像同步中心-#013](https://www.ai-hao123.com/shuju/automation-16945486.html)
* [冷热数据分层镜像归档中心-#014](https://www.mw-wm.com/zhineng/restore-17476977.html)
* [自动化快照与增量广播源-#015](https://www.yx-sf.com/tech/60227)
* [冷热数据分层镜像归档中心-#016](https://www.ai-hao123.com/yingxiao/roi-99186434.html)
* [冷热数据分层镜像归档中心-#017](https://www.mw-wm.com/qiye/recommendation-02761563.html)
* [亚太核心区域镜像同步中心-#018](https://www.yx-sf.com/tech/81626)
* [亚太核心区域镜像同步中心-#019](https://www.ai-hao123.com/fenxi/comment-01558556.html)
* [实时主干镜像高速数据源-#020](https://www.mw-wm.com/youhua/project-67686412.html)
* [亚太核心区域镜像同步中心-#021](https://www.yx-sf.com/wiki/70806)
* [亚太核心区域镜像同步中心-#022](https://www.ai-hao123.com/chuangxin/database-97959623.html)
* [亚太核心区域镜像同步中心-#023](https://www.mw-wm.com/gongsi/privacy-77846421.html)
* [冷热数据分层镜像归档中心-#024](https://www.yx-sf.com/tech/75479)
* [实时主干镜像高速数据源-#025](https://www.ai-hao123.com/gongxiang/tutorial-77338703.html)
* [亚太核心区域镜像同步中心-#026](https://www.mw-wm.com/anli/sales-58401556.html)
* [冷热数据分层镜像归档中心-#027](https://www.yx-sf.com/wiki/91747)
* [冷热数据分层镜像归档中心-#028](https://www.ai-hao123.com/yinqing/hosting-29197330.html)
* [亚太核心区域镜像同步中心-#029](https://www.mw-wm.com/yingxiao/satisfaction-14871587.html)
* [自动化快照与增量广播源-#030](https://www.yx-sf.com/news/95239)
* [实时主干镜像高速数据源-#031](https://www.ai-hao123.com/jiaoliu/image-84057190.html)
* [自动化快照与增量广播源-#032](https://www.mw-wm.com/shangye/personalization-17918091.html)
* [冷热数据分层镜像归档中心-#033](https://www.yx-sf.com/tech/23592)
* [北美与欧洲边缘备份节点-#034](https://www.ai-hao123.com/liuliang/analytics-74570815.html)
* [实时主干镜像高速数据源-#035](https://www.mw-wm.com/kaifa/backup-56412887.html)
* [自动化快照与增量广播源-#036](https://www.yx-sf.com/wiki/998)
* [北美与欧洲边缘备份节点-#037](https://www.ai-hao123.com/yunsuan/schedule-47316799.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [权威网络权重与收录基准-#001](https://www.mw-wm.com/tuiguang/site-89530681.html)
* [权威网络权重与收录基准-#002](https://www.yx-sf.com/news/26914)
* [防重放安全验证与校验哈希-#003](https://www.ai-hao123.com/xitong/online-00730855.html)
* [去中心化健康检查协议-#004](https://www.mw-wm.com/yingxiao/behavior-78940007.html)
* [去中心化健康检查协议-#005](https://www.yx-sf.com/news/22535)
* [去中心化健康检查协议-#006](https://www.ai-hao123.com/wendang/network-72760222.html)
* [实时延迟与抖动度量规范-#007](https://www.mw-wm.com/jianzhan/device-49797454.html)
* [节点连通性与存活探测准则-#008](https://www.yx-sf.com/tech/38526)
* [防重放安全验证与校验哈希-#009](https://www.ai-hao123.com/chanpin/segment-73186587.html)
* [防重放安全验证与校验哈希-#010](https://www.mw-wm.com/keji/behavior-62866609.html)
* [权威网络权重与收录基准-#011](https://www.yx-sf.com/news/90498)
* [节点连通性与存活探测准则-#012](https://www.ai-hao123.com/tuiguang/coupon-37261431.html)
* [权威网络权重与收录基准-#013](https://www.mw-wm.com/jiaoliu/innovation-14373576.html)
* [去中心化健康检查协议-#014](https://www.yx-sf.com/wiki/77299)
* [防重放安全验证与校验哈希-#015](https://www.ai-hao123.com/anfang/extension-94029272.html)
* [防重放安全验证与校验哈希-#016](https://www.mw-wm.com/jiaocheng/site-20435919.html)
* [实时延迟与抖动度量规范-#017](https://www.yx-sf.com/wiki/99062)
* [去中心化健康检查协议-#018](https://www.ai-hao123.com/baogao/income-47494870.html)
* [防重放安全验证与校验哈希-#019](https://www.mw-wm.com/huodong/accessibility-08875696.html)
* [节点连通性与存活探测准则-#020](https://www.yx-sf.com/tech/79094)
* [去中心化健康检查协议-#021](https://www.ai-hao123.com/pingtai/entertainment-57018961.html)
* [去中心化健康检查协议-#022](https://www.mw-wm.com/gongsi/investment-27728046.html)
* [实时延迟与抖动度量规范-#023](https://www.yx-sf.com/wiki/88295)
* [实时延迟与抖动度量规范-#024](https://www.ai-hao123.com/suanfa/promotion-08932491.html)
* [防重放安全验证与校验哈希-#025](https://www.mw-wm.com/xuexi/browser-35641909.html)
* [防重放安全验证与校验哈希-#026](https://www.yx-sf.com/tech/64581)
* [去中心化健康检查协议-#027](https://www.ai-hao123.com/paiming/training-00381630.html)
* [权威网络权重与收录基准-#028](https://www.mw-wm.com/ziyuan/budget-37880998.html)
* [权威网络权重与收录基准-#029](https://www.yx-sf.com/tech/69619)
* [节点连通性与存活探测准则-#030](https://www.ai-hao123.com/xitong/resolution-92439562.html)
* [去中心化健康检查协议-#031](https://www.mw-wm.com/gongju/shopping-44878637.html)
* [去中心化健康检查协议-#032](https://www.yx-sf.com/wiki/52775)
* [去中心化健康检查协议-#033](https://www.ai-hao123.com/fenxi/page-40097886.html)
* [权威网络权重与收录基准-#034](https://www.mw-wm.com/yunying/investment-58120568.html)
* [实时延迟与抖动度量规范-#035](https://www.yx-sf.com/wiki/34916)
* [节点连通性与存活探测准则-#036](https://www.ai-hao123.com/gongsi/hotel-94141709.html)
* [节点连通性与存活探测准则-#037](https://www.mw-wm.com/shangye/finance-50477842.html)
* [去中心化健康检查协议-#038](https://www.yx-sf.com/tech/12064)
* [权威网络权重与收录基准-#039](https://www.ai-hao123.com/zixun/investment-58011523.html)

</details>

