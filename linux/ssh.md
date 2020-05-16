# Linux 远程登录

Linux一般作为服务器使用，这时我们就需要远程登录到Linux服务器来管理维护系统。

Linux系统中是通过ssh服务实现的远程登录功能，默认ssh服务端口号为 22。

## 从Windows远程登陆

Window系统上 Linux 远程登录客户端有SecureCRT, Putty, SSH Secure Shell等。putty下载地址：http://www.putty.org/

## 从Linux终端利用登录远程服务器

安装ssh并启动ssh服务：

```bash
apt install openssh-server && service sshd start
```

终端下使用ssh登录远程服务器：

```bash
ssh -p 30000 username@127.0.0.1
```

**-p** 后面是端口,默认不指定的话是22

**username** 是服务器用户名,**127.0.0.1** 是远程服务器 ip

回车输入密码即可登录

## 实验

在右侧实验区，尝试从本地登陆本地

```shell
ssh root@localhost
```
