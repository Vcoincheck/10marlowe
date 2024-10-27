# ACTUS và Marlowe

Hướng dẫn này cung cấp một cái nhìn tổng quan về ý tưởng chung của tiêu chuẩn ACTUS cho việc đại diện thuật toán của các hợp đồng tài chính, cùng với các ví dụ được triển khai trong Marlowe.

**ACTUS**\
Quỹ Nghiên cứu Tài chính ACTUS - [actusfrf.org](https://www.actusfrf.org) - đã tạo ra một tiêu chuẩn cho các hợp đồng tài chính, được phân loại thông qua một hệ thống phân loại và được mô tả trong một thông số kỹ thuật chi tiết.\
Các tiêu chuẩn ACTUS xây dựng trên sự hiểu biết rằng các hợp đồng tài chính là các thỏa thuận pháp lý giữa hai (hoặc nhiều) bên đối tác về việc trao đổi dòng tiền trong tương lai.&#x20;

Lịch sử, những thỏa thuận pháp lý như vậy được mô tả bằng ngôn ngữ tự nhiên dẫn đến sự mơ hồ và sự đa dạng nhân tạo. Để đáp ứng điều này, các tiêu chuẩn ACTUS định nghĩa các hợp đồng tài chính bằng một tập hợp các điều khoản hợp đồng và các hàm xác định ánh xạ các điều khoản này thành các nghĩa vụ thanh toán trong tương lai. Nhờ đó, có thể mô tả phần lớn các công cụ tài chính thông qua một tập hợp chỉ hơn 30 loại hợp đồng hoặc các mẫu mô-đun tương ứng.

Các thông số kỹ thuật ACTUS cung cấp một loạt các bài tập để triển khai trong Marlowe, và chúng tôi minh họa một cách tiếp cận trong ví dụ sau.

**Ví dụ về Trái phiếu Zero Coupon đơn giản**\
Trái phiếu zero-coupon là một loại chứng khoán nợ không trả lãi (coupon) nhưng được phát hành với mức giảm giá, mang lại lợi nhuận khi đến hạn khi trái phiếu được đổi lấy giá trị danh nghĩa đầy đủ của nó.\
Ví dụ, một nhà đầu tư `(investor)` có thể mua một trái phiếu có giá 1000 Lovelace với mức giảm giá 15%. Cô ấy trả 850 Lovelace cho người phát hành trái phiếu trước thời điểm bắt đầu, ở đây là `1672531200` (2023-01-01 00:00:00 GMT).

Một tháng sau, sau ngày đáo hạn (_maturity_), thời gian `1675209600` (2023-02-01 00:00:00 GMT), nhà đầu tư có thể đổi trái phiếu để lấy giá trị danh nghĩa đầy đủ của nó, tức là 1000 Lovelace.

```rust
When [
  (Case
     (Deposit
        "investor"
        "investor" ada
        (Constant 850))
     (Pay
        "investor"
        (Party "issuer") ada
        (Constant 850)
        (When [
           (Case
              (Deposit
                 "investor"
                 "issuer" ada
                 (Constant 1000))
              (Pay
                 "investor"
                 (Party "investor") ada
                 (Constant 1000)
                 Close))
            ]
            1675209600
            Close)))
     ]
     1672531200
     Close
```

Hợp đồng này có một nhược điểm đáng kể. Một khi nhà đầu tư `(investor)` đã gửi 850 Lovelace, số tiền này sẽ ngay lập tức được thanh toán cho người phát hành `(issuer)` (nếu nhà đầu tư không thực hiện đầu tư kịp thời, hợp đồng sẽ kết thúc). Sau đó, có hai kết quả có thể xảy ra:

1. `issuer` gửi 1000 Lovelace vào tài khoản của `investor`, và số tiền này sau đó sẽ được thanh toán ngay lập tức cho `investor`.
2. Nếu `investor` không thực hiện gửi tiền, thì hợp đồng sẽ được đóng và tất cả số tiền trong hợp đồng sẽ được hoàn lại, nhưng lúc này không còn tiền trong hợp đồng, vì vậy `investor` sẽ mất số tiền của mình.

Làm thế nào để chúng ta có thể tránh vấn đề về việc người phát hành không thực hiện nghĩa vụ?&#x20;

Có ít nhất hai cách để giải quyết vấn đề này: chúng ta có thể yêu cầu người phát hành gửi toàn bộ số tiền trước khi hợp đồng bắt đầu, nhưng điều này sẽ làm mất mục đích phát hành trái phiếu ngay từ đầu. Một cách thực tế hơn, chúng ta có thể yêu cầu một bên thứ ba làm người bảo lãnh cho thỏa thuận.

> **Exercise**
>
> Give a variant of the `zeroCouponBond` contract in which it is necessary for the `issuer` to put the full value of the up before the bond is issued.

> **Exercise**
>
> Give a variant of the `zeroCouponBond` contract which also includes a `guarantor` who puts up the full payment up before the bond is issued, and who will pay that counterparty if the issuer defaults; if the issuer does make the payment in time, then the guarantor should recover their money.

### Guaranteed Coupon Bond Example​ <a href="#guaranteed-coupon-bond-example" id="guaranteed-coupon-bond-example"></a>

This more complex bond involves an `investor` who deposits 1000 Lovelace, which is immediately paid to the `issuer`. The `issuer` then has to pay as interest 10 Lovelace every month. On maturity the investor should receive back the interest plus the full value of the bond.

```rust
couponBondFor3Month12Percent =
    -- investor deposits 1000 Lovelace
    When [ Case (Deposit "investor" "investor" ada (Constant 1000))
        -- and pays it to the issuer
        (Pay "investor" (Party "issuer") ada (Constant 1000)
            -- after 1 month expect to receive 10 Lovelace interest
            (When [ Case (Deposit "investor" "issuer" ada (Constant 10))
                -- and pay it to the investor
                (Pay "investor" (Party "investor" ) ada (Constant 10)
                    -- same for 2nd month
                    (When [ Case (Deposit "investor" "issuer" ada (Constant 10))
                        (Pay "investor" (Party "investor" ) ada (Constant 10)
                            -- after maturity date investor
                            -- expects to receive notional + interest payment
                            (When [ Case (Deposit "investor" "issuer" ada (Constant 1010))
                                (Pay "investor" (Party "investor" ) ada (Constant 1010) Close)]
                            1680307200 -- 2023-04-01 00:00:00 GMT
                            Close))]
                    1677628800 -- 2023-03-01 00:00:00 GMT
                    Close))]
            1675209600 -- 2023-02-01 00:00:00 GMT
            Close))]
    1672531200 -- 2023-01-01 00:00:00 GMT
    Close
```

> **Exercise**
>
> Give a variant of the `zcouponBondFor3Month12Percent` contract which also includes a `guarantor` who puts up the full payment up before the bond is issued, and who will pay that counterparty if the issuer defaults; if the issuer does make the payment in time, then the guarantor should recover their money.

### ACTUS Contract Terms​ <a href="#actus-contract-terms" id="actus-contract-terms"></a>

In general, an ACTUS contract is specified by the ACTUS contract terms. The technical specification of the ACTUS taxonomy defines all the parameters that are allowed in the ACTUS contract terms in a [dictionary](https://github.com/actusfrf/actus-dictionary/blob/master/actus-dictionary-terms.json). ACTUS contract terms fully describe a financial contract and, therefore, allow projected cashflows to be generated. The projected cashflows are then used as a basis to generate Marlowe contracts corresponding to ACTUS contract terms.

The ACTUS Principle At Maturity (PAM) contract is a loan with periodic interest payments on a fixed schedule and a final payment of the principal.

Example ACTUS contract terms: one installment of interest, followed by the repayment of the principal.

```rust
{
 "contractType": "PAM",
 "contractID": "pam example",
 "statusDate": "2023-12-31T00:00:00",
 "contractDealDate": "2023-12-28T00:00:00",
 "currency": "ADA",
 "notionalPrincipal": "20",
 "initialExchangeDate": "2024-01-01T00:00:00",
 "maturityDate": "2025-01-01T00:00:00",
 "nominalInterestRate": "0.1",
 "cycleAnchorDateOfInterestPayment": "2025-01-01T00:00:00",
 "cycleOfInterestPayment": "P1YL0",
 "dayCountConvention": "30E360",
 "endOfMonthConvention": "SD",
 "premiumDiscountAtIED": "   0",
 "rateMultiplier": "1.0",
 "contractRole": "RPA"
}
```

The example produces the following projected cashflows:

| Contract Id | Event Type | Event Time              | Amount | Currency |
| ----------- | ---------- | ----------------------- | ------ | -------- |
| PAM example | IED        | 2024-01-01 00:00:00:000 | -20.0  | ₳        |
| PAM example | IP         | 2025-01-01 00:00:00:000 | 2.0    | ₳        |
| PAM example | MD         | 2025-01-01 00:00:00:000 | 20.0   | ₳        |

The projected cashflows are translated into the following Marlowe contract:

```rust
When [
  (Case
     (Deposit
        (Role "party")
        (Role "party")
        (Token "" "")
        (Constant 20000000))
     (Pay
        (Role "party")
        (Party
           (Role "counterparty"))
        (Token "" "")
        (Constant 20000000)
        (When [
           (Case
              (Deposit
                 (Role "counterparty")
                 (Role "counterparty")
                 (Token "" "")
                 (Constant 2000000))
              (Pay
                 (Role "counterparty")
                 (Party
                    (Role "party"))
                 (Token "" "")
                 (Constant 2000000)
                 (When [
                    (Case
                       (Deposit
                          (Role "counterparty")
                          (Role "counterparty")
                          (Token "" "")
                          (Constant 20000000))
                       (Pay
                          (Role "counterparty")
                          (Party
                             (Role "party"))
                          (Token "" "")
                          (Constant 20000000) Close))] 1735689600000 Close)))] 1735689600000 Close)))] 1704067200000 Close
```

The ACTUS specification is implemented in a separate [repository](https://github.com/input-output-hk/actus-core).
