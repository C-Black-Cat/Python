# 03-mysql与Python交互

## 1. 准备数据

### 创建数据表

```sql
-- 创建 "京东" 数据库
create database jing_dong charset=utf8;

-- 使用 "京东" 数据库
use jing_dong;

-- 创建一个商品goods数据表
create table goods(
    id int unsigned primary key auto_increment not null,
    name varchar(150) not null,
    cate_name varchar(40) not null,
    brand_name varchar(40) not null,
    price decimal(10,3) not null default 0,
    is_show bit not null default 1,
    is_saleoff bit not null default 0
);
```

### 插入数据

```sql
-- 向goods表中插入数据

insert into goods values(0,'r510vc 15.6英寸笔记本','笔记本','华硕','3399',default,default); 
insert into goods values(0,'y400n 14.0英寸笔记本电脑','笔记本','联想','4999',default,default);
insert into goods values(0,'g150th 15.6英寸游戏本','游戏本','雷神','8499',default,default); 
insert into goods values(0,'x550cc 15.6英寸笔记本','笔记本','华硕','2799',default,default); 
insert into goods values(0,'x240 超极本','超级本','联想','4880',default,default); 
insert into goods values(0,'u330p 13.3英寸超极本','超级本','联想','4299',default,default); 
insert into goods values(0,'svp13226scb 触控超极本','超级本','索尼','7999',default,default); 
insert into goods values(0,'ipad mini 7.9英寸平板电脑','平板电脑','苹果','1998',default,default);
insert into goods values(0,'ipad air 9.7英寸平板电脑','平板电脑','苹果','3388',default,default); 
insert into goods values(0,'ipad mini 配备 retina 显示屏','平板电脑','苹果','2788',default,default); 
insert into goods values(0,'ideacentre c340 20英寸一体电脑 ','台式机','联想','3499',default,default); 
insert into goods values(0,'vostro 3800-r1206 台式电脑','台式机','戴尔','2899',default,default); 
insert into goods values(0,'imac me086ch/a 21.5英寸一体电脑','台式机','苹果','9188',default,default); 
insert into goods values(0,'at7-7414lp 台式电脑 linux ）','台式机','宏碁','3699',default,default); 
insert into goods values(0,'z220sff f4f06pa工作站','服务器/工作站','惠普','4288',default,default); 
insert into goods values(0,'poweredge ii服务器','服务器/工作站','戴尔','5388',default,default); 
insert into goods values(0,'mac pro专业级台式电脑','服务器/工作站','苹果','28888',default,default); 
insert into goods values(0,'hmz-t3w 头戴显示设备','笔记本配件','索尼','6999',default,default); 
insert into goods values(0,'商务双肩背包','笔记本配件','索尼','99',default,default); 
insert into goods values(0,'x3250 m4机架式服务器','服务器/工作站','ibm','6888',default,default); 
insert into goods values(0,'商务双肩背包','笔记本配件','索尼','99',default,default);
```



## 2. SQL演练

### 2.1. SQL语句的强化

- 查询类型cate_name为 '超极本' 的商品名称、价格

```sql
select name,price from goods where cate_name = '超级本';
```

![image-20221011171704377](D:\Typora\my_file\图片\image-20221011171704377.png)

- 显示商品的种类

```sql
select cate_name from goods group by cate_name;
select distinct cate_name from goods;
```

![image-20221011171725392](D:\Typora\my_file\图片\image-20221011171725392.png)

- 求所有电脑产品的平均价格,并且保留两位小数

```sql
select round(avg(price),2) as avg_price from goods;
```

![image-20221011171943008](D:\Typora\my_file\图片\image-20221011171943008.png)

- 显示每种商品的平均价格

```sql
select cate_name,avg(price) from goods group by cate_name;
```

![image-20221011172238259](D:\Typora\my_file\图片\image-20221011172238259.png)

- 查询每种类型的商品中 最贵、最便宜、平均价、数量

```sql
select cate_name,max(price),min(price),avg(price),count(*) from goods group by cate_name;
```

![image-20221011172704659](D:\Typora\my_file\图片\image-20221011172704659.png)

- 查询所有价格大于平均价格的商品，并且按价格降序排序

```sql
select id,name,price from goods 
where price > (select round(avg(price),2) as avg_price from goods) 
order by price desc;
```

![image-20221011173018319](D:\Typora\my_file\图片\image-20221011173018319.png)

- 查询每种类型中最贵的电脑信息

```sql
select * from goods
inner join 
    (
        select
        cate_name, 
        max(price) as max_price, 
        min(price) as min_price, 
        avg(price) as avg_price, 
        count(*) from goods group by cate_name
    ) as goods_new_info 
on goods.cate_name=goods_new_info.cate_name and goods.price=goods_new_info.max_price;
```

![image-20221011173802493](D:\Typora\my_file\图片\image-20221011173802493.png)

### 2.2. 创建 "商品分类"" 表

```sql
-- 创建商品分类表
create table if not exists goods_cates(
    id int unsigned primary key auto_increment,
    name varchar(40) not null
);
```

- 查询goods表中商品的种类

```sql
select cate_name from goods group by cate_name;
```

- 将分组结果写入到goods_cates数据表

```sql
insert into goods_cates (name) select cate_name from goods group by cate_name;

-- 这里的 insert 语句不需要写 values ，写了会报错。
```

![image-20221011221834336](D:\Typora\my_file\图片\image-20221011221834336.png)

### 2.3. 同步表数据

- 通过goods_cates数据表来更新goods表

```sql
update goods as g inner join goods_cates as c on g.cate_name=c.name set g.cate_name=c.id;
```

![image-20221011221926138](D:\Typora\my_file\图片\image-20221011221926138.png)

### 2.4. 创建 "商品品牌表" 表

- 通过 `create...select` 来创建数据表并且同时写入记录，一步到位。

```sql
-- select brand_name from goods group by brand_name;

-- 在创建数据表的时候一起插入数据
-- 注意: 需要对 brand_name 用as起别名，否则name字段就没有值。
create table goods_brands (
    id int unsigned primary key auto_increment,
    name varchar(40) not null) select brand_name as name from goods group by brand_name;
```

> 这个创建表的语句，如果执意brand_name不写别名，并且name加上default默认值或者去掉not null，那么创建出来的表会多出一列字段为brand_name的数据，且select查询出的数据都存放到brand_name里去了。
>
> 因为表中字段为name，那么后面的select语句查出来的字段名也要为name才能够导入到表中的name字段中，否则就可能会给表新加一个字段，而表中的name为null或默认值。

### 2.5. 同步数据

- 通过goods_brands数据表来更新goods数据表

```sql
update goods as g inner join goods_brands as b on g.brand_name=b.name set g.brand_name=b.id;
```

### 2.6. 修改表结构

- 查看 `goods` 的数据表结构,会发现 `cate_name` 和 `brand_name`对应的类型为 `varchar` 但是存储的都是数字。

```sql
desc goods;
```

- 通过`alter table`语句修改表结构

```sql
alter table goods  
change cate_name cate_id int unsigned not null,
change brand_name brand_id int unsigned not null;
```

### 2.7. 外键

- 分别在 `goods_cates` 和 `goods_brands`表中插入记录

```sql
insert into goods_cates(name) values ('路由器'),('交换机'),('网卡');
insert into goods_brands(name) values ('海尔'),('清华同方'),('神舟');
```

- 在 `goods` 数据表中写入任意记录

```sql
insert into goods (name,cate_id,brand_id,price)
values('LaserJet Pro P1606dn 黑白激光打印机', 12, 4,'1849');
```

- 查询所有商品的详细信息 (通过内连接)

```sql
select g.id,g.name,c.name,b.name,g.price from goods as g
inner join goods_cates as c on g.cate_id=c.id
inner join goods_brands as b on g.brand_id=b.id;
```

- 查询所有商品的详细信息 (通过左连接)

```sql
select g.id,g.name,c.name,b.name,g.price from goods as g
left join goods_cates as c on g.cate_id=c.id
left join goods_brands as b on g.brand_id=b.id;
```

- **如何防止无效信息的插入,就是可以在插入前判断类型或者品牌名称是否存在呢? 可以使用之前讲过的外键来解决。**
- 外键约束：对数据的有效性进行验证。
- 关键字：`foreign key`，只有 `innodb` 数据库引擎 支持外键约束。
- 对于已经存在的数据表 如何更新外键约束？

```sql
-- 给brand_id 添加外键约束成功
alter table goods add foreign key (brand_id) references goods_brands(id);
-- 给cate_id 添加外键失败
-- 会出现1452错误
-- 错误原因:已经添加了一个不存在的cate_id值12,因此需要先删除
alter table goods add foreign key (cate_id) references goods_cates(id);
```

- **如何在创建数据表的时候就设置外键约束呢?**
- **注意**：`goods` 中的 cate_id 的类型一定要和 `goods_cates` 表中的 id 类型一致。

```sql
create table goods(
    id int primary key auto_increment not null,
    name varchar(40) default '',
    price decimal(5,2),
    cate_id int unsigned,
    brand_id int unsigned,
    is_show bit default 1,
    is_saleoff bit default 0,
    foreign key(cate_id) references goods_cates(id),
    foreign key(brand_id) references goods_brands(id)
);
```

- 如何取消外键约束

```sql
-- 需要先获取外键约束名称,该名称系统会自动生成,可以通过查看表创建语句来获取名称
show create table goods;
-- 获取名称之后就可以根据名称来删除外键约束
alter table goods drop foreign key 外键名称;
-- 这个外键名称通过 show create table goods; 查看
```

![image-20221011233315109](D:\Typora\my_file\图片\image-20221011233315109.png)

这里面的 `goods_ibfk_1` 和 `goods_ibfk_2` 就是外键名称。

- **在实际开发中,很少会使用到外键约束,会极大的降低表更新的效率**

## 3. Python 中操作 MySQL 步骤

![img](D:\Typora\my_file\图片\WechatIMG9052-16655531000321.jpeg)

### 3.1 引入模块

- 在py文件中引入pymysql模块

```python
from pymysql import *
```

**注意**：

```
Ubuntu中一般Python的版本为3.5.x，需要注意几点
	1. pip19.1.1 以上版本不支持 Python3.5.x版本了，安装高版本的pip使用时会报错。
	2. pymysql 1.0.0及以上的版本不支持Python3.5.x了，就算能够安装，导入时会报错。
```



### 3.2 Connection 对象

- 用于建立与数据库的连接
- 创建对象：调用connect()方法

```python
conn=connect(参数列表)
```

- 参数 `host`：连接的mysql主机，如果本机是'localhost'
- 参数 `port`：连接的mysql主机的端口，默认是3306
- 参数 `database`：数据库的名称
- 参数 `user`：连接的用户名
- 参数 `password`：连接的密码
- 参数 `charset`：通信采用的编码方式，推荐使用utf8

#### 对象的方法

- `close()` 关闭连接
- `commit()` 提交
- `cursor()` 返回Cursor对象，用于执行sql语句并获得结果

### 3.3 Cursor对象

- 用于执行sql语句，使用频度最高的语句为select、insert、update、delete
- 获取Cursor对象：调用Connection对象的cursor()方法

```python
cs1=conn.cursor()
```

#### 对象的方法

- `close()` 关闭
- `execute(operation [, parameters ])` 执行语句，返回受影响的行数，主要用于执行insert、update、delete 语句，也可以执行 create、alter、drop 等语句
- `fetchone()` 执行查询语句时，获取查询结果集的第一个行数据，返回一个元组
- `fetchall()` 执行查询时，获取结果集的所有行，一行构成一个元组，再将这些元组装入一个元组返回

#### 对象的属性

- `rowcount` 只读属性，表示最近一次 `execute()` 执行后受影响的行数
- `connection` 获得当前连接对象

## 4. Python中进行增删改查

### 4.1 增删改

- `commit()` 
- 提交之前的操作，如果之前已经之执行过多次的 execute，那么就都进行提交.

```python
conn.commit()
```

> python中 `execute` 执行 insert 数据的操作时，需要再执行 commit 来提交这些操作，数据库的修改才会生效，才会保存这些数据，不提交就不会修改。但是只要是 execute 执行了的操作，不管有没有使用 commute 生效，id 都会自增。

- `rollback()`
- 回滚 execute 执行的操作。

```python
conn.rollback()
```

> 若 execute 执行了几条错误的 insert 数据的语句，此时还没有 commit 提交，可以执行 rollback 来回滚操作，此时执行 commit ，则之前的错误修改不会生效。但是 id 还是会自增。

**问题**：关于 id 的自增

```
为什么要execute一执行insert语句id就要自增呢，为什么不能等到执行了commit后，id再自增数据还没储存到数据库中但是id却自增了，这是不是不合理？
之所以要这样设置，是为了处理并发插入数据时的情况，当多条数据同一时间插入时，若等到执行commit才自增id，这就需要插入语句一条条生效，大大降低了语句的执行效率。
```



```python
from pymysql import *

def main():
    # 创建Connection连接
    conn = connect(host='localhost',port=3306,database='jing_dong',user='root',password='mysql',charset='utf8')
    # 获得Cursor对象
    cs1 = conn.cursor()
    # 执行insert语句，并返回受影响的行数：添加一条数据
    # 增加
    count = cs1.execute('insert into goods_cates(name) values("硬盘")')
    #打印受影响的行数
    print(count)

    count = cs1.execute('insert into goods_cates(name) values("光盘")')
    print(count)

    # # 更新
    # count = cs1.execute('update goods_cates set name="机械硬盘" where name="硬盘"')
    # # 删除
    # count = cs1.execute('delete from goods_cates where id=6')

    # 提交之前的操作，如果之前已经之执行过多次的execute，那么就都进行提交
    conn.commit()

    # 关闭Cursor对象
    cs1.close()
    # 关闭Connection对象
    conn.close()

if __name__ == '__main__':
    main()
```

### 4.2 查询一行数据

```py
from pymysql import *

def main():
    # 创建Connection连接
    conn = connect(host='localhost',port=3306,user='root',password='mysql',database='jing_dong',charset='utf8')
    # 获得Cursor对象
    cs1 = conn.cursor()
    # 执行select语句，并返回受影响的行数：查询一条数据
    count = cs1.execute('select id,name from goods where id>=4')
    # 打印受影响的行数
    print("查询到%d条数据:" % count)

    for i in range(count):
        # 获取查询的结果
        result = cs1.fetchone()
        # 打印查询的结果
        print(result)
        # 获取查询的结果

    # 关闭Cursor对象
    cs1.close()
    conn.close()

if __name__ == '__main__':
    main()
```

### 4.3 查询多行数据

```python
from pymysql import *

def main():
    # 创建Connection连接
    conn = connect(host='localhost',port=3306,user='root',password='mysql',database='jing_dong',charset='utf8')
    # 获得Cursor对象
    cs1 = conn.cursor()
    # 执行select语句，并返回受影响的行数：查询一条数据
    count = cs1.execute('select id,name from goods where id>=4')
    # 打印受影响的行数
    print("查询到%d条数据:" % count)

    # for i in range(count):
    #     # 获取查询的结果
    #     result = cs1.fetchone()
    #     # 打印查询的结果
    #     print(result)
    #     # 获取查询的结果

    result = cs1.fetchall()
    print(result)

    # 关闭Cursor对象
    cs1.close()
    conn.close()

if __name__ == '__main__':
    main()
```

小程序：京东数据查询

```python
from pymysql import *


class JD(object):
    def __init__(self):
        # 创建链接
        self.conn = connect(host='localhost',port=3306,database='jing_dong',user='root',password='root',charset='utf8')
        self.cursor = self.conn.cursor()
    
    def __del__(self):
        # 关闭链接
        self.cursor.close()
        self.conn.close()
        
    def select_execute(self,sql):
        # 执行SQL语句
        self.cursor.execute(sql)
        for items in self.cursor.fetchall():
            print(items)
        print("\n")

    def select_all_goods(self):
    # 1. 查询所有商品
        sql = "select * from goods"
        self.select_execute(sql)

    def select_goods_cates(self):
    # 2. 查询商品的种类
        sql = "select * from goods_cates"
        self.select_execute(sql)

    def select_goods_brands(self):
    # 3. 查询商品的品牌
        sql = "select * from goods_brands"
        self.select_execute(sql)
    
    def get_option(self):
        print("----------京东商城---------")
        print("1. 查询所有商品")
        print("2. 查询商品的种类")
        print("3. 查询商品的品牌")
        print("4. 退出")
        print("---------------------------")
        return input("请输入你的选择：")

    def run(self):
        while True:
            opt = self.get_option()
            if opt == "1":
                self.select_all_goods()
            elif opt == "2":
                self.select_goods_cates()
            elif opt == "3":
                self.select_goods_brands()
            elif opt == "4":
                print("Bye!")
                break
            else:
                print("请输入正确的序号")

def main():
    jd = JD()
    jd.run()

if __name__ == "__main__":
    main()
```

## 5. 参数化

- sql语句的参数化，可以有效防止 **sql注入**
- 注意：此处不同于python的字符串格式化，全部使用`%s`占位

### SQL注入

```py
from pymysql import *


class JD(object):

    def __init__(self):
        # 创建链接
        self.conn = connect(host='localhost',port=3306,database='jing_dong',user='root',password='root',charset='utf8')
        self.cursor = self.conn.cursor()
    
    def __del__(self):
        # 关闭链接
        self.cursor.close()
        self.conn.close()
        
    def select_execute(self,sql):
        # 执行SQL语句
        self.cursor.execute(sql)
        for items in self.cursor.fetchall():
            print(items)
        print("\n")

    def select_all_goods(self):
    # 1. 查询所有商品
        sql = "select * from goods"
        self.select_execute(sql)

    def select_goods_cates(self):
    # 2. 查询商品的种类
        sql = "select * from goods_cates"
        self.select_execute(sql)

    def select_goods_brands(self):
    # 3. 查询商品的品牌
        sql = "select * from goods_brands"
        self.select_execute(sql)

    def select_info_by_name(self):
        file_name = input("请输入商品的名字: ")
        sql = """select * from goods where name='%s';""" % file_name
        self.select_execute(sql)
    
    def get_option(self):
        print("----------京东商城---------")
        print("1. 查询所有商品")
        print("2. 查询商品的种类")
        print("3. 查询商品的品牌")
        print("4. 根据名字查询商品")
        print("5. 退出")
        print("---------------------------")
        return input("请输入你的选择：")
        

    def run(self):
        while True:
            opt = self.get_option()
            if opt == "1":
                self.select_all_goods()
            elif opt == "2":
                self.select_goods_cates()
            elif opt == "3":
                self.select_goods_brands()
            elif opt == "4":
                self.select_info_by_name()
            elif opt == "5":
                print("Bye!")
                break
            else:
                print("请输入正确的序号")


def main():
    jd = JD()
    jd.run()

if __name__ == "__main__":
    main()
```

![1](D:\Typora\my_file\图片\1-16657282567071.gif)

这里并没有按照提示的去输入商品名来查询，而是输入了一串奇怪的字符串 `' or 1=1 or '` 然后就将数据库的全部数据都查出来了。这是怎么做到的呢？将字符串拼接到 SQL 语句中：`select * from goods where '' or 1=1 '';` 这么一看就知道为什么能够查出所有的数据了。这种输入方式就是SQL注入。

> 一想到字符串，一般会想到引号（单引号、双引号），即 `'xxxxxxxx'`。但是要注意引号不是字符串内容的一部分，引号只是各种编程语言为了区分、界定字符串的方法，所以一般输出的字符串最外面是没有引号的，除非引号是字符串内容的一部分。

### 避免SQL注入

```py
from pymysql import *

def main():

    find_name = input("请输入物品名称：")

    # 创建Connection连接
    conn = connect(host='localhost',port=3306,user='root',password='mysql',database='jing_dong',charset='utf8')
    # 获得Cursor对象
    cs1 = conn.cursor()


    # # 非安全的方式
    # # 输入 " or 1=1 or "   (双引号也要输入)
    # sql = 'select * from goods where name="%s"' % find_name
    # print("""sql===>%s<====""" % sql)
    # # 执行select语句，并返回受影响的行数：查询所有数据
    # count = cs1.execute(sql)

    # 安全的方式
    # 构造参数列表
    params = [find_name]
    # 执行select语句，并返回受影响的行数：查询所有数据
    count = cs1.execute('select * from goods where name=%s', params)
    # 注意：
    # 如果要是有多个参数，需要进行参数化
    # 那么params = [数值1, 数值2....]，此时sql语句中有多个%s即可 

    # 打印受影响的行数
    print(count)
    # 获取查询的结果
    # result = cs1.fetchone()
    result = cs1.fetchall()
    # 打印查询的结果
    print(result)
    # 关闭Cursor对象
    cs1.close()
    # 关闭Connection对象
    conn.close()

if __name__ == '__main__':
    main()
```
