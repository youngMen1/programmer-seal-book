# 1.分布式调度项目ElasticJob正式迁入Apache仓库
## 1.1.基本介绍
ElasticJob（https://github.com/elasticjob）是一个分布式调度解决方案，提供分布式任务的分片，弹性伸缩，全自动发现，基于时间驱动、数据驱动、常驻任务和临时任务的多任务类型，任务聚合和动态调配资源，故障检测、自动修复，失效转移和重试，完善的运维平台和管理工具，以及对云原生的良好支持等功能特性，可以全面满足企业对于任务管理和批量作业的全面调度处理能力。
ElasticJob自2014年底开源以来，经历了5年多的发展，以其功能的丰富性，文档的全面性，代码的高质量，框架的易用性，积累了大量的忠实用户和良好的业内口碑（5.8K star），一直也是分布式调度框架领域最受大家欢迎的项目之一。

2020年6月，经过Apache ShardingSphere社区投票，接纳ElasticJob为其子项目。目前ElasticJob的四个子项目已经正式迁入Apache仓库。

![](/static/image/sdfsdfsdf44.jpg)

## 1.2.说明
Elastic-Job-Lite定位为轻量级无中心化解决方案，使用jar包的形式提供分布式任务的协调服务。
仓库地址：`https://github.com/apache/shardingsphere-elastic-job-lite`

Elastic-Job-Cloud采用自研Mesos Framework的解决方案，额外提供资源治理、应用分发以及进程隔离等功能。
仓库地址：`https://github.com/apache/shardingsphere-elastic-job-cloud`

官方文档
仓库地址：`https://github.com/apache/shardingsphere-elastic-job-doc`

官方Elastic-Job 示例
仓库地址：`https://github.com/apache/shardingsphere-elastic-job-example`

后续ElasticJob会作为Apache ShardingSphere ElasticJob子项目继续发展，并融入Apache ShardingSphere解决方案，成为生态体系中的一个重要调度组件。并且可以作为全功能的分布式调度框架，独立应用于各种业务调度场景。


