# Hợp đồng hoán đổi token

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

## Hợp đồng hoán đổi token <a href="#a-swap-of-tokens" id="a-swap-of-tokens"></a>

Là hợp đồng hoán đổi 1 token cho 1 hoặc nhiều token khác.

Ví dụ này bao gồm bốn giao dịch:

1. Francis Beaumont tạo hợp đồng Marlowe bán token.
2. Francis Beaumont gửi 500 token Globe vào hợp đồng.
3. John Webster gửi 300 token Swan vào hợp đồng, khiến hợp đồng đóng lại và thanh toán các token cho lợi ích của các bên còn lại.
4. Francis Beaumont rút 300 token Swan của mình từ địa chỉ thanh toán của Marlowe.
5. John Webster rút 500 token Globe của mình từ địa chỉ thanh toán của Marlowe.

Dưới đây là hợp đồng ở định dạng Blockly:

![Token swap](https://raw.githubusercontent.com/input-output-hk/real-world-marlowe/a288a9f391e68135e59e65a813db695e6223472d/nfts/swap/contract.png)

### Thiết lập <a href="#set-up" id="set-up"></a>

Sử dụng `mainnet`.

In \[1]:

```
. ../../mainnet.env
```

Sử dụng các vai trò trong ví dụ tiêu chuẩn.

In \[2]:

```
. ../../dramatis-personae/roles.env
```

### Token vai trò&#x20;

Hợp đồng này sử dụng Ada Handles làm token vai trò:

* Francis Beaumont = [$f.beaumont](https://pool.pm/asset1dv4kncr59t9cndrqdhdd28l656eppcq9mlcxq7)
* John Webster = [$j.webster](https://pool.pm/asset1zdcycnnmg6dx5dy030u4cu0zdn63r2scghg2p4)

Lưu ý: Chỉ sử dụng token đã được phát hành trước làm token vai trò Marlowe nếu bạn đã xem xét chính sách tiền tệ để tìm kiếm các lỗ hổng bảo mật.

Dưới đây là ký hiệu tiền tệ cho Ada handles trên mainnet:&#x20;

In \[3]:

```
echo "ROLES_CURRENCY = $ROLES_CURRENCY"
```

```
ROLES_CURRENCY = f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
```

### Policy ID cho token Globe và Swan <a href="#policy-id-for-the-globe-and-swan-tokens" id="policy-id-for-the-globe-and-swan-tokens"></a>

Chúng tôi đã trước đó phát hành token Globe và Swan với chính sách sau đây.&#x20;

In \[4]:

```
echo "FUNGIBLES_POLICY = $FUNGIBLES_POLICY"
```

```
FUNGIBLES_POLICY = 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
```

### Gửi token ban đầu <a href="#initial-funding" id="initial-funding"></a>

Gửi token Globe và Swan từ faucet cho các bên liên quan.

In \[5]:

```
ADA=1000000
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "332f37fbcd87aaf56dd3596ec7dfb8567dfac3e5d5d344553fdbaf3977d1560d#1" \
  --tx-in "332f37fbcd87aaf56dd3596ec7dfb8567dfac3e5d5d344553fdbaf3977d1560d#2" \
  --tx-in "9f56ccd99e5ee8f4bc92fb229ffaaafa7bd5f0bbf72dd3d7071b0b4cc11df7cd#0" \
  --tx-in "a1e8562663b89e7d793470a2773ea721f59819ca5028d76a012fdf638338214f#0" \
  --tx-out "${ROLE_ADDR[f.beaumont]}+$((3 * ADA))+500 $FUNGIBLES_POLICY.Globe" \
  --tx-out "${ROLE_ADDR[j.webster]}+$((3 * ADA))+300 $FUNGIBLES_POLICY.Swan" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+200 $FUNGIBLES_POLICY.Swan" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "$FAUCET_SKEY" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "707db5afc8b367df20e6818907b6cc88c74f5572cc306e37fb40e6d97e57edb7"
```

### Hợp đồng Marlowe <a href="#the-marlowe-contract" id="the-marlowe-contract"></a>

Hợp đồng Marlowe thực tế chỉ cần download file JSON được biên dịch từ hợp đồng này dạng Blockly đã được thiết kế sẵn trong [Marlowe Playground](https://play.marlowe.iohk.io/#/).

In \[6]:

```
json2yaml contract.json
```

```
timeout: 1676679830000
timeout_continuation: close
when:
- case:
    deposits: 500
    into_account:
      role_token: f.beaumont
    of_token:
      currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
      token_name: Globe
    party:
      role_token: f.beaumont
  then:
    timeout: 1676679840000
    timeout_continuation: close
    when:
    - case:
        deposits: 300
        into_account:
          role_token: j.webster
        of_token:
          currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
          token_name: Swan
        party:
          role_token: j.webster
      then:
        from_account:
          role_token: f.beaumont
        pay: 500
        then:
          from_account:
            role_token: j.webster
          pay: 300
          then: close
          to:
            party:
              role_token: f.beaumont
          token:
            currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
            token_name: Swan
        to:
          party:
            role_token: j.webster
        token:
          currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
          token_name: Globe
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
CONTRACT_ID = 7efd47341da9f2c101848035112109c25523e8113caf1a6ec46ee48bd6827e13#1
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
TxId "7efd47341da9f2c101848035112109c25523e8113caf1a6ec46ee48bd6827e13"
```

Xem giao dịch qua trình khám phá dữ liệu Cardano.

In \[9]:

```
echo "https://cardanoscan.io/transaction/${CONTRACT_ID%%#1}?tab=utxo"
```

```
https://cardanoscan.io/transaction/7efd47341da9f2c101848035112109c25523e8113caf1a6ec46ee48bd6827e13?tab=utxo
```

Chúng ta cũng có thể sử dụng công cụ như `marlowe-pipe` để truy xuất hợp đồng từ blockchain và hiển thị nó

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
        timeout: 1676679830000
        timeout_continuation: close
        when:
        - case:
            deposits: 500
            into_account:
              role_token: f.beaumont
            of_token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: Globe
            party:
              role_token: f.beaumont
          then:
            timeout: 1676679840000
            timeout_continuation: close
            when:
            - case:
                deposits: 300
                into_account:
                  role_token: j.webster
                of_token:
                  currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                  token_name: Swan
                party:
                  role_token: j.webster
              then:
                from_account:
                  role_token: f.beaumont
                pay: 500
                then:
                  from_account:
                    role_token: j.webster
                  pay: 300
                  then: close
                  to:
                    party:
                      role_token: f.beaumont
                  token:
                    currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                    token_name: Swan
                to:
                  party:
                    role_token: j.webster
                token:
                  currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                  token_name: Globe
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
      txId: 7efd47341da9f2c101848035112109c25523e8113caf1a6ec46ee48bd6827e13
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps: []
```

### Giao dịch số 2. Francis Beaumont nạp 500 token Globe vào trong hợp đồng <a href="#transaction-2.-francis-beaumont-deposits-500-globe-tokens-into-the-contract" id="transaction-2.-francis-beaumont-deposits-500-globe-tokens-into-the-contract"></a>

In \[11]:

```
TX_2=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[f.beaumont]}" \
  --to-party "${ROLE_NAME[f.beaumont]}" \
  --currency "$FUNGIBLES_POLICY" \
  --token-name Globe \
  --quantity 500 \
  --change-address "${ROLE_ADDR[f.beaumont]}" \
  --manual-sign tx-2.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_2 = $TX_2"
```

```
TX_2 = 3331f5b45e640372729e41b846f7ae47f5f2f4049b896907bce28da5afb60398
```

Ký và gửi giao dịch.

In \[12]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-2.unsigned \
  --required-signer "${ROLE_SKEY[f.beaumont]}" \
  --timeout 600
```

```
TxId "3331f5b45e640372729e41b846f7ae47f5f2f4049b896907bce28da5afb60398"
```

Kiểm tra xem token đã được chuyển vào hợp đồng..

In \[13]:

```
echo "https://cardanoscan.io/transaction/$TX_2?tab=utxo"
```

```
https://cardanoscan.io/transaction/3331f5b45e640372729e41b846f7ae47f5f2f4049b896907bce28da5afb60398?tab=utxo
```

Ta có thể thấy output trong hợp đồng Marlowe đã có 500 token Globe.

In \[14]:

```
cardano-cli query utxo --mainnet --tx-in "$TX_2#1"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
3331f5b45e640372729e41b846f7ae47f5f2f4049b896907bce28da5afb60398     1        3000000 lovelace + 500 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d.476c6f6265 + TxOutDatumHash ScriptDataInBabbageEra "66392b9c9369b5485904aad6be47ab5fe1a3cd59979f9cf1623f0c82c9df8755"
```

### Giao dịch số 3: John Webster chuyển 300 token Swan tokens vào trong hợp đồng, dẫn đến việc hợp đồng thanh toán cho các bên. <a href="#transaction-3.-john-webster-deposits-300-swan-tokens-into-the-contract-causing-it-to-pay-the-parties" id="transaction-3.-john-webster-deposits-300-swan-tokens-into-the-contract-causing-it-to-pay-the-parties"></a>

In \[15]:

```
TX_3=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[j.webster]}" \
  --to-party "${ROLE_NAME[j.webster]}" \
  --currency "$FUNGIBLES_POLICY" \
  --token-name Swan \
  --quantity 300 \
  --change-address "${ROLE_ADDR[j.webster]}" \
  --manual-sign tx-3.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_3 = $TX_3"
```

```
TX_3 = d841657906fafd29f32638189d57e90903f49dce8712a182b520462825a076fe
```

Ký và gửi giao dịch.

In \[16]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-3.unsigned \
  --required-signer "${ROLE_SKEY[j.webster]}" \
  --timeout 600
```

```
TxId "d841657906fafd29f32638189d57e90903f49dce8712a182b520462825a076fe"
```

Kiểm tra xem hợp đồng đã đóng và các token đã được trả đúng cho các bên hưởng lợi tương ứng.

In \[17]:

```
echo "https://cardanoscan.io/transaction/$TX_3?tab=utxo"
```

```
https://cardanoscan.io/transaction/d841657906fafd29f32638189d57e90903f49dce8712a182b520462825a076fe?tab=utxo
```

### Giao dịch số 4: Francis Beaumont rút 300 token Swan tokens từ địa chỉ vai trò <a href="#transaction-4.-francis-beaumont-withdraws-his-300-swan-tokens-from-the-role-payout-address" id="transaction-4.-francis-beaumont-withdraws-his-300-swan-tokens-from-the-role-payout-address"></a>

In \[18]:

```
TX_4=$(
marlowe withdraw \
  --contract "$CONTRACT_ID" \
  --role "${ROLE_NAME[f.beaumont]}" \
  --change-address "${ROLE_ADDR[f.beaumont]}" \
  --manual-sign tx-4.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_4 = $TX_4"
```

```
TX_4 = 0f7271ba8d0502a6579f2b5fe139245c3a257a5504c2528e4a5ce87c0b5552ff
```

Ký và gửi giao dịch.

In \[19]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-4.unsigned \
  --required-signer "${ROLE_SKEY[f.beaumont]}" \
  --timeout 600
```

```
TxId "0f7271ba8d0502a6579f2b5fe139245c3a257a5504c2528e4a5ce87c0b5552ff"
```

Kiểm tra xem token đã được rút thành công chưa.

In \[20]:

```
echo "https://cardanoscan.io/transaction/$TX_4?tab=utxo"
```

```
https://cardanoscan.io/transaction/0f7271ba8d0502a6579f2b5fe139245c3a257a5504c2528e4a5ce87c0b5552ff?tab=utxo
```

### Transaction 5. John Webster rút 500 token Globe từ địa chỉ vai trò <a href="#transaction-5.-john-webster-withdraws-his-500-globe-tokens-from-the-role-payout-address" id="transaction-5.-john-webster-withdraws-his-500-globe-tokens-from-the-role-payout-address"></a>

In \[21]:

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
TX_5 = 696d702616287cf1f3a18320d9b21ebf121aac4e9ca52b7f0b7d0b4173d0f5f5
```

Ký và gửi giao dịch.

In \[22]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-5.unsigned \
  --required-signer "${ROLE_SKEY[j.webster]}" \
  --timeout 600
```

```
TxId "696d702616287cf1f3a18320d9b21ebf121aac4e9ca52b7f0b7d0b4173d0f5f5"
```

Kiểm tra xem token đã được rút thành công chưa.

In \[23]:

```
echo "https://cardanoscan.io/transaction/$TX_5?tab=utxo"
```

```
https://cardanoscan.io/transaction/696d702616287cf1f3a18320d9b21ebf121aac4e9ca52b7f0b7d0b4173d0f5f5?tab=utxo
```

### Xem lại toàn bộ hợp đồng <a href="#view-the-whole-history-of-the-contract" id="view-the-whole-history-of-the-contract"></a>

Chúng ta sử dụng câu lệnh `marlowe-pipe` để in toàn bộ lịch sử hợp đồng.

In \[24]:

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
            deposits: 500
            into_account:
              role_token: f.beaumont
            of_token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: Globe
            party:
              role_token: f.beaumont
          then:
            timeout: 1676679840000
            timeout_continuation: close
            when:
            - case:
                deposits: 300
                into_account:
                  role_token: j.webster
                of_token:
                  currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                  token_name: Swan
                party:
                  role_token: j.webster
              then:
                from_account:
                  role_token: f.beaumont
                pay: 500
                then:
                  from_account:
                    role_token: j.webster
                  pay: 300
                  then: close
                  to:
                    party:
                      role_token: f.beaumont
                  token:
                    currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                    token_name: Swan
                to:
                  party:
                    role_token: j.webster
                token:
                  currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                  token_name: Globe
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
      txId: 7efd47341da9f2c101848035112109c25523e8113caf1a6ec46ee48bd6827e13
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps:
- contractId: 7efd47341da9f2c101848035112109c25523e8113caf1a6ec46ee48bd6827e13#1
  payouts: []
  redeemer:
  - input_from_party:
      role_token: f.beaumont
    into_account:
      role_token: f.beaumont
    of_token:
      currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
      token_name: Globe
    that_deposits: 500
  scriptOutput:
    address: 716a9391d6aa51af28dd876ebb5565b69d1e83e5ac7861506bd29b56b0
    assets:
      ada: 3000000
      tokens:
      - - policyId: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
          tokenName: Globe
        - 500
    datum:
      marloweContract:
        timeout: 1676679840000
        timeout_continuation: close
        when:
        - case:
            deposits: 300
            into_account:
              role_token: j.webster
            of_token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: Swan
            party:
              role_token: j.webster
          then:
            from_account:
              role_token: f.beaumont
            pay: 500
            then:
              from_account:
                role_token: j.webster
              pay: 300
              then: close
              to:
                party:
                  role_token: f.beaumont
              token:
                currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                token_name: Swan
            to:
              party:
                role_token: j.webster
            token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: Globe
      marloweParams:
        rolesCurrency: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
      marloweState:
        accounts:
        - - - address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
            - currency_symbol: ''
              token_name: ''
          - 3000000
        - - - role_token: f.beaumont
            - currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: Globe
          - 500
        boundValues: []
        choices: []
        minTime: 1676391874000
    utxo:
      txId: 3331f5b45e640372729e41b846f7ae47f5f2f4049b896907bce28da5afb60398
      txIx: 1
  step: apply
  txId: 3331f5b45e640372729e41b846f7ae47f5f2f4049b896907bce28da5afb60398
- contractId: 7efd47341da9f2c101848035112109c25523e8113caf1a6ec46ee48bd6827e13#1
  payouts:
  - - txId: d841657906fafd29f32638189d57e90903f49dce8712a182b520462825a076fe
      txIx: 2
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 1193870
        tokens:
        - - policyId: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
            tokenName: Swan
          - 300
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: f.beaumont
  - - txId: d841657906fafd29f32638189d57e90903f49dce8712a182b520462825a076fe
      txIx: 3
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 1198180
        tokens:
        - - policyId: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
            tokenName: Globe
          - 500
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: j.webster
  redeemer:
  - input_from_party:
      role_token: j.webster
    into_account:
      role_token: j.webster
    of_token:
      currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
      token_name: Swan
    that_deposits: 300
  scriptOutput: null
  step: apply
  txId: d841657906fafd29f32638189d57e90903f49dce8712a182b520462825a076fe
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: f.beaumont
  redeemingTx: 0f7271ba8d0502a6579f2b5fe139245c3a257a5504c2528e4a5ce87c0b5552ff
  step: payout
  utxo:
    txId: d841657906fafd29f32638189d57e90903f49dce8712a182b520462825a076fe
    txIx: 2
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: j.webster
  redeemingTx: 696d702616287cf1f3a18320d9b21ebf121aac4e9ca52b7f0b7d0b4173d0f5f5
  step: payout
  utxo:
    txId: d841657906fafd29f32638189d57e90903f49dce8712a182b520462825a076fe
    txIx: 3
```

### Gửi trả các token về faucet <a href="#return-the-beargarden-token-to-the-faucet" id="return-the-beargarden-token-to-the-faucet"></a>

Returning the token to the faucet is convenient housekeeping for this example.

In \[25]:

```
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "0f7271ba8d0502a6579f2b5fe139245c3a257a5504c2528e4a5ce87c0b5552ff#2" \
  --tx-in "696d702616287cf1f3a18320d9b21ebf121aac4e9ca52b7f0b7d0b4173d0f5f5#2" \
  --tx-in "707db5afc8b367df20e6818907b6cc88c74f5572cc306e37fb40e6d97e57edb7#3" \
  --tx-in "7efd47341da9f2c101848035112109c25523e8113caf1a6ec46ee48bd6827e13#0" \
  --tx-in "d841657906fafd29f32638189d57e90903f49dce8712a182b520462825a076fe#4" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+500 $FUNGIBLES_POLICY.Globe" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+500 $FUNGIBLES_POLICY.Swan" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "$FAUCET_SKEY" \
  --required-signer "${ROLE_SKEY[f.beaumont]}" \
  --required-signer "${ROLE_SKEY[j.webster]}" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "efc74dd7dfe0f033e17c26e93ab2db01b356f6642e4b0eb66ebd353771e98ff4"
```
