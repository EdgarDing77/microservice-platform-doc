# Transmittable-Threadlocal

## Introdcution

> **TransmittableThreadLocal** 是Alibaba开源的、用于解决 **“在使用线程池等会缓存线程的组件情况下传递ThreadLocal”** 问题的 InheritableThreadLocal 扩展。若希望 TransmittableThreadLocal 在线程池与主线程间传递，需配合 **TtlRunnable** 和 **TtlCallable** 使用。

「What」TransmittableThreadLocal（简称TTL）是alibaba提供的一个工具包中的类，主要作用就是解决线程池场景下的变量传递问题。继承自InheritableThreadLocal 主要用途：

1. 分布式跟踪系统 或 全链路压测（即链路打标）
2. 日志收集记录系统上下文
3. `Session`级`Cache`
4. 应用容器或上层框架跨应用代码给下层`SDK`传递信息

## 需求场景

在`ThreadLocal`的需求场景即是`TTL`的潜在需求场景，如果你的业务需要『在使用线程池等会池化复用线程的组件情况下传递`ThreadLocal`』则是`TTL`目标场景。

- [🔎 1. 分布式跟踪系统](https://github.com/alibaba/transmittable-thread-local/blob/master/docs/requirement-scenario.md#-1-分布式跟踪系统)
- 🌵 2. 日志收集记录系统上下文
  - [`Log4j2 MDC`的`TTL`集成](https://github.com/alibaba/transmittable-thread-local/blob/master/docs/requirement-scenario.md#log4j2-mdc的ttl集成)
  - [`Logback MDC`的`TTL`集成](https://github.com/alibaba/transmittable-thread-local/blob/master/docs/requirement-scenario.md#logback-mdc的ttl集成)
- [👜 3. `Session`级`Cache`](https://github.com/alibaba/transmittable-thread-local/blob/master/docs/requirement-scenario.md#-3-session级cache)
- 🛁 4. 应用容器或上层框架跨应用代码给下层`SDK`传递信息
  - [上面场景使用`TTL`的整体构架](https://github.com/alibaba/transmittable-thread-local/blob/master/docs/requirement-scenario.md#上面场景使用ttl的整体构架)

## 使用

### 分布式日志追踪

具体参考：分布式日志追踪系统

### 线程池替换

使用TTL替换`{@link ThreadPoolTaskExecutor}`

```java
public class CustomThreadPoolTaskExecutor extends ThreadPoolTaskExecutor{
    private static final long serialVersionUID = 8006794939825079647L;

    @Override
    public void execute(Runnable runnable) {
        Runnable ttlRunnable = TtlRunnable.get(runnable);
        super.execute(ttlRunnable);
    }

    @Override
    public <T> Future<T> submit(Callable<T> task) {
        Callable ttlCallable = TtlCallable.get(task);
        return super.submit(ttlCallable);
    }

    @Override
    public Future<?> submit(Runnable task) {
        Runnable ttlRunnable = TtlRunnable.get(task);
        return super.submit(ttlRunnable);
    }

    @Override
    public ListenableFuture<?> submitListenable(Runnable task) {
        Runnable ttlRunnable = TtlRunnable.get(task);
        return super.submitListenable(ttlRunnable);
    }

    @Override
    public <T> ListenableFuture<T> submitListenable(Callable<T> task) {
        Callable ttlCallable = TtlCallable.get(task);
        return super.submitListenable(ttlCallable);
    }
}

```

## Reference

- 官方仓库：https://github.com/alibaba/transmittable-thread-local
- https://juejin.cn/post/6998552093795549191
- LogbackMDC文档：http://logback.qos.ch/manual/mdc.html