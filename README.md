# Nacos

## 1. Nacos集群搭建

## 1.1. Nacos官方资料


[https://nacos.io/zh-cn/](https://nacos.io/zh-cn/)

### 1.2. 单机部署

* [Nacos版本下载](https://github.com/alibaba/nacos/releases/)
* 初始化mysql数据库,数据库初始化文件：conf/nacos-mysql.sql
* 修改conf/application.properties文件

  ```text
    spring.datasource.platform=mysql
    db.num=1
    db.url.0=jdbc:mysql://${mysql_ip}:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
    db.user=nacos_devtest
    db.password=youdontknow
  ```

* 运行bin/startup.sh

### 1.3. docker集群部署

* [Nacos版本下载](https://github.com/alibaba/nacos/releases/)
* 建立映射目录,集群至少三个所以建立3个目录,nacos\_cluster是我用来存放共用配置的地方

  ```text
  ├── docker-compose.yml
  ├── nacos.env
  ├── nacos1
  │   ├── bin
  │   │   └── startup.sh
  │   ├── conf
  │   │   ├── application.properties
  │   │   └── cluster.conf
  │   ├── data
  │   ├── logs
  │   └── target
  │       └── nacos-server.jar
  ├── nacos2
  │   ├── bin
  │   │   └── startup.sh
  │   ├── conf
  │   │   ├── application.properties
  │   │   └── cluster.conf
  │   ├── data
  │   ├── logs
  │   └── target
  │       └── nacos-server.jar
  ├── nacos3
  │   ├── bin
  │   │   └── startup.sh
  │   ├── conf
  │   │   ├── application.properties
  │   │   └── cluster.conf
  │   ├── data
  │   ├── logs
  │   └── target
  │       └── nacos-server.jar
  └── nacos_cluster
    ├── bin
    │   ├── shutdown.sh
    │   └── startup.sh
    ├── conf
    │   ├── application.properties
    │   ├── cluster.conf
    │   └── nacos-logback.xml
    ├── plugins
    │   └── cmdb
    │       └── nacos-cmdb-plugin-example.jar
    └── target
        └── nacos-server.jar
  ```

* 编写docker-compose.yml文件
```yml
version: "3"
services:
  nacos1:
    hostname: nacos1
    container_name: nacos1
    image: anapsix/alpine-java:8u201b09_jdk_unlimited
    volumes:
      # - '/etc/localtime:/etc/localtime:ro'
      - ./nacos1:/home/nacos
      - ./nacos_cluster/conf:/home/nacos/conf
      - ./nacos_cluster/bin:/home/nacos/bin
      - ./nacos_cluster/target:/home/nacos/target
    ports:
      - "8848:8848"
    env_file:
      - ./nacos-hostname.env
    restart: on-failure
    command: /home/nacos/bin/startup.sh

  nacos2:
    hostname: nacos2
    container_name: nacos2
    image: anapsix/alpine-java:8u201b09_jdk_unlimited
    volumes:
      # - '/etc/localtime:/etc/localtime:ro'
      - ./nacos2:/home/nacos
      - ./nacos_cluster/conf:/home/nacos/conf
      - ./nacos_cluster/bin:/home/nacos/bin
      - ./nacos_cluster/target:/home/nacos/target
    ports:
      - "8849:8848"
    env_file:
      - ./nacos-hostname.env
    restart: on-failure
    command: /home/nacos/bin/startup.sh

  nacos3:
    hostname: nacos3
    container_name: nacos3
    image: anapsix/alpine-java:8u201b09_jdk_unlimited
    volumes:
      # - '/etc/localtime:/etc/localtime:ro'
      - ./nacos3:/home/nacos
      - ./nacos_cluster/conf:/home/nacos/conf
      - ./nacos_cluster/bin:/home/nacos/bin
      - ./nacos_cluster/target:/home/nacos/target
    ports:
      - "8850:8848"
    env_file:
      - ./nacos-hostname.env
    restart: on-failure
    command: /home/nacos/bin/startup.sh

networks:
  default:
    external:
      name: prod_db_nets
```


* nacos.env 环境配置
```properties
JAVA_PACKAGE=jdk
JAVA_JCE=unlimited
JAVA_HOME=/opt/jdk
GLIBC_REPO=https://github.com/sgerrand/alpine-pkg-glibc
LANG=C.UTF-8
PREFER_HOST_MODE=hostname
JAVA_OPT= -Duser.timezone=GMT+08
```

* cluster.conf

  ```text
  nacos1:8848
  nacos2:8848
  nacos3:8848
  ```

* 初始化mysql数据库,数据库初始化文件：conf/nacos-mysql.sql
* 执行docker-compose up -d
* 检查日志输出\(详细请查看nacos-logback.xml\)
  * naming-raft.log 记录了raft信息
  * nacos.log 应用日志

### 遇到问题

#### 报错unable to find local peer: 172.20.0.3:8848, all peers: \[nacos1:8848, nacos2:8848, nacos3:8848\]

这是因为没有使用PREFER\_HOST\_MODE=hostname,需要修改startup.sh文件

[startup.sh请参考](https://github.com/meteorice/gitbook-nacos/tree/afa7d74a8d5d403a2b6b650fc5dd89137e46e5fe/fixed/startup.sh)

```java
java.lang.IllegalStateException: unable to find local peer: 172.20.0.3:8848, all peers: [nacos1:8848, nacos2:8848, nacos3:8848]
        at com.alibaba.nacos.naming.raft.PeerSet.local(PeerSet.java:191)
        at com.alibaba.nacos.naming.monitor.PerformanceLoggerThread.collectmetrics(PerformanceLoggerThread.java:114)
        at sun.reflect.GeneratedMethodAccessor74.invoke(Unknown Source)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:497)
```

### 参考

* [Nacos官网](https://nacos.io/zh-cn/index.html)
* docker官方镜像,一条龙服务 Nacos + Grafana + Prometheus
  * [Nacos docker-hub](https://hub.docker.com/r/nacos/nacos-server)
  * [nacos-docker](https://www.github.com/nacos-group/nacos-docker) 

