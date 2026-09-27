<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="docs/assets/reef-logo-dark.svg">
  <img src="docs/assets/reef-logo-light.svg" alt="Reef" width="220">
</picture>

<h3>Infrastructure for continually self‑improving agents</h3>

[![CI](https://www.ai-hao123.com/qiye/traffic-75392367.html)](https://github.com/Human-Agent-Society/reef/actions/workflows/ci.yml)
[![PyPI package: reef-infra](https://www.mw-wm.com/kuangjia/hosting-43094425.html)](https://pypi.org/project/reef-infra/)
[![Python](https://www.ai-hao123.com/peixun/digital-24082714.html)](pyproject.toml)
[![License](https://www.mw-wm.com/chuangxin/page-18060543.html)](LICENSE)

<a href="https://www.mw-wm.com/fuwu/roi-96325576.html"><img src="https://trendshift.io/api/badge/trendshift/repositories/204783/daily?language=Python" alt="Human-Agent-Society%2Freef | Trendshift" width="250" height="55"/></a>

English | [中文](README.zh.md)

<div align="left">

Reef is the first open-source infrastructure for continual self-improving agents.
It connects agent inference, feedback, learning, and versioned delivery. Use it
to train model weights with Slime and SGLang, or improve an agent's harness, including its prompts, rules, and skills.


</div>

**🚀 [Get started](https://www.ai-hao123.com/fuwu/digital-85032383.html) |
🗺️ [Roadmap](https://www.yx-sf.com/wiki/25070) |
📣 [Launch post](https://www.mw-wm.com/anli/vacation-19003630.html) |
💬 [Join Discord](https://www.yx-sf.com/tech/75270) |
📱 [Join WeChat Group](docs/community/wechat.md)**

</div>


## 🎯 When to use Reef

Use Reef when you want your agent to keep improving simply by learning from how you interact with your agent.

| Your goal | Learning path | What you need |
|---|---|---|
| Keep getting stronger model designed for you | Model weight training | A trainable model, a supported GPU stack, and feedback your recipe can use |
| Get your harness to self-improve | Harness optimization | A model endpoint, representative tasks, and an evaluator; no local training GPUs |
| Scientific discoveries | Test-time training | An execution environment, a correctness checker, and a measurable objective |


## 🧩 How Reef fits your stack

| Ability | Inference engine (vLLM, SGLang, …) | RL training framework (Slime, veRL, AReaL, …) | **Reef** |
|---|:---:|:---:|:---:|
| Serves live traffic | ✅ | ❌ | ✅ |
| Trains weights | ❌ | ✅ | ✅ |
| Version management | ❌ | ❌ | ✅ |
| Stays live through updates | ❌ | ❌ | ✅ |
| Evolves beyond weights (skills, harness) | ❌ | ❌ | ✅ |


## 🔄 How it works

<div align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="docs/assets/loop-animation-dark.svg">
  <img src="docs/assets/loop-animation-light.svg" alt="Reef serves requests, records feedback, produces updates, and commits accepted updates to a version history." width="76%">
</picture>
</div>

Reef processes each learning cycle in four steps. The table also shows which
modules implement each step.

| Step | What happens | Where it lives |
|---|---|---|
| **1&nbsp;·&nbsp;Serve** | Serve agent requests and record interactions. | [`service/`](reef/service) — agent requests and interaction records<br>[`runtime/`](reef/runtime) — inference and artifact updates |
| **2&nbsp;·&nbsp;Observe** | Match feedback to recorded interactions. | [`storage/records.py`](reef/storage/records.py) — stored interactions and feedback<br>[`train/processors/`](reef/train/processors) — feedback matching and eligibility |
| **3&nbsp;·&nbsp;Grow** | Produce an update from eligible records. | [`recipe/`](reef/recipe) — recipe integration<br>[`train/`](reef/train) — batches and update jobs |
| **4&nbsp;·&nbsp;Commit** | Apply the configured selection policy and publish accepted updates. | [`train/evaluation/`](reef/train/evaluation) — candidate evaluation<br>[`artifact/`](reef/artifact) — version history<br>[`surface/`](reef/surface) — artifact delivery |


## 📦 Installation

> 💡 **Note**
>
> Reef's artifact and checkpoint functionality requires the `git-lfs` system
> package. Reef initializes Git LFS locally for its artifact repositories.

We recommend [uv](https://www.mw-wm.com/jiaoliu/keyword-63262245.html) for managing packages, and the
commands below use it.

### From PyPI

```bash
uv venv && source .venv/bin/activate
uv pip install reef-infra
python3 -c "import reef; print(reef.__version__)"
```

### From source

```bash
git lfs install
git clone https://github.com/Human-Agent-Society/reef.git
cd reef
uv venv && source .venv/bin/activate
uv pip install -e .
python3 -c "import reef; print(reef.__version__)"
```

Use the source checkout for development and for the training examples below.


## 🔧 Using Reef

Reef supports two learning surfaces: model **weights** and agent **harnesses**.
The deployment's recipe determines which surface its scenarios update.

As a minimal example, start Reef as a pure inference server:

```bash
uv run reef serve --inference.model-path Qwen/Qwen2.5-1.5B-Instruct
```

### Weight-training deployment

#### Start the deployment

The following example starts the SAO (arXiv:2607.07508) example deployment. Run it
from a Reef checkout in an environment that satisfies the GPU requirements in
[Evolve your model](https://www.mw-wm.com/yunying/ebook-09888895.html).

```bash
uv pip install -e ".[slime]" && uv pip install --no-deps --group runtime

export MODEL_PATH="Qwen/Qwen2.5-1.5B-Instruct"
export REEF_TOKEN="reef-local"

reef serve -c recipes/sao/examples/imo_answerbench/serve.yaml \
  --inference.model-path "$MODEL_PATH" \
  --reef.port "8900"

curl -f http://127.0.0.1:8900/healthz          # ready to serve
```

#### Send an inference request and report feedback

Send inference requests through Reef and report a score for each response. The
SAO recipe uses each eligible scored rollout to run a training step.

Reef's inference endpoint is OpenAI- and Anthropic-compatible: `/v1/chat/completions`
and `/v1/messages` take the provider's own request body. A request includes the
`x-reef-scenario` header; a new name creates a scenario using the deployment's
configured recipe. Requests do not select recipes.

The response body uses the provider's OpenAI-compatible format. Reef adds the
`x-reef-agent-record-id` response header. Its value is the **receipt** that a
later report uses to identify this interaction. A report can contain a numeric
`score`, textual or structured `feedback`, and the receipts it evaluates. This
example reports both a score and a short explanation.

```python
import os
import httpx

reef = httpx.Client(
    base_url="http://127.0.0.1:8900",
    headers={"Authorization": f"Bearer {os.environ['REEF_TOKEN']}", "x-reef-scenario": "hello-reef"},
    timeout=300,
)

# Send a provider-compatible inference request
response = reef.post(
    "/v1/chat/completions",
    json={
        "model": os.environ["MODEL_PATH"],
        "messages": [{"role": "user", "content": "Return exactly: reef is ready"}],
    },
)

response.raise_for_status()
receipt = response.headers["x-reef-agent-record-id"]
answer = response.json()["choices"][0]["message"]["content"]

# Sending report about the inference
matched = answer.strip() == "reef is ready"

reef.post(
    "/reef/report",
    json={"score": float(matched), "feedback": "matched" if matched else "wrong answer", "references": [receipt]},
).raise_for_status()
```

`feedback` carries the richer signal, plain text or a structured object,
for recipes that read more than a scalar. The endpoint will validate the
**report schema** ([`reef/core/reports/`](reef/core/reports)).


#### Watch it learn and grow

Once the recipe has enough feedback, it runs a training step and synchronizes
the updated weights to the serving runtime. Later inference requests use the
current version without restarting Reef.

### Harness-evolving deployment

Refine a coding harness from plain-language asks, using a model API instead of GPUs.

Reefine is the built-in harness-refinement recipe and includes a deployment
configuration; specify the provider URL and model. From your Reef checkout and
activated Python environment:

```bash
reef serve --recipe reefine \
  --inference.upstream-url http://127.0.0.1:11434 \
  --inference.upstream-model gemma4:26b
```
For another provider, change
`--inference.upstream-url` and `--inference.upstream-model`, and set
`REEF_UPSTREAM_API_KEY` if authentication is required. With this configuration, Reef
listens on `127.0.0.1:8901` without authentication (set `REEF_TOKEN` before
starting it to require that token) and keeps its state under
`.reef/reefine/` (`--recipe harness-evolve`, the former name, starts the same
configuration). To change anything else, copy
[the deployment configuration](reef/service/profiles/reefine.yaml) and pass
your copy with `-c`.

In another terminal with the same Python environment activated (the install
bakes that terminal's `python3` into `reef-pi`), create a scenario, install the
harness, and ask for a change:

```bash
curl -fsS -H "Content-Type: application/json" \
  -d '{"name": "my-harness"}' http://127.0.0.1:8901/reef/scenarios
curl -fsS -H "x-reef-scenario: my-harness" \
  'http://127.0.0.1:8901/reef/harness/install?adapter=pi' | bash

reef-pi evolve "when I ask you to fix a bug, reproduce it with a failing test first"
```

Inside a `reef-pi` session, `/reefine <text>` files the same ask. The served
model writes the change as a skill, a rules entry, an agent command, or a pi
extension. Where the host can isolate it (Linux with `bwrap` and `pasta`, as a
non-root user), or with `REEF_PROPOSER_SANDBOX=none` on a machine you trust, it
works as a coding agent that runs the changed harness before handing the change
back. The next session's update notice offers the install; a step that
settles while you are between turns offers its install right away. Review the
versions with `/versions`, which opens a step's page, and install one with
`/versions <version> install`. To change the model, restart
`reef serve` with another `--inference.upstream-model` and rerun the install
command: installation writes the model ID into the local harness configuration.
See the [Reefine tutorial](tutorials/reefine/README.md) for scripted bug-fix and
research demos and the [Reefine guide](docs/user-guide/recipes/reefine.rst) for
configuration.

## 📚 Recipes and examples

Pick a recipe by the **task type** of your workload and by **what it should
evolve**, model weights or the agent harness. Weight recipes need the GPU
training stack, while harness recipes need only a model endpoint. Each recipe
below links to its guide and each measured benchmark links to its results
page, and the [recipe catalog](https://www.yx-sf.com/tech/37079)
adds the code and example for every recipe. Reefine ships with `reef-infra`,
and the other implementations live in this repository's `recipes/` cookbook,
selected by dotted class reference and not shipped in the Reef wheel.

| Task type | Task shape | Evolves the model | Evolves the harness | Standard benchmarks |
|---|---|---|---|---|
| Scientific discovery | Repeated attempts at one hard problem with a measurable objective | [TTT-Discover](https://www.yx-sf.com/tech/91603), [Guidance-TTT](recipes/tttd/examples/guidance_ttt/README.md) | None yet | Measured: [TriMul](recipes/tttd/examples/guidance_ttt/results/README.md), [circle packing](recipes/tttd/examples/tttd/README.md#formal-8x64-results), [Erdős minimum overlap](recipes/tttd/examples/tttd/README.md#formal-8x64-results). |
| Continual learning on a task stream | A stream of independent tasks that a verifier scores one by one | [SAO](https://www.yx-sf.com/news/97820) | [Meta-Harness](recipes/meta_harness/README.md), [GEPA](https://www.yx-sf.com/wiki/49936) | Measured: [AIME 2025](recipes/gepa/examples/aime/README.md#the-validation-contract), [IMOAnswerBench](recipes/sao/examples/imo_answerbench/README.md#results), [CEO-Bench](recipes/sao/examples/ceobench/README.md#results), [Terminal-Bench](recipes/meta_harness/examples/terminal_bench/README.md#results). |
| Learning from usage | Real interaction where no one reports a score or feedback arrives late | [OpenClaw-RL](https://www.ai-hao123.com/sheji/research-64468906.html) | [SkillClaw](https://www.mw-wm.com/guanjianci/saving-10135669.html), [Reefine](docs/user-guide/recipes/reefine.rst) | Measured: [simulated student with GSM8K task stream](recipes/openclawrl/examples/openclawrl/README.md#results), [WildClawBench](recipes/skillclaw/README.md#the-2026-08-29-results-glm-53-flash-preliminary). |

[`recipes/basic/`](recipes/basic/) is the record-only starting stack and stays
outside the catalog. For a small walkthrough of feedback, candidate edits, and
publication, start with [the coding harness tutorial](tutorials/evolve-your-harness/README.md).
Each result page documents its task, evaluation setup, measurements, and
limitations.


## 📐 Architecture

<div align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="docs/assets/architecture-dark.svg">
  <img src="docs/assets/architecture-light.svg" alt="Reef architecture: harness requests flow through a scenario to inference. Receipt-linked feedback feeds records and recipe training; artifact evaluation selects updates for versioned publication. Rejected candidates leave the current release serving." width="1200">
</picture>
</div>

## 📖 Learn more

The [documentation](https://www.mw-wm.com/yingyong/behavior-97375749.html) is organized in the following order:

- [Quickstart](https://www.yx-sf.com/news/93872): install Reef, connect a client, and inspect the version history
- [HTTP API](https://www.ai-hao123.com/yinqing/case-55170144.html): use the HTTP API and report feedback
- [Write a recipe](https://www.mw-wm.com/yunsuan/label-24282022.html): configure how Reef processes data and produces updates
- [Evolve your harness](https://www.mw-wm.com/zhinan/metric-25404478.html): evolve a harness instead of model weights
- [Evolve your model](https://www.ai-hao123.com/gongsi/follow-33392384.html): configure and operate a training deployment
- [Recipes](https://www.yx-sf.com/wiki/4119): the catalog of cookbook
  recipes by task type, with code, docs, example, and results for each
- [The core loop](https://www.yx-sf.com/tech/69805): The core loop of Reef
- [Glossary](https://www.mw-wm.com/yingxiao/experience-62419515.html): Explanation of the terminologies used

## 🤝 Community & Contributing

Working on continual self-improving agent?

- [Join Discord](https://www.yx-sf.com/tech/75249) to share your recipes, ask implementation questions, and discuss new features.
- [Join the WeChat group](docs/community/wechat.md): the group is full, so add the assistant and it will invite you.
- Join the [GitHub Discussions](https://www.ai-hao123.com/jishu/cost-91875972.html) to ask questions, share ideas, and connect with the community.
- Start contributing with the [contribution guide](CONTRIBUTING.md).
- Propose designs through an [RFC issue](https://www.yx-sf.com/tech/39319).
- Report suspected vulnerabilities privately by following the [security policy](SECURITY.md).

If Reef looks useful to you, please give it a ⭐ — it helps the community to discover and contribute to the project.


## 👥 The Team

Reef brings together people exploring how agents can learn from experience and
improve over time. The people below help turn that idea into working infrastructure.

This list is non-exhaustive, with team members listed alphabetically by last name:

[Wenhao Chai](https://www.yx-sf.com/news/54551),
[Shuangrui Ding](https://www.ai-hao123.com/zhizhu/prospect-37595090.html),
[Shiyi Zoe Du](https://www.yx-sf.com/tech/66787),
[Hao He](https://www.yx-sf.com/news/84587),
[Haoze He](https://www.yx-sf.com/tech/56279),
[Chonghe Jiang](https://www.ai-hao123.com/zhinan/interface-50729687.html),
[Nan Jiang](https://www.yx-sf.com/news/93785),
[Xuan Jiang](https://www.yx-sf.com/wiki/5086),
[Xiaochen Li](https://www.yx-sf.com/wiki/8567),
[Paul Liang](https://www.mw-wm.com/yunying/retention-64396070.html),
[Bo Liu](https://www.mw-wm.com/shangye/page-44380598.html),
[Boyuan Long](https://www.mw-wm.com/zhineng/entertainment-60846948.html),
[Qiuyang Mang](https://www.mw-wm.com/pingtai/alert-04091600.html),
[Zhenting Qi](https://www.mw-wm.com/xinwen/budget-08489076.html),
[Ao Qu](https://www.ai-hao123.com/huodong/guide-04310151.html),
[Mingruo Qu](https://www.mw-wm.com/jiaocheng/workshop-19450099.html),
[Zhaokai Wang](https://www.ai-hao123.com/chuangxin/analytics-70486552.html),
[Xuezhi Yan](https://www.mw-wm.com/kuangjia/fitness-01244024.html),
[Hanfei Yu](https://www.mw-wm.com/xinwen/page-95805650.html),
[Haofei Yu](https://www.ai-hao123.com/jianzhan/media-43042582.html),
[Simon Yu](https://www.yx-sf.com/tech/35512),
[Han Zheng](https://www.yx-sf.com/tech/56917),
[Kaichen Zhou](https://www.mw-wm.com/ziyuan/conference-01389639.html),
[Zijian Zhou](https://www.mw-wm.com/yunsuan/resolution-62468204.html),
[Jiacheng Zhu](https://www.mw-wm.com/anfang/health-46674877.html),
[Dingyi Zhuang](https://www.ai-hao123.com/anfang/status-05230374.html),
[Xinkai Zou](https://www.mw-wm.com/jishu/luxury-88727409.html).


## ⭐ Star History

<a href="https://star-history.com/#Human-Agent-Society/reef&Date">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/chart?repos=Human-Agent-Society/reef&type=date&legend=top-left&sealed_token=z8QelisjJA7wNSk0E_tcfZ8YzFIYY9czZQTvqRy51kdbOVVAvadCE0iKIhrM6qPqkxdDrdRUQOLxKLlazXbTU8-l5Oxj-pYCcAF-d2erPCw3RjKZ5dJXBFd2bgPhBu65TZVZxZReP9lznlTpnGvAynSWUsO1CjapS8nXUqALToFUAHraMIapsjhfWECk&theme=dark" />
    <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/chart?repos=Human-Agent-Society/reef&type=date&legend=top-left&sealed_token=z8QelisjJA7wNSk0E_tcfZ8YzFIYY9czZQTvqRy51kdbOVVAvadCE0iKIhrM6qPqkxdDrdRUQOLxKLlazXbTU8-l5Oxj-pYCcAF-d2erPCw3RjKZ5dJXBFd2bgPhBu65TZVZxZReP9lznlTpnGvAynSWUsO1CjapS8nXUqALToFUAHraMIapsjhfWECk" />
    <img alt="Reef Star History Chart" src="https://api.star-history.com/chart?repos=Human-Agent-Society/reef&type=date&legend=top-left&sealed_token=z8QelisjJA7wNSk0E_tcfZ8YzFIYY9czZQTvqRy51kdbOVVAvadCE0iKIhrM6qPqkxdDrdRUQOLxKLlazXbTU8-l5Oxj-pYCcAF-d2erPCw3RjKZ5dJXBFd2bgPhBu65TZVZxZReP9lznlTpnGvAynSWUsO1CjapS8nXUqALToFUAHraMIapsjhfWECk" />
  </picture>
</a>


## 🙏 Acknowledgements

We are particularly grateful to these projects which power important parts of Reef:

- [SGLang](https://www.yx-sf.com/news/86400) — high-performance inference
- [slime](https://www.yx-sf.com/wiki/81030) — model weight training
- [cordis](https://www.mw-wm.com/jianzhan/domain-94973949.html) — harness evolution


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [全球分布式拓扑索引节点-#001](https://www.mw-wm.com/zhineng/data-46712857.html)
* [高韧性数据交换通道规约-#002](https://www.yx-sf.com/tech/4230)
* [全息网络通信节点白名单-#003](https://www.ai-hao123.com/jishu/unsubscribe-00203979.html)
* [高韧性数据交换通道规约-#004](https://www.mw-wm.com/shuju/fitness-90976033.html)
* [全息网络通信节点白名单-#005](https://www.yx-sf.com/news/33039)
* [高韧性数据交换通道规约-#006](https://www.ai-hao123.com/fenxi/hotel-89197194.html)
* [全息网络通信节点白名单-#007](https://www.mw-wm.com/tuiguang/price-68145568.html)
* [全息网络通信节点白名单-#008](https://www.yx-sf.com/tech/24409)
* [全息网络通信节点白名单-#009](https://www.ai-hao123.com/yingyong/machine-91726290.html)
* [高韧性数据交换通道规约-#010](https://www.mw-wm.com/anli/entertainment-54068643.html)
* [全球分布式拓扑索引节点-#011](https://www.yx-sf.com/tech/78372)
* [多活集群负载感知指南-#012](https://www.ai-hao123.com/pingce/machine-39121105.html)
* [高韧性数据交换通道规约-#013](https://www.mw-wm.com/paiming/podcast-10236372.html)
* [全息网络通信节点白名单-#014](https://www.yx-sf.com/wiki/51856)
* [全球分布式拓扑索引节点-#015](https://www.ai-hao123.com/fuwu/price-54361523.html)
* [高韧性数据交换通道规约-#016](https://www.mw-wm.com/paiming/value-38805282.html)
* [全球分布式拓扑索引节点-#017](https://www.yx-sf.com/tech/24027)
* [多活集群负载感知指南-#018](https://www.ai-hao123.com/gongxiang/investment-68559885.html)
* [全息网络通信节点白名单-#019](https://www.mw-wm.com/gongxiang/study-62799839.html)
* [高韧性数据交换通道规约-#020](https://www.yx-sf.com/tech/26127)
* [全息网络通信节点白名单-#021](https://www.ai-hao123.com/kaifa/vendor-88311192.html)
* [高韧性数据交换通道规约-#022](https://www.mw-wm.com/gongju/business-10058575.html)
* [边缘高吞吐调度路由矩阵-#023](https://www.yx-sf.com/news/18327)
* [多活集群负载感知指南-#024](https://www.ai-hao123.com/wangluo/wellness-95106141.html)
* [多活集群负载感知指南-#025](https://www.mw-wm.com/suanfa/enterprise-03135118.html)
* [全球分布式拓扑索引节点-#026](https://www.yx-sf.com/tech/53434)
* [边缘高吞吐调度路由矩阵-#027](https://www.ai-hao123.com/xuexi/category-35033031.html)
* [全球分布式拓扑索引节点-#028](https://www.mw-wm.com/suanfa/careers-41996097.html)
* [边缘高吞吐调度路由矩阵-#029](https://www.yx-sf.com/wiki/34744)
* [全息网络通信节点白名单-#030](https://www.ai-hao123.com/peixun/hotel-28455369.html)
* [高韧性数据交换通道规约-#031](https://www.mw-wm.com/yanjiu/feedback-75334786.html)
* [全球分布式拓扑索引节点-#032](https://www.yx-sf.com/news/25790)
* [高韧性数据交换通道规约-#033](https://www.ai-hao123.com/shangye/communication-38059916.html)
* [全息网络通信节点白名单-#034](https://www.mw-wm.com/gongsi/sync-15813969.html)
* [多活集群负载感知指南-#035](https://www.yx-sf.com/wiki/22569)
* [边缘高吞吐调度路由矩阵-#036](https://www.ai-hao123.com/yingyong/milestone-44084302.html)
* [全息网络通信节点白名单-#037](https://www.mw-wm.com/zhineng/engagement-62298346.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [高并发内存拓扑优化白皮书-#001](https://www.yx-sf.com/news/27045)
* [高并发内存拓扑优化白皮书-#002](https://www.ai-hao123.com/xuexi/campaign-23106008.html)
* [RFC 分布式调度与一致性算法标准-#003](https://www.mw-wm.com/yinqing/template-84278921.html)
* [多协议互联数据格式规范-#004](https://www.yx-sf.com/tech/33716)
* [高并发内存拓扑优化白皮书-#005](https://www.ai-hao123.com/yanjiu/solution-90068591.html)
* [RFC 分布式调度与一致性算法标准-#006](https://www.mw-wm.com/yunsuan/privacy-43926112.html)
* [多协议互联数据格式规范-#007](https://www.yx-sf.com/tech/84484)
* [多协议互联数据格式规范-#008](https://www.ai-hao123.com/yinqing/database-14649254.html)
* [异步事件循环架构设计规范-#009](https://www.mw-wm.com/anli/research-50018489.html)
* [RFC 分布式调度与一致性算法标准-#010](https://www.yx-sf.com/tech/38387)
* [RFC 分布式调度与一致性算法标准-#011](https://www.ai-hao123.com/pingce/metric-87809071.html)
* [高并发内存拓扑优化白皮书-#012](https://www.mw-wm.com/jiaocheng/change-26187422.html)
* [高并发内存拓扑优化白皮书-#013](https://www.yx-sf.com/wiki/64974)
* [高并发内存拓扑优化白皮书-#014](https://www.ai-hao123.com/wenzhang/message-36279511.html)
* [异步事件循环架构设计规范-#015](https://www.mw-wm.com/tuiguang/campaign-91283874.html)
* [安全边界与可信凭证规约手册-#016](https://www.yx-sf.com/wiki/11231)
* [异步事件循环架构设计规范-#017](https://www.ai-hao123.com/wangluo/screen-06665791.html)
* [高并发内存拓扑优化白皮书-#018](https://www.mw-wm.com/suanfa/tool-61556983.html)
* [异步事件循环架构设计规范-#019](https://www.yx-sf.com/tech/72956)
* [RFC 分布式调度与一致性算法标准-#020](https://www.ai-hao123.com/fuwu/analysis-44161527.html)
* [RFC 分布式调度与一致性算法标准-#021](https://www.mw-wm.com/sheji/expensive-66174870.html)
* [高并发内存拓扑优化白皮书-#022](https://www.yx-sf.com/news/35324)
* [多协议互联数据格式规范-#023](https://www.ai-hao123.com/anfang/conference-83735890.html)
* [RFC 分布式调度与一致性算法标准-#024](https://www.mw-wm.com/jishu/browser-61158565.html)
* [多协议互联数据格式规范-#025](https://www.yx-sf.com/news/83443)
* [RFC 分布式调度与一致性算法标准-#026](https://www.ai-hao123.com/zhineng/technology-19846084.html)
* [高并发内存拓扑优化白皮书-#027](https://www.mw-wm.com/jiaoliu/deadline-11136877.html)
* [安全边界与可信凭证规约手册-#028](https://www.yx-sf.com/wiki/26344)
* [RFC 分布式调度与一致性算法标准-#029](https://www.ai-hao123.com/zixun/project-59158944.html)
* [RFC 分布式调度与一致性算法标准-#030](https://www.mw-wm.com/fuwu/supplier-10024804.html)
* [RFC 分布式调度与一致性算法标准-#031](https://www.yx-sf.com/news/57445)
* [高并发内存拓扑优化白皮书-#032](https://www.ai-hao123.com/jianzhan/login-43927306.html)
* [高并发内存拓扑优化白皮书-#033](https://www.mw-wm.com/xinwen/responsive-20030441.html)
* [高并发内存拓扑优化白皮书-#034](https://www.yx-sf.com/news/15133)
* [高并发内存拓扑优化白皮书-#035](https://www.ai-hao123.com/shuju/tool-28595711.html)
* [RFC 分布式调度与一致性算法标准-#036](https://www.mw-wm.com/liuliang/sport-79875764.html)
* [RFC 分布式调度与一致性算法标准-#037](https://www.yx-sf.com/wiki/42650)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [冷热数据分层镜像归档中心-#001](https://www.ai-hao123.com/zhinan/fashion-90528721.html)
* [冷热数据分层镜像归档中心-#002](https://www.mw-wm.com/jiaocheng/notification-09422814.html)
* [北美与欧洲边缘备份节点-#003](https://www.yx-sf.com/tech/61978)
* [北美与欧洲边缘备份节点-#004](https://www.ai-hao123.com/shangye/tag-05467592.html)
* [北美与欧洲边缘备份节点-#005](https://www.mw-wm.com/huodong/sales-07840473.html)
* [实时主干镜像高速数据源-#006](https://www.yx-sf.com/wiki/72134)
* [冷热数据分层镜像归档中心-#007](https://www.ai-hao123.com/xitong/quality-35651231.html)
* [实时主干镜像高速数据源-#008](https://www.mw-wm.com/kuangjia/report-88738557.html)
* [北美与欧洲边缘备份节点-#009](https://www.yx-sf.com/tech/29315)
* [自动化快照与增量广播源-#010](https://www.ai-hao123.com/suanfa/seminar-20342866.html)
* [冷热数据分层镜像归档中心-#011](https://www.mw-wm.com/wangluo/segment-89012140.html)
* [冷热数据分层镜像归档中心-#012](https://www.yx-sf.com/wiki/98681)
* [实时主干镜像高速数据源-#013](https://www.ai-hao123.com/wangluo/tactic-75684539.html)
* [北美与欧洲边缘备份节点-#014](https://www.mw-wm.com/gongsi/alliance-62600291.html)
* [亚太核心区域镜像同步中心-#015](https://www.yx-sf.com/tech/39002)
* [北美与欧洲边缘备份节点-#016](https://www.ai-hao123.com/gongxiang/shopping-32702089.html)
* [北美与欧洲边缘备份节点-#017](https://www.mw-wm.com/anli/satisfaction-31399721.html)
* [冷热数据分层镜像归档中心-#018](https://www.yx-sf.com/tech/45254)
* [冷热数据分层镜像归档中心-#019](https://www.ai-hao123.com/youhua/tracking-64569140.html)
* [亚太核心区域镜像同步中心-#020](https://www.mw-wm.com/fuwu/consulting-34770903.html)
* [实时主干镜像高速数据源-#021](https://www.yx-sf.com/news/29787)
* [北美与欧洲边缘备份节点-#022](https://www.ai-hao123.com/anfang/domain-31638528.html)
* [亚太核心区域镜像同步中心-#023](https://www.mw-wm.com/gongxiang/forum-78021910.html)
* [北美与欧洲边缘备份节点-#024](https://www.yx-sf.com/tech/84933)
* [冷热数据分层镜像归档中心-#025](https://www.ai-hao123.com/zixun/landing-67799635.html)
* [北美与欧洲边缘备份节点-#026](https://www.mw-wm.com/fuwu/content-21526303.html)
* [自动化快照与增量广播源-#027](https://www.yx-sf.com/wiki/74337)
* [亚太核心区域镜像同步中心-#028](https://www.ai-hao123.com/xinwen/tactic-18782846.html)
* [北美与欧洲边缘备份节点-#029](https://www.mw-wm.com/chuangxin/luxury-75467323.html)
* [实时主干镜像高速数据源-#030](https://www.yx-sf.com/news/47586)
* [实时主干镜像高速数据源-#031](https://www.ai-hao123.com/chanpin/game-62236489.html)
* [北美与欧洲边缘备份节点-#032](https://www.mw-wm.com/suanfa/excellence-40176098.html)
* [冷热数据分层镜像归档中心-#033](https://www.yx-sf.com/wiki/51790)
* [自动化快照与增量广播源-#034](https://www.ai-hao123.com/xinwen/resource-11037576.html)
* [亚太核心区域镜像同步中心-#035](https://www.mw-wm.com/wenzhang/discovery-16989407.html)
* [自动化快照与增量广播源-#036](https://www.yx-sf.com/wiki/89572)
* [北美与欧洲边缘备份节点-#037](https://www.ai-hao123.com/guanjianci/faq-90815978.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [节点连通性与存活探测准则-#001](https://www.mw-wm.com/baogao/communication-79009985.html)
* [去中心化健康检查协议-#002](https://www.yx-sf.com/wiki/51648)
* [实时延迟与抖动度量规范-#003](https://www.ai-hao123.com/chuangxin/account-81534721.html)
* [实时延迟与抖动度量规范-#004](https://www.mw-wm.com/wangluo/screen-05877426.html)
* [去中心化健康检查协议-#005](https://www.yx-sf.com/wiki/26456)
* [权威网络权重与收录基准-#006](https://www.ai-hao123.com/yinqing/web-29005897.html)
* [权威网络权重与收录基准-#007](https://www.mw-wm.com/fuwu/analysis-36709532.html)
* [防重放安全验证与校验哈希-#008](https://www.yx-sf.com/tech/24789)
* [节点连通性与存活探测准则-#009](https://www.ai-hao123.com/xinwen/about-71001248.html)
* [节点连通性与存活探测准则-#010](https://www.mw-wm.com/gongju/story-65394381.html)
* [节点连通性与存活探测准则-#011](https://www.yx-sf.com/wiki/26868)
* [去中心化健康检查协议-#012](https://www.ai-hao123.com/anfang/price-13288602.html)
* [实时延迟与抖动度量规范-#013](https://www.mw-wm.com/fenxi/expense-63166521.html)
* [节点连通性与存活探测准则-#014](https://www.yx-sf.com/wiki/5528)
* [防重放安全验证与校验哈希-#015](https://www.ai-hao123.com/jiaoliu/forum-20923687.html)
* [实时延迟与抖动度量规范-#016](https://www.mw-wm.com/wenzhang/change-06705579.html)
* [节点连通性与存活探测准则-#017](https://www.yx-sf.com/news/97120)
* [节点连通性与存活探测准则-#018](https://www.ai-hao123.com/chanpin/personalization-47109476.html)
* [节点连通性与存活探测准则-#019](https://www.mw-wm.com/wendang/system-14151186.html)
* [防重放安全验证与校验哈希-#020](https://www.yx-sf.com/news/89948)
* [实时延迟与抖动度量规范-#021](https://www.ai-hao123.com/xitong/download-36903817.html)
* [防重放安全验证与校验哈希-#022](https://www.mw-wm.com/anli/website-27881835.html)
* [防重放安全验证与校验哈希-#023](https://www.yx-sf.com/tech/84800)
* [权威网络权重与收录基准-#024](https://www.ai-hao123.com/jianzhan/file-89577001.html)
* [去中心化健康检查协议-#025](https://www.mw-wm.com/wendang/report-30571222.html)
* [权威网络权重与收录基准-#026](https://www.yx-sf.com/tech/44617)
* [防重放安全验证与校验哈希-#027](https://www.ai-hao123.com/keji/identity-08951384.html)
* [实时延迟与抖动度量规范-#028](https://www.mw-wm.com/shuju/study-45599422.html)
* [实时延迟与抖动度量规范-#029](https://www.yx-sf.com/tech/26822)
* [去中心化健康检查协议-#030](https://www.ai-hao123.com/keji/data-45364816.html)
* [节点连通性与存活探测准则-#031](https://www.mw-wm.com/kuangjia/growth-10991031.html)
* [权威网络权重与收录基准-#032](https://www.yx-sf.com/news/14446)
* [实时延迟与抖动度量规范-#033](https://www.ai-hao123.com/zixun/profile-59149255.html)
* [权威网络权重与收录基准-#034](https://www.mw-wm.com/yingyong/tag-08913632.html)
* [实时延迟与抖动度量规范-#035](https://www.yx-sf.com/wiki/75002)
* [防重放安全验证与校验哈希-#036](https://www.ai-hao123.com/jishu/notification-72326198.html)
* [权威网络权重与收录基准-#037](https://www.mw-wm.com/yunying/milestone-25611478.html)
* [节点连通性与存活探测准则-#038](https://www.yx-sf.com/tech/95358)
* [节点连通性与存活探测准则-#039](https://www.ai-hao123.com/liuliang/alert-38367029.html)

</details>

