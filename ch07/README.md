## 7 优雅的断开套接字连接

本章讨论如何优雅的断开套接字的连接，之前用的方法不够优雅是因为，我们是调用 close 函数或 closesocket 函数单方面断开连接的。

### 7.1 基于TCP的半关闭

#### 7.1.1 单方面断开连接带来的问题

close和closesocket 意味着完全断开连接，不仅不能发送数据也不能接收数据。

可能会出现主机A调用close后，主机B向A传输的数据就不能被A接收了。

为了解决这类问题，只关闭一部分数据交换中的使用的流(Half-close)应运而生，断开一部分连接是指，可以传输数据但是无法接收，或可以接受数据但无法传输。顾名思义就是只关闭流的一半。

#### 7.1.2 套接字和流

两台主机中通过套接字建立连接后进入可交换数据的状态，又称「流形成的状态」。也就是把建立套接字后可交换数据的状态看做一种流。

考虑以下情况：
> 一旦客户端连接到服务器，服务器将约定的文件传输给客户端，客户端收到后发送字符串「Thank you」给服务器端。

![](https://camo.githubusercontent.com/f649a3256efa2017799986dd6afea0120ff784eb/68747470733a2f2f692e6c6f6c692e6e65742f323031392f30312f31382f356334313263336261323564642e706e67)

#### 7.1.3 针对优雅断开的shutdown函数

用于半关闭的函数，shutdown
```c++
#include <sys/socket.h>
// 成功时返回0，失败时返回-1
int shutdown(int sock,int howto);
/*
sock    需要断开的套接字文字描述符
howto   传递断开方式信息
*/
```
第二个参数决定断开方式
- `SHUT_RD` :断开输入流
- `SHUT_WR` :断开输出流
- `SHUT_RDWR` :同时断开I/O流

#### 7.1.4 为何需要半关闭

传输完成后仍需发送或接收数据

服务器端应最后向客户端传递EOF表示文件传输结束，客户端通过函数返回值接收EOF避免和文件内容冲突。`在断开输出流时向对方主机传输EOF`

#### 7.1.5 基于半关闭的文件传输程序

![](https://camo.githubusercontent.com/cdce176536821331fcc4a264652f66a0ae217417/68747470733a2f2f692e6c6f6c692e6e65742f323031392f30312f31382f356334313332363238306162352e706e67)



[file_server.c](./file_server.c)

[file_client.c](./file_client.c)

编译运行
```
gcc file_client.c -o fclient
gcc file_server.c -o fserver
./fserver 9190
./fclient 127.0.0.1 9190
```
结果会有thank you

![](https://s2.ax1x.com/2019/02/04/kJlfeS.png)

### 7.2 基于WINDOS的实现 

暂略

### 7.3 习题

> 以下答案仅代表本人个人观点，可能不是正确答案

1. **解释 TCP 中「流」的概念。UDP 中能否形成流？请说明原因。**

答：两台主机中通过套接字建立连接后进入可交换数据的状态，又称「流形成的状态」。也就是把建立套接字后可交换数据的状态看做一种流。UDP没有连接过程，不能形成流

2. **Linux 中的 close 函数或 Windows 中的 closesocket 函数属于单方面断开连接的方法，有可能带来一些问题。什么是单方面断开连接？什么情形下会出现问题？**

答：单方面断开连接就是例如主机A，B . 主机A调用close函数，那么主机A既不能接收B的数据，也不能向B发送数据。问题：可能B还有一定要A接收的数据，这样A就接收不到了

3. **什么是半关闭？针对输出流执行半关闭的主机处于何种状态？半关闭会导致对方主机接收什么消息？**

答：半关闭：传输数据但是无法接收，或可以接受数据但无法传输。顾名思义就是只关闭流的一半。 处于不能发送数据只能接收数据的状态，对输出流半关闭的主机会向连接主机发送EOF，对方知道你数据发送完了