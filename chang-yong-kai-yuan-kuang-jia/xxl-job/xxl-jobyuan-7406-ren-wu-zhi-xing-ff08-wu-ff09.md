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

在XxlJobTrigger.trigger中会根据jobId获取任务的基本配置信息（阻塞策略、路由策略、失败重试测试、分组服务器列表等等），然后根据路由策略选择是广播还是单播等，接下来就是组装消息体调用runExecutor方法发送http请求到任务执行器。



```
public static void trigger(int jobId) {
 
		//获取任务信息
        // load data
        XxlJobInfo jobInfo = XxlJobDynamicScheduler.xxlJobInfoDao.loadById(jobId);              // job info
        if (jobInfo == null) {
            logger.warn(">>>>>>>>>>>> trigger fail, jobId invalid，jobId={}", jobId);
            return;
        }
		//根据任务的分组信息找到分组，分组中存在服务器的IP和端口地址等
        XxlJobGroup group = XxlJobDynamicScheduler.xxlJobGroupDao.load(jobInfo.getJobGroup());  // group info
		
		//阻塞策略
        ExecutorBlockStrategyEnum blockStrategy = ExecutorBlockStrategyEnum.match(jobInfo.getExecutorBlockStrategy(), ExecutorBlockStrategyEnum.SERIAL_EXECUTION);  // block strategy
        //失败策略
		ExecutorFailStrategyEnum failStrategy = ExecutorFailStrategyEnum.match(jobInfo.getExecutorFailStrategy(), ExecutorFailStrategyEnum.NULL);    // fail strategy
        //执行路由测试
		ExecutorRouteStrategyEnum executorRouteStrategyEnum = ExecutorRouteStrategyEnum.match(jobInfo.getExecutorRouteStrategy(), null);    // route strategy
		//服务器地址
        ArrayList<String> addressList = (ArrayList<String>) group.getRegistryList();
 
		//广播模式
        // broadcast
        if (ExecutorRouteStrategyEnum.SHARDING_BROADCAST == executorRouteStrategyEnum && CollectionUtils.isNotEmpty(addressList)) {
			//依次调用所有的服务器
            for (int i = 0; i < addressList.size(); i++) {
                String address = addressList.get(i);
 
                // 1、save log-id
                XxlJobLog jobLog = new XxlJobLog();
                jobLog.setJobGroup(jobInfo.getJobGroup());
                jobLog.setJobId(jobInfo.getId());
                XxlJobDynamicScheduler.xxlJobLogDao.save(jobLog);
                logger.debug(">>>>>>>>>>> xxl-job trigger start, jobId:{}", jobLog.getId());
 
                // 2、prepare trigger-info
                //jobLog.setExecutorAddress(executorAddress);
                jobLog.setGlueType(jobInfo.getGlueType());
                jobLog.setExecutorHandler(jobInfo.getExecutorHandler());
                jobLog.setExecutorParam(jobInfo.getExecutorParam());
                jobLog.setTriggerTime(new Date());
 
                ReturnT<String> triggerResult = new ReturnT<String>(null);
                StringBuffer triggerMsgSb = new StringBuffer();
                triggerMsgSb.append(I18nUtil.getString("jobconf_trigger_admin_adress")).append("：").append(IpUtil.getIp());
                triggerMsgSb.append("<br>").append(I18nUtil.getString("jobconf_trigger_exe_regtype")).append("：")
                        .append( (group.getAddressType() == 0)?I18nUtil.getString("jobgroup_field_addressType_0"):I18nUtil.getString("jobgroup_field_addressType_1") );
                triggerMsgSb.append("<br>").append(I18nUtil.getString("jobconf_trigger_exe_regaddress")).append("：").append(group.getRegistryList());
                triggerMsgSb.append("<br>").append(I18nUtil.getString("jobinfo_field_executorRouteStrategy")).append("：").append(executorRouteStrategyEnum.getTitle()).append("("+i+"/"+addressList.size()+")"); // update01
                triggerMsgSb.append("<br>").append(I18nUtil.getString("jobinfo_field_executorBlockStrategy")).append("：").append(blockStrategy.getTitle());
                triggerMsgSb.append("<br>").append(I18nUtil.getString("jobinfo_field_executorFailStrategy")).append("：").append(failStrategy.getTitle());
                triggerMsgSb.append("<br>").append(I18nUtil.getString("jobinfo_field_timeout")).append("：").append(jobInfo.getExecutorTimeout());
 
                // 3、trigger-valid
                if (triggerResult.getCode()==ReturnT.SUCCESS_CODE && CollectionUtils.isEmpty(addressList)) {
                    triggerResult.setCode(ReturnT.FAIL_CODE);
                    triggerMsgSb.append("<br>----------------------<br>").append(I18nUtil.getString("jobconf_trigger_address_empty"));
                }
 
                if (triggerResult.getCode() == ReturnT.SUCCESS_CODE) {
                    // 4.1、trigger-param
                    TriggerParam triggerParam = new TriggerParam();
                    triggerParam.setJobId(jobInfo.getId());
                    triggerParam.setExecutorHandler(jobInfo.getExecutorHandler());
                    triggerParam.setExecutorParams(jobInfo.getExecutorParam());
                    triggerParam.setExecutorBlockStrategy(jobInfo.getExecutorBlockStrategy());
                    triggerParam.setExecutorTimeout(jobInfo.getExecutorTimeout());
                    triggerParam.setLogId(jobLog.getId());
                    triggerParam.setLogDateTim(jobLog.getTriggerTime().getTime());
                    triggerParam.setGlueType(jobInfo.getGlueType());
                    triggerParam.setGlueSource(jobInfo.getGlueSource());
                    triggerParam.setGlueUpdatetime(jobInfo.getGlueUpdatetime().getTime());
                    triggerParam.setBroadcastIndex(i);
                    triggerParam.setBroadcastTotal(addressList.size()); // update02
 
                    // 4.2、trigger-run (route run / trigger remote executor)
					//远程调用服务接口，执行任务
                    triggerResult = runExecutor(triggerParam, address);     // update03
                    triggerMsgSb.append("<br><br><span style=\"color:#00c0ef;\" > >>>>>>>>>>>"+ I18nUtil.getString("jobconf_trigger_run") +"<<<<<<<<<<< </span><br>").append(triggerResult.getMsg());
 
                    // 4.3、trigger (fail retry)
                    if (triggerResult.getCode()!=ReturnT.SUCCESS_CODE && failStrategy == ExecutorFailStrategyEnum.FAIL_TRIGGER_RETRY) {
                        triggerResult = runExecutor(triggerParam, address);  // update04
                        triggerMsgSb.append("<br><br><span style=\"color:#F39C12;\" > >>>>>>>>>>>"+ I18nUtil.getString("jobconf_fail_trigger_retry") +"<<<<<<<<<<< </span><br>").append(triggerResult.getMsg());
                    }
                }
 
                // 5、save trigger-info
                jobLog.setExecutorAddress(triggerResult.getContent());
                jobLog.setTriggerCode(triggerResult.getCode());
                jobLog.setTriggerMsg(triggerMsgSb.toString());
                XxlJobDynamicScheduler.xxlJobLogDao.updateTriggerInfo(jobLog);
 
                // 6、monitor trigger
                JobFailMonitorHelper.monitor(jobLog.getId());
                logger.debug(">>>>>>>>>>> xxl-job trigger end, jobId:{}", jobLog.getId());
 
            }
        } else {
			//单播模式
            // 1、save log-id
            XxlJobLog jobLog = new XxlJobLog();
            jobLog.setJobGroup(jobInfo.getJobGroup());
            jobLog.setJobId(jobInfo.getId());
            XxlJobDynamicScheduler.xxlJobLogDao.save(jobLog);
            logger.debug(">>>>>>>>>>> xxl-job trigger start, jobId:{}", jobLog.getId());
 
            // 2、prepare trigger-info
            //jobLog.setExecutorAddress(executorAddress);
            jobLog.setGlueType(jobInfo.getGlueType());
            jobLog.setExecutorHandler(jobInfo.getExecutorHandler());
            jobLog.setExecutorParam(jobInfo.getExecutorParam());
            jobLog.setTriggerTime(new Date());
 
            ReturnT<String> triggerResult = new ReturnT<String>(null);
            StringBuffer triggerMsgSb = new StringBuffer();
            triggerMsgSb.append(I18nUtil.getString("jobconf_trigger_admin_adress")).append("：").append(IpUtil.getIp());
            triggerMsgSb.append("<br>").append(I18nUtil.getString("jobconf_trigger_exe_regtype")).append("：")
                    .append( (group.getAddressType() == 0)?I18nUtil.getString("jobgroup_field_addressType_0"):I18nUtil.getString("jobgroup_field_addressType_1") );
            triggerMsgSb.append("<br>").append(I18nUtil.getString("jobconf_trigger_exe_regaddress")).append("：").append(group.getRegistryList());
            triggerMsgSb.append("<br>").append(I18nUtil.getString("jobinfo_field_executorRouteStrategy")).append("：").append(executorRouteStrategyEnum.getTitle());
            triggerMsgSb.append("<br>").append(I18nUtil.getString("jobinfo_field_executorBlockStrategy")).append("：").append(blockStrategy.getTitle());
            triggerMsgSb.append("<br>").append(I18nUtil.getString("jobinfo_field_executorFailStrategy")).append("：").append(failStrategy.getTitle());
            triggerMsgSb.append("<br>").append(I18nUtil.getString("jobinfo_field_timeout")).append("：").append(jobInfo.getExecutorTimeout());
 
            // 3、trigger-valid
            if (triggerResult.getCode()==ReturnT.SUCCESS_CODE && CollectionUtils.isEmpty(addressList)) {
                triggerResult.setCode(ReturnT.FAIL_CODE);
                triggerMsgSb.append("<br>----------------------<br>").append(I18nUtil.getString("jobconf_trigger_address_empty"));
            }
 
            if (triggerResult.getCode() == ReturnT.SUCCESS_CODE) {
                // 4.1、trigger-param
                TriggerParam triggerParam = new TriggerParam();
                triggerParam.setJobId(jobInfo.getId());
                triggerParam.setExecutorHandler(jobInfo.getExecutorHandler());
                triggerParam.setExecutorParams(jobInfo.getExecutorParam());
                triggerParam.setExecutorBlockStrategy(jobInfo.getExecutorBlockStrategy());
                triggerParam.setExecutorTimeout(jobInfo.getExecutorTimeout());
                triggerParam.setLogId(jobLog.getId());
                triggerParam.setLogDateTim(jobLog.getTriggerTime().getTime());
                triggerParam.setGlueType(jobInfo.getGlueType());
                triggerParam.setGlueSource(jobInfo.getGlueSource());
                triggerParam.setGlueUpdatetime(jobInfo.getGlueUpdatetime().getTime());
                triggerParam.setBroadcastIndex(0);
                triggerParam.setBroadcastTotal(1);
 
                // 4.2、trigger-run (route run / trigger remote executor)
				//路由后远程调用服务接口，执行任务
                triggerResult = executorRouteStrategyEnum.getRouter().routeRun(triggerParam, addressList);
                triggerMsgSb.append("<br><br><span style=\"color:#00c0ef;\" > >>>>>>>>>>>"+ I18nUtil.getString("jobconf_trigger_run") +"<<<<<<<<<<< </span><br>").append(triggerResult.getMsg());
 
                // 4.3、trigger (fail retry)
                if (triggerResult.getCode()!=ReturnT.SUCCESS_CODE && failStrategy == ExecutorFailStrategyEnum.FAIL_TRIGGER_RETRY) {
                    triggerResult = executorRouteStrategyEnum.getRouter().routeRun(triggerParam, addressList);
                    triggerMsgSb.append("<br><br><span style=\"color:#F39C12;\" > >>>>>>>>>>>"+ I18nUtil.getString("jobconf_fail_trigger_retry") +"<<<<<<<<<<< </span><br>").append(triggerResult.getMsg());
                }
            }
 
            // 5、save trigger-info
            jobLog.setExecutorAddress(triggerResult.getContent());
            jobLog.setTriggerCode(triggerResult.getCode());
            jobLog.setTriggerMsg(triggerMsgSb.toString());
            XxlJobDynamicScheduler.xxlJobLogDao.updateTriggerInfo(jobLog);
 
            // 6、monitor trigger
            JobFailMonitorHelper.monitor(jobLog.getId());
            logger.debug(">>>>>>>>>>> xxl-job trigger end, jobId:{}", jobLog.getId());
        }
 
    }

```

runExecutor方法中根据address服务器地址，XxlJobDynamicScheduler.getExecutorBiz中会获取代理类最终调用JettyClient的send方法。


