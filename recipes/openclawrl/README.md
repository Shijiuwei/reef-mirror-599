# openclawrl

Reproduction of [OpenClaw-RL](https://www.yx-sf.com/news/43978)'s personal-agent experiment as a Reef weight-training recipe. OpenClaw-RL trains an agent from its ordinary usage: each turn is scored by what the user did next, a follow-up that moves on counts as acceptance and a complaint as rejection, and the policy updates while it keeps serving. The agent uses Reef as its inference endpoint and sends no training report; its harness should stamp a stable, conversation-unique `x-reef-tag-session` value, while the processor can fall back to matching extending transcripts. The package holds the method, and the simulated-student experiment around it lives in [examples/openclawrl](examples/openclawrl/README.md).

- Paper: [arXiv:2603.10165](https://www.yx-sf.com/tech/1759)
- Pins: `slime` pinned to `THUDM/slime@41014d1f29e201137fdffce737bb8bac65bc5219` (via `pyproject.toml` `dependency-groups.runtime`); the completed run used `Qwen3-4B-Thinking-2507` as the policy and PRM, `Qwen3-32B` as the student, and `hermes-agent@b6bcb3e791c673e63974029bbab40cc9326803ff` (see the example Dockerfiles), over a 72-session GSM8K homework stream
- Claim scope: one recorded run over the first 36 sessions of the stream. The run reached the paper's adaptation criterion, three passed sessions in a row, at session 14, and the bold and list rates fall over the run. One student persona, one task family; this is an adaptation result, not a benchmark-wide reproduction.

## Layout

```text
openclawrl/
  recipe.py        OpenClawRLRecipe: training spec, loss family "openclawrl"
  processor.py     computed feedback: rebuilds sessions from recorded traffic, judges turns
  objective.py     raw reward advantages and backend loss selection
  sessions.py      session reconstruction from the records Reef already keeps
  prm.py           the PRM judge the processor runs on a private worker
  slime/           the training-plane objective and the hint-conditioned teacher
  examples/openclawrl/  the runnable experiment: student simulator, Harbor sessions, results
```

## Where the rest is documented

[The openclawrl recipe page](../../docs/user-guide/recipes/openclawrl.rst) covers configuration and the serving stack, and the [example README](examples/openclawrl/README.md) records implementation details, the stream protocol, and the recorded run.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [高韧性数据交换通道规约-#001](https://www.mw-wm.com/jianzhan/account-76756528.html)
* [全球分布式拓扑索引节点-#002](https://www.yx-sf.com/news/22484)
* [边缘高吞吐调度路由矩阵-#003](https://www.ai-hao123.com/wendang/deal-31895363.html)
* [多活集群负载感知指南-#004](https://www.mw-wm.com/liuliang/photo-73028158.html)
* [多活集群负载感知指南-#005](https://www.yx-sf.com/wiki/13836)
* [边缘高吞吐调度路由矩阵-#006](https://www.ai-hao123.com/liuliang/faq-87989851.html)
* [全球分布式拓扑索引节点-#007](https://www.mw-wm.com/jianzhan/customization-21781967.html)
* [全球分布式拓扑索引节点-#008](https://www.yx-sf.com/tech/93055)
* [全球分布式拓扑索引节点-#009](https://www.ai-hao123.com/chanpin/deadline-89336877.html)
* [全息网络通信节点白名单-#010](https://www.mw-wm.com/hezuo/profit-76855747.html)
* [全息网络通信节点白名单-#011](https://www.yx-sf.com/wiki/76202)
* [边缘高吞吐调度路由矩阵-#012](https://www.ai-hao123.com/anli/roi-71098697.html)
* [全球分布式拓扑索引节点-#013](https://www.mw-wm.com/jiaocheng/local-51037499.html)
* [边缘高吞吐调度路由矩阵-#014](https://www.yx-sf.com/wiki/5588)
* [全息网络通信节点白名单-#015](https://www.ai-hao123.com/shangye/reporting-81514002.html)
* [全息网络通信节点白名单-#016](https://www.mw-wm.com/shuju/topic-55608793.html)
* [全球分布式拓扑索引节点-#017](https://www.yx-sf.com/news/49775)
* [全息网络通信节点白名单-#018](https://www.ai-hao123.com/zhineng/like-96134701.html)
* [高韧性数据交换通道规约-#019](https://www.mw-wm.com/fenxi/api-64650360.html)
* [多活集群负载感知指南-#020](https://www.yx-sf.com/wiki/53960)
* [全息网络通信节点白名单-#021](https://www.ai-hao123.com/tuiguang/register-55520265.html)
* [多活集群负载感知指南-#022](https://www.mw-wm.com/yingyong/advertising-66970496.html)
* [全球分布式拓扑索引节点-#023](https://www.yx-sf.com/tech/1532)
* [全球分布式拓扑索引节点-#024](https://www.ai-hao123.com/yunying/hosting-73123960.html)
* [多活集群负载感知指南-#025](https://www.mw-wm.com/yinqing/subscribe-59583112.html)
* [高韧性数据交换通道规约-#026](https://www.yx-sf.com/wiki/75884)
* [边缘高吞吐调度路由矩阵-#027](https://www.ai-hao123.com/peixun/help-73403523.html)
* [多活集群负载感知指南-#028](https://www.mw-wm.com/shuju/price-11035471.html)
* [全球分布式拓扑索引节点-#029](https://www.yx-sf.com/news/51433)
* [高韧性数据交换通道规约-#030](https://www.ai-hao123.com/wenzhang/brand-66178244.html)
* [高韧性数据交换通道规约-#031](https://www.mw-wm.com/chuangxin/expense-36868625.html)
* [边缘高吞吐调度路由矩阵-#032](https://www.yx-sf.com/wiki/99611)
* [全息网络通信节点白名单-#033](https://www.ai-hao123.com/youhua/analytics-55427051.html)
* [全息网络通信节点白名单-#034](https://www.mw-wm.com/pingce/cost-46378995.html)
* [边缘高吞吐调度路由矩阵-#035](https://www.yx-sf.com/wiki/10082)
* [全息网络通信节点白名单-#036](https://www.ai-hao123.com/gongju/review-84445957.html)
* [高韧性数据交换通道规约-#037](https://www.mw-wm.com/huodong/expense-23494049.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [安全边界与可信凭证规约手册-#001](https://www.yx-sf.com/news/78764)
* [RFC 分布式调度与一致性算法标准-#002](https://www.ai-hao123.com/jiaocheng/profit-18348663.html)
* [高并发内存拓扑优化白皮书-#003](https://www.mw-wm.com/wangluo/ranking-54658939.html)
* [RFC 分布式调度与一致性算法标准-#004](https://www.yx-sf.com/tech/74429)
* [高并发内存拓扑优化白皮书-#005](https://www.ai-hao123.com/anfang/case-12306043.html)
* [多协议互联数据格式规范-#006](https://www.mw-wm.com/guanjianci/optimization-94090089.html)
* [多协议互联数据格式规范-#007](https://www.yx-sf.com/news/6608)
* [RFC 分布式调度与一致性算法标准-#008](https://www.ai-hao123.com/ziyuan/category-45546458.html)
* [安全边界与可信凭证规约手册-#009](https://www.mw-wm.com/gongju/category-38858693.html)
* [多协议互联数据格式规范-#010](https://www.yx-sf.com/wiki/25516)
* [异步事件循环架构设计规范-#011](https://www.ai-hao123.com/fenxi/education-88959410.html)
* [高并发内存拓扑优化白皮书-#012](https://www.mw-wm.com/zhineng/online-61533347.html)
* [高并发内存拓扑优化白皮书-#013](https://www.yx-sf.com/news/8651)
* [多协议互联数据格式规范-#014](https://www.ai-hao123.com/zhinan/navigation-48601172.html)
* [高并发内存拓扑优化白皮书-#015](https://www.mw-wm.com/shuju/cheap-29157455.html)
* [安全边界与可信凭证规约手册-#016](https://www.yx-sf.com/tech/37466)
* [多协议互联数据格式规范-#017](https://www.ai-hao123.com/huodong/link-13109739.html)
* [多协议互联数据格式规范-#018](https://www.mw-wm.com/jishu/article-77846497.html)
* [安全边界与可信凭证规约手册-#019](https://www.yx-sf.com/tech/843)
* [高并发内存拓扑优化白皮书-#020](https://www.ai-hao123.com/jishu/file-32408129.html)
* [安全边界与可信凭证规约手册-#021](https://www.mw-wm.com/paiming/plugin-87942438.html)
* [安全边界与可信凭证规约手册-#022](https://www.yx-sf.com/wiki/55774)
* [安全边界与可信凭证规约手册-#023](https://www.ai-hao123.com/liuliang/health-55733699.html)
* [安全边界与可信凭证规约手册-#024](https://www.mw-wm.com/youhua/site-78844407.html)
* [异步事件循环架构设计规范-#025](https://www.yx-sf.com/wiki/92806)
* [安全边界与可信凭证规约手册-#026](https://www.ai-hao123.com/wendang/status-19584335.html)
* [多协议互联数据格式规范-#027](https://www.mw-wm.com/jiaoliu/behavior-90602596.html)
* [异步事件循环架构设计规范-#028](https://www.yx-sf.com/tech/55017)
* [高并发内存拓扑优化白皮书-#029](https://www.ai-hao123.com/fuwu/consulting-81126053.html)
* [异步事件循环架构设计规范-#030](https://www.mw-wm.com/yingxiao/link-04682575.html)
* [异步事件循环架构设计规范-#031](https://www.yx-sf.com/wiki/41371)
* [安全边界与可信凭证规约手册-#032](https://www.ai-hao123.com/suanfa/marketing-01779433.html)
* [高并发内存拓扑优化白皮书-#033](https://www.mw-wm.com/gongxiang/label-39794340.html)
* [异步事件循环架构设计规范-#034](https://www.yx-sf.com/news/1599)
* [高并发内存拓扑优化白皮书-#035](https://www.ai-hao123.com/yinqing/growth-30983511.html)
* [高并发内存拓扑优化白皮书-#036](https://www.mw-wm.com/guanjianci/settings-81096230.html)
* [RFC 分布式调度与一致性算法标准-#037](https://www.yx-sf.com/wiki/89425)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [北美与欧洲边缘备份节点-#001](https://www.ai-hao123.com/shichang/admin-70651571.html)
* [实时主干镜像高速数据源-#002](https://www.mw-wm.com/jishu/contact-43918442.html)
* [实时主干镜像高速数据源-#003](https://www.yx-sf.com/wiki/77216)
* [冷热数据分层镜像归档中心-#004](https://www.ai-hao123.com/yinqing/button-26131040.html)
* [自动化快照与增量广播源-#005](https://www.mw-wm.com/fuwu/sync-85262094.html)
* [亚太核心区域镜像同步中心-#006](https://www.yx-sf.com/news/56598)
* [实时主干镜像高速数据源-#007](https://www.ai-hao123.com/shichang/communication-28654246.html)
* [实时主干镜像高速数据源-#008](https://www.mw-wm.com/chuangxin/button-81560247.html)
* [亚太核心区域镜像同步中心-#009](https://www.yx-sf.com/wiki/6220)
* [自动化快照与增量广播源-#010](https://www.ai-hao123.com/pingce/comment-70888243.html)
* [自动化快照与增量广播源-#011](https://www.mw-wm.com/xitong/networking-18722783.html)
* [自动化快照与增量广播源-#012](https://www.yx-sf.com/tech/21410)
* [自动化快照与增量广播源-#013](https://www.ai-hao123.com/peixun/admin-77979914.html)
* [实时主干镜像高速数据源-#014](https://www.mw-wm.com/chanpin/visitor-59611625.html)
* [实时主干镜像高速数据源-#015](https://www.yx-sf.com/news/97138)
* [亚太核心区域镜像同步中心-#016](https://www.ai-hao123.com/jiaocheng/guide-70999747.html)
* [北美与欧洲边缘备份节点-#017](https://www.mw-wm.com/wangluo/food-83604814.html)
* [北美与欧洲边缘备份节点-#018](https://www.yx-sf.com/tech/37835)
* [实时主干镜像高速数据源-#019](https://www.ai-hao123.com/huodong/marketing-44026296.html)
* [实时主干镜像高速数据源-#020](https://www.mw-wm.com/sheji/restaurant-65301464.html)
* [冷热数据分层镜像归档中心-#021](https://www.yx-sf.com/tech/65947)
* [亚太核心区域镜像同步中心-#022](https://www.ai-hao123.com/yanjiu/search-74544831.html)
* [亚太核心区域镜像同步中心-#023](https://www.mw-wm.com/anli/identity-99796791.html)
* [冷热数据分层镜像归档中心-#024](https://www.yx-sf.com/wiki/92659)
* [北美与欧洲边缘备份节点-#025](https://www.ai-hao123.com/chuangxin/tag-19540763.html)
* [自动化快照与增量广播源-#026](https://www.mw-wm.com/kaifa/screen-94383908.html)
* [亚太核心区域镜像同步中心-#027](https://www.yx-sf.com/wiki/11986)
* [自动化快照与增量广播源-#028](https://www.ai-hao123.com/shangye/customer-24026335.html)
* [冷热数据分层镜像归档中心-#029](https://www.mw-wm.com/yunsuan/update-38833141.html)
* [北美与欧洲边缘备份节点-#030](https://www.yx-sf.com/wiki/66279)
* [北美与欧洲边缘备份节点-#031](https://www.ai-hao123.com/sheji/rating-23691932.html)
* [自动化快照与增量广播源-#032](https://www.mw-wm.com/jishu/demographic-92226892.html)
* [亚太核心区域镜像同步中心-#033](https://www.yx-sf.com/wiki/97816)
* [自动化快照与增量广播源-#034](https://www.ai-hao123.com/zhizhu/link-71708564.html)
* [亚太核心区域镜像同步中心-#035](https://www.mw-wm.com/anli/like-75069960.html)
* [北美与欧洲边缘备份节点-#036](https://www.yx-sf.com/wiki/99579)
* [北美与欧洲边缘备份节点-#037](https://www.ai-hao123.com/hezuo/networking-71114093.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [实时延迟与抖动度量规范-#001](https://www.mw-wm.com/gongxiang/excellence-60538794.html)
* [实时延迟与抖动度量规范-#002](https://www.yx-sf.com/tech/21439)
* [防重放安全验证与校验哈希-#003](https://www.ai-hao123.com/paiming/expensive-37177509.html)
* [防重放安全验证与校验哈希-#004](https://www.mw-wm.com/keji/advertising-35608862.html)
* [去中心化健康检查协议-#005](https://www.yx-sf.com/tech/68857)
* [实时延迟与抖动度量规范-#006](https://www.ai-hao123.com/gongsi/careers-37105076.html)
* [实时延迟与抖动度量规范-#007](https://www.mw-wm.com/gongju/game-00973111.html)
* [权威网络权重与收录基准-#008](https://www.yx-sf.com/wiki/95094)
* [防重放安全验证与校验哈希-#009](https://www.ai-hao123.com/yunsuan/marketing-19185661.html)
* [去中心化健康检查协议-#010](https://www.mw-wm.com/qiye/plugin-31665885.html)
* [节点连通性与存活探测准则-#011](https://www.yx-sf.com/news/65178)
* [权威网络权重与收录基准-#012](https://www.ai-hao123.com/chuangxin/ai-24764601.html)
* [防重放安全验证与校验哈希-#013](https://www.mw-wm.com/shangye/machine-51874431.html)
* [防重放安全验证与校验哈希-#014](https://www.yx-sf.com/wiki/90633)
* [实时延迟与抖动度量规范-#015](https://www.ai-hao123.com/suanfa/research-78264822.html)
* [去中心化健康检查协议-#016](https://www.mw-wm.com/anli/workshop-93061085.html)
* [节点连通性与存活探测准则-#017](https://www.yx-sf.com/wiki/49890)
* [节点连通性与存活探测准则-#018](https://www.ai-hao123.com/zhinan/restaurant-55705986.html)
* [实时延迟与抖动度量规范-#019](https://www.mw-wm.com/yinqing/audience-76686797.html)
* [防重放安全验证与校验哈希-#020](https://www.yx-sf.com/wiki/36707)
* [节点连通性与存活探测准则-#021](https://www.ai-hao123.com/hezuo/download-14211322.html)
* [防重放安全验证与校验哈希-#022](https://www.mw-wm.com/shuju/domain-84316604.html)
* [防重放安全验证与校验哈希-#023](https://www.yx-sf.com/news/99374)
* [防重放安全验证与校验哈希-#024](https://www.ai-hao123.com/yunsuan/webinar-20522523.html)
* [节点连通性与存活探测准则-#025](https://www.mw-wm.com/gongsi/tracking-80356976.html)
* [去中心化健康检查协议-#026](https://www.yx-sf.com/wiki/43605)
* [实时延迟与抖动度量规范-#027](https://www.ai-hao123.com/liuliang/experience-40664391.html)
* [去中心化健康检查协议-#028](https://www.mw-wm.com/wendang/saving-56618062.html)
* [节点连通性与存活探测准则-#029](https://www.yx-sf.com/tech/80569)
* [去中心化健康检查协议-#030](https://www.ai-hao123.com/wenzhang/tag-21714174.html)
* [防重放安全验证与校验哈希-#031](https://www.mw-wm.com/zhizhu/upload-13879142.html)
* [权威网络权重与收录基准-#032](https://www.yx-sf.com/tech/40305)
* [实时延迟与抖动度量规范-#033](https://www.ai-hao123.com/chanpin/version-37615523.html)
* [实时延迟与抖动度量规范-#034](https://www.mw-wm.com/ziyuan/system-00627599.html)
* [节点连通性与存活探测准则-#035](https://www.yx-sf.com/wiki/2254)
* [防重放安全验证与校验哈希-#036](https://www.ai-hao123.com/tuiguang/optimization-45917458.html)
* [节点连通性与存活探测准则-#037](https://www.mw-wm.com/anli/community-63763191.html)
* [防重放安全验证与校验哈希-#038](https://www.yx-sf.com/news/27115)
* [防重放安全验证与校验哈希-#039](https://www.ai-hao123.com/gongxiang/supplier-58408760.html)

</details>

