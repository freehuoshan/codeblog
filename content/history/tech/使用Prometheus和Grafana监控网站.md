---
title: 使用Prometheus和Grafana监控网站
id: 27
date: 2023-04-10 18:19:22
auther: FREEDOM
cover: https://codeblog.net/upload/2021/04/image-cbd4c319130f4aeab7c20d5750b416e2.png
excerpt: 安装Wordpress安装Apachesudo apt updatesudo apt install apache2查看 Apache运行状态sudo systemctl status apache2输出如下内容说明Apache正在运行进入用户家目录，首先下载wordpress安装包wget h
permalink: /archives/apachewordpressprometheusgrafana
categories:
 - code
tags: 
 - wordpress
 - prometheus
 - grafana
---


该指南会首先在 UbuntU20.04上安装Apache2, 然后部署一个 Wordpress 站点到 Apache2服务器上，最后通过 Prometheus 监控该Wordpress网站的状态并使用 Grafana 面板展示。


## 安装Wordpress

### 安装Apache

```
sudo apt update
sudo apt install apache2
```
查看 Apache运行状态:
```shell
sudo systemctl status apache2
```
输出如下内容说明Apache正在运行:

![image.png](https://codeblog.net/upload/2021/03/image-431d1d44632f4302947be8e97ef04ce5.png)

在浏览器中访问 localhost 显示如下说明 Apache工作正常:

![image.png](https://codeblog.net/upload/2021/03/image-c3c4e385cf3e44bd8c6fd5c0db9949e9.png)

### 安装Mysql

```shell
sudo apt install mysql-server
```
查看Mysql运行状态,如下所示，说明Mysql已经安装成功:

![image.png](https://codeblog.net/upload/2021/03/image-762c364f94254524bc40b45829197912.png)

#### 配置Mysql:

```shell
sudo mysql_secure_installation
```
![image.png](https://codeblog.net/upload/2021/04/image-ecdfb9f07c394972a3a0e389c69118e9.png)

上边每个步骤的输入：
1. Y
2. 0
3. abc1234
4. abc1234
5. Y
6. Y
7. Y
8. Y
9. Y

刚才我们完成了Mysql的安装和一些基本配置，接下来我们为wordpress配置数据库环境。

#### 为Wordpress创建数据库

在命令行中，使用root用户登录Mysql:
```shell
sudo mysql -u root -p
CREATE DATABASE wpdb DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'WP&Password123';
GRANT ALL ON wpdb.* TO 'wpuser'@'localhost';
GRANT ALL PRIVILEGES ON wpdb.* TO 'wpuser'@'localhost';
```
![image.png](https://codeblog.net/upload/2021/04/image-56105240530b4583a626ec81752290db.png)

1. 输入上面配置Mysql过程中第三步骤设置的 root 密码
2. 创建数据库wpdb ，后边安装wordpress将使用
3. 创建用户wpuser,
4. 密码设置为 WP&Password123
5. 查看wpdb数据库已经创建成功

最后更新变更：

```shell
FLUSH PRIVILEGES;
```

### 安装PHP

```shell
sudo apt install php -y
sudo apt install libapache2-mod-php -y
sudo apt install php-mysql -y
sudo apt install php-curl php-gd php-xml php-mbstring  php-xmlrpc php-zip php-soap php-intl -y
```

然后我们需要重启一下 apache2:

```shell
sudo systemctl restart apache2
```

### 部署Wordpress

```shell
curl -O https://wordpress.org/latest.tar.gz
tar -xvf latest.tar.gz
mv wordpress/wp-config-sample.php wordpress/wp-config.php
sudo cp -a wordpress/. /var/www/html
sudo chown -R www-data /var/www/html
sudo sed -i 's#database_name_here#wpdb#g' /var/www/html/wp-config.php
sudo sed -i 's#username_here#wpuser#g' /var/www/html/wp-config.php
sudo sed -i 's#password_here#WP\&Password123#g' /var/www/html/wp-config.php
sudo rm /var/www/html/index.html
```
然后在浏览器输入 localhost :

![image.png](https://codeblog.net/upload/2021/04/image-f3896cfd88474eaf827dbd95d2a855b9.png)

选择界面语言，然后点击继续

![image.png](https://codeblog.net/upload/2021/04/image-79379da9c2604d2b8a8de2e6211860ba.png)

设置博客标题，创建一个用户，需要输入用户名和密码以及邮箱地址。然后点击安装 WordPress

![image.png](https://codeblog.net/upload/2021/04/image-63106d2e01e24b0c96d23b30e32faf80.png)

![image.png](https://codeblog.net/upload/2021/04/image-904bfa43ea914e6badec5c16ef151c11.png)

安装wordpress成功，点击登录。这是Wordpress的控制面板：

![image.png](https://codeblog.net/upload/2021/04/image-81c7ad3042994daf8561976cb632dc85.png)

Wordpress网站主页 localhost:

![image.png](https://codeblog.net/upload/2021/04/image-1a253bff8261479594861136d9c7affc.png)

## 安装Prometheus

#### 1. 创建Prometheus系统组

```shell
sudo groupadd --system prometheus
sudo useradd -s /sbin/nologin --system -g prometheus prometheus
```

#### 2. 创建Prometheus的数据和配置目录
```shell
sudo mkdir /var/lib/prometheus
for i in rules rules.d files_sd; do sudo mkdir -p /etc/prometheus/${i}; done
```
#### 3. 下载Prometheus
```shell
curl -s https://api.github.com/repos/prometheus/prometheus/releases/latest | grep browser_download_url | grep linux-amd64 | cut -d '"' -f 4 | wget -qi -
tar xvf prometheus*.tar.gz
cd prometheus*/
sudo mv prometheus promtool /usr/local/bin/
```
#### 4. 移动Prometheus的配置模板到/etc目录

```shell
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
sudo mv consoles/ console_libraries/ /etc/prometheus/
```
#### 5. 创建Prometheus的systemd服务文件

```shell
sudo tee /etc/systemd/system/prometheus.service<<EOF
[Unit]
Description=Prometheus
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=prometheus
Group=prometheus
ExecReload=/bin/kill -HUP \$MAINPID
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.external-url=

SyslogIdentifier=prometheus
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```

#### 修改目录权限

```shell
for i in rules rules.d files_sd; do sudo chown -R prometheus:prometheus /etc/prometheus/${i}; done
for i in rules rules.d files_sd; do sudo chmod -R 775 /etc/prometheus/${i}; done
sudo chown -R prometheus:prometheus /var/lib/prometheus/
```
#### 重新加载systemd daemon 并启动服务:

```shell
sudo systemctl daemon-reload 
sudo systemctl start prometheus 
sudo systemctl enable prometheus
```
在浏览器中访问 localhost: 9090 ：

![image.png](https://codeblog.net/upload/2021/04/image-5181e1123a9e4106a1c63411c6fd8e95.png)

### 安装prometheus-blackbox-exporter

```shell
apt -y install prometheus-blackbox-exporter
systemctl enable prometheus-blackbox-exporter
```

### 配置prometheus监控wordpress站点


```shell
sudo tee /etc/prometheus/prometheus.yml<<EOF
global:
  scrape_interval: 15s
  evaluation_interval: 1m

rule_files:
  - /etc/prometheus/rules/*.yml

scrape_configs:
  - job_name: 'website-monitoring-http'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
        # hostname or IP address of target Host
        - localhost:80
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: localhost:9115

  - job_name: "website-monitoring-icmp"
    metrics_path: /probe
    params:
      module: [icmp]
    static_configs:
      - targets:
        # hostname or IP address of target Host
        - localhost:80 
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: localhost:9115
EOF
```

重新启动prometheus：

```shell
sudo systemctl restart prometheus.service
```

## 安装 Grafana

```shell
sudo apt-get install -y apt-transport-https
sudo apt-get install -y software-properties-common wget
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
sudo apt update
sudo apt-get install grafana
sudo systemctl daemon-reload
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

在浏览器输入 localhost: 3000

![image.png](https://codeblog.net/upload/2021/04/image-994a10ff1e5b4c57ab9db893d00f90b4.png)

username: admin  password: amin

![image.png](https://codeblog.net/upload/2021/04/image-a2b5e5aae08e4ae49e56151bb766899f.png)

会自动要求重置密码：我这里设置的新密码为: abc1234

![image.png](https://codeblog.net/upload/2021/04/image-777068a214c24f4496055336970118da.png)

Grafana 安装成功。

#### 1. 添加数据源

![image.png](https://codeblog.net/upload/2021/04/image-9b547f5ca4a143fb8d57183ad69d960a.png)

![image.png](https://codeblog.net/upload/2021/04/image-96a8db850e694962b86a5ce4da3350d4.png)

![image.png](https://codeblog.net/upload/2021/04/image-3a75126c1c5b4fac85bb3ca416ad702f.png)

![image.png](https://codeblog.net/upload/2021/04/image-c6eb4a37db54447c9f218f608c1d4515.png)

![image.png](https://codeblog.net/upload/2021/04/image-16e25f4f873144488deb746031f6fbab.png)

添加 Prometheus数据源成功。

#### 2. 导入面板:

这里我们使用面板 https://grafana.com/grafana/dashboards/13041 ，该面板的ID 为 13041

![image.png](https://codeblog.net/upload/2021/04/image-a4b42b53ecba4e0ab44da6aa9d46d9cc.png)

![image.png](https://codeblog.net/upload/2021/04/image-0de993d94699450ea8f25c909a1529ba.png)

![image.png](https://codeblog.net/upload/2021/04/image-3fda51676bb54720aad70e10d4424a76.png)

在第一个输入框输入面板 ID，然后加载

![image.png](https://codeblog.net/upload/2021/04/image-c164983a8f994585887ad0a0fc1ab0c7.png)

选择我们刚添加的数据源，然后点击导入。

![image.png](https://codeblog.net/upload/2021/04/image-cbd4c319130f4aeab7c20d5750b416e2.png)

完成，可以看到面板正在监控我们刚刚创建的 Wordpress 站点。