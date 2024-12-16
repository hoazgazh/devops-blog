---
title: "Những thay đổi cần thiết về văn hoá để chuyển đổi kiến Trúc Cloud-Native"
seoTitle: "Những Thay Đổi Cần Thiết cho Cloud-Native"
seoDescription: "Thay đổi văn hóa, tổ chức và DevOps cùng triển khai liên tục là chìa khóa cho chuyển đổi kiến trúc cloud-native"
datePublished: Tue Dec 03 2024 14:11:55 GMT+0000 (Coordinated Universal Time)
cuid: cm48jfpy2000009jses5obwwj
slug: nhung-thay-doi-can-thiet-de-chuyen-doi-kien-truc-cloud-native
tags: devops, cloud-native

---

Trong làn sóng chuyển đổi sang kiến trúc ứng dụng cloud-native, một trong những thay đổi lớn nhất không phải là vấn đề kỹ thuật. Thay vào đó, chính là những thay đổi văn hóa và tổ chức, cách doanh nghiệp tạo nên môi trường giảm thiểu các quy trình và hoạt động không có giá trị, cải thiện hiệu suất. Đó là yếu tố then chốt giúp tổ chức thành công trong việc chuyển sang kiến trúc cloud-native.

Trong bài viết này, chúng ta sẽ tìm hiểu các thay đổi cần thiết về văn hóa.

## **Từ mô hình Silo chuyển sang DevOps: chuyển đổi để tăng tốc độ và hiệu quả**

Trong nhiều doanh nghiệp IT truyền thống, các đội ngũ thường được tổ chức theo các silo riêng biệt, bao gồm:

* Phát triển phần mềm
    
* Đảm bảo chất lượng (QA)
    
* Quản trị cơ sở dữ liệu (DBA)
    
* Quản trị hệ thống
    
* Vận hành IT
    
* Quản lý phát hành
    
* Quản lý dự án
    

Các silo này được tạo ra nhằm tối ưu hoá công việc chuyên biệt và dễ dàng quản lý nhân viên. Tuy nhiên, mô hình silo thường dẫn đến những khác biệt trong cách quản lý, công cụ, giao tiếp, và quan điểm mục tiêu của IT doanh nghiệp.

**Mâu thuẫn giữa nhóm phát triển và nhóm vận hành IT**

Một mâu thuẫn điển hình là sự khác biệt trong quan điểm giữa nhóm phát triển phần mềm và nhóm vận hành IT. Nhóm phát triển tập trung vào việc đem lại giá trị thông qua việc phát triển các tính năng mới, nghiã là đem lại sự thay đổi. Họ được khuyến khích dựa trên mức độ thay đổi mà họ mang lại.

Trái lại, nhóm vận hành IT thường được giao nhiệm vụ giữ cho hệ thống luôn ổn định, bền vững và có hiệu suất cao. Sự thay đổi là yếu tố tiềm tàng nguy cơ, do đó, họ có xu hướng giảm thiểu sự thay đổi nhằm duy trì các chỉ số hiệu suất chính (KPI) như thời gian trung bình giữa các lần hỏng hóc (MTBF) và thời gian trung bình để phục hồi (MTTR).

Kết quả, nhóm phát triển muốn thay đổi nhanh, trong khi nhóm vận hành muốn giữ đồn bền. Sự khác biệt này thường dẫn đến sự hợp tác không hiệu quả, giao tiếp không đồng bộ, và tạo ra những quy trình nặng nề và chậm chọap.

**Tại sao DevOps là giải pháp?**

DevOps được sinh ra để giải quyết vấn đề này. Thay vì chia các nhóm theo silo riêng biệt, DevOps kết nối các nhóm phát triển và vận hành thành một đội ngũ chung. Họ sử dụng chung công cụ, ngôn ngữ, và giao tiếp nhằm hướng tới một mục tiêu duy nhất: cung cấp giá trị nhanh chóng và an toàn.

Trong môi trường DevOps, các nhóm phát triển và vận hành cùng báo cáo cho một lãnh đạo chung, cùng nhau tìm cách để đảm bảo cung cấp giá trị một cách liên tục trong khi vẫn duy trì sự ổn định, hiệu suất và khả năng chịu lỗi của hệ thống.

Nhờ vào DevOps, sự quan liêu và các quy trình phức tạp được thay thế bằng sự tin tưởng và trách nhiệm. Các phương thức làm việc linh hoạt, nhất quán được áp dụng để cung cấp giá trị nhanh chóng nhưng vẫn bảo đảm tính an toàn.

**DevOps và Cloud-Native**

Cùng với DevOps, nhiều doanh nghiệp còn áp dụng kiến trúc cloud-native để đảm bảo tính linh hoạt, bổ trợ công nghệ và khả năng phát triển nhanh. Các kiến trúc này giúp cung cấp công nghệ cần thiết để đạt được mục tiêu cung cấp giá trị nhanh và an toàn cho tổ chức.

Tóm lại, DevOps giúp xóa bỏ rào cản giữa các nhóm silo truyền thống, tạo ra một môi trường hợp tác, tập trung vào cung cấp giá trị nhanh chóng và an toàn. Khi đó, sự an toàn và hiệu quả không còn mâu thuẫn nhau, mà hỗ trợ nhau phát triển.

## **Từ mô hình cân bằng ngắt quãng sang triển khai liên tục: tăng tốc và hiệu quả**

Trong nhiều doanh nghiệp, quy trình agile đã được áp dụng, nhưng thường chỉ dừng lại ở cấp độ nhóm phát triển. Mặc dù các nhóm phát triển đã thành công trong việc chuyển đổi sang cách làm việc agile, doanh nghiệp vẫn gặp khó khăn khi cần đưa các tính năng mới lên môi trường production. Câu hỏi "Khi nào chúng tôi có thể thấy những tính năng này ở môi trường production?" thường rất khó trả lời do cần phải cân nhắc nhiều yếu tố nằm ngoài tầm kiểm soát.

**Những yếu tố gây trở ngại đến việc triển khai**

Các yếu tố gây trở ngại thường bao gồm:

* Thời gian cần thiết để hoàn thành quá trình kiểm tra chất lượng độc lập.
    
* Khả năng tham gia vào lịch trình release production.
    
* Liệu bộ phận vận hành IT có thể cung cấp môi trường production kịp thời hay không.
    

Điều này dẫn đến tình trạng mà Dave West gọi là "waterscrumfall". Nhóm phát triển đã áp dụng agile, nhưng tổ chức tổng thể lại chưa thay đổi tương ứng. Thay vì mỗi vòng lặp phát triển đều dẫn đến triển khai sản xuất, mã nguồn lại được tích lũy để phát hành trong một chu trình truyền thống.

**Vấn đề của cách tiếp cận "cân bằng ngắt quãng"**

Thay vì mang lại giá trị và nhận phản hồi liên tục từ khách hàng, doanh nghiệp lại tiếp tục kiểu phát hành "cân bằng ngắt quãng", dẫn đến hai hậu quả lớn:

* **Khách Hàng Phải Chờ Đợi Value:** Khách hàng phải chờ nhiều tuần mà không thấy tính năng mới. Họ không thấy được giá trị của agile và có xu hướng quay lại lối suy nghĩ cũ, dồn càng nhiều yêu cầu vào mỗi lần release. Vì thiếu niềm tin vào nhịp độ triển khai, họ muốn bao gồm càng nhiều value càng tốt khi source code cuối cùng được release.
    
* **Thiếu Phản Hồi Thực Tế Cho Nhóm Phát Triển:** Nhóm phát triển không nhận được phản hồi thực tế từ khách hàng trong môi trường production. Demo chỉ là mô phỏng, và phản hồi value nhất chỉ đến khi người dùng thật sự sử dụng phần mềm. Khi phản hồi này bị trì hoãn, quy trình xây dựng sai càng này càng tăng lên, cùng với chi phí fix ngày càng tốn kém.
    

**Triển khai liên tục (CD): Giải pháp cho vấn đề này**

Để tận dụng hết tiềm năng của kiến trúc ứng dụng cloud-native, doanh nghiệp cần chuyển sang triển khai liên tục. Thay vì duy trì các silo và quy trình phức tạp như kiểu "waterscrumfall", DevOps mang đến sự thống nhất về giá trị từ đầu đến cuối.

Một mô hình hữu ích để hình dung vòng đời này là "Từ Khái Niệm Đến Tiền" của Mary và Tom Poppendieck trong cuốn sách *Implementing Lean Software Development*. Phương pháp này nhìn nhận toàn bộ chuỗi giá trị từ ý tưởng kinh doanh đến thời điểm nó mang lại lợi nhuận, và xây dựng một dòng giá trị nhất quán, kết nối con người và quy trình để tối ưu hoá mục tiêu.

**Hỗ Trợ Kỹ Thuật Cho Triển Khai Liên Tục**

Để hỗ trợ kỹ thuật cho mô hình triển khai liên tục, chúng ta cần áp dụng các pipeline tự động hóa để kiểm chứng mỗi vòng lặp (hay thậm chí mỗi lần commit mã nguồn) đều có thể triển khai một cách an toàn. Mọi bài kiểm tra cần được tự động hóa, và chỉ khi tất cả các kiểm tra đều thành công thì triển khai sản xuất mới được tiến hành.

Quyết định triển khai cuối cùng sẽ không còn phụ thuộc vào vấn đề kỹ thuật mà hoàn toàn là quyết định kinh doanh: liệu đã hợp lý về mặt kinh doanh để triển khai tính năng mới chưa? Nếu câu trả lời là có, việc triển khai có thể diễn ra chỉ với một cú nhấp chuột nhờ vào hệ thống triển khai tự động.

**DevOps và Cloud-Native: Tạo Ra Giá Trị Liên Tục**

DevOps, kết hợp với kiến trúc cloud-native, tạo điều kiện cho triển khai liên tục, giúp doanh nghiệp cung cấp giá trị nhanh chóng và liên tục cho khách hàng. Từ đó, chúng ta có thể đảm bảo rằng các tính năng mới luôn sẵn sàng đến tay khách hàng một cách nhanh nhất và an toàn nhất, đồng thời xây dựng niềm tin vào khả năng triển khai nhanh chóng và đáng tin cậy.

Tóm lại, từ "cân bằng ngắt quãng" chuyển sang triển khai liên tục giúp phá bỏ các rào cản về quy trình truyền thống, tối ưu hóa khả năng cung cấp giá trị liên tục và đảm bảo rằng sự an toàn và hiệu quả hỗ trợ lẫn nhau để phát triển.

## **Từ quản trị** tập trung **sang** phân tán: Tăng tính linh hoạt và hiệu quả

Trong lĩnh vực IT truyền thống, quản trị tập trung thường được áp dụng để duy trì các tiêu chuẩn, quy trình và kiểm duyệt các thiết kế cũng như thay đổi. Tuy nhiên, quản trị tập trung thường không đạt được những cải thiện chất lượng hay giảm chi phí như mong muốn. Ngược lại, nó còn tạo ra các điểm nghẽn và làm chậm quá trình triển khai.

Để thích nghi với kiến trúc cloud-native, các doanh nghiệp cần chuyển sang phân tán quản trị. Các nhóm phát triển ứng dụng cloud-native (còn gọi là “Business Capability Teams”) sẽ chịu trách nhiệm hoàn toàn về các khía cạnh của tính năng mà họ cung cấp. Các đội nhóm này tự quản lý từ dữ liệu, công nghệ, kiến trúc ứng dụng, thiết kế thành phần cho đến hợp đồng API kết nối với các bộ phận khác của tổ chức. Nếu cần đưa ra quyết định, họ có quyền tự quyết và thực thi mà không phải đợi sự phê duyệt từ các ủy ban trung tâm.

**Cân Bằng Quản Trị Bằng Các Cấu Trúc Tối Giản**

Sự tự trị của các nhóm được cân bằng bởi những cấu trúc tối giản, nhằm đảm bảo tính đồng nhất trong tích hợp giữa các dịch vụ phát triển và triển khai độc lập (ví dụ: ưu tiên sử dụng API HTTP REST JSON thay vì nhiều kiểu RPC khác nhau). Các cấu trúc này thường được xây dựng từ việc giải quyết các vấn đề gốc rễ liên quan đến khả năng chịu lỗi và các mối quan tâm cắt ngang khác. Các đội nhóm cũng được khuyến khích phát triển và thử nghiệm giải pháp của mình tại chỗ, sau đó hợp tác với các nhóm khác để xây dựng các mẫu và khung làm việc chung.

Khi một giải pháp ưu tiên cho toàn bộ tổ chức xuất hiện, quyền sở hữu giải pháp này thường được chuyển giao cho một đội khung/nền tảng đám mây, đội này có thể hoặc không thể tích hợp với nhóm vận hành nền tảng. Nhóm khung/nền tảng đám mây thường cũng là những người tiên phong trong việc phát triển giải pháp khi tổ chức đang cải tiến xung quanh sự hiểu biết chung về kiến trúc.

**Sự Thay Đổi Về Văn Hóa và Cách Quản Lý**

Chuyển sang kiến trúc cloud-native không chỉ đòi hỏi sự thay đổi về mặt kỹ thuật mà còn cần thay đổi về mặt văn hóa và cách quản lý. Để đạt được hiệu suất và linh hoạt tối đa, doanh nghiệp cần phá bỏ những rào cản từ các silo chuyên môn, thúc đẩy sự phối hợp và đồng bộ giữa các nhóm. Sự tự trị và phân quyền giúp các nhóm có thể ra quyết định nhanh chóng và cung cấp giá trị một cách liên tục, từ đó mang lại sự linh hoạt, sáng tạo và hiệu quả cho toàn tổ chức.