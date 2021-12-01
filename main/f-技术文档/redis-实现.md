# Redis-实现

[toc]

## 实现说明

> 针对djj-redis-spring-boot-starter通用逻辑操作的说明。

### 目录结构

```bash
├── pom.xml
├── src
│   └── main
│       ├── java
│       │   └── top
│       │       └── edgarding
│       │           └── infrastructure
│       │               └── redis
│       │                   ├── RedisAutoConfigure.java
│       │                   ├── constant
│       │                   │   └── RedisToolsConstant.java
│       │                   ├── lock
│       │                   │   └── RedissonDistributedLock.java
│       │                   ├── properties
│       │                   │   └── CacheManagerProperties.java
│       │                   └── template
│       │                       └── RedisRepository.java
│       └── resources
│           └── META-INF
│               └── spring.factories

```

- RedisToolsConstant.java
  - 主要针对集群和单机的配置常量：SINGLE（1）和CLUSTER（2）
- RedissonDistributedLock.java
  - 分布式锁实现，基本锁功能的抽象
- CacheManagerProperties.java
  - 统一管理的缓存配置
- RedisRepository.java
  - Redis操作方法

### 具体实现

#### cacheManager配置扩展

CacheManagerProperties.java

```java
@Setter
@Getter
@ConfigurationProperties(prefix = "djj.cache-manager")
public class CacheManagerProperties {
    private List<CacheConfig> configs;

    @Setter
    @Getter
    public static class CacheConfig {
        /**
         * cache key
         */
        private String key;
        /**
         * 过期时间，second
         */
        private long second = 60;
    }
}
```

```yml
djj:
  cache-manager:
    configs:
      - key: menu
        second: 300
      - key: user
        second: 1800
```

具体实现：

```java
    @Bean(name = "cacheManager")
    @Primary
    public CacheManager cacheManager(RedisConnectionFactory redisConnectionFactory
        , RedisSerializer<String> redisKeySerializer, RedisSerializer<Object> redisValueSerializer) {
        RedisCacheConfiguration
            difConf = getDefConf(redisKeySerializer, redisValueSerializer).entryTtl(Duration.ofHours(1));

        //自定义的缓存过期时间配置
        int configSize = cacheManagerProperties.getConfigs() == null ? 0 : cacheManagerProperties.getConfigs().size();
        Map<String, RedisCacheConfiguration> redisCacheConfigurationMap = new HashMap<>(configSize);
        if (configSize > 0) {
            cacheManagerProperties.getConfigs().forEach(e -> {
                RedisCacheConfiguration conf = getDefConf(redisKeySerializer, redisValueSerializer).entryTtl(Duration.ofSeconds(e.getSecond()));
                redisCacheConfigurationMap.put(e.getKey(), conf);
            });
        }

        return RedisCacheManager.builder(redisConnectionFactory)
            .cacheDefaults(difConf)
            .withInitialCacheConfigurations(redisCacheConfigurationMap)
            .build();
    }
```

#### Redis锁

```Java
@ConditionalOnClass(RedissonClient.class)
@ConditionalOnProperty(prefix = "djj.lock", name = "type", havingValue = "REDIS", matchIfMissing = true)
```

来实现Redis锁的选择，只有该工程引入RedissonClient，和配置文件有匹配的环境变量：在配置的时候

```YAML
djj:
  lock:
    type: redis
```

缓存Client的redis key

```java
SecurityConstants.CACHE_CLIENT_KEY + ":" + clientId
oauth_client_details:clientId
```

读写策略： 先从数据库读取

## Reference