# Hướng dẫn cài đặt Nginx + LuaJIT bắt Header
# Mình sẽ chèn nginx (đứng trước/đứng sau) để bắt xem Header, Body của các API nó hoạt động ra sao

```
Version mình sử dụng:
- Nginx-1.17.10 (Bản stable mới nhất tính đến 04.2020)
- LuaJIT-2.0.5
- lua-nginx-module-0.10.13
- ngx_devel_kit-0.3.1rc1
- Áp dụng cho cả Centos6/7. Ở đây mình đang test trên bản 7 (CentOS 6 mình cũng cài đặt thành công
```
Bước 1: Cài đặt thư viện cơ bản hỗ trợ nginx
```
yum install wget readline-devel pcre pcre-devel openssl openssl-devel zlib zlib-devel gcc 
```
Bước 2: Cài đặt / Chuẩn bị LuaJIT
```
- Tải LuaJIT về và giải nén
cd /opt
wget http://luajit.org/download/LuaJIT-2.0.5.tar.gz
tar zxf LuaJIT-2.0.5.tar.gz
cd LuaJIT-2.0.5

- Chèn export ở trên đầu, lưu file lại
vim Makefile
export PREFIX= /opt/luajit-2.0.5

- Thực hiện Compile LuaJIT
make 
make install
```

Set biến môi trường tạm thời
```
export LUAJIT_LIB=/usr/local/lib
export LUAJIT_INC=/usr/local/include/luajit-2.0
```

Bước 3: Cài đặt nginx và các thư viện:
```
cd /opt
wget http://nginx.org/download/nginx-1.17.10.tar.gz
wget https://github.com/openresty/lua-nginx-module/archive/v0.10.13.tar.gz
wget https://github.com/simplresty/ngx_devel_kit/archive/v0.3.1rc1.tar.gz
tar -xvzf nginx-1.17.10.tar.gz 
tar -xvzf v0.10.13.tar.gz 
tar -xvzf v0.3.1rc1.tar.gz 

cd nginx-1.17.10
mkdir -p /opt/nginx
mkdir /opt/nginx/logs/
mkdir /run

- Thực hiện Compile Nginx, ở đây mình đặt Nginx ở thư mục OPT ưa thích (không phải /etc)

./configure  --sbin-path=/usr/bin/nginx --prefix=/opt/nginx --conf-path=/opt/nginx/nginx.conf --error-log-path=/opt/nginx/logs/error.log --http-log-path=/opt/nginx/logs/access.log --with-pcre --pid-path=/var/run/nginx.pid --with-http_ssl_module --with-http_realip_module --with-http_stub_status_module --with-http_stub_status_module --with-http_gzip_static_module --with-http_realip_module  --with-ld-opt="-Wl,-rpath,/usr/local/lib" --add-module=/opt/lua-nginx-module-0.10.13 --add-module=/opt/ngx_devel_kit-0.3.1rc1
make
make install
```

Tạo file khởi động cho nginx (init file)
```
touch /etc/init.d/nginx
chmod 755 /etc/init.d/nginx
vim /etc/init.d/nginx
```
Nội dung file
```
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemon
#
# chkconfig:   - 85 15 
# description:  Nginx is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /etc/nginx/nginx.conf
# config:      /etc/sysconfig/nginx
# pidfile:     /var/run/nginx.pid
 
# Source function library.
. /etc/rc.d/init.d/functions
 
# Source networking configuration.
. /etc/sysconfig/network
 
# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0
 
nginx="/usr/bin/nginx"
prog=$(basename $nginx)
 
NGINX_CONF_FILE="/opt/nginx/nginx.conf"
 
[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx
 
lockfile=/var/lock/subsys/nginx
 
make_dirs() {
   # make required directories
   user=`$nginx -V 2>&1 | grep "configure arguments:" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
   if [ -z "`grep $user /etc/passwd`" ]; then
       useradd -M -s /bin/nologin $user
   fi
   options=`$nginx -V 2>&1 | grep 'configure arguments:'`
   for opt in $options; do
       if [ `echo $opt | grep '.*-temp-path'` ]; then
           value=`echo $opt | cut -d "=" -f 2`
           if [ ! -d "$value" ]; then
               # echo "creating" $value
               mkdir -p $value && chown -R $user $value
           fi
       fi
   done
}
 
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

