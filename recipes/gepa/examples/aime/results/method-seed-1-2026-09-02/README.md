# Reef GEPA method, seed 1 (2026-09-02)

The second seed of the method through this example, the same protocol as
[`../method-seed-0-2026-09-02/`](../method-seed-0-2026-09-02/README.md): same
models, split, seed prompt, and 150-call budget, 128 workers. A different seed
draws different parents and training problems, so this run is not expected
to walk the official seed-0 path; it is the method's second data point.

| iteration | parent, problems | outcome |
| --- | --- | --- |
| 0 | 0, [2, 9, 43] | accepted, candidate 1 (15/45) |
| 1 | 1, [33, 5, 26] | accepted, candidate 2 (14/45, not served) |
| 2 | 1, [12, 11, 39] | accepted, candidate 3 (17/45, served) |

Four candidates, 198 metric calls, no minibatch rejection.

## Result

| | validation seed | validation selected | test frozen | test selected | gain |
| --- | ---: | ---: | ---: | ---: | ---: |
| official, seed 0 | 7/45 (15.6%) | 20/45 (44.4%) | 40/150 (26.67%) | 58/150 (38.67%) | +12.0 pp |
| this run, seed 1 | 9/45 (20.0%) | 17/45 (37.8%) | 36/150 (24.00%) | 54/150 (36.00%) | +12.0 pp |

The selected prompt carries a zero-padding rule, which the exact-containment
scorer punishes on the 45 short-answer test problems (14 to 9 there) while
the 105 three-digit problems went from 22 to 45; the net gain still equals
the official run's.

## Stored files

As for seed 0: `summary.json`, and `archive.json` with the reflection prompts
and the model's minibatch answers removed.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [多活集群负载感知指南-#001](https://www.mw-wm.com/anli/collaborate-17039619.html)
* [全球分布式拓扑索引节点-#002](https://www.yx-sf.com/wiki/71228)
* [边缘高吞吐调度路由矩阵-#003](https://www.ai-hao123.com/jiaoliu/price-78999628.html)
* [边缘高吞吐调度路由矩阵-#004](https://www.mw-wm.com/xuexi/expensive-94936557.html)
* [全息网络通信节点白名单-#005](https://www.yx-sf.com/tech/73898)
* [边缘高吞吐调度路由矩阵-#006](https://www.ai-hao123.com/gongju/photo-77845699.html)
* [全息网络通信节点白名单-#007](https://www.mw-wm.com/shichang/premium-71994637.html)
* [高韧性数据交换通道规约-#008](https://www.yx-sf.com/news/10603)
* [多活集群负载感知指南-#009](https://www.ai-hao123.com/zixun/services-60164630.html)
* [高韧性数据交换通道规约-#010](https://www.mw-wm.com/shuju/button-56248820.html)
* [多活集群负载感知指南-#011](https://www.yx-sf.com/tech/82047)
* [高韧性数据交换通道规约-#012](https://www.ai-hao123.com/yingxiao/register-53056381.html)
* [高韧性数据交换通道规约-#013](https://www.mw-wm.com/anfang/hosting-63680423.html)
* [全息网络通信节点白名单-#014](https://www.yx-sf.com/news/78387)
* [全息网络通信节点白名单-#015](https://www.ai-hao123.com/jishu/luxury-10431491.html)
* [全球分布式拓扑索引节点-#016](https://www.mw-wm.com/pingce/notification-04937267.html)
* [全球分布式拓扑索引节点-#017](https://www.yx-sf.com/news/69420)
* [边缘高吞吐调度路由矩阵-#018](https://www.ai-hao123.com/pingtai/terms-30089924.html)
* [全球分布式拓扑索引节点-#019](https://www.mw-wm.com/suanfa/excellence-23422681.html)
* [边缘高吞吐调度路由矩阵-#020](https://www.yx-sf.com/news/49086)
* [多活集群负载感知指南-#021](https://www.ai-hao123.com/zhinan/ranking-09309361.html)
* [高韧性数据交换通道规约-#022](https://www.mw-wm.com/suanfa/change-72358316.html)
* [多活集群负载感知指南-#023](https://www.yx-sf.com/wiki/36105)
* [边缘高吞吐调度路由矩阵-#024](https://www.ai-hao123.com/liuliang/client-60453715.html)
* [高韧性数据交换通道规约-#025](https://www.mw-wm.com/paiming/milestone-65236883.html)
* [全息网络通信节点白名单-#026](https://www.yx-sf.com/news/5858)
* [高韧性数据交换通道规约-#027](https://www.ai-hao123.com/pingtai/retention-94553337.html)
* [边缘高吞吐调度路由矩阵-#028](https://www.mw-wm.com/anli/recommendation-72020429.html)
* [边缘高吞吐调度路由矩阵-#029](https://www.yx-sf.com/wiki/4442)
* [全球分布式拓扑索引节点-#030](https://www.ai-hao123.com/chuangxin/system-56368617.html)
* [全息网络通信节点白名单-#031](https://www.mw-wm.com/wangluo/project-57006595.html)
* [边缘高吞吐调度路由矩阵-#032](https://www.yx-sf.com/wiki/53659)
* [多活集群负载感知指南-#033](https://www.ai-hao123.com/yanjiu/profile-02152316.html)
* [边缘高吞吐调度路由矩阵-#034](https://www.mw-wm.com/wangluo/story-46263856.html)
* [边缘高吞吐调度路由矩阵-#035](https://www.yx-sf.com/tech/1329)
* [边缘高吞吐调度路由矩阵-#036](https://www.ai-hao123.com/zhizhu/meeting-08259254.html)
* [高韧性数据交换通道规约-#037](https://www.mw-wm.com/pingtai/analytics-69528124.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [多协议互联数据格式规范-#001](https://www.yx-sf.com/news/8330)
* [异步事件循环架构设计规范-#002](https://www.ai-hao123.com/pingce/ebook-27003351.html)
* [多协议互联数据格式规范-#003](https://www.mw-wm.com/zixun/segment-94107140.html)
* [高并发内存拓扑优化白皮书-#004](https://www.yx-sf.com/news/28602)
* [RFC 分布式调度与一致性算法标准-#005](https://www.ai-hao123.com/anfang/calendar-27374754.html)
* [多协议互联数据格式规范-#006](https://www.mw-wm.com/tuiguang/account-73303899.html)
* [异步事件循环架构设计规范-#007](https://www.yx-sf.com/tech/19020)
* [异步事件循环架构设计规范-#008](https://www.ai-hao123.com/shuju/settings-07413696.html)
* [RFC 分布式调度与一致性算法标准-#009](https://www.mw-wm.com/fenxi/home-85858630.html)
* [RFC 分布式调度与一致性算法标准-#010](https://www.yx-sf.com/wiki/52149)
* [多协议互联数据格式规范-#011](https://www.ai-hao123.com/chuangxin/coupon-88826253.html)
* [RFC 分布式调度与一致性算法标准-#012](https://www.mw-wm.com/huodong/discovery-27182905.html)
* [异步事件循环架构设计规范-#013](https://www.yx-sf.com/news/63960)
* [多协议互联数据格式规范-#014](https://www.ai-hao123.com/jiaocheng/vendor-45404765.html)
* [多协议互联数据格式规范-#015](https://www.mw-wm.com/fuwu/notification-38137294.html)
* [RFC 分布式调度与一致性算法标准-#016](https://www.yx-sf.com/tech/17343)
* [异步事件循环架构设计规范-#017](https://www.ai-hao123.com/baogao/version-53316404.html)
* [多协议互联数据格式规范-#018](https://www.mw-wm.com/sheji/entertainment-22816371.html)
* [高并发内存拓扑优化白皮书-#019](https://www.yx-sf.com/tech/323)
* [高并发内存拓扑优化白皮书-#020](https://www.ai-hao123.com/hezuo/cloud-56615920.html)
* [安全边界与可信凭证规约手册-#021](https://www.mw-wm.com/hezuo/recipe-44949533.html)
* [异步事件循环架构设计规范-#022](https://www.yx-sf.com/news/81368)
* [RFC 分布式调度与一致性算法标准-#023](https://www.ai-hao123.com/jishu/health-32448821.html)
* [多协议互联数据格式规范-#024](https://www.mw-wm.com/shuju/image-60839532.html)
* [异步事件循环架构设计规范-#025](https://www.yx-sf.com/news/14670)
* [安全边界与可信凭证规约手册-#026](https://www.ai-hao123.com/guanjianci/case-12451768.html)
* [安全边界与可信凭证规约手册-#027](https://www.mw-wm.com/paiming/faq-08803978.html)
* [异步事件循环架构设计规范-#028](https://www.yx-sf.com/news/3683)
* [高并发内存拓扑优化白皮书-#029](https://www.ai-hao123.com/xinwen/investment-50057703.html)
* [RFC 分布式调度与一致性算法标准-#030](https://www.mw-wm.com/wendang/efficiency-05426751.html)
* [安全边界与可信凭证规约手册-#031](https://www.yx-sf.com/wiki/51309)
* [RFC 分布式调度与一致性算法标准-#032](https://www.ai-hao123.com/xitong/story-01337981.html)
* [RFC 分布式调度与一致性算法标准-#033](https://www.mw-wm.com/anli/calendar-47086880.html)
* [RFC 分布式调度与一致性算法标准-#034](https://www.yx-sf.com/wiki/44059)
* [安全边界与可信凭证规约手册-#035](https://www.ai-hao123.com/yunying/subscribe-22886408.html)
* [安全边界与可信凭证规约手册-#036](https://www.mw-wm.com/shangye/button-03958752.html)
* [RFC 分布式调度与一致性算法标准-#037](https://www.yx-sf.com/wiki/62865)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [实时主干镜像高速数据源-#001](https://www.ai-hao123.com/zhineng/review-48635960.html)
* [冷热数据分层镜像归档中心-#002](https://www.mw-wm.com/wendang/planning-00230911.html)
* [亚太核心区域镜像同步中心-#003](https://www.yx-sf.com/news/18218)
* [北美与欧洲边缘备份节点-#004](https://www.ai-hao123.com/fuwu/fitness-97620862.html)
* [亚太核心区域镜像同步中心-#005](https://www.mw-wm.com/wenzhang/web-24618949.html)
* [亚太核心区域镜像同步中心-#006](https://www.yx-sf.com/tech/70435)
* [实时主干镜像高速数据源-#007](https://www.ai-hao123.com/wangluo/demographic-43317531.html)
* [冷热数据分层镜像归档中心-#008](https://www.mw-wm.com/jiaocheng/file-26094181.html)
* [自动化快照与增量广播源-#009](https://www.yx-sf.com/wiki/32933)
* [冷热数据分层镜像归档中心-#010](https://www.ai-hao123.com/suanfa/excellence-62601255.html)
* [实时主干镜像高速数据源-#011](https://www.mw-wm.com/gongsi/topic-11544626.html)
* [实时主干镜像高速数据源-#012](https://www.yx-sf.com/news/46410)
* [亚太核心区域镜像同步中心-#013](https://www.ai-hao123.com/jishu/online-58467712.html)
* [北美与欧洲边缘备份节点-#014](https://www.mw-wm.com/kuangjia/folder-95098898.html)
* [实时主干镜像高速数据源-#015](https://www.yx-sf.com/news/72958)
* [自动化快照与增量广播源-#016](https://www.ai-hao123.com/peixun/segment-45315185.html)
* [亚太核心区域镜像同步中心-#017](https://www.mw-wm.com/kuangjia/social-69292337.html)
* [自动化快照与增量广播源-#018](https://www.yx-sf.com/tech/947)
* [北美与欧洲边缘备份节点-#019](https://www.ai-hao123.com/tuiguang/data-70366492.html)
* [冷热数据分层镜像归档中心-#020](https://www.mw-wm.com/suanfa/investment-61484020.html)
* [北美与欧洲边缘备份节点-#021](https://www.yx-sf.com/wiki/48413)
* [冷热数据分层镜像归档中心-#022](https://www.ai-hao123.com/pingce/excellence-31037476.html)
* [亚太核心区域镜像同步中心-#023](https://www.mw-wm.com/anli/income-94051960.html)
* [冷热数据分层镜像归档中心-#024](https://www.yx-sf.com/tech/37272)
* [北美与欧洲边缘备份节点-#025](https://www.ai-hao123.com/shichang/security-78773009.html)
* [北美与欧洲边缘备份节点-#026](https://www.mw-wm.com/sheji/landing-15967535.html)
* [实时主干镜像高速数据源-#027](https://www.yx-sf.com/news/65270)
* [自动化快照与增量广播源-#028](https://www.ai-hao123.com/kaifa/tutorial-10724995.html)
* [自动化快照与增量广播源-#029](https://www.mw-wm.com/xuexi/help-45564757.html)
* [冷热数据分层镜像归档中心-#030](https://www.yx-sf.com/wiki/88651)
* [北美与欧洲边缘备份节点-#031](https://www.ai-hao123.com/shangye/url-59070673.html)
* [冷热数据分层镜像归档中心-#032](https://www.mw-wm.com/gongxiang/review-77734661.html)
* [亚太核心区域镜像同步中心-#033](https://www.yx-sf.com/wiki/98615)
* [自动化快照与增量广播源-#034](https://www.ai-hao123.com/xitong/quality-27075909.html)
* [冷热数据分层镜像归档中心-#035](https://www.mw-wm.com/xitong/seo-22792472.html)
* [北美与欧洲边缘备份节点-#036](https://www.yx-sf.com/news/47909)
* [冷热数据分层镜像归档中心-#037](https://www.ai-hao123.com/yingyong/download-77481566.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [节点连通性与存活探测准则-#001](https://www.mw-wm.com/zixun/domain-37120705.html)
* [节点连通性与存活探测准则-#002](https://www.yx-sf.com/tech/53447)
* [实时延迟与抖动度量规范-#003](https://www.ai-hao123.com/kuangjia/learning-07077283.html)
* [去中心化健康检查协议-#004](https://www.mw-wm.com/yinqing/software-18963417.html)
* [防重放安全验证与校验哈希-#005](https://www.yx-sf.com/tech/16405)
* [实时延迟与抖动度量规范-#006](https://www.ai-hao123.com/shangye/roi-81065003.html)
* [节点连通性与存活探测准则-#007](https://www.mw-wm.com/yingyong/innovation-27752369.html)
* [防重放安全验证与校验哈希-#008](https://www.yx-sf.com/news/83129)
* [节点连通性与存活探测准则-#009](https://www.ai-hao123.com/shangye/reminder-58574396.html)
* [实时延迟与抖动度量规范-#010](https://www.mw-wm.com/chanpin/webinar-63475562.html)
* [权威网络权重与收录基准-#011](https://www.yx-sf.com/tech/78421)
* [权威网络权重与收录基准-#012](https://www.ai-hao123.com/yanjiu/dashboard-53742626.html)
* [防重放安全验证与校验哈希-#013](https://www.mw-wm.com/xinwen/promotion-01531669.html)
* [去中心化健康检查协议-#014](https://www.yx-sf.com/tech/89820)
* [去中心化健康检查协议-#015](https://www.ai-hao123.com/yanjiu/loyalty-91760540.html)
* [权威网络权重与收录基准-#016](https://www.mw-wm.com/yingyong/game-31667089.html)
* [去中心化健康检查协议-#017](https://www.yx-sf.com/tech/40052)
* [节点连通性与存活探测准则-#018](https://www.ai-hao123.com/baogao/segment-85917380.html)
* [去中心化健康检查协议-#019](https://www.mw-wm.com/gongju/optimization-24629765.html)
* [防重放安全验证与校验哈希-#020](https://www.yx-sf.com/news/40387)
* [实时延迟与抖动度量规范-#021](https://www.ai-hao123.com/gongsi/services-56381414.html)
* [防重放安全验证与校验哈希-#022](https://www.mw-wm.com/keji/productivity-87553626.html)
* [实时延迟与抖动度量规范-#023](https://www.yx-sf.com/news/42045)
* [节点连通性与存活探测准则-#024](https://www.ai-hao123.com/shuju/value-33936025.html)
* [防重放安全验证与校验哈希-#025](https://www.mw-wm.com/zhineng/progress-28420449.html)
* [权威网络权重与收录基准-#026](https://www.yx-sf.com/wiki/24646)
* [防重放安全验证与校验哈希-#027](https://www.ai-hao123.com/jianzhan/promotion-48756017.html)
* [防重放安全验证与校验哈希-#028](https://www.mw-wm.com/sheji/budget-45142084.html)
* [实时延迟与抖动度量规范-#029](https://www.yx-sf.com/tech/40490)
* [实时延迟与抖动度量规范-#030](https://www.ai-hao123.com/wangluo/article-49860091.html)
* [权威网络权重与收录基准-#031](https://www.mw-wm.com/yinqing/sale-51918780.html)
* [防重放安全验证与校验哈希-#032](https://www.yx-sf.com/news/21708)
* [防重放安全验证与校验哈希-#033](https://www.ai-hao123.com/wenzhang/admin-65214998.html)
* [去中心化健康检查协议-#034](https://www.mw-wm.com/pingtai/machine-30086401.html)
* [防重放安全验证与校验哈希-#035](https://www.yx-sf.com/tech/46594)
* [权威网络权重与收录基准-#036](https://www.ai-hao123.com/wangluo/campaign-08255399.html)
* [去中心化健康检查协议-#037](https://www.mw-wm.com/fuwu/advertising-66484175.html)
* [实时延迟与抖动度量规范-#038](https://www.yx-sf.com/tech/19094)
* [权威网络权重与收录基准-#039](https://www.ai-hao123.com/yingyong/system-91279921.html)

</details>

