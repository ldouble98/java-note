#netty
Netty is an asynchronous event-driven network application framework
for rapid development of maintainable high performance protocol servers & clients.
## 如何使用
关键在编写ServerInitializer和clientInitializer。

在其中会申明调用的handler以及调用的顺序。

在handler中我们会自定义自己的处理逻辑。分别继承SimpleChannelInboundHandler和ChannelOutboundHandlerAdapter
在读入和写出时分别调用不同子类的处理逻辑。每一个连接到来时都对应一个pipeline，互相不影响。

在netty中 ByteBuf 和nio 中ByteBuffer是不同的概念。

当需要写出时，推荐使用堆外内存

- pipeline

```java
package demo1;
import io.netty.channel.Channel;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.socket.SocketChannel;
import io.netty.handler.codec.http.HttpServerCodec;
import io.netty.handler.codec.http2.Http2ChannelDuplexHandler;
/**
 * @Author: small_double
 * @Date: 2019/10/2 上午11:01
 */
public class TestServerInitializer extends ChannelInitializer<SocketChannel> {

    @Override
    protected void initChannel(SocketChannel socketChannel) throws Exception {
        ChannelPipeline channelPipeline = socketChannel.pipeline();
        channelPipeline.addLast("httpServerCodec",new HttpServerCodec());
        channelPipeline.addLast("testServerHandler", new TestServerHandler());
    }
}

```

- Handler
```java
package demo1;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.handler.codec.http.DefaultFullHttpResponse;
import io.netty.handler.codec.http.FullHttpMessage;
import io.netty.handler.codec.http.FullHttpResponse;
import io.netty.handler.codec.http.HttpHeaderNames;
import io.netty.handler.codec.http.HttpObject;
import io.netty.handler.codec.http.HttpRequest;
import io.netty.handler.codec.http.HttpResponseStatus;
import io.netty.handler.codec.http.HttpVersion;
import io.netty.util.CharsetUtil;

import java.nio.ByteBuffer;
import java.nio.charset.Charset;

/**
 * @Author: small_double
 * @Date: 2019/10/4 上午9:19
 */
public class TestServerHandler extends SimpleChannelInboundHandler<HttpObject> {

    @Override
    protected void channelRead0(ChannelHandlerContext channelHandlerContext, HttpObject httpObject) throws Exception {
        if (httpObject instanceof HttpRequest) {
            HttpRequest httpRequest = (HttpRequest) httpObject;
            String uri = httpRequest.uri();
            System.out.println(uri);
            ByteBuf byteBuf = Unpooled.copiedBuffer("hello world", CharsetUtil.UTF_8);
            HttpVersion version;
            HttpResponseStatus status;
            FullHttpResponse fullHttpResponse  = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.OK,byteBuf);
            fullHttpResponse.headers().set(HttpHeaderNames.CONTENT_TYPE,"text/plain");
            fullHttpResponse.headers().set(HttpHeaderNames.CONTENT_LENGTH,byteBuf.readableBytes());
            channelHandlerContext.writeAndFlush(fullHttpResponse);
        }
    }
}

```

- TestServer 基本是固定写法

```java
package demo1;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.Channel;
import io.netty.channel.ChannelFuture;
import io.netty.channel.EventLoop;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;

/**
 * @Author: small_double
 * @Date: 2019/10/2 上午10:46
 */
public class TestServer {
    public static void main(String[] args) {
        EventLoopGroup bossGroup=new NioEventLoopGroup(1);  //接收客户端连接的线程组
                EventLoopGroup workGroup=new NioEventLoopGroup(); //真正处理读写事件的线程组   16
                try {
                    ServerBootstrap serverBootstrap=new ServerBootstrap();
                    serverBootstrap.group(bossGroup,workGroup)
                            .channel(NioServerSocketChannel.class)  //服务端用什么通道
                            .childHandler(new TestServerLnitializer()); //已经连接上来的客户端
                    ChannelFuture channelFuture = serverBootstrap.bind(8989).sync();
                    channelFuture.channel().closeFuture().sync();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    bossGroup.shutdownGracefully();
                    workGroup.shutdownGracefully();
                }

    }
}

```

## 原理极其源码笔记

