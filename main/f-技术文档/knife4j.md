# Knife4j

## Introduction

Knife4j是为Java MVC框架集成Swagger生成Api文档的增强解决方案,前身是swagger-bootstrap-ui,取名kni4j是希望她能像一把匕首一样小巧,轻量,并且功能强悍!

Knife4j的前身是`swagger-bootstrap-ui`，为了契合微服务的架构发展,由于原来`swagger-bootstrap-ui`采用的是后端Java代码+前端Ui混合打包的方式,在微服务架构下显的很臃肿,因此项目正式更名为`knife4j`

更名后主要专注的方面

- 前后端Java代码以及前端Ui模块进行分离,在微服务架构下使用更加灵活
- 提供专注于Swagger的增强解决方案,不同于只是改善增强前端Ui部分

## Swagger2 注解说明

```java

@Api：用在请求的类上，表示对类的说明
    tags="说明该类的作用，可以在UI界面上看到的注解"
    value="该参数没什么意义，在UI界面上也看到，所以不需要配置"
 
 
@ApiOperation：用在请求的方法上，说明方法的用途、作用
    value="说明方法的用途、作用"
    notes="方法的备注说明"
 
 
@ApiImplicitParams：用在请求的方法上，表示一组参数说明
    @ApiImplicitParam：用在@ApiImplicitParams注解中，指定一个请求参数的各个方面
        name：参数名
        value：参数的汉字说明、解释
        required：参数是否必须传
        paramType：参数放在哪个地方
            · header --> 请求参数的获取：@RequestHeader
            · query --> 请求参数的获取：@RequestParam
            · path（用于restful接口）--> 请求参数的获取：@PathVariable
            · body（不常用）
            · form（不常用）    
        dataType：参数类型，默认String，其它值dataType="Integer"       
        defaultValue：参数的默认值
 
 
@ApiResponses：用在请求的方法上，表示一组响应
    @ApiResponse：用在@ApiResponses中，一般用于表达一个错误的响应信息
        code：数字，例如400
        message：信息，例如"请求参数没填好"
        response：抛出异常的类
 
 
@ApiModel：用于响应类上，表示一个返回响应数据的信息
            （这种一般用在post创建的时候，使用@RequestBody这样的场景，
            请求参数无法使用@ApiImplicitParam注解进行描述的时候）
```

## Spring Cloud Gateway继承Knife4j

以下为官方实例，具体实现上添加了自定义配置。

在我们使用Spring Boot等单体架构集成swagger项目时,是通过对包路径进行业务分组,然后在前端进行不同模块的展示,而在微服务架构下,我们的一个服务就类似于原来我们写的一个业务组

springfox-swagger提供的分组接口是`swagger-resource`,返回的是分组接口名称、地址等信息

在Spring Cloud微服务架构下,我们需要重写该接口,主要是通过网关的注册中心动态发现所有的微服务文档,代码如下：

```java
@Slf4j
@Component
@Primary
@AllArgsConstructor
public class SwaggerResourceConfig implements SwaggerResourcesProvider {

    private final RouteLocator routeLocator;
    private final GatewayProperties gatewayProperties;


    @Override
    public List<SwaggerResource> get() {
        List<SwaggerResource> resources = new ArrayList<>();
        List<String> routes = new ArrayList<>();
        routeLocator.getRoutes().subscribe(route -> routes.add(route.getId()));
        gatewayProperties.getRoutes().stream().filter(routeDefinition -> routes.contains(routeDefinition.getId())).forEach(route -> {
            route.getPredicates().stream()
                    .filter(predicateDefinition -> ("Path").equalsIgnoreCase(predicateDefinition.getName()))
                    .forEach(predicateDefinition -> resources.add(swaggerResource(route.getId(),
                            predicateDefinition.getArgs().get(NameUtils.GENERATED_NAME_PREFIX + "0")
                                    .replace("**", "v2/api-docs"))));
        });

        return resources;
    }

    private SwaggerResource swaggerResource(String name, String location) {
        log.info("name:{},location:{}",name,location);
        SwaggerResource swaggerResource = new SwaggerResource();
        swaggerResource.setName(name);
        swaggerResource.setLocation(location);
        swaggerResource.setSwaggerVersion("2.0");
        return swaggerResource;
    }
}
```

接口：

```java
@RestController
public class SwaggerHandler {

    @Autowired(required = false)
    private SecurityConfiguration securityConfiguration;

    @Autowired(required = false)
    private UiConfiguration uiConfiguration;

    private final SwaggerResourcesProvider swaggerResources;

    @Autowired
    public SwaggerHandler(SwaggerResourcesProvider swaggerResources) {
        this.swaggerResources = swaggerResources;
    }


    @GetMapping("/swagger-resources/configuration/security")
    public Mono<ResponseEntity<SecurityConfiguration>> securityConfiguration() {
        return Mono.just(new ResponseEntity<>(
                Optional.ofNullable(securityConfiguration).orElse(SecurityConfigurationBuilder.builder().build()), HttpStatus.OK));
    }

    @GetMapping("/swagger-resources/configuration/ui")
    public Mono<ResponseEntity<UiConfiguration>> uiConfiguration() {
        return Mono.just(new ResponseEntity<>(
                Optional.ofNullable(uiConfiguration).orElse(UiConfigurationBuilder.builder().build()), HttpStatus.OK));
    }

    @GetMapping("/swagger-resources")
    public Mono<ResponseEntity> swaggerResources() {
        return Mono.just((new ResponseEntity<>(swaggerResources.get(), HttpStatus.OK)));
    }
}
```

## 自定义配置

在Spring Cloud的微服务架构下,每个微服务其实并不需要引入前端的Ui资源,因此在每个微服务的Spring Boot项目下,引入knife4j提供的微服务starter

```xml
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-micro-spring-boot-starter</artifactId>
    <version>${knife4j.version}</version>
</dependency>
```

```java
@Data
@ConfigurationProperties("djj.swagger")
public class SwaggerProperties {
    /**是否开启swagger**/
    private Boolean enable;
    /**标题**/
    private String title = "";
    /**描述**/
    private String description = "";
    /**版本**/
    private String version = "";
    /**许可证**/
    private String license = "";
    /**许可证URL**/
    private String licenseUrl = "";
    /**服务条款URL**/
    private String termsOfServiceUrl = "";
    /**联系人**/
    private Contact contact = new Contact();

    /**swagger会解析的包路径**/
    private String basePackage = "";

    /**swagger会解析的url规则**/
    private List<String> basePath = new ArrayList<>();
    /**在basePath基础上需要排除的url规则**/
    private List<String> excludePath = new ArrayList<>();

    /**分组文档**/
    private Map<String, DocketInfo> docket = new LinkedHashMap<>();

    /**host信息**/
    private String host = "";

    /**全局参数配置**/
    private List<GlobalOperationParameter> globalOperationParameters;

    @Setter
    @Getter
    public static class GlobalOperationParameter{
        /**参数名**/
        private String name;

        /**描述信息**/
        private String description;

        /**指定参数类型**/
        private String modelRef;

        /**参数放在哪个地方:header,query,path,body.form**/
        private String parameterType;

        /**参数是否必须传**/
        private String required;
    }

    @Data
    public static class DocketInfo {
        /**标题**/
        private String title = "";
        /**描述**/
        private String description = "";
        /**版本**/
        private String version = "";
        /**许可证**/
        private String license = "";
        /**许可证URL**/
        private String licenseUrl = "";
        /**服务条款URL**/
        private String termsOfServiceUrl = "";

        private Contact contact = new Contact();

        /**swagger会解析的包路径**/
        private String basePackage = "";

        /**swagger会解析的url规则**/
        private List<String> basePath = new ArrayList<>();
        /**在basePath基础上需要排除的url规则**/
        private List<String> excludePath = new ArrayList<>();

        private List<GlobalOperationParameter> globalOperationParameters;
    }

    @Data
    public static class Contact {
        /**联系人**/
        private String name = "";
        /**联系人url**/
        private String url = "";
        /**联系人email**/
        private String email = "";
    }
}
```

## API文档中心

使用方式：`localhost:8099/doc.html`



## Reference

- **官方文档：**https://doc.xiaominfo.com/knife4j/documentation/
- **备份文档地址**：https://xiaoym.gitee.io/knife4j/
- **效果(2.0版)：**http://knife4j.xiaominfo.com/doc.html
- **效果(旧版本)：**http://swagger-bootstrap-ui.xiaominfo.com/doc.html
- **示例:**https://gitee.com/xiaoym/swagger-bootstrap-ui-demo
- **交流：** https://doc.xiaominfo.com/knife4j/documentation/help.html
- **源码分析**:https://www.xiaominfo.com/2019/05/20/springfox-0/