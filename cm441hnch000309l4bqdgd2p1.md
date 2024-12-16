---
title: "Kubernetes: Giới thiệu về Orchestration Container cho người mới bắt đầu"
seoTitle: "Kubernetes: Master Orchestration Platform"
seoDescription: "Tìm hiểu cách Kubernetes quản lý và mở rộng container qua các khái niệm cơ bản và thực hành triển khai ứng dụng đầu tiên.
kubernetes là gì?"
datePublished: Sat Nov 30 2024 10:38:27 GMT+0000 (Coordinated Universal Time)
cuid: cm441hnch000309l4bqdgd2p1
slug: kubernetes-bai-1-hieu-va-lam-chu-nen-tang-orchestration
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1732968003682/4939e043-6130-4986-b4cb-9b0588e239e3.webp
tags: kubernetes, devops

---

# 1\. **Giới Thiệu: Tại Sao Cần Kubernetes?**

Trong thế giới phát triển phần mềm hiện đại, **container** là một công cụ mạnh mẽ. Chúng ta có thể đóng gói ứng dụng, các thư viện và phụ thuộc vào một môi trường di động, dễ triển khai. **Docker** là ví dụ phổ biến của container. Nhưng bạn sẽ làm gì khi có hàng chục, thậm chí hàng trăm container cần quản lý?

Khi ứng dụng của bạn lớn hơn, bạn sẽ gặp một số vấn đề:

* **Quản lý và triển khai hàng loạt container**.
    
* **Tự động phân phối container tới các máy chủ phù hợp**.
    
* **Đảm bảo ứng dụng luôn sẵn sàng** khi container gặp lỗi.
    
* **Mở rộng ứng dụng linh hoạt** khi lưu lượng tăng cao.
    

Đây là lúc Kubernetes, hay còn gọi là **K8s (ca-tám-ét :v)**, xuất hiện. Kubernetes là một nền tảng mã nguồn mở giúp **điều phối container**, cung cấp các công cụ mạnh mẽ để triển khai, quản lý và mở rộng ứng dụng của bạn.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732962804303/0061063f-0c78-46f9-9a16-d9aafd6e5db0.png align="center")

---

# 2\. **Kubernetes Là Gì?**

**Kubernetes** (phát âm theo kiểu quốc tế thì: koo-ber-net-eez, mình thì toàn phát âm thành: cu-bơ-nết, thỉnh thoảng là cu-bơ-nây-tít :vv) được phát triển bởi Google và hiện là một dự án thuộc CNCF (Cloud Native Computing Foundation). Kubernetes giúp bạn tự động hóa nhiều công việc liên quan đến quản lý container, như:

* Khởi động lại container khi gặp lỗi.
    
* Phân phối tải (load balancing) cho ứng dụng.
    
* Mở rộng ứng dụng tự động theo nhu cầu (autoscaling).
    
* Triển khai phiên bản mới mà không làm gián đoạn dịch vụ (rolling updates).
    
* Quản lý cấu hình và thông tin nhạy cảm (ConfigMap & Secrets).
    

## **Lợi ích chính của Kubernetes**

* **Khả năng mở rộng (Scalability):** Kubernetes cho phép bạn mở rộng ứng dụng khi nhu cầu tăng cao mà không cần can thiệp thủ công.
    
* **Khả dụng cao (High Availability):** Kubernetes tự động phát hiện và khắc phục các lỗi xảy ra với container.
    
* **Tính di động (Portability):** Bạn có thể triển khai Kubernetes trên nhiều nền tảng, từ môi trường local (Minikube) đến các đám mây lớn như AWS, Google Cloud, Azure.
    

---

# 3\. **Kiến Trúc Kubernetes**

Để hiểu cách Kubernetes hoạt động, bạn cần biết các thành phần cơ bản trong kiến trúc của nó.

![Hình ảnh này mình tham chiếu trên docs trang chủ của k8s https://kubernetes.io/docs/concepts/architecture/](https://cdn.hashnode.com/res/hashnode/image/upload/v1732962868657/a2ac8250-5415-4025-bcb5-6d089ea8fc4a.png align="center")

## **3.1. Control Plane (Master Node)**

Control Plane chịu trách nhiệm quản lý toàn bộ hệ thống Kubernetes. Các thành phần chính bao gồm:

* **API Server:** Là cổng giao tiếp chính giữa người dùng (qua `kubectl`) và cluster.
    
* **Scheduler:** Quyết định node nào sẽ chạy pod dựa trên tài nguyên sẵn có.
    
* **Controller Manager:** Giám sát trạng thái của cluster và thực hiện hành động khi trạng thái thực tế khác trạng thái mong muốn.
    
* **ETCD:** Là cơ sở dữ liệu key-values phân tán để lưu trữ toàn bộ thông tin và cấu hình của cluster.
    

Tạm thời chưa đào sâu thì chúng ta sẽ được nhìn trực quan như thế này, bằng bộ cli **kubectl**:

```bash
╰─ k get pods
NAME                                     READY   STATUS    RESTARTS          AGE
coredns-76f75df574-x4b6x                 1/1     Running   23 (16d ago)      144d
coredns-76f75df574-xg424                 1/1     Running   25 (16d ago)      144d
etcd-docker-desktop                      1/1     Running   27 (16d ago)      144d
kube-apiserver-docker-desktop            1/1     Running   31 (3d23h ago)    144d
kube-controller-manager-docker-desktop   1/1     Running   29 (16d ago)      144d
kube-scheduler-docker-desktop            1/1     Running   135 (3d22h ago)   144d
storage-provisioner                      1/1     Running   58 (3d22h ago)    144d
```

Chưa cần tìm hiểu sâu, chỉ cần biết nó tàm tạm là như này đã, rồi chúng ta sẽ biết từng dòng từng cột một trong list output console trên là gì.

## **3.2. Worker Nodes**

Worker Nodes thực hiện công việc chính là chạy container. Các thành phần chính trên node bao gồm:

* **Kubelet:** Quản lý pod và container trên node.
    
* **Kube Proxy:** Định tuyến mạng giữa các pod và service.
    
* **Container Runtime:** Công cụ thực thi container (Docker, containerd, CRI-O).
    

Và tiếp tục trông nó sẽ như này:

```bash
NAME                                     READY   STATUS    RESTARTS          AGE
kube-proxy-9b9lx                         1/1     Running   13 (16d ago)      144d
```

## **3.3. Pod**

Pod là đơn vị triển khai nhỏ nhất trong Kubernetes. Một pod có thể chứa một hoặc nhiều container, chia sẻ chung mạng và ổ đĩa.

## **Sơ đồ kiến trúc**

```markdown
+---------------------------------------+
| Control Plane                         |
| +-----------------+  +--------------+ |
| | API Server      |  | Scheduler    | |
| +-----------------+  +--------------+ |
| +-----------------------------------+ |
| | Controller Manager                | |
| +-----------------------------------+ |
| +------------------+ +--------------+ |
| | ETCD (Database)  |                | |
+---------------------------------------+
                 |
                 |
+---------------------------------------+
| Worker Node                           |
| +----------------+  +----------------+ |
| | Kubelet        |  | Kube Proxy     | |
| +----------------+  +----------------+ |
| +-----------------------------------+ |
| | Pod                                | |
| | +-------------------------------+ | |
| | | Container(s)                  | | |
| | +-------------------------------+ | |
+---------------------------------------+
```

---

# 4\. **Thực Hành: Chạy Ứng Dụng Trên Kubernetes**

Dưới đây là hướng dẫn chi tiết để triển khai ứng dụng trên Kubernetes.

## **4.1. Cài Đặt Kubernetes**

Bạn có thể bắt đầu với Minikube để chạy Kubernetes trên máy cá nhân.

1. **Cài Minikube:**
    
    ```bash
    curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
    sudo install minikube-linux-amd64 /usr/local/bin/minikube
    minikube start
    ```
    
2. **Kiểm tra cài đặt:**
    
    ```bash
    kubectl version --client
    minikube status
    ```
    

## **4.2. Tạo Deployment**

Deployment giúp bạn triển khai và quản lý pod. Tạo file `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
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

Áp dụng file cấu hình:

```bash
kubectl apply -f deployment.yaml
```

Kiểm tra trạng thái pod:

```bash
kubectl get pods
```

---

## **4.3. Tạo Service**

Service cho phép bạn expose ứng dụng ra bên ngoài. Tạo file `service.yaml`:

```bash
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: NodePort
```

Áp dụng Service:

```bash
kubectl apply -f service.yaml
kubectl get svc
```

Truy cập ứng dụng:

```bash
minikube service nginx-service
```

---

# 5\. **Use Cases thường gặp**

Dưới đây mình tạm thời list 1 vài use case mà lý do đa số các sysops/devops lựa chọn k8s là hạ tầng để quản lý container.

## **5.1. Microservices**

Kubernetes hỗ trợ triển khai microservices, giúp bạn chia nhỏ ứng dụng thành các dịch vụ độc lập.

## **5.2. Autoscaling**

Bạn có thể sử dụng **Horizontal Pod Autoscaler (HPA)** để tự động scale pod:

```bash
kubectl autoscale deployment nginx-deployment --cpu-percent=50 --min=2 --max=10
```

## **5.3. CI/CD**

Kubernetes tích hợp tốt với Jenkins, GitLab CI/CD và ArgoCD, hỗ trợ triển khai tự động khi mã nguồn thay đổi.

## **5.4. High Availability**

Kubernetes giúp tự động khởi động lại pod khi xảy ra lỗi, đảm bảo ứng dụng luôn khả dụng.

---

# 6\. **Câu Hỏi Thường Gặp**

## **6.1. Kubernetes khác Docker như thế nào?**

* Docker là công cụ tạo container.
    
* Kubernetes là công cụ quản lý và điều phối container.
    

## **6.2. Pod và Container khác nhau thế nào?**

* Pod là nhóm chứa một hoặc nhiều container.
    
* Các container trong pod chia sẻ cùng mạng và ổ đĩa.
    
* Tóm lại Pod sẽ gom các container thành 1 phân vùng riêng, để controller k8s quản lý.
    

## **6.3. Làm sao để giám sát Kubernetes?**

Bạn có thể dùng các công cụ như:

* **Prometheus** và **Grafana** để giám sát.
    
* **Kubernetes Dashboard** để quản lý cluster qua giao diện web.
    

## **6.4. Kubernetes có chạy được trên Windows không?**

Kubernetes hỗ trợ Windows, nhưng phần lớn các công cụ và ứng dụng chủ yếu chạy tốt trên Linux.

## **6.5. Kubernetes có cần Docker không?**

Không bắt buộc. Kubernetes hỗ trợ nhiều container runtime như **containerd**, **CRI-O**, ngoài Docker.

Tức là chỉ cần 1 công cụ ảo hoá container, thì k8s sẽ tích hợp để điều phối được.

---

# 7\. **Tổng Kết**

Kubernetes là công cụ mạnh mẽ để quản lý container. Bài viết này đã cung cấp cho bạn cái nhìn cơ bản và một ví dụ thực hành để bạn bắt đầu. Ở các bài tiếp theo, chúng ta sẽ tìm hiểu các khái niệm nâng cao như **ConfigMap**, **Secrets**, **Ingress**, và tích hợp Kubernetes với CI/CD.

Bạn đã sẵn sàng khám phá Kubernetes chưa? Hãy bắt đầu triển khai ứng dụng đầu tiên ngay hôm nay! 🚀