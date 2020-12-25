---
title: Microservices (微服务)
date: 2021-09-10 13:25:11
summary:
tags: 
    - arch
    - microservices
categories:
    - arch
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co43-m.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co43.jpg
---

## 微服务设计
![服务设计原则](https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/post-content-pic/micro-service/service-design-principle.jpg)

## 服务稳定性
### 开发流程
- 开发前: 需求方案是否支持业务未来短期的内发展。梳理业务流程。强依赖、弱依赖。核心、非核心。
- 开发: 研发规范, 端上版本兼容。合理的日志、埋点, 方便监控和快速定位问题。
- review: code review, 空指针、协程泄露、死循环等测试阶段不容易发现的问题。配置 review, 数据库review。
- 测试: 较为完善的测试用例和测试流程, 保证核心流程没有严重bug
- 上线: 提前通知依赖方服务上线, 或确认是否要依赖方配合升级接口、平滑发布。梳理好回滚方案, 或者准备开关。
- 上线后: 线上及时回归测试。定期观察业务数据、系统监控、告警等。
- 维护: 容量规划、弹性扩容、容灾、异地多活。限流、熔断。
- 长期工作: 链路压测、混沌工程。链路追踪, 方便快速定位问题。


### 技术手段
- 系统水位告警
- 业务指标报警
- 弹性扩容
- 机器容灾
- 业务容灾
- 人工预案
- 自动降级
- 熔断
- 削峰
- 重试
- 故障演练
- 全链路压测

## 服务拆分的思路
1. 根据业务划分
2. 根据调用链层次划分, 在一个业务线中, 有b端、c端, 或者偏前台, 偏后台, 偏引擎等职责的服务。
3. 根据能力划分(业务服务多了以后, 可以抽象出公共能力, 如基础服务、中台、平台、等)
4. 根据核心/非核心功能来划分(提升核心服务稳定性, 非核心业务发布、重启、迭代、故障都不会影响核心服务。上下游依赖的话也要做好降级熔断。)
5. 根据组织架构划分(顺应公司战略目标, 服务的拆分也可以一定程度上参考组织架构的划分, 专业的人做更专业的事情)
6. 根据租户划分(基础服务、基础平台抽象出来后, 会同时被核心、非核心业务接入, 如果遇到风险, 在迫不得已时可以降级非核心业务)
7. 泳道化、set化(与租户划分思路类似, 适合一些特殊场景)


## 参考文章
1. [凤凰架构-微服务时代](http://icyfenix.cn/architecture/architect-history/microservices.html)
2. [思考：如何保证服务稳定性？](https://zhuanlan.zhihu.com/p/150377297?from_voters_page=true)

https://blog.csdn.net/qq_35606175/article/details/88402611

https://martinfowler.com/articles/microservices.html

https://martinfowler.com/microservices/

https://martinfowler.com/bliki/PresentationDomainDataLayering.html