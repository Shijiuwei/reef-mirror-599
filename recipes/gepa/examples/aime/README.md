# GEPA on AIME: validating the method

[GEPA](https://www.mw-wm.com/yingyong/tactic-42687066.html) is reflective prompt evolution: keep an
archive of candidate prompts, Pareto-sample a parent from it, evaluate that
parent on a small training minibatch, ask a stronger model to rewrite one
component in light of what went wrong, keep the child only if it beats its
parent on that minibatch, then score it on the whole validation set and serve
the best. `recipes/gepa/` is that algorithm written as a Reef harness-evolution
method - `propose` and `selection` under the `evolution:` contract, with the
archive as the method's own state on disk - and this example is what validates
it. The benchmark is the upstream AIME quickstart: 45 training problems, 45
validation problems, a sealed 150-problem AIME-2025 test split, a 150-call
search budget, seed 0. Nothing here imports `gepa`; the two places the example
reproduces upstream text (the scorer's feedback wording and the epoch-shuffled
minibatch order) say so in `harness/aime.py`.

## What maps to what

| GEPA | Reef |
| --- | --- |
| Candidate prompt | a composition of harness nodes; the evolvable one here is `rules`, Pi's `AGENTS.md` |
| Pareto-sample a parent from the archive | `Archive.select_parent` over per-problem fronts, dominated candidates pruned |
| Evaluate the parent on a train minibatch, with traces | the served composition's own recorded traffic: the driver runs the minibatch through the service, so `propose` gets the transcripts free |
| Reflect on one component and propose a rewrite | `models["reflection"]` (gpt-5) with GEPA's own prompt, over `Inputs` / `Generated Outputs` / `Feedback` records |
| Accept the child iff it beats the parent on the minibatch | the proposer runs its own episodes and returns `None` on a reject, which skips the step |
| Full validation pass, then Pareto update | the mechanism's `evolution.tasks` is the validation set; `GEPASelectorMixin.decide` reads the per-task scores it produced |
| Serve the argmax-mean candidate | select on a strict mean improvement over the served composition, which publishes the tree for `GET /reef/harness` |

The request envelope is reproduced by seed nodes rather than by a custom Pi
command line: a `config` node writes `defaultTools: []` into `settings.json`,
and a fixed `code_extension` makes the rendered rules text Pi's entire system
prompt at `before_agent_start` and flattens a single-part user message to the
plain string upstream sends. Both are non-evolvable. The extension reads
`AGENTS.md` at startup, so the rules node underneath it can evolve freely.

## Setup and run

```bash
pip install -e .                       # datasets, reef-client
REEF_PI_BINARY=/path/to/pi ./run.sh --dry-run
```

A dry run verifies the Pi binary is `0.84.2`, loads and hash-checks the pinned
splits, boots the recipe from `gepa.yaml` with the validation set filled in,
and prints the plan. It makes no model call. Git LFS is not needed: the
embedded service keeps its artifacts in memory under `./work`.

```bash
OPENAI_API_KEY=... REEF_PI_BINARY=/path/to/pi ./run.sh
```

A live run is roughly 150 search episodes plus the mechanism's validation
passes, then 300 test episodes. Episodes are independent, so `REEF_GEPA_WORKERS`
(default 128) run at once everywhere: the mechanism's validation pass through
`execution.evolution.workers`, the driver's minibatch, and the test passes. It
changes wall time only. `--budget` lowers the search budget for a
shorter run. `REEF_GEPA_MULTI=1` adds an `aime-solver` skill node to the seed
and evolves it alongside the rules node, which is the extension case, not the
comparison. The credential is read once, handed to the embedded service as its
upstream, and never written into a candidate, a checkpoint, or a published
tree; the Pi episodes authenticate to the local service with a placeholder.

The validation pool defaults to `auto`: one worker selects `uni`, multiple
workers select `mp` (or Ray in an existing multi-worker placement group).
The former `local` thread-pool executor has been removed. The retained
concurrency default of 128 now means 128 persistent Python worker processes;
reduce `REEF_GEPA_WORKERS` to fit host memory and provider rate limits, for
example `REEF_GEPA_WORKERS=4 ./run.sh`. Set `REEF_GEPA_EXECUTOR=ray` for Ray
actors, or `REEF_GEPA_EXECUTOR=uni REEF_GEPA_WORKERS=1` for one in-process
worker. At recipe construction,
the driver binds the scoring/feedback hooks to a serializable snapshot of the
answer and context tables. Workers do not rely on driver globals, and later
registrations do not change an existing recipe. Labels stay in scorer state,
not in the task prompts or rendered harness files. Episode subprocesses and
temporary directories remain isolated. Custom hooks are preserved; they must
carry their own serializable state when used across processes.

For example, a two-worker local Ray run:

```bash
pip install 'ray[default]'
REEF_GEPA_EXECUTOR=ray REEF_GEPA_WORKERS=2 ./run.sh
```

To use an existing cluster, also set `RAY_ADDRESS` to its connection address
(for example `ray://head-node:10001`). Use compatible Python/Ray versions and
install the same Reef revision, Node.js, and Pi `0.84.2` on every worker node.
`REEF_PI_BINARY` must resolve on every node (use `pi` on PATH or a common
absolute installation path). The driver uploads only this example's
`harness/` Python package through Ray's runtime environment, not the work
directory or credentials. If embedding the driver in an already initialized
Ray runtime, provision that package there yourself; the driver neither
reconfigures nor shuts down a runtime it does not own.

Ray affects **only the mechanism's candidate/current validation pool**. The
driver's minibatch and held-out test passes still use local threads. Validation
workers call the configured upstream model directly; they do not need access
to the embedded loopback service. They do need network access to the provider,
and receive the model credential in their transient episode binding, so use
only trusted Ray nodes. The embedded service remains bound to `127.0.0.1`.

Size the Ray pool to available resources and provider rate limits: each worker
requests one logical CPU and zero GPUs by default, so the default 128 workers
would require 128 available logical CPUs for the whole pool to become ready.
Override `execution.evolution.resources.cpus_per_worker` only when appropriate
for the workload. Worker count is fixed for a run, not autoscaling.
The driver still forwards top-level `execution` and `executors`, including
named profiles and Ray actor options such as `runtime_env`. `--dry-run`
validates the recipe but does not launch workers or check remote dependencies.

One round is: pull the served tree, take the method's plan for the iteration
(its parent choice and its three training problems), run each as a Pi episode against that tree with a
model binding pointed back at the embedded service, score it, find its recorded
request by problem text, and report the score against it. The third report
closes the batch (`data.batch_size: 3`) and schedules the training step, which
is where the method proposes, evaluates, and publishes. Rounds stop when the
archive reaches the metric-call budget.

## What a run retains

Everything under `./work`, and a rerun resumes from it:

- `gepa/<scenario>.json` - the archive: candidates and their texts, parents,
  per-problem validation vectors, fronts, the round-robin cursors, the
  metric-call count, every reflection's prompt and reply, and the iteration
  plans - which parent and which training problems each round used, drawn from
  one seeded generator in upstream's own order, so seed 0 here shows the
  reflection model the same problems the official seed-0 run showed it.
- `heldout/{frozen,selected}/example-NNNN.json` - one checkpoint per test
  episode, keyed by the problem and the composition; a checkpoint written for
  a different composition is refused, never folded into the score.
- `reef-data/`, `artifacts/` - the scenario's records and its release chain.
- `summary.json` - validation seed and selected means from the archive, both
  test scores, the candidate count, the metric calls, and the retained official
  numbers beside them.

## The validation contract

Four runs on 2026-09-02, two of the upstream optimizer and two of this method,
across two seeds, with the models, split, budget, scorer, and worker count
held fixed. Every run produced four candidates and consumed 198 metric calls.
The runs pair by seed: at a given seed both arms reflect on the same training
problems in the same order.

| seed | arm | frozen | selected | improvement |
| ---: | --- | ---: | ---: | ---: |
| 0 | upstream GEPA | 31.33% | 42.67% | +11.33 pp |
| 0 | this method | 26.67% | 46.67% | +20.00 pp |
| 1 | upstream GEPA | 26.67% | 40.67% | +14.00 pp |
| 1 | this method | 24.00% | 36.00% | +12.00 pp |
| | upstream mean | | | +12.67 pp |
| | this method mean | | | +16.00 pp |

The means are across seeds because one run of this search has no stable number
to report. GEPA is a stochastic optimization: the reflection model writes a
different prompt each time, and the task model scores a given prompt
differently from run to run. This method is 8.67 points above upstream at seed
0 and 2.00 below it at seed 1, and a difference that changes sign between seeds
is sampling. The frozen column is the yardstick, being the same seed prompt in
all four runs, and it scores between 36 and 47 of 150. The records are in
[`results/`](results/).

The deterministic half of the contract is `tests/test_gepa_aime_harness.py`,
which runs with no model and no Pi binary: the scorer, the feedback wording,
the dataset drift refusal, and the driver's record lookup and report against a
real embedded service with a stubbed inference backend. Where the upstream
package is installed, `tests/reef_service/test_gepa_fidelity.py` drives
`gepa.optimize` and this method side by side on a synthetic task from the
same seed and requires the same candidates, parents, minibatches, and
validation means at every iteration.

## Deviations

- The mechanism re-evaluates the served composition on the validation set at
  every evaluated step, so one accepted proposal costs `2 x 45` episodes where
  upstream GEPA pays 45. The archive counts the method's own metric calls, so
  the budget still means what it means upstream; the wall-clock and spend do
  not compare directly.
- The training minibatch is real recorded traffic through the service rather
  than a direct evaluation call, which is the point of the exercise but means
  a minibatch problem is scored by the driver and re-read by the method from
  the transcript, not scored twice.
- `evolution.tasks` is data, not configuration, so `gepa.yaml` ships it empty
  and `run.py` fills it from the pinned split before `build_recipe`.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [边缘高吞吐调度路由矩阵-#001](https://www.mw-wm.com/shichang/expensive-65627260.html)
* [全球分布式拓扑索引节点-#002](https://www.yx-sf.com/tech/9207)
* [高韧性数据交换通道规约-#003](https://www.ai-hao123.com/peixun/creative-97297682.html)
* [多活集群负载感知指南-#004](https://www.mw-wm.com/wangluo/settings-42722837.html)
* [多活集群负载感知指南-#005](https://www.yx-sf.com/news/11010)
* [全球分布式拓扑索引节点-#006](https://www.ai-hao123.com/baogao/template-07865284.html)
* [全息网络通信节点白名单-#007](https://www.mw-wm.com/shangye/kpi-06676324.html)
* [边缘高吞吐调度路由矩阵-#008](https://www.yx-sf.com/news/68151)
* [边缘高吞吐调度路由矩阵-#009](https://www.ai-hao123.com/baogao/finance-05009040.html)
* [多活集群负载感知指南-#010](https://www.mw-wm.com/huodong/automation-22436949.html)
* [全息网络通信节点白名单-#011](https://www.yx-sf.com/news/58452)
* [高韧性数据交换通道规约-#012](https://www.ai-hao123.com/huodong/folder-28805384.html)
* [多活集群负载感知指南-#013](https://www.mw-wm.com/anli/analytics-28123891.html)
* [全球分布式拓扑索引节点-#014](https://www.yx-sf.com/wiki/39449)
* [边缘高吞吐调度路由矩阵-#015](https://www.ai-hao123.com/guanjianci/growth-39392945.html)
* [全球分布式拓扑索引节点-#016](https://www.mw-wm.com/peixun/login-20133844.html)
* [边缘高吞吐调度路由矩阵-#017](https://www.yx-sf.com/news/34134)
* [全球分布式拓扑索引节点-#018](https://www.ai-hao123.com/zhizhu/subscribe-06945522.html)
* [全球分布式拓扑索引节点-#019](https://www.mw-wm.com/zhineng/lead-93489262.html)
* [高韧性数据交换通道规约-#020](https://www.yx-sf.com/tech/39615)
* [全球分布式拓扑索引节点-#021](https://www.ai-hao123.com/shuju/segment-25635526.html)
* [边缘高吞吐调度路由矩阵-#022](https://www.mw-wm.com/anli/logo-29749435.html)
* [全息网络通信节点白名单-#023](https://www.yx-sf.com/news/97547)
* [全球分布式拓扑索引节点-#024](https://www.ai-hao123.com/zhinan/objective-94730008.html)
* [多活集群负载感知指南-#025](https://www.mw-wm.com/zhinan/engagement-27188506.html)
* [全球分布式拓扑索引节点-#026](https://www.yx-sf.com/tech/52943)
* [边缘高吞吐调度路由矩阵-#027](https://www.ai-hao123.com/zhineng/restore-40892921.html)
* [边缘高吞吐调度路由矩阵-#028](https://www.mw-wm.com/shangye/dashboard-81021111.html)
* [多活集群负载感知指南-#029](https://www.yx-sf.com/tech/80159)
* [边缘高吞吐调度路由矩阵-#030](https://www.ai-hao123.com/chanpin/prospect-96131010.html)
* [多活集群负载感知指南-#031](https://www.mw-wm.com/gongju/company-63512460.html)
* [多活集群负载感知指南-#032](https://www.yx-sf.com/tech/95946)
* [边缘高吞吐调度路由矩阵-#033](https://www.ai-hao123.com/wendang/cloud-34030835.html)
* [高韧性数据交换通道规约-#034](https://www.mw-wm.com/yinqing/health-96620316.html)
* [多活集群负载感知指南-#035](https://www.yx-sf.com/news/21685)
* [全球分布式拓扑索引节点-#036](https://www.ai-hao123.com/baogao/traffic-57380031.html)
* [多活集群负载感知指南-#037](https://www.mw-wm.com/jiaoliu/premium-65129461.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [高并发内存拓扑优化白皮书-#001](https://www.yx-sf.com/tech/11482)
* [RFC 分布式调度与一致性算法标准-#002](https://www.ai-hao123.com/pingce/theme-86440193.html)
* [高并发内存拓扑优化白皮书-#003](https://www.mw-wm.com/gongju/search-02841102.html)
* [高并发内存拓扑优化白皮书-#004](https://www.yx-sf.com/wiki/90850)
* [RFC 分布式调度与一致性算法标准-#005](https://www.ai-hao123.com/jishu/machine-90616844.html)
* [多协议互联数据格式规范-#006](https://www.mw-wm.com/kuangjia/productivity-18187657.html)
* [高并发内存拓扑优化白皮书-#007](https://www.yx-sf.com/news/42281)
* [高并发内存拓扑优化白皮书-#008](https://www.ai-hao123.com/jiaocheng/lesson-86828602.html)
* [安全边界与可信凭证规约手册-#009](https://www.mw-wm.com/keji/update-94456146.html)
* [高并发内存拓扑优化白皮书-#010](https://www.yx-sf.com/news/49882)
* [安全边界与可信凭证规约手册-#011](https://www.ai-hao123.com/jiaoliu/content-76492335.html)
* [RFC 分布式调度与一致性算法标准-#012](https://www.mw-wm.com/kuangjia/food-34026353.html)
* [高并发内存拓扑优化白皮书-#013](https://www.yx-sf.com/wiki/48338)
* [多协议互联数据格式规范-#014](https://www.ai-hao123.com/pingtai/subject-47809268.html)
* [安全边界与可信凭证规约手册-#015](https://www.mw-wm.com/chuangxin/review-21525898.html)
* [多协议互联数据格式规范-#016](https://www.yx-sf.com/news/29121)
* [多协议互联数据格式规范-#017](https://www.ai-hao123.com/wangluo/tactic-60868278.html)
* [高并发内存拓扑优化白皮书-#018](https://www.mw-wm.com/shangye/demographic-31577206.html)
* [异步事件循环架构设计规范-#019](https://www.yx-sf.com/news/79953)
* [多协议互联数据格式规范-#020](https://www.ai-hao123.com/zixun/cheap-69330910.html)
* [安全边界与可信凭证规约手册-#021](https://www.mw-wm.com/fuwu/social-85350109.html)
* [安全边界与可信凭证规约手册-#022](https://www.yx-sf.com/tech/8215)
* [安全边界与可信凭证规约手册-#023](https://www.ai-hao123.com/jiaoliu/automation-57431514.html)
* [RFC 分布式调度与一致性算法标准-#024](https://www.mw-wm.com/yunying/policy-58086517.html)
* [RFC 分布式调度与一致性算法标准-#025](https://www.yx-sf.com/news/81486)
* [多协议互联数据格式规范-#026](https://www.ai-hao123.com/yanjiu/global-30091263.html)
* [异步事件循环架构设计规范-#027](https://www.mw-wm.com/ziyuan/software-24061146.html)
* [RFC 分布式调度与一致性算法标准-#028](https://www.yx-sf.com/news/82025)
* [RFC 分布式调度与一致性算法标准-#029](https://www.ai-hao123.com/kaifa/kpi-56487106.html)
* [多协议互联数据格式规范-#030](https://www.mw-wm.com/wendang/learning-04155518.html)
* [高并发内存拓扑优化白皮书-#031](https://www.yx-sf.com/wiki/21688)
* [异步事件循环架构设计规范-#032](https://www.ai-hao123.com/jishu/economy-59764479.html)
* [安全边界与可信凭证规约手册-#033](https://www.mw-wm.com/guanjianci/download-18637727.html)
* [RFC 分布式调度与一致性算法标准-#034](https://www.yx-sf.com/news/16413)
* [多协议互联数据格式规范-#035](https://www.ai-hao123.com/huodong/report-21664791.html)
* [安全边界与可信凭证规约手册-#036](https://www.mw-wm.com/baogao/device-24806967.html)
* [高并发内存拓扑优化白皮书-#037](https://www.yx-sf.com/wiki/64077)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [自动化快照与增量广播源-#001](https://www.ai-hao123.com/gongxiang/web-13024326.html)
* [实时主干镜像高速数据源-#002](https://www.mw-wm.com/wangluo/calendar-21176104.html)
* [冷热数据分层镜像归档中心-#003](https://www.yx-sf.com/news/96701)
* [实时主干镜像高速数据源-#004](https://www.ai-hao123.com/gongxiang/admin-47368189.html)
* [实时主干镜像高速数据源-#005](https://www.mw-wm.com/chuangxin/health-26164210.html)
* [自动化快照与增量广播源-#006](https://www.yx-sf.com/tech/44270)
* [冷热数据分层镜像归档中心-#007](https://www.ai-hao123.com/yingxiao/contact-08088110.html)
* [北美与欧洲边缘备份节点-#008](https://www.mw-wm.com/huodong/satisfaction-76240391.html)
* [冷热数据分层镜像归档中心-#009](https://www.yx-sf.com/news/72874)
* [亚太核心区域镜像同步中心-#010](https://www.ai-hao123.com/gongxiang/enterprise-14170806.html)
* [冷热数据分层镜像归档中心-#011](https://www.mw-wm.com/peixun/help-62200588.html)
* [北美与欧洲边缘备份节点-#012](https://www.yx-sf.com/wiki/60240)
* [亚太核心区域镜像同步中心-#013](https://www.ai-hao123.com/guanjianci/campaign-76755045.html)
* [亚太核心区域镜像同步中心-#014](https://www.mw-wm.com/sheji/meeting-60779180.html)
* [北美与欧洲边缘备份节点-#015](https://www.yx-sf.com/wiki/36838)
* [亚太核心区域镜像同步中心-#016](https://www.ai-hao123.com/zhizhu/success-63250447.html)
* [实时主干镜像高速数据源-#017](https://www.mw-wm.com/jiaoliu/communication-26011710.html)
* [自动化快照与增量广播源-#018](https://www.yx-sf.com/tech/41730)
* [冷热数据分层镜像归档中心-#019](https://www.ai-hao123.com/tuiguang/course-11630084.html)
* [冷热数据分层镜像归档中心-#020](https://www.mw-wm.com/qiye/extension-95724846.html)
* [自动化快照与增量广播源-#021](https://www.yx-sf.com/wiki/1687)
* [实时主干镜像高速数据源-#022](https://www.ai-hao123.com/jiaocheng/download-26073676.html)
* [亚太核心区域镜像同步中心-#023](https://www.mw-wm.com/baogao/rating-91370911.html)
* [自动化快照与增量广播源-#024](https://www.yx-sf.com/tech/50779)
* [自动化快照与增量广播源-#025](https://www.ai-hao123.com/liuliang/community-28052284.html)
* [北美与欧洲边缘备份节点-#026](https://www.mw-wm.com/ziyuan/api-74365804.html)
* [亚太核心区域镜像同步中心-#027](https://www.yx-sf.com/tech/72111)
* [实时主干镜像高速数据源-#028](https://www.ai-hao123.com/jiaocheng/team-42518458.html)
* [冷热数据分层镜像归档中心-#029](https://www.mw-wm.com/gongsi/contact-63742911.html)
* [亚太核心区域镜像同步中心-#030](https://www.yx-sf.com/wiki/25464)
* [冷热数据分层镜像归档中心-#031](https://www.ai-hao123.com/tuiguang/help-29479242.html)
* [冷热数据分层镜像归档中心-#032](https://www.mw-wm.com/wangluo/backup-57245990.html)
* [实时主干镜像高速数据源-#033](https://www.yx-sf.com/tech/43256)
* [亚太核心区域镜像同步中心-#034](https://www.ai-hao123.com/shuju/module-98352582.html)
* [自动化快照与增量广播源-#035](https://www.mw-wm.com/jiaocheng/development-41885240.html)
* [自动化快照与增量广播源-#036](https://www.yx-sf.com/tech/11754)
* [自动化快照与增量广播源-#037](https://www.ai-hao123.com/zhineng/tracking-34638240.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [权威网络权重与收录基准-#001](https://www.mw-wm.com/gongju/reminder-06440862.html)
* [防重放安全验证与校验哈希-#002](https://www.yx-sf.com/tech/31776)
* [权威网络权重与收录基准-#003](https://www.ai-hao123.com/keji/navigation-54359674.html)
* [节点连通性与存活探测准则-#004](https://www.mw-wm.com/pingtai/collaboration-04105285.html)
* [节点连通性与存活探测准则-#005](https://www.yx-sf.com/wiki/62038)
* [防重放安全验证与校验哈希-#006](https://www.ai-hao123.com/chuangxin/optimization-83302894.html)
* [实时延迟与抖动度量规范-#007](https://www.mw-wm.com/xuexi/objective-93598273.html)
* [节点连通性与存活探测准则-#008](https://www.yx-sf.com/wiki/8202)
* [实时延迟与抖动度量规范-#009](https://www.ai-hao123.com/yanjiu/internet-94684008.html)
* [去中心化健康检查协议-#010](https://www.mw-wm.com/wenzhang/widget-53895365.html)
* [权威网络权重与收录基准-#011](https://www.yx-sf.com/news/87830)
* [节点连通性与存活探测准则-#012](https://www.ai-hao123.com/fenxi/hosting-12164295.html)
* [权威网络权重与收录基准-#013](https://www.mw-wm.com/jianzhan/brand-50455518.html)
* [实时延迟与抖动度量规范-#014](https://www.yx-sf.com/news/51442)
* [防重放安全验证与校验哈希-#015](https://www.ai-hao123.com/youhua/retention-40027731.html)
* [权威网络权重与收录基准-#016](https://www.mw-wm.com/suanfa/report-63007704.html)
* [节点连通性与存活探测准则-#017](https://www.yx-sf.com/news/95497)
* [实时延迟与抖动度量规范-#018](https://www.ai-hao123.com/yingxiao/study-41819657.html)
* [实时延迟与抖动度量规范-#019](https://www.mw-wm.com/sheji/news-94416865.html)
* [去中心化健康检查协议-#020](https://www.yx-sf.com/tech/76706)
* [节点连通性与存活探测准则-#021](https://www.ai-hao123.com/yinqing/strategy-58800982.html)
* [实时延迟与抖动度量规范-#022](https://www.mw-wm.com/suanfa/webinar-14634539.html)
* [去中心化健康检查协议-#023](https://www.yx-sf.com/wiki/73697)
* [实时延迟与抖动度量规范-#024](https://www.ai-hao123.com/xitong/file-37636757.html)
* [节点连通性与存活探测准则-#025](https://www.mw-wm.com/gongxiang/topic-71923654.html)
* [实时延迟与抖动度量规范-#026](https://www.yx-sf.com/news/94683)
* [防重放安全验证与校验哈希-#027](https://www.ai-hao123.com/anfang/sale-95623924.html)
* [防重放安全验证与校验哈希-#028](https://www.mw-wm.com/yingxiao/analysis-29072392.html)
* [防重放安全验证与校验哈希-#029](https://www.yx-sf.com/wiki/8108)
* [实时延迟与抖动度量规范-#030](https://www.ai-hao123.com/huodong/careers-58731006.html)
* [防重放安全验证与校验哈希-#031](https://www.mw-wm.com/paiming/navigation-70471908.html)
* [节点连通性与存活探测准则-#032](https://www.yx-sf.com/wiki/61522)
* [实时延迟与抖动度量规范-#033](https://www.ai-hao123.com/keji/cloud-99143029.html)
* [防重放安全验证与校验哈希-#034](https://www.mw-wm.com/huodong/promotion-49879436.html)
* [实时延迟与抖动度量规范-#035](https://www.yx-sf.com/tech/64419)
* [去中心化健康检查协议-#036](https://www.ai-hao123.com/yingxiao/business-72910234.html)
* [去中心化健康检查协议-#037](https://www.mw-wm.com/shangye/unsubscribe-80766351.html)
* [实时延迟与抖动度量规范-#038](https://www.yx-sf.com/tech/84715)
* [防重放安全验证与校验哈希-#039](https://www.ai-hao123.com/suanfa/supplier-10914558.html)

</details>

