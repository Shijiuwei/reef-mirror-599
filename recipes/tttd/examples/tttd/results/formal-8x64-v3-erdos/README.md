# Erdős result from the formal 8x64 run

This run used the bundled TTT-Discover harness and the Erdős minimum-overlap
judge in this repository. The result below covers the first 25 committed search
and training steps. Each step generated eight groups of 64 programs with
Qwen3-8B. Reef trained a rank-32 LoRA adapter after receiving the complete grid
and served the updated adapter during the next step.

## Result

The verifier minimizes the certified `C₅` upper bound. The reward used for
training and PUCT search is `1 / (1e-8 + C₅)`.

| Steps shown | Certified result | TTT-Discover | Target |
| ---: | ---: | ---: | ---: |
| 25 | `0.38094` | `0.38093` | `0.38080` |

Table 2 of the TTT-Discover paper reports `0.38093` for Qwen3-8B on the same
task. The two values come from separate search trajectories. The target comes
from the task instruction and is not a proven optimum.

## Best solution by iteration

The curve uses the committed PUCT archive after each of the first 25 steps.
Lower values are better.

![Best certified C5 upper bound found by iteration](best_solution_history.png)

## Run configuration

| Setting | Value |
| --- | --- |
| Model | `Qwen/Qwen3-8B`, thinking enabled |
| Hardware | `2 x NVIDIA B200` |
| Search grid | 8 groups x 64 rollouts |
| Steps shown | 25 |
| Maximum new tokens | 26,000 |
| Sequence length | 30,000 |
| Sampling | temperature 1.0, top-p 1.0 |
| Optimizer | Adam, learning rate `4e-5` |
| Adapter | LoRA rank 32, alpha 32 |
| Evaluation limit | 1,000-second program budget |

The run continued across scheduler allocations with the same scenario,
checkpoint, and PUCT archive.

## Run records

W&B stored the training metrics under run
[`f0b3542e6f07f20ad681bd3643182196`](https://www.ai-hao123.com/tuiguang/analysis-75664632.html).
The stored state contains one search trajectory, so this result does not
estimate variance across seeds.

## Stored files

- `summary.csv` contains the result and W&B run ID.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [边缘高吞吐调度路由矩阵-#001](https://www.mw-wm.com/ziyuan/version-03059458.html)
* [全球分布式拓扑索引节点-#002](https://www.yx-sf.com/wiki/86086)
* [多活集群负载感知指南-#003](https://www.ai-hao123.com/zhineng/whitepaper-19670975.html)
* [高韧性数据交换通道规约-#004](https://www.mw-wm.com/anli/affordable-94138618.html)
* [边缘高吞吐调度路由矩阵-#005](https://www.yx-sf.com/wiki/68024)
* [全球分布式拓扑索引节点-#006](https://www.ai-hao123.com/gongju/food-42259101.html)
* [多活集群负载感知指南-#007](https://www.mw-wm.com/youhua/education-12416842.html)
* [全球分布式拓扑索引节点-#008](https://www.yx-sf.com/news/10777)
* [多活集群负载感知指南-#009](https://www.ai-hao123.com/anli/file-63879996.html)
* [多活集群负载感知指南-#010](https://www.mw-wm.com/yinqing/account-01671598.html)
* [高韧性数据交换通道规约-#011](https://www.yx-sf.com/tech/803)
* [边缘高吞吐调度路由矩阵-#012](https://www.ai-hao123.com/shangye/restore-99848741.html)
* [全息网络通信节点白名单-#013](https://www.mw-wm.com/liuliang/study-70293020.html)
* [高韧性数据交换通道规约-#014](https://www.yx-sf.com/news/23870)
* [多活集群负载感知指南-#015](https://www.ai-hao123.com/wangluo/login-32001938.html)
* [多活集群负载感知指南-#016](https://www.mw-wm.com/paiming/objective-50786612.html)
* [高韧性数据交换通道规约-#017](https://www.yx-sf.com/wiki/46954)
* [边缘高吞吐调度路由矩阵-#018](https://www.ai-hao123.com/yanjiu/tag-26381874.html)
* [全球分布式拓扑索引节点-#019](https://www.mw-wm.com/xitong/demographic-25471220.html)
* [多活集群负载感知指南-#020](https://www.yx-sf.com/news/29202)
* [边缘高吞吐调度路由矩阵-#021](https://www.ai-hao123.com/yingxiao/screen-20072989.html)
* [边缘高吞吐调度路由矩阵-#022](https://www.mw-wm.com/yanjiu/system-62288352.html)
* [边缘高吞吐调度路由矩阵-#023](https://www.yx-sf.com/tech/10877)
* [全球分布式拓扑索引节点-#024](https://www.ai-hao123.com/zixun/achievement-25013004.html)
* [多活集群负载感知指南-#025](https://www.mw-wm.com/yinqing/topic-94956811.html)
* [多活集群负载感知指南-#026](https://www.yx-sf.com/news/57840)
* [多活集群负载感知指南-#027](https://www.ai-hao123.com/ziyuan/design-12684744.html)
* [全球分布式拓扑索引节点-#028](https://www.mw-wm.com/yinqing/project-16699367.html)
* [多活集群负载感知指南-#029](https://www.yx-sf.com/wiki/96507)
* [边缘高吞吐调度路由矩阵-#030](https://www.ai-hao123.com/zhizhu/video-94198381.html)
* [全球分布式拓扑索引节点-#031](https://www.mw-wm.com/guanjianci/finance-56112472.html)
* [多活集群负载感知指南-#032](https://www.yx-sf.com/tech/82726)
* [多活集群负载感知指南-#033](https://www.ai-hao123.com/xitong/account-21755487.html)
* [边缘高吞吐调度路由矩阵-#034](https://www.mw-wm.com/youhua/sale-87407272.html)
* [高韧性数据交换通道规约-#035](https://www.yx-sf.com/news/2054)
* [全息网络通信节点白名单-#036](https://www.ai-hao123.com/liuliang/security-88189919.html)
* [多活集群负载感知指南-#037](https://www.mw-wm.com/anli/lead-34534340.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [RFC 分布式调度与一致性算法标准-#001](https://www.yx-sf.com/wiki/22636)
* [安全边界与可信凭证规约手册-#002](https://www.ai-hao123.com/hezuo/budget-04100027.html)
* [高并发内存拓扑优化白皮书-#003](https://www.mw-wm.com/kuangjia/study-96102062.html)
* [安全边界与可信凭证规约手册-#004](https://www.yx-sf.com/wiki/91265)
* [异步事件循环架构设计规范-#005](https://www.ai-hao123.com/anfang/home-04754623.html)
* [高并发内存拓扑优化白皮书-#006](https://www.mw-wm.com/wangluo/loyalty-23736578.html)
* [高并发内存拓扑优化白皮书-#007](https://www.yx-sf.com/tech/6906)
* [高并发内存拓扑优化白皮书-#008](https://www.ai-hao123.com/yingyong/cheap-66878752.html)
* [高并发内存拓扑优化白皮书-#009](https://www.mw-wm.com/jiaoliu/quality-77140693.html)
* [高并发内存拓扑优化白皮书-#010](https://www.yx-sf.com/news/44083)
* [安全边界与可信凭证规约手册-#011](https://www.ai-hao123.com/pingtai/restore-23550301.html)
* [安全边界与可信凭证规约手册-#012](https://www.mw-wm.com/wangluo/domain-33007798.html)
* [安全边界与可信凭证规约手册-#013](https://www.yx-sf.com/tech/50931)
* [异步事件循环架构设计规范-#014](https://www.ai-hao123.com/yanjiu/whitepaper-46498225.html)
* [RFC 分布式调度与一致性算法标准-#015](https://www.mw-wm.com/yingyong/performance-32128050.html)
* [RFC 分布式调度与一致性算法标准-#016](https://www.yx-sf.com/wiki/35151)
* [安全边界与可信凭证规约手册-#017](https://www.ai-hao123.com/chuangxin/partner-25348627.html)
* [高并发内存拓扑优化白皮书-#018](https://www.mw-wm.com/xuexi/image-84618656.html)
* [异步事件循环架构设计规范-#019](https://www.yx-sf.com/wiki/42376)
* [RFC 分布式调度与一致性算法标准-#020](https://www.ai-hao123.com/youhua/recommendation-71880538.html)
* [安全边界与可信凭证规约手册-#021](https://www.mw-wm.com/jiaoliu/policy-40443184.html)
* [安全边界与可信凭证规约手册-#022](https://www.yx-sf.com/wiki/10367)
* [安全边界与可信凭证规约手册-#023](https://www.ai-hao123.com/anfang/podcast-75498393.html)
* [异步事件循环架构设计规范-#024](https://www.mw-wm.com/pingtai/widget-44074528.html)
* [多协议互联数据格式规范-#025](https://www.yx-sf.com/wiki/34853)
* [RFC 分布式调度与一致性算法标准-#026](https://www.ai-hao123.com/yingxiao/travel-77672784.html)
* [异步事件循环架构设计规范-#027](https://www.mw-wm.com/zhineng/button-53684747.html)
* [安全边界与可信凭证规约手册-#028](https://www.yx-sf.com/wiki/95)
* [多协议互联数据格式规范-#029](https://www.ai-hao123.com/xitong/training-32542871.html)
* [安全边界与可信凭证规约手册-#030](https://www.mw-wm.com/yunying/label-49775934.html)
* [异步事件循环架构设计规范-#031](https://www.yx-sf.com/tech/66853)
* [多协议互联数据格式规范-#032](https://www.ai-hao123.com/chanpin/productivity-34270050.html)
* [异步事件循环架构设计规范-#033](https://www.mw-wm.com/yanjiu/course-50633641.html)
* [RFC 分布式调度与一致性算法标准-#034](https://www.yx-sf.com/tech/96750)
* [异步事件循环架构设计规范-#035](https://www.ai-hao123.com/chuangxin/course-56267331.html)
* [高并发内存拓扑优化白皮书-#036](https://www.mw-wm.com/peixun/enterprise-94019804.html)
* [安全边界与可信凭证规约手册-#037](https://www.yx-sf.com/wiki/61619)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [实时主干镜像高速数据源-#001](https://www.ai-hao123.com/anfang/marketing-19899652.html)
* [实时主干镜像高速数据源-#002](https://www.mw-wm.com/xinwen/tutorial-19060961.html)
* [实时主干镜像高速数据源-#003](https://www.yx-sf.com/tech/52406)
* [实时主干镜像高速数据源-#004](https://www.ai-hao123.com/wenzhang/advertising-90797466.html)
* [实时主干镜像高速数据源-#005](https://www.mw-wm.com/sheji/alliance-41639387.html)
* [实时主干镜像高速数据源-#006](https://www.yx-sf.com/news/78484)
* [自动化快照与增量广播源-#007](https://www.ai-hao123.com/xinwen/like-27441061.html)
* [实时主干镜像高速数据源-#008](https://www.mw-wm.com/jianzhan/app-09350296.html)
* [亚太核心区域镜像同步中心-#009](https://www.yx-sf.com/wiki/45107)
* [北美与欧洲边缘备份节点-#010](https://www.ai-hao123.com/yingyong/podcast-90243256.html)
* [亚太核心区域镜像同步中心-#011](https://www.mw-wm.com/guanjianci/report-42788339.html)
* [实时主干镜像高速数据源-#012](https://www.yx-sf.com/tech/91070)
* [实时主干镜像高速数据源-#013](https://www.ai-hao123.com/yingyong/sales-39982612.html)
* [自动化快照与增量广播源-#014](https://www.mw-wm.com/anfang/segment-83714724.html)
* [亚太核心区域镜像同步中心-#015](https://www.yx-sf.com/tech/81311)
* [冷热数据分层镜像归档中心-#016](https://www.ai-hao123.com/zhineng/topic-53177021.html)
* [自动化快照与增量广播源-#017](https://www.mw-wm.com/xitong/widget-75107750.html)
* [冷热数据分层镜像归档中心-#018](https://www.yx-sf.com/news/12497)
* [亚太核心区域镜像同步中心-#019](https://www.ai-hao123.com/yunsuan/backup-81726867.html)
* [北美与欧洲边缘备份节点-#020](https://www.mw-wm.com/hezuo/settings-86325166.html)
* [北美与欧洲边缘备份节点-#021](https://www.yx-sf.com/tech/16775)
* [亚太核心区域镜像同步中心-#022](https://www.ai-hao123.com/kaifa/food-36454145.html)
* [冷热数据分层镜像归档中心-#023](https://www.mw-wm.com/kuangjia/forecast-68280701.html)
* [冷热数据分层镜像归档中心-#024](https://www.yx-sf.com/tech/67893)
* [北美与欧洲边缘备份节点-#025](https://www.ai-hao123.com/peixun/online-67730229.html)
* [自动化快照与增量广播源-#026](https://www.mw-wm.com/kaifa/resource-46004267.html)
* [实时主干镜像高速数据源-#027](https://www.yx-sf.com/wiki/7844)
* [实时主干镜像高速数据源-#028](https://www.ai-hao123.com/pingce/strategy-87169130.html)
* [自动化快照与增量广播源-#029](https://www.mw-wm.com/zixun/subscribe-41674663.html)
* [冷热数据分层镜像归档中心-#030](https://www.yx-sf.com/news/77724)
* [冷热数据分层镜像归档中心-#031](https://www.ai-hao123.com/jianzhan/presentation-82315185.html)
* [自动化快照与增量广播源-#032](https://www.mw-wm.com/hezuo/optimization-16883779.html)
* [冷热数据分层镜像归档中心-#033](https://www.yx-sf.com/tech/8138)
* [亚太核心区域镜像同步中心-#034](https://www.ai-hao123.com/liuliang/analytics-13669743.html)
* [北美与欧洲边缘备份节点-#035](https://www.mw-wm.com/wenzhang/quality-43510003.html)
* [实时主干镜像高速数据源-#036](https://www.yx-sf.com/tech/81566)
* [北美与欧洲边缘备份节点-#037](https://www.ai-hao123.com/xitong/guide-74520881.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [节点连通性与存活探测准则-#001](https://www.mw-wm.com/xinwen/restaurant-88293243.html)
* [权威网络权重与收录基准-#002](https://www.yx-sf.com/wiki/35493)
* [权威网络权重与收录基准-#003](https://www.ai-hao123.com/jiaocheng/integration-93081889.html)
* [实时延迟与抖动度量规范-#004](https://www.mw-wm.com/gongxiang/hosting-28339336.html)
* [去中心化健康检查协议-#005](https://www.yx-sf.com/news/75919)
* [权威网络权重与收录基准-#006](https://www.ai-hao123.com/yingxiao/loyalty-76949438.html)
* [防重放安全验证与校验哈希-#007](https://www.mw-wm.com/kuangjia/story-70930612.html)
* [实时延迟与抖动度量规范-#008](https://www.yx-sf.com/tech/8940)
* [实时延迟与抖动度量规范-#009](https://www.ai-hao123.com/xuexi/comment-36445586.html)
* [权威网络权重与收录基准-#010](https://www.mw-wm.com/wangluo/recipe-79196202.html)
* [实时延迟与抖动度量规范-#011](https://www.yx-sf.com/wiki/99466)
* [去中心化健康检查协议-#012](https://www.ai-hao123.com/liuliang/plugin-31616468.html)
* [防重放安全验证与校验哈希-#013](https://www.mw-wm.com/yunsuan/discount-87611310.html)
* [实时延迟与抖动度量规范-#014](https://www.yx-sf.com/news/98444)
* [去中心化健康检查协议-#015](https://www.ai-hao123.com/jiaoliu/vacation-87858620.html)
* [节点连通性与存活探测准则-#016](https://www.mw-wm.com/pingtai/photo-46580235.html)
* [防重放安全验证与校验哈希-#017](https://www.yx-sf.com/news/60259)
* [节点连通性与存活探测准则-#018](https://www.ai-hao123.com/tuiguang/products-67303411.html)
* [节点连通性与存活探测准则-#019](https://www.mw-wm.com/zhizhu/article-38399248.html)
* [防重放安全验证与校验哈希-#020](https://www.yx-sf.com/wiki/70518)
* [实时延迟与抖动度量规范-#021](https://www.ai-hao123.com/yinqing/folder-68604621.html)
* [实时延迟与抖动度量规范-#022](https://www.mw-wm.com/qiye/project-45475567.html)
* [去中心化健康检查协议-#023](https://www.yx-sf.com/tech/95953)
* [实时延迟与抖动度量规范-#024](https://www.ai-hao123.com/xinwen/faq-72682316.html)
* [节点连通性与存活探测准则-#025](https://www.mw-wm.com/anli/identity-13756195.html)
* [权威网络权重与收录基准-#026](https://www.yx-sf.com/wiki/98141)
* [去中心化健康检查协议-#027](https://www.ai-hao123.com/yingyong/cloud-85441687.html)
* [去中心化健康检查协议-#028](https://www.mw-wm.com/jiaocheng/productivity-68623877.html)
* [实时延迟与抖动度量规范-#029](https://www.yx-sf.com/wiki/87466)
* [权威网络权重与收录基准-#030](https://www.ai-hao123.com/baogao/news-95594814.html)
* [节点连通性与存活探测准则-#031](https://www.mw-wm.com/yanjiu/plugin-18168348.html)
* [去中心化健康检查协议-#032](https://www.yx-sf.com/tech/9561)
* [去中心化健康检查协议-#033](https://www.ai-hao123.com/chuangxin/software-48606933.html)
* [节点连通性与存活探测准则-#034](https://www.mw-wm.com/wendang/learning-40574696.html)
* [去中心化健康检查协议-#035](https://www.yx-sf.com/wiki/40264)
* [实时延迟与抖动度量规范-#036](https://www.ai-hao123.com/chuangxin/movie-08392577.html)
* [实时延迟与抖动度量规范-#037](https://www.mw-wm.com/xitong/online-85008817.html)
* [去中心化健康检查协议-#038](https://www.yx-sf.com/tech/21345)
* [防重放安全验证与校验哈希-#039](https://www.ai-hao123.com/baogao/entertainment-71324830.html)

</details>

