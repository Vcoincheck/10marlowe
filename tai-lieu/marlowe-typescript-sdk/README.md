# Marlowe typescript SDK

### Giới thiệu TS-SDK  <a href="#ts-sdk-introduction" id="ts-sdk-introduction"></a>

_Lưu ý quan trọng: SDK hiện đang ở phiên bản beta. Những thay đổi không tương thích có thể xảy ra trong tương lai, vì vậy cần cẩn thận trước khi sử dụng trong môi trường thực thi._

Marlowe TypeScript SDK (TS-SDK) bao gồm các thư viện JavaScript và TypeScript, có sẵn dưới dạng gói npm.&#x20;

TS-SDK được thiết kế để hỗ trợ các nhà phát triển DApp với các công cụ cần thiết để xây dựng và tích hợp vào hệ sinh thái hợp đồng thông minh Marlowe trên blockchain Cardano.

### Tính năng chính

1. **Bộ công cụ hợp đồng thông minh**: Tạo và quản lý các hợp đồng thông minh Marlowe trên Cardano với các công cụ và thư viện trong TS-SDK:
   * [**@marlowe.io/language-core-v1**](https://input-output-hk.github.io/marlowe-ts-sdk/modules/\_marlowe\_io\_language\_core\_v1.html)
   * [**@marlowe.io/marlowe-object**](https://input-output-hk.github.io/marlowe-ts-sdk/modules/\_marlowe\_io\_marlowe\_object.html).
2. **Tích hợp với Marlowe Playground**: TS-SDK hoạt động tốt với [Marlowe Playground](https://playground.marlowe-lang.org/#/), một giao diện trực tuyến chuyên thiết kế, mô phỏng, và thử nghiệm hợp đồng Marlowe.
3. **Kết nối ví**: Với các module tích hợp sẵn (`CIP-30`, `Lucid adapters`), TS-SDK hỗ trợ tương tác mượt mà với các tiện ích mở rộng ví như [Lace](https://www.lace.io/), [Nami](https://namiwallet.io/) và [Eternl](https://eternl.io/app/mainnet/welcome). Điều này giúp truy cập dữ liệu ví dễ dàng và tích hợp hợp đồng Marlowe hiệu quả với nhiều giao diện ví khác nhau. (Xem [@marlowe.io/wallet](https://input-output-hk.github.io/marlowe-ts-sdk/modules/\_marlowe\_io\_wallet.html)).
4. **Tích hợp với Runtime**: TS-SDK hướng tới cung cấp Runtime Rest Client với đầy đủ các tính năng của `marlowe-runtime-web`, cho phép tận dụng tất cả tính năng Runtime trong môi trường JavaScript và TypeScript. (Xem [@marlowe.io/runtime-rest-client](https://input-output-hk.github.io/marlowe-ts-sdk/modules/\_marlowe\_io\_runtime\_rest\_client.html)).
5. **Phối hợp giữa ví và Runtime**: TS-SDK cung cấp các lớp trừu tượng cho Runtime Rest Client và ví, loại bỏ logic phức tạp, giúp tập trung vào logic kinh doanh cốt lõi. Việc xây dựng, ký và gửi giao dịch qua Cardano chưa bao giờ dễ dàng hơn! Gói này đơn giản hóa việc triển khai và quản lý vòng đời hợp đồng của bạn. (Xem [@marlowe.io/runtime-lifecycle](https://input-output-hk.github.io/marlowe-ts-sdk/modules/\_marlowe\_io\_runtime\_lifecycle.html)).
6. **Các ví dụ DApp mẫu (prototype)**: TS-SDK cung cấp các ví dụ DApp mẫu cô đọng, là nền tảng cho việc phát triển ứng dụng tùy chỉnh của bạn.

### Bắt đầu sử dụng:

Để bắt đầu, hãy xem phần Readme của [Marlowe TS-SDK](https://github.com/input-output-hk/marlowe-ts-sdk/). Tài liệu này bao gồm ví dụ, hướng dẫn chi tiết và công cụ, cung cấp tổng quan ngắn gọn về Marlowe TS-SDK và các tính năng của nó.

### Đóng góp và phản hồi của bạn:

Marlowe TypeScript SDK (TS-SDK) được phát hành theo giấy phép mã nguồn mở. Bạn có thể tự do đọc, sử dụng, triển khai, tùy chỉnh, cải thiện và phân nhánh mã này.&#x20;

Hãy xem các phần trong kho lưu trữ GitHub, đặc biệt là mục [thảo luận](https://github.com/input-output-hk/marlowe-ts-sdk/discussions). Phản hồi và đóng góp của bạn đóng vai trò quan trọng trong sự phát triển của dự án.

## Ví dụ về hợp đồng Marlowe:

Các ví dụ hợp đồng được phát hành trong gói npm [@marlowe.io/language-examples](https://input-output-hk.github.io/marlowe-ts-sdk/modules/\_marlowe\_io\_language\_examples.html) dưới dạng mã nguồn mở. Bạn có thể đọc, tùy chỉnh, cải thiện và phân nhánh chúng. Phản hồi và đóng góp của bạn được khuyến khích.

[Module vesting](https://input-output-hk.github.io/marlowe-ts-sdk/modules/\_marlowe\_io\_language\_examples.vesting.html): Triển khai hợp đồng vesting có sẵn trong TS-SDK, hỗ trợ Token Plans prototype và cung cấp triển khai hoàn chỉnh của hợp đồng Marlowe trong môi trường DApp web dựa trên TypeScript.

Atomic swap sử dụng tính năng vai trò mở (**sắp ra mắt**): Hợp đồng Marlowe này cho phép hai bên (người bán và người mua) trao đổi một token A lấy một token B mà không cần biết trước người mua.

## Prototype mã nguồn mở sử dụng TS-SDK:

Những prototype này được phát hành theo giấy phép mã nguồn mở. Bạn có thể đọc, triển khai, tùy chỉnh, cải thiện và phân nhánh chúng. Phản hồi và đóng góp của bạn được hoan nghênh.

[DApp Thanh Toán](https://docs.marlowe.iohk.io/docs/developer-tools/ts-sdk/payouts-dapp-prototype): Prototype này giúp người dùng tìm kiếm, theo dõi và rút token từ các hợp đồng thông minh Marlowe sử dụng role tokens, giúp đơn giản hóa quy trình theo dõi và rút tiền.

[Token Plans DApp](https://github.com/input-output-hk/marlowe-token-plans): Prototype này minh họa cách xây dựng DApp sử dụng Marlowe với các công nghệ web phổ biến như TypeScript và React. Đây là trường hợp sử dụng hợp đồng vesting có sẵn trong TS-SDK, cho phép tạo Token Plans ada trên Cardano.

_Lưu ý: Có thể bạn sẽ tìm thấy nhiều mẫu hơn trong các nguồn của Marlowe. Tuy nhiên trong phạm vi và mục tiêu của dự án. Chúng tôi chỉ cung cấp và giới thiệu các mẫu có tính ứng dụng cao nhất để tiết kiệm các nguồn lực_

