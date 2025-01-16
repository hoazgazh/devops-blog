---
title: "Chiến lược multi-account AWS cho digital bank"
seoTitle: "AWS Multi-Account Strategies for Digital Banks"
seoDescription: "Hướng dẫn chi tiết triển khai chiến lược multi-account AWS cho ngân hàng số, tối ưu bảo mật, quản trị và khả năng mở rộng công nghệ"
datePublished: Thu Jan 16 2025 09:45:11 GMT+0000 (Coordinated Universal Time)
cuid: cm5z5a6qt000b0ajp8mezc19m
slug: chien-luoc-multi-account-aws-cho-digital-bank
canonical: https://aws.amazon.com/vi/blogs/industries/defining-an-aws-multi-account-strategy-for-a-digital-bank/
tags: aws, digital-bank

---

## Giới thiệu

Trong những năm gần đây, lĩnh vực ngân hàng toàn cầu đang trải qua sự chuyển đổi đáng kể. Khách hàng ngày càng đòi hỏi trải nghiệm ngân hàng hoàn toàn số hóa, cá nhân hóa và luôn sẵn sàng. Những mong đợi thay đổi này, kết hợp với sự tiến bộ của công nghệ và sự hỗ trợ từ các quy định mở như PSD2 tại EU và giấy phép ngân hàng số tại khu vực Châu Á Thái Bình Dương, đã tạo nên một khung pháp lý thúc đẩy sự phát triển của ngân hàng số.

Không chỉ các ngân hàng truyền thống như Standard Chartered với Mox, Goldman Sachs với Marcus, hay MSB tại Việt Nam với TNEX đang xây dựng các nhánh ngân hàng số, mà còn có các ngân hàng mới nổi như GrabBank, N26 và NuBank. Theo ước tính của Boston Consulting Group (BCG), hiện có khoảng 249 ngân hàng số trên toàn cầu. Bạn có thể tìm hiểu thêm về việc xây dựng ngân hàng số trên AWS tại [trang web digital banking AWS](https://aws.amazon.com/vi/solutions/financial-services/digital-bank-launch/?marketplace-ppa-and-quickstart.sort-by=item.additionalFields.sortDate&marketplace-ppa-and-quickstart.sort-order=desc).

Trong bài viết này, sẽ tập trung vào các ngân hàng số muốn xây dựng hạ tầng công nghệ từ đầu, đảm bảo tính bảo mật, độ bền, tuân thủ quy định nhưng vẫn linh hoạt và có khả năng mở rộng. Các nguyên tắc được trình bày sẽ hữu ích cho cả DevOps và Data Engineers khi triển khai và quản lý hạ tầng AWS cho ngân hàng số.

## Tại sao nên triển khai môi trường AWS multi-account?

Nhiều tổ chức dịch vụ tài chính đối mặt với thách thức khi các đội ngũ có quy trình kinh doanh khác nhau cần được cách ly với các khối lượng công việc khác và có các yêu cầu về bảo mật và kiểm soát mạng khác nhau. Với mỗi AWS Account, các tổ chức có thể đạt được mức độ cách ly cao nhất cho các khối lượng công việc của mình. Sử dụng nhiều AWS Accounts đóng vai trò quan trọng trong việc đáp ứng các yêu cầu về kinh doanh, quản trị, bảo mật và vận hành.

### Lợi ích của việc triển khai môi trường AWS multi-account:

1. **Rapid Innovation với Các Yêu Cầu Đa Dạng**: Bạn có thể phân bổ các AWS Accounts cho các đội ngũ, dự án hoặc sản phẩm khác nhau trong công ty, đảm bảo mỗi nhóm có thể đổi mới nhanh chóng đồng thời đáp ứng các yêu cầu bảo mật riêng.
    
2. **Simplified Billing**: Sử dụng nhiều AWS Accounts giúp đơn giản hóa việc phân bổ chi phí AWS, giúp dễ dàng xác định sản phẩm hoặc dịch vụ nào chịu trách nhiệm cho mỗi khoản chi.
    
3. **Flexible Security Controls**: Bạn có thể sử dụng nhiều AWS Accounts để cách ly các khối lượng công việc hoặc ứng dụng có yêu cầu bảo mật đặc biệt hoặc cần tuân thủ nghiêm ngặt các quy định như HIPAA hoặc PCI.
    
4. **Easily Adapts to Business Processes**: Bạn có thể tổ chức nhiều AWS Accounts theo cách phản ánh tốt nhất các nhu cầu đa dạng của quy trình kinh doanh công ty, có các yêu cầu vận hành, quy định và ngân sách khác nhau.
    

Để tìm hiểu thêm, bạn có thể tham khảo tài liệu ["Organizing Your AWS Environment"](https://aws.amazon.com/architecture/multi-account-setup/) chia sẻ hướng dẫn về kiến trúc nhiều tài khoản. Chúng ta sẽ sử dụng hướng dẫn trong tài liệu này để định nghĩa chiến lược multi-account của một ngân hàng số. Bạn cũng có thể xem lại các khái niệm cơ bản về **AWS Organizations** bao gồm **Organization**, **Root**, **Account** và **Organizational Unit (OU)** - những khái niệm quan trọng để hiểu bài viết này.

## Triển khai digital bank multi-account

Một **Organizational Unit (OU)** là một chứa các tài khoản trong một **Root**. OU cũng có thể chứa các OU khác, cho phép bạn tạo ra một cấu trúc phân cấp giống như một cây ngược. Hãy xem xét các nhóm tài khoản hoặc OU mà bạn có thể tạo cho ngân hàng số. Một OU nên dựa trên chức năng hoặc tập hợp các kiểm soát chung thay vì phản chiếu cấu trúc báo cáo của công ty. AWS khuyến nghị bạn nên bắt đầu với bảo mật và hạ tầng trong tâm trí. Hầu hết các doanh nghiệp có các đội ngũ trung tâm cung cấp dịch vụ cho toàn bộ tổ chức. Do đó, AWS khuyến nghị tạo một bộ các OU nền tảng cho các chức năng cụ thể này. Sơ đồ dưới đây cho thấy cấu trúc OU được khuyến nghị cho một ngân hàng số.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1736997025628/0b9415ab-6329-4c75-b7d5-042cdef0ed35.png align="center")

Thêm hai **Workload OUs**, `Banking Applications` và `Data and Analytics`, vào các OU **Infrastructure**, **Security**, **Deployment**, **Policy Staging**, **Sandbox** và **Suspended** đã được khuyến nghị trong tài liệu "Organizing Your AWS Environment Using Multiple Accounts".

### Các OU Chính:

1. **OU Banking Applications**: Dùng để chạy các ứng dụng ngân hàng. Bao gồm các tài khoản cho các khối lượng công việc như **Core Banking**, **Payments**, **Onboarding**, **Credit Decisioning** và **Regulatory Reporting**. OU này có thể được chia nhỏ thành **Retail OU** và **Corporate OU** dựa trên sở thích kinh doanh của ngân hàng.
    
2. **OU Data & Analytics**: Cung cấp bảo mật, quản trị và kiểm soát chi phí cho các chức năng phân tích. Bao gồm các tài khoản cho **Data and Analytics** cùng với các khối lượng công việc **Machine Learning**.
    

## Cấu trúc account AWS cho digital bank

Sau khi xác định cấu trúc OU, chúng ta xem xét các tài khoản AWS sẽ là một phần của mỗi OU. Sơ đồ dưới đây cho thấy các tài khoản AWS nền tảng được khuyến nghị để thiết lập một ngân hàng số. Tuy nhiên, điều này có thể thay đổi dựa trên các dịch vụ mà ngân hàng số cung cấp.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1736997053830/0bf3b4dc-01f9-4971-978c-aaf8746626b4.png align="center")

### Các account trong OU Banking Applications:

Hãy xem xét các lớp chức năng cấp cao của một ngân hàng số giúp chúng ta xác định các tài khoản AWS cần thiết trong OU Banking Applications:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1736997075792/2ad3eff8-3ee5-4033-a533-0b8e58fdee8f.png align="center")

**Lớp Sản Phẩm Cốt Lõi**: Chứa các sản phẩm cốt lõi như **Core Banking**, được trừu tượng hóa bằng các **Microservices** tùy chỉnh, sau đó được tiêu thụ bởi lớp **Experience APIs**.

**Lớp Trải Nghiệm**: **APIs** được xây dựng đặc biệt cho giao diện người dùng, được lưu trữ bằng các **Digital Channels** và ứng dụng **Customer Engagement**.

**Quản Lý Sự Kiện**: Các thay đổi trong vòng đời khách hàng và ngân hàng cốt lõi sẽ phát ra các sự kiện tới **Event Hub**, và các **Subscribers** của các sự kiện này có thể điều phối các hành động cần thiết.

### Các account đề xuất:

1. **Core Products Account**: Chứa các sản phẩm cốt lõi như **Core Banking**, **Credit Decisioning**, **Payments System**, **Card Management**, **Risk and Compliance Systems**, **Accounting Software**, **Treasury**, v.v. Để tăng cường bảo mật và cách ly, khuyến nghị có các AWS Accounts riêng biệt cho các hệ thống cốt lõi này.
    
2. **Core Microservices Account**: Chứa các **Microservices** để tạo lớp trừu tượng và tổng hợp trên các sản phẩm cốt lõi, sau đó được phơi bày tới lớp **Experience API**. Sử dụng các dịch vụ AWS như **Amazon API Gateway**, **AWS Fargate**, **Amazon EKS** và **Amazon ECS** để xây dựng các microservices dựa trên container với các cơ sở dữ liệu như **Amazon DynamoDB** và **Amazon RDS**.
    
3. **API Management Layer Account**: **Experience APIs** sẽ được phơi bày cho người tiêu dùng cuối thông qua lớp này, bao gồm các khả năng quản lý API như **Rate Limiting**, **Throttling** và bảo vệ API bằng **AWS WAF** và **AWS Shield**.
    
4. **Digital Channel Application Account**: Chứa các ứng dụng web và di động cung cấp trải nghiệm ngân hàng cho khách hàng và đối tác, sử dụng các dịch vụ như **AWS Amplify**, **Amazon S3** và **Amazon EC2**.
    
5. **Customer Engagement Account**: Bao gồm các chức năng hỗ trợ khách hàng đa kênh, sản phẩm **CRM** và quản lý chiến dịch, sử dụng các dịch vụ như **Amazon Connect**, **Amazon Pinpoint** và **Amazon Lex**.
    
6. **Events Management Account**: Xử lý kiến trúc hướng sự kiện để mở rộng và cung cấp khả năng xử lý gần thời gian thực, sử dụng các dịch vụ như **Amazon MSK**, **Amazon Kinesis** và **Amazon SNS**.
    
7. **Open Banking Account**: Chứa các **APIs** để đáp ứng yêu cầu quy định quốc gia, tích hợp với các tài khoản microservices và chứa quản lý sự đồng thuận, môi trường sandbox và cổng thông tin dành cho đối tác.
    
8. **Data Bunker Account**: Chứa các bản sao lưu dữ liệu, mẫu **AWS CloudFormation** và **Amazon Machine Images (AMI)** được mã hóa tại chỗ và trong quá trình truyền tải. Quyền truy cập tối thiểu được cấp cho đội ngũ bảo mật của ngân hàng số.
    

**Lưu Ý:** Tất cả các tài khoản nên tuân thủ các **AWS Security Best Practices**. Cấu trúc tương tự có thể được áp dụng cho các môi trường **Development** và **Test** dưới OU Banking Applications, với nhiều môi trường test như **User Acceptance Testing (UAT)** và **System Integration Tests (SIT)**.

### Các account trong OU Data & Analytics:

Ngân hàng số đang đổi mới và cung cấp các dịch vụ cá nhân hóa cho khách hàng, tìm kiếm cơ hội bán chéo và cung cấp dịch vụ khách hàng tốt hơn. Để đạt được tất cả điều này, ngân hàng số cần một nền tảng phân tích bảo mật, linh hoạt và có khả năng mở rộng.

Một kiến trúc multi-account cho **OU Data & Analytics** hỗ trợ kiến trúc **Lake House**, cung cấp dữ liệu ở quy mô lớn, hiệu suất giá tốt nhất, truy cập dữ liệu liền mạch để giải quyết các thách thức kinh doanh và quản trị thống nhất. Khuyến nghị ba tài khoản để bắt đầu với cấu hình dễ thiết lập có thể mở rộng khi môi trường phát triển.

1. **Data Ingestion Account**: Kết nối dữ liệu từ nhiều nguồn vào tài khoản lưu trữ **Lake House** sử dụng các dịch vụ như **Amazon S3**, **Amazon Kinesis**, và **AWS DataSync**.
    
2. **Data Lake House Storage Account**: Cung cấp các thành phần lưu trữ bền vững, có khả năng mở rộng và chi phí hiệu quả để quản lý lượng lớn dữ liệu, sử dụng **Amazon S3**, **Amazon Redshift** và **AWS Glue**.
    
3. **Data Consumption Account**: Cung cấp các thành phần có khả năng mở rộng và hiệu suất cao để truy cập dữ liệu từ **Lake House**, sử dụng **Amazon SageMaker Studio**, **Amazon QuickSight** và **Amazon Athena**.
    

**Lưu Ý:** Cấu trúc tương tự có thể được áp dụng cho các môi trường **Development** và **Test** dưới OU Data & Analytics.

## AWS có thể giúp bạn thiết lập như thế nào?

Việc tạo và quản lý cấu trúc multi-account đòi hỏi các kiểm soát bảo mật, quản trị và chi phí cùng với các bước tự động hóa phù hợp. **AWS Control Tower** cung cấp cách dễ dàng nhất để thiết lập và quản trị một môi trường AWS multi-account an toàn, được gọi là **Landing Zone**. AWS Control Tower tạo **Landing Zone** của bạn bằng cách sử dụng **AWS Organizations**, mang lại quản lý tài khoản liên tục và quản trị cũng như triển khai các **Best Practices** dựa trên kinh nghiệm làm việc với hàng ngàn khách hàng khi họ chuyển sang đám mây.

Bạn có thể triển khai **AWS Control Tower**, mở rộng quản trị vào các tài khoản mới hoặc hiện có, và nhanh chóng có được cái nhìn tổng quan về trạng thái tuân thủ. Bạn có thể tự làm hoặc tận dụng **AWS Professional Services**, hoặc chọn từ một trong các đối tác **AWS Partner Network (APN)**.