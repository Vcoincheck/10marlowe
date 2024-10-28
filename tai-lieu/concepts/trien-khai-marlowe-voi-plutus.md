# Triển khai Marlowe với Plutus

Cho đến nay, các hướng dẫn này đã đề cập đến Marlowe như một sản phẩm độc lập; hướng dẫn này mô tả cách Marlowe được triển khai trên blockchain, sử dụng 'mockchain' cung cấp một mô phỏng chất lượng cao của lớp Cardano SL.

### **Triển khai**

Để triển khai các hợp đồng Marlowe, chúng tôi sử dụng trình biên dịch **PlutusTx**, trình biên dịch mã Haskell thành mã Plutus Core đã được tuần tự hóa, để tạo ra một kịch bản xác thực Cardano đảm bảo việc thực thi hợp đồng chính xác. Hình thức triển khai này dựa trên các phần mở rộng cho mô hình UTXO được mô tả trong tài liệu này.

Việc thực thi hợp đồng Marlowe trên blockchain bao gồm một chuỗi giao dịch, trong đó, ở mỗi giai đoạn, hợp đồng còn lại và trạng thái của nó được chuyển qua kịch bản dữ liệu, và các hành động cùng với đầu vào (tức là các lựa chọn và giá trị oracle) được chuyển qua kịch bản thu hồi.&#x20;

Mỗi bước trong việc thực thi hợp đồng là một giao dịch tiêu tốn một đầu ra kịch bản hợp đồng Marlowe bằng cách cung cấp một đầu vào hợp lệ trong kịch bản thu hồi, và tạo ra một đầu ra giao dịch với một hợp đồng Marlowe như một tiếp tục (hợp đồng còn lại) ngoài các đầu vào và đầu ra khác.

### **Không gian thiết kế**

Có nhiều cách để triển khai các hợp đồng Marlowe trên nền tảng Plutus. Chúng tôi có thể viết một trình biên dịch từ Marlowe sang Plutus để chuyển đổi từng hợp đồng Marlowe thành một kịch bản Plutus cụ thể. Tuy nhiên, chúng tôi đã chọn triển khai một trình thông dịch hợp đồng Marlowe.&#x20;

Phương pháp này có một số lợi thế:

* Đơn giản: chúng tôi triển khai một kịch bản Plutus duy nhất có thể được sử dụng cho tất cả các hợp đồng Marlowe, do đó giúp dễ dàng hơn trong việc triển khai, xem xét và kiểm tra những gì chúng tôi đã thực hiện.
* Gần gũi với ngữ nghĩa của Marlowe, như đã mô tả trong hướng dẫn trước, vì vậy việc xác thực dễ dàng hơn.
* Điều này có nghĩa là cùng một triển khai có thể được sử dụng cho cả việc thực thi trên chuỗi và ngoài chuỗi (ví dụ: ví) của mã Marlowe.
* Nó cho phép đánh giá hợp đồng phía khách hàng, nơi chúng tôi tái sử dụng mã tương tự để mô phỏng việc thực thi hợp đồng (ví dụ: trong IDE), và biên dịch nó sang WASM/JavaScript ở phía khách hàng (ví dụ: trong Plutus hoặc Marlowe Playground).
* Việc có một trình thông dịch duy nhất cho tất cả (hoặc một nhóm cụ thể) các hợp đồng Marlowe cho phép theo dõi blockchain cho những loại hợp đồng này, nếu mong muốn.
* Cuối cùng, có tiềm năng để đặc biệt hóa loại kịch bản này và triển khai một trình thông dịch chuyên biệt, hiệu quả cao trong Cardano CL.

Trong triển khai của chúng tôi, chúng tôi lưu trữ hợp đồng còn lại trong kịch bản dữ liệu, điều này làm cho nó trở nên rõ ràng với mọi người. Điều này đơn giản hóa việc phản ánh và xem xét hợp đồng.

### **Vòng đời hợp đồng trên mô hình UTXO mở rộng**

Triển khai hiện tại hoạt động trên mockchain, như đã mô tả trong TODO. Chúng tôi mong rằng sẽ chỉ cần thực hiện một số thay đổi tối thiểu để chạy trên triển khai sản xuất vì mockchain được thiết kế để trung thành với điều đó.

Như đã mô tả ở trên, trình thông dịch Marlowe được hiện thực hóa như một kịch bản xác thực. Chúng tôi có thể chia việc thực thi hợp đồng Marlowe thành ba giai đoạn: khởi tạo/tạo, thực thi và hoàn tất.

**Khởi tạo/Tạo**:&#x20;

* Khởi tạo và tạo hợp đồng được hiện thực hóa như một giao dịch với ít nhất một đầu ra kịch bản (hiện tại phải là đầu ra đầu tiên), với hợp đồng Marlowe cụ thể trong kịch bản dữ liệu, và được bảo vệ bởi kịch bản xác thực Marlowe.&#x20;
* Giao dịch này phải đưa một số tiền (ít nhất một Lovelace) vào đầu ra giao dịch đó, để nó trở thành một đầu ra giao dịch chưa tiêu (UTXO). Chúng tôi coi giá trị này là một khoản tiền gửi hợp đồng, có thể được sử dụng trong giai đoạn hoàn tất.&#x20;
* Lưu ý rằng chúng tôi không đặt bất kỳ hạn chế nào đối với các đầu vào giao dịch, có thể sử dụng bất kỳ đầu ra giao dịch nào khác, bao gồm cả các kịch bản.&#x20;
* Có thể khởi tạo một hợp đồng với một trạng thái cụ thể, chứa một số cam kết, như đã trình bày ở đây.

**Thực thi**: Việc thực thi hợp đồng Marlowe bao gồm một chuỗi giao dịch, trong đó hợp đồng và trạng thái còn lại được chuyển qua kịch bản dữ liệu, và các hành động cùng với đầu vào (tức là các lựa chọn và giá trị oracle) được chuyển qua các kịch bản thu hồi.

Mỗi bước là một giao dịch tiêu tốn đầu ra kịch bản hợp đồng Marlowe bằng cách cung cấp một đầu vào hợp lệ trong kịch bản thu hồi, và tạo ra một đầu ra giao dịch với một hợp đồng Marlowe như một tiếp tục, như có thể thấy ở đây.

Trình thông dịch Marlowe trước tiên xác thực hợp đồng và trạng thái hiện tại. Tức là, chúng tôi kiểm tra rằng hợp đồng sử dụng đúng các định danh, và giữ ít nhất những gì nó nên có, đó là khoản tiền gửi và các cam kết chưa thanh toán. Chúng tôi sau đó đánh giá hợp đồng tiếp tục và trạng thái của nó, sử dụng hàm `eval`.

```rust
eval :: Input -> Slot -> Ada -> Ada -> State -> Contract -> (State,Contract,Bool)
```

using the current slot number and at the same time checking that the input makes sense and that payments are within committed bounds; if the input is valid then it returns the new `State` and `Contract` and the Boolean `True`; otherwise it returns the current `State` and `Contract`, unchanged, together with the value `False`.

In a little more detail, in the type of `eval` above, `Input` is a combination of contract participant actions (i.e. `Commit`, `Payment`, `Redeem`), oracle values, and choices made. The two Ada parameters are the _current_ contract value, and the _result_ contract value. So, for example, if the contract is to perform a 20 Ada Payment and the input current amount is 100 Ada, then the result value will be 80 Ada. The `Contract` and `State` values are the current contract and its `State`, respectively, taken from the data script.

In general, on-chain code cannot generate transaction outputs, but can only validate whatever a user provides in a transaction. Every step in contract evaluation is created by a user, either manually or automatically (by a wallet, say), and published as a transaction. A user may therefore provide any `Contract` and its `State` as continuation. For example, consider the following contract

Sử dụng số slot hiện tại và đồng thời kiểm tra rằng đầu vào có ý nghĩa và rằng các khoản thanh toán nằm trong giới hạn cam kết; nếu đầu vào hợp lệ, thì nó trả về trạng thái `(State)` mới và hợp đồng `(Contract)` cùng với giá trị Boolean `True`; nếu không, nó trả về `State` và `Contract` hiện tại không thay đổi, cùng với giá trị `False`.

Một cách chi tiết hơn, trong kiểu của hàm `eval` ở trên, `Input` là sự kết hợp của các hành động của người tham gia hợp đồng (tức là `Commit, Payment, Redeem`), các giá trị oracle và các lựa chọn được thực hiện.&#x20;

Hai tham số Ada là giá trị hợp đồng hiện tại và giá trị hợp đồng kết quả. Vì vậy, ví dụ, nếu hợp đồng là thực hiện một khoản thanh toán 20 Ada và số tiền _hiện tại_ của _đầu vào_ là 100 Ada, thì giá trị kết quả sẽ là 80 Ada. Các giá trị `Contract` và `State` là hợp đồng hiện tại và trạng thái của nó, tương ứng, được lấy từ kịch bản dữ liệu.

Nói chung, mã trên chuỗi không thể tạo ra đầu ra giao dịch, mà chỉ có thể xác thực bất cứ điều gì mà người dùng cung cấp trong một giao dịch.&#x20;

Mỗi bước trong việc đánh giá hợp đồng được tạo ra bởi một người dùng, hoặc thủ công hoặc tự động (ví dụ: bằng 1 ví), và được công bố dưới dạng một giao dịch. Do đó, người dùng có thể cung cấp bất kỳ `Contract` và `State` nào của nó như là điều kiện để **tiếp tục**. Ví dụ, hãy xem xét hợp đồng sau đây.

```rust
Commit id Alice 100 (Both (Pay Alice to Bob 30 Ada) (Pay Alice to Charlie 70 Ada))
```

`Alice` cam kết 100 Ada và sau đó cả `Bob` và `Charlie` có thể thu thập 30 và 70 Ada mỗi người bằng cách phát hành giao dịch liên quan. Sau khi `Alice` đã thực hiện một cam kết, hợp đồng trở thành

```rust
Both (Pay Alice to Bob 30 Ada) (Pay Alice to Charlie 70 Ada)
```

`Bob` có thể phát hành một giao dịch với đầu vào Thanh toán `(Payment)` trong kịch bản thu hồi (**redeemer script**), và một đầu ra kịch bản với giá trị giảm 30 Ada, được bảo vệ bởi kịch bản xác thực Marlowe và với kịch bản dữ liệu chứa hợp đồng tiếp tục đã được đánh giá.

```rust
Pay Alice to Charlie 70 Ada
```

`Charlie` Có thể phát hành một giao dịch tương tự để nhận 70 Ada còn lại.

**Đảm bảo tính hợp lệ của việc thực thi.**&#x20;

Nhìn lại ví dụ này, giả sử thay vào đó Bob chọn cách xấu, phát hành một giao dịch với sự tiếp diễn sau đây: và lấy tất cả số tiền, như ở đây, khiến Charlie cảm thấy thất vọng hợp lý với tất cả những hợp đồng thông minh đó.&#x20;

Để tránh điều này, chúng ta phải đảm bảo rằng hợp đồng tiếp diễn mà chúng ta đánh giá phải bằng với hợp đồng trong kịch bản dữ liệu của đầu ra giao dịch của nó. Đây là phần khó khăn trong việc triển khai, vì chúng ta chỉ có hash của kịch bản dữ liệu của các đầu ra giao dịch có sẵn trong quá trình thực thi kịch bản xác thực.&#x20;

Nếu chúng ta có thể truy cập kịch bản dữ liệu trực tiếp, chúng ta có thể đơn giản kiểm tra rằng hợp đồng mong đợi bằng với hợp đồng được cung cấp. Nhưng điều đó sẽ làm phức tạp mọi thứ hơn nữa, vì chúng ta cần phải biết kiểu của tất cả các kịch bản dữ liệu trong một giao dịch, điều này là không thể thực hiện được một cách tổng quát.

Mẹo là yêu cầu kịch bản đổi trả đầu vào (`input redeemer script)` và kịch bản dữ liệu đầu ra `(output data script)` phải bằng nhau.&#x20;

Cả kịch bản đổi trả và kịch bản dữ liệu đều có cùng cấu trúc, cụ thể là một cặp `(Input, MarloweData)` trong đó&#x20;

* `Input` chứa các hành động hợp đồng `(Payment, Redeem)`, `Choices` và các giá trị `Oracle`.
* `MarloweData` chứa hợp đồng còn lại và trạng thái của nó.
* Trạng thái `(State)` ở đây là một tập hợp các `Commits` cộng với một tập hợp các lựa chọn `(Choices)` đã thực hiện.&#x20;

Để chi tiêu một đầu ra giao dịch được bảo vệ bởi kịch bản xác thực Marlowe, người dùng phải cung cấp một kịch bản đổi trả, là một cặp của `Input` và đầu ra mong đợi của việc diễn giải một hợp đồng Marlowe cho `Input` đã cho, tức là một cặp `Contract`, `State`. Hợp đồng và trạng thái mong đợi có thể được đánh giá chính xác trước đó bằng cách sử dụng hàm eval.&#x20;

Để đảm bảo rằng người dùng cung cấp `Contract` và `State` còn lại hợp lệ, kịch bản xác thực Marlowe sẽ so sánh hợp đồng và trạng thái đã đánh giá với những gì được cung cấp bởi người dùng, và sẽ từ chối giao dịch nếu chúng không khớp.&#x20;

Để đảm bảo rằng kịch bản dữ liệu của hợp đồng còn lại có cùng `Contract` and `State` như đã được truyền với kịch bản đổi trả, chúng tôi kiểm tra rằng hash của kịch bản dữ liệu là giống như của kịch bản đổi trả.

**Hoàn tất.**&#x20;

Khi một hợp đồng đánh giá thành Null và tất cả các Commits đã hết hạn được thanh toán, khoản tiền đặt cọc ban đầu của hợp đồng có thể được chi tiêu, loại bỏ hợp đồng khỏi tập hợp các đầu ra giao dịch chưa chi tiêu.

> **Exercise**
>
> _Advanced._ Explore running Marlowe contracts in Plutus. In order to be able to do this you will need to use the latest version of Marlowe, rather than `v1.3`.
