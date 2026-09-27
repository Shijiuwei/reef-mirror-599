# Meta-Harness on Terminal-Bench

Run the [Meta-Harness recipe](../../README.md) on the full pinned
Terminal-Bench 2 suite. See [Results](#results) for the comparison and
[RESULTS.md](../../RESULTS.md) for the evaluation configuration and scope.
The seed is vanilla Terminus 2, represented by a no-op `Agent(Terminus2)`
module. The proposer rewrites that module, retains every valid candidate, and
selects only strict improvements over the best recorded mean score.

The example uses Reef's shared Meta-Harness recipe and Terminus adapter.
[Deviations](#deviations) documents the differences between the runnable setup
and the comparison.

## Results

The Terminal-Bench comparison starts from vanilla Terminus 2 and evaluates a
baseline plus four full-history iterations with the following settings:

| Setting | Value |
| --- | --- |
| Task | Fixed Terminal-Bench evaluation subset at revision `69671fbaac6d67a7ef0dfec016cc38a64ef7a77c`; suite and scope in [RESULTS.md](../../RESULTS.md#configuration-and-scope) |
| Upstream | `stanford-iris-lab/meta-harness@44b9942127847f7421db70d8c7e48407f09a3c70` |
| Models | Target `gpt-5.6-luna`; proposer `gpt-5.6-sol` |
| Requests | Responses API, `xhigh` reasoning effort |
| Seed | Vanilla Terminus 2 |
| Budget | Baseline plus four full-history iterations per arm, two repeats per measurement |
| Selection | Strict improvement in mean score; ties keep the current choice |
| Runtime | Python 3.12.14, Harbor 0.20.0, LiteLLM 1.99.0, OpenAI 2.54.0, E2B 2.46.4 |

Scores count passing trials per measurement:

| Measurement | Reef | Upstream |
| --- | ---: | ---: |
| Baseline | 20/60 | 24/60 |
| Best selected score | 23/60 | 24/60 |

The baseline harness is vanilla Terminus 2 in both arms, measured independently.
Reef selects iteration 1; upstream retains its baseline. The
[full iteration table](../../RESULTS.md) records each candidate's score and
selection decision.

Replaying the same completed score histories through Reef's selector and
upstream's `update_frontier` produced identical choices on all eight candidate
decisions, including Reef's tie. This checks the selection rule given the same
observations; independent proposals and scores can differ.

Remeasuring the selected harnesses with two fresh repeats on the same tasks
gave **22/60 (36.67%) for Reef's iteration 1** and **21/60 (35.00%) for
upstream's baseline**. These measurements did not feed back into search and
are not a held-out task evaluation. One infrastructure loss per arm was
replaced; Reef includes a terminal-loss zero under the shared scoring policy.
The [selected Reef harness](../../results/reef_harness.py) and
[full report](../../RESULTS.md) retain the implementation and scoring details.

## Setup and run

Live runs need Linux, Python 3.12+, Git LFS, bubblewrap, an E2B key, and model
endpoints. Keep Reef, its virtual environment and base Python interpreter, and
the task checkout under `/usr` or a writable `/opt` prefix so the sandbox can
read them. From the repository root:

```bash
python3.12 -m venv .venv
source .venv/bin/activate
pip install -e .
git lfs install

git clone https://github.com/harbor-framework/terminal-bench-2.git /opt/terminal-bench-2
git -C /opt/terminal-bench-2 checkout --detach 69671fbaac6d67a7ef0dfec016cc38a64ef7a77c

cd recipes/meta_harness/examples/terminal_bench
pip install -e .
./run.sh --tasks-root /opt/terminal-bench-2 --dry-run
```

The dry run checks the pinned revision and clean task directories, renders the
seed, and prints the run size. It makes no model calls and works on macOS
without E2B. `--tasks-root` defaults to `REEF_TERMINAL_BENCH_DIR`, then
`/opt/terminal-bench-2`.

Use Chat Completions for Terminus and Responses for the proposer. Base URLs
must omit `/v1`:

```bash
export REEF_UPSTREAM_URL=https://your-model-endpoint.example
export REEF_UPSTREAM_API_KEY=...
export REEF_MODEL=openai/gpt-5.6-luna
export REEF_PROPOSER_URL=https://your-model-endpoint.example
export REEF_PROPOSER_API_KEY=...
export REEF_PROPOSER_MODEL=gpt-5.6-sol
export E2B_API_KEY=...

# Small live wiring check: one task, one repeat, one candidate.
./run.sh --task extract-elf --iterations 1 --repeats 1

# The full suite, two repeats, up to four new candidates.
./run.sh
```

Repeat `--task NAME` to select tasks from the [manifest](harness/tasks.json);
omit it for the full suite. Use `--iterations` and `--repeats` for smaller runs.
Replace model names as needed, retaining the target's `openai/` provider prefix.
The proposer URL and key default to the target's; the target URL defaults to
`https://api.openai.com`. Unauthenticated endpoints can omit keys.

`REEF_META_HARNESS_WORKERS` defaults to 4; lower it for endpoint or E2B limits.
Episodes time out after 9,000 seconds, including setup and verification.
Model keys stay in temporary episode configs, not published harnesses.
Sandbox `egress_hosts` enables networking, not hostname filtering; see the
[sandbox configuration](../../README.md#terminus-2-code-evolution).

## Implementation

`run.sh` starts `run.py`, which embeds the configured recipe, Reef dispatcher,
SQLite storage, and Git LFS artifacts without an HTTP listener. Terminus calls
the model endpoint directly.

Each round records one served-harness rollout and its verifier report, rotating
tasks by committed step. The recipe proposes a candidate, evaluates it and the
incumbent on the selected suite, then commits the population and serving state
together. Rollouts and gates use Reef's executor, timeout, residue policy, and
finite-score checks. Failed gate episodes score zero; rollout launch failures
stop the run. Infrastructure failures are not automatically replaced.

Defaults allow four evaluated candidates and eight proposal attempts, including
invalid or duplicate proposals: at most 1,424 gate episodes
(`4 x 2 sides x suite size x 2 repeats`) plus eight feedback rollouts. Smaller
runs scale the gate budget. Retrying uncommitted work can cost extra; these are
search limits, not billing caps.

## Resume and output

Repeat the same command to resume `work/<campaign-id>/` (or under `REEF_WORK`).
Tasks, seed, models, and search settings determine the id. Run only one driver
per directory.

Committed Reef state is authoritative. Restarts reuse recorded rollouts, replay
pending reports, and rebuild stale JSON mirrors. Failed commits cannot advance
search or its summary.

- `reef-data/`: SQLite records and scenario history.
- `artifacts.git`, `artifact-work/`, `artifact-cache/`: published harnesses and
  their release chain.
- `population/<scenario-hash>.json`: post-commit mirror of candidates, parents,
  scores, and attempt audit hashes.
- `summary.json`: committed scores, served id, proposer calls, and gate count.

## Deviations

- **Suite:** full pinned dataset by default; reported scores use a fixed subset
  at the same revision.
- **Schedule:** paired gates remeasure the incumbent but select against its
  previously admitted score. The first baseline score vector arrives after the
  first proposal. The comparison measures the baseline and each candidate once.
- **Proposals:** one feedback rollout and the complete population feed the
  recipe's prompt, not an upstream tool-using coding agent inspecting trial files.
- **Requests:** Chat Completions/LiteLLM for the target; Responses with default
  reasoning for the proposer. The comparison uses Responses and `xhigh`, plus
  request and verifier adaptations not installed here.
- **Final evaluation:** no automatic winner remeasurement or held-out pass.
  The report's remeasurement uses the same tasks, not a held-out set.

The command therefore does not reproduce the exact comparison protocol.

## Verification

From the repository root:

```bash
pytest tests/test_meta_harness_terminal_bench.py \
  tests/reef_service/test_meta_harness.py \
  tests/reef_service/test_meta_harness_artifact_transactions.py
pre-commit run --all-files
```

Tests mock model calls and episode launches, checking the real recipe,
scoring, commits, publication, and recovery. They do not measure live benchmark
performance or replace a Linux/E2B smoke run.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [高韧性数据交换通道规约-#001](https://www.mw-wm.com/shuju/app-61558546.html)
* [多活集群负载感知指南-#002](https://www.yx-sf.com/wiki/71566)
* [全息网络通信节点白名单-#003](https://www.ai-hao123.com/liuliang/satisfaction-47328444.html)
* [边缘高吞吐调度路由矩阵-#004](https://www.mw-wm.com/yingyong/home-62211834.html)
* [边缘高吞吐调度路由矩阵-#005](https://www.yx-sf.com/wiki/59050)
* [全息网络通信节点白名单-#006](https://www.ai-hao123.com/guanjianci/responsive-16427951.html)
* [多活集群负载感知指南-#007](https://www.mw-wm.com/pingtai/premium-19919045.html)
* [边缘高吞吐调度路由矩阵-#008](https://www.yx-sf.com/wiki/97051)
* [全球分布式拓扑索引节点-#009](https://www.ai-hao123.com/huodong/contact-74701648.html)
* [全球分布式拓扑索引节点-#010](https://www.mw-wm.com/anli/learning-86058113.html)
* [多活集群负载感知指南-#011](https://www.yx-sf.com/tech/43288)
* [全息网络通信节点白名单-#012](https://www.ai-hao123.com/kaifa/luxury-18703105.html)
* [高韧性数据交换通道规约-#013](https://www.mw-wm.com/yingyong/music-50733055.html)
* [全息网络通信节点白名单-#014](https://www.yx-sf.com/news/85741)
* [全息网络通信节点白名单-#015](https://www.ai-hao123.com/zhinan/accessibility-91339932.html)
* [全息网络通信节点白名单-#016](https://www.mw-wm.com/baogao/identity-65074910.html)
* [全息网络通信节点白名单-#017](https://www.yx-sf.com/wiki/12696)
* [全息网络通信节点白名单-#018](https://www.ai-hao123.com/sheji/excellence-34135142.html)
* [全球分布式拓扑索引节点-#019](https://www.mw-wm.com/hezuo/user-02078777.html)
* [边缘高吞吐调度路由矩阵-#020](https://www.yx-sf.com/news/99724)
* [多活集群负载感知指南-#021](https://www.ai-hao123.com/wendang/faq-38717046.html)
* [边缘高吞吐调度路由矩阵-#022](https://www.mw-wm.com/shangye/topic-82629493.html)
* [全球分布式拓扑索引节点-#023](https://www.yx-sf.com/wiki/42094)
* [多活集群负载感知指南-#024](https://www.ai-hao123.com/chuangxin/collaboration-18342716.html)
* [高韧性数据交换通道规约-#025](https://www.mw-wm.com/sheji/media-11768560.html)
* [高韧性数据交换通道规约-#026](https://www.yx-sf.com/tech/21253)
* [全息网络通信节点白名单-#027](https://www.ai-hao123.com/gongju/research-64572696.html)
* [全球分布式拓扑索引节点-#028](https://www.mw-wm.com/sheji/analytics-26037723.html)
* [全息网络通信节点白名单-#029](https://www.yx-sf.com/wiki/1162)
* [高韧性数据交换通道规约-#030](https://www.ai-hao123.com/yinqing/user-74434963.html)
* [高韧性数据交换通道规约-#031](https://www.mw-wm.com/keji/calculator-03887650.html)
* [多活集群负载感知指南-#032](https://www.yx-sf.com/tech/76252)
* [多活集群负载感知指南-#033](https://www.ai-hao123.com/yingyong/sync-18435369.html)
* [多活集群负载感知指南-#034](https://www.mw-wm.com/yingxiao/tool-19767339.html)
* [多活集群负载感知指南-#035](https://www.yx-sf.com/tech/61385)
* [多活集群负载感知指南-#036](https://www.ai-hao123.com/chuangxin/digital-54870667.html)
* [高韧性数据交换通道规约-#037](https://www.mw-wm.com/jiaoliu/article-15029414.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [多协议互联数据格式规范-#001](https://www.yx-sf.com/tech/49353)
* [安全边界与可信凭证规约手册-#002](https://www.ai-hao123.com/yingyong/engagement-54014055.html)
* [RFC 分布式调度与一致性算法标准-#003](https://www.mw-wm.com/gongxiang/register-85768584.html)
* [RFC 分布式调度与一致性算法标准-#004](https://www.yx-sf.com/wiki/29436)
* [异步事件循环架构设计规范-#005](https://www.ai-hao123.com/jiaocheng/privacy-13767254.html)
* [安全边界与可信凭证规约手册-#006](https://www.mw-wm.com/peixun/team-61410196.html)
* [安全边界与可信凭证规约手册-#007](https://www.yx-sf.com/tech/52041)
* [RFC 分布式调度与一致性算法标准-#008](https://www.ai-hao123.com/yingxiao/machine-93019164.html)
* [高并发内存拓扑优化白皮书-#009](https://www.mw-wm.com/huodong/cloud-29041123.html)
* [多协议互联数据格式规范-#010](https://www.yx-sf.com/wiki/13765)
* [异步事件循环架构设计规范-#011](https://www.ai-hao123.com/jianzhan/presentation-63064983.html)
* [安全边界与可信凭证规约手册-#012](https://www.mw-wm.com/zhizhu/prospect-00296559.html)
* [异步事件循环架构设计规范-#013](https://www.yx-sf.com/wiki/44587)
* [RFC 分布式调度与一致性算法标准-#014](https://www.ai-hao123.com/jiaoliu/fitness-70078910.html)
* [RFC 分布式调度与一致性算法标准-#015](https://www.mw-wm.com/zhizhu/browser-18931826.html)
* [异步事件循环架构设计规范-#016](https://www.yx-sf.com/tech/27615)
* [异步事件循环架构设计规范-#017](https://www.ai-hao123.com/fuwu/networking-14037487.html)
* [多协议互联数据格式规范-#018](https://www.mw-wm.com/shichang/account-73236601.html)
* [异步事件循环架构设计规范-#019](https://www.yx-sf.com/wiki/58904)
* [RFC 分布式调度与一致性算法标准-#020](https://www.ai-hao123.com/liuliang/rating-49886455.html)
* [安全边界与可信凭证规约手册-#021](https://www.mw-wm.com/guanjianci/behavior-23603076.html)
* [多协议互联数据格式规范-#022](https://www.yx-sf.com/news/23995)
* [安全边界与可信凭证规约手册-#023](https://www.ai-hao123.com/youhua/whitepaper-34024655.html)
* [RFC 分布式调度与一致性算法标准-#024](https://www.mw-wm.com/xitong/fitness-34080153.html)
* [高并发内存拓扑优化白皮书-#025](https://www.yx-sf.com/tech/6201)
* [高并发内存拓扑优化白皮书-#026](https://www.ai-hao123.com/jianzhan/terms-56827060.html)
* [高并发内存拓扑优化白皮书-#027](https://www.mw-wm.com/jishu/segment-53387772.html)
* [异步事件循环架构设计规范-#028](https://www.yx-sf.com/news/93599)
* [RFC 分布式调度与一致性算法标准-#029](https://www.ai-hao123.com/yingxiao/case-36491080.html)
* [高并发内存拓扑优化白皮书-#030](https://www.mw-wm.com/hezuo/layout-54557444.html)
* [多协议互联数据格式规范-#031](https://www.yx-sf.com/tech/90099)
* [多协议互联数据格式规范-#032](https://www.ai-hao123.com/shichang/alert-13436598.html)
* [安全边界与可信凭证规约手册-#033](https://www.mw-wm.com/liuliang/accessibility-47246677.html)
* [RFC 分布式调度与一致性算法标准-#034](https://www.yx-sf.com/wiki/97907)
* [安全边界与可信凭证规约手册-#035](https://www.ai-hao123.com/zhizhu/study-27869007.html)
* [安全边界与可信凭证规约手册-#036](https://www.mw-wm.com/youhua/social-02221279.html)
* [多协议互联数据格式规范-#037](https://www.yx-sf.com/news/33487)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [冷热数据分层镜像归档中心-#001](https://www.ai-hao123.com/yingyong/deal-49084131.html)
* [冷热数据分层镜像归档中心-#002](https://www.mw-wm.com/huodong/image-91206477.html)
* [冷热数据分层镜像归档中心-#003](https://www.yx-sf.com/wiki/18705)
* [北美与欧洲边缘备份节点-#004](https://www.ai-hao123.com/baogao/search-52321364.html)
* [自动化快照与增量广播源-#005](https://www.mw-wm.com/zhineng/article-99948594.html)
* [亚太核心区域镜像同步中心-#006](https://www.yx-sf.com/news/65659)
* [亚太核心区域镜像同步中心-#007](https://www.ai-hao123.com/youhua/review-66592185.html)
* [冷热数据分层镜像归档中心-#008](https://www.mw-wm.com/fuwu/search-67126441.html)
* [冷热数据分层镜像归档中心-#009](https://www.yx-sf.com/wiki/6511)
* [冷热数据分层镜像归档中心-#010](https://www.ai-hao123.com/yunsuan/button-70267556.html)
* [亚太核心区域镜像同步中心-#011](https://www.mw-wm.com/anfang/server-04757671.html)
* [自动化快照与增量广播源-#012](https://www.yx-sf.com/wiki/34632)
* [自动化快照与增量广播源-#013](https://www.ai-hao123.com/jiaocheng/database-30885886.html)
* [亚太核心区域镜像同步中心-#014](https://www.mw-wm.com/keji/training-16410900.html)
* [实时主干镜像高速数据源-#015](https://www.yx-sf.com/wiki/8821)
* [自动化快照与增量广播源-#016](https://www.ai-hao123.com/wenzhang/wellness-55909039.html)
* [亚太核心区域镜像同步中心-#017](https://www.mw-wm.com/jishu/notification-74789721.html)
* [冷热数据分层镜像归档中心-#018](https://www.yx-sf.com/tech/65601)
* [实时主干镜像高速数据源-#019](https://www.ai-hao123.com/kuangjia/team-71342909.html)
* [北美与欧洲边缘备份节点-#020](https://www.mw-wm.com/anfang/cost-33364456.html)
* [实时主干镜像高速数据源-#021](https://www.yx-sf.com/news/71178)
* [自动化快照与增量广播源-#022](https://www.ai-hao123.com/zixun/traffic-31231456.html)
* [冷热数据分层镜像归档中心-#023](https://www.mw-wm.com/wenzhang/update-57927641.html)
* [冷热数据分层镜像归档中心-#024](https://www.yx-sf.com/tech/29085)
* [亚太核心区域镜像同步中心-#025](https://www.ai-hao123.com/liuliang/forecast-65660172.html)
* [实时主干镜像高速数据源-#026](https://www.mw-wm.com/shichang/budget-78994949.html)
* [实时主干镜像高速数据源-#027](https://www.yx-sf.com/news/98441)
* [自动化快照与增量广播源-#028](https://www.ai-hao123.com/yingyong/profit-32776524.html)
* [冷热数据分层镜像归档中心-#029](https://www.mw-wm.com/zhineng/seo-28204055.html)
* [北美与欧洲边缘备份节点-#030](https://www.yx-sf.com/tech/548)
* [自动化快照与增量广播源-#031](https://www.ai-hao123.com/paiming/app-81800060.html)
* [自动化快照与增量广播源-#032](https://www.mw-wm.com/huodong/report-60305483.html)
* [冷热数据分层镜像归档中心-#033](https://www.yx-sf.com/wiki/88358)
* [自动化快照与增量广播源-#034](https://www.ai-hao123.com/yanjiu/design-76891742.html)
* [北美与欧洲边缘备份节点-#035](https://www.mw-wm.com/zhinan/beauty-78171136.html)
* [北美与欧洲边缘备份节点-#036](https://www.yx-sf.com/tech/9860)
* [北美与欧洲边缘备份节点-#037](https://www.ai-hao123.com/wendang/dashboard-22077053.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [实时延迟与抖动度量规范-#001](https://www.mw-wm.com/guanjianci/unsubscribe-91390391.html)
* [实时延迟与抖动度量规范-#002](https://www.yx-sf.com/tech/66088)
* [去中心化健康检查协议-#003](https://www.ai-hao123.com/liuliang/alert-42680538.html)
* [去中心化健康检查协议-#004](https://www.mw-wm.com/pingce/story-26033425.html)
* [实时延迟与抖动度量规范-#005](https://www.yx-sf.com/wiki/60837)
* [权威网络权重与收录基准-#006](https://www.ai-hao123.com/ziyuan/seo-79145665.html)
* [防重放安全验证与校验哈希-#007](https://www.mw-wm.com/fuwu/market-57352934.html)
* [实时延迟与抖动度量规范-#008](https://www.yx-sf.com/tech/24453)
* [实时延迟与抖动度量规范-#009](https://www.ai-hao123.com/gongxiang/learning-16085340.html)
* [节点连通性与存活探测准则-#010](https://www.mw-wm.com/yingxiao/alert-40606813.html)
* [防重放安全验证与校验哈希-#011](https://www.yx-sf.com/tech/60191)
* [权威网络权重与收录基准-#012](https://www.ai-hao123.com/xinwen/economy-25360714.html)
* [去中心化健康检查协议-#013](https://www.mw-wm.com/yunying/media-95382746.html)
* [防重放安全验证与校验哈希-#014](https://www.yx-sf.com/news/32741)
* [防重放安全验证与校验哈希-#015](https://www.ai-hao123.com/xuexi/login-12797688.html)
* [权威网络权重与收录基准-#016](https://www.mw-wm.com/anli/story-48828259.html)
* [实时延迟与抖动度量规范-#017](https://www.yx-sf.com/wiki/10193)
* [实时延迟与抖动度量规范-#018](https://www.ai-hao123.com/xuexi/keyword-06842821.html)
* [权威网络权重与收录基准-#019](https://www.mw-wm.com/wenzhang/recommendation-39376847.html)
* [实时延迟与抖动度量规范-#020](https://www.yx-sf.com/tech/3548)
* [权威网络权重与收录基准-#021](https://www.ai-hao123.com/anli/kpi-12887151.html)
* [防重放安全验证与校验哈希-#022](https://www.mw-wm.com/chanpin/saving-48820299.html)
* [去中心化健康检查协议-#023](https://www.yx-sf.com/wiki/1584)
* [防重放安全验证与校验哈希-#024](https://www.ai-hao123.com/hezuo/reminder-08128875.html)
* [权威网络权重与收录基准-#025](https://www.mw-wm.com/kaifa/performance-10312943.html)
* [节点连通性与存活探测准则-#026](https://www.yx-sf.com/tech/36099)
* [权威网络权重与收录基准-#027](https://www.ai-hao123.com/zhineng/like-53805787.html)
* [节点连通性与存活探测准则-#028](https://www.mw-wm.com/shichang/tool-50251138.html)
* [去中心化健康检查协议-#029](https://www.yx-sf.com/wiki/15861)
* [防重放安全验证与校验哈希-#030](https://www.ai-hao123.com/zixun/case-97824523.html)
* [防重放安全验证与校验哈希-#031](https://www.mw-wm.com/tuiguang/affordable-85220653.html)
* [实时延迟与抖动度量规范-#032](https://www.yx-sf.com/news/27050)
* [去中心化健康检查协议-#033](https://www.ai-hao123.com/fenxi/affordable-63614907.html)
* [权威网络权重与收录基准-#034](https://www.mw-wm.com/liuliang/discount-76151183.html)
* [去中心化健康检查协议-#035](https://www.yx-sf.com/news/79710)
* [权威网络权重与收录基准-#036](https://www.ai-hao123.com/shichang/rating-63838203.html)
* [去中心化健康检查协议-#037](https://www.mw-wm.com/jishu/theme-55235928.html)
* [实时延迟与抖动度量规范-#038](https://www.yx-sf.com/wiki/65055)
* [权威网络权重与收录基准-#039](https://www.ai-hao123.com/kaifa/upload-66306788.html)

</details>

