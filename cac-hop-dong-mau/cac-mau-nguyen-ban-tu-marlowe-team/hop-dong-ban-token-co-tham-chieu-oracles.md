# Hợp đồng Bán token có tham chiếu oracles

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

### Bán token có tham chiếu giá từ oracles

Đây là ví dụ về việc bán token sử dụng oracle giá để thiết lập chi phí của các token:

Ví dụ này bao gồm sáu giao dịch:

1. John Webster tạo hợp đồng Marlowe bán token.
2. John Webster gửi 1,000,000 token HOSKY vào hợp đồng.
3. Oracle giá báo cáo tỷ lệ trao đổi là 1 ADA = 11,074,197 HOSKY.
4. Elizabeth Cary gửi 90,300 lovelace vào hợp đồng, khiến hợp đồng thanh toán token cho cô và lovelace cho John Webster.
5. John Webster rút lovelace từ địa chỉ thanh toán của vai trò trong Marlowe.
6. Elizabeth rút 1,000,000 token HOSKY từ địa chỉ thanh toán của vai trò trong Marlowe.

Dưới đây là hợp đồng ở định dạng Blockly:

![Token sale with price oracle](https://raw.githubusercontent.com/input-output-hk/real-world-marlowe/a288a9f391e68135e59e65a813db695e6223472d/nfts/oracle/contract.png)

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

* Elizabeth Carey = [$e.cary](https://pool.pm/asset1tx4euajkdczmkawgkjy342agaq33885dlvp0jl)
* John Webster = [$j.webster](https://pool.pm/asset1zdcycnnmg6dx5dy030u4cu0zdn63r2scghg2p4)

_Lưu ý: Chỉ sử dụng token đã được phát hành trước làm token vai trò Marlowe nếu bạn đã xem xét chính sách tiền tệ để tìm kiếm các lỗ hổng bảo mật._

Dưới đây là ký hiệu tiền tệ cho Ada handles trên `mainnet`:

In \[3]:

```
echo "ROLES_CURRENCY = $ROLES_CURRENCY"
```

```
ROLES_CURRENCY = f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
```

### Policy ID cho token Hosky <a href="#policy-id-for-the-beargarden-token" id="policy-id-for-the-beargarden-token"></a>

Chúng tôi đã trước đó phát hành token Hosky với chính sách sau đây.&#x20;

In \[4]:

```
echo "HOSKY_POLICY = $HOSKY_POLICY"
echo "HOSKY_NAME = $HOSKY_NAME"
```

```
HOSKY_POLICY = a0028f350aaabe0545fdcb56b039bfb08e4bb4d8c4d7c3c7d481c235
HOSKY_NAME = HOSKY
```

### Gửi ban đầu <a href="#initial-funding" id="initial-funding"></a>

Gửi token HOSKY từ faucet tới John Webster.

In \[5]:

```
ADA=1000000
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "0f8060fd92a3918a9536f911f4e575fcbd9cad889e8d61807878f82d631be43d#4" \
  --tx-out "${ROLE_ADDR[j.webster]}+$((3 * ADA))+1000000 $HOSKY_POLICY.$HOSKY_NAME" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "$FAUCET_SKEY" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "1c5c3c72808a76fe4148c83f1c323417d746259f01c1d9b50df1d251799d7e2f"
```

### Hợp đồng Marlowe <a href="#the-marlowe-contract" id="the-marlowe-contract"></a>

Hợp đồng Marlowe thực tế chỉ cần download file JSON được biên dịch từ hợp đồng này dạng Blockly đã được thiết kế sẵn trong [Marlowe Playground](https://play.marlowe.iohk.io/#/).

In \[6]:

```
json2yaml contract.json
```

```
timeout: 1676767000000
timeout_continuation: close
when:
- case:
    deposits: 1000000
    into_account:
      role_token: j.webster
    of_token:
      currency_symbol: a0028f350aaabe0545fdcb56b039bfb08e4bb4d8c4d7c3c7d481c235
      token_name: HOSKY
    party:
      role_token: j.webster
  then:
    timeout: 1676767100000
    timeout_continuation: close
    when:
    - case:
        choose_between:
        - from: 1
          to: 1000000000
        for_choice:
          choice_name: ADA_HOSKY
          choice_owner:
            address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
      then:
        be:
          by:
            value_of_choice:
              choice_name: ADA_HOSKY
              choice_owner:
                address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
          divide:
            multiply:
              amount_of_token:
                currency_symbol: a0028f350aaabe0545fdcb56b039bfb08e4bb4d8c4d7c3c7d481c235
                token_name: HOSKY
              in_account:
                role_token: j.webster
            times: 1000000
        let: Cost
        then:
          timeout: 1676767100000
          timeout_continuation: close
          when:
          - case:
              deposits:
                use_value: Cost
              into_account:
                role_token: j.webster
              of_token:
                currency_symbol: ''
                token_name: ''
              party:
                role_token: e.cary
            then:
              from_account:
                role_token: j.webster
              pay:
                amount_of_token:
                  currency_symbol: a0028f350aaabe0545fdcb56b039bfb08e4bb4d8c4d7c3c7d481c235
                  token_name: HOSKY
                in_account:
                  role_token: j.webster
              then: close
              to:
                party:
                  role_token: e.cary
              token:
                currency_symbol: a0028f350aaabe0545fdcb56b039bfb08e4bb4d8c4d7c3c7d481c235
                token_name: HOSKY
```

### Giao dịch số 1: Tạo hợp đồng <a href="#transaction-1.-create-the-contract" id="transaction-1.-create-the-contract"></a>

Chúng ta sử dụng công cụ _Marlowe Runtime's command-line_ để xây dựng giao dịch tạo hợp đồng.

In \[7]:

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
CONTRACT_ID = 2a30bc48422c5195907adc8a872fa87bc8c68d701f897ef116560fdc16aa7830#1
```

Hợp đồng có thể được ký và gửi bằng bất kỳ ví hoặc dịch vụ nào. Để thuận tiện, chúng ta sử dụng `marlowe-cli`.

In \[8]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-1.unsigned \
  --required-signer "$FAUCET_SKEY" \
  --timeout 600
```

```
TxId "2a30bc48422c5195907adc8a872fa87bc8c68d701f897ef116560fdc16aa7830"
```

Xem hợp đồng qua trình khám phá dữ liệu blockchain Cardano (ví dụ: Cardanoscan.io).

In \[9]:

```
echo "https://cardanoscan.io/transaction/${CONTRACT_ID%%#1}?tab=utxo"
```

```
https://cardanoscan.io/transaction/2a30bc48422c5195907adc8a872fa87bc8c68d701f897ef116560fdc16aa7830?tab=utxo
```

Chúng ta cũng có thể sử dụng công cụ như `marlowe-pipe` để truy xuất hợp đồng từ blockchain và hiển thị nó.

In \[10]:

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
        timeout: 1676767000000
        timeout_continuation: close
        when:
        - case:
            deposits: 1000000
            into_account:
              role_token: j.webster
            of_token:
              currency_symbol: a0028f350aaabe0545fdcb56b039bfb08e4bb4d8c4d7c3c7d481c235
              token_name: HOSKY
            party:
              role_token: j.webster
          then:
            timeout: 1676767100000
            timeout_continuation: close
            when:
            - case:
                choose_between:
                - from: 1
                  to: 1000000000
                for_choice:
                  choice_name: ADA_HOSKY
                  choice_owner:
                    address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
              then:
                be:
                  by:
                    value_of_choice:
                      choice_name: ADA_HOSKY
                      choice_owner:
                        address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
                  divide:
                    multiply:
                      amount_of_token:
                        currency_symbol: a0028f350aaabe0545fdcb56b039bfb08e4bb4d8c4d7c3c7d481c235
                        token_name: HOSKY
                      in_account:
                        role_token: j.webster
                    times: 1000000
                let: Cost
                then:
                  timeout: 1676767100000
                  timeout_continuation: close
                  when:
                  - case:
                      deposits:
                        use_value: Cost
                      into_account:
                        role_token: j.webster
                      of_token:
                        currency_symbol: ''
                        token_name: ''
                      party:
                        role_token: e.cary
                    then:
                      from_account:
                        role_token: j.webster
                      pay:
                        amount_of_token:
                          currency_symbol: a0028f350aaabe0545fdcb56b039bfb08e4bb4d8c4d7c3c7d481c235
                          token_name: HOSKY
                        in_account:
                          role_token: j.webster
                      then: close
                      to:
                        party:
                          role_token: e.cary
                      token:
                        currency_symbol: a0028f350aaabe0545fdcb56b039bfb08e4bb4d8c4d7c3c7d481c235
                        token_name: HOSKY
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
      txId: 2a30bc48422c5195907adc8a872fa87bc8c68d701f897ef116560fdc16aa7830
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps: []
```

### Giao dịch số 2. John Webster nạp 1.000.000 token HOSKY vào hợp đồng. <a href="#transaction-2.-john-webster-deposits-the-1-000-000-hosky-tokens-into-the-contract" id="transaction-2.-john-webster-deposits-the-1-000-000-hosky-tokens-into-the-contract"></a>

In \[11]:

```
TX_2=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[j.webster]}" \
  --to-party "${ROLE_NAME[j.webster]}" \
  --currency "$HOSKY_POLICY" \
  --token-name "$HOSKY_NAME" \
  --quantity 1000000 \
  --change-address "${ROLE_ADDR[j.webster]}" \
  --manual-sign tx-2.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_2 = $TX_2"
```

```
TX_2 = 11f0141f19cf0cff9598c7c9c391ce19bcb7a373a87ae4b8fd4db18461221821
```

Ký và gửi giao dịch.

In \[12]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-2.unsigned \
  --required-signer "${ROLE_SKEY[j.webster]}" \
  --timeout 600
```

```
TxId "11f0141f19cf0cff9598c7c9c391ce19bcb7a373a87ae4b8fd4db18461221821"
```

Kiểm tra xem token đã được nạp vào hợp đồng

In \[13]:

```
echo "https://cardanoscan.io/transaction/$TX_2?tab=utxo"
```

```
https://cardanoscan.io/transaction/11f0141f19cf0cff9598c7c9c391ce19bcb7a373a87ae4b8fd4db18461221821?tab=utxo
```

Kiểm tra output trong hợp đồng và thấy đã có 1.000.000 token HOSKY .

In \[14]:

```
cardano-cli query utxo --mainnet --tx-in "$TX_2#1"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
11f0141f19cf0cff9598c7c9c391ce19bcb7a373a87ae4b8fd4db18461221821     1        3000000 lovelace + 1000000 a0028f350aaabe0545fdcb56b039bfb08e4bb4d8c4d7c3c7d481c235.484f534b59 + TxOutDatumHash ScriptDataInBabbageEra "320beb29eceb8eca03772d4bd6b4d37311374f24ee7e95fa853c8f80a88d5569"
```

### Giao dịch 3. Lấy giá từ Oracle làm tỷ lệ trao đổi giữa ADA và HOSKY.

In \[15]:

```
TX_3=$(
marlowe choose \
  --contract "$CONTRACT_ID" \
  --party "$FAUCET_ADDR" \
  --choice "ADA_HOSKY" \
  --value 11074197 \
  --change-address "$FAUCET_ADDR" \
  --manual-sign tx-3.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_3 = $TX_3"
```

```
TX_3 = 353dc2b8e92615fd2f6654d22ad862a176f0fdc88277d0917236c8b39b1c7cb5
```

Ký và gửi giao dịch.

In \[16]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-3.unsigned \
  --required-signer "$FAUCET_SKEY" \
  --timeout 600
```

```
TxId "353dc2b8e92615fd2f6654d22ad862a176f0fdc88277d0917236c8b39b1c7cb5"
```

Kiểm tra xem hợp đồng đã đóng và thanh toán cho các bên liên quan.

In \[17]:

```
echo "https://cardanoscan.io/transaction/$TX_3?tab=utxo"
```

```
https://cardanoscan.io/transaction/353dc2b8e92615fd2f6654d22ad862a176f0fdc88277d0917236c8b39b1c7cb5?tab=utxo
```

### Giao dịch số 4: Elizabeth Cary thanh toán Ada vào trong hợp đồng, dẫn tới việc thanh toán cho các bên. <a href="#transaction-4.-elizabeth-cary-pays-ada-into-the-contract-causing-it-to-pay-the-parties" id="transaction-4.-elizabeth-cary-pays-ada-into-the-contract-causing-it-to-pay-the-parties"></a>

Kiểm tra trạng thái của hợp đồng để thấy rằng chi phí của các token HOSKY được tính toán là 90,300 lovelace. Bởi vì số này thấp hơn giá trị UTxO tối thiểu cho sổ cái, Elizabeth Cary sẽ thực tế phải trả khoảng 1 ADA.

In \[18]:

```
echo '{"request" : "get", "contractId" : "'"$CONTRACT_ID"'"}' | marlowe-pipe 2> /dev/null | jq '.steps[1].scriptOutput.datum.marloweState' | json2yaml
```

```
accounts:
- - - address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
    - currency_symbol: ''
      token_name: ''
  - 3000000
- - - role_token: j.webster
    - currency_symbol: a0028f350aaabe0545fdcb56b039bfb08e4bb4d8c4d7c3c7d481c235
      token_name: HOSKY
  - 1000000
boundValues:
- - Cost
  - 90300
choices:
- - choice_name: ADA_HOSKY
    choice_owner:
      address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
  - 11074197
minTime: 1676748662000
```

In \[19]:

```
TX_4=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[e.cary]}" \
  --to-party "${ROLE_NAME[j.webster]}" \
  --lovelace 90300 \
  --change-address "${ROLE_ADDR[e.cary]}" \
  --manual-sign tx-4.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_4 = $TX_4"
```

```
TX_4 = ab6f688558f2014b05cf7035b915c3f1c9e13058319fa74584dc58e13cd66a43
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
TxId "ab6f688558f2014b05cf7035b915c3f1c9e13058319fa74584dc58e13cd66a43"
```

Kiểm tra xem hợp đồng đã đóng và thanh toán cho các bên liên quan.

In \[21]:

```
echo "https://cardanoscan.io/transaction/$TX_4?tab=utxo"
```

```
https://cardanoscan.io/transaction/ab6f688558f2014b05cf7035b915c3f1c9e13058319fa74584dc58e13cd66a43?tab=utxo
```

### Transaction 4. John Webster rút Ada từ địa chỉ thanh toán vai trò <a href="#transaction-4.-john-webster-withdraws-his-ada-from-the-role-payout-address" id="transaction-4.-john-webster-withdraws-his-ada-from-the-role-payout-address"></a>

In \[22]:

```
TX_5=$(
marlowe withdraw \
  --contract "$CONTRACT_ID" \
  --role "${ROLE_NAME[j.webster]}" \
  --change-address "${ROLE_ADDR[j.webster]}" \
  --manual-sign tx-5.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_5 = $TX_5"
```

```
TX_5 = c7d683da1ecdf9d8b48e2322b8c5dd23d135d4c86403249794e1668288361d2d
```

Ký và gửi giao dịch.

In \[23]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-5.unsigned \
  --required-signer "${ROLE_SKEY[j.webster]}" \
  --timeout 600
```

```
TxId "c7d683da1ecdf9d8b48e2322b8c5dd23d135d4c86403249794e1668288361d2d"
```

Kiểm tra xem John Webster đã rút thành công Ada từ địa chỉ thanh toán vai trò.

In \[24]:

```
echo "https://cardanoscan.io/transaction/$TX_5?tab=utxo"
```

```
https://cardanoscan.io/transaction/c7d683da1ecdf9d8b48e2322b8c5dd23d135d4c86403249794e1668288361d2d?tab=utxo
```

### Giao dịch số 5: Elizabeth Cary rút 1.000.000 token HOSKY từ địa chỉ thanh toán vai trò. <a href="#transaction-5.-elizabeth-cary-withdraws-her-1-000-000-hosky-from-the-role-payout-address" id="transaction-5.-elizabeth-cary-withdraws-her-1-000-000-hosky-from-the-role-payout-address"></a>

In \[25]:

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
TX_6 = 0b13b5d5e1d327dbeedf519c7973744d79c82d44128784b536fbbe2cb4a0569e
```

Ký và gửi giao dịch.

In \[26]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-6.unsigned \
  --required-signer "${ROLE_SKEY[e.cary]}" \
  --timeout 600
```

```
TxId "0b13b5d5e1d327dbeedf519c7973744d79c82d44128784b536fbbe2cb4a0569e"
```

Kiểm tra xem Elizabeth Cary đã rút thành công 1.000.000 token HOSKY từ địa chỉ thanh toán vai trò.

In \[27]:

```
echo "https://cardanoscan.io/transaction/$TX_6?tab=utxo"
```

```
https://cardanoscan.io/transaction/0b13b5d5e1d327dbeedf519c7973744d79c82d44128784b536fbbe2cb4a0569e?tab=utxo
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
        timeout: 1676767000000
        timeout_continuation: close
        when:
        - case:
            deposits: 1000000
            into_account:
              role_token: j.webster
            of_token:
              currency_symbol: a0028f350aaabe0545fdcb56b039bfb08e4bb4d8c4d7c3c7d481c235
              token_name: HOSKY
            party:
              role_token: j.webster
          then:
            timeout: 1676767100000
            timeout_continuation: close
            when:
            - case:
                choose_between:
                - from: 1
                  to: 1000000000
                for_choice:
                  choice_name: ADA_HOSKY
                  choice_owner:
                    address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
              then:
                be:
                  by:
                    value_of_choice:
                      choice_name: ADA_HOSKY
                      choice_owner:
                        address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
                  divide:
                    multiply:
                      amount_of_token:
                        currency_symbol: a0028f350aaabe0545fdcb56b039bfb08e4bb4d8c4d7c3c7d481c235
                        token_name: HOSKY
                      in_account:
                        role_token: j.webster
                    times: 1000000
                let: Cost
                then:
                  timeout: 1676767100000
                  timeout_continuation: close
                  when:
                  - case:
                      deposits:
                        use_value: Cost
                      into_account:
                        role_token: j.webster
                      of_token:
                        currency_symbol: ''
                        token_name: ''
                      party:
                        role_token: e.cary
                    then:
                      from_account:
                        role_token: j.webster
                      pay:
                        amount_of_token:
                          currency_symbol: a0028f350aaabe0545fdcb56b039bfb08e4bb4d8c4d7c3c7d481c235
                          token_name: HOSKY
                        in_account:
                          role_token: j.webster
                      then: close
                      to:
                        party:
                          role_token: e.cary
                      token:
                        currency_symbol: a0028f350aaabe0545fdcb56b039bfb08e4bb4d8c4d7c3c7d481c235
                        token_name: HOSKY
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
      txId: 2a30bc48422c5195907adc8a872fa87bc8c68d701f897ef116560fdc16aa7830
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps:
- contractId: 2a30bc48422c5195907adc8a872fa87bc8c68d701f897ef116560fdc16aa7830#1
  payouts: []
  redeemer:
  - input_from_party:
      role_token: j.webster
    into_account:
      role_token: j.webster
    of_token:
      currency_symbol: a0028f350aaabe0545fdcb56b039bfb08e4bb4d8c4d7c3c7d481c235
      token_name: HOSKY
    that_deposits: 1000000
  scriptOutput:
    address: 716a9391d6aa51af28dd876ebb5565b69d1e83e5ac7861506bd29b56b0
    assets:
      ada: 3000000
      tokens:
      - - policyId: a0028f350aaabe0545fdcb56b039bfb08e4bb4d8c4d7c3c7d481c235
          tokenName: HOSKY
        - 1000000
    datum:
      marloweContract:
        timeout: 1676767100000
        timeout_continuation: close
        when:
        - case:
            choose_between:
            - from: 1
              to: 1000000000
            for_choice:
              choice_name: ADA_HOSKY
              choice_owner:
                address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
          then:
            be:
              by:
                value_of_choice:
                  choice_name: ADA_HOSKY
                  choice_owner:
                    address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
              divide:
                multiply:
                  amount_of_token:
                    currency_symbol: a0028f350aaabe0545fdcb56b039bfb08e4bb4d8c4d7c3c7d481c235
                    token_name: HOSKY
                  in_account:
                    role_token: j.webster
                times: 1000000
            let: Cost
            then:
              timeout: 1676767100000
              timeout_continuation: close
              when:
              - case:
                  deposits:
                    use_value: Cost
                  into_account:
                    role_token: j.webster
                  of_token:
                    currency_symbol: ''
                    token_name: ''
                  party:
                    role_token: e.cary
                then:
                  from_account:
                    role_token: j.webster
                  pay:
                    amount_of_token:
                      currency_symbol: a0028f350aaabe0545fdcb56b039bfb08e4bb4d8c4d7c3c7d481c235
                      token_name: HOSKY
                    in_account:
                      role_token: j.webster
                  then: close
                  to:
                    party:
                      role_token: e.cary
                  token:
                    currency_symbol: a0028f350aaabe0545fdcb56b039bfb08e4bb4d8c4d7c3c7d481c235
                    token_name: HOSKY
      marloweParams:
        rolesCurrency: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
      marloweState:
        accounts:
        - - - address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
            - currency_symbol: ''
              token_name: ''
          - 3000000
        - - - role_token: j.webster
            - currency_symbol: a0028f350aaabe0545fdcb56b039bfb08e4bb4d8c4d7c3c7d481c235
              token_name: HOSKY
          - 1000000
        boundValues: []
        choices: []
        minTime: 1676748491000
    utxo:
      txId: 11f0141f19cf0cff9598c7c9c391ce19bcb7a373a87ae4b8fd4db18461221821
      txIx: 1
  step: apply
  txId: 11f0141f19cf0cff9598c7c9c391ce19bcb7a373a87ae4b8fd4db18461221821
- contractId: 2a30bc48422c5195907adc8a872fa87bc8c68d701f897ef116560fdc16aa7830#1
  payouts: []
  redeemer:
  - for_choice_id:
      choice_name: ADA_HOSKY
      choice_owner:
        address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
    input_that_chooses_num: 11074197
  scriptOutput:
    address: 716a9391d6aa51af28dd876ebb5565b69d1e83e5ac7861506bd29b56b0
    assets:
      ada: 3000000
      tokens:
      - - policyId: a0028f350aaabe0545fdcb56b039bfb08e4bb4d8c4d7c3c7d481c235
          tokenName: HOSKY
        - 1000000
    datum:
      marloweContract:
        timeout: 1676767100000
        timeout_continuation: close
        when:
        - case:
            deposits:
              use_value: Cost
            into_account:
              role_token: j.webster
            of_token:
              currency_symbol: ''
              token_name: ''
            party:
              role_token: e.cary
          then:
            from_account:
              role_token: j.webster
            pay:
              amount_of_token:
                currency_symbol: a0028f350aaabe0545fdcb56b039bfb08e4bb4d8c4d7c3c7d481c235
                token_name: HOSKY
              in_account:
                role_token: j.webster
            then: close
            to:
              party:
                role_token: e.cary
            token:
              currency_symbol: a0028f350aaabe0545fdcb56b039bfb08e4bb4d8c4d7c3c7d481c235
              token_name: HOSKY
      marloweParams:
        rolesCurrency: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
      marloweState:
        accounts:
        - - - address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
            - currency_symbol: ''
              token_name: ''
          - 3000000
        - - - role_token: j.webster
            - currency_symbol: a0028f350aaabe0545fdcb56b039bfb08e4bb4d8c4d7c3c7d481c235
              token_name: HOSKY
          - 1000000
        boundValues:
        - - Cost
          - 90300
        choices:
        - - choice_name: ADA_HOSKY
            choice_owner:
              address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
          - 11074197
        minTime: 1676748662000
    utxo:
      txId: 353dc2b8e92615fd2f6654d22ad862a176f0fdc88277d0917236c8b39b1c7cb5
      txIx: 1
  step: apply
  txId: 353dc2b8e92615fd2f6654d22ad862a176f0fdc88277d0917236c8b39b1c7cb5
- contractId: 2a30bc48422c5195907adc8a872fa87bc8c68d701f897ef116560fdc16aa7830#1
  payouts:
  - - txId: ab6f688558f2014b05cf7035b915c3f1c9e13058319fa74584dc58e13cd66a43
      txIx: 2
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 1206800
        tokens:
        - - policyId: a0028f350aaabe0545fdcb56b039bfb08e4bb4d8c4d7c3c7d481c235
            tokenName: HOSKY
          - 1000000
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: e.cary
  - - txId: ab6f688558f2014b05cf7035b915c3f1c9e13058319fa74584dc58e13cd66a43
      txIx: 3
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 1017160
        tokens: []
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: j.webster
  redeemer:
  - input_from_party:
      role_token: e.cary
    into_account:
      role_token: j.webster
    of_token:
      currency_symbol: ''
      token_name: ''
    that_deposits: 90300
  scriptOutput: null
  step: apply
  txId: ab6f688558f2014b05cf7035b915c3f1c9e13058319fa74584dc58e13cd66a43
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: j.webster
  redeemingTx: c7d683da1ecdf9d8b48e2322b8c5dd23d135d4c86403249794e1668288361d2d
  step: payout
  utxo:
    txId: ab6f688558f2014b05cf7035b915c3f1c9e13058319fa74584dc58e13cd66a43
    txIx: 3
```

### Trả lại token HOSKY về faucet <a href="#return-the-beargarden-tokens-to-the-faucet" id="return-the-beargarden-tokens-to-the-faucet"></a>

Trả lại token cho faucet là một cách quản lý tiện lợi cho ví dụ này.

In \[29]:

```
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "0b13b5d5e1d327dbeedf519c7973744d79c82d44128784b536fbbe2cb4a0569e#0" \
  --tx-in "0b13b5d5e1d327dbeedf519c7973744d79c82d44128784b536fbbe2cb4a0569e#2" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+1000000 $HOSKY_POLICY.$HOSKY_NAME" \
  --change-address "${ROLE_ADDR[e.cary]}" \
  --required-signer "${ROLE_SKEY[e.cary]}" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "3c262668919ee8605f81f568f4dc5984d27204449a5b14c6af379e94bdb7ab74"
```
