---
layout: post
title:  "Nginx 编译安装"
categories: Linux 高并发
tags: Linux Nginx
---

* content
{:toc}


### Nginx 编译安装

> 本文使用的是Tengine是由淘宝网发起的Web服务器项目。它在[Nginx](http://nginx.org/) 的基础上，针对大访问量网站的需求，添加了很多高级功能和特性

### 安装Tengine

#### 安装之前的准备

##### 1.  安装依赖包

```shell
yum -y install gcc openssl-devel pcre-devel zlib-devel
```



##### 2. **创建用户和用户组。为了方便nginx运行而不影响linux安全**

- 创建组：groupadd -r nginx
- 创建用户：useradd -r -g nginx -M nginx
- **-M 表示不创建用户的家目录**。



##### 3.  编译三步走，进入到 nginx 安装目录下

1.  **执行 configure， 执行前修改安装位置**

```shell
./configure \
  --conf-path=/opt/soft/tengine-2.1.0/nginx.conf \
  --prefix=/opt/soft/tengine-2.1.0/ \
  --error-log-path=/var/log/nginx/error.log \
  --http-log-path=/var/log/nginx/access.log \
  --pid-path=/var/run/nginx/nginx.pid  \
  --lock-path=/var/lock/nginx.lock \
  --with-http_ssl_module \
  --with-http_flv_module \
  --with-http_stub_status_module \
  --with-http_gzip_static_module \
  --http-client-body-temp-path=/var/tmp/nginx/client/ \
  --http-proxy-temp-path=/var/tmp/nginx/proxy/ \
  --http-fastcgi-temp-path=/var/tmp/nginx/fcgi/ \
  --http-uwsgi-temp-path=/var/tmp/nginx/uwsgi \
  --http-scgi-temp-path=/var/tmp/nginx/scgi \
  --with-pcre
```

1. 因为是虚拟机安装，所以在lock-path 后面去掉了用户和用户组的限制

   ```shell
    --user=nginx \
    --group=nginx \
   ```

2. `/var/tmp/nginx/client/` 需要手动创建

3. **进行编译和安装** `make && make install`



#### 把Ngnix命令加入到系统服务中

1. 创建一个nginx文件 `vim /etc/init.d/ngnix`
2. 把下列命令放入文件中

```shell
#!/bin/bash
#
# chkconfig: - 85 15
# description: nginx is a World Wide Web server. It is used to serve
# Source function library.
. /etc/rc.d/init.d/functions
 
# Source networking configuration.
. /etc/sysconfig/network
 
# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0
 
nginx="/opt/soft/tengine-2.2.1/sbin/nginx"  #这里根据安装目录配置
prog=$(basename $nginx)
 
NGINX_CONF_FILE="/opt/soft/tengine-2.2.1/conf/nginx.conf" #这里根据安装目录配置
 
#[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx
 
lockfile=/var/lock/subsys/nginx
 
#make_dirs() {
#   # make required directories
#   user=`nginx -V 2>&1 | grep "configure arguments:" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
#   options=`$nginx -V 2>&1 | grep 'configure arguments:'`
#   for opt in $options; do
#       if [ `echo $opt | grep '.*-temp-path'` ]; then
#           value=`echo $opt | cut -d "=" -f 2`
#           if [ ! -d "$value" ]; then
#               # echo "creating" $value
#               mkdir -p $value && chown -R $user $value
#           fi
#       fi
#   done
#}
 
start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
#    make_dirs
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}
 
stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}
 
restart() {
    configtest || return $?
    stop
    sleep 1
    start
}
 
reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
#  -HUP是nginx平滑重启参数  
    killproc $nginx -HUP
    RETVAL=$?
    echo
}
 
force_reload() {
    restart
}
 
configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}
 
rh_status() {
    status $prog
}
 
rh_status_q() {
    rh_status >/dev/null 2>&1
}
 
case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
```

1. 修改nginx执行权限 `chmod +x nginx`

2. 添加改文件到系统服务中去 `chkconfig --add nginx`

3. 查看是否添加成功`chkconfig --list nginx`

4. ngnix 启动、停止，重新装载 `service nginx start|stop|reload`






