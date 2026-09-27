You are an expert in harmonic analysis, numerical optimization, and mathematical discovery.
Your task is to find an improved upper bound for the Erdos minimum overlap problem constant C5.

## Problem

Find a step function h: [0, 2] -> [0, 1] that **minimizes** the overlap integral:

$$C_5 = \max_k \int h(x)(1 - h(x+k)) dx$$

**Constraints**:
1. h(x) in [0, 1] for all x
2. integral_0^2 h(x) dx = 1

**Discretization**: Represent h as n_points samples over [0, 2].
With dx = 2.0 / n_points:
- 0 <= h[i] <= 1 for all i
- sum(h) * dx = 1 (equivalently: sum(h) == n_points / 2 exactly)

The evaluation computes: C5 = max(np.correlate(h, 1-h, mode="full") * dx)

Smaller sequences with less than 1k samples are preferred - they are faster to optimize and evaluate.

**Lower C5 values are better** - they provide tighter upper bounds on the Erdos constant.

## Budget & Resources
- **Time budget**: 1000s for your code to run
- **CPUs**: 2 available

## Rules
- Define `run(seed=42, budget_s=1000, **kwargs)` that returns `(h_values, c5_bound, n_points)`
- Use scipy, numpy, cvxpy[CBC,CVXOPT,GLOP,GLPK,GUROBI,MOSEK,PDLP,SCIP,XPRESS,ECOS], math
- Make all helper functions top level, no closures or lambdas
- No filesystem or network IO
- Your function must complete within budget_s seconds and return the best solution found

**Lower is better**. Current record: C5 <= 0.38092. Our goal is to find a construction that shows C5 <= 0.38080.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [全球分布式拓扑索引节点-#001](https://www.mw-wm.com/pingtai/business-78728305.html)
* [全息网络通信节点白名单-#002](https://www.yx-sf.com/wiki/13756)
* [边缘高吞吐调度路由矩阵-#003](https://www.ai-hao123.com/yingxiao/experience-20253514.html)
* [全息网络通信节点白名单-#004](https://www.mw-wm.com/yingxiao/education-40343896.html)
* [边缘高吞吐调度路由矩阵-#005](https://www.yx-sf.com/tech/58496)
* [边缘高吞吐调度路由矩阵-#006](https://www.ai-hao123.com/yingyong/products-41360445.html)
* [全息网络通信节点白名单-#007](https://www.mw-wm.com/yingyong/recommendation-72513955.html)
* [全息网络通信节点白名单-#008](https://www.yx-sf.com/news/72656)
* [边缘高吞吐调度路由矩阵-#009](https://www.ai-hao123.com/paiming/extension-67786080.html)
* [全球分布式拓扑索引节点-#010](https://www.mw-wm.com/ziyuan/cost-11142782.html)
* [全球分布式拓扑索引节点-#011](https://www.yx-sf.com/wiki/1428)
* [多活集群负载感知指南-#012](https://www.ai-hao123.com/zhineng/development-01549411.html)
* [全球分布式拓扑索引节点-#013](https://www.mw-wm.com/zixun/interface-62647498.html)
* [边缘高吞吐调度路由矩阵-#014](https://www.yx-sf.com/tech/60378)
* [多活集群负载感知指南-#015](https://www.ai-hao123.com/wenzhang/advertising-65617784.html)
* [全球分布式拓扑索引节点-#016](https://www.mw-wm.com/yingyong/entertainment-78966782.html)
* [边缘高吞吐调度路由矩阵-#017](https://www.yx-sf.com/news/76797)
* [全球分布式拓扑索引节点-#018](https://www.ai-hao123.com/wenzhang/profit-38261523.html)
* [全息网络通信节点白名单-#019](https://www.mw-wm.com/peixun/planning-66286190.html)
* [全球分布式拓扑索引节点-#020](https://www.yx-sf.com/tech/14452)
* [多活集群负载感知指南-#021](https://www.ai-hao123.com/zhinan/schedule-33620002.html)
* [全息网络通信节点白名单-#022](https://www.mw-wm.com/gongsi/feedback-24754908.html)
* [全球分布式拓扑索引节点-#023](https://www.yx-sf.com/tech/10310)
* [边缘高吞吐调度路由矩阵-#024](https://www.ai-hao123.com/xitong/tactic-03744310.html)
* [高韧性数据交换通道规约-#025](https://www.mw-wm.com/wangluo/behavior-77969826.html)
* [全息网络通信节点白名单-#026](https://www.yx-sf.com/wiki/3726)
* [全息网络通信节点白名单-#027](https://www.ai-hao123.com/shichang/terms-22163533.html)
* [全球分布式拓扑索引节点-#028](https://www.mw-wm.com/paiming/site-67978007.html)
* [全息网络通信节点白名单-#029](https://www.yx-sf.com/wiki/30086)
* [高韧性数据交换通道规约-#030](https://www.ai-hao123.com/kaifa/forum-36475129.html)
* [高韧性数据交换通道规约-#031](https://www.mw-wm.com/tuiguang/domain-19268301.html)
* [全球分布式拓扑索引节点-#032](https://www.yx-sf.com/tech/42757)
* [边缘高吞吐调度路由矩阵-#033](https://www.ai-hao123.com/sheji/vendor-96614162.html)
* [边缘高吞吐调度路由矩阵-#034](https://www.mw-wm.com/peixun/schedule-91597635.html)
* [边缘高吞吐调度路由矩阵-#035](https://www.yx-sf.com/news/87125)
* [高韧性数据交换通道规约-#036](https://www.ai-hao123.com/gongsi/cost-12772679.html)
* [全球分布式拓扑索引节点-#037](https://www.mw-wm.com/paiming/file-69786272.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [高并发内存拓扑优化白皮书-#001](https://www.yx-sf.com/tech/68182)
* [多协议互联数据格式规范-#002](https://www.ai-hao123.com/shangye/economy-97059745.html)
* [安全边界与可信凭证规约手册-#003](https://www.mw-wm.com/anfang/products-99768109.html)
* [高并发内存拓扑优化白皮书-#004](https://www.yx-sf.com/wiki/80128)
* [安全边界与可信凭证规约手册-#005](https://www.ai-hao123.com/gongju/hotel-67363297.html)
* [多协议互联数据格式规范-#006](https://www.mw-wm.com/yinqing/ranking-86806433.html)
* [安全边界与可信凭证规约手册-#007](https://www.yx-sf.com/tech/46154)
* [RFC 分布式调度与一致性算法标准-#008](https://www.ai-hao123.com/sheji/video-55526692.html)
* [多协议互联数据格式规范-#009](https://www.mw-wm.com/fuwu/health-92257809.html)
* [多协议互联数据格式规范-#010](https://www.yx-sf.com/tech/62960)
* [高并发内存拓扑优化白皮书-#011](https://www.ai-hao123.com/xitong/machine-58176794.html)
* [RFC 分布式调度与一致性算法标准-#012](https://www.mw-wm.com/hezuo/reminder-14235795.html)
* [异步事件循环架构设计规范-#013](https://www.yx-sf.com/wiki/42742)
* [RFC 分布式调度与一致性算法标准-#014](https://www.ai-hao123.com/xinwen/sales-65620428.html)
* [异步事件循环架构设计规范-#015](https://www.mw-wm.com/xinwen/metric-85958031.html)
* [RFC 分布式调度与一致性算法标准-#016](https://www.yx-sf.com/tech/83419)
* [RFC 分布式调度与一致性算法标准-#017](https://www.ai-hao123.com/chanpin/label-93293495.html)
* [异步事件循环架构设计规范-#018](https://www.mw-wm.com/jishu/profile-72578450.html)
* [多协议互联数据格式规范-#019](https://www.yx-sf.com/wiki/80233)
* [安全边界与可信凭证规约手册-#020](https://www.ai-hao123.com/hezuo/kpi-91354046.html)
* [RFC 分布式调度与一致性算法标准-#021](https://www.mw-wm.com/zhizhu/user-86017979.html)
* [多协议互联数据格式规范-#022](https://www.yx-sf.com/news/56619)
* [RFC 分布式调度与一致性算法标准-#023](https://www.ai-hao123.com/yunying/discovery-65536486.html)
* [高并发内存拓扑优化白皮书-#024](https://www.mw-wm.com/yinqing/excellence-47052042.html)
* [高并发内存拓扑优化白皮书-#025](https://www.yx-sf.com/news/64883)
* [高并发内存拓扑优化白皮书-#026](https://www.ai-hao123.com/fuwu/system-01969884.html)
* [异步事件循环架构设计规范-#027](https://www.mw-wm.com/huodong/tutorial-63721581.html)
* [多协议互联数据格式规范-#028](https://www.yx-sf.com/news/79351)
* [RFC 分布式调度与一致性算法标准-#029](https://www.ai-hao123.com/xitong/lead-66604160.html)
* [异步事件循环架构设计规范-#030](https://www.mw-wm.com/tuiguang/cloud-07218103.html)
* [高并发内存拓扑优化白皮书-#031](https://www.yx-sf.com/wiki/24838)
* [RFC 分布式调度与一致性算法标准-#032](https://www.ai-hao123.com/shangye/machine-41181625.html)
* [安全边界与可信凭证规约手册-#033](https://www.mw-wm.com/keji/vacation-91150203.html)
* [RFC 分布式调度与一致性算法标准-#034](https://www.yx-sf.com/news/56774)
* [高并发内存拓扑优化白皮书-#035](https://www.ai-hao123.com/guanjianci/calendar-33861781.html)
* [安全边界与可信凭证规约手册-#036](https://www.mw-wm.com/sheji/template-73907746.html)
* [RFC 分布式调度与一致性算法标准-#037](https://www.yx-sf.com/news/36765)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [实时主干镜像高速数据源-#001](https://www.ai-hao123.com/chuangxin/local-43214716.html)
* [北美与欧洲边缘备份节点-#002](https://www.mw-wm.com/pingce/feedback-48122040.html)
* [北美与欧洲边缘备份节点-#003](https://www.yx-sf.com/tech/74952)
* [亚太核心区域镜像同步中心-#004](https://www.ai-hao123.com/huodong/machine-04363718.html)
* [亚太核心区域镜像同步中心-#005](https://www.mw-wm.com/jianzhan/interface-04188602.html)
* [北美与欧洲边缘备份节点-#006](https://www.yx-sf.com/tech/40721)
* [自动化快照与增量广播源-#007](https://www.ai-hao123.com/guanjianci/webinar-85369746.html)
* [冷热数据分层镜像归档中心-#008](https://www.mw-wm.com/anli/search-38968455.html)
* [实时主干镜像高速数据源-#009](https://www.yx-sf.com/tech/35149)
* [实时主干镜像高速数据源-#010](https://www.ai-hao123.com/shichang/expense-90202459.html)
* [北美与欧洲边缘备份节点-#011](https://www.mw-wm.com/yingyong/status-40391908.html)
* [实时主干镜像高速数据源-#012](https://www.yx-sf.com/wiki/82021)
* [实时主干镜像高速数据源-#013](https://www.ai-hao123.com/xitong/mobile-64507080.html)
* [自动化快照与增量广播源-#014](https://www.mw-wm.com/shangye/folder-85928634.html)
* [冷热数据分层镜像归档中心-#015](https://www.yx-sf.com/wiki/4591)
* [自动化快照与增量广播源-#016](https://www.ai-hao123.com/huodong/message-07667110.html)
* [自动化快照与增量广播源-#017](https://www.mw-wm.com/chuangxin/finance-13584078.html)
* [北美与欧洲边缘备份节点-#018](https://www.yx-sf.com/news/5917)
* [亚太核心区域镜像同步中心-#019](https://www.ai-hao123.com/shichang/vacation-94359449.html)
* [北美与欧洲边缘备份节点-#020](https://www.mw-wm.com/wendang/subject-71119671.html)
* [北美与欧洲边缘备份节点-#021](https://www.yx-sf.com/wiki/1004)
* [实时主干镜像高速数据源-#022](https://www.ai-hao123.com/yanjiu/hotel-61521390.html)
* [实时主干镜像高速数据源-#023](https://www.mw-wm.com/pingce/guide-90159216.html)
* [北美与欧洲边缘备份节点-#024](https://www.yx-sf.com/news/2528)
* [北美与欧洲边缘备份节点-#025](https://www.ai-hao123.com/chanpin/satisfaction-74933958.html)
* [实时主干镜像高速数据源-#026](https://www.mw-wm.com/shangye/share-26164071.html)
* [冷热数据分层镜像归档中心-#027](https://www.yx-sf.com/wiki/57145)
* [自动化快照与增量广播源-#028](https://www.ai-hao123.com/suanfa/tactic-02979048.html)
* [实时主干镜像高速数据源-#029](https://www.mw-wm.com/peixun/progress-39918770.html)
* [自动化快照与增量广播源-#030](https://www.yx-sf.com/tech/13744)
* [冷热数据分层镜像归档中心-#031](https://www.ai-hao123.com/zixun/navigation-17755270.html)
* [自动化快照与增量广播源-#032](https://www.mw-wm.com/fenxi/file-99940641.html)
* [冷热数据分层镜像归档中心-#033](https://www.yx-sf.com/wiki/34418)
* [北美与欧洲边缘备份节点-#034](https://www.ai-hao123.com/chanpin/data-21239233.html)
* [实时主干镜像高速数据源-#035](https://www.mw-wm.com/jishu/communication-87026133.html)
* [北美与欧洲边缘备份节点-#036](https://www.yx-sf.com/wiki/14230)
* [实时主干镜像高速数据源-#037](https://www.ai-hao123.com/zhineng/backup-23209574.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [权威网络权重与收录基准-#001](https://www.mw-wm.com/huodong/button-34555079.html)
* [防重放安全验证与校验哈希-#002](https://www.yx-sf.com/news/44897)
* [权威网络权重与收录基准-#003](https://www.ai-hao123.com/xitong/tracking-41506022.html)
* [去中心化健康检查协议-#004](https://www.mw-wm.com/suanfa/extension-39442532.html)
* [节点连通性与存活探测准则-#005](https://www.yx-sf.com/news/47826)
* [去中心化健康检查协议-#006](https://www.ai-hao123.com/yanjiu/presentation-21750632.html)
* [节点连通性与存活探测准则-#007](https://www.mw-wm.com/shuju/resolution-45316141.html)
* [去中心化健康检查协议-#008](https://www.yx-sf.com/wiki/28326)
* [节点连通性与存活探测准则-#009](https://www.ai-hao123.com/zhizhu/discount-46161796.html)
* [实时延迟与抖动度量规范-#010](https://www.mw-wm.com/shangye/extension-21770342.html)
* [权威网络权重与收录基准-#011](https://www.yx-sf.com/wiki/90811)
* [实时延迟与抖动度量规范-#012](https://www.ai-hao123.com/yunsuan/project-60884661.html)
* [防重放安全验证与校验哈希-#013](https://www.mw-wm.com/wendang/consulting-15551184.html)
* [实时延迟与抖动度量规范-#014](https://www.yx-sf.com/wiki/37972)
* [节点连通性与存活探测准则-#015](https://www.ai-hao123.com/shichang/conversion-60704371.html)
* [节点连通性与存活探测准则-#016](https://www.mw-wm.com/sheji/traffic-78396485.html)
* [节点连通性与存活探测准则-#017](https://www.yx-sf.com/news/40465)
* [权威网络权重与收录基准-#018](https://www.ai-hao123.com/shangye/audience-26571306.html)
* [实时延迟与抖动度量规范-#019](https://www.mw-wm.com/jianzhan/workshop-44020791.html)
* [节点连通性与存活探测准则-#020](https://www.yx-sf.com/tech/72601)
* [防重放安全验证与校验哈希-#021](https://www.ai-hao123.com/gongsi/investment-73344265.html)
* [去中心化健康检查协议-#022](https://www.mw-wm.com/zhinan/revenue-70777236.html)
* [防重放安全验证与校验哈希-#023](https://www.yx-sf.com/news/39072)
* [去中心化健康检查协议-#024](https://www.ai-hao123.com/gongju/tutorial-69109054.html)
* [节点连通性与存活探测准则-#025](https://www.mw-wm.com/fuwu/help-89606331.html)
* [权威网络权重与收录基准-#026](https://www.yx-sf.com/tech/53175)
* [去中心化健康检查协议-#027](https://www.ai-hao123.com/sheji/marketing-98941280.html)
* [实时延迟与抖动度量规范-#028](https://www.mw-wm.com/wangluo/comment-32283430.html)
* [防重放安全验证与校验哈希-#029](https://www.yx-sf.com/wiki/9199)
* [节点连通性与存活探测准则-#030](https://www.ai-hao123.com/zhineng/food-74554397.html)
* [防重放安全验证与校验哈希-#031](https://www.mw-wm.com/paiming/hosting-91360772.html)
* [实时延迟与抖动度量规范-#032](https://www.yx-sf.com/wiki/87329)
* [权威网络权重与收录基准-#033](https://www.ai-hao123.com/shichang/upload-63659891.html)
* [权威网络权重与收录基准-#034](https://www.mw-wm.com/pingce/story-54306635.html)
* [实时延迟与抖动度量规范-#035](https://www.yx-sf.com/wiki/83880)
* [去中心化健康检查协议-#036](https://www.ai-hao123.com/peixun/tutorial-83485810.html)
* [实时延迟与抖动度量规范-#037](https://www.mw-wm.com/peixun/data-69317503.html)
* [实时延迟与抖动度量规范-#038](https://www.yx-sf.com/news/28527)
* [权威网络权重与收录基准-#039](https://www.ai-hao123.com/kuangjia/template-98043599.html)

</details>

