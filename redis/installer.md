## 下载源
[http://www.redis.io/clients](http://www.redis.io/clients)
[https://github.com/nicolasff/phpredis](https://github.com/nicolasff/phpredis)

## 安装
```shell
wget http://download.redis.io/releases/redis-2.8.2.tar.gz
tar -zxf redis-2.8.2.tar.gz
mv redis-2.8.2 /usr/local/redis-2.8.2
cd /usr/local/redis-2.8.2/src
make && make test
ln -s /usr/local/redis-2.8.2/ /usr/local/redis
ln -s /usr/local/redis/src/redis-server /usr/local/bin/redis-server
ln -s /usr/local/redis/src/redis-cli /usr/local/bin/redis-cli
```
> 若编译提示`undefined reference to '__sync_add_and_fetch_4' `
> 
> 则参照 [http://blog.csdn.net/hmc20071120015/article/details/8142454](http://blog.csdn.net/hmc20071120015/article/details/8142454)
> 
> `src/Makefile`开头加 `CFLAGS= -march=i686`
> 
> `src/.make_settings`里的OPT，改为`OPT=-O2 -march=i686`

----------

> 若提示：`You need tcl 8.5 or newer in order to run the Redis test`
> 
> 则参照 [http://www.linuxfromscratch.org/blfs/view/cvs/general/tcl.html](http://www.linuxfromscratch.org/blfs/view/cvs/general/tcl.html)

```shell
yum -y install make gcc
#依赖tcl8.5
wget http://downloads.sourceforge.net/tcl/tcl8.5.10-src.tar.gz
tar zxvf tcl8.5.10-src.tar.gz
make && make install
```

## 安装扩展
```shell
wget https://codeload.github.com/nicolasff/phpredis/zip/master
unzip master

cd phpredis-master
/usr/local/php/bin/phpize
./configure --with-php-config=/usr/local/php/bin/php-config
make && make install
echo "extension = redis.so" >> /usr/local/php/etc/php.ini
```

## 启动服务脚本
> `redis.conf`中的`deamonize`默认是`no`，改成`yes`就可以在后台运行守护进程

```shell
vim /etc/init.d/redis
```

##### 启动文件内容
```shell
#!/bin/bash 
# 
# Init file for redis 
# 
# chkconfig: - 80 12 
# description: redis daemon 
# 
# processname: redis 
# config: /etc/redis.conf 
source /etc/init.d/functions
BIN="/usr/local/bin"
CONFIG="/usr/local/redis/redis.conf"
PIDFILE="/var/run/redis.pid"
### Read configuration 
[ -r "$SYSCONFIG" ] && source "$SYSCONFIG"
RETVAL=0
prog="redis-server"
desc="Redis Server"
start() {
    if [ -e $PIDFILE ];then
        echo "$desc already running...." 
        exit 1
    fi
    echo -n $"Starting $desc: " 
    daemon $BIN/$prog $CONFIG
    RETVAL=$?
    echo 
    [ $RETVAL -eq 0 ] && touch /var/lock/subsys/$prog
    return $RETVAL
}
stop() {
    echo -n $"Stop $desc: " 
    killproc $prog
    RETVAL=$?
    echo 
    [ $RETVAL -eq 0 ] && rm -f /var/lock/subsys/$prog $PIDFILE
    return $RETVAL
}
restart() {
    stop
    start
}
case "$1" in
    start)
        start
        ;;
    stop)
        stop
        ;;
    restart)
        restart
        ;;
    condrestart)
        [ -e /var/lock/subsys/$prog ] && restart
        RETVAL=$?
        ;;
    status)
        status $prog
        RETVAL=$?
        ;;
   *)
        echo $"Usage: $0 {start|stop|restart|condrestart|status}" 
        RETVAL=1
esac
exit $RETVAL
```

##### 开机启动
```shell
chmod +x  /etc/init.d/redis
chkconfig redis on
```