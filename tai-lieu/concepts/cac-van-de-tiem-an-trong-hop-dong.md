# Các vấn đề "tiềm ẩn" trong hợp đồng

Ngôn ngữ Marlowe được thiết kế để có càng ít cạm bẫy và bất ngờ càng tốt, nhằm giúp việc viết hợp đồng trở nên trực quan và tránh được những điều không mong muốn. Tuy nhiên, không thể loại trừ hoàn toàn các hợp đồng không nên được viết, mà không làm cho Marlowe trở nên khó sử dụng hơn. Hơn nữa, ngay cả khi một hợp đồng được viết đúng cách, người dùng vẫn có thể tương tác với nó theo những cách không hợp lệ, bằng cách phát hành các giao dịch không hợp lệ.

Trong tất cả các trường hợp, khi những hiệu ứng không mong muốn này xảy ra, Marlowe được thiết kế để hoạt động theo cách trực quan và bảo thủ nhất có thể. Tuy nhiên, điều đáng lưu ý là những vấn đề tiềm ẩn này, và xem xét cách Marlowe hoạt động trong những tình huống này. Đó là nội dung của hướng dẫn này.

#### Cảnh báo

Cảnh báo của Marlowe là những chỉ báo rằng một hợp đồng được viết sai. Một hợp đồng được viết tốt không bao giờ nên phát hành cảnh báo, bất kể cách mà người dùng tương tác với nó. Lý tưởng nhất, chúng ta muốn cấm các hợp đồng có thể phát hành cảnh báo khỏi việc được viết, nhưng điều đó sẽ yêu cầu các hợp đồng Marlowe phải có kiểu phụ thuộc, và việc viết các biểu thức kiểu phụ thuộc thì khó khăn hơn nhiều.

#### Phân tích tĩnh

Marlowe cho phép viết các hợp đồng phát hành cảnh báo, và chúng tôi cung cấp các công cụ phân tích tĩnh cho phép các nhà phát triển hợp đồng kiểm tra xem một hợp đồng cụ thể có thể phát hành cảnh báo hay không. Ngoài ra, chúng tôi cung cấp các hành vi dự phòng cho trường hợp hợp đồng phát sinh cảnh báo, bất chấp lời khuyên của chúng tôi. Chúng tôi cung cấp các hành vi dự phòng vì chúng tôi nhận thức rằng việc phân tích các hợp đồng lớn có thể rất tốn kém về mặt tính toán, và vì có thể có sai sót xảy ra. Chúng tôi muốn các hợp đồng viết sai thất bại theo cách ít gây hại nhất có thể, tức là theo cách bảo thủ.

#### Thanh toán không dương

Khi một hợp đồng dự kiến thanh toán một số tiền nhỏ hơn một đơn vị của một loại tiền tệ hoặc token, nó sẽ phát sinh cảnh báo `NonPositivePay` và sẽ không chuyển bất kỳ khoản tiền nào.

Các khoản thanh toán âm nên được thực hiện dưới dạng các khoản tiền gửi **dương** (khi thanh toán cho một người tham gia) hoặc các khoản thanh toán dương theo hướng ngược lại (khi thanh toán giữa các tài khoản).

#### Khoản nạp không dương <a href="#non-positive-deposits" id="non-positive-deposits"></a>

Khi một hợp đồng dự kiến nhận một số tiền nhỏ hơn một đơn vị của loại tiền tệ hoặc token, nó vẫn sẽ chờ một giao dịch IDeposit, nhưng giao dịch đó không cần phải chuyển bất kỳ khoản tiền nào vào hợp đồng và không có tiền nào được chuyển cho người tham gia phát hành giao dịch đó. Sau khi khoản tiền gửi "giả" này thành công, hợp đồng sẽ phát sinh cảnh báo NonPositiveDeposit.

Các khoản tiền gửi âm nên luôn được thực hiện dưới dạng các khoản thanh toán dương.

#### Thanh toán một phần

Khi một hợp đồng dự kiến thanh toán một số tiền lớn hơn số tiền có trong tài khoản nguồn, nó sẽ chỉ chuyển bất kỳ số tiền nào có sẵn trong tài khoản đó, ngay cả khi có đủ tiền trong tất cả các tài khoản của hợp đồng, và nó sẽ phát sinh cảnh báo PartialPay.

Nên tránh các khoản thanh toán một phần vì một hợp đồng không bao giờ tạo ra khoản thanh toán một phần là một hợp đồng rõ ràng. Các hợp đồng rõ ràng giúp người dùng yên tâm rằng chúng sẽ có thể thực thi, và rằng bất kỳ nơi nào trong hợp đồng nói rằng một khoản thanh toán sẽ diễn ra, thì nó sẽ thực sự xảy ra.

#### `Let` Shadowing (không được phân tích tĩnh)

Khi một hợp đồng đạt đến một cấu trúc `Let` mà định nghĩa lại một giá trị bằng một định danh đã được định nghĩa bởi một Let bên ngoài, hợp đồng sẽ phát sinh cảnh báo **Shadowing** và nó sẽ ghi đè lên định nghĩa trước đó.

Shadowing là một thực hành lập trình xấu vì nó dẫn đến sự nhầm lẫn. Việc sử dụng cùng một định danh cho nhiều thứ có thể khiến các nhà phát triển hoặc người dùng nghĩ rằng một lần sử dụng của nó sẽ được đánh giá thành một giá trị nào đó trong khi thực tế lại được đánh giá thành một giá trị khác.

### Bad smells​ <a href="#bad-smells" id="bad-smells"></a>

Có một số dấu hiệu "xấu" khác cho thấy hợp đồng có thể đã được thiết kế kém.

Những hợp đồng này là hợp lệ, nghĩa là chúng không nhất thiết phải gây ra bất kỳ cảnh báo nào, và chúng thực hiện những gì chúng nói rằng chúng sẽ làm, nhưng chúng có những đặc điểm cho thấy hoặc là nhà phát triển hợp đồng không hoàn toàn nhận thức được hậu quả của hợp đồng, hoặc nhà phát triển cố ý viết hợp đồng theo cách gây nhầm lẫn cho người đọc.

#### Sử dụng `Let` không xác định (nên phát sinh cảnh báo)

Khi một cấu trúc `Use` sử dụng một định danh chưa được định nghĩa, nó sẽ đánh giá thành giá trị mặc định là `0`. Không có cảnh báo nào được phát sinh, nhưng đây lại là một thực hành xấu vì nó có thể gây hiểu lầm. Thay vào đó, nên sử dụng (`Constant 0`) vì điều này làm rõ số tiền đang được đề cập.

#### Các phần không thể tiếp cận trong hợp đồng

Đây là dấu hiệu xấu chính trong các hợp đồng Marlowe. Nếu một phần của hợp đồng không thể tiếp cận, tại sao nó lại được bao gồm ngay từ đầu?

Dấu hiệu xấu này có nhiều hình thức khác nhau.

**Hợp đồng phụ** (Sub-Contract) **không thể tiếp cận**

Ví dụ:

```rust
If FalseObs contract1 contract2
```

Hợp đồng trước đó tương đương với hợp đồng `contract2`. Nói chung, bạn không nên bao giờ sử dụng `FalseObs`, và bạn chỉ nên sử dụng `TrueObs` như là Observation gốc của một cấu trúc `Case`.

**`Observation` luôn luôn là một lối tắt**

Ví dụ:

```rust
OrObs TrueObs observation1
```

Quan sát trước đó tương đương với `observation1`. Một lần nữa, bạn chỉ nên sử dụng TrueObs làm quan sát gốc của một cấu trúc Case.

**`When` Nhánh là không thể tiếp cận.**

Ví dụ:

```rust
When [ Case (Notify TrueObs) contract1
     , Case (Notify TrueObs) contract2 ]
     10
     contract3
```

`contract2` không thể tiếp cận, toàn bộ `Case` có thể bị loại bỏ khỏi hợp đồng và hành vi của hợp đồng sẽ không thay đổi. Điều này có thể dẫn đến việc mã trở nên khó hiểu và không cần thiết.

**Thời gian chờ&#x20;**_**lồng nhau**_**&#x20;không tăng**

```rust
When []
     10
     When [ Case (Notify TrueObs)
                 contract1 ]
          10
          contract2
```

`contract1` Không thể truy cập: sau khối số `10`, hợp đồng sẽ trực tiếp chuyển đổi thành `contract2`. `When` bên trong không tạo ra sự khác biệt nào cho hợp đồng.

### Vấn đề khả năng sử dụng

Ngay cả khi một hợp đồng tránh được các cảnh báo và không có mã không thể truy cập, nó vẫn có thể cho phép những người dùng độc hại ép buộc những người dùng khác vào các tình huống không mong muốn mà nhà phát triển hợp đồng không có ý định ban đầu.

### Thời điểm không hợp lý của các cấu trúc `When`

Xem xét hợp đồng sau:

```rust
When [Case (Choice (ChoiceId "choice1" (Role "alice")) [Bound 0 10])
           (When [Case (Choice (ChoiceId "choice2" (Role "bob")) [Bound 0 10])
                       Close
                 ]
            10
            (Pay (Role "bob") (Party (Role "alice"))
                 ada
                 (Constant 10)
                 Close
            )
        )
     ]
     10
     Close
```

Không có gì sai về nguyên tắc với hợp đồng này, nhưng nếu (Vai trò "alice") đưa ra lựa chọn của mình vào khối 9, thì sẽ gần như không thể cho bob đưa ra lựa chọn của mình kịp thời và nhận lại tiền trong tài khoản của mình (Vai trò "bob"). Trừ khi đây là một phần của một trò chơi và đó là hiệu ứng mong muốn, thì đây có khả năng là một hợp đồng không công bằng đối với (Vai trò "bob").

Nói chung, là một thói quen tốt để đảm bảo rằng các cấu trúc `When` có thời hạn tăng dần, và sự tăng lên giữa các thời hạn là hợp lý để các bên khác nhau có thể thực hiện và nhận các giao dịch của họ được chấp nhận bởi blockchain.&#x20;

Có nhiều lý do khiến sự tham gia của một bên có thể bị trì hoãn: sự cố cung cấp năng lượng, sự gia tăng đột ngột trong số lượng giao dịch đang chờ xử lý trong blockchain, các cuộc tấn công mạng, v.v. Vì vậy, điều quan trọng là phải cho phép đủ thời gian, và rộng rãi với các thời hạn và với sự gia tăng thời hạn.

### Lỗi

Cuối cùng, ngay cả khi một hợp đồng được viết hoàn hảo. Người dùng có thể sử dụng nó sai cách, và chúng tôi gọi những cách sử dụng sai đó là lỗi.

Trong tất cả các trường hợp, bất cứ khi nào một giao dịch gây ra lỗi, giao dịch sẽ không có hiệu lực đối với Hợp đồng hoặc Trạng thái của nó.&#x20;

Trên thực tế, ví của người dùng sẽ biết trước liệu một giao dịch có gây ra lỗi hay không, vì các giao dịch là xác định, vì vậy người dùng không bao giờ cần gửi một giao dịch sai đến blockchain.

**Khoảng thời gian không rõ ràng**

Khi một giao dịch đến một thời điểm timeout, khoảng thời gian của nó phải không mập mờ về việc thời hạn đã qua hay chưa. Ví dụ, nếu `When` ở trên cùng của một hợp đồng có thời hạn 1700003600 và một giao dịch với khoảng thời gian \[1700000000, 1700007200] được phát hành, giao dịch sẽ gây ra lỗi `AmbiguousTimeIntervalError`, vì không thể biết liệu thời hạn đã qua chỉ bằng cách nhìn vào giao dịch. Để tránh điều này, giao dịch phải được chia thành hai giao dịch riêng biệt:

* Một giao dịch với khoảng thời gian \[1700000000, 1700003599].
* Một giao dịch khác với khoảng thời gian \[1700003600, 1700007200].

**Áp dụng không khớp**

Nếu một giao dịch không cung cấp các đầu vào mà Hợp đồng mong đợi, thì hợp đồng sẽ phát sinh lỗi `NoMatchError`, và toàn bộ giao dịch sẽ bị loại bỏ.

**Giao dịch vô dụng**

Nếu một giao dịch không có bất kỳ hiệu lực nào trên Hợp đồng hoặc Trạng thái, nó sẽ dẫn đến lỗi `UselessTransaction`, và toàn bộ giao dịch sẽ bị loại bỏ.&#x20;

Lý do chúng tôi loại bỏ các giao dịch vô dụng là vì chúng mở ra khả năng cho các cuộc tấn công từ chối dịch vụ (DoS), vì một kẻ tấn công tiềm năng có thể làm ngập hợp đồng bằng các giao dịch không cần thiết và ngăn cản các giao dịch cần thiết được đưa vào blockchain.
