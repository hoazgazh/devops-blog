---
title: "Đổi Tên CloudFormation Stack Trên AWS mà Không Xóa Tài Nguyên"
seoTitle: "Rename AWS CloudFormation Stack Without Deletion"
seoDescription: "Hướng dẫn đổi tên AWS CloudFormation stack mà không xóa tài nguyên, sử dụng chính sách Retain và chức năng import"
datePublished: Sat Jan 11 2025 16:59:26 GMT+0000 (Coordinated Universal Time)
cuid: cm5sfldqg000009l68grx4hvq
slug: doi-ten-cloudformation-stack-tren-aws-ma-khong-xoa-tai-nguyen
tags: aws

---

Trong môi trường DevOps, việc quản lý hạ tầng dưới dạng mã (Infrastructure as Code - IaC) là một phần quan trọng để đảm bảo tính nhất quán và khả năng tái sử dụng. AWS CloudFormation là một dịch vụ mạnh mẽ giúp tự động hóa việc triển khai và quản lý các tài nguyên AWS. Tuy nhiên, đôi khi bạn có thể gặp tình huống cần thay đổi tên của một stack CloudFormation mà không muốn xóa các tài nguyên đã được tạo ra bởi stack đó. Bài viết này sẽ hướng dẫn bạn cách thực hiện điều này một cách an toàn và hiệu quả.

## Tình Huống Thực Tế

Giả sử bạn có một CloudFormation stack gồm các tài nguyên sau:

* **Amazon Simple Storage Service (Amazon S3) bucket**
    
* **Amazon Elastic Compute Cloud (Amazon EC2) instance**
    
* **Amazon Elastic Block Store (Amazon EBS) Volume**
    

Do một vấn đề bảo mật quan trọng, nhóm DevOps được yêu cầu đổi tên stack CloudFormation này. Tuy nhiên, các tài nguyên đã được tạo bởi stack không thể bị xóa vì lý do kinh doanh. Vậy làm thế nào để đổi tên stack mà không mất các tài nguyên?

## Giải Pháp

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1736614822893/431fe8af-be06-442e-815a-c75429ad7739.jpeg align="center")

Để đổi tên một CloudFormation stack mà không xóa các tài nguyên, bạn có thể sử dụng thuộc tính `DeletionPolicy` với giá trị `Retain`. Dưới đây là các bước chi tiết để thực hiện:

### Bước 1: Triển Khai Stack Mới với Chính Sách Retain

1. **Tạo Stack Mới:**
    
    * Tạo một stack CloudFormation mới với cấu hình giống hệt stack hiện tại nhưng sử dụng tên mới.
        
2. **Thêm Thuộc Tính Retain:**
    
    * Trong template của stack mới, thêm thuộc tính `DeletionPolicy` với giá trị `Retain` cho tất cả các tài nguyên mà bạn không muốn bị xóa khi stack bị xóa.
        
    
    ```yaml
    Resources:
      MyS3Bucket:
        Type: AWS::S3::Bucket
        DeletionPolicy: Retain
      MyEC2Instance:
        Type: AWS::EC2::Instance
        DeletionPolicy: Retain
      MyEBSVolume:
        Type: AWS::EC2::Volume
        DeletionPolicy: Retain
    ```
    

### Bước 2: Xóa Stack Gốc

1. **Xóa Stack Cũ:**
    
    * Sau khi stack mới đã được triển khai và tất cả tài nguyên đã được chuyển sang stack mới với chính sách `Retain`, bạn có thể xóa stack gốc.
        
    * Vì đã thiết lập `DeletionPolicy` là `Retain`, các tài nguyên sẽ không bị xóa khi stack gốc bị xóa.
        

### Bước 3: Nhập Các Tài Nguyên Vào Stack Mới

1. **Tạo Stack Mới Với Tên Mới:**
    
    * Tạo một stack mới với tên mong muốn và sử dụng chức năng nhập (import) để đưa các tài nguyên đã được giữ lại từ stack gốc vào stack mới.
        
2. **Cấu Hình Import:**
    
    * Sử dụng AWS CloudFormation để nhập các tài nguyên vào stack mới. Bạn có thể tham khảo [hướng dẫn import tài nguyên vào stack CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/resource-import.html) để thực hiện các bước chi tiết.
        

### Bước 4: Loại Bỏ Thuộc Tính Retain

1. **Cập Nhật Template:**
    
    * Sau khi các tài nguyên đã được nhập vào stack mới, cập nhật template CloudFormation để loại bỏ thuộc tính `DeletionPolicy: Retain`.
        
2. **Cập Nhật Stack:**
    
    * Triển khai cập nhật stack để đảm bảo rằng `DeletionPolicy` không còn được áp dụng, giúp quản lý tài nguyên chính xác theo template.
        

## Ví Dụ Thực Tế

Giả sử bạn có một stack CloudFormation có tên `OldStackName` với các tài nguyên như sau:

```yaml
Resources:
  MyS3Bucket:
    Type: AWS::S3::Bucket
  MyEC2Instance:
    Type: AWS::EC2::Instance
  MyEBSVolume:
    Type: AWS::EC2::Volume
```

### Bước 1: Tạo Stack Mới với Chính Sách Retain

Tạo một template mới `new-template.yaml`:

```yaml
Resources:
  MyS3Bucket:
    Type: AWS::S3::Bucket
    DeletionPolicy: Retain
  MyEC2Instance:
    Type: AWS::EC2::Instance
    DeletionPolicy: Retain
  MyEBSVolume:
    Type: AWS::EC2::Volume
    DeletionPolicy: Retain
```

Triển khai stack mới với tên `NewStackName`:

```bash
aws cloudformation create-stack --stack-name NewStackName --template-body file://new-template.yaml
```

### Bước 2: Xóa Stack Gốc

Xóa stack cũ mà không xóa tài nguyên:

```bash
aws cloudformation delete-stack --stack-name OldStackName
```

### Bước 3: Nhập Các Tài Nguyên Vào Stack Mới

Sử dụng chức năng import của CloudFormation để nhập các tài nguyên vào stack `NewStackName`. Bạn cần định nghĩa các tài nguyên để import trong template và sử dụng lệnh tương ứng.

### Bước 4: Loại Bỏ Thuộc Tính Retain

Cập nhật template `new-template.yaml` để loại bỏ `DeletionPolicy`:

```yaml
Resources:
  MyS3Bucket:
    Type: AWS::S3::Bucket
  MyEC2Instance:
    Type: AWS::EC2::Instance
  MyEBSVolume:
    Type: AWS::EC2::Volume
```

Triển khai cập nhật stack:

```bash
aws cloudformation update-stack --stack-name NewStackName --template-body file://new-template.yaml
```