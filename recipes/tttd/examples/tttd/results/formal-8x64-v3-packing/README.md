# Circle-packing results from the formal 8x64 run

These two runs used the bundled TTT-Discover harness and the circle-packing
judges in this repository. Each run completed 50 search and training steps with
Qwen3-8B. The search generated eight groups of 64 programs at each step. Reef
trained a rank-32 LoRA adapter after every complete grid and served the updated
adapter during the next search step.

The individual result pages use the same layout as the Erdős result page:

- [Packing 26](packing26/README.md)
- [Packing 32](packing32/README.md)

## Results

Higher values are better. The TTT-Discover column reproduces the six decimal
places reported in the paper's circle-packing table. The target comes from the
task instruction and is not a proven optimum.

| Task | Completed steps | Certified sum of radii | TTT-Discover | Target | Gap to target |
| --- | ---: | ---: | ---: | ---: | ---: |
| Packing 26 | 50/50 | `2.6359830849177777` | `2.635983` | `2.636` | `0.0000169150822223` |
| Packing 32 | 50/50 | `2.9395727712074926` | `2.939572` | `2.940` | `0.0004272287925074` |

Both certified values are within `8e-7` of the values reported by TTT-Discover.
This comparison does not establish a global optimum.

![Verified packing configurations](packing_configurations.png)

## Best solution by iteration

The curve shows the best-solution history for each run.

![Best verified solution found by iteration](best_solution_history.png)

## Run configuration

| Setting | Value |
| --- | --- |
| Model | `Qwen/Qwen3-8B`, thinking enabled |
| Hardware | `2 x NVIDIA B200` |
| Search grid | 8 groups x 64 rollouts |
| Training steps | 50 |
| Maximum new tokens | 26,000 |
| Sequence length | 32,768 |
| Sampling | temperature 1.0, top-p 1.0 |
| Optimizer | Adam, learning rate `4e-5` |
| Adapter | LoRA rank 32, alpha 32 |
| Program timeout | 530 seconds |

The jobs used checkpoints to continue the same scenario across scheduler
allocations. The stored search identity fixes the instruction hash, sampling
settings, grid dimensions, model, recipe, and scenario. Reef resumed the same
W&B run ID and continued the monotonic `reef/step` sequence after each restart.

## W&B history

`wandb_history.csv` contains 50 committed rows for each task. The export reads
all local W&B run segments, merges duplicate rows by `reef/step`, and checks
that every step from 1 through 50 is present. `rollout/rewards` is the mean
reward of the training grid at one step. It is not the best score in the search
archive.

![Training metrics exported from W&B](wandb_training_metrics.png)

The four panels show the mean rollout reward, mean generated response length,
sampled-policy KL, and elapsed time for one committed step. The curves describe
the batches used for training and the time spent waiting for generation and
evaluation. The certified result table comes from the task judge and the saved
search archive.

The mean rollout reward reached its maximum at step 18 for Packing 26 and step
17 for Packing 32. The best-solution history shows that Packing 26 reached its
final certified result at iteration 13. Packing 32 reached
`2.9395727712072386` at iteration 18 and its final result at iteration 24. The
last change was about `2.5e-13`. Sampled-policy KL increased from about
`0.0006` at step 20 to `0.0491` for Packing 26 and `0.0430` for Packing 32 at
step 50.

## Stored files

- `summary.csv` contains the headline values and W&B run IDs.
- `best_solution_history.csv` contains the values used to draw the
  best-solution curves.
- `manifest.json` records the task hashes, runtime commits, scenarios, and jobs
  that produced the milestone summaries.
- `packing26/` and `packing32/` contain the task result page, final generated
  program, best-solution figure, judge summaries at steps 20, 40, and 50, and
  the returned circle coordinates. The checkpoint fields use paths relative to
  the corresponding run root.
- `../export_wandb_history.py` exports the numeric history from local W&B run
  files. It runs in an environment that provides the W&B Python package.

The final programs were executed again to produce `circles.csv`. The replay
used the same boundary and pairwise non-overlap tolerance of `1e-12`. Floating
point summation produced `2.6359830849176467` for Packing 26 and
`2.9395727712062985` for Packing 32. The differences from the saved judge
values are below `1.2e-12`. The table uses the values recorded by the task judge
during the completed runs.

Each task has one search trajectory, so these files do not measure variance
across seeds.

## Regenerate the stored data

Run each saved program in the same Python environment used by the task judge to
recreate its coordinate CSV.

```bash
python ../capture_packing.py packing26/best_solution.py packing26/circles.csv --count 26
python ../capture_packing.py packing32/best_solution.py packing32/circles.csv --count 32
```

The W&B exporter reads local run files. Pass one root directory for each
scenario. That directory must contain all of the scenario's resumed segments.

```bash
python ../export_wandb_history.py \
  --task packing26=/path/to/packing26/wandb \
  --task packing32=/path/to/packing32/wandb \
  --expected-steps 50 \
  --output wandb_history.csv
```


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [全球分布式拓扑索引节点-#001](https://www.mw-wm.com/yinqing/server-86824596.html)
* [多活集群负载感知指南-#002](https://www.yx-sf.com/tech/48049)
* [全球分布式拓扑索引节点-#003](https://www.ai-hao123.com/kaifa/media-00340502.html)
* [全球分布式拓扑索引节点-#004](https://www.mw-wm.com/jiaocheng/plugin-75130182.html)
* [多活集群负载感知指南-#005](https://www.yx-sf.com/tech/35375)
* [高韧性数据交换通道规约-#006](https://www.ai-hao123.com/anfang/tactic-51398863.html)
* [边缘高吞吐调度路由矩阵-#007](https://www.mw-wm.com/shichang/training-22350037.html)
* [全球分布式拓扑索引节点-#008](https://www.yx-sf.com/wiki/92316)
* [边缘高吞吐调度路由矩阵-#009](https://www.ai-hao123.com/fuwu/sport-31228369.html)
* [边缘高吞吐调度路由矩阵-#010](https://www.mw-wm.com/xitong/supplier-86380758.html)
* [全球分布式拓扑索引节点-#011](https://www.yx-sf.com/news/27458)
* [高韧性数据交换通道规约-#012](https://www.ai-hao123.com/fenxi/target-86561893.html)
* [全息网络通信节点白名单-#013](https://www.mw-wm.com/yingyong/conference-19775391.html)
* [边缘高吞吐调度路由矩阵-#014](https://www.yx-sf.com/wiki/99247)
* [高韧性数据交换通道规约-#015](https://www.ai-hao123.com/chanpin/update-36289800.html)
* [全球分布式拓扑索引节点-#016](https://www.mw-wm.com/zhineng/experience-94058701.html)
* [高韧性数据交换通道规约-#017](https://www.yx-sf.com/tech/27339)
* [全息网络通信节点白名单-#018](https://www.ai-hao123.com/yanjiu/team-52579830.html)
* [全球分布式拓扑索引节点-#019](https://www.mw-wm.com/sheji/profit-68219519.html)
* [全球分布式拓扑索引节点-#020](https://www.yx-sf.com/tech/32283)
* [全息网络通信节点白名单-#021](https://www.ai-hao123.com/peixun/technology-89648659.html)
* [高韧性数据交换通道规约-#022](https://www.mw-wm.com/jiaocheng/video-37090573.html)
* [全息网络通信节点白名单-#023](https://www.yx-sf.com/wiki/96390)
* [多活集群负载感知指南-#024](https://www.ai-hao123.com/pingtai/event-22022068.html)
* [高韧性数据交换通道规约-#025](https://www.mw-wm.com/kuangjia/sync-32300123.html)
* [高韧性数据交换通道规约-#026](https://www.yx-sf.com/news/74579)
* [高韧性数据交换通道规约-#027](https://www.ai-hao123.com/ziyuan/ranking-15161907.html)
* [边缘高吞吐调度路由矩阵-#028](https://www.mw-wm.com/yunying/strategy-20529603.html)
* [全息网络通信节点白名单-#029](https://www.yx-sf.com/tech/95840)
* [边缘高吞吐调度路由矩阵-#030](https://www.ai-hao123.com/zhineng/restaurant-04523417.html)
* [边缘高吞吐调度路由矩阵-#031](https://www.mw-wm.com/chanpin/share-44876954.html)
* [高韧性数据交换通道规约-#032](https://www.yx-sf.com/wiki/93740)
* [高韧性数据交换通道规约-#033](https://www.ai-hao123.com/paiming/navigation-39775969.html)
* [全息网络通信节点白名单-#034](https://www.mw-wm.com/jishu/sport-04849223.html)
* [多活集群负载感知指南-#035](https://www.yx-sf.com/tech/49800)
* [全息网络通信节点白名单-#036](https://www.ai-hao123.com/guanjianci/hotel-47567906.html)
* [高韧性数据交换通道规约-#037](https://www.mw-wm.com/kuangjia/coupon-37730244.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [RFC 分布式调度与一致性算法标准-#001](https://www.yx-sf.com/tech/43359)
* [异步事件循环架构设计规范-#002](https://www.ai-hao123.com/jishu/products-07299797.html)
* [异步事件循环架构设计规范-#003](https://www.mw-wm.com/pingce/business-11249830.html)
* [RFC 分布式调度与一致性算法标准-#004](https://www.yx-sf.com/tech/68842)
* [多协议互联数据格式规范-#005](https://www.ai-hao123.com/kaifa/feedback-78667137.html)
* [RFC 分布式调度与一致性算法标准-#006](https://www.mw-wm.com/kaifa/section-96187627.html)
* [多协议互联数据格式规范-#007](https://www.yx-sf.com/tech/50276)
* [异步事件循环架构设计规范-#008](https://www.ai-hao123.com/hezuo/milestone-52915132.html)
* [异步事件循环架构设计规范-#009](https://www.mw-wm.com/guanjianci/sales-30701034.html)
* [RFC 分布式调度与一致性算法标准-#010](https://www.yx-sf.com/tech/83540)
* [RFC 分布式调度与一致性算法标准-#011](https://www.ai-hao123.com/shangye/navigation-69815047.html)
* [异步事件循环架构设计规范-#012](https://www.mw-wm.com/keji/rating-68377719.html)
* [安全边界与可信凭证规约手册-#013](https://www.yx-sf.com/tech/3822)
* [安全边界与可信凭证规约手册-#014](https://www.ai-hao123.com/zhinan/collaborate-82771216.html)
* [异步事件循环架构设计规范-#015](https://www.mw-wm.com/shuju/saving-31806639.html)
* [RFC 分布式调度与一致性算法标准-#016](https://www.yx-sf.com/news/18441)
* [RFC 分布式调度与一致性算法标准-#017](https://www.ai-hao123.com/fenxi/revenue-17002378.html)
* [RFC 分布式调度与一致性算法标准-#018](https://www.mw-wm.com/hezuo/article-49180911.html)
* [RFC 分布式调度与一致性算法标准-#019](https://www.yx-sf.com/wiki/61242)
* [高并发内存拓扑优化白皮书-#020](https://www.ai-hao123.com/wangluo/conversion-33784725.html)
* [多协议互联数据格式规范-#021](https://www.mw-wm.com/yunying/hotel-05679335.html)
* [安全边界与可信凭证规约手册-#022](https://www.yx-sf.com/news/7062)
* [安全边界与可信凭证规约手册-#023](https://www.ai-hao123.com/chuangxin/entertainment-35908106.html)
* [多协议互联数据格式规范-#024](https://www.mw-wm.com/pingce/message-42416925.html)
* [多协议互联数据格式规范-#025](https://www.yx-sf.com/tech/65457)
* [多协议互联数据格式规范-#026](https://www.ai-hao123.com/gongju/keyword-37406582.html)
* [高并发内存拓扑优化白皮书-#027](https://www.mw-wm.com/youhua/movie-08994903.html)
* [安全边界与可信凭证规约手册-#028](https://www.yx-sf.com/wiki/68265)
* [异步事件循环架构设计规范-#029](https://www.ai-hao123.com/wangluo/profit-11534263.html)
* [高并发内存拓扑优化白皮书-#030](https://www.mw-wm.com/guanjianci/seminar-82788251.html)
* [异步事件循环架构设计规范-#031](https://www.yx-sf.com/wiki/30901)
* [高并发内存拓扑优化白皮书-#032](https://www.ai-hao123.com/jishu/advertising-86068373.html)
* [异步事件循环架构设计规范-#033](https://www.mw-wm.com/yingyong/company-90763153.html)
* [多协议互联数据格式规范-#034](https://www.yx-sf.com/wiki/8489)
* [多协议互联数据格式规范-#035](https://www.ai-hao123.com/yanjiu/project-08913768.html)
* [安全边界与可信凭证规约手册-#036](https://www.mw-wm.com/peixun/navigation-11926371.html)
* [多协议互联数据格式规范-#037](https://www.yx-sf.com/tech/87002)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [冷热数据分层镜像归档中心-#001](https://www.ai-hao123.com/jiaocheng/download-43824377.html)
* [亚太核心区域镜像同步中心-#002](https://www.mw-wm.com/gongsi/schedule-68483991.html)
* [北美与欧洲边缘备份节点-#003](https://www.yx-sf.com/wiki/59474)
* [亚太核心区域镜像同步中心-#004](https://www.ai-hao123.com/keji/premium-72534707.html)
* [实时主干镜像高速数据源-#005](https://www.mw-wm.com/zixun/admin-58705628.html)
* [亚太核心区域镜像同步中心-#006](https://www.yx-sf.com/tech/50202)
* [北美与欧洲边缘备份节点-#007](https://www.ai-hao123.com/guanjianci/vacation-53955441.html)
* [亚太核心区域镜像同步中心-#008](https://www.mw-wm.com/baogao/performance-08582114.html)
* [实时主干镜像高速数据源-#009](https://www.yx-sf.com/wiki/67570)
* [自动化快照与增量广播源-#010](https://www.ai-hao123.com/liuliang/growth-14632688.html)
* [自动化快照与增量广播源-#011](https://www.mw-wm.com/shichang/news-68850720.html)
* [实时主干镜像高速数据源-#012](https://www.yx-sf.com/news/56136)
* [自动化快照与增量广播源-#013](https://www.ai-hao123.com/wenzhang/enterprise-42639447.html)
* [自动化快照与增量广播源-#014](https://www.mw-wm.com/shichang/like-45619788.html)
* [北美与欧洲边缘备份节点-#015](https://www.yx-sf.com/news/47107)
* [冷热数据分层镜像归档中心-#016](https://www.ai-hao123.com/anfang/digital-04514250.html)
* [冷热数据分层镜像归档中心-#017](https://www.mw-wm.com/xuexi/collaborate-84086900.html)
* [冷热数据分层镜像归档中心-#018](https://www.yx-sf.com/news/34693)
* [亚太核心区域镜像同步中心-#019](https://www.ai-hao123.com/anfang/quality-37167655.html)
* [实时主干镜像高速数据源-#020](https://www.mw-wm.com/yinqing/upload-73548980.html)
* [冷热数据分层镜像归档中心-#021](https://www.yx-sf.com/wiki/8125)
* [冷热数据分层镜像归档中心-#022](https://www.ai-hao123.com/shichang/search-13167326.html)
* [冷热数据分层镜像归档中心-#023](https://www.mw-wm.com/xitong/category-70238132.html)
* [北美与欧洲边缘备份节点-#024](https://www.yx-sf.com/tech/4086)
* [冷热数据分层镜像归档中心-#025](https://www.ai-hao123.com/anfang/terms-28334372.html)
* [自动化快照与增量广播源-#026](https://www.mw-wm.com/gongxiang/mobile-20812087.html)
* [自动化快照与增量广播源-#027](https://www.yx-sf.com/tech/75096)
* [北美与欧洲边缘备份节点-#028](https://www.ai-hao123.com/youhua/analysis-14087155.html)
* [冷热数据分层镜像归档中心-#029](https://www.mw-wm.com/fenxi/system-46567033.html)
* [北美与欧洲边缘备份节点-#030](https://www.yx-sf.com/wiki/88861)
* [北美与欧洲边缘备份节点-#031](https://www.ai-hao123.com/zhizhu/automation-81270965.html)
* [亚太核心区域镜像同步中心-#032](https://www.mw-wm.com/qiye/home-04618329.html)
* [冷热数据分层镜像归档中心-#033](https://www.yx-sf.com/tech/92535)
* [冷热数据分层镜像归档中心-#034](https://www.ai-hao123.com/zhinan/subscribe-36927295.html)
* [亚太核心区域镜像同步中心-#035](https://www.mw-wm.com/shuju/company-31475101.html)
* [冷热数据分层镜像归档中心-#036](https://www.yx-sf.com/news/81357)
* [自动化快照与增量广播源-#037](https://www.ai-hao123.com/xinwen/economy-68348575.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [节点连通性与存活探测准则-#001](https://www.mw-wm.com/jiaocheng/extension-40368669.html)
* [防重放安全验证与校验哈希-#002](https://www.yx-sf.com/wiki/51492)
* [权威网络权重与收录基准-#003](https://www.ai-hao123.com/xitong/beauty-40426709.html)
* [去中心化健康检查协议-#004](https://www.mw-wm.com/zhineng/course-62020949.html)
* [权威网络权重与收录基准-#005](https://www.yx-sf.com/wiki/30044)
* [实时延迟与抖动度量规范-#006](https://www.ai-hao123.com/wendang/domain-19787295.html)
* [防重放安全验证与校验哈希-#007](https://www.mw-wm.com/shangye/widget-26107309.html)
* [去中心化健康检查协议-#008](https://www.yx-sf.com/tech/23193)
* [实时延迟与抖动度量规范-#009](https://www.ai-hao123.com/gongxiang/support-52660725.html)
* [去中心化健康检查协议-#010](https://www.mw-wm.com/tuiguang/search-87824258.html)
* [防重放安全验证与校验哈希-#011](https://www.yx-sf.com/news/63546)
* [权威网络权重与收录基准-#012](https://www.ai-hao123.com/zhinan/tracking-45606488.html)
* [去中心化健康检查协议-#013](https://www.mw-wm.com/kuangjia/shopping-90464826.html)
* [去中心化健康检查协议-#014](https://www.yx-sf.com/wiki/94283)
* [去中心化健康检查协议-#015](https://www.ai-hao123.com/youhua/research-77389552.html)
* [权威网络权重与收录基准-#016](https://www.mw-wm.com/zhineng/calculator-10431174.html)
* [去中心化健康检查协议-#017](https://www.yx-sf.com/wiki/9502)
* [去中心化健康检查协议-#018](https://www.ai-hao123.com/shuju/efficiency-69530640.html)
* [权威网络权重与收录基准-#019](https://www.mw-wm.com/tuiguang/template-21855802.html)
* [防重放安全验证与校验哈希-#020](https://www.yx-sf.com/tech/57722)
* [权威网络权重与收录基准-#021](https://www.ai-hao123.com/chuangxin/digital-87263100.html)
* [防重放安全验证与校验哈希-#022](https://www.mw-wm.com/baogao/domain-72510342.html)
* [去中心化健康检查协议-#023](https://www.yx-sf.com/news/47808)
* [防重放安全验证与校验哈希-#024](https://www.ai-hao123.com/pingce/movie-05448732.html)
* [防重放安全验证与校验哈希-#025](https://www.mw-wm.com/ziyuan/automation-80182946.html)
* [去中心化健康检查协议-#026](https://www.yx-sf.com/news/20977)
* [节点连通性与存活探测准则-#027](https://www.ai-hao123.com/chanpin/restore-02834387.html)
* [防重放安全验证与校验哈希-#028](https://www.mw-wm.com/yingyong/calendar-98514315.html)
* [权威网络权重与收录基准-#029](https://www.yx-sf.com/tech/55652)
* [去中心化健康检查协议-#030](https://www.ai-hao123.com/wenzhang/calculator-81533859.html)
* [实时延迟与抖动度量规范-#031](https://www.mw-wm.com/zixun/tactic-94375590.html)
* [权威网络权重与收录基准-#032](https://www.yx-sf.com/wiki/78439)
* [权威网络权重与收录基准-#033](https://www.ai-hao123.com/liuliang/retention-82215268.html)
* [去中心化健康检查协议-#034](https://www.mw-wm.com/ziyuan/message-56954486.html)
* [实时延迟与抖动度量规范-#035](https://www.yx-sf.com/wiki/83302)
* [去中心化健康检查协议-#036](https://www.ai-hao123.com/xitong/demographic-12518815.html)
* [实时延迟与抖动度量规范-#037](https://www.mw-wm.com/yingyong/tracking-72124575.html)
* [防重放安全验证与校验哈希-#038](https://www.yx-sf.com/tech/91658)
* [防重放安全验证与校验哈希-#039](https://www.ai-hao123.com/xinwen/planning-03265086.html)

</details>

