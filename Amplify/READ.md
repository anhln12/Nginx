Nginx Amplify là công cụ tuyệt vời giúp giám sát tình trạng NGINX và Server theo thời gian thực, qua đó giúp phân tích và tối ưu các ứng dụng hoạt động dựa trên NGINX.

Nội dung bài viết:

1. Cài đặt Nginx Amplify
1.1. Đăng kí tài khoản Nginx Amplify
1.2. Cài đặt NGINX Amplify Agent trên hệ thống

2. Cài đặt nâng cao
2.1 Thông số Stub Status
2.2. Thông số Log.

3. Sử dụng Nginx Amplify
3.1. Graphs - Đồ thị giám sát
3.2. Dashboard - Bảng giám sát, điều khiển
3.3. Analyzer - Báo cáo về thiết lập hệ thống
3.4. Alerts - Cảnh báo hệ thống

1. Cài đặt Nginx Amplify
Yêu cầu hệ thống cho Nginx Amplify:

Chỉ hoạt động trên hệ điều hành Linux, cụ thể:
RHEL/CentOS/OEL 6 và RHEL/CentOS/OEL 7
Ubuntu 12.04, 14.04, 16.04, 17.04, 17.10
Debian 7, 8, 9
Amazon Linux 2017.09, Gentoo Linux (phiên bản Experimental Ebuild)
Chỉ hoạt động với Python 2.6, 2.7 và chưa hỗ trợ Python 3.0.
1.1. Đăng kí tài khoản Nginx Amplify
Làm theo các bước sau để đăng kí tài khoản.

Truy cập trang đăng kí Amplify Sign Up: https://amplify.nginx.com/dashboard
Điền đầy đủ thông tin và chấp nhận Terms of Service rồi click Sign Up
Tại cửa sổ tiếp theo, bạn cung cấp thêm một số thông tin về bản thân để Nginx support được tốt hơn.
Như vậy, bạn đã hoàn tất đăng kí. Lưu ý, Nginx Amplify sẽ bắt đầu giám sát hệ thống của bạn khi và chỉ khi bạn cài đặt thành công Amplify Agent trên mỗi server và kết nối thông qua API.

1.2. Cài đặt NGINX Amplify Agent trên hệ thống
Công cụ thu thập dữ liệu về hoạt động hệ thống thông qua việc cài đặt Nginx Amplify Agent.

Click vào biểu tượng +New System góc trên bên trái sau khi login bạn sẽ thấy hướng dẫn cài đặt Agent lên hệ thống của mình. Ví dụ như trong hình:
<img width="358" alt="image" src="https://github.com/anhln12/nginx/assets/18412583/f441c000-57a7-45f9-92ba-92f5b3ce51ed">

Kết nối SSH đến server
Sử dụng curl hoặc wget để download script install (trước đó nên chuyển về thư mục gốc cd /root)
Chạy lệnh cài đặt Amplify Agent
Nếu không có vấn đề gì xảy ra, bạn sẽ nhận được thông báo cài đặt thành công như bên dưới:
<img width="329" alt="image" src="https://github.com/anhln12/nginx/assets/18412583/f038dfb4-bcf4-4763-a162-bd0fa472333b">
Bạn chờ 1 vài phút sẽ thấy hệ thống hiển thị trên khu vực Systems & các đồ thị ở mục Graphs.

```
curl -L -O https://github.com/nginxinc/nginx-amplify-agent/raw/master/packages/install.sh
API_KEY='1b20b36c0f8e818eafb3c63ca2490cd2' sh ./install.sh
service amplify-agent restart
```
2. Cài đặt nâng cao
2.1. Thông số Stub Status
Quy trở lại website, nhân nút Continue để đến bước 2, cấu hình stub_status
![Uploading image.png…]()


