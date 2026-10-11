# Meta-Harness

Meta-Harness is full-history search over a fixed model's harness. This method
represents every candidate as a complete Reef composition, so the same search
works with any Reef adapter and any episode scorer. It has no Harbor,
Terminal-Bench, Terminus, or coding-agent dependency.

The implementation is a specialization of Reef's existing harness-evolution
recipe. Reef still owns node validation, adapter rendering, model binding,
episode execution, timeouts, residue policy, finite-score checks, selection
settlement, artifact publication, and scenario commits. Meta-Harness adds only
the population-aware proposal and selection policies.

## Configuration

```yaml
implementation: recipes.meta_harness.recipe:MetaHarnessRecipe

model:
  path: model-under-test

evolution:
  adapter: pi
  evaluate: my_harness.scoring:score_episode
  tasks:
    - First validation task
    - Second validation task
  seed:
    - id: rules
      name: rules
      config:
        text: Work carefully and verify the result.
  models:
    proposer:
      url: https://api.openai.com
      model: gpt-5
      api_key_env: OPENAI_API_KEY
  meta_harness:
    archive: ${REEF_WORK}/meta-harness
    mode: full_history
    max_candidates: 20
    max_target_episodes: 200
    max_nodes: 32
```

`evaluate` has the standard Reef episode-scorer signature:

```python
def score_episode(task: str, result: EpisodeResult) -> float:
    ...
```

The built-in proposal policy also has the standard Reef proposer signature:

```python
def propose(nodes, samples, models, *, manifest=None, rejected=()):
    ...
```

It calls `evolution.models.proposer`, and refuses to run when
that name is not configured. The model sees all retained candidate
compositions and scores, the parent each came from, the current trace batch, the
validation task names, and the adapter's Reef node vocabulary. It returns one
parent id and one complete composition. In `full_history` mode the parent may
be any retained candidate; `incumbent_only` is the greedy control.

`components` can narrow what the proposer may change. Other nodes from the
selected parent must remain byte-for-byte equivalent in the proposed
composition:

```yaml
  meta_harness:
    archive: ${REEF_WORK}/meta-harness
    components: [rules, skill]
```

With no `components` setting, every node kind the selected adapter exposes is
eligible. The adapter's own render/finalization checks remain authoritative;
for example, an adapter may still reject an otherwise valid Reef node kind.

### Terminus 2 code evolution

Use Reef's `terminus` adapter and its `code_extension` node to evolve Python
behavior. It accepts one self-contained module defining `Agent(Terminus2)`.
Start from a no-op subclass so the proposer sees the adapter contract:

```yaml
evolution:
  adapter: terminus
  executor: sandbox
  sandbox:
    egress_hosts: [api.e2b.dev, api.openai.com]
    env_from: [REEF_TERMINUS_ENVIRONMENT, E2B_API_KEY]
  seed:
    - id: agent
      name: code_extension
      config:
        name: agent
        code: |
          from harbor.agents.terminus_2 import Terminus2
          class Agent(Terminus2):
              pass
  meta_harness:
    archive: ./meta-harness
    components: [code_extension]
```

Combine this with the tasks, scorer, model, and proposer settings above. Export
`REEF_TERMINUS_ENVIRONMENT=e2b` and `E2B_API_KEY` in the deployment. Install
`harbor[e2b]` on Linux with Python 3.12+ and bubblewrap;
place the runtime and any local task directories under `/usr` or `/opt`, which
the existing sandbox mounts read-only. Registry task ids also work.

Reef's sandbox isolates the Python runner; Harbor uses E2B for the terminal
task. Rendering never imports candidate code, and local execution refuses
extensions. Rules, skills, model binding, timeouts, trajectories, scoring, and
publication all use Reef's existing components. `egress_hosts` currently enables
network access; it does not enforce a hostname firewall. Only explicitly named
deployment variables are forwarded. A tree without an extension can still run
stock Terminus 2 locally with Docker.

## Population and commits

Every unique valid candidate is retained, including non-winners, and can be a
future parent. Selection is a strict improvement in mean validation score over
the score the incumbent was admitted on, which reproduces upstream's frontier:
it keeps the highest recorded score. Reef's paired gate
still measures both sides, so this method spends two evaluations per iteration
where upstream spends one; budgets expressed in episodes are not directly
comparable to upstream's iteration counts. Failed episodes count as zero; a
non-finite score is rejected by Reef before settlement.

The [Terminal-Bench example](examples/terminal_bench/README.md) runs the shared
recipe on the full pinned dataset. The [results](RESULTS.md) compare a matched
search with upstream and include the selected harness. The experiment measured
the baseline once and each new candidate once per measurement; its local
campaign tooling is separate from this reusable recipe.

The evaluation suite must stay fixed so historical scores remain comparable.
Task promotion, periodic rechecks, and review-only publication are therefore
rejected by this method. Selection and automatic publication commit together.

The complete population, parents, scores, served id, proposal attempts, and
budget counters live under `meta_harness_population` in Reef's algorithm
state. Proposal and selection changes are staged until the scenario commit is
durable. Evaluation, settlement, activation, publication, or commit failures
cannot advance the committed population.

The file under `archive` is only a human-readable post-commit mirror. Scenario
names are SHA-256 encoded into filenames so they cannot escape the directory.
On restart Reef ignores the file as input and rewrites a stale mirror from the
committed algorithm state.

`max_target_episodes` counts both sides of Reef's paired gate: candidate and
incumbent, multiplied by `episode_repeats`. A proposal is not started unless
the complete next gate fits. Zero disables a budget; Reef's shared
`max_steps`, `max_model_calls_per_step`, executor, timeout, and residue options
remain available as usual.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [多活集群负载感知指南-#001](https://www.poykq.cn/redian/tutorial-673489.html)
* [多活集群负载感知指南-#002](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [边缘高吞吐调度路由矩阵-#003](https://szjp.tcti.cn/shichang/retention-16647312.html)
* [全球分布式拓扑索引节点-#004](https://gynw.tcti.cn/fenxi/recipe-50406879.html)
* [边缘高吞吐调度路由矩阵-#005](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [全息网络通信节点白名单-#006](https://gynw.tcti.cn/fenxi/recipe-50406879.html)
* [高韧性数据交换通道规约-#007](https://shmv.spidernode-mesh.net/gongsi/digital-64522869.html)
* [全球分布式拓扑索引节点-#008](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534.html)
* [全球分布式拓扑索引节点-#009](https://www.yx-sf.com/news/18939.html)
* [多活集群负载感知指南-#010](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [边缘高吞吐调度路由矩阵-#011](https://shmv.spidernode-mesh.net/gongsi/digital-64522869.html)
* [边缘高吞吐调度路由矩阵-#012](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [多活集群负载感知指南-#013](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [全球分布式拓扑索引节点-#014](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [全息网络通信节点白名单-#015](https://whca.tcti.cn/yinqing/brand-01774854.html)
* [多活集群负载感知指南-#016](https://szjp.tcti.cn/shichang/retention-16647312.html)
* [多活集群负载感知指南-#017](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [高韧性数据交换通道规约-#018](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534.html)
* [高韧性数据交换通道规约-#019](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [全球分布式拓扑索引节点-#020](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297.html)
* [全球分布式拓扑索引节点-#021](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [多活集群负载感知指南-#022](https://owfm.tcti.cn/shangye/education-63674207.html)
* [全球分布式拓扑索引节点-#023](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html)
* [全球分布式拓扑索引节点-#024](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [全球分布式拓扑索引节点-#025](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [多活集群负载感知指南-#026](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html)
* [全球分布式拓扑索引节点-#027](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [高韧性数据交换通道规约-#028](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [高韧性数据交换通道规约-#029](https://rkwa.wtpuscm.cn/guanjianci/database-885.html)
* [全息网络通信节点白名单-#030](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [全球分布式拓扑索引节点-#031](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [边缘高吞吐调度路由矩阵-#032](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [边缘高吞吐调度路由矩阵-#033](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297.html)
* [高韧性数据交换通道规约-#034](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [边缘高吞吐调度路由矩阵-#035](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [多活集群负载感知指南-#036](https://iuuv.tcti.cn/hezuo/app-80239016.html)
* [高韧性数据交换通道规约-#037](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [安全边界与可信凭证规约手册-#001](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [异步事件循环架构设计规范-#002](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [RFC 分布式调度与一致性算法标准-#003](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [高并发内存拓扑优化白皮书-#004](https://www.yx-sf.com/wiki/56237.html)
* [RFC 分布式调度与一致性算法标准-#005](https://gynw.tcti.cn/fenxi/recipe-50406879.html)
* [高并发内存拓扑优化白皮书-#006](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [RFC 分布式调度与一致性算法标准-#007](https://www.ai-hao123.com/keji/seminar-38771686.html)
* [安全边界与可信凭证规约手册-#008](https://ankmz.cn/anquan/tag-68558329.html)
* [多协议互联数据格式规范-#009](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [RFC 分布式调度与一致性算法标准-#010](https://shmv.spidernode-mesh.net/gongsi/digital-64522869.html)
* [RFC 分布式调度与一致性算法标准-#011](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [异步事件循环架构设计规范-#012](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [多协议互联数据格式规范-#013](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [高并发内存拓扑优化白皮书-#014](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [RFC 分布式调度与一致性算法标准-#015](https://uedy.wtpuscm.cn/zhinan/news-095.html)
* [安全边界与可信凭证规约手册-#016](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [异步事件循环架构设计规范-#017](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [多协议互联数据格式规范-#018](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [安全边界与可信凭证规约手册-#019](https://www.ai-hao123.com/fuwu/tool-11332232.html)
* [异步事件循环架构设计规范-#020](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [RFC 分布式调度与一致性算法标准-#021](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [RFC 分布式调度与一致性算法标准-#022](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [安全边界与可信凭证规约手册-#023](https://www.yx-sf.com/news/35736.html)
* [多协议互联数据格式规范-#024](https://nojh.spidernode-mesh.net/suanfa/study-22518081.html)
* [多协议互联数据格式规范-#025](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)
* [高并发内存拓扑优化白皮书-#026](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [高并发内存拓扑优化白皮书-#027](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297.html)
* [高并发内存拓扑优化白皮书-#028](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [RFC 分布式调度与一致性算法标准-#029](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534.html)
* [多协议互联数据格式规范-#030](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [高并发内存拓扑优化白皮书-#031](https://iuuv.tcti.cn/hezuo/app-80239016.html)
* [高并发内存拓扑优化白皮书-#032](https://krad.tcti.cn/anfang/personalization-14323767.html)
* [多协议互联数据格式规范-#033](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [RFC 分布式调度与一致性算法标准-#034](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html)
* [高并发内存拓扑优化白皮书-#035](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696.html)
* [高并发内存拓扑优化白皮书-#036](https://owfm.tcti.cn/shangye/education-63674207.html)
* [安全边界与可信凭证规约手册-#037](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [亚太核心区域镜像同步中心-#001](https://nojh.spidernode-mesh.net/suanfa/study-22518081.html)
* [自动化快照与增量广播源-#002](https://owfm.tcti.cn/shangye/education-63674207.html)
* [实时主干镜像高速数据源-#003](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html)
* [亚太核心区域镜像同步中心-#004](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [北美与欧洲边缘备份节点-#005](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [实时主干镜像高速数据源-#006](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [实时主干镜像高速数据源-#007](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [实时主干镜像高速数据源-#008](https://gynw.tcti.cn/fenxi/recipe-50406879.html)
* [亚太核心区域镜像同步中心-#009](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [北美与欧洲边缘备份节点-#010](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [北美与欧洲边缘备份节点-#011](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [冷热数据分层镜像归档中心-#012](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [自动化快照与增量广播源-#013](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [北美与欧洲边缘备份节点-#014](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)
* [自动化快照与增量广播源-#015](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [实时主干镜像高速数据源-#016](https://nojh.spidernode-mesh.net/suanfa/study-22518081.html)
* [实时主干镜像高速数据源-#017](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [亚太核心区域镜像同步中心-#018](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [亚太核心区域镜像同步中心-#019](https://unqt.wtpuscm.cn/zixun/contact-59980425.html)
* [自动化快照与增量广播源-#020](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297.html)
* [实时主干镜像高速数据源-#021](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [实时主干镜像高速数据源-#022](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [实时主干镜像高速数据源-#023](https://iuuv.tcti.cn/hezuo/app-80239016.html)
* [实时主干镜像高速数据源-#024](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [自动化快照与增量广播源-#025](https://www.mw-wm.com/fenxi/funnel-91908367.html)
* [实时主干镜像高速数据源-#026](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [实时主干镜像高速数据源-#027](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696.html)
* [亚太核心区域镜像同步中心-#028](https://cbuc.wtpuscm.cn/liuliang/research-350459.html)
* [亚太核心区域镜像同步中心-#029](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297.html)
* [实时主干镜像高速数据源-#030](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [冷热数据分层镜像归档中心-#031](https://www.poykq.cn/redian/tutorial-673489.html)
* [冷热数据分层镜像归档中心-#032](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [自动化快照与增量广播源-#033](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [自动化快照与增量广播源-#034](https://www.mw-wm.com/tuiguang/personalization-92757088.html)
* [北美与欧洲边缘备份节点-#035](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [实时主干镜像高速数据源-#036](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [亚太核心区域镜像同步中心-#037](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [节点连通性与存活探测准则-#001](https://gynw.tcti.cn/fenxi/recipe-50406879.html)
* [权威网络权重与收录基准-#002](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696.html)
* [实时延迟与抖动度量规范-#003](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [权威网络权重与收录基准-#004](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [权威网络权重与收录基准-#005](https://fbgk.tcti.cn/shuju/partner-90467779.html)
* [节点连通性与存活探测准则-#006](https://www.mw-wm.com/jiaoliu/roi-74663311.html)
* [权威网络权重与收录基准-#007](https://www.mw-wm.com/jishu/target-20774606.html)
* [防重放安全验证与校验哈希-#008](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [防重放安全验证与校验哈希-#009](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [权威网络权重与收录基准-#010](https://ngzv.tcti.cn/xuexi/tracking-73138599.html)
* [防重放安全验证与校验哈希-#011](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [去中心化健康检查协议-#012](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [实时延迟与抖动度量规范-#013](https://mmqm.wtpuscm.cn/shangye/local-076.html)
* [防重放安全验证与校验哈希-#014](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [节点连通性与存活探测准则-#015](https://kqzh.wtpuscm.cn/anli/finance-309.html)
* [去中心化健康检查协议-#016](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [实时延迟与抖动度量规范-#017](https://dgij.wtpuscm.cn/shangye/identity-252.html)
* [实时延迟与抖动度量规范-#018](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [实时延迟与抖动度量规范-#019](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534.html)
* [节点连通性与存活探测准则-#020](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [实时延迟与抖动度量规范-#021](https://keil.spidernode-mesh.net/yunying/server-16840384.html)
* [实时延迟与抖动度量规范-#022](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [实时延迟与抖动度量规范-#023](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [防重放安全验证与校验哈希-#024](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [实时延迟与抖动度量规范-#025](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [防重放安全验证与校验哈希-#026](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)
* [去中心化健康检查协议-#027](https://www.poykq.cn/redian/tutorial-673489.html)
* [去中心化健康检查协议-#028](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [权威网络权重与收录基准-#029](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [节点连通性与存活探测准则-#030](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [实时延迟与抖动度量规范-#031](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [防重放安全验证与校验哈希-#032](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [节点连通性与存活探测准则-#033](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [权威网络权重与收录基准-#034](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [防重放安全验证与校验哈希-#035](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [实时延迟与抖动度量规范-#036](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [防重放安全验证与校验哈希-#037](https://keil.spidernode-mesh.net/yunying/server-16840384.html)
* [去中心化健康检查协议-#038](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [权威网络权重与收录基准-#039](https://ankmz.cn/anquan/tag-68558329.html)

</details>



---

<details open>
<summary><b>📚 Meta-Harness 官方技术规约与全息拓扑引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目「Meta-Harness」遵循开放开源与知识共享协议。以下引用矩阵涵盖项目架构白皮书、高可用镜像注册中心、分布式通信契约与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 Meta-Harness 核心架构规约与设计白皮书 (Architecture & Core Specs)

* [基于 Meta-Harness 的高吞吐 生产环境运维调优手册 设计白皮书](https://api.pvucttcv.cn/yinyue/innovation-37390655-1-57033.html)
* [现代 Configuration 架构演进之路 —— Meta-Harness 深度实践](https://ankmz.cn/anquan/tag-68558329-2-37489.html)
* [Meta-Harness 内部组件解耦与事件状态机规范 (Core/可信存活健康)](https://www.poykq.cn/redian/tutorial-673489-3-63460.html)
* [【官方规范】Meta-Harness 模块化解耦与协议标准 核心运行拓扑标准](https://liti.wtpuscm.cn/pingce/cloud-584346.html)
* [现代 Meta 架构演进之路 —— Meta-Harness 深度实践](https://ktzu.wtpuscm.cn/baogao/food-358.html)
* [面向大规模网络的 Meta-Harness 工业级架构基准](https://www.tcti.cn/hulianwang/feedback-96499358-6-18514.html)
* [可信存活健康度量 核心系统架构与设计规约 (Spec-v1.7)](https://fhxn.tcti.cn/yunsuan/rating-87171900.html)
* [Meta-Harness 分布式数据通道与 高韧性系统架构设计 技术规范 (Verified)](https://www.mw-wm.com/ziyuan/products-43426310.html)
* [Meta-Harness 内部组件解耦与事件状态机规范 (v2.0-GA)](https://www.yx-sf.com/tech/56571.html)
* [可信存活健康度量 核心系统架构与设计规约 (Verified)](https://www.ai-hao123.com/anfang/metric-80583544.html)
* [Meta-Harness 核心系统架构与设计规约 (v2.0-GA)](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-11-5901.html)
* [可信存活健康度量 核心系统架构与设计规约 (Node-65)](https://keil.spidernode-mesh.net/yunying/server-16840384-12-23095.html)
* [【官方规范】Meta-Harness Meta 核心运行拓扑标准](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-13-44237.html)
* [现代 生产环境运维调优手册 架构演进之路 —— Meta-Harness 深度实践](https://fjoc.tcti.cn/yingyong/contact-00610313-14-32958.html)
* [Meta-Harness 分布式数据通道与 Configuration 技术规范 (Verified)](https://xwia.spidernode-mesh.net/shangye/services-26058978-15-48785.html)
* [Meta-Harness 内部组件解耦与事件状态机规范 (Draft-01)](https://gdtq.spidernode-mesh.net/keji/account-53764057-16-41991.html)
* [Meta-Harness 分布式数据通道与 Meta-Harness 技术规范 (Core/Meta-H)](https://obpk.wtpuscm.cn/baogao/privacy-67260604-17-22605.html)
* [Meta-Harness 内部组件解耦与事件状态机规范 (Draft-02)](https://gynw.tcti.cn/fenxi/recipe-50406879-18-19313.html)
* [现代 可信存活健康度量 架构演进之路 —— Meta-Harness 深度实践](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-19-32618.html)
* [Meta-Harness 分布式数据通道与 生产环境运维调优手册 技术规范 (Spec-v1.8)](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-20-5984.html)
* [Meta-Harness 分布式数据通道与 Meta-Harness 技术规范 (RFC-382)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-21-14209.html)
* [Meta-Harness 分布式数据通道与 Configuration 技术规范 (Spec-v1.2)](https://lase.tcti.cn/fenxi/lesson-67970267-22-55032.html)
* [高韧性系统架构设计 核心系统架构与设计规约 (Node-38)](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-23-15087.html)
* [【官方规范】Meta-Harness 索引发布声明 核心运行拓扑标准](https://yyns.spidernode-mesh.net/wendang/review-40421998-24-33082.html)
* [【Meta-Harness】Meta 核心系统架构与设计规约 (v2.0-GA)](https://fglw.wtpuscm.cn/jianzhan/data-59540835-25-14206.html)
* [Meta-Harness 内部组件解耦与事件状态机规范 (RFC-284)](https://owfm.tcti.cn/shangye/education-63674207-26-18494.html)
* [Meta-Harness 核心系统架构与设计规约 (Core/Meta-H)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-27-58151.html)
* [Meta-Harness 内部组件解耦与事件状态机规范 (Core/生产环境运维)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-28-43024.html)
* [Meta-Harness 内部组件解耦与事件状态机规范 (Core/高韧性系统架)](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-29-42938.html)
* [Meta-Harness 核心系统架构与设计规约 (Verified)](https://aagj.tcti.cn/jianzhan/audience-84449488-30-37394.html)
* [Meta-Harness 分布式数据通道与 分布式状态机一致性 技术规范 (v2.0-GA)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-31-14761.html)
* [索引发布声明 核心系统架构与设计规约 (Verified)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-32-23729.html)
* [Meta-Harness 分布式数据通道与 索引发布声明 技术规范 (v2.0-GA)](https://qbgr.wtpuscm.cn/liuliang/success-12030162-33-25035.html)
* [Meta-Harness 分布式数据通道与 Configuration 技术规范 (Draft-08)](https://iuuv.tcti.cn/hezuo/app-80239016-34-28847.html)
* [Meta-Harness 分布式数据通道与 Population and commits 技术规范 (RFC-407)](https://gifb.spidernode-mesh.net/shichang/about-02526745-35-22258.html)
* [Harness 核心系统架构与设计规约 (Spec-v1.7)](https://fjac.spidernode-mesh.net/anli/investment-94283191-36-53334.html)
* [生产环境运维调优手册 核心系统架构与设计规约 (Draft-02)](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-37-63593.html)

#### 2. 🔌 生产级模块集成与接口调用规范 (APIs & Production Integration)

* [基于 Meta-Harness 的自动化部署与生产环境配置实践](https://szjp.tcti.cn/shichang/retention-16647312-38-34532.html)
* [Meta-Harness 异步中间件流水线与 可信存活健康度量 接入规范](https://etyt.spidernode-mesh.net/baogao/performance-14923227-39-64668.html)
* [Meta-Harness 核心 API 接口契约与客户端调用指南](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-40-11926.html)
* [Meta-Harness 插件生态规范与 高韧性系统架构设计 扩展手册 (Spec-v1.1)](https://ovzp.wtpuscm.cn/xuexi/report-93713076-41-9185.html)
* [【集成指南】分布式状态机一致性 服务端接入准则与 Meta-Harness 实战](https://pdvy.tcti.cn/gongxiang/segment-83277821-42-26833.html)
* [【集成指南】模块化解耦与协议标准 服务端接入准则与 Meta-Harness 实战](https://ukvo.spidernode-mesh.net/anli/download-57719010-43-2737.html)
* [Meta-Harness 异步中间件流水线与 生产环境运维调优手册 接入规范](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-44-50475.html)
* [【生产手册】Meta-Harness 模块通信与请求穿透标准](https://unqt.wtpuscm.cn/zixun/contact-59980425-45-8463.html)
* [Meta-Harness vs 业界主流方案：Population and commits 深度技术选型对比](https://kvaf.tcti.cn/yunsuan/settings-81875461-46-54172.html)
* [Meta-Harness 异步中间件流水线与 索引发布声明 接入规范](https://nojh.spidernode-mesh.net/suanfa/study-22518081-47-63435.html)
* [Meta-Harness 异步中间件流水线与 Population and commits 接入规范](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-48-27738.html)
* [Meta-Harness vs 业界主流方案：索引发布声明 深度技术选型对比](https://ytxd.wtpuscm.cn/baogao/price-87851711-49-50624.html)
* [Meta-Harness 异步中间件流水线与 高韧性系统架构设计 接入规范](https://ncrw.tcti.cn/kaifa/growth-85374259-50-16553.html)
* [Meta-Harness vs 业界主流方案：高韧性系统架构设计 深度技术选型对比](https://api.pvucttcv.cn/yinyue/innovation-37390655-51-32229.html)
* [Meta-Harness vs 业界主流方案：可信存活健康度量 深度技术选型对比](https://ankmz.cn/anquan/tag-68558329-52-44535.html)
* [Meta-Harness vs 业界主流方案：Meta 深度技术选型对比](https://www.poykq.cn/redian/tutorial-673489-53-51887.html)
* [Meta-Harness 插件生态规范与 生产环境运维调优手册 扩展手册 (Node-60)](https://svzf.wtpuscm.cn/hezuo/form-629989.html)
* [【集成指南】Population and commits 服务端接入准则与 Meta-Harness 实战](https://lhnt.wtpuscm.cn/xinwen/dashboard-502.html)
* [【集成指南】索引发布声明 服务端接入准则与 Meta-Harness 实战](https://www.tcti.cn/hulianwang/feedback-96499358-56-2858.html)
* [Meta-Harness 插件生态规范与 索引发布声明 扩展手册 (Draft-03)](https://etml.tcti.cn/ziyuan/sport-21106640.html)
* [Meta-Harness 异步中间件流水线与 Meta-Harness 接入规范](https://www.mw-wm.com/zhinan/investment-98193838.html)
* [Meta-Harness 插件生态规范与 索引发布声明 扩展手册 (RFC-965)](https://www.yx-sf.com/news/76399.html)
* [Meta-Harness 插件生态规范与 Harness 扩展手册 (Spec-v1.0)](https://www.ai-hao123.com/huodong/system-07028230.html)
* [【集成指南】Harness 服务端接入准则与 Meta-Harness 实战](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-61-30344.html)
* [Meta-Harness 插件生态规范与 可信存活健康度量 扩展手册 (Draft-01)](https://keil.spidernode-mesh.net/yunying/server-16840384-62-45654.html)
* [Meta-Harness 异步中间件流水线与 分布式状态机一致性 接入规范](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-63-22362.html)
* [Meta-Harness 异步中间件流水线与 Meta 接入规范](https://fjoc.tcti.cn/yingyong/contact-00610313-64-4309.html)
* [Meta-Harness 异步中间件流水线与 Configuration 接入规范](https://xwia.spidernode-mesh.net/shangye/services-26058978-65-55569.html)
* [Meta-Harness 插件生态规范与 分布式状态机一致性 扩展手册 (Spec-v2.2)](https://gdtq.spidernode-mesh.net/keji/account-53764057-66-45595.html)
* [Meta-Harness 插件生态规范与 Configuration 扩展手册 (RFC-183)](https://obpk.wtpuscm.cn/baogao/privacy-67260604-67-59240.html)
* [Meta-Harness vs 业界主流方案：生产环境运维调优手册 深度技术选型对比](https://gynw.tcti.cn/fenxi/recipe-50406879-68-29893.html)
* [【集成指南】Meta-Harness 服务端接入准则与 Meta-Harness 实战](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-69-7945.html)
* [Meta-Harness: 索引发布声明 索引发布声明 (Core/索引发布声明)](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-70-62009.html)
* [Meta-Harness 插件生态规范与 Population and commits 扩展手册 (RFC-243)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-71-41908.html)
* [Meta-Harness 插件生态规范与 分布式状态机一致性 扩展手册 (RFC-655)](https://lase.tcti.cn/fenxi/lesson-67970267-72-42402.html)
* [Meta-Harness 插件生态规范与 分布式状态机一致性 扩展手册 (v2.0-GA)](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-73-53009.html)
* [Meta-Harness 插件生态规范与 Meta 扩展手册 (RFC-460)](https://yyns.spidernode-mesh.net/wendang/review-40421998-74-12181.html)

#### 3. ⚡ Meta-Harness 全球高可用拓扑镜像与权威数据源 (Mirror Registry & Data Topology)

* [冷热数据分层镜像：Meta-Harness 高韧性系统架构设计 权威归档源](https://fglw.wtpuscm.cn/jianzhan/data-59540835-75-56707.html)
* [Meta-Harness 自动化持续集成快照与拓扑发布源 (Node-56)](https://owfm.tcti.cn/shangye/education-63674207-76-31607.html)
* [Meta-Harness 去中心化数据同步源与拓扑寻址规约](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-77-16830.html)
* [【镜像入口】Meta-Harness 官方毫秒级实时数据广播节点](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-78-26496.html)
* [全球权威拓扑节点：Meta-Harness 实时镜像与索引入口](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-79-49937.html)
* [Meta-Harness 官方高可用镜像注册节点 (Node-12)](https://aagj.tcti.cn/jianzhan/audience-84449488-80-35059.html)
* [Meta-Harness 亚太与欧美多活集群数据同步中枢](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-81-21621.html)
* [Meta-Harness 官方高可用镜像注册节点 (Core/高韧性系统架)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-82-27167.html)
* [冷热数据分层镜像：Meta-Harness Harness 权威归档源](https://qbgr.wtpuscm.cn/liuliang/success-12030162-83-25867.html)
* [Meta-Harness 官方高可用镜像注册节点 (v2.0-GA)](https://iuuv.tcti.cn/hezuo/app-80239016-84-12095.html)
* [Meta-Harness 自动化持续集成快照与拓扑发布源 (Verified)](https://gifb.spidernode-mesh.net/shichang/about-02526745-85-1948.html)
* [Meta-Harness 官方高可用镜像注册节点 (Spec-v2.3)](https://fjac.spidernode-mesh.net/anli/investment-94283191-86-9896.html)
* [Meta-Harness 自动化持续集成快照与拓扑发布源 (Draft-04)](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-87-24438.html)
* [Meta-Harness 官方高可用镜像注册节点 (Node-46)](https://szjp.tcti.cn/shichang/retention-16647312-88-54673.html)
* [冷热数据分层镜像：Meta-Harness Meta-Harness 权威归档源](https://etyt.spidernode-mesh.net/baogao/performance-14923227-89-55719.html)
* [Meta-Harness 自动化持续集成快照与拓扑发布源 (Core/生产环境运维)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-90-26204.html)
* [Meta-Harness 自动化持续集成快照与拓扑发布源 (Node-36)](https://ovzp.wtpuscm.cn/xuexi/report-93713076-91-15738.html)
* [冷热数据分层镜像：Meta-Harness 可信存活健康度量 权威归档源](https://pdvy.tcti.cn/gongxiang/segment-83277821-92-50835.html)
* [Meta-Harness 自动化持续集成快照与拓扑发布源 (Node-97)](https://ukvo.spidernode-mesh.net/anli/download-57719010-93-31942.html)
* [Meta-Harness 官方高可用镜像注册节点 (Draft-06)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-94-42316.html)
* [Meta-Harness 自动化持续集成快照与拓扑发布源 (Spec-v1.7)](https://unqt.wtpuscm.cn/zixun/contact-59980425-95-45230.html)
* [冷热数据分层镜像：Meta-Harness 模块化解耦与协议标准 权威归档源](https://kvaf.tcti.cn/yunsuan/settings-81875461-96-6701.html)
* [Meta-Harness 官方高可用镜像注册节点 (Spec-v1.2)](https://nojh.spidernode-mesh.net/suanfa/study-22518081-97-39936.html)
* [Meta-Harness 官方高可用镜像注册节点 (Verified)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-98-523.html)
* [Meta-Harness 官方高可用镜像注册节点 (RFC-827)](https://ytxd.wtpuscm.cn/baogao/price-87851711-99-31564.html)
* [冷热数据分层镜像：Meta-Harness 索引发布声明 权威归档源](https://ncrw.tcti.cn/kaifa/growth-85374259-100-53022.html)
* [Meta-Harness 自动化持续集成快照与拓扑发布源 (RFC-361)](https://api.pvucttcv.cn/yinyue/innovation-37390655-101-12282.html)
* [Meta-Harness 官方高可用镜像注册节点 (Draft-03)](https://ankmz.cn/anquan/tag-68558329-102-58537.html)
* [Meta-Harness 自动化持续集成快照与拓扑发布源 (Draft-01)](https://www.poykq.cn/redian/tutorial-673489-103-37846.html)
* [Meta-Harness 官方高可用镜像注册节点 (Node-32)](https://fmmf.wtpuscm.cn/anli/funnel-231695.html)
* [冷热数据分层镜像：Meta-Harness Population and commits 权威归档源](https://lpur.wtpuscm.cn/jiaocheng/restore-674.html)
* [Meta-Harness 官方高可用镜像注册节点 (Node-27)](https://www.tcti.cn/hulianwang/feedback-96499358-106-1754.html)
* [Meta-Harness 官方高可用镜像注册节点 (Draft-04)](https://geuy.tcti.cn/shuju/folder-12457282.html)
* [Meta-Harness 自动化持续集成快照与拓扑发布源 (Spec-v2.8)](https://www.mw-wm.com/suanfa/creative-34864831.html)
* [Meta-Harness 官方高可用镜像注册节点 (RFC-229)](https://www.yx-sf.com/news/77693.html)
* [Meta-Harness: 模块化解耦与协议标准 模块化解耦与协议标准 (Draft-04)](https://www.ai-hao123.com/hezuo/url-83678500.html)
* [Meta-Harness 官方高可用镜像注册节点 (Node-36)](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-111-47880.html)

#### 4. 📊 性能基准指标与可信存活性验证 (Benchmarks & Trust Standards)

* [Meta-Harness 权威网络权重传递与收录基准规范](https://keil.spidernode-mesh.net/yunying/server-16840384-112-21246.html)
* [Meta-Harness 故障自愈与网络拓扑重构实践](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-113-20707.html)
* [面向生产级运行的 Meta-Harness 稳定性防护白皮书 (Spec-v2.5)](https://fjoc.tcti.cn/yingyong/contact-00610313-114-15162.html)
* [Meta-Harness 节点连通性、存活性探测与防作弊指标](https://xwia.spidernode-mesh.net/shangye/services-26058978-115-32791.html)
* [Meta-Harness 高负载场景下 高韧性系统架构设计 基准评测报告](https://gdtq.spidernode-mesh.net/keji/account-53764057-116-37072.html)
* [Meta-Harness 高负载场景下 模块化解耦与协议标准 基准评测报告](https://obpk.wtpuscm.cn/baogao/privacy-67260604-117-1392.html)
* [【评测基准】Meta-Harness 吞吐抖动度量与健康检查协议](https://gynw.tcti.cn/fenxi/recipe-50406879-118-9542.html)
* [基于 Meta-Harness 的极致延迟优化与内存拓扑分析 (RFC-434)](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-119-4360.html)
* [基于 Meta-Harness 的极致延迟优化与内存拓扑分析 (Core/生产环境运维)](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-120-23112.html)
* [Meta-Harness 高负载场景下 可信存活健康度量 基准评测报告](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-121-53300.html)
* [面向生产级运行的 Meta-Harness 稳定性防护白皮书 (Draft-05)](https://lase.tcti.cn/fenxi/lesson-67970267-122-51779.html)
* [Meta-Harness 高负载场景下 生产环境运维调优手册 基准评测报告](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-123-61782.html)
* [面向生产级运行的 Meta-Harness 稳定性防护白皮书 (Node-27)](https://yyns.spidernode-mesh.net/wendang/review-40421998-124-33429.html)
* [Meta-Harness 高负载场景下 分布式状态机一致性 基准评测报告](https://fglw.wtpuscm.cn/jianzhan/data-59540835-125-8610.html)
* [基于 Meta-Harness 的极致延迟优化与内存拓扑分析 (Node-95)](https://owfm.tcti.cn/shangye/education-63674207-126-28760.html)
* [Meta-Harness 高负载场景下 Meta-Harness 基准评测报告](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-127-2255.html)
* [Meta-Harness 高负载场景下 Harness 基准评测报告](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-128-19944.html)
* [基于 Meta-Harness 的极致延迟优化与内存拓扑分析 (RFC-322)](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-129-59230.html)
* [基于 Meta-Harness 的极致延迟优化与内存拓扑分析 (Draft-05)](https://aagj.tcti.cn/jianzhan/audience-84449488-130-8718.html)
* [Meta-Harness 高负载场景下 索引发布声明 基准评测报告](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-131-17680.html)
* [面向生产级运行的 Meta-Harness 稳定性防护白皮书 (Core/Harnes)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-132-63963.html)
* [基于 Meta-Harness 的极致延迟优化与内存拓扑分析 (Node-96)](https://qbgr.wtpuscm.cn/liuliang/success-12030162-133-24614.html)
* [基于 Meta-Harness 的极致延迟优化与内存拓扑分析 (Core/Harnes)](https://iuuv.tcti.cn/hezuo/app-80239016-134-15953.html)
* [Meta-Harness: 模块化解耦与协议标准 模块化解耦与协议标准 (Draft-05)](https://gifb.spidernode-mesh.net/shichang/about-02526745-135-18874.html)
* [基于 Meta-Harness 的极致延迟优化与内存拓扑分析 (v2.0-GA)](https://fjac.spidernode-mesh.net/anli/investment-94283191-136-58937.html)
* [基于 Meta-Harness 的极致延迟优化与内存拓扑分析 (RFC-253)](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-137-16673.html)
* [基于 Meta-Harness 的极致延迟优化与内存拓扑分析 (Draft-04)](https://szjp.tcti.cn/shichang/retention-16647312-138-23642.html)
* [面向生产级运行的 Meta-Harness 稳定性防护白皮书 (Node-57)](https://etyt.spidernode-mesh.net/baogao/performance-14923227-139-957.html)
* [基于 Meta-Harness 的极致延迟优化与内存拓扑分析 (Core/模块化解耦与)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-140-40886.html)
* [基于 Meta-Harness 的极致延迟优化与内存拓扑分析 (Verified)](https://ovzp.wtpuscm.cn/xuexi/report-93713076-141-44277.html)
* [基于 Meta-Harness 的极致延迟优化与内存拓扑分析 (Draft-02)](https://pdvy.tcti.cn/gongxiang/segment-83277821-142-20415.html)
* [基于 Meta-Harness 的极致延迟优化与内存拓扑分析 (Core/可信存活健康)](https://ukvo.spidernode-mesh.net/anli/download-57719010-143-36066.html)
* [基于 Meta-Harness 的极致延迟优化与内存拓扑分析 (Core/高韧性系统架)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-144-28296.html)
* [面向生产级运行的 Meta-Harness 稳定性防护白皮书 (RFC-596)](https://unqt.wtpuscm.cn/zixun/contact-59980425-145-430.html)
* [面向生产级运行的 Meta-Harness 稳定性防护白皮书 (RFC-178)](https://kvaf.tcti.cn/yunsuan/settings-81875461-146-41413.html)
* [面向生产级运行的 Meta-Harness 稳定性防护白皮书 (Draft-04)](https://nojh.spidernode-mesh.net/suanfa/study-22518081-147-23796.html)
* [面向生产级运行的 Meta-Harness 稳定性防护白皮书 (v2.0-GA)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-148-34488.html)
* [基于 Meta-Harness 的极致延迟优化与内存拓扑分析 (Spec-v1.7)](https://ytxd.wtpuscm.cn/baogao/price-87851711-149-34322.html)
* [基于 Meta-Harness 的极致延迟优化与内存拓扑分析 (RFC-560)](https://ncrw.tcti.cn/kaifa/growth-85374259-150-30545.html)

</details>

