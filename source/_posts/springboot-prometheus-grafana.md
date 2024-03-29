---
title: springboot应用基于prometheus监控自定义指标
tags:
  - springboot
  - prometheus
categories:
  - java
  - monitor
date: 2021-12-07 10:07:10
---



系统开发中需要统计很多应用指标, 比如: 接口请求QPS, 接口响应时间统计, 接口耗时发布等. SpringBoot自带的`spring-actuator`中集成了`Micrometer`进行度量统计, 然后我们再用`Promethues`收集存储指标后, 在`Grafana`中以图标展示.



# 1. Micrometer

## 1.1 Micrometer提供的度量类库

在`Micrometer`中, `Meter`接口是用于收集应用中度量数据的, 它是由`MeterRegistry`创建和保存的, 可以将`MeterRegistry`理解为`Meter`的工厂和缓存中心. 一般而言, 每个JVM应用在使用`Micrometer`时都需要创建一个`MeterRegistry`的具体实现. 

`MeterRegistry`在`Micrometer`中是一个抽象类, 主要实现包括:

- `SimpleMeterRegistry` : 每个Meter的最新数据可以收集到`SimpleMeterRegistry`实例中，但是这些数据不会发布到其他系统，也就是说数据是位于应用的内存中的。**适合调试的时候使用**
- `CompositeMeterRegistry`: 多个`MeterRegistry`的聚合
- 全局的`MeterRegistry`: 工厂类`io.micrometer.core.instrument.Metrics`中持有一个静态final类型的`CompositeMeterRegistry`实例

## 1.2 Meter

### Counter

`Counter`是一种比较简单的`Meter`, 它的值**只能单调递增**. 可以用来统计比如: 接口调用次数, 订单总量等. 使用实例

```java
MeterRegistry meterRegistry = new SimpleMeterRegistry();
Counter counter = meterRegistry.counter("http.request", "createOrder", "/order/create");
counter.increment();
System.out.println(counter.measure()); // [Measurement{statistic='COUNT', value=1.0}]
```

通过Tag可以区分不同的场景，对于下单，可以使用不同的Tag标记不同的业务来源或者是按日期划分，对于Http请求总量记录，可以使用Tag区分不同的URL。用下单业务举个例子：

```java
//实体
@Data
public class Order {

    private String orderId;
    private Integer amount;
    private String channel;
    private LocalDateTime createTime;
}


public class CounterMain {

    private static final DateTimeFormatter FORMATTER = DateTimeFormatter.ofPattern("yyyy-MM-dd");

    static {
            Metrics.addRegistry(new SimpleMeterRegistry());
        }

        public static void main(String[] args) throws Exception {
            Order order1 = new Order();
            order1.setOrderId("ORDER_ID_1");
            order1.setAmount(100);
            order1.setChannel("CHANNEL_A");
            order1.setCreateTime(LocalDateTime.now());
            createOrder(order1);
            Order order2 = new Order();
            order2.setOrderId("ORDER_ID_2");
            order2.setAmount(200);
            order2.setChannel("CHANNEL_B");
            order2.setCreateTime(LocalDateTime.now());
            createOrder(order2);
            // 通过Micrometer的Search接口查找指标数据
            Search.in(Metrics.globalRegistry).meters().forEach(each -> {
                StringBuilder builder = new StringBuilder();
                builder.append("name:")
                        .append(each.getId().getName())
                        .append(",tags:")
                        .append(each.getId().getTags())
                        .append(",type:").append(each.getId().getType())
                        .append(",value:").append(each.measure());
                System.out.println(builder.toString());
            });
    }

    private static void createOrder(Order order) {
        //忽略订单入库等操作
        Metrics.counter("order.create",
                "channel", order.getChannel(),
                "createTime", FORMATTER.format(order.getCreateTime())).increment();
    }
}
```

控制台输出

```
name:order.create,tags:[tag(channel=CHANNEL_A), tag(createTime=2018-11-10)],type:COUNTER,value:[Measurement{statistic='COUNT', value=1.0}]
name:order.create,tags:[tag(channel=CHANNEL_B), tag(createTime=2018-11-10)],type:COUNTER,value:[Measurement{statistic='COUNT', value=1.0}]
```

上面的例子是使用全局静态方法工厂类Metrics去构造Counter实例，实际上，`io.micrometer.core.instrument.Counter`接口提供了一个内部建造器类`Counter.Builder`去实例化`Counter`，Counter.Builder的使用方式如下：

```java
public static void main(String[] args) throws Exception{
    Counter counter = Counter.builder("name")  //名称
        .baseUnit("unit") //基础单位
        .description("desc") //描述
        .tag("tagKey", "tagValue")  //标签
        .register(new SimpleMeterRegistry());//绑定的MeterRegistry
    counter.increment();
}
```

### FunctionCounter

`FunctionCounter`是`Counter`的特化类型, 它把计数器值增加的动作抽象成JDK1.8的接口类型`ToDoubleFunction`. 它的使用场景与`Counter`一致, 下面介绍下其语法:

```java
public static void main(String[] args) throws Exception {
    MeterRegistry registry = new SimpleMeterRegistry();
    AtomicInteger n = new AtomicInteger(0);
    //这里ToDoubleFunction匿名实现其实可以使用Lambda表达式简化为AtomicInteger::get
    FunctionCounter.builder("functionCounter", n, new ToDoubleFunction<AtomicInteger>() {
        @Override
        public double applyAsDouble(AtomicInteger value) {
            return value.get();
        }
    }).baseUnit("function")
        .description("functionCounter")
        .tag("createOrder", "CHANNEL-A")
        .register(registry);
    //下面模拟三次计数      
    n.incrementAndGet();
    n.incrementAndGet();
    n.incrementAndGet();
}
```

`FunctionCounter`使用的一个明显的好处是，我们不需要感知FunctionCounter实例的存在，实际上我们只需要操作作为FunctionCounter实例构建元素之一的AtomicInteger实例即可，这种接口的设计方式在很多框架里面都可以看到。


### Timer

`Timer`适用于记录耗时比较短的事件的执行时间, 通过时间发布展示事件的序列和发生频率. 所有的`Timer`实现至少记录了发生的事件的数量和这些事件的总耗时, 从而生成一个时间序列. `Timer`的基本单位基于服务端的指标而定, 但是实际上我们不需要过于关注Timer的基本单位，因为Micrometer在存储生成的时间序列的时候会自动选择适当的基本单位。Timer接口提供的常用方法如下：

```java
public interface Timer extends Meter {
    ...
    void record(long var1, TimeUnit var3);

    default void record(Duration duration) {
        this.record(duration.toNanos(), TimeUnit.NANOSECONDS);
    }

    <T> T record(Supplier<T> var1);

    <T> T recordCallable(Callable<T> var1) throws Exception;

    void record(Runnable var1);

    default Runnable wrap(Runnable f) {
        return () -> {
            this.record(f);
        };
    }

    default <T> Callable<T> wrap(Callable<T> f) {
        return () -> {
            return this.recordCallable(f);
        };
    }

    long count();

    double totalTime(TimeUnit var1);

    default double mean(TimeUnit unit) {
        return this.count() == 0L ? 0.0D : this.totalTime(unit) / (double)this.count();
    }

    double max(TimeUnit var1);
    ...
}
```

使用下单业务做个例子:

```java
public class TimerMain {

        private static final Random R = new Random();

        static {
            Metrics.addRegistry(new SimpleMeterRegistry());
        }

        public static void main(String[] args) throws Exception {
            Order order1 = new Order();
            order1.setOrderId("ORDER_ID_1");
            order1.setAmount(100);
            order1.setChannel("CHANNEL_A");
            order1.setCreateTime(LocalDateTime.now());
            Timer timer = Metrics.timer("timer", "createOrder", "cost");
            timer.record(() -> createOrder(order1));
        }

        private static void createOrder(Order order) {
            try {
                TimeUnit.SECONDS.sleep(R.nextInt(5)); //模拟方法耗时
            } catch (InterruptedException e) {
                //no-op
            }
        }
}
```

在实际生产环境中，可以通过spring-aop把记录方法耗时的逻辑抽象到一个切面中，这样就能减少不必要的冗余的模板代码。上面的例子是通过Mertics构造Timer实例，实际上也可以使用Builder构造：

```java
MeterRegistry registry = ...
Timer timer = Timer
    .builder("my.timer")
    .description("a description of what this timer does") // 可选
    .tags("region", "test") // 可选
    .register(registry);
```

另外，Timer的使用还可以基于它的内部类Timer.Sample，通过start和stop两个方法记录两者之间的逻辑的执行耗时。例如：

```java
Timer.Sample sample = Timer.start(registry);

// 这里做业务逻辑
Response response = ...

sample.stop(registry.timer("my.timer", "response", response.status()));
```

### FunctionTimer

`FunctionTimer`是`Timer`的特化类型, 它需要两个函数:一个用于计数, 一个用于统计总耗时. 它的构造器方法入参如下:

```java
public interface FunctionTimer extends Meter {
    static <T> Builder<T> builder(String name, T obj, ToLongFunction<T> countFunction,
                                  ToDoubleFunction<T> totalTimeFunction,
                                  TimeUnit totalTimeFunctionUnit) {
        return new Builder<>(name, obj, countFunction, totalTimeFunction, totalTimeFunctionUnit);
    }
    ...
}        
```

简单的使用方式如下:

```java
public static void main(String[] args) throws Exception {
    //这个是为了满足参数,暂时不需要理会
    Object holder = new Object();
    AtomicLong totalTimeNanos = new AtomicLong(0);
    AtomicLong totalCount = new AtomicLong(0);
    // 注意总时间的单位
    FunctionTimer.builder("functionTimer", holder, p -> totalCount.get(), 
                          p -> totalTimeNanos.get(), TimeUnit.NANOSECONDS)
        .register(new SimpleMeterRegistry());
    totalTimeNanos.addAndGet(10000000);
    totalCount.incrementAndGet();
}
```

### LongTaskTimer

`LongTaskTimer`也是`Timer`的一种特化类型, 主要用于记录长时间执行的任务的持续时间. 在Spring中可以简单地使用@Scheduled和@Timed注解，基于spring-aop完成定时调度任务的总耗时记录：

```java
@Timed(value = "aws.scrape", longTask = true)
@Scheduled(fixedDelay = 360000)
void scrapeResources() {
    //这里做相对耗时的业务逻辑
}
```

也可以手动使用`LongTaskTimer`

```java
public static void main(String[] args) throws Exception{
    MeterRegistry meterRegistry = new SimpleMeterRegistry();
    LongTaskTimer longTaskTimer = meterRegistry.more().longTaskTimer("longTaskTimer");
    longTaskTimer.record(() -> {
        //这里编写Task的逻辑
    });
    
    //或者这样
    Metrics.more().longTaskTimer("longTaskTimer").record(()-> {
        //这里编写Task的逻辑
    });
}
```

### Gauge

`Gauge`是用来记录可以上下浮动的单数值度量`Meter`, 测量值用`ToDoubleFunction`参数的返回值, 如当前的内存使用情况, 队列中的消息数量等. Gauge一般用于监测有自然上界的事件或者任务，而Counter一般使用于无自然上界的事件或者任务的监测，所以像Http请求总量计数应该使用Counter而非Gauge。 `MeterRegistry`中提供了一些便于构建用于观察数值、函数、集合和映射的Gauge相关的方法：

```java
List<String> list = registry.gauge("listGauge", Collections.emptyList(), new ArrayList<>(), List::size); 
List<String> list2 = registry.gaugeCollectionSize("listSize2", Tags.empty(), new ArrayList<>()); 
Map<String, Integer> map = registry.gaugeMapSize("mapGauge", Tags.empty(), new HashMap<>());
```

上面的三个方法通过`MeterRegistry`构建Gauge并且返回了集合或者映射实例，使用这些集合或者映射实例就能在其size变化过程中记录这个变更值。更重要的优点是，我们不需要感知Gauge接口的存在，只需要像平时一样使用集合或者映射实例就可以了。此外，Gauge还支持`java.lang.Number`的子类，`java.util.concurrent.atomic`包中的`AtomicInteger`和`AtomicLong`，还有Guava提供的`AtomicDouble`:

```java
AtomicInteger n = registry.gauge("numberGauge", new AtomicInteger(0));
n.set(1);
n.set(2);
```

除了使用MeterRegistry创建Gauge之外，还可以使用建造器流式创建：

```java
//一般我们不需要操作Gauge实例
Gauge gauge = Gauge
    .builder("gauge", myObj, myObj::gaugeValue)
    .description("a description of what this gauge does") // 可选
    .tags("region", "test") // 可选
    .register(registry);
```

举个相对实际的例子，假设我们需要对登录后的用户发送一条[短信](https://cloud.tencent.com/product/sms?from=10680)或者推送，做法是消息先投放到一个阻塞队列，再由一个线程消费消息进行其他操作：

```java
private static final MeterRegistry REGISTRY = new SimpleMeterRegistry();
private static final BlockingQueue<Message> QUEUE = new ArrayBlockingQueue<>(500);
private static BlockingQueue<Message> REAL_QUEUE;

static {
    REAL_QUEUE = REGISTRY.gauge("messageGauge", QUEUE, Collection::size);
}

public static void main(String[] args) throws Exception {
    consume();
    Message message = new Message();
    message.setUserId(1L);
    message.setContent("content");
    REAL_QUEUE.put(message);
}

private static void consume() throws Exception {
    new Thread(() -> {
        while (true) {
            try {
                Message message = REAL_QUEUE.take();
                //handle message
                System.out.println(message);
            } catch (InterruptedException e) {
                //no-op
            }
        }
    }).start();
}
```

### TimeGauge

`TimeGauge`是`Gauge`的特化类型, 相比`Gauge`, 它的构造器中多了一个`TimeUnit`类型的参数, 用于指定`ToDoubleFunction`入参的时间单位. 

```java
private static final SimpleMeterRegistry REGISTRY = new SimpleMeterRegistry();

public static void main(String[] args) throws Exception{
    AtomicInteger count = new AtomicInteger();
    TimeGauge.Builder<AtomicInteger> timeGauge = TimeGauge.builder("timeGauge", count,
                                                                   TimeUnit.SECONDS, AtomicInteger::get);
    timeGauge.register(R);
    count.addAndGet(10086);
    print();
    count.set(1);
    print();
}

private static void print()throws Exception{
    Search.in(REGISTRY).meters().forEach(each -> {
        StringBuilder builder = new StringBuilder();
        builder.append("name:")
            .append(each.getId().getName())
            .append(",tags:")
            .append(each.getId().getTags())
            .append(",type:").append(each.getId().getType())
            .append(",value:").append(each.measure());
        System.out.println(builder.toString());
    });
}
```

控制台输出

```
name:timeGauge,tags:[],type:GAUGE,value:[Measurement{statistic='VALUE', value=10086.0}]
name:timeGauge,tags:[],type:GAUGE,value:[Measurement{statistic='VALUE', value=1.0}]
```

### DistributionSummary

`Summary`主要用于跟踪事件的发布, 使用方式与`Timer`类型, 但是它的记录值并不依赖与时间单位. 常见的场景: 记录请求的有效负载大小. 使用`MeterRegistry`创建`DistributionSummary`的方式如下:

```java
DistributionSummary summary = registry.summary("response.size");
```

使用构造器流式创建:

```java
DistributionSummary summary = DistributionSummary
    .builder("response.size")
    .description("a description of what this summary does") // 可选
    .baseUnit("bytes") // 可选
    .tags("region", "test") // 可选
    .scale(100) // 可选
    .register(registry);
```

使用例子:

```java
private static final DistributionSummary SUMMARY  = DistributionSummary.builder("cacheHitPercent")
    .register(new SimpleMeterRegistry());

private static final LoadingCache<String, String> CACHE = CacheBuilder.newBuilder()
    .maximumSize(1000)
    .recordStats()
    .expireAfterWrite(60, TimeUnit.SECONDS)
    .build(new CacheLoader<String, String>() {
        @Override
        public String load(String s) throws Exception {
            return selectFromDatabase();
        }
    });

public static void main(String[] args) throws Exception{
    String key = "dog";
    String value = CACHE.get(key);
    record();
}

private static void record()throws Exception{
    CacheStats stats = CACHE.stats();
    BigDecimal hitCount = new BigDecimal(stats.hitCount());
    BigDecimal requestCount = new BigDecimal(stats.requestCount());
    SUMMARY.record(hitCount.divide(requestCount, 2, BigDecimal.ROUND_HALF_DOWN).doubleValue());
}
```



# 2. SpringBoot应用开发

## 2.1 Maven依赖配置

在`pom.xml`中增加如下依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

然后spring的配置类中, 为所有指标配置默认标签(应用名, 本机IP)

```java
@Bean
MeterRegistryCustomizer<MeterRegistry> configurer(@Value("${spring.application.name}") String applicationName) {
    // 添加全局标签, micrometer 1.1.0 之后可以通过配置文件方式添加
    return registry -> registry.config().commonTags("application", applicationName).commonTags("ip", cn.hutool.core.net.NetUtil.getLocalhostStr());
}
```

在spring配置文件中将actuator相关端点暴露出去

```yaml
management:
  metrics:
    tags:
      # micrometer 1.1.0之后使用该方式配置全局标签
      application: ${spring.application.name}
  endpoints:
    web:
      exposure:
        include: "*"
```

## 2.2 自定义指标开发

下面在一个接口内演示所有指标的用法

```java
@RestController
@RequestMapping("/user")
public class UserController {
    @Autowired
    MeterRegistry registry;
    
    private Counter counter;
    private AtomicInteger gauge;
    private DistributionSummary summary;
    private LongTaskTimer long_task_timer;
    private DistributionSummary summarySimple;

    @PostConstruct
    private void init() {
        // Gauge.builder("lego.thread-pool.pending_task", new AtomicInteger(37), AtomicInteger::get);
        Tags tags = Tags.of("endpoint", "user");
        counter = Counter.builder("counter_user").tag("method", "IndexController.core").register(registry);
        gauge = registry.gauge("guage_user", new AtomicInteger(0));
        summarySimple = registry.summary("summary_user");
        
        // 客户端计算百分位publishPercentiles 发布直方图publishPercentileHistogram(指标较多, 谨慎开启)
        summary = DistributionSummary.builder("summary_quantile").scale(100).publishPercentiles(0.5, 0.8, 0.95)
                                     .publishPercentileHistogram().register(registry);
        long_task_timer = registry.more().longTaskTimer("long_task_timer_user");
    }

    @GetMapping
    public void testMetrics() throws InterruptedException {
        counter.increment();
        gauge.getAndSet(RandomUtils.nextInt(0, 100000));

        Timer.Sample sample = Timer.start(registry);
        // 模拟业务操作
        Thread.sleep(RandomUtils.nextInt(10, 1000));
        sample.stop(Timer.builder("timer_user").description("一段描述")
                         .tags("endpoint", "user").publishPercentileHistogram(true)
                         .publishPercentiles(0.5, 0.75, 0.9)
                         .register(registry));


        summarySimple.record(RandomUtils.nextDouble(10, 1000));
        summary.record(RandomUtils.nextDouble(10, 1000));
        long_task_timer.start();
        long_task_timer.record(() -> RandomUtils.nextInt(10, 10000));
    }
}    
```

应用启动后, 请求`http://localhost:端口/user`接口, 模拟业务接口访问, 然后在打开`http://localhost:端口/actuator/prometheus`就可以看到所有指标了. 除了我们自定义的指标, 还是许多默认的JVM, http请求的指标, 这些指标在`Grafana`中使用已经配置好的图标模板[JVM (Micrometer)](https://grafana.com/grafana/dashboards/4701) 或者 [Spring Boot Statistics](https://grafana.com/grafana/dashboards/12464)就可以展示了.

# 3.Promethues/Grafana配置

## 3.1 Prometheus配置

应用端配置好以后, 还需要配置`Promethues`去定时从应用拉取数据并保存起来. `Promethues`安装好以后, 编辑其配置文件(`prometheus.yml`)

```yaml
scrape_configs:
	# 新增如下行
   - job_name: 'springboot-micrometer'
   	 # 请求间隔
     scrape_interval: 5s
     # 请求路径
     metrics_path: '/actuator/prometheus'
     # 应用服务地址
     static_configs:
     - targets: ['172.31.47.13:8880']
```

然后重启`Promethues`. 然后打开`Promethues`的管理页面`http://IP:9090/targets`, 选择`Status` > `Targets`, 查看应用端点状态是否为`UP`. 点击`Graph`可以通过`PromQL`查询指标数据, 如: long_task_timer_user_seconds_duration_sum

## 3.2 Grafana图标配置

`Promethues`中有数据以后, 还需要在`Grafana`中以图表展示, 方便查看. 首先在`Grafana`配置好`Promethues`数据源, 然后可以新建一个`Dashboard`, 在其中配置不同指标的`Panel`. 配置语法如下:

- Counter

  - counter_user_total{instance="$instance", application="$application"}
  - rate(counter_user_total{instance="$instance", application="$application"}[1m])

- Timer

  - timer_user_seconds_count{instance="$instance", application="$application"}
  - rate(timer_user_seconds_sum{instance="$instance", application="$application"}[1m])/rate(timer_user_seconds_count{instance="$instance", application="$application"}[1m])

- Gauge

  - guage_user{application="$application", instance="$instance"}

- Summary

  - irate(http_server_requests_seconds_count{instance="$instance", application="$application", uri!~".*actuator.*"}[3m])

    > 可以在`Legend`填写标签, 这样可以根据标签值自动新建折线 {{method}} [{{status}}] - {{uri}}

## 3.3 Promethues 查询语法PromQL

#### 3.3.1 操作符

- 数学运算

  `PromQL`支持的数学运算符: `+`(加) `-`(减) `*`(乘) `/`(除) `%`(取余) `^`(幂运算)

  ```
  node_memory_free_bytes_total / (1024 * 1024)
  ```

- 布尔运算

  支持的布尔运算符: `==`(相等) `!=`(不等) `>`(大于) `<`(小于) `>=`(大于等于) `<=`(小于等于)
  
  ```
  (node_memory_bytes_total - node_memory_free_bytes_total) / node_memory_bytes_total > 0.95
  ```
  
- bool运算符

  通过bool运算符对布尔运算结果进行修改
  
  ```
  # 如果大于1000返回1(true) 否则返回0(false)
  http_requests_total > bool 1000
  ```
  
- 集合运算符

  支持的集合运算符: `and`(且) `or`(或) `unless`(排除)
  
  ***vector1 and vector2*** 会产生一个由vector1的元素组成的新的向量。该向量包含vector1中完全匹配vector2中的元素组成。
  
  ***vector1 or vector2*** 会产生一个新的向量，该向量包含vector1中所有的样本数据，以及vector2中没有与vector1匹配到的样本数据。
  
  ***vector1 unless vector2*** 会产生一个新的向量，新向量中的元素由vector1中没有与vector2匹配的元素组成。
  
- 操作符优先级

  ```
  100 * (1 - avg (irate(node_cpu{mode='idle'}[5m])) by(job) )
  ```
  
  - ^
  - *, /, %
  - +, -
  - ==, !=, <=, <, >=, >
  - and, unless
  - or


#### 3.3.2 集合函数

- sum(求和)

  ```
  # 假设系统的日志counter指标如下
  logback_events_total{application="untitled",ip="172.17.0.1",level="info",} 12.0
  logback_events_total{application="untitled",ip="172.17.0.1",level="debug",} 707.0
  logback_events_total{application="untitled",ip="172.17.0.1",level="trace",} 0.0
  logback_events_total{application="untitled",ip="172.17.0.1",level="warn",} 106.0
  logback_events_total{application="untitled",ip="172.17.0.1",level="error",} 101.0
  
  # 计算各个level日志的总量
  sum(logback_events_total)
  ```

- min(最小值)

- max(最大值)

- avg(平均值) avg_over_time(一段时间内的平均值)

  ```
  # 计算某个点的平均值  等同于  guage_user{application="$application", instance="$instance"}
  avg(guage_user{application="$application", instance="$instance"})
  
  # 计算1分钟内的平均值 
  avg_over_time(guage_user{application="$application", instance="$instance"}[1m])
  ```

  

- stddev(标准差)

- stdvar(标准方差)

- count(计数)

- count_values(相同数值分组计数)

- bottomk (最小的n条数据)

- topk (最大的n条数据)


#### 3.3.3 内置函数

- increase

  取时间范围内的最后一个值减去第一个值, 得到增长量

  ```
  # 每分钟的请求数(过去一分钟内的增长量) 如果tag多个值时会返回多条数据
  increase(prometheus_http_requests_total{job="api-server"}[1m])
  
  # 计算5分钟内的每秒增长率
  increase(http_requests_total{job="api-server"}[5m])/300
  
  # 5分钟的请求总数
  sum(increase(prometheus_http_requests_total[5m]))
  
  # 按code统计5分钟的请求总数 配置到Grafana时Legend填上{{code}}, 就可以按code展示多条曲线
  sum(increase(prometheus_http_requests_total[5m])) by (code)
  ```

- rate

  计算每秒的增长率

  ```
  # 计算5分钟内的每秒增长率
  rate(http_requests_total{job="api-server"}[5m])
  
  # 等同于
  increase(http_requests_total{job="api-server"}[5m])/300
  ```

- irate

  使用`increase`或者1函数计算平均增长率时, 会有"长尾问题", 无法反应时间窗口内的数据突变. 比如某一时刻数据突增, 但是平摊到时间窗口后增长率并不突出. 为了解决该问题, 可以使用`irate`函数, 它通过时间窗口内的最后两个数据的差值来计算增长率, 所以`irate`反应的是数据的瞬时增长率. 

  由于`irate`灵敏度更高, 所以在需要分享长期趋势或在告警规则中, 这种灵敏度反而会造成干扰, 因此此时更推荐使用`rate`函数.

  ```
  # 5m内的数据增长率 
  irate(http_requests_total{job="api-server"}[5m])
  ```

- predict_linear 预测

  基于简单线性回归, 对时间范围内的数据进行统计, 预测某段时间后的数值. **只适用于Gauge类型数据**, 数据有一定的局部单调性时才比较准, 不要忽上忽下剧烈变化.

  ```
  # 基于5分钟的数据, 预测对300秒后数值
  predict_linear(guage_user[5m], 300)
  ```
  
- histogram_quantile 分位数

  统计不同百分比

  ```
  # TYPE timer_user_seconds histogram
  timer_user_seconds{application="untitled",endpoint="user",ip="172.17.0.1",quantile="0.5",} 0.484442112
  timer_user_seconds{application="untitled",endpoint="user",ip="172.17.0.1",quantile="0.75",} 0.70254592
  timer_user_seconds{application="untitled",endpoint="user",ip="172.17.0.1",quantile="0.9",} 0.903872512
  timer_user_seconds_bucket{application="untitled",endpoint="user",ip="172.17.0.1",le="0.001",} 0.0
  timer_user_seconds_bucket{application="untitled",endpoint="user",ip="172.17.0.1",le="0.001048576",} 0.0
  timer_user_seconds_bucket{application="untitled",endpoint="user",ip="172.17.0.1",le="0.001398101",} 0.0
  timer_user_seconds_bucket{application="untitled",endpoint="user",ip="172.17.0.1",le="0.001747626",} 0.0
  timer_user_seconds_bucket{application="untitled",endpoint="user",ip="172.17.0.1",le="0.002097151",} 0.0
  timer_user_seconds_bucket{application="untitled",endpoint="user",ip="172.17.0.1",le="0.002446676",} 0.0
  ...
  
  # 在过去1小时内的P95（95%的请求耗时都小于等于这个值）
  histogram_quantile(0.95, rate(timer_user_seconds_bucket[1h]))
  ```

  

参考:

[JVM应用度量框架Micrometer实战](https://www.throwx.cn/2018/11/17/jvm-micrometer-prometheus/) [通过micrometer实时监控线程池的各项指标](https://www.throwx.cn/2019/04/14/jvm-micrometer-thread-pool-monitor/)

[prometheus-book](https://yunlzheng.gitbook.io/prometheus-book/)

[Micrometer](https://micrometer.io/docs/concepts)

[Promethues内置函数](https://prometheus.io/docs/prometheus/latest/querying/functions/)
