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
1. transient 关键字
   作用范围：Java原生序列化
   功能：标记字段不参与序列化过程
   使用方式：
```java
  public class Example implements Serializable {
      private String serializedField;
      private transient String nonSerializedField; // 不会被序列化
  }
  
```
2.@Transient 注解
作用范围：JPA/Hibernate 持久化操作
功能：标记字段不持久化到数据库
注意：不影响序列化操作，只是告诉ORM框架不要将该字段映射到数据库
使用方式：
```java
  public class User {
      private String name;
      
      @Transient
      private String temporaryData; // 不存储到数据库，但仍可能被序列化
  }
  
```
3. @JsonIgnore 注解
   作用范围：Jackson序列化框架
   功能：标记字段在JSON序列化/反序列化过程中被忽略
   使用方式：
```java
  public class User {
    private String name;

    @JsonIgnore
    private String password; // 在JSON序列化时不包含此字段
}

```
主要区别总结

| 特性 | transient | @Transient | JsonIgnore |
|----|-----------| -------- |------------|
| 影响Java原生序列化 | ✅ 是       | ❌ 否 | ❌ 否        |
| 影响JPA/Hibernate 持久化操作 | ❌ 否       | ✅ 是 | ❌ 否        |
| 影响Jackson序列化框架 | ❌ 否(除非配置) | ❌ 否 | ✅ 是        |
| 作用域 | 字段级别 | 字段/方法级 | 字段/方法级     |

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