# Cài đặt Nginx theo source

# Cài đặt gói cần thiết để complile Nginx từ source
```
yum groupinstall " Development Tools"  -y
yum install zlib-devel pcre-devel openssl-devel wget -y
yum install epel-release -y
```

# Cài đặt thêm các thành phần gói phụ thuộc
```
yum install perl perl-devel perl-ExtUtils-Embed libxslt libxslt-devel libxml2 libxml2-devel gd gd-devel GeoIP GeoIP-devel -y
```

wget https://nginx.org/download/nginx-1.18.0.tar.gz
tar -xzf nginx-1.18.0.tar.gz

./configure --with-pcre --prefix=/etc/nginx --user=nginx --group=nginx --with-threads --with-file-aio --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_addition_module --with-http_xslt_module=dynamic --with-http_image_filter_module --with-http_geoip_module=dynamic --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_auth_request_module --with-http_random_index_module --with-http_secure_link_module --with-http_degradation_module --with-http_slice_module --with-http_stub_status_module --without-http_charset_module --with-http_perl_module --with-mail=dynamic --with-mail_ssl_module --with-stream=dynamic --with-stream_ssl_module --with-stream_realip_module --with-stream_geoip_module=dynamic --with-stream_ssl_preread_module

make

make install

# Tạo user và tiến hành phân quyền owner cho thư mục

```
useradd nginx
chown -R nginx:nginx /etc/nginx/
```

# Tạo file để chạy lệnh mỗi khi stop hoặc start service nginx

```
cat >> /usr/lib/systemd/system/nginx.service << "EOF"
[Unit]
Description=nginx - high performance web server
Documentation=https://nginx.org/en/docs/
After=network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/var/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/conf/nginx.conf
ExecStart=/usr/sbin/nginx -c /etc/nginx/conf/nginx.conf
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID

[Install]
WantedBy=multi-user.target
EOF
```

Start Service nginx và cấu hình auto start service mỗi khi server reboot
```
systemctl start nginx
systemctl enable nginx
```
