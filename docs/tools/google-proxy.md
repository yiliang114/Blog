# 腾讯云香港翻墙

## 步骤

以 ubuntu 为例:

1. 使用 ssh 登录到需要设置翻墙中转站的服务器.

2. 获取 root 权限(若非 root 用户登录,可以使用**sudo su**).

3. 执行**sudo apt-get update** 更新系统.

4. 执行**sudo apt-get install python-pip** 安装 python.

5. 执行 **sudo pip install shadowsocks **安装 shadowsocks

6. 启动 shadowsocks:

   方法(1):直接命令启动

   **sudo ssserver -p PORT -k PASSWORD -m aes-256-cfb --log-file /var/log/shadowsocks.log -d start**

   方法(2):指定配置文件启动

   **sudo touch /etc/shadowsocks.json**

   **sudo vim /etc/shadowsocks.json**

   ```
   {
    "server": "0.0.0.0",
    "server_port": 8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password": "PASSWORD",
    "timeout": 300,
    "method":"aes-256-cfb"
   }
   ```

   执行命令:**sudo ssserver -c /etc/shadowsocks.json -d start**

## 报错

搭建 ss 之后连上代理 访问 google，发现报错： 500 Internal Privoxy Error

登录服务器之后，查看一下 8388 端口的占用状况：

`netstat -noa | grep :8388`
![这里写图片描述](https://img-blog.csdn.net/20180507204011558?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dyZWVrTXJ6eko=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
能够输出相关端口信息说明运行正常，那么通常是因为服务器防火墙的端口没有打开，比如说我这里就是 因为刚买的服务器，在配置安全组中没有将 8388 端口开放给外网访问，因为在我的 pc 上虽然开启了 ss，但是访问 google 会报 500 错误。

那么接下来只要防火墙端口开放就好了。至于是手动选择开放 8388 端口 还是 在服务器控制台配置安全组配置一下，就看你自己了。
![这里写图片描述](https://img-blog.csdn.net/20180507204019631?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dyZWVrTXJ6eko=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
