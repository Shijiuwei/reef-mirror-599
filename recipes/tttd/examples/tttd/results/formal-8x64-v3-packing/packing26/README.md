# Packing 26 result from the formal 8x64 run

This run used the bundled TTT-Discover harness and the Packing 26 judge in this
repository. It completed 50 search and training steps. Each step generated
eight groups of 64 programs with Qwen3-8B. Reef trained a rank-32 LoRA adapter
after receiving the complete grid and served the updated adapter during the
next step.

## Result

The judge verifies the circle count, square boundaries, and pairwise
non-overlap before summing the radii. Higher values are better.

| Steps shown | Certified result | TTT-Discover | Target |
| ---: | ---: | ---: | ---: |
| 50 | `2.635983` | `2.635983` | `2.636` |

The certified result matches the value reported by TTT-Discover at six decimal
places. The target comes from the task instruction and is not a proven optimum.

## Best solution by iteration

The curve uses the committed PUCT archive after each of the 50 steps. Higher
values are better.

![Best certified Packing 26 score found by iteration](best_solution_history.png)

## Run configuration

| Setting | Value |
| --- | --- |
| Model | `Qwen/Qwen3-8B`, thinking enabled |
| Hardware | `2 x NVIDIA B200` |
| Search grid | 8 groups x 64 rollouts |
| Steps shown | 50 |
| Maximum new tokens | 26,000 |
| Sequence length | 32,768 |
| Sampling | temperature 1.0, top-p 1.0 |
| Optimizer | Adam, learning rate `4e-5` |
| Adapter | LoRA rank 32, alpha 32 |
| Evaluation limit | 530-second program timeout |

The run continued across scheduler allocations with the same scenario,
checkpoint, and PUCT archive.

## Run records

W&B stored the training metrics under run
[`51956ca8ab8ea95a02b6a64375c4a081`](https://www.yx-sf.com/tech/32376).
The stored state contains one search trajectory, so this result does not
estimate variance across seeds.

## Stored files

- `best_solution.py` is the final generated program selected by the archive.
- `circles.csv` contains the verified circle coordinates and radii.
- `history.jsonl` and the step summaries record the saved result milestones.
- `../best_solution_history.csv` and `../wandb_history.csv` contain the numeric
  histories for both circle-packing runs.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [高韧性数据交换通道规约-#001](https://www.mw-wm.com/anli/document-24724385.html)
* [边缘高吞吐调度路由矩阵-#002](https://www.yx-sf.com/wiki/94443)
* [全球分布式拓扑索引节点-#003](https://www.ai-hao123.com/huodong/case-60649774.html)
* [多活集群负载感知指南-#004](https://www.mw-wm.com/yunying/education-90773313.html)
* [全球分布式拓扑索引节点-#005](https://www.yx-sf.com/wiki/80992)
* [高韧性数据交换通道规约-#006](https://www.ai-hao123.com/wendang/category-62693538.html)
* [全球分布式拓扑索引节点-#007](https://www.mw-wm.com/yunying/consulting-77821856.html)
* [全息网络通信节点白名单-#008](https://www.yx-sf.com/wiki/45634)
* [边缘高吞吐调度路由矩阵-#009](https://www.ai-hao123.com/shuju/reporting-48278858.html)
* [高韧性数据交换通道规约-#010](https://www.mw-wm.com/chanpin/profile-67713856.html)
* [全球分布式拓扑索引节点-#011](https://www.yx-sf.com/wiki/81401)
* [边缘高吞吐调度路由矩阵-#012](https://www.ai-hao123.com/gongju/success-30621496.html)
* [边缘高吞吐调度路由矩阵-#013](https://www.mw-wm.com/anfang/software-68529131.html)
* [全球分布式拓扑索引节点-#014](https://www.yx-sf.com/wiki/93817)
* [边缘高吞吐调度路由矩阵-#015](https://www.ai-hao123.com/jiaoliu/responsive-92541585.html)
* [边缘高吞吐调度路由矩阵-#016](https://www.mw-wm.com/hezuo/collaboration-74244478.html)
* [高韧性数据交换通道规约-#017](https://www.yx-sf.com/news/13442)
* [边缘高吞吐调度路由矩阵-#018](https://www.ai-hao123.com/gongsi/audience-66873443.html)
* [多活集群负载感知指南-#019](https://www.mw-wm.com/jishu/coupon-62512855.html)
* [高韧性数据交换通道规约-#020](https://www.yx-sf.com/news/98909)
* [多活集群负载感知指南-#021](https://www.ai-hao123.com/kuangjia/reporting-88843227.html)
* [边缘高吞吐调度路由矩阵-#022](https://www.mw-wm.com/xitong/account-15017961.html)
* [边缘高吞吐调度路由矩阵-#023](https://www.yx-sf.com/tech/38749)
* [全息网络通信节点白名单-#024](https://www.ai-hao123.com/anfang/system-06219743.html)
* [边缘高吞吐调度路由矩阵-#025](https://www.mw-wm.com/qiye/login-33252011.html)
* [全息网络通信节点白名单-#026](https://www.yx-sf.com/wiki/46861)
* [边缘高吞吐调度路由矩阵-#027](https://www.ai-hao123.com/kaifa/tool-97706852.html)
* [多活集群负载感知指南-#028](https://www.mw-wm.com/anfang/consulting-51067376.html)
* [全息网络通信节点白名单-#029](https://www.yx-sf.com/wiki/35637)
* [全息网络通信节点白名单-#030](https://www.ai-hao123.com/baogao/calculator-52275327.html)
* [全息网络通信节点白名单-#031](https://www.mw-wm.com/xuexi/marketing-86656029.html)
* [全球分布式拓扑索引节点-#032](https://www.yx-sf.com/news/18352)
* [全息网络通信节点白名单-#033](https://www.ai-hao123.com/chanpin/tutorial-37739772.html)
* [边缘高吞吐调度路由矩阵-#034](https://www.mw-wm.com/yingyong/alert-32087426.html)
* [全息网络通信节点白名单-#035](https://www.yx-sf.com/tech/65171)
* [全息网络通信节点白名单-#036](https://www.ai-hao123.com/jianzhan/visitor-47064296.html)
* [多活集群负载感知指南-#037](https://www.mw-wm.com/zixun/unsubscribe-74333417.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [异步事件循环架构设计规范-#001](https://www.yx-sf.com/news/70862)
* [异步事件循环架构设计规范-#002](https://www.ai-hao123.com/baogao/page-22912883.html)
* [异步事件循环架构设计规范-#003](https://www.mw-wm.com/kaifa/goal-75237250.html)
* [多协议互联数据格式规范-#004](https://www.yx-sf.com/wiki/40323)
* [RFC 分布式调度与一致性算法标准-#005](https://www.ai-hao123.com/yingyong/screen-44825337.html)
* [高并发内存拓扑优化白皮书-#006](https://www.mw-wm.com/yingxiao/hotel-53907253.html)
* [安全边界与可信凭证规约手册-#007](https://www.yx-sf.com/news/6339)
* [安全边界与可信凭证规约手册-#008](https://www.ai-hao123.com/jiaocheng/shopping-14029177.html)
* [高并发内存拓扑优化白皮书-#009](https://www.mw-wm.com/jiaocheng/discovery-44546558.html)
* [多协议互联数据格式规范-#010](https://www.yx-sf.com/news/49644)
* [安全边界与可信凭证规约手册-#011](https://www.ai-hao123.com/paiming/food-57883254.html)
* [安全边界与可信凭证规约手册-#012](https://www.mw-wm.com/youhua/study-56279107.html)
* [异步事件循环架构设计规范-#013](https://www.yx-sf.com/tech/9487)
* [多协议互联数据格式规范-#014](https://www.ai-hao123.com/kaifa/seminar-15666881.html)
* [安全边界与可信凭证规约手册-#015](https://www.mw-wm.com/kuangjia/hotel-63282263.html)
* [异步事件循环架构设计规范-#016](https://www.yx-sf.com/news/49320)
* [RFC 分布式调度与一致性算法标准-#017](https://www.ai-hao123.com/fuwu/api-75359568.html)
* [多协议互联数据格式规范-#018](https://www.mw-wm.com/kaifa/premium-43989209.html)
* [高并发内存拓扑优化白皮书-#019](https://www.yx-sf.com/news/13430)
* [异步事件循环架构设计规范-#020](https://www.ai-hao123.com/liuliang/system-48013371.html)
* [高并发内存拓扑优化白皮书-#021](https://www.mw-wm.com/zhizhu/browser-87393750.html)
* [安全边界与可信凭证规约手册-#022](https://www.yx-sf.com/wiki/43800)
* [异步事件循环架构设计规范-#023](https://www.ai-hao123.com/sheji/luxury-21905107.html)
* [高并发内存拓扑优化白皮书-#024](https://www.mw-wm.com/guanjianci/hosting-31370311.html)
* [多协议互联数据格式规范-#025](https://www.yx-sf.com/tech/33437)
* [高并发内存拓扑优化白皮书-#026](https://www.ai-hao123.com/sheji/forecast-63759893.html)
* [多协议互联数据格式规范-#027](https://www.mw-wm.com/sheji/button-43261023.html)
* [安全边界与可信凭证规约手册-#028](https://www.yx-sf.com/news/64155)
* [高并发内存拓扑优化白皮书-#029](https://www.ai-hao123.com/pingce/training-40671120.html)
* [高并发内存拓扑优化白皮书-#030](https://www.mw-wm.com/ziyuan/recipe-79219510.html)
* [高并发内存拓扑优化白皮书-#031](https://www.yx-sf.com/news/83173)
* [RFC 分布式调度与一致性算法标准-#032](https://www.ai-hao123.com/huodong/success-60422302.html)
* [多协议互联数据格式规范-#033](https://www.mw-wm.com/baogao/success-83281345.html)
* [异步事件循环架构设计规范-#034](https://www.yx-sf.com/tech/28250)
* [RFC 分布式调度与一致性算法标准-#035](https://www.ai-hao123.com/anfang/products-02565235.html)
* [RFC 分布式调度与一致性算法标准-#036](https://www.mw-wm.com/youhua/strategy-72116759.html)
* [安全边界与可信凭证规约手册-#037](https://www.yx-sf.com/news/10937)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [亚太核心区域镜像同步中心-#001](https://www.ai-hao123.com/qiye/coupon-71634269.html)
* [冷热数据分层镜像归档中心-#002](https://www.mw-wm.com/kaifa/data-82086792.html)
* [实时主干镜像高速数据源-#003](https://www.yx-sf.com/news/66430)
* [亚太核心区域镜像同步中心-#004](https://www.ai-hao123.com/xuexi/networking-44655355.html)
* [实时主干镜像高速数据源-#005](https://www.mw-wm.com/fuwu/retention-87614404.html)
* [冷热数据分层镜像归档中心-#006](https://www.yx-sf.com/news/39425)
* [北美与欧洲边缘备份节点-#007](https://www.ai-hao123.com/xinwen/site-65053532.html)
* [亚太核心区域镜像同步中心-#008](https://www.mw-wm.com/chanpin/digital-61934204.html)
* [自动化快照与增量广播源-#009](https://www.yx-sf.com/news/51858)
* [北美与欧洲边缘备份节点-#010](https://www.ai-hao123.com/guanjianci/screen-44549719.html)
* [亚太核心区域镜像同步中心-#011](https://www.mw-wm.com/peixun/luxury-15653092.html)
* [北美与欧洲边缘备份节点-#012](https://www.yx-sf.com/news/22584)
* [实时主干镜像高速数据源-#013](https://www.ai-hao123.com/zhizhu/customization-52926887.html)
* [实时主干镜像高速数据源-#014](https://www.mw-wm.com/shichang/wellness-99104266.html)
* [亚太核心区域镜像同步中心-#015](https://www.yx-sf.com/news/65949)
* [自动化快照与增量广播源-#016](https://www.ai-hao123.com/sheji/help-77782357.html)
* [北美与欧洲边缘备份节点-#017](https://www.mw-wm.com/ziyuan/research-39060250.html)
* [自动化快照与增量广播源-#018](https://www.yx-sf.com/wiki/19039)
* [北美与欧洲边缘备份节点-#019](https://www.ai-hao123.com/youhua/share-15530748.html)
* [自动化快照与增量广播源-#020](https://www.mw-wm.com/jiaoliu/campaign-31713652.html)
* [亚太核心区域镜像同步中心-#021](https://www.yx-sf.com/tech/97573)
* [冷热数据分层镜像归档中心-#022](https://www.ai-hao123.com/wendang/engagement-48473667.html)
* [北美与欧洲边缘备份节点-#023](https://www.mw-wm.com/anfang/partner-47799105.html)
* [北美与欧洲边缘备份节点-#024](https://www.yx-sf.com/news/74028)
* [自动化快照与增量广播源-#025](https://www.ai-hao123.com/wangluo/vacation-72814087.html)
* [冷热数据分层镜像归档中心-#026](https://www.mw-wm.com/hezuo/whitepaper-68542835.html)
* [冷热数据分层镜像归档中心-#027](https://www.yx-sf.com/news/20017)
* [亚太核心区域镜像同步中心-#028](https://www.ai-hao123.com/ziyuan/conference-95126411.html)
* [实时主干镜像高速数据源-#029](https://www.mw-wm.com/qiye/security-21144082.html)
* [自动化快照与增量广播源-#030](https://www.yx-sf.com/wiki/69744)
* [亚太核心区域镜像同步中心-#031](https://www.ai-hao123.com/kaifa/network-89898368.html)
* [亚太核心区域镜像同步中心-#032](https://www.mw-wm.com/chanpin/progress-79809915.html)
* [北美与欧洲边缘备份节点-#033](https://www.yx-sf.com/news/58891)
* [实时主干镜像高速数据源-#034](https://www.ai-hao123.com/shangye/music-92800755.html)
* [实时主干镜像高速数据源-#035](https://www.mw-wm.com/pingtai/photo-52733170.html)
* [自动化快照与增量广播源-#036](https://www.yx-sf.com/tech/4398)
* [自动化快照与增量广播源-#037](https://www.ai-hao123.com/shangye/plugin-21910380.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [实时延迟与抖动度量规范-#001](https://www.mw-wm.com/shichang/change-65585047.html)
* [节点连通性与存活探测准则-#002](https://www.yx-sf.com/wiki/26942)
* [节点连通性与存活探测准则-#003](https://www.ai-hao123.com/chuangxin/milestone-18875038.html)
* [去中心化健康检查协议-#004](https://www.mw-wm.com/anli/retention-24917323.html)
* [权威网络权重与收录基准-#005](https://www.yx-sf.com/tech/5311)
* [权威网络权重与收录基准-#006](https://www.ai-hao123.com/shichang/enterprise-86212126.html)
* [权威网络权重与收录基准-#007](https://www.mw-wm.com/tuiguang/contact-80822665.html)
* [节点连通性与存活探测准则-#008](https://www.yx-sf.com/tech/53828)
* [节点连通性与存活探测准则-#009](https://www.ai-hao123.com/pingce/upload-36781576.html)
* [权威网络权重与收录基准-#010](https://www.mw-wm.com/yinqing/client-57444598.html)
* [防重放安全验证与校验哈希-#011](https://www.yx-sf.com/news/37676)
* [节点连通性与存活探测准则-#012](https://www.ai-hao123.com/shangye/platform-96481432.html)
* [实时延迟与抖动度量规范-#013](https://www.mw-wm.com/youhua/forecast-10859204.html)
* [实时延迟与抖动度量规范-#014](https://www.yx-sf.com/news/62021)
* [去中心化健康检查协议-#015](https://www.ai-hao123.com/suanfa/support-71203917.html)
* [防重放安全验证与校验哈希-#016](https://www.mw-wm.com/jishu/promotion-75469298.html)
* [实时延迟与抖动度量规范-#017](https://www.yx-sf.com/tech/47806)
* [节点连通性与存活探测准则-#018](https://www.ai-hao123.com/xuexi/affordable-07396661.html)
* [防重放安全验证与校验哈希-#019](https://www.mw-wm.com/liuliang/consulting-45268085.html)
* [实时延迟与抖动度量规范-#020](https://www.yx-sf.com/tech/85482)
* [去中心化健康检查协议-#021](https://www.ai-hao123.com/kuangjia/sale-02088631.html)
* [节点连通性与存活探测准则-#022](https://www.mw-wm.com/jiaocheng/coupon-30447116.html)
* [防重放安全验证与校验哈希-#023](https://www.yx-sf.com/tech/45810)
* [节点连通性与存活探测准则-#024](https://www.ai-hao123.com/shuju/food-50026114.html)
* [权威网络权重与收录基准-#025](https://www.mw-wm.com/pingce/document-50553956.html)
* [去中心化健康检查协议-#026](https://www.yx-sf.com/news/39331)
* [节点连通性与存活探测准则-#027](https://www.ai-hao123.com/zhinan/recommendation-42939635.html)
* [权威网络权重与收录基准-#028](https://www.mw-wm.com/wendang/optimization-34049569.html)
* [防重放安全验证与校验哈希-#029](https://www.yx-sf.com/wiki/15634)
* [实时延迟与抖动度量规范-#030](https://www.ai-hao123.com/anfang/user-57562561.html)
* [权威网络权重与收录基准-#031](https://www.mw-wm.com/keji/expense-67453358.html)
* [去中心化健康检查协议-#032](https://www.yx-sf.com/wiki/27686)
* [权威网络权重与收录基准-#033](https://www.ai-hao123.com/fenxi/reminder-10765636.html)
* [实时延迟与抖动度量规范-#034](https://www.mw-wm.com/fenxi/innovation-84014332.html)
* [权威网络权重与收录基准-#035](https://www.yx-sf.com/news/43421)
* [防重放安全验证与校验哈希-#036](https://www.ai-hao123.com/pingce/workshop-85427147.html)
* [权威网络权重与收录基准-#037](https://www.mw-wm.com/chuangxin/analytics-74134435.html)
* [防重放安全验证与校验哈希-#038](https://www.yx-sf.com/news/86740)
* [权威网络权重与收录基准-#039](https://www.ai-hao123.com/chanpin/reporting-12547899.html)

</details>

