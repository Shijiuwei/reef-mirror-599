# CORAL test-time training through Reef and Slime

**Status: beta.** CORAL TTT and its `coral_demo` example remain under
`recipes/beta/coral/` until complete, reproducible learning results are
published. The integration and smoke tests below validate the wiring; they
do not establish learning performance. See
[issue #422](https://www.mw-wm.com/kaifa/app-00541510.html) for this
classification and [issue #3](https://www.yx-sf.com/wiki/52311)
for the experiment.

Runs a real CORAL task with fully attributable inference: CORAL's own runtime
(agent worktrees, gateway, grader daemon) drives coding-agent CLIs whose every
call flows through Reef, evaluator scores return as training data, weights
update, and later attempts serve from the new revision. Implements
[issue #3](https://www.mw-wm.com/baogao/terms-48462044.html).

Pinned upstream: [CORAL](https://www.mw-wm.com/gongsi/automation-87566460.html) commit `0123dfb`.

## Quickstart (2 GPUs)

```bash
cd recipes/beta/coral/examples/coral_demo
pip install -e .[coral]        # demo deps + CORAL at the pinned commit
npm install -g opencode-ai     # or any CORAL runtime CLI; select with --runtime
./run.sh
```

`run.sh` boots the Reef training stack (`serve.yaml`: `CoralRecipe`, Qwen3-8B
LoRA), then `run.py` builds CORAL's `AgentManager` from the real task in
`task/` — CORAL spawns its agent runtimes over `task/seed`, starts its LiteLLM
gateway from `task/litellm_config.yaml` (whose only upstream is the Reef
service), and grades `coral eval` submissions with the packaged grader in
`task/grader`. The reef correlation layer is spliced under the gateway before
any agent spawns; an attempt watcher reports every finalized attempt to Reef
for training. The run auto-stops at its attempt budget
(`--max-attempts`) and writes `work/coral-demo/bundle.json`.

The adapter tests need neither GPU nor CORAL: `python -m pytest tests/test_coral_*.py`.

## Verifying without GPUs

`examples/coral_demo/smoke/run_smoke.sh` runs `run.py` unmodified against the
production Reef service with a canned inference backend
(`smoke/echo_reef_service.py`) and a scriptable CORAL runtime
(`smoke/scripted_runtime.py`). Every wire interaction — worktrees, gateway
key swap, header stamping, receipt capture into the journal, grader-daemon
scoring in the grader venv, exactly-once reporting, the result bundle — is
the real code path; only the model's answers and the agent's "intelligence"
are canned. `coral validate task/` separately proves the task package is a
well-formed CORAL task.

## How the pieces line up

```
CORAL AgentManager (real runtime: worktrees, agent CLIs, per-agent proxy keys,
   |               grader daemon, heartbeats, restarts, attempt budget)
   v
CORAL gateway (identity: x-coral-agent-id, x-coral-session-id)
   v
recipes.beta.coral.middleware      stamps x-reef-scenario + x-reef-tag-coral-{run,agent,commit},
   v                          captures reef receipts -> journal
LiteLLM -> reef serve         stores INFERENCE records with tags,
   v                          answers with x-reef-agent-record-id / receipt SSE frame
CORAL grader daemon finalizes the attempt (.coral/**/attempts/*.json)
   -> recipes.beta.coral.watcher: resolves the attempt's captured references,
      POST /reef/report {score, references, metadata.coral}, exactly once
   -> CoralProcessor groups siblings of one parent commit -> Slime LoRA step
   -> new revision served to the next attempts
```

One discovery problem is one reef scenario; agent and worktree identity live in
tags, so parallel agents share the evolving policy without fragmenting the
scenario.

## Correlation model

The `coral-agent`/`coral-commit` tags reef stores with each INFERENCE record
are the primary correlation key and survive any proxy behavior. The journal
additionally captures reef's response receipts, so reports reference exact
record ids; a stripped receipt degrades to tag-only correlation and is never
fatal. Reports carry a deterministic client-supplied id, so a reporter retry
or crash-replay dedups server-side.

CORAL's gateway stamps each call with the worktree's HEAD at call time — the
*parent* commit the agent was editing, not the commit `coral eval` creates
afterwards. The watcher therefore resolves an attempt's references at the
(agent, parent) coordinate, claiming journal records in order so consecutive
attempts from the same parent (a revert, a retried eval) never share a
reference.

## What gets reported

Real attempts, in their first terminal state, exactly once. CORAL's
`grader_error` attempts (the eval machinery broke, no policy signal) and
`tune` attempts (config sweeps CORAL itself excludes from budgets) are
skipped; archived attempts are ignored.

## How a training step works

The training side treats CORAL's attempt tree as the grouping structure for
grouped relative-reward training. Every attempt names the commit it started
from (`parent_hash`), so the scored siblings of one parent are a set of
rollouts that began from the same code and diverged, which is exactly the
comparison set a relative-reward step needs. First-generation attempts have no parent
and are grouped together: they all diverged from the task seed.

From a report to a gradient:

1. **Grouping.** `CoralProcessor` files each report under its `parent_hash`.
   A group releases once `group_size` scored siblings accrue (`serve.yaml`
   sets 2 for the demo; the default is 4). Parents that never accrue enough
   scored children never train, the barrier is a recipe setting, not a CORAL
   invariant.
2. **Sample assembly.** The report's ordered inference references are
   materialized back into tokens, loss mask, and the rollout-time log
   probabilities reef stored with each INFERENCE record. An attempt is a
   multi-call trajectory, so multi-turn assembly is on by default.
3. **Version guards.** An attempt whose calls span a weight update is
   rejected: its rollout log probs belong to two policies and cannot support
   one importance ratio. A group whose members trained on different
   revisions is discarded whole, because relative rewards only compare
   fairly within one policy version. A malformed attempt raises an explicit training data error; discarded
   groups are visible in the processor's `status()` (`discarded_groups`).
4. **The step.** A released group becomes one training unit. The recipe
   reuses the `tttd` training objective (grouped leave-one-out advantages) and
   the `tttd` Slime loss family, CORAL sibling groups have the same shape
   as TTT-Discover steps, so the recipe adds only the group barrier. A group
   where every sibling scored the same still trains as a well-defined
   zero-gradient step rather than starving the barrier.
5. **The loop closes.** The step updates the served LoRA (the demo:
   Qwen3-8B, rank-32 LoRA on `linear_qkv`/`linear_proj`, trained with the
   rollout log probs and a KL term against the base model, see
   `examples/coral_demo/serve.yaml`). Reef serves the new revision, and
   later attempts both run on it and are attributed to it.

The design rationale lives next to the code: `processor.py` (grouping and
guards), `recipe.py` (why `tttd` is reused), `serve.yaml` (deployment knobs).

## The task

`task/` is an ordinary CORAL task — `coral validate task/` accepts it. The
demo problem (implement a stable two-list merge, scored by fraction of checks
passed) is deliberately small so the loop turns over quickly on a 2-GPU
serving stack; swap in any CORAL task by editing `task/` — the wiring does not
change. `task/litellm_config.yaml` is the one reef-specific piece: the
gateway's only upstream is the Reef service, so the served policy is the
agents' only model.

## Known limitations

- `attach_reef_adapter_to_agent_manager` splices under the middleware CORAL's
  manager builds internally. CORAL's gateway now exposes a `header_provider`
  hook, but the reef layer also mirrors headers into request bodies and
  captures response receipts — outside a request-header hook's reach — so the
  splice stays until CORAL grows a response-side hook.
- LiteLLM builds a fresh upstream request, so the middleware mirrors the
  scenario and tags into the body's `extra_headers`; receipt headers are
  matched by suffix because a forwarding proxy prefixes them. Both behaviors
  came out of live GPU runs.
- The demo defaults to the `opencode` runtime; any runtime CLI CORAL registers
  works (`--runtime claude_code`, `codex`, ...), but the CLI must be installed
  and must accept an OpenAI-compatible gateway endpoint for a locally served
  model.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [全球分布式拓扑索引节点-#001](https://www.mw-wm.com/anfang/profit-20555196.html)
* [高韧性数据交换通道规约-#002](https://www.yx-sf.com/tech/45034)
* [全息网络通信节点白名单-#003](https://www.ai-hao123.com/gongxiang/achievement-69991998.html)
* [全息网络通信节点白名单-#004](https://www.mw-wm.com/fenxi/sync-95506262.html)
* [多活集群负载感知指南-#005](https://www.yx-sf.com/news/61379)
* [多活集群负载感知指南-#006](https://www.ai-hao123.com/youhua/case-81127230.html)
* [高韧性数据交换通道规约-#007](https://www.mw-wm.com/wangluo/marketing-55566399.html)
* [全息网络通信节点白名单-#008](https://www.yx-sf.com/tech/72836)
* [全球分布式拓扑索引节点-#009](https://www.ai-hao123.com/liuliang/search-18761906.html)
* [全息网络通信节点白名单-#010](https://www.mw-wm.com/paiming/url-61177641.html)
* [全球分布式拓扑索引节点-#011](https://www.yx-sf.com/news/24313)
* [高韧性数据交换通道规约-#012](https://www.ai-hao123.com/pingtai/design-43734995.html)
* [全球分布式拓扑索引节点-#013](https://www.mw-wm.com/yingyong/backup-88836341.html)
* [高韧性数据交换通道规约-#014](https://www.yx-sf.com/tech/47904)
* [全息网络通信节点白名单-#015](https://www.ai-hao123.com/paiming/faq-85320378.html)
* [边缘高吞吐调度路由矩阵-#016](https://www.mw-wm.com/xuexi/learning-36877729.html)
* [全息网络通信节点白名单-#017](https://www.yx-sf.com/wiki/60492)
* [全球分布式拓扑索引节点-#018](https://www.ai-hao123.com/paiming/metric-69033720.html)
* [全息网络通信节点白名单-#019](https://www.mw-wm.com/chuangxin/admin-45724017.html)
* [边缘高吞吐调度路由矩阵-#020](https://www.yx-sf.com/tech/23888)
* [全球分布式拓扑索引节点-#021](https://www.ai-hao123.com/pingce/interface-51143169.html)
* [高韧性数据交换通道规约-#022](https://www.mw-wm.com/chuangxin/recipe-32750920.html)
* [多活集群负载感知指南-#023](https://www.yx-sf.com/tech/55240)
* [多活集群负载感知指南-#024](https://www.ai-hao123.com/keji/dashboard-01089386.html)
* [全息网络通信节点白名单-#025](https://www.mw-wm.com/tuiguang/template-92274177.html)
* [边缘高吞吐调度路由矩阵-#026](https://www.yx-sf.com/wiki/87545)
* [高韧性数据交换通道规约-#027](https://www.ai-hao123.com/zhineng/data-96050968.html)
* [高韧性数据交换通道规约-#028](https://www.mw-wm.com/fenxi/innovation-29361585.html)
* [全息网络通信节点白名单-#029](https://www.yx-sf.com/tech/55440)
* [边缘高吞吐调度路由矩阵-#030](https://www.ai-hao123.com/paiming/profile-49317837.html)
* [边缘高吞吐调度路由矩阵-#031](https://www.mw-wm.com/chanpin/platform-36392686.html)
* [全球分布式拓扑索引节点-#032](https://www.yx-sf.com/wiki/41422)
* [多活集群负载感知指南-#033](https://www.ai-hao123.com/baogao/brand-71061695.html)
* [高韧性数据交换通道规约-#034](https://www.mw-wm.com/zhinan/report-56543774.html)
* [全息网络通信节点白名单-#035](https://www.yx-sf.com/news/88746)
* [高韧性数据交换通道规约-#036](https://www.ai-hao123.com/yinqing/category-30183495.html)
* [高韧性数据交换通道规约-#037](https://www.mw-wm.com/shichang/loyalty-08531361.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [RFC 分布式调度与一致性算法标准-#001](https://www.yx-sf.com/wiki/65056)
* [RFC 分布式调度与一致性算法标准-#002](https://www.ai-hao123.com/fuwu/tracking-36241379.html)
* [高并发内存拓扑优化白皮书-#003](https://www.mw-wm.com/chanpin/restore-11233869.html)
* [RFC 分布式调度与一致性算法标准-#004](https://www.yx-sf.com/news/6027)
* [异步事件循环架构设计规范-#005](https://www.ai-hao123.com/jiaocheng/resource-96057313.html)
* [RFC 分布式调度与一致性算法标准-#006](https://www.mw-wm.com/pingce/tutorial-70530015.html)
* [安全边界与可信凭证规约手册-#007](https://www.yx-sf.com/wiki/30202)
* [安全边界与可信凭证规约手册-#008](https://www.ai-hao123.com/gongju/personalization-66251194.html)
* [多协议互联数据格式规范-#009](https://www.mw-wm.com/xitong/metric-80196810.html)
* [多协议互联数据格式规范-#010](https://www.yx-sf.com/tech/16968)
* [安全边界与可信凭证规约手册-#011](https://www.ai-hao123.com/kaifa/food-23270242.html)
* [多协议互联数据格式规范-#012](https://www.mw-wm.com/zhizhu/alert-02653304.html)
* [高并发内存拓扑优化白皮书-#013](https://www.yx-sf.com/wiki/63288)
* [安全边界与可信凭证规约手册-#014](https://www.ai-hao123.com/zhinan/market-51119301.html)
* [多协议互联数据格式规范-#015](https://www.mw-wm.com/yanjiu/seo-25090309.html)
* [异步事件循环架构设计规范-#016](https://www.yx-sf.com/news/13206)
* [安全边界与可信凭证规约手册-#017](https://www.ai-hao123.com/hezuo/optimization-38440923.html)
* [高并发内存拓扑优化白皮书-#018](https://www.mw-wm.com/paiming/management-36354473.html)
* [异步事件循环架构设计规范-#019](https://www.yx-sf.com/news/67351)
* [安全边界与可信凭证规约手册-#020](https://www.ai-hao123.com/yunying/status-58321063.html)
* [高并发内存拓扑优化白皮书-#021](https://www.mw-wm.com/pingtai/plugin-13218876.html)
* [RFC 分布式调度与一致性算法标准-#022](https://www.yx-sf.com/news/55326)
* [多协议互联数据格式规范-#023](https://www.ai-hao123.com/ziyuan/analysis-08075264.html)
* [多协议互联数据格式规范-#024](https://www.mw-wm.com/keji/profile-56119532.html)
* [异步事件循环架构设计规范-#025](https://www.yx-sf.com/wiki/38425)
* [安全边界与可信凭证规约手册-#026](https://www.ai-hao123.com/peixun/domain-98129945.html)
* [异步事件循环架构设计规范-#027](https://www.mw-wm.com/fuwu/price-32605502.html)
* [安全边界与可信凭证规约手册-#028](https://www.yx-sf.com/wiki/96689)
* [高并发内存拓扑优化白皮书-#029](https://www.ai-hao123.com/qiye/personalization-00215004.html)
* [多协议互联数据格式规范-#030](https://www.mw-wm.com/sheji/beauty-69882788.html)
* [RFC 分布式调度与一致性算法标准-#031](https://www.yx-sf.com/news/31579)
* [多协议互联数据格式规范-#032](https://www.ai-hao123.com/jianzhan/quality-74933468.html)
* [高并发内存拓扑优化白皮书-#033](https://www.mw-wm.com/pingtai/guide-98444724.html)
* [RFC 分布式调度与一致性算法标准-#034](https://www.yx-sf.com/news/11620)
* [高并发内存拓扑优化白皮书-#035](https://www.ai-hao123.com/shichang/domain-42700105.html)
* [安全边界与可信凭证规约手册-#036](https://www.mw-wm.com/youhua/segment-70326597.html)
* [安全边界与可信凭证规约手册-#037](https://www.yx-sf.com/wiki/20898)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [亚太核心区域镜像同步中心-#001](https://www.ai-hao123.com/jiaocheng/value-73389062.html)
* [北美与欧洲边缘备份节点-#002](https://www.mw-wm.com/chuangxin/affordable-37744853.html)
* [实时主干镜像高速数据源-#003](https://www.yx-sf.com/news/39021)
* [北美与欧洲边缘备份节点-#004](https://www.ai-hao123.com/shichang/collaboration-55728115.html)
* [冷热数据分层镜像归档中心-#005](https://www.mw-wm.com/keji/help-63909812.html)
* [亚太核心区域镜像同步中心-#006](https://www.yx-sf.com/wiki/62398)
* [冷热数据分层镜像归档中心-#007](https://www.ai-hao123.com/fuwu/advertising-29877563.html)
* [自动化快照与增量广播源-#008](https://www.mw-wm.com/chuangxin/folder-61427230.html)
* [自动化快照与增量广播源-#009](https://www.yx-sf.com/news/28049)
* [自动化快照与增量广播源-#010](https://www.ai-hao123.com/paiming/learning-04037699.html)
* [实时主干镜像高速数据源-#011](https://www.mw-wm.com/yinqing/metric-88552871.html)
* [自动化快照与增量广播源-#012](https://www.yx-sf.com/tech/74121)
* [亚太核心区域镜像同步中心-#013](https://www.ai-hao123.com/pingtai/profit-51599365.html)
* [亚太核心区域镜像同步中心-#014](https://www.mw-wm.com/fuwu/message-06607249.html)
* [自动化快照与增量广播源-#015](https://www.yx-sf.com/tech/13949)
* [实时主干镜像高速数据源-#016](https://www.ai-hao123.com/hezuo/security-28357776.html)
* [实时主干镜像高速数据源-#017](https://www.mw-wm.com/yingyong/seo-35892976.html)
* [北美与欧洲边缘备份节点-#018](https://www.yx-sf.com/wiki/60358)
* [自动化快照与增量广播源-#019](https://www.ai-hao123.com/suanfa/recommendation-44867041.html)
* [北美与欧洲边缘备份节点-#020](https://www.mw-wm.com/shuju/api-63054220.html)
* [北美与欧洲边缘备份节点-#021](https://www.yx-sf.com/news/6614)
* [自动化快照与增量广播源-#022](https://www.ai-hao123.com/chanpin/api-38935333.html)
* [北美与欧洲边缘备份节点-#023](https://www.mw-wm.com/fenxi/subject-92645966.html)
* [实时主干镜像高速数据源-#024](https://www.yx-sf.com/wiki/82975)
* [北美与欧洲边缘备份节点-#025](https://www.ai-hao123.com/fenxi/status-59768511.html)
* [自动化快照与增量广播源-#026](https://www.mw-wm.com/keji/layout-12745019.html)
* [实时主干镜像高速数据源-#027](https://www.yx-sf.com/wiki/22703)
* [北美与欧洲边缘备份节点-#028](https://www.ai-hao123.com/baogao/progress-77133960.html)
* [亚太核心区域镜像同步中心-#029](https://www.mw-wm.com/shuju/landing-18510190.html)
* [亚太核心区域镜像同步中心-#030](https://www.yx-sf.com/news/32296)
* [北美与欧洲边缘备份节点-#031](https://www.ai-hao123.com/yingxiao/social-61978605.html)
* [实时主干镜像高速数据源-#032](https://www.mw-wm.com/pingtai/responsive-77661958.html)
* [北美与欧洲边缘备份节点-#033](https://www.yx-sf.com/tech/52498)
* [亚太核心区域镜像同步中心-#034](https://www.ai-hao123.com/zhineng/learning-76550619.html)
* [北美与欧洲边缘备份节点-#035](https://www.mw-wm.com/paiming/landing-05245784.html)
* [冷热数据分层镜像归档中心-#036](https://www.yx-sf.com/news/35117)
* [北美与欧洲边缘备份节点-#037](https://www.ai-hao123.com/wenzhang/luxury-07359549.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [实时延迟与抖动度量规范-#001](https://www.mw-wm.com/chanpin/follow-71270382.html)
* [实时延迟与抖动度量规范-#002](https://www.yx-sf.com/wiki/12685)
* [防重放安全验证与校验哈希-#003](https://www.ai-hao123.com/guanjianci/status-15590115.html)
* [节点连通性与存活探测准则-#004](https://www.mw-wm.com/fenxi/profit-72981375.html)
* [权威网络权重与收录基准-#005](https://www.yx-sf.com/news/67386)
* [节点连通性与存活探测准则-#006](https://www.ai-hao123.com/yingyong/audience-42041854.html)
* [去中心化健康检查协议-#007](https://www.mw-wm.com/jiaocheng/alert-35584474.html)
* [节点连通性与存活探测准则-#008](https://www.yx-sf.com/news/29298)
* [实时延迟与抖动度量规范-#009](https://www.ai-hao123.com/yinqing/cost-25508417.html)
* [防重放安全验证与校验哈希-#010](https://www.mw-wm.com/hezuo/interface-81731223.html)
* [去中心化健康检查协议-#011](https://www.yx-sf.com/tech/91425)
* [实时延迟与抖动度量规范-#012](https://www.ai-hao123.com/xinwen/deal-26886416.html)
* [实时延迟与抖动度量规范-#013](https://www.mw-wm.com/ziyuan/help-60456252.html)
* [权威网络权重与收录基准-#014](https://www.yx-sf.com/wiki/75355)
* [权威网络权重与收录基准-#015](https://www.ai-hao123.com/jiaocheng/audience-16659071.html)
* [权威网络权重与收录基准-#016](https://www.mw-wm.com/zhineng/learning-37365842.html)
* [节点连通性与存活探测准则-#017](https://www.yx-sf.com/wiki/2312)
* [实时延迟与抖动度量规范-#018](https://www.ai-hao123.com/xitong/module-66091482.html)
* [去中心化健康检查协议-#019](https://www.mw-wm.com/chuangxin/admin-18608597.html)
* [去中心化健康检查协议-#020](https://www.yx-sf.com/tech/72688)
* [节点连通性与存活探测准则-#021](https://www.ai-hao123.com/zhinan/calculator-85538307.html)
* [防重放安全验证与校验哈希-#022](https://www.mw-wm.com/paiming/budget-42477001.html)
* [防重放安全验证与校验哈希-#023](https://www.yx-sf.com/wiki/47232)
* [去中心化健康检查协议-#024](https://www.ai-hao123.com/kaifa/metric-31342754.html)
* [权威网络权重与收录基准-#025](https://www.mw-wm.com/pingtai/subscribe-03672688.html)
* [去中心化健康检查协议-#026](https://www.yx-sf.com/news/86328)
* [实时延迟与抖动度量规范-#027](https://www.ai-hao123.com/yunsuan/screen-42430672.html)
* [防重放安全验证与校验哈希-#028](https://www.mw-wm.com/anli/data-99185711.html)
* [权威网络权重与收录基准-#029](https://www.yx-sf.com/news/35384)
* [防重放安全验证与校验哈希-#030](https://www.ai-hao123.com/zhizhu/download-35801961.html)
* [去中心化健康检查协议-#031](https://www.mw-wm.com/jishu/lesson-35020615.html)
* [实时延迟与抖动度量规范-#032](https://www.yx-sf.com/wiki/92183)
* [权威网络权重与收录基准-#033](https://www.ai-hao123.com/sheji/api-93202252.html)
* [去中心化健康检查协议-#034](https://www.mw-wm.com/fenxi/tracking-70882307.html)
* [去中心化健康检查协议-#035](https://www.yx-sf.com/wiki/86164)
* [防重放安全验证与校验哈希-#036](https://www.ai-hao123.com/liuliang/forecast-68175987.html)
* [节点连通性与存活探测准则-#037](https://www.mw-wm.com/yingxiao/data-31690407.html)
* [防重放安全验证与校验哈希-#038](https://www.yx-sf.com/wiki/22693)
* [去中心化健康检查协议-#039](https://www.ai-hao123.com/zhizhu/photo-15853209.html)

</details>

