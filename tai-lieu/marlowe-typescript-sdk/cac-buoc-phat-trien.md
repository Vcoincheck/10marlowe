# Các bước phát triển

### Phát triển <a href="#development" id="development"></a>

### Xây dựng <a href="#build" id="build"></a>

Để bắt đầu phát triển SDK, bạn cần cài đặt các dependencies và xây dựng các gói (package).

Nếu muốn xây dựng một gói riêng lẻ, bạn có thể sử dụng cờ `-w` hoặc thực thi lệnh xây dựng từ thư mục của gói đó.

```
# From the root folder
$ npm run build -w @marlowe.io/language-core-v1
# Or you can enter the package and build
$ cd packages/language/core/v1
$ npm run build
```

### Clean <a href="#clean" id="clean"></a>

Để xóa các tệp build artifacts, bạn có thể sử dụng lệnh `clean`.

### Kiểm thử <a href="#tests" id="tests"></a>

Lưu ý: Nên làm sạch và xây dựng lại các gói trước khi chạy kiểm thử để đảm bảo bạn đang làm việc với phiên bản mã mới nhất.

```
$ npm run clean && npm run build
```

#### Kiểm thử đơn vị&#x20;

Để chạy kiểm thử đơn vị cho tất cả các gói, từ thư mục gốc, bạn có thể thực hiện lệnh `test`:&#x20;

```
//$ npm test
```

Nếu bạn muốn chạy kiểm thử cho một gói cụ thể, bạn có thể sử dụng cờ -w hoặc thực hiện lệnh từ thư mục của gói đó.

```
# Từ thư mục root
$ npm run clean && npm run build && npm test -w @marlowe.io/language-core-v1
# Hoặc bạn có thể vào thư mục của gói và chạy kiểm thử. Bạn sẽ cần dọn dẹp và xây dựng lại gói cục bộ một cách phù hợp.
# Việc này phụ thuộc của gói hiện tại nếu bạn sửa đổi một trong số chúng.
# ví dụ: packages/language/core/v1 phụ thuộc vào packages/adapter. Hãy chắc chắn rằng bạn đã xây dựng chính xác gói này trước khi chạy thử nghiệm của bạn theo cách đó.
$ cd packages/language/core/v1
$ npm test
```

### Kiểm thử tích hợp/E2E&#x20;

#### Thiết lập tệp cấu hình môi trường&#x20;

* Tạo tệp `./env/.test-env.json` tại thư mục gốc của dự án.&#x20;
* Sao chép/Dán đoạn mã sau và cung cấp các tham số cần thiết.

```
{
  "bank": {
    "seedPhrase": "<seed phrase separated by spaces>"
  },
  "lucid": {
    "blockfrostProjectId": "<your-blockfrost-project-id>",
    "blockfrostUrl": "<your-blockfrost-url>"
  },
  "network": "Preprod",
  "runtimeURL": "http://<path-to-a-runtime-instance>:<a-port>"
}
```

**Cách tạo Seed Phrase cho một ví **_**trắng**_** ?**

Tại thư mục gốc của dự án :

```
npm run -w @marlowe.io/testing-kit genSeedPhrase
```

1. Sao chép/dán các từ bên trong dấu ngoặc kép vào tệp môi trường.&#x20;
2. Đi đến một trong những tiện ích mở rộng ví yêu thích của bạn và khôi phục ví bằng cụm từ hạt giống này.&#x20;
3. Lấy Địa chỉ Thanh toán từ các tiện ích mở rộng trình duyệt này để cấp phát cho Ngân hàng của bạn với faucet.&#x20;

#### Cách thêm tAda vào Ví trắng qua faucet?&#x20;

1. Lấy địa chỉ thanh toán Ví trắng của bạn.
2. Truy cập [https://docs.cardano.org/cardano-testnet/tools/faucet](https://docs.cardano.org/cardano-testnet/tools/faucet) và yêu cầu test Ada cho địa chỉ này.
3. Chờ một chút cho đến khi giao dịch được xác nhận và bạn sẽ có thể chạy các bài kiểm tra.&#x20;

#### Chạy các bài kiểm tra E2E&#x20;

Để chạy các bài kiểm tra e2e cho tất cả các gói, từ thư mục gốc, bạn có thể thực hiện lệnh `test:e2e`:&#x20;

Nếu bạn muốn chạy bài kiểm tra cho một gói duy nhất, bạn có thể sử dụng cờ `-w` hoặc thực hiện lệnh xây dựng từ thư mục gói.

```
# From the root folder
$ npm run clean && npm run build && npm run test:e2e -w @marlowe.io/runtime-lifecycle
# Or you can enter the package folder and test. You will have to clean and build properly the local package
# dependencies of this current package if you modify one of them
$ cd packages/runtime/client/rest
$ npm run test:e2e
```

### Tạo tài liệu <a href="#documentation" id="documentation"></a>

> ⚠ Bạn cần [xây dựng các gói](https://github.com/input-output-hk/marlowe-ts-sdk/blob/main/doc/howToDevelop.md#build) trước khi có thể biên dịch tài liệu!

Nếu đây là lần đầu tiên bạn phục vụ tài liệu, bạn sẽ cần phải xây dựng giao diện (**theme**) trước.

```
$ cd doc/theme
$ npm run build
$ cd ../../
```

Để biên dịch tất cả tài liệu

```
// $ npm run docs
```

Phục vụ (serve) tài liệu.

```
// $ npm run serve
```

Bây giờ bạn có thể xem tài liệu tại địa chỉ `localhost:1337`.

Tài liệu được xây dựng bằng TypeDoc, được xuất bản qua GitHub Pages và được lưu trữ tại [https://input-output-hk.github.io/marlowe-ts-sdk](https://input-output-hk.github.io/marlowe-ts-sdk).

Mỗi dự án con cần có một tệp `typedoc.json` trong thư mục gốc của dự án con như được chỉ định trong trường workspaces trong `./packages.json`. Ví dụ, nếu có một dự án được chỉ định là "some-project":

```
// ./packages.json
{
  ...,
  "workspaces": ["./path/to/some-project"]
}
```

Cần có một tệp `typedoc.json` trong `./path/to/some-project`, và tệp này cần có các thuộc tính tương tự như ví dụ dưới đây:

```
// ./path/to/some-project/typedoc.json
{
  "entryPointStrategy": "expand",
  "entryPoints": ["./src"],
  "tsconfig": "./tsconfig.json"
}
```

#### Changelog&#x20;

Dự án này quản lý lịch sử thay đổi của nó bằng công cụ [scriv](https://github.com/nedbat/scriv).&#x20;

Các Pull Request (PR) sẽ tự động được kiểm tra để xem có mục nhập mới nào trong thư mục `./changelog.d` hay không.&#x20;

Nếu một PR không cần mục nhập vào lịch sử thay đổi, bạn có thể thêm nhãn `No Changelog Required` để vô hiệu hóa kiểm tra này.

Để tạo một mẫu mục nhập lịch sử thay đổi mới, hãy thực hiện các bước sau:

1. Tạo một tệp mục nhập mới bằng cách sử dụng lệnh sau.
2. Chỉnh sửa tệp mới với nội dung phù hợp cho PR của bạn và thực hiện commit. Đọc tài liệu của scriv để tìm hiểu thêm về cách sử dụng công cụ này.

Để tập hợp tất cả các mục nhập lịch sử thay đổi vào một tệp duy nhất, hãy thực hiện lệnh `std` từ shell nix và chạy kịch bản `build-changelog`.&#x20;

Lệnh này sẽ xóa tất cả các mục nhập từ thư mục `changelog.d` và cập nhật tệp `CHANGELOG.md`.

### Xuất bản&#x20;

Hiện tại, SDK được xuất bản thủ công lên npm. Nhiệm vụ PLT-6939 ghi lại công việc để tự động hóa quá trình này thông qua CI.

Trước khi xuất bản, hãy xác minh rằng mọi thứ hoạt động như mong đợi:

* Clean & Build

```
$ npm run clean && npm run build
```

* Test

```
$ npm t && npm run test:e2e
```

* Để kiểm tra các gói, bạn có thể đóng gói tất cả các gói khác nhau thành các tệp tarball bằng cách sử dụng lệnh:

```
$ npm --workspaces pack --pack-destination dist
```

Và trong một dự án riêng biệt, bạn có thể cài đặt các tệp tarball bằng cách sử dụng URL tệp khi khai báo phụ thuộc.

```
{
  "dependencies": {
    "@marlowe.io/runtime-lifecycle": "file:<path-to-dist>/marlowe.io-runtime-lifecycle-0.4.0-beta.tgz",
    "@marlowe.io/runtime-rest-client": "file:<path-to-dist>/marlowe.io-runtime-rest-client-0.4.0-beta.tgz",
    "@marlowe.io/adapter": "file:<path-to-dist>/marlowe.io-adapter-0.4.0-beta.tgz",
    "@marlowe.io/runtime-core": "file:<path-to-dist>/marlowe.io-runtime-core-0.4.0-beta.tgz",
    "@marlowe.io/language-core-v1": "file:<path-to-dist>/marlowe.io-language-core-v1-0.4.0-beta.tgz",
    "@marlowe.io/language-examples": "file:<path-to-dist>/marlowe.io-language-examples-0.4.0-beta.tgz",
    "@marlowe.io/wallet": "file:<path-to-dist>/marlowe.io-wallet-0.4.0-beta.tgz"
  }
}
```

TODO \[\[Kiểm tra trước khi xuất bản]] Hướng dẫn bản đồ cục bộ và hướng dẫn đóng gói này TODO hướng dẫn về cách xuất bản thủ công

Để kiểm tra rằng cơ chế xuất /nhập hoạt động trên trình duyệt sử dụng bản đồ nhập, bạn có thể:

1. Tạo một nhánh kiểm tra trong một nhánh fork của kho lưu trữ.
2. Dọn dẹp và xây dựng dự án bằng lệnh `npm run clean && npm run build`.
3. Xóa `dist` khỏi `.gitignore`.
4. Cam kết các thư mục `dist` của các gói khác nhau.
5. Lấy hash git đầy đủ của cam kết bằng lệnh `git rev-parse HEAD`.
6. Xuất bản nhánh trong fork của bạn.
7. Chỉnh sửa `rollup/config.mjs` và đặt đúng chủ sở hữu và phiên bản khi xây dựng bản đồ nhập cho jsdelivr-gh.
8. Xây dựng lại dự án bằng lệnh `npm run build`.
9. Chỉnh sửa HTML trong thư mục `pocs` để sử dụng `/dist/jsdelivr-gh-importmap.js`.
10. Từ thư mục gốc, chạy lệnh `npx http-server --port 1337 -c-1 -o ./`.
11. Xác minh rằng mỗi ví dụ poc hoạt động đúng cách.
