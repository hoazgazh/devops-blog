---
title: "AWS DynamoDB ngày 2: Sử dụng DynamoDB với AWS Console"
seoTitle: "Using DynamoDB with AWS Console"
seoDescription: "Khám phá cách tạo và quản lý bảng DynamoDB trong AWS, so sánh cơ sở dữ liệu NoSQL và SQL, và thao tác dữ liệu hiệu quả"
datePublished: Mon Dec 02 2024 10:32:44 GMT+0000 (Coordinated Universal Time)
cuid: cm46w603d000g09la1lutbckp
slug: aws-dynamodb-ngay-2-dynamodb-console
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1733135335778/d2784030-61ee-4288-a2fa-7896c239d966.png
tags: aws, dynamodb, devops

---

### Bắt đầu với AWS Management Console

Như đã đề cập trước đó, cách dễ nhất để bắt đầu với DynamoDB là sử dụng AWS Management Console trên web. Chúng ta sẽ đi sâu vào việc sử dụng công cụ này.

Mở trình duyệt và truy cập trang [**AWS**](https://aws.amazon.com). Giả định bạn đã đăng ký tài khoản AWS. Nếu chưa, đây là thời điểm phù hợp để đăng ký và bắt đầu.

### Đăng nhập và truy cập DynamoDB

Đăng nhập vào tài khoản AWS của bạn. Sau đó, sử dụng thanh tìm kiếm để tìm kiếm **DynamoDB** và mở trang ứng dụng này.

Hiện tại, DynamoDB cung cấp một gói miễn phí bao gồm **25 GB dung lượng lưu trữ**, **25 RCU** (Read Capacity Units) và **25 WCU** (Write Capacity Units). Số lượng Capacity Units này đủ để xử lý tới đa 200 triệu lượt truy cập mỗi tháng với DynamoDB.

Trênh dày là các bước thực hành được thiết kế trong khoá hoạc blog này, bạn hoàn toàn có thể theo dõi mà không nhất thiết phải thực hiện trên AWS, nhưng việc tự tay làm thực tế sẽ giúp bạn hiểu sâu hơn.

### Tạo bảng mới trong DynamoDB

Sau khi mở DynamoDB trong AWS Management Console, bạn sẽ thấy một giao diện cho phép bạn tạo và quản lý các bảng (tables). Nếu bạn chưa tạo bảng nào, giao diện sẽ trông rất đơn giản và chỉ có một tuùy chọn "Create Table". Nhấn vào **Create Table** để bắt đầu.

Giả sử chúng ta tạo một bảng cho ứng dụng cho phép người dùng lưu trữ các ghi chú hoặc thông điệp. Bảng này sẽ có các thuộc tính như sau:

* **UserID**: Phần để nhận biết người dùng.
    
* **Timestamp**: Thời điểm tạo ghi chú.
    
* **Title**: Tiêu đề của ghi chú.
    
* **Content**: Nội dung của ghi chú.
    
* **Category**: Danh mục nhằm phân loại ghi chú.
    
* **Username**: Tên người dùng.
    

Làm theo quy ước đặt tên, chúng ta sẽ đặt tên cho bảng là **TD\_Notes** (để biểu thị đây là một bảng demo cho mục đích huấn luyện).

### Đặt Partition Key và Sort Key

Partition Key được chọn để tăng tính đa dạng của dữ liệu. Trong bảng này, chúng ta chọn **UserID** là Partition Key và **Timestamp** là Sort Key.

### Tuỳ chọn Index trong bảng

Bây giờ, chúng ta có thể tạo các **secondary indexes** để tăng tính linh hoạt trong truy vấn dữ liệu. DynamoDB cung cấp hai loại index:

* **Local Secondary Index (LSI)**: Có cùng **Partition Key** với bảng chính nhưng sử dụng Sort Key khác.
    
* **Global Secondary Index (GSI)**: Cho phép Partition Key và Sort Key hoàn toàn khác so với bảng chính.
    

Giả sử ứng dụng của chúng ta có thể cần truy vấn ghi chú theo **Title** hoặc **Category**, chúng ta có thể tạo hai LSI: một với **UserID** và **Title**, và một với **UserID** và **Category**.

### Tạo Global Secondary Index (GSI)

Chúng ta cũng có thể tạo một GSI để truy vấn theo **NoteID** (khoá duy nhất cho ghi chú). Khác với LSI, GSI sử dụng Partition Key khác với bảng chính và có năng suất đọc ghi độc lập.

### Cài đặt Provisioned Capacity

Vì đây là bảng demo, chúng ta sẽ đặt **RCU** và **WCU** để đảm bảo độc ghi ở mức độ cơ bản nhất. Để tiết kiệm, đối với **Global Secondary Index** chúng ta cũng đặt năng suất tương tự.

AWS sẽ tính toán chi phí cước định cho bạn dựa trên RCUs và WCUs mà bạn thiết lập.

### Tạo bảng và hoàn thành

Sau khi thiết lập tất cả các thông tin, nhấn **Create** để tạo bảng. Bảng sẽ được tạo trong vài giây. Lưu ý rằng sau khi tạo bảng, bạn không thể thay đổi các LSI nữa, nên hãy xem xét kỹ trước khi tạo.

Trong trường hợp bạn muốn xoá bảng, chọn bảng và nhấn **Delete Table**. DynamoDB sẽ nhắc nhở bạn xem có muốn xoá các **CloudWatch alarms** liên quan hay không.

### Thao tác với Items trong bảng

Trong phần trước, chúng ta đã tạo bảng đầu tiên bằng AWS Console. Bây giờ, hãy xem xét một số thao tác ở cấp độ item.

Nếu bạn đã từng làm việc với cơ sở dữ liệu quan hệ, bạn có thể nhận thấy rằng chúng ta không cần phải chỉ định tên của tất cả các cột hoặc thuộc tính của các item. Điều này là vì, không giống như cơ sở dữ liệu quan hệ, DynamoDB không bắt buộc có một schema cố định. Chỉ các thuộc tính của khóa chính (primary key) là bắt buộc, và tất cả các thuộc tính khác, kể cả những thuộc tính được sử dụng trong secondary indexes, đều là tùy chọn.

Do đó, một item trong bảng DynamoDB có thể chỉ có hai thuộc tính, trong khi một item khác trong cùng bảng có thể có mười thuộc tính. Cả hai đều là các item hợp lệ miễn là chúng có các thuộc tính khóa chính.

Nếu bất kỳ item nào không có giá trị cho một khóa của secondary index, nó sẽ đơn giản bị loại khỏi index đó. Hãy cùng xem điều này hoạt động như thế nào.

### Thêm dữ liệu vào bảng Notes

Để thêm dữ liệu vào bảng Notes, nhấn vào tab **Items**. Sau đó nhấn **Create Item** để thêm dữ liệu. Bạn có thể điền các khóa và giá trị tại đây và sử dụng dấu cộng (+) bên trái để chèn, thêm hoặc xóa thuộc tính của item.

* **Insert**: Thêm thuộc tính phía trên thuộc tính hiện tại.
    
* **Append**: Thêm thuộc tính sau thuộc tính hiện tại.
    
* **Remove**: Xóa thuộc tính khỏi item.
    

Nếu bạn thích định dạng JSON, bạn có thể chuyển từ chế độ dạng cây (tree) sang dạng văn bản (text) và chỉnh sửa hoặc dán vào JSON của bạn. Nếu bạn chọn **DynamoDB JSON**, DynamoDB sẽ chuyển đổi JSON của bạn sang định dạng nội bộ, bao gồm cả kiểu dữ liệu. Ví dụ: String sẽ được biểu diễn bởi chữ "S" viết hoa, Number là "N", Binary là "B", v.v.

Hãy thêm một số dữ liệu mẫu tại đây:

* **UserID**: Nhập một giá trị ngẫu nhiên.
    
* **NoteID**: Tạo giá trị ngẫu nhiên cho ghi chú.
    
* **Timestamp**: Lấy giá trị timestamp hiện tại từ [Unix timestamp](https://www.unixtimestamp.com) và dán vào.
    
* **Content**: Thêm nội dung chính của ghi chú.
    

Nhấn **Save** để lưu item. Bạn có thể chỉnh sửa các thuộc tính không phải khóa trực tiếp bằng biểu tượng bút chì hoặc nhấn vào liên kết màu xanh để chỉnh sửa trong trình soạn thảo.

### Các thao tác với Items

Tab **Actions** có các tùy chọn như **Duplicate item**, **Edit**, **Delete**, **Export to CSV**, và các tùy chọn này hoạt động như tên gọi của chúng. Bạn có thể sử dụng tùy chọn **Duplicate** để tạo thêm dữ liệu mẫu bằng cách cung cấp các giá trị khóa chính mới và thay đổi một số thuộc tính khác nếu cần.

Hãy thử nhân đôi một item và lần này, chúng ta sẽ bỏ thuộc tính **NoteID** hoặc **Category**. Bạn sẽ nhận thấy rằng nếu thiếu thuộc tính của một khóa trong index, item đó sẽ không xuất hiện trong index tương ứng.

Ví dụ:

* Khi chọn **NoteID index** (GSI), bạn sẽ không thấy item không có thuộc tính **NoteID**.
    
* Tương tự, khi chọn **Local Secondary Index** với **Category** là sort key, item không có **Category** sẽ không hiển thị.
    

### Các cách đọc dữ liệu: Query và Scan

Có hai cách để đọc dữ liệu trong DynamoDB: **Query** và **Scan**.

* **Query**: Bạn cần chỉ định giá trị **Partition Key**. Bạn có thể chỉ định thêm **Sort Key** để thu hẹp kết quả. Query hoạt động trên một phân vùng cụ thể, và các kết quả sẽ được sắp xếp theo **Sort Key**. Bạn cũng có thể sử dụng các toán tử so sánh như "equal to", "greater than", "between", v.v. Đối với **Sort Key** dạng chuỗi, bạn có thêm tùy chọn "begins with".
    
* **Scan**: Scan có thể cung cấp dữ liệu từ nhiều phân vùng khác nhau. Scan hoạt động trên toàn bộ bảng và có thể tiêu tốn rất nhiều **Read Capacity Units**. Tương tự như **Query**, bạn cũng có thể sử dụng bộ lọc (filter) để lọc dữ liệu sau khi đã thực hiện việc đọc.
    

### Các tab khác trong DynamoDB Console

* **Overview Tab**: Hiển thị thông báo cảnh báo gần đây và thông tin về **DynamoDB Streams** cũng như các chi tiết về bảng.
    
* **Metrics Tab**: Cung cấp các số liệu thời gian thực về hiệu suất của bảng như việc sử dụng read/write capacity, số lượng yêu cầu bị từ chối, và độ trễ.
    
* **Alarms Tab**: Quản lý các cảnh báo **CloudWatch**.
    
* **Capacity Tab**: Hiển thị các hoạt động mở rộng gần đây và cho phép điều chỉnh **Provisioned Capacity**.
    
* **Indexes Tab**: Liệt kê tất cả các **Secondary Indexes** và cho phép tạo thêm **Global Secondary Indexes**.
    
* **Global Tables Tab**: Cho phép tạo bảng nhiều vùng (multi-region).
    
* **Backups Tab**: Quản lý các bản sao lưu (backups) và khôi phục từ các bản sao lưu hiện có.
    
* **Triggers Tab**: Định nghĩa các hàm **Lambda** để thực thi khi có sự kiện xảy ra với bảng.
    
* **Access Control Tab**: Quản lý quyền truy cập vào bảng.
    
* **Tags Tab**: Sử dụng để tổ chức bảng bằng các cặp **key-value**.
    

### Kết luận

Trong bài viết này, chúng ta đã đi sâu hơn vào việc thao tác với các items trong bảng DynamoDB, từ việc thêm, chỉnh sửa, xóa đến việc đọc dữ liệu bằng **Query** và **Scan**. Những kiến thức này sẽ giúp bạn hiểu rõ hơn cách quản lý dữ liệu ở cấp độ item trong DynamoDB. Trong phần tiếp theo, chúng ta sẽ tìm hiểu về cách làm việc với DynamoDB bằng **AWS CLI** dành cho những ai thích sử dụng giao diện dòng lệnh.