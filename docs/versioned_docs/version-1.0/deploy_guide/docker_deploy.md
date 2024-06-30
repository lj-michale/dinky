---
sidebar_position: 2
position: 2
id: docker_deploy
title: Docker 部署
---

:::danger 注意

Dinky v1.0.0 的 Docker 镜像正在开发中，敬请期待。以下步骤目前由于镜像未发布，无法使用。请耐心等待。如您对 Docker 镜像有兴趣，欢迎加入我们的开发群，一起参与开发。

:::


## Docker 快速使用教程
本教程使用三种不同的方式通过 Docker 完成 Dinky 的部署，如果你想要快速体验，推荐使用 standalone-server 镜像， 如果你想要体验比较完整的服务，推荐使用 docker-compose 启动服务。如果你已经有自己的数据库服务 你想要沿用这些基础服务，你可以参考沿用已有的 Mysql 、 Hadoop 和 Flink  服务完成部署。

## 前置条件
- Docker 1.13.1+
- Docker Compose 1.28.0+（可选）

##  Docker启动
使用 H2 本地数据库快速启动
```shell
docker run --restart=always -p 8888:8888 \
  --name dinky \
  dinky/dinky:1.0.3-flink1.17

```
使用 外部mysql数据库
```bash
docker run --restart=always -p 8888:8888 \
  --name dinky \
  -e DB_ACTIVE=mysql \
  -e MYSQL_ADDR=127.0.0.1:3306 \ 
  -e MYSQL_DATABASE=dinky \ 
  -e MYSQL_USERNAME=dinky \
  -e MYSQL_PASSWORD=dinky \
  -v /opt/lib:/opt/flink/lib/customJar/ \ 
  dinky/dinky:1.0.3-flink1.17

```
:::tip 说明
由于mysql与Apache 2.0协议不兼容，dinky无法默认提供mysql驱动，所以需要您手动提供mysql依赖并放到`/opt/flink/lib/customJar/`
下面，上面已经给出了映射，如果你有自己的依赖目录，修改即可
:::

使用 外部postgres数据库
```bash
docker run --restart=always -p 8888:8888 \
  --name dinky \
  -e DB_ACTIVE=pgsql \
  -e POSTGRES_ADDR=127.0.0.1:5432 \ 
  -e POSTGRES_DATABASE=dinky \ 
  -e POSTGRES_USERNAME=dinky \
  -e POSTGRES_PASSWORD=dinky \
  -v /opt/lib:/opt/flink/lib/customJar/ \ 
  dinky/dinky:1.0.3-flink1.17
```
---
### 使用docker-compose 
docker-compose可快速帮你搭建起来dinky与flink集群环境，
下载dinky源码后，在 `deploy/docker` 下面即可找到`docker-compose.yml`和 `.env` 文件，
```bash
cd deploy/docker/
ls -al
-rwxrwxrwx 1 root root  765 May 29 11:29 docker-compose.dev.yml
-rwxrwxrwx 1 root root  699 May 29 11:29 docker-compose.yml
-rwxrwxrwx 1 root root 1603 May 29 11:29 Dockerfile
-rwxrwxrwx 1 root root 1718 May 29 11:46 .env

```
编辑 `.env` 文件，修改你想要的配置，如果只是快速体验，无需修改任何内容
```shell
#定义dinky版本号
DINKY_VERSION=1.0.3
#定义Flink版本（不要写小版本号）
FLINK_VERSION=1.17

# 自定义jar包依赖本地路径（例如：mysql驱动）
CUSTOM_JAR_PATH=/opt/dinky/extends/

# 使用那种数据库，默认h2
DB_ACTIVE=h2
# h2数据库持久化文件路径
H2_DB=./tmp/db/h2

## 使用mysql数据库时打开注释并填写内容
## 如果 DB_ACTIVE 配置为mysaql，请修改下面配置，否则忽略
##MYSQL_ADDR=127.0.0.1:3306
#MYSQL_DATABASE=dinky
#MYSQL_USERNAME=dinky
#MYSQL_PASSWORD=dinky

## 使用pg数据库时打开注释并填写内容
## 如果 DB_ACTIVE 配置为pgsql，请修改下面配置，否则忽略
##POSTGRES_USER=dinky
#POSTGRES_PASSWORD=dinky
#POSTGRES_ADDR=localhost:5432
#POSTGRES_DB=dinky

# 时区
TZ=Asia/Shanghai
# 自带Flink集群配置，一般不用改
FLINK_PROPERTIES="jobmanager.rpc.address: jobmanager"

```
启动集群
```shell
docker-compose up -d
```


## 本地开发

可使用
```shell
docker compose --profile standalone -f docker-compose.yml -f docker-compose.dev.yml up
```
进行docker镜像的构建及运行(/docker/.env文件配置相关环境变量).

如果需要前后端分离, 可使用
```shell
docker compose --profile ms -f docker-compose.yml -f docker-compose.dev.yml up
```
(适配自身nginx的docker/web/default.conf配置)
对于1.15上版本,需要手动将容器中/opt/diny/plugin/flink{version}/flink-table-planner-loader*.jar移除,
替换为相应版本的flink-table-planner_*.jar文件.
创建容器时,可映射到容器/opt/diny/customJar文件夹,添加自定义jar包.

欢迎补充