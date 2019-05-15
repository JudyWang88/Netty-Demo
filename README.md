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



----------

# Nio #
下面我将要从IO到Nio分别介绍他们的关键点,这部分的代码在com.judy.nio包下面总共分为12个Demo,最后演示了一个客户端和服务器端实现通过Nio进行通信过程,我会从每个demo入手,然后依次讲解知识点 , 介绍Nio中的核心:Selector , Channel , Buffer. 
   

1. Selector中核心属性: SelectorKeys,事件.

2. Channel: ServerSocketChannel, SocketChannel. 

3. Buffer: position ,limit,  capacity之间的关系.

4. Buffer中的方法flip(),clear().以及底层方法HeapByteBuffer DirectByteBuffer之间的关系, Buffer是如何与数据进行交互?

5. Buffer的数据类型总共有几种? 为什么是7种? 哪一种类型没有?

6. Buffer中底层allocate()方法做了什么?

7. Channel与Buffer是如何进行交互数据的?

8. Selector如何与Channel进行交互的? 如果实现一个线程就可以接受多个客户端的数据?

9. Selector,Channel,buffer整体流是什么?


## NioTest1 ##

主要简单简单什么是Buffer 

1 首先需要创建Buffer对象,分配的内存空间为10

    IntBuffer buffer = IntBuffer.allocate(10);

2 向Buffer对象输入数据
    
    buffer.put(randomNumber);

3 转变Buffer的状态,读写状态变化(必须写,后续介绍为什么)
  
     buffer.flip();

4 读取buffer中的数据

    System.out.println(buffer.get());
    

上面只是简单的介绍了一下Buffer是如何操作数据的, 在Nio中直接操作数据的只有Buffer,这点必须记住~ Buffer底层操作的是数组类型.

## NioTest2 ##

主要介绍从Buffer是如何从Channel中读取文件流数据

1.读取文件流
 
      FileInputStream fileInputStream = new FileInputStream("NioTest2.txt");
    
2.得到文件流的Channel,通道
     
      FileChannel channel = fileInputStream.getChannel();

3.创建buffer对象

    ByteBuffer buffer = ByteBuffer.allocate(512);

4.Buffer读取Channel中数据  
    
     channel.read(buffer);

5.Buffer状态转变
 
     buffer.flip();

6.读取Buffer中的数据

    buffer.get();


## NioTest3 ##

主要介绍Buffer写数据到Channel,写入文件(由于大部分内存相似,所以我只说核心的代码) 从Test1 和Test2 可以看出Channel至于Buffer进行交互,

 并且如果程序想要读写操作数据,必须是通过Buffer才可以, 而Channel知识操作数据而已. 

1.向Buffer中写入数据

      allocate.put(message[i]);

2.Buffer状态转换
      
     allocate.flip();

3.把Buffer中的数据写入到Channel中

     channel.write(allocate);

## NioTest4 ##

主要介绍position,limit,capacity的关系

### allocate初始化###

1 首先我们先看Buffer中allocate()方法的源码是如何实现,我选择的是intbuffer.

     public static IntBuffer allocate(int capacity) {
        if (capacity < 0)
            throw new IllegalArgumentException();
        return new HeapIntBuffer(capacity, capacity);
    }


     HeapIntBuffer(int cap, int lim) { 
        super(-1, 0, lim, cap, new int[cap], 0);     
    }


    IntBuffer(int mark, int pos, int lim, int cap,   // package-private
                 int[] hb, int offset)
    {
        super(mark, pos, lim, cap);
        this.hb = hb;
        this.offset = offset;
    }


      Buffer(int mark, int pos, int lim, int cap) {       // package-private
        if (cap < 0)
            throw new IllegalArgumentException("Negative capacity: " + cap);
        this.capacity = cap;
        limit(lim);
        position(pos);
        if (mark >= 0) {
            if (mark > pos)
                throw new IllegalArgumentException("mark > position: ("
                                                   + mark + " > " + pos + ")");
            this.mark = mark;
        }
    }  


我们知道capacity表示的意思是初始化buffer容量,从上面代码我们可以看出我们初始化的值最终赋给了cap. 而cap表示的意思就是buffer数组的最大容量,并且这个容量是不可以再更改的;
   
lim 的值在初始化的时候与cap是同等大小;

pos 的值在初始化的时候值为0;

### pos ,lim .cap 改变互相牵引 ###

1 buffer中存放数据

    buffer.put(randomNumber);

> position的值加1

2 buffer转换读写状态

    buffer.filp()

3 查看buffer中limit的大小 

    buffer.limit();

> limit的值变成1， position变为0， cap的值不变

通过NioTest4 的代码我们会发现limit和position和cap之间的 当buffer写入的时候pos的值 一直add. 当buffer转换状态的时候, pos要回到索引为0的位置, limit要到pos的位置, 也就是我们读取buffer中数据的时候只能呢读到limit的索引位置, 而cap表示我们最大的数组剩余的空间容量



## NioTest5 ##

Buffer中的数据类型.除啦Boolean类型没有其他都有

## NioTest6,7 ##
 
Buffer的分割和切片,Buffer只可以设置只读,看代码就会

## NioTest8 ##

主要是buffer底层是如何操作的

在Buffer中我们知道初始化数据有两种数据,第一个是allocate()方法,第二是allocateDirect() 他们的两个不同的地方在于allocate是数据放到堆内存上, 而allocateDirect()把数据放到jvm之外, 但是最终代码执行都是到jvm之外,跟踪源码的时候会发现address,为什么把他发到父类,原因是因为初始化加载的时候效率提高. 
       
    // NOTE: hoisted here for speed in JNI GetDirectBufferAddress
    long address;



## NioTest9 ##

主要讲的是buffer的另一种特性直接跟内存打交道,不对磁盘直接做操作,减少了开销,性能得到优化


## NioTest12 ##

channel是和selector如何交互了,在io上面提升了什么? 为什么Nio是异步的?


1 创建Selector

     Selector selector = Selector.open();

2 创建服务端SeverSocketChannel

     ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

3 设置为非阻塞

     serverSocketChannel.configureBlocking(false);

4 得到ServerSocket套接字,然后接受端口号

    InetSocketAddress address = new InetSocketAddress(ports[i]);

> 注意:这里的端口号只是针对的客户端和服务端连接的端口号,但是针对发送数据的时候,他的端口号会自己变化.

5 Channel注册到Selector上(这一步是重点)
    
      serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

> 首先解释一下serverSocketChannel他的意思就是连接客户端channel, 这就话的意思就是我们把连接客户端事件的channel注册到Selector上,事件是SelectionKey.OP_ACCEPT

6 获取连接数量

     int select = selector.select();

> select是阻塞的连接,也就是当客户端端不连接,或者说没有事件的时候,他会一直阻塞,他返回的是连接的数量


7  SelectionKey是什么?
    
    Set<SelectionKey> selectionKeys = selector.selectedKeys();
    
> SelectionKey是什么?

>  是一个事件

>  每次当channel注册到selector的时候都会出现一个SelectionKey

>  为什么会有SelectionKey?

>  通过SelectionKey可以获取Selector已经注册到Channel上发生的事件
 
>  通过SelectionKey可以后去Channel,可以与客户端进行交互

>  什么时候创建SelectionKey?
  
>  在Channel注册到selector的时候会创建一个Channel


8 从SelectionKey中获取channel

  ` ServerSocketChannel serverSocketChannel = (ServerSocketChannel) selectionKey.channel();`

9 务端需要返回给客户端一个serverSocket对象(表示客户端已经跟服务端建立好了连接)-->所以之后的操作都是针对的SocketChannel对象(serverSocketChannel对象现在的任务已经完成了,用不上了(就是起到建立连接))

    client = serverSocketChannel.accept();

10  配置为非阻塞的

    client.configureBlocking(false);

11 客户端SocketChannel连接到selector

 client.register(selector, SelectionKey.OP_READ);



> 目前Selector上面注册了两个Channel对象,一个是ServerSocketChannel,另一个是SocketChannel,他们两个的作用是不一样的



> ServerSocketChannel关注的事件是连接,SocketChannel关注的事件是客户端和服务器端读取事件

12 声明buffer读取获取写数据 然后交给指定的Channel进行发送数据(下面的代码只是伪代码,具体代码看NIoTest12或者NioServer)

      client = (SocketChannel) selectionKey.channel();
      ByteBuffer readBuffer = ByteBuffer.allocate(1024);
      int readCount = client.read(readBuffer);
      readBuffer.flip();
      writeBuffer.put((senderKey + ":" + receivedMessage).getBytes());

## NIoClient与NIoServer有什么不同?

1 在client不是bind端口,而是连接

       InetSocketAddress inetSocketAddress = new InetSocketAddress("localhost", 8888);
       socketChannel.connect(inetSocketAddress);

2 当Channel注册到Selector,服务端和客户端建立了连接,我们需要做判断是否连接成功

      `selectionKey.isConnectable()如果为true则表示建立了连接`
       clientChannel.isConnectionPending() 如果为ture则表示连接在Channel中是挂起也就是正在进行的状态

 

> 其他的操作跟客户端都差不多


## 总体分析##
  
   

> 多个 Channel把事件注册到Selector, Selector来进行监听, 当有数据的时候 Channel会把数据给Buffer读取.,当然在这个过程中是在while循环中


    








