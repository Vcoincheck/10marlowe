# Marlowe Playground

This tutorial gives an overview of the [Marlowe Playground](https://play.marlowe.iohk.io/#/), an online tool that allows users to create, to analyse, to interact with and to simulate the operation of embedded Marlowe contracts.

**Marlowe Playground** là công cụ trực tuyến hỗ trợ người dùng tạo, phân tích, tương tác, và mô phỏng hợp đồng thông minh viết bằng Marlowe.&#x20;

Đây là công cụ quan trọng để giúp người dùng hiểu cách hợp đồng sẽ hoạt động sau khi triển khai lên blockchain mà không cần phải thực sự triển khai.&#x20;

Người dùng có thể mô phỏng hành vi của hợp đồng theo từng bước trong trình duyệt với Marlowe Playground.

**Cách sử dụng Marlowe Playground**:

1.  **Các cách viết hợp đồng**: Người dùng có thể viết hợp đồng theo bốn cách:

    * Trực tiếp bằng ngôn ngữ Marlowe.
    * Sử dụng biểu diễn qua giao diện kéo-thả **Blockly** của Marlowe.
    * Nhúng Marlowe vào **Haskell** hoặc **JavaScript**.

    Các hợp đồng viết bằng Blockly, Haskell, hoặc JavaScript có thể được "biên dịch" thành Marlowe trong Playground để xem trước và mô phỏng.
2. **Mô phỏng và phân tích hợp đồng**: Sau khi viết xong, bạn có thể chuyển sang trình mô phỏng (_**Simulator**_) để phân tích và mô phỏng hợp đồng.
3. **Lưu trữ và tải lại dự án**: Dự án có thể được lưu dưới dạng github gist hoặc lưu tạm thời trong trình duyệt giữa các phiên làm việc (tuy nhiên dữ liệu này sẽ mất khi bộ nhớ cache của trình duyệt được cập nhật).

Phần còn lại của phần này sẽ đi sâu vào chi tiết về cách thức hoạt động của Marlowe Playground. Trên trang chủ của Marlowe Playground, bạn sẽ thấy các tùy chọn và công cụ cho từng bước mô phỏng, chỉnh sửa, và xây dựng hợp đồng.

## Bước đầu​ <a href="#getting-started" id="getting-started"></a>

​ Trang chính của **Marlowe Playground** cung cấp một giao diện thân thiện và dễ sử dụng với các tùy chọn để bắt đầu tạo, tải hoặc khám phá các hợp đồng thông minh. Dưới đây là những gì bạn sẽ thấy:

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

The title bar has a link to this tutorial at the right-hand side, and the footer has some general links.

Trang cung cấp ba tùy chọn:

1. **Mở dự án hiện có**: Tùy chọn này sẽ mở một dự án đã được lưu trước đó. Xem phần "Lưu và Mở Dự Án" bên dưới để biết thêm chi tiết về cách thiết lập thao tác này.
2. **Mở ví dụ**: Tùy chọn này sẽ tải một ví dụ vào dự án hiện tại trong môi trường được người dùng chọn.
3. **Bắt đầu dự án mới**: Tại đây, bạn có thể chọn bắt đầu bằng Javascript, Haskell, Marlowe hoặc Blockly.

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

Mỗi lựa chọn này sẽ được giải thích chi tiết ngay sau đây. Bất kể bạn bắt đầu từ đâu, bạn đều có cơ hội mô phỏng các hợp đồng mà mình phát triển.

**Trình soạn thảo chương trình** trong Playground là Monaco editor — [Monaco Editor](https://microsoft.github.io/monaco-editor/) — và có sẵn nhiều tính năng của nó, bao gồm cả menu xuất hiện khi nhấp chuột phải.

### **Trình soạn thảo JavaScript: phát triển hợp đồng nhúng**

\
Để biết chi tiết về cách định nghĩa Marlowe nhúng trong JavaScript, bạn có thể tham khảo phần Marlowe nhúng trong JavaScript.&#x20;

Chúng ta có thể sử dụng JavaScript để làm cho các định nghĩa hợp đồng dễ đọc hơn bằng cách sử dụng các định nghĩa JS cho các thành phần phụ, viết tắt, và các hàm mẫu đơn giản. Hình dưới đây minh họa trình soạn thảo JS.

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

**Trình soạn thảo JS** ở ví dụ dưới là **Escrow with Collateral**. Để mô tả một hợp đồng Marlowe trong trình soạn thảo này, một giá trị thuộc kiểu **Contract** phải được trả về như kết quả của hàm đã cung cấp bằng cách sử dụng lệnh `return`.

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

Trình soạn thảo hỗ trợ tính năng tự động hoàn thành, kiểm tra lỗi trong khi chỉnh sửa và cung cấp thông tin về các liên kết khi di chuột. Đặc biệt, khi di chuột qua bất kỳ liên kết nào đã nhập sẽ hiển thị kiểu của nó (trong TypeScript). Các nút trong thanh tiêu đề cung cấp các chức năng tiêu chuẩn:

* **Tạo dự án mới**
* **Mở một dự án đã có**
* **Mở một trong các ví dụ tích hợp**
* **Đổi tên dự án hiện tại**
* **Lưu dự án hiện tại với tên hiện có (nếu có)**
* **Lưu dự án hiện tại dưới một tên mới**

Khi bạn nhấn nút **Compile** (ở góc trên bên phải), mã trong trình soạn thảo sẽ được thực thi, và đối tượng JSON được trả về từ hàm sẽ được phân tích thành một hợp đồng Marlowe thực tế; sau đó, bạn có thể nhấn **Send to simulator** để bắt đầu mô phỏng hợp đồng.

Nếu biên dịch thành công, mã đã biên dịch sẽ được hiển thị bằng cách chọn **Generated code** ở chân trang; mã này cũng có thể được thu nhỏ lại sau đó.

<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

Nếu hợp đồng không thể chuyển đổi thành công sang Marlowe, các lỗi cũng sẽ được hiển thị trong một lớp phủ, có thể truy cập bằng cách chọn **Errors** ở chân trang.

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

Nhìn lại chân trang, bạn có thể thấy rằng có thể truy cập vào kết quả của **Static analysis**, như được mô tả dưới đây, cũng như kiểm tra và chỉnh sửa **Metadata** của hợp đồng.

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

Trình chỉnh sửa metadata chứa các trường để mô tả chung về mỗi hợp đồng, nhưng cũng hỗ trợ việc nhập thông tin mô tả cho các yếu tố như:

* **Vai trò (roles)**
* **Tham số (parameters)**
* **Lựa chọn (choices)**

Khi một hợp đồng được biên dịch thành công, trình chỉnh sửa metadata sẽ nhắc bạn thêm metadata cho các trường tương ứng với các phần tử phù hợp của hợp đồng và xóa các trường không tương ứng với bất kỳ điều gì trong hợp đồng.

### Trình chỉnh sửa Haskell: phát triển hợp đồng nhúng

Trình chỉnh sửa hỗ trợ việc phát triển các hợp đồng Marlowe được mô tả bằng Haskell. Chúng ta có thể sử dụng Haskell để làm cho các định nghĩa hợp đồng trở nên rõ ràng hơn bằng cách sử dụng các định nghĩa Haskell cho các thành phần phụ, viết tắt, và các hàm mẫu đơn giản.&#x20;

Trình chỉnh sửa Haskell được hiển thị trong hình ảnh dưới đây.

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

Trình chỉnh sửa hỗ trợ tính năng tự động hoàn thành, kiểm tra lỗi trong quá trình chỉnh sửa và thông tin về các biến khi di chuột qua. Các nút trên thanh tiêu đề cung cấp các chức năng tiêu chuẩn như sau:

* **Tạo một dự án mới**
* **Mở một dự án đã tồn tại**
* **Mở một trong những ví dụ có sẵn**
* **Đổi tên dự án hiện tại**
* **Lưu dự án hiện tại với tên hiện tại (nếu có)**
* **Lưu dự án hiện tại với tên mới**

Trình chỉnh sửa Haskell đang mở trên ví dụ về hợp đồng Escrow có sẵn trong các ví dụ. Để mô tả một hợp đồng Marlowe trong trình chỉnh sửa, chúng ta phải định nghĩa một giá trị cấp cao nhất có tên là `contract` thuộc loại `Contract`; chính giá trị này sẽ được chuyển đổi thành Marlowe thuần khi nhấn nút **Compile** (ở góc trên bên phải).&#x20;

Nếu biên dịch thành công, mã đã biên dịch sẽ được hiển thị bằng cách chọn **Generated code** ở chân trang.

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

Khi biên dịch thành công, kết quả có thể được gửi đến trình mô phỏng hoặc đến Blockly: các tùy chọn này được cung cấp bởi các nút **Send to Simulator** và **Send to Blockly** ở góc trên bên phải của trang.

Nếu hợp đồng không thể được chuyển đổi thành công sang Marlowe, các lỗi cũng sẽ được hiển thị bằng cách chọn **Errors** ở chân trang.

<figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

Khi xem lại phần chân trang, bạn có thể thấy rằng bạn có thể truy cập kết quả của phân tích tĩnh, như được mô tả bên dưới, cũng như xem và chỉnh sửa **metadata** của hợp đồng.

<figure><img src="../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

Trình chỉnh sửa metadata chứa các trường mô tả tổng quát cho từng hợp đồng, nhưng cũng hỗ trợ việc nhập thông tin mô tả về các yếu tố sau:

* **Vai trò (roles)**
* **Tham số (parameters)**
* **Lựa chọn (choices)**

Khi một hợp đồng được biên dịch thành công, trình chỉnh sửa siêu dữ liệu sẽ nhắc bạn thêm siêu dữ liệu cho các trường tương ứng với các phần tử thích hợp của hợp đồng, và xóa các trường không tương ứng với bất kỳ phần nào trong hợp đồng.

Metadata của hợp đồng không chỉ cung cấp tài liệu cho các hợp đồng Marlowe, mà còn được sử dụng trong ứng dụng web front-end, là ứng dụng khách cuối sẽ được sử dụng để chạy các hợp đồng Marlowe trên blockchain Cardano cùng với Marlowe Runtime.

### Trình chỉnh sửa Blockly

Trình chỉnh sửa Blockly cung cấp một cơ chế để tạo và xem các hợp đồng dưới dạng hình ảnh thay vì văn bản. Lưu ý rằng trình chỉnh sửa Blockly cũng cung cấp quyền truy cập vào trình chỉnh sửa siêu dữ liệu và phân tích tĩnh.

### Phát triển hợp đồng trong Marlowe

Cũng có thể tạo các hợp đồng trong Marlowe "thô". Marlowe được chỉnh sửa trong trình chỉnh sửa Marlowe, và điều này cung cấp định dạng tự động (khi nhấp chuột phải) và hỗ trợ cả các ô trống.

<figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

Các ô trống cho phép một chương trình được xây dựng theo cách từ trên xuống. Nhấp vào biểu tượng bóng đèn bên cạnh một ô trống sẽ hiển thị một menu hoàn thiện, trong mỗi trường hợp sẽ thay thế từng thành phần con bằng một ô trống mới.&#x20;

Ví dụ, chọn "`Pay`" để lấp đầy ô trống cấp cao sẽ dẫn đến kết quả như sau (sau khi định dạng bằng cách nhấp chuột phải):

<figure><img src="../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

Các ô trống có thể được kết hợp với việc chỉnh sửa văn bản thông thường, cho phép bạn sử dụng sự kết hợp của các cấu trúc từ dưới lên và từ trên xuống trong việc xây dựng các hợp đồng Marlowe.&#x20;

Hơn nữa, các hợp đồng có ô trống có thể được chuyển đổi giữa Marlowe và Blockly: các ô trống trong Marlowe trở thành các ô trống nguyên bản trong Blockly.&#x20;

Để chuyển sang Blockly, hãy sử dụng tùy chọn "Xem dưới dạng khối-View as blocks" ở góc trên bên phải của màn hình, và ngược lại.

#### Mô phỏng các hợp đồng và mẫu Marlowe

Dù hợp đồng được viết theo cách nào, khi nó được gửi đi để mô phỏng, đây là giao diện được nhìn thấy đầu tiên. Ở đây, chúng ta đang xem ví dụ về trái phiếu không lãi suất.

<figure><img src="../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

Trước khi bắt đầu mô phỏng, bạn cần cung cấp một số thông tin sau:

1. **Thời gian ban đầu(**initial time): Thời gian bắt đầu mà mô phỏng sẽ được thực hiện.
2. **Các tham số thời gian**(timeout parameters): Tại đây, bạn cung cấp thời gian mà người cho vay phải gửi số tiền, và thời gian mà người vay cần phải trả số tiền đó cùng với lãi suất.
3. **Các tham số giá trị**(value parameters): Trong trường hợp này, số tiền đã cho vay và số tiền lãi (thêm vào) cần phải trả.

Mã code được hiển thị ở đây trình bày hợp đồng hoàn chỉnh đang được mô phỏng. Khi mô phỏng bắt đầu, phần còn lại của hợp đồng cần được mô phỏng sẽ được đánh dấu. Phần chân trang cung cấp dữ liệu về quá trình mô phỏng.

#### Ví dụ

Đối với ví dụ của chúng ta, hãy điền các tham số như sau:

<figure><img src="../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

Mô phỏng được bắt đầu bằng cách nhấn nút **Start simulation**. Khi điều này được thực hiện, các hành động có sẵn để tiến triển hợp đồng sẽ được hiển thị. Lưu ý rằng toàn bộ hợp đồng sẽ được đánh dấu, cho thấy rằng chưa có phần nào của hợp đồng được thực thi.

<figure><img src="../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

Trong trường hợp này, có 4 hành động tiềm năng: Người cho vay (_Lender_) có thể thực hiện một khoản tiền gửi 10,000 Ada, hoặc thời gian có thể tiến đến phút tiếp theo, thời gian chờ (trong trường hợp này là thời hạn vay (_Loan_) mà chúng ta vừa thiết lập, tại đó việc chờ đợi khoản tiền gửi sẽ hết thời gian), hoặc trực tiếp đến thời điểm hết hạn của hợp đồng.&#x20;

Hai hành động tổng quát khác cũng có thể được thực hiện:

* **Hoàn tác** sẽ hoàn tác hành động cuối cùng được thực hiện trong mô phỏng. Điều này có nghĩa là chúng ta có thể khám phá một hợp đồng một cách tương tác, thực hiện một số bước, hoàn tác một số bước và sau đó tiến hành theo một hướng khác.
* **Đặt lại** sẽ đưa hợp đồng và trạng thái của nó trở về các giá trị ban đầu: hợp đồng đầy đủ và trạng thái trống. Nó cũng dừng mô phỏng.

Đối với ví dụ của chúng ta, hãy chọn để Người cho vay thực hiện khoản tiền gửi 10,000 Ada. Chúng ta có thể làm điều đó bằng cách nhấn nút **+** bên cạnh đầu vào này. Sau khi làm như vậy, chúng ta thấy:

<figure><img src="../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

Ở bên phải màn hình chúng ta thấy rằng khoản tiền gửi đã được thực hiện, tiếp theo là một khoản thanh toán tự động cho Người vay (_Borrower_). Chúng ta cũng có thể thấy rằng phần được làm nổi bật đã thay đổi để phản ánh rằng việc gửi tiền ban đầu và thanh toán đã được thực hiện.

Phần còn lại của hợp đồng là việc trả nợ: nếu chúng ta chọn hành động này bởi Người vay, chúng ta sẽ thấy rằng hợp đồng đã hoàn tất.

<figure><img src="../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

Nhật ký ở bên phải màn hình hiện cung cấp danh sách đầy đủ các hành động được thực hiện bởi các bên tham gia và bởi chính hợp đồng.&#x20;

Một ghi chú cuối cùng: chúng tôi đã chọn không tiến hành thời gian vào bất kỳ thời điểm nào; điều này nhất quán với thiết kế hợp đồng. Mặt khác, chúng tôi không thấy bất kỳ hành động timeout nào xảy ra. Tại sao bạn không thử điều này một mình?

### **Mô phỏng Oracle**

Như chúng tôi đã đề cập trước đó trong phần về Marlowe từng bước, Playground cung cấp các giá trị oracle cho các mô phỏng cho vai trò "kraken". Khi mô phỏng đến điểm simulating này:

<figure><img src="../../.gitbook/assets/image (29).png" alt="" width="375"><figcaption></figcaption></figure>

Giá trị được điền trước (_pre-filled_) như sau:

<figure><img src="../../.gitbook/assets/image (30).png" alt="" width="563"><figcaption></figcaption></figure>

## Lưu trữ và mở các hợp đồng​ <a href="#saving-and-opening-projects" id="saving-and-opening-projects"></a>

Các dự án có thể được lưu trên GitHub, và vì vậy, khi bạn lưu một dự án lần đầu tiên, bạn sẽ được nhắc như sau:

<figure><img src="../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

Nếu bạn chọn **Login**, bạn sẽ được "chuyển" tới màn hình đăng nhập Github:

<figure><img src="../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

Khi bạn chọn "Mở Dự Án", bạn sẽ được trình bày với một lựa chọn như thế này:

<figure><img src="../../.gitbook/assets/image (33).png" alt="" width="563"><figcaption></figcaption></figure>

Marlow Playground không cung cấp cơ chế để xóa các dự án, nhưng điều này có thể được thực hiện trực tiếp trên GitHub.

### **Phân tích hợp đồng**

Phân tích _tĩnh_ của một hợp đồng được thực hiện bằng cách chọn tab "Phân tích tĩnh-Static analysis" ở cuối trang.

<figure><img src="../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

Để phân tích một mẫu hợp đồng, cần cung cấp giá trị cho bất kỳ tham số nào của nó, như bạn có thể thấy trong ảnh chụp màn hình.

* Nút "**Phân tích cảnh báo-Analyse for warnings**" sẽ dẫn đến việc phân tích hợp đồng hiện tại trong trạng thái hiện tại. Kết quả có thể là hợp đồng đã vượt qua tất cả các kiểm tra, hoặc giải thích cách nó thất bại, kèm theo chuỗi giao dịch dẫn đến lỗi.&#x20;

_Ví dụ: hãy thử điều này với hợp đồng `Escrow`, thay đổi số tiền ký quỹ ban đầu từ Alice thành một giá trị nhỏ hơn 450 lovelace. Thông tin chi tiết hơn được cung cấp trong phần phân tích tĩnh._

* Nút "**Phân tích khả năng tiếp cận-Analyse reachability**" sẽ kiểm tra xem có bất kỳ phần nào của hợp đồng sẽ không bao giờ được thực hiện, bất kể cách các bên tham gia tương tác với hợp đồng.
* Nút "**Phân tích hoàn trả trên Close-Analyse for refunds on Close**" sẽ kiểm tra xem có thể xảy ra hoàn trả cho bất kỳ cấu trúc `Close` nào hay không, hoặc liệu trong mọi trường hợp `Close`, tất cả các quỹ trong hợp đồng đã được hoàn trả.

Hãy sử dụng Marlowe Playground để tương tác với các hợp đồng mẫu và đặc biệt thử nghiệm các hợp đồng với các giá trị tham số khác nhau, cũng như điều chỉnh chúng theo nhiều cách để xem các hợp đồng có thể thất bại như thế nào trong việc đáp ứng phân tích.
