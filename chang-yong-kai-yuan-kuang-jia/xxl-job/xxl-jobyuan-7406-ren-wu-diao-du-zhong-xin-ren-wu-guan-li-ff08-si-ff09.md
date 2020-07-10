# 1.XXL-JOB原理--任务调度中心任务管理（四）

## 1.1.基本介绍

在任务调度中心可以进行新建任务，新建任务之后可以在任务列表中查看相关任务，任务可以根据我们配置的cron表达式进行任务调度，或者也可以在任务列表中执行、暂停、删除和查看相关运行日志等操作。

### 1.1.1.任务调度中心管理

1.新建任务  
 ![](/static/image/2018091514365420.png)

2.任务列表任务操作  
 ![](/static/image/20180915143942763.png)

## 1.2.任务创建与操作

我们了解到xxl-job是基于quartz来实现定时任务的\(其实任务调度中心任务执行基于quartz，任务执行器就是一个被调用执行的服务\)

### 1.2.1.任务创建

调用JobInfoController的add方法添加任务

```
    @RequestMapping("/add")
    @ResponseBody
    public ReturnT<String> add(XxlJobInfo jobInfo) {
        // 添加定时器
        return xxlJobService.add(jobInfo);
    }
```
#### 1.任务创建
调用XxlJobService的add方法，新建quartz任务

```
@Override
    public ReturnT<String> add(XxlJobInfo jobInfo) {
        // valid
        // 获取任务分组
        XxlJobGroup group = xxlJobGroupDao.load(jobInfo.getJobGroup());

        // 省略部分任务相关信息检查的代码

        // add in db
        // 将任务持久化到数据库的xxl_job_qrtz_trigger_info
        xxlJobInfoDao.save(jobInfo);
        if (jobInfo.getId() < 1) {
            return new ReturnT<String>(ReturnT.FAIL_CODE, (I18nUtil.getString("jobinfo_field_add")+I18nUtil.getString("system_fail")) );
        }

        // add in quartz
        String qz_group = String.valueOf(jobInfo.getJobGroup());
        String qz_name = String.valueOf(jobInfo.getId());
        try {
            // 添加任务到quartz中
            XxlJobDynamicScheduler.addJob(qz_name, qz_group, jobInfo.getJobCron());
            //XxlJobDynamicScheduler.pauseJob(qz_name, qz_group);
            return ReturnT.SUCCESS;
        } catch (SchedulerException e) {
            logger.error(e.getMessage(), e);
            try {
                xxlJobInfoDao.delete(jobInfo.getId());
                XxlJobDynamicScheduler.removeJob(qz_name, qz_group);
            } catch (SchedulerException e1) {
                logger.error(e.getMessage(), e1);
            }
            return new ReturnT<String>(ReturnT.FAIL_CODE, (I18nUtil.getString("jobinfo_field_add")+I18nUtil.getString("system_fail"))+":" + e.getMessage());
        }
    }
```

数据库持久化数据：

![](/static/image/20180915163211609.png)

在XxlJobDynamicScheduler中调用quartz的创建任务方法构建定时任务，定时任务执行类为QuartzJobBean的子类RemoteHttpJobBean，在定时任务执行时RemoteHttpJobBean会调用任务执行器接口，执行相关任务


```
public static boolean addJob(String jobName, String jobGroup, String cronExpression) throws SchedulerException {
    	// TriggerKey : name + group
		//创建定时器别名
        TriggerKey triggerKey = TriggerKey.triggerKey(jobName, jobGroup);
        JobKey jobKey = new JobKey(jobName, jobGroup);
        
        // TriggerKey valid if_exists
        if (checkExists(jobName, jobGroup)) {
            logger.info(">>>>>>>>> addJob fail, job already exist, jobGroup:{}, jobName:{}", jobGroup, jobName);
            return false;
        }
        
        // CronTrigger : TriggerKey + cronExpression	// withMisfireHandlingInstructionDoNothing 忽略掉调度终止过程中忽略的调度
        CronScheduleBuilder cronScheduleBuilder = CronScheduleBuilder.cronSchedule(cronExpression).withMisfireHandlingInstructionDoNothing();
		//创建cron定时器
        CronTrigger cronTrigger = TriggerBuilder.newTrigger().withIdentity(triggerKey).withSchedule(cronScheduleBuilder).build();
 
        // JobDetail : jobClass
		//定时执行类
		Class<? extends Job> jobClass_ = RemoteHttpJobBean.class;   // Class.forName(jobInfo.getJobClass());
        
		//创建任务
		JobDetail jobDetail = JobBuilder.newJob(jobClass_).withIdentity(jobKey).build();
        /*if (jobInfo.getJobData()!=null) {
        	JobDataMap jobDataMap = jobDetail.getJobDataMap();
        	jobDataMap.putAll(JacksonUtil.readValue(jobInfo.getJobData(), Map.class));	
        	// JobExecutionContext context.getMergedJobDataMap().get("mailGuid");
		}*/
        
        // schedule : jobDetail + cronTrigger
		//添加定时任务到quartz
        Date date = scheduler.scheduleJob(jobDetail, cronTrigger);
 
        logger.info(">>>>>>>>>>> addJob success, jobDetail:{}, cronTrigger:{}, date:{}", jobDetail, cronTrigger, date);
        return true;
    }

```

#### 2.删除任务

删除任务简单来说就是将任务从quartz中移除，不再执行就好

JobInfoController中提供了删除任务接口



```
@RequestMapping("/remove")
@ResponseBody
public ReturnT<String> remove(int id) {
	return xxlJobService.remove(id);
}

```
在XxlJobService中根据逻辑id删除任务，并且删除数据库中相关持久化数据


```
    @Override
	public ReturnT<String> remove(int id) {
		XxlJobInfo xxlJobInfo = xxlJobInfoDao.loadById(id);
        String group = String.valueOf(xxlJobInfo.getJobGroup());
        String name = String.valueOf(xxlJobInfo.getId());
 
		try {
			//从quartz中移除任务，并将数据库中相关数据删除
			XxlJobDynamicScheduler.removeJob(name, group);
			xxlJobInfoDao.delete(id);
			xxlJobLogDao.delete(id);
			xxlJobLogGlueDao.deleteByJobId(id);
			return ReturnT.SUCCESS;
		} catch (SchedulerException e) {
			logger.error(e.getMessage(), e);
		}
		return ReturnT.FAIL;
	}

```

在XxlJobDynamicScheduler中调用removeJob方法删除相关任务

```
public static boolean removeJob(String jobName, String jobGroup) throws SchedulerException {
    	// TriggerKey : name + group
        TriggerKey triggerKey = TriggerKey.triggerKey(jobName, jobGroup);
        boolean result = false;
        if (checkExists(jobName, jobGroup)) {
	    // 删除任务
            result = scheduler.unscheduleJob(triggerKey);
            logger.info(">>>>>>>>>>> removeJob, triggerKey:{}, result [{}]", triggerKey, result);
        }
        return true;
    }

```

### 3.暂停任务

暂停任务同意调用quartz的接口进行任务暂停处理

JobInfoController中提供接口处理

```
    @RequestMapping("/pause")
	@ResponseBody
	public ReturnT<String> pause(int id) {
		return xxlJobService.pause(id);
	}

```

调用XxlJobService的paus进行任务停止


```
    @Override
	public ReturnT<String> pause(int id) {
        XxlJobInfo xxlJobInfo = xxlJobInfoDao.loadById(id);
        String group = String.valueOf(xxlJobInfo.getJobGroup());
        String name = String.valueOf(xxlJobInfo.getId());
 
		try {
		// 暂停任务
            boolean ret = XxlJobDynamicScheduler.pauseJob(name, group);	// jobStatus do not store
            return ret?ReturnT.SUCCESS:ReturnT.FAIL;
		} catch (SchedulerException e) {
			logger.error(e.getMessage(), e);
			return ReturnT.FAIL;
		}
	}

```

调用XxlJobDynamicScheduler的pauseJob进行任务停止，最终还是执行quartz 的接口完成任务暂停

```
public static boolean pauseJob(String jobName, String jobGroup) throws SchedulerException {
    	// TriggerKey : name + group
    	TriggerKey triggerKey = TriggerKey.triggerKey(jobName, jobGroup);
        
        boolean result = false;
        if (checkExists(jobName, jobGroup)) {
            //任务暂停
            scheduler.pauseTrigger(triggerKey);
            result = true;
            logger.info(">>>>>>>>>>> pauseJob success, triggerKey:{}", triggerKey);
        } else {
        	logger.info(">>>>>>>>>>> pauseJob fail, triggerKey:{}", triggerKey);
        }
        return result;
    }

```

### 4.恢复任务

恢复任务就是将暂停的任务继续重新执行

JobInfoController中提供接口重新执行任务



```
    @RequestMapping("/resume")
	@ResponseBody
	public ReturnT<String> resume(int id) {
		return xxlJobService.resume(id);
	}

```

调用XxlJobService的resume接口继续任务


```
    @Override
	public ReturnT<String> resume(int id) {
        XxlJobInfo xxlJobInfo = xxlJobInfoDao.loadById(id);
        String group = String.valueOf(xxlJobInfo.getJobGroup());
        String name = String.valueOf(xxlJobInfo.getId());
 
		try {
            //继续任务
			boolean ret = XxlJobDynamicScheduler.resumeJob(name, group);
			return ret?ReturnT.SUCCESS:ReturnT.FAIL;
		} catch (SchedulerException e) {
			logger.error(e.getMessage(), e);
			return ReturnT.FAIL;
		}
	}

```

在XllJobDynamicScheduler中调用quartz的接口重新开启任务



```
public static boolean resumeJob(String jobName, String jobGroup) throws SchedulerException {
    	// TriggerKey : name + group
    	TriggerKey triggerKey = TriggerKey.triggerKey(jobName, jobGroup);
        
        boolean result = false;
        if (checkExists(jobName, jobGroup)) {
            //继续任务
            scheduler.resumeTrigger(triggerKey);
            result = true;
            logger.info(">>>>>>>>>>> resumeJob success, triggerKey:{}", triggerKey);
        } else {
        	logger.info(">>>>>>>>>>> resumeJob fail, triggerKey:{}", triggerKey);
        }
        return result;
    }

```
### 5.更新任务

任务更新操作就是更新一下任务相关参数

XxlJobController中提供update接口，完成任务更新操作

```
    @RequestMapping("/update")
	@ResponseBody
	public ReturnT<String> update(XxlJobInfo jobInfo) {
		return xxlJobService.update(jobInfo);
	}

```

在XxlJobService中调用update更新任务，并更新相关数据库中的数据

```
@Override
	public ReturnT<String> update(XxlJobInfo jobInfo) {
 
		//省略部分判断代码
		// stage job info
		XxlJobInfo exists_jobInfo = xxlJobInfoDao.loadById(jobInfo.getId());
		if (exists_jobInfo == null) {
			return new ReturnT<String>(ReturnT.FAIL_CODE, (I18nUtil.getString("jobinfo_field_id")+I18nUtil.getString("system_not_found")) );
		}
		//String old_cron = exists_jobInfo.getJobCron();
 
		exists_jobInfo.setJobCron(jobInfo.getJobCron());
		exists_jobInfo.setJobDesc(jobInfo.getJobDesc());
		exists_jobInfo.setAuthor(jobInfo.getAuthor());
		exists_jobInfo.setAlarmEmail(jobInfo.getAlarmEmail());
		exists_jobInfo.setExecutorRouteStrategy(jobInfo.getExecutorRouteStrategy());
		exists_jobInfo.setExecutorHandler(jobInfo.getExecutorHandler());
		exists_jobInfo.setExecutorParam(jobInfo.getExecutorParam());
		exists_jobInfo.setExecutorBlockStrategy(jobInfo.getExecutorBlockStrategy());
		exists_jobInfo.setExecutorFailStrategy(jobInfo.getExecutorFailStrategy());
		exists_jobInfo.setExecutorTimeout(jobInfo.getExecutorTimeout());
		exists_jobInfo.setChildJobId(jobInfo.getChildJobId());
        xxlJobInfoDao.update(exists_jobInfo);
 
		// fresh quartz
		String qz_group = String.valueOf(exists_jobInfo.getJobGroup());
		String qz_name = String.valueOf(exists_jobInfo.getId());
        try {
            //重新配置定时任务
            boolean ret = XxlJobDynamicScheduler.rescheduleJob(qz_group, qz_name, exists_jobInfo.getJobCron());
            return ret?ReturnT.SUCCESS:ReturnT.FAIL;
        } catch (SchedulerException e) {
            logger.error(e.getMessage(), e);
        }
 
		return ReturnT.FAIL;
	}

```


在XxlJobDynamicScheduler中调用quartz相关方法重新配置定时任务



```
public static boolean rescheduleJob(String jobGroup, String jobName, String cronExpression) throws SchedulerException {
    	
    	// TriggerKey valid if_exists
        if (!checkExists(jobName, jobGroup)) {
        	logger.info(">>>>>>>>>>> rescheduleJob fail, job not exists, JobGroup:{}, JobName:{}", jobGroup, jobName);
            return false;
        }
        
        // TriggerKey : name + group
        TriggerKey triggerKey = TriggerKey.triggerKey(jobName, jobGroup);
        CronTrigger oldTrigger = (CronTrigger) scheduler.getTrigger(triggerKey);
        //任务存在直接更新一下
        if (oldTrigger != null) {
            // avoid repeat
            String oldCron = oldTrigger.getCronExpression();
            if (oldCron.equals(cronExpression)){
                return true;
            }
 
            // CronTrigger : TriggerKey + cronExpression
            CronScheduleBuilder cronScheduleBuilder = CronScheduleBuilder.cronSchedule(cronExpression).withMisfireHandlingInstructionDoNothing();
            oldTrigger = oldTrigger.getTriggerBuilder().withIdentity(triggerKey).withSchedule(cronScheduleBuilder).build();
 
            // rescheduleJob
            scheduler.rescheduleJob(triggerKey, oldTrigger);
        } else {
            //不存在和直接创建新的任务类似
            // CronTrigger : TriggerKey + cronExpression
            CronScheduleBuilder cronScheduleBuilder = CronScheduleBuilder.cronSchedule(cronExpression).withMisfireHandlingInstructionDoNothing();
            CronTrigger cronTrigger = TriggerBuilder.newTrigger().withIdentity(triggerKey).withSchedule(cronScheduleBuilder).build();
 
            // JobDetail-JobDataMap fresh
            JobKey jobKey = new JobKey(jobName, jobGroup);
            JobDetail jobDetail = scheduler.getJobDetail(jobKey);
            /*JobDataMap jobDataMap = jobDetail.getJobDataMap();
            jobDataMap.clear();
            jobDataMap.putAll(JacksonUtil.readValue(jobInfo.getJobData(), Map.class));*/
 
            // Trigger fresh
            HashSet<Trigger> triggerSet = new HashSet<Trigger>();
            triggerSet.add(cronTrigger);
 
            scheduler.scheduleJob(jobDetail, triggerSet, true);
        }
 
        logger.info(">>>>>>>>>>> resumeJob success, JobGroup:{}, JobName:{}", jobGroup, jobName);
        return true;
    }

```

### 6.立即执行任务





