# 01-GIL锁

一个关于GIL的面试题，如下：

>描述Python GIL的概念，以及它对**Python多线程**的影响？编写一个多线程抓取网页的程序，并阐明多线程抓取程序是否可比单线程性能有提高，并解释原因。

参考答案：

>1. Python语言和GIL没有一点关系，仅仅是由于历史原因在CPython虚拟机（解释器）难以移除GIL。
>2. GIL，全局解释器锁，每个线程在执行的过程中都需要先获取GIL，保证同一时刻只有一个线程可以执行代码。
>3. 线程释放GIL锁的情况：在IO操作等可能会引起阻塞system call之前，可以暂时释放GIL，但是在执行完毕后，必须重新获取GIL Python3.x使用计时器（执行时间到达阈值后，当前线程释放GIL）或Python2.x，tickets计数达到100。
>4. Python使用多进程是可以利用多核的CPU资源的。
>5. 多线程爬取比单线程性能有提升，因为遇到IO堵塞会自动释放GIL锁。
>

## 进一步认识多线程

之前学习了多进程编程，但是对它的了解仅停留在代码逻辑方面，多进程在Linux中的实际表现却没有过多的了解。现在通过 `htop` 命令来查看Linux虚拟机运行多线程代码后内存资源的使用情况。

**单线程运行死循环**

首次来查看运行一个死循环，系统资源的使用情况：

```python
# 主线程死循环，占满CPU
while True:
    pass
```

> 在当前终端界面下，按住 `ctrl+shift+t` 打开一个新终端，在新终端中可以使用 `htop` 命令来查看系统资源的使用情况。

![捕获](D:\Typora\my_file\图片\捕获-16618507684121.PNG)

可以看到Ubuntu的一个核的使用率已经达到100%。

若是运行两次死循环，则Ubuntu这两个核都会被占满。

![捕获](D:\Typora\my_file\图片\捕获-16618511446723.PNG)

**多线程运行死循环**

运行两个死循环，将Ubuntu两个核的资源占满了，现在使用多线程来运行这两个死循环，查看系统资源的使用情况。

```python
import threading

# 子线程死循环
def test():
    while True:
        pass

t1 = threading.Thread(target=test)
t1.start()

while True:
    pass
```

结果：

![image-20220830172534417](D:\Typora\my_file\图片\image-20220830172534417.png)

**多进程运行死循环**

```python
import multiprocessing

def test():
    while True:
        pass

t1 = multiprocessing.Process(target=test)
t1.start()

while True:
    pass
```

结果：

![image-20220830174141145](D:\Typora\my_file\图片\image-20220830174141145.png)

>可以看到，使用两个线程来运行死循环，占用了系统两个核各一半的资源，而上面运行两个死循环时Ubuntu两个核的资源都占满了，多进程运行死循环时两个核资源也占满了。从这个结果就可以说明多线程（有GIL）运行程序不是并行的，如果是并行的，则两个核都会占满（如多进程运行程序）。
>
>那为什么会出现两个核都只占用一半资源的情况呢？
>
>因为上面的多线程虽然两个核都在运行程序，但是同一时间内只有一个程序在运行，即一个核在运行程序时另一个核没有运行。因此在在一定时间内，内核运行程序的时间和闲置的时间大致相等，所以每个内核资源的占用大约为50%。
>
>而产生这种现象的原因是，多线程中存在GIL（全局解释器锁）。
>
>**注**：这里的多线程指的是CPython解释器处理下的多线程，Python语言本身没有问题，存在问题的是CPython解释器。
>
>我们平时运行程序时，会使用 `python3 xxx.py` 。其中，这个Python3是一个程序，我们使用这个程序来执行后面的Python代码，实质上这个程序就是Python解释器。

GIL：保证多线程程序同一时间只有一个线程在运行。

多线程是可以并行执行，发挥多核CPU的性能的，但是由C语言编写的Python解释器存在GIL的问题，所以使用CPython运行的Python程序的多线程变成了并发。

**线程与进程适用情况**

```
计算密集型程序：进程

IO密集型程序（文件读写、U盘读写、网络的收与发）：线程、协程
	IO操作有大量的等待时间，多线程可以利用这些等待时间来执行其他线程，发福的降低了等待时间
```

## 克服GIL

一般来说，解决GIL的方法有2种：

1. Python多线程会存在GIL的问题是因为使用的是CPython解释器，最直接的方法是使用其他解释器，如：JPython。

2. 使用其他语言来实现线程要执行的内容

   ```c
   #include<stdio.h>
   
   void test(){
       printf("----test----\n");
   }
   
   int main(){
       test();
       printf("hello world\n");
       return 0;
   }
   
   // 这是一个C语言程序，C语言是编译语言，需要先编译成一个可执行文件系统才能执行。
   ```

   使用 `gcc test.c` 来编译C文件，生成一个 `a.out` 的二进制文件，Linux系统可以直接运行。
   
   ![image-20220905103910658](D:\Typora\my_file\图片\image-20220905103910658.png)
   
   **python运行C代码**
   
   ```c
   // C语言来实现死循环--DeadLoop.c
   void DeadLoop(){
   	while(1){
   		;
   	}
   }
   // 这里面的 ; 是一个占位符相当于Python中的pass
   ```
   
   使用 `gcc` 工具生成对应C代码的动态链接库（Linux平台下）
   
   ```shell
   gcc DeadLoop.c -shared -o libdead_loop.so
   ```
   
   Python代码
   
   ```python
   # main.py
   from ctypes import *
   from threading import Thread
   
   # 加载动态库
   lib = cdll.LoadLibrary("./libdead_loop.so")
   # 创建一个子线程，让其执行C语言编写的函数，此函数是一个死循环
   t = Thread(target=lib.DeadLoop)
   t.start()
   # 主线程
   while True:
       pass
   ```
   
   ![捕获](D:\Typora\my_file\图片\捕获-16623466548501.PNG)

可以看到此时的多线程是并行执行。

















