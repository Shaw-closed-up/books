# Docker 跑通第一个Docker容器

Docker 允许你在容器内运行应用程序， 使用 **docker run** 命令来在容器内运行一个应用程序。

### 使用容器输出Hello world





```bash
docker run --rm --name c0 ubuntu:20.04 echo "Hello world"
```

各个参数解析：

- **docker:** Docker 的二进制执行文件。
- **run:** 与前面的 docker 组合来运行一个容器。
- **ubuntu:20.04** 指定要运行的镜像，Docker 首先从本地主机上查找镜像是否存在，如果不存在，Docker 就会从镜像仓库 Docker Hub 下载公共镜像。其中`ubuntu`是镜像名,`20.04`是该镜像的版本号。镜像名与版本号之间使用`:`进行分割。如果省略了版本号只写镜像名，则镜像版本号自动填充为`latest`，意为最新版。但强烈建议，务必指定镜像的版本号，防止出现版本冲突，环境依赖等问题。
- echo "Hello world":** 在启动的容器里执行的命令

以上命令完整的意思可以解释为：Docker 以 ubuntu20.04 镜像创建一个新容器，然后在容器里执行 bin/echo "Hello world"，然后输出结果。

## 运行交互式的容器

我们通过 docker 的两个参数 -i -t，让 docker 运行的容器实现**"对话"**的能力：

```bash
docker run -i -t --name c1 --hostname c1 ubuntu:20.04 bash
```

各个参数解析：

- `-i`:允许你对容器内的标准输入 (STDIN) 进行交互
- `-t`: 在新容器内指定一个伪终端`tty`
- `--name`:是指新建容器的名称
- `--hostname`:是指新建容器的主机名

注意第二行 `root@c1:/#`，此时我们已进入一个 ubuntu 20.04系统的容器

我们尝试在容器中运行命令 cat /etc/issue查看当前系统的版本信息

```bash
cat /etc/issue
```

我们可以通过运行 exit 命令或者使用 CTRL+D 来退出容器。

```bash
exit
```

注意看命令行提示符变成 `root@{host0.hostname}:/#`，这表明我们已经退出了c1这个容器，返回到当前系统环境中。

## 停止容器

我们使用 **docker stop** 命令来停止容器:

通过 **docker ps** 查看，容器已经停止工作:

```bash
docker ps
```

可以看到容器已经不在了。

也可以用下面的命令来停止:

```bash
docker stop c2
docker ps -a #检查所有窗口，无论是否在运行
```

## 删除已经停止的容器

```bash
docker rm c2
docker ps -a #检查所有窗口，无论是否在运行
```
