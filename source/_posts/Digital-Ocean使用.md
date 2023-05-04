
---
title: Hello World
date: 2022-01-11 21:18:02
url: 
tags:
- Digital-Ocean
password: sfczj2019
abstract: 有东西被加密了, 请输入密码查看.
message: 您好, 这里需要密码.
wrong_pass_message: 抱歉, 这个密码看着不太对, 请再试试.
wrong_hash_message: 抱歉, 这个文章不能被校验, 不过您还是能看看解密后的内容.
---
# DO使用

## 购买服务器

因为centos8停止更新的缘故，选了debian11，服务器选旧金山（圣弗朗西斯科）区域只能选3不知道为什么，密码时直接选择ssh，windows可以用puttygen生成公钥[https://docs.digitalocean.com/products/droplets/how-to/connect-with-ssh/putty/]
设置后就可以远程登陆了注意登录用户名是root

## 设置debian11服务器

美化命令行
进入vim

```bash
vim ~/.bashrc
```

按下shift+g跳到最后一行，按下o在最后新建一行进行编辑，输入

```bash
PS1="\[\e[01;32m\]\h \[\e[01;35m\]\A \[\e[01;34m\](\w) \[\e[01;36m\]\\$ \[\e[0m\]"
```

按下Esc退出编辑模式，输入:wq保存并退出，然后重新加载更改后的配置文件

```bash
source ~/.bashrc
```

禁止ping

## 打开bbr

bbr需要系统内核4.9.0以上
查看内核版本

```bash
uname -r
```

打开BBR

```bash
echo 'net.core.default_qdisc=fq' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_congestion_control=bbr' | sudo tee -a /etc/sysctl.conf
```

```bash
sysctl -p
```

查看内核模块是否已经加载

```bash
lsmod | grep bbr
```

## 更改时区

国外服务器默认使用UTC时间
利用timedatectl程序可以更改
将硬件时钟调整为与本地时钟一致, 0 为设置为 UTC 时间

```bash
timedatectl set-local-rtc 1
```

设置系统时区为上海

```bash
timedatectl set-timezone Asia/Shanghai
```

## ssh登录服务器

在购买时已经在服务器上保存了公钥
自己手动配置步骤如下
生成之后需要把公钥上传到服务器

1. 保存为服务器端的 ~/.ssh/authorized_keys 文件(公钥 id_rsa 需重命名为 authorized_keys和 /etc/ssh/sshd_config 中的配置保持一致)
2. 确保只有当前用户可读写同时更改authorized_keys文件和.ssh文件夹的权限，分别是600和700权限

    ```bash
    chown fengjiansu:fengjiansu ~/.ssh/
    chmod 700 /root/.ssh
    chmod 600 /root/.ssh/authorized_keys
    ```

3. 修改ssh的配置文件，以便ssh使用密钥

   ```bash
    # 打开ssh配置文件

    vim /etc/ssh/sshd_config

    # 将以下这项去掉注释并改为yes，以启用密钥验证

    PubkeyAuthentication yes

    # 指定公钥数据库文件

    AuthorsizedKeysFile .ssh/authorized_keys

    # 为了减少被扫描端口的次数，把端口从22改为其他端口，如2333

    Port 2333

    # 需要使用root用户登录，要确保这一行为yes

    PermitRootLogin yes

    # StrictModes修改为no,默认为yes

    # 如果不修改用key登陆是出现server refused our key

    # 如果StrictModes为yes必需保证存放公钥的文件夹的拥有与登陆用户名是相同的

    # “StrictModes”设置ssh在接收登录请求之前是否检查用户家目录和rhosts文件的权限和所有权

    # 很有有必要这样做，因为新手经常会把自己的目录和文件设成任何人都有写权限

    StrictModes no

   ```

4. 启一下ssh以便修改的配置生效

   ```bash
    #ubuntu系统
    service ssh restart

    #debian系统
    /etc/init.d/ssh restart

    # CentOS 使用以下命令
    systemctl restart sshd.service
   ```

## dash切换为bash

```bash
ls -l /bin/sh
```

可以查到当前系统shell使用的版本

执行以下命令，并选择No，可以将默认的shell切换回bash

```bash
sudo dpkg-reconfigure dash
```

## 新建用户

以下以用户名fengjiansu为例进行设置

```bash
# 新建用户fengjiansu，同时添加家目录/home/fengjiansu
useradd -d /home/fengjiansu/ -m fengjiansu(adduser fengjiansu)

# 设置用户密码
passwd fengjiansu

# 切换用户
su fengjiansu

#给用户sudo权限
usermod -aG sudo fengjiansu

# 删除用户和相应目录
deluser --remove-home fengjiansu

```

创建用户有两个命令：useradd 和 adduser ，useradd 只是添加用户，但是这样的用户没有/home家目录，也没有提示设置密码，而 adduser 则家目录和密码都有、
此时打开PUTTY点击Connection的Data改变用户名以普通用户的身份登录要用此用户登录服务器还需要复制公钥文件到 fengjiansu 用户目录下

```bash
# 复制公钥所在文件夹 /root/.ssh 到 fengjiansu 用户目录下
cp -r /root/.ssh /home/fengjiansu

# 因为是用 root 用户复制的文件，所以还需要将 fengjiansu 用户目录下的 .ssh 文件夹和其中的文件所有者设置为 fengjiansu(这个命令需要将目录切换到fengjiansu目录下)
 cd /home/fengjiansu
chown -R fengjiansu:fengjiansu .ssh
```

此时可以尝试用 fengjiansu 用户登录，如果能登录成功，那么就在服务器禁用root用户登录可以通过Xshell的Xftp更改内容更方便

```bash
# 编辑ssh配置文件
sudo vim /etc/ssh/sshd_config

# 将允许root用户登录一行中的yes改为no
PermitRootLogin no
```

改完记得重启sshd

```bash
systemctl restart sshd
```

## 密钥转换

windows用Xshell时所需密钥为openssh格式可借助puttygen工具
打开后点load打开ppk文件
之后点击Conversions点击Export OpenSSH keys得到debian11-id_rsa.pub公钥

## Xshell使用

### 删除显示乱码

文件→属性→终端→键盘，把 delete 和 backspace 序列改为 ASCII 127即可

### 上下左右键乱码

命令提示符并没有显示登录用户名和主机名，没有当前路径名，什么都没有
通过/etc/passwd文件可以看出来，原因是新建的用户fengjiansu使用了不同的shell
这可能不是xshell的问题，而是服务器的设置问题

```bash
root:x:0:0:root:/root:/bin/bash
fengjiansu:x:1000:1000:/home/fengjiansu:/bin/sh
```

修改/etc/passwd文件

```bash
root:x:0:0:root:/root:/bin/bash
fengjiansu:x:1000:1000:/home/fengjiansu:/bin/bash

```
