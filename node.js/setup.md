# node.js 环境安装配置

## 云课程环境安装

```bash
#更新apt源并安装npm,nodejs
apt update && apt-get install npm nodejs -y

#验证
node -v
```

当有提示时，即证明您已经在您当前的云环境中安装成功，您就可以开始进行本课程下一步的学习了。

注：当云环境的生命周期失效后，需要重新进行安装。

## 在您本地环境安装

### Linux/Mac系统源码安装

以下部分我们将介绍在Ubuntu Linux下安装 Node.js 。 其他的Linux系统，如Centos等类似如下安装步骤。

在 Github 上获取 Node.js 源码：

```bash
git clone https://github.com/joyent/node.git
```

在完成下载后，将源码包名改为 'node'。

```bash
mv joyent node
```

修改目录权限：

```bash
chmod 755 -R node
```

```bash
#创建编译文件。
./node/configure
```

```bash
make
```

```bash
make install
```

最后我们输入'node --version' 命令来查看Node.js是否安装成功。

```bash
node --version
```

### Windows

不推荐在Windows上使用





