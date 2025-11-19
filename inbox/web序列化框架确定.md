# 不同序列化框架的工作原理
1. Java原生序列化机制
```java
public class JavaNativeSerialization {
    // Java原生序列化过程：
    // 1. 通过反射直接访问所有字段（包括private）
    // 2. 忽略transient关键字标记的字段
    // 3. 完全忽略getter/setter方法
}
```
2. Jackson序列化机制

Jackson默认使用getter方法，但可以配置：
```java
public class JacksonSerialization {
    // Jackson的三种工作模式：
}
```
模式A：基于getter方法（默认）
```java
public class ObjectWithGetters {
    private String value = "test";
    
    // Jackson会调用此方法获取属性值
    public String getValue() { return value; }
    
    // 这个方法不会被Jackson识别为属性getter
    public String retrieveValue() { return value; }
}
```
模式B：基于字段（需要配置）
```java
 // 配置Jackson使用字段
ObjectMapper mapper = new ObjectMapper();
mapper.setVisibility(PropertyAccessor.FIELD, Visibility.ANY);
mapper.setVisibility(PropertyAccessor.GETTER, Visibility.NONE);
public class ObjectWithFields {
    public String publicField = "public";      // 会被序列化
    private String privateField = "private";   // 会被序列化（通过反射）
}
```
模式C：自动检测（默认行为）
```java
// Jackson默认行为：同时检查getter和字段
// 优先级：getter > 字段
public class MixedObject {
    private String name = "fieldValue";
    
    // 这个getter会覆盖字段的序列化
    public String getName() { 
        return "getterValue"; // 序列化输出：getterValue
    }
}

```
3. 不同RPC框架的序列化机制
Dubbo + Hessian、Spring Cloud Feign + Jackson、gRPC + Protobuf

# 序列化相关注解、关键字
@Transient 、@JsonIgnore 、transient


# 获取web支持的序列化框架版本
```java
@RestController
public class UserController {
    
    /** 可以注入处理适配器进行*/
    @Autowired
    private RequestMappingHandlerAdapter handlerAdapter;
    
    @PostMapping("queryPlatformInfo")
    public QueryResultModel<List<AuthUser>> queryPlatformInfo(
            @RequestBody Page page, 
            HttpServletRequest request) throws ServiceException {
        
        // 获取当前请求的序列化信息
        List<HttpMessageConverter<?>> converters = handlerAdapter.getMessageConverters();
        
        for (HttpMessageConverter<?> converter : converters) {
            if (converter.canRead(Page.class, request.getContentType())) {
                System.out.println("使用的序列化器: " + converter.getClass().getSimpleName());
                
                if (converter instanceof MappingJackson2HttpMessageConverter) {
                    MappingJackson2HttpMessageConverter jacksonConverter = 
                        (MappingJackson2HttpMessageConverter) converter;
                    ObjectMapper mapper = jacksonConverter.getObjectMapper();
                    
                    // 获取Jackson版本
                    String jacksonVersion = mapper.version().toString();
                    System.out.println("Jackson版本: " + jacksonVersion);
                }
            }
        }
        
        // 业务逻辑...
    }
}
```