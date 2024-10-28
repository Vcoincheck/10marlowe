# Ngữ nghĩa của Marlowe 2.0

Hướng dẫn này cung cấp ngữ nghĩa chính thức cấp cao của Marlowe 2.0 bằng cách mô tả các loại đầu vào được hỗ trợ và các hàm ngữ nghĩa chính.

Nói chung, các bên tham gia trong một hợp đồng Marlowe giao tiếp với hợp đồng bằng cách phát hành các giao dịch. Tại mỗi khối, một chuỗi giao dịch có thể được xử lý — theo thứ tự — bởi một hợp đồng.

Từ góc độ ngữ nghĩa, một giao dịch bao gồm một danh sách các đầu vào, và toàn bộ giao dịch có thể được ký bởi một hoặc nhiều bên tham gia. Cụ thể, chúng tôi đại diện cho một giao dịch dưới dạng một danh sách `AnyInputs` cùng với một tập hợp các số nguyên đại diện cho tập hợp các người ký.

Như chúng tôi đã đề cập trong hướng dẫn trước, các đầu vào có thể có một trong bốn loại: `Commit`, `Pay`, `Choice` và `Oracle`. Trong số này, `Commit` và `Pay` được coi là các hành động và thường liên quan đến việc chuyển tiền giữa một bên tham gia và hợp đồng. `Choice` và `Oracle` được sử dụng để cung cấp thông tin từ thế giới bên ngoài vào hợp đồng.

### Hàm `applyTransaction` <a href="#the-applytransaction-function" id="the-applytransaction-function"></a>

Việc thực hiện một giao dịch trong một hợp đồng Marlowe được mô hình hóa bởi hàm cấp cao `applyTransaction`. Hàm này nhận giao dịch cần xử lý và trạng thái hiện tại của hợp đồng (bao gồm trạng thái nội bộ, hợp đồng còn lại và số tiền còn lại trong hợp đồng), và trả về trạng thái mới của hợp đồng cùng với số tiền mà mỗi bên tham gia dự kiến sẽ chuyển đến hoặc từ hợp đồng như một phần của giao dịch.

Hàm `applyTransaction` có kiểu Haskell như sau:

```rust
applyTransaction :: [AnyInput] -> S.Set Person -> BlockNumber -> State -> Contract -> Integer
                    -> MApplicationResult (Integer, TransactionOutcomes, State, Contract)
```

Theo đó, hàm `applyTransaction` gọi ba hàm phụ chính: `reduce`, `fetchPrimitive` và `eval`.

* Hàm `reduce` (và hàm phụ của nó là `reduceRec`) được áp dụng trước và sau mỗi đầu vào
* `fetchPrimitive` chỉ được áp dụng cho các đầu vào là hành động (tức là: `Commits` và `Pays`)
* `eval` được áp dụng cho kết quả của `fetchPrimitive` bất cứ khi nào thích hợp.

Ngoài ba hàm này, còn có ba hàm bổ sung được áp dụng trong mỗi giao dịch. Trước khi xử lý các đầu vào, hàm `expireCommits` được áp dụng, và sau khi xử lý các đầu vào, các hàm `redeemMoney` và `simplify` được áp dụng.

### Xử lý hành động​ <a href="#action-processing" id="action-processing"></a>

Hãy bắt đầu bằng cách xem xét các phần của quá trình xử lý mà ngữ nghĩa thực hiện hoàn toàn cho các hành động (tức là, các đầu vào thuộc loại `Commit` và `Pay`).&#x20;

Một hành động được biểu diễn bởi `IdAction` của chính nó. Hàm `fetchPrimitive` sẽ duyệt qua các phần của hợp đồng hiện đang có hiệu lực (được kích hoạt) và tìm kiếm một phần nguyên thủy phù hợp với `IdAction` được cung cấp bởi đầu vào. `fetchPrimitive` chỉ thành công nếu có chính xác một phần nguyên thủy tương ứng với `IdAction` đã cho.

```rust
fetchPrimitive idAction blockNum state (Both leftContract rightContract) =
  case (go leftContract, go rightContract) of
     (Picked (result, cont), NoMatch) -> Picked (result, (Both cont rightContract))
     (NoMatch, Picked (result, cont)) -> Picked (result, (Both leftContract cont))
     (NoMatch, NoMatch) -> NoMatch
     _                  -> MultipleMatches
  where go = fetchPrimitive idAction blockNum state
```

Nếu nó thành công, `fetchPrimitive` sẽ trả về một `DetachedPrimitive`:

```rust
data DetachedPrimitive = DCommit IdCommit Person Integer Timeout
                       | DPay IdCommit Person Integer
```

`fetchPrimitive`cũng sẽ trả về hợp đồng sau khi loại bỏ `primitive` đã chọn; hợp đồng mới này sẽ được coi là hợp đồng còn lại nếu việc đánh giá thành công. `DetachedPrimitive` được truyền cho `eval`, cùng với `State` hiện tại của hợp đồng.

```rust
fetchPrimitive :: IdAction -> BlockNumber -> State -> Contract
               -> FetchResult (DetachedPrimitive, Contract)
```

#### `Commit`[​](broken-reference) <a href="#commit" id="commit"></a>

Một `Commit` cho phép một người tham gia chuyển một số tiền vào hợp đồng. Số tiền này sẽ được ghi lại trong hợp đồng dưới định danh `idCommit` và các khoản thanh toán trong tương lai có thể sử dụng định danh này như một nguồn tiền.&#x20;

Khi khối được chỉ định bởi timeout (`Timeout` thứ hai trong `Commit`) được đạt đến, bất kỳ số tiền nào từ Commit chưa được sử dụng (thông qua các khoản thanh toán) sẽ bị đóng băng, và người tham gia đã thực hiện cam kết sẽ có thể rút số tiền này bằng giao dịch tiếp theo mà họ ký.

```rust
eval (DCommit idCommit person value timeout) state =
  if (isCurrentCommit idCommit state) || (isExpiredCommit idCommit state)
  then InconsistentState
  else Result ( addOutcome person (- value) emptyOutcome
              , addCommit idCommit person value timeout state )
              NoProblem
```

Để một cam kết (`commitment`) có hiệu lực, không được phép có bất kỳ cam kết nào với định danh `idCommit` đã được phát hành trước đó (chỉ cho phép một cam kết cho mỗi định danh).

Nếu cam kết không hết hạn (`timeout`) và được phát hành đúng cách, hợp đồng còn lại, như được trả về bởi hàm `fetchPrimitive`, sẽ sử dụng phần tiếp theo đầu tiên (hợp đồng đầu tiên trong `Commit`).

```rust
fetchPrimitive idAction blockNum state (Commit idActionC idCommit person value _ timeout continuation _) =
  if (idAction == idActionC && notCurrentCommit && notExpiredCommit)
  then Picked ((DCommit idCommit person actualValue timeout),
               continuation)
  else NoMatch
  where notCurrentCommit = not (isCurrentCommit idCommit state)
        notExpiredCommit = not (isExpiredCommit idCommit state)
        actualValue = evalValue blockNum state value
```

Mặt khác, nếu bất kỳ thời gian hết hạn nào của cam kết (`Commit`) được đạt đến, hàm `reduceRec` (hàm phụ trợ của `reduce`) sẽ chỉ ra rằng cam kết sẽ được giảm xuống phần tiếp theo thứ hai của nó.

```rust
reduceRec blockNum state env c@(Commit _ _ _ _ timeout_start timeout_end _ continuation) =
  if (isExpired blockNum timeout_start) || (isExpired blockNum timeout_end)
  then reduceRec blockNum state env continuation
  else c
```

#### `Pay`​ <a href="#pay" id="pay"></a>

Một hành động `Pay` cho phép một người tham gia yêu cầu một khoản tiền từ hợp đồng. Khoản tiền này sẽ được trừ từ số tiền đã cam kết dưới định danh `idCommit`, và hợp đồng chỉ chi trả tối đa đến số tiền còn lại dưới `idCommit`, ngay cả khi còn tiền dư trong hợp đồng. Trong mọi trường hợp, `Pay` sẽ không chi trả bất kỳ khoản tiền nào nếu cam kết `idCommit` đã hết hạn.

```rust
eval (DPay idCommit person value) state =
  if (not $ isCurrentCommit idCommit state)
  then (if (not $ isExpiredCommit idCommit state)
        then Result (emptyOutcome, state) CommitNotMade
        else Result (emptyOutcome, state) CommitIsExpired)
  else (if value > maxValue
        then Result ( addOutcome person maxValue emptyOutcome
                    , fromJust $ discountAvailableMoneyFromCommit idCommit maxValue state )
                    NotEnoughMoneyLeftInCommit
        else Result ( addOutcome person value emptyOutcome
                    , fromJust $ discountAvailableMoneyFromCommit idCommit value state )
                    NoProblem)
  where maxValue = getAvailableAmountInCommit idCommit state
```

Nếu hành động `Pay` không hết hạn và được yêu cầu đúng cách, hợp đồng còn lại, như được trả về bởi hàm `fetchPrimitive`, sẽ sử dụng `continuation` đầu tiên (hợp đồng đầu tiên trong `Pay`).

```rust
fetchPrimitive idAction blockNum state (Pay idActionC idCommit person value _ continuation _) =
  if (idAction == idActionC)
  then Picked ((DPay idCommit person actualValue), continuation)
  else NoMatch
  where actualValue = evalValue blockNum state value
```

Mặt khác, nếu thời gian hết hạn của hành động `Pay` được đạt tới, hàm `reduceRec` (hàm phụ trợ của hàm `reduce`) quy định rằng hành động `Pay` sẽ được rút gọn xuống `continuation` thứ hai.

```rust
reduceRec blockNum state env c@(Pay _ _ _ _ timeout _ continuation) =
  if isExpired blockNum timeout
  then reduceRec blockNum state env continuation
  else c
```

### Xử lý đầu vào không hành động​ <a href="#non-action-input-processing" id="non-action-input-processing"></a>

Các đầu vào `Choice` và `Oracle` được xử lý rất khác so với các hành động. Chúng tương đối độc lập với trạng thái của hợp đồng và có thể được phát hành bất kỳ lúc nào, miễn là các giá trị được cung cấp có thể được hợp đồng sử dụng.&#x20;

Nói cách khác, trong mã hợp đồng phải có một phần kiểm tra giá trị `Choice` hoặc `Oracle` để người tham gia có thể cung cấp giá trị đó. Nếu không, hợp đồng không cần biết giá trị này, và việc cung cấp nó cũng chỉ làm thêm sự lộn xộn và tải cho hợp đồng và blockchain, điều này có thể dẫn đến các vấn đề như DoS. Vì những lý do này, ngữ nghĩa Marlowe 2.0 không cho phép cung cấp thông tin không cần thiết.

Ngoài điều đó ra, điều duy nhất mà Marlowe làm khi nhận được các giá trị `Choice` và `Oracle` là ghi chúng vào trạng thái để hàm reduce có thể truy cập chúng.

### Các tổ hợp và `Null`

Trong phần này, chúng tôi mô tả ngữ nghĩa của các hợp đồng Marlowe còn lại, có thể được sử dụng để kết hợp các hợp đồng khác nhau và để quyết định giữa chúng, tùy thuộc vào thông tin mà hợp đồng biết được tại bất kỳ thời điểm nào.&#x20;

Ngữ nghĩa của các tổ hợp này chủ yếu được định nghĩa bởi hàm `reduceRec` (hàm phụ trợ của `reduce`). Tuy nhiên, hành vi của chúng cũng ảnh hưởng đến các hàm khác, đặc biệt là `fetchPrimitive` và `simplify`.

Ví dụ, các quy tắc kích hoạt của mỗi cấu trúc được phản ánh trong `fetchPrimitive`, tức là, nếu cấu trúc ngay lập tức kích hoạt các hợp đồng con của nó, điều đó được chuyển thành `fetchPrimitive` có thể kiểm tra đệ quy một số hợp đồng con của nó.&#x20;

Đây là trường hợp của hợp đồng con thứ hai của `Let`:

```rust
fetchPrimitive idAction blockNum state (Let label boundContract subContract) =
  case fetchPrimitive idAction blockNum state subContract of
     Picked (result, cont) -> Picked (result, Let label boundContract cont)
     NoMatch -> NoMatch
     MultipleMatches -> MultipleMatches
```

Trong trường hợp của các cấu trúc khác như `When`, hàm `fetchPrimitive` sẽ để nguyên toàn bộ cấu trúc đó mà không thay đổi.

```rust
fetchPrimitive _ _ _ _ = NoMatch
```

#### `Null`​ <a href="#null" id="null"></a>

Hợp đồng `Null` không thực hiện bất kỳ hành động nào và sẽ luôn giữ nguyên trạng thái.

```rust
reduceRec _ _ _ Null = Null
```

Tuy nhiên, nó được sử dụng bởi hàm `simplify` và có thể được dùng để thay thế một hợp đồng bằng một hợp đồng nhỏ hơn nhưng tương đương. Ví dụ, `Both Null Null` có thể được giảm thành `Null`.

#### `Both`

Cấu trúc `Both` cho phép hai hợp đồng hoạt động đồng thời. Điều này giống như việc có hai hợp đồng riêng biệt được triển khai đồng thời, ngoại trừ việc khi sử dụng `Both`, chúng sẽ chia sẻ `State`, và do đó, các `Commit` trong một trong những hợp đồng có thể được sử dụng cho các `Pay` trong hợp đồng còn lại.&#x20;

Chúng tôi cũng đã rất cẩn thận để đảm bảo rằng `Both` là đối xứng, nghĩa là việc viết `Both A B` sẽ tương đương với việc viết `Both B A`, bất kể A và B là gì.

```rust
reduceRec blockNum state env (Both cont1 cont2) = Both (go cont1) (go cont2)
  where go = reduceRec blockNum state env
```

#### `Choice`​ <a href="#choice" id="choice"></a>

Cấu trúc `Choice` ngay lập tức chọn giữa hai hợp đồng dựa vào giá trị của một `Observation`. Vào thời điểm `Choice` được kích hoạt, giá trị của `obs` sẽ quyết định liệu `Choice` có được giảm thành `cont1` (nếu đúng) hay thành `cont2` (nếu sai).

```rust
reduceRec blockNum state env (Choice obs cont1 cont2) =
  reduceRec blockNum state env (if (evalObservation blockNum state obs)
                                then cont1
                                else cont2)
```

#### `When`​ <a href="#when" id="when"></a>

Cấu trúc `When` trì hoãn một hợp đồng theo thời gian cho đến khi một `Observation` trở thành đúng. `When` sẽ không kích hoạt bất kỳ hợp đồng con nào của nó cho đến khi `Observation` trở thành đúng hoặc cho đến khi đạt đến khối `timeout`. Nếu `obs` là đúng, thì `When` sẽ được giảm thành `cont1`; nếu timeout đã đạt, thì `When` sẽ được giảm thành `cont2`.

```rust
reduceRec blockNum state env c@(When obs timeout cont1 cont2) =
  if isExpired blockNum timeout
  then go cont2
  else if evalObservation blockNum state obs
       then go cont1
       else c
  where go = reduceRec blockNum state env
```

Đáng lưu ý rằng, vì Marlowe theo mô hình "**pull**", nên không chỉ đơn giản là `Observation` trở thành đúng thì When mới tiến triển; hợp đồng cần phải được kích hoạt trong khi `Observation` là đúng. Hợp đồng có thể được kích hoạt vào bất kỳ lúc nào bằng cách phát hành một giao dịch không cần có bất kỳ đầu vào nào và không cần phải được ký; thực tế, bất kỳ ai cũng có thể kích hoạt hợp đồng.

#### `While`​ <a href="#while" id="while"></a>

Cấu trúc `While` hoạt động theo cách ngược lại với `When`, có nghĩa là trong khi `When` được giảm xuống khi `Observation` trở thành đúng (true), `While` sẽ được giảm xuống khi `Observation` trở thành sai (false).

```rust
reduceRec blockNum state env (While obs timeout contractWhile contractAfter) =
  if isExpired timeout blockNum
  then go contractAfter
  else if evalObservation blockNum state obs
       then (While obs timeout (go contractWhile) contractAfter)
       else go contractAfter
  where go = reduceRec blockNum state env
```

Tuy nhiên, có một sự khác biệt cơ bản: `When` không kích hoạt hợp đồng con của nó cho đến khi nó được giảm, trong khi `While` kích hoạt hợp đồng con của nó ngay lập tức, tương tự như hành vi của `Both`. Và có một điều độc đáo về `While`: nó có thể xóa một hợp đồng đã được kích hoạt khi `Observation` trở thành đúng.&#x20;

Ví dụ, trong hợp đồng sau:

```rust
While
  (NotObs
     (ChoseSomething (1, 1))) 20
  (Commit 1 1 1
     (ValueFromChoice (1, 1)
        (Constant 20)) 10 20 Null Null) Null
```

Khi lựa chọn (1, 1) được thực hiện, sẽ không còn khả năng sử dụng `Commit` nữa.

#### `Scale`​ <a href="#scale" id="scale"></a>

Cấu trúc `Scale` điều chỉnh các khoản thanh toán bởi `Commit` và `Pay`. Nó nhận ba giá trị `(Values)`, giá trị đầu tiên là tử số, giá trị thứ hai là mẫu số, và giá trị thứ ba là giá trị mặc định.

Ngay khi cấu trúc `Scale` được kích hoạt, nó sẽ kích hoạt hợp đồng phụ và đánh giá tất cả ba giá trị và thay thế chúng bằng một `Constant` (để chúng không thay đổi nữa).

```rust
reduceRec blockNum state env (Scale divid divis def contract) =
  Scale (Constant vsDivid) (Constant vsDivis) (Constant vsDef) (go contract)
  where go = reduceRec blockNum state env
        vsDivid = evalValue blockNum state divid
        vsDivis = evalValue blockNum state divis
        vsDef = evalValue blockNum state def
```

Một khi được đánh giá, bất kỳ `Commit` hoặc `Pay` nào bên trong (trong hợp đồng) sẽ có số tiền của chúng được điều chỉnh như sau:

* Nếu mẫu số là `0`, thì số tiền sẽ được thay thế bằng giá trị mặc định.
* Nếu mẫu số không phải là `0`, thì số tiền sẽ được nhân với tử số và chia (sử dụng phép chia nguyên) cho mẫu số.

```rust
scaleValue :: Integer -> Integer -> Integer -> Integer -> Integer
scaleValue divid divis def val = if (divis == 0) then def else ((val * divid) `div` divis)
```

Quá trình điều chỉnh các `Commit` và `Pay` được thực hiện bởi hàm `fetchPrimitive`.

```rust
fetchPrimitive idAction blockNum state (Scale divid divis def subContract) =
  case fetchPrimitive idAction blockNum state subContract of
     Picked (result, cont) -> Picked (scaleResult sDivid sDivis sDef result,
                                      Scale divid divis def cont)
     NoMatch -> NoMatch
     MultipleMatches -> MultipleMatches
  where sDivid = evalValue blockNum state divid
        sDivis = evalValue blockNum state divis
        sDef = evalValue blockNum state def
```

Một khi không còn `Commit` hoặc `Pay` nào bên trong một Scale, nó sẽ bị loại bỏ bởi hàm `simplify`.

#### `Let` and `Use`​ <a href="#let-and-use" id="let-and-use"></a>

Cấu trúc `Let` gán hợp đồng phụ đầu tiên `boundContract` của nó cho một định danh `label`.

```rust
reduceRec blockNum state env (Let label boundContract contract) =
  case lookupEnvironment label env of
    Nothing -> let newEnv = addToEnvironment label checkedBoundContract env in
               Let label checkedBoundContract $ reduceRec blockNum state newEnv contract
    Just _ -> let freshLabel = getFreshLabel env contract in
              let newEnv = addToEnvironment freshLabel checkedBoundContract env in
              let fixedContract = relabel label freshLabel contract in
              Let freshLabel checkedBoundContract $ reduceRec blockNum state newEnv fixedContract
  where checkedBoundContract = nullifyInvalidUses env boundContract
```

Chúng tôi nghĩ rằng mỗi `Let` nên có một nhãn `(label)` định danh khác nhau, vì việc tái sử dụng chúng dẫn đến nhầm lẫn, nhầm lẫn dẫn đến lỗi, và lỗi dẫn đến bóng tối... hoặc đó là việc mất tiền tiềm năng.

Tuy nhiên, chúng tôi đã viết ngữ nghĩa Marlowe sao cho sự xuất hiện gần nhất của một `Let` (cái ở bên trong) có ưu tiên trong trường hợp xung đột (điều này thường được gọi là đè nén).&#x20;

Cách thực hiện điều này là bất cứ khi nào có sự xung đột của các định danh, định danh được định nghĩa mới sẽ được đổi tên thành một định danh mới mà chưa được sử dụng (trong cây hiện tại), đó là những gì hàm `relabel` thực hiện.

Bên trong sự tiếp diễn thứ hai (`contract`) của một `Let` có thể sử dụng cấu trúc `Use` để đại diện cho một bản sao của `boundContract` mà sẽ chỉ được mở rộng khi hợp đồng `Use` được kích hoạt.

```rust
reduceRec blockNum state env (Use label) =
  case lookupEnvironment label env of
    Just contract -> reduceRec blockNum state env contract
    Nothing -> Null
```

Lưu ý rằng nếu `Use` không được định nghĩa, nó sẽ mở rộng thành `Null`.&#x20;

Tương tự như `Scale`, cấu trúc `Let` chỉ được giảm khi không còn các cấu trúc `Use` tương ứng nào sử dụng nó; quy trình dọn dẹp này được thực hiện bởi hàm `simplify`.
