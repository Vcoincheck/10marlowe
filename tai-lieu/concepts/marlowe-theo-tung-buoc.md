# Marlowe theo từng bước

Marlowe có sáu cách để xây dựng hợp đồng. Năm trong số này -- `Pay`, `Let`, `If`, `When` và `Assert` -- xây dựng một hợp đồng phức tạp từ các hợp đồng đơn giản hơn, và cách thứ sáu..., `Close`,...là một hợp đồng đơn giản. Tại mỗi bước thực thi, bên cạnh việc trả về một trạng thái mới và hợp đồng tiếp tục, có thể xảy ra các hiệu ứng... -- payments -- và warnings cũng có thể được tạo ra.

Việc giải thích những hợp đồng này, chúng tôi cũng sẽ giải thích thông qua các giá trị (_values)_, quan sát (_observations)_ và hành động (_actions)_, chúng được sử dụng để cung cấp thông tin bên ngoài và đầu vào cho một hợp đồng đang chạy nhằm kiểm soát cách mà nó sẽ tiến triển.

### Pay​ <a href="#pay" id="pay"></a>

Một hợp đồng thanh toán `Pay a p t v cont` sẽ tạo ra một khoản thanh toán `v` của mã thông báo `t` từ tài khoản `a` tới người nhận `p`, _p_ sẽ là một trong những người tham gia hợp đồng hoặc một tài khoản khác trong hợp đồng.&#x20;

Cảnh báo sẽ được tạo ra nếu giá trị `v` là âm, hoặc số dư không đủ trong tài khoản để thực hiện thanh toán đầy đủ (ngay cả khi có số dư dương của các token khác trong tài khoản). Trong trường hợp sau, một khoản thanh toán một phần (của tất cả số tiền có sẵn) sẽ được thực hiện. Hợp đồng tiếp tục là hợp đồng được cung cấp trong hợp đồng: `cont`.

### Close​ <a href="#close" id="close"></a>

`Close` thể hiện cho việc hợp đồng được đóng (hoặc chấm dứt). Hành động duy nhất mà nó thực hiện là hoàn tiền cho các chủ tài khoản có số dư dương. Điều này được thực hiện một tài khoản mỗi bước, nhưng tất cả các tài khoản sẽ được hoàn tiền trong một giao dịch duy nhất.

Trước khi thảo luận sâu hơn về các dạng hợp đồng khác, chúng ta sẽ xem qua về _values_, _observations_ và _actions_.

### Giá trị, quan sát và hành động​ <a href="#values-observations-and-actions" id="values-observations-and-actions"></a>

**Giá trị (Values)** bao gồm một số lượng thay đổi theo thời gian, bao gồm "số slot hiện tại", "số dư hiện tại của một số token trong một tài khoản", và bất kỳ lựa chọn nào đã được thực hiện; chúng tôi gọi đây là các giá trị không ổn định (_volatile)_ .&#x20;

Các giá trị cũng có thể được kết hợp bằng cách sử dụng phép cộng, phép trừ, phép phủ định, phép nhân và phép chia, và có thể phụ thuộc vào một quan sát. Mặc dù chúng được Marlowe hỗ trợ, việc sử dụng phép nhân và phép chia có thể khiến quá trình phân tích tĩnh trở nên khó khăn....&#x20;

**Quan sát (Observations)** là các giá trị Boolean được tạo ra bằng cách so sánh các giá trị, và có thể được kết hợp bằng cách sử dụng các toán tử Boolean tiêu chuẩn. Cũng có thể quan sát xem bất kỳ lựa chọn nào đã được thực hiện (đối với một lựa chọn cụ thể đã được xác định)

Các **quan sát** sẽ có một giá trị tại mỗi bước thực thi. Mặt khác, các **hành động** xảy ra tại những thời điểm cụ thể trong quá trình thực thi. Như đã đề cập trước đó, các hành động có thể là:

* Nạp tiền
* Thực hiện một lựa chọn giữa các lựa chọn khác nhau, bao gồm một giá trị _oracle_ (xem phần tiếp theo)
* Thông báo một giá trị bên ngoài nào đó.

### Oracles​ <a href="#oracles" id="oracles"></a>

Các oracle đang được phát triển cho blockchain Cardano nói chung, và sẽ có sẵn để sử dụng trong Marlowe trên Cardano. Trong thời gian chờ đợi, chúng tôi đã giới thiệu một nguyên mẫu (_prototype_) oracle, được triển khai trong Marlowe Playground.&#x20;

Chúng tôi mô hình hóa các Oracle như những lựa chọn được thực hiện bởi một người tham gia với một vai trò Oracle cụ thể.: `"kraken"`.

Nếu một vai trò (_role_) trong hợp đồng là `"kraken"`, và role đó đưa ra một lựa chọn như `"dir-adausd"` khi đó trong trình mô phỏng của Playground, lựa chọn này sẽ được điền sẵn (_pre-filled)_, dựa trên dữ liệu từ Cryptowat.ch, với giá trị hiện tại của tỷ lệ chuyển đổi trực tiếp ADA/USD. Bạn có thể tìm thấy tất cả các cặp tiền tệ được hỗ trợ ở đây:

[https://api.cryptowat.ch/markets/kraken](https://api.cryptowat.ch/markets/kraken)

Cũng có thể lấy các tỷ lệ nghịch của các cặp tiền tệ đã liệt kê bằng cách thêm tiền tố `inv-`. Ví dụ, `"inv-adausd"` hệ thống sẽ trả về giá trị của tỷ lệ chuyển đổi USD/ADA.

Lưu ý rằng Marlowe hiện tại chỉ hỗ trợ các **số nguyên** như đầu vào lựa chọn. Vậy làm thế nào chúng ta có thể sử dụng giá hiện tại của ADA/USD, có thể là $0.098924? Chúng tôi đơn giản nhân giá với 1 0 8 10 8 , vì vậy giá sẽ xuất hiện dưới dạng 9892400. Bạn có thể sử dụng `DivValue` với giá trị thu được sau khi thực hiện các phép tính của bạn.&#x20;

Ví dụ, bạn muốn mua USDT với giá 12 ADA, sử dụng giá Oracle.&#x20;

Lấy giá:

```rust
When [Choice (ChoiceId "dir-adausdt" (Role "kraken") [Bound 1000000 10000000] ...
```

Tính toán số lượng USDT

```rust
Let (ValueId "amount of USDT in microcents")
  (MulValue
    (Constant 12)
    (ChoiceValue (ChoiceId "dir-adausdt" (Role "kraken"))))
```

Chia kết quả cho 1 0 6 10 6 để có được số tiền bằng đơn vị cent USDT

```rust
DivValue (UseValue (ValueId "amount of USDT in microcents")) (Constant 1000000)
```

### If​ <a href="#if" id="if"></a>

Nhóm câu lệnh điều kiện `If obs cont1 cont2` sẽ tiếp tục với `cont1` or `cont2`, phụ thuộc vào giá trị Boolean của quan sát `obs` khi cấu trúc này được thực thi.

Đây là cấu trúc phức tạp nhất cho các hợp đồng, có dạng: `When cases timeout cont`. Nó là một hợp đồng con dùng để kích hoạt các hành động (_triggered on actions)_, có thể xảy ra hoặc không xảy ra tại bất kỳ slot nào cụ thể: những gì xảy ra khi các hành động khác nhau xảy ra được mô tả bởi các trường hợp trong hợp đồng.

Trong một hợp đồng có điều kiện `When cases timeout cont`, danh sách các trường hợp (`cases)` bao gồm một tập hợp các trường hợp. Mỗi trường hợp sẽ theo dạng `Case ac co` trong đó `ac` là một hành động và `co` là tiếp tục một hợp đồng khác. Khi một hành động cụ thể, chẳng hạn như `ac`, xảy ra, trạng thái sẽ được cập nhật tương ứng và hợp đồng sẽ tiếp tục với phần tiếp theo tương ứng `co`.

Để đảm bảo rằng hợp đồng tiến triển cuối cùng, hợp đồng `When cases timeout cont` sẽ tiếp tục với `cont` khi timeout - thời điểm một số slot cụ thể.

### Let​ <a href="#let" id="let"></a>

Hợp đồng `Let id val cont` là dạng hợp đồng cho phép một hợp đồng gán tên cho một giá trị bằng cách sử dụng một định danh. Trong trường hợp này, biểu thức `val` được đánh giá và lưu trữ với tên `id`. Sau đó, hợp đồng tiếp tục với `cont`.

Ngoài việc cho phép sử dụng các ký hiệu viết tắt, cơ chế này cũng giúp chúng ta có thể lưu giữ các giá trị biến động có thể thay đổi theo thời gian, ví dụ: giá dầu hiện tại hoặc số slot hiện tại, tại một thời điểm cụ thể trong quá trình thực thi hợp đồng, để sử dụng sau này trong quá trình thực thi hợp đồng.

### Assert​ <a href="#assert" id="assert"></a>

Hợp đồng `Assert obs cont` không có bất kỳ ảnh hưởng nào đến trạng thái của hợp đồng; nó ngay lập tức tiếp tục với `cont`, nhưng sẽ đưa ra cảnh báo khi quan sát `obs` là sai. Nó có thể được sử dụng để đảm bảo rằng một thuộc tính nào đó luôn đúng tại bất kỳ điểm nào trong hợp đồng, vì phân tích tĩnh sẽ thất bại nếu bất kỳ quá trình thực thi nào gây ra cảnh báo.
