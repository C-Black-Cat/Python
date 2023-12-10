# 03-简单web服务器实现

## 1.实现简单web服务器

实现过程非常简单，要使用之前的网络通信的知识，不同点就是需要将网页的相关信息发送给客户端。

```python
import socket


def send_msg(new_client):
    html_content = "HTTP/1.1 200 OK \r\n"
    html_content += "\r\n<h1>Hello, world</h1>"
    new_client.send(html_content.encode("utf-8"))
    new_client.close()

def main():
    """用来完成整体的控制"""
    # 1.创建TCP套接字
    tcp_server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM) 
    # 2.绑定ip,端口
    addr = ("", 7890)
    tcp_server_socket.bind(addr)

    # 3.设置监听
    tcp_server_socket.listen(128)
    print("------等待连接------")
    while True:
        # 4.接收数据
        new_client, client_addr = tcp_server_socket.accept()
        recv_data = new_client.recv(1024)
        print(recv_data.decode("utf-8"))
        
        # 5.发送数据
        send_msg(new_client)
        
	tcp_server_socket.close()

if __name__ == "__main__":
    main()
```

![web](D:\Typora\my_file\图片\web.gif)

但是结束程序之后再次运行却出现了问题：

![image-20220801144952718](D:\Typora\my_file\图片\image-20220801144952718.png)

明明已经关闭了程序的进程，为什么程序还会报 `Address already in use` 的错误呢？原因就是下面要讲到的TCP的 `三次握手和四次挥手` 。

## 2.TCP三次握手四次挥手

**三次握手**

![img](D:\Typora\my_file\图片\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NjQ3ODA5,size_16,color_FFFFFF,t_70.jpeg)

在TCP通信中，三次握手发生在客户端调用了connect方法的时候。三次握手的本质就是通知通讯双方在通信前准备好资源。1. 客户端向服务器发送一个数据包告诉服务器它要进行通信，让服务器准备好资源。2. 服务器接收到了数据包后，如果同意通信则会发送一个数据包告诉客户端同意通信以及准备好了资源。3. 客户端接收到服务器发送的包后，为了让服务器知道自己已经接收到它的回复，客户端会再向服务器发送一个数据包。

在这三次通信中，双方都互相知道对方已经准备好资源来进行通信。

**四次挥手**

![在这里插入图片描述](D:\Typora\my_file\图片\20190829114453187.png)

> TCP是全双工通信，即有两个通道用来通信，一个接收一个发送，且接收和发送南公园同时进行。

client在调用close的时候，会关闭client的发送通道，且向sever发送一个数据包（注意：这里关闭发送，只是不能发送正常的通信数据，数据包还是可以发送的）。server收到数据包后，会让应用程序的recv解堵塞，且不再接收数据，即关闭server的接收通道（指的是正常的通信数据，数据包还可以接收）。

之后Server调用close，关闭了server的发送通道，并向Client发送一个数据包，客户端接收到数据包后关闭接收Client的接收通道。

在四次挥手中存在着一个问题，即发送方怎么确定最后发送的数据包有被收到？

如果发生数据包来回应已经接收到信息，则会造成双方不停地来回发送应答包，显然不合理。

**解决方法**：先调用最后一次close的一方 **B** 需要设置**超时时间**（2MSL）。超过了超时时间没有接收到另一方 **A** 的确认包，**B** 就会重新发送一遍数据包。因此为了能够接收到重发的数据包，也是为了确定**B** 是否接收到自己的确认包，**A**不能直接释放资源，还需要等待一段时间（2MSL）。如果多次重发都没有接收到应答包，则可能是另一方出现故障，停止发送。

![img](D:\Typora\my_file\图片\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NjQ3ODA5,size_16,color_FFFFFF,t_70-16594065380905.jpeg)

这也就能解释上面程序的报错的原因：程序结束之后，先调用close的一方在最后还要等待一段时间（2MSL）才能释放资源（端口），因此立马运行程序会造成资源占用的问题。

在上面的程序中，是服务器先调用了close。客户端是浏览器，服务器是程序端。浏览器使用随机的端口号，即使先调用close，造成端口暂时被占用，因为使用随机端口，再次运行也不会报错。而服务器需要绑定端口，如果服务器先调用close，则在一段时间内再次运行程序就会报错。

> MSL(Maximum Segment Lifetime，报文最大生存时间)，即报文从发送到被接收的时间。
>
> TCP Segment在网络上的存活时间不会超过MSL（RFC793定义了MSL为2分钟，Linux设置成了30s）

**解决服务器端口占用问题**

```python
# 在套接字创建后，添加以下代码：
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
```

## 3.返回网页页面

在上面的程序中，使用浏览器来访问TCP服务器，我们发送了一个 Hello world，网页上也显示出Hello world字样。如果将发送的内容换成网页源码，那么程序是否就能够像一个真的web服务器一样给浏览器返回页面呢？

代码如下：

```python
import socket


def send_msg(new_client):
    html_content = "HTTP/1.1 200 OK \r\n"
    # html_content += "\r\n<h1>Hello, world</h1>"
    new_client.send(html_content.encode("utf-8"))

    f = open("./html/index.html", "rb")
    html_content = f.read()
    f.close()

    new_client.send(html_content)
    new_client.close()

def main():
    """用来完成整体的控制"""
    # 1.创建TCP套接字
    tcp_server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM) 
    tcp_server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    # 2.绑定ip,端口
    addr = ("", 7890)
    tcp_server_socket.bind(addr)

    # 3.设置监听
    tcp_server_socket.listen(128)
    print("------等待连接------")
    while True:
        # 4.接收数据
        new_client, client_addr = tcp_server_socket.accept()
        recv_data = new_client.recv(1024)
        print(recv_data.decode("utf-8"))
        
        # 5.发送数据
        send_msg(new_client)
	tcp_server_socket.close()

if __name__ == "__main__":
    main()
```

## 4.完善程序

上面的程序只能返回固定的页面，进一步修改程序，使得程序能够返回指定的网页。

```python
import socket
import re


def service_client(new_client):
    # 1.接收浏览器发送过来的请求,即HTTP请求
    # GET / HTTP/1.1
    recv_data = new_client.recv(1024).decode("utf-8")
    print(recv_data)
    resp = recv_data.splitlines()
    print("")
    print(resp)
    
	if resp:
        # GET /index.html HTTP/1.1
        # get post put del
        # 匹配请求的html文件
        file_name = re.match(r"[^/]+(/[^ ]*)", resp[0])
        if file_name:
            file_name = file_name.group(1)
            # 默认打开index.html
            if file_name == "/":
                file_name = "/index.html"
        print("**"*20, file_name)
        # 2.返回HTTP格式的数据给浏览器
        try:
            f = open("./html"+file_name, "rb")
        except:
            # 注意：服务器的响应头格式要按照如下编写，少了response += "\r\n"的话，会出选网页报错，或图片等数据加载不出的现象。
            response = "HTTP/1.1 404 NOT FOUND\r\n"
            response += "\r\n"

            response += "------file not found------"
            new_client.send(response.encode("utf-8"))
        else:
            html_content = f.read()
            f.close()
            response = "HTTP/1.1 200 OK \r\n"
            response += "\r\n"
            # html_content += "\r\n<h1>Hello, world</h1>"
            new_client.send(response.encode("utf-8"))
            new_client.send(html_content)

    new_client.close()

def main():
    """用来完成整体的控制"""
    # 1.创建TCP套接字
    tcp_server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM) 
    tcp_server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    # 2.绑定ip,端口
    addr = ("", 7890)
    tcp_server_socket.bind(addr)

    # 3.设置监听
    tcp_server_socket.listen(128)
    print("------等待连接------")
    while True:
        # 4.等待客户端连接
        new_client, client_addr = tcp_server_socket.accept()
        # 5.发送数据
        service_client(new_client)
	tcp_server_socket.close()

if __name__ == "__main__":
    main()
```

![web](D:\Typora\my_file\图片\web-16594318681191.gif)

