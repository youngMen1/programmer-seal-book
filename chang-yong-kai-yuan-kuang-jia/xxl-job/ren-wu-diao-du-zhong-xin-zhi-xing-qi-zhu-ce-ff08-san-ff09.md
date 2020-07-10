# 1.XXL-JOB原理--任务调度中心执行器注册（三）
## 1.1.基本介绍   
在上一篇博客XXL-JOB学习--执行器注册（一）中我们介绍了xxl-job执行器注册到任务调度中心的流程及相关注册信息，接下来我们看看任务调度中心接受任务注册后做了哪些事情。
### 1.1.1.注册地址
地址：http://127.0.0.1:8080/api
任务调度中心对外提供注册地址/api用来接受任务执行器注册的相关服务器信息

1.xxl-job admin通过JobApiController来对外提供/api接口


```
@Controller
public class JobApiController {
    private static Logger logger = LoggerFactory.getLogger(JobApiController.class);
 
    private RpcResponse doInvoke(HttpServletRequest request) {
        try {
            // deserialize request
            byte[] requestBytes = HttpClientUtil.readBytes(request);
            if (requestBytes == null || requestBytes.length==0) {
                RpcResponse rpcResponse = new RpcResponse();
                rpcResponse.setError("RpcRequest byte[] is null");
                return rpcResponse;
            }
			//反序列化数据
            RpcRequest rpcRequest = (RpcRequest) HessianSerializer.deserialize(requestBytes, RpcRequest.class);
 
            // invoke
			//调用服务注册方法
            RpcResponse rpcResponse = NetComServerFactory.invokeService(rpcRequest, null);
            return rpcResponse;
        } catch (Exception e) {
            logger.error(e.getMessage(), e);
 
            RpcResponse rpcResponse = new RpcResponse();
            rpcResponse.setError("Server-error:" + e.getMessage());
            return rpcResponse;
        }
    }
 
	//对外提供api接口
    @RequestMapping(AdminBiz.MAPPING)
    @PermessionLimit(limit=false)
    public void api(HttpServletRequest request, HttpServletResponse response) throws IOException {
 
        // invoke
        RpcResponse rpcResponse = doInvoke(request);
 
        // serialize response
        byte[] responseBytes = HessianSerializer.serialize(rpcResponse);
 
        response.setContentType("text/html;charset=utf-8");
        response.setStatus(HttpServletResponse.SC_OK);
        //baseRequest.setHandled(true);
 
        OutputStream out = response.getOutputStream();
        out.write(responseBytes);
        out.flush();
    }
 
}

```
2.在NetComServerFactory类中调用invokeService方法，根据反射调用AdminBiz接口的实现类AdminBizImpl的register方法完成服务注册操作


```
public static RpcResponse invokeService(RpcRequest request, Object serviceBean) {
		if (serviceBean==null) {
			serviceBean = serviceMap.get(request.getClassName());
		}
		if (serviceBean == null) {
			// TODO
		}
 
		RpcResponse response = new RpcResponse();
 
		if (System.currentTimeMillis() - request.getCreateMillisTime() > 180000) {
			response.setResult(new ReturnT<String>(ReturnT.FAIL_CODE, "The timestamp difference between admin and executor exceeds the limit."));
			return response;
		}
		if (accessToken!=null && accessToken.trim().length()>0 && !accessToken.trim().equals(request.getAccessToken())) {
			response.setResult(new ReturnT<String>(ReturnT.FAIL_CODE, "The access token[" + request.getAccessToken() + "] is wrong."));
			return response;
		}
 
		try {
			//接口AdminBiz的实现类AdminBizImpl
			Class<?> serviceClass = serviceBean.getClass();
			//AdminBiz的register方法
			String methodName = request.getMethodName();
			Class<?>[] parameterTypes = request.getParameterTypes();
			Object[] parameters = request.getParameters();
 
			FastClass serviceFastClass = FastClass.create(serviceClass);
			//调用AdminBizImpl的register方法
			FastMethod serviceFastMethod = serviceFastClass.getMethod(methodName, parameterTypes);
 
			Object result = serviceFastMethod.invoke(serviceBean, parameters);
 
			response.setResult(result);
		} catch (Throwable t) {
			t.printStackTrace();
			response.setError(t.getMessage());
		}
 
		return response;
	}

```
调用的类：
20180915141957718.png
参数：基本的服务器信息
20180915142414402.png


