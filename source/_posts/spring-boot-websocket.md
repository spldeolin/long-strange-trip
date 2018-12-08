---
title: Spring Boot集成WebSocket

date: 2018-08-17 09:35:00

tags:
- Spring Boot
- Websocket

categories: Java

permalink: spring-boot-websocket
---



## POM

~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
~~~



## 配置类

~~~java
@Configuration
public class WebsocketConfig {

    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }

}
~~~



## WebSocket服务

~~~java
@ServerEndpoint(value = "/overview")
@Component
public class OverviewWebsocket {

    private static AtomicInteger onlineCount = new AtomicInteger(0);

    private static CopyOnWriteArraySet<OverviewWebsocket> webSocketSet = new CopyOnWriteArraySet<>();

    // 客户端会话
    private Session session;

    @OnOpen
    public void onOpen(Session session) {
        this.session = session;
        webSocketSet.add(this);
        log.info("客户端加入，当前会话数" + onlineCount.incrementAndGet());

        try {
            sendMessage("SUCCESS");
        } catch (IOException e) {
            log.error(e);
        }
    }

    @OnClose
    public void onClose() {
        webSocketSet.remove(this);
        log.info("客户端退出，当前会话数" + onlineCount.decrementAndGet());
    }


    @OnMessage
    public void onMessage(String message) {
        log.info("群体推送 " + message);

        for (OverviewWebsocket item : webSocketSet) {
            try {
                item.sendMessage(message);
            } catch (IOException e) {
                log.error(e);
            }
        }
    }

    @OnError
    public void onError(Throwable e) {
        log.error(e);
    }

    private void sendMessage(String message) throws IOException {
        this.session.getBasicRemote().sendText(message);
    }

}
~~~



## 客户端加入

服务端已经搭建完毕，可以让客户端测试一下

~~~javascript
var websocket = new WebSocket('ws://192.168.100.173:8080/overview'); // 本地IP
websocket.onmessage = function(msg) {console.log(msg)};
~~~



## 推送消息

尝试一下定时推送

~~~java
@Component
@EnableScheduling
public class SomeService {

    @Autowired
    private OverviewWebSocket overview;

    @Scheduled(fixedDelay = 2000)
    public void send() {
        Equipment user = Equipment.builder().name("西行寺幽幽子").build();
        overview.onMessage(Jsons.toJson(user));
    }

}
~~~

