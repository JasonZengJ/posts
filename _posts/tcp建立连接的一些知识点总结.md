title: tcp建立连接的一些知识点总结
date: 2015-08-11 01:12:37
tags: Objective-c
---

参考资料：[Unix Network Programming, Volume 1, 3rd Edition](http://book.douban.com/subject/1756533/)

##tcp 建立连接、关闭连接、以及一些细节说明

* 建立连接：
  
  1、客户端发送SYNj同步包 给服务端，请求建立连接
  
  2、服务端返回客户端确认包 ACKj+1 和 服务端同步包SYNy
  
  3、客户端返回服务端确认包 ACKy+1，
  
  4、连接建立

* 关闭连接：
  
  1、一端调用close方法，表示这一端主动关闭连接，发送一个FIN 数据包，表示这一端数据传输完毕。
  
  2、另一端接收到FIN数据包，返回一个ACK数据包，并把接收到的FIN数据包作为一个文件结束符发给这一端的应用程序，表示应用程序不会在连接上接收到任何额外的数据了。
  
  3、应用程序接收到FIN的文件结束符后，发送一个FIN报文给主动关闭连接的一端。
  
  4、主动关闭连接的一端接收到FIN报文，返回一个ACK报文，然后彻底关闭连接.
  

* 建立tcp连接的两端在三次握手时会协商tcp mss大小:pc1 发出syn报文，其中option选项填充的mss字段一般为1460，同样www server收到syn报文后，会发送syn＋ack报文应答，option选项填充的mss字段也为1460；协商双方会比较syn和syn+ack报文中mss字段大小，选择较小的mss作为发送tcp分片的大小。通过比较，协商双方的tcp mss都是1460。
 
* TCP Window :简单来说, TCP Receive Window是在TCP连接两端都有的缓冲区, 用于暂时保存到来的数据. 在这个缓冲区中的数据会被发送到应用程序中, 为新到来的数据腾出空间. 如果这个缓冲满了, 那么数据的接收方会警告发送方在缓冲去清空之前已经不能在收取更多的数据了. 这其中涉及到一些细节, 但那都是很基本的东西. 一般, 设备会在TCP Header信息中通知对方当前它的TCPWindows的大小.一般默认65535

* 为了提供新系统与旧系统的互通，必须要双方的SYN中Window scale option 一样才能缩放windows.
 
  **it can scale its windows only if the other end also sends the option with its SYN**

* 为什么TCP建立连接3次握手，而关闭连接要四次：因为在主动关闭的一方，已经发送FIN数据包了，在等待接收ACK的时候，被动关闭一方可能还在传输数据给主动关闭的一方。active close 发送FIN后，表示我不会再发送数据了，然后passive close一方接收到 FIN后，先返回一个ACK表示我知道你不会再传输数据了，也就是说passive close一方关闭接收数据通道，如果有在传输数据，会等待数据传输完后，再发送FIN 数据包给active close方，表示我也不会再发送数据了，然后 active close 方接收到这个FIN后，返回一个 ACK表示收到，然后关闭passive close一方发送数据的通道。

  在建立连接的时候，SYN没有携带数据，只带有IP header,TCP header 和一些TCP Option。而在关闭连接的时候，另一端数据可能还没发送完。
  
* 一个 SYN包就是仅SYN标记设为1的TCP包 ，ACK，FIN等类似(可查看tcp包头)

* PSH：表示是带有PUSH标志的TCP数据包。接收方因此请求数据报一到便可送往应用程序而不必等到缓冲区装满时才传送。

* RST：用于复位由于主机崩溃或其它原因而出现的错误的连接。还可以用于拒绝非法的数据报或拒绝连接请求。  重新建立连接。

* URG：紧急指针（urgent pointer）有效。 
 
  
##TCP的状态变化   

####1、11种不同的状态

CLOSED、LISTEN、SYN_SENT、SYN_RCVD、ESTABLISHED、FIN_WAIT_1、FIN_WAIT_2、CLOSE_WAIT、LAST_ACK、TIME_WAIT、CLOSING

####2、建立连接
 
 * 服务器   CLOSED => LISTEN  ，监听端口，可接收SYN数据包。
 * 应用程序 CLOSED => SYN_SENT，开始发送SYN数据包。
 * 服务器   LISTEN => SYN_RCVD，接收到SYN数据包，并发送SYN 和 ACK包。
 * 应用程序接收到带有ACK的SYN数据包，SYN_SENT => ESTABLISHED，然后发送ACK数据包。
 * 服务器接收到ACK数据包，然后 SYN_RCVD => ESTABLISHED

####3、断开连接

**一般情况：passive close 收到FIN数据包时，还在发送数据**

* active  close ESTABLISHED => FIN_WAIT_1,发送FIN数据包
* passive close ESTABLISHED => CLOSE_WAIT,接收到FIN数据包，并发送ACK数据包
* avtive  close FIN_WAIT_1  => FIN_WAIT_2,接收到ACK数据包，等待FIN  
* passive close CLOSE_WAIT  => LAST_ACK,发送FIN数据包
* active  close FIN_WAIT_2  => TIME_WAIT,接收到FIN数据包，并发送ACK数据包，并在2MSL超时后, TIME_WAIT => CLOSED。
* passive close LAST_ACK => CLOSED，接收到ACK数据包。

**active close发送FIN的同时，passive close也发送FIN**

* 两个都是active close 并且：FIN_WAIT_1 => CLOSING，发送过FIN，而未收到ACK，却收到FIN，然后发送ACK
* 接收到ACK后，CLOSING => TIME_WAIT,2MSL超时后，TIME_WAIT => CLOSED

**passive close 收到FIN数据包时，已经没有发送数据了**

* active  close ESTABLISHED => FIN_WAIT_1,发送FIN数据包
* passive close ESTABLISHED => CLOSE_WAIT,接收到FIN数据包，并发送ACK和FIN标记为1的tcp数据包，然后CLOSE_WAIT => LAST_ACK
* active  close FIN_WAIT_1  => TIME_WAIT,接收到FIN和ACK标记为1的tcp数据包，并返回ACK标记为1的tcp数据包，并在2MSL超市后，TIME_WAIT => CLOSED
* passive close LAST_ACK => CLOSED，接收到ACK数据包。

####其它说明

* TIME_WAIT:这个状态是防止最后一次握手的数据报没有传送到对方那里而准备的(注意这不是四次握手，这是第四次握手的保险状态)。这个状态在很大程度上保证了双方都可以正常结束.

####附图
![TCP-state-transition-diagram.png](https://raw.githubusercontent.com/JasonZengJ/Images/master/blog/TCP-state-transition-diagram.png)