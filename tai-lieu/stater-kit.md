# Stater kit

### Giới thiệu[​](https://docs.marlowe.iohk.io/docs/getting-started/marlowe-starter-kit#introduction) <a href="#introduction" id="introduction"></a>

Marlowe Stater kit nhằm cung cấp các hướng dẫn toàn diện cho các nhà phát triển muốn thử nghiệm với các hợp đồng Marlowe đơn giản trên blockchain Cardano.&#x20;

Nếu bạn chưa quen thuộc với ngôn ngữ Marlowe, bạn có thể muốn thử nghiệm với [Marlowe Playground](https://playground.marlowe-lang.org/#/) trước khi bắt đầu với bộ khởi động.

### Lessons[​](https://docs.marlowe.iohk.io/docs/getting-started/marlowe-starter-kit#lessons) <a href="#lessons" id="lessons"></a>

#### Cơ bản <a href="#basic-operations" id="basic-operations"></a>

1. [**Using Marlowe Runtime's CLI**](https://github.com/input-output-hk/marlowe-starter-kit/blob/main/lessons/01-runtime-cli/01-runtime-cli.ipynb) -- Hướng dẫn từng bước để tạo trái phiếu zero-coupon bằng giao diện dòng lệnh của Marlowe Runtime:
2. [**Using Marlowe Runtime's REST API**](https://github.com/input-output-hk/marlowe-starter-kit/blob/main/lessons/02-runtime-rest/02-runtime-rest.ipynb) -- Hướng dẫn từng bước để tạo trái phiếu zero-coupon bằng giao diện Marlowe Runtime's REST API.

#### Các hợp đồng[​](https://docs.marlowe.iohk.io/docs/getting-started/marlowe-starter-kit#contracts) <a href="#contracts" id="contracts"></a>

3. [**Zero-coupon bond using Marlowe's CLI**](https://github.com/input-output-hk/marlowe-starter-kit/blob/main/lessons/03-marlowe-cli/03-marlowe-cli.ipynb) -- Chứng minh các bước để tạo trái phiếu zero-coupon bằng giao diện dòng lệnh của Marlowe.
4. [**Escrow using Marlowe Runtime's REST API**](https://github.com/input-output-hk/marlowe-starter-kit/blob/main/lessons/04-escrow-rest/04-escrow-rest.ipynb) -- Chứng minh làm thế nào để tạo một hợp đồng ký quỹ bằng Marlowe Runtime's REST API.
5. [**Swap of Ada for Djed using Marlowe Runtime's REST API**](https://github.com/input-output-hk/marlowe-starter-kit/blob/main/lessons/05-swap-rest/05-swap-rest.ipynb) -- ví dụ về hợp đồng hoán đổi token cho phép các bên trao đổi ada lấy một token. Trong ví dụ này, hợp đồng sử dụng token vai trò để cấp quyền và Runtime tạo ra chúng.
6. [**Simple web application using a CIP-30 wallet**](https://github.com/input-output-hk/marlowe-starter-kit/tree/main/lessons/06-cip30#example-of-using-marlowe-runtime-with-a-cip30-wallet) -- Cách sử dụng ví CIP-30 tương thích với Babbage, chẳng hạn như Nami, để ký các giao dịch Marlowe.

#### An toàn & kiểm tra[​](https://docs.marlowe.iohk.io/docs/getting-started/marlowe-starter-kit#safety--testing) <a href="#safety--testing" id="safety--testing"></a>

7. [**Checking the safety of a Marlowe contract**](https://github.com/input-output-hk/marlowe-starter-kit/blob/main/lessons/07-safety/07-safety.ipynb) -- Hướng dẫn cách sử dụng một trong ba công cụ để kiểm tra tính an toàn của một hợp đồng trước khi gửi nó lên blockchain.

#### Additional Topics[​](https://docs.marlowe.iohk.io/docs/getting-started/marlowe-starter-kit#additional-topics) <a href="#additional-topics" id="additional-topics"></a>

8. [**Experimental web application using a CIP-45 wallet**](https://github.com/input-output-hk/marlowe-starter-kit/tree/main/lessons/08-cip45) -- Ví dụ về cách sử dụng ví tiêu chuẩn CIP-45, như là Eternl để ký các giao dịch Marlowesan.
9. [**Minting with Marlowe tools**](https://github.com/input-output-hk/marlowe-starter-kit/blob/main/lessons/09-minting/09-minting.ipynb) -- Chứng minh cách tạo token gốc Cardano bằng cách sử dụng công cụ CLI của Marlowe và cách sử dụng các token như vậy trong một hợp đồng Marlowe.
