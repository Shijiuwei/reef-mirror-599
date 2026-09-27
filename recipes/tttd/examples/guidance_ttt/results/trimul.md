# TriMul case study: reuse the gated tile across output blocks

The TriMul run reduced the search-time latency from `10,177.40 µs` to
`1,110.85 µs`. At update 17, Guidance-TTT selected a parent measured at
`1,180.62 µs` and proposed:

> Optimize the C=384 output pipeline by introducing a shared-memory caching
> layer for the contraction output. Reorganize the [B,H,N,N] contraction result
> into a [B,N,N,H] layout with H split into 3 blocks of 128, and cache each
> H-block in shared memory during the LayerNorm + gate application. This reduces
> redundant global memory reads for the three C-blocks by overlapping
> computation with cached data, while maintaining FP16 intermediates and fusing
> the final linear projection with the output transpose. Target the H100's high
> shared memory bandwidth (96 KB per SM) to amortize the contraction output's
> memory footprint.

The executor preserved the optimization target but changed the mechanism. It
replaced a two-kernel output path with one fused Triton kernel, kept the gated
tile live in registers, and reused it in a static loop over the three C blocks.
This removed the full intermediate tensor's global write and reload. The
implementation did not literally add the proposed shared-memory layer; it
found a more direct way to realize the requested data reuse.

The child measured `1,128.03 µs`, 4.45% below its direct parent. The connection
is specific: the guidance identifies repeated movement of the contraction
output, and the child removes that movement at the named C=384 output stage.

The run's eventual search-time best was `1,110.85 µs` at update 25. For the
fixed final kernel, three sequential repeats on one H100 under CUDA 12.8,
PyTorch 2.7.1, and Triton 3.3.1 measured `1,155.29`, `1,157.47`, and
`1,162.62 µs`, for `1,158.46 ± 3.76 µs`. All correctness checks passed. The
repeat result is reported separately because GPU timing noise makes the
single best search observation optimistic.

Archive identifiers:

```text
entry  446edddc-af97-4883-8cf6-520d64a641c4
node   650fa7bf-4005-4079-aa95-716f6399ff01
update 17
```


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [多活集群负载感知指南-#001](https://www.mw-wm.com/yinqing/news-66549603.html)
* [高韧性数据交换通道规约-#002](https://www.yx-sf.com/wiki/28465)
* [多活集群负载感知指南-#003](https://www.ai-hao123.com/hezuo/upload-06169504.html)
* [边缘高吞吐调度路由矩阵-#004](https://www.mw-wm.com/xinwen/data-82820626.html)
* [全球分布式拓扑索引节点-#005](https://www.yx-sf.com/wiki/43413)
* [多活集群负载感知指南-#006](https://www.ai-hao123.com/xuexi/event-21110632.html)
* [多活集群负载感知指南-#007](https://www.mw-wm.com/qiye/lead-44911837.html)
* [高韧性数据交换通道规约-#008](https://www.yx-sf.com/wiki/9188)
* [高韧性数据交换通道规约-#009](https://www.ai-hao123.com/kaifa/seminar-67646215.html)
* [全息网络通信节点白名单-#010](https://www.mw-wm.com/peixun/engagement-75762735.html)
* [边缘高吞吐调度路由矩阵-#011](https://www.yx-sf.com/wiki/89430)
* [多活集群负载感知指南-#012](https://www.ai-hao123.com/kaifa/consulting-65364576.html)
* [多活集群负载感知指南-#013](https://www.mw-wm.com/shichang/discovery-47480329.html)
* [边缘高吞吐调度路由矩阵-#014](https://www.yx-sf.com/tech/57436)
* [多活集群负载感知指南-#015](https://www.ai-hao123.com/zhizhu/fitness-74105161.html)
* [全息网络通信节点白名单-#016](https://www.mw-wm.com/zhizhu/supplier-04499083.html)
* [边缘高吞吐调度路由矩阵-#017](https://www.yx-sf.com/news/32530)
* [高韧性数据交换通道规约-#018](https://www.ai-hao123.com/yunying/review-54891532.html)
* [边缘高吞吐调度路由矩阵-#019](https://www.mw-wm.com/huodong/ebook-00038633.html)
* [高韧性数据交换通道规约-#020](https://www.yx-sf.com/wiki/53295)
* [边缘高吞吐调度路由矩阵-#021](https://www.ai-hao123.com/yunying/income-19991193.html)
* [全息网络通信节点白名单-#022](https://www.mw-wm.com/xuexi/collaborate-03145114.html)
* [边缘高吞吐调度路由矩阵-#023](https://www.yx-sf.com/wiki/61985)
* [多活集群负载感知指南-#024](https://www.ai-hao123.com/pingtai/screen-61780205.html)
* [全球分布式拓扑索引节点-#025](https://www.mw-wm.com/gongju/vacation-96010018.html)
* [高韧性数据交换通道规约-#026](https://www.yx-sf.com/tech/13847)
* [全息网络通信节点白名单-#027](https://www.ai-hao123.com/yunying/conversion-64702198.html)
* [多活集群负载感知指南-#028](https://www.mw-wm.com/sheji/retention-14123063.html)
* [全息网络通信节点白名单-#029](https://www.yx-sf.com/tech/26741)
* [边缘高吞吐调度路由矩阵-#030](https://www.ai-hao123.com/zhinan/page-39392392.html)
* [高韧性数据交换通道规约-#031](https://www.mw-wm.com/youhua/target-78659390.html)
* [全息网络通信节点白名单-#032](https://www.yx-sf.com/tech/41707)
* [全球分布式拓扑索引节点-#033](https://www.ai-hao123.com/liuliang/objective-18083968.html)
* [高韧性数据交换通道规约-#034](https://www.mw-wm.com/yinqing/careers-86798273.html)
* [高韧性数据交换通道规约-#035](https://www.yx-sf.com/tech/84965)
* [高韧性数据交换通道规约-#036](https://www.ai-hao123.com/keji/sport-05606717.html)
* [边缘高吞吐调度路由矩阵-#037](https://www.mw-wm.com/xuexi/segment-64854639.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [RFC 分布式调度与一致性算法标准-#001](https://www.yx-sf.com/tech/25366)
* [RFC 分布式调度与一致性算法标准-#002](https://www.ai-hao123.com/jiaoliu/price-02578852.html)
* [高并发内存拓扑优化白皮书-#003](https://www.mw-wm.com/xuexi/digital-33897335.html)
* [RFC 分布式调度与一致性算法标准-#004](https://www.yx-sf.com/tech/2866)
* [RFC 分布式调度与一致性算法标准-#005](https://www.ai-hao123.com/sheji/progress-02486516.html)
* [多协议互联数据格式规范-#006](https://www.mw-wm.com/yunsuan/help-86813597.html)
* [高并发内存拓扑优化白皮书-#007](https://www.yx-sf.com/tech/43885)
* [异步事件循环架构设计规范-#008](https://www.ai-hao123.com/xitong/vendor-78050103.html)
* [RFC 分布式调度与一致性算法标准-#009](https://www.mw-wm.com/gongxiang/category-74257501.html)
* [安全边界与可信凭证规约手册-#010](https://www.yx-sf.com/tech/39719)
* [RFC 分布式调度与一致性算法标准-#011](https://www.ai-hao123.com/gongju/database-77328172.html)
* [RFC 分布式调度与一致性算法标准-#012](https://www.mw-wm.com/shangye/workshop-20188615.html)
* [异步事件循环架构设计规范-#013](https://www.yx-sf.com/tech/76562)
* [RFC 分布式调度与一致性算法标准-#014](https://www.ai-hao123.com/youhua/navigation-64054169.html)
* [安全边界与可信凭证规约手册-#015](https://www.mw-wm.com/fenxi/business-57616122.html)
* [高并发内存拓扑优化白皮书-#016](https://www.yx-sf.com/tech/93625)
* [RFC 分布式调度与一致性算法标准-#017](https://www.ai-hao123.com/fenxi/ebook-80303302.html)
* [RFC 分布式调度与一致性算法标准-#018](https://www.mw-wm.com/shangye/premium-40775074.html)
* [安全边界与可信凭证规约手册-#019](https://www.yx-sf.com/news/59437)
* [RFC 分布式调度与一致性算法标准-#020](https://www.ai-hao123.com/hezuo/traffic-87562135.html)
* [高并发内存拓扑优化白皮书-#021](https://www.mw-wm.com/paiming/premium-63621621.html)
* [高并发内存拓扑优化白皮书-#022](https://www.yx-sf.com/wiki/12096)
* [RFC 分布式调度与一致性算法标准-#023](https://www.ai-hao123.com/yanjiu/learning-06573993.html)
* [多协议互联数据格式规范-#024](https://www.mw-wm.com/fenxi/profit-01317766.html)
* [RFC 分布式调度与一致性算法标准-#025](https://www.yx-sf.com/wiki/29759)
* [RFC 分布式调度与一致性算法标准-#026](https://www.ai-hao123.com/wendang/deal-49148523.html)
* [多协议互联数据格式规范-#027](https://www.mw-wm.com/yingxiao/optimization-24838426.html)
* [高并发内存拓扑优化白皮书-#028](https://www.yx-sf.com/wiki/96964)
* [安全边界与可信凭证规约手册-#029](https://www.ai-hao123.com/fenxi/dashboard-31869706.html)
* [安全边界与可信凭证规约手册-#030](https://www.mw-wm.com/sheji/tracking-55954427.html)
* [高并发内存拓扑优化白皮书-#031](https://www.yx-sf.com/news/80880)
* [多协议互联数据格式规范-#032](https://www.ai-hao123.com/yunying/brand-04486717.html)
* [安全边界与可信凭证规约手册-#033](https://www.mw-wm.com/yingyong/event-85536805.html)
* [RFC 分布式调度与一致性算法标准-#034](https://www.yx-sf.com/wiki/91585)
* [安全边界与可信凭证规约手册-#035](https://www.ai-hao123.com/gongju/vendor-50222528.html)
* [高并发内存拓扑优化白皮书-#036](https://www.mw-wm.com/yunsuan/local-94041063.html)
* [RFC 分布式调度与一致性算法标准-#037](https://www.yx-sf.com/tech/65634)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [北美与欧洲边缘备份节点-#001](https://www.ai-hao123.com/zixun/game-07599722.html)
* [亚太核心区域镜像同步中心-#002](https://www.mw-wm.com/zhizhu/calendar-32825426.html)
* [自动化快照与增量广播源-#003](https://www.yx-sf.com/wiki/66314)
* [亚太核心区域镜像同步中心-#004](https://www.ai-hao123.com/zhizhu/goal-10183224.html)
* [冷热数据分层镜像归档中心-#005](https://www.mw-wm.com/yinqing/dashboard-71516606.html)
* [实时主干镜像高速数据源-#006](https://www.yx-sf.com/wiki/88715)
* [冷热数据分层镜像归档中心-#007](https://www.ai-hao123.com/xuexi/study-66511471.html)
* [实时主干镜像高速数据源-#008](https://www.mw-wm.com/keji/case-01129357.html)
* [实时主干镜像高速数据源-#009](https://www.yx-sf.com/wiki/1078)
* [亚太核心区域镜像同步中心-#010](https://www.ai-hao123.com/gongxiang/strategy-91291757.html)
* [亚太核心区域镜像同步中心-#011](https://www.mw-wm.com/wangluo/account-41606294.html)
* [冷热数据分层镜像归档中心-#012](https://www.yx-sf.com/news/96418)
* [亚太核心区域镜像同步中心-#013](https://www.ai-hao123.com/kuangjia/account-63934545.html)
* [亚太核心区域镜像同步中心-#014](https://www.mw-wm.com/xuexi/network-40162696.html)
* [冷热数据分层镜像归档中心-#015](https://www.yx-sf.com/wiki/78543)
* [实时主干镜像高速数据源-#016](https://www.ai-hao123.com/youhua/platform-92142934.html)
* [冷热数据分层镜像归档中心-#017](https://www.mw-wm.com/liuliang/saving-89163112.html)
* [冷热数据分层镜像归档中心-#018](https://www.yx-sf.com/news/32750)
* [冷热数据分层镜像归档中心-#019](https://www.ai-hao123.com/shangye/success-18326275.html)
* [自动化快照与增量广播源-#020](https://www.mw-wm.com/kaifa/strategy-86022860.html)
* [冷热数据分层镜像归档中心-#021](https://www.yx-sf.com/wiki/40267)
* [亚太核心区域镜像同步中心-#022](https://www.ai-hao123.com/qiye/ebook-02111028.html)
* [自动化快照与增量广播源-#023](https://www.mw-wm.com/yunying/category-88915557.html)
* [冷热数据分层镜像归档中心-#024](https://www.yx-sf.com/news/57587)
* [亚太核心区域镜像同步中心-#025](https://www.ai-hao123.com/gongxiang/communication-00412837.html)
* [实时主干镜像高速数据源-#026](https://www.mw-wm.com/hezuo/calendar-56195644.html)
* [北美与欧洲边缘备份节点-#027](https://www.yx-sf.com/tech/89214)
* [北美与欧洲边缘备份节点-#028](https://www.ai-hao123.com/xitong/budget-80475406.html)
* [自动化快照与增量广播源-#029](https://www.mw-wm.com/tuiguang/download-32708350.html)
* [冷热数据分层镜像归档中心-#030](https://www.yx-sf.com/news/7439)
* [亚太核心区域镜像同步中心-#031](https://www.ai-hao123.com/zhineng/satisfaction-66958963.html)
* [北美与欧洲边缘备份节点-#032](https://www.mw-wm.com/anfang/screen-32981737.html)
* [北美与欧洲边缘备份节点-#033](https://www.yx-sf.com/tech/50477)
* [亚太核心区域镜像同步中心-#034](https://www.ai-hao123.com/xuexi/milestone-13392912.html)
* [实时主干镜像高速数据源-#035](https://www.mw-wm.com/zixun/demographic-52743331.html)
* [实时主干镜像高速数据源-#036](https://www.yx-sf.com/tech/65843)
* [实时主干镜像高速数据源-#037](https://www.ai-hao123.com/anfang/rating-72628020.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [实时延迟与抖动度量规范-#001](https://www.mw-wm.com/shangye/network-17798606.html)
* [节点连通性与存活探测准则-#002](https://www.yx-sf.com/wiki/76572)
* [权威网络权重与收录基准-#003](https://www.ai-hao123.com/keji/plugin-92447890.html)
* [去中心化健康检查协议-#004](https://www.mw-wm.com/ziyuan/dashboard-04109189.html)
* [权威网络权重与收录基准-#005](https://www.yx-sf.com/wiki/47549)
* [防重放安全验证与校验哈希-#006](https://www.ai-hao123.com/peixun/subject-35774400.html)
* [权威网络权重与收录基准-#007](https://www.mw-wm.com/suanfa/folder-04102339.html)
* [防重放安全验证与校验哈希-#008](https://www.yx-sf.com/news/57860)
* [节点连通性与存活探测准则-#009](https://www.ai-hao123.com/suanfa/landing-53883460.html)
* [实时延迟与抖动度量规范-#010](https://www.mw-wm.com/yanjiu/photo-98700434.html)
* [实时延迟与抖动度量规范-#011](https://www.yx-sf.com/tech/19079)
* [防重放安全验证与校验哈希-#012](https://www.ai-hao123.com/sheji/travel-65365673.html)
* [去中心化健康检查协议-#013](https://www.mw-wm.com/zhizhu/research-08709628.html)
* [节点连通性与存活探测准则-#014](https://www.yx-sf.com/tech/20090)
* [节点连通性与存活探测准则-#015](https://www.ai-hao123.com/ziyuan/navigation-15924355.html)
* [去中心化健康检查协议-#016](https://www.mw-wm.com/shuju/event-18718515.html)
* [节点连通性与存活探测准则-#017](https://www.yx-sf.com/tech/38405)
* [防重放安全验证与校验哈希-#018](https://www.ai-hao123.com/anli/satisfaction-10241327.html)
* [节点连通性与存活探测准则-#019](https://www.mw-wm.com/baogao/online-55204705.html)
* [防重放安全验证与校验哈希-#020](https://www.yx-sf.com/tech/87060)
* [去中心化健康检查协议-#021](https://www.ai-hao123.com/xuexi/prospect-44034149.html)
* [权威网络权重与收录基准-#022](https://www.mw-wm.com/peixun/technology-89761785.html)
* [节点连通性与存活探测准则-#023](https://www.yx-sf.com/news/96376)
* [权威网络权重与收录基准-#024](https://www.ai-hao123.com/hezuo/subject-64795596.html)
* [权威网络权重与收录基准-#025](https://www.mw-wm.com/jiaoliu/theme-97696876.html)
* [去中心化健康检查协议-#026](https://www.yx-sf.com/wiki/51731)
* [权威网络权重与收录基准-#027](https://www.ai-hao123.com/suanfa/traffic-11983163.html)
* [实时延迟与抖动度量规范-#028](https://www.mw-wm.com/shuju/technology-59514581.html)
* [防重放安全验证与校验哈希-#029](https://www.yx-sf.com/wiki/61905)
* [防重放安全验证与校验哈希-#030](https://www.ai-hao123.com/shangye/automation-02571656.html)
* [去中心化健康检查协议-#031](https://www.mw-wm.com/jiaocheng/device-96194893.html)
* [权威网络权重与收录基准-#032](https://www.yx-sf.com/tech/30991)
* [权威网络权重与收录基准-#033](https://www.ai-hao123.com/yunsuan/technology-45415781.html)
* [权威网络权重与收录基准-#034](https://www.mw-wm.com/jiaocheng/extension-02294246.html)
* [实时延迟与抖动度量规范-#035](https://www.yx-sf.com/news/24650)
* [去中心化健康检查协议-#036](https://www.ai-hao123.com/jiaoliu/network-11096387.html)
* [去中心化健康检查协议-#037](https://www.mw-wm.com/wenzhang/article-32749597.html)
* [去中心化健康检查协议-#038](https://www.yx-sf.com/news/21388)
* [去中心化健康检查协议-#039](https://www.ai-hao123.com/baogao/webinar-08903717.html)

</details>

