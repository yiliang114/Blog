# 轻量应用服务器 MySQL 远程连接踩坑

不算是给阿里云打广告吧，因为被阿里云的“云服务器 ECS” 和 “轻量应用服务器”搞的很蛋疼。很多年前，阿里云的学生机“云翼计划”默认就只有“云服务器 ECS”，所以上个月我准备去买一年的学生机的时候，几乎就选择性忽略框框中的内容，选择系统镜像就直接下单了。

![这里写图片描述](https://pic1.zhimg.com/80/v2-777df1cf8577588921a93dc486c2af41_hd.jpg)

买完之后开开心心安装 node nginx mysql docker 各种之后，等到我需要远程连接 mysql 的时候，花了我几个小时时间踩了一个坑。

## 轻量应用服务器 MySQL 远程连接踩坑

### 1. 登录服务器

我没有去研究过服务器的什么 linux ssh 免密登录，因为我对 linux 不是太熟悉，介绍免密登录的博客看的我有点头大。我采用一种比较简单粗暴并且又能偷懒的方式来“免密登录”。

简单说就是 明文、脚本登录 ，即把 ssh 命令以及密码写入脚本里，要登录就执行脚本就行了。

```
touch aliyun.sh
vi aliyun.sh
```

拷贝以下内容, 记得替换你自己的 ip 和密码：

```
#!/usr/bin/expect

set timeout 10

spawn ssh  root@xxx.xx.xx.xxx

expect {

"*password:" {send "password\r"}
}

interact
```

每次需要登录服务器的时候就 `./aliyun.sh` 就行了。文件不泄露，一般也安全的吧。虽然办法土，但还算是节约了几秒钟的时间，主要还是为了偷懒。（为了安全起见，公司的服务器就别这么搞了，不过一般公司服务器也要安全的话 也会用 pin + token 来保证。 who cares ？）

### 2. 安装 mysql 相关

#### 查看是否安装 Mysql

```
sudo netstat -tap | grep mysql
```

如果为空则没有安装，进行安装：

```
sudo apt-get install mysql-server mysql-client
```

根据提示输入密码，然后确认

#### Mysql 连接测试

```
mysql -u root -h localhost -p
```

启动，停止，重启 Mysql 命令

```
sudo service mysql start
sudo service mysql stop
sudo service mysql restart
```

#### Ubuntu 安装 mysql 忘记初始密码解决方法

```
//打开这个文件 /etc/mysql/debian.cnf
//查看默认分配的密码
[client]
host = localhost
user = debian-sys-maint
password = eyPDN7kavhmjCZUn （记住这个密码）
socket = [表情]ar/run/mysqld/mysqld.sock
```

### 输入命令进入 mysql 修改用户密码

```
// 输入命令后把上面的密码粘贴进去
mysql -u debian-sys-maint -p
//进入到mysql界面厚修改密码
update mysql.user set authentication_string=password('newpassword') where user='root';

//都要使用刷新权限列表
flush privileges;
```

### 3. 终于开始要远程连接 mysql 了...

如果是完全按照上面的操作，直接打开 navicat ，输入 ip、port 这些信息 test connect 的时候一定会报下面的这种错误的：

```
2003 - Can't connect to MySQL server on 'xxx.xx.xx.xxx' (61 "Connection refused")
```

好的，我知道了，那我开一下防火墙端口，设置一个 mysql 的账号用来远程连接吧。这些网上基本都有很详细的教程，我简单列一下命令：（主要也是给我自己偷懒，知道大概的步骤是怎么样的，但是命令太长，记不住，浪费几秒钟。）

```
sudo su

// 进入mysql
mysql -u root -p

// 新建一个有远程连接权限的账号
//$username表示用户名,%表示所有的电脑都可以连接,也可以设置某个ip地址运行连接,$password表示密码
GRANT ALL PRIVILEGES ON *.* TO '$username'@'%' IDENTIFIED BY '$password' WITH GRANT OPTION;

flush privileges;

// 查看所有用户和权限
SELECT DISTINCT CONCAT('User: ''',user,'''@''',host,''';') AS query FROM mysql.user;

exit;

// 修改mysql配置
vim /etc/mysql/mysql.conf.d/mysqld.cnf // 如果你是按照我的方式安装的mysql， bind-address 这一行是在这个文件

#bind-address = 127.0.0.1 这一行注释

// 重启mysql
/etc/init.d/mysql restart
```

这个时候 `talent ip 3306` 还不通，说明防火墙 3306 端口没有开。按照网上所介绍的阿里云安全组规则去打开端口，具体也不介绍了，网上很多，看图也能明白。

![这里写图片描述](https://pic1.zhimg.com/80/v2-8852fbf2c74555fceb4e9c6d8b596668_hd.jpg)

![这里写图片描述](https://pic1.zhimg.com/80/v2-e22f395fc0cb034514fc782980f86496_hd.jpg)

舒服了，终于写完了，test connect 一下，报错：

```
MYSQL ERROR 2003 (HY000): Can't connect to MySQL server on 'xxx.xxx.xx.xxx'
```

啊，不应该啊，我都是按照流程来的，于是重新看了一遍，发现没有漏掉的、也没有不对的。很郁闷啊，觉得自己的操作完全没有问题，大家都说配置一下安全组就好了，一定是阿里云想坑我！

接着排查，但都正常。

```
ps -ef | grep mysql
netstat -tlanp | grep 3306
```

...
各种折腾中...
...

**下面就是我想说的坑点了：** 我发现我买的竟然是 **轻量应用服务器**。。。 讲道理我也不知道这个是个什么鬼，后来就发现了

![这里写图片描述](https://pic4.zhimg.com/80/v2-777df1cf8577588921a93dc486c2af41_hd.jpg)

原来左侧是由切换的。后来发现了轻量应用服务器的防火墙配置：

![这里写图片描述](https://pic3.zhimg.com/80/v2-278bf0987d3037bb8e3f8173755a00b6_hd.jpg)

这里配置了 3306 之后， `talent ip 3306` 就通了，记得应用类型要选 `MYSQL` ，接着 test connect 就绿了。

哇！ 所以我到底是被谁坑了！

## 坑都被坑了之我就看一下有没有亏

轻量应用服务器优势，我只看见了 40g ssd，其他的我也不管，大概是没有亏？

## 总结

我其实的确是想吐槽的，我一直以为我买的是普通的云服务器，网上搜远程连接 mysql 的时候，几乎是没有关于轻量应用服务器滴，这就又引导我到了“李鬼”那里。看了一下阿里云的论坛，似乎现在也不能将我这个一年的轻量应用服务器换成 ECS 了。算了算了，也行吧，反正也能用，我也不挑的。

还有啊，你们家阿里云的广告啊，都贴到我们公司门口了！我们家云不要面子的啊！啊西八，我一定会贴回来的。

所以，最终还是我自己坑的我自己 ToT... 菜。
