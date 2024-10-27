# Marlowe nhúng trong JavaScript

Marlowe được viết dưới dạng kiểu dữ liệu Haskell, vì vậy việc mô tả các hợp đồng thông minh Marlowe bằng Haskell là rất đơn giản. Nhưng vì các hợp đồng Marlowe chỉ là một dạng dữ liệu, chúng ta cũng có thể đại diện chúng bằng các ngôn ngữ khác hỗ trợ việc tuần tự hóa thành JSON hoặc CBOR.

Ở đây, chúng tôi mô tả một thư viện viết bằng TypeScript có thể được sử dụng để tạo ra các hợp đồng thông minh Marlowe từ TypeScript hoặc JavaScript theo cách tương tự như khi sử dụng Haskell. Nếu bạn không quen thuộc với TypeScript, bạn cũng có thể sử dụng API như thể nó được viết bằng JavaScript vì TypeScript là một tập hợp con của JavaScript.

Bạn có thể thử nghiệm thư viện này trực tuyến trong Marlowe Playground bằng cách chọn "Start in JavaScript" trên trang chính, hoặc mở một trong các ví dụ JavaScript.

Chúng tôi bắt đầu phần này bằng cách giải thích về việc nhúng, sau đó giải thích một vài điểm cụ thể về việc nhúng trong JavaScript, và cuối cùng trình bày một ví dụ về một hợp đồng đầy đủ được mô tả bằng việc nhúng JS.

**Sử dụng Trình soạn thảo JS trong Marlowe Playground**\
Việc triển khai thư viện tự nó là khá đơn giản, và bạn có thể tìm thấy toàn bộ mã nguồn của nó tại đây:

* [https://github.com/input-output-hk/marlowe-playground/blob/main/marlowe-playground-client/src/Language/Javascript/MarloweJS.ts](https://github.com/input-output-hk/marlowe-playground/blob/main/marlowe-playground-client/src/Language/Javascript/MarloweJS.ts)

Nó dựa trên nguyên tắc rằng cho mỗi kiểu Haskell có một kiểu TypeScript tương ứng, và tương ứng với mỗi trình tạo có một định nghĩa hằng số.

```rust
import {
   Address, Role, Account, Party, ada, AvailableMoney, Constant, ConstantParam,
   NegValue, AddValue, SubValue, MulValue, DivValue, ChoiceValue, TimeIntervalStart,
   TimeIntervalEnd, UseValue, Cond, AndObs, OrObs, NotObs, ChoseSomething,
   ValueGE, ValueGT, ValueLT, ValueLE, ValueEQ, TrueObs, FalseObs, Deposit,
   Choice, Notify, Close, Pay, If, When, Let, Assert, SomeNumber, AccountId,
   ChoiceId, Token, ValueId, Value, EValue, Observation, Bound, Action, Payee,
   Case, Timeout, ETimeout, TimeParam, Contract
} from 'marlowe-js';
```

Thư viện JavaScript/TypeScript cung cấp các định nghĩa hằng cho các cấu trúc Marlowe không có tham số, như là trường hợp của `TimeIntervalStart`:

```rust
const TimeIntervalStart: Value
```

Hoặc hợp đồng `Close`:

Các cấu trúc có tham số được đại diện dưới dạng hàm, như trong trường hợp của `AvailableMoney`:

```rust
const AvailableMoney: (token: Token, accountId: Party) => Value
```

Bạn có thể thấy các khai báo kiểu cho từng cấu trúc và kiểu bằng cách di chuột qua định danh trong phần khai báo nhập ở đầu tệp xuất hiện trong trình soạn thảo của tab JS Editor.&#x20;

Phần khai báo nhập và phần khai báo hàm đều được làm mờ để báo hiệu rằng chúng không được phép sửa đổi; mã nguồn tạo hợp đồng phải được viết bên trong thân hàm được cung cấp, và hợp đồng kết quả phải được trả về như một kết quả của hàm (sử dụng lệnh return).

Về mặt nội bộ, các hàm và hằng số của thư viện JavaScript/TypeScript trả về một đại diện JSON của các cấu trúc Marlowe. Ví dụ, hàm AvailableMoney được định nghĩa như sau:

```rust
const AvailableMoney =
    function (token: Token, accountId: Party) => Value {
        return { "amount_of_token": token,
                 "in_account": accountId };
    };
```

Khi bạn nhấn nút **Compile** trong trình soạn thảo JS của Marlowe Playground, mã trong phần thân của tab sẽ được thực thi, và đối tượng JSON được trả về bởi hàm trong quá trình thực thi sẽ được phân tích thành một hợp đồng Marlowe thực tế. Khi điều đó thành công, bạn có thể chọn Send to Simulator; cách thức hoạt động của điều này sẽ được mô tả trong phần tiếp theo.

Về nguyên tắc, bạn có thể viết mã JavaScript tạo ra đại diện JSON của Marlowe trực tiếp, nhưng bạn không cần phải lo lắng về JSON chút nào khi sử dụng thư viện JS.

Khi bạn sử dụng thư viện Marlowe JS và việc sử dụng các hàm và hằng số của thư viện kiểm tra kiểu thành công, thì kết quả của mã của bạn sẽ tạo ra một đại diện JSON hợp lệ của một hợp đồng Marlowe, do đó chúng tôi đảm bảo an toàn cho việc tạo hợp đồng thông qua hệ thống kiểu của TypeScript.

Có một kiểu quan trọng mà không có trong định nghĩa Haskell của Marlowe, chúng tôi đã gọi kiểu đó là _SomeNumber_, và nó được định nghĩa như sau:

```rust
type SomeNumber = string | number | bigint
```

Lý do chúng tôi có kiểu này là vì kiểu gốc cho số trong JavaScript và TypeScript mất độ chính xác khi được sử dụng với các số nguyên lớn. Điều này là do việc triển khai của nó dựa trên các số dấu phẩy động.

Biểu thức sau đây là đúng trong JavaScript:

```rust
9007199254740992 == 9007199254740993
```

Điều này có thể gây ra vấn đề cho các hợp đồng tài chính, vì nó cuối cùng có thể dẫn đến mất tiền.

Do đó, chúng tôi khuyến nghị sử dụng kiểu `bigint`. Nhưng chúng tôi hỗ trợ ba cách biểu diễn số cho sự thuận tiện và tương thích ngược với các phiên bản cũ của JavaScript:

* **Số gốc**:
  * Chúng dễ sử dụng.
  * Cú pháp rất đơn giản và có thể sử dụng với các toán tử tiêu chuẩn, ví dụ: `32 + 57`.
  * Chúng mất độ chính xác cho các số lớn.
* **Biểu diễn chuỗi**:
  * Cú pháp chỉ cần thêm dấu nháy quanh các số.
  * Bạn không thể sử dụng các toán tử tiêu chuẩn trực tiếp, ví dụ: `"32" + "57" = "3257"`.
  * Chúng không mất độ chính xác.
* **Kiểu** `bigint`:
  * Chúng dễ sử dụng (chỉ cần thêm `n` sau các hằng số số).
  * Cú pháp rất đơn giản và có thể sử dụng với các toán tử tiêu chuẩn, ví dụ: `32n + 57n`.
  * Chúng không mất độ chính xác.

Tất cả các biểu diễn này đều được chuyển đổi thành `BigNumber` nội bộ, nhưng có thể xảy ra mất độ chính xác nếu sử dụng số gốc, vì `BigNumber` được xây dựng trước khi chuyển đổi xảy ra và API không thể làm gì về điều đó.

### Kiểu loại `EValue` và việc nạp chồng boolean

\
Trong Haskell, các quan sát boolean hằng số được đại diện bởi `TrueObs` và `FalseObs`, và các giá trị số nguyên hằng số được đại diện bởi `Constant` theo sau bởi một `Integer`.&#x20;

Trong JavaScript và TypeScript, bạn cũng có thể sử dụng các bộ xây dựng này, nhưng bạn không cần phải làm như vậy, vì loại `Observation` được nạp chồng để cũng chấp nhận các boolean gốc của JavaScript.

Các hàm mà trong Haskell nhận một `Value`, trong JavaScript lại nhận một `EValue` thay vào đó, và `EValue` được định nghĩa như sau:

```rust
type EValue = SomeNumber | Value
```

### Ví dụ: Viết một hợp đồng Hoán đổi bằng TypeScript

Cho dù chúng ta bắt đầu bằng cách chỉnh sửa một ví dụ hiện có hay tạo một hợp đồng JavaScript mới, chúng ta đều tự động được cung cấp danh sách import và khai báo hàm.&#x20;

Chúng ta có thể bắt đầu bằng cách xóa tất cả những gì không được làm mờ, và bắt đầu viết bên trong dấu ngoặc nhọn của định nghĩa hàm đã cung cấp.

Giả sử chúng ta muốn viết một hợp đồng để Alice có thể hoán đổi 1000 Ada với Bob để lấy 100 đô la. Đầu tiên, hãy tính toán các số lượng mà chúng ta muốn làm việc với mỗi đơn vị, chúng ta có thể định nghĩa một số hằng số số học bằng cách sử dụng `const`:

```rust
const lovelacePerAda : SomeNumber = 1000000n;
const amountOfAda : SomeNumber = 1000n;
const amountOfLovelace : SomeNumber = lovelacePerAda * amountOfAda;
const amountOfDollars : SomeNumber = 100n;
```

Số tiền trong hợp đồng phải được viết bằng Lovelace, đây là 0.000001 Ada. Vì vậy, chúng ta tính toán số lượng Lovelace bằng cách nhân 1,000 Ada với 1,000,000. Số lượng đô la là 100 trong ví dụ của chúng ta.

API đã cung cấp một bộ xây dựng cho đồng tiền ADA, và hiện tại không có ký hiệu tiền tệ nào trong Cardano cho đô la, nhưng hãy tưởng tượng rằng có, và hãy định nghĩa nó như sau:

```rust
const dollars : Token = Token("85bb65", "dollar")
```

Chuỗi "85bb65" thực sự sẽ tương ứng với ký hiệu tiền tệ, đó là một mã băm và phải được viết dưới dạng base16 (biểu diễn hệ thập lục của một chuỗi byte). Và chuỗi "dollar" sẽ tương ứng với tên token.

Bây giờ, hãy định nghĩa một kiểu đối tượng để giữ thông tin về các bên và những gì họ muốn trao đổi cho tiện lợi:

```rust
type SwapParty = {
 party: Party;
 currency: Token;
 amount: SomeNumber;
};
```

Chúng ta sẽ lưu trữ tên của bên tham gia trong trường `party`, tên của tiền tệ trong trường `currency`, và số lượng tiền tệ mà bên này muốn trao đổi trong trường `amount`. Dưới đây là cách cập nhật kiểu đối tượng để phù hợp với yêu cầu này:

```rust
const alice : SwapParty = {
   party: Role("alice"),
   currency: ada,
   amount: amountOfLovelace
}

const bob : SwapParty = {
   party: Role("bob"),
   currency: dollars,
   amount: amountOfDollars
}
```

Chúng ta có thể bắt đầu viết hợp đồng bằng cách định nghĩa các khoản tiền gửi. Đầu tiên, chúng ta cần thông tin về bên tham gia sẽ thực hiện khoản tiền gửi, thời gian chờ tối đa cho khoản tiền gửi đó, và hợp đồng tiếp theo sẽ được thực thi nếu khoản tiền gửi thành công. Dưới đây là cách triển khai điều này:

```rust
function makeDeposit(src: SwapParty, timeout: ETimeout,
                     timeoutContinuation: Contract, continuation: Contract): Contract {
    return When([Case(Deposit(src.party, src.party, src.currency, src.amount),
                      continuation)],
                timeout,
                timeoutContinuation);
}
```

Chúng ta chỉ cần một cấu trúc `When` với một trường hợp duy nhất đại diện cho một khoản tiền gửi của bên nguồn vào tài khoản của chính họ. Bằng cách này, nếu chúng ta hủy bỏ hợp đồng trước khi thực hiện giao dịch, mỗi bên sẽ lấy lại số tiền họ đã gửi.

Tiếp theo, chúng ta định nghĩa một trong hai khoản thanh toán của giao dịch. Chúng ta lấy bên nguồn và bên đích làm tham số, cũng như hợp đồng tiếp theo sẽ được thực thi sau khi thực hiện thanh toán.

```rust
const makePayment = function (src: SwapParty, dest: SwapParty,
                              continuation: Contract): Contract {
    return Pay(src.party, Party(dest.party), src.currency, src.amount,
               continuation);
}
```

Để thực hiện điều này, chúng ta chỉ cần sử dụng cấu trúc `Pay` để thanh toán từ tài khoản mà bên nguồn đã gửi tiền đến bên đích.

Cuối cùng, chúng ta có thể kết hợp tất cả các phần lại với nhau:

```rust
const contract: Contract = makeDeposit(alice, 1700000000n, Close,
                             makeDeposit(bob, 1700003600n, Close,
                                 makePayment(alice, bob,
                                     makePayment(bob, alice,
                                         Close))))

return contract;
```

Hợp đồng có bốn bước như sau:

1. Alice có thể gửi tiền cho đến thời gian POSIX 1700000000 (2023-11-14 22:13:20 GMT).
2. Bob có thể gửi tiền cho đến thời gian POSIX 1700003600 (2023-11-14 23:13:20 GMT), một giờ sau, nếu không Alice sẽ được hoàn tiền và hợp đồng sẽ bị hủy.
3. Sau đó, chúng ta sẽ thanh toán số tiền gửi của Alice cho Bob.
4. Cuối cùng, chúng ta sẽ thanh toán số tiền gửi của Bob cho Alice.

Và đó là tất cả. Bạn có thể tìm thấy mã nguồn đầy đủ cho một phiên bản hợp đồng thông minh hoán đổi đã được định dạng trong các ví dụ trong Marlowe Playground, mà chúng ta sẽ xem xét tiếp theo.
