---
title: "apt proxy with nexus"
seoTitle: "Configuring Apt Proxy with Nexus Proxy Server"
seoDescription: "Learn how to configure an apt proxy with Nexus for successful apt-get updates and installing utilities like wget and curl"
datePublished: Wed Mar 26 2025 09:16:51 GMT+0000 (Coordinated Universal Time)
cuid: cm8ppnj0t000009gw3e475ibf
slug: apt-proxy-with-nexus
tags: linux, apt

---

Có thể thử các bước sau trong terminal của pod để thay đổi tất cả các file cấu hình apt (bao gồm cả file trong /etc/apt/sources.list.d) nhằm đảm bảo apt-get update thành công và sau đó cài đặt wget, curl được:

1. **Thay đổi nguồn cho các file apt:**  
    Vì thông báo cho biết "Ubuntu sources have moved to the /etc/apt/sources.list.d/ubuntu.sources file", nên cần thay đổi cả file đó:
    
    ```markdown
    # Định nghĩa biến localhost
    APT_ARCHIVE=https://localhost/repository/apt-archive.ubuntu.com-proxy/
    APT_SECURITY=https://localhost/repository/apt-security.ubuntu.com-proxy/
    
    # Thay thế URL trong /etc/apt/sources.list
    sed -i "s|http://archive.ubuntu.com/|${APT_ARCHIVE}|g" /etc/apt/sources.list
    sed -i "s|http://security.ubuntu.com/|${APT_SECURITY}|g" /etc/apt/sources.list
    
    # Nếu file /etc/apt/sources.list.d/ubuntu.sources tồn tại, thay thế URL ở đó
    if [ -f /etc/apt/sources.list.d/ubuntu.sources ]; then
        sed -i "s|http://archive.ubuntu.com/|${APT_ARCHIVE}|g" /etc/apt/sources.list.d/ubuntu.sources
        sed -i "s|http://security.ubuntu.com/|${APT_SECURITY}|g" /etc/apt/sources.list.d/ubuntu.sources
    fi
    ```
    
2. **Cập nhật apt và cài đặt wget, curl:**
    
    ```markdown
    apt-get update && apt-get install -y wget curl
    ```
    

Như vậy, sau khi chạy các lệnh trên, apt-get update sẽ sử dụng mirror proxy đã cấu hình. Sau đó, có thể cài đặt wget và curl để tải file hoặc thực hiện các tác vụ khác.