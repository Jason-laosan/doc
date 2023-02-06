# Kafka中offsets.retention.minutes和log.retention.minutes之间的区别
前言
在Kafka中，我们可能会发现两个与retention相关的配置：

log.retention.minutes
offsets.retention.minutes
那么它们之前的差别是什么呢？

定义
首先让我们看看它们在官方文档中的定义

|名称|描述|类型|默认值|有效值|重要性|
| :--: |:--: |:--:|:--:|:--:|:--:|
|  log.retention.minutes | The number of minutes to keep a log file before deleting it (in minutes), secondary to log.retention.ms property. If not set, the value in log.retention.hours is used
在删除日志文件之前保留日志文件的分钟数（以分钟为单位），优先级弱于 log.retention.ms。 如果未设置，则使用log.retention.hours中的值 | int | null |  null | null  | 高 |
| offsets.retention.minutes | Log retention window in minutes for offsets topic主题偏移量日志文的保留时长(分钟) | int | 1440 | [1,...] | 高 |


## 两者的差别
log.retention.minutes设定的是消息日志的保留时长，而offsets.retention.minutes则是记录topic的偏移量日志的保留时长。

偏移量是指向消费者已消耗的最新消息的指针。 比如，你消费了10条消息，那么偏移量将移动10个位置。 这个偏移量会被记录到日志中，以便我们下次消费时知道应该从哪个offset开始继续消费。
而offsets.retention.minutes允许我们将偏移量重置，即它会清除过期的记录主题偏移量的日志，一旦记录主题偏移量的日志被清楚，我们将不知道之前消费到具体哪个offset。这个设置并不会影响消息日志的保留时间。

比如我们将offsets.retention.minutes设为10，即十分钟。然后最后一次主题A的消费偏移量是100，但是十分钟内我们没有继续消费，该记录主题A的消费偏移量100的日志将会被清除，也就是下次继续消费主题A的消息时，我们不知道上一次消费哪里了(注意，主题A所存储的消息依旧在broker上，并没有被删除), 在这种情况下，将会根据auto.offset.reset 的设置，读取最早(smallest)/最晚(largest)的消息。

一般来说，记录topic的偏移量日志的保留时长需要设置的比消息日志的保留时长更大。
### springboot-kafka 配置项
```
spring:
  kafka:
    bootstrap-servers: 192.168.0.22:9092
    producer:
      retries: 3 # 设置大于0的值，则客户端会将发哦是那个失败的记录重新发送
      batch-size: 16384
      buffer-memory: 33554432
      # 指定消息key 和消息体的编解码方式
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
    consumer:
      group-id: default-group
      #是否开始自动提交
      enable-auto-commit: false
      # earliest 当各个分区下有已提交的offset时， 从提交的offset开始消费，无提交时，从头开始
      # latest 当各个分区下有已提交的offset时， 从提交的offset开始消费，无提交的offset时， 消费新产生的该分区下的数据
      # none 当哥哥分区都存在已提交的offsite时 ， 从offset后开始消费， 只要有一个分区不存在已提交的offset， 则抛出异常
      auto-offset-reset: earliest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
    listener:
        # 当每一条记录被消费者监听器处理之后提交
        #  RECORD
        # 当每一批poll()的数据被消费者监听器 处理之后提交
        # BATCH
        # 当每一批poll()的数据被消费者监听器 处理之后 距离上次提交时间大于TIME时提交
        # TIME
        # 当每一批poll()的数据被消费者监听器 处理之后 被处理record数量大于等于count时提交
        # COUNT
        # TIME | COUNT  有一个条件满足时提交
        # COUNT_TIME
        # 当每一批poll()的数据被消费者监听器 处理之后 手动调用 Acknowledgment.acknowledge()后提交
        # MANUAL
        #  手动调用 Acknowledgment.acknowledge()后立即提交， 一般使用这种
        # MANUAL_IMMEDIATE
        ack-mode: manual
```