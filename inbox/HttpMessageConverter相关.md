    @Autowired
    private HttpMessageConverters mvcConverters;

@SpringBootApplication(scanBasePackages = {"top.ibase4j", "com.lesso"})
@ComponentScan(excludeFilters = @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = {com.lesso.util.BusinessLicenseOCRUtil.class,com.lesso.util.NacosValueUtil.class,com.lesso.util.SendSmsUtil.class,com.lesso.util.CardUtil.class}))

同时使用了 @SpringBootApplication(scanBasePackages = ...) 和 @ComponentScan。当单独使用 @ComponentScan 时，如果没有指定 basePackages，它会覆盖 @SpringBootApplication 的扫描配置，默认只扫描当前类所在包（com.lesso），导致 top.ibase4j 未被扫描。