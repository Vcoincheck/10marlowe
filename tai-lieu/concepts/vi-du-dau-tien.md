# Ví dụ đầu tiên

Hướng dẫn này giới thiệu một hợp đồng tài chính đơn giản bằng mã giả, trước khi giải thích cách nó được điều chỉnh để hoạt động trong Marlowe, cung cấp ví dụ đầu tiên về một hợp đồng Marlowe.

### Hợp đồng ký quỹ (escrow) đơn giản​ <a href="#a-simple-escrow-contract" id="a-simple-escrow-contract"></a>

Giả sử rằng `alice` muốn mua một con mèo từ `bob`, but neither of them trusts the other. Nhưng cả hai đều không tin tưởng lẫn nhau. May mắn thay, họ có một người bạn chung là `carol` người mà cả hai đều tin tưởng sẽ giữ thái độ trung lập (nhưng không đủ tin tưởng để giao tiền cho cô ấy và nhờ cô ấy làm trung gian). Do đó, họ đồng ý với hợp đồng sau, được viết bằng mã giả chức năng đơn giản. Loại hợp đồng này là một ví dụ đơn giản về hợp đồng ký quỹ (escrow).

```rust
When aliceChoice
     (When bobChoice
           (If (aliceChosen `ValueEQ` bobChosen)
               agreement
               arbitrate))
```

Hợp đồng được mô tả bằng cách sử dụng các constructor của một kiểu dữ liệu Haskell. Constructor ngoài cùng, `When`, có hai tham số: tham số đầu tiên là _một quan sát_ (observation) và tham số thứ hai là _một hợp đồng khác_. Ý nghĩa dự định của điều này là khi hành động xảy ra, hợp đồng thứ hai sẽ được kích hoạt.

Hợp đồng thứ hai thực sự là một `When` khác — yêu cầu quyết định từ Bob — nhưng bên trong đó có một lựa chọn: nếu Alice và Bob đồng ý về điều cần làm, hành động sẽ được thực hiện; nếu không, Carol sẽ được yêu cầu phân xử và đưa ra quyết định.

Nói chung, `When` cung cấp một danh sách các trường hợp (cases), mỗi trường hợp có một hành động và một hợp đồng tương ứng được kích hoạt khi hành động đó xảy ra. Bằng cách này, chúng ta có thể cho phép Bob có tùy chọn thực hiện lựa chọn đầu tiên, thay vì Alice, như sau:

```rust
When [ Case aliceChoice
            (When [ Case bobChoice
                        (If (aliceChosen `ValueEQ` bobChosen)
                           agreement
                           arbitrate) ],
       Case bobChoice
            (When [ Case aliceChoice
                        (If (aliceChosen `ValueEQ` bobChosen)
                            agreement
                            arbitrate) ]
     ]
```

Trong hợp đồng này, hoặc Alice hoặc Bob có thể thực hiện lựa chọn đầu tiên; người còn lại sau đó sẽ đưa ra lựa chọn. Nếu họ đồng ý, thì hành động sẽ được thực hiện; nếu không, Carol sẽ phân xử. Trong phần còn lại của hướng dẫn, chúng ta sẽ quay lại phiên bản đơn giản hơn, trong đó Alice thực hiện lựa chọn đầu tiên.

> **Bài tập**
>
> Hãy nghĩ về việc thực hiện hợp đồng này trong thực tế. Giả sử Alice đã cam kết một số tiền cho hợp đồng. Điều gì sẽ xảy ra nếu Bob chọn không tham gia thêm nữa?
>
> Chúng ta đã giả định rằng Alice đã cam kết thanh toán của cô ấy, nhưng giả sử rằng chúng ta muốn thiết kế một hợp đồng để đảm bảo điều đó: chúng ta sẽ cần làm gì?

### Ký quỹ trong Marlowe​ <a href="#escrow-in-marlowe" id="escrow-in-marlowe"></a>

Hợp đồng Marlowe bao gồm các cấu trúc bổ sung để đảm bảo rằng chúng tiến triển đúng cách. Mỗi lần chúng ta sử dụng _`When`_, chúng ta cần cung cấp hai yếu tố bổ sung:

* Một giới hạn thời gian (timeout) sau đó hợp đồng sẽ tiếp tục tiến triển, và
* Hợp đồng tiếp nối mà nó sẽ tiến triển tới.

### Thêm **timeouts**

Trước tiên, hãy xem xét cách thay đổi những gì chúng ta đã viết để xử lý trường hợp điều kiện của _`When`_ không bao giờ trở thành đúng. Vì vậy, chúng ta thêm các giá trị giới hạn thời gian (_`timeout`_) và các giá trị tiếp nối (_continuation values_) vào mỗi _`When`_ xuất hiện trong hợp đồng.

```rust
When [ Case aliceChoice
            (When [ Case bobChoice
                        (If (aliceChosen `ValueEQ` bobChosen)
                           agreement
                           arbitrate) ]
                  1700007200    -- ADDED
                  arbitrate)    -- ADDED
      ]
      1700003600   -- ADDED
      Close        -- ADDED
```

`When` bên ngoài yêu cầu lựa chọn đầu tiên được thực hiện bởi Alice: nếu Alice chưa thực hiện lựa chọn nào trước thời gian POSIX `1700003600` (2023-11-14 23:13:20 GMT), hợp đồng sẽ được đóng và tất cả số tiền trong hợp đồng sẽ được hoàn lại.&#x20;

`Close` thường là bước cuối cùng trong mọi "đường đi" qua một hợp đồng Marlowe, và tác dụng của nó là hoàn lại tiền trong hợp đồng cho các bên tham gia; chúng tôi sẽ mô tả điều này chi tiết hơn khi xem xét Marlowe từng bước trong một bài hướng dẫn sau. Trong trường hợp cụ thể này, việc hoàn lại sẽ diễn ra vào thời gian POSIX `1700003600` (2023-11-14 23:13:20 GMT).

Xem xét các cấu trúc bên trong, nếu lựa chọn của Alice đã được thực hiện, thì chúng ta sẽ chờ một lựa chọn từ Bob. Nếu điều đó không xảy ra trước thời gian POSIX `1700007200` (2023-11-15 00:13:20 GMT), thì Carol sẽ được yêu cầu phân xử.

### Thêm các cam kết (commitments​) <a href="#adding-commitments" id="adding-commitments"></a>

Tiếp theo, chúng ta nên xem xét cách tiền mặt được cam kết như là bước đầu tiên của hợp đồng.

```rust
When [Case (Deposit "alice" "alice" ada price)   -- ADDED
 (When [ Case aliceChoice
             (When [ Case bobChoice
                         (If (aliceChosen `ValueEQ` bobChosen)
                            agreement
                            arbitrate) ]
                   1700007200
                   arbitrate)
       ]
       1700003600
       Close)
   ]
   1700000000                              -- ADDED
   Close                                   -- ADDED
```

Một khoản tiền gửi với giá trị `price` được yêu cầu từ "alice": nếu khoản tiền này được cung cấp, thì nó sẽ được giữ trong một tài khoản, cũng gọi là "alice". Các tài khoản như thế này chỉ tồn tại trong suốt thời gian hợp đồng; mỗi tài khoản thuộc về một hợp đồng duy nhất.&#x20;

Có một thời gian chờ tại POSIX time `1700000000` (2023-11-14 22:13:20 GMT) để thực hiện khoản tiền gửi; nếu thời gian đó đạt được mà không có khoản tiền gửi nào được thực hiện, hợp đồng sẽ được đóng lại và tất cả số tiền đã có trong hợp đồng sẽ được hoàn trả. Trong trường hợp này, đó đơn giản là kết thúc hợp đồng.

### **Định nghĩa**

Chúng ta sẽ thấy sau này rằng các phần của mô tả hợp đồng này, chẳng hạn như `arbitrate`, `agreement`, và `price`, sử dụng việc nhúng Haskell của Marlowe DSL để cung cấp một số định nghĩa ngắn gọn.&#x20;

Chúng ta cũng sử dụng chuỗi quá tải để làm cho một số mô tả—ví dụ, về tài khoản—ngắn gọn hơn. Những điều này sẽ được thảo luận chi tiết hơn khi chúng ta xem xét Marlowe được nhúng trong Haskell.

> **Bài tập**\
> Bình luận về sự lựa chọn các giá trị timeout và xem xét các lựa chọn thay thế.\
> Ví dụ, điều gì sẽ xảy ra nếu `timeout` `1700003600` (2023-11-14 23:13:20 GMT) trong phần `When` được thay thế bằng `1700007200` (2023-11-15 00:13:20 GMT), và ngược lại?&#x20;
>
> Có hợp lý không khi có cùng một `timeout`, ví dụ `1700010800` (2023-11-15 01:13:20 GMT) cho mỗi `When`? Nếu không, tại sao không?

Ví dụ này đã cho thấy nhiều thành phần của ngôn ngữ hợp đồng Marlowe; trong bài hướng dẫn tiếp theo, chúng tôi sẽ trình bày toàn bộ ngôn ngữ.

### **Ghi chú**

Trong khi tên của Alice, Bob và những người khác được "gắn chặt" vào hợp đồng ở đây, chúng ta sẽ thấy sau này rằng chúng có thể được đại diện bởi các vai trò trong một tài khoản, chẳng hạn như người mua và người bán.&#x20;

Các vai trò này có thể được liên kết với các người tham gia cụ thể khi một hợp đồng được thực thi; chúng tôi sẽ thảo luận thêm về điều này trong phần tiếp theo.

### **Bối cảnh**

Các tài liệu này đề cập đến công trình ban đầu về việc sử dụng lập trình hàm để mô tả các hợp đồng tài chính.

* [Composing contracts: an adventure in financial engineering](https://www.microsoft.com/en-us/research/publication/composing-contracts-an-adventure-in-financial-engineering/)
* [Certified symbolic management of financial multi-party contracts](https://dl.acm.org/citation.cfm?id=2784747)
