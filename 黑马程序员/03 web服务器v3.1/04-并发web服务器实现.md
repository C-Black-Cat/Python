# 04-并发web服务器实现

## 前言

浏览器返回的内容分为两部分：`Response Header` 和 `Response Body`。

- Response Header 告诉浏览器要遵守的协议、编码格式等。

- Response Body 是浏览器要展示的内容，浏览器根据 Response Header 的内容来解析。

在前面，我们实现了一个简单的web服务器。但是这个服务器是单任务的，即服务完一个Client才能去服务下一个Client（一两个人连接时感受不出来，人数一多，服务就会非常慢）。既然是一个服务器，那就需要尽可能的能够快一点。因此，使用之前学习的多进程来优化这个服务器。

```python
while True:
    # 4.等待客户端连接
    new_client, client_addr = tcp_server_socket.accept()
    # 5.发送数据
    service_client(new_client)
```

在原来的程序中，accept接收用户连接，之后调用server_client函数为客户进行服务，只有服务完之后才能继续调用accept接收用户连接，且accept会发生堵塞。为了提升服务器的效率，使用多进程来改进。

## 改进方案

还是使用一个accept来接收用户连接，只是使用多个进程来处理任务。改进的优势在于，现在只需只需完创建进程这部分代码，就能继续接收用户连接，服务用户交给子进程去处理，相较于之前需要服务完用户后才能继续接收新的用户，有更高的效率。

```python
while True:
    # 4.等待客户端连接
    new_client, client_addr = tcp_server_socket.accept()
    p = multiprocessing.Process(target=server_client, args=(new_client, ))
 	p.start()
    
    new_client.close()
```

> 这里有一个问题：为什么要在主进程中加上 `new_client.close()` ，在子线程中明明已经加上了。
>
> **原因：**使用 `Process` 创建子进程时，子进程会复制父进程的资源，除了一些独有的资源（如PID），基本上子线程都会复制。以上面的代码为例：
>
> 客户端连接后，accept会创建一个new_socket，之后创建子进程，子进程复制主进程资源。在子进程中也有一个new_socket，且这两个new_socket指向同一个客户端。
>
> Linux中一切设备皆文件。如键盘、耳机、网卡、显示器等，这些设备都被抽象为文件。而这里的标记客户的套接字在底层也是一个文件，这个文件会有一个称为“文件描述符（fd）”的值来标记。这个文件描述符就是一个数字对应一个特殊的文件，例如网络接口。
>
> ![02-多进程、线程实现http服务器20220805-102707](D:\Typora\my_file\图片\02-多进程、线程实现http服务器20220805-102707.png)
>
> 创建子进程后，两个套接字都指向同一个fd，所以要想关闭套接字，就需要调用两次close这个文件描述符才会关闭，再之后触发四次挥手，因此会在主进程中添加close的调用。

## 多进程实现

```python
import socket
import re
import multiprocessing


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
    	p = multiprocessing.Process(target=server_client, args=(new_client, ))
 		p.start()
    
    	new_client.close()


if __name__ == "__main__":
    main()
```

## 多线程实现

多线程实现只需要将上面多进程的库 `multiprocessing` 替换为 `threading`，以及将 `Process` 替换成 `Thread` 类。同时将主线程中的 `new_client.close()` 删除。

> **vim中替换字符串**
>
> - 在末行模式输入：`%s/multiprocessing/threading/g`，即可替换。

## 协程实现

协程实现在资源的利用率上是最高的。

导入gevent模块，以及monkey类，在代码开头插入 `monkey.patch_all()` 这句代码，将main函数中的while循环替换成以下内容。

```python
while True:
	new_client, client_addr = tcp_server_socket.accept()
	gevent.spawn(service_client, new_client)
```

## 单进程、单线程、非堵塞实现并发的原理

在上面的程序中我们使用多线程、多进程和协程来提高web服务器的效率，然而使用单进程单线程也能够提升效率。

之所以要使用多进程等方式来改进服务器，是因为web服务器在运行时会因为连接或数据接收等原因出现堵塞的情况，若是仅使用单进程单线程，多个用户连接服务器时就很大可能会增加排队处理的时间，这样会极大地降低用户的体验。

单进程、单线程也能实现并发处理的原因就在于：取消了套接字的堵塞。即，套接字在调用accept方法和recv方法时，如果没有立即接收到消息，不会出现堵塞的现象而是会**产生异常**。

> 只要套接字对象调用了 `setblocking(False)` 就可以设置套接字为非阻塞状态。

```python
client_socket_list = list()

while True:
    # time.sleep(0.5)  # 验证时可以加延时操作，在实际运行时不要加
    try:
        new_socket, new_addr = tcp_server_socket.accept()
    except Exception as e:
        print("---没有新的客户端到来---")
    else:
        print("---只要没有产生异常，就意味着来了一个新的客户端---")
        new_socket.setblocking(False)  # 设置套接字为非阻塞的状态
        client_socket_list.append(new_socket)
    for client_socket in client_socket_list:
        try:
            recv_data = client_socket.recv(1024)
        except Exception as e:
            print(e)
            print("---这个客户端没有发生过来数据---")
        else:
            if recv_data:
            	# 对方发送过来数据 
            	print("---客户端发送过来了数据---")
            else:
                # 对方调用close导致了recv返回空值
                client_socket_list.remove(client_socket)
                client_socket.close()
```

因为上面的这段代码是并发处理，因此在实际运行过程中每个套接字运行都不能有延时操作。

> 以上代码重在理解。

## 长连接和短连接

TCP在真正的读写操作之前，server与client之间必须建立一个连接，当读写操作完成后，双方不再需要这个连接时它们可以释放这个连接。连接的建立通过三次握手，释放则需要四次握手，所以说每个连接的建立都是需要资源消耗和时间消耗的。

### TCP短连接

模拟一种tcp短连接的情况：

1. client 向 server 发起连接请求
2. server 接到请求，双方建立连接
3. client 向 server 发送信息
4. server 回应 client
5. 一次读写完成，此时双方任何一个都可以发起 close 操作

在步骤5中，一般都是 client 先发起 close 操作，当然也不排除有特殊的情况。

从上面的描述看，短连接一般只会在 client/server 间传递一次读写操作。

### TCP长连接

再模拟一种长连接情况：

1. client 向 server 发起连接
2. server 接到请求，双方建立连接
3. client 向 server 发送信息
4. server 回应 client
5. 一次读写完成，连接不关闭
6. 后续读写操作
7. 长时间操作之后，client 发起关闭请求

### 短连接的操作步骤

![08-长连接、短连接20220808-170759(1)](D:\Typora\my_file\图片\08-长连接、短连接20220808-170759(1).png)

### 长连接的操作步骤

![08-长连接、短连接20220808-170759(1)](D:\Typora\my_file\图片\08-长连接、短连接20220808-170759(1)-16599498883492.png)

之前的代码都是短连接，即发完一次信息就关闭了连接。

### 长连接实现

```python
import socket
import re
import multiprocessing


def service_client(new_client, recv_data):
    # 1.接收浏览器发送过来的请求,即HTTP请求
    # GET / HTTP/1.1
    # recv_data = new_client.recv(1024).decode("utf-8")
    # print(recv_data)
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
            response_body = html_content
            
            response_header = "HTTP/1.1 200 OK \r\n"
            response_header += "Content-Length:%d\r\n" % len(reponse_body)  # 
            response_header += "\r\n"
            
            # html_content += "\r\n<h1>Hello, world</h1>"
            reponse = response_header.encode("utf-8") + response_body
            
            new_client.send(response)

    # new_client.close()  # 不使用close，长连接

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
    tcp_server_socket.setblocking(False)
    print("------等待连接------")
    
    client_socket_list = list()
    while True:
        # time.sleep(0.5)  # 验证时可以加延时操作，在实际运行时不要加
        try:
            new_socket, new_addr = tcp_server_socket.accept()
        except Exception as e:
            print("---没有新的客户端到来---")
        else:
            print("---只要没有产生异常，就意味着来了一个新的客户端---")
            new_socket.setblocking(False)  # 设置套接字为非阻塞的状态
            client_socket_list.append(new_socket)
            
        for client_socket in client_socket_list:
            try:
                recv_data = client_socket.recv(1024).decode("uft-8")
            except Exception as e:
                print(e)
                print("---这个客户端没有发生过来数据---")
            else:
                if recv_data:
                    # 对方发送过来数据
                    service_client(client_socket, recv_data)
                    # print("---客户端发送过来了数据---")
                else:
                    # 对方调用close导致了recv返回空值
                    client_socket.close()
                    client_socket_list.remove(client_socket)
	tcp_server_socket.close()
			
    
if __name__ == "__main__":
    main()
```

> 如果服务器在调用 recv 之前就由客户端发送数据过来，那么操作系统会将这些数据暂时存放在内存中，当服务器一调用 recv 操作系统就将内存中的数据传给 recv 作为它的返回值。

> 上面的程序的更改细节：
>
> 1. 取消了service_client函数中调用close的语句。这是为了实现长连接，只使用一个套接字来重复通信。
> 2. 在服务器返回的 response_header 中增加了 Content-Length 。这是因为在长连接中，一个套接字可以重复收发数据，服务器发送完数据后，浏览器还是会进行加载（即导航栏会有一个圈在转），因为浏览器不知道服务器是否发生完了数据。此时添加的这个 Content-Length 的意思是，当浏览器接收到 Content-Length 的大小的数据时就停止加载，注意这个停止加载并不是断开套接字连接。
> 3. 监听套接字 tcp_server_socket 和 服务套接字 new_socket 都设置了不堵塞。

## epoll原理

### IO多路复用

就是我们说的select、poll、epoll，有些地方也称这种 IO 方式为 event driven IO。

select/epoll 的好处就在于单个process 就可以同时处理多个网络连接的 IO。

它的基本原理就是 select、poll、epoll 这个function 会不断地轮询所负责的所有 socket，当某个socket 有数据到达了，就通知用户进度。

### 轮询和事件通知

> 使用了epoll为什么能够提高程序运行效率呢？
>
> 在上面我们使用单进程、单线程的程序中，我们定义了一个套接字列表，而这个列表是存放在系统的应用程序内存空间中，这些程序中套接字不能直接监听数据是否到来，它需要请求操作系统去监听。因此它需要拷贝套接字的 fd 到系统内存（程序内存和系统内存是分离开的）中，操作系统查看是否有数据到来，有则返回数据，没有则返回异常。处理完一个套接字后，采用轮询的方式（即循环遍历套接字列表中的套接字）来返回系统监听的结果。但是操作系统还要运行很多其他的进程，并不是一接收到套接字的 fd 就开始监听，需要等待一段时间才进行监听。因此在轮询过程中，花费了很多的时间，效率非常低。
>
> 而使用了epoll的程序则不一样。在系统中有一个特殊的内存——应用程序和kernel共享。使用epoll之后，创建的套接字列表就存放在这个共享的内存中，那么 kernel 就不需要复制套接字而是可以直接使用套接字进行监听，节省了复制所消耗的时间和资源。同时epoll模式使用的是事件通知的方式来返回系统监听的结果。接收到了哪个套接字的数据就返回给该套接字，而不用一个个去调用套接字的监听查看是否有数据到来（之所以可以不用调用套接字就能监听是因为这些套接字存放在共享内存中）。
>
> ![11-epoll版的http服务器20220809-151921](D:\Typora\my_file\图片\11-epoll版的http服务器20220809-151921.png)

在这个内存中所有添加的要监听的，要判断数据是否到来的套接字，它的文件描述符操作系统再去检测时不是使用轮询的方式而是事件通知。

```python
import socket
import re
import time
import select


def service_client(new_client, recv_data):
    # 1.接收浏览器发送过来的请求,即HTTP请求
    # GET / HTTP/1.1
    # recv_data = new_client.recv(1024).decode("utf-8")
    # print(recv_data)
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
            f = open("./kamisama.moe"+file_name, "rb")
        except:
            # 注意：服务器的响应头格式要按照如下编写，少了response += "\r\n"的话，会出选网页报错，或图片等数据加载不出的现象。
            response = "HTTP/1.1 404 NOT FOUND\r\n"
            response += "\r\n"

            response += "------file not found------"
            new_client.send(response.encode("utf-8"))
        else:
            html_content = f.read()
            f.close()
            response_body = html_content
            
            response_header = "HTTP/1.1 200 OK \r\n"
            response_header += "Content-Length:%d\r\n" % len(response_body)  # 
            response_header += "\r\n"
            
            # html_content += "\r\n<h1>Hello, world</h1>"
            response = response_header.encode("utf-8") + response_body
            
            new_client.send(response)

    # new_client.close()  # 不使用close，长连接

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
    tcp_server_socket.setblocking(False)
    print("------等待连接------")
    
    # 创建一个epoll对象
    epl = select.epoll()
    
    # 将监听套接字对应的fd注册到epoll中
    epl.register(tcp_server_socket.fileno(), select.EPOLLIN)

    client_socket_dict = dict()

    while True:
        fd_event_list = epl.poll()  # 默认会堵塞,直到 os 检测到数据到来，通过数据通知的方式告诉这个程序，此时才会解堵塞
        # [(fd, event), (套接字对应的文件描述符, 这个文件描述符到底是什么事件　例如：可以调用recv接收等)]
        for fd, event in fd_event_list:
            print(fd_event_list)
            if fd == tcp_server_socket.fileno():
                new_socket, new_addr = tcp_server_socket.accept()
                epl.register(new_socket.fileno(), select.EPOLLIN)
                client_socket_dict[new_socket.fileno()] = new_socket

            elif event == select.EPOLLIN:
                recv_data = client_socket_dict[fd].recv(1024).decode("utf-8")
                if recv_data:
                    service_client(client_socket_dict[fd], recv_data)
                else:
                    client_socket_dict[fd].close()
                    epl.unregister(fd)
                    del client_socket_dict[fd]

    # 关闭套接字
    tcp_server_socket.close()
			
    
if __name__ == "__main__":
    main()
```















