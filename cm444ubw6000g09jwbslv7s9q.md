---
title: "Kubernetes: Giới thiệu tổng quan về Deployment"
seoTitle: "Kubernetes Deployments: Beginner's Guide"
seoDescription: "Learn about Kubernetes deployments, their functions, and how to utilize them for seamless application management, updates, and recovery in clusters"
datePublished: Sat Nov 30 2024 12:12:17 GMT+0000 (Coordinated Universal Time)
cuid: cm444ubw6000g09jwbslv7s9q
slug: kubernetes-gioi-thieu-tong-quan-ve-deployment
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1732968702741/3d97636f-f678-459a-be07-e55b2a5f0613.webp
tags: deployment, kubernetes, devops

---

# **1\. Deployment Là Gì?**

## **1.1. Định Nghĩa**

Deployment là một đối tượng (object) trong Kubernetes được thiết kế để quản lý và triển khai Pod theo cách tự động và linh hoạt. Nó cung cấp các tính năng mạnh mẽ để đảm bảo ứng dụng của bạn luôn chạy ở trạng thái ổn định, bao gồm:

* Tự động triển khai hoặc cập nhật các Pod.
    
* Triển khai cập nhật mà không làm gián đoạn dịch vụ (Rolling Updates).
    
* Khôi phục trạng thái cũ khi gặp sự cố (Rollback).
    
* Duy trì số lượng Pod luôn khớp với cấu hình mong muốn (Replicas).
    
* Tự động khởi động lại Pod khi Pod bị lỗi.
    

## **1.2. Deployment Hoạt Động Như Thế Nào?**

Deployment sử dụng một đối tượng phụ gọi là **ReplicaSet**, chịu trách nhiệm đảm bảo số lượng Pod đúng như yêu cầu. Nếu Pod bị xóa hoặc gặp sự cố, ReplicaSet sẽ tạo ra Pod mới. Deployment quản lý toàn bộ vòng đời của ReplicaSet, bao gồm:

* Tạo ReplicaSet mới khi cần cập nhật ứng dụng.
    
* Xóa ReplicaSet cũ khi không còn cần thiết.
    
* Tự động phân phối và cập nhật Pod một cách có kiểm soát.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732968521056/980bc308-d9f1-410e-b859-6caf8476d5a5.png align="center")

---

# **2\. Tại Sao Sử Dụng Deployment?**

## **2.1. Triển khai ứng dụng dễ dàng**

Thay vì phải tự quản lý từng Pod, Deployment tự động hóa quá trình này, giúp bạn dễ dàng triển khai ứng dụng và đảm bảo ứng dụng luôn hoạt động.

## **2.2. Hỗ trợ cập nhật liền mạch**

* **Rolling Updates:** Cập nhật từng Pod, đảm bảo không có thời gian chết (downtime).
    
* **Rollback:** Nhanh chóng quay lại phiên bản trước nếu phiên bản hiện tại gặp sự cố.
    

## **2.3. Tự động phục hồi**

Khi một hoặc nhiều Pod gặp lỗi, Deployment sẽ tự động khởi tạo các Pod thay thế, đảm bảo ứng dụng luôn sẵn sàng.

---

# **3\. Các Thành Phần Chính Trong Deployment**

## **3.1. ReplicaSet**

* **ReplicaSet** đảm bảo số lượng Pod luôn khớp với cấu hình mong muốn.
    
* Deployment tự động tạo và quản lý ReplicaSet, bạn không cần can thiệp trực tiếp vào ReplicaSet.
    

## **3.2. Pod Template**

* Đây là nơi định nghĩa các thông số cấu hình cho các Pod được tạo (image, port, biến môi trường, volume,...).
    
* **Pod Template** nằm trong `.spec.template`.
    

## **3.3. Strategy (Chiến Lược Cập Nhật)**

Deployment hỗ trợ 2 chiến lược cập nhật chính:

1. **RollingUpdate** (Mặc định): Thay thế Pod cũ bằng Pod mới từng bước.
    
2. **Recreate:** Xóa tất cả Pod cũ trước khi tạo Pod mới.
    

---

# **4\. Use Case Thực Tế Của Deployment**

## **4.1. Triển khai một ứng dụng web**

Giả sử bạn có ứng dụng Node.js và cần triển khai 3 bản sao (replica) để đảm bảo ứng dụng hoạt động ổn định. Deployment sẽ tự động quản lý việc tạo và duy trì các Pod này.

---

## **4.2. Rolling Updates**

Cập nhật ứng dụng từ phiên bản cũ (v1.0.0) sang phiên bản mới (v1.1.0) mà không cần dừng ứng dụng.

---

## **4.3. Rollback**

Khi phiên bản mới của ứng dụng gặp lỗi, bạn có thể sử dụng Deployment để quay lại phiên bản trước chỉ với một lệnh.

---

# **5\. Triển Khai Và Quản Lý Deployment**

## **5.1. Tạo Một Deployment Đơn Giản**

Tạo file `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```

Áp dụng file:

```yaml
kubectl apply -f deployment.yaml
```

Kiểm tra Deployment:

```yaml
kubectl get deployments
kubectl describe deployment nginx-deployment
```

Kiểm tra Pod được tạo:

```yaml
kubectl get pods
```

---

## **5.2. Rolling Updates**

Cập nhật Deployment sang phiên bản mới của Nginx:

```yaml
kubectl set image deployment/nginx-deployment nginx=nginx:1.22
```

Theo dõi quá trình cập nhật:

```yaml
kubectl rollout status deployment/nginx-deployment
```

---

## **5.3. Rollback**

Quay lại phiên bản trước nếu có lỗi:

```yaml
kubectl rollout undo deployment/nginx-deployment
```

Xem lịch sử các lần cập nhật:

```yaml
kubectl rollout history deployment/nginx-deployment
```

---

## **5.4. Xóa Deployment**

Xóa Deployment và tất cả Pod liên quan:

```yaml
kubectl delete deployment nginx-deployment
```

---

# **6\. Các Chiến Lược Cập Nhật Deployment**

## **6.1. Rolling Update**

* **Triển khai từng bước:** Pod mới được tạo và thay thế Pod cũ dần dần.
    
* **Không downtime:** Luôn đảm bảo số lượng Pod tối thiểu hoạt động.
    
* **Cấu hình mẫu:**
    

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

## **6.2. Recreate**

* **Xóa toàn bộ Pod cũ:** Trước khi tạo Pod mới.
    
* **Dễ dàng:** Nếu ứng dụng không yêu cầu độ sẵn sàng cao.
    
* **Cấu hình mẫu:**
    

```yaml
strategy:
  type: Recreate
```

---

# **7\. Các Tính Năng Khác Của Deployment**

## **7.1. Tạm dừng và tiếp tục Deployment**

Tạm dừng quá trình rollout để thực hiện nhiều thay đổi:

```yaml
kubectl rollout pause deployment/nginx-deployment
```

Tiếp tục quá trình:

```yaml
kubectl rollout resume deployment/nginx-deployment
```

---

## **7.2. Tăng số lượng Pod**

Thay đổi số lượng Pod đang chạy:

```yaml
kubectl scale deployment nginx-deployment --replicas=5
```

---

## **7.3. Kiểm soát tiến trình cập nhật**

Cấu hình `progressDeadlineSeconds` để giới hạn thời gian cập nhật:

```yaml
spec:
  progressDeadlineSeconds: 600
```

---

## **7.4. Quản lý lịch sử Deployment**

Giới hạn số lượng lịch sử cần lưu giữ:

```yaml
spec:
  revisionHistoryLimit: 5
```

---

# **8\. Câu Hỏi Thường Gặp Về Deployment**

## **8.1. Deployment khác gì ReplicaSet?**

* **ReplicaSet**: Đảm bảo số lượng Pod đúng với cấu hình.
    
* **Deployment**: Quản lý ReplicaSet, hỗ trợ cập nhật liền mạch và rollback.
    

---

## **8.2. Có thể cập nhật những gì trong Deployment?**

Bạn có thể cập nhật:

* Image của container.
    
* Biến môi trường.
    
* Volume.
    
* Chiến lược cập nhật.
    

---

## **8.3. Làm thế nào để khắc phục một Deployment bị lỗi?**

* Kiểm tra log của Deployment và Pod:
    
    ```bash
    kubectl describe deployment/nginx-deployment
    kubectl logs pod-name
    ```
    
* Rollback về phiên bản trước:
    
    ```bash
    kubectl rollout undo deployment/nginx-deployment
    ```
    

---

# **9\. Kết Luận**

Deployment là công cụ mạnh mẽ, giúp quản lý vòng đời ứng dụng một cách hiệu quả, từ triển khai, cập nhật đến phục hồi khi gặp sự cố. Bằng cách sử dụng Deployment, bạn có thể dễ dàng vận hành và duy trì ứng dụng trong môi trường Kubernetes.

Hãy thử tạo Deployment đầu tiên của bạn và trải nghiệm cách Kubernetes giúp tự động hóa các tác vụ phức tạp! 🚀