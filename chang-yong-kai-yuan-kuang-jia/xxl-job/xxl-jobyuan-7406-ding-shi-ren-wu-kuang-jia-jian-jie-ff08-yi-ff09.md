# 1.XXL-JOB原理--定时任务框架简介（一）
**官方介绍：**
```
http://www.xuxueli.com/xxl-job/#/?id=%E4%B8%80%E3%80%81%E7%AE%80%E4%BB%8B
```
**最新版本架构图：**
![](/static/image/微信截图_20200710140240.png)
## 1.1.基本介绍
目前我们在项目中可能接触到定时任务框架quartz，应用也是比较广泛的，其也是支持分布式任务调度的，
通过数据库竞争锁来实现，当然会有很多的局限性（可能这也是xxl-job出现的原因），
quartz支持多种数据库（`https://github.com/quartz-scheduler/quartz/tree/master/quartz-core/src/main/resources/org/quartz/impl/jdbcjobstore`），
xxl-job其实也是在quartz的基础上实现的，但是修改了任务调度的模式，并且任务调度采用注册和RPC调用方式来实现。
20180910211104223.png

### 技术栈
mysql、SSM，内置jetty作为RPC服务调用、quartz

### xxl-job支持Postgresql数据库
目前由于xxl-job只支持mysql数据库，目前在github上拉了一个分支支持Postgresql 地址GitHub地址
`https://github.com/IAMTJW/xxl-job`



