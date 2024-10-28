# Tổng quan

### Kiến trúc&#x20;

Phần backend cho runtime Marlowe bao gồm một dịch vụ chỉ mục và truy vấn chuỗi `(marlowe-chain-indexer / marlowe-chain-sync)`, một dịch vụ chỉ mục và truy vấn hợp đồng cho các hợp đồng Marlowe `(marlowe-indexer / marlowe-sync)`, và một dịch vụ tạo giao dịch cho các hợp đồng Marlowe `(marlowe-tx)`.&#x20;

Các dịch vụ backend này hoạt động cùng nhau và dựa vào [`cardano-node`](https://github.com/input-output-hk/cardano-node/blob/master/README.rst) để kết nối với blockchain và[ PostgreSQL](https://www.postgresql.org/) để lưu trữ dữ liệu bền vững. Truy cập vào các dịch vụ backend được cung cấp qua một khách hàng dòng lệnh `(marlowe-runtime-cli)` hoặc một máy chủ `REST/WebSocket` (web-server) sử dụng payload JSON.&#x20;

Các ứng dụng web có thể tích hợp với ví nhẹ [**CIP-30**](https://cips.cardano.org/cips/cip30/) để ký giao dịch, trong khi các ứng dụng doanh nghiệp có thể tích hợp với [**cardano-wallet**](https://github.com/input-output-hk/cardano-wallet/blob/master/README.md), [**cardano-cli**](https://github.com/input-output-hk/cardano-node/blob/master/cardano-cli/README.md) hoặc [**cardano-hw-cli**](https://github.com/vacuumlabs/cardano-hw-cli/blob/develop/README.md) để ký giao dịch.

![Sơ đồ kiến trúc của Marlowe Runtime](https://docs.marlowe.iohk.io/assets/images/Marlowe-Runtime-Architecture-31-Jan-2024-96a2a56901d6ef3b014a1c1f4b480e87.png)

### Triển khai

Có ba phương pháp khả dụng để triển khai các [dịch vụ Marlowe Runtime](https://docs.marlowe.iohk.io/docs/platform-and-architecture/architecture), phần backend cho việc quản lý các hợp đồng Marlowe trên blockchain Cardano.

Ba phương pháp triển khai là:

* Triển khai qua dịch vụ (không phụ thuộc vào hệ điều hành, dựa trên web)
* Triển khai cục bộ chỉ sử dụng Docker (không phụ thuộc vào hệ điều hành, kiến trúc Intel, Windows, Linux hoặc Mac)
* Triển khai cục bộ kết hợp giữa Docker và Nix (chỉ dành cho Linux)

_Lưu ý: Kiến trúc ARM không được hỗ trợ tại thời điểm này. Điều này bao gồm cả máy Apple M1 và M2._

### **Triển khai dịch vụ**

Phương pháp triển khai đơn giản và nhanh nhất là sử dụng dịch vụ được cung cấp bởi [Demeter.run](https://demeter.run/).

* Phương pháp này không gắn liền với bất kỳ nền tảng cụ thể nào.
* Yêu cầu hệ thống duy nhất là một trình duyệt.
* Chỉ mất vài phút để bắt đầu.

Nền tảng phát triển web3 Demeter.run cung cấp một số tiện ích mở rộng. Hai tiện ích mở rộng sau đây phải được kích hoạt cho các dịch vụ Marlowe Runtime:

* Tiện ích mở rộng Marlowe Runtime Cardano
* Tiện ích mở rộng Cardano Node

Hai tiện ích mở rộng này cùng nhau cung cấp các dịch vụ backend Marlowe Runtime và một node Cardano, vì vậy bạn không cần phải thiết lập biến môi trường, cài đặt công cụ, chạy toàn bộ stack các dịch vụ backend tại chỗ, hoặc chờ đồng bộ nút Cardano.

Để biết chi tiết và hướng dẫn từng bước, hãy xem tài liệu [triển khai Demeter.run](https://github.com/input-output-hk/marlowe-starter-kit/blob/main/docs/demeter-run.md) trong bộ khởi động Marlowe.

Ngoài ra, bạn có thể xem [video hướng dẫn](https://www.youtube.com/watch?v=IHfVRO\_7KeM\&list=PLnPTB0CuBOByGbvUmubLs0a3Y0b\_HqGPD\&index=3) ngắn (2:32) về quy trình này.

### **Triển khai cục bộ chỉ sử dụng Docker**

Bạn có thể triển khai các dịch vụ Marlowe Runtime, Jupyter notebooks và một nút Cardano cục bộ thông qua Docker. Quy trình này chạy mọi thứ trong các container Docker.

* Cần kiến trúc dựa trên Intel.
* Cần cài đặt Docker cục bộ.
* Cần tài nguyên hệ thống đáng kể trên máy cục bộ của bạn.
* Xem Tài nguyên Đề xuất để Triển khai Marlowe Runtime.
* Có thể mất vài giờ hoặc hơn để đồng bộ nút Cardano lần đầu.\
  Để biết thêm chi tiết, hãy xem phần Triển khai cục bộ với Docker trong bộ khởi động Marlowe.

### **Triển khai cục bộ kết hợp giữa Docker và Nix**

Bạn có thể triển khai các dịch vụ Marlowe Runtime và một nút Cardano trong Docker, kết hợp với việc sử dụng shell Nix để chạy các Jupyter notebooks từ HOST và thiết lập tất cả các công cụ và biến môi trường cần thiết.

* Cần Linux.
* Cần cài đặt Docker cục bộ.
* Cần Nix.\
  Để biết chi tiết, hãy xem phần Triển khai Runtime với Docker + Jupyter notebook với Nix trong tài liệu Triển khai cục bộ với Docker trong bộ khởi động Marlowe.

### **Thời gian cần thiết để đồng bộ nút mạng chính công khai, preprod hoặc preview**

Thời gian cần thiết để đồng bộ nút Cardano lần đầu thay đổi theo một yếu tố mười, tùy thuộc vào môi trường phần cứng bạn đang sử dụng. Trừ khi bạn có SSD nhanh, bạn có thể gặp thời gian đồng bộ chậm.&#x20;

Thời gian đồng bộ trên các mạng preview hoặc preprod có thể chỉ mất từ 20 phút đến vài giờ hoặc hơn. Thời gian đồng bộ lần đầu trên mainnet có thể thay đổi từ chỉ 14 giờ đến vài ngày.

Cho đến khi quá trình đồng bộ đạt đến mức mà các hợp đồng Marlowe đã đạt đến nút preview, bảng "block" trong schema "marlowe" sẽ trống. Sau số khối 7791724 (mainnet), 224384 (preprod) hoặc 39646 (preview), bạn sẽ bắt đầu thấy dữ liệu trong bảng block.&#x20;

_Quan trọng là bạn phải nhận thức rằng bạn sẽ không thể khởi động trên mainnet chỉ trong một hoặc hai giờ. Thời gian đồng bộ ban đầu cần một chút kiên nhẫn._
