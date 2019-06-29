---
title: docker mysql galera clustering
tags:
  - docker
  - container
  - swarm
  - redis
  - sentinel
categories:
  - docker
date: 2019-06-29 17:30:23
---


# mysql galera clustering 

- galera 공식 사이트 : http://galeracluster.com/2015/05/getting-started-galera-with-docker-part-1/
- marriadb 설명 : https://mariadb.com/kb/en/library/getting-started-with-mariadb-galera-cluster/

## docker hub에 있는 image 이용하기

1. image  pull

```sbtshell
sudo docker pull erkules/galera
```

2. container 실행

```sbtshell
sudo docker run -d -p 3306:3306 -p 4567:4567 -p 4444:4444 -p 4568:4568 --name nodea erkules/galera --wsrep-cluster-address=gcomm://
```

## image 만들어서 사용하기

1. 필요한 파일 만들기

- Dockerfile

```sbtshell
FROM ubuntu:16.04
ENV VERSION 20180305
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update

RUN apt-get install -y software-properties-common
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv BC19DDBA
RUN add-apt-repository 'deb http://releases.galeracluster.com/mysql-wsrep-5.7/ubuntu/ xenial main'
RUN add-apt-repository 'deb http://releases.galeracluster.com/galera-3/ubuntu/ xenial main'

COPY galera.pref /etc/apt/preferences.d/galera.pref

RUN apt-get update

RUN apt-get -y install galera-3 galera-arbitrator-3 mysql-wsrep-5.7 rsync lsof

RUN rm -r /var/lib/mysql

COPY my.cnf /etc/mysql/my.cnf
COPY entrypoint.sh /entrypoint.sh
RUN chmod 700 /entrypoint.sh
EXPOSE 3306/tcp
ENTRYPOINT ["/entrypoint.sh"]

LABEL mysql_verion=5.7

```

- docker.cnf

```sbtshell
[mysqld]
bind-address   = 0.0.0.0
user           = mysql
log_bin
[mysql]
prompt = "\h\> "
```

- my.cnf

```sbtshell
[mysqld]
user                              = mysql
bind-address                      = 0.0.0.0
wsrep_provider                    = /usr/lib/galera/libgalera_smm.so
wsrep_sst_method                  = rsync
default_storage_engine            = innodb
binlog_format                     = row
innodb_autoinc_lock_mode          = 2
innodb_flush_log_at_trx_commit    = 0
query_cache_size                  = 0
query_cache_type                  = 0
skip_name_resolve

```

- entrypoint.sh

```sbtshell
#!/bin/bash
# Taken from the official mysql-repo
# And changed for simplification of course :)
# I.e. DATADIR is always /var/lib/mysql
# We don't force the usage of MYSQL_ALLOW_EMPTY_PASSWORD
# erkan.yanar@linsenraum.de
set -e
set -x
# Check ENV (MYSQL_) and stop if they are not known variables
# TODO

tempSqlFile='/tmp/mysql-first-time.sql'

mysqld --initialize --datadir=/var/lib/mysql

cat > "$tempSqlFile" <<-EOSQL
        -- What's done in this file shouldn't be replicated
        --  or products like mysql-fabric won't work
        SET @@SESSION.SQL_LOG_BIN=0;

        DELETE FROM mysql.user ;
        CREATE USER 'root'@'%' IDENTIFIED BY '${MYSQL_ROOT_PASSWORD}' ;
        GRANT ALL ON *.* TO 'root'@'%' WITH GRANT OPTION ;
        DROP DATABASE IF EXISTS test ;
    EOSQL


    if [ "$MYSQL_DATABASE" ]; then
        echo "CREATE DATABASE IF NOT EXISTS \`$MYSQL_DATABASE\` ;" >> "$tempSqlFile"
    fi

    if [ "$MYSQL_USER" -a "$MYSQL_PASSWORD" ]; then
        echo "CREATE USER '$MYSQL_USER'@'%' IDENTIFIED BY '$MYSQL_PASSWORD' ;" >> "$tempSqlFile"
        echo "CREATE USER '$MYSQL_USER'@'localhost' IDENTIFIED BY '$MYSQL_PASSWORD' ;" >> "$tempSqlFile"
  fi
    if [ "$MYSQL_USER" -a ! "$MYSQL_PASSWORD" ]; then
        echo "CREATE USER '$MYSQL_USER'@'%'  ;"         >> "$tempSqlFile"
        echo "CREATE USER '$MYSQL_USER'@'localhost'  ;" >> "$tempSqlFile"
  fi

  if [ "$MYSQL_USER" -a  "$MYSQL_DATABASE"  ]; then
      echo "GRANT ALL ON \`$MYSQL_DATABASE\`.* TO '$MYSQL_USER'@'%' ;" >> "$tempSqlFile"
      echo "GRANT ALL ON \`$MYSQL_DATABASE\`.* TO '$MYSQL_USER'@'localhost' ;" >> "$tempSqlFile"
  fi

    echo 'FLUSH PRIVILEGES ;' >> "$tempSqlFile"

    set -- "$@" --init-file="$tempSqlFile"

echo @a
set -- mysqld "$@"
chown -R mysql:mysql /var/lib/mysql
mkdir -p /var/run/mysqld
chown -R mysql:mysql /var/run/mysqld
echo "Checking to upgrade the schema"
echo "A failed upgrade is ok when there was no upgrade"
# mysql_upgrade || true
exec "$@"
```

- galera.pref

```sbtshell
# Prefer Codership repository
Package: *
Pin: origin releases.galeracluster.com
Pin-Priority: 1001
```

2. 빌드하기

```sbtshell
#sudo docker build -t 이름:버전 docker_file_path
sudo docker build -t galera:1.0 /vagrant/galera
```

3. container 실행하기

```sbtshell
sudo docker run -d -p 3306:3306 -p 4567:4567 -p 4444:4444 -p 4568:4568 galera:1.0 --wsrep-cluster-address=gcomm://
```
