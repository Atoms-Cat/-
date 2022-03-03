# 基于Docker 部署OpenSips 2.4.x

### 参考
``` shell
docker pull opensips/opensips
```

### 1、编辑Dockerfile文件
```dockerfile
FROM debian:10.10
RUN apt-get update && apt-get install -y gnupg2 wget lsb-release
RUN wget -O - https://files.freeswitch.org/repo/deb/debian-release/fsstretch-archive-keyring.asc | apt-key add -
RUN echo "deb http://files.freeswitch.org/repo/deb/debian-release/ `lsb_release -sc` main" > /etc/apt/sources.list.d/freeswitch.list
RUN echo "deb-src http://files.freeswitch.org/repo/deb/debian-release/ `lsb_release -sc` main" >> /etc/apt/sources.list.d/freeswitch.list
RUN apt-get update -y
RUN apt-get install vim sngrep less flex libmicrohttpd* libjson-c-dev libmongoc-dev -y
# Install dependencies required for the build
RUN apt-get build-dep freeswitch -y
RUN apt-get install default-libmysqlclient-dev mariadb-server -y
RUN mkdir -p /tools/software/opensips-2.4.11
ADD ./opensips-2.4.11 /tools/software/opensips-2.4.11
RUN cd /tools/software/opensips-2.4.11 && make all include_modules="db_mysql proto_tls proto_wss tls_mgm cachedb_mongodb cachedb_redis httpd json xcap presence presence_xml dialplan"
RUN cd /tools/software/opensips-2.4.11 && make include_modules="db_mysql proto_tls proto_wss tls_mgm cachedb_mongodb cachedb_redis httpd json xcap presence presence_xml dialplan" prefix="/usr/local/opensips" install
RUN mkdir -p /usr/local/etc/opensips/
RUN cp /usr/local/opensips/etc/opensips/opensips.cfg /usr/local/etc/opensips/opensips.cfg
VOLUME ["/usr/local/opensips/etc"]
VOLUME ["/usr/local/etc/opensips/"]
COPY docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
```

