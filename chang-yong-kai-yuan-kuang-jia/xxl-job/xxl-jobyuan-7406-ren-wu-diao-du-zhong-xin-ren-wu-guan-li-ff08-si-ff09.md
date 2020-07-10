# 1.XXL-JOB原理--任务调度中心任务管理（四）

## 1.1.基本介绍

 在任务调度中心可以进行新建任务，新建任务之后可以在任务列表中查看相关任务，任务可以根据我们配置的cron表达式进行任务调度，或者也可以在任务列表中执行、暂停、删除和查看相关运行日志等操作。
 
 ### 1.1.1.任务调度中心管理
 
 1.新建任务
 ![](/static/image/2018091514365420.png)
 
 2.任务列表任务操作
 ![](/static/image/20180915143942763.png)
 
 
 
 ## 1.2.任务创建与操作
 
 我们了解到xxl-job是基于quartz来实现定时任务的(其实任务调度中心任务执行基于quartz，任务执行器就是一个被调用执行的服务)
 ### 1.2.1.任务创建
 
调用JobInfoController的add方法添加任务



```
       @RequestMapping("/add")
	@ResponseBody
	public ReturnT<String> add(XxlJobInfo jobInfo) {
		//添加定时器
		return xxlJobService.add(jobInfo);
	}

```
调用XxlJobService的add方法，新建quartz任务


```
@Override
	public ReturnT<String> add(XxlJobInfo jobInfo) {
		// valid
		//获取任务分组
		XxlJobGroup group = xxlJobGroupDao.load(jobInfo.getJobGroup());
		
		//省略部分任务相关信息检查的代码
 
		// add in db
		//将任务持久化到数据库的xxl_job_qrtz_trigger_info
		xxlJobInfoDao.save(jobInfo);
		if (jobInfo.getId() < 1) {
			return new ReturnT<String>(ReturnT.FAIL_CODE, (I18nUtil.getString("jobinfo_field_add")+I18nUtil.getString("system_fail")) );
		}
 
		// add in quartz
        String qz_group = String.valueOf(jobInfo.getJobGroup());
        String qz_name = String.valueOf(jobInfo.getId());
        try {
			//添加任务到quartz中
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

 
 
 