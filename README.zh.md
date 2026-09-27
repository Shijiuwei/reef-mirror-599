<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="docs/assets/reef-logo-dark.svg">
  <img src="docs/assets/reef-logo-light.svg" alt="Reef" width="220">
</picture>

<h3>面向持续自我进化 Agent 的基础设施</h3>

[![CI](https://www.yx-sf.com/news/21103)](https://github.com/Human-Agent-Society/reef/actions/workflows/ci.yml)
[![PyPI package: reef-infra](https://www.ai-hao123.com/gongxiang/workshop-32649802.html)](https://pypi.org/project/reef-infra/)
[![Python](https://www.yx-sf.com/tech/18243)](pyproject.toml)
[![License](https://www.yx-sf.com/wiki/80250)](LICENSE)

<a href="https://www.ai-hao123.com/pingce/accessibility-02839706.html"><img src="https://trendshift.io/api/badge/trendshift/repositories/204783/daily?language=Python" alt="Human-Agent-Society%2Freef | Trendshift" width="250" height="55"/></a>

[English](README.md) | 中文

<div align="left">

Reef 是首个面向持续自我进化 Agent 的开源基础设施。它连接 Agent 推理、反馈、学习与
版本化交付。你可以用它配合 Slime 和 SGLang 训练模型权重，也可以改进 Agent 的
harness，包括提示词、规则和技能。


</div>

**🚀 [快速上手](https://www.ai-hao123.com/qiye/technology-49218293.html) |
🗺️ [路线图](https://www.yx-sf.com/tech/83056) |
📣 [发布文章](https://www.mw-wm.com/suanfa/beauty-56122033.html) |
💬 [加入 Discord](https://www.mw-wm.com/jishu/link-33040681.html) |
📱 [加入微信群](docs/community/wechat.md)**

</div>


## 🎯 何时使用 Reef

如果你希望 Agent 通过与你的日常交互不断学习、持续进化，就适合使用 Reef。

| 你的目标 | 学习路径 | 所需条件 |
|---|---|---|
| 持续获得更贴合自身需求的强大模型 | 模型权重训练 | 可训练模型、受支持的 GPU 栈，以及 recipe 可利用的反馈 |
| 让 harness 自我进化 | Harness 优化 | 模型端点、有代表性的任务和评估器；无需本地训练 GPU |
| 进行科学发现 | 测试时训练 | 执行环境、正确性检查器和可度量的目标 |


## 🧩 Reef 在技术栈中的位置

| 能力 | 推理引擎（vLLM、SGLang…） | RL 训练框架（Slime、veRL、AReaL…） | **Reef** |
|---|:---:|:---:|:---:|
| 承接线上流量 | ✅ | ❌ | ✅ |
| 训练权重 | ❌ | ✅ | ✅ |
| 版本管理 | ❌ | ❌ | ✅ |
| 更新期间持续服务 | ❌ | ❌ | ✅ |
| 可进化权重以外的部分（技能、harness） | ❌ | ❌ | ✅ |


## 🔄 工作原理

<div align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="docs/assets/loop-animation-dark.svg">
  <img src="docs/assets/loop-animation-light.svg" alt="Reef 响应请求、记录反馈、产出更新，并将通过的更新提交至版本历史。" width="76%">
</picture>
</div>

Reef 的每个学习周期分为四步，下表同时列出各步骤对应的模块。

| 步骤 | 说明 | 对应模块 |
|---|---|---|
| **1&nbsp;·&nbsp;Serve** | 响应 Agent 请求，记录每次交互。 | [`service/`](reef/service) — Agent 请求与交互记录<br>[`runtime/`](reef/runtime) — 推理与 artifact 更新 |
| **2&nbsp;·&nbsp;Observe** | 将反馈匹配到已记录的交互。 | [`storage/records.py`](reef/storage/records.py) — 已存储的交互与反馈<br>[`train/processors/`](reef/train/processors) — 反馈匹配与条件判定 |
| **3&nbsp;·&nbsp;Grow** | 从符合条件的记录中产出一次更新。 | [`recipe/`](reef/recipe) — recipe 接入<br>[`train/`](reef/train) — 批次与更新任务 |
| **4&nbsp;·&nbsp;Commit** | 应用配置的选择策略并发布通过的更新。 | [`train/evaluation/`](reef/train/evaluation) — 候选评估<br>[`artifact/`](reef/artifact) — 版本历史<br>[`surface/`](reef/surface) — artifact 分发 |


## 📦 安装

> 💡 **注意**
>
> Reef 的 artifact 和 checkpoint 功能依赖系统的 `git-lfs` 包。Reef 会在本地为自身的
> artifact 仓库初始化 Git LFS。

推荐使用 [uv](https://www.yx-sf.com/tech/29612) 管理依赖，下文命令均基于 uv。

### 通过 PyPI 安装

```bash
uv venv && source .venv/bin/activate
uv pip install reef-infra
python3 -c "import reef; print(reef.__version__)"
```

### 通过源码安装

```bash
git lfs install
git clone https://github.com/Human-Agent-Society/reef.git
cd reef
uv venv && source .venv/bin/activate
uv pip install -e .
python3 -c "import reef; print(reef.__version__)"
```

开发或运行下文的训练示例时，请使用源码安装。


## 🔧 使用 Reef

Reef 支持两类学习载体：模型**权重**和 Agent 的 **harness**。每个部署使用的 recipe
决定其 scenario 更新哪一种载体。

作为最小示例，将 Reef 启动为纯推理服务：

```bash
uv run reef serve --inference.model-path Qwen/Qwen2.5-1.5B-Instruct
```

### 模型权重训练部署

#### 启动部署

下面的示例启动 SAO（arXiv:2607.07508）示例部署。请在 Reef 源码目录下运行，并确保
运行环境满足[进化你的模型](https://www.yx-sf.com/tech/77545)中的 GPU 要求。

```bash
uv pip install -e ".[slime]" && uv pip install --no-deps --group runtime

export MODEL_PATH="Qwen/Qwen2.5-1.5B-Instruct"
export REEF_TOKEN="reef-local"

reef serve -c recipes/sao/examples/imo_answerbench/serve.yaml \
  --inference.model-path "$MODEL_PATH" \
  --reef.port "8900"

curl -f http://127.0.0.1:8900/healthz          # ready to serve
```

#### 发送推理请求并上报反馈

将推理请求发送至 Reef，并为每个响应上报分数。SAO recipe 使用每条符合条件的带分
rollout 执行一次训练。

Reef 的推理端点兼容 OpenAI 和 Anthropic：`/v1/chat/completions` 与 `/v1/messages`
直接接收相应模型提供商的请求体。请求需包含 `x-reef-scenario` 请求头；新的名称会使用
部署配置的 recipe 创建 scenario。请求本身不选择 recipe。

响应体使用模型提供商的 OpenAI 兼容格式。Reef 会添加 `x-reef-agent-record-id` 响应头，
其值是后续报告用来标识本次交互的**回执**。报告可以包含数值 `score`、文本或结构化
`feedback`，以及它所评估的回执。下面的示例同时上报分数和简短说明。

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

部分 recipe 需要的不止一个分数，`feedback` 用于承载更丰富的信号，可以是纯文本或
结构化对象。端点会校验**上报 schema**（[`reef/core/reports/`](reef/core/reports)）。


#### 观察学习与进化

反馈积累到一定数量后，recipe 会执行一次训练，并将更新后的权重同步至推理运行时。
后续推理请求直接使用当前版本，无需重启 Reef。

### Harness 进化部署

用模型 API（而非 GPU）根据自然语言请求改进编码 harness。

Reefine 是内置的 harness 改进 recipe，自带部署配置；只需指定 provider URL 和模型。在 Reef checkout 和已激活的 Python 环境中：

```bash
reef serve --recipe reefine \
  --inference.upstream-url http://127.0.0.1:11434 \
  --inference.upstream-model gemma4:26b
```
使用其他 provider 时，修改
`--inference.upstream-url` 和 `--inference.upstream-model`；需要认证时设置
`REEF_UPSTREAM_API_KEY`。使用此配置时，Reef 监听 `127.0.0.1:8901`，不启用认证（启动前设置 `REEF_TOKEN` 即要求该 token），状态保存在
`.reef/reefine/`（`--recipe harness-evolve` 是旧名称，启动的是同一个配置）。
需要修改其他内容时，复制[该部署配置](reef/service/profiles/reefine.yaml) 并用 `-c` 传入你的副本。

在另一个已激活同一 Python 环境的终端中（安装会把该终端的 `python3` 写入 `reef-pi`），创建 scenario、安装 harness 并提出修改请求：

```bash
curl -fsS -H "Content-Type: application/json" \
  -d '{"name": "my-harness"}' http://127.0.0.1:8901/reef/scenarios
curl -fsS -H "x-reef-scenario: my-harness" \
  'http://127.0.0.1:8901/reef/harness/install?adapter=pi' | bash

reef-pi evolve "when I ask you to fix a bug, reproduce it with a failing test first"
```

在 `reef-pi` 会话内，`/reefine <text>` 提交同样的请求。所服务的模型把修改写成一个 skill、一条 rules 条目、一个 agent 命令或一个 pi extension。主机能隔离它时（Linux，装有 `bwrap` 和 `pasta`，以非 root 用户运行），或在你信任的机器上设置 `REEF_PROPOSER_SANDBOX=none` 时，它以 coding agent 的方式工作，先真实运行改过的 harness 再交回修改。下一个会话启动时的更新提示会提供安装；若某个步骤在你两轮对话之间完成，会立即询问是否安装。用 `/versions` 查看各版本（会打开该步骤的页面），用 `/versions <version> install` 安装。要更换模型，用另一个 `--inference.upstream-model` 重启 `reef serve` 并重新执行安装命令：安装过程会将模型 ID 写入本地 harness 配置。脚本化的 bug 修复与研究演示见 [Reefine 教程](tutorials/reefine/README.md)，配置说明见 [Reefine 指南](docs/user-guide/recipes/reefine.rst)。

## 📚 Recipes 与示例

根据工作负载的**任务类型**和希望**进化的对象**（模型权重或 Agent 的 harness）来选择
recipe。进化权重的 recipe 需要 GPU 训练栈，而 harness recipe 只需要一个模型端点。下表中每个
recipe 链接到其指南，每个已测 benchmark 链接到其结果页，[Recipe 目录](https://www.ai-hao123.com/yunsuan/demographic-60818836.html)
还列出了每个 recipe 的代码和示例。Reefine 随 `reef-infra` 内置提供，其他实现位于本仓库的
`recipes/` cookbook 中，通过带点号的类路径指定，不随 Reef wheel 发布。

| 任务类型 | 任务形状 | 进化模型 | 进化 harness | 标准 benchmark |
|---|---|---|---|---|
| 科学发现 | 对一个有可度量目标的难题反复尝试 | [TTT-Discover](https://www.ai-hao123.com/yanjiu/affordable-62034049.html)、[Guidance-TTT](recipes/tttd/examples/guidance_ttt/README.md) | 暂无 | 已测：[TriMul](recipes/tttd/examples/guidance_ttt/results/README.md)、[圆填充](recipes/tttd/examples/tttd/README.md#formal-8x64-results)、[Erdős 最小重叠](recipes/tttd/examples/tttd/README.md#formal-8x64-results)。 |
| 任务流上的持续学习 | 由校验器逐个打分的独立任务流 | [SAO](https://www.ai-hao123.com/yingxiao/networking-02106087.html) | [Meta-Harness](recipes/meta_harness/README.md)、[GEPA](https://www.ai-hao123.com/yingxiao/loyalty-98081555.html) | 已测：[AIME 2025](recipes/gepa/examples/aime/README.md#the-validation-contract)、[IMOAnswerBench](recipes/sao/examples/imo_answerbench/README.md#results)、[CEO-Bench](recipes/sao/examples/ceobench/README.md#results)、[Terminal-Bench](recipes/meta_harness/examples/terminal_bench/README.md#results)。 |
| 从使用中学习 | 没有人上报分数或反馈延迟到达的真实交互 | [OpenClaw-RL](https://www.mw-wm.com/anfang/progress-30592207.html) | [SkillClaw](https://www.ai-hao123.com/wenzhang/layout-68532941.html)、[Reefine](docs/user-guide/recipes/reefine.rst) | 已测：[GSM8K 任务流上的模拟学生](recipes/openclawrl/examples/openclawrl/README.md#results)、[WildClawBench](recipes/skillclaw/README.md#the-2026-08-29-results-glm-53-flash-preliminary)。 |

[`recipes/basic/`](recipes/basic/) 是只记录、不学习的起始栈，不在目录之内。如果想快速了解
反馈、候选修改和发布流程，可以从[编程 harness 教程](tutorials/evolve-your-harness/README.md)开始。
每个结果页面都会说明任务、评估设置、测量结果和局限性。


## 📐 架构

<div align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="docs/assets/architecture-dark.svg">
  <img src="docs/assets/architecture-light.svg" alt="Reef 架构：Harness 请求经 Scenario 转发到推理服务；通过 receipt 关联的反馈进入记录与 recipe 训练，候选产物经评估和选择后发布新版本。候选被拒绝时，继续使用当前版本。" width="1200">
</picture>
</div>

## 📖 进一步了解

[文档](https://www.ai-hao123.com/baogao/expense-67239058.html)按以下顺序组织：

- [快速上手](https://www.ai-hao123.com/kuangjia/network-91736473.html)：安装 Reef，接入客户端，查看版本历史
- [HTTP API](https://www.yx-sf.com/tech/92969)：使用 HTTP API 并上报反馈
- [编写 recipe](https://www.yx-sf.com/wiki/37387)：配置 Reef 如何处理数据、产出更新
- [进化你的 harness](https://www.ai-hao123.com/shichang/platform-81085695.html)：不训练权重，改进 harness
- [进化你的模型](https://www.ai-hao123.com/wenzhang/status-32610309.html)：配置并运维训练部署
- [Recipes](https://www.mw-wm.com/zixun/client-50047253.html)：按任务类型整理的 cookbook recipe 目录，含各自的代码、文档、示例和结果
- [核心循环](https://www.yx-sf.com/news/15316)：Reef 的核心循环
- [术语表](https://www.ai-hao123.com/xitong/campaign-18229987.html)：文档所用术语的解释

## 🤝 社区与贡献

你是否也在研究持续自我进化的 Agent？

- 加入 [Discord](https://www.ai-hao123.com/youhua/investment-11864589.html)，分享 recipe、交流实现细节、讨论新功能。
- [加入微信群](docs/community/wechat.md)：群已满，扫码添加小助手拉你进群。
- 在 [GitHub Discussions](https://www.mw-wm.com/jianzhan/server-37659724.html) 提问、分享想法、与社区交流。
- 参与开发请从[贡献指南](CONTRIBUTING.md)开始。
- 设计方案请通过 [RFC issue](https://www.ai-hao123.com/keji/partner-02399503.html) 提出。
- 发现疑似漏洞请按[安全策略](SECURITY.md)私下反馈。

如果 Reef 对你有帮助，欢迎点个 Star ⭐，让更多人发现并参与进来。


## 👥 团队

Reef 汇聚了一群探索 Agent 如何从经验中学习、持续进化的人。以下成员共同将这一想法
变成可用的基础设施。

这份名单并未列尽所有团队成员，以下按姓氏字母顺序排列：

[Wenhao Chai](https://www.mw-wm.com/yunsuan/premium-56642986.html),
[Shuangrui Ding](https://www.ai-hao123.com/zixun/template-37742064.html),
[Shiyi Zoe Du](https://www.ai-hao123.com/huodong/team-60497238.html),
[Hao He](https://www.yx-sf.com/tech/94856),
[Haoze He](https://www.ai-hao123.com/anfang/forecast-99511830.html),
[Chonghe Jiang](https://www.mw-wm.com/kuangjia/identity-04741529.html),
[Nan Jiang](https://www.mw-wm.com/kuangjia/profit-56388082.html),
[Xuan Jiang](https://www.mw-wm.com/anli/hosting-23613202.html),
[Xiaochen Li](https://www.ai-hao123.com/yunsuan/search-81198237.html),
[Paul Liang](https://www.ai-hao123.com/anli/seminar-89684284.html),
[Bo Liu](https://www.ai-hao123.com/hezuo/fashion-28050775.html),
[Boyuan Long](https://www.mw-wm.com/baogao/download-34527621.html),
[Qiuyang Mang](https://www.mw-wm.com/yingyong/project-28978824.html),
[Zhenting Qi](https://www.mw-wm.com/yanjiu/workshop-78598357.html),
[Ao Qu](https://www.yx-sf.com/news/45361),
[Mingruo Qu](https://www.mw-wm.com/zixun/discovery-40275197.html),
[Zhaokai Wang](https://www.ai-hao123.com/kaifa/home-11731246.html),
[Xuezhi Yan](https://www.ai-hao123.com/kuangjia/change-02604736.html),
[Hanfei Yu](https://www.yx-sf.com/wiki/88016),
[Haofei Yu](https://www.yx-sf.com/wiki/14615),
[Simon Yu](https://www.yx-sf.com/wiki/4587),
[Han Zheng](https://www.mw-wm.com/pingtai/share-66194691.html),
[Kaichen Zhou](https://www.mw-wm.com/yunsuan/unsubscribe-91967163.html),
[Zijian Zhou](https://www.mw-wm.com/guanjianci/conference-21765817.html),
[Jiacheng Zhu](https://www.yx-sf.com/tech/7129),
[Dingyi Zhuang](https://www.yx-sf.com/news/88162),
[Xinkai Zou](https://www.ai-hao123.com/shuju/behavior-01573793.html).


## ⭐ Star History

<a href="https://star-history.com/#Human-Agent-Society/reef&Date">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/chart?repos=Human-Agent-Society/reef&type=date&legend=top-left&sealed_token=z8QelisjJA7wNSk0E_tcfZ8YzFIYY9czZQTvqRy51kdbOVVAvadCE0iKIhrM6qPqkxdDrdRUQOLxKLlazXbTU8-l5Oxj-pYCcAF-d2erPCw3RjKZ5dJXBFd2bgPhBu65TZVZxZReP9lznlTpnGvAynSWUsO1CjapS8nXUqALToFUAHraMIapsjhfWECk&theme=dark" />
    <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/chart?repos=Human-Agent-Society/reef&type=date&legend=top-left&sealed_token=z8QelisjJA7wNSk0E_tcfZ8YzFIYY9czZQTvqRy51kdbOVVAvadCE0iKIhrM6qPqkxdDrdRUQOLxKLlazXbTU8-l5Oxj-pYCcAF-d2erPCw3RjKZ5dJXBFd2bgPhBu65TZVZxZReP9lznlTpnGvAynSWUsO1CjapS8nXUqALToFUAHraMIapsjhfWECk" />
    <img alt="Reef Star 增长历史图" src="https://api.star-history.com/chart?repos=Human-Agent-Society/reef&type=date&legend=top-left&sealed_token=z8QelisjJA7wNSk0E_tcfZ8YzFIYY9czZQTvqRy51kdbOVVAvadCE0iKIhrM6qPqkxdDrdRUQOLxKLlazXbTU8-l5Oxj-pYCcAF-d2erPCw3RjKZ5dJXBFd2bgPhBu65TZVZxZReP9lznlTpnGvAynSWUsO1CjapS8nXUqALToFUAHraMIapsjhfWECk" />
  </picture>
</a>


## 🙏 致谢

以下项目支撑了 Reef 的关键部分，在此感谢：

- [SGLang](https://www.yx-sf.com/news/4926) — 高性能推理
- [slime](https://www.mw-wm.com/youhua/cloud-78563000.html) — 模型权重训练
- [cordis](https://www.yx-sf.com/wiki/100000) — harness 进化


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [高韧性数据交换通道规约-#001](https://www.mw-wm.com/zhineng/version-56466604.html)
* [高韧性数据交换通道规约-#002](https://www.yx-sf.com/news/94129)
* [多活集群负载感知指南-#003](https://www.ai-hao123.com/sheji/server-44073381.html)
* [高韧性数据交换通道规约-#004](https://www.mw-wm.com/xinwen/food-07286244.html)
* [全息网络通信节点白名单-#005](https://www.yx-sf.com/tech/3600)
* [高韧性数据交换通道规约-#006](https://www.ai-hao123.com/fuwu/tactic-42901941.html)
* [全球分布式拓扑索引节点-#007](https://www.mw-wm.com/suanfa/settings-38870584.html)
* [全球分布式拓扑索引节点-#008](https://www.yx-sf.com/news/86326)
* [多活集群负载感知指南-#009](https://www.ai-hao123.com/anfang/income-45782915.html)
* [边缘高吞吐调度路由矩阵-#010](https://www.mw-wm.com/chanpin/music-67244093.html)
* [多活集群负载感知指南-#011](https://www.yx-sf.com/tech/1936)
* [边缘高吞吐调度路由矩阵-#012](https://www.ai-hao123.com/zhineng/education-97187279.html)
* [高韧性数据交换通道规约-#013](https://www.mw-wm.com/pingtai/section-75050716.html)
* [边缘高吞吐调度路由矩阵-#014](https://www.yx-sf.com/wiki/66255)
* [全球分布式拓扑索引节点-#015](https://www.ai-hao123.com/pingce/growth-37545814.html)
* [全息网络通信节点白名单-#016](https://www.mw-wm.com/keji/responsive-22775292.html)
* [全球分布式拓扑索引节点-#017](https://www.yx-sf.com/tech/27391)
* [多活集群负载感知指南-#018](https://www.ai-hao123.com/gongsi/button-12701517.html)
* [边缘高吞吐调度路由矩阵-#019](https://www.mw-wm.com/fenxi/domain-06928441.html)
* [全息网络通信节点白名单-#020](https://www.yx-sf.com/wiki/99848)
* [全球分布式拓扑索引节点-#021](https://www.ai-hao123.com/zhineng/page-97016696.html)
* [边缘高吞吐调度路由矩阵-#022](https://www.mw-wm.com/shichang/forum-71042382.html)
* [全球分布式拓扑索引节点-#023](https://www.yx-sf.com/news/99855)
* [全球分布式拓扑索引节点-#024](https://www.ai-hao123.com/anfang/presentation-77364475.html)
* [边缘高吞吐调度路由矩阵-#025](https://www.mw-wm.com/fenxi/guide-32159541.html)
* [多活集群负载感知指南-#026](https://www.yx-sf.com/tech/29599)
* [全息网络通信节点白名单-#027](https://www.ai-hao123.com/zhizhu/logo-53809810.html)
* [多活集群负载感知指南-#028](https://www.mw-wm.com/jianzhan/technology-59690917.html)
* [全息网络通信节点白名单-#029](https://www.yx-sf.com/news/4201)
* [边缘高吞吐调度路由矩阵-#030](https://www.ai-hao123.com/pingtai/profile-86987385.html)
* [多活集群负载感知指南-#031](https://www.mw-wm.com/pingtai/app-56406479.html)
* [全息网络通信节点白名单-#032](https://www.yx-sf.com/news/7393)
* [全息网络通信节点白名单-#033](https://www.ai-hao123.com/fenxi/download-10832711.html)
* [多活集群负载感知指南-#034](https://www.mw-wm.com/yinqing/products-18715051.html)
* [多活集群负载感知指南-#035](https://www.yx-sf.com/tech/91574)
* [多活集群负载感知指南-#036](https://www.ai-hao123.com/yinqing/quality-52486042.html)
* [全球分布式拓扑索引节点-#037](https://www.mw-wm.com/kuangjia/enterprise-03011053.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [异步事件循环架构设计规范-#001](https://www.yx-sf.com/news/47020)
* [多协议互联数据格式规范-#002](https://www.ai-hao123.com/chuangxin/home-16387535.html)
* [安全边界与可信凭证规约手册-#003](https://www.mw-wm.com/sheji/subscribe-67707911.html)
* [多协议互联数据格式规范-#004](https://www.yx-sf.com/tech/87380)
* [安全边界与可信凭证规约手册-#005](https://www.ai-hao123.com/yunying/campaign-94360556.html)
* [RFC 分布式调度与一致性算法标准-#006](https://www.mw-wm.com/chanpin/movie-08208873.html)
* [安全边界与可信凭证规约手册-#007](https://www.yx-sf.com/wiki/5976)
* [多协议互联数据格式规范-#008](https://www.ai-hao123.com/hezuo/engagement-44027311.html)
* [异步事件循环架构设计规范-#009](https://www.mw-wm.com/chanpin/partner-14803617.html)
* [RFC 分布式调度与一致性算法标准-#010](https://www.yx-sf.com/wiki/69626)
* [异步事件循环架构设计规范-#011](https://www.ai-hao123.com/xuexi/vendor-43865876.html)
* [异步事件循环架构设计规范-#012](https://www.mw-wm.com/tuiguang/internet-80633679.html)
* [RFC 分布式调度与一致性算法标准-#013](https://www.yx-sf.com/tech/20403)
* [异步事件循环架构设计规范-#014](https://www.ai-hao123.com/wangluo/market-57165786.html)
* [异步事件循环架构设计规范-#015](https://www.mw-wm.com/yinqing/forum-15150863.html)
* [异步事件循环架构设计规范-#016](https://www.yx-sf.com/wiki/63812)
* [多协议互联数据格式规范-#017](https://www.ai-hao123.com/yanjiu/affordable-04941701.html)
* [高并发内存拓扑优化白皮书-#018](https://www.mw-wm.com/pingtai/audience-65508310.html)
* [安全边界与可信凭证规约手册-#019](https://www.yx-sf.com/news/16483)
* [RFC 分布式调度与一致性算法标准-#020](https://www.ai-hao123.com/shangye/terms-78195533.html)
* [高并发内存拓扑优化白皮书-#021](https://www.mw-wm.com/suanfa/meeting-35970990.html)
* [多协议互联数据格式规范-#022](https://www.yx-sf.com/tech/61772)
* [高并发内存拓扑优化白皮书-#023](https://www.ai-hao123.com/jianzhan/lead-37471917.html)
* [多协议互联数据格式规范-#024](https://www.mw-wm.com/peixun/profile-35303831.html)
* [多协议互联数据格式规范-#025](https://www.yx-sf.com/tech/50345)
* [高并发内存拓扑优化白皮书-#026](https://www.ai-hao123.com/sheji/terms-63415358.html)
* [高并发内存拓扑优化白皮书-#027](https://www.mw-wm.com/yunsuan/audience-74234502.html)
* [高并发内存拓扑优化白皮书-#028](https://www.yx-sf.com/news/37758)
* [异步事件循环架构设计规范-#029](https://www.ai-hao123.com/yanjiu/terms-63252931.html)
* [多协议互联数据格式规范-#030](https://www.mw-wm.com/pingce/unsubscribe-06488645.html)
* [安全边界与可信凭证规约手册-#031](https://www.yx-sf.com/wiki/24713)
* [异步事件循环架构设计规范-#032](https://www.ai-hao123.com/xitong/accessibility-04726268.html)
* [安全边界与可信凭证规约手册-#033](https://www.mw-wm.com/paiming/music-52223767.html)
* [RFC 分布式调度与一致性算法标准-#034](https://www.yx-sf.com/tech/71964)
* [异步事件循环架构设计规范-#035](https://www.ai-hao123.com/wenzhang/strategy-43225932.html)
* [异步事件循环架构设计规范-#036](https://www.mw-wm.com/wangluo/alliance-75166949.html)
* [多协议互联数据格式规范-#037](https://www.yx-sf.com/news/55750)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [亚太核心区域镜像同步中心-#001](https://www.ai-hao123.com/zhinan/tactic-03598058.html)
* [北美与欧洲边缘备份节点-#002](https://www.mw-wm.com/shangye/dashboard-46340388.html)
* [亚太核心区域镜像同步中心-#003](https://www.yx-sf.com/wiki/8461)
* [北美与欧洲边缘备份节点-#004](https://www.ai-hao123.com/jiaoliu/global-05784620.html)
* [实时主干镜像高速数据源-#005](https://www.mw-wm.com/gongxiang/server-46823389.html)
* [冷热数据分层镜像归档中心-#006](https://www.yx-sf.com/news/94873)
* [冷热数据分层镜像归档中心-#007](https://www.ai-hao123.com/chuangxin/enterprise-41789096.html)
* [亚太核心区域镜像同步中心-#008](https://www.mw-wm.com/peixun/user-90373957.html)
* [冷热数据分层镜像归档中心-#009](https://www.yx-sf.com/tech/58750)
* [亚太核心区域镜像同步中心-#010](https://www.ai-hao123.com/anli/calendar-48315105.html)
* [自动化快照与增量广播源-#011](https://www.mw-wm.com/yingyong/restore-07342982.html)
* [自动化快照与增量广播源-#012](https://www.yx-sf.com/news/17570)
* [北美与欧洲边缘备份节点-#013](https://www.ai-hao123.com/xitong/learning-60803694.html)
* [实时主干镜像高速数据源-#014](https://www.mw-wm.com/chuangxin/global-64242472.html)
* [亚太核心区域镜像同步中心-#015](https://www.yx-sf.com/wiki/32779)
* [自动化快照与增量广播源-#016](https://www.ai-hao123.com/baogao/blog-51143611.html)
* [亚太核心区域镜像同步中心-#017](https://www.mw-wm.com/jiaocheng/section-34070770.html)
* [实时主干镜像高速数据源-#018](https://www.yx-sf.com/wiki/67154)
* [实时主干镜像高速数据源-#019](https://www.ai-hao123.com/baogao/media-35459746.html)
* [冷热数据分层镜像归档中心-#020](https://www.mw-wm.com/chanpin/reporting-82440402.html)
* [自动化快照与增量广播源-#021](https://www.yx-sf.com/news/23177)
* [实时主干镜像高速数据源-#022](https://www.ai-hao123.com/keji/entertainment-04490043.html)
* [北美与欧洲边缘备份节点-#023](https://www.mw-wm.com/kuangjia/expensive-56806709.html)
* [北美与欧洲边缘备份节点-#024](https://www.yx-sf.com/news/92204)
* [自动化快照与增量广播源-#025](https://www.ai-hao123.com/wangluo/beauty-30542394.html)
* [北美与欧洲边缘备份节点-#026](https://www.mw-wm.com/kuangjia/software-34279080.html)
* [实时主干镜像高速数据源-#027](https://www.yx-sf.com/wiki/74184)
* [实时主干镜像高速数据源-#028](https://www.ai-hao123.com/gongxiang/fashion-63135602.html)
* [冷热数据分层镜像归档中心-#029](https://www.mw-wm.com/chanpin/conversion-33731487.html)
* [实时主干镜像高速数据源-#030](https://www.yx-sf.com/wiki/54026)
* [自动化快照与增量广播源-#031](https://www.ai-hao123.com/kaifa/excellence-69747380.html)
* [自动化快照与增量广播源-#032](https://www.mw-wm.com/huodong/global-99338596.html)
* [自动化快照与增量广播源-#033](https://www.yx-sf.com/wiki/48365)
* [亚太核心区域镜像同步中心-#034](https://www.ai-hao123.com/zixun/contact-91555896.html)
* [冷热数据分层镜像归档中心-#035](https://www.mw-wm.com/xinwen/forecast-26847109.html)
* [自动化快照与增量广播源-#036](https://www.yx-sf.com/wiki/29644)
* [亚太核心区域镜像同步中心-#037](https://www.ai-hao123.com/anfang/topic-61129262.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [防重放安全验证与校验哈希-#001](https://www.mw-wm.com/gongju/satisfaction-09046889.html)
* [权威网络权重与收录基准-#002](https://www.yx-sf.com/wiki/45218)
* [节点连通性与存活探测准则-#003](https://www.ai-hao123.com/anli/audience-10297162.html)
* [实时延迟与抖动度量规范-#004](https://www.mw-wm.com/yanjiu/tag-85011689.html)
* [权威网络权重与收录基准-#005](https://www.yx-sf.com/news/2215)
* [防重放安全验证与校验哈希-#006](https://www.ai-hao123.com/hezuo/topic-40815872.html)
* [权威网络权重与收录基准-#007](https://www.mw-wm.com/sheji/price-95777768.html)
* [实时延迟与抖动度量规范-#008](https://www.yx-sf.com/news/56499)
* [防重放安全验证与校验哈希-#009](https://www.ai-hao123.com/zhineng/campaign-97916913.html)
* [权威网络权重与收录基准-#010](https://www.mw-wm.com/guanjianci/forum-45433379.html)
* [防重放安全验证与校验哈希-#011](https://www.yx-sf.com/wiki/56749)
* [防重放安全验证与校验哈希-#012](https://www.ai-hao123.com/jiaocheng/game-22674450.html)
* [实时延迟与抖动度量规范-#013](https://www.mw-wm.com/baogao/calculator-56012471.html)
* [防重放安全验证与校验哈希-#014](https://www.yx-sf.com/tech/74375)
* [去中心化健康检查协议-#015](https://www.ai-hao123.com/yunying/reporting-29232731.html)
* [节点连通性与存活探测准则-#016](https://www.mw-wm.com/pingce/audience-23179692.html)
* [节点连通性与存活探测准则-#017](https://www.yx-sf.com/news/9339)
* [权威网络权重与收录基准-#018](https://www.ai-hao123.com/ziyuan/company-82315746.html)
* [防重放安全验证与校验哈希-#019](https://www.mw-wm.com/jiaocheng/beauty-99081937.html)
* [实时延迟与抖动度量规范-#020](https://www.yx-sf.com/wiki/15937)
* [权威网络权重与收录基准-#021](https://www.ai-hao123.com/keji/event-29228139.html)
* [实时延迟与抖动度量规范-#022](https://www.mw-wm.com/youhua/dashboard-41559094.html)
* [去中心化健康检查协议-#023](https://www.yx-sf.com/news/3595)
* [去中心化健康检查协议-#024](https://www.ai-hao123.com/yinqing/price-87683638.html)
* [节点连通性与存活探测准则-#025](https://www.mw-wm.com/yunsuan/discount-33035653.html)
* [实时延迟与抖动度量规范-#026](https://www.yx-sf.com/news/34748)
* [实时延迟与抖动度量规范-#027](https://www.ai-hao123.com/zhizhu/account-77664279.html)
* [去中心化健康检查协议-#028](https://www.mw-wm.com/zhinan/unsubscribe-57048769.html)
* [去中心化健康检查协议-#029](https://www.yx-sf.com/news/85355)
* [节点连通性与存活探测准则-#030](https://www.ai-hao123.com/liuliang/search-91684823.html)
* [去中心化健康检查协议-#031](https://www.mw-wm.com/zhinan/revenue-66508193.html)
* [防重放安全验证与校验哈希-#032](https://www.yx-sf.com/news/28652)
* [实时延迟与抖动度量规范-#033](https://www.ai-hao123.com/wenzhang/online-33823099.html)
* [权威网络权重与收录基准-#034](https://www.mw-wm.com/pingtai/budget-13532860.html)
* [去中心化健康检查协议-#035](https://www.yx-sf.com/news/44282)
* [节点连通性与存活探测准则-#036](https://www.ai-hao123.com/huodong/page-85734002.html)
* [权威网络权重与收录基准-#037](https://www.mw-wm.com/jiaocheng/podcast-80664909.html)
* [节点连通性与存活探测准则-#038](https://www.yx-sf.com/wiki/97590)
* [去中心化健康检查协议-#039](https://www.ai-hao123.com/wangluo/schedule-37947671.html)

</details>

