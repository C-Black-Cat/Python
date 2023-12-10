# 03-私有化、import、封装继承多态

## 私有化

- `xx`：公有变量。
- `_x`：单前置下划线,私有化属性或方法，`from somemodule import *`禁止导入,类对象和子类可以访问。
- `__xx`：双前置下划线,避免与子类中的属性命名冲突，无法在外部直接访问(名字重整所以访问不到)。
- `__xx__`：双前后下划线,用户名字空间的魔法对象或属性。例如:`__init__` , __ 不要自己发明这样的名字。
- `xx_`：单后置下划线,用于避免与Python关键词的冲突。

通过name mangling（名字重整(目的就是以防子类意外重写基类的方法或者属性)如：_Class__object）机制就可以访问private了。

```python
#coding=utf-8

class Person(object):
    def __init__(self, name, age, taste):
        self.name = name
        self._age = age 
        self.__taste = taste

    def showperson(self):
        print(self.name)
        print(self._age)
        print(self.__taste)

    def dowork(self):
        self._work()
        self.__away()


    def _work(self):
        print('my _work')

    def __away(self):
        print('my __away')

class Student(Person):
    def construction(self, name, age, taste):
        self.name = name
        self._age = age 
        self.__taste = taste

    def showstudent(self):
        print(self.name)
        print(self._age)
        print(self.__taste)

    @staticmethod
    def testbug():
        _Bug.showbug()

# 模块内可以访问，当from  cur_module import *时，不导入
class _Bug(object):
    @staticmethod
    def showbug():
        print("showbug")

s1 = Student('jack', 25, 'football')
s1.showperson()
print('*'*20)

# 无法访问__taste,导致报错
# s1.showstudent() 
s1.construction('rose', 30, 'basketball')
s1.showperson()
print('*'*20)

s1.showstudent()
print('*'*20)

Student.testbug()
```

![私有变量](D:\Typora\my_file\图片\private-16623680357281.png)

#### 总结

- 父类中属性名为`__名字`的，子类不继承，子类不能访问。
- 如果在子类中向`__名字`赋值，那么会在子类中定义的一个与父类相同名字的属性。
- `_名`的变量、函数、类在使用`from xxx import *`时都不会被导入。

## import导入模块

### 1. import 搜索路径

![img](D:\Typora\my_file\图片\QQ20171023-213011@2x-16623682864573.png)

#### 路径搜索

- 从上面列出的目录里依次查找要导入的模块文件
- '' 表示当前路径
- 列表中的路径的先后顺序代表了python解释器在搜索模块时的先后顺序

#### 程序执行时添加新的模块路径

```python
sys.path.append('/home/itcast/xxx')
sys.path.insert(0, '/home/itcast/xxx')  # 可以确保先搜索这个路径
```

```python
In [37]: sys.path.insert(0,"/home/python/xxxx")
In [38]: sys.path
Out[38]: 
['/home/python/xxxx',
 '',
 '/usr/bin',
 '/usr/lib/python35.zip',
 '/usr/lib/python3.5',
 '/usr/lib/python3.5/plat-x86_64-linux-gnu',
 '/usr/lib/python3.5/lib-dynload',
 '/usr/local/lib/python3.5/dist-packages',
 '/usr/lib/python3/dist-packages',
 '/usr/lib/python3/dist-packages/IPython/extensions',
 '/home/python/.ipython']
```

### 2. 重新导入模块

模块被导入后，`import module`不能重新导入模块，重新导入需用`reload`

![img](D:\Typora\my_file\图片\QQ20171023-213646@2x-16623683228765.png)

![img](D:\Typora\my_file\图片\QQ20171023-213753@2x-16623683288017.png)

![img](D:\Typora\my_file\图片\QQ20171023-214117@2x-16623683354199.png)

![img](D:\Typora\my_file\图片\QQ20171023-214038@2x-166236834335211.png)

### 3. 多模块开发时的注意点

```python
# recv_msg.py模块
from common import RECV_DATA_LIST
# from common import HANDLE_FLAG
import common


def recv_msg():
    """模拟接收到数据，然后添加到common模块中的列表中"""
    print("--->recv_msg")
    for i in range(5):
        RECV_DATA_LIST.append(i)


def test_recv_data():
    """测试接收到的数据"""
    print("--->test_recv_data")
    print(RECV_DATA_LIST)


def recv_msg_next():
    """已经处理完成后，再接收另外的其他数据"""
    print("--->recv_msg_next")
    # if HANDLE_FLAG:
    if common.HANDLE_FLAG:
        print("------发现之前的数据已经处理完成，这里进行接收其他的数据(模拟过程...)----")
    else:
        print("------发现之前的数据未处理完，等待中....------")
```

```python
# handle_msg.py模块
from common import RECV_DATA_LIST
# from common import HANDLE_FLAG
import common

def handle_data():
    """模拟处理recv_msg模块接收的数据"""
    print("--->handle_data")
    for i in RECV_DATA_LIST:
        print(i)

    # 既然处理完成了，那么将变量HANDLE_FLAG设置为True，意味着处理完成
    # global HANDLE_FLAG
    # HANDLE_FLAG = True
    common.HANDLE_FLAG = True

def test_handle_data():
    """测试处理是否完成，变量是否设置为True"""
    print("--->test_handle_data")
    # if HANDLE_FLAG:
    if common.HANDLE_FLAG:
        print("=====已经处理完成====")
    else:
        print("=====未处理完成====")
```

```python
# common.py模块
RECV_DATA_LIST = list()  # 用来存储数据
HANDLE_FLAG = False  # 用来标记是否处理完数据
```

```python
# main.py模块
from recv_msg import *
from handle_msg import *


def main():
    # 1. 接收数据
    recv_msg()
    # 2. 测试是否接收完毕
    test_recv_data()
    # 3. 判断如果处理完成，则接收其它数据
    recv_msg_next()
    # 4. 处理数据
    handle_data()
    # 5. 测试是否处理完毕
    test_handle_data()
    # 6. 判断如果处理完成，则接收其它数据
    recv_msg_next()


if __name__ == "__main__":
    main()
```

![img](D:\Typora\my_file\图片\QQ20171024-080610@2x-166236847352313.png)

![img](D:\Typora\my_file\图片\QQ20171024-081134@2x-166236848032115.png)

> 1. `import common`
>
>    使用 `import common` 导入 `common.py` 模块时，**实际上是在当前程序中创建了一个名为 common 的变量，这个变量指向了 `common.py` 这个模块**，而 `common.HANDLE_FLAG` 指向的是 `common.py` 模块中的 `HANDLE_FLAG` 这个变量。因此在使用 `common.HANDLE_FLAG = True` 这条语句修改 `HANDLE_FLAG` 的值后， 其他程序调用`common.py` 模块中的 `HANDLE_FLAG` 变量，取到的是修改后的值。
>
> 2. `from common import HANDLE_FLAG`
>
>    - 使用 `from common import HANDLE_FLAG` 时，**实际上是在当前的程序中创建了一个名为 `HANDLE_FLAG` 的变量，但是这个变量指向的是 `common.py` 模块中变量 `HANDLE_FLAG` 的值 `False`**。所以使用 `HANDLE_FLAG = True` 这条语句修改 `HANDLE_FLAG` 的值，只是把 `HANDLE_FLAG` 从指向 `common.py` 模块中的值 False 改成指向内存中的值 True，并没有修改 `common.py` 模块中的值，而其他模块再次调用 `common.py` 中的 `HANDLE_FLAG` 的值时，取到的也是原值，取不到修改后的值。
>    - 但是如果 `HANDLE_FLAG` 的值是一个 `list`，使用 append 方法是可以修改到 `common.py` 模块中的 `HANDLE_FLAG` 的值的。
>

## 再议 封装、继承、多态

封装、继承、多态 是面向对象的3大特性

### 1. 为啥要封装

![img](D:\Typora\my_file\图片\QQ20171023-231854@2x-16624539059027.png)

![img](D:\Typora\my_file\图片\QQ20171023-232657@2x-16624538828955.png)

#### 好处

> 1. 在使用面向过程编程时，当需要对数据处理时，需要考虑用哪个模板中哪个函数来进行操作，但是当用面向对象编程时，因为已经将数据存储到了这个独立的空间中，这个独立的空间（即对象）中通过一个特殊的变量（__class__）能够获取到类（模板），而且这个类中的方法是有一定数量的，与此类无关的将不会出现在本类中，因此需要对数据处理时，可以很快速的定位到需要的方法是谁 这样更方便
> 2. 全局变量是只能有1份的，多很多个函数需要多个备份时，往往需要利用其它的变量来进行储存；而通过封装 会将用来存储数据的这个变量 变为了对象中的一个“全局”变量，只要对象不一样那么这个变量就可以再有1份，所以这样更方便
> 3. 代码划分更清晰

面向过程

```python
全局变量1
全局变量2
全局变量3
...

def 函数1():
    pass


def 函数2():
    pass


def 函数3():
    pass


def 函数4():
    pass


def 函数5():
    pass
```

面向对象

```python
class 类(object):
    属性1
    属性2

    def 方法1(self):
        pass

    def 方法2(self):
        pass

class 类2(object):
    属性3
    def 方法3(self):
        pass

    def 方法4(self):
        pass

    def 方法5(self):
        pass
```

> 注：Python类中的这个 `self` 只是一个形参，可以换成其他的变量名（一般不要改成其他变量名），比如：`this`；C++ 中也有一个 `this`，这个 `this` 是C++的一个关键字，不能改成其他的

### 2. 为啥要继承

![img](D:\Typora\my_file\图片\QQ20171023-234358@2x-16624538478971.png)

#### 说明

> 1. 能够提升代码的重用率，即开发一个类，可以在多个子功能中直接使用
> 2. 继承能够有效的进行代码的管理，当某个类有问题只要修改这个类就行，而其继承这个类的子类往往不需要就修改

### 3. 怎样理解多态

```python
class MiniOS(object):
    """MiniOS 操作系统类 """
    def __init__(self, name):
        self.name = name
        self.apps = []  # 安装的应用程序名称列表

    def __str__(self):
        return "%s 安装的软件列表为 %s" % (self.name, str(self.apps))

    def install_app(self, app):
        # 判断是否已经安装了软件
        if app.name in self.apps:
            print("已经安装了 %s，无需再次安装" % app.name)
        else:
            app.install()
            self.apps.append(app.name)


class App(object):
    def __init__(self, name, version, desc):
        self.name = name
        self.version = version
        self.desc = desc

    def __str__(self):
        return "%s 的当前版本是 %s - %s" % (self.name, self.version, self.desc)

    def install(self):
        print("将 %s [%s] 的执行程序复制到程序目录..." % (self.name, self.version))


class PyCharm(App):
    pass


class Chrome(App):
    def install(self):
        print("正在解压缩安装程序...")
        super().install()


linux = MiniOS("Linux")
print(linux)

pycharm = PyCharm("PyCharm", "1.0", "python 开发的 IDE 环境")
chrome = Chrome("Chrome", "2.0", "谷歌浏览器")

linux.install_app(pycharm)
linux.install_app(chrome)
linux.install_app(chrome)

print(linux)
```

运行结果

```
Linux 安装的软件列表为 []
将 PyCharm [1.0] 的执行程序复制到程序目录...
正在解压缩安装程序...
将 Chrome [2.0] 的执行程序复制到程序目录...
已经安装了 Chrome，无需再次安装
Linux 安装的软件列表为 ['PyCharm', 'Chrome']
```
