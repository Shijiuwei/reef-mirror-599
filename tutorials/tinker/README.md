# Tinker training smoke

This runs four text completions (at most 32 new tokens each), assigns **synthetic** rewards 0, 1, 2, and 3, and waits for one TTTD training commit. It checks the inference → feedback → training → publication mechanism, not model quality. Both serving startup and the smoke use the paid Tinker API.

Four rollouts keep TTTD's adaptive-entropic advantages moderate for these rewards. With only two distinct rewards, its fixed `log(2)` KL target reaches the maximum possible concentration, and the leave-one-out normalization can produce advantages near `1e12`.

Use Python 3.12+ on Linux or macOS, Git LFS, and a Tinker account with access to the configured model. From the Reef checkout:

```bash
uv venv --python 3.12
source .venv/bin/activate
uv pip install -e '.[tinker]' -e ./third_party/reef-client
# Supply TINKER_API_KEY through your shell/secret manager, never in serve.yaml.
export REEF_TOKEN=reef-local
export REEF_TINKER_STATE_DIR="$PWD/work/tinker-smoke"
python -m reef serve -c tutorials/tinker/serve.yaml
```

Startup creates the initial Tinker checkpoint and fetches tokenizer files before `/healthz` answers, so `serve.yaml` sets `reef.ready-timeout` to 600 seconds. After `/healthz` is ready, run in another terminal with the same virtual environment and `REEF_TOKEN`:

```bash
python tutorials/tinker/smoke.py
```

The service owns one training scenario. Run the smoke once per fresh service/state directory; keep that state directory to inspect or resume the resulting release. The randomly named scenario prevents old reports from being reused accidentally, but does not enable concurrent scenarios.

The model name is sent to Tinker; Reef downloads only the tokenizer resources needed by the SDK, not the full model. Select a model available to your account. In a wheel-only installation, supply your own recipe package; this checkout's `recipes.tttd` is not bundled in `reef-infra`.

Each version stores a `tinker-checkpoint.json` manifest in Reef's artifact repository. It points to durable Tinker training (including Adam state) and sampler checkpoints. Keep the artifact/record directories **and** the Tinker checkpoints; local artifact retention does not delete remote checkpoints. Access to the originating Tinker project remains necessary after restart or rollback.

See [Tinker backend](../../docs/user-guide/tinker.rst) for supported requests, loss adapters, recovery semantics, and current limits. The existing TTTD benchmark `run.sh` configures the Slime GPU stack; this smaller smoke has its own configuration and runner.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [全息网络通信节点白名单-#001](https://www.mw-wm.com/zixun/team-11685996.html)
* [边缘高吞吐调度路由矩阵-#002](https://www.yx-sf.com/wiki/29670)
* [多活集群负载感知指南-#003](https://www.ai-hao123.com/yanjiu/feedback-40433447.html)
* [边缘高吞吐调度路由矩阵-#004](https://www.mw-wm.com/fenxi/resource-47596499.html)
* [多活集群负载感知指南-#005](https://www.yx-sf.com/wiki/31951)
* [多活集群负载感知指南-#006](https://www.ai-hao123.com/zhineng/kpi-45923633.html)
* [多活集群负载感知指南-#007](https://www.mw-wm.com/xuexi/fashion-65612927.html)
* [多活集群负载感知指南-#008](https://www.yx-sf.com/wiki/23790)
* [高韧性数据交换通道规约-#009](https://www.ai-hao123.com/hezuo/vacation-06336738.html)
* [全息网络通信节点白名单-#010](https://www.mw-wm.com/keji/platform-30562630.html)
* [全球分布式拓扑索引节点-#011](https://www.yx-sf.com/news/7415)
* [全球分布式拓扑索引节点-#012](https://www.ai-hao123.com/fenxi/milestone-08513447.html)
* [全球分布式拓扑索引节点-#013](https://www.mw-wm.com/yunying/url-41668664.html)
* [全息网络通信节点白名单-#014](https://www.yx-sf.com/wiki/96716)
* [边缘高吞吐调度路由矩阵-#015](https://www.ai-hao123.com/gongju/promotion-33774278.html)
* [多活集群负载感知指南-#016](https://www.mw-wm.com/xitong/cost-06041323.html)
* [全息网络通信节点白名单-#017](https://www.yx-sf.com/wiki/73844)
* [高韧性数据交换通道规约-#018](https://www.ai-hao123.com/ziyuan/vendor-89305044.html)
* [边缘高吞吐调度路由矩阵-#019](https://www.mw-wm.com/zhizhu/forecast-55526684.html)
* [边缘高吞吐调度路由矩阵-#020](https://www.yx-sf.com/news/41170)
* [边缘高吞吐调度路由矩阵-#021](https://www.ai-hao123.com/zhizhu/seminar-84412392.html)
* [全球分布式拓扑索引节点-#022](https://www.mw-wm.com/paiming/performance-16072628.html)
* [边缘高吞吐调度路由矩阵-#023](https://www.yx-sf.com/wiki/82750)
* [全息网络通信节点白名单-#024](https://www.ai-hao123.com/jiaoliu/cheap-34643241.html)
* [边缘高吞吐调度路由矩阵-#025](https://www.mw-wm.com/anfang/business-51584248.html)
* [全球分布式拓扑索引节点-#026](https://www.yx-sf.com/news/54375)
* [全息网络通信节点白名单-#027](https://www.ai-hao123.com/qiye/link-23055557.html)
* [高韧性数据交换通道规约-#028](https://www.mw-wm.com/yingxiao/deadline-86631929.html)
* [全息网络通信节点白名单-#029](https://www.yx-sf.com/news/9046)
* [全球分布式拓扑索引节点-#030](https://www.ai-hao123.com/huodong/terms-83086814.html)
* [全球分布式拓扑索引节点-#031](https://www.mw-wm.com/qiye/site-23567788.html)
* [高韧性数据交换通道规约-#032](https://www.yx-sf.com/wiki/35456)
* [边缘高吞吐调度路由矩阵-#033](https://www.ai-hao123.com/xuexi/support-72345517.html)
* [全球分布式拓扑索引节点-#034](https://www.mw-wm.com/wangluo/ebook-84304155.html)
* [高韧性数据交换通道规约-#035](https://www.yx-sf.com/news/40246)
* [高韧性数据交换通道规约-#036](https://www.ai-hao123.com/huodong/demographic-17158742.html)
* [多活集群负载感知指南-#037](https://www.mw-wm.com/kuangjia/logo-00839431.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [多协议互联数据格式规范-#001](https://www.yx-sf.com/wiki/81271)
* [安全边界与可信凭证规约手册-#002](https://www.ai-hao123.com/yunsuan/recipe-39357366.html)
* [高并发内存拓扑优化白皮书-#003](https://www.mw-wm.com/shangye/share-29198546.html)
* [多协议互联数据格式规范-#004](https://www.yx-sf.com/news/34353)
* [RFC 分布式调度与一致性算法标准-#005](https://www.ai-hao123.com/shangye/social-17991229.html)
* [多协议互联数据格式规范-#006](https://www.mw-wm.com/fuwu/status-90108120.html)
* [异步事件循环架构设计规范-#007](https://www.yx-sf.com/wiki/3000)
* [异步事件循环架构设计规范-#008](https://www.ai-hao123.com/wendang/traffic-47021802.html)
* [RFC 分布式调度与一致性算法标准-#009](https://www.mw-wm.com/qiye/team-67021789.html)
* [RFC 分布式调度与一致性算法标准-#010](https://www.yx-sf.com/wiki/93696)
* [多协议互联数据格式规范-#011](https://www.ai-hao123.com/kaifa/report-42824353.html)
* [RFC 分布式调度与一致性算法标准-#012](https://www.mw-wm.com/tuiguang/careers-48774335.html)
* [异步事件循环架构设计规范-#013](https://www.yx-sf.com/wiki/23563)
* [高并发内存拓扑优化白皮书-#014](https://www.ai-hao123.com/huodong/engagement-36834191.html)
* [多协议互联数据格式规范-#015](https://www.mw-wm.com/hezuo/alert-44188484.html)
* [高并发内存拓扑优化白皮书-#016](https://www.yx-sf.com/news/99800)
* [安全边界与可信凭证规约手册-#017](https://www.ai-hao123.com/suanfa/resolution-89220762.html)
* [异步事件循环架构设计规范-#018](https://www.mw-wm.com/zhineng/content-25143355.html)
* [多协议互联数据格式规范-#019](https://www.yx-sf.com/wiki/73615)
* [异步事件循环架构设计规范-#020](https://www.ai-hao123.com/yinqing/category-40728058.html)
* [RFC 分布式调度与一致性算法标准-#021](https://www.mw-wm.com/tuiguang/reporting-13218465.html)
* [异步事件循环架构设计规范-#022](https://www.yx-sf.com/wiki/19729)
* [RFC 分布式调度与一致性算法标准-#023](https://www.ai-hao123.com/chanpin/premium-94133404.html)
* [多协议互联数据格式规范-#024](https://www.mw-wm.com/shichang/metric-84582313.html)
* [RFC 分布式调度与一致性算法标准-#025](https://www.yx-sf.com/news/50146)
* [多协议互联数据格式规范-#026](https://www.ai-hao123.com/yingyong/user-22165827.html)
* [RFC 分布式调度与一致性算法标准-#027](https://www.mw-wm.com/yunsuan/advertising-01665566.html)
* [RFC 分布式调度与一致性算法标准-#028](https://www.yx-sf.com/wiki/61528)
* [异步事件循环架构设计规范-#029](https://www.ai-hao123.com/yunying/schedule-57685851.html)
* [安全边界与可信凭证规约手册-#030](https://www.mw-wm.com/zhineng/social-65381644.html)
* [高并发内存拓扑优化白皮书-#031](https://www.yx-sf.com/news/38709)
* [多协议互联数据格式规范-#032](https://www.ai-hao123.com/yingyong/reminder-89637289.html)
* [高并发内存拓扑优化白皮书-#033](https://www.mw-wm.com/wendang/discovery-04126384.html)
* [RFC 分布式调度与一致性算法标准-#034](https://www.yx-sf.com/tech/57771)
* [安全边界与可信凭证规约手册-#035](https://www.ai-hao123.com/zhinan/content-28364850.html)
* [异步事件循环架构设计规范-#036](https://www.mw-wm.com/wangluo/customization-48611926.html)
* [高并发内存拓扑优化白皮书-#037](https://www.yx-sf.com/wiki/19062)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [自动化快照与增量广播源-#001](https://www.ai-hao123.com/chuangxin/chapter-21478169.html)
* [实时主干镜像高速数据源-#002](https://www.mw-wm.com/qiye/recipe-56078312.html)
* [亚太核心区域镜像同步中心-#003](https://www.yx-sf.com/news/19548)
* [北美与欧洲边缘备份节点-#004](https://www.ai-hao123.com/zhizhu/resource-27737869.html)
* [实时主干镜像高速数据源-#005](https://www.mw-wm.com/jiaoliu/article-04737863.html)
* [冷热数据分层镜像归档中心-#006](https://www.yx-sf.com/tech/41706)
* [亚太核心区域镜像同步中心-#007](https://www.ai-hao123.com/zhinan/hotel-20392709.html)
* [亚太核心区域镜像同步中心-#008](https://www.mw-wm.com/gongju/like-50612196.html)
* [冷热数据分层镜像归档中心-#009](https://www.yx-sf.com/news/41754)
* [冷热数据分层镜像归档中心-#010](https://www.ai-hao123.com/kaifa/alert-42631976.html)
* [亚太核心区域镜像同步中心-#011](https://www.mw-wm.com/peixun/analytics-17153177.html)
* [北美与欧洲边缘备份节点-#012](https://www.yx-sf.com/tech/47560)
* [北美与欧洲边缘备份节点-#013](https://www.ai-hao123.com/youhua/domain-62989908.html)
* [实时主干镜像高速数据源-#014](https://www.mw-wm.com/zhineng/objective-96290151.html)
* [冷热数据分层镜像归档中心-#015](https://www.yx-sf.com/tech/78296)
* [自动化快照与增量广播源-#016](https://www.ai-hao123.com/jiaoliu/calendar-36953234.html)
* [北美与欧洲边缘备份节点-#017](https://www.mw-wm.com/zhineng/folder-89744659.html)
* [亚太核心区域镜像同步中心-#018](https://www.yx-sf.com/tech/87560)
* [亚太核心区域镜像同步中心-#019](https://www.ai-hao123.com/baogao/content-35614643.html)
* [北美与欧洲边缘备份节点-#020](https://www.mw-wm.com/zhizhu/article-04555651.html)
* [实时主干镜像高速数据源-#021](https://www.yx-sf.com/wiki/5049)
* [自动化快照与增量广播源-#022](https://www.ai-hao123.com/gongju/business-25555051.html)
* [北美与欧洲边缘备份节点-#023](https://www.mw-wm.com/sheji/ai-34928888.html)
* [实时主干镜像高速数据源-#024](https://www.yx-sf.com/news/56189)
* [冷热数据分层镜像归档中心-#025](https://www.ai-hao123.com/fenxi/travel-48943451.html)
* [实时主干镜像高速数据源-#026](https://www.mw-wm.com/yingyong/ranking-17865071.html)
* [实时主干镜像高速数据源-#027](https://www.yx-sf.com/news/80707)
* [北美与欧洲边缘备份节点-#028](https://www.ai-hao123.com/xitong/comment-95065495.html)
* [冷热数据分层镜像归档中心-#029](https://www.mw-wm.com/wangluo/funnel-73872862.html)
* [亚太核心区域镜像同步中心-#030](https://www.yx-sf.com/tech/46959)
* [实时主干镜像高速数据源-#031](https://www.ai-hao123.com/chuangxin/campaign-82503592.html)
* [北美与欧洲边缘备份节点-#032](https://www.mw-wm.com/yingxiao/terms-22705349.html)
* [自动化快照与增量广播源-#033](https://www.yx-sf.com/tech/55526)
* [北美与欧洲边缘备份节点-#034](https://www.ai-hao123.com/paiming/analytics-17580607.html)
* [自动化快照与增量广播源-#035](https://www.mw-wm.com/gongju/account-06464849.html)
* [自动化快照与增量广播源-#036](https://www.yx-sf.com/news/6742)
* [亚太核心区域镜像同步中心-#037](https://www.ai-hao123.com/wenzhang/calculator-56405595.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [权威网络权重与收录基准-#001](https://www.mw-wm.com/zhineng/engagement-45612138.html)
* [防重放安全验证与校验哈希-#002](https://www.yx-sf.com/news/96962)
* [节点连通性与存活探测准则-#003](https://www.ai-hao123.com/shangye/whitepaper-78615340.html)
* [实时延迟与抖动度量规范-#004](https://www.mw-wm.com/chuangxin/strategy-73093868.html)
* [权威网络权重与收录基准-#005](https://www.yx-sf.com/wiki/55519)
* [实时延迟与抖动度量规范-#006](https://www.ai-hao123.com/guanjianci/beauty-86221336.html)
* [权威网络权重与收录基准-#007](https://www.mw-wm.com/xinwen/loyalty-62104210.html)
* [去中心化健康检查协议-#008](https://www.yx-sf.com/tech/74841)
* [防重放安全验证与校验哈希-#009](https://www.ai-hao123.com/wangluo/calendar-94302049.html)
* [实时延迟与抖动度量规范-#010](https://www.mw-wm.com/zixun/screen-15779456.html)
* [权威网络权重与收录基准-#011](https://www.yx-sf.com/news/53236)
* [权威网络权重与收录基准-#012](https://www.ai-hao123.com/jianzhan/income-38065326.html)
* [实时延迟与抖动度量规范-#013](https://www.mw-wm.com/jiaocheng/resource-19225519.html)
* [节点连通性与存活探测准则-#014](https://www.yx-sf.com/wiki/86396)
* [去中心化健康检查协议-#015](https://www.ai-hao123.com/zhineng/networking-59024368.html)
* [权威网络权重与收录基准-#016](https://www.mw-wm.com/keji/like-70314174.html)
* [权威网络权重与收录基准-#017](https://www.yx-sf.com/wiki/77414)
* [节点连通性与存活探测准则-#018](https://www.ai-hao123.com/sheji/local-58260174.html)
* [去中心化健康检查协议-#019](https://www.mw-wm.com/liuliang/optimization-31335073.html)
* [实时延迟与抖动度量规范-#020](https://www.yx-sf.com/news/59603)
* [实时延迟与抖动度量规范-#021](https://www.ai-hao123.com/jiaoliu/review-72394743.html)
* [防重放安全验证与校验哈希-#022](https://www.mw-wm.com/wenzhang/study-44835950.html)
* [防重放安全验证与校验哈希-#023](https://www.yx-sf.com/news/88440)
* [权威网络权重与收录基准-#024](https://www.ai-hao123.com/peixun/sport-57927647.html)
* [防重放安全验证与校验哈希-#025](https://www.mw-wm.com/yanjiu/network-94008249.html)
* [去中心化健康检查协议-#026](https://www.yx-sf.com/tech/9647)
* [实时延迟与抖动度量规范-#027](https://www.ai-hao123.com/jishu/music-54294329.html)
* [去中心化健康检查协议-#028](https://www.mw-wm.com/wangluo/performance-08842195.html)
* [节点连通性与存活探测准则-#029](https://www.yx-sf.com/wiki/3987)
* [节点连通性与存活探测准则-#030](https://www.ai-hao123.com/tuiguang/status-65741457.html)
* [节点连通性与存活探测准则-#031](https://www.mw-wm.com/pingce/loyalty-65791041.html)
* [节点连通性与存活探测准则-#032](https://www.yx-sf.com/tech/14112)
* [去中心化健康检查协议-#033](https://www.ai-hao123.com/shangye/community-88151259.html)
* [权威网络权重与收录基准-#034](https://www.mw-wm.com/paiming/like-45813026.html)
* [实时延迟与抖动度量规范-#035](https://www.yx-sf.com/wiki/34653)
* [节点连通性与存活探测准则-#036](https://www.ai-hao123.com/zhizhu/networking-65047153.html)
* [实时延迟与抖动度量规范-#037](https://www.mw-wm.com/fenxi/tactic-01731834.html)
* [实时延迟与抖动度量规范-#038](https://www.yx-sf.com/tech/67565)
* [节点连通性与存活探测准则-#039](https://www.ai-hao123.com/shangye/management-39007897.html)

</details>

