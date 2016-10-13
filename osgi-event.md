# osgi event 使用说明

## 概述

OSGi 的 Event Admin 服务规范提供了开发者基于发布 / 订阅模型，通过事件机制实现 Bundle 间协作的标准通讯方式。

事件发布者使用 Event Admin 服务发送基于主题 (Topic) 的事件，任何对某一主题感兴趣的事件订阅者都会收到该事件，并且做出相应的反应。

使用 Event Admin 提供的 postEvent 或者 sendEvent 方法发布事件，postEvent 使用同步的方式发布事件，即：等到所有的事件订阅者响应了该事件，方法调用返回，而 sendEvent 使用异步的方式发布事件。

## event的发布

```java
    protected EventAdmin eventAdmin;

    private void postEvent(String topic, Object obj) {
        if (eventAdmin != null) {
            Gson gson = new Gson();
            Dictionary properties = new Hashtable();
            properties.put("message", gson.toJson(obj));
            Event event = new Event(topic, properties);
            eventAdmin.postEvent(event);
        }
    }
```

## event的接收

### 需要实现EventHandler。

```java
public class AuditBusinessEventImpl implements EventHandler {

}
```

### 服务的注册

```xml
    <service interface="org.osgi.service.event.EventHandler" ref="auditBusinessEventImpl">
        <service-properties>
            <entry key="event.topics">
                <array value-type="java.lang.String">
                    <value>com/kalix/business/delete</value>
                    <value>com/kalix/business/save</value>
                    <value>com/kalix/business/update</value>
                </array>
            </entry>
        </service-properties>
    </service>
```

既可以如上注册多个事件，也可以如下注册单独一个事件。

```xml
    <service interface="org.osgi.service.event.EventHandler" ref="workFlowMessageEventImpl">
        <service-properties>
            <entry key="event.topics" value="com/kalix/workflow/engine/message"/>
        </service-properties>
    </service>
```

> **Note**:

在post的接收代码中使用shiroService获得当前登录的用户时，是无法获得正确的用户名称的。

```java
    @Override
    public void handleEvent(Event event) {
        String userName = shiroService.getCurrentUserRealName();
    }
```

OSGi Event Admin 的事件模型，与传统的 JMS 相比，OSGi Event Admin 事件不是持久化的，也就是说，通过 Event Admin 发布的事件只在本次应用会话期间存在，当 OSGi 应用重新启动后，所有的未处理的事件将全部丢失。

由于 OSGi 服务的动态特性，使用该模型时，必须考虑到以下的情况：由于订阅事件的 Bundle 可以在任意运行时刻注册实现了 org.osgi.service.event.EventHandler 接口的事件订阅服务，所以开发者不能期望订阅者总能够收到在本次会话期间事件发布者发布的所有事件，一个典型的情况是，如果在发布者发布事件的时候，订阅 者服务尚未注册，其结果是，该订阅者错过这次事件。

# 小结：
OSGi 服务层所提供的服务模型，其动态发现、动态绑定、契约与实现相分离等特性，可以帮助开发者实现组件之间依赖保持在接口之间，而与具体实现代码相对松散的应 用逻辑。在此基础上 OSGi 提供了基于事件的 Event Admin 服务。Event Admin 服务本质上是基于消息的发布 / 订阅机制，对于 OSGi 的开发者来说，Event Admin 服务可以帮助他们实现组件间更为松散的，不依赖任何接口定义的协作逻辑。