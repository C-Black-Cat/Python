# 05-TCP下载文件

### 1.案例：文件下载器

**服务器代码**

```python
from socket import *
import sys


def send_file_content(file_name, client_socket):
    """获取文件内容"""
    file_content = None
    try:
        f = open(file_name, "rb")
        file_content = f.read()
        f.close()
    except Exception:
        print("没有下载的文件:%s" % file_name)
    if file_content:
        client_socket.send(file_content)
        print("%s发送成功!" % file_name)

def main():
    if len(sys.argv) != 2:
        print("请按照如下方式运行:python3 xxxx.py 7890")
        return
    else:
        port = int(sys.argv[1])

    # 创建套接字
    tcp_server_socket = socket(AF_INET, SOCK_STREAM)
    # 绑定信息
    tcp_server_socket.bind(('', port))
    # 监听
    tcp_server_socket.listen(128)
    while True:
        print("----等待客户端连接----")
        client_socket, client_addr = tcp_server_socket.accept()
        print("已连接: ", client_addr)
        # 接收数据
        print("等待接收信息...")
        recv_data = client_socket.recv(1024)

        # 发送数据
        if recv_data:
            file_name = recv_data.decode("utf-8")
            print("下载的文件是:%s" % file_name)
            send_file_content(file_name, client_socket)
        else:
            print("信息为空或接收出错！")
            break
        client_socket.close()
    # 关闭套接字
    tcp_server_socket.close()
    
if __name__ == "__main__":
    main()
```

**客户端代码**

```python
from socket import *


def download_file():
    # 创建套接字
    tcp_client_socket = socket(AF_INET, SOCK_STREAM)
    # 链接服务器
    # server_ip = input("请输入服务器的IP:")
    # server_port = int(input("请输入服务器的port:"))
    server_addr = ("192.168.1.130", 7890)
    tcp_client_socket.connect(server_addr)
    # 发送要下载的文件名
    file_name = input("请输入要下载的文件:")
    if file_name:
        tcp_client_socket.send(file_name.encode("utf-8"))
    else:
        print("输入为空")
        return
    # 等待接收数据
    recv_data = tcp_client_socket.recv(1024*1024)

    # 保存下载文件
    if recv_data:
        with open("[Download]"+file_name, "wb") as f:
            f.write(recv_data)
        print("%s下载完成" % file_name)
    else:
        print("%s download failed" % file_name)
    # 关闭套接字
    tcp_client_socket.close()


def main():
    while True:
        print("---------------Menu---------------")
        print("1.下载文件")
        print("0.退出")
        print("----------------------------------")
        try:
            option = int(input("请输入功能(1/0):"))
            if 1 == option:
                download_file()
            elif 0 == option:
                print("Bye.")
                break
            else:
                print("请输入正确的选项")
        except Exception:
            print("请输入正确的选项")


if __name__ == "__main__":
    main()
```

![image-20220715164505223](D:\Typora\my_file\图片\image-20220715164505223.png)

> `with open` 适用于新建一个文件，当用来打开一个文件时，如果文件不存在就会报错，所以打开文件适合用 `try + open`。

### 2.listen参数

`socket.listen(n)`
 简单来说，这里的 `n` 表示socket的”排队个数“。

一般情况下，一个进程只有一个主线程（也就是单线程），那么socket允许的最大连接数为: `n+1`
 如果服务器是多线程，比如上面的代码例子是开了2个线程，那么socket允许的最大连接数就是: `n+2`

换句话说：`排队的人数(就是那个n)` + `正在就餐的人数（服务器正在处理的socket连接数)` = `允许接待的总人数（socket允许的最大连接数）`

详见：https://blog.csdn.net/weixin_42118352/article/details/122982988

### 3.总结

#### TCP注意点

1. tcp服务器一般情况下都需要绑定IP和端口，否则客户端找不到这个服务器。
2. tcp客户端一般不绑定，因为是主动链接服务器，所以只需要确定好服务器的IP、port等信息就好，本地客户端可以随机。
3. tcp服务器中通过listen可以将socket创建出来的主动套接字变为被动的，这是做tcp服务器时必须要做的。
4. 当客户端需要链接服务器时，就需要使用connect进行链接，udp是不需要链接的而是直接发送，但是tcp必须先链接，只有链接成功才能通信。
5. 当一个tcp客户端链接服务器时，服务器端会有一个新的套接字，这个套接字用来标记这个客户端，单独为这个客户端服务。
6. 调用listen后的套接字是被动套接字，用来接收新的客户端的链接请求的，而accept返回的新套接字是标记这个新客户端的。
7. 关闭listen后的套接字意味着被动套接字关闭了，会导致新的客户端不能链接到服务器，而之前链接成功的客户端可以正常通信。
8. 关闭accept返回的套接字意味着这个客户端已经服务完成。
9. **当客户端的套接字调用close后，服务器端的recv会解阻塞，相当于recv接收到了一个空的返回值**（不是空格，返回值长度为零），因此服务器可以通过返回数据的长度来区别客户端是否已经下线。

#### UDP、TCP套接字通信流程

**UDP**

1. 创建套接字（socket）
2. 绑定地址信息（bind/可选，发数据不绑/收数据绑定）
3. 收发数据（recvfrom/sendto）
4. 关闭套接字（close）

**TCP客户端**

1. 创建套接字（socket）
2. 绑定端口（bind，一般不绑定：因为TCP客户端connect服务器时，服务器就知道客户端的IP和port了）
3. 链接服务器（connect）
4. 收发数据（recv/send）
5. 关闭套接字（close）

**TCP服务器**

1. 创建套接字（socket）
2. 绑定端口（bind，必绑定）
3. 监听端口（listen）
4. 阻塞、创建新套接字、建立连接（accept）
5. 收发数据（recv/send）
6. 关闭套接字（close）
