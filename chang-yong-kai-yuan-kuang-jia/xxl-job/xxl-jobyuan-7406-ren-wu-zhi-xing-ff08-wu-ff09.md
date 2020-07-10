# 1.XXL-JOB原理--任务执行（五）
## 1.1.基本介绍
任务调度中心发送任务执行请求
任务发送执行的操作有两种：

* （1）根据配置的cron表达式周期性执行相关任务

* （2）在任务调度中心主动执行任务

在注册quartz定时任务时已经注册执行类为RemoteHttpJobBean，所以周期性执行定时任务会调用RemoteHttpJobBean的executeInternal方法，在executeInternal中会调用JobTriggerPoolHelper.trigger(jobId)，通过任务调度中心主动执行任务时也是会调用JobTriggerPoolHelper.trigger(jobId)方法，所以接下来我们要看的是JobTriggerPoolHelper.trigger(jobId)中做的逻辑处理就好。


```
public class RemoteHttpJobBean extends QuartzJobBean {
	private static Logger logger = LoggerFactory.getLogger(RemoteHttpJobBean.class);
 
	@Override
	protected void executeInternal(JobExecutionContext context)
			throws JobExecutionException {
 
		// load jobId
		JobKey jobKey = context.getTrigger().getJobKey();
		Integer jobId = Integer.valueOf(jobKey.getName());
 
		// trigger
		//XxlJobTrigger.trigger(jobId);
		JobTriggerPoolHelper.trigger(jobId);
	}
 
}

```

JobTriggerPoolHelper.trigger所做的操作是将任务提交给一个线程池（任务调度中心默认开启50个线程），在线程池中调用XxlJobTrigger.trigger。


```
public void addTrigger(final int jobId){
        triggerPool.execute(new Runnable() {
            @Override
            public void run() {
                XxlJobTrigger.trigger(jobId);
            }
        });
    }

```



