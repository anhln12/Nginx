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

Khi truy cập các file tĩnh, bạn cũng không nên log lại vì quá trình log sẽ làm giảm tốc độ xử lý của Nginx, chúng ta nên bỏ luôn việc log khi truy cập các file tĩnh.

Thường thì việc liên lạc giữa Nginx và PHP-FPM sẽ sử dụng tcp socket. Việc này có thể sẽ làm chậm tốc độ đáng kể so với việc sử dụng unix socket. Do đó bạn cần chỉnh lại thay vì sử dụng tcp socket nên sử dụng unix socket cho việc truyền tải thông tin. Nếu bạn sử dụng ssd thì việc này có thể sẽ càng hiệu quả.

```
location ~* .php$ {
fastcgi_index  index.php;
fastcgi_pass  127.0.0.1:9000;
include     fastcgi_params;
fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
fastcgi_param  SCRIPT_NAME    $fastcgi_script_name;
}
```

Bạn nên chuyển thành như sau:
```
location ~* .php$ {
fastcgi_index  index.php;
#Chinh tai day
fastcgi_pass  unix:/var/run/php-fpm/php-fpm.sock;
include     fastcgi_params;
fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
fastcgi_param  SCRIPT_NAME    $fastcgi_script_name;
}
```

Bạn cũng không nên cho phép truy cập các file hoặc thư mục ẩn. File và thư mục ẩn trên Linux sẽ có dấu chấm (.) trước tên file, thư mục. Vì vậy, bạn có thể cấu hình như sau để không cho phép nó truy cập trực tiếp vào.
```
location ~ /. {
access_log off;
log_not_found off;
deny all;
}
```

# Cách tối ưu PHP-FPM
Điều chỉnh đường dẫn file sock giống như trong file Nginx ở trên:
```
listen = /var/run/php-fpm/php-fpm.sock
user = site
group = site
request_slowlog_timeout = 5s
slowlog = /var/log/php-fpm/slowlog-site.log
listen.allowed_clients = 127.0.0.1
pm = dynamic
pm.max_children = 5
pm.start_servers = 3
pm.min_spare_servers = 2
pm.max_spare_servers = 4
pm.max_requests = 200
listen.backlog = -1
pm.status_path = /status
request_terminate_timeout = 120s
rlimit_files = 131072
rlimit_core = unlimited
catch_workers_output = yes
env[HOSTNAME] = $HOSTNAME
env[TMP] = /tmp
env[TMPDIR] = /tmp
env[TEMP] = /tmp
```

Đối với PHP-FPM thì các bạn chỉ cần quan tâm đến các thông số sau: pm , pm.max_children , pm.start_servers , pm.min_spare_servers và pm.max_spare_servers .

Trong đó, pm là chế độ quản lý process của PHP-FPM gồm có static ondemand, dynamic. Thường ta sử dụng dynamic như trên. Theo đó thì ý nghĩa các thông số pm.max_children , pm.start_servers , pm.min_spare_servers , pm.max_spare_servers lần lượt sẽ là:
```
pm.max_children = Số process con (child processes) tối đa được tạo (tương đương tổng số request có thể phục vụ)
pm.start_servers = Tổng số child processes được tạo khi khởi động php-fpm (được tính bằng công thức `min_spare_servers + (max_spare_servers – min_spare_servers) / 2` )
pm.min_spare_servers = Tổng số child process nhàn rỗi tối thiểu được duy trì.
pm.max_spare_servers = Tổng số child process nhàn rỗi tối đa được duy trì.
```

Cách tính pm.max_children

Bước 1: Xác định số lượng bộ nhớ cần thiết trung bình cho 1 process php-fpm
Bạn tiến hành chạy lệnh:
```
ps -ylC php-fpm --sort:rss
```

Output sẽ tương tự như sau, column RSS sẽ chứa giá trị mà ta cần xác định:
```
S   UID   PID  PPID  C PRI  NI   RSS    SZ WCHAN  TTY          TIME CMD
S     0 24439     1  0  80   0  6364 57236 -      ?        00:00:00 php-fpm
S    33 24701 24439  2  80   0 61588 63335 -      ?        00:04:07 php-fpm
S    33 25319 24439  2  80   0 61620 63314 -      ?        00:02:35 php-fpm
```

Kết quả trên là khoảng 61620KB, tương đương với ~60MB/process.

Bước 2: Tính toán ra pm.max_children

Nếu server hiện tại có khoảng 4GB RAM đang chạy cả web và DB, chúng ta sẽ tính con số ước lượng là DB hoạt động khoảng 1GB và 0.5G dành cho cả buffer. Với các con số như vậy thì dung lượng RAM còn lại hoạt động của php-fpm tương đương với: 4 – 4 – 0,5 = 2,5 GB RAM hoặc 2560 Mb.
```
pm.max_children = 2560 Mb / 60 Mb = 42
```
Làm tròn con số xuống để đảm bảo server hoạt động không bị quá tải thì pm.max_children = 40 .

Cách tính pm.min_spare_servers

pm.min_spare_servers có giá trị tương đương với 20% của pm.max_children .

Nếu với giá trị trên thì pm.min_spare_servers = 20% * 40 = 8 .

Cách tính pm.max_spare_servers

pm.max_spare_servers có giá trị tương đương với 60% của pm.max_children .

Nếu với gái trị như trên thì pm.max_spare_servers = 60% * 40 = 24

Cách tính pm.max_requests

Tham số này chính là số lượng request xử lý đồng thời mà server có thể chịu tải được, giá trị phụ thuộc vào pm.max_children và số lượng request trên 1s vào server. Con số này đôi khi cũng không đúng, nhưng có 1 phương pháp đó là sử dụng tool ab của apache sau đó giảm giá trị dần dần sao cho phù hợp.
```
ab -n 5000 -c 100 http://domain.com/
```

Khi chạy command trên thì có nghĩa là tạo 5000 request với 100 session hoạt đồng cùng lúc truy cập vào url http://domain.com . Giá trị pm.max_requests có thể set là 1000, sau đó tăng/giảm dần đến mức phù hợp.
