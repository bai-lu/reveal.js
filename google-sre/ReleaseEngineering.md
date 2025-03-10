# Release Engineering



## 目录

1. 发布工程简介 <!-- 发布工程指的是什么 -->
2. 发布工程哲学  <!-- 发布工程的4个主要原则 -->
3. 持续构建与部署 <!-- google 是怎么做持续构建和部署的, 我米常见的CI/CD场景是怎么做的 -->
4. 配置管理  <!-- 配置管理是发布工程的一环, 在这个标题我会向大家分享几种配置管理的方式 -->
5. 思考和建议 <!-- 云服务的发布工程经历了的3个阶段,  SRE和发布工程师以及研发团队怎么配合发布工程的落地  -->



## 发布工程

软件工程内部的一个学科, 这个学科专注于**构建**和**交付**软件

Note: 首先从学术视角:  软件工程内部的一个学科, 这个学科专注于**构建**和**交付**软件, 发布工程关注从 源代码管理->编译->打包->发布 的整个流程


## 发布 ➡ 工程化

为保障服务可靠性运行需要可靠的发布流程, SRE需要保证二进制文件和配置文件是以一种**自动化的, 可重现的**方式构建出来的

1. 自动化  <!-- .element: class="fragment" data-fragment-index="1" --> <!-- 之前羽成哥分享的自动化章节, 自动化非常重要, 自动话的价值在于解决效率问题, 减少人工参与, 速度快, 不出错 -->
2. 变更可回溯, 强调流程及安全策略   <!-- .element: class="fragment" data-fragment-index="2" -->  <!-- 为保障服务可靠性运行不仅需要高效的发布, 还需要保障发布流程是可靠的, 可回溯的 -->

Note: 站在 SRE/工程师视角 就很简单了, 将发布这个重复且重要的动作 工程化 



## 发布系统哲学 <!-- 发布工程的4个主要原则 -->


## 自服务模型

发布过程是**自动化**的, 工程师仅在出现问题时才需要干预

Note: 自动化程度足够高, 自动化到基本不需要人参与, google 的基础设施很厉害, 这个早自动化就实现到这个水平了


## 追求速度⚡️

1. 提升用户体验的变更应该快速上线 <!-- .element: class="fragment" data-fragment-index="1" -->
2. 频繁发布使每个线上版本差异变小 <!-- .element: class="fragment" data-fragment-index="2" -->

Note: 1. 雷总提到互联网7字诀: 专注口碑极致快, 站在提升用户体验的角度, 让用户更早地体验新feature, 在互联网的竞争中取得优势 2. 频繁的发布使, 工程师测试和故障排除更容易 , 距个例子: 有的服务变更很频繁, 有的服务变更很少, 长时间不更新 代码的很多公共依赖都升级了, 这时候进行发布非常容易出问题而且很难去排查修复


## 密闭性

构建工具必须确保**一致性**和**可重复性**

<!-- Note: 一致性: 环境无关, 两台不同的机器上同一版本编译产物应该是相同的 -->
<!-- Note: 可重复性: 编译工具保障 在回滚时和调试时 可复现旧版本 -->


## 强调流程和策略

1. 代码评审 <!-- .element: class="fragment" data-fragment-index="1" -->  <!-- code review Merge 到主分支权限管理 -->
2. 权限管理 <!-- .element: class="fragment" data-fragment-index="2" -->  <!-- 权限收敛, 确保在发布过程中只有指定的人才能执行指定的操作 -->
3. 审计 <!-- .element: class="fragment" data-fragment-index="3" --> <!-- 每次发布的审批过程, 自动测试报告, 编译发布日志一起存档, 方便后续排查问题 -->

Note: 多重安全和访问控制 机制可以确保在发布过程中只有指定的人才能执行指定的操作 发布工程不是仅仅最求效率, 还要保证 发布过程是可靠



## 持续构建和部署


## google Rapid

![External Image](https://lh3.googleusercontent.com/jqiQKoCx7lZ40RJVF7fGYCbAJ4cB5SmMs1TeeGafya_qu50UyNWO97EAE1mNdO00vN3pQwrUC5gYvajlGELrSnaa7FSX1idvuccc=s900)

Note: 
蓝色部分: 编排文件, 定义了工作流 类似于 gitlab-ci.yaml 或 github action, 
蓝图以内部配置语言编写，用于定义构建和测试目标、部署规则和管理信息（如项目所有者）。基于角色的访问控制列表确定谁可以对 Rapid 项目执行特定操作
红色部分: rapid系统组件, 提供CI/CD 能力 , 类似 gitlab runner
绿色部分: 外部对接服务, 对接了自动测试服务,  git, 编译打包服务, 部署服务
-- 这样看, 我米内部都提供了类似的系统上, 不过各个系统的没什么关联, 主要看各个业务的应用程度, 不同业务间可能差别很大,  云服务的nginx 一站式管理用 gitlab ci 编排了我米多个系统, 比如编译发布系统, k8s, 镜像服务, 域名监控, 证书自动更新, 实现了发布工程的落地


## 云服务nginx一站式管理

```yaml [2|3-4|5-7|8-10|11]
stages:
  - syntax-check # 测试
  - build_package # 编译
  - build_image
  - zkups-preview-deploy  # 灰度
  - conf-preview-deploy
  - monitor-scripts-preview-deploy
  - zkups-production-deploy  # 部署
  - conf-production-deploy
  - monitor-scripts-production-deploy
  - update-domain-monitor # 监控
```

![External Image](assets/pipeline.png)


![External Image](assets/nginx-deploy.png)

Note: 
1. 每次commit 都会提交一次 自动测试
2. 提交MR 团队内做code review
3. 选出一个commit 作为上线版本, 进行组内 approve 审批
4. 调用发布系统工具/k8s 进行编译, 部署
5. 附: gitlab pipeline https://git.n.xiaomi.com/micloud-sre/nginx-deploy/-/pipelines/1033926
6. 附: github actions https://github.com/bai-lu/docker-clash/runs/3226048259



## 配置管理

1. 配置文件和二进制打成一个包 <!-- 大部分简单的项目 -->
2. 配置文件独立部署  <!-- nginx 将配置文件和二进制部署拆开的形式 -->
3. 配置文件放到外部存储 <!-- 需要做一些开关, 控制灰度流量等场景 zookeeper etcd consul  -->

总结: 根据具体情况决定最有效的方式
<!-- 配置管理也是发布工程的一环, SRE需要保证二进制文件和配置文件是以一种可重现的, 自动化的方式构建出来的 -->



## 思考和建议


## 云服务发布工程升级历程


## 初期手动上线

SRE负责所有业务变更的上线, 受限于工具水平, 需要**手动**编译和发布

Note: 早期集群较少, 服务较为简单, 虽然 SRE 累些也可以稳定上线
SRE 作为变更接口人
后来服务, 集群越来越多, 手动上线太慢, SRE 精力消耗在上线太多, hold 不住了


## 初步自动化

服务迁移融合云发布系统, 实现了编译自动化, 多集群并发发布

Note: 缓解了SRE的上线压力, 非核心上线交由对应研发操作, 工程师依然要在发布上花费大量精力


## 容器化

服务迁移MICE + 原生k8s

Note: 发布工程基本落地, 上线成本降低, MICE的镜像是自动构建的, 发布只需要页面点几下 , 原生k8s的上线做了CI/CD, 只需要提交git, 后面上线都是自动的, 研发团队发现我自己上和SRE上流程完全一样, 还不用等SRE 有空了操作,  于是现在主要是变更发起人操作上线
研发变更的是服务
sre变更的是nginx


## 总结: 

1. 发布工程高效落地依赖平台的自动化程度 <!--工具好用了, 工程师干活效率才高 -->
2. 发布工程高效落地, 工程师才能聚焦于业务功能 <!-- 研发业务功能迭代, SRE管理业务入口 -->


## 发布工程的角色分工


* 发布工程师: 发布工程领域的专家, 给出最佳实践, 使各个团队使用统一的发布流程, 避免重复造轮子 <!-- 往往是不专业的, 接受研发团队和SRE的需求-->
* 研发团队: 适配最佳实践 <!-- 研发团队不应该只关系业务, 不关心发布工程, 将发布工作仍给其他工程师处理。 这样工作量没有减少, 只是在团队间进行转移, 应该积极配合 SRE和发布工程师 适配最佳实践, 将高效 可靠的发布工程落地 -->
* SRE: 与发布工程师和研发团队紧密合作, 推动发布工程落地 <!-- 根据业务特征向发布工厂师提需求, 主动发现发布流程中效率低和不可靠的环节并反馈优化-->

<!-- 总结: 合作共赢 -->


## 从一开始就进行发布工程


团队应该在开发流程开始就留出一定资源进行发布工程工作, 今早采用最佳实践和最佳流程可以降低成本, 以免未来重新改动这些系统

<!--感到疼了再改, 代价很高 1. 后期适配成本 2. 很多人感到累了, 工具不好用人给气跑了-->



## Thanks