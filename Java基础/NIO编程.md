NIO工作示意图：

![spylAf.png](https://s3.ax1x.com/2021/01/03/spylAf.png)

#### Buffer：缓冲区，一个可以读写的内存区域

- 有这几种Buffer：ByteBuffer, CharBuffer, DoubleBuffer, IntBuffer, LongBuffer, ShortBuffer
- 注意StringBuffer不是Buffer缓冲区
- 四个主要属性：capacity 容量， position 读写位置， limit 界限， mark 标记(用于重复一个读/写操作)

#### Channel：通道

- 全双工，双向，支持读写（而String流是单向的）
- 支持异步读写
- 和Buffer配合，提高效率
- **ServerSocketChannel**：服务器TCP Socket接入通道，接收客户端
- **SocketChannel**：TCP Socket通道，可支持阻塞/非阻塞通讯
- **DatagramChannel**：UDP通道

- **FileChannel**：文件通道

#### Selector 多路选择器

- 每个一段时间，不断轮询注册在其上的Chnnel

- 如果有一个Channel有接入、读、写操作，就会被轮询出来

- 根据SelectionKey可以获取相应的Channel，进行后序IO操作

- 避免创建过多的线程

- SelectionKey四种类型：

  - OP_CONNECT
- OP_ACCEPT
  - OP_READ
  - OP_WRITE

---

#### 实例

```java
package com.jooks;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Set;

public class NioServer {
    public static void main(String[] args) {
        int port = 8001;
        Selector selector = null;
        ServerSocketChannel servChannel = null;

        // 创建好服务端的Selector和ServerSocketChannel
        try {
            selector = Selector.open(); //创建Selector
            servChannel = ServerSocketChannel.open(); //创建ServerSocketChannel
            servChannel.configureBlocking(false); //设置ServerSocketChannel为非阻塞
            servChannel.socket().bind(new InetSocketAddress(port), 1024); //将ServerSocketChannel绑定到本机的8001端口
            servChannel.register(selector, SelectionKey.OP_ACCEPT); //把服务端的Selector和ServerSocketChannel进行绑定(注册)
            System.out.println("服务器在8001端口守候");
        } catch (IOException e) {
            e.printStackTrace();
            System.exit(1);
        }

        while (true) {
            try {
                selector.select(1000); //开始轮询
                Set<SelectionKey> selectionKeys = selector.selectedKeys(); //拿到SelectionKey集合
                Iterator<SelectionKey> it = selectionKeys.iterator();
                SelectionKey key = null;
                //System.out.println(1);
                while (it.hasNext()) {
                    key = it.next();
                    it.remove();
                    try {
                        handleInput(selector, key);
                    } catch (Exception e) {
                        if (key != null) {
                            key.cancel();
                            if (key.channel() != null) {
                                key.channel().close();
                            }
                        }
                    }
                }
            } catch (IOException e) {
                e.printStackTrace();
            }

            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }


    }

    private static void handleInput(Selector selector, SelectionKey key) throws IOException {
        if (key.isValid()) {
            //处理新接入的请求
            //System.out.println(111);
            if (key.isAcceptable()) {
                //System.out.println("ac");
                ServerSocketChannel serverSocketChannel = (ServerSocketChannel) key.channel();
                SocketChannel socketChannel = serverSocketChannel.accept(); //若没有连接将得到一个null，若有将得到一个connection
                socketChannel.configureBlocking(false); //设置为非阻塞
                socketChannel.register(selector, SelectionKey.OP_READ);
            }
            if (key.isReadable()) {
                //读取数据
                //System.out.println("rb");
                SocketChannel socketChannel = (SocketChannel) key.channel();
                ByteBuffer readBuffer = ByteBuffer.allocate(1024); //创建好Buffer
                int readBytes = socketChannel.read(readBuffer); //将数据读进buffer
                if (readBytes > 0) {
                    readBuffer.flip();
                    byte[] bytes = new byte[readBuffer.remaining()];
                    readBuffer.get(bytes); //将buffer中的数据写入bytes中
                    String request = new String(bytes, "UTF-8"); //将bytes转化为String
                    System.out.println("client said: " + request);

                    String response = request + " 666";

                    doWrite(socketChannel, response); //把response发送给服务端
                } else if (readBytes < 0) {
                    // 对端链路关闭
                    key.cancel();
                    socketChannel.close();
                } else ; //读到0字节，忽略
            }
        }
    }

    public static void doWrite(SocketChannel channel, String response) throws IOException {
        if (response != null && response.trim().length() > 0) {
            byte[] bytes = response.getBytes();
            ByteBuffer writeBuffer = ByteBuffer.allocate(bytes.length);
            writeBuffer.put(bytes);
            writeBuffer.flip();
            channel.write(writeBuffer);
        }
    }
}

```



```java
package com.jooks;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Set;
import java.util.UUID;

public class NioClient {
    public static void main(String[] args) {
        // 定义好服务端的IP和端口
        String host = "127.0.0.1";
        int port = 8001;

        Selector selector = null;
        SocketChannel socketChannel = null;

        try {
            selector = Selector.open();
            socketChannel = SocketChannel.open();
            socketChannel.configureBlocking(false); //设置为非阻塞

            // 如果连接成功，则注册到多路复用器上，发送请求消息，读应答
            if (socketChannel.connect(new InetSocketAddress(host, port))) {
                socketChannel.register(selector, SelectionKey.OP_READ);
                doWrite(socketChannel);
            } else {
                socketChannel.register(selector, SelectionKey.OP_CONNECT);
                //System.out.println(2);
            }
        } catch (IOException e) {
            e.printStackTrace();
            System.exit(1);
        }

        // 可以理解成临时转变为服务端，接受数据
        while (true) {
            try {
                selector.select(1000);
                Set<SelectionKey> selectionKeys = selector.selectedKeys();
                Iterator<SelectionKey> iterator = selectionKeys.iterator();
                SelectionKey key = null;
                while (iterator.hasNext()) {
                    key = iterator.next();
                    iterator.remove();
                    try {
                        handleInput(selector, key);
                    } catch (Exception e) {
                        if (key != null) {
                            key.cancel();
                            if (key.channel() != null) {
                                key.channel().close();
                            }
                        }
                    }
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

    }

    public static void doWrite(SocketChannel socketChannel) throws IOException {
        byte[] str = UUID.randomUUID().toString().getBytes();
        ByteBuffer writeBuffer = ByteBuffer.allocate(str.length);
        writeBuffer.put(str);
        writeBuffer.flip();
        socketChannel.write(writeBuffer);
    }

    public static void handleInput(Selector selector, SelectionKey selectionKey) throws IOException, InterruptedException {
        if (selectionKey.isValid()) {
            // 判断是否连接成功
            SocketChannel socketChannel = (SocketChannel) selectionKey.channel();
            if (selectionKey.isConnectable()) {
                if (socketChannel.finishConnect()) {
                    socketChannel.register(selector, SelectionKey.OP_READ);
                }
            }
            if (selectionKey.isReadable()) {
                ByteBuffer readBuffer = ByteBuffer.allocate(1024);
                int readBytes = socketChannel.read(readBuffer);
                if (readBytes > 0) {
                    readBuffer.flip();
                    byte[] bytes = new byte[readBuffer.remaining()];
                    readBuffer.get(bytes);
                    String body = new String(bytes, "UTF-8");
                    System.out.println("Server said : " + body);
                } else if (readBytes < 0) {
                    // 对端链路关闭
                    selectionKey.cancel();
                    socketChannel.close();
                } else ; //读到0个字节，忽略
            }
            Thread.sleep(3000);
            doWrite(socketChannel);
        }
    }
}
```

