---
title: "Workshop: Hướng Dẫn Triển Khai Forward Proxy Nginx"
seoTitle: "Guide to Setting Up Nginx Forward Proxy"
seoDescription: "Tìm hiểu cách triển khai forward proxy với Squid và Nginx để cải thiện bảo mật và vượt tường lửa trên hệ thống Ubuntu và macOS"
datePublished: Sat Nov 30 2024 17:26:49 GMT+0000 (Coordinated Universal Time)
cuid: cm44g2tsf000309l1hvvp5dp3
slug: workshop-huong-dan-trien-khai-forward-proxy-nginx
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1732987506410/589588f9-10c5-4423-b10c-c550821696d9.jpeg
tags: nginx, devops, forward-proxy

---

Trong bài viết này, chúng ta sẽ tìm hiểu về forward proxy và cách triển khai nó trong thực tế. Forward proxy là một loại máy chủ trung gian đứng giữa người dùng cuối và Internet, giúp che giấu thông tin của người dùng và cung cấp các tính năng khác như vượt tường lửa, kiểm soát truy cập, và tăng cường bảo mật. Hãy cùng nhau khám phá cách triển khai forward proxy bằng Squid Proxy và Nginx, từ cơ bản đến triển khai thực tế.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732987543077/14e1ae97-1521-4158-a70f-b6e472123894.png align="center")

# Forward Proxy Là Gì?

Forward proxy là một máy chủ đứng giữa thiết bị của người dùng và Internet. Khi người dùng gửi yêu cầu đến một trang web, forward proxy sẽ nhận yêu cầu đó và chuyển tiếp tới trang web đích. Nó giống như một lớp bảo vệ trung gian, che giấu thông tin của người dùng và giúp họ có thể truy cập các tài nguyên bị hạn chế.

Một số lợi ích chính của forward proxy bao gồm:

1. **Ẩn danh tính**: Người dùng có thể ẩn địa chỉ IP thực của mình khi truy cập Internet, điều này giúp tăng cường quyền riêng tư.
    
2. **Bỏ qua hạn chế truy cập**: Người dùng có thể sử dụng forward proxy để vượt qua các hạn chế truy cập, chẳng hạn như các website bị chặn bởi tổ chức hoặc quốc gia.
    
3. **Quản lý và kiểm soát truy cập**: Doanh nghiệp có thể sử dụng forward proxy để kiểm soát truy cập Internet của nhân viên, giới hạn các trang web không mong muốn.
    

# Các Bước Triển Khai Forward Proxy với Squid Proxy trên Ubuntu và macOS

#### Bước 1: Cài Đặt Squid Proxy

Squid là một phần mềm forward proxy mã nguồn mở phổ biến và dễ sử dụng. Trên Ubuntu hoặc Debian, bạn có thể cài đặt Squid bằng lệnh sau:

```
sudo apt update
sudo apt install squid
```

Trên macOS, bạn có thể cài đặt Squid bằng Homebrew:

```
brew install squid
```

#### Bước 2: Cấu Hình Squid Proxy

Sau khi cài đặt Squid, chúng ta cần cấu hình nó để hoạt động như một forward proxy. Bạn có thể chỉnh sửa tệp cấu hình `/etc/squid/squid.conf` (đối với Ubuntu) hoặc `/usr/local/etc/squid/squid.conf` (đối với macOS) bằng lệnh sau:

Ubuntu/Debian:

```
sudo nano /etc/squid/squid.conf
```

macOS:

```
# Kiểm tra xem Squid đã được cài đặt ở đâu
SQUID_PATH=$(which squid)
SQUID_CONF_PATH=$(dirname $SQUID_PATH)/../etc/squid.conf
nano $SQUID_CONF_PATH
```

Thêm hoặc chỉnh sửa các dòng sau để cho phép kết nối từ mạng cục bộ:

```
acl localnet src 192.168.1.0/24   # Cho phép mạng cục bộ truy cập vào proxy
http_access allow localnet
http_access deny all              # Từ chối tất cả các yêu cầu khác
```

Lưu và thoát khỏi trình soạn thảo.

#### Bước 3: Khởi Động Lại Squid

Sau khi chỉnh sửa cấu hình, bạn cần khởi động lại Squid để áp dụng thay đổi:

Ubuntu/Debian:

```
sudo systemctl restart squid
```

macOS:

```
brew services restart squid
```

Kiểm tra lại xem squid đang chạy port nào trên macos:

```bash
SQUID_PATH=$(which squid)                                
SQUID_CONF_PATH=$(dirname $SQUID_PATH)/../etc/squid.conf
cat $SQUID_CONF_PATH | grep -i "http_port"
```

#### Bước 4: Cấu Hình Trình Duyệt để Sử Dụng Forward Proxy

Để kiểm tra forward proxy hoạt động, bạn cần cấu hình trình duyệt của mình để sử dụng proxy. Truy cập vào cài đặt mạng của trình duyệt, và nhập địa chỉ IP của máy chủ proxy (ví dụ: `192.168.1.1`) và cổng Squid đang nghe (thường là `3128`).

# Forward Proxy với Nginx trên Ubuntu và macOS

Ngoài Squid, bạn cũng có thể sử dụng Nginx để triển khai forward proxy, mặc dù chức năng này không phải là điểm mạnh của Nginx như reverse proxy. Dưới đây là cách cấu hình cơ bản.

#### Bước 1: Cài Đặt Nginx

Nếu bạn chưa cài đặt Nginx, bạn có thể thực hiện lệnh sau trên Ubuntu/Debian:

```
sudo apt update
sudo apt install nginx
```

Hoặc trên macOS với Homebrew:

```
brew install nginx
```

#### Bước 2: Cấu Hình Nginx như một Forward Proxy

Mở tệp cấu hình Nginx:

Ubuntu/Debian:

```
sudo nano /etc/nginx/nginx.conf
```

macOS:

```
# Kiểm tra xem Nginx đã được cài đặt ở đâu
# Mình thích dùng vim trên macos nên bạn muốn dùng nano thì replace vim bằng nano nhé !!
NGINX_PATH=$(which nginx)
NGINX_CONF_PATH=$(dirname $NGINX_PATH)/../etc/nginx/nginx.conf
vim $NGINX_CONF_PATH
```

Thêm cấu hình sau để Nginx hoạt động như một forward proxy đơn giản:

```
server {
    listen 8888;
    location / {
        proxy_pass $scheme://$http_host$request_uri;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

Lưu và thoát khỏi trình soạn thảo.

#### Bước 3: Kiểm Tra và Khởi Động Lại Nginx

Kiểm tra cấu hình Nginx có hợp lệ không:

```
sudo nginx -t
```

Sau đó khởi động lại Nginx để áp dụng thay đổi:

Ubuntu/Debian:

```
sudo systemctl restart nginx
```

macOS:

```
# Kiểm tra xem Nginx đã được cài đặt ở đâu và reload cấu hình
NGINX_PATH=$(which nginx)
NGINX_CONF_PATH=$(dirname $NGINX_PATH)/../etc/nginx/nginx.conf
sudo nginx -s reload
```

### Các Tình Huống Sử Dụng Thực Tế của Forward Proxy

1. **Vượt Tường Lửa và Truy Cập Nội Dung Bị Chặn**: Forward proxy thường được sử dụng để giúp người dùng vượt qua các hạn chế truy cập, chẳng hạn như khi một số nội dung bị chặn bởi quốc gia hoặc công ty.
    
2. **Kiểm Soát Truy Cập Internet trong Doanh Nghiệp**: Các doanh nghiệp sử dụng forward proxy để kiểm soát truy cập của nhân viên vào Internet, giúp ngăn chặn các truy cập vào những trang web không phù hợp hoặc không cần thiết.
    
3. **Ẩn Danh Người Dùng**: Forward proxy có thể giúp ẩn địa chỉ IP của người dùng, bảo vệ quyền riêng tư của họ khi truy cập vào Internet. Nhưng mà chỉ khi proxy server và client không chung server nhé, nếu không thì nó lại chung 1 IP mất rồi :vvv.
    

# Workshop Thực Hành: Tích Hợp Forward Proxy với HTTPS trên Ubuntu và macOS

Để bảo mật thông tin, bạn có thể tích hợp forward proxy với SSL. Điều này đặc biệt hữu ích khi bạn muốn đảm bảo tất cả các kết nối từ người dùng đến proxy đều được mã hóa.

#### Bước 1: Cài Đặt Chứng Chỉ SSL

Bạn có thể cài đặt Certbot để tự động lấy chứng chỉ SSL từ Let’s Encrypt:

Ubuntu/Debian:

```
sudo apt install certbot python3-certbot-nginx
```

macOS:

```
brew install certbot
```

#### Bước 2: Lấy Chứng Chỉ SSL

Chạy lệnh sau để lấy chứng chỉ SSL cho tên miền của bạn (dành cho macOS và Ubuntu):

```
sudo certbot --nginx -d example.com -d www.example.com
```

Certbot sẽ tự động cấu hình Nginx để sử dụng SSL cho tên miền của bạn. Sau khi hoàn tất, Nginx sẽ được cấu hình để chuyển hướng tất cả lưu lượng HTTP sang HTTPS, giúp tăng cường bảo mật.

#### Bước 3: Cấu Hình Squid hoặc Nginx để Sử Dụng SSL

Cập nhật cấu hình Squid hoặc Nginx để sử dụng chứng chỉ SSL mà bạn đã lấy từ Let’s Encrypt. Điều này đảm bảo rằng tất cả lưu lượng đều được mã hóa, bảo vệ an toàn thông tin người dùng.

# Kết Luận

Forward proxy là một công cụ hữu ích trong việc bảo mật, quản lý truy cập và giúp người dùng vượt qua các hạn chế truy cập trên Internet. Bằng cách sử dụng Squid Proxy hoặc Nginx, bạn có thể dễ dàng triển khai forward proxy để phù hợp với nhu cầu của mình trên cả Ubuntu và macOS. Qua workshop này, hy vọng bạn đã hiểu rõ hơn về cách forward proxy hoạt động và cách triển khai nó trong môi trường thực tế.

Hãy tiếp tục thử nghiệm và cập nhật thêm những kinh nghiệm của bạn vào blog này để chia sẻ với cộng đồng!