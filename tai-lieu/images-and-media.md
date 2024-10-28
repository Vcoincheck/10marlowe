# Marlowe Runner

Video này giải thích cách triển khai và thực thi các hợp đồng được tạo ra trong Marlowe Playground bằng cách sử dụng Marlowe Runner (phiên bản beta).&#x20;

Marlowe Runner là một công cụ dành cho nhà phát triển với giao diện thân thiện để triển khai và thực thi hợp đồng thông minh trên Cardano, ngay từ trình duyệt.



{% embed url="https://youtu.be/B5XcH0j7Y7w" %}

### Một công cụ dành cho nhà phát triển và một DApp để chạy hợp đồng trên blockchain

\
Runner giúp việc triển khai hợp đồng trở nên đơn giản cho cả những người xây dựng DApp và các nhà phát triển truyền thống, cho phép thực thi các hợp đồng được tạo ra trong Playground mà không cần yêu cầu bất kỳ kiến thức lập trình hay phối hợp backend nào.

### Mẫu DApp tùy chỉnh

Hơn nữa, Marlowe Runner là một ứng dụng mã nguồn mở dành cho nhà phát triển, giúp bạn dễ dàng sao chép để tạo ra một giao diện người dùng tùy chỉnh. Vì vậy, nó không chỉ phục vụ như một công cụ để chạy các hợp đồng trên blockchain mà còn như một mẫu DApp tùy chỉnh mà bạn có thể sử dụng để tạo DApp của riêng mình cho trường hợp sử dụng cụ thể của bạn.

### Dễ sử dụng

Sử dụng Runner không yêu cầu kiến thức về công cụ dòng lệnh, vì vậy nó rất đơn giản và trực quan. Bạn chỉ cần chỉ định mạng mà bạn muốn làm việc, kết nối ví đang ở cùng mạng và có sẵn thông tin mật khẩu để có thể ký giao dịch với ví của bạn. Bạn cũng cần có bất kỳ mã thông báo hoặc quỹ cần thiết nào có sẵn trong ví của bạn.

### Triển khai hợp đồng của bạn

Khi bạn đã hoàn thành việc tạo, mô phỏng và kiểm tra hợp đồng thông minh Marlowe của mình trong Playground, bạn có thể triển khai hợp đồng thông minh của mình lên Runner bằng một trong những phương pháp sau:\
Từ Playground, chọn 'Gửi đến Trình mô phỏng,' sau đó chọn 'Xuất sang Marlowe Runner.'\
Ngoài ra, tải hợp đồng của bạn từ Playground dưới dạng tệp JSON, sau đó tải lên Runner.

> * [**Access Runner on the preview network**](https://preview.runner.marlowe.iohk.io/)
> * [**Access Runner on the pre-production network**](https://preprod.runner.marlowe.iohk.io/)

### Chi tiết kỹ thuật[​](https://docs.marlowe.iohk.io/docs/getting-started/runner#technical-details) <a href="#technical-details" id="technical-details"></a>

#### Vai trò

Khi một hợp đồng sử dụng vai trò được gửi đến Runner, Runner sẽ luôn yêu cầu người dùng cung cấp địa chỉ cho mỗi vai trò.

#### Minting token vai trò

Runner sẽ mint các token vai trò cần thiết cho hợp đồng. Ai sở hữu các token này sẽ có quyền ủy quyền để trở thành một bên trong hợp đồng đó. (Việc mint thực tế được thực hiện bởi Runtime.)

#### Chế độ xem đồ thị nguồn

Khi xây dựng một DApp hợp đồng từ TS-SDK, Haskell, PureScript hoặc các ngôn ngữ khác, việc xem nó trong Runner có thể hữu ích để phân tích logic của hợp đồng từ chế độ xem 'Nguồn'.

#### Tiến hành hợp đồng

Các nút bấm sẽ trở nên hoạt động khi có các hành động có sẵn cho vai trò hợp đồng liên kết với địa chỉ ví của bạn. Nếu nút bấm hành động không được kích hoạt, điều đó có nghĩa là trạng thái hợp đồng đang chờ một bên khác trong hợp đồng thực hiện hành động.

#### Gửi tiền và lựa chọn

Tùy thuộc vào các điều kiện của hợp đồng và vai trò của bạn, bạn có thể có tùy chọn để thực hiện một số lựa chọn và gửi quỹ.

#### Rút tiền

Khi một hợp đồng cho phép rút ada hoặc các token khác, chức năng rút tiền sẽ hiển thị trong Runner. Cần có ví được ủy quyền và chữ ký để thực hiện rút tiền.

#### Merkle hóa

Trong khi Marlowe hỗ trợ merkle hóa, Runner vẫn chưa hỗ trợ tính năng này.

#### Trạng thái

Trạng thái của các hợp đồng Marlowe được xác định bởi các đầu vào cho hợp đồng ở mỗi bước, giúp hành vi của hợp đồng dễ hiểu và dự đoán hơn.

### Các loại ví[​](https://docs.marlowe.iohk.io/docs/getting-started/runner#wallets) <a href="#wallets" id="wallets"></a>

Runner hỗ trợ các ví web3 [**Lace**](https://www.lace.io/), [**Nami**](https://namiwallet.io/), and [**Eternl**](https://eternl.io/app/mainnet/welcome).

Hiện tại, Runner không tương thích với các ví phần cứng (hardware) như Ledger và Trezor.&#x20;

### Hợp đồng phức tạp và tinh vi hơn

Đối với những kịch bản hợp đồng đặc biệt phức tạp và tinh vi, bạn có thể cần tùy chỉnh cách tiếp cận của mình. Runner chưa được thiết kế để quản lý các hợp đồng rất phức tạp.

### Hợp đồng phù hợp với Runner

Các hợp đồng phù hợp và thực thi trên chuỗi sẽ hoạt động trong Runner. Các ví dụ hợp đồng có trong Playground là phù hợp với Runner.

### Những điều cần cân nhắc trước khi triển khai

#### Giới hạn trên chuỗi

Có một số giới hạn trên chuỗi liên quan đến giới hạn ký tự của tên vai trò và token, kích thước giao dịch, giới hạn chi phí giao dịch, số lượng tài khoản, địa chỉ giao dịch không hợp lệ, cảnh báo công cụ Runtime và việc kiểm tra mà bạn nên nhận thức. Để biết thêm chi tiết, vui lòng xem:

* [**Marlowe's on-chain limitations**](https://docs.marlowe.iohk.io/docs/platform-and-architecture/on-chain-limitations)
* [**A comprehensive guide to Marlowe's security: audit outcomes, built-in functional restrictions, and ledger security features**](https://iohk.io/en/blog/posts/2023/06/27/a-comprehensive-guide-to-marlowes-security-audit-outcomes-built-in-functional-restrictions-and-ledger-security-features/)
