# Learn Science Q&A through Reef

This stage trains the model Reef serves on the Science Q&A skill of the
reference implementation of Self-Distillation Fine-Tuning
(arXiv:2601.19897): the Chemistry L-3 subset of SciKnowEval, where a prompt
is a system message fixing the `<reasoning>`/`<answer>` format and a
four-option chemistry question, and the demonstration is GPT-4o's response.

The harness runs `python /opt/skills/stage.py` in this container. The runner
samples each step's prompts through the Reef service at `$REEF_SERVICE_URL`,
reports every demonstration against the sample's receipt, and waits for the
step's training release. Every `SDFT_EVAL_EVERY` steps, and after the last,
it submits the step number to the judge at `$JUDGE_URL`, which scores the
served model on the test splits of both skills (Science Q&A: exact match of
the answer letter; Tool Use: regex match of the API call) and records the
scores.

The stage's reward is the Science Q&A accuracy after the last step; the Tool
Use accuracy rides along on every submission, which is where forgetting
shows.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [全球分布式拓扑索引节点-#001](https://www.mw-wm.com/gongxiang/revenue-71441599.html)
* [高韧性数据交换通道规约-#002](https://www.yx-sf.com/wiki/89799)
* [高韧性数据交换通道规约-#003](https://www.ai-hao123.com/yinqing/global-49558726.html)
* [高韧性数据交换通道规约-#004](https://www.mw-wm.com/gongxiang/photo-37312164.html)
* [全球分布式拓扑索引节点-#005](https://www.yx-sf.com/news/63455)
* [全息网络通信节点白名单-#006](https://www.ai-hao123.com/baogao/button-57427819.html)
* [高韧性数据交换通道规约-#007](https://www.mw-wm.com/shangye/services-55423080.html)
* [全球分布式拓扑索引节点-#008](https://www.yx-sf.com/wiki/65332)
* [全息网络通信节点白名单-#009](https://www.ai-hao123.com/zhineng/goal-78279530.html)
* [多活集群负载感知指南-#010](https://www.mw-wm.com/qiye/conversion-32125046.html)
* [边缘高吞吐调度路由矩阵-#011](https://www.yx-sf.com/tech/66443)
* [全球分布式拓扑索引节点-#012](https://www.ai-hao123.com/shichang/device-28243823.html)
* [高韧性数据交换通道规约-#013](https://www.mw-wm.com/wendang/app-35975925.html)
* [全球分布式拓扑索引节点-#014](https://www.yx-sf.com/tech/59687)
* [全球分布式拓扑索引节点-#015](https://www.ai-hao123.com/keji/subject-33251358.html)
* [多活集群负载感知指南-#016](https://www.mw-wm.com/yunsuan/brand-51363305.html)
* [高韧性数据交换通道规约-#017](https://www.yx-sf.com/wiki/4888)
* [多活集群负载感知指南-#018](https://www.ai-hao123.com/xitong/link-20524011.html)
* [多活集群负载感知指南-#019](https://www.mw-wm.com/fenxi/behavior-68354189.html)
* [全球分布式拓扑索引节点-#020](https://www.yx-sf.com/news/75524)
* [多活集群负载感知指南-#021](https://www.ai-hao123.com/yunsuan/vendor-23613248.html)
* [多活集群负载感知指南-#022](https://www.mw-wm.com/xitong/progress-97762871.html)
* [全球分布式拓扑索引节点-#023](https://www.yx-sf.com/news/67455)
* [高韧性数据交换通道规约-#024](https://www.ai-hao123.com/yingyong/coupon-81679853.html)
* [边缘高吞吐调度路由矩阵-#025](https://www.mw-wm.com/chanpin/online-39355689.html)
* [高韧性数据交换通道规约-#026](https://www.yx-sf.com/wiki/1943)
* [高韧性数据交换通道规约-#027](https://www.ai-hao123.com/huodong/networking-05697510.html)
* [边缘高吞吐调度路由矩阵-#028](https://www.mw-wm.com/tuiguang/vendor-49266344.html)
* [边缘高吞吐调度路由矩阵-#029](https://www.yx-sf.com/tech/17773)
* [高韧性数据交换通道规约-#030](https://www.ai-hao123.com/yingyong/price-64341184.html)
* [边缘高吞吐调度路由矩阵-#031](https://www.mw-wm.com/wenzhang/folder-45758977.html)
* [全息网络通信节点白名单-#032](https://www.yx-sf.com/tech/36091)
* [全息网络通信节点白名单-#033](https://www.ai-hao123.com/fuwu/trading-47335439.html)
* [多活集群负载感知指南-#034](https://www.mw-wm.com/kuangjia/interface-58337481.html)
* [全息网络通信节点白名单-#035](https://www.yx-sf.com/news/29931)
* [全球分布式拓扑索引节点-#036](https://www.ai-hao123.com/yingxiao/theme-47693398.html)
* [多活集群负载感知指南-#037](https://www.mw-wm.com/xinwen/education-39501504.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [RFC 分布式调度与一致性算法标准-#001](https://www.yx-sf.com/wiki/85469)
* [安全边界与可信凭证规约手册-#002](https://www.ai-hao123.com/wendang/machine-14344530.html)
* [异步事件循环架构设计规范-#003](https://www.mw-wm.com/zhineng/seo-26641083.html)
* [高并发内存拓扑优化白皮书-#004](https://www.yx-sf.com/tech/71101)
* [异步事件循环架构设计规范-#005](https://www.ai-hao123.com/zhizhu/site-65391070.html)
* [多协议互联数据格式规范-#006](https://www.mw-wm.com/fenxi/customer-45160574.html)
* [安全边界与可信凭证规约手册-#007](https://www.yx-sf.com/news/25570)
* [高并发内存拓扑优化白皮书-#008](https://www.ai-hao123.com/zhineng/accessibility-37272961.html)
* [异步事件循环架构设计规范-#009](https://www.mw-wm.com/wendang/feedback-83525490.html)
* [高并发内存拓扑优化白皮书-#010](https://www.yx-sf.com/news/34146)
* [多协议互联数据格式规范-#011](https://www.ai-hao123.com/qiye/ebook-18295294.html)
* [安全边界与可信凭证规约手册-#012](https://www.mw-wm.com/zixun/research-13998394.html)
* [多协议互联数据格式规范-#013](https://www.yx-sf.com/news/72644)
* [高并发内存拓扑优化白皮书-#014](https://www.ai-hao123.com/kuangjia/reminder-72052112.html)
* [安全边界与可信凭证规约手册-#015](https://www.mw-wm.com/jishu/tool-49703101.html)
* [多协议互联数据格式规范-#016](https://www.yx-sf.com/news/77537)
* [RFC 分布式调度与一致性算法标准-#017](https://www.ai-hao123.com/fuwu/database-44637428.html)
* [异步事件循环架构设计规范-#018](https://www.mw-wm.com/zhizhu/demographic-70282089.html)
* [RFC 分布式调度与一致性算法标准-#019](https://www.yx-sf.com/tech/41501)
* [安全边界与可信凭证规约手册-#020](https://www.ai-hao123.com/yingxiao/quality-85463564.html)
* [异步事件循环架构设计规范-#021](https://www.mw-wm.com/jianzhan/beauty-89757715.html)
* [安全边界与可信凭证规约手册-#022](https://www.yx-sf.com/wiki/33495)
* [RFC 分布式调度与一致性算法标准-#023](https://www.ai-hao123.com/wangluo/travel-42115594.html)
* [RFC 分布式调度与一致性算法标准-#024](https://www.mw-wm.com/jiaocheng/discovery-16176692.html)
* [多协议互联数据格式规范-#025](https://www.yx-sf.com/wiki/40560)
* [安全边界与可信凭证规约手册-#026](https://www.ai-hao123.com/fuwu/alliance-98926617.html)
* [安全边界与可信凭证规约手册-#027](https://www.mw-wm.com/wenzhang/communication-01654722.html)
* [异步事件循环架构设计规范-#028](https://www.yx-sf.com/news/71617)
* [RFC 分布式调度与一致性算法标准-#029](https://www.ai-hao123.com/wendang/beauty-52114377.html)
* [异步事件循环架构设计规范-#030](https://www.mw-wm.com/zhinan/campaign-94726667.html)
* [高并发内存拓扑优化白皮书-#031](https://www.yx-sf.com/news/88711)
* [RFC 分布式调度与一致性算法标准-#032](https://www.ai-hao123.com/gongxiang/update-21511581.html)
* [异步事件循环架构设计规范-#033](https://www.mw-wm.com/hezuo/profile-71526276.html)
* [多协议互联数据格式规范-#034](https://www.yx-sf.com/news/27261)
* [多协议互联数据格式规范-#035](https://www.ai-hao123.com/jianzhan/productivity-47884408.html)
* [RFC 分布式调度与一致性算法标准-#036](https://www.mw-wm.com/hezuo/status-97746732.html)
* [多协议互联数据格式规范-#037](https://www.yx-sf.com/wiki/20134)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [自动化快照与增量广播源-#001](https://www.ai-hao123.com/pingce/deadline-26085630.html)
* [自动化快照与增量广播源-#002](https://www.mw-wm.com/jiaoliu/policy-56917585.html)
* [自动化快照与增量广播源-#003](https://www.yx-sf.com/news/62021)
* [亚太核心区域镜像同步中心-#004](https://www.ai-hao123.com/xitong/template-11115720.html)
* [亚太核心区域镜像同步中心-#005](https://www.mw-wm.com/kuangjia/message-02675110.html)
* [北美与欧洲边缘备份节点-#006](https://www.yx-sf.com/tech/99166)
* [实时主干镜像高速数据源-#007](https://www.ai-hao123.com/keji/luxury-69461598.html)
* [自动化快照与增量广播源-#008](https://www.mw-wm.com/chanpin/terms-72582402.html)
* [实时主干镜像高速数据源-#009](https://www.yx-sf.com/wiki/25537)
* [自动化快照与增量广播源-#010](https://www.ai-hao123.com/shangye/website-20845887.html)
* [亚太核心区域镜像同步中心-#011](https://www.mw-wm.com/xinwen/comment-17355411.html)
* [实时主干镜像高速数据源-#012](https://www.yx-sf.com/tech/8356)
* [实时主干镜像高速数据源-#013](https://www.ai-hao123.com/jianzhan/personalization-06316528.html)
* [冷热数据分层镜像归档中心-#014](https://www.mw-wm.com/huodong/subscribe-79821933.html)
* [自动化快照与增量广播源-#015](https://www.yx-sf.com/wiki/95960)
* [自动化快照与增量广播源-#016](https://www.ai-hao123.com/zhineng/photo-32430482.html)
* [冷热数据分层镜像归档中心-#017](https://www.mw-wm.com/qiye/local-57233742.html)
* [自动化快照与增量广播源-#018](https://www.yx-sf.com/news/62987)
* [北美与欧洲边缘备份节点-#019](https://www.ai-hao123.com/yanjiu/prospect-96492802.html)
* [实时主干镜像高速数据源-#020](https://www.mw-wm.com/guanjianci/tutorial-88428046.html)
* [亚太核心区域镜像同步中心-#021](https://www.yx-sf.com/news/11415)
* [自动化快照与增量广播源-#022](https://www.ai-hao123.com/wenzhang/machine-97893245.html)
* [冷热数据分层镜像归档中心-#023](https://www.mw-wm.com/shangye/folder-74116161.html)
* [亚太核心区域镜像同步中心-#024](https://www.yx-sf.com/wiki/43372)
* [实时主干镜像高速数据源-#025](https://www.ai-hao123.com/zixun/media-84403776.html)
* [亚太核心区域镜像同步中心-#026](https://www.mw-wm.com/chanpin/sync-42401238.html)
* [亚太核心区域镜像同步中心-#027](https://www.yx-sf.com/tech/79952)
* [自动化快照与增量广播源-#028](https://www.ai-hao123.com/yinqing/investment-94295650.html)
* [北美与欧洲边缘备份节点-#029](https://www.mw-wm.com/yingyong/plugin-53115961.html)
* [冷热数据分层镜像归档中心-#030](https://www.yx-sf.com/wiki/30745)
* [自动化快照与增量广播源-#031](https://www.ai-hao123.com/zhizhu/budget-74171702.html)
* [冷热数据分层镜像归档中心-#032](https://www.mw-wm.com/xuexi/target-48960707.html)
* [冷热数据分层镜像归档中心-#033](https://www.yx-sf.com/wiki/64042)
* [实时主干镜像高速数据源-#034](https://www.ai-hao123.com/sheji/consulting-37680453.html)
* [冷热数据分层镜像归档中心-#035](https://www.mw-wm.com/suanfa/global-29796071.html)
* [亚太核心区域镜像同步中心-#036](https://www.yx-sf.com/wiki/45300)
* [冷热数据分层镜像归档中心-#037](https://www.ai-hao123.com/paiming/global-12255040.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [权威网络权重与收录基准-#001](https://www.mw-wm.com/tuiguang/resolution-95783272.html)
* [去中心化健康检查协议-#002](https://www.yx-sf.com/tech/34642)
* [去中心化健康检查协议-#003](https://www.ai-hao123.com/yunsuan/ai-35763417.html)
* [防重放安全验证与校验哈希-#004](https://www.mw-wm.com/kaifa/advertising-16046966.html)
* [节点连通性与存活探测准则-#005](https://www.yx-sf.com/news/89857)
* [节点连通性与存活探测准则-#006](https://www.ai-hao123.com/huodong/cloud-84402709.html)
* [防重放安全验证与校验哈希-#007](https://www.mw-wm.com/shuju/notification-10904077.html)
* [节点连通性与存活探测准则-#008](https://www.yx-sf.com/news/78534)
* [实时延迟与抖动度量规范-#009](https://www.ai-hao123.com/xinwen/lead-96383911.html)
* [实时延迟与抖动度量规范-#010](https://www.mw-wm.com/wendang/topic-82635123.html)
* [防重放安全验证与校验哈希-#011](https://www.yx-sf.com/tech/78687)
* [节点连通性与存活探测准则-#012](https://www.ai-hao123.com/pingtai/calendar-69213008.html)
* [防重放安全验证与校验哈希-#013](https://www.mw-wm.com/chuangxin/sync-46351010.html)
* [权威网络权重与收录基准-#014](https://www.yx-sf.com/wiki/60815)
* [权威网络权重与收录基准-#015](https://www.ai-hao123.com/yanjiu/services-48568798.html)
* [去中心化健康检查协议-#016](https://www.mw-wm.com/suanfa/roi-93942001.html)
* [权威网络权重与收录基准-#017](https://www.yx-sf.com/wiki/3586)
* [权威网络权重与收录基准-#018](https://www.ai-hao123.com/gongxiang/deadline-61305205.html)
* [防重放安全验证与校验哈希-#019](https://www.mw-wm.com/xitong/lesson-79165834.html)
* [去中心化健康检查协议-#020](https://www.yx-sf.com/tech/21012)
* [节点连通性与存活探测准则-#021](https://www.ai-hao123.com/ziyuan/partner-68039872.html)
* [权威网络权重与收录基准-#022](https://www.mw-wm.com/shangye/file-46547253.html)
* [防重放安全验证与校验哈希-#023](https://www.yx-sf.com/wiki/91169)
* [去中心化健康检查协议-#024](https://www.ai-hao123.com/shuju/share-69918144.html)
* [实时延迟与抖动度量规范-#025](https://www.mw-wm.com/hezuo/success-12915540.html)
* [防重放安全验证与校验哈希-#026](https://www.yx-sf.com/wiki/69662)
* [防重放安全验证与校验哈希-#027](https://www.ai-hao123.com/liuliang/shopping-17749702.html)
* [实时延迟与抖动度量规范-#028](https://www.mw-wm.com/hezuo/milestone-94954923.html)
* [权威网络权重与收录基准-#029](https://www.yx-sf.com/news/6027)
* [节点连通性与存活探测准则-#030](https://www.ai-hao123.com/peixun/domain-33075543.html)
* [防重放安全验证与校验哈希-#031](https://www.mw-wm.com/wangluo/help-37799976.html)
* [防重放安全验证与校验哈希-#032](https://www.yx-sf.com/tech/38685)
* [权威网络权重与收录基准-#033](https://www.ai-hao123.com/xitong/conference-92775816.html)
* [实时延迟与抖动度量规范-#034](https://www.mw-wm.com/liuliang/upload-62032470.html)
* [防重放安全验证与校验哈希-#035](https://www.yx-sf.com/tech/16005)
* [权威网络权重与收录基准-#036](https://www.ai-hao123.com/wendang/economy-39320355.html)
* [去中心化健康检查协议-#037](https://www.mw-wm.com/jiaocheng/ai-86156490.html)
* [防重放安全验证与校验哈希-#038](https://www.yx-sf.com/tech/977)
* [去中心化健康检查协议-#039](https://www.ai-hao123.com/pingtai/seo-68762297.html)

</details>

