# Ubuntu虚拟机出现ping: unknown host www.baidu.com

### 方法一：有效

- 首先查看 `/etc/network/interfaces`

![image-20221012000848496](D:\Typora\my_file\图片\image-20221012000848496.png)

只设置了动态获取IP。

```shell
# 最后一行写入dns-nameservers 8.8.8.8   // (可以写入多个域名服务器，空格隔开即可)
dns-nameservers 8.8.8.8 114.114.114.114
```

需要重启Ubuntu。

### 方法二：无效

- 网上有的说修改 `/etc/resolv.conf`，在里面加上DNS，再重启网络，但是这种方法不行。

![image-20221012001031323](D:\Typora\my_file\图片\image-20221012001031323.png)

这里面注释都说了不能直接修改这个文件，修改了也没用，一重启网络就会被覆盖。

### 方法三：不好使

- 修改 `/etc/resolvconf/resolv.conf.d/base`（这个文件默认是空的）

```shell
sudo vi /etc/resolvconf/resolv.conf.d/base

# 在里面插入：
nameserver 114.114.114.114
nameserver 8.8.8.8

# 如果有多个DNS就一行一个
# 修改好保存，然后执行：

resolvconf -u

# 再看/etc/resolv.conf，最下面就多了2行：

cat /etc/resolv.conf
```

![image-20221012002003334](D:\Typora\my_file\图片\image-20221012002003334.png)

![image-20221012002118130](D:\Typora\my_file\图片\image-20221012002118130.png)

这个方法里面说要重启网络 `/etc/init.d/networking restart` 或者 `service networking restart` 但是我一重启，就又 ping 不通了，再次执行 `resolvconf -u` 才能 ping 通。