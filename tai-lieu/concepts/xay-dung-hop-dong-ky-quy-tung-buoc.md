# Xây dựng hợp đồng Ký quỹ từng bước

### Tutorial. Building upon an escrow contract.​ <a href="#tutorial-building-upon-an-escrow-contract" id="tutorial-building-upon-an-escrow-contract"></a>

### Hợp đồng số 1: Tiết kiệm.​ <a href="#contract-1-locked-savings" id="contract-1-locked-savings"></a>

**Mục tiêu:**

* Tiết kiệm một khoản tiền cho tương lai.

**Điều kiện hợp đồng:**

* Alice có thời gian từ thời điểm 0 đến thời điểm 10 để thực hiện một khoản đặt cọc (cam kết) 500 ada.&#x20;
* Các khoản tiền sẽ bị khóa cho đến thời điểm 100.&#x20;
* Chỉ sau thời điểm 100, các khoản tiền mới có thể được rút.

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

**Marlowe code:**

```rust
CommitCash (IdentCC 1) 1
           (ConstMoney 500)
           10 100 Null Null
```

**Các trường hợp thử nghiệm:**

* Ada không thể được _cam kết_ sau thời điểm 10.
* Ada không thể được rút trước thời điểm 100.
* Ada có thể được rút sau thời điểm 100.

### Hợp đồng số 2: Thanh toán đơn giản với thời hạn yêu cầu.​ <a href="#contract-2-simple-payment-with-time-limit-to-claim" id="contract-2-simple-payment-with-time-limit-to-claim"></a>

**Mục tiêu:**

* Thanh toán cho một sản phẩm hoặc dịch vụ.

**Điều kiện hợp đồng:**

* Alice có thời gian từ thời điểm 0 đến thời điểm 10 để thực hiện một khoản đặt cọc 500 ada.&#x20;
* Bob có thể yêu cầu thanh toán từ thời điểm 11 đến thời điểm 100.&#x20;
* Alice có thể rút lại khoản tiền sau thời điểm 100 nếu Bob không yêu cầu thanh toán.

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

**Marlowe code:**

```rust
CommitCash (IdentCC 1) 1
           (ConstMoney 500)
           10 100
           (Pay (IdentPay 1) 1 2
                (AvailableMoney (IdentCC 1))
                100 Null)
           Null
```

**Các trường hợp thử nghiệm:**

* Tiền mặt không thể được cam kết sau thời điểm 10.
* Ada có thể được yêu cầu bất kỳ lúc nào sau khi cam kết và trước khi thời điểm 100 được đạt.
* Ada không thể được rút trước thời điểm 100.
* Chỉ người thứ hai có thể yêu cầu thanh toán.
* Ada có thể được rút sau thời điểm 100.

### **Hợp đồng số 3: Ủy quyền thanh toán**

**Mục tiêu:**

* Thanh toán cho một sản phẩm hoặc dịch vụ nhưng thanh toán phải được ủy quyền từ người mua.

**Điều kiện hợp đồng:**

* Alice có thời gian từ thời điểm 0 đến thời điểm 10 để gửi 500 ada.
* Alice có thời gian từ 11 đến 40 để chọn xem cô ấy có thực sự ủy quyền thanh toán cho Bob hay không.
* Nếu Alice chọn thanh toán, Bob có thể yêu cầu thanh toán từ thời điểm 41 đến thời điểm 100.
* Các khoản tiền có thể được rút lại bởi Alice sau thời điểm 100.

<figure><img src="../../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

**Marlowe code:**

```rust
CommitCash (IdentCC 1) 1
           (ConstMoney 500)
           10 100
           (When (PersonChoseThis (IdentChoice 1) 1 1)
                 40
                 (Pay (IdentPay 1) 1 2
                      (AvailableMoney (IdentCC 1))
                      100 Null)
                 (RedeemCC (IdentCC 1) Null))
           Null
```

**Các trường hợp thử nghiệm:**

* Alice ủy quyền cho khoản thanh toán
* Alice không ủy quyền

### Hợp đồng số 4: **Thanh toán trừ khi bị từ chối rõ ràng** <a href="#contract-4-pay-unless-explicit-rejection" id="contract-4-pay-unless-explicit-rejection"></a>

**Mục tiêu:**\
Thanh toán cho Bob trừ khi Alice rõ ràng từ chối khoản thanh toán.

**Điều kiện hợp đồng:**

* Alice có thời gian từ thời điểm 0 đến thời điểm 10 để gửi 500 ada.
* Alice có thời gian từ 11 đến 40 để chọn xem cô ấy có từ chối khoản thanh toán hay không.
* Nếu Alice chọn không thanh toán, các khoản tiền có thể được rút lại.
* Nếu Alice chọn thanh toán hoặc không đưa ra lựa chọn, Bob có thể yêu cầu thanh toán từ thời điểm 41 đến thời điểm 100.
* Các khoản tiền có thể được rút lại bởi Alice sau thời điểm 100.

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

**Marlowe code:**

```rust
CommitCash (IdentCC 1) 1
           (ConstMoney 500)
           10 100
           (When (PersonChoseSomething (IdentChoice 1) 1)
                 40
                 (Choice (PersonChoseThis (IdentChoice 1) 1 0)
                         (RedeemCC (IdentCC 1) Null)
                         (Pay (IdentPay 1) 1 2
                              (AvailableMoney (IdentCC 1))
                              100 Null))
                 (Pay (IdentPay 2) 1 2
                      (AvailableMoney (IdentCC 1))
                      100 Null))
           Null
```

**Các trường hợp thử nghiệm:**

* Bob có thể thu tiền ngay cả khi Alice không đưa ra hướng dẫn.
* Alice có thể hủy thanh toán.
* Bob không thể yêu cầu thanh toán trước block 40 hoặc trước khi có sự chấp thuận từ Alice.

### Hợp đồng số 5: Ký quỹ đơn giản​ <a href="#contract-5-simple-escrow" id="contract-5-simple-escrow"></a>

**Mục tiêu:**

* Thanh toán cho Bob khi hai trong ba người bỏ phiếu **ủng hộ thanh toán**
* &#x20; Hoàn tiền cho Alice khi hai trong ba người bỏ phiếu **không thanh toán**.

**Điều kiện hợp đồng:**

* Alice có thời gian từ thời điểm 0 đến thời điểm 10 để gửi 500 ada.
* Alice có thời gian từ 11 đến 40 để bỏ phiếu xem cô ấy có chấp thuận hay từ chối khoản thanh toán.
* Bob có thời gian từ 11 đến 60 để bỏ phiếu xem anh ấy có chấp thuận hay từ chối khoản thanh toán.
* Carol có thời gian từ 11 đến 60 để bỏ phiếu xem cô ấy có chấp thuận hay từ chối khoản thanh toán.
* Nếu hai trong ba người tham gia bỏ phiếu không thanh toán, các khoản tiền có thể được rút lại sau thời điểm 100.
* Nếu hai trong ba người tham gia bỏ phiếu ủng hộ thanh toán, Bob có thể yêu cầu thanh toán từ thời điểm 61 đến thời điểm 100.
* Các khoản tiền có thể được rút lại bởi Alice sau thời điểm 100.

<figure><img src="../../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

**Marlowe Code:**

```rust
CommitCash (IdentCC 1) 1
           (ConstMoney 500)
           10 100
           (When (OrObs (AndObs (PersonChoseThis (IdentChoice 1) 1 1)
                                (PersonChoseThis (IdentChoice 1) 2 1))
                        (OrObs (AndObs (PersonChoseThis (IdentChoice 1) 1 1)
                                       (PersonChoseThis (IdentChoice 1) 3 1))
                               (AndObs (PersonChoseThis (IdentChoice 1) 2 1)
                                       (PersonChoseThis (IdentChoice 1) 3 1))))
                 100
                 (Pay (IdentPay 1) 1 2
                      (AvailableMoney (IdentCC 1))
                      100 Null)
                 Null)
           Null
```

**Các trường hợp thử nghiệm**:

* Thanh toán chỉ có thể được yêu cầu khi 2 trong 3 người tham gia đã bỏ phiếu ủng hộ thanh toán.
* Alice và Bob đồng ý thanh toán.
* Bob và Carol đồng ý thanh toán.
* Alice và Carol đồng ý thanh toán.
* Chỉ người thứ hai (Bob) có thể yêu cầu thanh toán.
* Ada có thể được rút lại sau khối 100.

### Hợp đồng số 6: Ký quỹ hoàn chỉnh​ <a href="#contract-6-complete-escrow" id="contract-6-complete-escrow"></a>

**Mục tiêu**:

* Thanh toán cho Bob khi hai trong ba người bỏ phiếu ủng hộ thanh toán
* Hoàn tiền cho Alice khi hai trong ba người bỏ phiếu không thanh toán.&#x20;
* Cải tiến Hợp đồng 5 để cho phép Alice được hoàn tiền sớm hơn nếu kết quả bỏ phiếu không thanh toán.

**Điều kiện hợp đồng:**

* Alice có thời gian từ thời điểm 0 đến thời điểm 10 để gửi 500 ada.
* Alice có thời gian từ 11 đến 40 để bỏ phiếu xem cô ấy có chấp thuận hay từ chối khoản thanh toán.
* Bob có thời gian từ 11 đến 60 để bỏ phiếu xem anh ấy có chấp thuận hay từ chối khoản thanh toán.
* Carol có thời gian từ 11 đến 60 để bỏ phiếu xem cô ấy có chấp thuận hay từ chối khoản thanh toán.
* Nếu hai trong ba người tham gia bỏ phiếu không thanh toán, các khoản tiền có thể được rút lại ngay lập tức.
* Nếu hai trong ba người tham gia bỏ phiếu ủng hộ thanh toán, Bob có thể yêu cầu thanh toán từ thời điểm 61 đến thời điểm 100.
* Các khoản tiền có thể được rút lại bởi Alice sau thời điểm 100.

<figure><img src="../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

**Luồng hoạt động của hợp đồng theo sơ đồ hình cây.**

<figure><img src="../../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

**Marlowe Code:**

```rust
CommitCash (IdentCC 1) 1
           (ConstMoney 500)
           10 100
           (When (OrObs (OrObs (AndObs (PersonChoseThis (IdentChoice 1) 1 1)
                                       (PersonChoseThis (IdentChoice 1) 2 1))
                               (OrObs (AndObs (PersonChoseThis (IdentChoice 1) 1 1)
                                              (PersonChoseThis (IdentChoice 1) 3 1))
                                      (AndObs (PersonChoseThis (IdentChoice 1) 2 1)
                                              (PersonChoseThis (IdentChoice 1) 3 1))))
                        (OrObs (AndObs (PersonChoseThis (IdentChoice 1) 1 0)
                                       (PersonChoseThis (IdentChoice 1) 2 0))
                               (OrObs (AndObs (PersonChoseThis (IdentChoice 1) 1 0)
                                              (PersonChoseThis (IdentChoice 1) 3 0))
                                      (AndObs (PersonChoseThis (IdentChoice 1) 2 0)
                                              (PersonChoseThis (IdentChoice 1) 3 0)))))
                 100
                 (Choice (OrObs (AndObs (PersonChoseThis (IdentChoice 1) 1 1)
                                        (PersonChoseThis (IdentChoice 1) 2 1))
                                (OrObs (AndObs (PersonChoseThis (IdentChoice 1) 1 1)
                                               (PersonChoseThis (IdentChoice 1) 3 1))
                                       (AndObs (PersonChoseThis (IdentChoice 1) 2 1)
                                               (PersonChoseThis (IdentChoice 1) 3 1))))
                         (Pay (IdentPay 1) 1 2
                              (AvailableMoney (IdentCC 1))
                              100 Null)
                         (RedeemCC (IdentCC 1) Null))
                 Null)
           Null
```

**Các trường hợp thử nghiệm**:

* Kiểm tra rằng khi cả Alice và Carol đều chọn KHÔNG thanh toán, Alice có thể ngay lập tức rút lại khoản tiền.
