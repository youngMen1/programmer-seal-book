# 1.XXL-JOB原理--执行器注册（二）

## 1.1.基本介绍

### 1.1.1.xxl-job添加执行器到任务调度中心有两种方式

* （1）客户端执行器自动将名称和机器地址注册到任务调度中心

* （2）可以在任务调度中心手动录入执行器名称和相关的机器地址（多个机器地址用逗号隔开）

![](/static/image/20180914205517173.png)

### 1.1.2.自动注册流程

1.在执行器客户端配置执行器名称和任务调度中心地址：

```
### 调度中心部署跟地址 [选填]：如调度中心集群部署存在多个地址则用逗号分隔。执行器将会使用该地址进行"执行器心跳注册"和"任务结果回调"；为空则关闭自动注册；
xxl:
  job:
    admin:
      addresses: http://192.168.1.28:8080/xxl-job-admin
### 执行器通讯TOKEN [选填]：非空时启用；
    accessToken: ''
### 执行器AppName [选填]：执行器心跳注册分组依据；为空则关闭自动注册
    executor:
      appname: xxl-job-order-service
### 执行器IP [选填]：默认为空表示自动获取IP，多网卡时可手动设置指定IP，该IP不会绑定Host仅作为通讯实用；地址信息用于 "执行器注册" 和 "调度中心请求并触发任务"；
      ip: ''
### 执行器端口号 [选填]：小于等于0则自动获取；默认端口为9999，单机部署多个执行器时，注意要配置不同执行器端口；
      port: 16004
### 执行器运行日志文件存储磁盘路径 [选填] ：需要对该路径拥有读写权限；为空则使用默认路径；
      logpath: /data/applogs/xxl-job/jobhandler
### 执行器日志文件保存天数 [选填] ： 过期日志自动清理, 限制值大于等于3时生效; 否则, 如-1, 关闭自动清理功能；
      logretentiondays: 20
```

2.相关注册执行代码

在执行器启动时会读取配置，当存在任务调度中心地址会依次向任务调度中心注册其地址

XxlJobExecutor类在进行初始化时会进行如下操作。

```
// 当存在多个任务调度中心时，创建代理类并注册，在NetComClientProxy
private static void initAdminBizList(String adminAddresses, String accessToken) throws Exception {
        if (adminAddresses!=null && adminAddresses.trim().length()>0) {
            for (String address: adminAddresses.trim().split(",")) {
                if (address!=null && address.trim().length()>0) {
                    String addressUrl = address.concat(AdminBiz.MAPPING);
                    AdminBiz adminBiz = (AdminBiz) new NetComClientProxy(AdminBiz.class, addressUrl, accessToken).getObject();
                    if (adminBizList == null) {
                        adminBizList = new ArrayList<AdminBiz>();
                    }
                    adminBizList.add(adminBiz);
                }
            }
        }
    }
```

在XxlJobExecutor被调用时执行getObject方法，完成向任务调度中心发送请求进行服务注册操作。

```
@Override
    public Object getObject() throws Exception {
        return Proxy.newProxyInstance(Thread.currentThread()
                .getContextClassLoader(), new Class[] { iface },
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

                        // filter method like "Object.toString()"
                        if (Object.class.getName().equals(method.getDeclaringClass().getName())) {
                            logger.error(">>>>>>>>>>> xxl-rpc proxy class-method not support [{}.{}]", method.getDeclaringClass().getName(), method.getName());
                            throw new RuntimeException("xxl-rpc proxy class-method not support");
                        }

                        // request
                        RpcRequest request = new RpcRequest();
                        request.setServerAddress(serverAddress);
                        request.setCreateMillisTime(System.currentTimeMillis());
                        request.setAccessToken(accessToken);
                        request.setClassName(method.getDeclaringClass().getName());
                        request.setMethodName(method.getName());
                        request.setParameterTypes(method.getParameterTypes());
                        request.setParameters(args);

                        // send
                        // 向任务调度中心发送请求进行服务注册
                        RpcResponse response = client.send(request);

                        // valid response
                        if (response == null) {
                            throw new Exception("Network request fail, response not found.");
                        }
                        if (response.isError()) {
                            throw new RuntimeException(response.getError());
                        } else {
                            return response.getResult();
                        }

                    }
                });
```

在JettyClient中调用send方法完成服务注册操作

```
public RpcResponse send(RpcRequest request) throws Exception {
        try {
            // serialize request
            byte[] requestBytes = HessianSerializer.serialize(request);

            // reqURL
            String reqURL = request.getServerAddress();
            if (reqURL!=null && reqURL.toLowerCase().indexOf("http")==-1) {
                reqURL = "http://" + request.getServerAddress() + "/";    // IP:PORT, need parse to url
            }
            // 发送post请求进行服务注册，简单注册一下IP和端口信息等
            // remote invoke
            byte[] responseBytes = HttpClientUtil.postRequest(reqURL, requestBytes);
            if (responseBytes == null || responseBytes.length==0) {
                RpcResponse rpcResponse = new RpcResponse();
                rpcResponse.setError("Network request fail, RpcResponse byte[] is null");
                return rpcResponse;
            }

            // deserialize response
            RpcResponse rpcResponse = (RpcResponse) HessianSerializer.deserialize(responseBytes, RpcResponse.class);
            return rpcResponse;
        } catch (Exception e) {
            logger.error(e.getMessage(), e);

            RpcResponse rpcResponse = new RpcResponse();
            rpcResponse.setError("Network request error: " + e.getMessage());
            return rpcResponse;
        }
    }
```

# 2.总结

其实执行器注册到任务调度的信息非常简单，可以就简单的认为为应用的一些基本信息，IP、端口和应用名称等等，并不用将具体的任务类等信息注册到任务调度中心，所以任务调度中心无法感知执行器一些具体信息，只能需要靠运维和技术人员在任务调度中心进行选择配置，否则可能会无法正常执行任务。

