---
title: "Kiểm tra User Tồn Tại Trong Hệ Thống 100 Triệu User Bằng Cách Dùng Bitmap Trong Redis (Phần 2)"
seoTitle: "Redis Bitmap for User Presence Check"
seoDescription: "Xây dựng ứng dụng Python sử dụng bitmap trong Redis để quản lý 100 triệu user, tối ưu hiệu năng và so sánh với cơ sở dữ liệu truyền thống"
datePublished: Sat Dec 14 2024 03:18:27 GMT+0000 (Coordinated Universal Time)
cuid: cm4nlxqvb000009laept8dd77
slug: kiem-tra-user-ton-tai-trong-he-thong-100-trieu-user-bang-cach-dung-bitmap-trong-redis-phan-2
tags: redis, devops

---

### 1\. **Xây Dựng Ứng Dụng Thực Tế Sử Dụng Bitmap Trong Redis**

Trong phần 1, chúng ta đã tìm hiểu về khái niệm Bitmap trong Redis và cách sử dụng các lệnh cơ bản như `SETBIT`, `GETBIT`, và `BITCOUNT` để lưu trữ và kiểm tra sự tồn tại của user. Trong phần này, chúng ta sẽ xây dựng một ứng dụng Python thực tế để quản lý 100 triệu user sử dụng Bitmap trong Redis.

---

### 2\. **Chuẩn Bị Môi Trường**

#### **Cài Đặt Redis**

Nếu bạn chưa cài đặt Redis, hãy cài đặt bằng cách:

* **Ubuntu/Linux:**
    
    ```bash
    sudo apt update
    sudo apt install redis-server
    ```
    
* **macOS (sử dụng Homebrew):**
    
    ```bash
    brew install redis
    ```
    

Khởi động Redis:

```bash
redis-server
```

#### **Cài Đặt Thư Viện Redis cho Python**

Chúng ta sẽ sử dụng thư viện `redis` trong Python để giao tiếp với Redis:

```bash
pip install redis

# Hoặc dùng pip3 nếu dùng python3
pip3 install redis --break-system-packages
```

---

### 3\. **Xây Dựng Chức Năng Kiểm Tra User**

Dưới đây là code Python để thêm và kiểm tra user bằng Bitmap trong Redis:

```python
import redis

# Kết nối tới Redis
redis_client = redis.StrictRedis(host='localhost', port=6379, db=0)

def add_user(user_id):
    """
    Đánh dấu user với ID cụ thể là tồn tại trong hệ thống.
    """
    redis_client.setbit("users_bitmap", user_id, 1)
    print(f"User {user_id} đã được thêm vào hệ thống.")

def check_user(user_id):
    """
    Kiểm tra xem user với ID cụ thể có tồn tại hay không.
    """
    exists = redis_client.getbit("users_bitmap", user_id)
    if exists:
        print(f"User {user_id} tồn tại trong hệ thống.")
    else:
        print(f"User {user_id} không tồn tại trong hệ thống.")

def count_users():
    """
    Đếm tổng số user đã được thêm vào hệ thống.
    """
    count = redis_client.bitcount("users_bitmap")
    print(f"Tổng số user trong hệ thống: {count}")

# Thêm một số user
add_user(42)
add_user(1000000)
add_user(99999999)

# Kiểm tra user
check_user(42)
check_user(500)

# Đếm tổng số user
count_users()
```

---

### 4\. **Giải Thích Code**

1. **Kết Nối Redis**:
    
    ```python
    redis_client = redis.StrictRedis(host='localhost', port=6379, db=0)
    ```
    
    * Kết nối tới Redis trên [`localhost`](http://localhost) cổng 6379.
        
2. **Hàm** `add_user`:
    
    ```python
    def add_user(user_id):
        redis_client.setbit("users_bitmap", user_id, 1)
    ```
    
    * Đặt bit tại vị trí `user_id` thành `1` để đánh dấu user đã tồn tại.
        
3. **Hàm** `check_user`:
    
    ```python
    def check_user(user_id):
        exists = redis_client.getbit("users_bitmap", user_id)
    ```
    
    * Kiểm tra bit tại vị trí `user_id`.
        
    * Nếu bit là `1`, user tồn tại; nếu là `0`, user không tồn tại.
        
4. **Hàm** `count_users`:
    
    ```python
    def count_users():
        count = redis_client.bitcount("users_bitmap")
    ```
    
    * Đếm tổng số bit có giá trị `1` trong Bitmap để biết tổng số user đã được thêm.
        
5. **Chạy thử nghiệm**:
    
    * Thêm một số user và kiểm tra sự tồn tại của họ.
        
    * Đếm tổng số user hiện có trong hệ thống.
        

```bash
python3  main.py  # Exec python              
```

Outputs:

```bash
User 42 đã được thêm vào hệ thống.
User 1000000 đã được thêm vào hệ thống.
User 99999999 đã được thêm vào hệ thống.
User 42 tồn tại trong hệ thống.
User 500 không tồn tại trong hệ thống.
Tổng số user trong hệ thống: 3
```

### 5\. **So Sánh Với Phương Pháp Truyền Thống**

#### **Phương Pháp Truyền Thống Sử Dụng Cơ Sở Dữ Liệu Quan Hệ**

Nếu dùng cơ sở dữ liệu như MySQL hay PostgreSQL để lưu trữ 100 triệu user, bạn sẽ cần:

1. **Tạo bảng**:
    
    ```sql
    CREATE TABLE users (
        id BIGINT PRIMARY KEY
    );
    ```
    
2. **Chèn dữ liệu**:
    
    ```sql
    INSERT INTO users (id) VALUES (42), (1000000), (99999999);
    ```
    
3. **Kiểm tra user**:
    
    ```sql
    SELECT * FROM users WHERE id = 42;
    ```
    

**Nhược điểm**:

* **Tốn nhiều bộ nhớ**: Mỗi bản ghi trong cơ sở dữ liệu tốn nhiều byte.
    
* **Tốc độ chậm**: Truy vấn phải qua nhiều bước xử lý, làm chậm hiệu năng khi số lượng user lớn.
    

#### **Ưu Điểm Của Bitmap Trong Redis**

* **Bộ nhớ nhỏ**: 100 triệu user chỉ tốn khoảng **12.5 MB**.
    
* **Tốc độ truy vấn O(1)**: Kiểm tra và thêm user diễn ra rất nhanh.
    
* **Đơn giản**: Code sử dụng Bitmap ngắn gọn và dễ hiểu.
    

---

### 6\. **Các Lưu Ý Khi Sử Dụng Bitmap**

1. **Giới hạn vị trí bit**: Redis cho phép lưu trữ các bit tại vị trí lên đến **2^32 (khoảng 4.2 tỷ)**. Nếu user ID vượt quá con số này, bạn cần giải pháp khác hoặc phân chia dữ liệu.
    
2. **Xóa dữ liệu**: Bitmap không cho phép xóa riêng lẻ từng bit dễ dàng. Để xóa một user, bạn phải đặt bit tại vị trí đó thành `0`.
    
3. **Dữ liệu không an toàn**: Nếu Redis không được cấu hình lưu trữ bền vững, dữ liệu có thể bị mất khi Redis khởi động lại. Hãy cấu hình Redis Persistence hoặc sao lưu thường xuyên.
    

---

### 7\. **Kết Luận Phần 2**

Trong phần này, chúng ta đã:

1. Xây dựng một ứng dụng Python thực tế để lưu trữ và kiểm tra user bằng Bitmap trong Redis.
    
2. So sánh với phương pháp truyền thống sử dụng cơ sở dữ liệu quan hệ.
    
3. Thảo luận về ưu điểm và các lưu ý khi sử dụng Bitmap.
    

Trong **Phần 3**, chúng ta sẽ mở rộng ứng dụng này để:

1. **Xử lý trường hợp mở rộng quy mô lớn hơn 100 triệu user.**
    
2. **Tối ưu hóa hiệu suất khi thêm và kiểm tra nhiều user cùng lúc.**
    
3. **Triển khai Redis Cluster để đảm bảo hiệu năng và độ tin cậy.**