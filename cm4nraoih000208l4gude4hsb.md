---
title: "Kiểm tra User Tồn Tại Trong Hệ Thống 100 Triệu User Bằng Cách Dùng Bitmap Trong Redis (Phần 4)"
seoTitle: "Redis Bitmap Check: User Existence Verification"
seoDescription: "Sử dụng Bitmap trong Redis và Docker Compose để xây dựng API kiểm tra user qua email cho hệ thống 100 triệu user"
datePublished: Sat Dec 14 2024 05:48:29 GMT+0000 (Coordinated Universal Time)
cuid: cm4nraoih000208l4gude4hsb
slug: kiem-tra-user-ton-tai-trong-he-thong-100-trieu-user-bang-cach-dung-bitmap-trong-redis-phan-4

---

### 1\. **Giới Thiệu Phần 4**

Ở **Phần 3**, chúng ta đã mở rộng ứng dụng kiểm tra user bằng cách xử lý quy mô lớn hơn 100 triệu user, tối ưu hóa hiệu suất với Redis Pipeline và triển khai Redis Cluster để đảm bảo khả năng mở rộng.

Trong **Phần 4**, chúng ta sẽ tập trung vào:

1. **Xử lý user dạng email để sử dụng Bitmap trong Redis**.
    
2. **Xây dựng API kiểm tra user sử dụng Flask và Redis hỗ trợ email**.
    
3. **Triển khai API và Redis trên Docker bằng Docker Compose**.
    
4. **Khắc phục các lỗi thường gặp và bảo mật Redis**.
    

---

### 2\. **Xử Lý User Dạng Email Để Sử Dụng Bitmap Trong Redis**

#### **Chuyển Email Thành Số Nguyên Bằng Hashing**

Bitmap yêu cầu sử dụng số nguyên làm vị trí bit, do đó chúng ta cần chuyển đổi email thành số nguyên bằng **hàm băm MD5**. Kết quả băm sẽ được chuyển thành số nguyên và giới hạn trong khoảng `2^32` để phù hợp với Bitmap.

```python
import hashlib

def email_to_int(email):
    """Chuyển email thành một số nguyên bằng MD5 hash."""
    md5_hash = hashlib.md5(email.encode()).hexdigest()
    return int(md5_hash, 16) % (2**32)

# Ví dụ
email = "test@gmail.com"
email_id = email_to_int(email)
print(f"Email: {email} => ID: {email_id}")
```

---

### 3\. **Xây Dựng API Kiểm Tra User Sử Dụng Flask và Redis**

#### **Cấu Trúc Dự Án**

```
user_api/
│-- app.py
│-- requirements.txt
│-- config.py
└-- Dockerfile
```

* `requirements.txt`:
    
    ```plaintext
    Flask==3.0.0
    redis==5.0.1
    ```
    
* [`config.py`](http://config.py):
    
    ```python
    REDIS_HOST = 'redis'  # Tên service Redis trong Docker Compose
    REDIS_PORT = 6379
    REDIS_DB = 0
    ```
    

#### **Code API Flask (**[`app.py`](http://app.py))

```python
from flask import Flask, request, jsonify
import redis
import hashlib
import random
import string
import time
from config import REDIS_HOST, REDIS_PORT, REDIS_DB

app = Flask(__name__)
redis_client = redis.StrictRedis(host=REDIS_HOST, port=REDIS_PORT, db=REDIS_DB)

def email_to_int(email):
    """Chuyển email thành một số nguyên bằng MD5 hash."""
    md5_hash = hashlib.md5(email.encode()).hexdigest()
    return int(md5_hash, 16) % (2**32)

def get_bitmap_key(email_id):
    """Xác định key Bitmap dựa trên email ID."""
    partition_size = 100000000
    partition_index = email_id // partition_size
    return f"users_bitmap_{partition_index}"

def generate_random_email():
    """Tạo email ngẫu nhiên."""
    username = ''.join(random.choices(string.ascii_lowercase + string.digits, k=8))
    domain = random.choice(["example.com", "test.com", "mail.com", "demo.org"])
    return f"{username}@{domain}"

@app.route('/add_user', methods=['POST'])
def add_user():
    """API thêm user bằng email."""
    data = request.get_json()
    email = data.get('email')
    
    if email is None or not isinstance(email, str):
        return jsonify({'error': 'Invalid email'}), 400
    
    email_id = email_to_int(email)
    bitmap_key = get_bitmap_key(email_id)
    offset = email_id % 100000000
    redis_client.setbit(bitmap_key, offset, 1)
    
    return jsonify({'message': f'Email {email} đã được thêm vào {bitmap_key}, offset: {offset}'}), 201

@app.route('/user-generate', methods=['POST'])
def generate_users():
    """API tạo 100.000 user ngẫu nhiên."""
    start_time = time.time()
    total_users = 100000

    for _ in range(total_users):
        email = generate_random_email()
        email_id = email_to_int(email)
        bitmap_key = get_bitmap_key(email_id)
        offset = email_id % 100000000
        redis_client.setbit(bitmap_key, offset, 1)

    end_time = time.time()
    return jsonify({'message': f'{total_users} users đã được tạo thành công.', 'time_taken': f'{end_time - start_time:.2f} seconds'}), 201

@app.route('/check_user', methods=['GET'])
def check_user():
    """API kiểm tra user bằng email."""
    email = request.args.get('email')
    
    if email is None or not isinstance(email, str):
        return jsonify({'error': 'Invalid email'}), 400
    
    email_id = email_to_int(email)
    bitmap_key = get_bitmap_key(email_id)
    offset = email_id % 100000000
    exists = redis_client.getbit(bitmap_key, offset)
    
    if exists:
        return jsonify({'email': email, 'exists': True}), 200
    else:
        return jsonify({'email': email, 'exists': False}), 404
@app.route('/')
def home():
    return jsonify({'message': 'User API is running!'}), 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5005, debug=True)
```

---

### 4\. **Triển Khai API và Redis Trên Docker**

#### **Tạo** `Dockerfile` Cho Flask API

```Dockerfile
# Sử dụng Python 3.9
FROM python:3.9

# Sao chép project vào container
WORKDIR /app
COPY . /app

# Cài đặt các dependencies
RUN pip install -r requirements.txt

# Expose cổng 5000
EXPOSE 5000

# Chạy ứng dụng Flask
CMD ["python", "app.py"]
```

#### **Tạo** `docker-compose.yml`

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "5000:5000"  # Nếu cổng 5000 bận, đổi thành "5001:5000"
    depends_on:
      - redis

  redis:
    image: redis:latest
    ports:
      - "6379:6379"
```

#### **Cấu Trúc Thư Mục Cuối Cùng**

```json
user_api/
│-- app.py
│-- requirements.txt
│-- config.py
│-- Dockerfile
└-- docker-compose.yml
```

#### **Chạy Docker Compose**

Từ thư mục dự án, chạy lệnh sau để khởi động cả Flask API và Redis:

```bash
docker-compose up --build
```

#### **Kiểm Tra Ứng Dụng**

1. **Thêm User**:
    
    ```bash
    curl -X POST http://127.0.0.1:5000/add_user \
         -H "Content-Type: application/json" \
         -d '{"email": "test@gmail.com"}'
    ```
    
2. Thêm 100k User:
    
    ```bash
     curl -X POST "http://127.0.0.1:5005/user-generate"
    ```
    

1. **Kiểm Tra User**:
    
    ```bash
    curl "http://127.0.0.1:5000/check_user?email=test@gmail.com"
    ```
    
2. **Kiểm Tra Redis**:
    
    Kết nối vào Redis CLI và kiểm tra bit:
    
    ```bash
    docker exec -it <redis_container_name> redis-cli
    GETBIT users_bitmap_0 <offset>
    ```
    

---

### 5\. **Khắc Phục Lỗi Thường Gặp**

#### **Lỗi: Cổng 5000 Đã Được Sử Dụng**

Nếu bạn gặp lỗi cổng `5000` đã bị chiếm dụng:

* Đổi cổng trong `docker-compose.yml`:
    
    ```yaml
    ports:
      - "5001:5000"
    ```
    

#### **Lỗi: Redis Không Kết Nối Được**

* Đảm bảo Redis container đang chạy:
    
    ```bash
    docker ps
    ```
    
* Kiểm tra logs:
    
    ```bash
    docker-compose logs redis
    ```
    

---

### 6\. **Bảo Mật Redis**

1. **Thêm Password Cho Redis**
    
    Cập nhật `docker-compose.yml`:
    
    ```yaml
    redis:
      image: redis:latest
      command: redis-server --requirepass mypassword
      ports:
        - "6379:6379"
    ```
    
2. **Cập Nhật Kết Nối Redis Trong** [`config.py`](http://config.py)
    
    ```python
    REDIS_PASSWORD = 'mypassword'
    redis_client = redis.StrictRedis(host=REDIS_HOST, port=REDIS_PORT, db=REDIS_DB, password=REDIS_PASSWORD)
    ```
    

---

### 7\. **Kết Luận**

Trong phần này, chúng ta đã:

1. **Xử lý user dạng email để sử dụng Bitmap trong Redis**.
    
2. **Xây dựng API Flask để thêm và kiểm tra user bằng email**.
    
3. **Triển khai API và Redis trên Docker bằng Docker Compose**.
    
4. **Khắc phục lỗi cổng và bảo mật Redis**.
    

Bạn đã có một hệ thống kiểm tra user hoàn chỉnh, có thể triển khai trên môi trường thực tế! 🚀