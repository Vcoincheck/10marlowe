# Mô hình Marlowe

_**Lưu ý: Bản dịch có thể giữ nguyên gốc tiếng anh các thuật ngữ chuyên môn được thấy thường xuyên trong marlowe.**_

**Marlowe** được thiết kế để hỗ trợ việc thực thi các hợp đồng tài chính trên blockchain, và cụ thể là làm việc trên Cardano. Các hợp đồng được xây dựng bằng cách kết hợp một số lượng nhỏ các cấu trúc có thể được kết hợp để mô tả nhiều loại hợp đồng tài chính khác nhau.

Trước khi mô tả các cấu trúc đó, chúng ta cần xem xét cách tiếp cận tổng quát của chúng tôi trong việc mô hình hóa các hợp đồng trong Marlowe và bối cảnh mà các hợp đồng Marlowe được thực thi, đó là blockchain Cardano.&#x20;

### Các hợp đồng​ <a href="#contracts" id="contracts"></a>

Các hợp đồng trong Marlowe chạy trên một blockchain, nhưng cần tương tác với thế giới bên ngoài chuỗi (off-chain).&#x20;

Các bên tham gia hợp đồng, mà chúng tôi cũng gọi là các người tham gia, có thể thực hiện các hành động khác nhau: họ có thể được yêu cầu gửi tiền, hoặc lựa chọn giữa các lựa chọn khác nhau.&#x20;

Thông báo (_Notification_) là một hình thức đầu vào khác được sử dụng để cho hợp đồng biết rằng một điều kiện nhất định đã được đáp ứng; bất kỳ ai cũng có thể làm điều này, và điều này chỉ cần thiết bởi vì khi một hợp đồng trở nên ngừng hoạt động (quiescent), nó không thể "thức dậy" một mình; nó chỉ có thể phản hồi với các đầu vào.

Việc thực thi một hợp đồng cũng có thể tạo ra các hiệu ứng bên ngoài, bằng cách thực hiện các khoản thanh toán cho các bên trong hợp đồng.

#### Participants, roles, and public key​ <a href="#participants-roles-and-public-key" id="participants-roles-and-public-key"></a>

Chúng ta nên tách biệt khái niệm người tham gia (_participant)_, vai trò (_role) và_ khóa công khai (_public keys)_ trong một hợp đồng Marlowe. Một participant (or party) trong hợp đồng có thể được đại diện bởi một `role` hoặc một `public key` (các khóa công khai cuối cùng sẽ được thay thế [addresses](https://docs.cardano.org/learn/cardano-addresses)).

Vai trò _(Roles)_ được đại diện bởi các mã thông báo (_tokens_) và được phân phối cho các địa chỉ vào thời điểm hợp đồng được triển khai lên blockchain. Sau đó, bất kỳ ai có mã thông báo đại diện cho một vai trò có thể thực hiện các hành động được giao cho vai trò đó và nhận các khoản thanh toán được phát hành cho vai trò đó.

Điều này cho phép các vai trò trong hợp đồng được giao dịch giữa các người tham gia thông qua một cơ chế mã thông báo hóa (_tokenisation)_. Điều này sẽ có sẵn trong triển khai trên chuỗi của Marlowe nhưng mô phỏng trong Marlowe Playground chỉ trình bày các vai trò hợp đồng.&#x20;

Khóa công khai _(Public key)_ được đại diện bởi băm (hash) của một khóa công khai (hoặc cuối cùng là địa chỉ). Việc sử dụng các khóa công khai để đại diện cho các bên đơn giản hơn vì không yêu cầu xử lý mã thông báo, nhưng chúng không thể được giao dịch, vì một khi bạn biết khóa bí mật (_Private key_) cho một khóa công khai nhất định, bạn không thể chứng minh rằng bạn đã quên nó.

#### Accounts​ <a href="#accounts" id="accounts"></a>

Mô hình Marlowe cho phép một hợp đồng lưu trữ tài sản. Tất cả các bên tham gia trong hợp đồng ngầm sở hữu một tài khoản với tên của họ.&#x20;

Tất cả tài sản được lưu trữ trong hợp đồng phải thuộc về tài khoản của một trong các bên; theo cách này, khi hợp đồng được đóng, tất cả tài sản còn lại trong hợp đồng thuộc về một ai đó, và do đó có thể được hoàn trả cho các chủ sở hữu tương ứng.&#x20;

Các tài khoản này là cục bộ: chúng chỉ tồn tại trong suốt thời gian thực thi hợp đồng và trong thời gian đó, chúng chỉ có thể truy cập từ bên trong hợp đồng.

#### Steps and states​ <a href="#steps-and-states" id="steps-and-states"></a>

Các hợp đồng Marlowe mô tả một loạt các bước(_steps)_, thông thường bằng cách mô tả bước đầu tiên, cùng với một hợp đồng phụ khác mô tả những gì cần làm tiếp theo. Ví dụ, hợp đồng `Pay a p t v cont` nói rằng "thực hiện thanh toán giá trị `v` của mã thông báo `t` cho bên `p` từ tài khoản `a`, và sau đó tiếp tục theo hợp đồng `cont`". Chúng ta gọi `cont` là phần tiếp theo(_continuation)_ của hợp đồng.

Trong quá trình thực thi hợp đồng, chúng ta cần theo dõi hợp đồng hiện tại _current contract_ (tức là phần còn lại của hợp đồng): sau khi thực hiện một bước trong ví dụ trên, hợp đồng hiện tại là phần tiếp theo, `cont`. Chúng tôi cũng phải theo dõi một số thông tin khác, chẳng hạn như số tiền trong mỗi tài khoản: chúng tôi gọi thông tin này là trạng thái: nó có thể thay đổi tại mỗi bước. Một bước(_step_) cũng có thể chứng kiến một hành động diễn ra, chẳng hạn như tiền được gửi vào, hoặc một hiệu ứng được tạo ra, ví dụ: một khoản thanh toán. (): , cont. &#x20;

### Blockchain​ <a href="#blockchain" id="blockchain"></a>

Trong khi Marlowe được thiết kế để hoạt động với các blockchain nói chung, một số chi tiết về cách nó tương tác với blockchain là liên quan khi mô tả ngữ nghĩa và triển khai của Marlowe. Một blockchain dựa trên UTXO là một chuỗi các khối, mỗi khối chứa một tập hợp các giao dịch.&#x20;

Mỗi giao dịch có một tập hợp các đầu vào và đầu ra (input & output), và blockchain được xây dựng bằng cách liên kết các đầu ra giao dịch chưa tiêu (UTXO) với các đầu vào của một giao dịch mới. Tối đa chỉ có một khối có thể được tạo ra trong mỗi slot, mỗi slot dài 1 giây.&#x20;

Các cơ chế mà qua đó các khối này được tạo ra, và bởi ai, là không liên quan ở đây, nhưng các hợp đồng sẽ được diễn đạt theo thời gian POSIX.

#### UTXO, wallets and contracts​ <a href="#utxo-wallets-and-contracts" id="utxo-wallets-and-contracts"></a>

Giá trị trên blockchain nằm trong các UTXO, được bảo vệ bằng mật mã bởi một khóa riêng do chủ sở hữu nắm giữ. Những khóa này có thể được sử dụng để đổi lấy (_redeem)_ đầu ra và do đó sử dụng chúng làm đầu vào cho các giao dịch mới, có thể được xem như là việc tiêu dùng giá trị trong các đầu vào. Người dùng thường theo dõi các khóa riêng của họ và các giá trị liên kết với chúng trong một ví crypto an toàn về mặt mật mã.

Ngoài ra, các UTXO có thể được bảo vệ bằng một kịch bản (script), và đó chính là bản chất của một hợp đồng, một kịch bản bảo vệ một UTXO, và nó có thể tự lan tỏa qua một chuỗi các giao dịch.

Để tương tác với một hợp đồng đang chạy trên blockchain, người dùng sẽ cần sử dụng **Marlowe REST API** hoặc **Marlowe CLI**. Thông qua đó người dùng sẽ tương tác với ví của họ để xác thực các giao dịch chi tiêu tài sản, vì các khoản gửi được thực hiện từ ví của người dùng và các khoản thanh toán được nhận bởi họ.

Tuy nhiên, cần lưu ý rằng đây chắc chắn là những hành động ngoài chuỗi (off-chain) cần được khởi xướng bởi mã chạy ngoài chuỗi, thường sẽ thông qua **Marlowe REST API** hoặc **Marlowe CLI**: chúng không thể được thực hiện bởi hợp đồng đang chạy trên chuỗi (on-chain) tự nó.

#### Omniscient simulation​ <a href="#omniscient-simulation" id="omniscient-simulation"></a>

Marlowe Playground hỗ trợ mô phỏng hợp đồng. Đây là một mô phỏng toàn thức (_omniscient_ simulation), trong đó người dùng có khả năng thực hiện bất kỳ hành động nào cho bất kỳ vai trò nào, và do đó có thể quan sát việc thực thi từ quan điểm của tất cả người dùng cùng một lúc.&#x20;

Điều này tương phản với trải nghiệm chạy một hợp đồng bằng cách sử dụng một client, trong đó mỗi người tham gia thấy hợp đồng từ quan điểm của riêng họ. Đặc biệt, các người tham gia chỉ có thể tương tác với một hợp đồng đang chạy chờ đầu vào từ họ; nếu không phải trường hợp đó, thì họ sẽ thấy rằng việc thực thi hợp đồng đang chờ sự tham gia từ người khác.

#### Các giá trị và mã thông báo​ <a href="#values-and-tokens" id="values-and-tokens"></a>

Trong các ví dụ trước, bất cứ khi nào một `Value` được yêu cầu, chúng ta đều hoàn toàn sử dụng Ada. Điều này hoàn toàn hợp lý, vì Ada là đồng tiền cơ bản được hỗ trợ bởi Cardano.

Marlowe đưa ra một mô hình tổng thể hơn về _value_, điều đó có nghĩa là [native tokens](https://docs.cardano.org/native-tokens/learn) cũng thể sử dụng được, các loại mã thông báo(token) được sử dụng có thể là _fungible(FT)_, _non-fungible(NFT)_, hoặc _hỗn hợp (mixed)_. Vậy một `Value` trong Marlowe được thể hiện thế nào?

```haskell
newtype Value = Value
    {getValue :: Map CurrencySymbol (Map TokenName Integer)}
```

Các `CurrencySymbol` and `TokenName` đều là những bao bọc(_wrapped_) đơn giản quanh `ByteString`.

Khái niệm về giá trị(_values_) này bao gồm Ada, token có thể thay thế (nghĩa là tiền tệ), token không thể thay thế hoặc NFT (token tùy chỉnh không thể hoán đổi với các token khác), và nhiều trường hợp hỗn hợp kỳ lạ hơn:

* Ada có _empty `bytestring`_ là `CurrencySymbol` và `TokenName`.
* Một token có thể thay thế (_fungible_) được đại diện bởi một `CurrencySymbol` cho chính xác một `TokenName` có thể có một số lượng nguyên không âm tùy ý (trong đó Ada là một trường hợp đặc biệt).
* Một lớp token không thể thay thế(_non-fungible_) là một `CurrencySymbol` với nhiều `TokenName`, mỗi cái có số lượng bằng một. Mỗi cái tên trong số này tương ứng với một token không thể thay thế duy nhất.
* Các token hỗn hợp(mixed) là những token có nhiều `TokenName` và số lượng lớn hơn một.

Cardano cung cấp một cách đơn giản để giới thiệu một loại tiền tệ mới bằng cách đúc nó thông qua các kịch bản chính sách đúc. Điều này hiệu quả cho việc nhúng các tiêu chuẩn Ethereum ERC-20/ERC-721 như là các giá trị nguyên thủy trong Cardano.&#x20;

Trong Marlowe, chúng tôi sử dụng các token tùy chỉnh để đại diện cho các người tham gia trong mỗi hợp đồng thực thi trên chuỗi.

### Thực thi hợp đồng Marlowe <a href="#executing-a-marlowe-contract" id="executing-a-marlowe-contract"></a>

Việc thực thi một hợp đồng Marlowe trên blockchain Cardano có nghĩa là hạn chế các giao dịch do người dùng tạo ra theo logic của hợp đồng. Nếu, tại một thời điểm cụ thể trong quá trình thực thi, một hợp đồng mong đợi một khoản gửi 100 Ada từ Alice, thì chỉ giao dịch đó mới thành công; bất kỳ giao dịch nào khác sẽ bị từ chối.

Một giao dịch chứa một danh sách có thứ tự các đầu vào (_inputs)_ hoặc hành động (_actions)_. Trình thông dịch Marlowe được thực thi trong quá trình xác thực giao dịch. Đầu tiên, nó đánh giá hợp đồng từng bước một (_step by step_) cho đến khi không thể thay đổi thêm mà không cần xử lý bất kỳ đầu vào nào nữa, một điều kiện được gọi là _quiescent_. Ở giai đoạn này, chúng tôi tiến hành qua bất kỳ `When` nào với các thời hạn đã hết và tất cả các cấu trúc `If`, `Let`, `Pay`, and `Close` mà không chi tiêu bất kỳ đầu vào (_inputs_) nào.&#x20;

Đầu vào đầu tiên sau đó được xử lý, và hợp đồng lại được thực hiện từng bước cho đến khi đạt trạng thái quiescent, và quá trình này được lặp lại cho đến khi tất cả các đầu vào được xử lý. Tại mỗi bước, hợp đồng hiện tại và trạng thái sẽ thay đổi, một số đầu vào có thể được xử lý, và các khoản thanh toán được thực hiện.

Một giao dịch như được hiển thị trong sơ đồ dưới đây sẽ được thêm vào blockchain. Những gì chúng tôi làm tiếp theo là mô tả chi tiết về hình thức của các hợp đồng Marlowe và cách chúng được đánh giá từng bước một.

Chúng tôi đã chỉ ra rằng hành vi của một hợp đồng Marlowe là độc lập với cách thức các đầu vào được tập hợp thành các giao dịch, vì vậy khi chúng tôi mô phỏng hành động của một hợp đồng, chúng tôi không cần phải nhóm các đầu vào thành các giao dịch một cách rõ ràng.&#x20;

Để cụ thể hơn, chúng ta có thể nghĩ rằng mỗi giao dịch có tối đa một đầu vào. Trong khi ngữ nghĩa của một hợp đồng là độc lập với cách các đầu vào được nhóm thành các giao dịch, chi phí thực thi có thể thấp hơn nếu nhiều đầu vào có thể được nhóm lại thành một giao dịch duy nhất.

Trong mô phỏng toàn thức có sẵn trong Marlowe Playground, chúng tôi có thể an toàn trừu tượng hóa việc nhóm giao dịch, vì việc nhóm này không ảnh hưởng đến hành vi của hợp đồng.

**Xây dựng một giao dịch**

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>
