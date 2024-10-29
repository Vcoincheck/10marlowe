# Nguyên mẫu ứng dụng thanh toán (Payouts)

### Khám phá, theo dõi và rút tokens&#x20;

[**payouts DApp prototype**](https://github.com/input-output-hk/marlowe-payouts) là một ví dụ về ứng dụng phi tập trung được thiết kế để giúp người dùng phát hiện, theo dõi và rút tiền từ các hợp đồng thông minh Marlowe sử dụng token vai trò. Nó cho phép những người nắm giữ token vai trò trong các hợp đồng Marlowe rút tiền đã nhận, đơn giản hóa quá trình theo dõi và rút tiền thanh toán của họ.

Bởi vì token vai trò có thể là [ada handles](https://handle.tools/), [ada domains](https://www.adadomains.io/) hoặc token tùy chỉnh, người dùng có nhiều sự linh hoạt trong việc sử dụng thành phần DApp thanh toán này.

#### Xây dựng với Marlowe TS-SDK​ <a href="#built-with-the-marlowe-ts-sdk" id="built-with-the-marlowe-ts-sdk"></a>

Các nhà phát triển Marlowe đã xây dựng nguyên mẫu DApp thanh toán bằng cách sử dụng Marlowe TS-SDK, cung cấp một cách mạnh mẽ để viết các ứng dụng web.&#x20;

_TS-SDK là một tập hợp các thư viện JavaScript và TypeScript giúp các nhà phát triển DApp tương tác với hệ sinh thái Marlowe._

#### **Mục đích sử dụng**

Nguyên mẫu DApp thanh toán minh họa (_Payouts_) các chức năng có sẵn thông qua Marlowe TS-SDK. Các nhà phát triển DApp có thể nhìn nhận DApp thanh toán từ nhiều góc độ khác nhau:

* Như một ví dụ về một ứng dụng độc lập.
* Như chức năng mà họ có thể tích hợp vào một DApp.
* Như một thành phần mà họ có thể tích hợp vào một ví.

Ví dụ, có thể tích hợp DApp thanh toán vào một ví để khi bạn truy cập vào ví, bạn sẽ thấy tất cả các token có sẵn mà bạn có thể rút từ các hợp đồng Marlowe mà không cần sử dụng một ứng dụng riêng biệt.

#### Sử dụng cùng với Marlowe Runner​ <a href="#use-alongside-marlowe-runner" id="use-alongside-marlowe-runner"></a>

Sử dụng chức năng DApp thanh toán cùng với **Runner** ![](https://vcc.gitbook.io/\~gitbook/image?url=https%3A%2F%2Fimg.shields.io%2Fbadge%2Fbeta-2D0EB1\&width=300\&dpr=4\&quality=100\&sign=16a00593\&sv=1), bạn có thể kết nối một ví được ủy quyền nhận tiền với DApp thanh toán khi tiến hành một hợp đồng thông qua Runner. Khi ví đã được kết nối, Runner sẽ hiển thị danh sách các hợp đồng mà bạn tham gia.

DApp thanh toán sẽ liệt kê các token có sẵn mà một vai trò được ủy quyền có thể rút cho ví đó. Người nhận nhấn nút ‘Rút tiền’ và thấy một thông báo để ký giao dịch. Sau khi người nhận ký giao dịch, giao dịch sẽ được xác nhận. Người nhận sau đó có thể thấy số tiền trong ví của họ.
