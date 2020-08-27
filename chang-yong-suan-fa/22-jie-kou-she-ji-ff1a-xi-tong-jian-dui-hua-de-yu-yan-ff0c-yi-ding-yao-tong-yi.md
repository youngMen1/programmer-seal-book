``# 22 | 接口设计：系统间对话的语言，一定要统一

你好，我是朱晔。今天，我要和你分享的主题是，在做接口设计时一定要确保系统之间对话的语言是统一的。

我们知道，开发一个服务的第一步就是设计接口。接口的设计需要考虑的点非常多，比如接口的命名、参数列表、包装结构体、接口粒度、版本策略、幂等性实现、同步异步处理方式等。

这其中，和接口设计相关比较重要的点有三个，分别是包装结构体、版本策略、同步异步处理方式。今天，我就通过我遇到的实际案例，和你一起看看因为接口设计思路和调用方理解不一致所导致的问题，以及相关的实践经验。

## 接口的响应要明确表示接口的处理结果

我曾遇到过一个处理收单的收单中心项目，下单接口返回的响应体中，包含了 success、code、info、message 等属性，以及二级嵌套对象 data 结构体。在对项目进行重构的时候，我们发现真的是无从入手，接口缺少文档，代码一有改动就出错。


有时候，下单操作的响应结果是这样的：success 是 true、message 是 OK，貌似代表下单成功了；但 info 里却提示订单存在风险，code 是一个 5001 的错误码，data 中能看到订单状态是 Cancelled，订单 ID 是 -1，好像又说明没有下单成功。



```

{
  "success": true,
  "code": 5001,
  "info": "Risk order detected",
  "message": "OK",
  "data": {
    "orderStatus": "Cancelled",
    "orderId": -1
  }
}
```

有些时候，这个下单接口又会返回这样的结果：success 是 false，message 提示非法用户 ID，看上去下单失败；但 data 里的 orderStatus 是 Created、info 是空、code 是 0。那么，这次下单到底是成功还是失败呢？



```

{
  "success": false,
  "code": 0,
  "info": "",
  "message": "Illegal userId",
  "data": {
    "orderStatus": "Created",
    "orderId": 0
  }
}
```
这样的结果，让我们非常疑惑：
* 结构体的 code 和 HTTP 响应状态码，是什么关系？
* success 到底代表下单成功还是失败？
* info 和 message 的区别是什么？
*data 中永远都有数据吗？
* 什么时候应该去查询 data？


造成如此混乱的原因是：这个收单服务本身并不真正处理下单操作，只是做一些预校验和预处理；真正的下单操作，需要在收单服务内部调用另一个订单服务来处理；订单服务处理完成后，会返回订单状态和 ID。

在一切正常的情况下，下单后的订单状态就是已创建 Created，订单 ID 是一个大于 0 的数字。而结构体中的 message 和 success，其实是收单服务的处理异常信息和处理成功与否的结果，code、info 是调用订单服务的结果。

对于第一次调用，收单服务自己没问题，success 是 true，message 是 OK，但调用订单服务时却因为订单风险问题被拒绝，所以 code 是 5001，info 是 Risk order detected，data 中的信息是订单服务返回的，所以最终订单状态是 Cancelled。

对于第二次调用，因为用户 ID 非法，所以收单服务在校验了参数后直接就返回了 success 是 false，message 是 Illegal userId。因为请求没有到订单服务，所以 info、code、data 都是默认值，订单状态的默认值是 Created。因此，第二次下单肯定失败了，但订单状态却是已创建。

可以看到，如此混乱的接口定义和实现方式，是无法让调用者分清到底应该怎么处理的。**为了将接口设计得更合理，我们需要考虑如下两个原则：**

* 对外隐藏内部实现。虽然说收单服务调用订单服务进行真正的下单操作，但是直接接口其实是收单服务提供的，收单服务不应该“直接”暴露其背后订单服务的状态码、错误描述。

* 设计接口结构时，明确每个字段的含义，以及客户端的处理方式。

基于这两个原则，我们调整一下返回结构体，去掉外层的 info，即不再把订单服务的调用结果告知客户端：



```

@Data
public class APIResponse<T> {
    private boolean success;
    private T data;
    private int code;
    private String message;
}
```
并明确接口的设计逻辑：

* 如果出现非 200 的 HTTP 响应状态码，就代表请求没有到收单服务，可能是网络出问题、网络超时，或者网络配置的问题。这时，肯定无法拿到服务端的响应体，客户端可以给予友好提示，比如让用户重试，不需要继续解析响应结构体。

* 如果 HTTP 响应码是 200，解析响应体查看 success，为 false 代表下单请求处理失败，可能是因为收单服务参数验证错误，也可能是因为订单服务下单操作失败。这时，根据收单服务定义的错误码表和 code，做不同处理。比如友好提示，或是让用户重新填写相关信息，其中友好提示的文字内容可以从 message 中获取。

* success 为 true 的情况下，才需要继续解析响应体中的 data 结构体。data 结构体代表了业务数据，通常会有下面两种情况。

1.通常情况下，success 为 true 时订单状态是 Created，获取 orderId 属性可以拿到订单号。

2.特殊情况下，比如收单服务内部处理不当，或是订单服务出现了额外的状态，虽然 success 为 true，但订单实际状态不是 Created，这时可以给予友好的错误提示。

cd799f2bdb407bcb9ff5ad452376a6ed.jpg

明确了接口的设计逻辑，我们就是可以实现收单服务的服务端和客户端来模拟这些情况了。首先，实现服务端的逻辑：



```

@GetMapping("server")
public APIResponse<OrderInfo> server(@RequestParam("userId") Long userId) {
    APIResponse<OrderInfo> response = new APIResponse<>();
    if (userId == null) {
        //对于userId为空的情况，收单服务直接处理失败，给予相应的错误码和错误提示
        response.setSuccess(false);
        response.setCode(3001);
        response.setMessage("Illegal userId");
    } else if (userId == 1) {
        //对于userId=1的用户，模拟订单服务对于风险用户的情况
        response.setSuccess(false);
        //把订单服务返回的错误码转换为收单服务错误码
        response.setCode(3002);
        response.setMessage("Internal Error, order is cancelled");
        //同时日志记录内部错误
        log.warn("用户 {} 调用订单服务失败，原因是 Risk order detected", userId);
    } else {
        //其他用户，下单成功
        response.setSuccess(true);
        response.setCode(2000);
        response.setMessage("OK");
        response.setData(new OrderInfo("Created", 2L));
    }
    return response;
}
```
客户端代码，则可以按照流程图上的逻辑来实现，同样模拟三种出错情况和正常下单的情况：

* error==1 的用例模拟一个不存在的 URL，请求无法到收单服务，会得到 404 的 HTTP 状态码，直接进行友好提示，这是第一层处理。


c1ddea0ebf6d86956d68efb0424a6b36.png

* error==2 的用例模拟 userId 参数为空的情况，收单服务会因为缺少 userId 参数提示非法用户。这时，可以把响应体中的 message 展示给用户，这是第二层处理。

f36d21beb95ce0e7ea96dfde96f21847.png

* error==3 的用例模拟 userId 为 1 的情况，因为用户有风险，收单服务调用订单服务出错。处理方式和之前没有任何区别，因为收单服务会屏蔽订单服务的内部错误。

412c64e66a574d8252ac8dd59b4cfe2c.png

但在服务端可以看到如下错误信息：



```

[14:13:13.951] [http-nio-45678-exec-8] [WARN ] [.c.a.d.APIThreeLevelStatusController:36  ] - 用户 1 调用订单服务失败，原因是 Risk order detected
```
error==0 的用例模拟正常用户，下单成功。这时可以解析 data 结构体提取业务结果，作为兜底，需要判断订单状态，如果不是 Created 则给予友好提示，否则查询 orderId 获得下单的订单号，这是第三层处理。
f57ae156de7592de167bd09aaadb8348.png

客户端的实现代码如下：


```

@GetMapping("client")
public String client(@RequestParam(value = "error", defaultValue = "0") int error) {
   String url = Arrays.asList("http://localhost:45678/apiresposne/server?userId=2",
        "http://localhost:45678/apiresposne/server2",
        "http://localhost:45678/apiresposne/server?userId=",
        "http://localhost:45678/apiresposne/server?userId=1").get(error);

    //第一层，先看状态码，如果状态码不是200，不处理响应体
    String response = "";
    try {
        response = Request.Get(url).execute().returnContent().asString();
    } catch (HttpResponseException e) {
        log.warn("请求服务端出现返回非200", e);
        return "服务器忙，请稍后再试！";
    } catch (IOException e) {
        e.printStackTrace();
    }

    //状态码为200的情况下处理响应体
    if (!response.equals("")) {
        try {
            APIResponse<OrderInfo> apiResponse = objectMapper.readValue(response, new TypeReference<APIResponse<OrderInfo>>() {
            });
            //第二层，success是false直接提示用户
            if (!apiResponse.isSuccess()) {
                return String.format("创建订单失败，请稍后再试，错误代码： %s 错误原因：%s", apiResponse.getCode(), apiResponse.getMessage());
            } else {
                //第三层，往下解析OrderInfo
                OrderInfo orderInfo = apiResponse.getData();
                if ("Created".equals(orderInfo.getStatus()))
                    return String.format("创建订单成功，订单号是：%s，状态是：%s", orderInfo.getOrderId(), orderInfo.getStatus());
                else
                    return String.format("创建订单失败，请联系客服处理");
            }
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }
    }
    return "";
}
```

**相比原来混乱的接口定义和处理逻辑，改造后的代码，明确了接口每一个字段的含义，以及对于各种情况服务端的输出和客户端的处理步骤，对齐了客户端和服务端的处理逻辑。**那么现在，你能回答前面那 4 个让人疑惑的问题了吗？


最后分享一个小技巧。为了简化服务端代码，我们可以把包装 API 响应体 APIResponse 的工作交由框架自动完成，这样直接返回 DTO OrderInfo 即可。对于业务逻辑错误，可以抛出一个自定义异常：



```

@GetMapping("server")
public OrderInfo server(@RequestParam("userId") Long userId) {
    if (userId == null) {
        throw new APIException(3001, "Illegal userId");
    }

    if (userId == 1) {
        ...
        //直接抛出异常
        throw new APIException(3002, "Internal Error, order is cancelled");
    }
    //直接返回DTO
    return new OrderInfo("Created", 2L);
}
```
在 APIException 中包含错误码和错误消息：



```

public class APIException extends RuntimeException {
    @Getter
    private int errorCode;
    @Getter
    private String errorMessage;

    public APIException(int errorCode, String errorMessage) {
        super(errorMessage);
        this.errorCode = errorCode;
        this.errorMessage = errorMessage;
    }

    public APIException(Throwable cause, int errorCode, String errorMessage) {
        super(errorMessage, cause);
        this.errorCode = errorCode;
        this.errorMessage = errorMessage;
    }
}
```

然后，定义一个 @RestControllerAdvice 来完成自动包装响应体的工作：

1.通过实现 ResponseBodyAdvice 接口的 beforeBodyWrite 方法，来处理成功请求的响应体转换。
2.实现一个 @ExceptionHandler 来处理业务异常时，APIException 到 APIResponse 的转换。



```

//此段代码只是Demo，生产级应用还需要扩展很多细节
@RestControllerAdvice
@Slf4j
public class APIResponseAdvice implements ResponseBodyAdvice<Object> {

    //自动处理APIException，包装为APIResponse
    @ExceptionHandler(APIException.class)
    public APIResponse handleApiException(HttpServletRequest request, APIException ex) {
        log.error("process url {} failed", request.getRequestURL().toString(), ex);
        APIResponse apiResponse = new APIResponse();
        apiResponse.setSuccess(false);
        apiResponse.setCode(ex.getErrorCode());
        apiResponse.setMessage(ex.getErrorMessage());
        return apiResponse;
    }

    //仅当方法或类没有标记@NoAPIResponse才自动包装
    @Override
    public boolean supports(MethodParameter returnType, Class converterType) {
        return returnType.getParameterType() != APIResponse.class
                && AnnotationUtils.findAnnotation(returnType.getMethod(), NoAPIResponse.class) == null
                && AnnotationUtils.findAnnotation(returnType.getDeclaringClass(), NoAPIResponse.class) == null;
    }

    //自动包装外层APIResposne响应
    @Override
    public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType selectedContentType, Class<? extends HttpMessageConverter<?>> selectedConverterType, ServerHttpRequest request, ServerHttpResponse response) {
        APIResponse apiResponse = new APIResponse();
        apiResponse.setSuccess(true);
        apiResponse.setMessage("OK");
        apiResponse.setCode(2000);
        apiResponse.setData(body);
        return apiResponse;
    }
}
```
在这里，我们实现了一个 @NoAPIResponse 自定义注解。如果某些 @RestController 的接口不希望实现自动包装的话，可以标记这个注解：


```

@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface NoAPIResponse {
}
```

在 ResponseBodyAdvice 的 support 方法中，我们排除了标记有这个注解的方法或类的自动响应体包装。比如，对于刚才我们实现的测试客户端 client 方法不需要包装为 APIResponse，就可以标记上这个注解：


```

@GetMapping("client")
@NoAPIResponse
public String client(@RequestParam(value = "error", defaultValue = "0") int error)
```

这样我们的业务逻辑中就不需要考虑响应体的包装，代码会更简洁。

## 要考虑接口变迁的版本控制策略

接口不可能一成不变，需要根据业务需求不断增加内部逻辑。如果做大的功能调整或重构，涉及参数定义的变化或是参数废弃，导致接口无法向前兼容，这时接口就需要有版本的概念。在考虑接口版本策略设计时，我们需要注意的是，最好一开始就明确版本策略，并考虑在整个服务端统一版本策略。

## 第一，版本策略最好一开始就考虑。

既然接口总是要变迁的，那么最好一开始就确定版本策略。比如，确定是通过 URL Path 实现，是通过 QueryString 实现，还是通过 HTTP 头实现。这三种实现方式的代码如下：



```

//通过URL Path实现版本控制
@GetMapping("/v1/api/user")
public int right1(){
    return 1;
}
//通过QueryString中的version参数实现版本控制
@GetMapping(value = "/api/user", params = "version=2")
public int right2(@RequestParam("version") int version) {
    return 2;
}
//通过请求头中的X-API-VERSION参数实现版本控制
@GetMapping(value = "/api/user", headers = "X-API-VERSION=3")
public int right3(@RequestHeader("X-API-VERSION") int version) {
    return 3;
}
```

这样，客户端就可以在配置中处理相关版本控制的参数，有可能实现版本的动态切换。

这三种方式中，URL Path 的方式最直观也最不容易出错；QueryString 不易携带，不太推荐作为公开 API 的版本策略；HTTP 头的方式比较没有侵入性，如果仅仅是部分接口需要进行版本控制，可以考虑这种方式。

**第二，版本实现方式要统一。**

之前，我就遇到过一个 O2O 项目，需要针对商品、商店和用户实现 REST 接口。虽然大家约定通过 URL Path 方式实现 API 版本控制，但实现方式不统一，有的是 /api/item/v1，有的是 /api/v1/shop，还有的是 /v1/api/merchant：


```

@GetMapping("/api/item/v1")
public void wrong1(){
}


@GetMapping("/api/v1/shop")
public void wrong2(){
}


@GetMapping("/v1/api/merchant")
public void wrong3(){
}
```

显然，商品、商店和商户的接口开发同学，没有按照一致的 URL 格式来实现接口的版本控制。更要命的是，我们可能开发出两个 URL 类似接口，比如一个是 /api/v1/user，另一个是 /api/user/v1，这到底是一个接口还是两个接口呢？相比于在每一个接口的 URL Path 中设置版本号，更理想的方式是在框架层面实现统一。如果你使用 Spring 框架的话，可以按照下面的方式自定义 RequestMappingHandlerMapping 来实现。首先，创建一个注解来定义接口的版本。@APIVersion 自定义注解可以应用于方法或 Controller 上：



```

@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface APIVersion {
    String[] value();
}
```
然后，定义一个 APIVersionHandlerMapping 类继承 RequestMappingHandlerMapping。RequestMappingHandlerMapping 的作用，是根据类或方法上的 @RequestMapping 来生成 RequestMappingInfo 的实例。我们覆盖 registerHandlerMethod 方法的实现，从 @APIVersion 自定义注解中读取版本信息，拼接上原有的、不带版本号的 URL Pattern，构成新的 RequestMappingInfo，来通过注解的方式为接口增加基于 URL 的版本号：


```

public class APIVersionHandlerMapping extends RequestMappingHandlerMapping {
    @Override
    protected boolean isHandler(Class<?> beanType) {
        return AnnotatedElementUtils.hasAnnotation(beanType, Controller.class);
    }


    @Override
    protected void registerHandlerMethod(Object handler, Method method, RequestMappingInfo mapping) {
        Class<?> controllerClass = method.getDeclaringClass();
        //类上的APIVersion注解
        APIVersion apiVersion = AnnotationUtils.findAnnotation(controllerClass, APIVersion.class);
        //方法上的APIVersion注解
        APIVersion methodAnnotation = AnnotationUtils.findAnnotation(method, APIVersion.class);
        //以方法上的注解优先
        if (methodAnnotation != null) {
            apiVersion = methodAnnotation;
        }

        String[] urlPatterns = apiVersion == null ? new String[0] : apiVersion.value();
       
        PatternsRequestCondition apiPattern = new PatternsRequestCondition(urlPatterns);
        PatternsRequestCondition oldPattern = mapping.getPatternsCondition();
        PatternsRequestCondition updatedFinalPattern = apiPattern.combine(oldPattern);
        //重新构建RequestMappingInfo
        mapping = new RequestMappingInfo(mapping.getName(), updatedFinalPattern, mapping.getMethodsCondition(),
                mapping.getParamsCondition(), mapping.getHeadersCondition(), mapping.getConsumesCondition(),
                mapping.getProducesCondition(), mapping.getCustomCondition());
        super.registerHandlerMethod(handler, method, mapping);
    }
}
```
最后，也是特别容易忽略的一点，要通过实现 WebMvcRegistrations 接口，来生效自定义的 APIVersionHandlerMapping：



```

@SpringBootApplication
public class CommonMistakesApplication implements WebMvcRegistrations {
...
    @Override
    public RequestMappingHandlerMapping getRequestMappingHandlerMapping() {
        return new APIVersionHandlerMapping();
    }
}
```
这样，就实现了在 Controller 上或接口方法上通过注解，来实现以统一的 Pattern 进行版本号控制：



```

@GetMapping(value = "/api/user")
@APIVersion("v4")
public int right4() {
    return 4;
}
```

加上注解后，访问浏览器查看效果：

f8fae105eae532e93e329ae2d3253502.png
