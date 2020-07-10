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


 
 
 