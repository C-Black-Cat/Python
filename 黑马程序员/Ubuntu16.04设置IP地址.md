# Ubuntu16.04设置IP地址

# Ubuntu中IP地址配置文件：
```shell
# Ubuntu中IP地址配置文件：
/etc/network/interfaces
	# 配置示例：
	# interfaces(5) file used by ifup(8) and ifdown(8)
    auto lo
    iface lo inet loopback

    auto ens33
    # 自动获取IP
    # iface ens33 inet dhcp
    iface ens33 inet static
    address 192.168.119.149
    network 255.255.255.0
    gateway 192.168.119.1
# 更新network配置：
sudo /etc/init.d/networking restart
```