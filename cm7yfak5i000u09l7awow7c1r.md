---
title: "Conventional Commits – Quy ước viết commit message chuẩn"
seoTitle: "Standardize commit messages"
seoDescription: "Tìm hiểu Conventional Commits – quy ước viết commit message để đảm bảo lịch sử thay đổi rõ ràng và hỗ trợ tự động hóa quá trình phát triển phần mềm"
datePublished: Fri Mar 07 2025 06:57:03 GMT+0000 (Coordinated Universal Time)
cuid: cm7yfak5i000u09l7awow7c1r
slug: conventional-commits-quy-uoc-viet-commit-message-chuan

---

## Tổng quan về Conventional Commits

Blog được tham khảo từ [https://www.conventionalcommits.org/](https://www.conventionalcommits.org/)

Conventional Commits là một **đặc tả (specification)** nhẹ về cách viết thông điệp commit trong Git. Mục tiêu của nó là giúp cả **con người và máy móc** có thể dễ dàng hiểu được lịch sử thay đổi của dự án thông qua commit messages ([Bạn đang viết commit message như thế nào?](https://viblo.asia/p/ban-dang-viet-commit-message-nhu-the-nao-gDVK22A0KLj#:~:text=Conventional%20Commits%20l%C3%A0%20m%E1%BB%99t%20b%E1%BB%99,c%C3%A1ch%20t%E1%BB%B1%20%C4%91%E1%BB%99ng%20ch%E1%BA%B3ng%20h%E1%BA%A1n)). Quy tắc này đưa ra một tập hợp các quy định đơn giản, tạo sự thống nhất và rõ ràng cho commit history ([Làm thế nào để viết Conventional Commits cho dễ sử dụng](https://viblo.asia/p/lam-the-nao-de-viet-conventional-commits-cho-de-su-dung-07LKXbb2lV4#:~:text=,m%C3%B4%20t%E1%BA%A3%20c%C3%A1c%20feature%2C%20fix)). Nhờ đó, đội ngũ phát triển có thể nhanh chóng nắm bắt được **mục đích mỗi thay đổi**, và các công cụ tự động (như trong CI/CD) có thể phân tích commit để thực hiện các tác vụ quản lý phiên bản.

**Lợi ích**: Sử dụng Conventional Commits mang lại nhiều lợi ích thiết thực:

* **Rõ ràng và nhất quán**: Mọi người trong nhóm đều tuân theo một chuẩn chung, giúp tránh tình trạng commit message mỗi người một kiểu. Ví dụ, thay vì những commit mơ hồ như *“update code”* hay *“sửa lỗi”*, bạn sẽ có commit rõ ràng như `fix(auth): sửa logic xác thực token` – nhìn vào biết ngay phạm vi và nội dung thay đổi.
    
* **Dễ dàng tra cứu lịch sử**: Lịch sử commit trở nên có trật tự. Bạn có thể tìm kiếm commit theo loại (feat, fix, ...) hoặc scope bằng regex một cách thuận tiện. Điều này cũng giúp reviewer hiểu nhanh bối cảnh thay đổi và bỏ qua những commit không ảnh hưởng chức năng (như kiểu *style* chỉ format code) ([Karma - Git Commit Msg](http://karma-runner.github.io/6.4/dev/git-commit-msg.html#:~:text=,ignoring%20style%20changes)).
    
* **Hỗ trợ tự động hóa**: Vì tuân thủ theo khuôn mẫu chung, commit messages có thể được công cụ phân tích để tự động tạo **CHANGELOG**, xác định phiên bản phát hành (theo Semantic Versioning), hoặc kích hoạt các bước CI/CD phù hợp.
    

## Cấu trúc của một commit message chuẩn

Một commit message theo chuẩn Conventional Commits có dạng:

```markdown
<type>[optional scope]: <description>

[optional body]

[optional footer]
```

Trong đó:

* **type**: Bắt buộc. Một từ khóa xác định *loại* thay đổi, ví dụ thêm tính năng hoặc sửa lỗi.
    
* **scope**: (Tùy chọn) Phạm vi ảnh hưởng của commit, đặt trong ngoặc đơn ngay sau type. Scope giúp người đọc biết phần nào của dự án bị ảnh hưởng. Ví dụ: `feat(authentication): ...` cho biết thay đổi thuộc module authentication.
    
* **description**: Bắt buộc. Mô tả ngắn gọn về thay đổi chính trong commit. Nên viết ở thì hiện tại, không viết hoa ký tự đầu, kết thúc không dấu chấm.
    
* **body**: (Tùy chọn) Phần mô tả chi tiết hơn, giải thích **tại sao** cần thực hiện thay đổi hoặc thêm thông tin cần thiết khác. Phần body cách phần tiêu đề một dòng trống, và có thể dài nhiều dòng.
    
* **footer**: (Tùy chọn) Chứa thông tin bổ sung như ID của issue/ticket liên quan, người review, hoặc ghi chú **BREAKING CHANGE**. Mỗi nội dung footer nằm trên một dòng riêng.
    

Ví dụ commit message ngắn theo cấu trúc trên:

```markdown
feat: thêm chức năng thanh toán qua PayPal

Thêm module thanh toán mới, tích hợp API PayPal vào hệ thống.
```

Ví dụ trên có type `feat` (feature), không có scope, description ngắn gọn ở dòng đầu. Phần body mô tả thêm chi tiết về thay đổi.

Một ví dụ khác có scope và footer:

```markdown
fix(payment): xử lý sai số trong tính tổng hóa đơn

Đã sửa hàm tính tổng tiền để làm tròn số thập phân đúng cách.

BREAKING CHANGE: thay đổi kiểu dữ liệu trả về của hàm calculateTotal().
```

Ở ví dụ này, type `fix` biểu thị commit sửa lỗi, scope là `payment`. Description nêu vấn đề được giải quyết. Trong phần footer, có `BREAKING CHANGE` để cảnh báo rằng hàm `calculateTotal()` đã thay đổi hành vi (đây là thay đổi phá vỡ, không tương thích ngược).

## Các loại commit phổ biến

Conventional Commits định nghĩa một số **loại (type)** chuẩn để phân loại commit. Những type thường gặp gồm:

* **feat**: Thêm **feature** mới cho codebase (tính năng mới). Commit `feat` tương ứng việc tăng phiên bản **MINOR** theo Semantic Versioning.
    
* **fix**: Sửa một **lỗi bug** trong codebase. Commit `fix` tương ứng việc tăng phiên bản **PATCH**.
    
* **docs**: Thay đổi **tài liệu** (chỉ sửa đổi những file documentation, ví dụ README, hướng dẫn sử dụng).
    
* **style**: Thay đổi về **định dạng code** mà không ảnh hưởng đến logic chương trình (ví dụ: chỉnh code format, thêm dấu chấm phẩy, sửa dấu cách).
    
* **refactor**: **Tái cấu trúc** code mà **không** thêm tính năng mới hay sửa lỗi (ví dụ: đổi tên biến, tách hàm). Mục đích để cải thiện cấu trúc code cho sạch hơn.
    
* **perf**: Thay đổi **hiệu năng** (performance) nhằm tối ưu tốc độ/chất lượng chạy của chương trình, không thêm tính năng mới.
    
* **test**: Bổ sung hoặc sửa đổi các **test case**, thêm test còn thiếu, refactor code test (không ảnh hưởng code sản phẩm).
    
* **chore**: Thực hiện các công việc **lặt vặt** không ảnh hưởng trực tiếp đến mã nguồn ứng dụng hoặc chức năng người dùng. Ví dụ: cập nhật thư viện, cấu hình build, dọn dẹp code.
    
* **build**: Thay đổi liên quan đến **hệ thống build** hoặc các công cụ phụ trợ (ví dụ: thay đổi file webpack, Gradle, npm scripts).
    
* **ci**: Thay đổi cấu hình **CI/CD** (ví dụ: chỉnh sửa file cấu hình Jenkins/Travis, GitHub Actions pipeline).
    

Ngoài ra, tùy dự án có thể định nghĩa thêm các type khác nếu cần, miễn là thống nhất trong nhóm. Ví dụ có thể có `revert` (đảo ngược commit), hoặc các nhãn tùy chỉnh riêng. (Thực tế, Conventional Commits không bắt buộc cứng nhắc ngoài `feat` và `fix`; nhóm dự án có thể tự mở rộng thêm danh mục type phục vụ nhu cầu nội bộ ([Bạn đang viết commit message như thế nào?](https://viblo.asia/p/ban-dang-viet-commit-message-nhu-the-nao-gDVK22A0KLj#:~:text=,ta%20t%E1%BB%B1%20%C4%91%E1%BB%8Bnh%20ngh%C4%A9a%20nh%C3%A9))).

## Cách xử lý breaking changes trong commit messages

**Breaking change** là thay đổi gây **phá vỡ tương thích ngược** (backward incompatible) – ví dụ thay đổi API công khai mà các phiên bản cũ không dùng được. Trong Conventional Commits, breaking changes cần được đánh dấu rõ ràng để người dùng và công cụ biết đây là thay đổi lớn (major).

Có hai cách để ghi chú breaking change:

1. **Thêm “!”** vào ngay sau type (và scope nếu có) trong tiêu đề commit. Ví dụ: `feat!: thay đổi cấu trúc xác thực người dùng`. Dấu “!” này giúp nhấn mạnh commit này có thay đổi phá vỡ.
    
2. **Thêm dòng “BREAKING CHANGE:”** ở phần footer hoặc ngay đầu phần body của commit message ([Bạn đang viết commit message như thế nào?](https://viblo.asia/p/ban-dang-viet-commit-message-nhu-the-nao-gDVK22A0KLj#:~:text=,tr%C6%B0%E1%BB%9Bc%20%C4%91%C3%B3%20s%E1%BA%BD%20kh%C3%B4ng%20c%C3%B2n)) ([Bạn đang viết commit message như thế nào?](https://viblo.asia/p/ban-dang-viet-commit-message-nhu-the-nao-gDVK22A0KLj#:~:text=,VD)). Sau cụm từ này là mô tả ngắn gọn về thay đổi phá vỡ. Ví dụ:
    
    ```markdown
    BREAKING CHANGE: endpoint /api/login thay đổi cách thức xác thực, 
    yêu cầu client sử dụng OAuth2 thay cho basic auth.
    ```
    
    Nếu dự án có nhiều thay đổi phá vỡ, có thể liệt kê từng thay đổi trong các dòng footer riêng biệt.
    

Khi một commit được đánh dấu breaking change theo cách trên, công cụ quản lý phiên bản sẽ hiểu rằng cần tăng **MAJOR version** cho lần phát hành kế tiếp. Nói cách khác, kết hợp `feat`/`fix` với breaking change sẽ không còn chỉ là minor/patch nữa, mà sẽ bump lên phiên bản lớn tiếp theo (X.y.z -&gt; (X+1).0.0 theo SemVer).

## Ứng dụng Conventional Commits trong CI/CD và Semantic Versioning

Nhờ format thống nhất và có ngữ nghĩa, Conventional Commits mở ra khả năng tự động hóa trong quy trình CI/CD và quản lý phiên bản:

* **Tự động tăng version (Semantic Versioning)**: Conventional Commits kết hợp chặt chẽ với **Semantic Versioning (SemVer)** ([Bạn đang viết commit message như thế nào?](https://viblo.asia/p/ban-dang-viet-commit-message-nhu-the-nao-gDVK22A0KLj#:~:text=ph%E1%BB%A5c%20v%E1%BB%A5%20cho%20vi%E1%BB%87c%20versioning,c%C3%A1ch%20t%E1%BB%B1%20%C4%91%E1%BB%99ng%20ch%E1%BA%B3ng%20h%E1%BA%A1n)). Các commit loại `fix` sẽ trigger tăng patch version, `feat` trigger tăng minor version, còn commit có breaking change sẽ trigger tăng major version. Các công cụ như **semantic-release** có thể phân tích lịch sử commit để xác định phiên bản kế tiếp là bao nhiêu một cách tự động. Điều này giúp loại bỏ sai sót thủ công khi đặt version.
    
* **Tự động sinh CHANGELOG**: Dựa trên commit history tuân thủ quy ước, ta có thể tự động tạo file CHANGELOG.md liệt kê các thay đổi cho mỗi phiên bản. Công cụ **conventional-changelog** hoặc plugin **release-notes-generator** của semantic-release sẽ lấy thông tin từ commit messages để tổng hợp lại thành danh sách các tính năng mới, sửa lỗi, cải tiến, v.v. Ví dụ, thay vì phải đọc `git log` thủ công, ta dùng semantic-release để tạo changelog sau mỗi lần phát hành ([Bạn đang viết commit message như thế nào?](https://viblo.asia/p/ban-dang-viet-commit-message-nhu-the-nao-gDVK22A0KLj#:~:text=git%20log%20v1.0.0...HEAD%20)).
    
* **Tích hợp trong pipeline CI/CD**: Nhiều đội phát triển cấu hình pipeline CI/CD để kiểm tra và phát hành dựa trên commit. Ví dụ: trên nhánh chính, mỗi khi có commit mới được merge, pipeline sẽ chạy tool semantic-release. Tool này sẽ:
    
    1. Phân tích commit messages (sử dụng **commit-analyzer** theo chuẩn Conventional Commits) để quyết định bump phiên bản kiểu PATCH, MINOR hay MAJOR.
        
    2. Tạo release notes (danh sách thay đổi) và cập nhật vào file CHANGELOG.md.
        
    3. Đóng gói phiên bản mới (ví dụ cập nhật version trong package.json, tạo tag git tương ứng) và triển khai (deploy) nếu cần ([GIT - Semantic versioning and conventional commits](https://blog.opensight.ch/git-semantic-versioning-und-conventional-commits/#:~:text=%2A%20commit,generator%29%20to%20a%20file%20CHANGELOG.md)) ([GIT - Semantic versioning and conventional commits](https://blog.opensight.ch/git-semantic-versioning-und-conventional-commits/#:~:text=Writes%20the%20next%20version%20number,gitlab)).
        
    4. Tự động commit các thay đổi (như cập nhật version, changelog) và push lên repository (thông qua bot hoặc plugin git) ([GIT - Semantic versioning and conventional commits](https://blog.opensight.ch/git-semantic-versioning-und-conventional-commits/#:~:text=Execute%20any%20command,changes%20to%20the%20Gitlab%20repository)).
        

Nhờ vậy, Conventional Commits giúp quá trình phát hành (release) gần như hoàn toàn tự động: đúng phiên bản, đủ thông tin changelog, giảm thiểu công sức thủ công.

## Ví dụ minh họa thực tế

Giả sử chúng ta có một dự án phần mềm **E-Shop** và nhóm quyết định áp dụng Conventional Commits:

* Nhà phát triển A thêm một tính năng giỏ hàng mới, commit: `feat(cart): thêm chức năng thêm sản phẩm vào giỏ hàng`
    
* Nhà phát triển B sửa một lỗi hiển thị giá sản phẩm sai, commit: `fix(ui): sửa lỗi hiển thị sai tổng giá ở trang giỏ hàng`
    
* Nhà phát triển C cải thiện hiệu năng xử lý đơn hàng, commit: `perf(order): tối ưu thuật toán sắp xếp đơn hàng`
    
* Nhà phát triển D cập nhật tài liệu hướng dẫn tích hợp thanh toán, commit: `docs: cập nhật hướng dẫn tích hợp cổng thanh toán PayPal`
    

Sau khi merge các commit này vào nhánh chính, nhóm chuẩn bị phát hành phiên bản mới. Công cụ semantic-release sẽ quét các commit:

* Có hai commit loại `feat` -&gt; sẽ tăng **MINOR version** (ví dụ từ 1.3.0 lên 1.4.0).
    
* Có commit `fix` và `perf` (sửa lỗi và cải thiện hiệu năng). Chúng thường tương ứng mức PATCH, nhưng do đã có commit feat nên phiên bản sẽ tăng ở mức minor như trên.
    
* Không có commit breaking change, nên không cần tăng major.
    

Kết quả, phiên bản mới **1.4.0** được tạo. File **CHANGELOG.md** cũng được sinh tự động, có thể như sau:

```markdown
## [1.4.0] - 2025-03-07
### Features
- **cart**: thêm chức năng thêm sản phẩm vào giỏ hàng.
- **order**: tối ưu thuật toán sắp xếp đơn hàng. (perf)

### Bug Fixes
- **ui**: sửa lỗi hiển thị sai tổng giá ở trang giỏ hàng.

### Documentation
- Cập nhật hướng dẫn tích hợp cổng thanh toán PayPal.
```

Trong changelog trên, các commit được phân nhóm theo loại (feature, fix, docs, v.v.) rất rõ ràng. Điều này có được là nhờ việc tuân thủ chặt chẽ Conventional Commits trong suốt quá trình phát triển dự án.

*Lưu ý:* Với commit có **breaking change**, changelog sẽ có thêm mục **Breaking Changes** mô tả chi tiết phần thay đổi không tương thích. Đồng thời version sẽ tăng major, ví dụ từ 1.4.0 lên 2.0.0.

Ngoài ra, nhiều dự án mã nguồn mở lớn đã áp dụng Conventional Commits làm chuẩn commit. Điển hình như **Angular**, **Electron**, v.v. Các commit trong Angular thường có dạng như `fix(router): ...`, `feat(core): ...` kèm mô tả ngắn, và Angular sử dụng những thông tin này để tạo release notes tự động cho mỗi phiên bản. Việc cộng đồng contributor tuân thủ chung một convention giúp dự án dễ dàng quản lý hàng trăm commit từ nhiều người.

## Mẹo và công cụ hỗ trợ

Áp dụng Conventional Commits sẽ hiệu quả hơn khi kết hợp với một số công cụ tự động kiểm tra và hỗ trợ:

* **Commitlint**: Một công cụ kiểm tra cú pháp commit message. Bạn có thể tích hợp commitlint với cấu hình **@commitlint/config-conventional** (bộ rule Conventional Commits) để đảm bảo mọi commit tuân thủ đúng format. Thường dùng kèm với Husky (git hook) để chạy tự động khi commit.
    
* **Husky**: Thư viện cho phép dễ dàng thêm các **Git hook**. Ví dụ, cấu hình Husky để chạy *commitlint* ở hook `commit-msg`. Nếu thông điệp commit không đúng chuẩn, commit sẽ bị chặn lại, giúp duy trì kỷ luật ngay từ máy của developer.
    
* **Commitizen**: Công cụ giúp lập trình viên viết commit message theo format Conventional Commits một cách dễ dàng hơn. Khi dùng Commitizen (`git cz` thay cho `git commit`), bạn sẽ được hướng dẫn điền từng phần (type, scope, description, v.v.) qua giao diện dòng lệnh tương tác.
    
* **Các plugin IDE**: Nếu dùng VS Code, có extension **Conventional Commits** hỗ trợ tự động điền khuôn mẫu commit. Tương tự, các IDE khác (JetBrains, v.v.) cũng có plugin hỗ trợ viết message đúng chuẩn một cách thuận tiện.
    
* **semantic-release / standard-version**: Đây là các công cụ tự động phát hành phiên bản. Chúng sử dụng commit message Conventional Commits để bump version, tạo tag, và cập nhật changelog. Bạn có thể tích hợp chúng vào pipeline CI để mỗi khi merge code, phiên bản mới sẽ được phát hành kèm changelog mà không cần thao tác thủ công.
    

**Mẹo nhỏ**: Hãy thống nhất quy ước commit ngay từ đầu dự án và thông báo cho tất cả thành viên trong nhóm. Ban đầu, mọi người có thể mất chút thời gian làm quen với cách viết mới, nhưng về lâu dài lợi ích mang lại rất lớn: lịch sử rõ ràng, tự động hóa việc release, và dự án trông chuyên nghiệp hơn hẳn. Nên bổ sung hướng dẫn Conventional Commits vào tài liệu dự án (ví dụ: file README hoặc CONTRIBUTING.md) để ai cũng có thể tra cứu khi cần.