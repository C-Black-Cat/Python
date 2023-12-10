# 05-类对象和实例对象访问属性的区别和property属性

## 再论静态方法和类方法

### 1. 类属性、实例属性

它们在定义和使用中有所区别，而最本质的区别是内存中保存的位置不同，

- 实例属性属于对象
- 类属性属于类

```python
class Province(object):
    # 类属性
    country = '中国'

    def __init__(self, name):
        # 实例属性
        self.name = name


# 创建一个实例对象
obj = Province('山东省')
# 直接访问实例属性
print(obj.name)
# 直接访问类属性
Province.country
```

> 使用 `obj.country = xxx` 这种方式是修改不了类属性的，即使再次调用 `obj.country` 时`country` 的值会变为 `xxx`，但是`Province.country`的值没有改变，要想改变`Province.country`的值（类属性），应该要使用 `obj.__class__.country = xxx`。

由上述代码可以看出【实例属性需要通过对象来访问】【类属性通过类访问】，在使用上可以看出实例属性和类属性的归属是不同的。

其在内容的存储方式类似如下图：

![img](D:\Typora\my_file\图片\QQ20171025-204706@2x-16626190064413.png)

由上图看出：

- 类属性在内存中只保存一份
- 实例属性在每个对象中都要保存一份

#### 应用场景：

- 通过类创建实例对象时，如果每个对象需要具有相同名字的属性，那么就使用类属性，用一份既可

### 2. 实例方法、静态方法和类方法

方法包括：实例方法、静态方法和类方法，**三种方法在内存中都归属于类**，区别在于**调用方式不同**。

- 实例方法：由对象调用；至少一个self参数；执行实例方法时，自动将调用该方法的对象赋值给self；
- 类方法：由类调用； 至少一个cls参数；执行类方法时，自动将调用该方法的类赋值给cls；
- 静态方法：由类调用；无默认参数；

```python
class Foo(object):
    def __init__(self, name):
        self.name = name

    def ord_func(self):
        """ 定义实例方法，至少有一个self参数 """
        # print(self.name)
        print('实例方法')
    # 实例方法传入的 self 参数实际上是将实例对象的引用传入函数。

    @classmethod
    def class_func(cls):
        """ 定义类方法，至少有一个cls参数 """
        print('类方法')
    # 类方法传入的 cls 参数实际上是将类对象的引用出入函数。

    @staticmethod
    def static_func():
        """ 定义静态方法 ，无默认参数"""
        print('静态方法')



f = Foo("中国")
# 调用实例方法
f.ord_func()

# 调用类方法
Foo.class_func()

# 调用静态方法
Foo.static_func()
```

> 为什么要有静态方法？
>
> 就是为了实现一些功能，且不像实例方法和类方法一样需要传递参数（self、cls）。

![img](D:\Typora\my_file\图片\QQ20171025-210349@2x-16626190382455.png)

#### 对比

- 相同点：对于所有的方法而言，均属于类，所以 在内存中也只保存一份
- 不同点：方法调用者不同、调用方法时自动传入的参数不同。实例对象可以有多个，类对象只有一个。

实例对象可以调用：类方法、实例方法、静态方法。

类对象只可以调用：类方法、静态方法。

## property属性

### 1. 什么是property属性

一种用起来像是使用的实例属性一样的特殊属性，可以对应于某个方法

```python
# ############### 定义 ###############
class Foo:
    def func(self):
        pass

    # 定义property属性
    @property
    def prop(self):
        pass  # 一般返回一个值

# ############### 调用 ###############
foo_obj = Foo()
foo_obj.func()  # 调用实例方法
foo_obj.prop  # 调用property属性
```

![img](D:\Typora\my_file\图片\QQ20171025-211026@2x-16626292264357.png)

##### property属性的定义和调用要注意一下几点：

- 定义时，在实例方法的基础上添加 @property 装饰器；并有且仅有一个self参数

- 调用时，无需括号

  ```python
    方法：foo_obj.func()
    property属性：foo_obj.prop
  ```

### 2. 简单的实例

> 对于京东商城中显示电脑主机的列表页面，每次请求不可能把数据库中的所有内容都显示到页面上，而是通过分页的功能局部显示，所以在向数据库中请求数据时就要显示的指定获取从第m条到第n条的所有数据 这个分页的功能包括：
>
> - 根据用户请求的当前页和总数据条数计算出 m 和 n
> - 根据m 和 n 去数据库中请求数据

```python
# ############### 定义 ###############
class Pager:
    def __init__(self, current_page):
        # 用户当前请求的页码（第一页、第二页...）
        self.current_page = current_page
        # 每页默认显示10条数据
        self.per_items = 10 

    @property
    def start(self):
        val = (self.current_page - 1) * self.per_items
        return val

    @property
    def end(self):
        val = self.current_page * self.per_items
        return val

# ############### 调用 ###############
p = Pager(1)
p.start  # 就是起始值，即：m
p.end  # 就是结束值，即：n
```

#### 从上述可见

- $\textcolor{green}{Python的property属性的功能是：property属性内部进行一系列的逻辑计算，最终将计算结果返回。}$
- 使用 property 属性来装饰方法，在调用方法时就相当于将方法当做属性来调用，而且不需要考虑传递什么参数的问题，使得方法的调用更加简单。

### 3. property属性的有两种方式

- 装饰器 即：在方法上应用装饰器
- 类属性 即：在类中定义值为property对象的类属性

#### 3.1 装饰器方式

在类的实例方法上应用@property装饰器

Python中的类有`经典类`和`新式类`，`新式类`的属性比`经典类`的属性丰富。（ 如果类继object，那么该类是新式类 ）

##### 经典类，具有一种@property装饰器

什么是**经典类**：在Python2中，定义的类没有继承 `object`。

```python
# ############### 定义 ###############    
class Goods:
    @property
    def price(self):
        return "laowang"
# ############### 调用 ###############
obj = Goods()
result = obj.price  # 自动执行 @property 修饰的 price 方法，并获取方法的返回值
print(result)
```

##### 新式类，具有三种@property装饰器

什么是**新式类**：继承了 `object` 的类。在Python3中，不管在定义类时有没有在括号里写 `object` ，都默认为继承了 `object`。

```python
#coding=utf-8
# ############### 定义 ###############
class Goods:
    """python3中默认继承object类
        以python2、3执行此程序的结果不同，因为只有在python3中才有@xxx.setter  @xxx.deleter
    """
    @property
    def price(self):
        print('@property')

    @price.setter
    def price(self, value):
        print('@price.setter')

    @price.deleter
    def price(self):
        print('@price.deleter')

# ############### 调用 ###############
obj = Goods()
obj.price          # 自动执行 @property 修饰的 price 方法，并获取方法的返回值
obj.price = 123    # 自动执行 @price.setter 修饰的 price 方法，并将  123 赋值给方法的参数
del obj.price      # 自动执行 @price.deleter 修饰的 price 方法
```

**注意**

- 经典类中的属性只有一种访问方式，其对应被 `@property` 修饰的方法
- 新式类中的属性有三种访问方式，并分别对应了三个被`@property`、`@方法名.setter`、`@方法名.deleter` 修饰的方法
- 这三种装饰器只能修改实例对象的属性

由于新式类中具有三种访问方式，我们可以根据它们几个属性的访问特点，分别将三个方法定义为对同一个属性：获取、修改、删除

```python
class Goods(object):

    def __init__(self):
        # 原价
        self.original_price = 100
        # 折扣
        self.discount = 0.8

    @property
    def price(self):
        # 实际价格 = 原价 * 折扣
        new_price = self.original_price * self.discount
        return new_price

    @price.setter
    def price(self, value):
        self.original_price = value

    @price.deleter
    def price(self):
        del self.original_price

obj = Goods()
obj.price         # 获取商品价格
obj.price = 200   # 修改商品原价
del obj.price     # 删除商品原价
```

#### 3.2 类属性方式，创建值为property对象的类属性

- 当使用类属性的方式创建property属性时，`经典类`和`新式类`无区别

```python
class Foo:
    def get_bar(self):
        return 'laowang'

    BAR = property(get_bar)

obj = Foo()
reuslt = obj.BAR  # 自动调用get_bar方法，并获取方法的返回值
print(reuslt)
```

property方法中有个四个参数

![image-20220917151609477](D:\Typora\my_file\图片\image-20220917151609477.png)

- 第一个参数是方法名，调用 `对象.属性` 时自动触发执行方法
- 第二个参数是方法名，调用 `对象.属性 ＝ XXX` 时自动触发执行方法
- 第三个参数是方法名，调用 `del 对象.属性` 时自动触发执行方法
- 第四个参数是字符串，调用 `对象.属性.__doc__` ，此参数是该属性的描述信息

```python
#coding=utf-8
class Foo(object):
    def get_bar(self):
        print("getter...")
        return 'laowang'

    def set_bar(self, value): 
        """必须两个参数"""
        print("setter...")
        return 'set value' + value

    def del_bar(self):
        print("deleter...")
        return 'laowang'

    BAR = property(get_bar, set_bar, del_bar, "description...")

obj = Foo()

obj.BAR  # 自动调用第一个参数中定义的方法：get_bar
obj.BAR = "alex"  # 自动调用第二个参数中定义的方法：set_bar方法，并将“alex”当作参数传入
desc = Foo.BAR.__doc__  # 自动获取第四个参数中设置的值：description...
print(desc)
del obj.BAR  # 自动调用第三个参数中定义的方法：del_bar方法
```

由于`类属性方式`创建property属性具有3种访问方式，我们可以根据它们几个属性的访问特点，分别将三个方法定义为对同一个属性：获取、修改、删除

```python
class Goods(object):

    def __init__(self):
        # 原价
        self.original_price = 100
        # 折扣
        self.discount = 0.8

    def get_price(self):
        # 实际价格 = 原价 * 折扣
        new_price = self.original_price * self.discount
        return new_price

    def set_price(self, value):
        self.original_price = value

    def del_price(self):
        del self.original_price

    PRICE = property(get_price, set_price, del_price, '价格属性描述...')

obj = Goods()
obj.PRICE         # 获取商品价格
obj.PRICE = 200   # 修改商品原价
del obj.PRICE     # 删除商品原价
```

### 4. Django框架中应用了property属性（了解）

WEB框架 Django 的视图中 request.POST 就是使用的类属性的方式创建的属性

```python
class WSGIRequest(http.HttpRequest):
    def __init__(self, environ):
        script_name = get_script_name(environ)
        path_info = get_path_info(environ)
        if not path_info:
            # Sometimes PATH_INFO exists, but is empty (e.g. accessing
            # the SCRIPT_NAME URL without a trailing slash). We really need to
            # operate as if they'd requested '/'. Not amazingly nice to force
            # the path like this, but should be harmless.
            path_info = '/'
        self.environ = environ
        self.path_info = path_info
        self.path = '%s/%s' % (script_name.rstrip('/'), path_info.lstrip('/'))
        self.META = environ
        self.META['PATH_INFO'] = path_info
        self.META['SCRIPT_NAME'] = script_name
        self.method = environ['REQUEST_METHOD'].upper()
        _, content_params = cgi.parse_header(environ.get('CONTENT_TYPE', ''))
        if 'charset' in content_params:
            try:
                codecs.lookup(content_params['charset'])
            except LookupError:
                pass
            else:
                self.encoding = content_params['charset']
        self._post_parse_error = False
        try:
            content_length = int(environ.get('CONTENT_LENGTH'))
        except (ValueError, TypeError):
            content_length = 0
        self._stream = LimitedStream(self.environ['wsgi.input'], content_length)
        self._read_started = False
        self.resolver_match = None

    def _get_scheme(self):
        return self.environ.get('wsgi.url_scheme')

    def _get_request(self):
        warnings.warn('`request.REQUEST` is deprecated, use `request.GET` or '
                      '`request.POST` instead.', RemovedInDjango19Warning, 2)
        if not hasattr(self, '_request'):
            self._request = datastructures.MergeDict(self.POST, self.GET)
        return self._request

    @cached_property
    def GET(self):
        # The WSGI spec says 'QUERY_STRING' may be absent.
        raw_query_string = get_bytes_from_wsgi(self.environ, 'QUERY_STRING', '')
        return http.QueryDict(raw_query_string, encoding=self._encoding)

    # ############### 看这里看这里  ###############
    def _get_post(self):
        if not hasattr(self, '_post'):
            self._load_post_and_files()
        return self._post

    # ############### 看这里看这里  ###############
    def _set_post(self, post):
        self._post = post

    @cached_property
    def COOKIES(self):
        raw_cookie = get_str_from_wsgi(self.environ, 'HTTP_COOKIE', '')
        return http.parse_cookie(raw_cookie)

    def _get_files(self):
        if not hasattr(self, '_files'):
            self._load_post_and_files()
        return self._files

    # ############### 看这里看这里  ###############
    POST = property(_get_post, _set_post)

    FILES = property(_get_files)
    REQUEST = property(_get_request)
```

#### 综上所述:

- 定义property属性共有两种方式，分别是【装饰器】和【类属性】，而【装饰器】方式针对经典类和新式类又有所不同。
- 通过使用property属性，能够简化调用者在获取数据的流程

## property属性-应用

### 1. 私有属性添加getter和setter方法

```python
class Money(object):
    def __init__(self):
        self.__money = 0

    def getMoney(self):
        return self.__money

    def setMoney(self, value):
        if isinstance(value, int):
            self.__money = value
        else:
            print("error:不是整型数字")
```

### 2. 使用property升级getter和setter方法

```python
class Money(object):
    def __init__(self):
        self.__money = 0

    def getMoney(self):
        return self.__money

    def setMoney(self, value):
        if isinstance(value, int):
            self.__money = value
        else:
            print("error:不是整型数字")

    # 定义一个属性，当对这个money设置值时调用setMoney,当获取值时调用getMoney
    money = property(getMoney, setMoney)  

a = Money()
a.money = 100  # 调用setMoney方法
print(a.money)  # 调用getMoney方法
#100
```

### 3. 使用property取代getter和setter方法

- 重新实现一个属性的设置和读取方法,可做边界判定

```python
class Money(object):
    def __init__(self):
        self.__money = 0

    # 使用装饰器对money进行装饰，那么会自动添加一个叫money的属性，当调用获取money的值时，调用装饰的方法
    @property
    def money(self):
        return self.__money

    # 使用装饰器对money进行装饰，当对money设置值时，调用装饰的方法
    @money.setter
    def money(self, value):
        if isinstance(value, int):
            self.__money = value
        else:
            print("error:不是整型数字")

a = Money()
a.money = 100
print(a.money)
```