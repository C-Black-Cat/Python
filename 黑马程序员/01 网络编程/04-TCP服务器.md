# 04-TCP服务器

### 1.TCP服务器

#### 理解TCP服务器

**生活中的电话机**

要想让别人能够打通我们的电话且进行通信，需要做以下几件事：

1. 买一个手机
2. 插上电话卡
3. 设置手机为正常接听状态（即能够响铃）
4. 等待别人拨打

**TCP服务器**

如同上面的电话机过程一样，在程序中，如果要完成一个TCP服务器的功能，需要的流程如下：

1. 创建一个socket套接字
2. `bind` 绑定 IP和port 
3. `listen` 使套接字变为可以被动链接
4. `accept` 等待客户的链接
5. `recv`/`send` 接收发送数据

#### TCP服务器代码创建流程

```python
from socket import *

# 1.创建套接字
tcp_server_socket = socket(AF_INET, SOCK_STREAM)

# 2.本地信息
server_addr = ("", 7788)

# 3.绑定
tcp_server_socket.bind(server_addr)

# 4.监听端口
# 使用socket创建的套接字默认的属性是主动的，使用listen将其变为被动的，这样就可以接收别人的链接了
tcp_server_socket.listen(128)	# 这个套接字作为监听套接字，不为客户提供具体服务，只用来监听的客户的链接

# 5.接收新链接
# 如果有新的客户端来链接服务器，那么就产生一个新的套接字专门为这个客户端服务
client_socket, client_addr = tcp_server_socket.accept()	 # accept在等待新客户时会产生阻塞

# 6.接收对方发送的数据
recv_data = client_socket.recv(1024)
print("接收到的数据为:", recv_data.decode('utf-8'))

# 7.发送数据到客户端
client_socket.send("thank you!".encode('utf-8'))

# 8.关闭为这个客户端服务的套接字（注意：不是关闭TCP服务器的套接字），只要关闭了，就不能再为这个客户端服务了，如果还需要服务，只能再次链接。
client_sokcet.close()
```

> 注：
>
> - `accept()` 函数返回的值是一个元组。`a, b = (11, "ab")` 这种形式叫做 `拆包`。左边的变量数要和元组中的参数个数一样，否则会报错。
> - 理解 `listen()和accept()`
>   - 服务器创建的套接字调用 `listen()` 函数，作为监听套接字使用。监听套接字不为新链接的客户端提供具体服务，只是等待是否有新的客户端链接服务器；监听到有链接的客户端后，调用 `accept()`  产生新的套接字来为客户端服务。
>   - `listen()`和`accept()`函数的运作过程可以类比为：拨打10086人工服务的过程。
>     - 拨打10086时，系统会自动分配一个人工客服来为我们提供服务。10086这个号码可以看作是一个监听套接字，它等待我们打入电话，接通后系统为我们分配一个人工客服来提供服务，每个客服可以看作一个新建的套接字。

```python
from socket import *


def main():
    # 1.创建套接字
    tcp_server_socket = socket(AF_INET, SOCK_STREAM)
    # 2.绑定本地信息
    server_addr = ('', 7788)
    tcp_server_socket.bind(server_addr)
    # 3.让套接字变为被动链接
    tcp_server_socket.listen(128)
    print("----------1----------")
    # 4.等待链接
    client_socket, client_addr = tcp_server_socket.accept()
    print("----------2----------")
    print(client_addr)

    # 接收客户端发送过来的请求
    recv_data = client_socket.recv(1024)
    print(recv_data.decode('utf-8'))

    # 发送数据给客户端
    client_socket.send("你好,同志!".encode("utf-8"))

    # 关闭套接字
    client_socket.close()
    tcp_server_socket.close()

if __name__ == "__main__":
    main()
```

**运行结果**

TCP客户端未连接服务器

![image-20220714231204114](D:\Typora\my_file\图片\image-20220714231204114.png)

点击“连接网络”

![image-20220714231256833](D:\Typora\my_file\图片\image-20220714231256833.png)

发送数据

![image-20220714231353790](D:\Typora\my_file\图片\image-20220714231353790.png)

> 从上面结果可以看到：
>
> 在 `accept()` 处会发生阻塞，当客户端和服务器建立连接后，解堵塞。

### 2.循环为多个客户端服务

```python
from socket import *


def main():
    # 1.创建套接字
    tcp_server_socket = socket(AF_INET, SOCK_STREAM)
    # 2.绑定本地信息
    server_addr = ('', 7788)
    tcp_server_socket.bind(server_addr)
    # 3.让套接字变为被动链接
    tcp_server_socket.listen(128)

    while True:
        print("等待一个新的客户端的到来...")
        # 4.等待链接
        client_socket, client_addr = tcp_server_socket.accept()
        print(client_addr, "已连接")
		
        while True:
            # 接收客户端发送过来的请求
            recv_data = client_socket.recv(1024)
            print(recv_data.decode('utf-8'))

            # 发送数据给客户端
            send_msg = input("给客户端发送信息:")
            client_socket.send(send_msg.encode("utf-8"))

        # 关闭套接字
        # 关闭accept()返回的套接字，意味着 不会再为这个客户端服务
        client_socket.close()
        print("已经服务完毕!")
	
    # 如果监听套接字关闭了，那么就会导致不能再次等待新客户的到来，即xxxx.accept就会失败
    tcp_server_socket.close()

if __name__ == "__main__":
    main()
```

> 上面的程序可以类比成在银行排队办理业务。

**上面的程序有一个缺点，客户端关闭通信后怎么处理？**

上面的程序在客户端关闭连接后，还会执行向客户端发送数据的命令。

**思路**

首先，客户端连接服务器后，给客户端提供服务的socket会调用 `recv`，处于阻塞状态。

`recv` 解阻塞有两种方法：

1. 客户端发送过来数据。
2. 客户端调用 `close`，即断开连接。（网络调试助手中点击`断开网络`按钮，即可断开连接）

那么怎么让程序区分套接字的解阻塞是接收到数据还是客户端断开了连接呢？

客户端不能发送空数据，且发送空格的显示效果和断开连接的显示效果一致。

**解决方法是：**在 `recv` 阻塞时，客户端断开连接，那么 `recv` 的返回值为空；客户端发送了数据，即使是空格，`recv` 的返回值也不为空。因此可以使用一个 `if语句` 来判断客户端是否断开连接。

```python
if recv_data:
	client_socket.send(send_msg.encode("utf-8"))
else:
	break
```



















