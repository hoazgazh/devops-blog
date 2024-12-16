---
title: "Service Account trong Kubernetes: Giới thiệu, Cách tấn công, Bảo vệ, và Chaos Testing"
seoTitle: "Kubernetes Service Accounts: Defense and Testing"
seoDescription: "Learn about Service Accounts in Kubernetes: their role, how they can be attacked, protection strategies, and Chaos Testing for improved security"
datePublished: Thu Nov 28 2024 15:53:22 GMT+0000 (Coordinated Universal Time)
cuid: cm41huy1i000f0al53wr5ckjy
slug: service-account-trong-kubernetes-gioi-thieu-cach-tan-cong-bao-ve-va-chaos-testing
tags: kubernetes, devops

---

### 1\. Service Account (SA) trong Kubernetes là gì?

Trong Kubernetes, Service Account (SA) là một thành phần rất quan trọng để giúp các pod và container có thể tương tác với API Server. Mỗi Pod đều có một SA gắn liền với nó. SA được sử dụng để cung cấp các quyền truy cập nhất định cho các Pod khi có nhu cầu truy vấn và tương tác với các tài nguyên trong cộng Kubernetes.

Khi bạn tạo một Pod, Kubernetes sẽ tự động gắn cho Pod một Service Account có tên là "default". SA này sẽ chỉ được cấp các quyền cơ bản để pod có thể truy cập và làm việc với các tài nguyên ở mức giới hạn.

Service Account có thể được hiểu như một dạng tài khoản đặc biệt dành cho ứng dụng và pod, khác với User Account (tài khoản người dùng), SA được sử dụng để cấp quyền cho các ứng dụng tự động và không liên quan đến con người.

**Phân biệt User Account và Service Account:**

* **User Account**: Được sử dụng bởi con người để truy cập và quản lý Kubernetes cluster. Các User Account thường được quản lý bởi hệ thống ngoài (như LDAP hoặc Active Directory).
    
* **Service Account**: Được sử dụng bởi các ứng dụng, pod để tương tác với Kubernetes API. Service Account được quản lý trực tiếp trong Kubernetes.
    

### 2\. Cách SA Hoạt Động trong Cụm và Pod/Container

Mỗi khi có một Pod được khởi chạy, Kubernetes sẽ tạo một JWT token được lưu vào đường dẫn `/var/run/secrets/`[`kubernetes.io/serviceaccount/token`](http://kubernetes.io/serviceaccount/token) trong Pod đó. Token này để giúp các container trong Pod tìm kiếm và tương tác với API Server. Token này đóng vai trò như chìa khóa, giúp container có thể tác động tới các resource của cụm tùy theo quyền của SA.

**Chi tiết cách hoạt động của SA trong Pod:**

* **Gán Token Tự Động:** Mặc định, Kubernetes tự động gán token cho tất cả các Pod thông qua cấu hình `automountServiceAccountToken: true`. Token này được container sử dụng để xác thực khi tương tác với Kubernetes API.
    
* **Liên kết với Role và RoleBinding:** SA được liên kết với Role hoặc ClusterRole thông qua RoleBinding hoặc ClusterRoleBinding. Điều này cho phép xác định chính xác những quyền mà Pod được phép thực hiện.
    
* **Cơ chế Xác Thực (Authentication) và Phân Quyền (Authorization):** Khi Pod gửi yêu cầu đến Kubernetes API Server, token SA sẽ được xác thực, và hệ thống sẽ kiểm tra quyền thông qua cơ chế phân quyền RBAC (Role-Based Access Control).
    

Ngoài ra, Service Account còn liên kết với các đối tượng quan trọng khác trong Kubernetes như Role, RoleBinding, ClusterRole, và ClusterRoleBinding để kiểm soát mức độ quyền hạn của Pod trong toàn bộ cụm.

* **Role** và **RoleBinding**: Được sử dụng để quản lý quyền truy cập trong một namespace cụ thể.
    
* **ClusterRole** và **ClusterRoleBinding**: Dùng để quản lý quyền truy cập ở cấp độ toàn cụm, và có thể được sử dụng trong nhiều namespace khác nhau.
    

Tóm lại, Service Account hoạt động như sau:

* Mỗi Pod được gán một Service Account.
    
* Token được sử dụng trong container để tìm kiếm và tương tác với API Server.
    
* Quyền truy cập của Pod sẽ phụ thuộc vào quyền của SA gắn cho Pod.
    
* SA có thể được liên kết với các Role và RoleBinding để kiểm soát mức độ truy cập tài nguyên.
    

### 3\. Cách Tấn Công Vào Service Account

Bây giờ chúng ta sẽ đối mặt với những nguy cơ mà SA có thể gây ra, nhất là trong các trường hợp bạn quản lý sai cách hoặc các lỗ hổng từ việc phân quyền không hợp lý.

#### 3.1 Tấn Công Nhỏ Vào Token SA

Khi Pod được tạo, token của SA được lưu trực tiếp trong file trong container. Tin tặc có thể khai thác việc truy cập shell vào container để truy cập token này và dùng cho các mục đích độc hại.

**Step by Step Tấn Công Token SA:**

1. **Attacker có quyền truy cập shell trong container:** Attacker có thể sử dụng các lỗi bảo mật khác để truy cập vào container, như các lỗ hổng ứng dụng hoặc cấu hình sai của Kubernetes.
    
2. **Tìm file token:** Token được lưu tại `/var/run/secrets/`[`kubernetes.io/serviceaccount/token`](http://kubernetes.io/serviceaccount/token), attacker có thể thực hiện:
    
    ```sh
    cat /var/run/secrets/kubernetes.io/serviceaccount/token
    ```
    
3. **Sử dụng token để truy vấn Kubernetes API:** Với token này, attacker có thể thực hiện các yêu cầu đến API Server của Kubernetes, ví dụ:
    
    ```sh
    curl -k -H "Authorization: Bearer <token>" https://<API_SERVER_IP>/api/v1/namespaces/
    ```
    
4. **Dùng token để tìm hiểu về cấu trúc cluster, tài nguyên nhạy cảm:** Attacker có thể tiếp tục truy vấn thêm để khám phá tài nguyên và tìm hiểu về cấu trúc của cluster.
    

**Hậu quả:** Tin tặc có thể truy cập vào các thông tin nhạy cảm của cluster, thao túng các tài nguyên hoặc gây gián đoạn dịch vụ.

#### 3.2 Tấn Công Bằng Việc Gắn Quyền Quá Rộng

Một lỗi phổ biến khác là gán cho Service Account các quyền (Role hoặc ClusterRole) quá rộng. Ví dụ, bạn có thể vô tình gán quyền "cluster-admin" cho SA của một Pod mà không cần thiết, dẫn đến việc Pod đó có khả năng kiểm soát toàn bộ cụm Kubernetes.

**Ví dụ tấn công:**

1. **Attacker tìm thấy một Pod sử dụng SA có quyền "cluster-admin":** Thông qua việc liệt kê các SA trong cluster và kiểm tra các quyền của chúng, attacker có thể tìm ra SA có quyền "cluster-admin".
    
2. **Sử dụng token của SA để thực hiện các hành động độc hại:** Ví dụ như xóa namespaces, sửa đổi cấu hình của cluster, hoặc triển khai workload ác ý, từ đó hoàn toàn kiểm soát được cluster.
    

**Hậu quả:** Cluster có thể bị attacker chiếm quyền hoàn toàn, gây mất an toàn thông tin và ảnh hưởng nghiêm trọng đến dịch vụ.

#### 3.3 Tấn Công Thông Qua Pod Injection

Một hình thức tấn công khác là "Pod Injection". Attacker có thể lợi dụng lỗ hổng hoặc cấu hình sai trong hệ thống CI/CD để tiêm các Pod độc hại, từ đó truy cập vào token của SA hoặc thậm chí gán thêm quyền cho SA.

**Step by Step Pod Injection:**

1. **Truy cập hệ thống CI/CD có quyền điều khiển Kubernetes:** Attacker thâm nhập vào hệ thống CI/CD và có quyền thực thi lệnh tạo pod.
    
2. **Tạo pod với một Service Account có quyền:** Attacker có thể cấu hình pod mới với SA có quyền rộng hoặc lợi dụng SA mặc định của namespace.
    
3. **Truy cập token và thực hiện các hành động độc hại:** Từ Pod mới, attacker có thể lấy token của SA và thực hiện các hành vi độc hại trong cluster.
    

**Hậu quả:** Toàn bộ cluster có thể bị kiểm soát nếu SA của Pod mới có quyền hạn lớn.

### 4\. Cách Bảo Vệ Service Account

#### 4.1 Nguyên Tắc Ít Quyền Nhất (Least Privilege)

Hãy đảm bảo mỗi SA chỉ có những quyền tối thiểu cần thiết để hoàn thành nhiệm vụ. Bạn có thể làm điều này bằng cách tạo Role và RoleBinding riêng lẻ, thay vì sử dụng ClusterRole với quyền cao.

**Ví dụ:**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: read-pods
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: read-pods
subjects:
- kind: ServiceAccount
  name: my-service-account
  namespace: default
```

#### 4.2 Không Sử Dụng Service Account Mặc Định

Mỗi namespace có một SA mặc định tên là "default". Hãy tạo và sử dụng các SA cụ thể cho từng ứng dụng của bạn thay vì sử dụng SA mặc định, để có thể kiểm soát chặt chẽ hơn quyền truy cập.

**Ví dụ:** Tạo một Service Account mới và sử dụng nó cho Pod thay vì SA mặc định:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: custom-sa
  namespace: default
```

Sau đó gắn nó vào Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  namespace: default
spec:
  serviceAccountName: custom-sa
  containers:
  - name: my-container
    image: my-image
```

#### 4.3 Sử Dụng PodSecurityPolicy và NetworkPolicies

PodSecurityPolicy có thể được sử dụng để hạn chế khả năng của Pod trong việc truy cập vào các tài nguyên không cần thiết. NetworkPolicies cũng rất quan trọng để kiểm soát lưu lượng truy cập giữa các Pod.

**Ví dụ PodSecurityPolicy:** PodSecurityPolicy có thể hạn chế những quyền như chạy dưới quyền root, gắn volume không an toàn, hoặc giới hạn quyền mạng của container.

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted-psp
spec:
  privileged: false
  runAsUser:
    rule: 'MustRunAsNonRoot'
  seLinux:
    rule: 'RunAsAny'
  supplementalGroups:
    rule: 'MustRunAs'
    ranges:
    - min: 1
      max: 65535
  fsGroup:
    rule: 'MustRunAs'
    ranges:
    - min: 1
      max: 65535
```

#### 4.4 Hạn Chế Token Automount

Bạn có thể cấu hình Pod để không tự động gắn token của SA vào Pod nếu không cần thiết bằng cách sử dụng thuộc tính `automountServiceAccountToken`.

**Ví dụ:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  namespace: default
spec:
  serviceAccountName: my-service-account
  automountServiceAccountToken: false
  containers:
  - name: my-container
    image: my-image
```

Việc này giúp giảm nguy cơ bị đánh cắp token của SA nếu Pod không cần sử dụng đến nó.

#### 4.5 Giám Sát và Ghi Log

Luôn giám sát và ghi log các hoạt động liên quan đến Service Account. Sử dụng các công cụ giám sát như Prometheus và Grafana để theo dõi hoạt động của Kubernetes API và phát hiện các hành vi bất thường.

**Ví dụ:** Thiết lập Alert khi có nhiều yêu cầu thất bại từ một SA cụ thể hoặc khi có sự thay đổi quyền hạn bất thường.

### 5\. Chaos Testing Service Account

Chaos Engineering giúp kiểm tra mức độ bảo mật của cluster và khả năng chịu lỗi khi SA bị tấn công. Dưới đây là một ví dụ về việc Chaos Testing với SA.

#### 5.1 Ví Dụ Chaos Testing

1. **Mục tiêu:** Xóa một Service Account quan trọng và xem phản ứng của hệ thống.
    
2. **Công cụ:** Sử dụng Chaos Mesh hoặc Litmus để triển khai thử nghiệm.
    
3. **Kịch bản:**
    
    * **Xóa SA sử dụng:**
        
        ```sh
        kubectl delete serviceaccount critical-sa -n critical-namespace
        ```
        
    * **Kiểm tra phản ứng của các Pod đang phụ thuộc vào SA đó:** Xem xét cách chúng phản ứng và ghi nhận lỗi xảy ra, từ đó đưa ra các biện pháp khắc phục phù hợp như cấu hình lại SA hoặc thiết lập cơ chế tự động thay thế SA khi gặp sự cố.
        
    * **Mở rộng Chaos Testing:** Ngoài việc xóa SA, bạn có thể thực hiện các thử nghiệm như tạm ngừng hoặc thay đổi quyền của SA, xem các workload có thể khôi phục và tiếp tục hoạt động hay không.
        

### 6\. Kết Luận

Service Account là một phần quan trọng trong Kubernetes, đảm bảo tính an toàn và hiệu quả của các Pod trong quá trình giao tiếp với API Server. Tuy nhiên, việc sử dụng và quản lý SA không đúng cách có thể dẫn đến những rủi ro lớn. Do đó, hãy luôn áp dụng nguyên tắc ít quyền nhất, tạo SA cụ thể cho từng workload, và liên tục kiểm tra bảo mật thông qua Chaos Testing để đảm bảo tính an toàn cho hệ thống của bạn.

Việc hiểu rõ Service Account và triển khai đúng cách sẽ giúp bạn bảo vệ cụm Kubernetes khỏi những cuộc tấn công tiềm ẩn và tối ưu hóa bảo mật cho hệ thống. Hãy luôn luôn kiểm tra và cập nhật cấu hình bảo mật cho SA và các tài nguyên liên quan. Thực hiện giám sát liên tục và kiểm tra bảo mật định kỳ để đảm bảo rằng hệ thống của bạn luôn an toàn và sẵn sàng đối phó với các tình huống tấn công.