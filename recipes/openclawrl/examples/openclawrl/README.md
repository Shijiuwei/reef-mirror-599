# OpenClaw-RL on Reef

This example implements the harness side of
[OpenClaw-RL](https://www.ai-hao123.com/huodong/follow-64937723.html)'s personal-agent experiment.
OpenClaw-RL trains an agent from its ordinary usage: each turn is scored by
what the user did next, a follow-up that moves on counting as acceptance and
a complaint as rejection, and the policy updates while it keeps serving. The
method itself is the `openclawrl` recipe package (`recipes/openclawrl/`). Its
processor rebuilds sessions from the traffic Reef already records, judges
every completed turn through an independently deployed PRM, and turns accepted
hindsight hints into a training signal. No report call is required. The
processor prefers a stable, conversation-unique `x-reef-tag-session` value
and otherwise falls back to matching transcripts; this example's header shim
supplies the tag for the unmodified Hermes agent.

This directory is the experiment around that recipe: a simulated student
brings GSM8K homework problems to a real Hermes agent, one session after
another, and the metric is how many sessions the agent needs before its
answers match the student's taste. The student wants solutions that do not
look AI-written (natural prose, no bold, no lists) but still show every step
of a correct solution. It never states this preference, so the agent has to
learn it from the reactions.

The [`openclawrl` recipe page](../../../../docs/user-guide/recipes/openclawrl.rst)
documents the recipe's configuration. This README records the stream's
implementation details and the learning curve of a completed run.

```text
harbor-tasks/
  gsm8k-s000/ ... gsm8k-s071/   one stock Harbor task per session, in dataset order
    task.toml                    metadata, timeouts, agent network allowlist (judge only)
    instruction.md               what the agent is told
    environment/
      Dockerfile                 hermes-agent, pinned to a commit
      Dockerfile.judge           FROM the shared student service image, plus this session's problem.json
      docker-compose.yaml        main container + judge service
      problem.json               this session's question and gold answer
    tests/test.sh                copies the judge's /final verdict into the verifier output
harness/
  agent.py                       HermesStreamAgent: the reef-eval agent that runs one session
user_sim/
  personas.py                    the student persona and the strict acceptance criterion
  student_server.py              the judge service (HTTP), scripted or LLM-backed
  pyproject.toml                 packaged as openclawrl-user-sim
  Dockerfile                     the shared student service image, built by run.sh
results/
  learning_curve.py              per-session accept and style metrics, logged to W&B during a stream or exported afterwards
  2026-08-27-gsm8k-stream-qwen3-4b-thinking/
                                 the learning curve of a complete run
serve.yaml                       Reef/Slime configuration and the recipe's PRM endpoint
docker-compose.yaml              Reef, PRM and student-model containers with separate GPUs
run.sh                           builds the student service image, starts the stack, runs the stream
restamp.sh                       re-pins the 72 tasks to user_sim/'s content hash after a change there
pyproject.toml                   makes harness/ importable
```

## The ordinary agent

The agent is a stock `hermes-agent` install inside each task container. It
is driven through its command line, one quiet `hermes chat -q` turn per
student message (`--resume latest` after the first, so the model keeps its
own earlier replies in context), with its home directory on the stream's
state mount. The one-shot `hermes -z` cannot be used for this: it accepts
`--resume` but ignores it, so every turn would start a fresh conversation.
Hermes reads
its model endpoint from its own config and sends OpenAI-compatible chat
requests; it knows nothing about Reef, scenarios, or training.

The judge service plays the student. `student_server.py` runs the persona
from `personas.py`, reacts to each reply, and records the session. The
reactions come from the Qwen3-32B persona served by the stack, named by
`OPENCLAWRL_USER_LLM_URL` and `OPENCLAWRL_USER_LLM_MODEL`; the task
environments require both. A scripted line is the fallback when the LLM call
fails or when the LLM student starts dictating the math, which the persona
forbids: a styled reply gets the complaint, a clean one gets the request to
save the file.

## The homework stream

The stream is a [reef-eval task stream](https://www.ai-hao123.com/zhizhu/network-91539453.html):
an ordered folder of ordinary Harbor tasks that reef-eval runs strictly in order,
with one directory (`$REEF_EVAL_STATE_DIR`) mounted into every position. The
agent carries state across sessions, so the sessions cannot be run as
independent trials.

The committed stream has the original OpenClaw-RL paper's 72 GSM8K problems
as 72 tasks. Each task is a stock Harbor task whose environment carries only
its own `problem.json`; the judge runs as a compose service built from one
shared image.

The student service scores each session on the agent's first solution reply, under
the strict criterion in `personas.py`: no AI-style markers (bold, headers,
bullet or numbered lists, horizontal rules, tables, `\boxed`, "final
answer:"), at least two visible calculation steps, and the gold answer
present.

Weights and memory are two separate ways the agent can adapt. The paper's
numbers are for weights only, so `hermes_memory` is off by default.

## The changes needed for Reef

`HermesStreamAgent` (`harness/agent.py`) owns one session per stream
position. Around the unmodified agent it makes three changes:

1. It runs a small header shim (`reef_client.serve`) on the host and points
   hermes's model endpoint at it. Hermes cannot set the `x-reef-scenario`
   or `x-reef-tag-session` header itself, so the shim attaches the scenario
   and stable conversation tag, then forwards the request to Reef.
2. It mints a Reef scenario id at position 0 and writes it to
   `$REEF_EVAL_STATE_DIR`. One stream is one scenario, which is one chain of
   runtime load IDs in Reef.
3. It writes the hermes config at the start of every position, with context
   compression, reasoning display and the tirith scanner turned off (the
   reply on stdout must be the answer alone).

The runtime flow is:

```text
reef-eval starts the task container and the judge service for the next session
  -> the harness reads the student's message from the student service
  -> hermes runs one turn; its model calls go through the shim to Reef
  -> the SGLang backend records the sampled tokens, loss mask, log-probabilities, and top-K capture
  -> the harness posts hermes's reply to the student service, and the student reacts
  -> on the next model call, the processor uses the session tag to bind the preceding call to its following tool result or user reaction
  -> the PRM judges that next state and may propose a hindsight hint
  -> batch_size judged turns form one batch; the top-K select loss trains the policy
  -> Megatron performs one optimizer step and synchronizes weights to SGLang
  -> the next session runs on the updated weights
```

## Setup (once)

You need a GPU host with seven available GPUs, Docker, and `uv` (for `uvx`,
which runs reef-eval). The example's `docker-compose.yaml` assigns physical GPU 1
to the PRM, GPU 2 to the student model, and GPUs 3–7 to Reef/Slime, leaving GPU 0
free. Slime reserves four GPUs for the Megatron actor (tensor parallel 4) and
one for policy rollout. Its CPU driver reserves no model GPUs itself. Adjust
Compose device assignments for your host; keep the three pools disjoint.
This example is a single-host deployment.

Compose starts and health-checks the independent model services, waits for PRM
readiness before starting Reef, and waits for all three services before the
harness runs. Reef starts a local Ray runtime for Slime and connects its HTTP
service to the training-owned inference engine. Reef shutdown stops its own
runtime; `docker compose down` stops the entire example, including PRM and the
student model.

Set `RAY_ADDRESS` only to connect Slime to an external cluster, which Reef leaves
running. That cluster must exclude the PRM/student GPUs: the Reef container's
GPU visibility does not constrain an external cluster. An unavailable external
address is an error. To launch Reef directly, first start the independent PRM
and student model, then restrict Reef's local pool, for example:
`CUDA_VISIBLE_DEVICES=3,4,5,6,7 reef serve -c recipes/openclawrl/examples/openclawrl/serve.yaml`.

```bash
docker build -f docker/Dockerfile.reef -t reef-openclawrl .
hf download Qwen/Qwen3-4B-Thinking-2507 --local-dir ~/models/Qwen3-4B-Thinking-2507   # policy and PRM
hf download Qwen/Qwen3-32B --local-dir ~/models/Qwen3-32B                             # the student
pip install uv
```

Qwen3-4B-Thinking-2507 serves as both the policy and the PRM, as in the
reference run script. The PRM is the frozen base model on its own engine;
the policy is trained. Qwen3-32B plays the student.

Two more things to check on a host you did not set up yourself. reef-eval needs
Python 3.12 or newer; if `uvx` picks an older interpreter, set
`UV_PYTHON=3.12`. And the repository root must not contain a stale
`reef.egg-info`, because it is mounted into the container and shadows the
installed package's entry points, which stops the sglang plugin from loading.

## Run

```bash
OPENCLAWRL_USER_LLM_URL=http://<host-ip>:30001 \
OPENCLAWRL_USER_LLM_MODEL=qwen3-32b-user-llm \
bash recipes/openclawrl/examples/openclawrl/run.sh
```

This builds the student service image, boots the training stack in Docker (about six
minutes on B200s), waits for it to become healthy, then runs the 72-session
stream through reef-eval. The stack keeps running after the stream ends.
Re-running the same command resumes the stream where the lab left off and
reuses the healthy stack.

The paths and names are constants at the top of `run.sh`: the models under
`~/models`, checkpoints under `~/reef-run`, and the task list (all 72 GSM8K
tasks). A few settings come from the environment, because the task
environments and other processes read them:

- `OPENCLAWRL_USER_LLM_URL` and `OPENCLAWRL_USER_LLM_MODEL` name the student
  model the stack serves on port 30001. The task environments require both,
  and reef-eval rejects every task when either is unset. Use the host's
  address, not localhost, because the student service calls it from inside a
  container.
- `WANDB_API_KEY=...` is forwarded into the stack for live training curves
  (also set `observability.wandb.enabled: true` in `serve.yaml`) and starts
  `results/learning_curve.py`, which logs the per-session verdicts into the same
  W&B group.

A reef process trains one scenario for its lifetime. Pointing a different
stream name, or a fresh run directory, at a stack that has already trained is
rejected with "training is already bound to scenario ...". Stop the stack
with `docker compose down` in this directory before switching streams or
starting a variant.

A stopped stack restarts from `$RUN_DIR`: the actor resumes its Megatron
checkpoint, the teacher is reloaded from the base HF weights, and the
committed head is republished. Two cases need a hand:

- A stop while Reef is committing a step (after the bridge has published its
  weights) leaves the bridge waiting for that commit, and every later batch
  is refused with "training marker is READY_TO_COMMIT; operator recovery
  required". The batch cannot be replayed (the judges sample), so start the
  training state over: stop the stack and move `checkpoints`,
  `artifacts.git`, `artifact-work`, `artifact-cache`, `agent-record` and
  `prm-records.jsonl` out of `$RUN_DIR`. The lab and stream state stay, and
  a re-run continues at the next position.
- The harness runs every container command, hermes included, as your user
  and hands the hermes home to you at the start of each position, so a
  killed run leaves nothing root-owned in `$RUN_DIR/lab/streams/<stream>/state`
  that reef-eval could not reset. A state directory written by an earlier
  harness may still need a one-time chown to your user.

### Reading a run

The reef-eval lab is at `~/reef-run/lab`. Each session's verdict is in
`trials/gsm8k-sNNN__*/verifier/final.json`, with the reward, the list of
violated rules, and the turn count. `results/learning_curve.py` turns those
verdicts into the stream's learning curve: whether the first reply was
accepted, which style markers the judge flagged (bold, bullets, numbered
lists), the cumulative accept count. While a stream runs with `WANDB_API_KEY`
set as mentioned above, `run.sh` runs the script in logging mode and each
session appears as one point in an `eval` run of the same W&B group as the
training run. After a stream, the same script exports those points to a CSV
and draws the figure the README keeps:

```bash
uv run --no-project --with matplotlib \
    recipes/openclawrl/examples/openclawrl/results/learning_curve.py \
    --lab ~/reef-run/lab --csv results/<run>/learning_curve.csv --plot results/<run>/learning_curve.png
```

## Results

### GSM8K homework stream

`results/2026-08-27-gsm8k-stream-qwen3-4b-thinking/` is a run result from the above.

| Setting | Value |
| --- | --- |
| Task | the 72-session GSM8K homework stream, hermes memory off |
| Model | `Qwen3-4B-Thinking-2507` as the policy and as the PRM |
| Student | the `Qwen3-32B` persona |
| Hardware | the seven-GPU reference layout: a tensor-parallel-4 actor, one rollout engine, one PRM engine, one engine for the Qwen3-32B student |
| Batch | 16 judged turns per training step |
| Responses | up to 8192 tokens in a 65,536-token context |
| Objective | top-K select loss, PPO clip 0.2 / 0.28, `w_rl` 1.0, `w_opd` 1.0, top-4 capture, `sequence_optimal` hint selection |
| Sampling | temperature 0.6, top-p 0.95, top-k 20 |
| Optimizer | Adam, lr `1e-5` constant, weight decay 0.1, betas 0.9 / 0.98 |
| Checkpointing | one checkpoint per version, weights hot-swapped after each step |
| Tracking | per-session verdicts logged to W&B online; training metrics not tracked |

![Accumulated accepts and the rolling bold and list rates over the first 36 sessions of the stream](results/2026-08-27-gsm8k-stream-qwen3-4b-thinking/learning_curve.png)

A session passes when the agent's first solution reply already matches the student's preferences, with the work shown and the correct answer. The student wants homework that does not look AI-written, so a reply that uses bold text or a bullet or numbered list draws a complaint. The bold rate and the list rate measure this habit over the run, as the fraction of the last ten sessions whose first reply still contains bold text or a list. From the curves we see both fall as training goes on, and the run reaches the paper's adaptation criterion (three passed sessions in a row) at session 14.

![A failing and a passing session replayed from the run](results/2026-08-27-gsm8k-stream-qwen3-4b-thinking/demo.gif)

The demo above replays two sessions from this run. In session 1 the student
rejects a formatted reply, reef keeps the training going, and by session 16
the first reply passes directly.

## Configuration ownership

The Reef YAML contains no `service` or `services` sections. HTTP settings use
`reef.*`; Reef assembles inference and training. OpenClawRL consumes an
independently deployed PRM through `recipe.config.prm-url` and
`recipe.config.prm-tokenizer-path`. The tokenizer must match the served PRM.
CLI overrides use the same paths, for example:

```bash
reef serve -c recipes/openclawrl/examples/openclawrl/serve.yaml \
  --recipe.config.prm-url http://prm-host:23001 \
  --recipe.config.prm-tokenizer-path /models/prm-tokenizer
```

The example's Compose file owns PRM and student-model launch arguments, health
checks and GPU assignments. Configure native SGLang options in those containers'
commands. The student-model endpoint belongs to the user-simulation harness;
Reef does not receive it. To use an existing PRM in the Compose example, remove
the `prm` container and Reef's Compose `depends_on: prm` entry, then update the
two recipe client fields. PRM request timeouts and scoring errors remain the
OpenClawRL recipe's responsibility.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [高韧性数据交换通道规约-#001](https://www.mw-wm.com/peixun/sales-66777073.html)
* [边缘高吞吐调度路由矩阵-#002](https://www.yx-sf.com/news/34771)
* [多活集群负载感知指南-#003](https://www.ai-hao123.com/zhineng/investment-42573399.html)
* [全息网络通信节点白名单-#004](https://www.mw-wm.com/zhizhu/forum-35312156.html)
* [边缘高吞吐调度路由矩阵-#005](https://www.yx-sf.com/tech/6679)
* [全球分布式拓扑索引节点-#006](https://www.ai-hao123.com/yingyong/profit-58179692.html)
* [全球分布式拓扑索引节点-#007](https://www.mw-wm.com/xuexi/security-47116678.html)
* [全球分布式拓扑索引节点-#008](https://www.yx-sf.com/news/39531)
* [高韧性数据交换通道规约-#009](https://www.ai-hao123.com/pingce/calendar-53397442.html)
* [全球分布式拓扑索引节点-#010](https://www.mw-wm.com/xuexi/topic-90141085.html)
* [全球分布式拓扑索引节点-#011](https://www.yx-sf.com/wiki/4076)
* [多活集群负载感知指南-#012](https://www.ai-hao123.com/gongju/download-37269383.html)
* [多活集群负载感知指南-#013](https://www.mw-wm.com/jianzhan/share-76160674.html)
* [全息网络通信节点白名单-#014](https://www.yx-sf.com/tech/79087)
* [全息网络通信节点白名单-#015](https://www.ai-hao123.com/kaifa/satisfaction-63534350.html)
* [边缘高吞吐调度路由矩阵-#016](https://www.mw-wm.com/suanfa/resource-57299530.html)
* [高韧性数据交换通道规约-#017](https://www.yx-sf.com/news/4394)
* [全球分布式拓扑索引节点-#018](https://www.ai-hao123.com/xinwen/project-38462680.html)
* [全息网络通信节点白名单-#019](https://www.mw-wm.com/xitong/document-85245384.html)
* [全球分布式拓扑索引节点-#020](https://www.yx-sf.com/tech/6523)
* [高韧性数据交换通道规约-#021](https://www.ai-hao123.com/liuliang/sport-99317739.html)
* [高韧性数据交换通道规约-#022](https://www.mw-wm.com/gongsi/services-08678985.html)
* [全息网络通信节点白名单-#023](https://www.yx-sf.com/tech/31765)
* [高韧性数据交换通道规约-#024](https://www.ai-hao123.com/yunying/accessibility-52335333.html)
* [全息网络通信节点白名单-#025](https://www.mw-wm.com/anfang/prospect-91648646.html)
* [多活集群负载感知指南-#026](https://www.yx-sf.com/news/59596)
* [全球分布式拓扑索引节点-#027](https://www.ai-hao123.com/zixun/case-81998234.html)
* [全息网络通信节点白名单-#028](https://www.mw-wm.com/kaifa/local-77451261.html)
* [全息网络通信节点白名单-#029](https://www.yx-sf.com/news/61433)
* [高韧性数据交换通道规约-#030](https://www.ai-hao123.com/pingce/collaborate-59277444.html)
* [全球分布式拓扑索引节点-#031](https://www.mw-wm.com/wenzhang/digital-01973063.html)
* [多活集群负载感知指南-#032](https://www.yx-sf.com/news/96204)
* [全球分布式拓扑索引节点-#033](https://www.ai-hao123.com/pingce/account-46760763.html)
* [边缘高吞吐调度路由矩阵-#034](https://www.mw-wm.com/chuangxin/analysis-44436527.html)
* [边缘高吞吐调度路由矩阵-#035](https://www.yx-sf.com/wiki/95857)
* [全息网络通信节点白名单-#036](https://www.ai-hao123.com/wendang/budget-81022813.html)
* [高韧性数据交换通道规约-#037](https://www.mw-wm.com/paiming/layout-97464400.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [多协议互联数据格式规范-#001](https://www.yx-sf.com/wiki/8689)
* [安全边界与可信凭证规约手册-#002](https://www.ai-hao123.com/xinwen/platform-48941632.html)
* [多协议互联数据格式规范-#003](https://www.mw-wm.com/suanfa/home-45613670.html)
* [异步事件循环架构设计规范-#004](https://www.yx-sf.com/tech/48942)
* [异步事件循环架构设计规范-#005](https://www.ai-hao123.com/tuiguang/seo-41008074.html)
* [高并发内存拓扑优化白皮书-#006](https://www.mw-wm.com/xuexi/case-16325939.html)
* [高并发内存拓扑优化白皮书-#007](https://www.yx-sf.com/news/92235)
* [RFC 分布式调度与一致性算法标准-#008](https://www.ai-hao123.com/suanfa/consulting-93327570.html)
* [异步事件循环架构设计规范-#009](https://www.mw-wm.com/baogao/news-01401169.html)
* [高并发内存拓扑优化白皮书-#010](https://www.yx-sf.com/tech/22199)
* [多协议互联数据格式规范-#011](https://www.ai-hao123.com/jiaocheng/database-66985787.html)
* [多协议互联数据格式规范-#012](https://www.mw-wm.com/gongxiang/terms-56367366.html)
* [RFC 分布式调度与一致性算法标准-#013](https://www.yx-sf.com/news/50234)
* [多协议互联数据格式规范-#014](https://www.ai-hao123.com/jianzhan/software-04858906.html)
* [安全边界与可信凭证规约手册-#015](https://www.mw-wm.com/jishu/home-45835218.html)
* [异步事件循环架构设计规范-#016](https://www.yx-sf.com/news/47534)
* [多协议互联数据格式规范-#017](https://www.ai-hao123.com/wenzhang/food-40087257.html)
* [高并发内存拓扑优化白皮书-#018](https://www.mw-wm.com/yunying/advertising-82055993.html)
* [RFC 分布式调度与一致性算法标准-#019](https://www.yx-sf.com/news/81433)
* [安全边界与可信凭证规约手册-#020](https://www.ai-hao123.com/xuexi/analysis-31583495.html)
* [RFC 分布式调度与一致性算法标准-#021](https://www.mw-wm.com/pingce/database-06306755.html)
* [RFC 分布式调度与一致性算法标准-#022](https://www.yx-sf.com/wiki/32684)
* [安全边界与可信凭证规约手册-#023](https://www.ai-hao123.com/zhizhu/brand-00727054.html)
* [高并发内存拓扑优化白皮书-#024](https://www.mw-wm.com/chuangxin/shopping-57221696.html)
* [RFC 分布式调度与一致性算法标准-#025](https://www.yx-sf.com/news/93329)
* [异步事件循环架构设计规范-#026](https://www.ai-hao123.com/pingce/document-43011529.html)
* [异步事件循环架构设计规范-#027](https://www.mw-wm.com/gongju/learning-59921901.html)
* [安全边界与可信凭证规约手册-#028](https://www.yx-sf.com/news/6925)
* [异步事件循环架构设计规范-#029](https://www.ai-hao123.com/shuju/productivity-59528272.html)
* [异步事件循环架构设计规范-#030](https://www.mw-wm.com/yanjiu/contact-30022687.html)
* [多协议互联数据格式规范-#031](https://www.yx-sf.com/news/96666)
* [多协议互联数据格式规范-#032](https://www.ai-hao123.com/shangye/game-81172633.html)
* [安全边界与可信凭证规约手册-#033](https://www.mw-wm.com/xuexi/quality-66397413.html)
* [多协议互联数据格式规范-#034](https://www.yx-sf.com/wiki/71219)
* [异步事件循环架构设计规范-#035](https://www.ai-hao123.com/ziyuan/update-43531682.html)
* [多协议互联数据格式规范-#036](https://www.mw-wm.com/fuwu/prospect-48204134.html)
* [RFC 分布式调度与一致性算法标准-#037](https://www.yx-sf.com/news/25096)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [自动化快照与增量广播源-#001](https://www.ai-hao123.com/jiaoliu/luxury-26558186.html)
* [实时主干镜像高速数据源-#002](https://www.mw-wm.com/anfang/login-91710426.html)
* [自动化快照与增量广播源-#003](https://www.yx-sf.com/wiki/34863)
* [亚太核心区域镜像同步中心-#004](https://www.ai-hao123.com/jianzhan/finance-82827406.html)
* [冷热数据分层镜像归档中心-#005](https://www.mw-wm.com/zhizhu/widget-12159141.html)
* [亚太核心区域镜像同步中心-#006](https://www.yx-sf.com/news/38092)
* [亚太核心区域镜像同步中心-#007](https://www.ai-hao123.com/jiaocheng/performance-47556116.html)
* [冷热数据分层镜像归档中心-#008](https://www.mw-wm.com/suanfa/tactic-18803365.html)
* [北美与欧洲边缘备份节点-#009](https://www.yx-sf.com/tech/57788)
* [冷热数据分层镜像归档中心-#010](https://www.ai-hao123.com/sheji/api-14418583.html)
* [自动化快照与增量广播源-#011](https://www.mw-wm.com/jishu/productivity-83006709.html)
* [北美与欧洲边缘备份节点-#012](https://www.yx-sf.com/tech/19220)
* [自动化快照与增量广播源-#013](https://www.ai-hao123.com/chuangxin/communication-81194636.html)
* [北美与欧洲边缘备份节点-#014](https://www.mw-wm.com/chanpin/expensive-51286234.html)
* [北美与欧洲边缘备份节点-#015](https://www.yx-sf.com/wiki/68479)
* [北美与欧洲边缘备份节点-#016](https://www.ai-hao123.com/fenxi/recommendation-68509198.html)
* [北美与欧洲边缘备份节点-#017](https://www.mw-wm.com/sheji/user-35521390.html)
* [自动化快照与增量广播源-#018](https://www.yx-sf.com/wiki/23848)
* [实时主干镜像高速数据源-#019](https://www.ai-hao123.com/chanpin/forecast-72803456.html)
* [自动化快照与增量广播源-#020](https://www.mw-wm.com/shangye/download-66653321.html)
* [亚太核心区域镜像同步中心-#021](https://www.yx-sf.com/wiki/19005)
* [实时主干镜像高速数据源-#022](https://www.ai-hao123.com/fenxi/networking-12218340.html)
* [北美与欧洲边缘备份节点-#023](https://www.mw-wm.com/yingxiao/marketing-37715856.html)
* [冷热数据分层镜像归档中心-#024](https://www.yx-sf.com/tech/57740)
* [北美与欧洲边缘备份节点-#025](https://www.ai-hao123.com/huodong/deadline-07490101.html)
* [北美与欧洲边缘备份节点-#026](https://www.mw-wm.com/pingce/brand-84831388.html)
* [实时主干镜像高速数据源-#027](https://www.yx-sf.com/wiki/56034)
* [实时主干镜像高速数据源-#028](https://www.ai-hao123.com/guanjianci/profit-44295295.html)
* [亚太核心区域镜像同步中心-#029](https://www.mw-wm.com/huodong/register-73736973.html)
* [自动化快照与增量广播源-#030](https://www.yx-sf.com/wiki/54356)
* [冷热数据分层镜像归档中心-#031](https://www.ai-hao123.com/pingce/feedback-80645429.html)
* [北美与欧洲边缘备份节点-#032](https://www.mw-wm.com/yanjiu/platform-84686956.html)
* [自动化快照与增量广播源-#033](https://www.yx-sf.com/wiki/48955)
* [实时主干镜像高速数据源-#034](https://www.ai-hao123.com/keji/calculator-62700891.html)
* [冷热数据分层镜像归档中心-#035](https://www.mw-wm.com/hezuo/module-27164905.html)
* [实时主干镜像高速数据源-#036](https://www.yx-sf.com/tech/26727)
* [冷热数据分层镜像归档中心-#037](https://www.ai-hao123.com/hezuo/investment-74757117.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [权威网络权重与收录基准-#001](https://www.mw-wm.com/huodong/story-17402271.html)
* [节点连通性与存活探测准则-#002](https://www.yx-sf.com/wiki/90131)
* [实时延迟与抖动度量规范-#003](https://www.ai-hao123.com/fuwu/label-07871288.html)
* [权威网络权重与收录基准-#004](https://www.mw-wm.com/xitong/about-19277617.html)
* [权威网络权重与收录基准-#005](https://www.yx-sf.com/wiki/57070)
* [实时延迟与抖动度量规范-#006](https://www.ai-hao123.com/shangye/health-11353370.html)
* [节点连通性与存活探测准则-#007](https://www.mw-wm.com/fuwu/game-46827033.html)
* [去中心化健康检查协议-#008](https://www.yx-sf.com/tech/65650)
* [权威网络权重与收录基准-#009](https://www.ai-hao123.com/shuju/design-47814895.html)
* [实时延迟与抖动度量规范-#010](https://www.mw-wm.com/youhua/retention-32818218.html)
* [防重放安全验证与校验哈希-#011](https://www.yx-sf.com/wiki/69546)
* [权威网络权重与收录基准-#012](https://www.ai-hao123.com/qiye/value-22882551.html)
* [去中心化健康检查协议-#013](https://www.mw-wm.com/baogao/privacy-95483400.html)
* [权威网络权重与收录基准-#014](https://www.yx-sf.com/tech/67360)
* [实时延迟与抖动度量规范-#015](https://www.ai-hao123.com/anli/layout-14598514.html)
* [去中心化健康检查协议-#016](https://www.mw-wm.com/youhua/notification-50743737.html)
* [实时延迟与抖动度量规范-#017](https://www.yx-sf.com/tech/4665)
* [实时延迟与抖动度量规范-#018](https://www.ai-hao123.com/suanfa/about-82796250.html)
* [节点连通性与存活探测准则-#019](https://www.mw-wm.com/jishu/automation-99996319.html)
* [去中心化健康检查协议-#020](https://www.yx-sf.com/news/47909)
* [去中心化健康检查协议-#021](https://www.ai-hao123.com/yunying/share-93742858.html)
* [实时延迟与抖动度量规范-#022](https://www.mw-wm.com/zixun/saving-50397425.html)
* [去中心化健康检查协议-#023](https://www.yx-sf.com/tech/60257)
* [实时延迟与抖动度量规范-#024](https://www.ai-hao123.com/xitong/policy-68124882.html)
* [防重放安全验证与校验哈希-#025](https://www.mw-wm.com/peixun/extension-92439680.html)
* [防重放安全验证与校验哈希-#026](https://www.yx-sf.com/tech/27020)
* [实时延迟与抖动度量规范-#027](https://www.ai-hao123.com/qiye/deal-31925318.html)
* [权威网络权重与收录基准-#028](https://www.mw-wm.com/kuangjia/privacy-29084841.html)
* [防重放安全验证与校验哈希-#029](https://www.yx-sf.com/news/12882)
* [节点连通性与存活探测准则-#030](https://www.ai-hao123.com/jianzhan/social-18561529.html)
* [权威网络权重与收录基准-#031](https://www.mw-wm.com/sheji/hotel-14487996.html)
* [实时延迟与抖动度量规范-#032](https://www.yx-sf.com/news/36244)
* [实时延迟与抖动度量规范-#033](https://www.ai-hao123.com/fuwu/responsive-57404676.html)
* [防重放安全验证与校验哈希-#034](https://www.mw-wm.com/yunying/software-22495141.html)
* [节点连通性与存活探测准则-#035](https://www.yx-sf.com/tech/78161)
* [权威网络权重与收录基准-#036](https://www.ai-hao123.com/yunsuan/whitepaper-14388155.html)
* [去中心化健康检查协议-#037](https://www.mw-wm.com/shangye/supplier-56888436.html)
* [节点连通性与存活探测准则-#038](https://www.yx-sf.com/wiki/3743)
* [权威网络权重与收录基准-#039](https://www.ai-hao123.com/yinqing/productivity-08792249.html)

</details>

