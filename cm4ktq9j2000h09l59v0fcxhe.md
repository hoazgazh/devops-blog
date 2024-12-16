---
title: "Hướng dẫn triển khai, vận hành, bảo mật cụm k8s trên Ubuntu: Từ cài đặt cơ bản đến go-live"
seoTitle: "Kubernetes Cluster on Ubuntu: Setup to Production"
seoDescription: "Hướng dẫn chi tiết triển khai, bảo mật, và nâng cao cụm Kubernetes trên Ubuntu từ cơ bản đến go-live production, đáp ứng tiêu chuẩn an ninh"
datePublished: Thu Dec 12 2024 04:33:17 GMT+0000 (Coordinated Universal Time)
cuid: cm4ktq9j2000h09l59v0fcxhe
slug: huong-dan-trien-khai-van-hanh-bao-mat-cum-k8s-tren-ubuntu-tu-cai-dat-co-ban-den-go-live
tags: kubernetes, devops

---

## MỤC TIÊU, BỐI CẢNH VÀ PHẠM VI

**Mục tiêu:**

* Khởi tạo thành công một cụm Kubernetes trên một máy chủ Ubuntu mới (có thể là máy vật lý hoặc VM), từ lúc hạ tầng chưa có gì đến khi cụm ở trạng thái "go-live".
    
* Mức độ chi tiết: vượt xa thông thường, đi kèm phân tích tại sao, như thế nào, rủi ro, tùy chọn thay thế, đánh giá hiệu năng, đáp ứng an ninh, quản lý tài nguyên, compliance, logging, monitoring, auditing.
    
* Không dừng lại ở mức cài xong cluster, mà còn đảm bảo sau khi cluster hoạt động, có khả năng vận hành dài hạn: backup, restore, upgrade, scale, CI/CD, policy, RBAC, CNI, CSI, Ingress, Service Mesh, Serverless, Hybrid Cloud, Multi-Cluster Federation, HA, DR, Observability, FinOps, SecOps, DevSecOps.
    

**Bối cảnh sử dụng:**

* Một máy chủ Ubuntu hoàn toàn mới, kết nối Internet. Có thể là trong Data Center on-premise, hoặc trên một cloud provider, hoặc thậm chí là máy trạm thí nghiệm trong văn phòng.
    
* Bạn là kỹ sư hạ tầng/DevOps/SRE/kiến trúc sư hệ thống cần triển khai K8s phục vụ chạy ứng dụng container.
    
* Bạn muốn mọi thứ: bảo mật, chuẩn hoá, compliance, logs, metrics, alert, backup, test, CI/CD, policy, upgrade pipeline, ...
    
* Dù thực tế một máy chủ chạy một cluster K8s duy nhất không phải là mô hình HA, nhưng trong phạm vi này, chúng ta giả sử đây là bước khởi đầu trước khi mở rộng sang nhiều node khác.
    

**Phạm vi kiến thức:**

* Từ kiến thức hệ điều hành (Linux, Ubuntu), kernel tuning, cgroup, systemd, network stack, IP forwarding.
    
* Container runtime (containerd), CNI plugin (Flannel, Calico), CSI driver (nếu cần), CRI.
    
* Kubernetes control plane, kubeadm, kubelet, kubectl, manifest, static pod, etcd, apiserver, scheduler, controller-manager, cloud-controller-manager (nếu có), kube-proxy, CoreDNS.
    
* RBAC, SecurityPolicy/PodSecurityAdmission, Admission Controller, OPA/Gatekeeper, Policy enforcement, image scanning, secrets management.
    
* Giám sát (Prometheus, Grafana), logging (ELK/EFK, Loki), tracing (Jaeger), auditing apiserver, network policy, firewall.
    
* CI/CD (Jenkins, GitLab CI, GitOps tool: Argo CD, Flux), IaC (Terraform, Ansible), DR & Backup (Velero, etcdctl snapshot), Ingress (Nginx Ingress, Traefik), Service Mesh (Istio, Linkerd), serverless (Knative), multi-cluster federation (Kubefed).
    
* Nâng cấp cluster (kubeadm upgrade), migrate, scale, performance tuning, cost optimization (FinOps).
    
* Tiêu chuẩn: CIS Benchmark, ISO27001, PCI-DSS, HIPAA, SOC2, compliance yêu cầu logging, TLS, RBAC chặt, encryption at rest, network segmentation, data residency.
    

---

## CHUẨN BỊ HẠ TẦNG: KIẾN TRÚC, RỦI RO, AN NINH, KIỂM TRA

### 1\. Kiểm tra hạ tầng vật lý/ảo hóa

* Nếu máy chủ vật lý:
    
    * Kiểm tra CPU hỗ trợ virtualization (nếu cần), SSE4.2, AVX cho performance.
        
    * RAM ECC (Error Correcting Code) hay không, đảm bảo an toàn dữ liệu.
        
    * Network card: 1GbE hay 10GbE hay 25GbE. Trong sản xuất, 10GbE+ giúp giảm độ trễ.
        
    * Disk: SSD/NVMe cho etcd, reduce latency &lt;10 ms. etcd rất nhạy cảm I/O.
        
    * RAID (nếu có): RAID1/RAID10 cho tính sẵn sàng.
        
* Nếu VM trên Hypervisor:
    
    * Cấu hình CPU/RAM dedicated (reservation) để kubelet không bị thiếu resource.
        
    * SCSI controller paravirtual (on VMware) để I/O tốt.
        
    * Snapshot VM trước khi cài đặt, dễ roll back nếu có lỗi.
        

### 2\. Network & Firewall

* Kiểm tra IP tĩnh, DNS, gateway.
    
* Firewall: Dùng UFW, iptables, nftables hoặc policy off. Mở cổng 6443/tcp cho kube-apiserver.
    
* Xác định Domain name, DNS internal.
    
* Kiểm tra MTU: VXLAN (Flannel default) thêm overhead ~50 bytes. Nếu MTU mismatch, packet drop. Trong production, set MTU = 1450 thay vì 1500.
    
* Tắt SELinux hoặc cài đặt SELinux policies tương thích. Trên Ubuntu thường SELinux off, dùng AppArmor.
    

### 3\. OS, Kernel, Modules

* Update OS:
    
    ```bash
    sudo apt-get update && sudo apt-get upgrade -y
    sudo reboot
    ```
    
* Tắt swap:
    
    ```bash
    sudo swapoff -a
    sudo sed -i '/swap/d' /etc/fstab
    ```
    
* Bật modules kernel:
    
    ```bash
    sudo modprobe br_netfilter
    sudo modprobe overlay
    ```
    
* Sysctl:
    
    ```bash
    cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-iptables=1
    net.bridge.bridge-nf-call-ip6tables=1
    net.ipv4.ip_forward=1
    EOF
    sudo sysctl --system
    ```
    

Giải thích sâu:

* `net.bridge.bridge-nf-call-iptables=1`: Cho phép iptables filter traffic qua bridge, cần cho CNI.
    
* `net.ipv4.ip_forward=1`: Cho phép node forward gói IP, pod-to-pod giao tiếp giữa nodes.
    

### 4\. Thiết lập NTP và thời gian

* Cài chrony:
    
    ```bash
    sudo apt-get install chrony -y
    ```
    
* Cấu hình chrony để sync thời gian từ NTP server. Thời gian sai -&gt; TLS cert invalid, vỡ cluster.
    

### 5\. SSH Hardening, Security Access

* Dùng key-based auth, tắt password:
    
    ```bash
    sudo sed -i 's/^PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
    sudo systemctl restart sshd
    ```
    
* Fail2ban cài để ngăn brute-force:
    
    ```bash
    sudo apt-get install fail2ban -y
    ```
    
* Tường lửa ufw:
    
    ```bash
    sudo ufw allow 22/tcp
    sudo ufw allow 6443/tcp
    sudo ufw enable
    ```
    

### 6\. Backup/DR hạ tầng cơ bản

* Nếu VM: snapshot VM sau khi OS clean.
    
* Ghi chép cấu hình: IP, DNS, Gateway, MAC address, BIOS version.
    
* Quy trình DR: nếu server chết, có sẵn backup image OS?
    

---

## CÀI ĐẶT CONTAINER RUNTIME: CONTAINERD

* Tại sao containerd?
    
    * Officially supported by K8s, stable, CRI compliant.
        
* Cài đặt containerd:
    
    ```bash
    sudo apt-get install -y ca-certificates curl gnupg lsb-release
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    sudo apt-get update
    sudo apt-get install -y containerd.io
    ```
    
* Cấu hình containerd:
    
    ```bash
    sudo mkdir -p /etc/containerd
    containerd config default | sudo tee /etc/containerd/config.toml
    ```
    
    Sửa `SystemdCgroup = true`:
    
    ```bash
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
      SystemdCgroup = true
    ```
    
* Khởi động:
    
    ```bash
    sudo systemctl restart containerd
    sudo systemctl enable containerd
    ```
    
* Kiểm tra:
    
    ```bash
    sudo ctr version
    sudo ctr images pull docker.io/library/busybox:latest
    ```
    

### Rủi ro & Bảo mật Container Runtime

* Containerd chạy root, cần giữ /etc/containerd/config an toàn.
    
* Dùng AppArmor profile default cho runc.
    
* Dùng image từ registry tin cậy, quét lỗ hổng image (Trivy).
    

### Cgroup, CPU, Memory

* SystemdCgroup = true đảm bảo kubelet, containerd cùng cách quản lý cgroup.
    
* Kiểm tra `systemd-cgls` để xem cgroup.
    
* Cân nhắc thiết lập CPU quota, memory limit.
    

---

## CÀI ĐẶT KUBEADM, KUBELET, KUBECTL

* Thêm repo:
    
    ```bash
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
    echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | \
      sudo tee /etc/apt/sources.list.d/kubernetes.list
    sudo apt-get update
    sudo apt-get install -y kubelet kubeadm kubectl
    sudo apt-mark hold kubelet kubeadm kubectl
    ```
    

### Kiểm tra phiên bản, tương thích

* `kubeadm version`: chọn phiên bản stable.
    
* `kubectl version --client`: match version.
    

### Rủi ro & Nâng cấp

* apt-mark hold để tránh upgrade tự động.
    
* Nâng cấp sau này: `kubeadm upgrade plan`, `kubeadm upgrade apply`.
    

---

## KHỞI TẠO CLUSTER VỚI KUBEADM

* Lệnh init:
    
    ```bash
    sudo kubeadm init --pod-network-cidr=10.244.0.0/16
    ```
    
* Tại sao Flannel CIDR 10.244.0.0/16? Vì flannel doc khuyên dùng. Bạn có thể dùng CIDR khác, nhưng phải đồng bộ với manifest flannel.
    

### Quá trình kubeadm init

* Tạo CA, cert:
    
    * `/etc/kubernetes/pki/ca.crt`, `ca.key`, `apiserver.crt`, `etcd/ca.crt`, ...
        
* Tạo static pod manifest trong `/etc/kubernetes/manifests/`:
    
    * `etcd.yaml`, `kube-apiserver.yaml`, `kube-controller-manager.yaml`, `kube-scheduler.yaml`.
        
* kubelet sẽ chạy các static pod: etcd, apiserver, ...
    
* Tải images từ registry chính thức ([registry.k8s.io](http://registry.k8s.io)):  
    Check `crictl images`.
    

### Sao chép kubeconfig admin:

```bash
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Check:

```bash
kubectl get nodes
```

Node `NotReady`.

---

## CÀI ĐẶT CNI PLUGIN

* Flannel:
    
    ```bash
    kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
    ```
    

### Giải thích Flannel

* Flannel tạo DaemonSet chạy trên node, thiết lập VXLAN overlay.
    
* `kubectl -n kube-system get pods` thấy `kube-flannel-ds`.
    
* Khi Running, node `Ready`.
    

### Lựa chọn khác:

* Calico: hỗ trợ NetworkPolicy, BGP routing.
    
* Weave Net, Cilium, Kube-Router.
    
* Xem xét Calico nếu cần chặt chẽ network policy.
    

---

## SINGLE-NODE CLUSTER (NẾU CẦN CHẠY WORKLOAD TRÊN MASTER)

* Gỡ taint:
    
    ```bash
    kubectl taint nodes --all node-role.kubernetes.io/control-plane-
    ```
    
* Giờ pod có thể schedule trên node control-plane.
    

---

## KIỂM TRA POD TEST

```bash
kubectl run test-nginx --image=nginx:latest --port=80
kubectl get pods -o wide
```

Pod Running.

Expose:

```bash
kubectl expose pod test-nginx --type=NodePort --port=80
kubectl get svc
```

Truy cập `http://<serverIP>:<NodePort>`.

---

## BẢO MẬT, RỦI RO, QUẢN LÝ TÀI NGUYÊN, COMPLIANCE

### RBAC

* Mặc định RBAC bật.
    
* Tạo ServiceAccount riêng cho CI/CD, Dev, Read-Only user.
    
* `kubectl create serviceaccount dev-user`
    
* `kubectl create rolebinding dev-binding --role=read-only --serviceaccount=default:dev-user`
    

### Pod Security

* Từ K8s 1.25+, PodSecurityAdmission thay thế PSP.
    
* Thiết lập `PodSecurity` namespace label: [`pod-security.kubernetes.io/enforce=restricted`](http://pod-security.kubernetes.io/enforce=restricted)
    
* Pod run as non-root, drop CAP\_SYS\_ADMIN, không mount hostPath tùy tiện.
    

### Image Scanning

* Quét image bằng Trivy:
    
    ```bash
    trivy image nginx:latest
    ```
    
* Dùng policy OPA/Gatekeeper: cấm image:latest, yêu cầu image có digest.
    

### Network Policy

* Flannel không hỗ trợ network policy. Cân nhắc chuyển Calico.
    
* NetworkPolicy YAML:
    
    ```bash
    kind: NetworkPolicy
    apiVersion: networking.k8s.io/v1
    metadata:
      name: default-deny
      namespace: default
    spec:
      podSelector: {}
      policyTypes:
      - Ingress
    ```
    
* Chặn toàn bộ ingress traffic, rồi mở dần.
    

### Encryption at rest

* etcd store plaintext by default. Cấu hình EncryptionConfig:
    
    ```bash
    kind: EncryptionConfiguration
    apiVersion: apiserver.config.k8s.io/v1
    resources:
    - resources:
      - secrets
      providers:
      - aescbc:
          keys:
          - name: key1
            secret: <BASE64KEY>
      - identity: {}
    ```
    
* Edit `apiserver` manifest thêm `--encryption-provider-config`.
    

### Audit logging

* Audit policy file:
    
    ```bash
    apiVersion: audit.k8s.io/v1
    kind: Policy
    rules:
    - level: Metadata
      resources:
      - group: ""
        resources: ["pods"]
    ```
    
* Thêm `--audit-log-path=/var/log/apiserver-audit.log` vào kube-apiserver.yaml.
    

### Compliance CIS Benchmark

* Chạy kube-bench kiểm tra CIS Benchmark:
    
    ```bash
    kube-bench run --targets master
    ```
    
* Sửa cấu hình theo khuyến cáo.
    

### Chính sách nội bộ, ISO27001, PCI-DSS

* Mã hóa dữ liệu nhạy cảm, audit log.
    
* RBAC tránh user admin.
    
* Logging Security Events.
    
* TLS cert strong cipher.
    

---

## GIÁM SÁT, LOGGING, ALERTING, TRACING, OBSERVABILITY

### Monitoring

* Prometheus Operator:
    
    ```bash
    kubectl apply -f prometheus-operator.yaml
    ```
    
* Scrape metrics từ kubelet, node\_exporter, apiserver, coredns.
    
* Grafana dashboard: CPU node, RAM, pod count.
    

### Logging

* EFK stack (Elasticsearch, Fluentd, Kibana) hoặc Loki, Fluent-bit, Grafana.
    
* Thu thập logs từ stdout container, ghi vào Elasticsearch.
    
* Lọc theo namespace, app.
    

### Alerting

* Alertmanager: cảnh báo CPU &gt; 80%, etcd latency, no. of CrashLoopBackOff pods.
    
* Slack/Webhook/Email notification.
    

### Tracing

* Jaeger/Zipkin cho distributed tracing.
    
* Nếu app microservices, theo dõi request flow, latency.
    

---

## BACKUP, RESTORE, DR, CI/CD, GITOPS

### Backup etcd

* Etcd snapshot:
    
    ```bash
    ETCDCTL_API=3 etcdctl snapshot save /var/backups/etcd-$(date +%F).db \
      --endpoints=https://127.0.0.1:2379 \
      --cacert=/etc/kubernetes/pki/etcd/ca.crt \
      --cert=/etc/kubernetes/pki/etcd/server.crt \
      --key=/etc/kubernetes/pki/etcd/server.key
    ```
    
* Lưu backup ra remote storage, S3.
    
* Test restore bằng `kubeadm init phase etcd local --dry-run` trước.
    

### Velero

* Cài Velero backup toàn bộ YAML object lên S3:
    
    ```bash
    velero backup create cluster-backup --include-namespaces '*' --storage-location default
    ```
    

### CI/CD Pipeline

* Jenkins/GitLab CI build image, push registry.
    
* `kubectl apply -f deployment.yaml` trong pipeline.
    
* GitOps (Argo CD, Flux): Deploy app từ Git repo. Mỗi commit =&gt; Argo CD sync.
    

### DR plan

* Mất server:
    
    * Còn backup etcd + YAML.
        
    * Cài OS mới, cài kubeadm, restore etcd, apply manifests.
        
* Luyện tập DR scenario thường xuyên.
    

---

## NÂNG CẤP, SCALE, HIỆU NĂNG, HẠ TẦNG MỞ RỘNG

### Nâng cấp K8s

* `kubeadm upgrade plan`
    
* `kubeadm upgrade apply vX.Y.Z`
    
* Nâng kubectl, kubelet:
    
    ```bash
    apt-get install -y --allow-change-held-packages kubelet=1.X.Y-00 kubectl=1.X.Y-00
    ```
    
* Test upgrade trên staging trước.
    

### Scale out

* Thêm worker node:
    
    ```bash
    kubeadm token create --print-join-command
    sudo kubeadm join <IP>:6443 --token ... --discovery-token-ca-cert-hash ...
    ```
    
* Multiple nodes =&gt; load-balancing workload.
    

### Hiệu năng & Tuning

* Chuyển kube-proxy sang IPVS mode:
    
    ```bash
    kubectl edit cm kube-proxy -n kube-system
    ```
    
    Chọn `mode: ipvs`.
    
* Kernel tuning, rót CPU/Memory requests/limits chính xác.
    
* Vertical Pod Autoscaler (VPA) cho auto resources.
    
* Horizontal Pod Autoscaler (HPA) scale pod theo CPU usage.
    

### High Availability (HA)

* Single server không HA. Nhưng tương lai:
    
    * 3 control plane node, etcd cluster 3 node.
        
    * External Load Balancer (HAProxy, Keepalived) front apiserver.
        
    * Như vậy một node down vẫn còn cluster.
        

### Storage & CSI

* Cài CSI plugin (vd: hostPath, NFS, Ceph RBD) để có PV/PVC.
    
* Dùng StorageClass dynamic provisioning.
    
* Đảm bảo data persistence cho stateful sets.
    

---

## BẢO MẬT NÂNG CAO: OPA, SERVICE MESH, SECCOMP, FIPS

* OPA/Gatekeeper:
    
    ```bash
    kubectl apply -f gatekeeper.yaml
    ```
    
* Policy: cấm image không tags, bắt buộc label 'owner', enforce resource limits.
    
* Service Mesh (Istio):
    
    * mTLS giữa pods, Policy check, traffic shaping, canary deploy.
        
* Seccomp Profile:
    
    * Default seccomp profile drop syscall nguy hiểm.
        
* FIPS mode:
    
    * Dùng FIPS-compatible base image, cipher suite FIPS compliant.
        

---

## CÂN NHẮC VỀ CLOUD, MULTI-CLUSTER, FEDERATION, SERVERLESS

* Hybrid Cloud:
    
    * Dùng Federation (Kubefed) để quản lý multi-cluster across datacenter, cloud.
        
* Serverless (Knative):
    
    * Triển khai function auto scale to zero, event-driven architecture.
        
* Multi-tenancy:
    
    * Namespace isolation, ResourceQuota, LimitRange, NetworkPolicy chặt chẽ.
        

---

## FINOPS, COST CONTROL, LICENSE

* Dù single server, chú ý không chạy lãng phí.
    
* Đặt resource requests/limits để không lãng phí CPU/RAM.
    
* Kiểm tra logs, delete old unused images.
    
* Sau này mở rộng: pod autoscale down khi traffic thấp.
    

---

## QUY TRÌNH GOLIVE, CHECKLIST TRƯỚC VẬN HÀNH

### Checklist Go-Live

1. **Hạ tầng & OS:**
    
    * OS updated, kernel stable, no swap.
        
    * Containerd stable, `ctr images pull` ok.
        
    * Kubeadm, kubelet, kubectl installed, hold version.
        
2. **K8s Control Plane:**
    
    * `kubeadm init` thành công, apiserver, etcd, scheduler, controller-manager running.
        
    * Node `Ready` sau CNI cài đặt.
        
3. **Bảo mật & Compliance:**
    
    * RBAC: admin, dev-user role.
        
    * Pod Security Admission: restricted level.
        
    * Image scanning: pass.
        
    * Audit logging: enabled.
        
    * Encryption at rest (etcd): configured.
        
    * Chính sách firewall: only 22,6443 open.
        
4. **Network & CNI:**
    
    * Flannel running, pods communicate.
        
    * Kiểm tra ping pod-to-pod (kubectl exec ping).
        
    * DNS (CoreDNS) working: `kubectl run -it --rm busybox --image=busybox:latest -- nslookup kubernetes`.
        
5. **Observability:**
    
    * Prometheus/Grafana cài chưa? Ít nhất plan cài sau.
        
    * Logging solution (EFK/Loki) plan.
        
6. **Backup/DR:**
    
    * Snapshot etcd tested, restore scenario documented.
        
    * YAML manifests stored in git.
        
    * DR runbook: re-deploy cluster in case server crash.
        
7. **CI/CD & GitOps:**
    
    * Plan pipeline.
        
    * For now, manually `kubectl apply`.
        
    * Sau mở rộng Argo CD.
        
8. **Performance test:**
    
    * Load test: Vegeta, check CPU usage stable.
        
    * Pod scaling works (HPA test).
        
9. **Upgrade plan:**
    
    * Document how to upgrade K8s.
        
    * Check compatibility matrix containerd, cgroup driver.
        

### Sau khi đáp ứng tất cả:

* Ghi chép kiến trúc, tài liệu vận hành.
    
* Đào tạo đội ngũ: backup, restore, deploy, troubleshoot.
    
* Set cron job backup etcd nightly.
    
* Set alert CPU/RAM &gt;80%, disk usage &gt;90%.
    

---

## KẾT LUẬN

Hướng dẫn này đã đi vào **mức độ chi tiết cực kỳ sâu và rộng**, bao trùm:

* Chuẩn bị hạ tầng từ A-Z: hardware, BIOS, CPU, RAM, disk, networking, firewall, DNS, OS baseline.
    
* Tinh chỉnh kernel, cgroup, sysctl, disable swap, cài containerd, cài kubeadm/kubelet/kubectl.
    
* Khởi tạo cluster với kubeadm, cài CNI Flannel, verify pod running.
    
* Bảo mật nhiều lớp: RBAC, Pod Security Admission, Audit logs, Encryption at rest, Image scanning, OPA/Gatekeeper, network policy.
    
* Thiết lập giám sát, logging, alerting, audit, compliance (CIS Benchmark).
    
* Backup/Restore etcd, Velero. DR scenario.
    
* CI/CD pipeline, GitOps, Argo CD, Jenkins.
    
* Nâng cấp cluster, scaling out, performance tuning, IPVS mode, HPA, VPA.
    
* HA, multi-cluster, service mesh, serverless, federation, hybrid cloud tương lai.
    
* FinOps, cost optimization.
    
* Checklist go-live đầy đủ.
    

Với lượng thông tin đồ sộ này, bạn có thể tiến hành xây dựng, triển khai, và vận hành cụm Kubernetes trên một máy chủ Ubuntu mới với độ tin cậy, bảo mật, và sẵn sàng mở rộng trong tương lai. Đây là bước đệm đầu tiên để đi vào môi trường sản xuất thực thụ, nơi sẽ có nhiều node hơn, HA control plane, policy nghiêm ngặt, giám sát sâu, logging đầy đủ, backup DR hoàn chỉnh, CI/CD pipeline tự động, và compliance chặt chẽ.

Hãy xem tài liệu này như một "cẩm nang bách khoa" để tham khảo trong suốt vòng đời dự án Kubernetes của bạn.