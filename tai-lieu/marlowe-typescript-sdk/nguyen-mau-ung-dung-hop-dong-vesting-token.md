# Nguyên mẫu ứng dụng "hợp đồng Vesting token"

### Tổng quan

**Token Plan Prototype** bao gồm các thành phần sau:

* Một DApp web được hỗ trợ bởi công nghệ hợp đồng thông minh Marlowe trên nền tảng Cardano.
* Một trường hợp sử dụng của hợp đồng vesting được phát triển và có sẵn trong Marlowe TS-SDK.
* Một minh chứng về cách xây dựng DApps được hỗ trợ bởi Marlowe với các công nghệ web phổ biến như TypeScript và React Framework.

#### Ví dụ đã triển khai

Chúng tôi mời bạn thử nghiệm một ví dụ đã được triển khai của [**Token Plan Prototype**](https://token-plans-preprod.prod.scdev.aws.iohkdev.io/).

#### Hợp đồng Vesting

Chúng tôi đã thiết kế hợp đồng vesting trong **Marlowe Playground** và sau đó tích hợp nó vào **Marlowe TS-SDK** để sử dụng trong trường hợp sử dụng **Token Plans**.

Hợp đồng vesting có sẵn để sử dụng trong gói npm `@marlowe.io/language-examples`.

```
import { Vesting } from "@marlowe.io/language-examples";
```

Mã nguồn mở có thể xem tại đây: [`\marlowe-ts-sdk/blob/main/packages/language/examples/src/vesting.ts`](https://github.com/input-output-hk/marlowe-ts-sdk/blob/main/packages/language/examples/src/vesting.ts)

Tài liệu về hợp đồng Vesting bằng TS-SDK [API Reference](https://input-output-hk.github.io/marlowe-ts-sdk/modules/\_marlowe\_io\_language\_examples.vesting.html).

#### Cấu hình

Để cấu hình ứng dụng, máy chủ HTTP của bạn cần phục vụ một tệp `config.json`. Để thực hiện điều này, hãy tạo tệp `public/config.json` với cấu trúc sau:

```
{ "marloweWebServerUrl": 'https://link-to-runner-instance', "develMode": false }
```

#### Nguyền mẫu (Prototype) <a href="#the-prototype" id="the-prototype"></a>

The prototype cho phép bạn tạo các Kế hoạch Token ₳ trên Cardano.

* Kế hoạch Token ₳ được tạo bởi một "Nhà Cung Cấp Token."&#x20;
* Nhà cung cấp sẽ gửi một số lượng ₳ nhất định với một kế hoạch theo thời gian xác định cách thức giải ngân ₳ này theo thời gian cho một "Người Nhận."

Lưu ý: Trong bối cảnh của nguyên mẫu này, chúng tôi đã kết hợp quan điểm của hai bên tham gia này để đơn giản hóa việc trình diễn trường hợp sử dụng.&#x20;

Mục đích ở đây không phải là cung cấp dịch vụ cho bạn trên Cardano, mà là để trình bày khả năng công nghệ Marlowe với một trường hợp sử dụng cụ thể và hoàn toàn mã nguồn mở. Chúng tôi rất khuyến khích bạn xem xét kỹ lưỡng cách mà nguyên mẫu này được triển khai.

**Lộ Trình**&#x20;

Phiên bản hiện tại là một ví dụ hợp đồng Marlowe hoàn chỉnh tích hợp trong một DApp Web. Đây là lần lặp đầu tiên và hiện tại bị giới hạn ở ba giai đoạn cho mỗi Hợp Đồng Vesting.

Lần lặp thứ hai sẽ cho phép bạn tạo một số lượng giai đoạn vô hạn. Tính năng Marlowe còn thiếu cần được cung cấp ở cấp độ DApp này được gọi là Hợp Đồng Chạy Dài Hay Merkle hóa Hợp Đồng. Các khả năng này đã có sẵn trong Runtime nhưng chưa có sẵn trong Marlowe TS-SDK.

#### **Cách Chạy tại môi trường cục bộ**&#x20;

Để khởi động DApp trên máy cục bộ của bạn, hãy làm theo các bước sau:

1.  **Nhân bản từ kho lưu trữ**

    ```
    git clone https://github.com/input-output-hk/marlowe-payouts
    cd marlowe-payouts
    ```
2. **Cài đặt các phần độc lập**
3.  **Cấu hình nguồn dẫn Marlowe trong `.env`**



    Để đảm bảo DApp giao tiếp chính xác với Marlowe Runtime và các instance quét, bạn cần thiết lập các URL thích hợp trong tệp `.env`.

    **Các bước thực hiện:**

    1. **Mở tệp .env**: Điều hướng đến thư mục gốc của dự án và mở tệp .env bằng trình soạn thảo văn bản mà bạn ưa thích.
    2. **Thiết lập URL Marlowe Runtime Web**: Tìm dòng `MARLOWE_RUNTIME_WEB_URL=<Your-Runtime-Instance>`. Thay thế `<Your-Runtime-Instance>` bằng URL thực tế của instance Marlowe Runtime của bạn. MARLOWE\_RUNTIME\_WEB\_URL=[https://example-runtime-url.com](https://example-runtime-url.com/)
    3. **Set the Marlowe Scan URL**: **Thiết lập URL Marlowe Scan**: Tìm dòng `MARLOWE_SCAN_URL=<Your-Scan-Instance>`. Thay thế `<Your-Scan-Instance>` bằng URL thực tế của instance quét Marlowe của bạn. MARLOWE\_SCAN\_URL=[https://example-scan-url.com](https://example-scan-url.com/)
4. **Run the DApp**

