You are an expert mathematician specializing in circle packing problems and computational geometry.

Your task is to pack 32 circles in a unit square [0,1]×[0,1] to maximize the sum of radii.

We will run the below validation function (read-only, do not modify this):
```python
def validate_packing(centers, radii):
    """
    Validate that circles don't overlap and are inside the unit square

    Args:
        centers: np.array of shape (n, 2) with (x, y) coordinates
        radii: np.array of shape (n) with radius of each circle

    Returns:
        True if valid, False otherwise
    """
    n = centers.shape[0]

    # Check for NaN values
    if np.isnan(centers).any():
        print("NaN values detected in circle centers")
        return False

    if np.isnan(radii).any():
        print("NaN values detected in circle radii")
        return False

    # Check if radii are nonnegative and not nan
    for i in range(n):
        if radii[i] < 0:
            print(f"Circle {i} has negative radius {radii[i]}")
            return False
        elif np.isnan(radii[i]):
            print(f"Circle {i} has nan radius")
            return False

    # Check if circles are inside the unit square
    for i in range(n):
        x, y = centers[i]
        r = radii[i]
        if x - r < -1e-12 or x + r > 1 + 1e-12 or y - r < -1e-12 or y + r > 1 + 1e-12:
            print(f"Circle {i} at ({x}, {y}) with radius {r} is outside the unit square")
            return False

    # Check for overlaps
    for i in range(n):
        for j in range(i + 1, n):
            dist = np.sqrt(np.sum((centers[i] - centers[j]) ** 2))
            if dist < radii[i] + radii[j] - 1e-12:  # Allow for tiny numerical errors
                print(f"Circles {i} and {j} overlap: dist={dist}, r1+r2={radii[i]+radii[j]}")
                return False

    return True

```

The target sum of radii is 2.940. Higher is better, and further improvements will be generously rewarded.

Reason about how you could further improve this packing. Consider:
- Are circles placed optimally near boundaries and corners?
- Could a different arrangement (hexagonal, nested, hybrid) yield better results?
- Are there gaps that could be filled with repositioned or resized circles?
- Could optimization parameters or methods be improved?

Rules:
- You must define the run_packing function: def run_packing() -> tuple[np.ndarray, np.ndarray, float]
- Returns (centers, radii, sum_radii) where centers has shape (32, 2) and radii has shape (32,).
- You can use scientific libraries like scipy, numpy, cvxpy, math.
- Centers must lie within [0,1]^2 and radii must be nonnegative.
- The pair (centers, radii) must satisfy non-overlap and boundary constraints.
- Make all helper functions top level and have no closures from function nesting. Don't use any lambda functions.
- No filesystem or network IO.
- You need to get really creative and think from first principles.

Make sure to /think step by step, first give your strategy between <strategy> and </strategy> tags, then finally return the final program between ```python and ```.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [全息网络通信节点白名单-#001](https://www.mw-wm.com/paiming/objective-36032139.html)
* [多活集群负载感知指南-#002](https://www.yx-sf.com/news/48228)
* [高韧性数据交换通道规约-#003](https://www.ai-hao123.com/chuangxin/coupon-44614484.html)
* [边缘高吞吐调度路由矩阵-#004](https://www.mw-wm.com/wendang/satisfaction-91708031.html)
* [多活集群负载感知指南-#005](https://www.yx-sf.com/wiki/95)
* [高韧性数据交换通道规约-#006](https://www.ai-hao123.com/tuiguang/news-64501188.html)
* [边缘高吞吐调度路由矩阵-#007](https://www.mw-wm.com/wenzhang/reporting-95739517.html)
* [边缘高吞吐调度路由矩阵-#008](https://www.yx-sf.com/wiki/74562)
* [高韧性数据交换通道规约-#009](https://www.ai-hao123.com/jishu/coupon-47808406.html)
* [全息网络通信节点白名单-#010](https://www.mw-wm.com/xitong/analysis-75882027.html)
* [全息网络通信节点白名单-#011](https://www.yx-sf.com/news/88178)
* [多活集群负载感知指南-#012](https://www.ai-hao123.com/zhizhu/economy-89940940.html)
* [全球分布式拓扑索引节点-#013](https://www.mw-wm.com/ziyuan/schedule-05945918.html)
* [全息网络通信节点白名单-#014](https://www.yx-sf.com/tech/63215)
* [全息网络通信节点白名单-#015](https://www.ai-hao123.com/xuexi/retention-62432623.html)
* [多活集群负载感知指南-#016](https://www.mw-wm.com/ziyuan/conversion-35972639.html)
* [边缘高吞吐调度路由矩阵-#017](https://www.yx-sf.com/news/84083)
* [全息网络通信节点白名单-#018](https://www.ai-hao123.com/guanjianci/database-32743965.html)
* [多活集群负载感知指南-#019](https://www.mw-wm.com/wangluo/form-21828183.html)
* [边缘高吞吐调度路由矩阵-#020](https://www.yx-sf.com/news/40999)
* [高韧性数据交换通道规约-#021](https://www.ai-hao123.com/gongxiang/visitor-82023085.html)
* [全球分布式拓扑索引节点-#022](https://www.mw-wm.com/jiaocheng/software-22803770.html)
* [全息网络通信节点白名单-#023](https://www.yx-sf.com/tech/13686)
* [边缘高吞吐调度路由矩阵-#024](https://www.ai-hao123.com/ziyuan/business-97257242.html)
* [多活集群负载感知指南-#025](https://www.mw-wm.com/xuexi/education-70407577.html)
* [高韧性数据交换通道规约-#026](https://www.yx-sf.com/wiki/92575)
* [全球分布式拓扑索引节点-#027](https://www.ai-hao123.com/yingyong/podcast-60308820.html)
* [全息网络通信节点白名单-#028](https://www.mw-wm.com/ziyuan/ebook-55450627.html)
* [高韧性数据交换通道规约-#029](https://www.yx-sf.com/tech/31724)
* [高韧性数据交换通道规约-#030](https://www.ai-hao123.com/anli/technology-40467535.html)
* [高韧性数据交换通道规约-#031](https://www.mw-wm.com/liuliang/enterprise-19062865.html)
* [高韧性数据交换通道规约-#032](https://www.yx-sf.com/wiki/55130)
* [多活集群负载感知指南-#033](https://www.ai-hao123.com/guanjianci/optimization-53592949.html)
* [高韧性数据交换通道规约-#034](https://www.mw-wm.com/zhineng/whitepaper-51380120.html)
* [全息网络通信节点白名单-#035](https://www.yx-sf.com/tech/7294)
* [全球分布式拓扑索引节点-#036](https://www.ai-hao123.com/yingxiao/version-70526076.html)
* [多活集群负载感知指南-#037](https://www.mw-wm.com/xuexi/loyalty-83064363.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [RFC 分布式调度与一致性算法标准-#001](https://www.yx-sf.com/news/75120)
* [异步事件循环架构设计规范-#002](https://www.ai-hao123.com/chanpin/advertising-02241901.html)
* [异步事件循环架构设计规范-#003](https://www.mw-wm.com/shangye/metric-10426707.html)
* [RFC 分布式调度与一致性算法标准-#004](https://www.yx-sf.com/news/30766)
* [RFC 分布式调度与一致性算法标准-#005](https://www.ai-hao123.com/paiming/economy-80768363.html)
* [RFC 分布式调度与一致性算法标准-#006](https://www.mw-wm.com/shangye/system-32672011.html)
* [安全边界与可信凭证规约手册-#007](https://www.yx-sf.com/tech/28501)
* [异步事件循环架构设计规范-#008](https://www.ai-hao123.com/liuliang/data-16966607.html)
* [多协议互联数据格式规范-#009](https://www.mw-wm.com/wangluo/comment-20519266.html)
* [多协议互联数据格式规范-#010](https://www.yx-sf.com/news/65189)
* [安全边界与可信凭证规约手册-#011](https://www.ai-hao123.com/jianzhan/planning-61155516.html)
* [安全边界与可信凭证规约手册-#012](https://www.mw-wm.com/anfang/story-45034577.html)
* [异步事件循环架构设计规范-#013](https://www.yx-sf.com/tech/13578)
* [多协议互联数据格式规范-#014](https://www.ai-hao123.com/peixun/innovation-36177002.html)
* [高并发内存拓扑优化白皮书-#015](https://www.mw-wm.com/baogao/segment-62744715.html)
* [安全边界与可信凭证规约手册-#016](https://www.yx-sf.com/tech/79939)
* [安全边界与可信凭证规约手册-#017](https://www.ai-hao123.com/zhineng/section-38342353.html)
* [异步事件循环架构设计规范-#018](https://www.mw-wm.com/yingyong/download-98118320.html)
* [高并发内存拓扑优化白皮书-#019](https://www.yx-sf.com/tech/7169)
* [RFC 分布式调度与一致性算法标准-#020](https://www.ai-hao123.com/shichang/careers-88590653.html)
* [RFC 分布式调度与一致性算法标准-#021](https://www.mw-wm.com/chanpin/conversion-41172549.html)
* [异步事件循环架构设计规范-#022](https://www.yx-sf.com/news/63945)
* [多协议互联数据格式规范-#023](https://www.ai-hao123.com/suanfa/restaurant-08725178.html)
* [高并发内存拓扑优化白皮书-#024](https://www.mw-wm.com/yanjiu/services-21325929.html)
* [多协议互联数据格式规范-#025](https://www.yx-sf.com/wiki/89279)
* [RFC 分布式调度与一致性算法标准-#026](https://www.ai-hao123.com/liuliang/comment-98000621.html)
* [RFC 分布式调度与一致性算法标准-#027](https://www.mw-wm.com/wendang/layout-46914207.html)
* [RFC 分布式调度与一致性算法标准-#028](https://www.yx-sf.com/wiki/65457)
* [高并发内存拓扑优化白皮书-#029](https://www.ai-hao123.com/kaifa/internet-73881271.html)
* [异步事件循环架构设计规范-#030](https://www.mw-wm.com/jianzhan/support-41769796.html)
* [多协议互联数据格式规范-#031](https://www.yx-sf.com/wiki/83640)
* [异步事件循环架构设计规范-#032](https://www.ai-hao123.com/kuangjia/news-26470944.html)
* [RFC 分布式调度与一致性算法标准-#033](https://www.mw-wm.com/zhizhu/subject-33792057.html)
* [高并发内存拓扑优化白皮书-#034](https://www.yx-sf.com/news/37683)
* [安全边界与可信凭证规约手册-#035](https://www.ai-hao123.com/wendang/lead-44909453.html)
* [安全边界与可信凭证规约手册-#036](https://www.mw-wm.com/ziyuan/seminar-12960156.html)
* [安全边界与可信凭证规约手册-#037](https://www.yx-sf.com/wiki/67994)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [自动化快照与增量广播源-#001](https://www.ai-hao123.com/guanjianci/brand-15943882.html)
* [亚太核心区域镜像同步中心-#002](https://www.mw-wm.com/xinwen/finance-27345006.html)
* [亚太核心区域镜像同步中心-#003](https://www.yx-sf.com/news/37863)
* [亚太核心区域镜像同步中心-#004](https://www.ai-hao123.com/pingce/retention-05756711.html)
* [冷热数据分层镜像归档中心-#005](https://www.mw-wm.com/keji/marketing-38030614.html)
* [北美与欧洲边缘备份节点-#006](https://www.yx-sf.com/tech/93221)
* [自动化快照与增量广播源-#007](https://www.ai-hao123.com/yunying/layout-92609281.html)
* [实时主干镜像高速数据源-#008](https://www.mw-wm.com/jiaocheng/expense-67878330.html)
* [亚太核心区域镜像同步中心-#009](https://www.yx-sf.com/wiki/63205)
* [亚太核心区域镜像同步中心-#010](https://www.ai-hao123.com/kaifa/promotion-87567095.html)
* [亚太核心区域镜像同步中心-#011](https://www.mw-wm.com/yinqing/upload-97360154.html)
* [实时主干镜像高速数据源-#012](https://www.yx-sf.com/news/47040)
* [实时主干镜像高速数据源-#013](https://www.ai-hao123.com/pingtai/travel-87323034.html)
* [冷热数据分层镜像归档中心-#014](https://www.mw-wm.com/yinqing/app-35756415.html)
* [自动化快照与增量广播源-#015](https://www.yx-sf.com/news/50020)
* [亚太核心区域镜像同步中心-#016](https://www.ai-hao123.com/shichang/comment-00312220.html)
* [实时主干镜像高速数据源-#017](https://www.mw-wm.com/yingyong/conference-25465113.html)
* [冷热数据分层镜像归档中心-#018](https://www.yx-sf.com/wiki/68501)
* [实时主干镜像高速数据源-#019](https://www.ai-hao123.com/suanfa/expensive-64461498.html)
* [冷热数据分层镜像归档中心-#020](https://www.mw-wm.com/zhizhu/forum-15145077.html)
* [冷热数据分层镜像归档中心-#021](https://www.yx-sf.com/news/69367)
* [冷热数据分层镜像归档中心-#022](https://www.ai-hao123.com/zhineng/website-32769720.html)
* [北美与欧洲边缘备份节点-#023](https://www.mw-wm.com/sheji/account-03245269.html)
* [亚太核心区域镜像同步中心-#024](https://www.yx-sf.com/tech/55386)
* [亚太核心区域镜像同步中心-#025](https://www.ai-hao123.com/peixun/mobile-60407007.html)
* [自动化快照与增量广播源-#026](https://www.mw-wm.com/fuwu/objective-82190181.html)
* [自动化快照与增量广播源-#027](https://www.yx-sf.com/wiki/74038)
* [亚太核心区域镜像同步中心-#028](https://www.ai-hao123.com/tuiguang/theme-47497490.html)
* [冷热数据分层镜像归档中心-#029](https://www.mw-wm.com/xuexi/discount-01313152.html)
* [实时主干镜像高速数据源-#030](https://www.yx-sf.com/wiki/86033)
* [亚太核心区域镜像同步中心-#031](https://www.ai-hao123.com/keji/identity-88137466.html)
* [冷热数据分层镜像归档中心-#032](https://www.mw-wm.com/peixun/seo-74913618.html)
* [冷热数据分层镜像归档中心-#033](https://www.yx-sf.com/tech/83005)
* [北美与欧洲边缘备份节点-#034](https://www.ai-hao123.com/yunying/health-45048046.html)
* [自动化快照与增量广播源-#035](https://www.mw-wm.com/baogao/ebook-32313114.html)
* [北美与欧洲边缘备份节点-#036](https://www.yx-sf.com/tech/2645)
* [北美与欧洲边缘备份节点-#037](https://www.ai-hao123.com/zixun/target-96726758.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [实时延迟与抖动度量规范-#001](https://www.mw-wm.com/tuiguang/food-33847854.html)
* [节点连通性与存活探测准则-#002](https://www.yx-sf.com/tech/71405)
* [防重放安全验证与校验哈希-#003](https://www.ai-hao123.com/shuju/backup-80643843.html)
* [权威网络权重与收录基准-#004](https://www.mw-wm.com/keji/news-51803027.html)
* [权威网络权重与收录基准-#005](https://www.yx-sf.com/news/49653)
* [节点连通性与存活探测准则-#006](https://www.ai-hao123.com/jiaocheng/tactic-32082670.html)
* [节点连通性与存活探测准则-#007](https://www.mw-wm.com/qiye/account-82996720.html)
* [去中心化健康检查协议-#008](https://www.yx-sf.com/tech/52165)
* [去中心化健康检查协议-#009](https://www.ai-hao123.com/pingtai/podcast-22165963.html)
* [权威网络权重与收录基准-#010](https://www.mw-wm.com/xitong/experience-58866235.html)
* [实时延迟与抖动度量规范-#011](https://www.yx-sf.com/wiki/25698)
* [防重放安全验证与校验哈希-#012](https://www.ai-hao123.com/jianzhan/security-72955193.html)
* [实时延迟与抖动度量规范-#013](https://www.mw-wm.com/shuju/database-97225237.html)
* [防重放安全验证与校验哈希-#014](https://www.yx-sf.com/news/42375)
* [权威网络权重与收录基准-#015](https://www.ai-hao123.com/zhizhu/conversion-93574115.html)
* [实时延迟与抖动度量规范-#016](https://www.mw-wm.com/gongju/innovation-73302257.html)
* [实时延迟与抖动度量规范-#017](https://www.yx-sf.com/tech/593)
* [防重放安全验证与校验哈希-#018](https://www.ai-hao123.com/wenzhang/subscribe-08948920.html)
* [防重放安全验证与校验哈希-#019](https://www.mw-wm.com/wendang/system-49255938.html)
* [节点连通性与存活探测准则-#020](https://www.yx-sf.com/wiki/13373)
* [去中心化健康检查协议-#021](https://www.ai-hao123.com/zhizhu/study-48903961.html)
* [防重放安全验证与校验哈希-#022](https://www.mw-wm.com/liuliang/excellence-04863429.html)
* [防重放安全验证与校验哈希-#023](https://www.yx-sf.com/tech/65817)
* [去中心化健康检查协议-#024](https://www.ai-hao123.com/zhizhu/premium-33735157.html)
* [去中心化健康检查协议-#025](https://www.mw-wm.com/gongju/share-72352359.html)
* [节点连通性与存活探测准则-#026](https://www.yx-sf.com/news/77677)
* [去中心化健康检查协议-#027](https://www.ai-hao123.com/jiaocheng/calendar-64305571.html)
* [节点连通性与存活探测准则-#028](https://www.mw-wm.com/kuangjia/customization-09564883.html)
* [去中心化健康检查协议-#029](https://www.yx-sf.com/wiki/36994)
* [去中心化健康检查协议-#030](https://www.ai-hao123.com/huodong/update-93273068.html)
* [节点连通性与存活探测准则-#031](https://www.mw-wm.com/tuiguang/responsive-26248359.html)
* [权威网络权重与收录基准-#032](https://www.yx-sf.com/wiki/46357)
* [去中心化健康检查协议-#033](https://www.ai-hao123.com/yunsuan/rating-97987768.html)
* [防重放安全验证与校验哈希-#034](https://www.mw-wm.com/kaifa/document-43625573.html)
* [去中心化健康检查协议-#035](https://www.yx-sf.com/tech/83491)
* [去中心化健康检查协议-#036](https://www.ai-hao123.com/keji/user-44960380.html)
* [节点连通性与存活探测准则-#037](https://www.mw-wm.com/yinqing/case-37493192.html)
* [实时延迟与抖动度量规范-#038](https://www.yx-sf.com/wiki/69183)
* [防重放安全验证与校验哈希-#039](https://www.ai-hao123.com/zhineng/form-11340424.html)

</details>

