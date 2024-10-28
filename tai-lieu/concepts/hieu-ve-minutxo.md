# Hiểu về minUTXO

Thuật ngữ **minUTXO** đề cập đến "Đầu ra tối thiểu của Giao dịch chưa chi tiêu". Tối thiểu này tồn tại để ngăn chuỗi tăng trưởng vô hạn về kích thước. Ngoài ra, minUTXO còn ngăn người dùng gửi thư rác giao dịch.

Có một số [phép tính](https://github.com/input-output-hk/cardano-ledger/blob/master/doc/explanations/min-utxo-alonzo.rst) liên quan để xác định tối thiểu này. Tuy nhiên, trong bối cảnh Marlowe, minUTXO được đặt trong hợp đồng thông minh tại thời điểm tạo. Hầu hết các hợp đồng đặt số tiền này ở đâu đó giữa hai đến ba ada.

Một ví dụ thực tiễn là cờ `--min-utxo` được sử dụng trong việc tạo hợp đồng cho `marlowe-runtime-cli`. Một ứng dụng [mẫu](https://github.com/input-output-hk/marlowe-cardano/blob/587333d67887998c8f15566fdcfeac713acbbf32/marlowe-apps/create-example-contract.sh#L51) cho thấy cách thức này yêu cầu một số lượng lovelace tối thiểu để lưu trữ trong hợp đồng; nếu không, lệnh sẽ thất bại.

minUTXO cũng được thi hành tương tự đối với các cách tương tác khác với Runtime. Khi sử dụng REST API, phần thân hợp đồng yêu cầu cần một `minUTxODeposit`. Điều này cũng sẽ xảy ra với TypeScript SDK hoặc bất kỳ thư viện khách tương tự nào được xây dựng trên Marlowe.

Đối với các token gốc, minUTXO ngụ ý rằng một số lượng ada không bằng 0 phải được gửi cùng với các token gốc.

### Đọc thêm​ <a href="#additional-reading" id="additional-reading"></a>

* [Minimum ada value requirement](https://cardano-ledger.readthedocs.io/en/latest/explanations/min-utxo-mary.html)
* [Minimum ada value with native tokens](https://docs.cardano.org/native-tokens/minimum-ada-value-requirement/)
