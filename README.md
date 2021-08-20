# lskypro-dockerCompose-php7.3apache



## 1、思路

用Docker-Compose构建mysql+lskypro，然后用LNMP反向代理到域名。



## 2、拉取镜像

首先感谢这位<a href="https://github.com/Handsomedoggy/lsky-pro">大哥的Dockerfile文件</a>，真的标准，我是用他的构建的。

镜像包：https://hub.docker.com/repository/docker/zyugat/lskypro

```shell
docker pull zyugat/lskypro:1.6.3
docker pull zyugat/lskypro:1.6.3-noVolume
```



构建文件：https://github.com/zyugat/lsky-DockerCompose-php7.3apache



## 3、构建文件

`./mysql/init`：初始化数据库

```sql
CREATE DATABASE IF NOT EXISTS lsky DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
```



`docker-compose`

```yaml
# yml文件
version: '3'
services:
  mysql-main:
    image: mysql:5.7.22
    restart: always
    #名称(可以为空)
    hostname: mysql-main
    #容器名称
    container_name: mysql-main 
    # 修改加密规则
    command: --default-authentication-plugin=mysql_native_password
    ports: 
    - "9306:3306" 
    volumes:
      - mysql-data:/var/lib/mysql
      - mysql-conf:/etc/mysql
      - mysql-log:/var/log/mysql
      - ./mysql/init:/docker-entrypoint-initdb.d
    environment:
      MYSQL_ROOT_PASSWORD: mysqlpsw
    networks:
      - mysql-net
    
  lskypro:
    # build:
    #   # 指定dockerfile文件的所在路径
    #   context: ./lskypro
    #   # 指定Dockerfile文件名称
    #   dockerfile: Dockerfile
    image: zyugat/lskypro:1.6.3
    restart: always
    hostname: lskypro
    container_name: lskypro
    ports: 
      - "9080:80" 
    volumes:
      - lsky-data:/var/www/html
    networks:
      - mysql-net

volumes:
  mysql-data:
  mysql-conf:
  mysql-log:
  lsky-data:

networks:
  mysql-net:
```



## 4、运行

进入目录，我的是：`cd /home/docker`

运行：`docker-compose up -d`

测试：`IP:9080`

如果能打开就说明，你已经成功一半了，如果无法访问，检查防火墙有没有开启9080端口



## 5、配置lnmp反向代理

添加虚拟主机：`lnmp vhost add`

自己添加，这里不做多说明。

添加完后，编辑文件：`vim /usr/local/nginx/conf/vhost/你的域名.conf`，在server里添加：

```
        location / {
            proxy_pass http://127.0.0.1:9080;
            index  index.html index.htm index.jsp;
        }
```



**防跨目录设置**

参考：https://github.com/wisp-x/lsky-pro/issues/31

```
cd /opt/lnmp1.x/tools
./remove_open_basedir_restriction.sh
```



**你可能会遇到的问题：**

使用域名访问的时候报错，无法找到static目录，同时一直在转圈，无法保存配置。

```
GET https://域名/static/app/iconfont/iconfont.css net::ERR_ABORTED 404
GET https://域名/static/app/css/app.css?v=1.5 404
GET https://域名/static/app/css/markdown.css?v=1.0 net::ERR_ABORTED 404
...
```

解决办法：

编辑文件：`vim /usr/local/nginx/conf/vhost/你的域名.conf`

将以下内容全部注释，即可。

```
        # location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
        # {
        #     expires      30d;
        # }

        # location ~ .*\.(js|css)?$
        # {
        #     expires      12h;
        # }

        # location ~ /.well-known {
        #     allow all;
        # }

        # location ~ /\.
        # {
        #     deny all;
        # }
```



## 6、获取连接数据库的地址

**查询方式：`docker network inspect docker_mysql-net`**

**连接数据库地址：`172.19.0.1`**



## 7、参考网址

镜像包：https://hub.docker.com/repository/docker/zyugat/lskypro

构建文件：https://github.com/zyugat/lsky-DockerCompose-php7.3apache

https://github.com/Handsomedoggy/lsky-pro

https://lnmp.org/faq/lnmp-vhost-add-howto.html#user.ini

https://github.com/wisp-x/lsky-pro/issues/31

https://laosu.ml/2021/07/02/%E5%BC%80%E6%BA%90%E7%9A%84%E5%85%B0%E7%A9%BA%E5%9B%BE%E5%BA%8ALskyPro/?highlight=lsky
