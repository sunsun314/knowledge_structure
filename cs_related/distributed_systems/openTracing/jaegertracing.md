# JAEGER

## 简单demo记录
通过以下方法进行tracer的初始化，方便后续进行调用
```
 public static JaegerTracer init(String service) {
        final String endPoint = "http://10.186.60.96:14268/api/traces";

        final CompositeReporter compositeReporter = new CompositeReporter(
                new RemoteReporter.Builder()
                        .withSender(new HttpSender.Builder(endPoint).build())
                        .build(),
                new LoggingReporter()
        );

        final Metrics metrics = new Metrics(new NoopMetricsFactory());

        JaegerTracer.Builder builder = new JaegerTracer.Builder(service)
                .withReporter(compositeReporter)
                .withMetrics(metrics)
                .withExpandExceptionLogs()
                .withSampler(new ConstSampler(true));

        return builder.build();
    }
```
使用以下代码进行span的激活和调用
```
private void sayHello(String helloTo) {
        Span span = tracer.buildSpan("say-hello").start();
        span.setTag("hello-to", helloTo);

        String helloStr = String.format("Hello, %s!", helloTo);
        span.log(ImmutableMap.of("event", "string-format", "value", helloStr));

        System.out.println(helloStr);
        span.log(ImmutableMap.of("event", "println"));

        span.finish();
    }
```

## 注意关键点

+ Span 跨度，算是trace表现的最小单位
+ Scope 影响范围，这个算是trace对于层级描述的关键对象，值得额外的注意

值得额外注意的是在对应激活并产生Scope的代码
```
public class ThreadLocalScopeManager implements ScopeManager {
    final ThreadLocal<ThreadLocalScope> tlsScope = new ThreadLocal<ThreadLocalScope>();

    @Override
    public Scope activate(Span span) {
        return new ThreadLocalScope(this, span);
    }

    @Override
    public Span activeSpan() {
        ThreadLocalScope scope = tlsScope.get();
        return scope == null ? null : scope.span();
    }
}
```
上面这段代码说明了Scope在本质上是一个ThreadLocalScope对象，并且存储在一个ThreadLocal的对象tlsScope中  
只不过比较奇怪的是，这个对象只有activeSpan获取到Span的方法，但是并没有反向获取，也就是获取到Scope的方式  
Scope虽然产自于一个Span，但是实际上由于其特有的方式，并不受到Span的生命周期限制，所以，值得注意的是Scope需要手动close，而不finish Span就可以了