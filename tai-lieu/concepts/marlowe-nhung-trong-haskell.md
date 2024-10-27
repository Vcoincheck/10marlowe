# Marlowe nhúng trong Haskell

Trong hướng dẫn này, chúng ta quay trở lại ví dụ về tài khoản ký quỹ và chỉ cho cách chúng ta có thể sử dụng việc nhúng Marlowe vào Haskell để tạo ra các mô tả hợp đồng Marlowe dễ đọc hơn, có tính mô-đun và có thể tái sử dụng.

### Hợp đồng ký quỹ đơn giản, xem lại

Nhắc lại rằng chúng ta đã phát triển hợp đồng Marlowe này trong hướng dẫn trước. Mặc dù chúng ta đã trình bày nó ở đó như một hợp đồng "đơn khối", nhưng chúng ta có thể sử dụng các định nghĩa trong Haskell để làm cho nó dễ đọc hơn.&#x20;

Để bắt đầu, chúng ta có thể tách biệt cam kết ban đầu khỏi phần hoạt động bên trong của hợp đồng:

```rust
contract :: Contract
contract = When [Case (Deposit "alice" "alice" ada price) inner]
                1700000000
                Close

inner :: Contract
inner =
  When [ Case aliceChoice
              (When [ Case bobChoice
                          (If (aliceChosen `ValueEQ` bobChosen)
                              agreement
                              arbitrate) ]
                    1700007200
                    arbitrate)
       ]
       1700003600
       Close
```

Nhiều thuật ngữ ở đây được định nghĩa trong Haskell. Chủ yếu, chúng ta có hai hợp đồng liên quan đến những gì xảy ra khi có sự đồng ý `agreement` giữa Alice và Bob, và nếu không, Carol ở giữa sẽ phân xử `arbitrate` như thế nào:

```rust
agreement :: Contract
agreement =
  If (aliceChosen `ValueEQ` (Constant 0))
     (Pay "alice" (Party "bob") ada price Close)
     Close

arbitrate :: Contract
arbitrate =
  When [ Case carolClose Close,
         Case carolPay (Pay "alice" (Party "bob") ada price Close) ]
       1700010800
       Close
```

Trong các hợp đồng này, chúng tôi cũng sử dụng các cách viết tắt đơn giản như:

```rust
price :: Value
price = Constant 450
```

để chỉ định giá của con mèo, và do đó giá trị của số tiền đang bị giữ trong quỹ _ký quỹ_.

Chúng tôi cũng có thể mô tả các lựa chọn mà Alice và Bob đã đưa ra, lưu ý rằng chúng tôi cũng yêu cầu một giá trị mặc định `defValue` trong trường hợp các lựa chọn chưa được thực hiện.

```rust
aliceChosen, bobChosen :: Value

aliceChosen = ChoiceValue (ChoiceId choiceName "alice")
bobChosen   = ChoiceValue (ChoiceId choiceName "bob")

defValue = Constant 42

choiceName :: ChoiceName
choiceName = "choice"
```

Trong việc mô tả các lựa chọn, chúng ta có thể đặt tên hợp lý cho các giá trị số:

```rust
pay,refund,both :: [Bound]

pay    = [Bound 0 0]
refund = [Bound 1 1]
both   = [Bound 0 1]
```

và định nghĩa các hàm mới (hoặc "mẫu") cho chính mình. Trong trường hợp này, chúng ta định nghĩa:

```rust
choice :: Party -> [Bound] -> Action

choice party bounds =
  Choice (ChoiceId choiceName party) bounds
```

như một cách để làm cho việc diễn đạt các lựa chọn trở nên đơn giản và dễ đọc hơn:

```rust
alicePay, aliceRefund, aliceChoice :: Action
alicePay    = choice "alice" pay
aliceRefund = choice "alice" refund
aliceChoice = choice "alice" both
```

Với tất cả các định nghĩa này, chúng ta có thể viết lại hợp đồng ở đầu phần này theo cách làm rõ ý định của nó. Nếu viết bằng Marlowe "thuần túy" hoặc bằng cách mở rộng các định nghĩa này, chúng ta sẽ có hợp đồng này:

```rust
When [
  (Case
     (Deposit
        "alice" "alice" ada
        (Constant 450))
     (When [
           (Case
              (Choice
                 (ChoiceId "choice" "alice") [
                 (Bound 0 1)])
              (When [
                 (Case
                    (Choice
                       (ChoiceId "choice" "bob") [
                       (Bound 0 1)])
                    (If
                       (ValueEQ
                          (ChoiceValue
                             (ChoiceId "choice" "alice"))
                          (ChoiceValue
                             (ChoiceId "choice" "bob")))
                       (If
                          (ValueEQ
                             (ChoiceValue
                                (ChoiceId "choice" "alice"))
                             (Constant 0))
                          (Pay
                             "alice"
                             (Party "bob") ada
                             (Constant 450) Close) Close)
                       (When [
                             (Case
                                (Choice
                                   (ChoiceId "choice" "carol") [
                                   (Bound 1 1)]) Close)
                             ,
                             (Case
                                (Choice
                                   (ChoiceId "choice" "carol") [
                                   (Bound 0 0)])
                                (Pay
                                   "alice"
                                   (Party "bob") ada
                                   (Constant 450) Close))] 100 Close)))] 60
                 (When [
                       (Case
                          (Choice
                             (ChoiceId "choice" "carol") [
                             (Bound 1 1)]) Close)
                       ,
                       (Case
                          (Choice
                             (ChoiceId "choice" "carol") [
                             (Bound 0 0)])
                          (Pay
                             "alice"
                             (Party "bob") ada
                             (Constant 450) Close))] 100 Close)))
      ]
```

> **Bài tập**
>
> Bạn có thể thêm những viết tắt nào khác vào hợp đồng ở đầu trang?
>
> Bạn có thể tìm thấy bất kỳ hàm nào mà bạn có thể định nghĩa để làm cho hợp đồng ngắn gọn hơn hoặc có cấu trúc hơn không?

Ví dụ này đã cho thấy cách nhúng trong Haskell mang lại cho chúng ta một ngôn ngữ diễn đạt hơn, chỉ đơn giản bằng cách tái sử dụng một số tính năng cơ bản của Haskell, cụ thể là định nghĩa các hằng số và hàm. Trong bài hướng dẫn tiếp theo, bạn sẽ tìm hiểu về cách định nghĩa hợp đồng bằng cách nhúng JavaScript.

**Ghi chú**\
Nhiều ví dụ khác về việc sử dụng Haskell để xây dựng các hợp đồng Marlowe có thể được tìm thấy trong Marlowe Playground, nơi cũng có thể xây dựng các hợp đồng Marlowe được nhúng trong Haskell.
