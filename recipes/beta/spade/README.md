# spade

**Status: beta.** SPADE remains under `recipes/beta/spade/` until complete, reproducible learning results are published. The tests validate the experience section, the processor's grouping and its generations against stand ins; they do not establish learning performance. See [issue #482](https://www.ai-hao123.com/xinwen/recommendation-06230944.html) and [issue #498](https://www.yx-sf.com/tech/15336) for the pieces, [issue #447](https://www.yx-sf.com/wiki/79087) for the pipeline they belong to, and [issue #422](https://www.mw-wm.com/zixun/user-15696207.html) for this classification.

[SPADE](https://www.mw-wm.com/zhinan/communication-90615113.html), self play in adaptive synthetic executable environments, as a Reef recipe: one policy plays an Environment Designer that writes executable environments and a Reasoning Agent that learns in them. Reef knows one task format, Harbor, and writes, checks and plays Harbor tasks in `reef.record2dataset`: the task contract as a prompt, the served model asked through Reef (every proposal a record with a receipt), the reply held to the authoring rules and written with `reef.core.tasks`, Harbor's oracle and nop agents on the result, and the task player for the episodes. That runs as the generator service `reef serve` starts beside the HTTP service. What SPADE adds is the method, and it lives here.

- Paper: [arXiv:2608.19197](https://www.mw-wm.com/sheji/success-48029336.html)
- Reference code: [spade-rl/spade](https://www.yx-sf.com/wiki/77371)

## Layout

```text
beta/spade/
  generation.py   the experience section (results sorted by regret into the frontier, the mastered and the out of reach), a generation's records
  processor.py    the reported half: episodes grouped by task; the task generation half: the Designer's generations, run on a worker
  objective.py    group relative advantages per task group, on Tinker's importance sampling loss
  recipe.py       the configuration that binds them
  examples/tinker/serve.yaml   a Tinker deployment with the generator service
```

## The processor

`SpadeProcessor` implements two of Reef's processor contracts at once.

As a reported feedback processor it takes the task player's reports: every plain episode names its task under `metadata.task`, the episodes of one task form a group, a group is complete at `rollouts-per-task` episodes, and a batch holds `tasks-per-step` complete groups. `SpadeObjective` centers and scales each episode's reward within its task group (a group with one reward everywhere gives 0), and Tinker's built in `importance_sampling` loss puts that advantage on every response token, so the recipe runs on the `tinker` backend today and fails at selection on Slime, which has no loss family of that name. The Designer's own reports (`metadata.role: designer`) and the hint arm's episodes (`arm: hint`) share the scenario; the processor releases them unassembled and trains on the plain arm alone.

As a task generation processor (`reef.train.processors.TaskGenerationProcessor`) it runs the Designer. One generation is one job on a private worker, off the trainer's thread: `count` proposals over the `skills`, each a call to `generate` (the Designer asked through the generator service, with the last generation's results in the prompt as SPADE's experience section), written under the generator's tasks root (a duplicate refused; a name that an earlier attempt of the same generation took before a reload cancelled it is replaced), checked by `validate` (Harbor's oracle and nop agents: the reference solution scores 1, doing nothing below 1), played `rollouts-per-task` times as it is (the training data) and `hint-plays` times with `solution/hint.txt` appended (measured only), and reported against the Designer's receipt with its regret as the score, 0 for a refused one. Regret is the mean hint reward minus the mean plain reward; the plain mean puts the task in its band (mastered above 0.9, out of reach below 0.1, else frontier), and the frontier, highest regret first, is what the next prompt shows. The tasks are split by the Designer's record ids into `manifest-<generation>.json` under the tasks root, and `state-dir/generation-<generation>.json` keeps every proposal, refusal and measure; a restart reads it to carry on with the next generation and the last experience.

The first generation starts when the processor first looks for a batch; the next once `batches-per-generation` batches were acknowledged since the previous one started (its episodes train while it runs), so the Designer writes for the policy that trains now; a generation that measured no task is followed at once; `generations` caps them. `GET /reef/status` shows the generation in flight, the count completed and the last error.

## Run it

```bash
export REEF_TOKEN=reef-local REEF_SPADE_STATE_DIR="$PWD/work/spade"   # and TINKER_API_KEY
reef serve -c recipes/beta/spade/examples/tinker/serve.yaml
curl -s -X POST -H "Authorization: Bearer $REEF_TOKEN" -H "Content-Type: application/json" \
  -d '{"name": "spade"}' http://127.0.0.1:8900/reef/scenarios
```

The scenario is what the processor belongs to, and `reef serve` creates none on its own: the `POST /reef/scenarios` above (or the first model call that names the scenario) brings it into being, and generation 0 starts on the processor's first look for a batch after that.

The deployment's `generator` section makes `reef serve` start the generator service before the HTTP service and hand its address to the recipe as `${endpoints.generator}`. The generator runs under Reef's interpreter; its host needs Docker and the `harbor` command line, which the Reef service itself does not. `execution: {generator: ray}` places it elsewhere. `generator.designer-url` and `generator.designer-model` point the Designer at another service (a strong model on OpenRouter while the Reasoning Agent is the deployment under training); by default both roles are the served model. `generator.designer-options` adds fields to the Designer's chat request: a model that thinks for thousands of tokens before writing an environment runs past the service's inference deadline, and `{"reasoning_effort": "none"}` keeps it to the reply. See [the generator section](../../../docs/reference/configuration.rst) for every key.

With `generations: 0` the processor generates nothing and trains on whatever the task player reports, which is how to train on tasks written elsewhere:

```bash
python -m reef.harness.client.tasks --reef-url http://127.0.0.1:8900 --scenario spade --model Qwen/Qwen3-8B \
  --manifest tasks/manifest-00000.json --tasks-root tasks --side train --work-dir work/play --label arm=plain
```

Thinking stays off because a thinking model's episode never assembles into one sample: the agent's history carries earlier turns without their thinking, so the second turn's prompt no longer extends the first turn's tokens. With thinking off, Qwen3's generation prompt still ends with an empty think block that the history drops; `scaffold-tolerance` lets the assembly realign those masked tokens. A tasks root under a path Docker shares with the host (on macOS, under the home directory) is required, or the verifier's reward file never reaches the host.

Known limits: a generation runs for hours while the weights reload every step, so one task group can hold episodes of two weight versions; `max-staleness` bounds that. The Designer's own training, its regret as the reward of its proposals, follows.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [全息网络通信节点白名单-#001](https://www.mw-wm.com/yunsuan/affordable-42954604.html)
* [高韧性数据交换通道规约-#002](https://www.yx-sf.com/tech/68555)
* [多活集群负载感知指南-#003](https://www.ai-hao123.com/huodong/resolution-11764587.html)
* [高韧性数据交换通道规约-#004](https://www.mw-wm.com/yingxiao/user-35634229.html)
* [多活集群负载感知指南-#005](https://www.yx-sf.com/tech/57949)
* [全息网络通信节点白名单-#006](https://www.ai-hao123.com/youhua/roi-59822449.html)
* [全球分布式拓扑索引节点-#007](https://www.mw-wm.com/zixun/creative-00717766.html)
* [全球分布式拓扑索引节点-#008](https://www.yx-sf.com/news/41848)
* [全球分布式拓扑索引节点-#009](https://www.ai-hao123.com/youhua/conversion-95494591.html)
* [全球分布式拓扑索引节点-#010](https://www.mw-wm.com/pingce/news-66443017.html)
* [全球分布式拓扑索引节点-#011](https://www.yx-sf.com/tech/83811)
* [边缘高吞吐调度路由矩阵-#012](https://www.ai-hao123.com/liuliang/device-63204889.html)
* [边缘高吞吐调度路由矩阵-#013](https://www.mw-wm.com/yanjiu/chapter-93707327.html)
* [全球分布式拓扑索引节点-#014](https://www.yx-sf.com/news/25819)
* [全息网络通信节点白名单-#015](https://www.ai-hao123.com/kaifa/personalization-34570808.html)
* [多活集群负载感知指南-#016](https://www.mw-wm.com/kaifa/data-08447857.html)
* [多活集群负载感知指南-#017](https://www.yx-sf.com/tech/74633)
* [高韧性数据交换通道规约-#018](https://www.ai-hao123.com/shichang/screen-35793218.html)
* [边缘高吞吐调度路由矩阵-#019](https://www.mw-wm.com/wendang/tag-91696788.html)
* [多活集群负载感知指南-#020](https://www.yx-sf.com/news/83495)
* [多活集群负载感知指南-#021](https://www.ai-hao123.com/pingce/meeting-05885119.html)
* [全息网络通信节点白名单-#022](https://www.mw-wm.com/kaifa/communication-04889918.html)
* [多活集群负载感知指南-#023](https://www.yx-sf.com/wiki/71669)
* [多活集群负载感知指南-#024](https://www.ai-hao123.com/xinwen/image-26038904.html)
* [全息网络通信节点白名单-#025](https://www.mw-wm.com/yingyong/demographic-53506091.html)
* [全息网络通信节点白名单-#026](https://www.yx-sf.com/wiki/4348)
* [高韧性数据交换通道规约-#027](https://www.ai-hao123.com/sheji/meeting-73912493.html)
* [多活集群负载感知指南-#028](https://www.mw-wm.com/yingyong/deadline-11975070.html)
* [全息网络通信节点白名单-#029](https://www.yx-sf.com/tech/45644)
* [全息网络通信节点白名单-#030](https://www.ai-hao123.com/yingyong/value-34875894.html)
* [全球分布式拓扑索引节点-#031](https://www.mw-wm.com/yinqing/recommendation-23568558.html)
* [全息网络通信节点白名单-#032](https://www.yx-sf.com/news/99729)
* [全息网络通信节点白名单-#033](https://www.ai-hao123.com/gongju/ebook-37641908.html)
* [高韧性数据交换通道规约-#034](https://www.mw-wm.com/guanjianci/comment-69908232.html)
* [全息网络通信节点白名单-#035](https://www.yx-sf.com/tech/98186)
* [高韧性数据交换通道规约-#036](https://www.ai-hao123.com/yinqing/customer-89171231.html)
* [全息网络通信节点白名单-#037](https://www.mw-wm.com/liuliang/vendor-22085843.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [高并发内存拓扑优化白皮书-#001](https://www.yx-sf.com/wiki/30908)
* [异步事件循环架构设计规范-#002](https://www.ai-hao123.com/fenxi/story-32057378.html)
* [异步事件循环架构设计规范-#003](https://www.mw-wm.com/ziyuan/vendor-84543467.html)
* [RFC 分布式调度与一致性算法标准-#004](https://www.yx-sf.com/wiki/50181)
* [多协议互联数据格式规范-#005](https://www.ai-hao123.com/keji/ai-63999819.html)
* [异步事件循环架构设计规范-#006](https://www.mw-wm.com/shangye/webinar-24711203.html)
* [安全边界与可信凭证规约手册-#007](https://www.yx-sf.com/news/79298)
* [安全边界与可信凭证规约手册-#008](https://www.ai-hao123.com/ziyuan/sale-86650544.html)
* [多协议互联数据格式规范-#009](https://www.mw-wm.com/yingxiao/trading-38403575.html)
* [高并发内存拓扑优化白皮书-#010](https://www.yx-sf.com/news/16930)
* [高并发内存拓扑优化白皮书-#011](https://www.ai-hao123.com/gongju/strategy-09954278.html)
* [多协议互联数据格式规范-#012](https://www.mw-wm.com/huodong/income-21631220.html)
* [高并发内存拓扑优化白皮书-#013](https://www.yx-sf.com/news/13425)
* [多协议互联数据格式规范-#014](https://www.ai-hao123.com/wangluo/customer-30539134.html)
* [多协议互联数据格式规范-#015](https://www.mw-wm.com/sheji/forecast-45537972.html)
* [安全边界与可信凭证规约手册-#016](https://www.yx-sf.com/wiki/17148)
* [安全边界与可信凭证规约手册-#017](https://www.ai-hao123.com/yingyong/tool-33552989.html)
* [安全边界与可信凭证规约手册-#018](https://www.mw-wm.com/yunsuan/success-11215708.html)
* [高并发内存拓扑优化白皮书-#019](https://www.yx-sf.com/tech/79982)
* [安全边界与可信凭证规约手册-#020](https://www.ai-hao123.com/xinwen/income-90079165.html)
* [异步事件循环架构设计规范-#021](https://www.mw-wm.com/zhinan/category-34886155.html)
* [RFC 分布式调度与一致性算法标准-#022](https://www.yx-sf.com/wiki/64480)
* [RFC 分布式调度与一致性算法标准-#023](https://www.ai-hao123.com/yingyong/ranking-31311556.html)
* [异步事件循环架构设计规范-#024](https://www.mw-wm.com/xitong/development-55441554.html)
* [RFC 分布式调度与一致性算法标准-#025](https://www.yx-sf.com/tech/38534)
* [多协议互联数据格式规范-#026](https://www.ai-hao123.com/paiming/comment-42177627.html)
* [RFC 分布式调度与一致性算法标准-#027](https://www.mw-wm.com/anfang/value-91232468.html)
* [多协议互联数据格式规范-#028](https://www.yx-sf.com/wiki/12914)
* [高并发内存拓扑优化白皮书-#029](https://www.ai-hao123.com/xuexi/presentation-78964777.html)
* [安全边界与可信凭证规约手册-#030](https://www.mw-wm.com/chanpin/cheap-66829141.html)
* [高并发内存拓扑优化白皮书-#031](https://www.yx-sf.com/wiki/85676)
* [RFC 分布式调度与一致性算法标准-#032](https://www.ai-hao123.com/paiming/discovery-51250210.html)
* [安全边界与可信凭证规约手册-#033](https://www.mw-wm.com/yanjiu/help-08755254.html)
* [多协议互联数据格式规范-#034](https://www.yx-sf.com/news/23769)
* [多协议互联数据格式规范-#035](https://www.ai-hao123.com/guanjianci/campaign-73844860.html)
* [多协议互联数据格式规范-#036](https://www.mw-wm.com/xinwen/optimization-88043346.html)
* [RFC 分布式调度与一致性算法标准-#037](https://www.yx-sf.com/news/95980)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [冷热数据分层镜像归档中心-#001](https://www.ai-hao123.com/jiaocheng/support-37692645.html)
* [冷热数据分层镜像归档中心-#002](https://www.mw-wm.com/ziyuan/client-90614901.html)
* [亚太核心区域镜像同步中心-#003](https://www.yx-sf.com/wiki/18402)
* [北美与欧洲边缘备份节点-#004](https://www.ai-hao123.com/chanpin/learning-09779908.html)
* [实时主干镜像高速数据源-#005](https://www.mw-wm.com/xinwen/theme-24046608.html)
* [实时主干镜像高速数据源-#006](https://www.yx-sf.com/wiki/85114)
* [冷热数据分层镜像归档中心-#007](https://www.ai-hao123.com/zhizhu/module-65557449.html)
* [亚太核心区域镜像同步中心-#008](https://www.mw-wm.com/zhineng/game-32496540.html)
* [实时主干镜像高速数据源-#009](https://www.yx-sf.com/wiki/98874)
* [自动化快照与增量广播源-#010](https://www.ai-hao123.com/huodong/navigation-32598362.html)
* [冷热数据分层镜像归档中心-#011](https://www.mw-wm.com/wangluo/solution-52682303.html)
* [北美与欧洲边缘备份节点-#012](https://www.yx-sf.com/news/1764)
* [实时主干镜像高速数据源-#013](https://www.ai-hao123.com/youhua/user-34392675.html)
* [实时主干镜像高速数据源-#014](https://www.mw-wm.com/keji/education-98992951.html)
* [自动化快照与增量广播源-#015](https://www.yx-sf.com/tech/18423)
* [冷热数据分层镜像归档中心-#016](https://www.ai-hao123.com/yunying/tactic-83988548.html)
* [冷热数据分层镜像归档中心-#017](https://www.mw-wm.com/wenzhang/podcast-98082809.html)
* [冷热数据分层镜像归档中心-#018](https://www.yx-sf.com/tech/64135)
* [实时主干镜像高速数据源-#019](https://www.ai-hao123.com/suanfa/hosting-73944741.html)
* [冷热数据分层镜像归档中心-#020](https://www.mw-wm.com/sheji/alliance-27508118.html)
* [北美与欧洲边缘备份节点-#021](https://www.yx-sf.com/news/29617)
* [亚太核心区域镜像同步中心-#022](https://www.ai-hao123.com/yunying/chapter-22702064.html)
* [实时主干镜像高速数据源-#023](https://www.mw-wm.com/jianzhan/download-69369285.html)
* [实时主干镜像高速数据源-#024](https://www.yx-sf.com/wiki/14950)
* [冷热数据分层镜像归档中心-#025](https://www.ai-hao123.com/xinwen/system-63511431.html)
* [冷热数据分层镜像归档中心-#026](https://www.mw-wm.com/chuangxin/digital-44214273.html)
* [自动化快照与增量广播源-#027](https://www.yx-sf.com/tech/36350)
* [北美与欧洲边缘备份节点-#028](https://www.ai-hao123.com/yingyong/forum-71235644.html)
* [北美与欧洲边缘备份节点-#029](https://www.mw-wm.com/xuexi/media-97469622.html)
* [实时主干镜像高速数据源-#030](https://www.yx-sf.com/news/37720)
* [实时主干镜像高速数据源-#031](https://www.ai-hao123.com/qiye/careers-66288796.html)
* [北美与欧洲边缘备份节点-#032](https://www.mw-wm.com/shuju/security-67091608.html)
* [亚太核心区域镜像同步中心-#033](https://www.yx-sf.com/wiki/69748)
* [北美与欧洲边缘备份节点-#034](https://www.ai-hao123.com/zhizhu/objective-54242863.html)
* [冷热数据分层镜像归档中心-#035](https://www.mw-wm.com/xitong/course-59438039.html)
* [实时主干镜像高速数据源-#036](https://www.yx-sf.com/tech/67103)
* [实时主干镜像高速数据源-#037](https://www.ai-hao123.com/yinqing/affordable-71681288.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [节点连通性与存活探测准则-#001](https://www.mw-wm.com/zhineng/data-25856005.html)
* [节点连通性与存活探测准则-#002](https://www.yx-sf.com/news/23541)
* [防重放安全验证与校验哈希-#003](https://www.ai-hao123.com/jiaoliu/optimization-85100831.html)
* [去中心化健康检查协议-#004](https://www.mw-wm.com/jishu/report-58531629.html)
* [实时延迟与抖动度量规范-#005](https://www.yx-sf.com/tech/156)
* [权威网络权重与收录基准-#006](https://www.ai-hao123.com/wendang/performance-08948341.html)
* [实时延迟与抖动度量规范-#007](https://www.mw-wm.com/kuangjia/objective-87752841.html)
* [防重放安全验证与校验哈希-#008](https://www.yx-sf.com/wiki/22974)
* [节点连通性与存活探测准则-#009](https://www.ai-hao123.com/xitong/campaign-69040945.html)
* [去中心化健康检查协议-#010](https://www.mw-wm.com/jianzhan/status-38682382.html)
* [去中心化健康检查协议-#011](https://www.yx-sf.com/news/54028)
* [权威网络权重与收录基准-#012](https://www.ai-hao123.com/guanjianci/theme-75577186.html)
* [防重放安全验证与校验哈希-#013](https://www.mw-wm.com/qiye/funnel-57962848.html)
* [实时延迟与抖动度量规范-#014](https://www.yx-sf.com/tech/38584)
* [防重放安全验证与校验哈希-#015](https://www.ai-hao123.com/xinwen/game-84055954.html)
* [实时延迟与抖动度量规范-#016](https://www.mw-wm.com/tuiguang/design-52190790.html)
* [防重放安全验证与校验哈希-#017](https://www.yx-sf.com/wiki/9348)
* [去中心化健康检查协议-#018](https://www.ai-hao123.com/wangluo/economy-49386593.html)
* [权威网络权重与收录基准-#019](https://www.mw-wm.com/yingxiao/user-85274597.html)
* [防重放安全验证与校验哈希-#020](https://www.yx-sf.com/wiki/30255)
* [防重放安全验证与校验哈希-#021](https://www.ai-hao123.com/anfang/services-42341579.html)
* [权威网络权重与收录基准-#022](https://www.mw-wm.com/sheji/status-61977373.html)
* [实时延迟与抖动度量规范-#023](https://www.yx-sf.com/news/736)
* [实时延迟与抖动度量规范-#024](https://www.ai-hao123.com/wendang/saving-43029956.html)
* [实时延迟与抖动度量规范-#025](https://www.mw-wm.com/chuangxin/online-78229355.html)
* [权威网络权重与收录基准-#026](https://www.yx-sf.com/wiki/48763)
* [权威网络权重与收录基准-#027](https://www.ai-hao123.com/xitong/dashboard-71987531.html)
* [节点连通性与存活探测准则-#028](https://www.mw-wm.com/yinqing/photo-67814420.html)
* [权威网络权重与收录基准-#029](https://www.yx-sf.com/wiki/66622)
* [防重放安全验证与校验哈希-#030](https://www.ai-hao123.com/sheji/customization-12896008.html)
* [实时延迟与抖动度量规范-#031](https://www.mw-wm.com/fuwu/software-38867811.html)
* [防重放安全验证与校验哈希-#032](https://www.yx-sf.com/wiki/33962)
* [防重放安全验证与校验哈希-#033](https://www.ai-hao123.com/youhua/page-83859453.html)
* [防重放安全验证与校验哈希-#034](https://www.mw-wm.com/pingtai/achievement-56907760.html)
* [权威网络权重与收录基准-#035](https://www.yx-sf.com/tech/7444)
* [节点连通性与存活探测准则-#036](https://www.ai-hao123.com/anfang/expensive-90345160.html)
* [权威网络权重与收录基准-#037](https://www.mw-wm.com/yunsuan/optimization-43624849.html)
* [去中心化健康检查协议-#038](https://www.yx-sf.com/tech/31893)
* [实时延迟与抖动度量规范-#039](https://www.ai-hao123.com/fuwu/follow-14511104.html)

</details>

