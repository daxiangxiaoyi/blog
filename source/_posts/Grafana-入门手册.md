---
title: Grafana 入门手册
date: 2019-01-09 11:33:10
categories:
tags:
---


## Grafana 是什么


Grafana是一个开源的，拥有丰富dashboard和图表编辑的指标分析平台，和Kibana不同的是Grafana专注于时序类图表分析，而且支持多种数据源，如Graphite、InfluxDB、Elasticsearch、Mysql、K8s、Zabbix等。
## Grafana能干什么

Grafana最早其实应该是Kibana3的一个分支，拥有自己的权限管理和用户管理系统，而Kibana没有权限管理。Kibana和ES结合紧密，支持强大的ES语法，比较适合做一些多维度的分析和查询，而Grafana更适合用于展示，图形比Kibana美观很多。


## Grafana的安装


Grafana官方提供Ubuntu/Debian、Centos/Redhat、Windows、Mac平台的安装包，可以从源码构建，并且可以使用Docker构建容器。这里主要介绍一下在Centos下的安装和Docker安装，其他安装方式请参考[官方网站](http://docs.grafana.org/installation/)。

### Centos下安装

* **直接使用Yum**

```
sudo yum install <rpm package url>
```

  例如

```
sudo yum install https://dl.grafana.com/oss/release/grafana-5.4.2-1.x86_64.rpm
```
* **通过YUM 仓库安装**

在文件中/etc/yum.repos.d/grafana.repo 写入
```
[grafana]
name=grafana
baseurl=https://packages.grafana.com/oss/rpm
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
```
然后
```
sudo yum install grafana
```
启动Grafana

> systemctl start grafana-server

### Docker安装

通过官方镜像安装运行Grafana非常简单，一条命令就搞定
>docker run -d -p 3000:3000 grafana/grafana

配置文件的修改，所有配置文件conf/grafana.ini中的选项都可以用环境变量覆盖，格式为GF_<SectionName>_<KeyName>，例如
```
docker run \
  -d \
  -p 3000:3000 \
  --name=grafana \
  -e "GF_SERVER_ROOT_URL=http://grafana.server.name" \
  -e "GF_SECURITY_ADMIN_PASSWORD=secret" \
  grafana/grafana
```
**插件的安装**
除了进入容器内以grafana-cli plugins install <plugin name>外，还可以把要安装的插件传给GF_INSTALL_PLUGINS，多个插件直接用逗号隔开。例如
```
docker run \
  -d \
  -p 3000:3000 \
  --name=grafana \
  -e "GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-simple-json-datasource" \
  grafana/grafana
```
当然了我们也可以构建自己的镜像：

在[github]( https://github.com/grafana/grafana/tree/master/packaging/docker) 有一个custom文件夹，包含了Dockerfile
```
cd custom
docker build -t grafana:latest-with-plugins \
  --build-arg "GRAFANA_VERSION=latest" \
  --build-arg "GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-simple-json-datasource" .


docker run \
  -d \
  -p 3000:3000 \
  --name=grafana \
  grafana:latest-with-plugins
```
**这种方式删除容器后数据会丢失，官方建议持久化**

```
# create a persistent volume for your data in /var/lib/grafana (database and plugins)
docker volume create grafana-storage

# start grafana

docker run \
  -d \
  -p 3000:3000 \
  --name=grafana \
  -v grafana-storage:/var/lib/grafana \
  grafana/grafana

```
## Grafana的使用

启动Grafana后，在浏览器输入http:://localhost:3000/ 就进入Grafana的登陆页面，默认用户名和密码都是admin。

### 添加数据源

Grafana使用的第一步是添加数据源，点击左侧菜单栏的齿轮形图标，Configuration->Data Sources->Add data source，然后选择对应的数据源填写相关的连接信息即可。

### 制作dashboard

Dashboard由Row组成，而Row由基本组件Panel组成。添加Row也是在add panel处添加，每一个panel的数据源都可以是任意一个我们添加的data source，这里有很多快捷键，比如常用的Ctrl+S 保存，Ctrl+F 搜索，Ctrl+H 隐藏所有的控制面板。

### 基本概念

>**Data Source：**数据源一般多为时序数据。

>**Organization：**支持多组织部署，组织可以理解为不同的部门，不同的需求方等。

>**User:**用户名就是Grafana里的账户，一个用户可以属于一个或者多个组织，而且可以根据角色分配不同的权限。

>**Row:**Row是一组Panels，是Dashboard的逻辑划分。一般来说，Rows都是12个单位的宽度，可以自适应大小屏幕。

>**Panel:**Grafana里的基本图像模块，可以拖拽来调节大小和位置，有Graph，Singlestat，Dashlist，Table和Text几种类型。

>**Query Editor:**可以编辑数据源的不同指标。

>**Dashboard:**这个就很好理解了，通常意义上的仪表盘。可以使用Templating和Annotations。


Grafana还有很多高级的玩法，比如报警，模板，API等，入门篇里就不展开了。快来开始你的第一个炫酷的dashboard吧！
