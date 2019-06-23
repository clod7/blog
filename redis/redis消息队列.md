# Redis去做消息队列总结

## redis push/pop VS pub/sub

```
    1.push/pop每条消息只会有一个消费者消费，而pub/sub可以有多个

对于任务队列来说，push/pop足够，但真的在做分布式消息分发的时候，还是pub/sub吧。

    2.从编程角度看，pub/sub中sub通道需要保持长连接，而push/pop,  pop需要Consumer进程定时去刷新。

前者可以满足实时要求，但是对编程架构有要求，而后者在实时性上有缺陷，但是对编程架构要求较低。
```


## redis vs kafka

```
    1.redis是内存数据库，只是它的list数据类型刚好可以用作消息队列而已

kafka是消息队列，消息的存储模型只是其中的一个环节，还提供了消息ACK和队列容量、消费速率等消息相关的功能，更加完善

    2.redis 发布订阅除了表示不同的 topic 外，并不支持分组

kafka每个consumer属于一个特定的consumer group（default group）, 同一topic的一条消息只能被同一个

consumer group内的一个consumer消费，但多个consumer group可同时消费这一消息。

    3.处理数据大小的级别不同
    
    4.redis取出数据，处理该数据时出现异常情况，则数据丢失。kafka可以进行回滚。
```

## 总结
如果消息量不大可以考虑redis，当消息量比较大或者消息比较重要时应使用kafka。