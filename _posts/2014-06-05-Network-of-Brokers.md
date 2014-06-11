---
layout: post
category : activemq
tagline: "Supporting tagline"
tags : [activemq, network, brokers]
---

为了提供可扩展的处理大数据量的能力你通常希望将多个 Brokder 连接成一个网络，在这个网络上你可以想连多少客户端就连多少，同时基于你的客户端数量和网络拓扑结构运行任意多个 Broker。

如果你使用 [客户端/服务器或者星型拓扑结构](http://activemq.apache.org/topologies.html) 那么你的 Broker 已经成为单点故障，这必将是你希望组建 Broker 网络（或集群）以实现 Broker、机器或者网络故障后恢复的又一个原因。

从 ActiveMQ 1.1 起已经支持 Broker 网络，通过这个网络允许我们实现 [分布式的队列和主题](http://activemq.apache.org/how-to-distributed-queues-work.html)。

这可以使客户端连接到网络中的任意一个 Broker，并且在连接失败时自动切换到另一个，从客户端的角度来看就类似一个 Broker 的 HA 集群。

**注意：**一般来说网络连接是单向的，激活这个连接的 Broker 把消息传递到他连接到的 Broker。从 ActiveMQ 5.x 开始，网络连接可以配置成双向的，这对中心处于防火墙后的星型拓扑等情况是很有用的。

## 配置一个 Broker 网络 ##
配置 Broker 网络最简单的途径是通过 [XML](http://activemq.apache.org/xml-configuration.html)。主要有以下两种途径：

1. 通过 URI 的硬编码(在 [activemq.xsd.html](http://activemq.codehaus.org/maven/activemq.xsd.html) 中的 networkConnector / networkChannel 结点）。
2. 使用 Broker 的探测发现机制（多播或者rendezvous）。

### 通过固定的 URI 列表 ###
这个使用了[固定的 URI 列表](http://svn.apache.org/viewvc/activemq/trunk/activemq-core/src/test/resources/org/apache/activemq/usecases/receiver.xml?view=markup)

    <?xml version="1.0" encoding="UTF-8"?> 
    <beans xmlns="http://activemq.org/config/1.0">
    
    <broker brokerName="receiver" persistent="false"  useJmx="false">  
        <networkConnectors>
            <networkConnector uri="static:(tcp://localhost:62001)"/>
        </networkConnectors>
 
        <persistenceAdapter>
            <memoryPersistenceAdapter/>
        </persistenceAdapter>
 
        <transportConnectors>
            <transportConnector uri="tcp://localhost:62002"/>
        </transportConnectors>
    </broker>
    
    </beans>
ActiveMQ 也支持 TCP 之外的例如 HTTP 协议进行网络连接。

### 通过多播自动发现 ###
这个例子使用了多播[发现机制](http://activemq.apache.org/discovery.html)

    <?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://activemq.org/config/1.0">
	 
	    <broker name="sender" persistent="false" useJmx="false">  
	        <networkConnectors>
	            <networkConnector uri="multicast://default"/>
	        </networkConnectors>
	 
	        <persistenceAdapter>
	            <memoryPersistenceAdapter/>
	        </persistenceAdapter>
	 
	        <transportConnectors>
	            <transportConnector uri="tcp://localhost:0" discoveryUri="multicast://default"/>
	        </transportConnectors>
	    </broker>
	 
	</beans>
## 启动网络连接器 ##
默认情况下，网络连接器作为按序启动的 Broker 的一部分是串行初始化的。当某些结点很慢的时候就会使别的结点也不能及时启动。版本 5.5 中支持 Broker 设置 networkConnectorStartAsync="true" 属性，这样就可以使 Broker 在启动时并行非同步的启动网络连接器。

## 静态发现 ##
通过 <code>static:</code> 发现你可以将 URL 直接列出。ActiveMQ 会为每个地址启动网络连接器。
    
    <networkConnectors>
        <networkConnector uri="static:(tcp://host1:61616,tcp://host2:61616,tcp://..)"/>
    </networkConnectors>
在静态网络连接器进行重连时主要有以下几个有用的参数：
<table>
<tr><th>属性名</th><th>默认值</th><th>描述</th></tr>
<tr><td>initialReconnectDelay</td><td>1000</td><td>当 useExponentialBackOff 为 false 时重连前的等待时间(ms)</td></tr>
<tr><td>maxReconnectDelay</td><td>300000</td><td>重连前的等待时间(ms)</td></tr>
<tr><td>useExponentialBackOff</td><td>true</td><td>每次重连失败是不是增加等待时间</td></tr>
<tr><td>backOffMultiplier</td><td>2</td><td>重连失败增加等待时间时的增加倍数</td></tr>
</table>
例如：

	uri="static:(tcp://host1:61616,tcp://host2:61616)?maxReconnectDelay=5000&useExponentialBackOff=false"
## 主从发现 ##
Broker 网络一般的配置方式是激活 Broker 和 n+1 Broker 之间的网络桥接。主要的配置项包括<code>failover:</code>，和相关的为使其正常工作的必须配置的其它选项。出于这个原因，ActiveMQ v5.6+ 版本中可以通过以<code>masterslave:</code>开头的连接地址进行便捷设置：

	<networkConnectors>
	  <networkConnector uri="masterslave:(tcp://host1:61616,tcp://host2:61616,tcp://..)"/>
	</networkConnectors>

这些 URI 的列表顺序是：主，从1，从2，……，从n

另：<code>static:</code>中的可选配置项在<code>masterslave:</code>中也是可用的。

## 网络连接器可选属性 ##
<table>
<tr><th>属性名</th><th>默认值</th><th>描述</th></tr>
<tr><td>name</td><td>bridge</td><td>网络的名字，如果在两个 Broker 间配置多个网络连接器需要使用不同的名字</td></tr>
<tr><td>dynamicOnly</td><td>false</td><td>如果是 true，只有在持久订阅激活时才激活对应的网络订阅，默认是启动激活</td></tr>
<tr><td>decreaseNetworkConsumerPriority </td><td>false</td><td>如果是 true，按 -5 优先级启动，消费者距离生产者距离（网络跳数）过远时降低分发优先级。为 false 时，所有消费者会和本地消费者一样，都为 0 优先级</td></tr>
<tr><td>networkTTL</td><td>1</td><td>网络中的消息和订阅可通过的最大 Broker 数（messageTTL + consumerTTL）</td></tr>
<tr><td>messageTTL</td><td>1</td><td>(版本5.9)消息在网络中可通过的最大 Broker 数</td></tr>
<tr><td>consumerTTL</td><td>1</td><td>(版本5.9)订阅在网络中可通过的最大 Broker 数</td></tr>
<tr><td>conduitSubscriptions</td><td>true</td><td>对同一个目标的多个订阅消费者会被网络当作一个消费者对待</td></tr>
<tr><td>excludedDestinations</td><td>empty</td><td>符合该列表的不会被允许穿过网络</td></tr>
<tr><td>dynamicallyIncludedDestinations </td><td>empty</td><td>符合该列表的允许穿过网络，配置为空表示只要不在 excluded 列表中的全允许</td></tr>
<tr><td>staticallyIncludedDestinations</td><td>empty</td><td>符合该列表的会一直允许穿过网络，即使没有消费者订阅</td></tr>
<tr><td>duplex</td><td>false</td><td>如果是 true，一个网络连接会同时被生产者和消费者使用，一般在中心位于防火墙后的星型拓扑被使用</td></tr>
<tr><td>prefetchSize</td><td>1000</td><td>设置网络连接器上的消费者的预取值，必须大于 0 因为网络消费者不进行消息轮询</td></tr>
<tr><td>suppressDuplicateQueueSubscriptions</td><td>false</td><td>(版本5.3+）如果为 true，从网络发起的重复的消息订阅会被忽略，设定有 Broker A、B、C，网络通过组播自发现，A上的一个消费者会发起到B、C的网络订阅。同时，C也会连接到B（基于来自A的网络消费者），B也会连接到C。当为 true 时，C和B之间的网络桥接（和他们已经存在的到达A的消息订阅重复）会被忽略。通过这种方式减少路由选择会增大死路由的可能，所以 networkTTL 也需要相应增大以作适应这种需求</td></tr>
<tr><td>bridgeTempDestinations</td><td>true</td><td>是否为网络中的临时队列而广播咨询消息。临时队列一般会为请求-应答消息创建。广播临时队列信息默认是打开的，这样一个请求-应答消息的消费者就可以在连接到网络中的其它Broker时仍能发送定义在 JMSReplyTo 头信息中的临时队列上的应答信息。在一个绝大部分都是请求-应答模式的应用场景中，这会产生大量额外流量，因为每条消息都会设置一个唯一的JMSReplyTo地址(这会导致一个新的临时目标地址通过咨询消息的广播被创建)。当禁用该特性会减少对应的这部分网络流量，不过也要求请求-应答消息的生产者消费者连接到同一个 Broker 上。远程消费者(通过网络中的另一个 Broker 连接的)也将会抛出「临时队列不存在」的异常而不能成功发送应答消息。</td></tr>
<tr><td>alwaysSyncSend</td><td>false</td><td>(版本5.6)设置为 true 时，非持久化的消息会通过请求/应答模式发送到远程 Broker。这个选项会使网络对持久消息和非持久消息等同看待。</td></tr>
<tr><td>staticBridge</td><td>false</td><td>(版本5.6)设置为 true 时，Broker 不会动态响应新的消费者。只能通过staticallyIncludedDestinations创建订阅需求。</td></tr>
</table>

### 可靠性 ###
Broker 网络会进行可靠的消息存储和转发。如果消息源是稳定的，队列上的消息是持久的或者是持久的主题订阅，这个网络将会继续保证这种稳定性。

然而当消息源是不可靠的时候网络本身不会增加可靠性。按定义非持久的主题订阅和临时主题（包括队列）是非持久的。当网络中存在不稳定的消息源时，出现故障时可能会导致消息丢失。

### 什么时候使用管道订阅 ###
ActiveMQ 依赖活动消费者(订阅)的相关信息进行网络内的消息传递。Broker 会把远程 Broker 上的订阅和本地订阅等同看待，并将相关的消息为每个订阅者路由一个拷贝。通过主题订阅和多于一个远程订阅，一个远程 Broker 会认为每个消息拷贝都是可用的，所以当它把消息路由到本地连接时，就出现了重复。因此默认的管道订阅会整合所有的订阅信息来制止网络内的重复发送。按这种方式，一个远程 Broker 上的多个订阅会被当作该网络 Broker 上的一个单独订阅。

然而，如果你只使用队列的话重复订阅是一个有用的特性。因为负载均衡算法会尝试均摊消息负载，且只有当标志<code>conduitSubscriptions</code>被设为<code>false</code>时才会进行网络内消费者的负载均摊。举个例子，假设现个Broker，A和B，通过转发桥相连。在A上你有一个消费者订阅了一个叫 Q.TEST 的队列。B上有两个消费者也订阅到 Q.TEST。所有的消费者具有相同的优先级。然后启动A上的一个生产者往 Q.TEST 里写入了30个消息。默认情况下(<code>conduitSubscriptions=true</code>)，15个消息会被发送到A上的消费者，其余的15个被发送到B上的两个消费者。消息负载没有在三个消费者之间均衡传播，因为默认情况下，A会把B上的两个消费者当做一个来处理。如果你把<code>conduitSubscriptions</code>设置为<code>false</code>，然后这三个消费者将均获得10个消息。

### 双向网络连接 ###
默认情况下一个网络桥接会通过一个连接按需要进行消息的单方向转发。当<code>dupex=true</code>时，同一个网络连接也会进行相反方向的网络桥接，也即一个双向桥。网络桥接的的配置会被传播到另一个 Broker 上，所以一个双向桥可能是一个确切的复本或者就是本身。

设定有两个Broker，A和B，那么 A到B的双向桥 = A到B的默认桥 + B到A的默认桥。

注意，如果你希望在两个 Broker 间配置多个双向桥来实现增大吞吐量或者区分对待主题和队列，那么你必须为每个桥提供唯一的名字：

	<networkConnectors>
        <networkConnector name="SYSTEM1" duplex="true" uri="static:(tcp://10.x.x.x:61616)">
            <dynamicallyIncludedDestinations>
                <topic physicalName="outgoing.System1" />
            </dynamicallyIncludedDestinations>
        </networkConnector>
        <networkConnector name="SYSTEM2" duplex="true" uri="static:(tcp://10.x.x.x:61616)">
            <dynamicallyIncludedDestinations>
                <topic physicalName="outgoing.System2"/>
            </dynamicallyIncludedDestinations>
        </networkConnector>
	</networkConnectors>

### 管道订阅和消费者选择器 ###
管道订阅会忽略本地 Broker 上的消费者选择器并将所有的消息发送的这个远程订阅上。这个远程 Broker 上的选择器会在消息分发到消费者之前进行消息处理。这种在 Broker 网络中的队列上进行基于选择器的消息处理的方式可能引发问题。想像一种情况，你有一个生产 Broker 转发消息到两个接收 Broker，并且每个接收 Broker 上有一个依赖不同选择器的消费者。因为生产者端没有选择器，最终可能所有的消息都到达了同一个目标 Broker，然后某些消息就可能不会被消费。如果你需要支持这种场景，请关掉<code>conduitSubscription</code>特性。

### 配置陷阱 ###
>如果 Broker 的<code>advisorySupport</code>属性被禁用了，那么 ActiveMQ 将不能按照预期的动态创建消费者运行。这时，一个完全的静态配置是唯一的选择。仔细阅读以下章节。

### Broker 网络和咨询消息 ###
Broker 网络非常依咨询消息，从原理上讲，一个远端的消费者进行消息订阅是通过它实现的。默认情况下，当网络连接器启动时会定义一个订阅了主题<code>ActiveMQ.Advisory.Consumer.> </code>的消费者(暂时忽略临时队列和主题)。通过这种方式，当消费者新建或者断开到远端 Broker 的连接时，本地 Broker 会收到提示并把它当做一个新的本地消费者对待。

### 动态网络 ###
现在开始动态网络配置。这意味着我们希望只在远程 Broker 上有订阅的消费者的时候才把消息发过去。如果我们想把这种行为只限制在几个特殊的队列或者主题可以使用<code>dynamicallyIncludedDestinations</code>，就像：

	<networkConnector uri="static:(tcp://host)">
	    <dynamicallyIncludedDestinations>
	        <queue physicalName="include.test.foo"/>
	        <topic physicalName="include.test.bar"/>
	    </dynamicallyIncludedDestinations>
	</networkConnector>
在 ActiveMQ 的早期版本中，远程 Broker 上的所有消费者都需要相同的咨询过滤器。在消息分发时进行消息过滤。在一个巨大的网络中这不是一个最优的解决方案，因为它创建了大量的「advisory」流量和负载。从版本5.6起， Broker 会自动创建适当的咨询过滤器并且只处理在<code>dynamicallyIncludedDestination</code>中的队列和主题。在上面的例子中就是<code>ActiveMQ.Advisory.Consumer.Queue.include.test.foo</code>、<code>ActiveMQ.Advisory.Consumer.Topic.include.test.bar</code>。这将显著提高在复杂和高负载环境中的处理效率。

那么在老版本中的 Broker 上我们需要怎么做呢？幸运的是，我们可以通过稍复杂的配置做到相同的事情。实际上的咨询过滤器对哪些队列和主题感兴趣是通过连接器上的<code>destinationFilter</code>属性修改的。它的默认值是<code>></code>，通过<code>ActiveMQ.Advisory.Consumer.</code>前缀串联。所以同样的例子需要这么干：

	<networkConnector uri="static:(tcp://host)" 
			destinationFilter="Queue.include.test.foo,
			ActiveMQ.Advisory.Consumer.Topic.include.test.bar">
	  	<dynamicallyIncludedDestinations>
		    <queue physicalName="include.test.foo"/>
		    <topic physicalName="include.test.bar"/>
	  	</dynamicallyIncludedDestinations>
	</networkConnector>

注意第一个没有前缀因为它已经默认存在了。这种方式在设置和维护时会稍麻烦点，不过它也可以工作。如新你在使用 5.6 或者更新的版本你只需要添加到<code>dynamicallyIncludedDestinations </code>中就可以了。

这同样也解释了为什么动态网络中关闭咨询消息时不能正常工作，即在这种情况下 Broker 不能对消费者进行动态应答。

### 完全的静态网络 ###
如果你希望完全保护 Broker 不受任何远程 Broker 上的消费者的影响，或者你希望把 Broker 当做一个简单代理并不论是不是有消费者都进行全部消息的转发，静态网络或许是你该考虑的。

	<networkConnector uri="static:(tcp://host)" staticBridge="true">
	    <staticallyIncludedDestinations>
	        <queue physicalName="always.include.queue"/>
	    </staticallyIncludedDestinations>
	</networkConnector>

属性<code>staticBridge</code>从版本5.6开始可用，它代表 Broker 不订阅远程 Broker 上的任何咨询主题，代表它不对存在的任何消费者感兴趣。另外，你需要在<code>staticallyIncludedDestinations</code>中添加一串地址定义。这和在该队列上有一个额外的消费者作用类似，只有这些消息会被正常的转发到远程 Broker 上。由于在 ActiveMQ 的早期版本中没有<code>staticBridge</code>参数，你可以将这个 Broker 的<code>destinationFilter</code>设置到一个没用的咨询主题，就像

	<networkConnector uri="static:(tcp://host)" destinationFilter="NO_DESTINATION">
        <staticallyIncludedDestinations>
            <queue physicalName="always.include.queue"/>
        </staticallyIncludedDestinations>
	</networkConnector>

如果这样配置了，Broker 会尝试监听<code>ActiveMQ.Advisory.Consumer.NO_DESTINATION</code>上的新消费者，由于它一直不会有消息，也就不会收到远程 Broker 上的消费者信息。

### 使用 NetworkConnector 属性的例子 ###

    <networkConnectors>
        <networkConnector uri="static:(tcp://localhost:61617)" 
				name="bridge" conduitSubscriptions="true" 
				decreaseNetworkConsumerPriority="false">
	        <dynamicallyIncludedDestinations>
	            <queue physicalName="include.test.foo"/>
	            <topic physicalName="include.test.bar"/>
	        </dynamicallyIncludedDestinations>
	        <excludedDestinations>
	            <queue physicalName="exclude.test.foo"/>
	            <topic physicalName="exclude.test.bar"/>
	        </excludedDestinations>
	        <staticallyIncludedDestinations>
	            <queue physicalName="always.include.queue"/>
	            <topic physicalName="always.include.topic"/>
	        </staticallyIncludedDestinations>
        </networkConnector>
    </networkConnectors>

两个 Broker 间可以有多个网络连接器。每个网络连接器都会使用一个单独的底层的传输连接，所以你可能希望通过这么做来增大吞吐量，或者实现一个更灵活的配置。

例如，如果使用分布式队列，你可能希望只在接收者可用时，实现网络内消息接收的同权重处理。如

	<networkConnectors>
        <networkConnector uri="static:(tcp://localhost:61617)" 
				name="queues_only" conduitSubscriptions="false" 
				decreaseNetworkConsumerPriority="false">
            <excludedDestinations>
        	    <topic physicalName=">"/>
        	</excludedDestinations>
        </networkConnector>
    </networkConnectors>

**注意：**在excludedDestinations和dynamicallyIncludedDestinations属性中你只可以使用[wildcards](http://activemq.apache.org/wildcards.html)。<br />
**注意：****不要**改变桥的名字或者Broker的名字如果你正在网络中使用持久化的主题订阅。因为 ActiveMQ 内部会使用网络名和Broker名为网络产生一个唯一的并且重复可用的订阅名。

### 被卡住的信息（版本5.6） ###
默认情况下，一个消息被重新发回到它的来源 Broker 是不被允许的。这可以保证消息不会在复杂网络拓扑中形成回路。在某些情况下，可能真的需要将消息发回源队列。想像一种情况下两个 Broker 之间有个双向桥。生产者和消费者各自采用 failover 连接到随机的一个上。如果一个 Broker 因为维护重启了，那么通过网络传播累积在这个 Broker 上的消息将在消费者连接到它之前是不可用的。这个问题的一种解决方案是强制一个客户端采用[rebalanceClusterClients](http://activemq.apache.org/failover-transport-reference.html#FailoverTransportReference-BrokersideOptionsforFailover)进行重连。另一种方案是允许消息在本地没有消费者时被发回到源队列。

通过设置一个带有<code>replayWhenNoConsumers=true</code>的<code>conditionalNetworkBridgeFilterFactory</code>可以让队列实现上述行为。这个<code>conditionalNetworkBridgeFilterFactory</code>可以提供一个基于 Broker 时间的发回延时配置。

	<destinationPolicy>
		<policyMap>
		    <policyEntries>
		        <policyEntry queue="TEST.>" enableAudit="false">
		        	<conditionalNetworkBridgeFilterFactory replayWhenNoConsumers="true"/>
		        </policyEntry>
		    </policyEntries>
	    </policyMap>
	</destinationPolicy>

**注意：**在5.9之前的版本中使用<code>replayWhenNoConsumers=true</code>时，还需要通过<code>enableAudit=false</code>关闭指针重复检测，因为它可能会认为发回的消息是重复的（取决于网络桥在接收和传回消息时的时间差）。在这篇[博客](http://tmielke.blogspot.de/2012/03/i-have-messages-on-queue-but-they-dont.html)中有详细解释。

### 网络消费者吞吐限制 ###
<code>conditionalNetworkBridgeFilterFactory</code>允许为特定的队列和主题设定一个比例限制，然后网络上的消费者就可以进行吞吐量限制了。预抓取在网络消费者处理消息相当快的时候可以说是没用的，这时即时采用极小的预取和更低的优先级都不能避免本地消费者接收不到消息。吞吐量限制可以解决这种问题。

## 开始运行一个 Broker 网络 ##
如果你在不同的命令行窗口中执行下列命令你就可以得到两个自检测的 Broker 和两个固定地址的客户端

	maven -o server -Dconfig=xbean:file:src/test/resources/org/apache/activemq/usecases/receiver.xml
	maven -o server -Dconfig=xbean:file:src/test/resources/org/apache/activemq/usecases/sender.xml
	maven -o consumer -Durl=tcp://localhost:62002
	maven -o producer -Durl=tcp://localhost:62001
或者也可以使用零配置策略完成相同的事情就像这样

	maven -o server -Dconfig=src/test/org/activemq/usecases/receiver-zeroconf.xml
	maven -o server -Dconfig=src/test/org/activemq/usecases/sender-zeroconf.xml
	maven -o consumer -Durl=tcp://localhost:62002
	maven -o producer -Durl=tcp://localhost:62001

转载注明出处：[{{page.title}}]({{permalink}})