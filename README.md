# Netty-Demo
First example-2019年5月10日09:16:11-客户端和服务端实现简单通信

Thrid example-2019年5月11日15:27:42-多个客户端与服务端实现广播通信

## 1. Netty-Demo 多个客户端与服务端实现广播通信
重点内容

### 1. channelGroup
   存储多个连接channel,根据channelGroup实现与多个客户端发送消息,具体业务代码
   
      channelGroup.forEach(ch->{
            if (channel!=ch) {
                ch.writeAndFlush(channel.remoteAddress() + "[别人发送的消息]" + msg+"\n");
            }else {
                ch.writeAndFlush( "[自己发送的消息]" + msg+"\n");
            }
        });
### 2. 客户端输入方式
客户端通过system.in()输入,通过channel发送消息   
    
     BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(System.in));
            //客户端输入,把bufferedReader数据放到管道输出
            for (; ; ) {
                channel.writeAndFlush(bufferedReader.readLine() + "\r\n");
            }
            
### 3. writeAndFlush和write

writeAndFlush 表示把数据从缓冲区发送到客户端,write表示发送到缓冲区,并没有发送到客户端

### 4. 重载SimpleChannelInboundHandler方法

   void handlerAdded(ChannelHandlerContext ctx)-->表示客户端和服务端建立好连接
   
   void handlerRemoved(ChannelHandlerContext ctx)--->连接断开
   
   void channelActive(ChannelHandlerContext ctx)---->连接处于活动状态
   
   void channelInactive(ChannelHandlerContext ctx)----->表示连接下线
   
   void exceptionCaught(ChannelHandlerContext ctx, Throwable cause)---->出现异常
   
   void channelRead0(ChannelHandlerContext ctx, String msg)--->具体执行业务逻辑部分,也就是连接成功之后必须走的方法
   
   
 
## 2. Netty-Demo 心跳机制

主要更改的地方在服务端

在重写void initChannel(SocketChannel ch)方法中,引入IdleStateHandler()方法,他表示空闲状态监测处理器

在具体的handler处理器中extends的是ChannelInboundHandlerAdapter,也就是SimpleChannelInboundHandler的父类

重写userEventTriggered(ChannelHandlerContext ctx, Object evt)方法,我们根据返回的状态来判断服务端是什么状态

发起人是ping,返回成为pong

 
    @Override
     public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
        if (evt instanceof IdleStateEvent) {
            IdleStateEvent event = (IdleStateEvent) evt;
            String eventType = null;
            switch (event.state()) {
                case READER_IDLE:
                    eventType = "读空闲";
                    break;
                case WRITER_IDLE:
                    eventType = "写空闲";
                    break;
                case ALL_IDLE:
                    eventType = "读写空闲";
                    break;
            }
            System.out.println(ctx.channel().remoteAddress()+"超时操作:"+eventType);
            ctx.channel().close();
        }
    }



## 3. Netty-Demo 实现webSocket功能
1重要的handler
WebSocketServerProtocolHandler




