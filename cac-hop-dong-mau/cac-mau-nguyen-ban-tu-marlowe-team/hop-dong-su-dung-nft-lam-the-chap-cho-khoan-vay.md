# Hợp đồng sử dụng NFT làm thế chấp cho khoản vay

### <mark style="color:red;">**Cảnh báo**</mark>**!** <a href="#caution" id="caution"></a>

Trước khi chạy một hợp đồng Marlowe trên mainnet, hãy làm theo các bước sau để tránh mất tiền:

1. Hiểu [ngôn ngữ Marlowe](https://marlowe.iohk.io/).
2. Hiểu mô hình [Extended UTxO](https://docs.cardano.org/learn/eutxo-explainer) của Cardano.
3. Đọc và hiểu Hướng dẫn [Thực hành Tốt nhất của Marlowe](https://github.com/input-output-hk/marlowe-cardano/blob/main/marlowe/best-practices.md).
4. Đọc và hiểu [Hướng dẫn Bảo mật](https://github.com/input-output-hk/marlowe-cardano/blob/main/marlowe/security.md) của Marlowe.
5. Sử dụng [Marlowe Playground](https://playground.marlowe-lang.org/#/) để đánh dấu cảnh báo, thực hiện phân tích tĩnh và mô phỏng hợp đồng.
6. Sử dụng công cụ `marlowe-cli run analyze` của [Marlowe CLI](https://github.com/input-output-hk/marlowe-cardano/blob/main/marlowe-cli/ReadMe.md) để kiểm tra xem hợp đồng có thể chạy trên mạng Cardano hay không.
7. Chạy tất cả các đường thực thi của hợp đồng trên [testnet](https://docs.cardano.org/cardano-testnet/overview) Cardano.

### &#x20;<a href="#caution" id="caution"></a>

***

### Hợp đồng sử dụng NFT làm thế chấp cho khoản vay

Bên vay mượn 90 Djed với token BearGarden làm tài sản thế chấp; sau khi họ hoàn trả khoản vay 90 Djed gốc và 10 Djed lãi cho bên cho vay, họ sẽ nhận lại token BearGarden. Nếu không hoàn trả trước thời hạn, bên cho vay sẽ giữ lại token BearGarden.

Ví dụ này bao gồm bảy giao dịch:

1. Elizabeth Cary tạo hợp đồng Marlowe bán token.
2. Elizabeth Cary gửi 1 token BearGarden vào hợp đồng.
3. Mary Herbert gửi 90 Djed vào hợp đồng, khiến hợp đồng cho Elizabeth Cary vay 90 Djed.
4. Elizabeth Cary rút 90 Djed từ địa chỉ thanh toán của vai trò trong Marlowe.
5. Sau đó, Elizabeth Cary gửi 100 Djed vào hợp đồng, hợp đồng trả lại token cho cô và chuyển 100 Djed cho Mary Herbert.
6. Elizabeth Cary rút 1 token BearGarden từ địa chỉ thanh toán của vai trò trong Marlowe.
7. Mary Herbert rút 100 Djed từ địa chỉ thanh toán của vai trò trong Marlowe.

Dưới đây là hợp đồng ở định dạng Blockly:

![Loan collateralize by an NFT](https://raw.githubusercontent.com/input-output-hk/real-world-marlowe/a288a9f391e68135e59e65a813db695e6223472d/nfts/collateral/contract.png)

### Thiết lập <a href="#set-up" id="set-up"></a>

Sử dụng `mainnet`.

In \[1]:

```
. ../../mainnet.env
```

Sử dụng các vai trò ví dụ tiêu chuẩn.

In \[2]:

```
. ../../dramatis-personae/roles.env
```

### Token vai trò&#x20;

Hợp đồng này sử dụng Ada Handles làm token vai trò:

* Elizabeth Cary = [$e.cary](https://pool.pm/asset1tx4euajkdczmkawgkjy342agaq33885dlvp0jl)
* Mary Herbert = [$m.herbert](https://pool.pm/asset1a38nhu84xquj7whe3xqr80uyf99mh2r7hzf277)

_Lưu ý: Chỉ sử dụng token đã được phát hành trước làm token vai trò Marlowe nếu bạn đã xem xét chính sách tiền tệ để tìm kiếm các lỗ hổng bảo mật._

Dưới đây là ký hiệu tiền tệ cho Ada handles trên `mainnet`:

In \[3]:

```
echo "ROLES_CURRENCY = $ROLES_CURRENCY"
```

```
ROLES_CURRENCY = f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
```

### Policy ID cho token BearGarden <a href="#policy-id-for-the-beargarden-token" id="policy-id-for-the-beargarden-token"></a>

Chúng tôi đã trước đó phát hành token BearGarden với chính sách sau đây.&#x20;

In \[4]:

```
echo "FUNGIBLES_POLICY = $FUNGIBLES_POLICY"
```

```
FUNGIBLES_POLICY = 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
```

### Policy ID cho Djed <a href="#policy-ids-for-djed" id="policy-ids-for-djed"></a>

Dưới đây là policy cho các stablecoin mà chúng ta sử dụng.

In \[5]:

```
echo "DJED_POLICY = $DJED_POLICY"
echo "DJED_NAME = $DJED_NAME"
```

```
DJED_POLICY = 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
DJED_NAME = DjedMicroUSD
```

### Gửi ban đầu <a href="#initial-funding" id="initial-funding"></a>

Gửi token BearGarden từ faucet cho Elizabeth Cary và Djed cho cả Elizabeth Cary và Mary Herbert.

In \[6]:

```
ADA=1000000
DJED=1000000
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "68384929c41d65b8583254265b7178edfe6668e68789890a60242fec3c58bf7c#0" \
  --tx-in "a54cdaeaa68371466a6ddd0a8b20c092b9473bf5840e00f9e3dbcbd0c0967e2a#1" \
  --tx-in "b6f5cd7769917917f2de5daeeeb5760d240bd8bf486c147d5a41c95140bef5b6#1" \
  --tx-out "${ROLE_ADDR[e.cary]}+$((3 * ADA))+$((10 * DJED)) $DJED_POLICY.$DJED_NAME" \
  --tx-out "${ROLE_ADDR[m.herbert]}+$((3 * ADA))+$((90 * IUSD)) $DJED_POLICY.$DJED_NAME" \
  --tx-out "${ROLE_ADDR[e.cary]}+$((3 * ADA))+1 $FUNGIBLES_POLICY.BearGarden" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+499 $FUNGIBLES_POLICY.BearGarden" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "$FAUCET_SKEY" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "b7896f5909e59c1408fb36111cb4620e2f3d6400c98332fee80d739246bcaead"
```

In \[7]:

```
ADA=1000000
DJED=1000000
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "b7896f5909e59c1408fb36111cb4620e2f3d6400c98332fee80d739246bcaead#4" \
  --tx-in "4a288753dfa52ae965d5a100ee353d89c19fac936c3b5376dc8ab02946bdd386#0" \
  --tx-out "${ROLE_ADDR[e.cary]}+$((3 * ADA))+1 $FUNGIBLES_POLICY.BearGarden" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+498 $FUNGIBLES_POLICY.BearGarden" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "$FAUCET_SKEY" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "46cccae10f766e788eb7f2f362072a2acc3b9a2138c435d63551323cc8cd158d"
```

### Hợp đồng Marlowe <a href="#the-marlowe-contract" id="the-marlowe-contract"></a>

Hợp đồng Marlowe thực tế chỉ cần download file JSON được biên dịch từ hợp đồng này dạng Blockly đã được thiết kế sẵn trong [Marlowe Playground](https://play.marlowe.iohk.io/#/).

In \[8]:

```
json2yaml contract.json
```

```
timeout: 1676679830000
timeout_continuation: close
when:
- case:
    deposits: 1
    into_account:
      role_token: e.cary
    of_token:
      currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
      token_name: BearGarden
    party:
      role_token: e.cary
  then:
    timeout: 1676679840000
    timeout_continuation: close
    when:
    - case:
        deposits: 90000000
        into_account:
          role_token: m.herbert
        of_token:
          currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
          token_name: DjedMicroUSD
        party:
          role_token: m.herbert
      then:
        from_account:
          role_token: m.herbert
        pay: 90000000
        then:
          timeout: 1676679850000
          timeout_continuation:
            from_account:
              role_token: e.cary
            pay: 1
            then: close
            to:
              party:
                role_token: m.herbert
            token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
          when:
          - case:
              deposits: 100000000
              into_account:
                role_token: m.herbert
              of_token:
                currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
                token_name: DjedMicroUSD
              party:
                role_token: e.cary
            then: close
        to:
          party:
            role_token: e.cary
        token:
          currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
          token_name: DjedMicroUSD
```

### Giao dịch số 1: Tạo hợp đồng <a href="#transaction-1.-create-the-contract" id="transaction-1.-create-the-contract"></a>

Chúng ta sử dụng công cụ _Marlowe Runtime's command-line_ để xây dựng giao dịch tạo hợp đồng.

In \[9]:

```
CONTRACT_ID=$(
marlowe create \
  --core-file contract.json \
  --role-token-policy-id "$ROLES_CURRENCY" \
  --min-utxo "$((3 * ADA))" \
  --change-address "$FAUCET_ADDR" \
  --manual-sign tx-1.unsigned \
| jq -r 'fromjson | .contractId' \
)
echo "CONTRACT_ID = $CONTRACT_ID"
```

```
CONTRACT_ID = 4a288753dfa52ae965d5a100ee353d89c19fac936c3b5376dc8ab02946bdd386#1
```

Hợp đồng có thể được ký và gửi bằng bất kỳ ví hoặc dịch vụ nào. Để thuận tiện, chúng ta sử dụng `marlowe-cli`.

In \[10]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-1.unsigned \
  --required-signer "$FAUCET_SKEY" \
  --timeout 600
```

```
TxId "4a288753dfa52ae965d5a100ee353d89c19fac936c3b5376dc8ab02946bdd386"
```

Xem hợp đồng qua trình khám phá dữ liệu blockchain Cardano (ví dụ: Cardanoscan.io).

In \[11]:

```
echo "https://cardanoscan.io/transaction/${CONTRACT_ID%%#1}?tab=utxo"
```

```
https://cardanoscan.io/transaction/4a288753dfa52ae965d5a100ee353d89c19fac936c3b5376dc8ab02946bdd386?tab=utxo
```

Chúng ta cũng có thể sử dụng công cụ như `marlowe-pipe` để truy xuất hợp đồng từ blockchain và hiển thị nó.

In \[12]:

```
echo '{"request" : "get", "contractId" : "'"$CONTRACT_ID"'"}' | marlowe-pipe 2> /dev/null | json2yaml
```

```
creation:
  output:
    address: 716a9391d6aa51af28dd876ebb5565b69d1e83e5ac7861506bd29b56b0
    assets:
      ada: 3000000
      tokens: []
    datum:
      marloweContract:
        timeout: 1676679830000
        timeout_continuation: close
        when:
        - case:
            deposits: 1
            into_account:
              role_token: e.cary
            of_token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
            party:
              role_token: e.cary
          then:
            timeout: 1676679840000
            timeout_continuation: close
            when:
            - case:
                deposits: 90000000
                into_account:
                  role_token: m.herbert
                of_token:
                  currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
                  token_name: DjedMicroUSD
                party:
                  role_token: m.herbert
              then:
                from_account:
                  role_token: m.herbert
                pay: 90000000
                then:
                  timeout: 1676679850000
                  timeout_continuation:
                    from_account:
                      role_token: e.cary
                    pay: 1
                    then: close
                    to:
                      party:
                        role_token: m.herbert
                    token:
                      currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                      token_name: BearGarden
                  when:
                  - case:
                      deposits: 100000000
                      into_account:
                        role_token: m.herbert
                      of_token:
                        currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
                        token_name: DjedMicroUSD
                      party:
                        role_token: e.cary
                    then: close
                to:
                  party:
                    role_token: e.cary
                token:
                  currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
                  token_name: DjedMicroUSD
      marloweParams:
        rolesCurrency: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
      marloweState:
        accounts:
        - - - address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
            - currency_symbol: ''
              token_name: ''
          - 3000000
        boundValues: []
        choices: []
        minTime: 0
    utxo:
      txId: 4a288753dfa52ae965d5a100ee353d89c19fac936c3b5376dc8ab02946bdd386
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps: []
```

### Transaction 2. Elizabeth Cary nạp token BearGarden vào hợp đồng <a href="#transaction-2.-elizabeth-cary-deposits-the-beargarden-token-into-the-contract" id="transaction-2.-elizabeth-cary-deposits-the-beargarden-token-into-the-contract"></a>

Hợp đồng quy định rằng Elizabeth Cary gửi một token BearGarden vào tài khoản của cô ấy trong hợp đồng Marlowe.

In \[13]:

```
TX_2=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[e.cary]}" \
  --to-party "${ROLE_NAME[e.cary]}" \
  --currency "$FUNGIBLES_POLICY" \
  --token-name BearGarden \
  --quantity 1 \
  --change-address "${ROLE_ADDR[e.cary]}" \
  --manual-sign tx-2.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_2 = $TX_2"
```

```
TX_2 = 1395ae199837f758bfc19f003b9465c137daf998d3937fba92b0d50f88ecdace
```

Ký và gửi giao dịch.

In \[14]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-2.unsigned \
  --required-signer "${ROLE_SKEY[e.cary]}" \
  --timeout 600
```

```
TxId "1395ae199837f758bfc19f003b9465c137daf998d3937fba92b0d50f88ecdace"
```

Kiểm tra xem token đã được chuyển vào hợp đồng.

In \[15]:

```
echo "https://cardanoscan.io/transaction/$TX_2?tab=utxo"
```

```
https://cardanoscan.io/transaction/1395ae199837f758bfc19f003b9465c137daf998d3937fba92b0d50f88ecdace?tab=utxo
```

Kiểm tra output trong hợp đồng và thấy đã có 1 token BearGarden trong đó.

In \[16]:

```
cardano-cli query utxo --mainnet --tx-in "$TX_2#1"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
1395ae199837f758bfc19f003b9465c137daf998d3937fba92b0d50f88ecdace     1        3000000 lovelace + 1 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d.4265617247617264656e + TxOutDatumHash ScriptDataInBabbageEra "fbd1de02388b1aec1565475525bb0ce76048ed50b3ee6031dc7ad24b1879fa0c"
```

### Giao dịch số 3. Mary Herbert gửi 90 Djed vào hợp đồng, khiến hợp đồng thanh toán 90 Djed cho Elizabeth Cary.&#x20;

Việc gửi 90 Djed này kích hoạt hợp đồng thanh toán 90 Djed cho quyền lợi của Elizabeth Cary.

In \[17]:

```
TX_3=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[m.herbert]}" \
  --to-party "${ROLE_NAME[m.herbert]}" \
  --currency "$DJED_POLICY" \
  --token-name "$DJED_NAME" \
  --quantity "$((90 * DJED))" \
  --change-address "${ROLE_ADDR[m.herbert]}" \
  --manual-sign tx-3.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_3 = $TX_3"
```

```
TX_3 = 075b605dc43bc1a3411ddc68da59eae5ba82d02297abe7d630ad62d7fcc710f0
```

Ký và gửi giao dịch.

In \[18]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-3.unsigned \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --timeout 600
```

```
TxId "075b605dc43bc1a3411ddc68da59eae5ba82d02297abe7d630ad62d7fcc710f0"
```

Hợp đồng giữ lại token BearGarden nhưng đã thanh toán 90 Djed vào địa chỉ thanh toán vai trò của Elizabeth Cary.

In \[19]:

```
echo "https://cardanoscan.io/transaction/$TX_3?tab=utxo"
```

```
https://cardanoscan.io/transaction/075b605dc43bc1a3411ddc68da59eae5ba82d02297abe7d630ad62d7fcc710f0?tab=utxo
```

### Giao dịch số 4: Elizabeth Cary rút 90 Djed từ địa chỉ vai trò. <a href="#transaction-4.-elizabeth-cary-withdraws-90-djed-from-the-role-payout-address" id="transaction-4.-elizabeth-cary-withdraws-90-djed-from-the-role-payout-address"></a>

In \[20]:

```
TX_4=$(
marlowe withdraw \
  --contract "$CONTRACT_ID" \
  --role "${ROLE_NAME[e.cary]}" \
  --change-address "${ROLE_ADDR[e.cary]}" \
  --manual-sign tx-4.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_4 = $TX_4"
```

```
TX_4 = 76914d0d4836ff09102034f96195873451ec1df6f88679f9d0a8c9e1e87ce49f
```

Ký và gửi giao dịch.

In \[21]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-4.unsigned \
  --required-signer "${ROLE_SKEY[e.cary]}" \
  --timeout 600
```

```
TxId "76914d0d4836ff09102034f96195873451ec1df6f88679f9d0a8c9e1e87ce49f"
```

Kiểm tra Elizabeth Cary đã rút thành công 90 Djed từ địa chỉ thanh toán vai trò?

In \[22]:

```
echo "https://cardanoscan.io/transaction/$TX_4?tab=utxo"
```

```
https://cardanoscan.io/transaction/76914d0d4836ff09102034f96195873451ec1df6f88679f9d0a8c9e1e87ce49f?tab=utxo
```

### Giao dịch số 5: Elizabeth Cary hoàn trả 100 Djed, khiến token BearGarden của cô được trả lại và Mary Herbert nhận được khoản thanh toán. <a href="#transaction-5.-elizabeth-cary-repays-100-djed-causing-her-token-to-be-returned-and-mary-herbert-to-b" id="transaction-5.-elizabeth-cary-repays-100-djed-causing-her-token-to-be-returned-and-mary-herbert-to-b"></a>

In \[23]:

```
TX_5=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[e.cary]}" \
  --to-party "${ROLE_NAME[m.herbert]}" \
  --currency "$DJED_POLICY" \
  --token-name "$DJED_NAME" \
  --quantity "$((100 * DJED))" \
  --change-address "${ROLE_ADDR[e.cary]}" \
  --manual-sign tx-5.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_5 = $TX_5"
```

```
TX_5 = 63d1d6ae2b7885b163275e26774339a4423321113399615a60c805aa1885cafa
```

Ký và gửi giao dịch.

In \[24]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-5.unsigned \
  --required-signer "${ROLE_SKEY[e.cary]}" \
  --timeout 600
```

```
TxId "63d1d6ae2b7885b163275e26774339a4423321113399615a60c805aa1885cafa"
```

Hợp đồng đã đóng sau khi nhận khoản hoàn trả, đồng thời thanh toán token BearGarden và Djed vào các địa chỉ thanh toán vai trò.

In \[25]:

```
echo "https://cardanoscan.io/transaction/$TX_5?tab=utxo"
```

```
https://cardanoscan.io/transaction/63d1d6ae2b7885b163275e26774339a4423321113399615a60c805aa1885cafa?tab=utxo
```

### Giao dịch 6. Elizabeth Cary rút 1 token BearGarden từ địa chỉ thanh toán vai trò.

In \[26]:

```
TX_6=$(
marlowe withdraw \
  --contract "$CONTRACT_ID" \
  --role "${ROLE_NAME[e.cary]}" \
  --change-address "${ROLE_ADDR[e.cary]}" \
  --manual-sign tx-6.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_6 = $TX_6"
```

```
TX_6 = 4d3c27b72174c53f66a576903b1ebdd0b8fe73a38c1ba77c66bdc267a6b544f8
```

Ký và gửi giao dịch.

In \[27]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-6.unsigned \
  --required-signer "${ROLE_SKEY[e.cary]}" \
  --timeout 600
```

```
TxId "4d3c27b72174c53f66a576903b1ebdd0b8fe73a38c1ba77c66bdc267a6b544f8"
```

Kiểm tra xem địa chỉ đã nhận được token?

In \[28]:

```
echo "https://cardanoscan.io/transaction/$TX_6?tab=utxo"
```

```
https://cardanoscan.io/transaction/4d3c27b72174c53f66a576903b1ebdd0b8fe73a38c1ba77c66bdc267a6b544f8?tab=utxo
```

### Giao dịch số 7: Mary Herbert rút 100 Djed từ địa chỉ vai trò <a href="#transaction-7.-mary-herbert-withdraws-her-100-djed-from-the-role-payout-address" id="transaction-7.-mary-herbert-withdraws-her-100-djed-from-the-role-payout-address"></a>

In \[29]:

```
TX_7=$(
marlowe withdraw \
  --contract "$CONTRACT_ID" \
  --role "${ROLE_NAME[m.herbert]}" \
  --change-address "${ROLE_ADDR[m.herbert]}" \
  --manual-sign tx-7.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_7 = $TX_7"
```

```
TX_7 = 0386d51ae39a4faab998329532cdfde488210dee66eaa01f30aea62c68cfb91a
```

Ký và gửi giao dịch.

In \[30]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-7.unsigned \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --timeout 600
```

```
TxId "0386d51ae39a4faab998329532cdfde488210dee66eaa01f30aea62c68cfb91a"
```

Kiểm tra xem đã nhận được Djed?

In \[31]:

```
echo "https://cardanoscan.io/transaction/$TX_7?tab=utxo"
```

```
https://cardanoscan.io/transaction/0386d51ae39a4faab998329532cdfde488210dee66eaa01f30aea62c68cfb91a?tab=utxo
```

### Xem lại toàn bộ lịch sử hợp đồng <a href="#view-the-whole-history-of-the-contract" id="view-the-whole-history-of-the-contract"></a>

Chúng ta sử dụng `marlowe-pipe` để in ra toàn bộ lịch sử của hợp đồng.

In \[32]:

```
echo '{"request" : "get", "contractId" : "'"$CONTRACT_ID"'"}' | marlowe-pipe 2> /dev/null | json2yaml
```

```
creation:
  output:
    address: 716a9391d6aa51af28dd876ebb5565b69d1e83e5ac7861506bd29b56b0
    assets:
      ada: 3000000
      tokens: []
    datum:
      marloweContract:
        timeout: 1676679830000
        timeout_continuation: close
        when:
        - case:
            deposits: 1
            into_account:
              role_token: e.cary
            of_token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
            party:
              role_token: e.cary
          then:
            timeout: 1676679840000
            timeout_continuation: close
            when:
            - case:
                deposits: 90000000
                into_account:
                  role_token: m.herbert
                of_token:
                  currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
                  token_name: DjedMicroUSD
                party:
                  role_token: m.herbert
              then:
                from_account:
                  role_token: m.herbert
                pay: 90000000
                then:
                  timeout: 1676679850000
                  timeout_continuation:
                    from_account:
                      role_token: e.cary
                    pay: 1
                    then: close
                    to:
                      party:
                        role_token: m.herbert
                    token:
                      currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                      token_name: BearGarden
                  when:
                  - case:
                      deposits: 100000000
                      into_account:
                        role_token: m.herbert
                      of_token:
                        currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
                        token_name: DjedMicroUSD
                      party:
                        role_token: e.cary
                    then: close
                to:
                  party:
                    role_token: e.cary
                token:
                  currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
                  token_name: DjedMicroUSD
      marloweParams:
        rolesCurrency: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
      marloweState:
        accounts:
        - - - address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
            - currency_symbol: ''
              token_name: ''
          - 3000000
        boundValues: []
        choices: []
        minTime: 0
    utxo:
      txId: 4a288753dfa52ae965d5a100ee353d89c19fac936c3b5376dc8ab02946bdd386
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps:
- contractId: 4a288753dfa52ae965d5a100ee353d89c19fac936c3b5376dc8ab02946bdd386#1
  payouts: []
  redeemer:
  - input_from_party:
      role_token: e.cary
    into_account:
      role_token: e.cary
    of_token:
      currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
      token_name: BearGarden
    that_deposits: 1
  scriptOutput:
    address: 716a9391d6aa51af28dd876ebb5565b69d1e83e5ac7861506bd29b56b0
    assets:
      ada: 3000000
      tokens:
      - - policyId: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
          tokenName: BearGarden
        - 1
    datum:
      marloweContract:
        timeout: 1676679840000
        timeout_continuation: close
        when:
        - case:
            deposits: 90000000
            into_account:
              role_token: m.herbert
            of_token:
              currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
              token_name: DjedMicroUSD
            party:
              role_token: m.herbert
          then:
            from_account:
              role_token: m.herbert
            pay: 90000000
            then:
              timeout: 1676679850000
              timeout_continuation:
                from_account:
                  role_token: e.cary
                pay: 1
                then: close
                to:
                  party:
                    role_token: m.herbert
                token:
                  currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                  token_name: BearGarden
              when:
              - case:
                  deposits: 100000000
                  into_account:
                    role_token: m.herbert
                  of_token:
                    currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
                    token_name: DjedMicroUSD
                  party:
                    role_token: e.cary
                then: close
            to:
              party:
                role_token: e.cary
            token:
              currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
              token_name: DjedMicroUSD
      marloweParams:
        rolesCurrency: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
      marloweState:
        accounts:
        - - - address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
            - currency_symbol: ''
              token_name: ''
          - 3000000
        - - - role_token: e.cary
            - currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
          - 1
        boundValues: []
        choices: []
        minTime: 1676342104000
    utxo:
      txId: 1395ae199837f758bfc19f003b9465c137daf998d3937fba92b0d50f88ecdace
      txIx: 1
  step: apply
  txId: 1395ae199837f758bfc19f003b9465c137daf998d3937fba92b0d50f88ecdace
- contractId: 4a288753dfa52ae965d5a100ee353d89c19fac936c3b5376dc8ab02946bdd386#1
  payouts:
  - - txId: 075b605dc43bc1a3411ddc68da59eae5ba82d02297abe7d630ad62d7fcc710f0
      txIx: 3
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 1236970
        tokens:
        - - policyId: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
            tokenName: DjedMicroUSD
          - 90000000
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: e.cary
  redeemer:
  - input_from_party:
      role_token: m.herbert
    into_account:
      role_token: m.herbert
    of_token:
      currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
      token_name: DjedMicroUSD
    that_deposits: 90000000
  scriptOutput:
    address: 716a9391d6aa51af28dd876ebb5565b69d1e83e5ac7861506bd29b56b0
    assets:
      ada: 3000000
      tokens:
      - - policyId: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
          tokenName: BearGarden
        - 1
    datum:
      marloweContract:
        timeout: 1676679850000
        timeout_continuation:
          from_account:
            role_token: e.cary
          pay: 1
          then: close
          to:
            party:
              role_token: m.herbert
          token:
            currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
            token_name: BearGarden
        when:
        - case:
            deposits: 100000000
            into_account:
              role_token: m.herbert
            of_token:
              currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
              token_name: DjedMicroUSD
            party:
              role_token: e.cary
          then: close
      marloweParams:
        rolesCurrency: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
      marloweState:
        accounts:
        - - - address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
            - currency_symbol: ''
              token_name: ''
          - 3000000
        - - - role_token: e.cary
            - currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
          - 1
        boundValues: []
        choices: []
        minTime: 1676342149000
    utxo:
      txId: 075b605dc43bc1a3411ddc68da59eae5ba82d02297abe7d630ad62d7fcc710f0
      txIx: 1
  step: apply
  txId: 075b605dc43bc1a3411ddc68da59eae5ba82d02297abe7d630ad62d7fcc710f0
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: e.cary
  redeemingTx: 76914d0d4836ff09102034f96195873451ec1df6f88679f9d0a8c9e1e87ce49f
  step: payout
  utxo:
    txId: 075b605dc43bc1a3411ddc68da59eae5ba82d02297abe7d630ad62d7fcc710f0
    txIx: 3
- contractId: 4a288753dfa52ae965d5a100ee353d89c19fac936c3b5376dc8ab02946bdd386#1
  payouts:
  - - txId: 63d1d6ae2b7885b163275e26774339a4423321113399615a60c805aa1885cafa
      txIx: 2
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 1211110
        tokens:
        - - policyId: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
            tokenName: BearGarden
          - 1
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: e.cary
  - - txId: 63d1d6ae2b7885b163275e26774339a4423321113399615a60c805aa1885cafa
      txIx: 3
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 1236970
        tokens:
        - - policyId: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
            tokenName: DjedMicroUSD
          - 100000000
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: m.herbert
  redeemer:
  - input_from_party:
      role_token: e.cary
    into_account:
      role_token: m.herbert
    of_token:
      currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
      token_name: DjedMicroUSD
    that_deposits: 100000000
  scriptOutput: null
  step: apply
  txId: 63d1d6ae2b7885b163275e26774339a4423321113399615a60c805aa1885cafa
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: e.cary
  redeemingTx: 4d3c27b72174c53f66a576903b1ebdd0b8fe73a38c1ba77c66bdc267a6b544f8
  step: payout
  utxo:
    txId: 63d1d6ae2b7885b163275e26774339a4423321113399615a60c805aa1885cafa
    txIx: 2
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: m.herbert
  redeemingTx: 0386d51ae39a4faab998329532cdfde488210dee66eaa01f30aea62c68cfb91a
  step: payout
  utxo:
    txId: 63d1d6ae2b7885b163275e26774339a4423321113399615a60c805aa1885cafa
    txIx: 3
```

### Trả lại token BearGarden và Djed về faucet <a href="#return-the-beargarden-tokens-to-the-faucet" id="return-the-beargarden-tokens-to-the-faucet"></a>

Trả lại token cho faucet là một cách quản lý tiện lợi cho ví dụ này.

In \[33]:

```
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "0386d51ae39a4faab998329532cdfde488210dee66eaa01f30aea62c68cfb91a#2" \
  --tx-in "4d3c27b72174c53f66a576903b1ebdd0b8fe73a38c1ba77c66bdc267a6b544f8#2" \
  --tx-in "6e41846908175fbbea9a5c808901c5e1a1b790c1f899faa1c5c6bb887b5456e2#4" \
  --tx-in "63d1d6ae2b7885b163275e26774339a4423321113399615a60c805aa1885cafa#4" \
  --tx-in "b6f5cd7769917917f2de5daeeeb5760d240bd8bf486c147d5a41c95140bef5b6#0" \
  --tx-in "46cccae10f766e788eb7f2f362072a2acc3b9a2138c435d63551323cc8cd158d#2" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+$((100 * DJED)) $DJED_POLICY.$DJED_NAME" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+499 $FUNGIBLES_POLICY.BearGarden" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "$FAUCET_SKEY" \
  --required-signer "${ROLE_SKEY[e.cary]}" \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "0afae60d6282c0f4d2f91eb3046b2e7c22489b70c97e4c18b92c0968403be754"
```
