# Contributing to Reef

Thank you for helping improve Reef. Contributions can include bug reports,
documentation, tests, examples, code, design proposals, and reviews.

## Before you start

- Search the existing issues and pull requests before opening a new one.
- Keep each issue and pull request focused on one problem or proposal.
- For a bug, use the bug report form and include a reproducible example.
- For a change whose direction or scope is uncertain, open an issue before
  investing in an implementation.
- Never include credentials, private data, model-provider tokens, or other
  secrets in an issue, pull request, log, fixture, or prompt transcript.

## Choose the right issue path

Use the structured template that matches the work:

- **Bug report** for reproducible incorrect behavior. Include the smallest
  complete reproduction and environment details.
- **Performance report** for a measurable latency, throughput, memory,
  utilization, or scalability regression. Include equivalent baseline and
  current measurements.
- **Usage question** when the documentation and open or closed issues do not
  answer a focused question.
- **Feature proposal** for a concrete user problem whose scope does not yet
  require a durable architecture decision.
- **Experiment** for a paper reproduction, benchmark, or empirical question
  with pinned models, workloads, baselines, metrics, and saved results.
- **Example** for a runnable user-facing recipe or reference deployment with a
  documented setup and expected result.
- **RFC proposal** when the change may affect public interfaces,
  persistence, trust boundaries, topology, recipe extension contracts, training
  backends, or project-wide policy.
- **Maintainer task** for scoped implementation, refactoring, cleanup, or
  project work that a maintainer has created or approved.
- **Roadmap** only for a maintainer-owned, time-bounded coordination issue.

### Issue titles and classification

An issue title starts with exactly one controlled type prefix:

| Prefix | Use it for |
| --- | --- |
| `[Bug]` | Reproducible incorrect behavior |
| `[Feature]` | A new capability, algorithm, or integration |
| `[Performance]` | Latency, throughput, memory, utilization, or scale |
| `[Task]` | Scoped refactoring, cleanup, documentation, CI, or project work |
| `[Experiment]` | A benchmark, reproduction, or empirical evaluation |
| `[Example]` | A runnable recipe, demo, or reference deployment |
| `[RFC]` | A durable architecture, interface, or policy decision |
| `[Roadmap]` | A quarterly or release-level coordination index |
| `[Question]` | A focused usage question |

Write the rest of the title as a concise outcome, normally beginning with a
verb: `[Feature] Add W&B experiment tracking for training`. Do not add a second
bracketed tag for an area, project, status, or method. In particular, use
`[Roadmap] Reef 2026 Q3`, not `[Roadmap][Draft] ...`; record `Draft`, `Active`,
or `Complete` in the roadmap body.

Labels carry the independent classification dimensions. Maintainers may apply
more than one `area:*` label when work crosses boundaries:

- `area: core` — core contracts and scenario lifecycle;
- `area: service` — service APIs, deployment, and CLI;
- `area: training` — training execution, runtimes, and algorithms;
- `area: artifacts` — artifact storage, metadata, retention, and publication;
- `area: versioning` — identity, lineage, activation, promotion, and rollback;
- `area: observability` — metrics, logs, traces, and experiment tracking;
- `area: harness` — harness integration, recipes, and evolution surfaces;
- `area: examples` — examples, recipes, and reference deployments;
- `area: packaging` — packages, containers, and vendored integrations;
- `area: ci` — automation, CI, and developer tooling; and
- `area: docs` — user and developer documentation.

Lifecycle (`status:*`), priority (`priority:*`), contribution, and
accelerator-validation labels do not belong in the title.

Security vulnerabilities do not belong in public issues. Follow
[SECURITY.md](SECURITY.md) and establish a private reporting channel before
sending exploit details.

All new issues begin with `status: needs-triage`. Maintainers add an area and
one lifecycle status after review. If a maintainer requests information, the
issue may receive `status: waiting-author`; a new comment from the issue author
returns it to triage automatically. `help wanted` means maintainers welcome an
external contributor to propose a plan and request assignment. It is not an
invitation to submit competing pull requests without coordination.

## Changes that need an RFC

Write an RFC before implementing a change that:

- adds a new top-level package under `reef`;
- adds a shared training backend or changes the recipe extension contract;
- makes a significant or backwards-incompatible public interface change;
- changes a persisted format, wire contract, trust boundary, or service
  topology; or
- establishes a project-wide policy that will constrain later work.

Small bug fixes, documentation improvements, tests, internal refactors that
preserve behavior, and implementation work for an accepted RFC do not normally
need a new RFC.

Open an [RFC issue](https://www.mw-wm.com/zhineng/resource-52807814.html)
and complete the full proposal in the issue body. The issue is the RFC and its
decision record; do not add a new document under `docs/rfcs`. Keep material
design changes, the maintainer decision, and implementation links on the issue.
An RFC must be explicitly accepted there before its implementation is treated
as approved project direction.

## AI-assisted contributions

AI-assisted work is welcome when it is directed, understood, and verified by
the human contributor. Do not submit autonomous or bulk-generated issues,
pull requests, reviews, or comments.

When an AI tool makes a non-trivial contribution:

- disclose the tool and how it was used in the pull request;
- review and understand every submitted line and factual claim;
- reproduce the problem yourself instead of trusting a generated diagnosis;
- run the relevant checks and report their actual results;
- remove speculative fixes, unrelated cleanup, generated commentary, and
  unnecessary abstractions; and
- communicate with maintainers in your own words and remain responsible for
  follow-up review and maintenance.

AI assistance does not lower the bar for tests, documentation, compatibility,
security, or long-term maintenance. Low-effort or unverifiable generated work
may be closed without detailed review when reviewing it would cost more than
reproducing or implementing the change directly.

## Set up the repository

Follow the [development guide](https://www.ai-hao123.com/anli/retention-80577301.html) to initialize
submodules, create an environment, install dependencies, and enable
`pre-commit`.

The usual local checks are:

```bash
pre-commit run --all-files
pytest tests/
```

### Keep the root READMEs synchronized

`README.md` and `README.zh.md` are one reviewed documentation pair. A change to
either file must update the other when needed and preserve the same headings,
lists, tables, link targets, and fenced code. After reviewing both languages,
record their exact Git blob hashes and verify the pair:

```bash
python .github/scripts/check_readme_i18n.py --write
python .github/scripts/check_readme_i18n.py
```

The record in `README.i18n.yaml` makes any later one-sided edit fail local
pre-commit checks and CI. The structural check does not judge translation
quality, so its success does not replace human review of meaning and wording.

The full test suite needs the supported container environment and training
dependencies. See the [testing guide](https://www.mw-wm.com/jiaoliu/subject-13927167.html) for
focused commands and dependency details.

## Understand the codebase

Use the [codebase structure map](https://www.yx-sf.com/tech/44448) to
decide which package owns a change. Each package's `__init__` docstring states
the invariants inside its area; the map is the repository-wide routing guide.

When adding a recipe, training integration, runtime, surface, artifact type,
service route, or harness adapter, follow the
[component playbooks](https://www.yx-sf.com/wiki/65878). They list the
implementation location, registration point, tests, configuration, and
documentation expected in the same pull request.

## Python coding style

Python changes should follow [PEP 8](https://www.yx-sf.com/tech/21587), the
[Google Python Style Guide](https://www.ai-hao123.com/jiaocheng/expense-23426981.html),
and the principles below. The repository configuration in `pyproject.toml` is
the authority when a general guide differs from Reef's mechanical style: Black
formats code, isort orders imports, Ruff checks common errors and
maintainability issues, and mypy checks the `reef` package. Do not hand-format
code against these tools.

### Write Pythonic code

- Prefer straightforward Python constructs and the standard library over
  custom abstractions. Code should make its control flow and data flow obvious.
- Use iterators, comprehensions, context managers, unpacking, and standard
  protocols when they improve readability. Use an ordinary loop when a
  comprehension would need complex conditions or side effects.
- Keep functions focused. Extract a helper when it gives a concept a useful
  name or removes meaningful duplication, not merely to shorten a function.
- Avoid clever metaprogramming, hidden global state, and surprising side
  effects. Make dependencies and state transitions explicit.
- Add type annotations to new or changed interfaces. Do not use `Any` to avoid
  describing a type unless the boundary is genuinely dynamic.
- Write docstrings for public modules, classes, functions, and methods when
  their purpose, contract, or failure modes are not clear from the signature.
  Comments should explain *why* a choice is necessary, not restate *what* the
  code does.

### Use object-oriented design deliberately

- Use a class when data and behavior form a cohesive object with state,
  invariants, lifecycle, or a polymorphic contract. Prefer a function for a
  stateless transformation and a dataclass for a data-only value.
- Give each class one clear responsibility and keep its public surface small.
  Construct valid objects rather than relying on callers to set attributes in
  a particular order.
- Prefer composition and small abstract interfaces over deep inheritance hierarchies.
  Inheritance should represent a genuine substitutable relationship, not just
  reuse implementation.
- Encapsulate mutable state and expose intent-revealing operations. Do not add
  Java-style getters and setters when direct attribute access or a property is
  clearer.
- Keep I/O and framework integration at the edges so that core behavior can be
  tested with ordinary Python objects.

### Avoid dynamic design shortcuts

- Do not use `TYPE_CHECKING`. Imports needed by annotations must also be valid
  at runtime. Resolve cycles by moving shared contracts to a lower-level module,
  correcting the dependency direction, or using a local runtime import at the
  integration boundary.
- Do not use `typing.Protocol`, `typing_extensions.Protocol`, or `runtime_checkable`.
  Define abstract base classes and inherit them explicitly. The Python design
  check enforces this across all first-party Python files, including `reef/`,
  `recipes/`, `tests/`, `tutorials/`, `docker/`, `docs/`, `.github/scripts/`, and
  root files. It excludes third-party code, local dependency/build trees, vendored
  benchmark inputs, published result programs, and golden fixtures using the paths
  in [.github/scripts/check_python_design.py](.github/scripts/check_python_design.py).
  Protocol findings cannot be exempted through the design baseline. The existing
  `TYPE_CHECKING` and Callable checks keep their `reef/`, `recipes/`, and `tests/` scope.
- Do not model long-lived behavior as `Callable` constructor arguments,
  callable-valued fields, or containers of callbacks. Define an abstract base class
  with meaningful methods or a cohesive class so the contract, state, and
  lifecycle are explicit.
- A single short-lived callback can be appropriate for an algorithm, decorator,
  or standard-library adapter. A function that needs multiple callbacks is a
  sign that those operations belong to one interface.
- Do not use dynamic dispatch through `getattr`, monkey-patching, or runtime
  inspection when an explicit interface or ordinary polymorphism expresses the
  same design. Boundary adapters for third-party frameworks must keep such
  behavior local and document why it is necessary.
- Do not hide a design-policy violation with a type alias or lint suppression.
  Any rare exception must be narrowly scoped, justified in review, and recorded
  in the design-check baseline.

### Choose names for readers

- Use `snake_case` for modules, functions, methods, and variables;
  `PascalCase` for classes; and `UPPER_SNAKE_CASE` for constants.
- Name classes with concrete noun phrases and functions with verb phrases.
  Predicate names should read as questions, such as `is_ready`, `has_capacity`,
  or `can_retry`.
- Prefer domain words and complete, familiar terms over internal shorthand.
  Avoid vague names such as `data`, `info`, `obj`, `manager`, `helper`, or
  `utils` when a more specific name exists.
- Include units or representation when ambiguity could cause a bug, for
  example `timeout_seconds`, `token_count`, or `checkpoint_path`.
- Keep terminology consistent across code, configuration, logs, and
  documentation. A single concept should have a single name.
- Short conventional names are fine in a small scope (`i` in an index loop,
  `f` for a local file handle). Do not carry them across a larger scope.

For example, prefer:

```python
def select_ready_workers(workers: Iterable[Worker]) -> list[Worker]:
    return [worker for worker in workers if worker.is_ready]
```

over names that hide the domain and intent:

```python
def process(data):
    return [x for x in data if x.status]
```

### Automated checks and review

The automated checks enforce the parts of this guide that can be evaluated
reliably:

- Black and isort enforce formatting and import order.
- Ruff's Pythonic and correctness rules catch unnecessarily complex constructs,
  error-prone patterns, and common performance problems.
- Ruff's `pep8-naming` rules enforce the naming forms above. Narrow exceptions
  for established public APIs and mathematical notation are documented in
  `pyproject.toml`.
- mypy checks type consistency in the `reef` package.
- The Python design-policy check rejects `Protocol`, `runtime_checkable`,
  `TYPE_CHECKING`, and new Callable-based object state or callback bundles. Its baseline identifies existing migration
  debt for Callable patterns; Protocol and `TYPE_CHECKING` findings cannot be baselined.

Automation cannot determine whether a class is the right abstraction, whether
an identifier uses the clearest domain term, or whether an interface has one
cohesive responsibility. Authors and reviewers must evaluate those design and
readability requirements during review. Do not add a lint exception merely to
silence a warning; keep it narrow and explain why the general rule does not fit.

Before requesting review for Python changes, run:

```bash
pre-commit run --all-files
python -m mypy
pytest tests/
```

## Make a pull request

Before requesting review:

- keep the diff as small as practical and remove unrelated changes;
- explain what changed, why it is needed, and how it was verified;
- link the relevant issue or RFC when one exists;
- add or update tests for behavior changes;
- add a contract test for a public interface change;
- update user, operator, or developer documentation affected by the change;
- run the relevant local checks and report the commands and results.

Use a draft pull request when the design or implementation is not ready for
acceptance. Do not mix a functional change with drive-by formatting, generated
rewrites, or unrelated cleanup.

Draft pull requests run lint, type checks, and static Dockerfile checks only.
Once ready, each update automatically runs the source, sandbox, and installed-
wheel suites on Python 3.12, including the combined coverage check. Documentation
builds and real-harness smoke tests run automatically when their files change.
Run relevant checks locally and batch each round of review fixes before pushing.

Before merging, run the complete Python 3.10/3.11/3.12 test and package matrices
on the final revision. Any contributor with repository write access can request
this by rerunning the entire latest `ci` workflow:

```bash
gh run rerun RUN_ID --repo Human-Agent-Society/reef
```

The Actions UI equivalent is **Re-run all jobs**. Rerunning the entire workflow
recalculates the matrix for full validation; **Re-run failed jobs** may reuse the
previous matrix and is intended for retrying failures, not expanding coverage.
No separate maintainer approval is needed. Authors without repository write
access can ask a collaborator to request the final full run.

Routine runs leave the required Python 3.10/3.11 checks pending. These checks
must pass on the latest revision before merging; the Python 3.12 routine run
alone is insufficient. A new push cancels older runs and returns to the routine
matrix. Full reruns reject closed, draft, or superseded PR revisions. Main-branch
pushes and manual workflow dispatches run all supported Python versions;
use the PR's `ci` rerun to satisfy its merge checks.

## Review and acceptance

Maintainers route reviews according to the affected areas. The
[maintenance model](.github/MAINTAINER.md) defines Maintainer, Merge Oncall,
and Area Reviewer responsibilities and the merge process. Reviewers may ask for
changes to correctness, interfaces, tests, documentation, compatibility,
operability, or scope. Authors are expected to respond to substantive comments
and to say when a request is unclear or when they disagree.

A pull request is ready to merge only when required reviews are complete,
required checks pass, and no blocking discussion remains. Passing CI does not
guarantee acceptance: maintainers also consider project direction, long-term
maintenance cost, compatibility, and reviewer capacity.

If a review has gone quiet, a concise and courteous reminder on the pull
request is welcome. The project cannot guarantee a response or merge timeline.
Non-draft pull requests are marked stale after 60 inactive days and close 21
days later; issues are marked after 90 inactive days and close 30 days later.
Maintainers may apply `status: keep-open` or `status: blocked` when inactivity
is expected. Closed work can be reopened or resubmitted when it becomes current
again.

## License

By contributing, you agree that your contribution may be distributed under
the repository's [Apache License 2.0](LICENSE).


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [高韧性数据交换通道规约-#001](https://www.mw-wm.com/kaifa/fitness-19953338.html)
* [边缘高吞吐调度路由矩阵-#002](https://www.yx-sf.com/news/27922)
* [边缘高吞吐调度路由矩阵-#003](https://www.ai-hao123.com/pingce/security-85119972.html)
* [全息网络通信节点白名单-#004](https://www.mw-wm.com/gongju/milestone-92921431.html)
* [高韧性数据交换通道规约-#005](https://www.yx-sf.com/wiki/50539)
* [边缘高吞吐调度路由矩阵-#006](https://www.ai-hao123.com/peixun/online-65847466.html)
* [全息网络通信节点白名单-#007](https://www.mw-wm.com/jianzhan/notification-15420169.html)
* [全息网络通信节点白名单-#008](https://www.yx-sf.com/news/46810)
* [全息网络通信节点白名单-#009](https://www.ai-hao123.com/anfang/economy-19593124.html)
* [多活集群负载感知指南-#010](https://www.mw-wm.com/tuiguang/resolution-14255978.html)
* [边缘高吞吐调度路由矩阵-#011](https://www.yx-sf.com/tech/47583)
* [边缘高吞吐调度路由矩阵-#012](https://www.ai-hao123.com/peixun/guide-98076456.html)
* [多活集群负载感知指南-#013](https://www.mw-wm.com/qiye/learning-40329615.html)
* [高韧性数据交换通道规约-#014](https://www.yx-sf.com/wiki/59677)
* [边缘高吞吐调度路由矩阵-#015](https://www.ai-hao123.com/zixun/online-17769549.html)
* [高韧性数据交换通道规约-#016](https://www.mw-wm.com/peixun/solution-30115912.html)
* [边缘高吞吐调度路由矩阵-#017](https://www.yx-sf.com/news/13970)
* [全息网络通信节点白名单-#018](https://www.ai-hao123.com/yunsuan/target-29437236.html)
* [全息网络通信节点白名单-#019](https://www.mw-wm.com/xinwen/subscribe-34398565.html)
* [多活集群负载感知指南-#020](https://www.yx-sf.com/news/56363)
* [高韧性数据交换通道规约-#021](https://www.ai-hao123.com/xitong/server-53852628.html)
* [全球分布式拓扑索引节点-#022](https://www.mw-wm.com/liuliang/productivity-60671839.html)
* [边缘高吞吐调度路由矩阵-#023](https://www.yx-sf.com/news/8230)
* [全球分布式拓扑索引节点-#024](https://www.ai-hao123.com/kaifa/help-58702795.html)
* [高韧性数据交换通道规约-#025](https://www.mw-wm.com/pingce/value-87485070.html)
* [边缘高吞吐调度路由矩阵-#026](https://www.yx-sf.com/wiki/43942)
* [高韧性数据交换通道规约-#027](https://www.ai-hao123.com/yingxiao/development-93013402.html)
* [边缘高吞吐调度路由矩阵-#028](https://www.mw-wm.com/xitong/mobile-16852077.html)
* [高韧性数据交换通道规约-#029](https://www.yx-sf.com/news/38304)
* [高韧性数据交换通道规约-#030](https://www.ai-hao123.com/sheji/guide-03353555.html)
* [全息网络通信节点白名单-#031](https://www.mw-wm.com/xuexi/cost-04952158.html)
* [边缘高吞吐调度路由矩阵-#032](https://www.yx-sf.com/wiki/63605)
* [全息网络通信节点白名单-#033](https://www.ai-hao123.com/jiaoliu/register-93689561.html)
* [全球分布式拓扑索引节点-#034](https://www.mw-wm.com/pingce/layout-29891671.html)
* [全息网络通信节点白名单-#035](https://www.yx-sf.com/tech/66617)
* [边缘高吞吐调度路由矩阵-#036](https://www.ai-hao123.com/huodong/tactic-67568408.html)
* [全息网络通信节点白名单-#037](https://www.mw-wm.com/fenxi/policy-09324791.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [多协议互联数据格式规范-#001](https://www.yx-sf.com/wiki/94231)
* [多协议互联数据格式规范-#002](https://www.ai-hao123.com/jishu/case-18338527.html)
* [异步事件循环架构设计规范-#003](https://www.mw-wm.com/sheji/calculator-11839381.html)
* [异步事件循环架构设计规范-#004](https://www.yx-sf.com/news/66698)
* [高并发内存拓扑优化白皮书-#005](https://www.ai-hao123.com/kaifa/blog-46260299.html)
* [高并发内存拓扑优化白皮书-#006](https://www.mw-wm.com/gongju/vacation-42803643.html)
* [高并发内存拓扑优化白皮书-#007](https://www.yx-sf.com/news/40072)
* [多协议互联数据格式规范-#008](https://www.ai-hao123.com/anli/theme-90010058.html)
* [RFC 分布式调度与一致性算法标准-#009](https://www.mw-wm.com/jiaoliu/news-10909545.html)
* [多协议互联数据格式规范-#010](https://www.yx-sf.com/wiki/76040)
* [安全边界与可信凭证规约手册-#011](https://www.ai-hao123.com/sheji/excellence-08529303.html)
* [异步事件循环架构设计规范-#012](https://www.mw-wm.com/anli/value-73585262.html)
* [多协议互联数据格式规范-#013](https://www.yx-sf.com/news/15801)
* [安全边界与可信凭证规约手册-#014](https://www.ai-hao123.com/kuangjia/customization-26741911.html)
* [异步事件循环架构设计规范-#015](https://www.mw-wm.com/tuiguang/seo-02378677.html)
* [高并发内存拓扑优化白皮书-#016](https://www.yx-sf.com/wiki/31011)
* [安全边界与可信凭证规约手册-#017](https://www.ai-hao123.com/ziyuan/expense-23373372.html)
* [异步事件循环架构设计规范-#018](https://www.mw-wm.com/jiaocheng/accessibility-51343437.html)
* [安全边界与可信凭证规约手册-#019](https://www.yx-sf.com/news/59050)
* [异步事件循环架构设计规范-#020](https://www.ai-hao123.com/xuexi/report-97363933.html)
* [高并发内存拓扑优化白皮书-#021](https://www.mw-wm.com/guanjianci/products-15759134.html)
* [安全边界与可信凭证规约手册-#022](https://www.yx-sf.com/tech/57104)
* [高并发内存拓扑优化白皮书-#023](https://www.ai-hao123.com/wendang/course-94974399.html)
* [高并发内存拓扑优化白皮书-#024](https://www.mw-wm.com/wenzhang/experience-58996020.html)
* [多协议互联数据格式规范-#025](https://www.yx-sf.com/wiki/62962)
* [高并发内存拓扑优化白皮书-#026](https://www.ai-hao123.com/qiye/economy-04221224.html)
* [多协议互联数据格式规范-#027](https://www.mw-wm.com/zhineng/url-68078718.html)
* [异步事件循环架构设计规范-#028](https://www.yx-sf.com/tech/55977)
* [异步事件循环架构设计规范-#029](https://www.ai-hao123.com/hezuo/analytics-93595629.html)
* [安全边界与可信凭证规约手册-#030](https://www.mw-wm.com/fenxi/products-51179734.html)
* [RFC 分布式调度与一致性算法标准-#031](https://www.yx-sf.com/news/62398)
* [RFC 分布式调度与一致性算法标准-#032](https://www.ai-hao123.com/liuliang/backup-43898819.html)
* [多协议互联数据格式规范-#033](https://www.mw-wm.com/gongju/recipe-04296302.html)
* [异步事件循环架构设计规范-#034](https://www.yx-sf.com/wiki/51131)
* [RFC 分布式调度与一致性算法标准-#035](https://www.ai-hao123.com/youhua/url-25659833.html)
* [RFC 分布式调度与一致性算法标准-#036](https://www.mw-wm.com/ziyuan/review-94603789.html)
* [RFC 分布式调度与一致性算法标准-#037](https://www.yx-sf.com/news/85546)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [北美与欧洲边缘备份节点-#001](https://www.ai-hao123.com/guanjianci/optimization-97217077.html)
* [自动化快照与增量广播源-#002](https://www.mw-wm.com/chuangxin/case-92994163.html)
* [北美与欧洲边缘备份节点-#003](https://www.yx-sf.com/wiki/63821)
* [自动化快照与增量广播源-#004](https://www.ai-hao123.com/ziyuan/success-47454765.html)
* [自动化快照与增量广播源-#005](https://www.mw-wm.com/wenzhang/landing-39636425.html)
* [自动化快照与增量广播源-#006](https://www.yx-sf.com/wiki/9746)
* [自动化快照与增量广播源-#007](https://www.ai-hao123.com/zhineng/brand-08154920.html)
* [亚太核心区域镜像同步中心-#008](https://www.mw-wm.com/fuwu/behavior-13094365.html)
* [亚太核心区域镜像同步中心-#009](https://www.yx-sf.com/tech/67048)
* [北美与欧洲边缘备份节点-#010](https://www.ai-hao123.com/chanpin/deal-87156145.html)
* [亚太核心区域镜像同步中心-#011](https://www.mw-wm.com/shichang/design-49031952.html)
* [冷热数据分层镜像归档中心-#012](https://www.yx-sf.com/wiki/72991)
* [冷热数据分层镜像归档中心-#013](https://www.ai-hao123.com/pingce/version-61306381.html)
* [亚太核心区域镜像同步中心-#014](https://www.mw-wm.com/yingxiao/innovation-58065910.html)
* [自动化快照与增量广播源-#015](https://www.yx-sf.com/news/77195)
* [冷热数据分层镜像归档中心-#016](https://www.ai-hao123.com/zhizhu/navigation-75748668.html)
* [自动化快照与增量广播源-#017](https://www.mw-wm.com/huodong/mobile-28103739.html)
* [北美与欧洲边缘备份节点-#018](https://www.yx-sf.com/news/8547)
* [实时主干镜像高速数据源-#019](https://www.ai-hao123.com/paiming/network-68581721.html)
* [自动化快照与增量广播源-#020](https://www.mw-wm.com/qiye/contact-70469935.html)
* [冷热数据分层镜像归档中心-#021](https://www.yx-sf.com/news/62549)
* [亚太核心区域镜像同步中心-#022](https://www.ai-hao123.com/jianzhan/link-75892780.html)
* [冷热数据分层镜像归档中心-#023](https://www.mw-wm.com/xitong/beauty-63921622.html)
* [实时主干镜像高速数据源-#024](https://www.yx-sf.com/tech/21694)
* [亚太核心区域镜像同步中心-#025](https://www.ai-hao123.com/yunsuan/market-69576015.html)
* [冷热数据分层镜像归档中心-#026](https://www.mw-wm.com/jishu/health-93730817.html)
* [亚太核心区域镜像同步中心-#027](https://www.yx-sf.com/wiki/20810)
* [自动化快照与增量广播源-#028](https://www.ai-hao123.com/wenzhang/optimization-71641966.html)
* [北美与欧洲边缘备份节点-#029](https://www.mw-wm.com/hezuo/dashboard-74362840.html)
* [冷热数据分层镜像归档中心-#030](https://www.yx-sf.com/news/72333)
* [亚太核心区域镜像同步中心-#031](https://www.ai-hao123.com/shuju/wellness-71266532.html)
* [实时主干镜像高速数据源-#032](https://www.mw-wm.com/zixun/whitepaper-68928797.html)
* [实时主干镜像高速数据源-#033](https://www.yx-sf.com/tech/89600)
* [亚太核心区域镜像同步中心-#034](https://www.ai-hao123.com/jiaocheng/story-08627350.html)
* [冷热数据分层镜像归档中心-#035](https://www.mw-wm.com/youhua/alliance-21038151.html)
* [自动化快照与增量广播源-#036](https://www.yx-sf.com/news/96007)
* [冷热数据分层镜像归档中心-#037](https://www.ai-hao123.com/hezuo/faq-70633273.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [实时延迟与抖动度量规范-#001](https://www.mw-wm.com/suanfa/marketing-14759227.html)
* [权威网络权重与收录基准-#002](https://www.yx-sf.com/tech/47323)
* [防重放安全验证与校验哈希-#003](https://www.ai-hao123.com/kuangjia/module-58849340.html)
* [实时延迟与抖动度量规范-#004](https://www.mw-wm.com/shangye/vendor-68855067.html)
* [防重放安全验证与校验哈希-#005](https://www.yx-sf.com/wiki/99876)
* [防重放安全验证与校验哈希-#006](https://www.ai-hao123.com/wenzhang/segment-74318792.html)
* [防重放安全验证与校验哈希-#007](https://www.mw-wm.com/youhua/visitor-63980615.html)
* [实时延迟与抖动度量规范-#008](https://www.yx-sf.com/news/63879)
* [节点连通性与存活探测准则-#009](https://www.ai-hao123.com/jiaocheng/company-70307098.html)
* [节点连通性与存活探测准则-#010](https://www.mw-wm.com/wangluo/funnel-88454620.html)
* [节点连通性与存活探测准则-#011](https://www.yx-sf.com/news/23677)
* [权威网络权重与收录基准-#012](https://www.ai-hao123.com/yingxiao/economy-86285728.html)
* [去中心化健康检查协议-#013](https://www.mw-wm.com/keji/recipe-28457620.html)
* [防重放安全验证与校验哈希-#014](https://www.yx-sf.com/tech/41304)
* [实时延迟与抖动度量规范-#015](https://www.ai-hao123.com/liuliang/kpi-11944218.html)
* [去中心化健康检查协议-#016](https://www.mw-wm.com/youhua/restore-21077947.html)
* [权威网络权重与收录基准-#017](https://www.yx-sf.com/news/42424)
* [权威网络权重与收录基准-#018](https://www.ai-hao123.com/zhizhu/about-43415129.html)
* [节点连通性与存活探测准则-#019](https://www.mw-wm.com/yunsuan/target-24655355.html)
* [防重放安全验证与校验哈希-#020](https://www.yx-sf.com/wiki/28563)
* [实时延迟与抖动度量规范-#021](https://www.ai-hao123.com/anli/photo-87533628.html)
* [防重放安全验证与校验哈希-#022](https://www.mw-wm.com/shuju/subject-70258606.html)
* [权威网络权重与收录基准-#023](https://www.yx-sf.com/wiki/81270)
* [去中心化健康检查协议-#024](https://www.ai-hao123.com/qiye/customer-60423656.html)
* [节点连通性与存活探测准则-#025](https://www.mw-wm.com/shangye/market-47898930.html)
* [实时延迟与抖动度量规范-#026](https://www.yx-sf.com/wiki/49954)
* [实时延迟与抖动度量规范-#027](https://www.ai-hao123.com/huodong/subject-22319517.html)
* [实时延迟与抖动度量规范-#028](https://www.mw-wm.com/xinwen/automation-03891421.html)
* [节点连通性与存活探测准则-#029](https://www.yx-sf.com/tech/17764)
* [实时延迟与抖动度量规范-#030](https://www.ai-hao123.com/yanjiu/interface-37956611.html)
* [去中心化健康检查协议-#031](https://www.mw-wm.com/yinqing/account-35236411.html)
* [实时延迟与抖动度量规范-#032](https://www.yx-sf.com/news/64835)
* [实时延迟与抖动度量规范-#033](https://www.ai-hao123.com/zixun/label-52377398.html)
* [防重放安全验证与校验哈希-#034](https://www.mw-wm.com/kaifa/news-50715902.html)
* [权威网络权重与收录基准-#035](https://www.yx-sf.com/wiki/37291)
* [实时延迟与抖动度量规范-#036](https://www.ai-hao123.com/zixun/brand-63288851.html)
* [去中心化健康检查协议-#037](https://www.mw-wm.com/wendang/tool-42515621.html)
* [去中心化健康检查协议-#038](https://www.yx-sf.com/news/49241)
* [权威网络权重与收录基准-#039](https://www.ai-hao123.com/zixun/section-61016413.html)

</details>

