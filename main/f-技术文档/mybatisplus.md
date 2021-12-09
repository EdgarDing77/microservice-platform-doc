# MyBatis Plus

## Introduction

「What」[MyBatis-Plus (opens new window)](https://github.com/baomidou/mybatis-plus)（简称 MP）是一个 [MyBatis (opens new window)](https://www.mybatis.org/mybatis-3/)的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生。

![Image](https://cdn.jsdelivr.net/gh/edgarding77/microservice-platform-doc@latest/image/func/mybatis-plus-overview.png)

## 实体表构建

### 注解

- @TableName
- @TableId：主键描述
  - type：AUTO-数据库ID自增 ｜…
- @TableField：字段注解
  - value：数据库字段名
  - exist：是否为数据库表字段（默认true）
  - fill：FieldFill.DEAFULT-字段自动填充策略有四种
    -  DEAFULT：默认不处理
    - INSERT：插入时填充字段
    - UPDATE：更新时填充字段
    - INSERT_UPDATE：插入和更新时填充字段
- @TableLogic：表字段逻辑处理注解（逻辑删除）
- @Version：乐观锁注解



## ORM层实现

### 自动填充功能

**原理：**

- 通过实现`MetaObjectHandler`接口：com.baomidou.mybatisplus.core.handlers.MetaObjectHandler
- 注解填充字段 `@TableField(.. fill = FieldFill.INSERT)` 生成器策略部分也可以配置！

```java
public class User {
    // 注意！这里需要标记为填充字段
    @TableField(.. fill = FieldFill.INSERT)
    private String fillField;
    ....
}
```

自定义实现类`MyMetaObjectHandler`：

```java
@Slf4j
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {

    @Override
    public void insertFill(MetaObject metaObject) {
        log.info("start insert fill ....");
        this.strictInsertFill(metaObject, "createTime", LocalDateTime.class, LocalDateTime.now()); // 起始版本 3.3.0(推荐使用)
        // 或者
        this.strictInsertFill(metaObject, "createTime", () -> LocalDateTime.now(), LocalDateTime.class); // 起始版本 3.3.3(推荐)
        // 或者
        this.fillStrategy(metaObject, "createTime", LocalDateTime.now()); // 也可以使用(3.3.0 该方法有bug)
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        log.info("start update fill ....");
        this.strictUpdateFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now()); // 起始版本 3.3.0(推荐)
        // 或者
        this.strictUpdateFill(metaObject, "updateTime", () -> LocalDateTime.now(), LocalDateTime.class); // 起始版本 3.3.3(推荐)
        // 或者
        this.fillStrategy(metaObject, "updateTime", LocalDateTime.now()); // 也可以使用(3.3.0 该方法有bug)
    }
}
```

注意事项：

- 填充原理是直接给`entity`的属性设置值!!!
- 注解则是指定该属性在对应情况下必有值,如果无值则入库会是`null`
- `MetaObjectHandler`提供的默认方法的策略均为:如果属性有值则不覆盖,如果填充值为`null`则不填充
- 字段必须声明`TableField`注解,属性`fill`选择对应策略,该声明告知`Mybatis-Plus`需要预留注入`SQL`字段
- 填充处理器`MyMetaObjectHandler`在 Spring Boot 中需要声明`@Component`或`@Bean`注入
- 要想根据注解`FieldFill.xxx`和`字段名`以及`字段类型`来区分必须使用父类的`strictInsertFill`或者`strictUpdateFill`方法
- 不需要根据任何来区分可以使用父类的`fillStrategy`方法

**具体实现：**

```java
    /**
     * 插入填充，字段为空自动填充
     */
    @Override
    public void insertFill(MetaObject metaObject) {
        this.strictInsertFill(metaObject, CommonConstants.CREATE_TIME
            , LocalDateTime::now, LocalDateTime.class);
        this.strictInsertFill(metaObject, CommonConstants.UPDATE_TIME
            , LocalDateTime::now, LocalDateTime.class);
    }

    /**
     * 更新填充
     */
    @Override
    public void updateFill(MetaObject metaObject) {
        this.strictInsertFill(metaObject, CommonConstants.UPDATE_TIME
            , LocalDateTime::now, LocalDateTime.class);   
    }
```

### 分页插件实现

因为实现了多租户，需要在配置的时候需要填多租户`TenantLineInnerInterceptor`先，具体实现如下：

```java
@EnableConfigurationProperties(MyBatisPlusAutoFillProperties.class)
public class MyBatisPlusAutoConfigure {
    @Autowired
    private TenantLineHandler tenantLineHandler;
    @Autowired
    private MyBatisPlusAutoFillProperties autoFillProperties;
    @Autowired
    private TenantProperties tenantProperties;

    /**
     * 自动注入开启
     * @return
     */
    @Bean
    @ConditionalOnMissingBean
    @ConditionalOnProperty(prefix = "djj.mybatis-plus.auto-fill", name = "enable", havingValue = "true", matchIfMissing = true)
    public MetaObjectHandler metaObjectHandler() {
        return new DateMetaObjectHandler(autoFillProperties);
    }

    /**
     * 分页插件，自动识别数据库类型
     * @return
     */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        if(tenantProperties.getEnable()) {
            interceptor.addInnerInterceptor(new TenantLineInnerInterceptor(tenantLineHandler));
        }
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor());
        return interceptor;
    }

    @Bean
    public ConfigurationCustomizer configurationCustomizer() {
        return configuration -> configuration.setUseDeprecatedExecutor(false);
    }
}

```

### 不需要隔离的语句

通过使用拦截忽略注解 `@InterceptorIgnore`。

|      属性名      |  类型  | 默认值 |                  描述                  |
| :--------------: | :----: | :----: | :------------------------------------: |
|    tenantLine    | String |   ""   |                行级租户                |
| dynamicTableName | String |   ""   |                动态表名                |
|   blockAttack    | String |   ""   | 攻击 SQL 阻断解析器,防止全表更新与删除 |
|    illegalSql    | String |   ""   |              垃圾SQL拦截               |

> 该注解作用于 xxMapper.java 方法之上 各属性代表对应的插件 各属性不给值则默认为 false 设置为 true 忽略拦截 更多说明详见源码注释

## 插件

### **InnerInterceptor**

我们提供的插件都将基于此接口来实现功能

目前已有的功能:

- 自动分页: PaginationInnerInterceptor
- 多租户: TenantLineInnerInterceptor
- 动态表名: DynamicTableNameInnerInterceptor
- 乐观锁: OptimisticLockerInnerInterceptor
- sql性能规范: IllegalSQLInnerInterceptor
- 防止全表更新与删除: BlockAttackInnerInterceptor

注意:

使用多个功能需要注意顺序关系,建议使用如下顺序

- 多租户,动态表名
- 分页,乐观锁
- sql性能规范,防止全表更新与删除

总结: 对sql进行单次改造的优先放入,不对sql进行改造的最后放入

### 分页插件

分页插件配置：

```java
@Configuration
@MapperScan("scan.your.mapper.package")
public class MybatisPlusConfig {

    /**
     * 新的分页插件,一缓和二缓遵循mybatis的规则,需要设置 MybatisConfiguration#useDeprecatedExecutor = false 避免缓存出现问题(该属性会在旧插件移除后一同移除)
     */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.H2));
        return interceptor;
    }

    @Bean
    public ConfigurationCustomizer configurationCustomizer() {
        return configuration -> configuration.setUseDeprecatedExecutor(false);
    }
}
```

### 多租户

```java
public interface TenantLineHandler {

    /**
     * 获取租户 ID 值表达式，只支持单个 ID 值
     * <p>
     *
     * @return 租户 ID 值表达式
     */
    Expression getTenantId();

    /**
     * 获取租户字段名
     * <p>
     * 默认字段名叫: tenant_id
     *
     * @return 租户字段名
     */
    default String getTenantIdColumn() {
        return "tenant_id";
    }

    /**
     * 根据表名判断是否忽略拼接多租户条件
     * <p>
     * 默认都要进行解析并拼接多租户条件
     *
     * @param tableName 表名
     * @return 是否忽略, true:表示忽略，false:需要解析并拼接多租户条件
     */
    default boolean ignoreTable(String tableName) {
        return false;
    }
}
```

结合分页的实现参考：

```java
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new TenantLineInnerInterceptor(new TenantLineHandler() {
            @Override
            public Expression getTenantId() {
                return new LongValue(1);
            }

            // 这是 default 方法,默认返回 false 表示所有表都需要拼多租户条件
            @Override
            public boolean ignoreTable(String tableName) {
                return !"user".equalsIgnoreCase(tableName);
            }
        }));
        // 如果用了分页插件注意先 add TenantLineInnerInterceptor 再 add PaginationInnerInterceptor
        // 用了分页插件必须设置 MybatisConfiguration#useDeprecatedExecutor = false
//        interceptor.addInnerInterceptor(new PaginationInnerInterceptor());
        return interceptor;
    }
```

## CRUD接口

### Service CRUD接口

> 说明:
>
> - 通用 Service CRUD 封装[IService (opens new window)](https://gitee.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-extension/src/main/java/com/baomidou/mybatisplus/extension/service/IService.java)接口，进一步封装 CRUD 采用 `get 查询单行` `remove 删除` `list 查询集合` `page 分页` 前缀命名方式区分 `Mapper` 层避免混淆，
> - 泛型 `T` 为任意实体对象
> - 建议如果存在自定义通用 Service 方法的可能，请创建自己的 `IBaseService` 继承 `Mybatis-Plus` 提供的基类
> - 对象 `Wrapper` 为 [条件构造器](https://baomidou.com/guide/wrapper.html)

#### save

```java
// 插入一条记录（选择字段，策略插入）
boolean save(T entity);
// 插入（批量）
boolean saveBatch(Collection<T> entityList);
// 插入（批量）
boolean saveBatch(Collection<T> entityList, int batchSize);
```

#### saveOrUpdate

```java
// TableId 注解存在更新记录，否插入一条记录
boolean saveOrUpdate(T entity);
// 根据updateWrapper尝试更新，否继续执行saveOrUpdate(T)方法
boolean saveOrUpdate(T entity, Wrapper<T> updateWrapper);
// 批量修改插入
boolean saveOrUpdateBatch(Collection<T> entityList);
// 批量修改插入
boolean saveOrUpdateBatch(Collection<T> entityList, int batchSize);
```

**参数说明**

|     类型      |    参数名     |               描述               |
| :-----------: | :-----------: | :------------------------------: |
|       T       |    entity     |             实体对象             |
|  Wrapper<T>   | updateWrapper | 实体对象封装操作类 UpdateWrapper |
| Collection<T> |  entityList   |           实体对象集合           |
|      int      |   batchSize   |           插入批次数量           |



`

### 条件构造器

说明:

- 以下出现的第一个入参`boolean condition`表示该条件**是否**加入最后生成的sql中，例如：`query.like(StringUtils.isNotBlank(name), Entity::getName, name) .eq(age!=null && age >= 0, Entity::getAge, age)`
- 以下代码块内的多个方法均为从上往下补全个别`boolean`类型的入参,默认为`true`
- 以下出现的泛型`Param`均为`Wrapper`的子类实例(均具有`AbstractWrapper`的所有方法)
- 以下方法在入参中出现的`R`为泛型,在普通wrapper中是`String`,在LambdaWrapper中是**函数**(例:`Entity::getId`,`Entity`为实体类,`getId`为字段`id`的**getMethod**)
- 以下方法入参中的`R column`均表示数据库字段,当`R`具体类型为`String`时则为数据库字段名(**字段名是数据库关键字的自己用转义符包裹!**)!而不是实体类数据字段名!!!,另当`R`具体类型为`SFunction`时项目runtime不支持eclipse自家的编译器!!!
- 以下举例均为使用普通wrapper,入参为`Map`和`List`的均以`json`形式表现!
- 使用中如果入参的`Map`或者`List`为**空**,则不会加入最后生成的sql中!!!
- 有任何疑问就点开源码看,看不懂**函数**的[点击我学习新知识(opens new window)](https://www.jianshu.com/p/613a6118e2e0)

警告:

不支持以及不赞成在 RPC 调用中把 Wrapper 进行传输

1. wrapper 很重
2. 传输 wrapper 可以类比为你的 controller 用 map 接收值(开发一时爽,维护火葬场)
3. 正确的 RPC 调用姿势是写一个 DTO 进行传输,被调用方再根据 DTO 执行相应的操作
4. 我们拒绝接受任何关于 RPC 传输 Wrapper 报错相关的 issue 甚至 pr



## Reference

- 官方文档：https://baomidou.com/guide/
- 多租户：https://baomidou.com/guide/interceptor-tenant-line.html#tenantlineinnerinterceptor
- 官方实现demo：https://github1s.com/baomidou/mybatis-plus-samples/blob/master/mybatis-plus-sample-tenant/src/main/java/com/baomidou/mybatisplus/samples/tenant/config/MybatisPlusConfig.java