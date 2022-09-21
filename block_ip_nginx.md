# Block địa chỉ IP trên Nginx

Trường hợp muốn chặn 1 IP cụ thể: VD chặn IP: 192.168.1.1
```
location / {
  deny 192.168.1.1;
}
```

Chặn IP truy cập tới 1 thư mục cụ thể
```
location /images {
  deny 192.168.1.1;
}
```

Trường hợp bạn muốn chặn hết chỉ cho 1 IP truy cập
```
location / {
  allow 192.168.1.1;
  deny all;
}
```

Bạn có thể kết hợp nhiều luật như chặn hết tất cả các IP 192.168.1.1 truy cập thư mục image
```
location /images {
   allow 192.168.1.1;
   deny all;
}
location / {
   deny all;
}
```

