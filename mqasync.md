---
title: 使用消息队列扩展异步执行的实现方式
date: 2020-11-06 17:52:00
---
# 背景

你可能在你的项目中用过Spring的@Async注解，以此来将部分方法转化为异步执行，从而提高请求的响应效率

但在服务架构不断的演进之中，这种丢入线程池处理的方式带来的缺陷也愈发明显：

-   不利于监控
-   如果意外停机，尚未处理的任务会尽数丢失
-   在集群中的某个节点要处理大量异步任务时，无法将压力分担到集群中其他节点
-   项目中若集成了使用ThreadLocal特性的模块或第三方组件，需要注意上下文丢失的问题

# 思路

使用消息队列作为异步任务的实现方式，这样我们就可以：

-   大量成熟的MQ中间件都提供了可视化管理平台，监控更加方便
-   可以用消息队列Header来保存上下文，如用户信息、token等
-   消息队列的发布-订阅模式可以最大程度利用集群的业务处理能力
-   更容易保证任务的顺序性
-   如果有服务节点宕机，可以利用消息确认、消息重试等机制保证任务执行的正确性

# 实现

为了保证业务代码和实现方案解耦，类似于@Aync方案，我们同样采用注解+拦截器的方式进行逻辑注入

```java
@Around("@annotation(org.springframework.amqp.rabbit.annotation.RabbitListener)")
	public Object cut(ProceedingJoinPoint pjp) throws Throwable {
		...
	}
```

实现思路大同小异，就是读取注解中的队列声明确认发布-订阅关系，然后以丢入消息队列来替换丢入线程池 

```java
    private String resolveKey(Queue[] queues) {
		String s = this.beanFactory.resolveEmbeddedValue(queues[0].value());
		return (String) resolver.evaluate(s, evalContext);
	}
```

```java
rabbitTemplate.convertAndSend(resolveKey(queues), args[0]);
```

为消息队列注入Json转换器，方便对象传输

```java
    @Bean
    public Jackson2JsonMessageConverter producerJackson2MessageConverter() {
        return new Jackson2JsonMessageConverter();
    }
```

如有需要，我们可以将上下文的用户信息、token等写入消息的Header中

```java
    private MessagePostProcessor beforePublishPostProcessor() {
        return message -> {
//          setting up context to message header
            return message;
        };
    }
```

被异步调用的service代码：

```java
@Service
@Slf4j
public class DemoService {
    
    @RabbitListener(queuesToDeclare = @Queue("mytestqueue"))
    public void checkSome(List<String> tagTuple) {
        log.warn("check here {}", tagTuple);
    }
}
```

调用service的controller：

@RestController **public class** DemoController {
    
    @Autowired
  **private** DemoService **demoService**;
    
    @RequestMapping(**"check"**)
    **public** Integer checkSome() {
        ArrayList<String> tagTuple = **new** ArrayList<>();
        tagTuple.add(**"bar"**);
        tagTuple.add(**"foo"**);
        **demoService**.checkSome(tagTuple);
        **return** 0;
    }
}

执行查看效果

```
17:00:14.584TRACE[AbstractHandlerMapping.java:411]Mapped to org.smop.duplex.sample.DemoController#checkSome()
17:00:21.263WARN [DemoService.java:16]check here [bar, foo]
```

如有帮助或启发，还请点个👍

附代码仓库：

[https://github.com/s-mop/homer](https://github.com/s-mop/homer)
