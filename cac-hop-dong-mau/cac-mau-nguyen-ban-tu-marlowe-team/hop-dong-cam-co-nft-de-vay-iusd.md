# Hợp đồng Cầm cố NFT để vay iUSD

### <mark style="color:red;">**Cảnh báo**</mark>**!** <a href="#caution" id="caution"></a>

Trước khi chạy một hợp đồng Marlowe trên mainnet, hãy làm theo các bước sau để tránh mất tiền:

1. Hiểu [ngôn ngữ Marlowe](https://marlowe.iohk.io/).
2. Hiểu mô hình [Extended UTxO](https://docs.cardano.org/learn/eutxo-explainer) của Cardano.
3. Đọc và hiểu Hướng dẫn [Thực hành Tốt nhất của Marlowe](https://github.com/input-output-hk/marlowe-cardano/blob/main/marlowe/best-practices.md).
4. Đọc và hiểu [Hướng dẫn Bảo mật](https://github.com/input-output-hk/marlowe-cardano/blob/main/marlowe/security.md) của Marlowe.
5. Sử dụng [Marlowe Playground](https://playground.marlowe-lang.org/#/) để đánh dấu cảnh báo, thực hiện phân tích tĩnh và mô phỏng hợp đồng.
6. Sử dụng công cụ `marlowe-cli run analyze` của [Marlowe CLI](https://github.com/input-output-hk/marlowe-cardano/blob/main/marlowe-cli/ReadMe.md) để kiểm tra xem hợp đồng có thể chạy trên mạng Cardano hay không.
7. Chạy tất cả các đường thực thi của hợp đồng trên [testnet](https://docs.cardano.org/cardano-testnet/overview) Cardano.

***

### Cầm cố NFT để vay iUSD

Dưới đây là ví dụ về việc cầm cố token BearGarden để vay 100 iUSD:

Bên vay nhận 100 iUSD cho việc cầm cố token BearGarden làm tài sản thế chấp; nếu họ hoàn trả 100 iUSD, họ sẽ nhận lại token BearGarden. Tuy nhiên, bất kỳ lúc nào trước khi thanh toán, bên cho vay có thể quyết định giữ lại token BearGarden.

Ví dụ này bao gồm sáu giao dịch:

1. Elizabeth Cary tạo hợp đồng Marlowe bán token.
2. Elizabeth Cary gửi 1 token BearGarden vào hợp đồng.
3. Mary Herbert gửi iUSD vào hợp đồng, khiến hợp đồng thanh toán 100 iUSD vào địa chỉ thanh toán vai trò cho Elizabeth Cary.
4. Elizabeth Cary rút 100 iUSD từ địa chỉ thanh toán của vai trò trong Marlowe.
5. Sau đó, Mary Herbert chọn rút token từ hợp đồng, khiến Elizabeth Cary không có cơ hội hoàn trả 100 iUSD và nhận lại token của mình.
6. Mary Herbert rút 1 token BearGarden từ địa chỉ thanh toán của vai trò trong Marlowe.

Dưới đây là hợp đồng ở định dạng Blockly:

![Pawning an NFT](https://raw.githubusercontent.com/input-output-hk/real-world-marlowe/a288a9f391e68135e59e65a813db695e6223472d/nfts/pawn/contract.png)

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

### Policy ID cho iUSD <a href="#policy-ids-for-iusd" id="policy-ids-for-iusd"></a>

Dưới đây là policy id cho các stablecoin mà chúng ta sử dụng.

In \[5]:

```
echo "IUSD_POLICY = $IUSD_POLICY"
echo "IUSD_NAME = $IUSD_NAME"
```

```
IUSD_POLICY = f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
IUSD_NAME = iUSD
```

### Gửi ban đầu <a href="#initial-funding" id="initial-funding"></a>

Gửi token BearGarden fungible từ faucet cho Elizabeth Cary và iUSD cho Mary Herbert.

In \[6]:

```
ADA=1000000
IUSD=1000000
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "0afae60d6282c0f4d2f91eb3046b2e7c22489b70c97e4c18b92c0968403be754#0" \
  --tx-in "0afae60d6282c0f4d2f91eb3046b2e7c22489b70c97e4c18b92c0968403be754#2" \
  --tx-in "357298faa4c200ce3f1696671c0455be01866bdbcbfc80a3893d62e394b4a9c2#1" \
  --tx-in "46cccae10f766e788eb7f2f362072a2acc3b9a2138c435d63551323cc8cd158d#0" \
  --tx-out "${ROLE_ADDR[m.herbert]}+$((3 * ADA))+$((100 * IUSD)) $IUSD_POLICY.$IUSD_NAME" \
  --tx-out "${ROLE_ADDR[e.cary]}+$((3 * ADA))+1 $FUNGIBLES_POLICY.BearGarden" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+498 $FUNGIBLES_POLICY.BearGarden" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "$FAUCET_SKEY" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "ab37dafea127cc78131781447874130272ca9e93094ea3dbc9a001c1a1d42c1f"
```

### Hợp đồng Marlowe <a href="#the-marlowe-contract" id="the-marlowe-contract"></a>

Hợp đồng Marlowe thực tế chỉ cần download file JSON được biên dịch từ hợp đồng này dạng Blockly đã được thiết kế sẵn trong [Marlowe Playground](https://play.marlowe.iohk.io/#/).

In \[7]:

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
        deposits: 100000000
        into_account:
          role_token: m.herbert
        of_token:
          currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
          token_name: iUSD
        party:
          role_token: m.herbert
      then:
        from_account:
          role_token: m.herbert
        pay: 100000000
        then:
          from_account:
            role_token: e.cary
          pay: 1
          then:
            timeout: 1676679850000
            timeout_continuation: close
            when:
            - case:
                deposits: 100000000
                into_account:
                  role_token: m.herbert
                of_token:
                  currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                  token_name: iUSD
                party:
                  role_token: e.cary
              then:
                from_account:
                  role_token: m.herbert
                pay: 1
                then: close
                to:
                  party:
                    role_token: e.cary
                token:
                  currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                  token_name: BearGarden
            - case:
                choose_between:
                - from: 0
                  to: 0
                for_choice:
                  choice_name: Withdraw Token
                  choice_owner:
                    role_token: m.herbert
              then: close
          to:
            account:
              role_token: m.herbert
          token:
            currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
            token_name: BearGarden
        to:
          party:
            role_token: e.cary
        token:
          currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
          token_name: iUSD
```

### Giao dịch số 1: Tạo hợp đồng <a href="#transaction-1.-create-the-contract" id="transaction-1.-create-the-contract"></a>

Chúng ta sử dụng công cụ _Marlowe Runtime's command-line_ để xây dựng giao dịch tạo hợp đồng.

In \[8]:

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
CONTRACT_ID = 9f56ccd99e5ee8f4bc92fb229ffaaafa7bd5f0bbf72dd3d7071b0b4cc11df7cd#1
```

Hợp đồng có thể được ký và gửi bằng bất kỳ ví hoặc dịch vụ nào. Để thuận tiện, chúng ta sử dụng `marlowe-cli`.

In \[9]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-1.unsigned \
  --required-signer "$FAUCET_SKEY" \
  --timeout 600
```

```
TxId "9f56ccd99e5ee8f4bc92fb229ffaaafa7bd5f0bbf72dd3d7071b0b4cc11df7cd"
```

Xem hợp đồng qua trình khám phá dữ liệu blockchain Cardano (ví dụ: Cardanoscan.io).

In \[10]:

```
echo "https://cardanoscan.io/transaction/${CONTRACT_ID%%#1}?tab=utxo"
```

```
https://cardanoscan.io/transaction/9f56ccd99e5ee8f4bc92fb229ffaaafa7bd5f0bbf72dd3d7071b0b4cc11df7cd?tab=utxo
```

Chúng ta cũng có thể sử dụng công cụ như `marlowe-pipe` để truy xuất hợp đồng từ blockchain và hiển thị nó.

In \[11]:

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
                deposits: 100000000
                into_account:
                  role_token: m.herbert
                of_token:
                  currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                  token_name: iUSD
                party:
                  role_token: m.herbert
              then:
                from_account:
                  role_token: m.herbert
                pay: 100000000
                then:
                  from_account:
                    role_token: e.cary
                  pay: 1
                  then:
                    timeout: 1676679850000
                    timeout_continuation: close
                    when:
                    - case:
                        deposits: 100000000
                        into_account:
                          role_token: m.herbert
                        of_token:
                          currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                          token_name: iUSD
                        party:
                          role_token: e.cary
                      then:
                        from_account:
                          role_token: m.herbert
                        pay: 1
                        then: close
                        to:
                          party:
                            role_token: e.cary
                        token:
                          currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                          token_name: BearGarden
                    - case:
                        choose_between:
                        - from: 0
                          to: 0
                        for_choice:
                          choice_name: Withdraw Token
                          choice_owner:
                            role_token: m.herbert
                      then: close
                  to:
                    account:
                      role_token: m.herbert
                  token:
                    currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                    token_name: BearGarden
                to:
                  party:
                    role_token: e.cary
                token:
                  currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                  token_name: iUSD
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
      txId: 9f56ccd99e5ee8f4bc92fb229ffaaafa7bd5f0bbf72dd3d7071b0b4cc11df7cd
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps: []
```

### Giao dịch số 2: Elizabeth Cary nạp token BearGarden vào trong hợp đồng <a href="#transaction-2.-elizabeth-cary-deposits-the-beargarden-token-into-the-contract" id="transaction-2.-elizabeth-cary-deposits-the-beargarden-token-into-the-contract"></a>

Hợp đồng quy định rằng Elizabeth Cary gửi một token BearGarden vào tài khoản của cô ấy trong hợp đồng Marlowe.

In \[12]:

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
TX_2 = 36b7532ccc7ab0f802e2ba9986606ca1c5aa4fc69998e8c7c3fe64b23bce896f
```

Ký và gửi giao dịch.

In \[13]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-2.unsigned \
  --required-signer "${ROLE_SKEY[e.cary]}" \
  --timeout 600
```

```
TxId "36b7532ccc7ab0f802e2ba9986606ca1c5aa4fc69998e8c7c3fe64b23bce896f"
```

Kiểm tra xem hợp đồng đã nhận được token?

In \[14]:

```
echo "https://cardanoscan.io/transaction/$TX_2?tab=utxo"
```

```
https://cardanoscan.io/transaction/36b7532ccc7ab0f802e2ba9986606ca1c5aa4fc69998e8c7c3fe64b23bce896f?tab=utxo
```

Kiểm tra output trong hợp đồng và thấy đã có 1 token BearGarden trong đó.

In \[15]:

```
cardano-cli query utxo --mainnet --tx-in "$TX_2#1"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
36b7532ccc7ab0f802e2ba9986606ca1c5aa4fc69998e8c7c3fe64b23bce896f     1        3000000 lovelace + 1 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d.4265617247617264656e + TxOutDatumHash ScriptDataInBabbageEra "15bf522d1bc4ad3ba63c93d193a736905a3db4bf6cd5ef8849a82874cbe7d6f2"
```

### Transaction 3. Mary Herbert nạp 100 iUSD vào hợp đồng, dẫn tới việc thanh toán cho Elizabeth Cary. <a href="#transaction-3.-mary-herbert-deposits-100-iusd-into-the-contract-causing-it-to-pay-the-elizabeth-cary" id="transaction-3.-mary-herbert-deposits-100-iusd-into-the-contract-causing-it-to-pay-the-elizabeth-cary"></a>

Việc gửi 100 Djed kích hoạt hợp đồng thanh toán 100 iUSD cho quyền lợi của Elizabeth Cary.

In \[16]:

```
TX_3=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[m.herbert]}" \
  --to-party "${ROLE_NAME[m.herbert]}" \
  --currency "$IUSD_POLICY" \
  --token-name "$IUSD_NAME" \
  --quantity "$((100 * IUSD))" \
  --change-address "${ROLE_ADDR[m.herbert]}" \
  --manual-sign tx-3.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_3 = $TX_3"
```

```
TX_3 = 542117e8a7087442910bb95d0734313decd368d8fb6f968b86d2ff26de5e291d
```

Ký và gửi giao dịch.

In \[17]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-3.unsigned \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --timeout 600
```

```
TxId "542117e8a7087442910bb95d0734313decd368d8fb6f968b86d2ff26de5e291d"
```

kiểm tra việc thanh toán cho địa chỉ thanh toán vai trò.

In \[18]:

```
echo "https://cardanoscan.io/transaction/$TX_3?tab=utxo"
```

```
https://cardanoscan.io/transaction/542117e8a7087442910bb95d0734313decd368d8fb6f968b86d2ff26de5e291d?tab=utxo
```

### Transaction 4. Elizabeth Cary rút 100 iUSD từ địa chỉ thanh toán vai trò <a href="#transaction-4.-elizabeth-cary-withdraws-100-iusd-from-the-role-payout-address" id="transaction-4.-elizabeth-cary-withdraws-100-iusd-from-the-role-payout-address"></a>

In \[19]:

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
TX_4 = c69a94bd086828e7cbbec84626b7218695201a5dc2adb5717447051776734584
```

Ký và gửi giao dịch.

In \[20]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-4.unsigned \
  --required-signer "${ROLE_SKEY[e.cary]}" \
  --timeout 600
```

```
TxId "c69a94bd086828e7cbbec84626b7218695201a5dc2adb5717447051776734584"
```

Kiểm tra xem Elizabeth Cary đã rút thành công 100 Djed từ địa chỉ thanh toán vai trò.

In \[21]:

```
echo "https://cardanoscan.io/transaction/$TX_4?tab=utxo"
```

```
https://cardanoscan.io/transaction/c69a94bd086828e7cbbec84626b7218695201a5dc2adb5717447051776734584?tab=utxo
```

### Giao dịch 5. Mary Herbert _chọn_ rút token từ hợp đồng.&#x20;

Hành động này khiến Elizabeth Cary không có cơ hội hoàn trả 100 iUSD và nhận lại token của mình.

In \[22]:

```
TX_5=$(
marlowe choose \
  --contract "$CONTRACT_ID" \
  --choice "Withdraw Token" \
  --value 0 \
  --party "${ROLE_NAME[m.herbert]}" \
  --change-address "${ROLE_ADDR[m.herbert]}" \
  --manual-sign tx-5.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_5 = $TX_5"
```

```
TX_5 = e2cda7eb4f16e1d1bacfc76e2e703383fb42a8179b03f5dc98664c7e32f0c1e4
```

Ký và gửi giao dịch.

In \[23]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-5.unsigned \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --timeout 600
```

```
TxId "e2cda7eb4f16e1d1bacfc76e2e703383fb42a8179b03f5dc98664c7e32f0c1e4"
```

Kiểm tra xem hợp đồng đã đóng và thanh toán token BearGarden cùng với iUSD vào địa chỉ thanh toán vai trò.

In \[24]:

```
echo "https://cardanoscan.io/transaction/$TX_5?tab=utxo"
```

```
https://cardanoscan.io/transaction/e2cda7eb4f16e1d1bacfc76e2e703383fb42a8179b03f5dc98664c7e32f0c1e4?tab=utxo
```

### Transaction 6. Mary Herbert rút token BearGarden từ địa chỉ thanh toán vai trò <a href="#transaction-6.-mary-herbert-withdraws-her-1-beargarden-from-the-role-payout-address" id="transaction-6.-mary-herbert-withdraws-her-1-beargarden-from-the-role-payout-address"></a>

In \[25]:

```
TX_6=$(
marlowe withdraw \
  --contract "$CONTRACT_ID" \
  --role "${ROLE_NAME[m.herbert]}" \
  --change-address "${ROLE_ADDR[m.herbert]}" \
  --manual-sign tx-6.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_6 = $TX_6"
```

```
TX_6 = bcac57e8121e98442dff7b20d860a1a139677609a2b4e0d8aa2b7eafffaf73b3
```

Ký và gửi giao dịch.

In \[26]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-6.unsigned \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --timeout 600
```

```
TxId "bcac57e8121e98442dff7b20d860a1a139677609a2b4e0d8aa2b7eafffaf73b3"
```

Kiểm tra xem iUSD đã được rút thành công.

In \[27]:

```
echo "https://cardanoscan.io/transaction/$TX_6?tab=utxo"
```

```
https://cardanoscan.io/transaction/bcac57e8121e98442dff7b20d860a1a139677609a2b4e0d8aa2b7eafffaf73b3?tab=utxo
```

### Xem lại toàn bộ lịch sử của hợp đồng <a href="#view-the-whole-history-of-the-contract" id="view-the-whole-history-of-the-contract"></a>

Chúng ta sử dụng `marlowe-pipe` để in ra toàn bộ lịch sử của hợp đồng.

In \[28]:

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
                deposits: 100000000
                into_account:
                  role_token: m.herbert
                of_token:
                  currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                  token_name: iUSD
                party:
                  role_token: m.herbert
              then:
                from_account:
                  role_token: m.herbert
                pay: 100000000
                then:
                  from_account:
                    role_token: e.cary
                  pay: 1
                  then:
                    timeout: 1676679850000
                    timeout_continuation: close
                    when:
                    - case:
                        deposits: 100000000
                        into_account:
                          role_token: m.herbert
                        of_token:
                          currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                          token_name: iUSD
                        party:
                          role_token: e.cary
                      then:
                        from_account:
                          role_token: m.herbert
                        pay: 1
                        then: close
                        to:
                          party:
                            role_token: e.cary
                        token:
                          currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                          token_name: BearGarden
                    - case:
                        choose_between:
                        - from: 0
                          to: 0
                        for_choice:
                          choice_name: Withdraw Token
                          choice_owner:
                            role_token: m.herbert
                      then: close
                  to:
                    account:
                      role_token: m.herbert
                  token:
                    currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                    token_name: BearGarden
                to:
                  party:
                    role_token: e.cary
                token:
                  currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                  token_name: iUSD
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
      txId: 9f56ccd99e5ee8f4bc92fb229ffaaafa7bd5f0bbf72dd3d7071b0b4cc11df7cd
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps:
- contractId: 9f56ccd99e5ee8f4bc92fb229ffaaafa7bd5f0bbf72dd3d7071b0b4cc11df7cd#1
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
            deposits: 100000000
            into_account:
              role_token: m.herbert
            of_token:
              currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
              token_name: iUSD
            party:
              role_token: m.herbert
          then:
            from_account:
              role_token: m.herbert
            pay: 100000000
            then:
              from_account:
                role_token: e.cary
              pay: 1
              then:
                timeout: 1676679850000
                timeout_continuation: close
                when:
                - case:
                    deposits: 100000000
                    into_account:
                      role_token: m.herbert
                    of_token:
                      currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                      token_name: iUSD
                    party:
                      role_token: e.cary
                  then:
                    from_account:
                      role_token: m.herbert
                    pay: 1
                    then: close
                    to:
                      party:
                        role_token: e.cary
                    token:
                      currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                      token_name: BearGarden
                - case:
                    choose_between:
                    - from: 0
                      to: 0
                    for_choice:
                      choice_name: Withdraw Token
                      choice_owner:
                        role_token: m.herbert
                  then: close
              to:
                account:
                  role_token: m.herbert
              token:
                currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                token_name: BearGarden
            to:
              party:
                role_token: e.cary
            token:
              currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
              token_name: iUSD
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
        minTime: 1676387561000
    utxo:
      txId: 36b7532ccc7ab0f802e2ba9986606ca1c5aa4fc69998e8c7c3fe64b23bce896f
      txIx: 1
  step: apply
  txId: 36b7532ccc7ab0f802e2ba9986606ca1c5aa4fc69998e8c7c3fe64b23bce896f
- contractId: 9f56ccd99e5ee8f4bc92fb229ffaaafa7bd5f0bbf72dd3d7071b0b4cc11df7cd#1
  payouts:
  - - txId: 542117e8a7087442910bb95d0734313decd368d8fb6f968b86d2ff26de5e291d
      txIx: 3
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 1202490
        tokens:
        - - policyId: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
            tokenName: iUSD
          - 100000000
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: e.cary
  redeemer:
  - input_from_party:
      role_token: m.herbert
    into_account:
      role_token: m.herbert
    of_token:
      currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
      token_name: iUSD
    that_deposits: 100000000
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
        timeout_continuation: close
        when:
        - case:
            deposits: 100000000
            into_account:
              role_token: m.herbert
            of_token:
              currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
              token_name: iUSD
            party:
              role_token: e.cary
          then:
            from_account:
              role_token: m.herbert
            pay: 1
            then: close
            to:
              party:
                role_token: e.cary
            token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
        - case:
            choose_between:
            - from: 0
              to: 0
            for_choice:
              choice_name: Withdraw Token
              choice_owner:
                role_token: m.herbert
          then: close
      marloweParams:
        rolesCurrency: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
      marloweState:
        accounts:
        - - - address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
            - currency_symbol: ''
              token_name: ''
          - 3000000
        - - - role_token: m.herbert
            - currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
          - 1
        boundValues: []
        choices: []
        minTime: 1676387638000
    utxo:
      txId: 542117e8a7087442910bb95d0734313decd368d8fb6f968b86d2ff26de5e291d
      txIx: 1
  step: apply
  txId: 542117e8a7087442910bb95d0734313decd368d8fb6f968b86d2ff26de5e291d
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: e.cary
  redeemingTx: c69a94bd086828e7cbbec84626b7218695201a5dc2adb5717447051776734584
  step: payout
  utxo:
    txId: 542117e8a7087442910bb95d0734313decd368d8fb6f968b86d2ff26de5e291d
    txIx: 3
- contractId: 9f56ccd99e5ee8f4bc92fb229ffaaafa7bd5f0bbf72dd3d7071b0b4cc11df7cd#1
  payouts:
  - - txId: e2cda7eb4f16e1d1bacfc76e2e703383fb42a8179b03f5dc98664c7e32f0c1e4
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
        tokenName: m.herbert
  redeemer:
  - for_choice_id:
      choice_name: Withdraw Token
      choice_owner:
        role_token: m.herbert
    input_that_chooses_num: 0
  scriptOutput: null
  step: apply
  txId: e2cda7eb4f16e1d1bacfc76e2e703383fb42a8179b03f5dc98664c7e32f0c1e4
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: m.herbert
  redeemingTx: bcac57e8121e98442dff7b20d860a1a139677609a2b4e0d8aa2b7eafffaf73b3
  step: payout
  utxo:
    txId: e2cda7eb4f16e1d1bacfc76e2e703383fb42a8179b03f5dc98664c7e32f0c1e4
    txIx: 2
```

### Trả lại token BearGarden về faucet <a href="#return-the-beargarden-tokens-to-the-faucet" id="return-the-beargarden-tokens-to-the-faucet"></a>

Trả lại token cho faucet là một cách quản lý tiện lợi cho ví dụ này.

In \[29]:

```
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "bcac57e8121e98442dff7b20d860a1a139677609a2b4e0d8aa2b7eafffaf73b3#2" \
  --tx-in "c69a94bd086828e7cbbec84626b7218695201a5dc2adb5717447051776734584#2" \
  --tx-in "ab37dafea127cc78131781447874130272ca9e93094ea3dbc9a001c1a1d42c1f#3" \
  --tx-in "e2cda7eb4f16e1d1bacfc76e2e703383fb42a8179b03f5dc98664c7e32f0c1e4#3" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+$((100 * IUSD)) $IUSD_POLICY.$IUSD_NAME" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+499 $FUNGIBLES_POLICY.BearGarden" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "$FAUCET_SKEY" \
  --required-signer "${ROLE_SKEY[e.cary]}" \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "a1e8562663b89e7d793470a2773ea721f59819ca5028d76a012fdf638338214f"
```
