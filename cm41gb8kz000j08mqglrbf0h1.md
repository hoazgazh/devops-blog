---
title: "IAM trong AWS: Cách Quản Lý Nhận Diện và Truy Cập Đám Mây Hiệu Quả"
seoTitle: "Quản lý Truy cập AWS với IAM"
seoDescription: "Tìm hiểu cách IAM quản lý quyền truy cập và nhận diện trên AWS, bảo mật tài nguyên và giao tiếp giữa các dịch vụ trong cloud"
datePublished: Thu Nov 28 2024 15:10:03 GMT+0000 (Coordinated Universal Time)
cuid: cm41gb8kz000j08mqglrbf0h1
slug: iam-trong-aws-cach-quan-ly-nhan-dien-va-truy-cap-dam-may-hieu-qua
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1732969112479/4801ea30-e597-4799-b7c1-4bdb2be64184.webp
tags: aws, devops

---

Trong thế giới điện toán đám mây, bảo mật là một yếu tố không thể thiếu, và AWS Identity and Access Management (IAM) đóng vai trò quan trọng trong việc đảm bảo rằng chỉ những người hoặc ứng dụng được phép mới có thể truy cập tài nguyên. Blog này sẽ giúp bạn hiểu cơ bản về IAM, cách xác thực và cách các dịch vụ bên ngoài cloud giao tiếp với cloud AWS.

---

### **IAM là gì?**

IAM (Identity and Access Management) trong AWS là một dịch vụ cho phép bạn kiểm soát quyền truy cập vào tài nguyên AWS. Nó giúp bạn:

* Quản lý ai có thể truy cập (người dùng, nhóm, vai trò).
    
* Xác định tài nguyên nào họ có thể truy cập.
    
* Giới hạn cách họ có thể sử dụng tài nguyên đó.
    

---

### **Thành phần chính của IAM**

1. **Người dùng (Users):**
    
    * Tài khoản đại diện cho một cá nhân hoặc ứng dụng.
        
    * Ví dụ: Bạn tạo một người dùng tên "AdminNgoc" để quản lý tài nguyên AWS.
        
2. **Nhóm (Groups):**
    
    * Tập hợp các người dùng có cùng quyền.
        
    * Ví dụ: Tạo nhóm "Developers" để cấp quyền truy cập vào dịch vụ EC2 hoặc Lambda.
        
3. **Vai trò (Roles):**
    
    * Tương tự người dùng, nhưng dành cho các ứng dụng hoặc dịch vụ AWS (ví dụ, EC2 hoặc Lambda) để thực hiện các tác vụ mà không cần thông tin đăng nhập.
        
    * Ví dụ: Một vai trò cho phép EC2 tải dữ liệu từ S3 mà không cần thông tin tài khoản.
        
4. **Chính sách (Policies):**
    
    * Tập hợp các quy tắc xác định quyền (cho phép hoặc từ chối).
        
    * Ví dụ: Policy cho phép đọc dữ liệu từ S3 Bucket.
        

---

### **Cách xác thực trong AWS**

AWS hỗ trợ hai loại xác thực chính:

1. **Xác thực bằng Console (UI):**
    
    * Dùng tài khoản IAM hoặc root để đăng nhập vào giao diện AWS Management Console.
        
    * Hạn chế: Không nên dùng tài khoản root cho các tác vụ thường xuyên.
        
2. **Xác thực bằng API hoặc SDK:**
    
    * Dùng Access Key ID và Secret Access Key.
        
    * Các ứng dụng bên ngoài AWS sử dụng cặp khóa này để giao tiếp với các dịch vụ AWS.
        

---

### **Ví dụ: Cách giao tiếp từ bên ngoài AWS**

#### **Tình huống: Ứng dụng ngoài cloud truy cập S3 Bucket**

1. **Cài đặt IAM Policy:**
    
    * Tạo một policy cho phép đọc ghi từ một bucket S3.
        
    
    ```json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": "s3:*",
          "Resource": "arn:aws:s3:::my-example-bucket/*"
        }
      ]
    }
    ```
    
2. **Tạo IAM User và gán Policy:**
    
    * Tạo một người dùng IAM (ví dụ: "AppUser") và gán policy vừa tạo.
        
3. **Cấp Access Key và Secret Key:**
    
    * Khi tạo người dùng, tải xuống Access Key ID và Secret Access Key.
        
4. **Cấu hình ứng dụng:**
    
    * Trong ứng dụng (ví dụ: Python), sử dụng thư viện `boto3` để giao tiếp với S3.
        
    
    ```python
    import boto3
    
    # Cấu hình AWS
    s3_client = boto3.client(
        's3',
        aws_access_key_id='YourAccessKeyID',
        aws_secret_access_key='YourSecretAccessKey'
    )
    
    # Tải tệp lên S3
    s3_client.upload_file('local_file.txt', 'my-example-bucket', 'remote_file.txt')
    
    print("Tệp đã được tải lên thành công!")
    ```
    

---

### **Cách dịch vụ trong AWS giao tiếp với nhau**

Dịch vụ AWS sử dụng **IAM Roles** để giao tiếp mà không cần thông tin đăng nhập. Ví dụ:

* Một EC2 instance tải dữ liệu từ S3 mà không cần Access Key.
    

**Cách thực hiện:**

1. Tạo một IAM Role với quyền truy cập S3.
    
2. Gán vai trò đó cho EC2 instance.
    
3. EC2 tự động sử dụng vai trò để truy cập S3.
    

---

### **Lời kết**

IAM trong AWS giúp quản lý quyền truy cập hiệu quả, từ cá nhân, ứng dụng đến các dịch vụ trong AWS. Hiểu và áp dụng đúng IAM không chỉ giúp bảo vệ tài nguyên mà còn giúp hệ thống hoạt động linh hoạt và an toàn hơn.

Bạn đã từng triển khai IAM chưa? Hãy chia sẻ trải nghiệm của bạn trong phần bình luận!