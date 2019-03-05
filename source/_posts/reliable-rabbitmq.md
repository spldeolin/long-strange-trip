---

title: RabbitMQ可靠消息落地方案

date: 2019-03-05 16:27

updated: 2019-03-05 16:27

tags:
- RabbitMQ

categories: Java

permalink: reliable-rabbitmq

---

## 简介

这篇POST将会介绍让RabbitMQ消息可靠投递的解决方案。

方案采用方式是，消息入库，结合定时任务重发异常消息。

最终能达到的效果是消息迟早能进入消息队列，或是被标记，以供人工处理。



## 思路

1. 为每一个需要发消息的业务实体建立一个对应的消息实体，如t_order和t_order_message
2. producer 业务实体入库，消息入库（状态为“投递中”），消息id与业务实体绑定
3. producer 发送消息，如果出错则本地回滚第1.步的入库操作
4. 消息落到队列后，rabbitmq会回调producer
5. producer confirm_call_back将数据库中消息的状态改为“成功”
6. 定时扫描“投递中”的消息，重发，记录重发次数，到达阈值后状态改为“失败”

其他： consumer幂等（通过消费端消息入库来实现）



## 代码片段

1. 消息表的表结构

   | 字段名        | 类型     | 备注                                             |
   | ------------- | -------- | ------------------------------------------------ |
   | id            | bigint   |                                                  |
   | exchange      | varchar  |                                                  |
   | routing_key   | varchar  |                                                  |
   | message       | varchar  | 业务实体的序列化或JSON，用于消息重发             |
   | retried_count | int      | 已经重发的次数，超过某个值时会被被标记为投递失败 |
   | status        | tinyint  | 1投递中，2投递成功，3投递失败等待人工补偿        |
   | next_retry_at | datetime | 下一次重发的时间                                 |

2. producer采用消息确认模式

   ~~~yaml
   spring:
     rabbitmq:
       publisher-confirms: true
       publisher-returns: true
       template:
         mandatory: true
   ~~~

3. producer 配置confirm_call_back

   ~~~java
   new RabbitTemplate.ConfirmCallback() {
   
       @Override
       public void confirm(CorrelationData data, boolean ack, String cause) {
           Long messageId = Long.valueOf(correlationData.getId());
               log.info("消息call_back：{}", messageId);
               OrderMessageEntity message = orderMessageMapper.selectById(messageId);
               if (ack) {
                   // customer ack
                   message.setStatus(MessageStatusEnum.SUCCESS.getValue());
               } else {
                   // customer nack
                   message.setRetriedCount(message.getRetriedCount() + 1);
                   message.setNextRetryAt(message.getNextRetryAt().plusMinutes(OrderMessageConstant.NextRetryIntervalMinutes));
               }
               orderMessageMapper.updateById(message);
       }
   }
   ~~~

4. producer 发消息

   ~~~java
   public void createOrder(OrderEntity order) {
       // 业务入库
       OrderEntity order = OrderEntity.mock();
       orderMapper.insert(order);
   
       // 消息入库
       OrderMessageEntity orderMessage = OrderMessageEntity
               .buildByOrder(order, OrderMessageConstant.exchange, OrderMessageConstant.routingKey);
       orderMessageMapper.insert(orderMessage);
       Long messageId = orderMessage.getId();
   
       // 消息绑定业务实体
       order.setOrderMessageId(messageId);
       orderMapper.updateById(order);
   
       // 发送消息
       log.info("发送消息：{}", messageId);
       orderSender.send(OrderMessageConstant.exchange, OrderMessageConstant.routingKey, messageId, order);
   }
   ~~~

5. customer采用自动ack

   ~~~yaml
   spring:
     rabbitmq:
       listener:
         simple:
           acknowledge-mode: auto
   ~~~

6. customer接受消息

   ~~~java
   @RabbitListener(bindings = @QueueBinding(
           value = @Queue(OrderConstant.MessageConstant.queueName),
           exchange = @Exchange(name = OrderConstant.MessageConstant.exchange, type = "topic"),
           key = OrderConstant.MessageConstant.routingKey))
   public void receiverOrder(Message message) {
       OrderEntity orderEntity = Jsons.toObject(new String(message.getBody()), OrderEntity.class);
       log.info("消费订单 {}", orderEntity);
   }
   ~~~

7. 定时任务

   ~~~java
   @Scheduled(fixedDelay = 60_000)
   public void task() {
       // "发送中"的消息
       LambdaQueryWrapper<OrderMessageEntity> query = new LambdaQueryWrapper<>();
       query.eq(OrderMessageEntity::getStatus, MessageStatusEnum.SENDING.getValue());
       List<OrderMessageEntity> sendingMessages = orderMessageMapper.selectList(query);
   
       for (OrderMessageEntity sendingMessage : sendingMessages) {
           if (sendingMessage.getRetriedCount() >= OrderMessageConstant.maxRetryCount) {
               // 超过最大重试次数，标记为失败
               sendingMessage.setStatus(MessageStatusEnum.FAILURE.getValue());
               orderMessageMapper.updateById(sendingMessage);
           } else {
               if (LocalDateTime.now().isAfter(sendingMessage.getNextRetryAt())) {
                   // 记录重试次数
                   sendingMessage.setRetriedCount(sendingMessage.getRetriedCount() + 1);
                   orderMessageMapper.updateById(sendingMessage);
   
                   // 发消息
                   log.info("重发消息：{}", sendingMessage.getId());
                   orderSender
                           .send(sendingMessage.getExchange(), sendingMessage.getRoutingKey(), sendingMessage.getId(),
                                   sendingMessage.getMessage());
               }
           }
       }
   ~~~

   

## 源码

[spldeolin](https://github.com/spldeolin) / [reliable-rabbit](https://github.com/spldeolin/reliable-rabbit)

如有错误或是疑问，欢迎反馈