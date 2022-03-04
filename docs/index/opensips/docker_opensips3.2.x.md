# 基于Docker 部署OpenSips 2.4.x

### 参考
``` shell
docker pull opensips/opensips
```

### 1、编辑Dockerfile文件
```dockerfile
FROM debian:11.1
RUN apt-get update -y
# 编译安装opensips，所需要依赖环境
RUN apt-get install -y sngrep vim wget gcc bison flex make openssl unixodbc libxml2 libncurses5-dev gnupg2 less libsctp* libmysqlclient* zlib* libpq* libmicrohttpd* libjson-c-dev libmongoc-dev default-libmysqlclient-dev mariadb-server libhiredis* libxml2* libpcre*
# 安装opensips-cli工具
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 049AD65B
RUN echo "deb https://apt.opensips.org bullseye cli-nightly" >/etc/apt/sources.list.d/opensips-cli.list
RUN apt-get update -y && apt-get install -y opensips-cli
# 把opensips 3.2.3源码文件夹添加到镜像里，注意：opensips-3.2.3文件夹需要跟Dockerfile文件是同一层目录
RUN mkdir -p /tools/software/opensips-3.2.3
ADD ./opensips-3.2.3 /tools/software/opensips-3.2.3
# include_modules 选择需要的模块进行编译安装
RUN cd /tools/software/opensips-3.2.3 && make all include_modules="db_mysql db_postgres proto_tls proto_wss tls_mgm cachedb_mongodb cachedb_redis httpd json xcap presence presence_xml dialplan" prefix="/usr/local/opensips" install
# 添加 opensips.cfg 配置文件
RUN mkdir -p /usr/local/etc/opensips/
RUN cp /usr/local/opensips/etc/opensips/opensips.cfg /usr/local/etc/opensips/opensips.cfg

VOLUME ["/usr/local/opensips/etc"]
VOLUME ["/usr/local/etc/opensips/"]
EXPOSE 5060/udp

ENTRYPOINT ["/usr/local/opensips/sbin/opensips", "-FE"]
```
### 2、利用opensips-cli初始化数据库结构
* 基于第一步docker镜像新建出容器
```shell
docker run --name opensips -it -v /data/opensips:/usr/local/opensips/etc/opensips -d opensips-3.2.3
```
* 进入容器
```shell
docker exec -it opensips bash
```
* 新增 opensips-cli.cfg 配置
```shell
vim ~/.opensips-cli.cfg

# 默认配置：https://github.com/OpenSIPS/opensips-cli/blob/master/etc/default.cfg
# 数据库参考：https://github.com/OpenSIPS/opensips-cli/blob/master/docs/modules/database.md
# mi配置：https://github.com/OpenSIPS/opensips-cli/blob/master/docs/modules/mi.md
```
opensips-cli.cfg 配置如下
```cfg
[default]
log_level: WARNING
prompt_name: opensips-cli
prompt_intro: Welcome to OpenSIPS Command Line Interface!
prompt_emptyline_repeat_cmd: False
history_file: ~/.opensips-cli.history
history_file_size: 1000
output_type: pretty-print
communication_type: fifo
fifo_file: /tmp/opensips_fifo

# 选择模块添加数据库表结构
database_modules: ALL

# 数据库脚本目录
database_schema_path: /usr/local/opensips/share/opensips

# 数据库管理员账号
#database_admin_url: postgres://root@localhost
database_admin_url: mysql://root@192.168.15.130

# 会新建数据库账号：opensips，密码：opensipsrw
# database_url: postgres://opensips:opensipsrw@192.168.15.130
database_url: mysql://opensips:opensipsrw@192.168.15.130
# 数据库名称
database_name: opensips
```
* 执行命令生成数据库
```shell
# 输入数据库管理员账号的密码
opensips-cli -x database create
```
