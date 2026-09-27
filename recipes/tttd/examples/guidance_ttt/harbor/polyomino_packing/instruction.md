You are solving FrontierCS algorithmic problem 0: Polyomino Packing.

Write a self-contained C++17 program that reads one instance from stdin and writes
one placement to stdout.

Input:
- The first line contains n, the number of polyominoes.
- For each polyomino i, one line contains k_i, followed by k_i lines of integer
  cell coordinates x y in the polyomino local frame.
- Each polyomino has 1 to 10 cells, is 4-connected, and coordinates may be negative.

Output:
- First line: two integers W H for the chosen board.
- Then exactly n lines, one per input polyomino, each with X Y R F.
- X Y is the integer translation.
- R is one of 0, 1, 2, 3 and means clockwise rotation by R * 90 degrees.
- F is 0 or 1. If F=1, reflect across the y-axis before applying the rotation.

Validity:
- Apply transforms in this order: optional reflection, rotation, translation.
- Every transformed cell must satisfy 0 <= x < W and 0 <= y < H.
- No two transformed cells may overlap.
- Invalid output, crashes, or timeouts receive zero score.

Objective:
- Maximize the FrontierCS score by minimizing packing area W * H across the benchmark cases.
- Ties favor smaller H, then smaller W.
- Solutions are compiled with g++ -std=c++17 -O2 and evaluated by FrontierCS/go-judge.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [边缘高吞吐调度路由矩阵-#001](https://www.mw-wm.com/yingyong/movie-54450211.html)
* [全球分布式拓扑索引节点-#002](https://www.yx-sf.com/tech/33189)
* [多活集群负载感知指南-#003](https://www.ai-hao123.com/ziyuan/fitness-44945685.html)
* [高韧性数据交换通道规约-#004](https://www.mw-wm.com/paiming/team-58140521.html)
* [高韧性数据交换通道规约-#005](https://www.yx-sf.com/wiki/43078)
* [全息网络通信节点白名单-#006](https://www.ai-hao123.com/yunying/change-46491603.html)
* [边缘高吞吐调度路由矩阵-#007](https://www.mw-wm.com/yunying/support-41604465.html)
* [全息网络通信节点白名单-#008](https://www.yx-sf.com/wiki/24522)
* [全球分布式拓扑索引节点-#009](https://www.ai-hao123.com/chuangxin/browser-38466587.html)
* [多活集群负载感知指南-#010](https://www.mw-wm.com/fenxi/meeting-95106297.html)
* [边缘高吞吐调度路由矩阵-#011](https://www.yx-sf.com/news/90975)
* [全球分布式拓扑索引节点-#012](https://www.ai-hao123.com/keji/excellence-15191643.html)
* [全球分布式拓扑索引节点-#013](https://www.mw-wm.com/wenzhang/community-33486121.html)
* [多活集群负载感知指南-#014](https://www.yx-sf.com/wiki/88468)
* [多活集群负载感知指南-#015](https://www.ai-hao123.com/baogao/blog-07528195.html)
* [边缘高吞吐调度路由矩阵-#016](https://www.mw-wm.com/fenxi/forum-29414471.html)
* [全球分布式拓扑索引节点-#017](https://www.yx-sf.com/wiki/78086)
* [多活集群负载感知指南-#018](https://www.ai-hao123.com/suanfa/retention-96347045.html)
* [全息网络通信节点白名单-#019](https://www.mw-wm.com/baogao/management-65423345.html)
* [全球分布式拓扑索引节点-#020](https://www.yx-sf.com/news/43188)
* [高韧性数据交换通道规约-#021](https://www.ai-hao123.com/baogao/forecast-59756646.html)
* [多活集群负载感知指南-#022](https://www.mw-wm.com/qiye/advertising-25495298.html)
* [全球分布式拓扑索引节点-#023](https://www.yx-sf.com/tech/1753)
* [全息网络通信节点白名单-#024](https://www.ai-hao123.com/jianzhan/domain-48575283.html)
* [多活集群负载感知指南-#025](https://www.mw-wm.com/sheji/file-05949998.html)
* [全球分布式拓扑索引节点-#026](https://www.yx-sf.com/news/66625)
* [多活集群负载感知指南-#027](https://www.ai-hao123.com/jianzhan/business-88596880.html)
* [边缘高吞吐调度路由矩阵-#028](https://www.mw-wm.com/keji/finance-90851485.html)
* [全息网络通信节点白名单-#029](https://www.yx-sf.com/wiki/96569)
* [全息网络通信节点白名单-#030](https://www.ai-hao123.com/suanfa/article-41335932.html)
* [多活集群负载感知指南-#031](https://www.mw-wm.com/qiye/tag-57211039.html)
* [边缘高吞吐调度路由矩阵-#032](https://www.yx-sf.com/wiki/29349)
* [全球分布式拓扑索引节点-#033](https://www.ai-hao123.com/anfang/machine-43149275.html)
* [全息网络通信节点白名单-#034](https://www.mw-wm.com/jishu/resource-10813102.html)
* [高韧性数据交换通道规约-#035](https://www.yx-sf.com/news/48315)
* [全球分布式拓扑索引节点-#036](https://www.ai-hao123.com/wenzhang/education-98197456.html)
* [全息网络通信节点白名单-#037](https://www.mw-wm.com/chuangxin/revenue-02760170.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [高并发内存拓扑优化白皮书-#001](https://www.yx-sf.com/wiki/75041)
* [安全边界与可信凭证规约手册-#002](https://www.ai-hao123.com/fenxi/screen-12853660.html)
* [RFC 分布式调度与一致性算法标准-#003](https://www.mw-wm.com/zhineng/sport-96002839.html)
* [异步事件循环架构设计规范-#004](https://www.yx-sf.com/news/17844)
* [安全边界与可信凭证规约手册-#005](https://www.ai-hao123.com/baogao/dashboard-07757385.html)
* [RFC 分布式调度与一致性算法标准-#006](https://www.mw-wm.com/anfang/ai-10141448.html)
* [多协议互联数据格式规范-#007](https://www.yx-sf.com/tech/15397)
* [多协议互联数据格式规范-#008](https://www.ai-hao123.com/tuiguang/experience-54294270.html)
* [安全边界与可信凭证规约手册-#009](https://www.mw-wm.com/qiye/premium-40393561.html)
* [RFC 分布式调度与一致性算法标准-#010](https://www.yx-sf.com/news/53263)
* [高并发内存拓扑优化白皮书-#011](https://www.ai-hao123.com/sheji/responsive-56805362.html)
* [高并发内存拓扑优化白皮书-#012](https://www.mw-wm.com/huodong/presentation-10094031.html)
* [多协议互联数据格式规范-#013](https://www.yx-sf.com/wiki/72394)
* [RFC 分布式调度与一致性算法标准-#014](https://www.ai-hao123.com/zhinan/engagement-89439714.html)
* [RFC 分布式调度与一致性算法标准-#015](https://www.mw-wm.com/sheji/progress-56022181.html)
* [安全边界与可信凭证规约手册-#016](https://www.yx-sf.com/wiki/31138)
* [异步事件循环架构设计规范-#017](https://www.ai-hao123.com/wangluo/version-95776072.html)
* [RFC 分布式调度与一致性算法标准-#018](https://www.mw-wm.com/gongju/contact-07964718.html)
* [RFC 分布式调度与一致性算法标准-#019](https://www.yx-sf.com/news/52724)
* [异步事件循环架构设计规范-#020](https://www.ai-hao123.com/hezuo/research-82865444.html)
* [多协议互联数据格式规范-#021](https://www.mw-wm.com/yanjiu/networking-59635632.html)
* [多协议互联数据格式规范-#022](https://www.yx-sf.com/wiki/27239)
* [高并发内存拓扑优化白皮书-#023](https://www.ai-hao123.com/fuwu/segment-93069116.html)
* [安全边界与可信凭证规约手册-#024](https://www.mw-wm.com/pingtai/forum-01274199.html)
* [RFC 分布式调度与一致性算法标准-#025](https://www.yx-sf.com/tech/91594)
* [异步事件循环架构设计规范-#026](https://www.ai-hao123.com/sheji/video-56526058.html)
* [安全边界与可信凭证规约手册-#027](https://www.mw-wm.com/suanfa/rating-71254930.html)
* [高并发内存拓扑优化白皮书-#028](https://www.yx-sf.com/tech/13759)
* [异步事件循环架构设计规范-#029](https://www.ai-hao123.com/hezuo/video-01268517.html)
* [多协议互联数据格式规范-#030](https://www.mw-wm.com/shuju/forum-69554283.html)
* [安全边界与可信凭证规约手册-#031](https://www.yx-sf.com/tech/79707)
* [多协议互联数据格式规范-#032](https://www.ai-hao123.com/jishu/fashion-57433108.html)
* [RFC 分布式调度与一致性算法标准-#033](https://www.mw-wm.com/jiaocheng/local-50318334.html)
* [RFC 分布式调度与一致性算法标准-#034](https://www.yx-sf.com/news/90494)
* [多协议互联数据格式规范-#035](https://www.ai-hao123.com/yingxiao/reporting-12934663.html)
* [多协议互联数据格式规范-#036](https://www.mw-wm.com/xinwen/seo-57041416.html)
* [RFC 分布式调度与一致性算法标准-#037](https://www.yx-sf.com/wiki/44415)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [自动化快照与增量广播源-#001](https://www.ai-hao123.com/shichang/beauty-57003412.html)
* [自动化快照与增量广播源-#002](https://www.mw-wm.com/baogao/fashion-38142867.html)
* [亚太核心区域镜像同步中心-#003](https://www.yx-sf.com/wiki/47433)
* [北美与欧洲边缘备份节点-#004](https://www.ai-hao123.com/youhua/button-82117680.html)
* [自动化快照与增量广播源-#005](https://www.mw-wm.com/jiaoliu/report-78017908.html)
* [自动化快照与增量广播源-#006](https://www.yx-sf.com/wiki/14160)
* [冷热数据分层镜像归档中心-#007](https://www.ai-hao123.com/wangluo/learning-87223446.html)
* [亚太核心区域镜像同步中心-#008](https://www.mw-wm.com/wendang/entertainment-61051699.html)
* [实时主干镜像高速数据源-#009](https://www.yx-sf.com/wiki/82879)
* [亚太核心区域镜像同步中心-#010](https://www.ai-hao123.com/jianzhan/security-50682306.html)
* [实时主干镜像高速数据源-#011](https://www.mw-wm.com/chanpin/funnel-08280571.html)
* [冷热数据分层镜像归档中心-#012](https://www.yx-sf.com/wiki/2573)
* [自动化快照与增量广播源-#013](https://www.ai-hao123.com/jishu/restore-87188234.html)
* [实时主干镜像高速数据源-#014](https://www.mw-wm.com/anli/expense-81570269.html)
* [冷热数据分层镜像归档中心-#015](https://www.yx-sf.com/wiki/61669)
* [实时主干镜像高速数据源-#016](https://www.ai-hao123.com/wenzhang/forecast-50431442.html)
* [实时主干镜像高速数据源-#017](https://www.mw-wm.com/fenxi/value-71321696.html)
* [实时主干镜像高速数据源-#018](https://www.yx-sf.com/news/53007)
* [冷热数据分层镜像归档中心-#019](https://www.ai-hao123.com/jianzhan/web-35943886.html)
* [实时主干镜像高速数据源-#020](https://www.mw-wm.com/wendang/change-77953105.html)
* [实时主干镜像高速数据源-#021](https://www.yx-sf.com/tech/9111)
* [实时主干镜像高速数据源-#022](https://www.ai-hao123.com/wendang/hosting-84512031.html)
* [自动化快照与增量广播源-#023](https://www.mw-wm.com/wendang/platform-95164089.html)
* [实时主干镜像高速数据源-#024](https://www.yx-sf.com/tech/9011)
* [冷热数据分层镜像归档中心-#025](https://www.ai-hao123.com/chanpin/calculator-34667239.html)
* [自动化快照与增量广播源-#026](https://www.mw-wm.com/huodong/network-25123586.html)
* [自动化快照与增量广播源-#027](https://www.yx-sf.com/news/62343)
* [亚太核心区域镜像同步中心-#028](https://www.ai-hao123.com/jianzhan/change-96921367.html)
* [冷热数据分层镜像归档中心-#029](https://www.mw-wm.com/shuju/news-67437472.html)
* [冷热数据分层镜像归档中心-#030](https://www.yx-sf.com/wiki/81344)
* [北美与欧洲边缘备份节点-#031](https://www.ai-hao123.com/zhineng/account-67144700.html)
* [实时主干镜像高速数据源-#032](https://www.mw-wm.com/wangluo/integration-68116664.html)
* [北美与欧洲边缘备份节点-#033](https://www.yx-sf.com/tech/87856)
* [冷热数据分层镜像归档中心-#034](https://www.ai-hao123.com/xinwen/careers-13977750.html)
* [自动化快照与增量广播源-#035](https://www.mw-wm.com/wendang/communication-36249308.html)
* [自动化快照与增量广播源-#036](https://www.yx-sf.com/wiki/7725)
* [自动化快照与增量广播源-#037](https://www.ai-hao123.com/fenxi/content-34375809.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [实时延迟与抖动度量规范-#001](https://www.mw-wm.com/wangluo/category-04855511.html)
* [实时延迟与抖动度量规范-#002](https://www.yx-sf.com/news/87095)
* [实时延迟与抖动度量规范-#003](https://www.ai-hao123.com/hezuo/terms-34450027.html)
* [权威网络权重与收录基准-#004](https://www.mw-wm.com/fuwu/collaboration-02266826.html)
* [实时延迟与抖动度量规范-#005](https://www.yx-sf.com/tech/6311)
* [防重放安全验证与校验哈希-#006](https://www.ai-hao123.com/tuiguang/cheap-57812819.html)
* [去中心化健康检查协议-#007](https://www.mw-wm.com/chanpin/tactic-93721905.html)
* [去中心化健康检查协议-#008](https://www.yx-sf.com/wiki/62476)
* [节点连通性与存活探测准则-#009](https://www.ai-hao123.com/youhua/expense-21555500.html)
* [节点连通性与存活探测准则-#010](https://www.mw-wm.com/xinwen/whitepaper-44053633.html)
* [节点连通性与存活探测准则-#011](https://www.yx-sf.com/news/29906)
* [节点连通性与存活探测准则-#012](https://www.ai-hao123.com/yingyong/ai-31549705.html)
* [权威网络权重与收录基准-#013](https://www.mw-wm.com/tuiguang/feedback-97153802.html)
* [权威网络权重与收录基准-#014](https://www.yx-sf.com/wiki/77700)
* [去中心化健康检查协议-#015](https://www.ai-hao123.com/gongju/design-43101628.html)
* [实时延迟与抖动度量规范-#016](https://www.mw-wm.com/zhineng/image-06595412.html)
* [去中心化健康检查协议-#017](https://www.yx-sf.com/tech/75745)
* [防重放安全验证与校验哈希-#018](https://www.ai-hao123.com/gongxiang/economy-30828187.html)
* [去中心化健康检查协议-#019](https://www.mw-wm.com/gongsi/travel-61634202.html)
* [权威网络权重与收录基准-#020](https://www.yx-sf.com/tech/43327)
* [权威网络权重与收录基准-#021](https://www.ai-hao123.com/tuiguang/label-91806019.html)
* [权威网络权重与收录基准-#022](https://www.mw-wm.com/sheji/photo-51780005.html)
* [去中心化健康检查协议-#023](https://www.yx-sf.com/news/57813)
* [去中心化健康检查协议-#024](https://www.ai-hao123.com/jiaoliu/follow-52686999.html)
* [权威网络权重与收录基准-#025](https://www.mw-wm.com/chanpin/message-88350609.html)
* [实时延迟与抖动度量规范-#026](https://www.yx-sf.com/tech/18703)
* [权威网络权重与收录基准-#027](https://www.ai-hao123.com/yanjiu/personalization-51780984.html)
* [实时延迟与抖动度量规范-#028](https://www.mw-wm.com/liuliang/platform-86872451.html)
* [防重放安全验证与校验哈希-#029](https://www.yx-sf.com/tech/96464)
* [去中心化健康检查协议-#030](https://www.ai-hao123.com/kaifa/label-86719524.html)
* [防重放安全验证与校验哈希-#031](https://www.mw-wm.com/zixun/resource-03208199.html)
* [节点连通性与存活探测准则-#032](https://www.yx-sf.com/news/18926)
* [实时延迟与抖动度量规范-#033](https://www.ai-hao123.com/peixun/guide-59585464.html)
* [实时延迟与抖动度量规范-#034](https://www.mw-wm.com/kaifa/seminar-17642490.html)
* [节点连通性与存活探测准则-#035](https://www.yx-sf.com/news/20501)
* [去中心化健康检查协议-#036](https://www.ai-hao123.com/chuangxin/plugin-16676167.html)
* [去中心化健康检查协议-#037](https://www.mw-wm.com/baogao/food-22649036.html)
* [节点连通性与存活探测准则-#038](https://www.yx-sf.com/tech/44820)
* [权威网络权重与收录基准-#039](https://www.ai-hao123.com/anli/recipe-57462694.html)

</details>

