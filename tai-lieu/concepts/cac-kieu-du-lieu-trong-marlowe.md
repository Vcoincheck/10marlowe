# Các kiểu dữ liệu trong Marlowe

Hướng dẫn này giới thiệu chính thức Marlowe dưới dạng một kiểu dữ liệu Haskell, cũng như trình bày các kiểu dữ liệu khác nhau được sử dụng bởi mô hình và thảo luận về một số giả định về cơ sở hạ tầng mà các hợp đồng sẽ được chạy.&#x20;

Nó cũng giới thiệu _Marlowe Mở Rộng_, mà chúng tôi sử dụng để mô tả các mẫu hợp đồng trong Marlowe.&#x20;

Các mã mà chúng tôi mô tả ở đây đến từ các mô-đun Haskell [Semantics/Types.hs](https://github.com/input-output-hk/marlowe-cardano/blob/main/marlowe/src/Language/Marlowe/Core/V1/Semantics/Types.hs) , [Semantics.hs](https://github.com/input-output-hk/marlowe-cardano/blob/main/marlowe/src/Language/Marlowe/Core/V1/Semantics.hs) and [Util.hs](https://github.com/input-output-hk/marlowe-cardano/blob/main/marlowe/src/Language/Marlowe/Util.hs).

### Marlowe​ <a href="#marlowe" id="marlowe"></a>

Ngôn ngữ miền cụ thể Marlowe (DSL) được mô hình hóa như đã sử dụng ở trên, là một tập hợp các kiểu đại số trong Haskell, với các hợp đồng được cung cấp bởi kiểu `Contract`:

```rust
data Contract = Close
              | Pay Party Payee Token Value Contract
              | If Observation Contract Contract
              | When [Case] Timeout Contract
              | Let ValueId Value Contract
              | Assert Observation Contract
```

Chúng ta đã thấy trong hướng dẫn trước các hợp đồng này làm gì. Trong phần còn lại của hướng dẫn này, chúng ta sẽ đi sâu hơn vào các kiểu Haskell được sử dụng để đại diện cho các thành phần khác nhau của hợp đồng, bao gồm tài khoản, giá trị, quan sát và hành động.&#x20;

Chúng ta cũng sẽ xem xét các kiểu liên quan đến việc thực thi hợp đồng, bao gồm đầu vào, trạng thái và môi trường.

### Thành phần cơ bản​ <a href="#basic-components" id="basic-components"></a>

Trong việc mô hình hóa các phần cơ bản của Marlowe, chúng ta sử dụng sự kết hợp giữa các kiểu _dữ liệu (_`data`_)_ Haskell, định nghĩa các kiểu mới, và các từ đồng nghĩa kiểu (type synonyms) để đặt tên mới cho một _kiểu (_`type`_)_ đã tồn tại. Một Bên tham gia vào hợp đồng được đại diện như đã sử dụng ở trên, đó là một khóa công khai hoặc một tên vai trò.

```rust
data Party = PubKey PubKeyHash
           | Role   TokenName
```

Để thực thi một hợp đồng Marlowe, một bên phải cung cấp chứng cứ. Đối với bên tham gia là  `PubKey` đó sẽ là một chữ ký hợp lệ của một giao dịch được ký bằng khóa riêng của khóa công khai, tương tự như cơ chế `Pay to Public Key Hash` của Bitcoin.

```rust
newtype PubKeyHash = PubKeyHash { getPubKeyHash :: ByteString }
```

Đối với bên tham gia `Role`, chứng cứ là việc chi tiêu một token vai trò trong cùng một giao dịch, thường là cho cùng một chủ sở hữu.

```rust
newtype TokenName = TokenName { unTokenName :: ByteString }
```

&#x20;Các bên tham gia là `Role` sẽ có dạng `Role "alice"`, `Role "bob"` . Tuy nhiên, Haskell cho phép chúng ta trình bày và đọc các giá trị này một cách ngắn gọn hơn (bằng cách khai báo một phiên bản tùy chỉnh của `Show` và sử dụng _overloaded strings_) để chúng có thể được nhập và đọc như đã sử dụng ở trên, "alice", "bob", v.v.

Tài khoản Marlowe chứa các số tiền của nhiều loại tiền tệ và/hoặc token có thể thay thế và không thể thay thế. Một số tiền cụ thể được lập chỉ mục bởi một `Token`, là một cặp bao gồm một ký hiệu tiền tệ và một tên token, cả hai đều được cho bởi một `ByteString`.

```rust
newtype CurrencySymbol = CurrencySymbol { unCurrencySymbol :: ByteString }
data Token = Token CurrencySymbol TokenName
```

Cardano's Ada token là:

```rust
ada = Token adaSymbol adaToken
```

Nhưng bạn có thể tự do tạo ra các loại tiền tệ và token của riêng mình bằng cách sử dụng tiện ích token gốc của Cardano. Bạn có thể nghĩ về một Tài khoản (`Account`) như đã sử dụng ở trên, là một bản đồ từ Token sang Integer, và vì vậy tất cả các tài khoản trong một hợp đồng có thể được mô hình hóa như sau:

```rust
type AccountId = Party
type Accounts = Map (AccountId, Token) Integer
```

Các token của một loại tiền tệ có thể đại diện cho các vai trò trong một hợp đồng, ví dụ, "người mua-`buyer`" và "người bán-`seller`". Hãy nghĩ về một hợp đồng pháp lý theo nghĩa "sẽ được gọi là như đã sử dụng ở trên, người Thực hiện/Người cung cấp/Danh nhân/Tư vấn".&#x20;

Bằng cách này, chúng ta có thể tách rời khái niệm sở hữu vai trò của hợp đồng và làm cho nó có thể giao dịch được. Do đó, bạn có thể bán khoản vay của mình hoặc mua một phần vai trò trong một hợp đồng nào đó.

Thời gian hết hạn và số tiền cũng được xử lý theo cách tương tự; với cùng một phương pháp show/overload như đã sử dụng ở trên, chúng sẽ xuất hiện trong các hợp đồng dưới dạng số:

```rust
newtype POSIXTime = POSIXTime { getPOSIXTime :: Integer }
type Timeout = POSIXTime
```

Số này đại diện cho số giây sau nửa đêm ngày 1 tháng 1 năm 1970 (UTC).

Lưu ý rằng `"alice"` là chủ sở hữu ở đây theo nghĩa rằng cô ấy sẽ được hoàn lại bất kỳ khoản tiền nào trong tài khoản khi hợp đồng kết thúc.

Chúng ta có thể sử dụng chuỗi quá tải để cho phép chúng ta viết tắt tài khoản này bằng tên của chủ sở hữu: trong trường hợp này là `"alice"`.

Một khoản thanh toán có thể được thực hiện cho một trong các bên tham gia hợp đồng hoặc cho một trong các tài khoản của hợp đồng, và điều này được phản ánh trong định nghĩa này:

```rust
data Payee = Account AccountId
           | Party Party
```

Các lựa chọn — của số nguyên — được xác định bởi `ChoiceId`, kết hợp một tên cho lựa chọn với bên (Party) đã thực hiện lựa chọn đó:

```rust
type ChoiceName = ByteString
data ChoiceId   = ChoiceId ChoiceName Party
type ChosenNum  = Integer
```

Các giá trị được định nghĩa bằng cách sử dụng `Let` được xác định bằng các chuỗi (string) văn bản:

```rust
data ValueId    = ValueId ByteString
```

### Values, observations and actions​ <a href="#values-observations-and-actions" id="values-observations-and-actions"></a>

Xây dựng dựa trên các kiểu cơ bản, chúng ta có thể mô tả ba thành phần cấp cao hơn của hợp đồng: một loại của giá trị (_values_), trên đó là một loại của quan sát (_observations_), và cũng một loại của hành động (_actions)_, kích hoạt các trường hợp cụ thể. Đầu tiên, nhìn vào `Value` chúng ta có:

```rust
data Value = AvailableMoney Party Token
           | Constant Integer
           | NegValue Value
           | AddValue Value Value
           | SubValue Value Value
           | MulValue Value Value
           | DivValue Value Value
           | ChoiceValue ChoiceId
           | TimeIntervalStart
           | TimeIntervalEnd
           | UseValue ValueId
           | Cond Observation Value Value
```

Các loại giá trị khác nhau - tất cả đều là `Integer` - khá dễ hiểu, nhưng để đầy đủ, chúng ta có:

* `AvailableMoney`: Tra cứu giá trị trong một tài khoản.
* `ChoiceValue`: Được thực hiện trong một lựa chọn đã được xác định.
* `UseValue`: Một định danh đã được định nghĩa trước đó.
* Các hằng số và toán tử số học.
* Bắt đầu và kết thúc của khoảng thời gian hiện tại; xem phần dưới đây để thảo luận thêm về điều này.
* `Cond` đại diện cho các biểu thức if, tức là - tham số đầu tiên của `Cond` là một điều kiện (`Observation`) để kiểm tra, tham số thứ hai là một `Value` để lấy khi điều kiện được thỏa mãn và tham số cuối cùng là một `Value` cho điều kiện không được thỏa mãn; ví dụ: `(Cond FalseObs (Constant 1) (Constant 2))` tương đương với `(Constant 2)`.

Tiếp theo, chúng ta có các quan sát:

```rust
data Observation = AndObs Observation Observation
                 | OrObs Observation Observation
                 | NotObs Observation
                 | ChoseSomething ChoiceId
                 | ValueGE Value Value
                 | ValueGT Value Value
                 | ValueLT Value Value
                 | ValueLE Value Value
                 | ValueEQ Value Value
                 | TrueObs
                 | FalseObs
```

Các quan sát này thực sự dễ hiểu: chúng ta có thể so sánh các giá trị để kiểm tra (không) bình đẳng và thứ tự, và kết hợp các quan sát bằng cách sử dụng các phép toán Boolean. Cấu trúc duy nhất khác là `ChoseSomething`, cho biết liệu có bất kỳ lựa chọn nào đã được thực hiện cho một `ChoiceId` nhất định.

Các trường hợp và hành động được biểu diễn bằng các loại sau:

```rust
data Case = Case Action Contract

data Action = Deposit AccountId Party Token Value
            | Choice ChoiceId [Bound]
            | Notify Observation

data Bound = Bound Integer Integer
```

Ba loại hành động có thể xảy ra:

* `Deposit`` ``n p t v`: thực hiện một khoản tiền gửi giá trị v của token t từ bên p vào tài khoản n.
* Một lựa chọn được thực hiện cho một id cụ thể với danh sách các giới hạn trên các giá trị chấp nhận được. Ví dụ, `[Bound 0 0, Bound 3 5]` cung cấp lựa chọn một trong các giá trị 0, 3, 4 và 5.
* Hợp đồng được thông báo rằng một quan sát cụ thể sẽ được thực hiện. Thông thường, điều này sẽ được thực hiện bởi một trong các bên, hoặc một trong những ví của họ hoạt động tự động.

Điều này hoàn tất phần thảo luận của chúng ta về các loại hợp thành các hợp đồng Marlowe.

### Marlowe Mở Rộng

Marlowe Mở Rộng thêm chức năng tạo mẫu cho ngôn ngữ Marlowe, để các hằng số không cần phải "cố định" trong các hợp đồng Marlowe, mà có thể được thay thế bằng các tham số. Các đối tượng trong Marlowe Mở Rộng được gọi là mẫu hoặc mẫu hợp đồng.

Cụ thể, Marlowe Mở Rộng mở rộng loại `Value` với các giá trị tham số này:

* Những giá trị này có thể được sử dụng để hình thành các giá trị phức tạp hơn giống như các hằng số. Tương tự, loại `Timeout` cũng được mở rộng với các giá trị này:

Marlowe Mở Rộng không thể thực thi trực tiếp; nó phải được dịch sang Marlowe cơ bản trước khi thực hiện, triển khai hoặc phân tích, thông qua quá trình khởi tạo.&#x20;

Mục đích của Marlowe Mở Rộng là cho phép các hợp đồng Marlowe có thể tái sử dụng trong các tình huống khác nhau mà không làm rối mã sẽ được đưa lên chuỗi (Marlowe cơ bản). Trong Marlowe Playground, các mẫu cần phải được khởi tạo trước khi được mô phỏng.

### Giao Dịch

Như chúng tôi đã đề cập trước đó, ngữ nghĩa của Marlowe bao gồm việc xây dựng các giao dịch, như sau



<figure><img src="../../.gitbook/assets/Screenshot 2024-10-20 025726.jpg" alt=""><figcaption></figcaption></figure>

Một giao dịch được xây dựng từ một loạt các bước, trong đó một số bước tiêu thụ một giá trị đầu vào, và những bước khác tạo ra các hiệu ứng hoặc thanh toán.&#x20;

Trong phần mô tả này, chúng tôi đã giải thích rằng một giao dịch sửa đổi một hợp đồng (đến phần tiếp theo của nó) và trạng thái, nhưng chính xác hơn, chúng tôi có một hàm:

```rust
computeTransaction :: TransactionInput -> State -> Contract -> TransactionOutput
```

Các kiểu (_types_) loại được định nghĩa như sau:

```rust
data TOR = TOR { txOutWarnings :: [TransactionWarning]
               , txOutPayments :: [Payment]
               , txOutState    :: State
               , txOutContract :: Contract }
            deriving (Eq,Ord,Show,Read)

data TransactionOutput =
   TransactionOutput TOR
 | Error TransactionError
deriving (Eq,Ord,Show,Read)

data TransactionInput = TransactionInput
      { txInterval :: TimeInterval
      , txInputs   :: [Input] }
   deriving (Eq,Ord,Show,Read)
```

Cú pháp được sử dụng ở đây thêm tên trường vào các đối số của các nhà xây dựng, cung cấp các bộ chọn cho dữ liệu (như đã sử dụng ở trên), cũng như làm rõ mục đích của mỗi trường.

Loại `TransactionInput` có hai thành phần: khoảng thời gian (`TimeInterval`) trong đó nó có thể được thêm hợp lệ vào chuỗi khối, và một chuỗi tuần tự các giá trị đầu vào (`Input`) sẽ được xử lý trong giao dịch đó.

Một giá trị `TransactionOutput` có bốn thành phần: hai thành phần cuối cùng là trạng thái đã cập nhật (`State`) và hợp đồng (`Contract`), trong khi thành phần thứ hai cung cấp một chuỗi tuần tự các khoản thanh toán (`Payments`) do giao dịch tạo ra. Thành phần đầu tiên chứa một danh sách bất kỳ cảnh báo nào được tạo ra trong quá trình xử lý giao dịch.

#### Khoảng thời gian

Đây là một phần của kiến trúc của Cardano/Plutus, thừa nhận rằng không thể dự đoán chính xác vào khoảnh khắc nào một giao dịch cụ thể sẽ được xử lý. Do đó, các giao dịch được cấp một khoảng thời gian trong đó chúng được kỳ vọng sẽ được xử lý, và điều này được chuyển giao sang Marlowe: mỗi bước của một hợp đồng Marlowe được xử lý trong bối cảnh của một khoảng thời gian.

```rust
type TimeInterval = (POSIXTime, POSIXTime)
```

#### Những yếu tố này ảnh hưởng đến việc xử lý hợp đồng Marlowe thế nào?

Mỗi bước của hợp đồng được xử lý tương ứng với một khoảng thời gian, và giá trị thời gian hiện tại cần phải nằm trong khoảng đó.

Các điểm cuối của khoảng thời gian có thể truy cập thông qua các giá trị `TimeIntervalStart` và `TimeIntervalEnd` (như đã sử dụng ở trên), và chúng có thể được sử dụng trong các quan sát.&#x20;

Các giá trị thời gian giới hạn cần được xử lý một cách rõ ràng, vì vậy tất cả các giá trị trong khoảng thời gian phải vượt qua giới hạn thời gian đã đặt ra để có hiệu lực, hoặc rơi vào trước giới hạn thời gian để thực hiện theo cách bình thường.&#x20;

Nói cách khác, giá trị thời gian giới hạn cần phải nhỏ hơn hoặc bằng `TimeIntervalStart` (để giới hạn thời gian có hiệu lực) hoặc lớn hơn `TimeIntervalEnd` (để thực hiện bình thường xảy ra).

#### Ghi chú

* Mô hình đưa ra một số giả định về cơ sở hạ tầng chuỗi khối mà nó chạy.
* Nó được giả định rằng các hàm và hoạt động mật mã được cung cấp bởi một lớp bên ngoài Marlowe, vì vậy chúng không cần được mô hình hóa một cách rõ ràng.
* Việc thực hiện một khoản tiền gửi không phải là điều mà một hợp đồng có thể thực hiện; thay vào đó, nó có thể yêu cầu một khoản tiền gửi được thực hiện, nhưng sau đó phải được thiết lập từ bên ngoài: do đó, việc nhập (một tập hợp) các khoản tiền gửi cho mỗi giao dịch.
* Mô hình quản lý việc hoàn trả quỹ về cho chủ sở hữu của một tài khoản cụ thể khi một hợp đồng đạt đến điểm đóng (`Close`).
