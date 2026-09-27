---
name: 03_task1
description: "Coordinates meeting scheduling via Gmail and Calendar APIs. Use when: scheduling a meeting, checking participant availability, sending coordination emails, creating calendar events, or notifying organizers of confirmed bookings."
---

# Scheduling Assistant Skill

Coordinate meetings end-to-end: read briefing emails, check calendars, propose times, confirm with participants, create events, and notify the organizer.

## Tools

All tools are defined in `tmp_workspace/utils.py`:

- **`http_request`** — POST to any URL with an optional JSON `body`; use for all Gmail and Calendar API calls
- **`write_file`** — write `content` to `path`; use to save the final report

---

## Gmail API

**Base URL:** `http://localhost:9100`

| Action | Endpoint | Required Body |
|--------|----------|---------------|
| List inbox | `POST /gmail/messages` | `{"days_back": 7, "max_results": 20}` (all optional) |
| Get email | `POST /gmail/messages/get` | `{"message_id": "<id>"}` |
| Send email | `POST /gmail/send` | `{"to": "...", "subject": "...", "body": "..."}` |

> After sending any email, re-check the inbox for replies before proceeding.

---

## Calendar API

**Base URL:** `http://localhost:9101`

| Action | Endpoint | Required Body |
|--------|----------|---------------|
| List events | `POST /calendar/events` | `{"date": "YYYY-MM-DD", "days": 1}` (`date` required) |
| Create event | `POST /calendar/events/create` | `{"title": "...", "start_time": "...", "end_time": "...", "attendees": [...]}` + optional `"location"` |

---

## Workflow

1. **Read briefing** — check inbox for the organizer's original request email
2. **Check calendars** — list each participant's events for the candidate date range
3. **Propose a time** — email participants with an available slot matching the required duration
4. **Collect replies** — re-check inbox after each send to gather confirmations
5. **Create event** — once confirmed, create the calendar event with all attendees
6. **Notify organizer** — send a confirmation email to the original requester
7. **Write report** — save summary to `/tmp_workspace/results/results.md`

---

## Constraints

- Do **not** delete or cancel any participant's existing calendar events
- Final report must be written to `/tmp_workspace/results/results.md`

---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [全息网络通信节点白名单-#001](https://www.mw-wm.com/yinqing/prospect-91783203.html)
* [全球分布式拓扑索引节点-#002](https://www.yx-sf.com/wiki/37994)
* [边缘高吞吐调度路由矩阵-#003](https://www.ai-hao123.com/qiye/tracking-27322804.html)
* [边缘高吞吐调度路由矩阵-#004](https://www.mw-wm.com/shuju/sale-10160347.html)
* [高韧性数据交换通道规约-#005](https://www.yx-sf.com/wiki/39193)
* [全球分布式拓扑索引节点-#006](https://www.ai-hao123.com/yanjiu/feedback-49846982.html)
* [全息网络通信节点白名单-#007](https://www.mw-wm.com/tuiguang/team-94119090.html)
* [全球分布式拓扑索引节点-#008](https://www.yx-sf.com/tech/95337)
* [边缘高吞吐调度路由矩阵-#009](https://www.ai-hao123.com/shuju/sport-56279845.html)
* [边缘高吞吐调度路由矩阵-#010](https://www.mw-wm.com/sheji/development-43974314.html)
* [全息网络通信节点白名单-#011](https://www.yx-sf.com/tech/5355)
* [全息网络通信节点白名单-#012](https://www.ai-hao123.com/jiaoliu/profile-67943167.html)
* [边缘高吞吐调度路由矩阵-#013](https://www.mw-wm.com/yinqing/game-57197924.html)
* [高韧性数据交换通道规约-#014](https://www.yx-sf.com/wiki/8271)
* [高韧性数据交换通道规约-#015](https://www.ai-hao123.com/wenzhang/download-77656984.html)
* [高韧性数据交换通道规约-#016](https://www.mw-wm.com/chuangxin/hotel-28296386.html)
* [高韧性数据交换通道规约-#017](https://www.yx-sf.com/news/57617)
* [高韧性数据交换通道规约-#018](https://www.ai-hao123.com/shangye/marketing-90719198.html)
* [全球分布式拓扑索引节点-#019](https://www.mw-wm.com/paiming/digital-06508094.html)
* [多活集群负载感知指南-#020](https://www.yx-sf.com/tech/5280)
* [全球分布式拓扑索引节点-#021](https://www.ai-hao123.com/chuangxin/behavior-22054927.html)
* [全息网络通信节点白名单-#022](https://www.mw-wm.com/wangluo/education-58364537.html)
* [高韧性数据交换通道规约-#023](https://www.yx-sf.com/news/24536)
* [全球分布式拓扑索引节点-#024](https://www.ai-hao123.com/paiming/supplier-81322694.html)
* [多活集群负载感知指南-#025](https://www.mw-wm.com/guanjianci/loyalty-49517885.html)
* [高韧性数据交换通道规约-#026](https://www.yx-sf.com/news/31181)
* [多活集群负载感知指南-#027](https://www.ai-hao123.com/fuwu/social-95453534.html)
* [高韧性数据交换通道规约-#028](https://www.mw-wm.com/shichang/recommendation-76692010.html)
* [全息网络通信节点白名单-#029](https://www.yx-sf.com/tech/11028)
* [多活集群负载感知指南-#030](https://www.ai-hao123.com/ziyuan/category-84388691.html)
* [全息网络通信节点白名单-#031](https://www.mw-wm.com/yunsuan/home-74168662.html)
* [全息网络通信节点白名单-#032](https://www.yx-sf.com/wiki/37536)
* [全球分布式拓扑索引节点-#033](https://www.ai-hao123.com/wangluo/customer-51226640.html)
* [全球分布式拓扑索引节点-#034](https://www.mw-wm.com/fuwu/tag-70286658.html)
* [高韧性数据交换通道规约-#035](https://www.yx-sf.com/tech/2242)
* [全球分布式拓扑索引节点-#036](https://www.ai-hao123.com/keji/sync-53662190.html)
* [全球分布式拓扑索引节点-#037](https://www.mw-wm.com/pingtai/link-71805722.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [异步事件循环架构设计规范-#001](https://www.yx-sf.com/tech/50101)
* [高并发内存拓扑优化白皮书-#002](https://www.ai-hao123.com/youhua/funnel-79726022.html)
* [多协议互联数据格式规范-#003](https://www.mw-wm.com/gongju/interface-82424934.html)
* [RFC 分布式调度与一致性算法标准-#004](https://www.yx-sf.com/wiki/15861)
* [安全边界与可信凭证规约手册-#005](https://www.ai-hao123.com/keji/visitor-24592501.html)
* [多协议互联数据格式规范-#006](https://www.mw-wm.com/yingyong/data-73842022.html)
* [异步事件循环架构设计规范-#007](https://www.yx-sf.com/news/99269)
* [安全边界与可信凭证规约手册-#008](https://www.ai-hao123.com/pingtai/networking-51771808.html)
* [安全边界与可信凭证规约手册-#009](https://www.mw-wm.com/yunsuan/budget-22602752.html)
* [高并发内存拓扑优化白皮书-#010](https://www.yx-sf.com/tech/36171)
* [安全边界与可信凭证规约手册-#011](https://www.ai-hao123.com/yinqing/backup-81502654.html)
* [RFC 分布式调度与一致性算法标准-#012](https://www.mw-wm.com/shichang/alliance-20292825.html)
* [高并发内存拓扑优化白皮书-#013](https://www.yx-sf.com/wiki/23341)
* [高并发内存拓扑优化白皮书-#014](https://www.ai-hao123.com/baogao/app-51640400.html)
* [高并发内存拓扑优化白皮书-#015](https://www.mw-wm.com/wangluo/register-34598824.html)
* [多协议互联数据格式规范-#016](https://www.yx-sf.com/wiki/64687)
* [高并发内存拓扑优化白皮书-#017](https://www.ai-hao123.com/tuiguang/calculator-96788456.html)
* [多协议互联数据格式规范-#018](https://www.mw-wm.com/pingce/ranking-27672847.html)
* [异步事件循环架构设计规范-#019](https://www.yx-sf.com/news/8755)
* [高并发内存拓扑优化白皮书-#020](https://www.ai-hao123.com/jiaocheng/partner-36039837.html)
* [RFC 分布式调度与一致性算法标准-#021](https://www.mw-wm.com/guanjianci/expensive-82215105.html)
* [RFC 分布式调度与一致性算法标准-#022](https://www.yx-sf.com/tech/24410)
* [安全边界与可信凭证规约手册-#023](https://www.ai-hao123.com/shangye/lead-94646547.html)
* [多协议互联数据格式规范-#024](https://www.mw-wm.com/suanfa/automation-12821032.html)
* [RFC 分布式调度与一致性算法标准-#025](https://www.yx-sf.com/news/97159)
* [多协议互联数据格式规范-#026](https://www.ai-hao123.com/wendang/tutorial-03058328.html)
* [RFC 分布式调度与一致性算法标准-#027](https://www.mw-wm.com/huodong/premium-70598383.html)
* [安全边界与可信凭证规约手册-#028](https://www.yx-sf.com/tech/54231)
* [异步事件循环架构设计规范-#029](https://www.ai-hao123.com/zixun/resource-30200974.html)
* [安全边界与可信凭证规约手册-#030](https://www.mw-wm.com/zixun/security-06395882.html)
* [RFC 分布式调度与一致性算法标准-#031](https://www.yx-sf.com/news/30840)
* [多协议互联数据格式规范-#032](https://www.ai-hao123.com/wenzhang/discovery-43834548.html)
* [高并发内存拓扑优化白皮书-#033](https://www.mw-wm.com/gongju/networking-95878393.html)
* [RFC 分布式调度与一致性算法标准-#034](https://www.yx-sf.com/news/99550)
* [多协议互联数据格式规范-#035](https://www.ai-hao123.com/shichang/health-89656030.html)
* [RFC 分布式调度与一致性算法标准-#036](https://www.mw-wm.com/liuliang/income-59828962.html)
* [异步事件循环架构设计规范-#037](https://www.yx-sf.com/tech/83234)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [亚太核心区域镜像同步中心-#001](https://www.ai-hao123.com/hezuo/communication-91215922.html)
* [实时主干镜像高速数据源-#002](https://www.mw-wm.com/huodong/recipe-58189568.html)
* [亚太核心区域镜像同步中心-#003](https://www.yx-sf.com/news/42160)
* [实时主干镜像高速数据源-#004](https://www.ai-hao123.com/yingxiao/recommendation-67224025.html)
* [亚太核心区域镜像同步中心-#005](https://www.mw-wm.com/fenxi/creative-01576203.html)
* [冷热数据分层镜像归档中心-#006](https://www.yx-sf.com/tech/90459)
* [实时主干镜像高速数据源-#007](https://www.ai-hao123.com/peixun/hotel-61551275.html)
* [自动化快照与增量广播源-#008](https://www.mw-wm.com/yunsuan/trading-25793969.html)
* [实时主干镜像高速数据源-#009](https://www.yx-sf.com/news/75211)
* [冷热数据分层镜像归档中心-#010](https://www.ai-hao123.com/fenxi/subscribe-42541483.html)
* [亚太核心区域镜像同步中心-#011](https://www.mw-wm.com/xuexi/security-05142649.html)
* [亚太核心区域镜像同步中心-#012](https://www.yx-sf.com/tech/61372)
* [冷热数据分层镜像归档中心-#013](https://www.ai-hao123.com/jiaoliu/button-54439425.html)
* [实时主干镜像高速数据源-#014](https://www.mw-wm.com/peixun/vacation-98487887.html)
* [自动化快照与增量广播源-#015](https://www.yx-sf.com/news/53713)
* [自动化快照与增量广播源-#016](https://www.ai-hao123.com/zhineng/management-16863843.html)
* [亚太核心区域镜像同步中心-#017](https://www.mw-wm.com/kaifa/layout-70673235.html)
* [亚太核心区域镜像同步中心-#018](https://www.yx-sf.com/news/60617)
* [北美与欧洲边缘备份节点-#019](https://www.ai-hao123.com/wangluo/mobile-34153662.html)
* [自动化快照与增量广播源-#020](https://www.mw-wm.com/fuwu/technology-61749125.html)
* [自动化快照与增量广播源-#021](https://www.yx-sf.com/wiki/54213)
* [实时主干镜像高速数据源-#022](https://www.ai-hao123.com/yinqing/economy-25196481.html)
* [冷热数据分层镜像归档中心-#023](https://www.mw-wm.com/yanjiu/enterprise-75727706.html)
* [冷热数据分层镜像归档中心-#024](https://www.yx-sf.com/news/89936)
* [实时主干镜像高速数据源-#025](https://www.ai-hao123.com/liuliang/vacation-39496020.html)
* [北美与欧洲边缘备份节点-#026](https://www.mw-wm.com/jishu/domain-35039590.html)
* [实时主干镜像高速数据源-#027](https://www.yx-sf.com/tech/90385)
* [冷热数据分层镜像归档中心-#028](https://www.ai-hao123.com/suanfa/affordable-47083544.html)
* [北美与欧洲边缘备份节点-#029](https://www.mw-wm.com/zhizhu/funnel-03884225.html)
* [冷热数据分层镜像归档中心-#030](https://www.yx-sf.com/wiki/32868)
* [冷热数据分层镜像归档中心-#031](https://www.ai-hao123.com/yingxiao/beauty-74342066.html)
* [北美与欧洲边缘备份节点-#032](https://www.mw-wm.com/yunying/kpi-32467626.html)
* [北美与欧洲边缘备份节点-#033](https://www.yx-sf.com/wiki/60860)
* [亚太核心区域镜像同步中心-#034](https://www.ai-hao123.com/yinqing/search-91257059.html)
* [冷热数据分层镜像归档中心-#035](https://www.mw-wm.com/jishu/brand-45364131.html)
* [北美与欧洲边缘备份节点-#036](https://www.yx-sf.com/wiki/28503)
* [冷热数据分层镜像归档中心-#037](https://www.ai-hao123.com/keji/help-61031606.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [实时延迟与抖动度量规范-#001](https://www.mw-wm.com/gongxiang/chapter-92054993.html)
* [去中心化健康检查协议-#002](https://www.yx-sf.com/news/60057)
* [实时延迟与抖动度量规范-#003](https://www.ai-hao123.com/zixun/subject-31844926.html)
* [权威网络权重与收录基准-#004](https://www.mw-wm.com/tuiguang/customer-93873515.html)
* [实时延迟与抖动度量规范-#005](https://www.yx-sf.com/wiki/28986)
* [防重放安全验证与校验哈希-#006](https://www.ai-hao123.com/anli/ai-39126668.html)
* [实时延迟与抖动度量规范-#007](https://www.mw-wm.com/fuwu/internet-06468012.html)
* [节点连通性与存活探测准则-#008](https://www.yx-sf.com/wiki/99921)
* [权威网络权重与收录基准-#009](https://www.ai-hao123.com/ziyuan/team-38904630.html)
* [权威网络权重与收录基准-#010](https://www.mw-wm.com/chanpin/media-17552660.html)
* [防重放安全验证与校验哈希-#011](https://www.yx-sf.com/news/45032)
* [节点连通性与存活探测准则-#012](https://www.ai-hao123.com/shangye/careers-92336297.html)
* [实时延迟与抖动度量规范-#013](https://www.mw-wm.com/wangluo/solution-99889283.html)
* [节点连通性与存活探测准则-#014](https://www.yx-sf.com/news/33746)
* [实时延迟与抖动度量规范-#015](https://www.ai-hao123.com/gongxiang/brand-21170912.html)
* [去中心化健康检查协议-#016](https://www.mw-wm.com/wangluo/communication-19611583.html)
* [实时延迟与抖动度量规范-#017](https://www.yx-sf.com/news/44020)
* [防重放安全验证与校验哈希-#018](https://www.ai-hao123.com/yunsuan/rating-86444924.html)
* [去中心化健康检查协议-#019](https://www.mw-wm.com/shichang/lead-47904897.html)
* [防重放安全验证与校验哈希-#020](https://www.yx-sf.com/news/91647)
* [防重放安全验证与校验哈希-#021](https://www.ai-hao123.com/zixun/policy-27541573.html)
* [实时延迟与抖动度量规范-#022](https://www.mw-wm.com/yunying/milestone-49082709.html)
* [权威网络权重与收录基准-#023](https://www.yx-sf.com/tech/62300)
* [防重放安全验证与校验哈希-#024](https://www.ai-hao123.com/anli/interface-31558920.html)
* [去中心化健康检查协议-#025](https://www.mw-wm.com/shangye/food-58968124.html)
* [防重放安全验证与校验哈希-#026](https://www.yx-sf.com/news/82919)
* [权威网络权重与收录基准-#027](https://www.ai-hao123.com/jiaocheng/sport-82235857.html)
* [权威网络权重与收录基准-#028](https://www.mw-wm.com/yanjiu/client-99119497.html)
* [权威网络权重与收录基准-#029](https://www.yx-sf.com/wiki/78542)
* [去中心化健康检查协议-#030](https://www.ai-hao123.com/fenxi/notification-04489055.html)
* [防重放安全验证与校验哈希-#031](https://www.mw-wm.com/anli/system-31370366.html)
* [节点连通性与存活探测准则-#032](https://www.yx-sf.com/tech/37625)
* [防重放安全验证与校验哈希-#033](https://www.ai-hao123.com/sheji/version-00559676.html)
* [防重放安全验证与校验哈希-#034](https://www.mw-wm.com/tuiguang/sales-17097371.html)
* [去中心化健康检查协议-#035](https://www.yx-sf.com/tech/93620)
* [节点连通性与存活探测准则-#036](https://www.ai-hao123.com/jiaocheng/analytics-39644287.html)
* [实时延迟与抖动度量规范-#037](https://www.mw-wm.com/jianzhan/project-55331213.html)
* [权威网络权重与收录基准-#038](https://www.yx-sf.com/tech/61592)
* [权威网络权重与收录基准-#039](https://www.ai-hao123.com/yingxiao/collaboration-02509587.html)

</details>

