# Tối ưu Nginx và PHP-FPM cho website có lưu lượng truy cập lớn
Tối ưu Nginx và PHP-FPM sẽ giúp chúng hoạt động hiệu quả hơn, sử dụng tài nguyên hợp lý hơn, tăng cường khả năng chịu tải của toàn bộ hệ thống, không gây lãng phí tài nguyên vô ích.

Mở file /etc/nginx/nginx.conf và điều chỉnh theo hướng dẫn dưới đây:
Đầu tiên bạn cần nắm công thức:
```
max_clients = worker_processes * worker_connections
```

Số lượng người truy cập tối đa Nginx có thể phục vụ bằng thông số worker_processes nhân với worker_connections . Mặc định sau khi cài đặt Nginx thì sẽ là:
```
worker_processes = 1
worker_connections = 1024.
```

Bạn chỉ cần chỉnh lại worker_processes bằng với số lượng CPU core trong server bạn cấu hình. Xem thông số CPU core thông qua lệnh
```
cat /proc/cpuinfo | grep processor
processor : 0
processor : 1
processor : 2
processor : 3
```
Trong trường hợp này là 4, bạn có thể chỉnh sửa thông số worker_processes bằng 4. Nhưng nếu bạn có ít hơn 4 mà vẫn ghi như vậy thì có thể khiến hệ thống hoạt động không ổn định và bị lỗi.

Với worker_connections mặc định là 1042 và worker_processes đã được điều chỉnh thành 4 thì lượng người truy cập tối đa đã lên đến 1024 * 4 = 4096.

Con số này đã đủ lớn rồi nên bạn không cần thay đổi gì thêm. Nếu bạn chỉ có 2 CPU Core nhưng muốn nâng cao số lượng truy cập thì bạn có thể nâng worker_connections lên thành 2048, nhưng nó có thể gây lỗi server nên cần thêm thông số sau đây vào file nginx.conf ngay trên đoạn cấu hình worker_connections .

```
worker_rlimit_nofile 2048;
```
Bạn cũng nên xóa thông tin phiên bản của Nginx đang sử dụng và các thông tin quan trọng của Nginx bằng việc sửa hoặc bổ sung thông số.
```
server_tokens off;
```

Có thể giới hạn kích thước body của các http request và buffer dùng xử lý http request thông qua việc thêm hai thông số sau đây vào file cấu hình:
```
client_max_body_size 20m;
client_body_buffer_size 128k;
```

Bạn cũng nên yêu cầu client cache lại các file tĩnh và ít bị thay đổi, điều đó sẽ giúp bạn tiết kiệm bằng thông hơn vì không cần phải tải lại các file tĩnh đó. Thêm nội dung sau vào từng virtual host trên Nginx:
```
location ~* .(jpg|jpeg|gif|png|css|js|ico|xml)$ {
access_log    off;
log_not_found   off;
expires      360d;
}
```
