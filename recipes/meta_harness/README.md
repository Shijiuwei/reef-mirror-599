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

* [多活集群负载感知指南-#001](https://www.mw-wm.com/guanjianci/discount-88916551.html)
* [多活集群负载感知指南-#002](https://www.yx-sf.com/news/95242)
* [边缘高吞吐调度路由矩阵-#003](https://www.ai-hao123.com/shangye/collaborate-27957819.html)
* [全球分布式拓扑索引节点-#004](https://www.mw-wm.com/shangye/podcast-81025935.html)
* [边缘高吞吐调度路由矩阵-#005](https://www.yx-sf.com/wiki/76585)
* [全息网络通信节点白名单-#006](https://www.ai-hao123.com/qiye/brand-81180535.html)
* [高韧性数据交换通道规约-#007](https://www.mw-wm.com/sheji/technology-70559021.html)
* [全球分布式拓扑索引节点-#008](https://www.yx-sf.com/tech/98996)
* [全球分布式拓扑索引节点-#009](https://www.ai-hao123.com/yinqing/productivity-65200431.html)
* [多活集群负载感知指南-#010](https://www.mw-wm.com/wendang/revenue-16052589.html)
* [边缘高吞吐调度路由矩阵-#011](https://www.yx-sf.com/tech/4696)
* [边缘高吞吐调度路由矩阵-#012](https://www.ai-hao123.com/shichang/help-61933174.html)
* [多活集群负载感知指南-#013](https://www.mw-wm.com/liuliang/management-20297656.html)
* [全球分布式拓扑索引节点-#014](https://www.yx-sf.com/news/54230)
* [全息网络通信节点白名单-#015](https://www.ai-hao123.com/yunsuan/objective-53870968.html)
* [多活集群负载感知指南-#016](https://www.mw-wm.com/kuangjia/hotel-01142112.html)
* [多活集群负载感知指南-#017](https://www.yx-sf.com/wiki/50845)
* [高韧性数据交换通道规约-#018](https://www.ai-hao123.com/wangluo/segment-41568050.html)
* [高韧性数据交换通道规约-#019](https://www.mw-wm.com/yingyong/change-07548919.html)
* [全球分布式拓扑索引节点-#020](https://www.yx-sf.com/tech/5489)
* [全球分布式拓扑索引节点-#021](https://www.ai-hao123.com/yingxiao/widget-04016304.html)
* [多活集群负载感知指南-#022](https://www.mw-wm.com/suanfa/global-94716030.html)
* [全球分布式拓扑索引节点-#023](https://www.yx-sf.com/wiki/44379)
* [全球分布式拓扑索引节点-#024](https://www.ai-hao123.com/liuliang/recipe-50355751.html)
* [全球分布式拓扑索引节点-#025](https://www.mw-wm.com/keji/entertainment-26996347.html)
* [多活集群负载感知指南-#026](https://www.yx-sf.com/tech/29686)
* [全球分布式拓扑索引节点-#027](https://www.ai-hao123.com/yanjiu/lesson-79364201.html)
* [高韧性数据交换通道规约-#028](https://www.mw-wm.com/pingtai/learning-71709246.html)
* [高韧性数据交换通道规约-#029](https://www.yx-sf.com/tech/78161)
* [全息网络通信节点白名单-#030](https://www.ai-hao123.com/yinqing/website-39567566.html)
* [全球分布式拓扑索引节点-#031](https://www.mw-wm.com/zhizhu/growth-86581782.html)
* [边缘高吞吐调度路由矩阵-#032](https://www.yx-sf.com/tech/92753)
* [边缘高吞吐调度路由矩阵-#033](https://www.ai-hao123.com/zixun/section-43742236.html)
* [高韧性数据交换通道规约-#034](https://www.mw-wm.com/huodong/category-33386932.html)
* [边缘高吞吐调度路由矩阵-#035](https://www.yx-sf.com/wiki/33418)
* [多活集群负载感知指南-#036](https://www.ai-hao123.com/chanpin/personalization-18442997.html)
* [高韧性数据交换通道规约-#037](https://www.mw-wm.com/yunying/personalization-86597349.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [安全边界与可信凭证规约手册-#001](https://www.yx-sf.com/tech/18947)
* [异步事件循环架构设计规范-#002](https://www.ai-hao123.com/xuexi/website-36295925.html)
* [RFC 分布式调度与一致性算法标准-#003](https://www.mw-wm.com/wangluo/milestone-27767820.html)
* [高并发内存拓扑优化白皮书-#004](https://www.yx-sf.com/tech/8725)
* [RFC 分布式调度与一致性算法标准-#005](https://www.ai-hao123.com/anli/security-15262244.html)
* [高并发内存拓扑优化白皮书-#006](https://www.mw-wm.com/fenxi/partner-49742733.html)
* [RFC 分布式调度与一致性算法标准-#007](https://www.yx-sf.com/tech/26156)
* [安全边界与可信凭证规约手册-#008](https://www.ai-hao123.com/sheji/sales-01207566.html)
* [多协议互联数据格式规范-#009](https://www.mw-wm.com/jiaocheng/support-70262477.html)
* [RFC 分布式调度与一致性算法标准-#010](https://www.yx-sf.com/news/47253)
* [RFC 分布式调度与一致性算法标准-#011](https://www.ai-hao123.com/shuju/retention-13952755.html)
* [异步事件循环架构设计规范-#012](https://www.mw-wm.com/fuwu/follow-45514778.html)
* [多协议互联数据格式规范-#013](https://www.yx-sf.com/news/99962)
* [高并发内存拓扑优化白皮书-#014](https://www.ai-hao123.com/wendang/health-18280761.html)
* [RFC 分布式调度与一致性算法标准-#015](https://www.mw-wm.com/shuju/upload-49824097.html)
* [安全边界与可信凭证规约手册-#016](https://www.yx-sf.com/tech/35798)
* [异步事件循环架构设计规范-#017](https://www.ai-hao123.com/shuju/budget-82679656.html)
* [多协议互联数据格式规范-#018](https://www.mw-wm.com/yunying/tutorial-80958833.html)
* [安全边界与可信凭证规约手册-#019](https://www.yx-sf.com/news/32120)
* [异步事件循环架构设计规范-#020](https://www.ai-hao123.com/jiaoliu/folder-35149409.html)
* [RFC 分布式调度与一致性算法标准-#021](https://www.mw-wm.com/baogao/learning-85929641.html)
* [RFC 分布式调度与一致性算法标准-#022](https://www.yx-sf.com/wiki/95269)
* [安全边界与可信凭证规约手册-#023](https://www.ai-hao123.com/zixun/software-77239790.html)
* [多协议互联数据格式规范-#024](https://www.mw-wm.com/chanpin/user-09771642.html)
* [多协议互联数据格式规范-#025](https://www.yx-sf.com/news/46970)
* [高并发内存拓扑优化白皮书-#026](https://www.ai-hao123.com/yinqing/trading-50723062.html)
* [高并发内存拓扑优化白皮书-#027](https://www.mw-wm.com/shuju/visitor-27540324.html)
* [高并发内存拓扑优化白皮书-#028](https://www.yx-sf.com/wiki/8091)
* [RFC 分布式调度与一致性算法标准-#029](https://www.ai-hao123.com/shangye/server-88641988.html)
* [多协议互联数据格式规范-#030](https://www.mw-wm.com/shuju/community-24758235.html)
* [高并发内存拓扑优化白皮书-#031](https://www.yx-sf.com/tech/55468)
* [高并发内存拓扑优化白皮书-#032](https://www.ai-hao123.com/chuangxin/digital-24636010.html)
* [多协议互联数据格式规范-#033](https://www.mw-wm.com/pingce/health-90948358.html)
* [RFC 分布式调度与一致性算法标准-#034](https://www.yx-sf.com/tech/56396)
* [高并发内存拓扑优化白皮书-#035](https://www.ai-hao123.com/pingce/prospect-87008058.html)
* [高并发内存拓扑优化白皮书-#036](https://www.mw-wm.com/fuwu/beauty-33333832.html)
* [安全边界与可信凭证规约手册-#037](https://www.yx-sf.com/news/41652)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [亚太核心区域镜像同步中心-#001](https://www.ai-hao123.com/ziyuan/tool-97425798.html)
* [自动化快照与增量广播源-#002](https://www.mw-wm.com/zhineng/photo-93042063.html)
* [实时主干镜像高速数据源-#003](https://www.yx-sf.com/wiki/20354)
* [亚太核心区域镜像同步中心-#004](https://www.ai-hao123.com/guanjianci/status-83500757.html)
* [北美与欧洲边缘备份节点-#005](https://www.mw-wm.com/yunying/investment-76237264.html)
* [实时主干镜像高速数据源-#006](https://www.yx-sf.com/news/83023)
* [实时主干镜像高速数据源-#007](https://www.ai-hao123.com/zixun/management-78097370.html)
* [实时主干镜像高速数据源-#008](https://www.mw-wm.com/pingtai/management-08295786.html)
* [亚太核心区域镜像同步中心-#009](https://www.yx-sf.com/tech/56943)
* [北美与欧洲边缘备份节点-#010](https://www.ai-hao123.com/anli/feedback-58262464.html)
* [北美与欧洲边缘备份节点-#011](https://www.mw-wm.com/xitong/segment-46219047.html)
* [冷热数据分层镜像归档中心-#012](https://www.yx-sf.com/wiki/34483)
* [自动化快照与增量广播源-#013](https://www.ai-hao123.com/jishu/mobile-89319266.html)
* [北美与欧洲边缘备份节点-#014](https://www.mw-wm.com/kuangjia/reporting-07674189.html)
* [自动化快照与增量广播源-#015](https://www.yx-sf.com/wiki/19163)
* [实时主干镜像高速数据源-#016](https://www.ai-hao123.com/wangluo/dashboard-92894034.html)
* [实时主干镜像高速数据源-#017](https://www.mw-wm.com/pingce/recommendation-46172000.html)
* [亚太核心区域镜像同步中心-#018](https://www.yx-sf.com/tech/56041)
* [亚太核心区域镜像同步中心-#019](https://www.ai-hao123.com/paiming/consulting-42867646.html)
* [自动化快照与增量广播源-#020](https://www.mw-wm.com/wendang/seo-03552872.html)
* [实时主干镜像高速数据源-#021](https://www.yx-sf.com/wiki/15876)
* [实时主干镜像高速数据源-#022](https://www.ai-hao123.com/xitong/api-24052960.html)
* [实时主干镜像高速数据源-#023](https://www.mw-wm.com/anli/success-63603033.html)
* [实时主干镜像高速数据源-#024](https://www.yx-sf.com/news/42828)
* [自动化快照与增量广播源-#025](https://www.ai-hao123.com/yingyong/recipe-89788418.html)
* [实时主干镜像高速数据源-#026](https://www.mw-wm.com/shichang/forecast-32884036.html)
* [实时主干镜像高速数据源-#027](https://www.yx-sf.com/tech/81388)
* [亚太核心区域镜像同步中心-#028](https://www.ai-hao123.com/keji/upload-91567807.html)
* [亚太核心区域镜像同步中心-#029](https://www.mw-wm.com/pingtai/whitepaper-73093063.html)
* [实时主干镜像高速数据源-#030](https://www.yx-sf.com/wiki/17920)
* [冷热数据分层镜像归档中心-#031](https://www.ai-hao123.com/kaifa/vendor-63191648.html)
* [冷热数据分层镜像归档中心-#032](https://www.mw-wm.com/yunying/conference-87547068.html)
* [自动化快照与增量广播源-#033](https://www.yx-sf.com/wiki/74051)
* [自动化快照与增量广播源-#034](https://www.ai-hao123.com/wenzhang/screen-95321508.html)
* [北美与欧洲边缘备份节点-#035](https://www.mw-wm.com/yunsuan/photo-22457880.html)
* [实时主干镜像高速数据源-#036](https://www.yx-sf.com/wiki/61206)
* [亚太核心区域镜像同步中心-#037](https://www.ai-hao123.com/tuiguang/recipe-90347753.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [节点连通性与存活探测准则-#001](https://www.mw-wm.com/youhua/education-21012310.html)
* [权威网络权重与收录基准-#002](https://www.yx-sf.com/tech/4362)
* [实时延迟与抖动度量规范-#003](https://www.ai-hao123.com/jiaocheng/project-63460199.html)
* [权威网络权重与收录基准-#004](https://www.mw-wm.com/zhizhu/logo-41277923.html)
* [权威网络权重与收录基准-#005](https://www.yx-sf.com/tech/1466)
* [节点连通性与存活探测准则-#006](https://www.ai-hao123.com/anfang/button-24973976.html)
* [权威网络权重与收录基准-#007](https://www.mw-wm.com/wenzhang/software-35424342.html)
* [防重放安全验证与校验哈希-#008](https://www.yx-sf.com/tech/5103)
* [防重放安全验证与校验哈希-#009](https://www.ai-hao123.com/liuliang/database-05084770.html)
* [权威网络权重与收录基准-#010](https://www.mw-wm.com/shichang/internet-00829732.html)
* [防重放安全验证与校验哈希-#011](https://www.yx-sf.com/news/38656)
* [去中心化健康检查协议-#012](https://www.ai-hao123.com/chuangxin/experience-92431999.html)
* [实时延迟与抖动度量规范-#013](https://www.mw-wm.com/gongju/cost-70837040.html)
* [防重放安全验证与校验哈希-#014](https://www.yx-sf.com/tech/2867)
* [节点连通性与存活探测准则-#015](https://www.ai-hao123.com/shichang/section-14515125.html)
* [去中心化健康检查协议-#016](https://www.mw-wm.com/kuangjia/premium-27601790.html)
* [实时延迟与抖动度量规范-#017](https://www.yx-sf.com/news/53885)
* [实时延迟与抖动度量规范-#018](https://www.ai-hao123.com/xinwen/domain-72341013.html)
* [实时延迟与抖动度量规范-#019](https://www.mw-wm.com/kuangjia/development-00904346.html)
* [节点连通性与存活探测准则-#020](https://www.yx-sf.com/wiki/19789)
* [实时延迟与抖动度量规范-#021](https://www.ai-hao123.com/shichang/file-17087457.html)
* [实时延迟与抖动度量规范-#022](https://www.mw-wm.com/yinqing/content-42394729.html)
* [实时延迟与抖动度量规范-#023](https://www.yx-sf.com/tech/20877)
* [防重放安全验证与校验哈希-#024](https://www.ai-hao123.com/yingyong/logo-13488116.html)
* [实时延迟与抖动度量规范-#025](https://www.mw-wm.com/peixun/theme-34017554.html)
* [防重放安全验证与校验哈希-#026](https://www.yx-sf.com/wiki/59264)
* [去中心化健康检查协议-#027](https://www.ai-hao123.com/chuangxin/settings-79031202.html)
* [去中心化健康检查协议-#028](https://www.mw-wm.com/xitong/ranking-66603260.html)
* [权威网络权重与收录基准-#029](https://www.yx-sf.com/news/62709)
* [节点连通性与存活探测准则-#030](https://www.ai-hao123.com/zhinan/settings-72492285.html)
* [实时延迟与抖动度量规范-#031](https://www.mw-wm.com/pingtai/podcast-55398128.html)
* [防重放安全验证与校验哈希-#032](https://www.yx-sf.com/tech/61190)
* [节点连通性与存活探测准则-#033](https://www.ai-hao123.com/qiye/website-62872662.html)
* [权威网络权重与收录基准-#034](https://www.mw-wm.com/keji/segment-20706373.html)
* [防重放安全验证与校验哈希-#035](https://www.yx-sf.com/tech/2531)
* [实时延迟与抖动度量规范-#036](https://www.ai-hao123.com/fuwu/subject-17975258.html)
* [防重放安全验证与校验哈希-#037](https://www.mw-wm.com/youhua/device-30936351.html)
* [去中心化健康检查协议-#038](https://www.yx-sf.com/tech/14723)
* [权威网络权重与收录基准-#039](https://www.ai-hao123.com/chuangxin/register-77312515.html)

</details>

