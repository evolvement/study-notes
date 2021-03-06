## 下载源
[http://nginx.org/download/](http://nginx.org/download/)

## 安装
```shell
wget http://nginx.org/download/nginx-1.4.4.tar.gz
tar -zxf nginx-1.4.4.tar.gz
cd nginx-1.4.4
./configure --prefix=/usr/local/nginx-1.4.4 --with-http_stub_status_module --with-http_ssl_module --with-http_gzip_static_module
make && make install
```

## 启动服务脚本
```shell
vim /etc/init.d/nginx
```


##### 启动文件内容
```
#! /bin/sh
#chkconfig: 2345 80 90
#description:auto_run
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
 
DESC="nginx daemon"
NAME=nginx
DAEMON=/usr/local/nginx/sbin/$NAME
CONFIGFILE=/usr/local/nginx/conf/$NAME.conf
PIDFILE=/usr/local/nginx/logs/$NAME.pid
SCRIPTNAME=/etc/init.d/$NAME
 
set -e
[ -x "$DAEMON" ] || exit 0
 
do_start() {
    $DAEMON -c $CONFIGFILE || echo -n "nginx already running"
}
 
do_stop() {
    kill -INT `cat $PIDFILE` || echo -n "nginx not running"
}
 
do_reload() {
    kill -HUP `cat $PIDFILE` || echo -n "nginx can't reload"
}
 
case "$1" in
    start)
    echo -n "Starting $DESC: $NAME"
    do_start
    echo "."
    ;;
stop)
    echo -n "Stopping $DESC: $NAME"
    do_stop
    echo "."
    ;;
reload|graceful)
    echo -n "Reloading $DESC configuration..."
    do_reload
    echo "."
    ;;
restart)
    echo -n "Restarting $DESC: $NAME"
    do_stop
    do_start
    echo "."
    ;;
*)
    echo "Usage: $SCRIPTNAME {start|stop|reload|restart}" >&2
    exit 3
    ;;
esac
exit 0
```

##### 开机启动
```shell
chmod +x  /etc/init.d/nginx
chkconfig nginx on
```