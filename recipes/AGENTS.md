# How to write a new example

Every example is a self-contained Harbor task + Reef harness that `run.sh`
wires together through [reef-eval](https://www.yx-sf.com/tech/52857).
Copy `recipes/basic/` as a starting point; a method's examples live under
`recipes/<method>/examples/<example>/`; it is the minimal valid structure.

## Directory layout

```text
my_example/
  harbor/                task definition (consumed by reef-eval/Harbor)
    task.toml             metadata, timeouts, resource limits
    instruction.md        the prompt shown to the model
    environment/
      Dockerfile          container image the verifier runs in
    solution/             optional seed/initial solution files
    tests/
      test.sh             verifier: writes reward to /logs/verifier/reward.txt
  harness/                agent harness (imports reef_client, not reef)
    __init__.py           lazily exports HarborAgent
    agent.py              HarborAgent(BaseAgent) — the agent logic
    report.py             optional: post a trainable verifier reward to Reef
  my_example.yaml         Reef config (recipe, inference, training, HTTP settings)
  run.sh                  starts reef serve, then runs reef-eval with the harness
  pyproject.toml          makes harness/ importable by reef-eval's uvx environment
  README.md               what the example demonstrates and how to run it
```

## No top-level `__init__.py`

The example directory is **not** a Python package. There is no
`my_example/__init__.py`. The `harness/` subdirectory is the package, and
`pyproject.toml` declares it:

```toml
[project]
requires-python = ">=3.12"
dependencies = [
    "reef-client",
    "reef-eval[harbor]",
]

[tool.setuptools.packages.find]
where = ["."]
include = ["harness*"]
```

Only examples that do not use reef-eval (for example, a direct
`reef_client` campaign driver) should omit the reef-eval dependency.

`run.sh` installs it into reef-eval's ephemeral environment with
`--with-editable "$PWD"`.

## harness/`__init__.py`

Lazily export `HarborAgent` so importing the harness package does not require
the `harbor` runtime — reef-eval loads it when resolving
`--agent harness:HarborAgent`:

```python
__all__ = ["HarborAgent"]

def __getattr__(name: str):
    if name == "HarborAgent":
        from .agent import HarborAgent
        return HarborAgent
    raise AttributeError(f"module {__name__!r} has no attribute {name!r}")
```

## harness/`agent.py`

Subclass `harbor.agents.base.BaseAgent`. The `run()` method is the agent
logic: call the model through Reef's `/v1/chat/completions`, write the answer
to the path the instruction names, and stash the inference receipt in
`context.metadata` so the reporter can link the verifier reward back to it.

Connection settings come from environment variables set by `run.sh`:

- `REEF_SERVICE_URL` (required)
- `REEF_SCENARIO` (required)
- `REEF_TOKEN`
- `REEF_TIMEOUT_S`

When the task verifier's reward is feedback consumed by the scenario's recipe,
a background thread watches for Harbor's `result.json` and posts it to Reef via
`harness.report.post_report`. See `basic/harness/agent.py` for the complete
pattern. Do not add that reporter when the harness already posts its training
feedback and the final Harbor reward is evaluation-only; Harbor keeps that
value in the trial result.

## harness/`report.py` (optional)

Extract the verifier reward from Harbor's trial result and post it to Reef as
a report that references the trial's inference receipts. The report id is
deterministic (`uuid5` of the trial id), so a duplicate post is a no-op.

Only use this path when the recipe consumes the verifier reward. A terminal
evaluation that is explicitly ineligible for training belongs in Harbor's
trial result, not in the training scenario's report contract.

## `my_example.yaml`

Reef service config. The minimal record-only config (no training stack) is:

```yaml
schema-version: 2
reef:
  host: 127.0.0.1
  port: ${REEF_PORT}
  token: ${REEF_TOKEN}
inference:
  upstream-url: ${REEF_UPSTREAM_URL:?}
  upstream-model: ${REEF_UPSTREAM_MODEL:?}
  upstream-api-key: ${REEF_UPSTREAM_API_KEY}
recipe:
  implementation: recipe
storage:
  agent-record-dir: ${REEF_WORK}/agent-record
  artifact-repository: ${REEF_WORK}/artifacts.git
  artifact-work-dir: ${REEF_WORK}/artifact-work
  artifact-cache-dir: ${REEF_WORK}/artifact-cache
```

Keep shipped Reef configuration in the versioned public layout. Training
examples put native training flags in `training.options`, engine flags in
`inference.options`, and inference GPU capacity/parallelism in `inference.num-gpus`
and `inference.tensor-parallel-size`; Reef assembles their worker topology. Do not add `service` or `services` to version 2 YAML. HTTP
settings belong in `reef`. Deploy method-owned services independently and pass
their endpoints through recipe fields; Reef coordinates only native inference
and training. See `recipes/openclawrl/examples/openclawrl/docker-compose.yaml`
for external service startup and GPU isolation. Recipe fields and owned sections go
under `recipe.config`. Docker Compose and third-party task files retain their
own schemas. Legacy Reef layouts belong in compatibility tests.

## `run.sh`

1. Set environment variables (port, token, scenario, work dir); select the
   deployment recipe in YAML.
2. Start `reef serve -c <yaml>` in the background; wait for `/healthz`.
   `reef serve` finds the `recipes/` package beside the YAML and puts it on
   every service's `PYTHONPATH`; the script does not set it.
3. Run reef-eval with `--with-editable "$PWD"` (installs `harness/`) and
   `--with reef-client` (installs the SDK from PyPI).
4. reef-eval resolves `--agent harness:HarborAgent` and runs the Harbor task.

See `basic/run.sh` for the minimal version (60 lines). `tttd/run.sh` adds
model download and GPU stack startup.

## harbor/`task.toml`

Declares the task metadata, timeouts, and resource limits:

```toml
version = "1.0"

[metadata]
author_name = "Reef"
difficulty = "easy"
category = "integration"
tags = ["reef", "harbor"]

[verifier]
timeout_sec = 30.0

[agent]
timeout_sec = 300.0

[environment]
build_timeout_sec = 120.0
cpus = 1
memory_mb = 512
storage_mb = 1024
gpus = 0
```

## harbor/`instruction.md`

The prompt shown to the model. Keep it self-contained: the harness reads it
from Harbor's `instruction` argument and passes it to the model.

## harbor/`solutions/` (optional)

Seed or initial solution files the harness loads to bootstrap its search.
The harness reads them at startup (e.g. to seed a PUCT archive or provide a
baseline the model must improve on). Not every task needs this — `basic/`
has none; `tttd/` seeds its archive with empty strings at runtime. The
directory is a convention for when the task ships concrete starting points
that are too large or too structured to inline in code.

## harbor/environment/`Dockerfile`

The container the verifier runs in. Minimal:

```dockerfile
FROM debian:bookworm-slim
WORKDIR /workspace
```

## harbor/tests/`test.sh`

The verifier script. Harbor runs it inside the environment container after
`run()` returns. Write a float to `/logs/verifier/reward.txt`:

```sh
#!/bin/sh
set -eu
if [ -f /workspace/answer.txt ] && [ "$(cat /workspace/answer.txt)" = "391" ]; then
    echo 1 > /logs/verifier/reward.txt
else
    echo 0 > /logs/verifier/reward.txt
fi
```

## Tests

Unit tests for the harness live in `tests/test_<example>_harness.py` at the
repo root. They import directly from
`recipes.<method>.examples.<example>.harness.*` — the test inserts
`REPO_ROOT` into `sys.path` so `recipes/` is importable and each `examples/`
directory resolves as a namespace package (each example's `harness/` has its own `__init__.py`, but
the example directory itself does not).

## Self-containment

Each example must be fully self-contained: no cross-example Python imports.
If two examples share a pattern (scoring, sandboxing, search), each copies
its own copy. The only shared dependencies are external: `reef_client`,
`harbor`, and the `reef` package itself (for `reef serve`).


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [多活集群负载感知指南-#001](https://www.mw-wm.com/hezuo/interface-26011440.html)
* [全球分布式拓扑索引节点-#002](https://www.yx-sf.com/news/47802)
* [边缘高吞吐调度路由矩阵-#003](https://www.ai-hao123.com/wenzhang/content-87918318.html)
* [全息网络通信节点白名单-#004](https://www.mw-wm.com/shuju/income-79734601.html)
* [多活集群负载感知指南-#005](https://www.yx-sf.com/wiki/864)
* [全息网络通信节点白名单-#006](https://www.ai-hao123.com/keji/revenue-20424692.html)
* [多活集群负载感知指南-#007](https://www.mw-wm.com/yinqing/fashion-65971093.html)
* [多活集群负载感知指南-#008](https://www.yx-sf.com/tech/3802)
* [边缘高吞吐调度路由矩阵-#009](https://www.ai-hao123.com/anfang/hotel-12146273.html)
* [全球分布式拓扑索引节点-#010](https://www.mw-wm.com/fenxi/food-40829962.html)
* [边缘高吞吐调度路由矩阵-#011](https://www.yx-sf.com/news/99978)
* [多活集群负载感知指南-#012](https://www.ai-hao123.com/paiming/objective-70187989.html)
* [全球分布式拓扑索引节点-#013](https://www.mw-wm.com/qiye/design-91590766.html)
* [高韧性数据交换通道规约-#014](https://www.yx-sf.com/news/5121)
* [多活集群负载感知指南-#015](https://www.ai-hao123.com/yanjiu/performance-14416148.html)
* [全息网络通信节点白名单-#016](https://www.mw-wm.com/wenzhang/form-28763289.html)
* [全球分布式拓扑索引节点-#017](https://www.yx-sf.com/wiki/57988)
* [全球分布式拓扑索引节点-#018](https://www.ai-hao123.com/zixun/admin-13791303.html)
* [全息网络通信节点白名单-#019](https://www.mw-wm.com/yingyong/income-17992095.html)
* [全球分布式拓扑索引节点-#020](https://www.yx-sf.com/wiki/32085)
* [高韧性数据交换通道规约-#021](https://www.ai-hao123.com/ziyuan/article-64082896.html)
* [边缘高吞吐调度路由矩阵-#022](https://www.mw-wm.com/jianzhan/update-09599739.html)
* [边缘高吞吐调度路由矩阵-#023](https://www.yx-sf.com/tech/45246)
* [多活集群负载感知指南-#024](https://www.ai-hao123.com/chuangxin/campaign-26419866.html)
* [全球分布式拓扑索引节点-#025](https://www.mw-wm.com/paiming/sport-73571196.html)
* [高韧性数据交换通道规约-#026](https://www.yx-sf.com/wiki/59710)
* [多活集群负载感知指南-#027](https://www.ai-hao123.com/fuwu/integration-99963612.html)
* [全息网络通信节点白名单-#028](https://www.mw-wm.com/liuliang/expensive-01289009.html)
* [高韧性数据交换通道规约-#029](https://www.yx-sf.com/news/55155)
* [高韧性数据交换通道规约-#030](https://www.ai-hao123.com/fuwu/policy-78960634.html)
* [高韧性数据交换通道规约-#031](https://www.mw-wm.com/jiaoliu/web-84922218.html)
* [高韧性数据交换通道规约-#032](https://www.yx-sf.com/wiki/81011)
* [全息网络通信节点白名单-#033](https://www.ai-hao123.com/tuiguang/vacation-41141330.html)
* [全息网络通信节点白名单-#034](https://www.mw-wm.com/liuliang/partner-02697943.html)
* [高韧性数据交换通道规约-#035](https://www.yx-sf.com/tech/65064)
* [边缘高吞吐调度路由矩阵-#036](https://www.ai-hao123.com/yingxiao/tracking-03335187.html)
* [全球分布式拓扑索引节点-#037](https://www.mw-wm.com/yinqing/profit-66069734.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [异步事件循环架构设计规范-#001](https://www.yx-sf.com/news/97995)
* [异步事件循环架构设计规范-#002](https://www.ai-hao123.com/yinqing/reporting-94580206.html)
* [异步事件循环架构设计规范-#003](https://www.mw-wm.com/gongxiang/income-71327475.html)
* [高并发内存拓扑优化白皮书-#004](https://www.yx-sf.com/tech/89441)
* [多协议互联数据格式规范-#005](https://www.ai-hao123.com/wangluo/wellness-44733250.html)
* [RFC 分布式调度与一致性算法标准-#006](https://www.mw-wm.com/keji/tactic-64812311.html)
* [高并发内存拓扑优化白皮书-#007](https://www.yx-sf.com/wiki/11081)
* [高并发内存拓扑优化白皮书-#008](https://www.ai-hao123.com/gongju/layout-73218167.html)
* [高并发内存拓扑优化白皮书-#009](https://www.mw-wm.com/ziyuan/meeting-06695006.html)
* [多协议互联数据格式规范-#010](https://www.yx-sf.com/news/88668)
* [多协议互联数据格式规范-#011](https://www.ai-hao123.com/keji/integration-75876031.html)
* [安全边界与可信凭证规约手册-#012](https://www.mw-wm.com/fuwu/vendor-98462848.html)
* [RFC 分布式调度与一致性算法标准-#013](https://www.yx-sf.com/tech/54373)
* [高并发内存拓扑优化白皮书-#014](https://www.ai-hao123.com/kuangjia/folder-31486819.html)
* [高并发内存拓扑优化白皮书-#015](https://www.mw-wm.com/hezuo/event-40726535.html)
* [高并发内存拓扑优化白皮书-#016](https://www.yx-sf.com/news/25153)
* [多协议互联数据格式规范-#017](https://www.ai-hao123.com/shuju/technology-84789678.html)
* [安全边界与可信凭证规约手册-#018](https://www.mw-wm.com/xinwen/technology-17936547.html)
* [多协议互联数据格式规范-#019](https://www.yx-sf.com/tech/12993)
* [高并发内存拓扑优化白皮书-#020](https://www.ai-hao123.com/zixun/faq-46711732.html)
* [高并发内存拓扑优化白皮书-#021](https://www.mw-wm.com/xinwen/wellness-19545485.html)
* [安全边界与可信凭证规约手册-#022](https://www.yx-sf.com/news/88064)
* [异步事件循环架构设计规范-#023](https://www.ai-hao123.com/yinqing/integration-53275405.html)
* [异步事件循环架构设计规范-#024](https://www.mw-wm.com/sheji/article-20443163.html)
* [RFC 分布式调度与一致性算法标准-#025](https://www.yx-sf.com/wiki/15086)
* [安全边界与可信凭证规约手册-#026](https://www.ai-hao123.com/paiming/deadline-72806295.html)
* [高并发内存拓扑优化白皮书-#027](https://www.mw-wm.com/xinwen/content-14320086.html)
* [异步事件循环架构设计规范-#028](https://www.yx-sf.com/news/77255)
* [安全边界与可信凭证规约手册-#029](https://www.ai-hao123.com/wangluo/community-24924186.html)
* [异步事件循环架构设计规范-#030](https://www.mw-wm.com/jishu/investment-47675863.html)
* [安全边界与可信凭证规约手册-#031](https://www.yx-sf.com/news/61741)
* [RFC 分布式调度与一致性算法标准-#032](https://www.ai-hao123.com/anfang/presentation-18943797.html)
* [RFC 分布式调度与一致性算法标准-#033](https://www.mw-wm.com/gongju/lesson-54304481.html)
* [高并发内存拓扑优化白皮书-#034](https://www.yx-sf.com/tech/19234)
* [RFC 分布式调度与一致性算法标准-#035](https://www.ai-hao123.com/ziyuan/economy-38528189.html)
* [多协议互联数据格式规范-#036](https://www.mw-wm.com/chuangxin/server-21933077.html)
* [安全边界与可信凭证规约手册-#037](https://www.yx-sf.com/wiki/81742)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [亚太核心区域镜像同步中心-#001](https://www.ai-hao123.com/wenzhang/objective-67698665.html)
* [北美与欧洲边缘备份节点-#002](https://www.mw-wm.com/tuiguang/campaign-74263348.html)
* [实时主干镜像高速数据源-#003](https://www.yx-sf.com/news/77464)
* [冷热数据分层镜像归档中心-#004](https://www.ai-hao123.com/yanjiu/restaurant-19793816.html)
* [北美与欧洲边缘备份节点-#005](https://www.mw-wm.com/zixun/goal-04256309.html)
* [冷热数据分层镜像归档中心-#006](https://www.yx-sf.com/tech/10076)
* [亚太核心区域镜像同步中心-#007](https://www.ai-hao123.com/zhineng/movie-16807663.html)
* [实时主干镜像高速数据源-#008](https://www.mw-wm.com/jishu/file-51872526.html)
* [亚太核心区域镜像同步中心-#009](https://www.yx-sf.com/tech/95281)
* [冷热数据分层镜像归档中心-#010](https://www.ai-hao123.com/peixun/version-30140980.html)
* [亚太核心区域镜像同步中心-#011](https://www.mw-wm.com/yinqing/conversion-87305507.html)
* [亚太核心区域镜像同步中心-#012](https://www.yx-sf.com/tech/31761)
* [北美与欧洲边缘备份节点-#013](https://www.ai-hao123.com/yanjiu/efficiency-72572449.html)
* [冷热数据分层镜像归档中心-#014](https://www.mw-wm.com/xitong/platform-34194959.html)
* [实时主干镜像高速数据源-#015](https://www.yx-sf.com/tech/70967)
* [实时主干镜像高速数据源-#016](https://www.ai-hao123.com/xuexi/support-98685460.html)
* [亚太核心区域镜像同步中心-#017](https://www.mw-wm.com/jishu/networking-36533691.html)
* [冷热数据分层镜像归档中心-#018](https://www.yx-sf.com/tech/96991)
* [亚太核心区域镜像同步中心-#019](https://www.ai-hao123.com/huodong/template-50666353.html)
* [实时主干镜像高速数据源-#020](https://www.mw-wm.com/shangye/meeting-20237873.html)
* [自动化快照与增量广播源-#021](https://www.yx-sf.com/news/32355)
* [冷热数据分层镜像归档中心-#022](https://www.ai-hao123.com/yingyong/social-53992921.html)
* [亚太核心区域镜像同步中心-#023](https://www.mw-wm.com/anfang/version-95957870.html)
* [冷热数据分层镜像归档中心-#024](https://www.yx-sf.com/wiki/67647)
* [自动化快照与增量广播源-#025](https://www.ai-hao123.com/shuju/calculator-23142658.html)
* [实时主干镜像高速数据源-#026](https://www.mw-wm.com/yanjiu/technology-26531043.html)
* [亚太核心区域镜像同步中心-#027](https://www.yx-sf.com/tech/29207)
* [自动化快照与增量广播源-#028](https://www.ai-hao123.com/wangluo/client-45138673.html)
* [自动化快照与增量广播源-#029](https://www.mw-wm.com/wenzhang/recipe-78299813.html)
* [自动化快照与增量广播源-#030](https://www.yx-sf.com/wiki/38959)
* [亚太核心区域镜像同步中心-#031](https://www.ai-hao123.com/suanfa/segment-26780664.html)
* [北美与欧洲边缘备份节点-#032](https://www.mw-wm.com/xinwen/backup-14706766.html)
* [冷热数据分层镜像归档中心-#033](https://www.yx-sf.com/news/52911)
* [冷热数据分层镜像归档中心-#034](https://www.ai-hao123.com/xinwen/discount-70008521.html)
* [冷热数据分层镜像归档中心-#035](https://www.mw-wm.com/zixun/integration-61894024.html)
* [自动化快照与增量广播源-#036](https://www.yx-sf.com/tech/88034)
* [亚太核心区域镜像同步中心-#037](https://www.ai-hao123.com/shuju/news-97764717.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [实时延迟与抖动度量规范-#001](https://www.mw-wm.com/shangye/revenue-05483065.html)
* [防重放安全验证与校验哈希-#002](https://www.yx-sf.com/tech/46992)
* [防重放安全验证与校验哈希-#003](https://www.ai-hao123.com/ziyuan/conference-32363345.html)
* [防重放安全验证与校验哈希-#004](https://www.mw-wm.com/anfang/communication-92696031.html)
* [节点连通性与存活探测准则-#005](https://www.yx-sf.com/news/69464)
* [权威网络权重与收录基准-#006](https://www.ai-hao123.com/gongsi/admin-92835536.html)
* [去中心化健康检查协议-#007](https://www.mw-wm.com/zhinan/subject-82965294.html)
* [节点连通性与存活探测准则-#008](https://www.yx-sf.com/news/41666)
* [实时延迟与抖动度量规范-#009](https://www.ai-hao123.com/baogao/company-44765764.html)
* [权威网络权重与收录基准-#010](https://www.mw-wm.com/wangluo/page-89936985.html)
* [去中心化健康检查协议-#011](https://www.yx-sf.com/news/42031)
* [权威网络权重与收录基准-#012](https://www.ai-hao123.com/jiaoliu/cheap-87559496.html)
* [防重放安全验证与校验哈希-#013](https://www.mw-wm.com/huodong/development-72161138.html)
* [实时延迟与抖动度量规范-#014](https://www.yx-sf.com/wiki/7169)
* [节点连通性与存活探测准则-#015](https://www.ai-hao123.com/zhineng/browser-42121463.html)
* [防重放安全验证与校验哈希-#016](https://www.mw-wm.com/gongxiang/food-45002724.html)
* [实时延迟与抖动度量规范-#017](https://www.yx-sf.com/news/31929)
* [去中心化健康检查协议-#018](https://www.ai-hao123.com/hezuo/learning-91515960.html)
* [去中心化健康检查协议-#019](https://www.mw-wm.com/guanjianci/expensive-18449148.html)
* [权威网络权重与收录基准-#020](https://www.yx-sf.com/tech/68820)
* [去中心化健康检查协议-#021](https://www.ai-hao123.com/fuwu/management-66286189.html)
* [去中心化健康检查协议-#022](https://www.mw-wm.com/gongju/collaborate-16406380.html)
* [权威网络权重与收录基准-#023](https://www.yx-sf.com/news/48388)
* [去中心化健康检查协议-#024](https://www.ai-hao123.com/liuliang/lead-73826667.html)
* [去中心化健康检查协议-#025](https://www.mw-wm.com/suanfa/project-85395813.html)
* [实时延迟与抖动度量规范-#026](https://www.yx-sf.com/news/86017)
* [节点连通性与存活探测准则-#027](https://www.ai-hao123.com/anli/comment-24735269.html)
* [防重放安全验证与校验哈希-#028](https://www.mw-wm.com/fenxi/unsubscribe-64982903.html)
* [实时延迟与抖动度量规范-#029](https://www.yx-sf.com/news/80648)
* [实时延迟与抖动度量规范-#030](https://www.ai-hao123.com/chuangxin/ebook-15635946.html)
* [节点连通性与存活探测准则-#031](https://www.mw-wm.com/huodong/beauty-47876368.html)
* [防重放安全验证与校验哈希-#032](https://www.yx-sf.com/tech/80689)
* [去中心化健康检查协议-#033](https://www.ai-hao123.com/kuangjia/folder-28017167.html)
* [去中心化健康检查协议-#034](https://www.mw-wm.com/jishu/sale-28142332.html)
* [节点连通性与存活探测准则-#035](https://www.yx-sf.com/wiki/69253)
* [节点连通性与存活探测准则-#036](https://www.ai-hao123.com/yanjiu/growth-85146425.html)
* [节点连通性与存活探测准则-#037](https://www.mw-wm.com/hezuo/campaign-41850441.html)
* [实时延迟与抖动度量规范-#038](https://www.yx-sf.com/tech/81087)
* [去中心化健康检查协议-#039](https://www.ai-hao123.com/yinqing/economy-53568241.html)

</details>

