# Preface

## The Problem
Nowadays we use general purpose applications or libraries to communicate with each other. For example, we often use an HTTP client library to retrieve information from a web server and to invoke a remote procedure call via web services. However, a general purpose protocol or its implementation sometimes does not scale very well. It is like how we don't use a general purpose HTTP server to exchange huge files, e-mail messages, and near-realtime messages such as financial information and multiplayer game data. What's required is a highly optimized protocol implementation that is dedicated to a special purpose. For example, you might want to implement an HTTP server that is optimized for AJAX-based chat application, media streaming, or large file transfer. You could even want to design and implement a whole new protocol that is precisely tailored to your need. Another inevitable case is when you have to deal with a legacy proprietary protocol to ensure the interoperability with an old system. What matters in this case is how quickly we can implement that protocol while not sacrificing the stability and performance of the resulting application.

如今我们很常用使用一些特制的应用和库在彼此之间练习。比如，我们通常使用http协议的客户端库来接收来自web服务器的消息；实现web服务的远程过程调用。然而，一种通用的协议或者它的实现有些时候并不能表现的很好。这就好比为什么我们不使用通用的HTTP协议的服务去完成大文件交换，电子邮件，和接近实时的信息,如财务有限公司，和多人游戏数据。
需要的是一个高度优化的协议来满足特殊的目的。比如，你想实现一个HTTP服务器用来完成基于AJAX的聊天系统，流媒体和大文件传输。你甚至想实现并实现一个全新的协议来满足你的需要。当然还有不可避免的情况就是你需要在新的协议中依旧兼容旧的协议以满足旧的逻辑。155/5000
在这种情况下，重要的是在不牺牲结果应用程序的稳定性和性能的情况下，我们可以多快地实现该协议。



## The Solution

The Netty project is an effort to provide an asynchronous event-driven network application framework and tooling for the rapid development of maintainable high-performance and high-scalability protocol servers and clients.

Netty这个项目擅于提供一个异步的事件驱动的网络框架和工具，用于快速开发可维护的、高性能的、高扩展的协议服务器和客户端。

In other words, Netty is an NIO client server framework that enables quick and easy development of network applications such as protocol servers and clients. It greatly simplifies and streamlines network programming such as TCP and UDP socket server development.

换句话说，Netty是一个NIO模型的客户服务框架，可以快速简单的开发一个协议的服务端和客户端。它极大地简化和简化了网络编程，如TCP和UDP套接字服务器的开发。它极大地简化和简化了网络编程，如TCP和UDP套接字服务器的开发。


'Quick and easy' does not mean that a resulting application will suffer from a maintainability or a performance issue. 
“快速和简单”并不意味着最终的应用程序将遭受可维护性或性能问题。


Netty has been designed carefully with the experiences learned from the implementation of a lot of protocols such as FTP, SMTP, HTTP, and various binary and text-based legacy protocols. As a result, Netty has succeeded to find a way to achieve ease of development, performance, stability, and flexibility without a compromise.
Netty经过精心的设计，使用了从许多协议(如FTP、SMTP、HTTP和各种基于二进制和文本的遗留协议)的实现中获得的经验。因此，Netty成功地找到了一种方法，在不妥协的情况下，实现了易于开发、性能、稳定性和灵活性。


## Writing a Discard Server
最简单的协议不是"Hello,World",而是Discard。那么什么是Discard？就是直接抛弃任何它接收到的消息。
```java
package io.netty.example.discard;

import io.netty.buffer.ByteBuf;

import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;

/**
 * Handles a server-side channel.
 */
public class DiscardServerHandler extends ChannelInboundHandlerAdapter { // (1)

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) { // (2)
        // Discard the received data silently.
        ((ByteBuf) msg).release(); // (3)
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) { // (4)
        // Close the connection when an exception is raised.
        cause.printStackTrace();
        ctx.close();
    }
}

```
1. DiscardServerHandler继承ChannelInboundHandlerAdapter。其中ChannelInboundHandlerAdapter是 ChannelInboundHandler.ChannelInboundHandler的实现类。ChannelInboundHandler.ChannelInboundHandler提供类很多事件处理方法同时我们也可以重写这些方法。
同时现在仅仅继承ChannelInboundHandlerAdapter就够了，而不用自己实现这些handler接口。
2. 我们重写了channelRead()事件的处理方法。这个方法在有消息被接收的时候被调用。在这个案例里面我们接收的消息是一个ByteBuf
3. 实现DISCARD协议的处理器会忽略接收到的消息。ByteBuf是一个引用计数对象，它会被释放在release()方法被调用的时候。请记住，释放传递给处理程序的引用计数对象是处理程序的责任。
通常情况下channelRead()的处理方法被实现像如下：
```java
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    try {
        // Do something with msg
    } finally {
        ReferenceCountUtil.release(msg);
    }
}
```
4. exceptionCaught()方法会被Throwable调用，当netty出现IO错误或者实现的handler在处理事件的过程中抛出了异常。
在大多数情况下，扑获的异常应该被log同时它所关联的channel应该被关闭。但是这些方法有不同的实现都取决于你想怎么进行处理在异常的情况下。比如，遇到异常的时候你想返回一个错误的代码然后在关闭链接。
   
综上所诉，我们明白了一个DISCARD的主要业务，接下来我们继续写DISCARD的main()来启动我的服务器。

如下代码：
```java
package io.netty.example.discard;
    
import io.netty.bootstrap.ServerBootstrap;

import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
    
/**
 * Discards any incoming data.
 */
public class DiscardServer {
    
    private int port;
    
    public DiscardServer(int port) {
        this.port = port;
    }
    
    public void run() throws Exception {
        EventLoopGroup bossGroup = new NioEventLoopGroup(); // (1)
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap b = new ServerBootstrap(); // (2)
            b.group(bossGroup, workerGroup)
             .channel(NioServerSocketChannel.class) // (3)
             .childHandler(new ChannelInitializer<SocketChannel>() { // (4)
                 @Override
                 public void initChannel(SocketChannel ch) throws Exception {
                     ch.pipeline().addLast(new DiscardServerHandler());
                 }
             })
             .option(ChannelOption.SO_BACKLOG, 128)          // (5)
             .childOption(ChannelOption.SO_KEEPALIVE, true); // (6)
    
            // Bind and start to accept incoming connections.
            ChannelFuture f = b.bind(port).sync(); // (7)
    
            // Wait until the server socket is closed.
            // In this example, this does not happen, but you can do that to gracefully
            // shut down your server.
            f.channel().closeFuture().sync();
        } finally {
            workerGroup.shutdownGracefully();
            bossGroup.shutdownGracefully();
        }
    }
    
    public static void main(String[] args) throws Exception {
        int port = 8080;
        if (args.length > 0) {
            port = Integer.parseInt(args[0]);
        }

        new DiscardServer(port).run();
    }
}
```
1. NioEventLoopGroup 是一个多线程的事件循环，负责I/O处理。Netty针对不同的传输协议提供了很多EventLoopGroup的实现。
在这个案例中我们正在实现的是服务端的应用，在这里会有两个NioEventLoopGroup会被用到。第一个是被称为'Boos'负责接收所有进入的连接。
第二个通常被称为'worker'负责boos接受到的所有的连接的传输，以及注册接受连接到worker。有多少线程被使用以及这些线程是
如何映射到对应的Channel中的都是取决于EventLoopGroup的实现，同时这个配置也可以同时构造函数进行设置。
   
   
2. ServerBootstrap 是一个设置服务的帮组类。可以使用Channel直接设置server，但是这样做有点荒诞，大多数情况都不推荐这样做。


3. 在这里我们特地使用了NioServerSocketChannel类来实例化一个新的Channel来接受进入的连接。


4. 特殊的处理器会被新接受的Channel调用。ChannelInitializer是一个特别的处理器可以帮助用户设置设置一个新的Channel。
正如你可以通过添加好几个handlers到新channel的ChannelPipeline来满足程序的业务逻辑功能。随着程序越来越复杂，通常你会
添加更多handlers到pipeline或者继承这个匿名类来达到效果。
   
   
5. 你也可以设置特殊的参数到Channel的实现。我们正在写的是TCP/IP的服务，所以我们允许设置socket的参数比如：tcpNoDelay和KeepAlive
如果需要了解更多的详细的信息可以参考ChannelOption的apidocs文档和ChannelConfig的实现介绍。
   
   
6. 你有没有注意到这里有option()和childOption()。option()是对于进入的新接受的NioServerSocketChannel。childOption()是对于父亲ServerChannel接受的Channel，
在这个案列中是NioSockerChannel。
   

## 接收数据

截止到目前我们已经完成了第一个server。我们需要测试它是不是真的可以工作。这里最简单的测试方法就是使用telnet命令。
例如可以使用
```shell
telnet ip port
> telnet 127.0.0.1 8080

```
但是我们可以看见这个服务是不是好的了？我们不能准缺的知道它是不是正常的服务因为他是一个discard服务，会丢失所有的连接。不会得到任何的响应。
为了证实他是在工作的我们需要修改我们的服务让他打印出它接受的内容。




我们已经知道了当数据被接受的时候channelRead()方法会被调用。让我们放一些代码到DiscardServerHandler的channelRead()的方法里面。
```java
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    ByteBuf in = (ByteBuf) msg;
    try {
        while (in.isReadable()) { // (1)
            System.out.print((char) in.readByte());
            System.out.flush();
        }
    } finally {
        ReferenceCountUtil.release(msg); // (2)
    }
}
```
1. 这个低效的循环实际上可以转换为
```java

System.out.println(in.toString(io.netty.util.CharsetUtil.US_ASCII))
```
2. 然后，你可以在finally里面使用
```java
in.release()
```

在修改之后，如果你在次使用telnet命令，你会看见服务会打印出它接受到的消息。






