# PHP 环境安装及配置
## 本课程在线环境的安装

### 安装PHP解析器,Apache2服务器

```bash
#更新源并安装php
apt update && apt-get install php apache2 -y


#启动服务器
service apache2 start
```
```
#apache2服务相关说明
查看状态： service apache2 status/start/stop/restart
Web目录： /var/www/html/
安装目录： /etc/apache2/
全局配置： /etc/apache2/apache2.conf
监听端口： /etc/apache2/ports.conf
虚拟主机： /etc/apache2/sites-enabled/000-default.conf
```

### 解析测试

***PHP解析***

```bash
#制作php探针文件
cat > /var/www/html/env.php <<EOF
<?php echo phpinfo();?>
EOF

#使用php解释器解析该探针文件
php /var/www/html/env.php
```

***apache2服务器解析测试***

```bash
#制作php探针
cat > /var/www/html/env.php <<EOF
<?php echo phpinfo();?>
EOF
```

点击URL查看由apache2解析的php探针文件：{http-server}/env.php

### 复制课程案例至目录

```
cp -r /share/lesson/php/* /var/www/html/
```

## 在您本地安装

### Windows

建议使用安装Virtul Box,进行虚拟机的安装。

