# 1.XXL-JOB原理--任务执行（五）
## 1.1.基本介绍
任务调度中心发送任务执行请求
任务发送执行的操作有两种：

* （1）根据配置的cron表达式周期性执行相关任务

* （2）在任务调度中心主动执行任务

在注册quartz定时任务时已经注册执行类为RemoteHttpJobBean，所以周期性执行定时任务会调用RemoteHttpJobBean的executeInternal方法，在executeInternal中会调用JobTriggerPoolHelper.trigger(jobId)，通过任务调度中心主动执行任务时也是会调用JobTriggerPoolHelper.trigger(jobId)方法，所以接下来我们要看的是JobTriggerPoolHelper.trigger(jobId)中做的逻辑处理就好。
————————————————
版权声明：本文为CSDN博主「归田」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq924862077/article/details/82717577