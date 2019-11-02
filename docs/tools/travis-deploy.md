### 使用 travis 部署静态页面

#### 让 travis 登录到服务器

0. 如果 `sudo apt install ruby` 出现 404 的情况就需要先执行 `sudo apt-get update`
1. 登录服务器之后，安装 ruby. `sudo apt install ruby`
1. 通过 ruby 的 gem 安装 travis . `sudo gem install travis`
1. 如果有报错的话一般是需要安装 ruby-dev `sudo apt-get install ruby-dev`
1. 如果 安装 travis 一直卡着不动的话，还是换用国内的 ruby 镜像吧。
1. `gem sources -l` 查看当前镜像，一般默认会显示 `https://rubygems.org/`
1. `gem update --system`
1. `gem sources --add https://gems.ruby-china.com/` 换用国内镜像
1. `gem sources -l` 查看镜像是否替换成功
1. 接下来就愉快地安装 travis `sudo gem install travis`
1. 接着登录 travis， 不然你的服务器怎么知道你是谁。

   ```shell
   $ travis login

   We need your GitHub login to identify you.
   This information will not be sent to Travis CI, only to api.github.com.
   The password will not be displayed.

   Try running with --github-token or --auto if you don't want to enter your password anyway.

   Username: xxx@xxx.xxx
   Password for xxx@xxx.xxx: ***
   Successfully logged in as demo!
   ```

1. https://segmentfault.com/a/1190000011218410#articleHeader3
1. https://www.cnblogs.com/dhuhank/p/5248712.html

有个错误，应该都在服务器上进行操作， ssh 秘钥理解有误，应该是服务器上进行 travis 添加 ssh ，本地的 mac 电脑有啥用？又不是部署到本地的。。。。

### ssh 免密登录

https://blog.stephencode.com/p/ssh-login-no-pwd

### Ruby-dev

https://blog.csdn.net/sp1206/article/details/80430493

### 去掉 \ 转义符，否则会报找不到 ssh 解密文件错误

https://www.jianshu.com/p/1226d159d514

### 只上传某些文件

https://www.jianshu.com/p/1226d159d514

- 脚本需要放在服务器根目录下面， deploy.sh
- travis 可以手动 rebuild

### 报错

```
error: cannot open .git/FETCH_HEAD: Permission denied
```

https://blog.csdn.net/LJFPHP/article/details/79508859

git 权限不够：

```
git config --list  //查看当前的config配置
git config --global user.name "youruser"    //修改用户名
git config --global user.email "你的邮箱"    //修改为你的邮箱
```

还是不行，添加服务器的 ssh 秘钥到 github 上。

### 通过了的配置

```
language: node_js
node_js:
- '8'
branchs:
  only:
  - master
before_install:
- openssl aes-256-cbc -K $encrypted_10f721522491_key -iv $encrypted_10f721522491_iv
  -in id_rsa.enc -out ~/.ssh/id_rsa -d
- chmod 600 ~/.ssh/id_rsa
after_success:
- ssh root@119.23.63.192 StrictHostKeyChecking=no 'cd /usr/local/workspace/kaiyuan/occupy-latrine/ && git pull'

addons:
  ssh_known_hosts: 119.23.63.192

```

https://juejin.im/post/59bf79f66fb9a00a6974e4eb

### 部署静态文件

https://andyliwr.github.io/2018/04/28/travis_node_publish2/

```
after_success:
 - chmod 600 ~/.ssh/id_rsa
 - scp -r dist/ travis@193.112.196.41:/var/www/admin

```

### vue nginx 部署

http://www.cnblogs.com/vipzhou/p/9255552.html
https://www.jb51.net/article/118767.htm

### Travis 自动发布 npm

https://juejin.im/post/5ab39fedf265da23a04979cb

### travis 部署 node

要点在于不要到 服务器上去 npm install
https://www.jb51.net/article/119912.htm

#### 步骤

```
travis login
travis encrypt-file ~/.ssh/id_rsa --add
```
