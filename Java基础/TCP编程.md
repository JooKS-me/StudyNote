原理图：

![rWBZ38.png](https://s3.ax1x.com/2020/12/25/rWBZ38.png)

#### ServerSocket：服务器码头

- 需要绑定port
- 若有多块网卡，需要绑定一个IP地址

#### Socket：运输通道

- 客户端需要绑定服务器的地址和port
- 客户端往Socket输入流写入数据，送到服务端
- 客户端从Socket输出流读取服务端过来的数据
- 服务端反之亦然



服务端等待响应时，处于阻塞状态

服务端可以同时响应多个客户端（通过多线程）

服务端每接受一个客户端，就启动一个独立的线程与之对应

客户端或服务端都可以选择关闭这对Socket的通道



实例：

```java
public class TcpServer {
    public static void main(String[] args) throws IOException, InterruptedException {
        ServerSocket serverSocket = new ServerSocket(8001); // 服务端端口设置为8001

        System.out.println("等待客户端连接...");
        Socket socket = serverSocket.accept(); // 阻塞，等待客户端连接
        System.out.println("连接成功！");

        InputStream ips = socket.getInputStream(); // 打开输入流
        OutputStream ops = socket.getOutputStream(); // 打开输出流

        ops.write("你好！欢迎连接！".getBytes());
        BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(ips));
        System.out.println(bufferedReader.readLine());

        bufferedReader.close();
        ops.close();
        ips.close();
        serverSocket.close();
    }
}
```

```java
public class TcpClient {
    public static void main(String[] args) throws IOException {
        Socket socket = new Socket(InetAddress.getByName("127.0.0.1"), 8001);

        InputStream ips = socket.getInputStream();
        BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(ips));

        OutputStream ops = socket.getOutputStream();
        DataOutputStream dos = new DataOutputStream(ops);

        BufferedReader brKey = new BufferedReader(new InputStreamReader(System.in));
        while (true) {
            String strWord = brKey.readLine();
            if (strWord.equalsIgnoreCase("quit")) {
                break;
            } else {
                System.out.println("I want to send: " + strWord);
                dos.writeBytes(strWord + System.getProperty("line.separator"));
                System.out.println("Server said: " + bufferedReader.readLine());
            }

        }

        brKey.close();
        dos.close();
        ops.close();
        bufferedReader.close();
        ips.close();
        socket.close();
    }
}
```

