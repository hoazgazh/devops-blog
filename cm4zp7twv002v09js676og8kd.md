---
title: "Cách Docker hoạt động như thế nào trong OS?"
seoTitle: "Docker: Hoạt Động Trong Hệ Điều Hành"
seoDescription: "Khám phá cách Docker tương tác với hệ điều hành và quản lý tài nguyên như RAM thông qua các thành phần Linux và công nghệ container hóa"
datePublished: Sun Dec 22 2024 14:23:31 GMT+0000 (Coordinated Universal Time)
cuid: cm4zp7twv002v09js676og8kd
slug: cach-docker-hoat-dong-nhu-the-nao-trong-os
tags: docker

---

Để hiểu sâu hơn về cách Docker hoạt động ở mức dịch vụ và hạ tầng, cũng như cách nó tương tác với hệ điều hành (OS) và quản lý tài nguyên hệ thống như bộ nhớ RAM, chúng ta cần đi vào chi tiết kỹ thuật hơn về kiến trúc Docker, các thành phần hệ thống liên quan, và cách Docker sử dụng các tính năng lõi của Linux để thực hiện container hóa. Dưới đây là một cái nhìn chi tiết về cơ chế hoạt động của Docker từ hệ thống OS đến việc quản lý tài nguyên như RAM.

---

## **1\. Tổng Quan Về Kiến Trúc Docker**

### **1.1. Các Thành Phần Chính**

1. **Docker Client**
    
    * Giao diện người dùng để gửi lệnh (`docker` command).
        
    * Giao tiếp với Docker Daemon thông qua API REST.
        
2. **Docker Daemon (**`dockerd`)
    
    * Tiến trình nền quản lý toàn bộ hệ sinh thái Docker.
        
    * Quản lý Docker Images, Containers, Networks, và Volumes.
        
    * Giao tiếp với các thành phần runtime như `containerd`.
        
3. **Containerd**
    
    * Một container runtime cấp thấp, quản lý vòng đời của container.
        
    * Thực hiện các tác vụ như tạo, chạy, dừng, xóa container.
        
    * Giao tiếp với kernel thông qua `runc`.
        
4. **Runc**
    
    * Công cụ CLI để chạy container theo chuẩn [OCI (Open Container Initiative)](https://opencontainers.org/).
        
    * Thực thi container bằng cách tạo các tiến trình cách ly sử dụng namespaces và cgroups.
        
5. **Linux Kernel**
    
    * Cung cấp các tính năng hỗ trợ container hóa như namespaces, cgroups, và hệ thống file OverlayFS.
        
6. **Docker Registry**
    
    * Kho lưu trữ các Docker Images (ví dụ: Docker Hub).
        
    * Docker Daemon tải xuống và đẩy các image từ/đến registry.
        

### **1.2. Sơ Đồ Kiến Trúc Tổng Thể**

```bash
+-------------------+
|   Docker Client   |  <-- Gửi lệnh từ người dùng (docker run, docker build)
+-------------------+
          |
          v
+-------------------+
|   Docker Daemon   |  <-- dockerd
+-------------------+
          |
          v
+-------------------+
|    containerd     |  <-- Quản lý vòng đời container
+-------------------+
          |
          v
+-------------------+
|      runc         |  <-- Thực thi container thực tế
+-------------------+
          |
          v
+-------------------+
|  Linux Kernel     |  <-- Quản lý tài nguyên, cách ly (namespaces, cgroups)
+-------------------+
```

---

## **2\. Chi Tiết Về Tương Tác Giữa Các Thành Phần**

### **2.1. Docker Client Gửi Lệnh Đến Docker Daemon**

Khi bạn thực hiện một lệnh Docker (ví dụ: `docker run nginx`), Docker Client sẽ gửi yêu cầu này tới Docker Daemon thông qua API REST. Giao thức mặc định là UNIX socket (`/var/run/docker.sock`) trên Linux hoặc thông qua TCP trên các hệ điều hành khác.

**Ví dụ:**

```bash
docker run -d -p 8080:80 --name webserver nginx
```

### **2.2. Docker Daemon Xử Lý Lệnh**

Docker Daemon sẽ thực hiện các bước sau:

1. **Phân Tích Lệnh**: Xác định yêu cầu từ lệnh `docker run` (chạy container từ image `nginx`).
    
2. **Kiểm Tra Image**: Kiểm tra xem image `nginx` đã tồn tại trên hệ thống chưa.
    
3. **Tải Image (nếu cần)**: Nếu image chưa có, Docker Daemon sẽ tải nó từ Docker Registry (ví dụ: Docker Hub).
    
4. **Tạo Container**: Docker Daemon yêu cầu `containerd` tạo một container mới từ image `nginx`.
    
5. **Cấu Hình Mạng**: Thiết lập mạng cho container, ánh xạ cổng host `8080` tới cổng container `80`.
    
6. **Gọi containerd**: Gửi yêu cầu tới `containerd` để khởi chạy container.
    
7. **Quản Lý Tài Nguyên**: Docker Daemon thiết lập các giới hạn tài nguyên (CPU, Memory) thông qua cgroups.
    

### **2.3. containerd Quản Lý Vòng Đời Container**

`containerd` thực hiện các tác vụ sau khi nhận yêu cầu từ Docker Daemon:

1. **Tạo Container**: Tạo container mới từ image đã được Docker Daemon cung cấp.
    
2. **Quản Lý Tài Nguyên**: Sử dụng cgroups để cấp phát và giới hạn tài nguyên cho container.
    
3. **Thiết Lập Môi Trường Cách Ly**: Sử dụng namespaces để cách ly container với hệ thống host và các container khác.
    
4. **Gọi runc**: Gửi yêu cầu tới `runc` để thực thi container.
    

### **2.4. runc Thực Thi Container**

`runc` sẽ thực hiện các bước sau để khởi chạy container:

1. **Thiết Lập Namespaces**: Tạo các namespaces như PID, NET, IPC, UTS, và Mount cho container.
    
2. **Áp Dụng cgroups**: Sử dụng cgroups để áp đặt giới hạn tài nguyên cho container.
    
3. **Mount File System**: Sử dụng OverlayFS để hợp nhất các lớp của image thành một hệ thống file duy nhất cho container.
    
4. **Khởi Chạy Tiến Trình Chính**: Thực thi tiến trình chính của container (ví dụ: tiến trình Nginx).
    

### **2.5. Container Chạy và Hoạt Động**

* Container Nginx chạy độc lập trong môi trường cách ly.
    
* Người dùng có thể truy cập Nginx thông qua cổng `8080` trên host.
    
* Docker Daemon và `containerd` tiếp tục theo dõi và quản lý container.
    

---

## **3\. Các Tính Năng Lõi của Linux Hỗ Trợ Docker**

### **3.1. Namespaces**

Namespaces cung cấp sự cách ly tài nguyên hệ thống cho các container. Docker sử dụng nhiều loại namespaces để đảm bảo mỗi container hoạt động như một môi trường độc lập.

#### **Các Loại Namespaces Docker Sử Dụng:**

1. **PID Namespace**
    
    * **Chức năng**: Cách ly danh sách tiến trình. Mỗi container có bảng PID riêng, không nhìn thấy các tiến trình bên ngoài container.
        
    * **Chi Tiết**: Khi tạo một container mới, Docker Daemon sử dụng `clone` hệ thống để tạo PID namespace mới, bắt đầu một tiến trình chính (init) trong namespace này.
        
2. **NET Namespace**
    
    * **Chức năng**: Cách ly các giao diện mạng, cổng, và cấu hình mạng.
        
    * **Chi Tiết**: Docker thiết lập các interface mạng ảo (veth pairs) để kết nối container với mạng host hoặc các mạng Docker bridge khác.
        
3. **IPC Namespace**
    
    * **Chức năng**: Cách ly các thông điệp giữa các tiến trình (Inter-Process Communication).
        
    * **Chi Tiết**: Giúp cách ly các đối tượng IPC như shared memory và semaphores.
        
4. **UTS Namespace**
    
    * **Chức năng**: Cách ly hostname và domain name.
        
    * **Chi Tiết**: Mỗi container có thể có hostname riêng, không ảnh hưởng đến hệ thống host hoặc các container khác.
        
5. **Mount Namespace**
    
    * **Chức năng**: Cách ly hệ thống tập tin, đảm bảo các container chỉ nhìn thấy những gì được mount vào chúng.
        
    * **Chi Tiết**: Sử dụng các điểm mount riêng biệt để mỗi container có hệ thống file độc lập.
        
6. **User Namespace**
    
    * **Chức năng**: Cách ly quyền người dùng và nhóm, cho phép container chạy với quyền hạn thấp hơn trên hệ thống host.
        
    * **Chi Tiết**: Chuyển đổi UID và GID của tiến trình trong container, giúp tăng cường bảo mật bằng cách không yêu cầu quyền root trên host.
        

### **3.2. Control Groups (cgroups)**

cgroups quản lý và giới hạn tài nguyên mà mỗi container có thể sử dụng, bao gồm CPU, bộ nhớ, I/O, và mạng.

#### **Các Loại cgroups Docker Sử Dụng:**

1. **CPU cgroup**
    
    * **Chức năng**: Giới hạn phần trăm CPU mà container có thể sử dụng.
        
    * **Chi Tiết**: Docker đặt các giá trị như `cpu.cfs_quota_us` và `cpu.cfs_period_us` để kiểm soát lượng CPU.
        
2. **Memory cgroup**
    
    * **Chức năng**: Đặt giới hạn bộ nhớ (RAM) cho container.
        
    * **Chi Tiết**: Các tham số như `memory.limit_in_bytes` và `memory.memsw.limit_in_bytes` được sử dụng để kiểm soát lượng RAM và swap.
        
3. **Block I/O cgroup**
    
    * **Chức năng**: Điều khiển tốc độ đọc/ghi vào đĩa.
        
    * **Chi Tiết**: Sử dụng các tham số như `blkio.weight` và `blkio.throttle.read_bps_device` để giới hạn băng thông I/O.
        
4. **Network cgroup**
    
    * **Chức năng**: Quản lý băng thông mạng.
        
    * **Chi Tiết**: Thông qua các công cụ bổ trợ như tc (traffic control).
        

### **3.3. Union File System (OverlayFS)**

OverlayFS là một hệ thống file hợp nhất (union file system) cho phép Docker hợp nhất nhiều lớp (layers) của image thành một hệ thống file duy nhất cho container. Điều này giúp tiết kiệm dung lượng lưu trữ và tăng hiệu suất.

#### **Cấu Trúc của OverlayFS:**

* **Read-Only Layers**: Các lớp không ghi được của image giữ nguyên và có thể chia sẻ giữa nhiều container.
    
* **Write Layer**: Mỗi container có một lớp ghi riêng để lưu trữ các thay đổi.
    

#### **Ví Dụ:**

Khi bạn chạy một container từ image `nginx`, Docker sẽ:

1. Sử dụng các lớp read-only từ image `nginx`.
    
2. Tạo một lớp ghi mới cho container.
    
3. Hợp nhất các lớp này thành một hệ thống file duy nhất thông qua OverlayFS.
    

### **3.4. Namespaces và cgroups Làm Việc Như Thế Nào Cùng Nhau**

Namespaces và cgroups là hai tính năng chính của kernel Linux hỗ trợ container hóa. Namespaces cung cấp sự cách ly tài nguyên, trong khi cgroups quản lý và giới hạn tài nguyên.

#### **Mối Quan Hệ Giữa Namespaces và cgroups:**

* **Namespaces** đảm bảo rằng các container không can thiệp vào nhau hoặc vào hệ thống host.
    
* **cgroups** đảm bảo rằng mỗi container không vượt quá giới hạn tài nguyên được cấp.
    

Ví dụ, khi một container sử dụng một lượng CPU hoặc bộ nhớ nhất định, cgroups đảm bảo rằng container không vượt quá giới hạn này, trong khi namespaces đảm bảo rằng container không thể nhìn thấy hoặc ảnh hưởng đến các container khác.

---

## **4\. Quản Lý Bộ Nhớ (Memory Management) Trong Docker**

### **4.1. Giới Hạn Bộ Nhớ Với cgroups**

Docker sử dụng cgroups để giới hạn và quản lý bộ nhớ mà mỗi container có thể sử dụng. Điều này đảm bảo rằng một container không thể tiêu thụ toàn bộ bộ nhớ hệ thống, từ đó ngăn chặn tình trạng quá tải hệ thống.

#### **Các Tham Số cgroups liên quan đến Bộ Nhớ:**

1. **memory.limit\_in\_bytes**
    
    * **Chức năng**: Đặt giới hạn bộ nhớ cho container.
        
    * **Ví Dụ**:
        
        ```bash
        docker run -d --memory="512m" --name memtest nginx
        ```
        
2. **memory.reservation\_in\_bytes**
    
    * **Chức năng**: Đặt mức khuyến nghị về bộ nhớ mà container nên sử dụng. Nếu hệ thống có tài nguyên, container có thể sử dụng nhiều hơn giới hạn này.
        
    * **Ví Dụ**:
        
        ```bash
        docker run -d --memory-reservation="256m" --name memtest nginx
        ```
        
3. **memory.swap**
    
    * **Chức năng**: Đặt giới hạn bộ nhớ swap mà container có thể sử dụng.
        
    * **Ví Dụ**:
        
        ```bash
        docker run -d --memory="512m" --memory-swap="1g" --name memtest nginx
        ```
        

#### **Cách cgroups Quản Lý Bộ Nhớ:**

* **Giới Hạn Bộ Nhớ**: Khi container cố gắng sử dụng bộ nhớ vượt quá giới hạn đã đặt (`memory.limit_in_bytes`), kernel sẽ kích hoạt OOM (Out-Of-Memory) killer để chấm dứt tiến trình trong container.
    
* **Giới Hạn Swap**: `memory.swap` giới hạn lượng swap mà container có thể sử dụng. Nếu không đặt giá trị này, Docker mặc định sẽ cho phép swap không giới hạn.
    
* **Giới Hạn Tài Nguyên Khác**: Các tham số như `memory.soft_limit` (mức khuyến nghị) và `memory.swappiness` (điều khiển mức swap) cũng được sử dụng để quản lý bộ nhớ.
    

### **4.2. Quản Lý Bộ Nhớ Trong Kernel**

Kernel Linux quản lý bộ nhớ thông qua nhiều cơ chế như paging, caching, và swapping. Docker tận dụng các tính năng này để quản lý bộ nhớ cho các container.

#### **Các Cơ Chế Kernel liên quan đến Bộ Nhớ:**

1. **Paging**
    
    * **Chức năng**: Di chuyển các trang bộ nhớ giữa RAM và bộ nhớ phụ (swap) khi cần thiết.
        
    * **Docker**: Khi một container cố gắng sử dụng nhiều bộ nhớ, kernel sẽ quyết định trang bộ nhớ nào cần di chuyển vào swap để giải phóng RAM cho container.
        
2. **Caching**
    
    * **Chức năng**: Kernel sử dụng phần bộ nhớ không sử dụng để cache các file hệ thống để tăng hiệu suất.
        
    * **Docker**: Khi container sử dụng bộ nhớ, kernel có thể giải phóng bộ nhớ cache để cung cấp cho container.
        
3. **Swapping**
    
    * **Chức năng**: Di chuyển các trang bộ nhớ ít sử dụng từ RAM vào swap để giải phóng RAM cho các tiến trình quan trọng hơn.
        
    * **Docker**: Swapping có thể ảnh hưởng đến hiệu suất container. Giới hạn swap (`memory.swap`) giúp kiểm soát lượng swap mà container có thể sử dụng.
        

### **4.3. Phân Tích Sử Dụng Bộ Nhớ**

Docker cung cấp các công cụ để theo dõi và phân tích việc sử dụng bộ nhớ của container.

#### **Sử Dụng Lệnh** `docker stats`:

```bash
docker stats
```

**Kết Quả:**

```bash
CONTAINER ID   NAME                CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O         PIDS
d1e8f7e1c8a9   my-nginx-container  0.07%     50MiB / 512MiB        9.77%     1.2kB / 0B        0B / 0B           2
```

* **MEM USAGE / LIMIT**: Hiển thị lượng bộ nhớ đã sử dụng và giới hạn được đặt.
    
* **MEM %**: Tỷ lệ phần trăm bộ nhớ đã sử dụng so với giới hạn.
    

#### **Sử Dụng Công Cụ Theo Dõi Bộ Nhớ:**

* **cAdvisor**: Công cụ do Google phát triển để theo dõi tài nguyên sử dụng bởi container.
    
* **Prometheus & Grafana**: Tích hợp để thu thập và hiển thị dữ liệu tài nguyên sử dụng.
    

### **4.4. Rootless Mode và Quản Lý Bộ Nhớ**

Podman hỗ trợ chạy container trong chế độ rootless, giúp tăng cường bảo mật bằng cách không yêu cầu quyền root. Docker cũng hỗ trợ rootless mode nhưng với một số hạn chế.

#### **Cách Rootless Mode Ảnh Hưởng đến Quản Lý Bộ Nhớ:**

* **Quyền Hạn**: Container rootless không có quyền truy cập toàn bộ tài nguyên hệ thống, do đó việc quản lý bộ nhớ có thể bị hạn chế hơn.
    
* **Tính Bảo Mật**: Giảm nguy cơ container gây hại cho hệ thống host nếu có sự cố.
    
* **Hiệu Suất**: Có thể có một số chi phí về hiệu suất do thêm các lớp cách ly.
    

---

## **5\. Quản Lý CPU Trong Docker**

Giống như bộ nhớ, Docker cũng sử dụng cgroups để quản lý và giới hạn CPU cho các container.

### **5.1. Giới Hạn CPU Với cgroups**

#### **Các Tham Số cgroups liên quan đến CPU:**

1. **cpu.cfs\_period\_us**
    
    * **Chức năng**: Đặt thời gian khoảng giữa các lần phân phối CPU (CPU period).
        
    * **Giá Trị Mặc Định**: 100000 (100ms).
        
2. **cpu.cfs\_quota\_us**
    
    * **Chức năng**: Đặt tổng thời gian CPU mà container có thể sử dụng trong mỗi period.
        
    * **Ví Dụ**:
        
        ```bash
        docker run -d --cpus="1.5" --name cpudemo nginx
        ```
        
        Docker sẽ tính toán `cpu.cfs_quota_us` dựa trên `cpu.cfs_period_us`.
        
3. **cpu\_shares**
    
    * **Chức năng**: Đặt tỷ lệ ưu tiên CPU cho container.
        
    * **Giá Trị Mặc Định**: 1024.
        
    * **Ví Dụ**:
        
        ```bash
        docker run -d --cpu-shares=512 --name cpudemo nginx
        ```
        

### **5.2. Cách cgroups Quản Lý CPU**

* **Giới Hạn CPU**: `cpu.cfs_quota_us` / `cpu.cfs_period_us` xác định số lượng CPU mà container có thể sử dụng. Ví dụ, nếu `cpu.cfs_quota_us` là 150000 và `cpu.cfs_period_us` là 100000, container có thể sử dụng 1.5 CPU.
    
* **Chia Sẻ CPU**: `cpu_shares` xác định tỷ lệ ưu tiên CPU giữa các container. Container có `cpu_shares` cao hơn sẽ được ưu tiên hơn trong việc sử dụng CPU khi có cạnh tranh.
    

### **5.3. Phân Tích Sử Dụng CPU**

Docker cung cấp các công cụ để theo dõi việc sử dụng CPU của container.

#### **Sử Dụng Lệnh** `docker stats`:

```bash
docker stats
```

**Kết Quả:**

```bash
CONTAINER ID   NAME                CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O         PIDS
d1e8f7e1c8a9   my-nginx-container  0.07%     50MiB / 512MiB        9.77%     1.2kB / 0B        0B / 0B           2
```

* **CPU %**: Tỷ lệ phần trăm CPU đã sử dụng bởi container.
    

#### **Sử Dụng Công Cụ Theo Dõi CPU:**

* **cAdvisor**: Theo dõi việc sử dụng CPU theo thời gian thực.
    
* **Prometheus & Grafana**: Thu thập và hiển thị dữ liệu CPU sử dụng cho các container.
    

---

## **6\. Quản Lý I/O và Storage Trong Docker**

### **6.1. Quản Lý I/O Với cgroups**

Docker sử dụng cgroups để giới hạn I/O cho container, đảm bảo rằng một container không thể chiếm dụng toàn bộ băng thông I/O của hệ thống.

#### **Các Tham Số cgroups liên quan đến I/O:**

1. **blkio.weight**
    
    * **Chức năng**: Đặt trọng số I/O cho container, xác định tỷ lệ băng thông I/O mà container có thể sử dụng so với các container khác.
        
    * **Giá Trị Mặc Định**: 500.
        
    * **Ví Dụ**:
        
        ```bash
        docker run -d --blkio-weight=300 --name iodemo nginx
        ```
        
2. **blkio.throttle.read\_bps\_device**
    
    * **Chức năng**: Giới hạn tốc độ đọc dữ liệu từ thiết bị.
        
    * **Ví Dụ**:
        
        ```bash
        docker run -d --device-read-bps /dev/sda:1mb --name iodemo nginx
        ```
        
3. **blkio.throttle.write\_bps\_device**
    
    * **Chức năng**: Giới hạn tốc độ ghi dữ liệu vào thiết bị.
        
    * **Ví Dụ**:
        
        ```bash
        docker run -d --device-write-bps /dev/sda:1mb --name iodemo nginx
        ```
        

### **6.2. Quản Lý Storage Với Docker**

Docker sử dụng các driver lưu trữ khác nhau để quản lý hệ thống file của container. Các driver này bao gồm OverlayFS, AUFS, Btrfs, và ZFS.

#### **OverlayFS**

* **Chức năng**: Hợp nhất nhiều lớp (layers) của image thành một hệ thống file duy nhất cho container.
    
* **Ưu Điểm**: Hiệu suất cao, nhẹ, hỗ trợ tốt cho việc chia sẻ layers giữa các container.
    

#### **Cấu Trúc OverlayFS:**

* **Lowerdir**: Các lớp read-only từ image.
    
* **Upperdir**: Lớp ghi (write layer) riêng cho container.
    
* **Merged**: Hệ thống file hợp nhất mà container thấy.
    

**Ví Dụ:**

Khi bạn chạy một container từ image `nginx`, Docker sẽ:

1. Mount các lớp read-only từ image `nginx` vào `Lowerdir`.
    
2. Tạo và mount một `Upperdir` mới cho container.
    
3. Sử dụng OverlayFS để hợp nhất `Lowerdir` và `Upperdir` thành `Merged`, nơi container sẽ thấy hệ thống file.
    

### **6.3. Quản Lý Volume và Bind Mounts**

* **Volumes**: Là các đối tượng được Docker quản lý để lưu trữ dữ liệu. Volumes được lưu trữ bên ngoài hệ thống file của container và có thể được chia sẻ giữa các container.
    
    **Ví Dụ:**
    
    ```bash
    docker run -d -v my_volume:/data --name volumedemo nginx
    ```
    
* **Bind Mounts**: Cho phép mount một thư mục từ hệ thống host vào container.
    
    **Ví Dụ:**
    
    ```bash
    docker run -d -v /host/path:/container/path --name binddemo nginx
    ```
    

### **6.4. Quản Lý I/O và Storage Hiệu Quả**

Docker tối ưu hóa việc quản lý I/O và storage thông qua các hệ thống file hợp nhất và caching. OverlayFS cho phép nhiều container chia sẻ các lớp read-only của image, giảm dung lượng lưu trữ và tăng hiệu suất khi khởi động container mới từ cùng một image.

---

## **7\. Quản Lý Mạng Trong Docker**

### **7.1. Cấu Hình Mạng Với Namespaces**

Docker sử dụng NET namespaces để cách ly mạng cho mỗi container. Mỗi container có thể có một giao diện mạng riêng, không nhìn thấy các giao diện mạng của container khác hoặc của hệ thống host.

### **7.2. Docker Network Drivers**

Docker hỗ trợ nhiều driver mạng khác nhau, mỗi driver có những đặc điểm và ứng dụng riêng:

1. **bridge**
    
    * **Chức năng**: Tạo một mạng ảo bridge cho các container trên cùng một host.
        
    * **Sử Dụng**: Mặc định cho các container không thuộc mạng nào khác.
        
2. **host**
    
    * **Chức năng**: Cho phép container chia sẻ mạng với hệ thống host.
        
    * **Sử Dụng**: Khi container cần hiệu suất mạng cao hoặc muốn truy cập trực tiếp các dịch vụ mạng trên host.
        
3. **overlay**
    
    * **Chức năng**: Kết nối các container trên nhiều host khác nhau, thường được sử dụng trong môi trường orchestrated như Docker Swarm hoặc Kubernetes.
        
    * **Sử Dụng**: Khi cần tạo mạng ảo xuyên nhiều host để container có thể giao tiếp.
        
4. **macvlan**
    
    * **Chức năng**: Tạo các giao diện mạng ảo riêng biệt với địa chỉ MAC độc lập.
        
    * **Sử Dụng**: Khi container cần xuất hiện như một thiết bị mạng độc lập trên mạng vật lý.
        
5. **none**
    
    * **Chức năng**: Container không có mạng.
        
    * **Sử Dụng**: Khi không cần kết nối mạng cho container.
        

### **7.3. Cách Docker Quản Lý Mạng**

#### **Tạo Mạng Mới:**

```bash
docker network create my_network
```

#### **Kết Nối Container với Mạng:**

```bash
docker run -d --network=my_network --name container1 nginx
docker run -d --network=my_network --name container2 nginx
```

* Các container trong cùng một mạng có thể giao tiếp với nhau thông qua tên container như hostname.
    

#### **DNS và Giao Tiếp Nội Bộ:**

Docker cung cấp một DNS internal để container có thể phân giải tên container thành địa chỉ IP nội bộ, giúp việc giao tiếp giữa các container dễ dàng hơn.

---

## **8\. Quản Lý Tài Nguyên CPU và Memory Thông Qua cgroups**

### **8.1. cgroups v1 vs cgroups v2**

* **cgroups v1**: Hỗ trợ phân chia tài nguyên thành các hierarchy độc lập, cho phép quản lý từng loại tài nguyên riêng biệt.
    
* **cgroups v2**: Hỗ trợ unified hierarchy, cho phép quản lý tất cả tài nguyên thông qua một hierarchy duy nhất, cải thiện tính nhất quán và đơn giản hóa việc quản lý tài nguyên.
    

Docker có thể hỗ trợ cả hai phiên bản cgroups, nhưng cgroups v2 đang dần được ưu tiên hơn vì các cải tiến về tính nhất quán và hiệu suất.

### **8.2. Giới Hạn Bộ Nhớ với cgroups**

#### **Thiết Lập Giới Hạn Bộ Nhớ:**

Docker sử dụng các tham số như `--memory` để giới hạn bộ nhớ của container.

**Ví Dụ:**

```bash
docker run -d --memory="256m" --name memdemo nginx
```

#### **Cách cgroups v1 Quản Lý Bộ Nhớ:**

* **memory.limit\_in\_bytes**: Giới hạn bộ nhớ tối đa mà container có thể sử dụng.
    
* **memory.memsw.limit\_in\_bytes**: Giới hạn bộ nhớ và swap tối đa mà container có thể sử dụng.
    

#### **Cách cgroups v2 Quản Lý Bộ Nhớ:**

* **memory.max**: Giới hạn bộ nhớ tối đa.
    
* **memory.swap.max**: Giới hạn swap tối đa.
    

#### **Ví Dụ: Kiểm Tra Giới Hạn Bộ Nhớ**

```bash
# Lấy PID của container
PID=$(docker inspect --format '{{.State.Pid}}' memdemo)

# Kiểm tra giới hạn bộ nhớ trong cgroups
cat /sys/fs/cgroup/memory/docker/$PID/memory.limit_in_bytes
```

### **8.3. Giới Hạn CPU với cgroups**

#### **Thiết Lập Giới Hạn CPU:**

Docker sử dụng các tham số như `--cpus` hoặc `--cpu-shares` để giới hạn CPU.

**Ví Dụ:**

```bash
docker run -d --cpus="1.5" --name cpudemo nginx
```

#### **Cách cgroups v1 Quản Lý CPU:**

* **cpu.cfs\_period\_us**: Đặt khoảng thời gian cho việc phân phối CPU.
    
* **cpu.cfs\_quota\_us**: Giới hạn tổng thời gian CPU mà container có thể sử dụng trong mỗi period.
    
* **cpu.shares**: Đặt tỷ lệ ưu tiên CPU giữa các container.
    

#### **Cách cgroups v2 Quản Lý CPU:**

* **cpu.max**: Giới hạn tối đa CPU mà container có thể sử dụng.
    

#### **Ví Dụ: Kiểm Tra Giới Hạn CPU**

```bash
# Lấy PID của container
PID=$(docker inspect --format '{{.State.Pid}}' cpudemo)

# Kiểm tra giới hạn CPU trong cgroups
cat /sys/fs/cgroup/cpu/docker/$PID/cpu.cfs_quota_us
cat /sys/fs/cgroup/cpu/docker/$PID/cpu.cfs_period_us
```

---

## **9\. Quản Lý Network Interface và Virtual Ethernet (veth)**

### **9.1. Tạo Các veth Pairs**

Docker sử dụng các cặp veth (virtual Ethernet) để kết nối container với mạng host hoặc bridge network.

#### **Cách Tạo veth Pair:**

1. **veth Pair**: Một cặp veth bao gồm hai interface mạng ảo, một bên nằm trong host và một bên trong container.
    
2. **Bridge Network**: Docker tạo một bridge network (ví dụ: `docker0`) trên host, kết nối các container thông qua các cặp veth.
    

#### **Ví Dụ:**

Khi chạy một container, Docker sẽ:

1. Tạo một cặp veth mới (`vethXYZ`).
    
2. Gắn một đầu của cặp veth vào namespace mạng của container.
    
3. Gắn đầu còn lại vào bridge network (`docker0`).
    
4. Cấu hình địa chỉ IP và các thông số mạng khác.
    

### **9.2. Cấu Hình Mạng Cho Container**

Docker cung cấp các tùy chọn để cấu hình mạng cho container, bao gồm:

* **Port Mapping**: Ánh xạ cổng từ host sang container.
    
    ```bash
    docker run -d -p 8080:80 --name webserver nginx
    ```
    
* **Network Aliases**: Đặt bí danh cho container trong mạng để dễ dàng truy cập.
    
    ```bash
    docker run -d --network=my_network --network-alias=myalias --name webserver nginx
    ```
    
* **Custom Networks**: Tạo các mạng tùy chỉnh với cấu hình riêng.
    
    ```bash
    docker network create --driver bridge my_network
    ```
    

### **9.3. DNS Resolution Trong Docker**

Docker cung cấp dịch vụ DNS nội bộ để container có thể phân giải tên container khác trong cùng một mạng.

#### **Cách Hoạt Động:**

* Docker DNS server hoạt động như một DNS resolver cho các container trong cùng một mạng.
    
* Khi một container cố gắng kết nối đến tên container khác, Docker DNS server sẽ phân giải tên đó thành địa chỉ IP nội bộ của container đích.
    

#### **Ví Dụ:**

Giả sử bạn có hai container `web` và `db` trong cùng một mạng `my_network`:

```bash
docker run -d --network=my_network --name web nginx
docker run -d --network=my_network --name db mysql
```

Trong container `web`, bạn có thể kết nối đến `db` thông qua hostname `db`:

```bash
ping db
```

---

## **10\. Quản Lý Storage Với Docker**

### **10.1. Docker Storage Drivers**

Docker sử dụng các storage drivers để quản lý hệ thống file của container. Các driver phổ biến bao gồm:

1. **OverlayFS (overlay2)**
    
    * **Ưu Điểm**: Hiệu suất cao, hỗ trợ nhiều lớp (layers).
        
    * **Sử Dụng**: Được khuyến nghị cho hầu hết các trường hợp.
        
2. **AUFS**
    
    * **Ưu Điểm**: Hỗ trợ nhiều lớp, tương thích tốt với các phiên bản Docker cũ.
        
    * **Nhược Điểm**: Ít hiệu suất hơn OverlayFS.
        
3. **Btrfs**
    
    * **Ưu Điểm**: Hỗ trợ snapshot, kiểm tra tính toàn vẹn của dữ liệu.
        
    * **Nhược Điểm**: Đòi hỏi hệ thống file Btrfs trên host.
        
4. **ZFS**
    
    * **Ưu Điểm**: Hiệu suất cao, hỗ trợ snapshot và clone.
        
    * **Nhược Điểm**: Cài đặt phức tạp, đòi hỏi hệ thống file ZFS.
        

### **10.2. OverlayFS Chi Tiết**

OverlayFS hợp nhất nhiều lớp hệ thống file thành một hệ thống file duy nhất cho container.

#### **Cấu Trúc OverlayFS:**

* **Lowerdir**: Các lớp read-only từ image.
    
* **Upperdir**: Lớp ghi (write layer) riêng cho container.
    
* **Merged**: Hệ thống file hợp nhất mà container thấy.
    

#### **Chi Tiết Hoạt Động:**

1. **Mount Lowerdir và Upperdir**: Docker Daemon mount các lớp read-only và lớp ghi vào OverlayFS.
    
2. **Union Layering**: OverlayFS hợp nhất các lớp này thành một hệ thống file duy nhất.
    
3. **Write Layer**: Các thay đổi từ container được ghi vào lớp ghi, không ảnh hưởng đến các lớp read-only.
    

### **10.3. Quản Lý Volume và Bind Mounts**

* **Volumes**
    
    * **Đặc Điểm**: Được Docker quản lý, lưu trữ bên ngoài hệ thống file của container.
        
    * **Ưu Điểm**: Dễ dàng chia sẻ giữa các container, quản lý dễ dàng.
        
    * **Cách Sử Dụng**:
        
        ```bash
        docker run -d -v my_volume:/data --name volumedemo nginx
        ```
        
* **Bind Mounts**
    
    * **Đặc Điểm**: Gắn thư mục từ hệ thống host vào container.
        
    * **Ưu Điểm**: Linh hoạt, dễ dàng chia sẻ dữ liệu với host.
        
    * **Cách Sử Dụng**:
        
        ```bash
        docker run -d -v /host/path:/container/path --name binddemo nginx
        ```
        

### **10.4. Quản Lý Image Với Docker**

Docker sử dụng các registry để lưu trữ và phân phối các Docker Images. Docker Daemon quản lý việc tải, lưu trữ và đẩy các image từ/đến registry.

#### **Các Hoạt Động Chính:**

1. **Pulling Images**: Tải image từ registry về host.
    
    ```bash
    docker pull nginx:latest
    ```
    
2. **Pushing Images**: Đẩy image từ host lên registry.
    
    ```bash
    docker push myrepo/myimage:tag
    ```
    
3. **Building Images**: Tạo image từ Dockerfile.
    
    ```bash
    docker build -t myimage:tag .
    ```
    

### **10.5. Layer Caching và Reusability**

Docker tối ưu hóa việc lưu trữ và sử dụng lại các layers để giảm dung lượng lưu trữ và tăng tốc độ xây dựng container.

#### **Cách Hoạt Động:**

* **Layer Caching**: Khi Docker build một image, nó lưu lại các layers đã tạo. Nếu các lệnh trong Dockerfile không thay đổi, Docker sẽ sử dụng lại các layers từ cache thay vì tạo lại.
    
* **Layer Reusability**: Các layers read-only có thể được chia sẻ giữa nhiều container, giảm lượng dữ liệu lưu trữ cần thiết.
    

---

## **11\. Quản Lý Tài Nguyên Tổng Thể Trong Docker**

### **11.1. Resource Allocation và Scheduling**

Docker quản lý việc phân bổ tài nguyên cho các container thông qua cgroups và các driver scheduler. Việc này đảm bảo rằng mỗi container có đủ tài nguyên để hoạt động hiệu quả mà không làm ảnh hưởng đến các container khác.

#### **Các Cơ Chế Chính:**

1. **cgroups**: Quản lý giới hạn và ưu tiên tài nguyên CPU, memory, I/O cho từng container.
    
2. **namespaces**: Cách ly tài nguyên hệ thống để mỗi container hoạt động độc lập.
    
3. **Scheduling**: Docker quyết định cách phân bổ tài nguyên dựa trên các giới hạn và ưu tiên đã thiết lập.
    

### **11.2. Quản Lý Tài Nguyên Với Docker Swarm và Kubernetes**

Khi sử dụng Docker trong môi trường orchestrated như Docker Swarm hoặc Kubernetes, việc quản lý tài nguyên trở nên phức tạp hơn với các tính năng bổ sung.

#### **Docker Swarm:**

* **Service Definitions**: Định nghĩa các service với giới hạn tài nguyên.
    
* **Scheduling**: Swarm scheduler quyết định placement của các container dựa trên tài nguyên sẵn có.
    

**Ví Dụ:**

```bash
docker service create --name web --replicas=3 --limit-cpu=0.5 --limit-memory=512M nginx
```

#### **Kubernetes:**

* **Resource Requests and Limits**: Định nghĩa yêu cầu và giới hạn tài nguyên cho các Pod.
    
* **Scheduler**: Kubernetes scheduler quyết định placement của Pod dựa trên tài nguyên và yêu cầu.
    

**Ví Dụ:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        memory: "256Mi"
        cpu: "250m"
      limits:
        memory: "512Mi"
        cpu: "500m"
```

---

## **12\. Quản Lý Networking và Security Trong Docker**

### **12.1. Docker Security Models**

Docker áp dụng các mô hình bảo mật để đảm bảo rằng các container không thể can thiệp vào nhau hoặc vào hệ thống host.

#### **Các Tính Năng Bảo Mật Chính:**

1. **User Namespaces**
    
    * Cách ly quyền người dùng giữa container và host.
        
    * Cho phép container chạy với người dùng không có quyền root trên host.
        
2. **AppArmor và SELinux**
    
    * Sử dụng các profiles bảo mật để hạn chế quyền truy cập của container vào hệ thống host.
        
    * **AppArmor**: Sử dụng profiles để kiểm soát quyền truy cập file và tài nguyên.
        
    * **SELinux**: Sử dụng labels và policies để kiểm soát quyền truy cập.
        
3. **Seccomp Profiles**
    
    * Giới hạn các syscall mà container có thể sử dụng.
        
    * Giảm nguy cơ tấn công từ các lỗ hổng syscall.
        
4. **Capabilities**
    
    * Giới hạn các quyền đặc biệt mà container có thể có.
        
    * Docker mặc định loại bỏ một số capabilities để tăng cường bảo mật.
        

### **12.2. Network Security**

Docker cung cấp các tính năng bảo mật mạng để kiểm soát giao tiếp giữa các container và với mạng bên ngoài.

#### **Các Tính Năng:**

1. **Network Policies**
    
    * Định nghĩa các quy tắc để kiểm soát lưu lượng mạng giữa các container.
        
    * Sử dụng các công cụ như Calico hoặc Cilium để quản lý network policies trong Kubernetes.
        
2. **Encrypted Networks**
    
    * Mã hóa lưu lượng mạng giữa các container để bảo vệ dữ liệu trong transit.
        
    * Sử dụng VPN hoặc các công cụ như Weave Net để mã hóa mạng.
        
3. **Service Meshes**
    
    * Sử dụng các service mesh như Istio hoặc Linkerd để quản lý và bảo mật giao tiếp giữa các dịch vụ.
        

### **12.3. Docker Security Best Practices**

* **Sử Dụng Users Namespaces**: Đảm bảo container không chạy với quyền root trên host.
    
* **Hạn Chế Capabilities**: Giảm các quyền đặc biệt cho container.
    
* **Sử Dụng Seccomp Profiles**: Giới hạn các syscall mà container có thể sử dụng.
    
* **Sử Dụng AppArmor/SELinux**: Áp dụng các profiles bảo mật để kiểm soát quyền truy cập.
    
* **Cập Nhật và Patch**: Luôn cập nhật Docker và các components liên quan để bảo vệ chống lại các lỗ hổng bảo mật mới.
    

---

## **13\. Quản Lý Log và Monitoring Trong Docker**

### **13.1. Log Drivers**

Docker hỗ trợ nhiều log drivers khác nhau để thu thập và lưu trữ log từ các container.

#### **Các Loại Log Drivers:**

1. **json-file** (Mặc định)
    
    * Lưu trữ log dưới dạng file JSON trên hệ thống host.
        
2. **syslog**
    
    * Gửi log đến hệ thống syslog của host hoặc một syslog server từ xa.
        
3. **journald**
    
    * Gửi log đến systemd journal.
        
4. **gelf**
    
    * Gửi log đến Graylog Extended Log Format (GELF) endpoint như Graylog hoặc Logstash.
        
5. **fluentd**
    
    * Gửi log đến Fluentd daemon.
        
6. **awslogs**
    
    * Gửi log đến Amazon CloudWatch Logs.
        
7. **splunk**
    
    * Gửi log đến Splunk.
        

#### **Cách Cấu Hình Log Driver:**

```bash
docker run -d --log-driver=syslog --name logdemo nginx
```

### **13.2. Log Aggregation và Centralization**

Để quản lý log hiệu quả, bạn có thể sử dụng các công cụ log aggregation như ELK Stack (Elasticsearch, Logstash, Kibana), Fluentd, hoặc Graylog.

#### **Ví Dụ: Gửi Log đến Fluentd**

```bash
docker run -d --log-driver=fluentd --log-opt fluentd-address=localhost:24224 --name logdemo nginx
```

### **13.3. Monitoring với cAdvisor và Prometheus**

#### **cAdvisor**

* **Chức năng**: Theo dõi và báo cáo tài nguyên sử dụng (CPU, memory, network, disk) cho các container.
    
* **Cách Sử Dụng**:
    
    ```bash
    docker run -d --name cadvisor -p 8080:8080 google/cadvisor
    ```
    

#### **Prometheus và Grafana**

* **Chức năng**: Thu thập và hiển thị dữ liệu monitoring từ cAdvisor hoặc các exporters khác.
    
* **Cách Cài Đặt**:
    
    * **Prometheus**: Cấu hình scrape targets để lấy dữ liệu từ cAdvisor.
        
    * **Grafana**: Tạo dashboards để hiển thị dữ liệu từ Prometheus.
        

---

## **14\. Quản Lý Tài Nguyên Bộ Nhớ RAM Trong Hệ Thống**

### **14.1. Bộ Nhớ RAM và Swap**

Docker sử dụng cgroups để quản lý bộ nhớ RAM và swap cho các container.

#### **Bộ Nhớ RAM:**

* **Giới Hạn RAM**: Docker có thể giới hạn lượng RAM mà mỗi container có thể sử dụng.
    
* **QoS (Quality of Service)**: Bằng cách đặt giới hạn RAM, Docker đảm bảo rằng không một container nào có thể tiêu thụ quá nhiều RAM và ảnh hưởng đến các container khác hoặc hệ thống host.
    

#### **Swap:**

* **Giới Hạn Swap**: Docker cho phép đặt giới hạn swap cho container để kiểm soát lượng dữ liệu được trang hóa từ RAM sang swap.
    
* **Thiết Lập Swap**:
    
    ```bash
    docker run -d --memory="512m" --memory-swap="1g" --name swapdemo nginx
    ```
    

### **14.2. Tối Ưu Bộ Nhớ Với cgroups và OOM Killer**

Khi một container cố gắng sử dụng bộ nhớ vượt quá giới hạn, cgroups sẽ kích hoạt OOM (Out-Of-Memory) killer để chấm dứt tiến trình trong container.

#### **Cách Docker Xử Lý OOM:**

1. **Giới Hạn Bộ Nhớ**: Docker đặt `memory.limit_in_bytes` trong cgroups.
    
2. **Theo Dõi Bộ Nhớ**: Kernel theo dõi việc sử dụng bộ nhớ của container.
    
3. **Kích Hoạt OOM Killer**: Khi container vượt quá giới hạn, OOM killer sẽ chọn chấm dứt tiến trình trong container để giải phóng bộ nhớ.
    
4. **Log và Thông Báo**: Docker sẽ ghi log về sự kiện OOM và có thể thông báo cho người dùng hoặc hệ thống giám sát.
    

### **14.3. Tối Ưu Hoạt Động Bộ Nhớ Host**

Để tối ưu hóa việc sử dụng bộ nhớ trên host, Docker và các container cần được cấu hình hợp lý.

#### **Các Chiến Lược Tối Ưu:**

1. **Giới Hạn Tài Nguyên Cho Mỗi Container**: Đặt giới hạn RAM và CPU để tránh tình trạng container nào có thể tiêu thụ toàn bộ tài nguyên.
    
2. **Sử Dụng Volumes và Bind Mounts Đúng Cách**: Giảm tải hệ thống file của container bằng cách sử dụng volumes và bind mounts.
    
3. **Tối Ưu Image Docker**: Sử dụng các image nhẹ hơn, tối giản các layer để giảm lượng bộ nhớ tiêu thụ.
    
4. **Giám Sát Tài Nguyên**: Sử dụng các công cụ như Prometheus và Grafana để theo dõi việc sử dụng tài nguyên và điều chỉnh cấu hình khi cần thiết.
    

---

## **15\. Thực Hiện Một Sự Kiện Chi Tiết: Xử Lý Bộ Nhớ Khi Container Chạy**

Hãy đi qua một ví dụ chi tiết về cách Docker quản lý bộ nhớ khi một container đang chạy.

### **15.1. Chạy Container với Giới Hạn Bộ Nhớ**

```bash
docker run -d --memory="256m" --memory-swap="512m" --name memtest nginx
```

### **15.2. Xem Giới Hạn Bộ Nhớ Trong cgroups**

```bash
# Lấy PID của container
PID=$(docker inspect --format '{{.State.Pid}}' memtest)

# Kiểm tra giới hạn bộ nhớ trong cgroups v1
cat /sys/fs/cgroup/memory/docker/$PID/memory.limit_in_bytes
cat /sys/fs/cgroup/memory/docker/$PID/memory.memsw.limit_in_bytes

# Hoặc trong cgroups v2
cat /sys/fs/cgroup/memory/docker/$PID/memory.max
```

### **15.3. Giám Sát Sử Dụng Bộ Nhớ**

Sử dụng lệnh `docker stats` để theo dõi việc sử dụng bộ nhớ:

```bash
docker stats memtest
```

**Kết Quả:**

```bash
CONTAINER ID   NAME     CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O         PIDS
d1e8f7e1c8a9   memtest  0.07%     50MiB / 256MiB        19.53%    1.2kB / 0B        0B / 0B           2
```

### **15.4. Kích Hoạt OOM Killer Khi Vượt Giới Hạn**

Nếu container cố gắng sử dụng bộ nhớ vượt quá giới hạn:

1. **Container cố gắng tăng bộ nhớ**: Ví dụ, chạy một ứng dụng tiêu thụ bộ nhớ lớn trong container.
    
2. **Kernel phát hiện vượt giới hạn**: cgroups nhận thấy container đã vượt quá `memory.limit_in_bytes`.
    
3. **Kích Hoạt OOM Killer**: Kernel sẽ chọn một tiến trình trong container để chấm dứt, thường là tiến trình tiêu thụ nhiều bộ nhớ nhất.
    
4. **Log sự kiện OOM**: Docker sẽ ghi log sự kiện OOM và có thể thông báo cho hệ thống giám sát.
    

#### **Kiểm Tra Log OOM:**

```bash
docker logs memtest
```

**Kết Quả có thể thấy thông báo về tiến trình bị chấm dứt do OOM.**

### **15.5. Phân Tích Sự Kiện OOM**

#### **Xem Chi Tiết cgroups và OOM Killer:**

```bash
# Xem sự kiện OOM trong log kernel
dmesg | grep -i 'out of memory'
```

**Ví Dụ Kết Quả:**

```bash
[12345.678901] Out of memory: Kill process 1234 (nginx) score 10 or sacrifice child
[12345.678902] Killed process 1234 (nginx) total-vm:1048576kB, anon-rss:524288kB, file-rss:0kB
```

---

## **16\. Deep Dive Vào Kernel Features: Namespaces và cgroups**

### **16.1. Namespaces Chi Tiết**

#### **PID Namespace**

* **Cách Thức Hoạt Động**:
    
    * Tạo một không gian tên riêng cho các tiến trình của container.
        
    * Các tiến trình trong container không thể nhìn thấy các tiến trình bên ngoài namespace.
        
* **Các Lệnh Liên Quan**:
    
    * `clone()` hệ thống call với flag `CLONE_NEWPID` để tạo PID namespace mới.
        
    * `setns()` để tham gia vào namespace đã tạo.
        

#### **NET Namespace**

* **Cách Thức Hoạt Động**:
    
    * Tạo một không gian tên riêng cho các giao diện mạng.
        
    * Container có thể có giao diện mạng riêng, không chia sẻ với host hoặc các container khác.
        
* **Các Lệnh Liên Quan**:
    
    * `clone()` hệ thống call với flag `CLONE_NEWNET` để tạo NET namespace mới.
        
    * `ip` command để cấu hình mạng trong container.
        

#### **IPC Namespace**

* **Cách Thức Hoạt Động**:
    
    * Cách ly các thông điệp giữa các tiến trình (shared memory, semaphores).
        
    * Container không thể giao tiếp IPC với host hoặc các container khác.
        

#### **UTS Namespace**

* **Cách Thức Hoạt Động**:
    
    * Cách ly hostname và domain name.
        
    * Container có thể đặt hostname riêng, không ảnh hưởng đến host.
        

#### **Mount Namespace**

* **Cách Thức Hoạt Động**:
    
    * Cách ly hệ thống file, cho phép container có hệ thống file riêng biệt.
        
    * Sử dụng `pivot_root` hoặc `chroot` để thiết lập root filesystem cho container.
        

#### **User Namespace**

* **Cách Thức Hoạt Động**:
    
    * Cách ly UID và GID giữa host và container.
        
    * Cho phép container chạy với người dùng không phải root trên host, tăng cường bảo mật.
        

### **16.2. Control Groups (cgroups) Chi Tiết**

#### **cgroups v1 và v2**

* **cgroups v1**: Phân chia tài nguyên thành các hierarchy riêng biệt cho từng loại tài nguyên.
    
* **cgroups v2**: Unified hierarchy, quản lý tất cả tài nguyên thông qua một hierarchy duy nhất.
    

#### **Thiết Lập cgroups**

* **cgroups v1**:
    
    * Tạo các thư mục trong `/sys/fs/cgroup` cho từng tài nguyên (cpu, memory, blkio, etc.).
        
    * Ghi giá trị vào các files để đặt giới hạn tài nguyên.
        
* **cgroups v2**:
    
    * Unified hierarchy, quản lý tất cả tài nguyên trong một filesystem `/sys/fs/cgroup`.
        
    * Sử dụng các files như `memory.max`, `cpu.max` để đặt giới hạn.
        

#### **API và Interface**

* Docker và containerd sử dụng các API của cgroups để quản lý tài nguyên.
    
* Các thư viện như `libcgroup` hoặc `systemd` APIs được sử dụng để tương tác với cgroups.
    

### **16.3. Runc và Libcontainer**

* **libcontainer**: Thư viện cung cấp các chức năng để tạo và quản lý container.
    
* **runc**: Công cụ CLI sử dụng libcontainer để thực thi container theo chuẩn OCI.
    

#### **Cách Runc Thực Thi Container:**

1. **Chạy Container**:
    
    ```bash
    runc run <container_id>
    ```
    
2. **Thiết Lập Namespaces và cgroups**:
    
    * Runc tạo các namespaces mới cho container.
        
    * Runc áp dụng các giới hạn tài nguyên thông qua cgroups.
        
3. **Mount OverlayFS**:
    
    * Runc sử dụng OverlayFS để hợp nhất các lớp hệ thống file của image thành hệ thống file của container.
        
4. **Khởi Chạy Tiến Trình Chính**:
    
    * Runc khởi chạy tiến trình chính của container trong môi trường cách ly.
        

---

## **17\. Phân Tích Memory Management Khi Container Chạy**

### **17.1. Phân Bổ Bộ Nhớ**

Khi một container chạy, Docker đặt các giới hạn bộ nhớ thông qua cgroups. Kernel Linux quản lý việc phân bổ bộ nhớ giữa các container và hệ thống host.

#### **Quy Trình Phân Bổ Bộ Nhớ:**

1. **Gửi Giới Hạn Bộ Nhớ**:
    
    * Docker Daemon thiết lập các tham số cgroups cho container, ví dụ:
        
        ```bash
        docker run -d --memory="256m" --name memtest nginx
        ```
        
    * Điều này tạo file `memory.limit_in_bytes` trong cgroups v1 hoặc `memory.max` trong cgroups v2.
        
2. **Kernel Quản Lý Bộ Nhớ**:
    
    * Kernel kiểm tra việc sử dụng bộ nhớ của container.
        
    * Nếu container cố gắng vượt quá giới hạn, OOM killer sẽ được kích hoạt để chấm dứt tiến trình trong container.
        
3. **Thu Thập và Giám Sát**:
    
    * Docker và các công cụ giám sát thu thập thông tin về việc sử dụng bộ nhớ để báo cáo cho người dùng.
        

### **17.2. Giải Pháp Giảm Sự Cố Quá Tải Bộ Nhớ**

#### **Giới Hạn và Reservation**

* **Giới Hạn (Limit)**: Đặt giới hạn tối đa bộ nhớ mà container có thể sử dụng.
    
* **Reservation**: Đặt mức khuyến nghị bộ nhớ mà container nên sử dụng nếu có thể.
    

#### **Tối Ưu Ứng Dụng**

* **Tối Giản Sử Dụng Bộ Nhớ**: Sử dụng các ứng dụng và services nhẹ hơn để giảm tiêu thụ bộ nhớ.
    
* **Caching và Garbage Collection**: Tối ưu hóa việc quản lý bộ nhớ trong ứng dụng để tránh rò rỉ bộ nhớ.
    

#### **Sử Dụng Swap Một Cách Thông Minh**

* **Giới Hạn Swap**: Đặt giới hạn swap để ngăn chặn container sử dụng quá nhiều swap, ảnh hưởng đến hiệu suất.
    
* **Tối Ưu Swappiness**: Điều chỉnh tham số `swappiness` trong kernel để kiểm soát khi nào hệ thống sử dụng swap.
    

### **17.3. Phân Tích Hiệu Suất Bộ Nhớ**

#### **Tài Nguyên Trên Host**

* **RAM**: Số lượng RAM vật lý trên host ảnh hưởng đến khả năng chạy nhiều container cùng lúc.
    
* **Swap**: Swap không nên được sử dụng quá nhiều vì ảnh hưởng đến hiệu suất.
    

#### **Quản Lý Bộ Nhớ Hiệu Quả**

* **Giới Hạn Tài Nguyên**: Đặt giới hạn CPU và bộ nhớ cho mỗi container để đảm bảo sự công bằng và hiệu quả sử dụng tài nguyên.
    
* **Tối Ưu Ứng Dụng**: Sử dụng các ứng dụng tối ưu hóa bộ nhớ, tránh rò rỉ bộ nhớ và tối ưu hóa việc sử dụng cache.
    

#### **Kiểm Tra và Tối Ưu**

* **Sử Dụng** `docker stats`: Theo dõi việc sử dụng bộ nhớ và CPU của các container.
    
* **Phân Tích Log OOM**: Kiểm tra log hệ thống để hiểu nguyên nhân và khắc phục sự cố OOM.
    

---

## **18\. Hiểu Sâu Về Memory Management Trong Kernel Linux**

### **18.1. Memory Management Overview**

Kernel Linux quản lý bộ nhớ thông qua nhiều cơ chế như:

* **Paging**: Chuyển các trang bộ nhớ giữa RAM và swap.
    
* **Caching**: Sử dụng bộ nhớ không sử dụng để cache dữ liệu từ hệ thống file.
    
* **Slab Allocator**: Quản lý bộ nhớ cho các đối tượng nhỏ trong kernel.
    

### **18.2. Interplay Between Docker and Kernel Memory Management**

Docker sử dụng các tính năng của kernel để quản lý bộ nhớ cho các container, đảm bảo rằng các container không vượt quá giới hạn và hệ thống host vẫn duy trì hiệu suất.

#### **Paging và Swapping:**

* Khi một container cố gắng sử dụng nhiều bộ nhớ hơn giới hạn, kernel sẽ chuyển các trang bộ nhớ ít sử dụng từ RAM sang swap.
    
* Nếu swap cũng bị giới hạn (`memory.swap`), kernel sẽ kích hoạt OOM killer để chấm dứt tiến trình trong container.
    

#### **Caching:**

* Kernel sử dụng phần bộ nhớ không sử dụng để cache dữ liệu từ hệ thống file.
    
* Docker sử dụng OverlayFS, giúp tối ưu hóa việc sử dụng cache bằng cách chia sẻ các layers read-only giữa các container.
    

#### **OOM Killer:**

* Khi hệ thống gặp tình trạng thiếu bộ nhớ, OOM killer sẽ lựa chọn chấm dứt tiến trình nào để giải phóng bộ nhớ.
    
* Docker cấu hình OOM killer thông qua cgroups để chọn tiến trình trong container bị chấm dứt.
    

### **18.3. Thực Hiện Memory Allocation và Deallocation**

#### **Memory Allocation:**

* Khi container yêu cầu bộ nhớ mới, kernel sẽ cấp phát các trang bộ nhớ.
    
* Nếu không có đủ RAM, kernel sẽ sử dụng swap để mở rộng bộ nhớ ảo.
    

#### **Memory Deallocation:**

* Khi container giải phóng bộ nhớ, kernel sẽ dọn dẹp các trang bộ nhớ và cập nhật cgroups để cho phép container sử dụng lại bộ nhớ.
    

#### **Swap Management:**

* Swap được sử dụng để mở rộng bộ nhớ ảo, nhưng có ảnh hưởng đến hiệu suất.
    
* Docker giới hạn swap để tránh việc container sử dụng quá nhiều swap, ảnh hưởng đến hệ thống host.
    

---

## **19\. Quản Lý Tài Nguyên Bộ Nhớ Với Kubernetes và Docker Swarm**

### **19.1. Kubernetes**

Kubernetes sử dụng các cơ chế như Resource Requests và Limits để quản lý tài nguyên cho các Pod.

#### **Resource Requests và Limits:**

* **Requests**: Yêu cầu tài nguyên tối thiểu mà container cần để chạy.
    
* **Limits**: Giới hạn tài nguyên tối đa mà container có thể sử dụng.
    

#### **Ví Dụ:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        memory: "128Mi"
        cpu: "250m"
      limits:
        memory: "256Mi"
        cpu: "500m"
```

### **19.2. Docker Swarm**

Docker Swarm cũng hỗ trợ quản lý tài nguyên cho các service thông qua giới hạn tài nguyên.

#### **Ví Dụ:**

```bash
docker service create --name web --replicas=3 --limit-cpu=0.5 --limit-memory=512M nginx
```

---

## **20\. Kết Luận**

Hiểu rõ cách Docker hoạt động ở mức dịch vụ và hạ tầng đòi hỏi sự nắm vững về kiến trúc Docker, cách Docker sử dụng các tính năng lõi của Linux như namespaces và cgroups, cũng như cách quản lý tài nguyên hệ thống như bộ nhớ RAM, CPU, và I/O. Docker tận dụng sự cách ly và quản lý tài nguyên của kernel Linux để cung cấp môi trường container hóa hiệu quả, linh hoạt và an toàn.

### **Tóm Tắt Các Điểm Chính:**

1. **Docker Architecture**: Docker Client gửi lệnh tới Docker Daemon, Docker Daemon quản lý container thông qua containerd và runc.
    
2. **Namespaces và cgroups**: Cách ly tài nguyên và quản lý tài nguyên hệ thống cho container.
    
3. **Memory Management**: Sử dụng cgroups để giới hạn và quản lý bộ nhớ, tránh tình trạng OOM.
    
4. **Storage và Networking**: Sử dụng OverlayFS cho hệ thống file, và các driver mạng để quản lý giao tiếp mạng.
    
5. **Security**: Sử dụng user namespaces, AppArmor/SELinux, và Seccomp để tăng cường bảo mật container.
    
6. **Monitoring và Logging**: Sử dụng các công cụ như cAdvisor, Prometheus, và Docker logs để theo dõi và giám sát container.