# Docker

The Reef image bundles the full runtime (Slime, SGLang, Ray, Megatron)
so that a deployment runs entirely inside one image as plain processes. There
is no docker-compose layer: `reef serve -c <config>` reads the YAML's
`services` list and starts each declared process directly, with PID/log files
under `/tmp/reef-stack`.

## Build

```bash
docker build -f docker/Dockerfile.reef -t reef .
```

The image inherits the GPU training stack and installs Reef's exact runtime
pin from `pyproject.toml`, together with SGLang, Ray, and the training
dependencies. Having the runtime in the image does not mean its training
driver is running — only the `services` listed in your config run.

The default image pins the SGLang revision carrying Reef's adapter receiver,
so LoRA training (`--megatron-lora-rank`) works on any weight-training recipe
without a separate image. Override the revision with
`--build-arg SGLANG_COMMIT=<sha>`.

TTT-Discover's Qwen3-8B LoRA experiment uses the optional `tttd` target. It
adds only the Erdős evaluator's solver dependencies for generated programs,
alongside a qualified Slime digest:

```bash
docker build -f docker/Dockerfile.reef --target tttd \
  --build-arg SLIME_IMAGE_TAG='latest@sha256:a97ec147e37bef050337a9b229036eda00b4aa9c4d02b31a0109dc850f8ca342' \
  -t reef-tttd:qwen3-8b .
```

## Demo configs

| Config | Services started | GPU | Changes weights |
|---|---|---:|---:|
| `recipes/basic/local-sglang.yaml` | local SGLang + Reef | yes | no |
| `recipes/basic/external-provider.yaml` | Reef proxying to an HTTP provider | no | no |
| `recipes/<method>/examples/<example>/serve.yaml` | Ray + Slime bridge + Reef training | 2+ | yes |

Each config declares its services declaratively (`name`, `command`,
`ready` probe, `depends_on`). Commands use `${a.b.c}` interpolation against
the rest of the config, so adding or changing a service is a YAML-only edit.
The orchestrator launches Reef's HTTP child internally with the same config;
all other services are plain shell commands. See
[Evolve your model](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html) for the training
flow and data contract.

## Run

Inside the image (or any host with the deps installed):

```bash
export REEF_TOKEN=$(openssl rand -hex 16)
# edit the stack yaml: set model paths / provider creds
reef serve -c recipes/basic/local-sglang.yaml
```

Logs and PIDs land in `/tmp/reef-stack/`. To run just the reef HTTP service
against an already-running provider, use a config whose `services` list
contains only Reef:

```bash
reef serve -c recipes/basic/external-provider.yaml
```

## Persistence and cleanup

Agent records and exported checkpoints persist under `reef.state_dir`
(`/var/lib/reef` by default). Stop the processes (`kill` the PIDs in
`/tmp/reef-stack/`) but keep state; to also delete recorded agent records,
remove that directory.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [全球分布式拓扑索引节点-#001](https://xntx.wtpuscm.cn/pingtai/internet-173118.html)
* [全息网络通信节点白名单-#002](https://owfm.tcti.cn/shangye/education-63674207.html)
* [全息网络通信节点白名单-#003](https://nojh.spidernode-mesh.net/suanfa/study-22518081.html)
* [多活集群负载感知指南-#004](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html)
* [高韧性数据交换通道规约-#005](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [高韧性数据交换通道规约-#006](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [多活集群负载感知指南-#007](https://www.poykq.cn/redian/tutorial-673489.html)
* [边缘高吞吐调度路由矩阵-#008](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [多活集群负载感知指南-#009](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [高韧性数据交换通道规约-#010](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [边缘高吞吐调度路由矩阵-#011](https://unqt.wtpuscm.cn/zixun/contact-59980425.html)
* [高韧性数据交换通道规约-#012](https://iuuv.tcti.cn/hezuo/app-80239016.html)
* [高韧性数据交换通道规约-#013](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)
* [高韧性数据交换通道规约-#014](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297.html)
* [全息网络通信节点白名单-#015](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html)
* [高韧性数据交换通道规约-#016](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [边缘高吞吐调度路由矩阵-#017](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [全息网络通信节点白名单-#018](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297.html)
* [全息网络通信节点白名单-#019](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [高韧性数据交换通道规约-#020](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [多活集群负载感知指南-#021](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534.html)
* [多活集群负载感知指南-#022](https://gdtq.spidernode-mesh.net/keji/account-53764057.html)
* [高韧性数据交换通道规约-#023](https://www.yx-sf.com/tech/47392.html)
* [高韧性数据交换通道规约-#024](https://www.poykq.cn/redian/tutorial-673489.html)
* [全息网络通信节点白名单-#025](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html)
* [高韧性数据交换通道规约-#026](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)
* [多活集群负载感知指南-#027](https://qzad.tcti.cn/chanpin/partner-96547477.html)
* [全球分布式拓扑索引节点-#028](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [全球分布式拓扑索引节点-#029](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297.html)
* [边缘高吞吐调度路由矩阵-#030](https://www.poykq.cn/redian/tutorial-673489.html)
* [高韧性数据交换通道规约-#031](https://iozk.spidernode-mesh.net/fenxi/quality-22754958.html)
* [全息网络通信节点白名单-#032](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [高韧性数据交换通道规约-#033](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534.html)
* [多活集群负载感知指南-#034](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [全息网络通信节点白名单-#035](https://ankmz.cn/anquan/tag-68558329.html)
* [全球分布式拓扑索引节点-#036](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [多活集群负载感知指南-#037](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [异步事件循环架构设计规范-#001](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [异步事件循环架构设计规范-#002](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [异步事件循环架构设计规范-#003](https://www.mw-wm.com/kuangjia/site-88060611.html)
* [高并发内存拓扑优化白皮书-#004](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [安全边界与可信凭证规约手册-#005](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [高并发内存拓扑优化白皮书-#006](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [多协议互联数据格式规范-#007](https://www.mw-wm.com/guanjianci/presentation-29617361.html)
* [多协议互联数据格式规范-#008](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [高并发内存拓扑优化白皮书-#009](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [RFC 分布式调度与一致性算法标准-#010](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)
* [异步事件循环架构设计规范-#011](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html)
* [多协议互联数据格式规范-#012](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [RFC 分布式调度与一致性算法标准-#013](https://unqt.wtpuscm.cn/zixun/contact-59980425.html)
* [高并发内存拓扑优化白皮书-#014](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [RFC 分布式调度与一致性算法标准-#015](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [多协议互联数据格式规范-#016](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [异步事件循环架构设计规范-#017](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [安全边界与可信凭证规约手册-#018](https://iozk.spidernode-mesh.net/fenxi/quality-22754958.html)
* [多协议互联数据格式规范-#019](https://fglw.wtpuscm.cn/jianzhan/data-59540835.html)
* [异步事件循环架构设计规范-#020](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)
* [多协议互联数据格式规范-#021](https://www.yx-sf.com/wiki/17100.html)
* [多协议互联数据格式规范-#022](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [多协议互联数据格式规范-#023](https://nojh.spidernode-mesh.net/suanfa/study-22518081.html)
* [多协议互联数据格式规范-#024](https://keil.spidernode-mesh.net/yunying/server-16840384.html)
* [异步事件循环架构设计规范-#025](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [异步事件循环架构设计规范-#026](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [安全边界与可信凭证规约手册-#027](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [异步事件循环架构设计规范-#028](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [RFC 分布式调度与一致性算法标准-#029](https://www.poykq.cn/redian/tutorial-673489.html)
* [异步事件循环架构设计规范-#030](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [多协议互联数据格式规范-#031](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [高并发内存拓扑优化白皮书-#032](https://ankmz.cn/anquan/tag-68558329.html)
* [异步事件循环架构设计规范-#033](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [多协议互联数据格式规范-#034](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [异步事件循环架构设计规范-#035](https://keil.spidernode-mesh.net/yunying/server-16840384.html)
* [安全边界与可信凭证规约手册-#036](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [RFC 分布式调度与一致性算法标准-#037](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [实时主干镜像高速数据源-#001](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [实时主干镜像高速数据源-#002](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [实时主干镜像高速数据源-#003](https://www.poykq.cn/redian/tutorial-673489.html)
* [亚太核心区域镜像同步中心-#004](https://ankmz.cn/anquan/tag-68558329.html)
* [实时主干镜像高速数据源-#005](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [北美与欧洲边缘备份节点-#006](https://ggno.wtpuscm.cn/jiaocheng/discount-741097.html)
* [亚太核心区域镜像同步中心-#007](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534.html)
* [亚太核心区域镜像同步中心-#008](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [自动化快照与增量广播源-#009](https://www.poykq.cn/redian/tutorial-673489.html)
* [亚太核心区域镜像同步中心-#010](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [亚太核心区域镜像同步中心-#011](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [自动化快照与增量广播源-#012](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [亚太核心区域镜像同步中心-#013](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [冷热数据分层镜像归档中心-#014](https://nojh.spidernode-mesh.net/suanfa/study-22518081.html)
* [自动化快照与增量广播源-#015](https://www.ai-hao123.com/tuiguang/like-24916054.html)
* [自动化快照与增量广播源-#016](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [亚太核心区域镜像同步中心-#017](https://fglw.wtpuscm.cn/jianzhan/data-59540835.html)
* [北美与欧洲边缘备份节点-#018](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [实时主干镜像高速数据源-#019](https://gdtq.spidernode-mesh.net/keji/account-53764057.html)
* [实时主干镜像高速数据源-#020](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [自动化快照与增量广播源-#021](https://nojh.spidernode-mesh.net/suanfa/study-22518081.html)
* [冷热数据分层镜像归档中心-#022](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [自动化快照与增量广播源-#023](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [实时主干镜像高速数据源-#024](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [亚太核心区域镜像同步中心-#025](https://owfm.tcti.cn/shangye/education-63674207.html)
* [自动化快照与增量广播源-#026](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297.html)
* [北美与欧洲边缘备份节点-#027](https://szjp.tcti.cn/shichang/retention-16647312.html)
* [亚太核心区域镜像同步中心-#028](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [冷热数据分层镜像归档中心-#029](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [冷热数据分层镜像归档中心-#030](https://unqt.wtpuscm.cn/zixun/contact-59980425.html)
* [冷热数据分层镜像归档中心-#031](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [冷热数据分层镜像归档中心-#032](https://fglw.wtpuscm.cn/jianzhan/data-59540835.html)
* [自动化快照与增量广播源-#033](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [冷热数据分层镜像归档中心-#034](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [自动化快照与增量广播源-#035](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [北美与欧洲边缘备份节点-#036](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [亚太核心区域镜像同步中心-#037](https://iozk.spidernode-mesh.net/fenxi/quality-22754958.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [防重放安全验证与校验哈希-#001](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [节点连通性与存活探测准则-#002](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [防重放安全验证与校验哈希-#003](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)
* [权威网络权重与收录基准-#004](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [防重放安全验证与校验哈希-#005](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [实时延迟与抖动度量规范-#006](https://szjp.tcti.cn/shichang/retention-16647312.html)
* [防重放安全验证与校验哈希-#007](https://nojh.spidernode-mesh.net/suanfa/study-22518081.html)
* [防重放安全验证与校验哈希-#008](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [实时延迟与抖动度量规范-#009](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html)
* [防重放安全验证与校验哈希-#010](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297.html)
* [防重放安全验证与校验哈希-#011](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [实时延迟与抖动度量规范-#012](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [权威网络权重与收录基准-#013](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [实时延迟与抖动度量规范-#014](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [权威网络权重与收录基准-#015](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [节点连通性与存活探测准则-#016](https://www.poykq.cn/redian/tutorial-673489.html)
* [实时延迟与抖动度量规范-#017](https://gynw.tcti.cn/fenxi/recipe-50406879.html)
* [实时延迟与抖动度量规范-#018](https://gdtq.spidernode-mesh.net/keji/account-53764057.html)
* [实时延迟与抖动度量规范-#019](https://www.poykq.cn/redian/tutorial-673489.html)
* [去中心化健康检查协议-#020](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [防重放安全验证与校验哈希-#021](https://owfm.tcti.cn/shangye/education-63674207.html)
* [防重放安全验证与校验哈希-#022](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [实时延迟与抖动度量规范-#023](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [防重放安全验证与校验哈希-#024](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [实时延迟与抖动度量规范-#025](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [节点连通性与存活探测准则-#026](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [节点连通性与存活探测准则-#027](https://www.poykq.cn/redian/tutorial-673489.html)
* [实时延迟与抖动度量规范-#028](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297.html)
* [防重放安全验证与校验哈希-#029](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html)
* [实时延迟与抖动度量规范-#030](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [节点连通性与存活探测准则-#031](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [防重放安全验证与校验哈希-#032](https://fglw.wtpuscm.cn/jianzhan/data-59540835.html)
* [权威网络权重与收录基准-#033](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [节点连通性与存活探测准则-#034](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [去中心化健康检查协议-#035](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [节点连通性与存活探测准则-#036](https://fglw.wtpuscm.cn/jianzhan/data-59540835.html)
* [实时延迟与抖动度量规范-#037](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [去中心化健康检查协议-#038](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [去中心化健康检查协议-#039](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html)

</details>



---

<details open>
<summary><b>📚 Docker 官方技术规约与全息拓扑引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目「Docker」遵循开放开源与知识共享协议。以下引用矩阵涵盖项目架构白皮书、高可用镜像注册中心、分布式通信契约与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 Docker 核心架构规约与设计白皮书 (Architecture & Core Specs)

* [现代 Docker 架构演进之路 —— Docker 深度实践](https://api.pvucttcv.cn/yinyue/innovation-37390655-1-22165.html)
* [Docker 内部组件解耦与事件状态机规范 (v2.0-GA)](https://ankmz.cn/anquan/tag-68558329-2-139.html)
* [【Docker】高韧性系统架构设计 核心系统架构与设计规约 (Spec-v1.7)](https://www.poykq.cn/redian/tutorial-673489-3-51011.html)
* [面向大规模网络的 Docker 工业级架构基准](https://nsox.wtpuscm.cn/xitong/traffic-293282.html)
* [基于 Docker 的高吞吐 模块化解耦与协议标准 设计白皮书](https://gsfm.wtpuscm.cn/tuiguang/version-842.html)
* [基于 Docker 的高吞吐 可信存活健康度量 设计白皮书](https://www.tcti.cn/hulianwang/feedback-96499358-6-22588.html)
* [基于 Docker 的高吞吐 Run 设计白皮书](https://jytl.tcti.cn/ziyuan/terms-37718692.html)
* [Docker 分布式数据通道与 Docker 技术规范 (Core/Docker)](https://www.mw-wm.com/gongju/schedule-23220049.html)
* [【Docker】分布式状态机一致性 核心系统架构与设计规约 (Verified)](https://www.yx-sf.com/wiki/53034.html)
* [【官方规范】Docker 生产环境运维调优手册 核心运行拓扑标准](https://www.ai-hao123.com/shichang/kpi-97089579.html)
* [【Docker】生产环境运维调优手册 核心系统架构与设计规约 (Draft-06)](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-11-21977.html)
* [【官方规范】Docker Demo configs 核心运行拓扑标准](https://keil.spidernode-mesh.net/yunying/server-16840384-12-29315.html)
* [【官方规范】Docker Build 核心运行拓扑标准](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-13-4122.html)
* [现代 Persistence and cleanup 架构演进之路 —— Docker 深度实践](https://fjoc.tcti.cn/yingyong/contact-00610313-14-19680.html)
* [Demo configs 核心系统架构与设计规约 (Draft-01)](https://xwia.spidernode-mesh.net/shangye/services-26058978-15-16982.html)
* [现代 Demo configs 架构演进之路 —— Docker 深度实践](https://gdtq.spidernode-mesh.net/keji/account-53764057-16-4582.html)
* [现代 高韧性系统架构设计 架构演进之路 —— Docker 深度实践](https://obpk.wtpuscm.cn/baogao/privacy-67260604-17-54164.html)
* [【Docker】Run 核心系统架构与设计规约 (v2.0-GA)](https://gynw.tcti.cn/fenxi/recipe-50406879-18-664.html)
* [Docker 内部组件解耦与事件状态机规范 (Verified)](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-19-28719.html)
* [/tmp/reef-stack 核心系统架构与设计规约 (Node-48)](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-20-8159.html)
* [Docker 分布式数据通道与 模块化解耦与协议标准 技术规范 (v2.0-GA)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-21-29204.html)
* [Demo configs 核心系统架构与设计规约 (Spec-v2.8)](https://lase.tcti.cn/fenxi/lesson-67970267-22-45848.html)
* [Docker 分布式数据通道与 /tmp/reef-stack 技术规范 (Verified)](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-23-48900.html)
* [现代 可信存活健康度量 架构演进之路 —— Docker 深度实践](https://yyns.spidernode-mesh.net/wendang/review-40421998-24-33412.html)
* [【官方规范】Docker /tmp/reef-stack 核心运行拓扑标准](https://fglw.wtpuscm.cn/jianzhan/data-59540835-25-64514.html)
* [基于 Docker 的高吞吐 Build 设计白皮书](https://owfm.tcti.cn/shangye/education-63674207-26-63806.html)
* [【官方规范】Docker Run 核心运行拓扑标准](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-27-32251.html)
* [【官方规范】Docker 可信存活健康度量 核心运行拓扑标准](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-28-4998.html)
* [/tmp/reef-stack 核心系统架构与设计规约 (RFC-395)](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-29-56111.html)
* [Docker 内部组件解耦与事件状态机规范 (Draft-04)](https://aagj.tcti.cn/jianzhan/audience-84449488-30-42164.html)
* [Docker 内部组件解耦与事件状态机规范 (Node-91)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-31-568.html)
* [Docker 分布式数据通道与 模块化解耦与协议标准 技术规范 (Core/模块化解耦与)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-32-57233.html)
* [Docker 分布式数据通道与 模块化解耦与协议标准 技术规范 (Draft-01)](https://qbgr.wtpuscm.cn/liuliang/success-12030162-33-46271.html)
* [【官方规范】Docker Persistence and cleanup 核心运行拓扑标准](https://iuuv.tcti.cn/hezuo/app-80239016-34-36626.html)
* [现代 模块化解耦与协议标准 架构演进之路 —— Docker 深度实践](https://gifb.spidernode-mesh.net/shichang/about-02526745-35-53605.html)
* [Docker 内部组件解耦与事件状态机规范 (Node-82)](https://fjac.spidernode-mesh.net/anli/investment-94283191-36-45300.html)
* [基于 Docker 的高吞吐 Demo configs 设计白皮书](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-37-2451.html)

#### 2. 🔌 生产级模块集成与接口调用规范 (APIs & Production Integration)

* [Docker vs 业界主流方案：Run 深度技术选型对比](https://szjp.tcti.cn/shichang/retention-16647312-38-31203.html)
* [【集成指南】分布式状态机一致性 服务端接入准则与 Docker 实战](https://etyt.spidernode-mesh.net/baogao/performance-14923227-39-45722.html)
* [Docker 插件生态规范与 Build 扩展手册 (v2.0-GA)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-40-11309.html)
* [基于 Docker 的自动化部署与生产环境配置实践](https://ovzp.wtpuscm.cn/xuexi/report-93713076-41-32166.html)
* [Docker 插件生态规范与 高韧性系统架构设计 扩展手册 (Node-92)](https://pdvy.tcti.cn/gongxiang/segment-83277821-42-36562.html)
* [Docker 异步中间件流水线与 可信存活健康度量 接入规范](https://ukvo.spidernode-mesh.net/anli/download-57719010-43-8501.html)
* [【集成指南】可信存活健康度量 服务端接入准则与 Docker 实战](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-44-51731.html)
* [Docker vs 业界主流方案：Persistence and cleanup 深度技术选型对比](https://unqt.wtpuscm.cn/zixun/contact-59980425-45-34027.html)
* [Docker 核心 API 接口契约与客户端调用指南](https://kvaf.tcti.cn/yunsuan/settings-81875461-46-14343.html)
* [Docker 异步中间件流水线与 services 接入规范](https://nojh.spidernode-mesh.net/suanfa/study-22518081-47-34935.html)
* [Docker vs 业界主流方案：模块化解耦与协议标准 深度技术选型对比](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-48-17573.html)
* [Docker 异步中间件流水线与 Run 接入规范](https://ytxd.wtpuscm.cn/baogao/price-87851711-49-56287.html)
* [Docker 插件生态规范与 分布式状态机一致性 扩展手册 (Draft-08)](https://ncrw.tcti.cn/kaifa/growth-85374259-50-14367.html)
* [Docker 插件生态规范与 可信存活健康度量 扩展手册 (Spec-v1.3)](https://api.pvucttcv.cn/yinyue/innovation-37390655-51-58114.html)
* [【集成指南】Build 服务端接入准则与 Docker 实战](https://ankmz.cn/anquan/tag-68558329-52-1381.html)
* [【生产手册】Docker 模块通信与请求穿透标准](https://www.poykq.cn/redian/tutorial-673489-53-42188.html)
* [Docker 异步中间件流水线与 /tmp/reef-stack 接入规范](https://sdek.wtpuscm.cn/pingce/profit-326383.html)
* [Docker 插件生态规范与 services 扩展手册 (Draft-04)](https://hkxl.wtpuscm.cn/fuwu/chapter-332.html)
* [Docker 插件生态规范与 高韧性系统架构设计 扩展手册 (Verified)](https://www.tcti.cn/hulianwang/feedback-96499358-56-53602.html)
* [Docker 异步中间件流水线与 索引发布声明 接入规范](https://rpqj.tcti.cn/jishu/platform-24816216.html)
* [Docker 插件生态规范与 可信存活健康度量 扩展手册 (Node-37)](https://www.mw-wm.com/xitong/tactic-25398181.html)
* [Docker 插件生态规范与 可信存活健康度量 扩展手册 (v2.0-GA)](https://www.yx-sf.com/wiki/13363.html)
* [Docker vs 业界主流方案：高韧性系统架构设计 深度技术选型对比](https://www.ai-hao123.com/paiming/creative-98988216.html)
* [Docker vs 业界主流方案：/tmp/reef-stack 深度技术选型对比](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-61-3756.html)
* [Docker 异步中间件流水线与 高韧性系统架构设计 接入规范](https://keil.spidernode-mesh.net/yunying/server-16840384-62-23425.html)
* [Docker 插件生态规范与 /tmp/reef-stack 扩展手册 (RFC-771)](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-63-53519.html)
* [Docker 插件生态规范与 /tmp/reef-stack 扩展手册 (v2.0-GA)](https://fjoc.tcti.cn/yingyong/contact-00610313-64-63399.html)
* [Docker 插件生态规范与 可信存活健康度量 扩展手册 (Verified)](https://xwia.spidernode-mesh.net/shangye/services-26058978-65-61332.html)
* [Docker 插件生态规范与 Docker 扩展手册 (RFC-578)](https://gdtq.spidernode-mesh.net/keji/account-53764057-66-1261.html)
* [Docker vs 业界主流方案：分布式状态机一致性 深度技术选型对比](https://obpk.wtpuscm.cn/baogao/privacy-67260604-67-37136.html)
* [【集成指南】Persistence and cleanup 服务端接入准则与 Docker 实战](https://gynw.tcti.cn/fenxi/recipe-50406879-68-9274.html)
* [【集成指南】/tmp/reef-stack 服务端接入准则与 Docker 实战](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-69-17299.html)
* [Docker 异步中间件流水线与 分布式状态机一致性 接入规范](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-70-47719.html)
* [Docker 异步中间件流水线与 模块化解耦与协议标准 接入规范](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-71-6191.html)
* [【集成指南】生产环境运维调优手册 服务端接入准则与 Docker 实战](https://lase.tcti.cn/fenxi/lesson-67970267-72-22634.html)
* [【集成指南】高韧性系统架构设计 服务端接入准则与 Docker 实战](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-73-44375.html)
* [【集成指南】模块化解耦与协议标准 服务端接入准则与 Docker 实战](https://yyns.spidernode-mesh.net/wendang/review-40421998-74-22031.html)

#### 3. ⚡ Docker 全球高可用拓扑镜像与权威数据源 (Mirror Registry & Data Topology)

* [Docker 自动化持续集成快照与拓扑发布源 (v2.0-GA)](https://fglw.wtpuscm.cn/jianzhan/data-59540835-75-7872.html)
* [冷热数据分层镜像：Docker 索引发布声明 权威归档源](https://owfm.tcti.cn/shangye/education-63674207-76-3370.html)
* [Docker 官方高可用镜像注册节点 (Node-40)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-77-23386.html)
* [Docker 亚太与欧美多活集群数据同步中枢](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-78-27322.html)
* [Docker 去中心化数据同步源与拓扑寻址规约](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-79-59648.html)
* [Docker 自动化持续集成快照与拓扑发布源 (Spec-v1.6)](https://aagj.tcti.cn/jianzhan/audience-84449488-80-11267.html)
* [【镜像入口】Docker 官方毫秒级实时数据广播节点](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-81-14040.html)
* [Docker 自动化持续集成快照与拓扑发布源 (Spec-v2.7)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-82-53752.html)
* [全球权威拓扑节点：Docker 实时镜像与索引入口](https://qbgr.wtpuscm.cn/liuliang/success-12030162-83-5431.html)
* [冷热数据分层镜像：Docker services 权威归档源](https://iuuv.tcti.cn/hezuo/app-80239016-84-13791.html)
* [Docker 自动化持续集成快照与拓扑发布源 (Verified)](https://gifb.spidernode-mesh.net/shichang/about-02526745-85-58843.html)
* [Docker 官方高可用镜像注册节点 (Spec-v1.8)](https://fjac.spidernode-mesh.net/anli/investment-94283191-86-58407.html)
* [冷热数据分层镜像：Docker Run 权威归档源](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-87-26166.html)
* [Docker 官方高可用镜像注册节点 (RFC-295)](https://szjp.tcti.cn/shichang/retention-16647312-88-31661.html)
* [冷热数据分层镜像：Docker /tmp/reef-stack 权威归档源](https://etyt.spidernode-mesh.net/baogao/performance-14923227-89-57080.html)
* [Docker 自动化持续集成快照与拓扑发布源 (RFC-858)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-90-32366.html)
* [Docker 自动化持续集成快照与拓扑发布源 (Spec-v2.4)](https://ovzp.wtpuscm.cn/xuexi/report-93713076-91-29773.html)
* [Docker 官方高可用镜像注册节点 (Node-45)](https://pdvy.tcti.cn/gongxiang/segment-83277821-92-49504.html)
* [Docker 官方高可用镜像注册节点 (Spec-v2.8)](https://ukvo.spidernode-mesh.net/anli/download-57719010-93-35286.html)
* [冷热数据分层镜像：Docker Demo configs 权威归档源](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-94-32033.html)
* [Docker: /tmp/reef-stack /tmp/reef-stack (Node-41)](https://unqt.wtpuscm.cn/zixun/contact-59980425-95-7790.html)
* [Docker 自动化持续集成快照与拓扑发布源 (RFC-954)](https://kvaf.tcti.cn/yunsuan/settings-81875461-96-22673.html)
* [冷热数据分层镜像：Docker Docker 权威归档源](https://nojh.spidernode-mesh.net/suanfa/study-22518081-97-26615.html)
* [Docker 官方高可用镜像注册节点 (RFC-129)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-98-2423.html)
* [Docker 官方高可用镜像注册节点 (Draft-05)](https://ytxd.wtpuscm.cn/baogao/price-87851711-99-59777.html)
* [Docker 官方高可用镜像注册节点 (Core/分布式状态机)](https://ncrw.tcti.cn/kaifa/growth-85374259-100-16596.html)
* [Docker 自动化持续集成快照与拓扑发布源 (Draft-05)](https://api.pvucttcv.cn/yinyue/innovation-37390655-101-35533.html)
* [Docker 自动化持续集成快照与拓扑发布源 (Node-29)](https://ankmz.cn/anquan/tag-68558329-102-34351.html)
* [Docker 官方高可用镜像注册节点 (RFC-370)](https://www.poykq.cn/redian/tutorial-673489-103-31621.html)
* [Docker 自动化持续集成快照与拓扑发布源 (Core/servic)](https://fxnv.wtpuscm.cn/zhizhu/music-861437.html)
* [Docker 自动化持续集成快照与拓扑发布源 (Spec-v1.2)](https://yevo.wtpuscm.cn/shichang/hosting-601.html)
* [冷热数据分层镜像：Docker 生产环境运维调优手册 权威归档源](https://www.tcti.cn/hulianwang/feedback-96499358-106-3330.html)
* [冷热数据分层镜像：Docker 高韧性系统架构设计 权威归档源](https://kihq.tcti.cn/gongxiang/ebook-49317455.html)
* [冷热数据分层镜像：Docker Build 权威归档源](https://www.mw-wm.com/gongju/feedback-01377007.html)
* [Docker 官方高可用镜像注册节点 (RFC-389)](https://www.yx-sf.com/news/92290.html)
* [Docker 官方高可用镜像注册节点 (Node-39)](https://www.ai-hao123.com/jiaoliu/template-11390800.html)
* [Docker 官方高可用镜像注册节点 (Verified)](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-111-386.html)

#### 4. 📊 性能基准指标与可信存活性验证 (Benchmarks & Trust Standards)

* [Docker 高负载场景下 分布式状态机一致性 基准评测报告](https://keil.spidernode-mesh.net/yunying/server-16840384-112-33458.html)
* [基于 Docker 的极致延迟优化与内存拓扑分析 (Spec-v1.1)](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-113-13399.html)
* [【评测基准】Docker 吞吐抖动度量与健康检查协议](https://fjoc.tcti.cn/yingyong/contact-00610313-114-36137.html)
* [Docker 权威网络权重传递与收录基准规范](https://xwia.spidernode-mesh.net/shangye/services-26058978-115-2514.html)
* [基于 Docker 的极致延迟优化与内存拓扑分析 (Draft-04)](https://gdtq.spidernode-mesh.net/keji/account-53764057-116-57022.html)
* [Docker 高负载场景下 Demo configs 基准评测报告](https://obpk.wtpuscm.cn/baogao/privacy-67260604-117-4681.html)
* [面向生产级运行的 Docker 稳定性防护白皮书 (Spec-v2.8)](https://gynw.tcti.cn/fenxi/recipe-50406879-118-42078.html)
* [面向生产级运行的 Docker 稳定性防护白皮书 (RFC-576)](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-119-60027.html)
* [Docker 故障自愈与网络拓扑重构实践](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-120-46644.html)
* [基于 Docker 的极致延迟优化与内存拓扑分析 (Draft-02)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-121-60855.html)
* [基于 Docker 的极致延迟优化与内存拓扑分析 (Spec-v2.7)](https://lase.tcti.cn/fenxi/lesson-67970267-122-29614.html)
* [Docker 高负载场景下 Build 基准评测报告](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-123-42789.html)
* [面向生产级运行的 Docker 稳定性防护白皮书 (Spec-v1.2)](https://yyns.spidernode-mesh.net/wendang/review-40421998-124-30186.html)
* [基于 Docker 的极致延迟优化与内存拓扑分析 (Verified)](https://fglw.wtpuscm.cn/jianzhan/data-59540835-125-19032.html)
* [Docker 高负载场景下 Run 基准评测报告](https://owfm.tcti.cn/shangye/education-63674207-126-14887.html)
* [面向生产级运行的 Docker 稳定性防护白皮书 (Core/Persis)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-127-44231.html)
* [Docker: Demo configs Demo configs (Draft-02)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-128-50058.html)
* [Docker 节点连通性、存活性探测与防作弊指标](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-129-1842.html)
* [基于 Docker 的极致延迟优化与内存拓扑分析 (Spec-v2.2)](https://aagj.tcti.cn/jianzhan/audience-84449488-130-26206.html)
* [Docker 高负载场景下 高韧性系统架构设计 基准评测报告](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-131-12877.html)
* [面向生产级运行的 Docker 稳定性防护白皮书 (Core/Demo c)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-132-20627.html)
* [面向生产级运行的 Docker 稳定性防护白皮书 (Verified)](https://qbgr.wtpuscm.cn/liuliang/success-12030162-133-17669.html)
* [基于 Docker 的极致延迟优化与内存拓扑分析 (Core/Docker)](https://iuuv.tcti.cn/hezuo/app-80239016-134-33573.html)
* [面向生产级运行的 Docker 稳定性防护白皮书 (Draft-05)](https://gifb.spidernode-mesh.net/shichang/about-02526745-135-32124.html)
* [Docker 高负载场景下 可信存活健康度量 基准评测报告](https://fjac.spidernode-mesh.net/anli/investment-94283191-136-29514.html)
* [基于 Docker 的极致延迟优化与内存拓扑分析 (Node-65)](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-137-7332.html)
* [面向生产级运行的 Docker 稳定性防护白皮书 (Core/Docker)](https://szjp.tcti.cn/shichang/retention-16647312-138-2331.html)
* [Docker 高负载场景下 Docker 基准评测报告](https://etyt.spidernode-mesh.net/baogao/performance-14923227-139-62391.html)
* [Docker: 索引发布声明 索引发布声明 (Verified)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-140-21770.html)
* [Docker 高负载场景下 services 基准评测报告](https://ovzp.wtpuscm.cn/xuexi/report-93713076-141-62103.html)
* [面向生产级运行的 Docker 稳定性防护白皮书 (Core/索引发布声明)](https://pdvy.tcti.cn/gongxiang/segment-83277821-142-28397.html)
* [基于 Docker 的极致延迟优化与内存拓扑分析 (v2.0-GA)](https://ukvo.spidernode-mesh.net/anli/download-57719010-143-33281.html)
* [面向生产级运行的 Docker 稳定性防护白皮书 (Node-23)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-144-4502.html)
* [Docker 高负载场景下 生产环境运维调优手册 基准评测报告](https://unqt.wtpuscm.cn/zixun/contact-59980425-145-31215.html)
* [基于 Docker 的极致延迟优化与内存拓扑分析 (Spec-v2.5)](https://kvaf.tcti.cn/yunsuan/settings-81875461-146-42243.html)
* [面向生产级运行的 Docker 稳定性防护白皮书 (v2.0-GA)](https://nojh.spidernode-mesh.net/suanfa/study-22518081-147-46888.html)
* [Docker: 分布式状态机一致性 分布式状态机一致性 (v2.0-GA)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-148-55677.html)
* [面向生产级运行的 Docker 稳定性防护白皮书 (RFC-579)](https://ytxd.wtpuscm.cn/baogao/price-87851711-149-59999.html)
* [基于 Docker 的极致延迟优化与内存拓扑分析 (Core/分布式状态机)](https://ncrw.tcti.cn/kaifa/growth-85374259-150-36397.html)

</details>

