## IoT MQ设计篇：调研与协议选型

### 概述

本篇是IoT MQ系列的第一篇，本篇主要从以下几个维度介绍下IoT MQ：

1. IoT MQ和Kafka，RocketMQ，RabbitMQ这些消息队列有什么区别
2. 目前IoT的传输协议有哪些，有什么区别，如何选择合适的协议作为基础协议？
3. IoT MQ的适用场景有哪些？

### IoT MQ到底是什么东东

IoT MQ（Internet of things message queue）主要用来传输各种物联网设备的消息，可以理解IoT MQ就是物联网中的消息中间件。对于很多开发者而言，对于像Kafka，RocketMQ，RabbitMQ这样的消息队列是很熟悉的，那么IoT MQ相对于其他这种消息队列有哪些区别呢。

1. **适用对象：** Kafka这一类的MQ的适用对象是各个系统，主要解决问题是各业务系统间的数据交互问题，其中两个最为重要的特征是：“异步解耦，削峰填谷”，IoT MQ的适用对象主要是各种物联网设备，例如手环健康统计；大棚温度，光照数据采集设备；手机APP等。最明显的特征是：“连接终端多种多样，网络不可靠”。
2. **设计复杂度：** 设计系统时，系统的复杂度主要有“高可用，高性能，高可靠，规模，成本，安全，高扩展”等，其中“三高”是比较难设计的，对于Kafka，RocketMQ等就是“三高”的系统，所以内部很复杂，使用协议也是其自定义的私有协议。对于IoT MQ而言，消息数据的高可靠相对而言没有那么重要，因为百分之90的物联网设备的场景都是网络不可靠的，消息偶尔丢失是可以的。但是对于Kafka这类MQ和IoT MQ还有两个很明显的区别是：Kafka这一类是“低连接高请求“的，而IoT MQ是高连接高请求的。。

### IoT MQ协议

目前比较流行的IoT通信协议主要有：mqtt，coap，nb-iot，xmapp，http等。这里主要介绍mqtt和coap协议，这也是目前使用最多的协议。

#### coap

coap（Constrained Application Protocol）应用受限协议，其底层的传输协议是UDP，简单来看，通信方式像是一个很简单的http协议，主要特征有：

1. 基于UDP，不需要考虑连接成本，但是需要自己控制消息的重传，顺序等，相应的消息可靠性降低，但是传输成本降低，传输的数据量很小,
2. 通信方式跟Http很像，更像是M2M的，例如通过手机查看智能手环采集的心率信息。每个终端都可以看成即是服务器也是客户端。
3. 广播消息等是额外补充的协议，收集coap协议终端的数据相对比较麻烦。

#### mqtt

mqtt（Message Queuing Telemetry Transpor）消息队列遥测传输协议，其底层的传输协议是TCP，通信方式与Kafka这类MQ的通信方式很像，主要分为客户端设备与Mqtt服务端两个概念，设备采集消息与传输到服务端，服务端进行中专，主要特征有：

1. 基于TCP，在物联网内通信需要长连接，但是设备数很多（百万级）并且网络不可靠，所以对于连接设计的处理非常重要。
2. 默认是广播的，topic数量需要维护并且数量非常大，

mqtt与coap目前是使用最广的物联网通信协议，其中mqtt实际上使用最多的物联网协议，对与这两种协议而言，主要应用场景都是物联网，coap基于UDP，复杂度主要在于消息的可靠性，顺序，重传等需要自己处理，同时UDP保证了很高的性能，没有连接的开销；mqtt基于TCP，复杂度主要是设备session的保存，长连接的处理等，所以coap更适合那种对于数据可靠性要求比较低，数据性能要求更高，网络更差的场景，mqtt则与之相反，同时mqtt也很好做一些扩展，例如websocket协议，TLS安全等。如果要进行数据的采集，因为mqtt有Broker的概念，与kafka这些很像，这也是我们选择mqtt协议为基础研发IoT MQ的原因。

### IoT MQ适用场景

1. 物联网设备信息上行采集与下行下发：例如净水池的设备每隔一小时采集一条水质报告传输到云上进行分析，通过设备管理平台可以下发一些消息对每个设备进行管理
2. 设备与设备之间简单交互（M2M），例如自己手机观察手环采集的心率信息，不需要额外的服务器，只需要手机与手环进行交互
3. 即时通讯：mqtt协议比较适合用来做即时通信，即app聊天，通知，弹幕等
4. 其它：一般来说，IoT 意味着设备多种多样且数量很多，网络不可靠等，所以这些协议都是以这样的环境产生的，在设计这样的mq时，除了考虑本身协议的实现和应用场景外，还要考虑如何做一些扩展，特别是运维管理，监控等

我在github上用Java和netty实现了一个mqtt协议的服务端，欢迎使用：[jmqtt](https://github.com/Cicizz/jmqtt)