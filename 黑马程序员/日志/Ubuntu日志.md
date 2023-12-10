# Ubuntu日志

问题：Ubuntu 中 使用 su 命令切换root用户时出现：su 认证失败

```
原因：
	Ubuntu安装后，root用户默认是被锁定了的，不允许登录，也不允许 su root
解决：
	sudo passwd
	输入当前用户的密码
	输入新的root密码
```

问题：Ubuntu创建新用户

```
sudo su		# 切换root用户获取权限
useradd -r -m -s /bin/bash darkframe 	# 创建用户
	-r:建立系统账号
	-m:创建用户的家目录，不加 -m 则 /home目录下无用户家目录
	-s:指定用户登录的shell
passwd darkframe	#设定密码


cat /etc/passwd		# 查看用户属性
su darkframe		# 切换到目录
```

问题：删除用户

```
userdel -r darkframe
	-r:删除 /home/ 目录下的用户文件夹
```

