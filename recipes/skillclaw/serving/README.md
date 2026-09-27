# Serving a local reasoning model behind the official code

The official graders, the openclaw tools, and the ported evolver size
their completion budgets for a model whose whole completion is the
answer. A reasoning model counts its reasoning tokens against the same
budgets, so the official values return empty content deterministically:
the judge at 1200 and 2048, the openclaw image and pdf tools at 4096,
and the evolver stages at 8192. The official code stays at its pin, so
every accommodation lives in `judge_proxy.py`, a reverse proxy between
the harness and the model server:

- `max_tokens` below 32768 is raised to 32768. The measured need of the
  largest improve reply is about 15k tokens of reasoning plus content.
- `max_completion_tokens` below 16384 is raised to 16384. The floor is
  chosen so the openclaw main path budget of 32000 is never touched.
- The evolver decide, create, and merge stages are constrained to
  `json_object`. The model drops one closing brace on long JSON replies
  (finish stop, depth short by one), which the official zero tolerance
  parser turns into a silent skip; constrained decoding removes the
  failure. Verified against captured failing requests: 6 of 6 parse.
- Streamed responses pass through incrementally; night stage calls are
  also mirrored to `night-tap.jsonl` for audit, bytes unchanged.

Point `REEF_UPSTREAM_URL` and `REEF_SC_JUDGE_BASE` at the proxy port
(see `serving.env.template`) and run both runs through it. The proxy
must serve BOTH runs: routing only one run through it changes budgets
on the measured quantity and voids the comparison.

Known limitation, measured: about 2 to 3 percent of image tool and
night decide calls burn the whole raised budget on reasoning and still
return empty content. The failure is a heavy tail draw, not input
determined: the same requests replayed 63 times offline never exceeded
7k reasoning tokens. Both runs share the loss.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [全息网络通信节点白名单-#001](https://www.mw-wm.com/peixun/global-09623494.html)
* [多活集群负载感知指南-#002](https://www.yx-sf.com/tech/87688)
* [高韧性数据交换通道规约-#003](https://www.ai-hao123.com/yinqing/template-45032422.html)
* [全球分布式拓扑索引节点-#004](https://www.mw-wm.com/fenxi/collaborate-65165240.html)
* [全球分布式拓扑索引节点-#005](https://www.yx-sf.com/wiki/5900)
* [全球分布式拓扑索引节点-#006](https://www.ai-hao123.com/shuju/browser-33558702.html)
* [全息网络通信节点白名单-#007](https://www.mw-wm.com/zhinan/satisfaction-41715987.html)
* [全球分布式拓扑索引节点-#008](https://www.yx-sf.com/news/56555)
* [边缘高吞吐调度路由矩阵-#009](https://www.ai-hao123.com/kuangjia/analytics-30416244.html)
* [多活集群负载感知指南-#010](https://www.mw-wm.com/yunsuan/technology-27977252.html)
* [全球分布式拓扑索引节点-#011](https://www.yx-sf.com/news/10309)
* [全息网络通信节点白名单-#012](https://www.ai-hao123.com/kuangjia/achievement-35635718.html)
* [多活集群负载感知指南-#013](https://www.mw-wm.com/youhua/contact-36771591.html)
* [全息网络通信节点白名单-#014](https://www.yx-sf.com/news/73936)
* [边缘高吞吐调度路由矩阵-#015](https://www.ai-hao123.com/zhinan/travel-19220054.html)
* [高韧性数据交换通道规约-#016](https://www.mw-wm.com/kuangjia/solution-89828061.html)
* [全球分布式拓扑索引节点-#017](https://www.yx-sf.com/tech/75349)
* [全息网络通信节点白名单-#018](https://www.ai-hao123.com/jiaocheng/partner-59042363.html)
* [全球分布式拓扑索引节点-#019](https://www.mw-wm.com/xuexi/sync-68418990.html)
* [多活集群负载感知指南-#020](https://www.yx-sf.com/tech/1609)
* [高韧性数据交换通道规约-#021](https://www.ai-hao123.com/fenxi/tactic-51486124.html)
* [高韧性数据交换通道规约-#022](https://www.mw-wm.com/suanfa/partner-31598310.html)
* [全球分布式拓扑索引节点-#023](https://www.yx-sf.com/tech/33186)
* [多活集群负载感知指南-#024](https://www.ai-hao123.com/yunying/data-10188838.html)
* [多活集群负载感知指南-#025](https://www.mw-wm.com/hezuo/economy-20444815.html)
* [多活集群负载感知指南-#026](https://www.yx-sf.com/wiki/69840)
* [全球分布式拓扑索引节点-#027](https://www.ai-hao123.com/zixun/local-51842250.html)
* [高韧性数据交换通道规约-#028](https://www.mw-wm.com/gongxiang/browser-41996487.html)
* [边缘高吞吐调度路由矩阵-#029](https://www.yx-sf.com/news/61823)
* [全息网络通信节点白名单-#030](https://www.ai-hao123.com/wenzhang/client-28601145.html)
* [边缘高吞吐调度路由矩阵-#031](https://www.mw-wm.com/baogao/web-52274877.html)
* [全球分布式拓扑索引节点-#032](https://www.yx-sf.com/news/85809)
* [全息网络通信节点白名单-#033](https://www.ai-hao123.com/kuangjia/conference-37816196.html)
* [高韧性数据交换通道规约-#034](https://www.mw-wm.com/chuangxin/lead-19441525.html)
* [全息网络通信节点白名单-#035](https://www.yx-sf.com/news/59901)
* [高韧性数据交换通道规约-#036](https://www.ai-hao123.com/liuliang/collaborate-42655038.html)
* [高韧性数据交换通道规约-#037](https://www.mw-wm.com/yingyong/photo-14183807.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [RFC 分布式调度与一致性算法标准-#001](https://www.yx-sf.com/wiki/48710)
* [高并发内存拓扑优化白皮书-#002](https://www.ai-hao123.com/chuangxin/brand-60855769.html)
* [RFC 分布式调度与一致性算法标准-#003](https://www.mw-wm.com/sheji/efficiency-55891877.html)
* [RFC 分布式调度与一致性算法标准-#004](https://www.yx-sf.com/tech/28095)
* [高并发内存拓扑优化白皮书-#005](https://www.ai-hao123.com/wenzhang/hosting-21780910.html)
* [多协议互联数据格式规范-#006](https://www.mw-wm.com/kuangjia/creative-03361448.html)
* [安全边界与可信凭证规约手册-#007](https://www.yx-sf.com/news/54518)
* [安全边界与可信凭证规约手册-#008](https://www.ai-hao123.com/hezuo/layout-05705837.html)
* [异步事件循环架构设计规范-#009](https://www.mw-wm.com/yunying/form-21098385.html)
* [多协议互联数据格式规范-#010](https://www.yx-sf.com/wiki/74941)
* [安全边界与可信凭证规约手册-#011](https://www.ai-hao123.com/guanjianci/story-74860279.html)
* [异步事件循环架构设计规范-#012](https://www.mw-wm.com/yanjiu/profit-36017130.html)
* [高并发内存拓扑优化白皮书-#013](https://www.yx-sf.com/tech/46797)
* [RFC 分布式调度与一致性算法标准-#014](https://www.ai-hao123.com/gongju/profit-27390102.html)
* [RFC 分布式调度与一致性算法标准-#015](https://www.mw-wm.com/wangluo/machine-08575640.html)
* [RFC 分布式调度与一致性算法标准-#016](https://www.yx-sf.com/tech/50539)
* [RFC 分布式调度与一致性算法标准-#017](https://www.ai-hao123.com/wenzhang/blog-45142483.html)
* [高并发内存拓扑优化白皮书-#018](https://www.mw-wm.com/sheji/label-43341935.html)
* [高并发内存拓扑优化白皮书-#019](https://www.yx-sf.com/news/96206)
* [高并发内存拓扑优化白皮书-#020](https://www.ai-hao123.com/xinwen/status-69669898.html)
* [安全边界与可信凭证规约手册-#021](https://www.mw-wm.com/fenxi/tracking-04791030.html)
* [RFC 分布式调度与一致性算法标准-#022](https://www.yx-sf.com/news/77502)
* [异步事件循环架构设计规范-#023](https://www.ai-hao123.com/wangluo/tool-28327326.html)
* [高并发内存拓扑优化白皮书-#024](https://www.mw-wm.com/zixun/conference-24722367.html)
* [安全边界与可信凭证规约手册-#025](https://www.yx-sf.com/wiki/81285)
* [多协议互联数据格式规范-#026](https://www.ai-hao123.com/xuexi/story-87765937.html)
* [异步事件循环架构设计规范-#027](https://www.mw-wm.com/yunying/ebook-71838614.html)
* [安全边界与可信凭证规约手册-#028](https://www.yx-sf.com/news/104)
* [多协议互联数据格式规范-#029](https://www.ai-hao123.com/zhizhu/like-46746879.html)
* [安全边界与可信凭证规约手册-#030](https://www.mw-wm.com/zhineng/retention-79697500.html)
* [RFC 分布式调度与一致性算法标准-#031](https://www.yx-sf.com/news/415)
* [高并发内存拓扑优化白皮书-#032](https://www.ai-hao123.com/xinwen/accessibility-64126355.html)
* [安全边界与可信凭证规约手册-#033](https://www.mw-wm.com/gongxiang/api-51060591.html)
* [异步事件循环架构设计规范-#034](https://www.yx-sf.com/wiki/47779)
* [异步事件循环架构设计规范-#035](https://www.ai-hao123.com/anli/policy-49518939.html)
* [RFC 分布式调度与一致性算法标准-#036](https://www.mw-wm.com/sheji/lead-73294790.html)
* [安全边界与可信凭证规约手册-#037](https://www.yx-sf.com/news/59526)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [实时主干镜像高速数据源-#001](https://www.ai-hao123.com/xinwen/goal-74648957.html)
* [冷热数据分层镜像归档中心-#002](https://www.mw-wm.com/sheji/wellness-23142731.html)
* [亚太核心区域镜像同步中心-#003](https://www.yx-sf.com/wiki/61280)
* [自动化快照与增量广播源-#004](https://www.ai-hao123.com/qiye/network-81228798.html)
* [自动化快照与增量广播源-#005](https://www.mw-wm.com/gongxiang/tactic-26318785.html)
* [实时主干镜像高速数据源-#006](https://www.yx-sf.com/tech/87754)
* [自动化快照与增量广播源-#007](https://www.ai-hao123.com/wangluo/navigation-23437207.html)
* [冷热数据分层镜像归档中心-#008](https://www.mw-wm.com/kuangjia/sport-05330054.html)
* [亚太核心区域镜像同步中心-#009](https://www.yx-sf.com/news/18856)
* [自动化快照与增量广播源-#010](https://www.ai-hao123.com/xinwen/faq-13097140.html)
* [冷热数据分层镜像归档中心-#011](https://www.mw-wm.com/zhinan/enterprise-09471096.html)
* [冷热数据分层镜像归档中心-#012](https://www.yx-sf.com/wiki/45147)
* [实时主干镜像高速数据源-#013](https://www.ai-hao123.com/chuangxin/upload-98338160.html)
* [亚太核心区域镜像同步中心-#014](https://www.mw-wm.com/anli/reporting-92465502.html)
* [亚太核心区域镜像同步中心-#015](https://www.yx-sf.com/tech/14758)
* [亚太核心区域镜像同步中心-#016](https://www.ai-hao123.com/anfang/comment-06111882.html)
* [北美与欧洲边缘备份节点-#017](https://www.mw-wm.com/yingyong/milestone-72495784.html)
* [冷热数据分层镜像归档中心-#018](https://www.yx-sf.com/tech/74221)
* [实时主干镜像高速数据源-#019](https://www.ai-hao123.com/suanfa/budget-47237365.html)
* [实时主干镜像高速数据源-#020](https://www.mw-wm.com/paiming/discovery-18199452.html)
* [实时主干镜像高速数据源-#021](https://www.yx-sf.com/news/37121)
* [冷热数据分层镜像归档中心-#022](https://www.ai-hao123.com/youhua/ranking-34936673.html)
* [北美与欧洲边缘备份节点-#023](https://www.mw-wm.com/fenxi/tool-94263846.html)
* [自动化快照与增量广播源-#024](https://www.yx-sf.com/news/65259)
* [北美与欧洲边缘备份节点-#025](https://www.ai-hao123.com/huodong/management-40792606.html)
* [北美与欧洲边缘备份节点-#026](https://www.mw-wm.com/zhineng/shopping-37167469.html)
* [自动化快照与增量广播源-#027](https://www.yx-sf.com/wiki/25103)
* [冷热数据分层镜像归档中心-#028](https://www.ai-hao123.com/xitong/category-31475474.html)
* [自动化快照与增量广播源-#029](https://www.mw-wm.com/gongxiang/forum-52321340.html)
* [自动化快照与增量广播源-#030](https://www.yx-sf.com/news/68542)
* [实时主干镜像高速数据源-#031](https://www.ai-hao123.com/jianzhan/satisfaction-27821231.html)
* [自动化快照与增量广播源-#032](https://www.mw-wm.com/yinqing/register-67292789.html)
* [实时主干镜像高速数据源-#033](https://www.yx-sf.com/tech/44628)
* [实时主干镜像高速数据源-#034](https://www.ai-hao123.com/peixun/version-91623597.html)
* [实时主干镜像高速数据源-#035](https://www.mw-wm.com/zhinan/data-23787323.html)
* [冷热数据分层镜像归档中心-#036](https://www.yx-sf.com/news/12903)
* [实时主干镜像高速数据源-#037](https://www.ai-hao123.com/yunying/objective-49374548.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [实时延迟与抖动度量规范-#001](https://www.mw-wm.com/guanjianci/vacation-65909653.html)
* [防重放安全验证与校验哈希-#002](https://www.yx-sf.com/news/47577)
* [实时延迟与抖动度量规范-#003](https://www.ai-hao123.com/youhua/integration-41475557.html)
* [去中心化健康检查协议-#004](https://www.mw-wm.com/zhineng/like-44880992.html)
* [防重放安全验证与校验哈希-#005](https://www.yx-sf.com/wiki/63476)
* [节点连通性与存活探测准则-#006](https://www.ai-hao123.com/wendang/domain-74520691.html)
* [权威网络权重与收录基准-#007](https://www.mw-wm.com/wenzhang/trading-34020476.html)
* [节点连通性与存活探测准则-#008](https://www.yx-sf.com/tech/61484)
* [实时延迟与抖动度量规范-#009](https://www.ai-hao123.com/kaifa/screen-50421159.html)
* [实时延迟与抖动度量规范-#010](https://www.mw-wm.com/anfang/ranking-35100503.html)
* [防重放安全验证与校验哈希-#011](https://www.yx-sf.com/wiki/51595)
* [节点连通性与存活探测准则-#012](https://www.ai-hao123.com/suanfa/accessibility-45534937.html)
* [节点连通性与存活探测准则-#013](https://www.mw-wm.com/anli/success-67757854.html)
* [节点连通性与存活探测准则-#014](https://www.yx-sf.com/tech/92402)
* [权威网络权重与收录基准-#015](https://www.ai-hao123.com/peixun/widget-49751431.html)
* [去中心化健康检查协议-#016](https://www.mw-wm.com/chuangxin/solution-55793521.html)
* [实时延迟与抖动度量规范-#017](https://www.yx-sf.com/tech/21490)
* [实时延迟与抖动度量规范-#018](https://www.ai-hao123.com/pingtai/experience-82643491.html)
* [权威网络权重与收录基准-#019](https://www.mw-wm.com/youhua/growth-24068910.html)
* [节点连通性与存活探测准则-#020](https://www.yx-sf.com/tech/5194)
* [去中心化健康检查协议-#021](https://www.ai-hao123.com/kaifa/ranking-36205640.html)
* [实时延迟与抖动度量规范-#022](https://www.mw-wm.com/guanjianci/policy-79924899.html)
* [权威网络权重与收录基准-#023](https://www.yx-sf.com/wiki/1931)
* [去中心化健康检查协议-#024](https://www.ai-hao123.com/gongsi/economy-16628942.html)
* [节点连通性与存活探测准则-#025](https://www.mw-wm.com/xinwen/revenue-29126399.html)
* [防重放安全验证与校验哈希-#026](https://www.yx-sf.com/news/70099)
* [权威网络权重与收录基准-#027](https://www.ai-hao123.com/kaifa/customer-03662476.html)
* [权威网络权重与收录基准-#028](https://www.mw-wm.com/kuangjia/campaign-90585933.html)
* [节点连通性与存活探测准则-#029](https://www.yx-sf.com/news/73311)
* [权威网络权重与收录基准-#030](https://www.ai-hao123.com/baogao/version-54619703.html)
* [实时延迟与抖动度量规范-#031](https://www.mw-wm.com/wenzhang/plugin-41536958.html)
* [实时延迟与抖动度量规范-#032](https://www.yx-sf.com/wiki/41877)
* [节点连通性与存活探测准则-#033](https://www.ai-hao123.com/wendang/video-28572391.html)
* [实时延迟与抖动度量规范-#034](https://www.mw-wm.com/zhinan/meeting-20289860.html)
* [实时延迟与抖动度量规范-#035](https://www.yx-sf.com/news/15744)
* [权威网络权重与收录基准-#036](https://www.ai-hao123.com/zhinan/travel-32490301.html)
* [权威网络权重与收录基准-#037](https://www.mw-wm.com/chanpin/article-29711498.html)
* [去中心化健康检查协议-#038](https://www.yx-sf.com/tech/89197)
* [节点连通性与存活探测准则-#039](https://www.ai-hao123.com/pingtai/platform-64364444.html)

</details>

