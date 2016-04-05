---
layout: post
title: Netty代码分析
date: 2012-06-11 21:45
author: onecoder
comments: true
categories: [Netty, Netty, 分布式, 分析]
---
<p class="Code">
	Netty提供异步的、事件驱动的网络应用程序框架和工具，用以快速开发高性能、高可靠性的网络服务器和客户端程序[官方定义]，整体来看其包含了以下内容：1.提供了丰富的协议编解码支持，2.实现自有的buffer系统，减少复制所带来的消耗，3.整套channel的实现，4.基于事件的过程流转以及完整的网络事件响应与扩展，5.丰富的example。本文并不对Netty实际使用中可能出现的问题做分析，只是从代码角度分析它的架构以及实现上的一些关键细节。</p>
首先来看下最如何使用Netty(其自带example很好展示了使用)，Netty普通使用一般是通过BootStrap来启动，BootStrap主要分为两类：1.面向连接(TCP)的BootStrap(ClientBootStrap和ServerBootstrap),2.非面向连接(UDP) 的(ConnectionlessBootstrap)。<br />
Netty整体架构很清晰的分成2个部分，ChannelFactory 和ChannelPipelineFactory,前者主要生产网络通信相关的Channel实例和ChannelSink实例，Netty提供的 ChannelFactory实现基本能够满足绝大部分用户的需求，当然你也可以定制自己的ChannelFactory,后者主要关注于具体传输数据的处理，同时也包括其他方面的内容，比如异常处理等等，只要是你希望的，你都可以往里添加相应的handler,一般 ChannelPipelineFactory由用户自己实现，因为传输数据的处理及其他操作和业务关联比较紧密，需要自定义处理的handler。
<p>
	现在，使用Netty的步骤实际上已经非常明确了，比如面向连接的Netty服务端客户端使用，第一步：实例化一个BootStrap,并且通过构造方法指定一个ChannelFactory实现，第二步：向bootstrap实例注册一个自己实现的ChannelPipelineFactory,第三步：如果是服务器端，bootstrap.bind(new InetSocketAddress(port))，然后等待客户端来连接,如果是客户端，bootstrap.connect(new InetSocketAddress(host,port))取得一个future,这个时候Netty会去连接远程主机，在连接完成后，会发起类型为 CONNECTED的ChannelStateEvent，并且开始在你自定义的Pipeline里面流转，如果你注册的handler有这个事件的响应方法的话那么就会调用到这个方法。在此之后就是数据的传输了。下面是一个简单客户端的代码解读。</p>
<pre class="brush:java;first-line:1;pad-line-numbers:true;highlight:null;collapse:false;">
// 实例化一个客户端Bootstrap实例，其中NioClientSocketChannelFactory实例由Netty提供

ClientBootstrap bootstrap = new ClientBootstrap(

new NioClientSocketChannelFactory(

Executors.newCachedThreadPool(),

Executors.newCachedThreadPool()));

// 设置PipelineFactory,由客户端自己实现

bootstrap.setPipelineFactory(new FactorialClientPipelineFactory(count));

//向目标地址发起一个连接

ChannelFuture connectFuture =

bootstrap.connect(new InetSocketAddress(host, port));

// 等待链接成功，成功后发起的connected事件将会使handler开始发送信息并且等待messageRecive,当然这只是示例。

Channel channel = connectFuture.awaitUninterruptibly().getChannel();

// 得到用户自定义的handler

FactorialClientHandler handler =

(FactorialClientHandler) channel.getPipeline().getLast();

// 从handler里面取数据并且打印，这里需要注意的是，handler.getFactorial使用了从结果队列result take数据的阻塞方法，而结果队列会在messageRecieve事件发生时被填充接收回来的数据

System.err.format(

&quot;Factorial of %,d is: %,d&quot;, count, handler.getFactorial());
</pre>
<p>
	&nbsp;</p>
Netty提供了NIO与BIO(OIO)两种模式处理这些逻辑，其中NIO主要通过一个BOSS线程处理等待链接的接入，若干个WORKER线程(从worker线程池中挑选一个赋给Channel实例，因为Channel实例持有真正的 java网络对象)接过BOSS线程递交过来的CHANNEL进行数据读写并且触发相应事件传递给pipeline进行数据处理，而BIO(OIO)方式服务器端虽然还是通过一个BOSS线程来处理等待链接的接入，但是客户端是由主线程直接connect,另外写数据C/S两端都是直接主线程写，而数据读操作是通过一个WORKER 线程BLOCK方式读取(一直等待，直到读到数据，除非channel关闭)。<br />
网络动作归结到最简单就是服务器端bind-&gt;accept-&gt;read-&gt;write,客户端 connect-&gt;read-&gt;write,一般bind或者connect后会有多次read、write。这种特性导致，bind,accept与read,write的线程分离，connect与read、write线程分离，这样做的好处就是无论是服务器端还是客户端吞吐量将有效增大，以便充分利用机器的处理能力，而不是卡在网络连接上，不过一旦机器处理能力充分利用后，这种方式反而可能会因为过于频繁的线程切换导致性能损失而得不偿失，并且这种处理模型复杂度比较高。<br />
采用什么样的网络事件响应处理机制对于网络吞吐量是非常重要的，Netty采用的是标准的SEDA（Staged Event-Driven Architecture）架构[http://en.wikipedia.org/wiki/ Staged_event-driven_architecture]，其所设计的事件类型，代表了网络交互的各个阶段，并且在每个阶段发生时，触发相应事件交给初始化时生成的pipeline实例进行处理。事件处理都是通过Channels类的静态方法调用开始的，将事件、channel传递给 channel持有的Pipeline进行处理，Channels类几乎所有方法都为静态，提供一种Proxy的效果(整个工程里无论何时何地都可以调用其静态方法触发固定的事件流转,但其本身并不关注具体的处理流程)。<br />
Channels部分事件流转静态方法 1．fireChannelOpen 2．fireChannelBound 3．fireChannelConnected 4．fireMessageReceived 5．fireWriteComplete 6．fireChannelInterestChanged 7．fireChannelDisconnected 8．fireChannelUnbound 9．fireChannelClosed 10.fireExceptionCaught 11.fireChildChannelStateChanged。
<p>
	Netty提供了全面而又丰富的网络事件类型，其将java中的网络事件分为了两种类型Upstream和Downstream。一般来说，Upstream类型的事件主要是由网络底层反馈给Netty的，比如messageReceived,channelConnected等事件，而Downstream类型的事件是由框架自己发起的，比如bind,write,connect,close等事件。 <img alt="" src="http://www.blogjava.net/images/blogjava_net/bucketli/EVENT.jpg" /> Netty的Upstream和Downstream网络事件类型特性也使一个Handler分为了3种类型，专门处理Upstream,专门处理Downstream,同时处理Upstream,Downstream。实现方式是某个具体Handler通过继承ChannelUpstreamHandler和ChannelDownstreamHandler类来进行区分。PipeLine在Downstream或者Upstream类型的网络事件发生时，会调用匹配事件类型的Handler响应这种调用。ChannelPipeline维持有所有handler有序链表，并且由handler自身控制是否继续流转到下一个handler(ctx.sendDownstream(e),这样设计有个好处就是随时终止流转，业务目的达到无需继续流转到下一个handler)。下面的代码是取得下一个处理Downstream事件的处理器。</p>
<pre class="brush:java;first-line:1;pad-line-numbers:true;highlight:null;collapse:false;">
DefaultChannelHandlerContext realCtx = ctx;

while (!realCtx.canHandleUpstream()) {

realCtx = realCtx.next;

if (realCtx == null) {

return null;

}

}

return realCtx;
</pre>
如果是一个网络会话最末端的事件，比如messageRecieve，那么可能在某个handler里面就直接结束整个会话，并把数据交给上层应用，但是如果是网络会话的中途事件，比如connect事件，那么当触发connect事件时，经过pipeline流转，最终会到达挂载pipeline最底下的ChannelSink实例中，这类实例主要作用就是发送请求和接收请求，以及数据的读写操作。 <img alt="" src="http://www.blogjava.net/images/blogjava_net/bucketli/SINK.jpg" /> NIO方式ChannelSink一般会有1个BOSS实例(implements Runnable)，以及若干个worker实例（不设置默认为cpu cores*2个worker），这在前面已经提起过，BOSS线程在客户端类型的ChannelSink和服务器端类型的ChannelSink触发条件不一样，客户端类型的BOSS线程是在发生connect事件时启动，主要监听connect是否成功，如果成功，将启动一个worker线程,将connected的channel交给这个线程继续下面的工作，而服务器端的BOSS线程是发生在bind事件时启动，它的工作也相对比较简单，对于channel.socket().accept()进来的请求向Nioworker进行工作分配即可。这里需要提到的是，Server端ChannelSink实现比较特别，无论是NioServerSocketPipelineSink 还是OioServerSocketPipelineSink的eventSunk方法实现都将channel分为 ServerSocketChannel和SocketChannel分开处理。这主要原因是Boss线程accept()一个新的连接生成一个 SocketChannel交给Worker进行数据接收。<br />
<pre class="brush:java;first-line:1;pad-line-numbers:true;highlight:null;collapse:false;">
public void eventSunk(

ChannelPipeline pipeline, ChannelEvent e) throws Exception {

Channel channel = e.getChannel();

if (channel instanceof NioServerSocketChannel) {

handleServerSocket(e);

} else if (channel instanceof NioSocketChannel) {

handleAcceptedSocket(e);

}

}

NioWorker worker = nextWorker();

worker.register(new NioAcceptedSocketChannel(

channel.getFactory(), pipeline, channel,

NioServerSocketPipelineSink.this, acceptedSocket,

worker, currentThread), null);
</pre>
<br />
另外两者实例化时都会走一遍如下流程：&nbsp;<br />
<br />
<pre class="brush:java;first-line:1;pad-line-numbers:true;highlight:null;collapse:false;">
setConnected();
fireChannelOpen(this);
fireChannelBound(this, getLocalAddress());
fireChannelConnected(this, getRemoteAddress());</pre>
<br />
而对应的ChannelSink里面的处理代码就不同于ServerSocketChannel了，因为走的是 handleAcceptedSocket（e）这一块代码，从默认实现代码来说,实例化调用 fireChannelOpen（this）;fireChannelBound（this,getLocalAddress()）;<br />
fireChannelConnected（this,getRemoteAddress（））没有什么意义，但是对于自己实现的ChannelSink有着特殊意义。具体的用途我没去了解，但是可以让用户插手Server accept连接到准备读写数据这一个过程的处理。<br />
<pre class="brush:java;first-line:1;pad-line-numbers:true;highlight:null;collapse:false;">
switch (state) {

case OPEN:

if (Boolean.FALSE.equals(value)) {

channel.worker.close(channel, future);

}

break;

case BOUND:

case CONNECTED:

if (value == null) {

channel.worker.close(channel, future);

}

break;

case INTEREST_OPS:

channel.worker.setInterestOps(channel, future, ((Integer) value).intValue());

break;

}
</pre>
<p>
	Netty提供了大量的handler来处理网络数据，但是大部分是CODEC相关的，以便支持多种协议，下面一个图绘制了现阶段Netty提供的Handlers(红色部分不完全) <img alt="" src="http://www.blogjava.net/images/blogjava_net/bucketli/HANDLER_small.JPG" /> Netty实现封装实现了自己的一套ByteBuffer系统，这个ByteBuffer系统对外统一的接口就是ChannelBuffer,这个接口从整体上来说定义了两类方法，一种是类似getXXX(int index&hellip;)，setXXX(int index&hellip;)需要指定开始操作buffer的起始位置，简单点来说就是直接操作底层buffer,并不用到Netty特有的高可重用性buffer特性，所以Netty内部对于这类方法调用非常少，另外一种是类似readXXX(),writeXXX()不需要指定位置的buffer操作，这类方法实现放在了AbstractChannelBuffer，其主要的特性就是维持buffer的位置信息，包括readerIndex,writerIndex,以及回溯作用的markedReaderIndex和markedWriterIndex，当用户调用readXXX()或者writeXXX()方法时，AbstractChannelBuffer会根据维护的readerIndex,writerIndex计算出读取位置，然后调用继承自己的ChannelBuffer的getXXX(int index&hellip;)或者setXXX(int index&hellip;)方法返回结果，这类方法在Netty内部被大量调用，因为这个特性最大的好处就是很方便地重用buffer而不必去费心费力维护index或者新建大量的ByteBuffer。</p>
另外WrappedChannelBuffer接口提供的是对ChannelBuffer的代理，他的用途说白了就是重用底层buffer，但是会转换一些buffer的角色，比如原本是读写皆可 ，wrap成ReadOnlyChannelBuffer,那么整个buffer只能使用readXXX()或者getXXX()方法，也就是只读，然后底层的buffer还是原来那个,再如一个已经进行过读写的ChannelBuffer被wrap成TruncatedChannelBuffer，那么新的buffer将会忽略掉被wrap的buffer内数据，并且可以指定新的writeIndex,相当于slice功能。 <img alt="" src="http://www.blogjava.net/images/blogjava_net/bucketli/BUFFER.jpg" /> Netty实现了自己的一套完整Channel系统，这个channel说实在也是对java 网络做了一层封装，加上了SEDA特性(基于事件响应，异步，多线程等)。其最终的网络通信还是依靠底下的java网络api。提到异步，不得不提到Netty的Future系统，从channel的定义来说，write,bind,connect,disconnect,unbind,close,甚至包括setInterestOps等方法都会返回一个channelFuture，这这些方法调用都会触发相关网络事件，并且在pipeline中流转。Channel很多方法调用基本上不会马上就执行到最底层，而是触发事件，在pipeline中走一圈，最后才在channelsink中执行相关操作，如果涉及网络操作，那么最终调用会回到Channel中，也就是serversocketchannel,socketchannel,serversocket,socket等java原生网络api的调用，而这些实例就是jboss实现的channel所持有的(部分channel)。 <img alt="" src="http://www.blogjava.net/images/blogjava_net/bucketli/CHANNEL_small.JPG" /> Netty新版本出现了一个特性zero-copy,这个机制可以使文件内容直接传输到相应channel上而不需要通过cpu参与，也就少了一次内存复制。Netty内部ChunkedFile 和 FileRegion 构成了non zero-copy 和zero-copy两种形式的文件内容传输机制，前者需要CPU参与，后者根据操作系统是否支持zero-copy将文件数据传输到特定channel,如果操作系统支持，不需要cpu参与，从而少了一次内存复制。ChunkedFile主要使用file的read,readFully等API，而FileRegion使用FileChannel的transferTo API，2者实现并不复杂。Zero-copy的特性还是得看操作系统的，本身代码没有很大的特别之处。
<p>
	最后总结下，Netty的架构思想和细节可以说让人眼前一亮，对于java网络IO的各个注意点，可以说Netty已经解决得比较完全了，同时Netty 的作者也是另外一个NIO框架MINA的作者，在实际使用中积累了丰富的经验，但是本文也只是一个新手对于Netty的初步理解，还没有足够的能力指出某一细节的所发挥的作用。</p>
<cite>转自：<a href="http://rdc.taobao.com/team/jm/archives/423">http://rdc.taobao.com/team/jm/archives/423</a></cite>
