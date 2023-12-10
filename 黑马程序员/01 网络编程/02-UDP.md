# 02-UDP

### 前言

#### 编写代码

![image-20220710172825866](D:\Typora\my_file\图片\image-20220710172825866.png)

#### 缩进问题

在 vim 中，按下 esc 进入命令模式，按下 `shift+v（即V大写）`就可以选中光标所在行，按上下键选择要选中的区域，按下 `shift+>` 向右缩进4格，按下 `shift+<` 向左缩进4格。 

![image-20220710173456394](D:\Typora\my_file\图片\image-20220710173456394.png)

#### Ubuntu运行python代码

![image-20220710233307259](D:\Typora\my_file\图片\image-20220710233307259.png)

出现这种错误是因为中文的编码问题，且这种问题多出现于使用python2运行代码的情况。解决方法：使用python3运行代码。

![image-20220710233454244](D:\Typora\my_file\图片\image-20220710233454244.png)

### 1.UDP网络程序--发送、接收数据

#### 程序流程

创建一个基于UDP地网络程序流程很简单，具体步骤如下：

1. 创建客户端套接字
2. 发送/接收数据
3. 关闭套接字

![01-udp发送数据demo-120220711-215603](D:\Typora\my_file\图片\01-udp发送数据demo-120220711-215603.png)

#### 代码

```python
from socket import *

# 1.创建UDP套接字
udp_socket = socket(AF_INET, SOCK_DGRAM)

# 2.准备接收方的地址
# '192.168.1.150' 表示目的IP
# 8080 表示目的端口
dest_addr = ('192.168.1.150', 8080) # 注意，这里等号右边是 元组，IP是字符串，端口是数字

# 3.从键盘获取数据
send_data = input('请输入要发送的数据: ')

# 4.发送数据到指定的电脑上的指定程序中
udp_socket.sendto(send_data.encode('utf-8'), dest_addr)

# 5.关闭套接字
udp_socket.close()
```

> 注：vim 中输入变量时，按下ctrl + N，可以查看补全信息

#### 运行结果

在Ubuntu中运行脚本：

![image-20220711221332343](D:\Typora\my_file\图片\image-20220711221332343.png)

**使用网络调试助手接收信息**

在Ubuntu虚拟机中运行程序，而网络调试助手可以安装在主机中（也可以在另一台虚拟机中）。

要使网络调试助手能够接收到信息，主机和Ubuntu虚拟机必须能够通信，即能够 `ping通` 。在这里我将Ubuntu的网络连接设置为 `NAT模式` （最好是设置为 `桥接模式`）。切换网络模式时，若虚拟机没有自动获取到IP，可以使用 `sudo dhclient` 获取IP。

 网络调试助手下载：https://crazycloud.lanzouy.com/ilSv507pl8tg

```python
import socket

def main():
    udp_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    ipaddr = input("请输入要通信的IP地址(默认为192.168.43.223): ")
    if ipaddr=="":
        ipaddr = "192.168.43.223"
    port = input("请输入端口号(默认为8080): ")
    if port=="":
        port = "8080"
    input_data = input("请输入要发送的信息: ")
    udp_socket.sendto(input_data.encode('utf-8'), (ipaddr, int(port)))

    udp_socket.close()


if __name__ == "__main__":
    main()
```

![image-20220712095403678](D:\Typora\my_file\图片\image-20220712095403678-16576738858141.png)

> 注：
>
> 1. 这里Ubuntu的IP地址为：192.168.1.130，而网络调试助手上显示数据来自192.168.43.223，而这个IP地址是主机的IP地址，出现这种情况是因为Ubuntu使用的是 `NAT模式` ，主机的IP作为虚拟机与外部网络的转接口，所以网络调试助手会将虚拟机的IP识别为主机IP。
> 2. 网络调试助手打开时，`本地IP地址` 会有一个值，这个值是电脑某张网卡的IP地址。使用网络调试助手的UDP通信时，`本地IP地址` 的值可以随便填，只要确保虚拟机中程序填写的目标IP地址（物理机某张网卡的地址）是能够和虚拟机ping通的。但是 `本地端口号` 要和程序中填入的端口号相同。
> 3. 从网络调试助手中可以看到，每一次运行程序时，程序所占用的端口号都不一样，每次运行程序时，端口号会随机变化。

### 2.UDP发送数据--程序编写

#### 循环发送消息

```python
import socket

def main():
    # 1.创建套接字
    udp_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    # 循环发送
    while True:
        # 输入IP、port、信息
        ipaddr = input("请输入要通信的IP地址(默认为192.168.43.223): ")
        port = input("请输入端口号(默认为8080): ")
        input_data = input("请输入要发送的信息: ")
        # 默认IP地址
        if ipaddr=="":
            ipaddr = "192.168.43.223"
        # 默认port
        if port=="":
            port = "8080"
        # 当输入为 exit 时，退出循环
        if input_data=="exit":
            break
        # 发送信息
        udp_socket.sendto(input_data.encode('utf-8'), (ipaddr, int(port)))
	# 关闭套接字
    udp_socket.close()


if __name__ == "__main__":
    main()
```

#### 运行结果

![image-20220712093241866](D:\Typora\my_file\图片\image-20220712093241866.png)

程序实现了循环发送的功能，并且输入 `exit` 程序就会退出。

但是看到左边网络调试助手接收到的结果，循环发送的消息并不会自动换行，而是紧接在上一条信息之后。

### 3.接收UDP数据

**要求：**使用网络调试助手发送数据，让程序来接收的信息。

![image-20220712094625386](D:\Typora\my_file\图片\image-20220712094625386.png)

网络调试助手要发送数据，就要填写Ubuntu虚拟机的IP地址和端口号。要让程序能够收到信息，程序就必须有一个端口号，且必须是 `固定的端口号`。若程序是随机的端口号，那么通信的时候不能确定程序的端口号，且每次通信时都需要重新填写端口号，甚至可能无法通信，非常不方便。

#### 程序流程

1. 创建套接字
2. 绑定本地自己的信息（IP和端口）
3. 接收数据
4. 关闭套接字

```python
from socket import *

# 1.创建套接字
udp_socket = socket(AF_INET, SOCK_DGRAM)

# 2.绑定本地的相关信息，如果一个程序不绑定，则系统会随机分配
local_addr = ('', 7788)		# ip地址和端口号，IP一般不写，表示本机的任何一个IP
udp_socket.bind(local_addr)	# 使用 bind 绑定固定端口号

# 3.等待接收对方发送的数据
recv_data = udp_socket.recvfrom(1024)	# 1024表示本此接收最大的字节数

# 4.显示接收到的数据
print(recv_data[0].decode('gbk'))

# 5.关闭套接字
udp_socket.close()
```

> `recvfrom()` 在数据没有到来之前会一直处于 `阻塞状态`，即程序一直等待数据的到来。

#### 运行结果

![image-20220713092057630](D:\Typora\my_file\图片\image-20220713092057630.png)

其中， `recvfrom()` 返回的是一个 `元组` 数据类型。

![image-20220713091132923](D:\Typora\my_file\图片\image-20220713091132923.png)

这个元组由两部分组成：`接收到的数据（byte类型）`和一个包含 `发送方IP地址和端口号`的元组。

```
import socket

def main():
	# 1.创建套接字
	udp_scket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
	
	# 2.绑定本地的相关信息，如果一个程序不绑定，则系统会随机分配
	localaddr = ('', 7788)
	udp_socket.bind(localaddr)
	
	# 3.等待接收对方发送的数据
	recv_data = udp_socet.recvfrom(1024)
	recv_msg = recv_data[0]		# 存储接收的数据
	send_addr = recv_data[1]	# 存储发送方的地址信息
	
	# 4.打印接收到的数据
	print("%s:%s" % (str(send_addr), recv_msg))	# recv_msg.decode('utf-8')
	
	# 5.关闭套接字
	udp_socket.close()
	
	
if __name__ == "__main__":
	main()
```

**结果**

![image-20220713095254591](D:\Typora\my_file\图片\image-20220713095254591.png)

将输出结果中的 `recv_msg` 改为 `recv_msg.decode('utf-8')`，结果为：

![image-20220713095621720](D:\Typora\my_file\图片\image-20220713095621720.png)

#### 编码

但是使用当主机的网络调试助手发送的是中文信息时，Ubuntu程序中继续使用 `utf-8` 解码，程序会报错。应该修改为 `gbk` 解码。

![image-20220713100152760](D:\Typora\my_file\图片\image-20220713100152760.png)

> 注：出现这个问题，不是因为程序有错误，是由Windows导致的：

> 1. Windows中默认的编码是 `gbk`，当Windows发送一个数据时，如果发送的是英文，它不会使用 `gbk` 格式进行编码；如果存在中文，Windows系统默认会按照 `gbk` 格式进行编码。Ubuntu虚拟机接收到编码的信息后默认使用 `utf-8` 进行解码，那么一定会出现错误。同时，Ubuntu发送含中文的数据到Windows中，若是使用系统默认的编码，则网络调试助手接收到的数据会产生乱码。
>
> 2. 修改程序，使用 `gbk` 解码从Windows中接收的信息，这样程序才不会报错。
>
> 3. 编码和解码都要使用相同的编码解码格式。
>
> 4. 要在Windows中正确显示中文，必须使用 `gbk` 格式编码解码；同理，在Linux中要正确显示中文，必须使用 `utf-8` 格式进行编码解码。

#### 程序：循环接收数据

```
from socket import *

def main():
    # 1. 创建套接字
    udp_socket = socket(AF_INET, SOCK_DGRAM)
    # 2. 绑定本地信息
    localiddr = ("", 7788)
    udp_socket.bind(localiddr)
    while True:
        # 3. 等待接收信息
        recv_data = udp_socket.recvfrom(1024)
        recv_msg = recv_data[0]
        send_addr = recv_data[1]
        if recv_msg == "exit":
            break
        # 4. 打印信息
        print("%s:%s" % (str(send_addr), recv_msg.decode('gbk')))
    # 5. 关闭套接字
    udp_socket.close()

if __name__ == "__main__":
    main()
```

![image-20220713110228657](D:\Typora\my_file\图片\image-20220713110228657.png)

### 4.端口绑定问题

在前面Ubuntu发送数据时，程序并没有绑定端口号，所以系统会自动给程序一个随机的端口号，所以每次启动程序，网络调试助手上显示的端口号都不一样。

> 注：对于UDP发送方，端口号可以不用绑定；对于UDP接收方，端口号需要绑定。
>
> 一个端口号不能同时被用两次。因为端口是用来唯一标识服务或程序的，因此端口号不能同时使用，且每个端口号也是唯一的。

```python
from socket import *

# 创建UDP套接字
udp_socket = socket(AF_INET, SOCK_DGRAM)

# 准备接收方的地址
	# '192.168.1.150' 表示目的IP
	# 8080 表示目的端口
dest_addr = ('192.168.1.150', 8080) # 注意，这里等号右边是 元组，IP是字符串，端口是数字

# 从键盘获取数据
send_data = input('请输入要发送的数据: ')

# 绑定端口号
udp_socket.bind(('', 7788))

# 发送数据到指定的电脑上的指定程序中
udp_socket.sendto(send_data.encode('utf-8'), dest_addr)

# 关闭套接字
udp_socket.close()
```

![image-20220713122659163](D:\Typora\my_file\图片\image-20220713122659163.png)

### 5.应用：UDP聊天器

#### 说明

> 在一个电脑中编写1个程序，有2个功能：
>
> - 1.获取键盘数据，并将其发送给对方
>
> - 2.接收数据并显示
>
> 并且功能数据进行选择以上的2个功能调用

#### 数据传输方式

- **单工通信：**在单工模式下，发送方可以发送数据，但发送方无法接收数据。这是一种单向通信。

  生活举例：单向行车道

- **半双工通信：**双向传送，在某一时刻，只能一方为发送，另一方为接收。（半双工通信允许信号在两个方向上传输，但某一时刻只允许信号在一个信道上单向传输。）

  生活举例：狭窄小胡同

- **全双工通信：**双向传送，在某一时刻，双方可以同时接收和发送数据。（全双工通信允许数据同时在两个方向上传输，即有两个信道，因此允许同时进行双向传输。）

  生活举例：双向大马路

![通信方式](D:\Typora\my_file\图片\通信方式.jpg)

> 套接字可以同时收发数据，属于 `全双工`。

#### 参考代码

```python
from socket import *


def send_msg(udp_socket):
    """发送消息"""
    print("")
    dest_ip = input("请输入对方的IP地址:")
    dest_port = int(input("请输入对方的port:"))
    msg = input("请输入信息:")
    udp_socket.sendto(msg.encode('gbk'), (dest_ip, dest_port))


def recv_msg(udp_socket):
    """接收消息"""
    print("")
    print("等待接收数据:")
    recv_data = udp_socket.recvfrom(1024)
    print("%s:%s" % (recv_data[1], recv_data[0].decode('gbk')))


def main():
    # 创建套接字
    udp_socket = socket(AF_INET, SOCK_DGRAM)
    # 绑定消息
    udp_socket.bind(('', 7788))
    # 循环发送或接收数据
    while True:
        print("--------------------")
        print("请输入功能:")
        print("1.发送消息")
        print("2.接收消息")
        print("0.退出程序")
        op = input("请输入选项:")

        if op == "1":
            # 发送数据
            try:
                send_msg(udp_socket)
            except Exception:
                print("发送数据时出错!")
        elif op == "2":
            # 接收数据
            try:
                recv_msg(udp_socket)
            except Exception:
                print("接收数据时出错!")
        elif op == "0":
            print("\n感谢使用!")
            break
        else:
            print("请输入正确的选项")
    udp_socket.close()
        
if __name__ == "__main__":
    main()
```

**本地环回地址：**127.0.0.1，只能用于本地通信，不能用于网络通信。

**补充**

> 运行代码出错时：
>
> ![image-20220713142353318](D:\Typora\my_file\图片\image-20220713142353318.png)
>
> 命令行输入 `vim demo_xxx.py +10` 能够快定位到错误行。

> **问题：**如果网络调试助手多次点击发送数据，然后程序转到 `recvfrom()` 接收数据，那么此时网络调试助手不用点击发送程序就能接收到数据。
>
> **原因：**当对方发送数据过来时，操作系统来者不拒，会先将数据存放起来，当应用程序调用 `recvfrom()` 时，程序会向操作系统询问是否有发送给它的数据，有就拿走，没有就等待(也就是阻塞状态)。
>
> 网络调试助手发送数据给Ubuntu虚拟机时，Ubuntu中的程序还未调用 `recvfrom()` 函数来接收数据，此时这些数据就由Ubuntu操作系统接收存储了，当程序调用 `recvfrom()` 后，操作系统立即会将数据传给程序。
>
> **类比：**上面的过程可以类比成接收快递的过程。比如你买了很多商品，如果你不在家，快递就会堆放在家门口，一直不回家的话，快递就会一直堆放在门口，直到你回家后接收了这些快递。
>
> **漏洞：**如果不调用recvfrom，操作系统会接收存储所有发送过来的数据。如果扫描一台电脑A的所有打开的端口，然后使用 while True，一停不停把一个好几G的数文件发送过去，这就会导致电脑A操作系统的内存被占用完，那么此时发送的正常数据就会被操作系统拒收，原本的程序就无法正常通信。
>
> 所以如果一个应用程序要接收数据，那么尽量要使用 while True来一直接收，如果只接收一次的话，可能会让别有心思的人在极短的时间内让你的电脑崩溃！！！

