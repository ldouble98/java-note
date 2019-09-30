#BIO、NIO、AIO
##BIO
传统的IO方式，会在`socket.accept()`方法和`read()`方法发生阻塞。
第一次阻塞等待客户端的连接，第二次阻塞等待客户端的IO操作，这样会导致cpu一直在那等待，造成资源的浪费。
传统的socket通信方式：

- client handler

```java
package Demo4;

import java.io.IOException;
import java.io.InputStream;
import java.net.Socket;

/**
 * @Author: small_double
 * @Date: 2019/9/29 下午4:14
 * client 处理器
 */
public class BioClientHandler implements Runnable {
    private Socket socket;

    public BioClientHandler(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        InputStream inputStream = null;
        try {
            inputStream = socket.getInputStream();
            byte[] bytes = new byte[1024];
            int count = 0;

            while ((count = inputStream.read(bytes)) != -1) {
                System.out.println("\n收到服务器端的数据：" +
                        new String(bytes, 0, count, "utf-8"));
                System.out.print("请输入要发送的消息：");
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

}

```

- Bio 客户端

```java
package Demo4;

import java.io.IOException;
import java.io.OutputStream;
import java.net.Socket;
import java.util.Scanner;

/**
 * @Author: small_double
 * @Date: 2019/9/29 下午4:11
 * BIO客户端
 */
public class BioClient {
    public static void main(String[] args) {
        Socket socket = null;
        OutputStream outputStream = null;
        try {
            socket = new Socket("127.0.0.1", 9090);
            new Thread(new BioClientHandler(socket)).start();
            outputStream = socket.getOutputStream();
            Scanner scanner = new Scanner(System.in);
            while (true) {
                String s = scanner.nextLine();
                if (s.trim().equals("bye")) {
                    break;
                }
                outputStream.write(s.getBytes());
                outputStream.flush();
            }

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (outputStream != null) {
                    outputStream.close();
                }
                if (socket != null) {
                    socket.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

- 服务端

```java
package Demo4;

import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.concurrent.Executor;
import java.util.concurrent.Executors;

/**
 * @Author: small_double
 * @Date: 2019/9/29 下午4:30
 */
public class BioServer {
    public BioServer() {
        ServerSocket serverSocket = null;
        try {
            serverSocket = new ServerSocket(9090);
            Executor executor = Executors.newFixedThreadPool(10);

            while (true) {
                Socket accept = serverSocket.accept();
                System.out.println("客户端：" + accept.getRemoteSocketAddress().toString() + "来连接了");
                executor.execute(new BioServerHandler(accept));

            }

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (serverSocket != null) {
                try {
                    serverSocket.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public void start() {

    }

    public static void main(String[] args) {
        new BioServer().start();
    }
}

```

- bio server handler

```java
package Demo4;

import demo3.Server;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * @Author: small_double
 * @Date: 2019/9/29 下午4:37
 * bio server handler
 */
public class BioServerHandler implements Runnable {
    private Socket serverSocket;
    public BioServerHandler(Socket serverSocket) {
        this.serverSocket = serverSocket;
    }

    @Override
    public void run() {
        InputStream inputStream = null;
        OutputStream outputStream = null;
        try {
            inputStream = serverSocket.getInputStream();
            outputStream = serverSocket.getOutputStream();
            int count=0;
            String content=null;
            byte[] bytes=new byte[1024];
            while ((count=inputStream.read(bytes))!=-1){
                String line=new String(bytes,0,count,"utf-8");
                System.out.println(line);
                content=line.trim().equalsIgnoreCase("SJ")?new SimpleDateFormat("yyyy-MM-dd hh:mm:ss").format(new Date()): "你发的啥？";
                outputStream.write(content.getBytes());
                outputStream.flush();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            if (inputStream != null) {
                try {
                    inputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (outputStream != null) {
                try {
                    outputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (serverSocket != null) {
                try {
                    serverSocket.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

    }
}

```

`bio`方式的缺点：浪费cpu资源，同步阻塞方式。

##NIO
nio与bio相比解决了bio两次阻塞的问题（解阻塞）

- NIO client

```java
package niodemo;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Set;

public class NioClient {

    private SocketChannel socketChannel;
    private Selector selector;
    private long timeOut=2000;

    public NioClient(){
        try {
            socketChannel=SocketChannel.open();
            socketChannel.configureBlocking(false);
            selector=Selector.open();
            if(socketChannel.connect(new InetSocketAddress("127.0.0.1",9090))){
                socketChannel.register(selector, SelectionKey.OP_READ);
            }else{
                socketChannel.register(selector, SelectionKey.OP_CONNECT);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }



    public void start(){
        try {
            while (true){
                selector.select(timeOut);
                Set<SelectionKey> selectionKeys = selector.selectedKeys();
                Iterator<SelectionKey> keyIterator = selectionKeys.iterator();
                while (keyIterator.hasNext()){
                    SelectionKey selectionKey = keyIterator.next();
                    if(selectionKey.isValid()){
                        if(selectionKey.isConnectable()){
                            SocketChannel socketChannel = (SocketChannel) selectionKey.channel();
                            if(socketChannel.finishConnect()){
                                socketChannel.register(selector,SelectionKey.OP_READ);
                                ByteBuffer byteBuffer=ByteBuffer.allocate(1024);
                                byteBuffer.put("你好".getBytes());
                                byteBuffer.flip();
                                socketChannel.write(byteBuffer);
                            }else{
                                System.err.println("可以尝试重试");
                            }
                        }
                        if(selectionKey.isReadable()){
                            SocketChannel socketChannel = (SocketChannel) selectionKey.channel();
                            ByteBuffer buffer=ByteBuffer.allocate(1024);
                            socketChannel.read(buffer);
                            System.out.println(new String(buffer.array()));
                        }
                    }
                    keyIterator.remove();
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        new NioClient().start();
    }
}
```

## AIO
异步非阻塞 IO
