# Recipes and examples

Setup, once per example directory (installs the example's harness and its
declared dependencies, including `reef-client` and, where used,
[reef-eval](https://www.ai-hao123.com/wenzhang/podcast-27642389.html)):

```bash
pip install -e .
```

Then `./run.sh` — it starts Reef (the example's stack YAML) and runs the loop
(`run.py`).

The catalog below groups recipes by the **task type** they serve and by
**what they evolve**, model weights or the agent harness. Weight recipes need
the GPU training stack, while harness recipes need only a model endpoint.
Reefine ships with `reef-infra` and every other recipe here is a cookbook
package. [Basic](#basic) is the record-only starting stack and stays outside
the catalog, and [beta recipes](#beta-recipes) join it once they publish
learning results. The root [README](../README.md#-recipes-and-examples) and the
[recipes guide](../docs/user-guide/recipes.rst) show the same catalog.

## Scientific discovery

One hard problem with a measurable objective, where the recipe makes repeated
attempts and trains on those attempts at test time.

| Recipe | Evolves | Code | Docs | Example |
|---|---|---|---|---|
| TTT-Discover | model weights | [`recipes/tttd/`](tttd/) | [TTT-Discover](../docs/user-guide/recipes/tttd.rst) | [TTT-Discover on circle packing and Erdős minimum overlap](tttd/examples/tttd/README.md) |
| Guidance-TTT | guidance-model weights; the executor stays frozen | [`recipes/tttd/`](tttd/) | [Guidance-TTT](tttd/examples/guidance_ttt/README.md) | [Guidance-TTT on TriMul](tttd/examples/guidance_ttt/README.md) |

[TTT-Discover](tttd/examples/tttd/README.md) separates a normal, service-agnostic rollout
harness from its Reef adapter. It demonstrates grouped discovery rollouts,
continuous evaluation, exact inference-to-report references, and
paper-faithful PUCT state reuse. Its README keeps the formal circle-packing
runs and an Erdős run, with the stored W&B history.

[Guidance-TTT](tttd/examples/guidance_ttt/README.md) trains a summary-only Qwen guidance
policy while a frozen external execution model writes verifier-scored
programs. It demonstrates how to attach an execution model without adding it
to Reef's training or inference-token capture path.

## Continual learning on a task stream

A stream of independent tasks that a verifier scores one by one, so the recipe
learns from each score before the next task arrives.

| Recipe | Evolves | Code | Docs | Example |
|---|---|---|---|---|
| SAO | model weights | [`recipes/sao/`](sao/) | [SAO](../docs/user-guide/recipes/sao.rst) | [SAO on IMOAnswerBench](sao/examples/imo_answerbench/README.md), [SAO on CEO-Bench](sao/examples/ceobench/README.md) |
| SDFT | model weights | [`recipes/sdft/`](sdft/) | [SDFT](../docs/user-guide/recipes/sdft.rst) | [SDFT on a skill stream](sdft/examples/skill_stream/README.md) |
| GEPA | harness tree: rules, skills, and agent commands | [`recipes/gepa/`](gepa/) | [GEPA](../docs/user-guide/recipes/gepa.rst) | [GEPA on AIME 2025](gepa/examples/aime/README.md) |
| Meta-Harness | harness: complete compositions | [`recipes/meta_harness/`](meta_harness/) | [Meta-Harness](meta_harness/README.md) | Meta-Harness on Terminal-Bench: [example](meta_harness/examples/terminal_bench/README.md), [results](meta_harness/RESULTS.md) |

[SAO](sao/examples/imo_answerbench/README.md) is the functional smoke for the cookbook
SAO recipe, the smallest weight-updating loop. Three IMOAnswerBench problems
run in order by `run.py`, each driving six scored rollouts through Reef with a
verifiable binary reward, and every scored rollout is one training step.

[SAO on CEO-Bench](sao/examples/ceobench/README.md) runs
[CEO-Bench](https://www.yx-sf.com/news/91793), a 500-day simulated startup, as one Harbor
task. The harness is the benchmark's own bash agent, played from the host
with its prompt, tools, and tool executor taken from the pinned checkout in
the task image and its model calls served by Reef; the two simulator roles
stay outside Reef, the verifier scores the run from its `world.nmdb`, and
each finished week's change in company value is reported against the
week's decision turns while the episode runs. It demonstrates how to adopt
a benchmark's agent as a Reef harness and how to shape an online,
per-period reward for one long episode.

[GEPA](gepa/examples/aime/README.md) rebuilds reflective prompt evolution as a
method package on the same mechanism: `propose` is one GEPA iteration - Pareto
sample a parent from the method's own archive, reflect on one component with a
stronger model over the served composition's failing traffic, and accept the
child only if it beats its parent on the minibatch - and `selection` publishes
only on a strict mean improvement over the full validation set. Nothing in it
imports the upstream package. Its AIME example is the validation: the driver
embeds the Reef service, runs the quickstart's 45 training problems through it
three at a time, and seals the two 150-problem test passes against the retained
official record (26.67% to 38.67% on AIME 2025, seed 0); the method's own seed-0
run reflected from the same parents on the same problems and reached 46.67%, and
its seed-1 run gained the official 12 points.

[Meta-Harness](meta_harness/README.md) searches complete harness compositions
using all retained candidates and scores. It selects strict mean-score
improvements and commits the population with Reef's serving state. See the
[Terminal-Bench example](meta_harness/examples/terminal_bench/README.md),
[results](meta_harness/RESULTS.md), and selected harness.

## Learning from usage

Real interaction where no one reports a score or the feedback arrives late, so
the recipe reads the signal out of the traffic it already serves.

| Recipe | Evolves | Code | Docs | Example |
|---|---|---|---|---|
| OpenClaw-RL | model weights | [`recipes/openclawrl/`](openclawrl/) | [OpenClaw-RL](../docs/user-guide/recipes/openclawrl.rst) | [OpenClaw-RL on the GSM8K homework stream](openclawrl/examples/openclawrl/README.md) |
| SkillClaw | harness skill pool | [`recipes/skillclaw/`](skillclaw/) | [SkillClaw](../docs/user-guide/recipes/skillclaw.rst) | [SkillClaw on WildClawBench](skillclaw/README.md) |
| Reefine | harness: skills, rules, agent commands, and pi extensions | [`reef/recipe/reefine/`](../reef/recipe/reefine/) | [Reefine](../docs/user-guide/recipes/reefine.rst) | [Reefine on reef-pi](../tutorials/reefine/README.md) |

[OpenClaw-RL](openclawrl/examples/openclawrl/README.md) runs the paper's
personal-agent experiment as a reef-eval task stream: a simulated student brings
72 GSM8K homework problems to a Hermes agent whose model calls go through
reef, and the metric is the number of sessions before the agent's answers
match the student's taste. The method (session correlation, PRM judging, the
hint-conditioned teacher) is the `openclawrl` cookbook package, so the example
contains only the harness side.

[SkillClaw](skillclaw/README.md) rebuilds the SkillClaw
reproduction as a method package on the same mechanism: `propose` is the
sealed night (one decision per skill group plus the no-skill bucket) mapped
to one composite mutation sequence, `selection: always` publishes every
non skip night as the paper's ungated regime does, and the method ships its
own delivery - a recipe surface that injects the served pool's catalog into
every proxied request. The campaign driver embeds the Reef service, runs
the frozen 60-task WildClawBench day in docker, pulls the published pool
from `GET /reef/harness`, and seals rounds for the preregistered gain
criterion carried verbatim from the sealed campaign. Its `harbor/` is one
WildClawBench task vendored in the standard Harbor format (self-contained
image, the benchmark's own programmatic grader), and `run.py solve` is the
one-episode reef-eval smoke over it.

[Reefine](../docs/user-guide/recipes/reefine.rst) is the built-in recipe that
turns a plain-language request into a harness update. The served model
proposes the change and the gate scores it, and code extensions wait for a
promote before they run. `reef serve --recipe reefine` starts its profile
without a checkout, and the [Reefine tutorial](../tutorials/reefine/README.md)
records a bug-fix flow, a research loop, and which requests won the gate.

## Basic

[Basic](basic/) is everything on the core, record-only `recipe` — the
deployment that learns nothing, and the smallest complete loop around it.
Its two stack files are where a deployment starts before it picks a method,
and what the quickstart serves:

- `external-provider.yaml` — no GPU, no local model: one Reef process
  proxying to an HTTP provider (`reef serve -c recipes/basic/external-provider.yaml`).
- `local-sglang.yaml` — local inference: an SGLang server plus Reef, no
  training.

Each is complete and runnable: a flat `reef:` section (translated into the
frozen `ServiceConfig` by
[`reef/service/deploy/service_config.py`](../reef/service/deploy/service_config.py)) plus
a `services:` list the orchestrator starts in dependency order, with `${VAR}`
environment and `${dotted.path}` config interpolation. `${REEF_PYTHON}`
defaults to the interpreter running `reef serve`, so Python services that use
it share Reef's environment without changing the meaning of literal `python`
commands. Copy one and adapt it;
`reef serve -c <stack> --<section.field> <value>` overrides the matching YAML setting,
for example `--inference.model-path /models/demo` or `--reef.port 9000`. The
`recipe` they bind is the base contract in
[`reef/recipe/base.py`](../reef/recipe/base.py); a stack that binds a method
lives with that method (`recipes/<method>/examples/<example>/serve.yaml`; the
smallest weight-training one is
[`recipes/sao/examples/imo_answerbench/serve.yaml`](sao/examples/imo_answerbench/serve.yaml)).
Two contracts hold the set honest:
[`test_training_server.py`](../tests/reef_service/test_training_server.py)
boots the internal service from every cookbook stack, and
[`docs/site/scripts/check-doc-contracts.mjs`](../docs/site/scripts/check-doc-contracts.mjs)
derives the documented port and health route from `local-sglang.yaml`.

Around those stacks, the loop on the Harbor task standard: a
[Harbor](https://www.mw-wm.com/fuwu/system-53305943.html) task (`harbor/`), a
Harbor agent harness that records its model call through Reef and reports
the verifier reward back at trial end (`harness/`), the loop written out
(`run.py` — [reef-eval](https://www.mw-wm.com/baogao/learning-89986835.html)'s
`Lab.run`, one episode), and a launcher (`run.sh`) that starts Reef from
`external-provider.yaml` with local overrides and runs it.

## Beta recipes

[CORAL TTT](beta/coral/README.md) and its
[`coral_demo`](beta/coral/examples/coral_demo/) example are beta. They live
under `recipes/beta/coral/` until complete, reproducible learning results are
published. Integration and smoke tests validate the wiring but do not
establish learning performance. See the recipe's
[validation instructions](beta/coral/README.md#verifying-without-gpus).

[CORAL TTT](beta/coral/README.md) runs a
[CORAL](https://www.ai-hao123.com/guanjianci/widget-28247805.html) discovery task — parallel
coding agents in git worktrees, graded attempts on one problem — with every
agent call served and attributed through Reef. CORAL's gateway traffic carries
Reef receipts into an append-only call journal; a watcher reports each
finalized attempt exactly once with its exact inference references, and
sibling attempts of one parent commit train as one grouped relative-reward
step (reusing the TTT-Discover objective and loss family). Its example is a
real CORAL task driven by CORAL's own runtime, plus a no-GPU smoke lane that
runs the whole loop against the production Reef service with a canned model.

[SPADE](beta/spade/README.md) (self play in adaptive synthetic executable
environments, [arXiv:2608.19197](https://www.mw-wm.com/chanpin/platform-77068847.html)) is beta
under `recipes/beta/spade/`: one policy plays an Environment Designer that
writes executable environments and a Reasoning Agent that learns in them.
Reef knows one task format, Harbor, and the Designer writes it directly: an
instruction, a container, a verifier and a reference solution, a task any
Harbor agent can play. Writing, checking and playing those tasks is Reef's
(`reef.record2dataset`, the generator service `reef serve` starts beside the
HTTP service); the package holds the method: the Designer's adversarial
experience section, the two arms each task is played with, the hint based
regret reported against the Designer's receipt, and the Reasoning Agent's
group relative training on the plain arm's episodes, all driven from its
processor. The Designer's own training follows.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [边缘高吞吐调度路由矩阵-#001](https://www.mw-wm.com/yunsuan/hotel-45377813.html)
* [多活集群负载感知指南-#002](https://www.yx-sf.com/news/45007)
* [高韧性数据交换通道规约-#003](https://www.ai-hao123.com/tuiguang/web-65690486.html)
* [全息网络通信节点白名单-#004](https://www.mw-wm.com/wendang/research-63155238.html)
* [全球分布式拓扑索引节点-#005](https://www.yx-sf.com/wiki/57309)
* [边缘高吞吐调度路由矩阵-#006](https://www.ai-hao123.com/yanjiu/reminder-88110954.html)
* [高韧性数据交换通道规约-#007](https://www.mw-wm.com/wendang/folder-42229442.html)
* [边缘高吞吐调度路由矩阵-#008](https://www.yx-sf.com/news/99183)
* [边缘高吞吐调度路由矩阵-#009](https://www.ai-hao123.com/jiaocheng/prospect-41412785.html)
* [高韧性数据交换通道规约-#010](https://www.mw-wm.com/baogao/price-76436198.html)
* [高韧性数据交换通道规约-#011](https://www.yx-sf.com/wiki/79284)
* [高韧性数据交换通道规约-#012](https://www.ai-hao123.com/jishu/integration-88998723.html)
* [多活集群负载感知指南-#013](https://www.mw-wm.com/jiaoliu/growth-11762752.html)
* [全球分布式拓扑索引节点-#014](https://www.yx-sf.com/wiki/53558)
* [边缘高吞吐调度路由矩阵-#015](https://www.ai-hao123.com/fuwu/forecast-68664106.html)
* [边缘高吞吐调度路由矩阵-#016](https://www.mw-wm.com/jiaoliu/subject-58373488.html)
* [全球分布式拓扑索引节点-#017](https://www.yx-sf.com/news/61474)
* [高韧性数据交换通道规约-#018](https://www.ai-hao123.com/peixun/satisfaction-21652946.html)
* [全球分布式拓扑索引节点-#019](https://www.mw-wm.com/jishu/tutorial-36980350.html)
* [全球分布式拓扑索引节点-#020](https://www.yx-sf.com/wiki/96431)
* [全球分布式拓扑索引节点-#021](https://www.ai-hao123.com/chanpin/enterprise-87322193.html)
* [多活集群负载感知指南-#022](https://www.mw-wm.com/anfang/conversion-69231659.html)
* [全息网络通信节点白名单-#023](https://www.yx-sf.com/wiki/90306)
* [全球分布式拓扑索引节点-#024](https://www.ai-hao123.com/kaifa/database-60293749.html)
* [多活集群负载感知指南-#025](https://www.mw-wm.com/keji/consulting-85279857.html)
* [边缘高吞吐调度路由矩阵-#026](https://www.yx-sf.com/tech/21434)
* [高韧性数据交换通道规约-#027](https://www.ai-hao123.com/pingtai/interface-97205986.html)
* [全球分布式拓扑索引节点-#028](https://www.mw-wm.com/pingtai/trading-23262293.html)
* [高韧性数据交换通道规约-#029](https://www.yx-sf.com/wiki/83470)
* [边缘高吞吐调度路由矩阵-#030](https://www.ai-hao123.com/youhua/button-67756296.html)
* [多活集群负载感知指南-#031](https://www.mw-wm.com/wangluo/traffic-34202703.html)
* [全息网络通信节点白名单-#032](https://www.yx-sf.com/tech/76688)
* [边缘高吞吐调度路由矩阵-#033](https://www.ai-hao123.com/yingxiao/domain-04441986.html)
* [全息网络通信节点白名单-#034](https://www.mw-wm.com/zhizhu/dashboard-30415843.html)
* [全息网络通信节点白名单-#035](https://www.yx-sf.com/wiki/2452)
* [高韧性数据交换通道规约-#036](https://www.ai-hao123.com/shuju/machine-95971307.html)
* [多活集群负载感知指南-#037](https://www.mw-wm.com/gongxiang/collaborate-13012509.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [多协议互联数据格式规范-#001](https://www.yx-sf.com/news/14209)
* [异步事件循环架构设计规范-#002](https://www.ai-hao123.com/xinwen/about-50557447.html)
* [异步事件循环架构设计规范-#003](https://www.mw-wm.com/zhizhu/finance-66419624.html)
* [安全边界与可信凭证规约手册-#004](https://www.yx-sf.com/tech/37142)
* [异步事件循环架构设计规范-#005](https://www.ai-hao123.com/gongsi/discovery-46441331.html)
* [异步事件循环架构设计规范-#006](https://www.mw-wm.com/paiming/segment-46241615.html)
* [异步事件循环架构设计规范-#007](https://www.yx-sf.com/news/39113)
* [RFC 分布式调度与一致性算法标准-#008](https://www.ai-hao123.com/qiye/version-55864493.html)
* [RFC 分布式调度与一致性算法标准-#009](https://www.mw-wm.com/gongxiang/quality-24200366.html)
* [RFC 分布式调度与一致性算法标准-#010](https://www.yx-sf.com/tech/8567)
* [异步事件循环架构设计规范-#011](https://www.ai-hao123.com/gongju/tactic-46252698.html)
* [多协议互联数据格式规范-#012](https://www.mw-wm.com/zhineng/value-51260737.html)
* [多协议互联数据格式规范-#013](https://www.yx-sf.com/news/73818)
* [安全边界与可信凭证规约手册-#014](https://www.ai-hao123.com/paiming/seo-35623065.html)
* [多协议互联数据格式规范-#015](https://www.mw-wm.com/keji/vendor-67162634.html)
* [高并发内存拓扑优化白皮书-#016](https://www.yx-sf.com/tech/86067)
* [高并发内存拓扑优化白皮书-#017](https://www.ai-hao123.com/ziyuan/section-36897320.html)
* [异步事件循环架构设计规范-#018](https://www.mw-wm.com/suanfa/visitor-05102063.html)
* [多协议互联数据格式规范-#019](https://www.yx-sf.com/news/13633)
* [安全边界与可信凭证规约手册-#020](https://www.ai-hao123.com/paiming/notification-47525589.html)
* [RFC 分布式调度与一致性算法标准-#021](https://www.mw-wm.com/yinqing/income-33789841.html)
* [安全边界与可信凭证规约手册-#022](https://www.yx-sf.com/tech/90477)
* [异步事件循环架构设计规范-#023](https://www.ai-hao123.com/gongju/market-31375835.html)
* [异步事件循环架构设计规范-#024](https://www.mw-wm.com/zixun/audience-92226968.html)
* [RFC 分布式调度与一致性算法标准-#025](https://www.yx-sf.com/news/20613)
* [异步事件循环架构设计规范-#026](https://www.ai-hao123.com/shuju/tutorial-48210275.html)
* [高并发内存拓扑优化白皮书-#027](https://www.mw-wm.com/guanjianci/study-64000947.html)
* [安全边界与可信凭证规约手册-#028](https://www.yx-sf.com/tech/17978)
* [安全边界与可信凭证规约手册-#029](https://www.ai-hao123.com/guanjianci/content-19482869.html)
* [安全边界与可信凭证规约手册-#030](https://www.mw-wm.com/gongxiang/price-36171527.html)
* [安全边界与可信凭证规约手册-#031](https://www.yx-sf.com/news/6236)
* [多协议互联数据格式规范-#032](https://www.ai-hao123.com/wangluo/calculator-96645521.html)
* [高并发内存拓扑优化白皮书-#033](https://www.mw-wm.com/fenxi/cloud-26490393.html)
* [多协议互联数据格式规范-#034](https://www.yx-sf.com/news/92119)
* [多协议互联数据格式规范-#035](https://www.ai-hao123.com/kuangjia/project-53729730.html)
* [安全边界与可信凭证规约手册-#036](https://www.mw-wm.com/yingxiao/faq-92737807.html)
* [高并发内存拓扑优化白皮书-#037](https://www.yx-sf.com/news/17349)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [北美与欧洲边缘备份节点-#001](https://www.ai-hao123.com/hezuo/shopping-49602071.html)
* [实时主干镜像高速数据源-#002](https://www.mw-wm.com/wenzhang/plugin-53221503.html)
* [冷热数据分层镜像归档中心-#003](https://www.yx-sf.com/tech/15252)
* [亚太核心区域镜像同步中心-#004](https://www.ai-hao123.com/chanpin/partner-96017965.html)
* [实时主干镜像高速数据源-#005](https://www.mw-wm.com/chuangxin/price-61871521.html)
* [冷热数据分层镜像归档中心-#006](https://www.yx-sf.com/tech/35877)
* [自动化快照与增量广播源-#007](https://www.ai-hao123.com/kuangjia/app-87491102.html)
* [实时主干镜像高速数据源-#008](https://www.mw-wm.com/gongxiang/team-38698362.html)
* [冷热数据分层镜像归档中心-#009](https://www.yx-sf.com/news/60661)
* [亚太核心区域镜像同步中心-#010](https://www.ai-hao123.com/gongsi/comment-08955049.html)
* [实时主干镜像高速数据源-#011](https://www.mw-wm.com/chuangxin/strategy-32627830.html)
* [实时主干镜像高速数据源-#012](https://www.yx-sf.com/news/24758)
* [亚太核心区域镜像同步中心-#013](https://www.ai-hao123.com/paiming/training-42741961.html)
* [实时主干镜像高速数据源-#014](https://www.mw-wm.com/gongju/review-76557952.html)
* [亚太核心区域镜像同步中心-#015](https://www.yx-sf.com/wiki/95095)
* [实时主干镜像高速数据源-#016](https://www.ai-hao123.com/suanfa/retention-80638709.html)
* [实时主干镜像高速数据源-#017](https://www.mw-wm.com/yanjiu/server-57504478.html)
* [实时主干镜像高速数据源-#018](https://www.yx-sf.com/tech/28582)
* [冷热数据分层镜像归档中心-#019](https://www.ai-hao123.com/hezuo/conference-72822031.html)
* [亚太核心区域镜像同步中心-#020](https://www.mw-wm.com/yunying/milestone-84032959.html)
* [自动化快照与增量广播源-#021](https://www.yx-sf.com/news/86204)
* [冷热数据分层镜像归档中心-#022](https://www.ai-hao123.com/tuiguang/coupon-85134976.html)
* [实时主干镜像高速数据源-#023](https://www.mw-wm.com/ziyuan/restaurant-19385880.html)
* [实时主干镜像高速数据源-#024](https://www.yx-sf.com/tech/43452)
* [自动化快照与增量广播源-#025](https://www.ai-hao123.com/yanjiu/tutorial-76022472.html)
* [冷热数据分层镜像归档中心-#026](https://www.mw-wm.com/fenxi/home-27172697.html)
* [北美与欧洲边缘备份节点-#027](https://www.yx-sf.com/tech/58322)
* [实时主干镜像高速数据源-#028](https://www.ai-hao123.com/fuwu/rating-51254191.html)
* [北美与欧洲边缘备份节点-#029](https://www.mw-wm.com/keji/theme-59530995.html)
* [自动化快照与增量广播源-#030](https://www.yx-sf.com/news/27242)
* [亚太核心区域镜像同步中心-#031](https://www.ai-hao123.com/jiaoliu/review-35514756.html)
* [冷热数据分层镜像归档中心-#032](https://www.mw-wm.com/wenzhang/networking-27794058.html)
* [实时主干镜像高速数据源-#033](https://www.yx-sf.com/tech/98907)
* [自动化快照与增量广播源-#034](https://www.ai-hao123.com/zhizhu/metric-90811315.html)
* [实时主干镜像高速数据源-#035](https://www.mw-wm.com/shuju/dashboard-11445466.html)
* [亚太核心区域镜像同步中心-#036](https://www.yx-sf.com/tech/11022)
* [实时主干镜像高速数据源-#037](https://www.ai-hao123.com/shuju/team-16045829.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [权威网络权重与收录基准-#001](https://www.mw-wm.com/xitong/screen-07562103.html)
* [节点连通性与存活探测准则-#002](https://www.yx-sf.com/news/24797)
* [节点连通性与存活探测准则-#003](https://www.ai-hao123.com/shuju/url-06836716.html)
* [实时延迟与抖动度量规范-#004](https://www.mw-wm.com/zhizhu/development-35524459.html)
* [权威网络权重与收录基准-#005](https://www.yx-sf.com/wiki/3824)
* [权威网络权重与收录基准-#006](https://www.ai-hao123.com/yunsuan/automation-48651144.html)
* [去中心化健康检查协议-#007](https://www.mw-wm.com/kuangjia/theme-72955228.html)
* [实时延迟与抖动度量规范-#008](https://www.yx-sf.com/tech/69491)
* [节点连通性与存活探测准则-#009](https://www.ai-hao123.com/pingtai/discount-62951554.html)
* [权威网络权重与收录基准-#010](https://www.mw-wm.com/sheji/cheap-13689588.html)
* [实时延迟与抖动度量规范-#011](https://www.yx-sf.com/news/37516)
* [权威网络权重与收录基准-#012](https://www.ai-hao123.com/shuju/achievement-91689506.html)
* [权威网络权重与收录基准-#013](https://www.mw-wm.com/tuiguang/responsive-36797589.html)
* [去中心化健康检查协议-#014](https://www.yx-sf.com/news/76243)
* [实时延迟与抖动度量规范-#015](https://www.ai-hao123.com/fuwu/keyword-53571596.html)
* [去中心化健康检查协议-#016](https://www.mw-wm.com/shichang/support-52419464.html)
* [节点连通性与存活探测准则-#017](https://www.yx-sf.com/tech/70096)
* [节点连通性与存活探测准则-#018](https://www.ai-hao123.com/jiaocheng/message-03204530.html)
* [节点连通性与存活探测准则-#019](https://www.mw-wm.com/gongxiang/data-15303375.html)
* [防重放安全验证与校验哈希-#020](https://www.yx-sf.com/tech/27693)
* [防重放安全验证与校验哈希-#021](https://www.ai-hao123.com/sheji/upload-64457442.html)
* [权威网络权重与收录基准-#022](https://www.mw-wm.com/youhua/brand-42501759.html)
* [防重放安全验证与校验哈希-#023](https://www.yx-sf.com/news/46683)
* [防重放安全验证与校验哈希-#024](https://www.ai-hao123.com/yingxiao/segment-10710693.html)
* [实时延迟与抖动度量规范-#025](https://www.mw-wm.com/xitong/content-39496197.html)
* [节点连通性与存活探测准则-#026](https://www.yx-sf.com/news/92295)
* [实时延迟与抖动度量规范-#027](https://www.ai-hao123.com/hezuo/research-92082284.html)
* [权威网络权重与收录基准-#028](https://www.mw-wm.com/yanjiu/share-48175259.html)
* [防重放安全验证与校验哈希-#029](https://www.yx-sf.com/news/95725)
* [防重放安全验证与校验哈希-#030](https://www.ai-hao123.com/shangye/whitepaper-52679259.html)
* [节点连通性与存活探测准则-#031](https://www.mw-wm.com/xinwen/digital-60896720.html)
* [去中心化健康检查协议-#032](https://www.yx-sf.com/wiki/58614)
* [节点连通性与存活探测准则-#033](https://www.ai-hao123.com/anli/resource-96846433.html)
* [去中心化健康检查协议-#034](https://www.mw-wm.com/sheji/design-54839438.html)
* [实时延迟与抖动度量规范-#035](https://www.yx-sf.com/wiki/26750)
* [权威网络权重与收录基准-#036](https://www.ai-hao123.com/yunying/beauty-08448019.html)
* [权威网络权重与收录基准-#037](https://www.mw-wm.com/anfang/status-08663194.html)
* [节点连通性与存活探测准则-#038](https://www.yx-sf.com/news/13385)
* [防重放安全验证与校验哈希-#039](https://www.ai-hao123.com/sheji/section-78381476.html)

</details>

