# 04-方法解析顺序表MRO

## 多继承以及MRO顺序

### 1. 单独调用父类的方法

```python
# coding=utf-8

print("******多继承使用类名.__init__ 发生的状态******")
class Parent(object):
    def __init__(self, name):
        print('parent的init开始被调用')
        self.name = name
        print('parent的init结束被调用')

class Son1(Parent):
    def __init__(self, name, age):
        print('Son1的init开始被调用')
        self.age = age
        Parent.__init__(self, name)
        print('Son1的init结束被调用')

class Son2(Parent):
    def __init__(self, name, gender):
        print('Son2的init开始被调用')
        self.gender = gender
        Parent.__init__(self, name)
        print('Son2的init结束被调用')

class Grandson(Son1, Son2):
    def __init__(self, name, age, gender):
        print('Grandson的init开始被调用')
        Son1.__init__(self, name, age)  # 单独调用父类的初始化方法
        Son2.__init__(self, name, gender)
        print('Grandson的init结束被调用')

gs = Grandson('grandson', 12, '男')
print('姓名：', gs.name)
print('年龄：', gs.age)
print('性别：', gs.gender)

print("******多继承使用类名.__init__ 发生的状态******\n\n")
```

运行结果:

```
******多继承使用类名.__init__ 发生的状态******
Grandson的init开始被调用
Son1的init开始被调用
parent的init开始被调用
parent的init结束被调用
Son1的init结束被调用
Son2的init开始被调用
parent的init开始被调用
parent的init结束被调用
Son2的init结束被调用
Grandson的init结束被调用
姓名： grandson
年龄： 12
性别： 男
******多继承使用类名.__init__ 发生的状态******
```

### 2. 多继承中super调用有所父类的被重写的方法

```python
print("******多继承使用super().__init__ 发生的状态******")
class Parent(object):
    def __init__(self, name, *args, **kwargs):  # 为避免多继承报错，使用不定长参数，接受参数
        print('parent的init开始被调用')
        self.name = name
        print('parent的init结束被调用')

class Son1(Parent):
    def __init__(self, name, age, *args, **kwargs):  # 为避免多继承报错，使用不定长参数，接受参数
        print('Son1的init开始被调用')
        self.age = age
        super().__init__(name, *args, **kwargs)  # 为避免多继承报错，使用不定长参数，接受参数
        print('Son1的init结束被调用')

class Son2(Parent):
    def __init__(self, name, gender, *args, **kwargs):  # 为避免多继承报错，使用不定长参数，接受参数
        print('Son2的init开始被调用')
        self.gender = gender
        super().__init__(name, *args, **kwargs)  # 为避免多继承报错，使用不定长参数，接受参数
        print('Son2的init结束被调用')

class Grandson(Son1, Son2):
    def __init__(self, name, age, gender):
        print('Grandson的init开始被调用')
        # 多继承时，相对于使用类名.__init__方法，要把每个父类全部写一遍
        # 而super只用一句话，执行了全部父类的方法，这也是为何多继承需要全部传参的一个原因
        # super(Grandson, self).__init__(name, age, gender)
        super().__init__(name, age, gender)
        print('Grandson的init结束被调用')

print(Grandson.__mro__)

gs = Grandson('grandson', 12, '男')
print('姓名：', gs.name)
print('年龄：', gs.age)
print('性别：', gs.gender)
print("******多继承使用super().__init__ 发生的状态******\n\n")
```

> - 当 Grandson类中的 `super().__init__(name, age, gender)` 运行时，会调用 父类的 `__init__` 方法，因为 `Grandson(Son1, Son2)` 中先写的是 Son1 类，所以会调用 Son1 类，因为 Grandson 类中的 `super()` 方法只调用了一次，所以它只会调用一次父类，因此 Son2 类不会被调用；Son1 类被调用后，Son1 中也有  `super()` 方法，但是 Son1 在调用 `super()` 方法时却不会调用它的父类 Parent，`super()` 方法是调用当前类的父类，但又不完全是，`super()` 的调用顺序是按照 `类.__mro__` 中的值的先后顺序。
>
> - 也就是说 `super()` 方法判断父类的标准是根据 `类.__mro__` 里面值的顺序。
>
> 注：`类.__mro__` 中的顺序是根据Python解释器中的 C3 算法计算出的。

**运行结果：**

```
******多继承使用super().__init__ 发生的状态******
(<class '__main__.Grandson'>, <class '__main__.Son1'>, <class '__main__.Son2'>, <class '__main__.Parent'>, <class 'object'>)
Grandson的init开始被调用
Son1的init开始被调用
Son2的init开始被调用
parent的init开始被调用
parent的init结束被调用
Son2的init结束被调用
Son1的init结束被调用
Grandson的init结束被调用
姓名： grandson
年龄： 12
性别： 男
******多继承使用super().__init__ 发生的状态******
```

#### 注意

> 1. 以上2个代码执行的结果不同。
> 2. 如果2个子类中都继承了父类，当在子类中通过父类名调用时，parent被执行了2次。
> 3. 如果2个子类中都继承了父类，当在子类中通过super调用时，parent被执行了1次。

#### 调用父类的三种方法

>1. `父类.方法()`：直接调用所指定的父类的方法。
>2. `super.方法()`：当前类的父类是按照 `最初调用super()方法的类.__mro__` 中值的顺序来确定的。
>3. `super(类名, self).方法()`：与上一种方法一样，只是可以指定要调用哪个类的父类。
>
>`super()` 方法能够保证多继承的情况下父类只会被调用一次。

### 3. 单继承中super

```python
print("******单继承使用super().__init__ 发生的状态******")
class Parent(object):
    def __init__(self, name):
        print('parent的init开始被调用')
        self.name = name
        print('parent的init结束被调用')

class Son1(Parent):
    def __init__(self, name, age):
        print('Son1的init开始被调用')
        self.age = age
        super().__init__(name)  # 单继承不能提供全部参数
        print('Son1的init结束被调用')

class Grandson(Son1):
    def __init__(self, name, age, gender):
        print('Grandson的init开始被调用')
        super().__init__(name, age)  # 单继承不能提供全部参数
        print('Grandson的init结束被调用')

gs = Grandson('grandson', 12, '男')
print('姓名：', gs.name)
print('年龄：', gs.age)
#print('性别：', gs.gender)
print("******单继承使用super().__init__ 发生的状态******\n\n")
```

### 总结

1. `super().__init__`相对于`类名.__init__`，在单继承上用法基本无差。
2. 但在多继承上有区别，super方法能保证每个父类的方法只会执行一次，而使用类名的方法会导致方法被执行多次，具体看前面的输出结果。
3. 多继承时，使用super方法，对父类的传参数，应该是由于python中super的算法导致的原因，必须把参数全部传递，否则会报错。
4. 单继承时，使用super方法，则不能全部传递，只能传父类方法所需的参数，否则会报错。
5. 多继承时，相对于使用`类名.__init__`方法，要把每个父类全部写一遍, 而使用super方法，只需写一句话便执行了全部父类的方法，这也是为何多继承需要全部传参的一个原因。

### 小试牛刀(以下为面试题)

> 以下的代码的输出将是什么? 说出你的答案并解释。

```python
class Parent(object):
    x = 1

class Child1(Parent):
    pass

class Child2(Parent):
    pass

print(Parent.x, Child1.x, Child2.x)
Child1.x = 2
print(Parent.x, Child1.x, Child2.x)
Parent.x = 3
print(Parent.x, Child1.x, Child2.x)
```

答案, 以上代码的输出是：

```
1 1 1
1 2 1
3 2 3
```

使你困惑或是惊奇的是关于最后一行的输出是 3 2 3 而不是 3 2 1。为什么改变了 Parent.x 的值还会改变 Child2.x 的值，但是同时 Child1.x 值却没有改变？

这个答案的关键是，在 Python 中，类变量在内部是作为字典处理的。如果一个变量的名字没有在当前类的字典中发现，将搜索祖先类（比如父类）直到被引用的变量名被找到（如果这个被引用的变量名既没有在自己所在的类又没有在祖先类中找到，会引发一个 AttributeError 异常 ）。

因此，在父类中设置 x = 1 会使得类变量 x 在引用该类和其任何子类中的值为 1。这就是因为第一个 print 语句的输出是 1 1 1。

随后，如果任何它的子类重写了该值（例如，我们执行语句 Child1.x = 2），然后，该值仅仅在子类中被改变。这就是为什么第二个 print 语句的输出是 1 2 1。

最后，如果该值在父类中被改变（例如，我们执行语句 Parent.x = 3），这个改变会影响到任何未重写该值的子类当中的值（在这个示例中被影响的子类是 Child2）。这就是为什么第三个 print 输出是 3 2 3。

## *args、**kwargs的另外用处拆包

这两个不定长参数的使用方法看下面这个例子就能明白。

```python
def test2(a, b, *args, **kwargs):
    print("----------")
    print(a)
    print(b)
    print(args)
    print(kwargs)

def test1(a, b, *args, **kwargs):
    print(a)
    print(b)
    print(args)
    print(kwargs)
    test2(a,b,args,kwargs)  # 相当于test2(11,22,(33,44,55,66), {"name":"langwang","age":18})
    test2(a,b,*args,kwargs)  # 相当于test2(11,22,33,44,55,66,{"name":"laowang","age":18})
    test2(a,b,*args,**kwargs) # 相当于test2(11,22,33,44,55,66,name="laowang",age=18)


test1(11,22,33,44,55,66,name="laowang",age=18)
```

输出结果

```
11
22
(33, 44, 55, 66)
{'age': 18, 'name': 'laowang'}
----------
11
22
((33, 44, 55, 66), {'age': 18, 'name': 'laowang'})
{}
----------
11
22
(33, 44, 55, 66, {'age': 18, 'name': 'laowang'})
{}
----------
11
22
(33, 44, 55, 66)
{'age': 18, 'name': 'laowang'}
```

> `*args`： 接收所有剩余的没有名字的参数。如：`(11,22), {"name":"A", "age"=11}`。`args` 中存放的是一个元组。
>
> `**kwargs`： 接收所有含有名字的参数。如：`name="zhangsan", age=18`。`kwargs` 中存放的值是一个字典。