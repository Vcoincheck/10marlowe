# Sử dụng Marlowe từ dòng lệnh ghci

Hướng dẫn này chỉ bạn cách sử dụng Marlowe từ bên trong Haskell, và đặc biệt cho thấy cách thực hiện một hợp đồng bằng cách sử dụng các ngữ nghĩa đã được trình bày trước đó.

### **Marlowe trong Haskell**

Hướng dẫn này hoạt động trên phiên bản Marlowe có trong nhánh chính của kho lưu trữ [marlowe-cardano](https://github.com/input-output-hk/marlowe-cardano). Chúng ta có thể chạy ghci bằng cách sử dụng _nix-shell_ có sẵn trong kho lưu trữ [marlowe-dependency-docs](https://github.com/input-output-hk/marlowe-dependency-docs):

```rust
git clone "https://github.com/input-output-hk/marlowe-dependency-docs.git"
cd marlowe-dependency-docs
nix-shell
ghci
```

Một phiên bản độc lập và cách chính thức hóa ngữ nghĩa cũng có thể được tìm thấy trong kho lưu trữ [Marlowe](https://github.com/input-output-hk/marlowe), nhưng một số chi tiết có thể khác biệt một chút, chẳng hạn như việc sử dụng slots thay vì thời gian POSIX.

### Các bước thực thi hợp đồng​ <a href="#stepping-through-contracts" id="stepping-through-contracts"></a>

Như chúng ta đã thấy trước đó, ngữ nghĩa của một giao dịch đơn lẻ được xác định bởi hàm

```rust
computeTransaction :: TransactionInput -> State -> Contract -> TransactionOutput
```

Các kiểu loại dữ liệu được định nghĩa như sau:

```rust
data TransactionInput = TransactionInput
    { txInterval :: TimeInterval
    , txInputs   :: [Input] }

data TransactionOutput =
   TransactionOutput
      { txOutWarnings :: [TransactionWarning]
      , txOutPayments :: [Payment]
      , txOutState    :: State
      , txOutContract :: Contract }
   | Error TransactionError
```

Các **trạng thái** (`States)` được định nghĩa như sau (với một hàm trợ giúp để định nghĩa một trạng thái ban đầu trống) :

```rust
data State = State { accounts    :: Accounts
                   , choices     :: Map ChoiceId ChosenNum
                   , boundValues :: Map ValueId Integer
                   , minTime     :: POSIXTime }

emptyState :: POSIXTime -> State
emptyState sn = State { accounts = Map.empty
                      , choices  = Map.empty
                      , boundValues = Map.empty
                      , minTime = sn }
```

Chúng ta có thể sử dụng các tính năng của ghci để thực hiện từng giao dịch trong một hợp đồng, và ở đây, chúng ta sẽ làm điều đó với hợp đồng ký quỹ được nhúng trong tệp [**Escrow.hs**](https://github.com/input-output-hk/marlowe-cardano/blob/main/marlowe-contracts/src/Marlowe/Contracts/Escrow.hs).

Để tải hợp đồng vào trình biên dịch, trước tiên chúng ta cần nhập một số thư viện và tạo các hàm tiện ích:

```rust
Prelude> :set -XOverloadedStrings
Prelude> import qualified Marlowe.Contracts.Escrow as Escrow
Prelude Escrow> import qualified Language.Marlowe.Extended.V1 as EM
Prelude Escrow EM> import Language.Marlowe as M
Prelude Escrow EM M> import qualified Data.Time.Clock.POSIX as P
Prelude Escrow EM M P> :set prompt "> "
> let toPOSIX = POSIXTime . floor . P.utcTimeToPOSIXSeconds . read
> let toEMPOSIX = EM.POSIXTime . floor . P.utcTimeToPOSIXSeconds . read
```

Ví dụ này được viết bằng Extended Marlowe, vì vậy chúng ta cần chuyển đổi nó sang core Marlowe trước khi thực thi:

```rust
> let Just contract = EM.toCore $ Escrow.escrow (EM.Constant 450) "bob" "alice" "carol" (toEMPOSIX "2023-02-01 00:00:00.000000 UTC") (toEMPOSIX "2023-03-01 00:00:00.000000 UTC") (toEMPOSIX "2023-04-01 00:00:00.000000 UTC") (toEMPOSIX "2023-05-01 00:00:00.000000 UTC") :: Maybe Contract
```

Bây giờ chúng ta có thể thực hiện từng bước một bằng cách sử dụng tính năng để tạo các biến gán cục bộ:

```rust
> let (TransactionOutput txWarn1 txPay1 state1 con1) = computeTransaction (TransactionInput (toPOSIX "2023-01-01 00:00:00.000000 UTC", toPOSIX "2023-01-31 23:59:59.000000 UTC") [NormalInput (IDeposit "bob" "alice" ada 450)]) (emptyState 0) contract
```

.Trong quá trình này, chúng ta đã sử dụng khớp mẫu đầu ra của một ứng dụng của hàm `computeTransaction`, hàm này nhận ba đầu vào: đầu vào thứ hai là một trạng thái ban đầu (tại slot số 0) và đầu vào thứ ba là hợp đồng ký quỹ ban đầu.&#x20;

Đầu vào đầu tiên là một `TransactionInput`, chứa một `TimeInterval` — ở đây là `((toPOSIX "2023-01-01 00:00:00.000000 UTC"), (toPOSIX "2023-01-31 23:59:59.000000 UTC"))` — và một khoản đặt cọc 450 Lovelace từ `"alice"` vào tài khoản của `"bob"`, cụ thể là `NormalInput (IDeposit "bob" "alice" ada 450).`

> **LƯU Ý**
>
> Nếu bạn muốn thử nghiệm điều này trong ghci, bạn có thể sao chép và dán từ các ví dụ mã: chúng nằm trong các cửa sổ cuộn ngang.

Đầu ra được khớp với `TransactionOutput txWarn1 txPay1 state1 con1` để chúng ta có thể xem xét các thành phần khác nhau một cách riêng biệt:

```rust
> txWarn1
[]
>  txPay1
[]
> state1
State {accounts = Map {unMap = [(("bob",Token "" ""),450)]}, choices = Map {unMap = []}, boundValues = Map {unMap = []}, minTime = POSIXTime {getPOSIXTime = 1672531200}}
> con1
When [Case (Choice (ChoiceId "Everything is alright" "alice") [Bound 0 0]) Close,Case (Choice (ChoiceId "Report problem" "alice") [Bound 1 1]) (Pay "bob" (Account "alice") (Token "" "") (Constant 450) (When [Case (Choice (ChoiceId "Confirm problem" "bob") [Bound 1 1]) Close,Case (Choice (ChoiceId "Dispute problem" "bob") [Bound 0 0]) (When [Case (Choice (ChoiceId "Dismiss claim" "carol") [Bound 0 0]) (Pay "alice" (Account "bob") (Token "" "") (Constant 450) Close),Case (Choice (ChoiceId "Confirm claim" "carol") [Bound 1 1]) Close] (POSIXTime {getPOSIXTime = 1682899200}) Close)] (POSIXTime {getPOSIXTime = 1680307200}) Close))] (POSIXTime {getPOSIXTime = 1677628800}) Close
```

Điều này cho thấy giao dịch không tạo ra cảnh báo hay thanh toán nào, nhưng cập nhật trạng thái để hiển thị số dư trong tài khoản của `"bob"`, và cập nhật hợp đồng, sẵn sàng nhận một lựa chọn từ Alice.

&#x20;Trong trạng thái tiếp theo, hợp đồng đang chờ đầu vào, và nếu Alice đồng ý rằng "Mọi thứ đều ổn", thì một khoản thanh toán cho Bob sẽ được tạo ra. Điều này được xác minh thông qua sự tương tác này trong GHCI:

```rust
> let (TransactionOutput txWarn2 txPay2 state2 con2) = computeTransaction (TransactionInput (toPOSIX "2023-02-01 00:00:00.000000 UTC", toPOSIX "2023-02-28 23:59:59.000000 UTC") [NormalInput (IChoice (ChoiceId "Everything is alright" "alice") 0)]) state1 con1
> txPay2
[Payment "bob" (Party "bob") (Value (Map [(,Map [("",450)])]))]
> con2
Close
> state2
State {accounts = Map {unMap = []}, choices = Map {unMap = [(ChoiceId "Everything is alright" "alice",0)]}, boundValues = Map {unMap = []}, minTime = POSIXTime {getPOSIXTime = 1675209600}}
```

Một cách thay thế để thực hiện điều này là thêm các định nghĩa này vào một tệp làm việc, ví dụ, **Build.hs**, nơi những định nghĩa này sẽ được lưu giữ. Thực sự, sẽ rất hợp lý khi bao gồm một số định nghĩa được sử dụng ở trên trong một tệp như vậy.

### **Các lộ trình thay thế qua hợp đồng**

\
Ta có một cách thực thi hợp đồng thay thế được đưa ra như sau:

* **Bước đầu tiên**: Alice gửi tiền như trong ví dụ trước.
* **Bước thứ hai**: Alice báo cáo một vấn đề và Bob không đồng ý. Điều này có thể được thực hiện như sau:

```rust
> let (TransactionOutput txWarn2 txPay2 state2 con2) = computeTransaction (TransactionInput (toPOSIX "2023-02-01 00:00:00.000000 UTC", toPOSIX "2023-02-28 23:59:59.000000 UTC") [NormalInput (IChoice (ChoiceId "Report problem" "alice") 1), NormalInput (IChoice (ChoiceId "Dispute problem" "bob") 0)]) state1 con1
> con2
When [Case (Choice (ChoiceId "Dismiss claim" "carol") [Bound 0 0]) (Pay "alice" (Account "bob") (Token "" "") (Constant 450) Close),Case (Choice (ChoiceId "Confirm claim" "carol") [Bound 1 1]) Close] (POSIXTime {getPOSIXTime = 1682899200}) Close
> state2
State {accounts = Map {unMap = [(("alice",Token "" ""),450)]}, choices = Map {unMap = [(ChoiceId "Report problem" "alice",1),(ChoiceId "Dispute problem" "bob",0)]}, boundValues = Map {unMap = []}, minTime = POSIXTime {getPOSIXTime = 1675209600}}
```

Điều này cho thấy rằng chúng ta đang ở trong một hợp đồng mà sự lựa chọn thuộc về Carol, và vẫn còn 450 Lovelace trong tài khoản của `"alice"`.

Lưu ý rằng chúng ta có hai đầu vào trong cùng một giao dịch, Marlowe hỗ trợ điều này miễn là giao dịch được ký bởi tất cả các bên liên quan, và khoảng thời gian là trước thời gian hết hạn của **When** sớm nhất.

* **Bước thứ ba**: Carol đưa ra lựa chọn. Nếu cô ấy chọn "Bác bỏ yêu cầu", khoản thanh toán cho Bob sẽ được thực hiện. Nếu cô ấy chọn "Xác nhận yêu cầu", Alice sẽ được hoàn lại tiền. Hãy làm điều đó ngay bây giờ:

```rust
> let (TransactionOutput txWarn3 txPay3 state3 con3) = computeTransaction (TransactionInput (toPOSIX "2023-04-01 00:00:00.000000 UTC", toPOSIX "2023-04-30 23:59:59.000000 UTC") [NormalInput (IChoice (ChoiceId "Confirm claim" "carol") 1)]) state2 con2
> txPay3
[Payment "alice" (Party "alice") (Value (Map [(,Map [("",450)])]))]
> con3
Close
> state3
State {accounts = Map {unMap = []}, choices = Map {unMap = [(ChoiceId "Report problem" "alice",1),(ChoiceId "Dispute problem" "bob",0),(ChoiceId "Confirm claim" "carol",1)]}, boundValues = Map {unMap = []}, minTime = POSIXTime {getPOSIXTime = 1680307200}}
```

Vì vậy, bây giờ hợp đồng đã sẵn sàng để **Đóng** `(Close)`, và do đó để hoàn lại bất kỳ số tiền nào còn lại, nhưng rõ ràng từ `state3` rằng không có tài khoản nào chứa số dư khác không, và do đó hợp đồng bị chấm dứt.

**Tại sao việc thực hiện từng bước lại hữu ích?** Nó tương đương với việc gỡ lỗi, và chúng ta có thể thấy trạng thái nội bộ của hợp đồng ở mỗi giai đoạn, sự tiếp diễn của hợp đồng, tức là những gì còn lại để thực thi, và các hành động được thực hiện ở mỗi bước.

**Bài tập**\
Khám phá một số cách khác để tương tác với hợp đồng.

> * Điều gì xảy ra khi Bob xác nhận có một vấn đề?
> * Điều gì xảy ra khi Bob và Alice có "tranh chấp" nhưng Carol "đứng" về phía Bob?
