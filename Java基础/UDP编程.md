原理图：

![rWJJ39.png](https://s3.ax1x.com/2020/12/25/rWJJ39.png)

#### DatagramSocket类：通讯的数据管道

- send和receive方法
- 绑定一个IP和Port，可指定

#### DatagramPacket类：封装数据的集装箱

- 发送时须含有地址标签：目的地IP+Port

实例：

```java
public class UdpRev {
    public static void main(String[] args) throws IOException {
        DatagramSocket datagramSocket = new DatagramSocket(3000); // 接收方端口为3000
        byte[] buffer = new byte[1024]; // 缓冲数组
        DatagramPacket datagramPacket = new DatagramPacket(buffer, 1024); // 包装缓冲数组

        datagramSocket.receive(datagramPacket);
        String str = new String(datagramPacket.getData(), 0, datagramPacket.getLength());
        System.out.println("对方说：" + str);
        System.out.println("来自端口" + datagramPacket.getPort());
    }
}
/* 最终打印
对方说：Hello!
来自端口47890
*/
```

```java
public class UdpSend {
    public static void main(String[] args) throws IOException {
        DatagramSocket datagramSocket = new DatagramSocket(47890); // 这里如果不指定的话就是随机的端口，
        String str = new String("Hello!");
        DatagramPacket datagramPacket = new DatagramPacket(str.getBytes(), 0, str.length(),
                InetAddress.getByName("127.0.0.1"), 3000);

        datagramSocket.send(datagramPacket);
        System.out.println("发送完毕");
    }
}
/* 最终打印
发送完毕
*/
```