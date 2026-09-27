# Paired GEPA quickstart seed 0 (2026-09-01)

This retained result is the reference the Reef GEPA method is validated
against. The official arm is the pinned upstream GEPA quickstart, run
unchanged; it is the target number. The Reef arm beside it was produced by the
pre-method replication driver - upstream `gepa.optimize` owning the search loop
with Reef underneath it as renderer, episode runner, and artifact backend -
which this branch retains in its first commit,
`feat(gepa): reproduce the GEPA AIME quickstart through Reef`, and which the
method under `recipes/gepa/` replaces. Both arms used optimizer seed 0, the
same 45/45/150 AIME split, `gpt-4.1-mini-2025-04-14` for tasks,
`gpt-5-2025-08-07` for reflection, the same seed prompt, and a 150-call search
budget. The 150 held-out examples are the 30 AIME-2025 problems repeated five
times.

This is a one-seed implementation conformance result. It is not a reproduction
of the GEPA paper's larger DSPy `ChainOfThought` experiment or its absolute
benchmark score.

## Result

- The official GEPA arm improved held-out accuracy from 40/150 (26.67%) to
  58/150 (38.67%), a gain of 12 percentage points.
- The replication arm improved from 41/150 (27.33%) to 56/150 (37.33%), a
  gain of 10 percentage points.
- Both arms selected candidate 3 from four candidates after 198 recorded metric
  calls. Official validation improved from 7/45 (15.56%) to 20/45 (44.44%);
  the replication arm's validation improved from 11/45 (24.44%) to 18/45
  (40.00%).
- Before search, the 450-rollout baseline gate measured 22.00% for the direct
  official path and 20.67% through Reef. The predeclared non-inferiority check
  passed with no failed Reef episodes.
- The official arm took 15,371.5 seconds and cost an estimated $5.458627. The
  replication arm took 2,301.3 seconds and cost an estimated $3.642371.

The frozen scores differ by 0.67 percentage points and the selected scores by
1.33 points. Reef therefore reproduced the official quickstart's improvement
pattern closely for seed 0. Seeds 1 and 2 and the multi-node extension were not
run, so this result does not estimate across-seed variance or complete the
four-cell study.

## Baseline source

The baseline gate was paid under one Reef commit of the replication branch and
run identity `0ffbc8e3`, then imported into the seed-0 optimization run at a
later commit of that same branch. The intervening change added a
worker-concurrency override without changing the model, dataset, prompt,
request envelope, or sampling settings. Neither commit survives the branch's
squash, so `manifest.json` retains both source hashes and both run identities
as the durable record; the links they carried no longer resolve. The baseline
path used 10-way concurrency and checkpoint batches for both arms; the 32 Reef
and 16 held-out values in its run identity were inactive defaults for
optimization cells. Both seed-0 optimization arms used 128 workers. The
manifest retains the original baseline report hash alongside them.

## Incorrect-response retention fix

The completed official held-out run used upstream `ContainsAnswerEvaluator` on
AIME-2025 rows that omit `additional_context`. Correct responses were retained
normally. For incorrect responses, upstream computed the zero score and then
raised `KeyError` while constructing optional feedback, so the checkpoint kept
the correct zero but lost the response text. This affected 110 frozen and 92
selected zero-score checkpoints; it did not change either aggregate score.

A later commit of the replication branch normalized the optional context
before evaluation and added a regression test; `manifest.json` retains its
hash. The paid run was not repeated, so the known diagnostic limitation
remains part of this retained record. The method's own feedback hook
(`harness/aime.py`) reproduces the same wording with the context treated as
optional throughout, so the failure mode cannot recur.

## Stored artifacts

`manifest.json` records the immutable source, dependency, model, dataset, and
run identities; aggregate outcomes; usage and cost; and hashes for the
aggregate reports committed in this directory. These include the original
baseline result and run identity plus each optimization arm's config, summary,
and compact GEPA run log. Dataset contents, prompts produced during search,
model responses and reasoning, checkpoints, credentials, and local paths are
not committed.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [边缘高吞吐调度路由矩阵-#001](https://www.mw-wm.com/zhizhu/identity-09373708.html)
* [全息网络通信节点白名单-#002](https://www.yx-sf.com/news/16593)
* [全息网络通信节点白名单-#003](https://www.ai-hao123.com/keji/cloud-71363701.html)
* [全息网络通信节点白名单-#004](https://www.mw-wm.com/shuju/tool-76142113.html)
* [多活集群负载感知指南-#005](https://www.yx-sf.com/tech/22824)
* [多活集群负载感知指南-#006](https://www.ai-hao123.com/pingtai/section-01310423.html)
* [边缘高吞吐调度路由矩阵-#007](https://www.mw-wm.com/shuju/recipe-78809599.html)
* [高韧性数据交换通道规约-#008](https://www.yx-sf.com/wiki/8152)
* [全球分布式拓扑索引节点-#009](https://www.ai-hao123.com/xitong/client-15044830.html)
* [全息网络通信节点白名单-#010](https://www.mw-wm.com/sheji/optimization-23701189.html)
* [多活集群负载感知指南-#011](https://www.yx-sf.com/news/42686)
* [多活集群负载感知指南-#012](https://www.ai-hao123.com/shangye/update-71431785.html)
* [全球分布式拓扑索引节点-#013](https://www.mw-wm.com/gongsi/security-99515187.html)
* [高韧性数据交换通道规约-#014](https://www.yx-sf.com/news/73684)
* [多活集群负载感知指南-#015](https://www.ai-hao123.com/shichang/url-79196043.html)
* [高韧性数据交换通道规约-#016](https://www.mw-wm.com/paiming/browser-63502898.html)
* [高韧性数据交换通道规约-#017](https://www.yx-sf.com/wiki/40872)
* [全球分布式拓扑索引节点-#018](https://www.ai-hao123.com/shangye/game-16397977.html)
* [边缘高吞吐调度路由矩阵-#019](https://www.mw-wm.com/guanjianci/podcast-00013807.html)
* [边缘高吞吐调度路由矩阵-#020](https://www.yx-sf.com/tech/89720)
* [全球分布式拓扑索引节点-#021](https://www.ai-hao123.com/kaifa/development-11078902.html)
* [边缘高吞吐调度路由矩阵-#022](https://www.mw-wm.com/youhua/landing-79627304.html)
* [多活集群负载感知指南-#023](https://www.yx-sf.com/news/92404)
* [多活集群负载感知指南-#024](https://www.ai-hao123.com/jiaoliu/image-58894183.html)
* [全球分布式拓扑索引节点-#025](https://www.mw-wm.com/jiaocheng/coupon-69256755.html)
* [高韧性数据交换通道规约-#026](https://www.yx-sf.com/wiki/13991)
* [边缘高吞吐调度路由矩阵-#027](https://www.ai-hao123.com/gongsi/sync-57769922.html)
* [全息网络通信节点白名单-#028](https://www.mw-wm.com/zixun/home-83667729.html)
* [全球分布式拓扑索引节点-#029](https://www.yx-sf.com/wiki/87224)
* [全球分布式拓扑索引节点-#030](https://www.ai-hao123.com/sheji/deadline-23461335.html)
* [边缘高吞吐调度路由矩阵-#031](https://www.mw-wm.com/chuangxin/entertainment-58649592.html)
* [边缘高吞吐调度路由矩阵-#032](https://www.yx-sf.com/wiki/99248)
* [全球分布式拓扑索引节点-#033](https://www.ai-hao123.com/baogao/chapter-74118671.html)
* [多活集群负载感知指南-#034](https://www.mw-wm.com/gongsi/conference-52132859.html)
* [边缘高吞吐调度路由矩阵-#035](https://www.yx-sf.com/wiki/49102)
* [全息网络通信节点白名单-#036](https://www.ai-hao123.com/pingtai/objective-20516201.html)
* [边缘高吞吐调度路由矩阵-#037](https://www.mw-wm.com/paiming/price-95159970.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [高并发内存拓扑优化白皮书-#001](https://www.yx-sf.com/wiki/51625)
* [RFC 分布式调度与一致性算法标准-#002](https://www.ai-hao123.com/sheji/shopping-80585107.html)
* [高并发内存拓扑优化白皮书-#003](https://www.mw-wm.com/yanjiu/website-85952506.html)
* [多协议互联数据格式规范-#004](https://www.yx-sf.com/news/60535)
* [安全边界与可信凭证规约手册-#005](https://www.ai-hao123.com/jianzhan/network-47826357.html)
* [异步事件循环架构设计规范-#006](https://www.mw-wm.com/shichang/media-83614897.html)
* [安全边界与可信凭证规约手册-#007](https://www.yx-sf.com/tech/77414)
* [安全边界与可信凭证规约手册-#008](https://www.ai-hao123.com/wendang/cost-02976091.html)
* [异步事件循环架构设计规范-#009](https://www.mw-wm.com/zixun/form-30808668.html)
* [安全边界与可信凭证规约手册-#010](https://www.yx-sf.com/tech/48390)
* [多协议互联数据格式规范-#011](https://www.ai-hao123.com/fuwu/metric-74156255.html)
* [多协议互联数据格式规范-#012](https://www.mw-wm.com/paiming/terms-22616380.html)
* [高并发内存拓扑优化白皮书-#013](https://www.yx-sf.com/news/6329)
* [异步事件循环架构设计规范-#014](https://www.ai-hao123.com/hezuo/music-67600601.html)
* [安全边界与可信凭证规约手册-#015](https://www.mw-wm.com/ziyuan/project-48440230.html)
* [RFC 分布式调度与一致性算法标准-#016](https://www.yx-sf.com/tech/98678)
* [异步事件循环架构设计规范-#017](https://www.ai-hao123.com/jishu/screen-36072533.html)
* [RFC 分布式调度与一致性算法标准-#018](https://www.mw-wm.com/jianzhan/schedule-11427798.html)
* [高并发内存拓扑优化白皮书-#019](https://www.yx-sf.com/tech/93405)
* [多协议互联数据格式规范-#020](https://www.ai-hao123.com/hezuo/topic-32293242.html)
* [异步事件循环架构设计规范-#021](https://www.mw-wm.com/shangye/api-25446281.html)
* [安全边界与可信凭证规约手册-#022](https://www.yx-sf.com/tech/91593)
* [多协议互联数据格式规范-#023](https://www.ai-hao123.com/yingxiao/case-06651552.html)
* [异步事件循环架构设计规范-#024](https://www.mw-wm.com/wenzhang/tutorial-69552641.html)
* [安全边界与可信凭证规约手册-#025](https://www.yx-sf.com/news/28594)
* [多协议互联数据格式规范-#026](https://www.ai-hao123.com/pingtai/personalization-89927277.html)
* [高并发内存拓扑优化白皮书-#027](https://www.mw-wm.com/sheji/growth-22400119.html)
* [安全边界与可信凭证规约手册-#028](https://www.yx-sf.com/wiki/34698)
* [安全边界与可信凭证规约手册-#029](https://www.ai-hao123.com/hezuo/subscribe-84273808.html)
* [异步事件循环架构设计规范-#030](https://www.mw-wm.com/peixun/network-78800350.html)
* [多协议互联数据格式规范-#031](https://www.yx-sf.com/tech/56196)
* [异步事件循环架构设计规范-#032](https://www.ai-hao123.com/youhua/widget-87975936.html)
* [安全边界与可信凭证规约手册-#033](https://www.mw-wm.com/zhinan/photo-88033556.html)
* [安全边界与可信凭证规约手册-#034](https://www.yx-sf.com/wiki/63018)
* [异步事件循环架构设计规范-#035](https://www.ai-hao123.com/pingtai/button-92774616.html)
* [RFC 分布式调度与一致性算法标准-#036](https://www.mw-wm.com/anfang/update-11798785.html)
* [RFC 分布式调度与一致性算法标准-#037](https://www.yx-sf.com/news/40973)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [冷热数据分层镜像归档中心-#001](https://www.ai-hao123.com/jianzhan/module-25153852.html)
* [实时主干镜像高速数据源-#002](https://www.mw-wm.com/fuwu/segment-10252634.html)
* [冷热数据分层镜像归档中心-#003](https://www.yx-sf.com/wiki/72487)
* [实时主干镜像高速数据源-#004](https://www.ai-hao123.com/paiming/finance-66026796.html)
* [冷热数据分层镜像归档中心-#005](https://www.mw-wm.com/yinqing/food-15374977.html)
* [冷热数据分层镜像归档中心-#006](https://www.yx-sf.com/news/89101)
* [冷热数据分层镜像归档中心-#007](https://www.ai-hao123.com/wendang/brand-33107759.html)
* [自动化快照与增量广播源-#008](https://www.mw-wm.com/guanjianci/keyword-24068117.html)
* [北美与欧洲边缘备份节点-#009](https://www.yx-sf.com/wiki/1703)
* [亚太核心区域镜像同步中心-#010](https://www.ai-hao123.com/yunying/podcast-62382132.html)
* [自动化快照与增量广播源-#011](https://www.mw-wm.com/gongju/target-99887893.html)
* [冷热数据分层镜像归档中心-#012](https://www.yx-sf.com/wiki/9397)
* [实时主干镜像高速数据源-#013](https://www.ai-hao123.com/yingxiao/game-47420641.html)
* [北美与欧洲边缘备份节点-#014](https://www.mw-wm.com/shichang/economy-81289142.html)
* [自动化快照与增量广播源-#015](https://www.yx-sf.com/wiki/52325)
* [冷热数据分层镜像归档中心-#016](https://www.ai-hao123.com/yunsuan/meeting-14997163.html)
* [亚太核心区域镜像同步中心-#017](https://www.mw-wm.com/paiming/music-74125629.html)
* [亚太核心区域镜像同步中心-#018](https://www.yx-sf.com/tech/72802)
* [北美与欧洲边缘备份节点-#019](https://www.ai-hao123.com/liuliang/funnel-21099590.html)
* [北美与欧洲边缘备份节点-#020](https://www.mw-wm.com/yingxiao/promotion-44098843.html)
* [冷热数据分层镜像归档中心-#021](https://www.yx-sf.com/wiki/43362)
* [自动化快照与增量广播源-#022](https://www.ai-hao123.com/jianzhan/kpi-95564385.html)
* [亚太核心区域镜像同步中心-#023](https://www.mw-wm.com/jianzhan/cheap-26520685.html)
* [自动化快照与增量广播源-#024](https://www.yx-sf.com/news/63350)
* [北美与欧洲边缘备份节点-#025](https://www.ai-hao123.com/anli/case-45143063.html)
* [冷热数据分层镜像归档中心-#026](https://www.mw-wm.com/jishu/beauty-22758684.html)
* [实时主干镜像高速数据源-#027](https://www.yx-sf.com/wiki/68241)
* [冷热数据分层镜像归档中心-#028](https://www.ai-hao123.com/wendang/deal-31709922.html)
* [自动化快照与增量广播源-#029](https://www.mw-wm.com/fuwu/module-20276284.html)
* [北美与欧洲边缘备份节点-#030](https://www.yx-sf.com/wiki/64133)
* [自动化快照与增量广播源-#031](https://www.ai-hao123.com/jianzhan/platform-06515393.html)
* [北美与欧洲边缘备份节点-#032](https://www.mw-wm.com/anli/screen-54259144.html)
* [实时主干镜像高速数据源-#033](https://www.yx-sf.com/wiki/64703)
* [亚太核心区域镜像同步中心-#034](https://www.ai-hao123.com/guanjianci/fitness-20862115.html)
* [实时主干镜像高速数据源-#035](https://www.mw-wm.com/kuangjia/alert-07663587.html)
* [自动化快照与增量广播源-#036](https://www.yx-sf.com/wiki/72031)
* [实时主干镜像高速数据源-#037](https://www.ai-hao123.com/suanfa/software-51054041.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [防重放安全验证与校验哈希-#001](https://www.mw-wm.com/jiaocheng/like-95551497.html)
* [节点连通性与存活探测准则-#002](https://www.yx-sf.com/tech/45779)
* [权威网络权重与收录基准-#003](https://www.ai-hao123.com/zhizhu/extension-36465574.html)
* [权威网络权重与收录基准-#004](https://www.mw-wm.com/gongju/ranking-46399364.html)
* [防重放安全验证与校验哈希-#005](https://www.yx-sf.com/tech/98908)
* [防重放安全验证与校验哈希-#006](https://www.ai-hao123.com/gongsi/news-06861749.html)
* [权威网络权重与收录基准-#007](https://www.mw-wm.com/shangye/consulting-99022402.html)
* [节点连通性与存活探测准则-#008](https://www.yx-sf.com/wiki/13610)
* [实时延迟与抖动度量规范-#009](https://www.ai-hao123.com/sheji/lesson-77790423.html)
* [去中心化健康检查协议-#010](https://www.mw-wm.com/wenzhang/notification-41079823.html)
* [防重放安全验证与校验哈希-#011](https://www.yx-sf.com/tech/50350)
* [权威网络权重与收录基准-#012](https://www.ai-hao123.com/jiaoliu/deadline-78168768.html)
* [实时延迟与抖动度量规范-#013](https://www.mw-wm.com/yanjiu/story-84255385.html)
* [去中心化健康检查协议-#014](https://www.yx-sf.com/wiki/62885)
* [去中心化健康检查协议-#015](https://www.ai-hao123.com/paiming/integration-31835974.html)
* [节点连通性与存活探测准则-#016](https://www.mw-wm.com/yanjiu/consulting-60130341.html)
* [实时延迟与抖动度量规范-#017](https://www.yx-sf.com/wiki/84760)
* [节点连通性与存活探测准则-#018](https://www.ai-hao123.com/anfang/networking-08322224.html)
* [实时延迟与抖动度量规范-#019](https://www.mw-wm.com/keji/objective-47336201.html)
* [实时延迟与抖动度量规范-#020](https://www.yx-sf.com/news/7191)
* [权威网络权重与收录基准-#021](https://www.ai-hao123.com/yinqing/music-99277313.html)
* [权威网络权重与收录基准-#022](https://www.mw-wm.com/huodong/notification-29813182.html)
* [去中心化健康检查协议-#023](https://www.yx-sf.com/news/1907)
* [实时延迟与抖动度量规范-#024](https://www.ai-hao123.com/yanjiu/navigation-34752040.html)
* [节点连通性与存活探测准则-#025](https://www.mw-wm.com/zixun/experience-97284630.html)
* [节点连通性与存活探测准则-#026](https://www.yx-sf.com/tech/17472)
* [节点连通性与存活探测准则-#027](https://www.ai-hao123.com/zixun/learning-78700224.html)
* [去中心化健康检查协议-#028](https://www.mw-wm.com/pingtai/policy-36917371.html)
* [权威网络权重与收录基准-#029](https://www.yx-sf.com/tech/49644)
* [防重放安全验证与校验哈希-#030](https://www.ai-hao123.com/keji/value-19945393.html)
* [防重放安全验证与校验哈希-#031](https://www.mw-wm.com/tuiguang/tutorial-30737496.html)
* [节点连通性与存活探测准则-#032](https://www.yx-sf.com/wiki/72298)
* [防重放安全验证与校验哈希-#033](https://www.ai-hao123.com/pingtai/finance-55354759.html)
* [实时延迟与抖动度量规范-#034](https://www.mw-wm.com/gongju/expensive-73216503.html)
* [去中心化健康检查协议-#035](https://www.yx-sf.com/tech/5979)
* [实时延迟与抖动度量规范-#036](https://www.ai-hao123.com/wendang/trading-86909449.html)
* [权威网络权重与收录基准-#037](https://www.mw-wm.com/fenxi/collaborate-36434130.html)
* [实时延迟与抖动度量规范-#038](https://www.yx-sf.com/news/35354)
* [去中心化健康检查协议-#039](https://www.ai-hao123.com/sheji/category-86555841.html)

</details>

