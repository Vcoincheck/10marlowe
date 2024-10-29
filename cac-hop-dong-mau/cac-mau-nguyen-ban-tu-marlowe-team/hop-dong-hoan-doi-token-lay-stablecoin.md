# Hợp đồng hoán đổi token lấy stablecoin

### <mark style="color:red;">**Cảnh báo**</mark>**!**

Trước khi chạy một hợp đồng Marlowe trên mainnet, hãy làm theo các bước sau để tránh mất tiền:

1. Hiểu [ngôn ngữ Marlowe](https://marlowe.iohk.io/).
2. Hiểu mô hình [Extended UTxO](https://docs.cardano.org/learn/eutxo-explainer) của Cardano.
3. Đọc và hiểu Hướng dẫn [Thực hành Tốt nhất của Marlowe](https://github.com/input-output-hk/marlowe-cardano/blob/main/marlowe/best-practices.md).
4. Đọc và hiểu [Hướng dẫn Bảo mật](https://github.com/input-output-hk/marlowe-cardano/blob/main/marlowe/security.md) của Marlowe.
5. Sử dụng [Marlowe Playground](https://playground.marlowe-lang.org/#/) để đánh dấu cảnh báo, thực hiện phân tích tĩnh và mô phỏng hợp đồng.
6. Sử dụng công cụ `marlowe-cli run analyze` của [Marlowe CLI](https://github.com/input-output-hk/marlowe-cardano/blob/main/marlowe-cli/ReadMe.md) để kiểm tra xem hợp đồng có thể chạy trên mạng Cardano hay không.
7. Chạy tất cả các đường thực thi của hợp đồng trên [testnet](https://docs.cardano.org/cardano-testnet/overview) Cardano.

***

### Hợp đồng hoán đổi token lấy stablecoin

Là hợp đồng bán 1 token để đổi lấy stablecoin Djed hoặc iUSD.

Ví dụ này bao gồm năm giao dịch:

1. Christopher Marlowe tạo hợp đồng Marlowe bán token.
2. Christopher Marlowe gửi 1 token BearGarden vào hợp đồng.
3. Jane Lumley gửi 50 Djed vào hợp đồng, khiến hợp đồng thanh toán token cho cô và Djed cho Christopher Marlowe.
4. Christopher Marlowe rút 50 Djed của mình từ địa chỉ thanh toán của Marlowe.
5. Jane Lumley rút 1 BearGarden của mình từ địa chỉ thanh toán của Marlowe.

Dưới đây là hợp đồng ở định dạng Blockly:

![Hoán đổi token lấy stablecoin](https://raw.githubusercontent.com/input-output-hk/real-world-marlowe/a288a9f391e68135e59e65a813db695e6223472d/nfts/stable/contract.png)

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

* Christopher Marlowe = $c.marlowe
* Jane Lumley = $j.lumley

Lưu ý: Chỉ sử dụng token đã được phát hành trước làm token vai trò Marlowe nếu bạn đã xem xét chính sách tiền tệ để tìm kiếm các lỗ hổng bảo mật.

Dưới đây là ký hiệu tiền tệ cho Ada handles trên mainnet: In \[3]:

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

### Policy IDs cho Djed và iUSD <a href="#policy-ids-for-djed-and-iusd" id="policy-ids-for-djed-and-iusd"></a>

Dưới đây là policy cho các stablecoin mà chúng ta sử dụng

In \[5]:

```
echo "DJED_POLICY = $DJED_POLICY"
echo "DJED_NAME = $DJED_NAME"
echo
echo "IUSD_POLICY = $IUSD_POLICY"
echo "IUSD_NAME = $IUSD_NAME"
```

```
DJED_POLICY = 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
DJED_NAME = DjedMicroUSD

IUSD_POLICY = f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
IUSD_NAME = iUSD
```

### Gửi token ban đầu <a href="#initial-funding" id="initial-funding"></a>

Gửi token BearGarden từ faucet cho Christopher Marlowe và stablecoin cho Jane Lumley.

In \[6]:

```
ADA=1000000
DJED=1000000
IUSD=1000000
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "22e84f55fb3035d4b1f031788473f65eac8a49776d6b0a2d3db9919de2f69aa3#0" \
  --tx-in "3ade722a48e57a6d59b310ed520df6f90114481712ec313cbd24cd0b60ecefc1#1" \
  --tx-in "c1246f0f0d47804be9ee4c0439065c438a5c03c852781a1c3e390a0683697885#4" \
  --tx-in "c2970d098870bbf9cd8d7f94fba69a7dd1f08171b580fc53a1e7aae57069311a#2" \
  --tx-in "e8e9b557a82c6e0bfbe267acbf54980bad2cbed8d9cb500d801d04e1938b1f24#0" \
  --tx-out "${ROLE_ADDR[j.lumley]}+$((3 * ADA))+$((100 * DJED)) $DJED_POLICY.$DJED_NAME" \
  --tx-out "${ROLE_ADDR[j.lumley]}+$((3 * ADA))+$((100 * IUSD)) $IUSD_POLICY.$IUSD_NAME" \
  --tx-out "${ROLE_ADDR[c.marlowe]}+$((3 * ADA))+500 $FUNGIBLES_POLICY.BearGarden" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "$FAUCET_SKEY" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "855c131e9b59a6607b4d28cfcb3a68b7f3f9672d03805c9252a6515f6e4ab226"
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
      role_token: c.marlowe
    of_token:
      currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
      token_name: BearGarden
    party:
      role_token: c.marlowe
  then:
    timeout: 1676679840000
    timeout_continuation: close
    when:
    - case:
        deposits: 50000000
        into_account:
          role_token: c.marlowe
        of_token:
          currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
          token_name: DjedMicroUSD
        party:
          role_token: j.lumley
      then:
        from_account:
          role_token: c.marlowe
        pay: 1
        then: close
        to:
          party:
            role_token: j.lumley
        token:
          currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
          token_name: BearGarden
    - case:
        deposits: 50000000
        into_account:
          role_token: c.marlowe
        of_token:
          currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
          token_name: iUSD
        party:
          role_token: j.lumley
      then:
        from_account:
          role_token: c.marlowe
        pay: 1
        then: close
        to:
          party:
            role_token: j.lumley
        token:
          currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
          token_name: BearGarden
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
CONTRACT_ID = 561c6ccdeba9505a385c92f3c3b9c3b007c92c34bbffef8f9a88f02eba1bd4de#1
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
TxId "561c6ccdeba9505a385c92f3c3b9c3b007c92c34bbffef8f9a88f02eba1bd4de"
```

In \[10]:

```
echo "https://cardanoscan.io/transaction/${CONTRACT_ID%%#1}?tab=utxo"
```

```
https://cardanoscan.io/transaction/561c6ccdeba9505a385c92f3c3b9c3b007c92c34bbffef8f9a88f02eba1bd4de?tab=utxo
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
              role_token: c.marlowe
            of_token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
            party:
              role_token: c.marlowe
          then:
            timeout: 1676679840000
            timeout_continuation: close
            when:
            - case:
                deposits: 50000000
                into_account:
                  role_token: c.marlowe
                of_token:
                  currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
                  token_name: DjedMicroUSD
                party:
                  role_token: j.lumley
              then:
                from_account:
                  role_token: c.marlowe
                pay: 1
                then: close
                to:
                  party:
                    role_token: j.lumley
                token:
                  currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                  token_name: BearGarden
            - case:
                deposits: 50000000
                into_account:
                  role_token: c.marlowe
                of_token:
                  currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                  token_name: iUSD
                party:
                  role_token: j.lumley
              then:
                from_account:
                  role_token: c.marlowe
                pay: 1
                then: close
                to:
                  party:
                    role_token: j.lumley
                token:
                  currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                  token_name: BearGarden
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
      txId: 561c6ccdeba9505a385c92f3c3b9c3b007c92c34bbffef8f9a88f02eba1bd4de
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps: []
```

### Giao dịch số 2: Christopher Marlowe gửi token BearGarden vào hợp đồng.

Logic của hợp đồng quy định rằng Christopher Marlowe sẽ gửi một token BearGarden vào tài khoản của mình trong hợp đồng Marlowe.

In \[12]:

```
TX_2=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[c.marlowe]}" \
  --to-party "${ROLE_NAME[c.marlowe]}" \
  --currency "$FUNGIBLES_POLICY" \
  --token-name BearGarden \
  --quantity 1 \
  --change-address "${ROLE_ADDR[c.marlowe]}" \
  --manual-sign tx-2.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_2 = $TX_2"
```

```
TX_2 = 4b8528698e34e16837649ef68769653196985180e1a71dc874d2fe8a53a8942a
```

Ký và gửi giao dịch.

In \[13]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-2.unsigned \
  --required-signer "${ROLE_SKEY[c.marlowe]}" \
  --timeout 600
```

```
TxId "4b8528698e34e16837649ef68769653196985180e1a71dc874d2fe8a53a8942a"
```

In \[14]:

```
echo "https://cardanoscan.io/transaction/$TX_2?tab=utxo"
```

```
https://cardanoscan.io/transaction/4b8528698e34e16837649ef68769653196985180e1a71dc874d2fe8a53a8942a?tab=utxo
```

Ta có thể thấy output trong hợp đồng Marlowe đã có 1 token BearGarden.

In \[15]:

```
cardano-cli query utxo --mainnet --tx-in "$TX_2#1"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
4b8528698e34e16837649ef68769653196985180e1a71dc874d2fe8a53a8942a     1        3000000 lovelace + 1 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d.4265617247617264656e + TxOutDatumHash ScriptDataInBabbageEra "d741f0d8c5e7ee85b174cb8178e539a56efc22d6f5884d86c1532592e648a0dd"
```

### Giao dịch số 3: Jane Lumley gửi 50 Djed vào hợp đồng, dẫn đến việc hợp đồng thanh toán cho các bên.

&#x20;Việc gửi 50 Djed khiến hợp đồng đóng lại vì nó thanh toán 1 BearGarden cho địa chỉ thanh toán của Marlowe để phục vụ cho lợi ích của Jane Lumley và 50 Djed cho lợi ích của Christopher Marlowe.

In \[16]:

```
TX_3=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[j.lumley]}" \
  --to-party "${ROLE_NAME[c.marlowe]}" \
  --currency "$DJED_POLICY" \
  --token-name "$DJED_NAME" \
  --quantity "$((50 * DJED))" \
  --change-address "${ROLE_ADDR[j.lumley]}" \
  --manual-sign tx-3.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_3 = $TX_3"
```

```
TX_3 = 8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887
```

Ký và gửi giao dịch.

In \[17]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-3.unsigned \
  --required-signer "${ROLE_SKEY[j.lumley]}" \
  --timeout 600
```

```
TxId "8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887"
```

In \[18]:

```
echo "https://cardanoscan.io/transaction/$TX_3?tab=utxo"
```

```
https://cardanoscan.io/transaction/8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887?tab=utxo
```

Ta thấy Chrisopher Marlowe vẫn đang giữ token vai trò của anh ấy.

In \[19]:

```
cardano-cli query utxo --mainnet --address "${ROLE_ADDR[c.marlowe]}"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
4b8528698e34e16837649ef68769653196985180e1a71dc874d2fe8a53a8942a     0        135613986 lovelace + TxOutDatumNone
4b8528698e34e16837649ef68769653196985180e1a71dc874d2fe8a53a8942a     2        5000000 lovelace + 1 f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a.632e6d61726c6f7765 + TxOutDatumNone
4b8528698e34e16837649ef68769653196985180e1a71dc874d2fe8a53a8942a     3        1180940 lovelace + 499 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d.4265617247617264656e + TxOutDatumNone
ca16af8573a2b844b8a3ea8d8299da37d6c63d83582a8f54eac927198884aae6     0        12201100 lovelace + TxOutDatumNone
```

Ta thấy Jane Lumley vẫn đang giữ token vai trò của cô ấy.

In \[20]:

```
cardano-cli query utxo --mainnet --address "${ROLE_ADDR[j.lumley]}"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
855c131e9b59a6607b4d28cfcb3a68b7f3f9672d03805c9252a6515f6e4ab226     2        3000000 lovelace + 100000000 f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880.69555344 + TxOutDatumNone
8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887     0        38850860 lovelace + TxOutDatumNone
8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887     1        5000000 lovelace + 1 f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a.6a2e6c756d6c6579 + TxOutDatumNone
8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887     5        1198180 lovelace + 50000000 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61.446a65644d6963726f555344 + TxOutDatumNone
```

Ta thấy địa chỉ thanh toán Marlowe's vẫn đang giữ 50 Djed và 1 BearGarden.

In \[21]:

```
cardano-cli query utxo --mainnet --tx-in "$TX_3#2" --tx-in "$TX_3#3"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887     2        1236970 lovelace + 50000000 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61.446a65644d6963726f555344 + TxOutDatumHash ScriptDataInBabbageEra "ea0afe598fe417d62bee6191c1838aaadb7d7aaa2849fe9bff19abeb5233199b"
8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887     3        1211110 lovelace + 1 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d.4265617247617264656e + TxOutDatumHash ScriptDataInBabbageEra "9a0959b845f659fd39e6be40797247b4b1f2a1af78710d69adc9eb48f983a789"
```

### Giao dịch số 4: Christopher Marlowe rút 50 Djed của mình từ địa chỉ thanh toán của vai trò.

In \[22]:

```
TX_4=$(
marlowe withdraw \
  --contract "$CONTRACT_ID" \
  --role "${ROLE_NAME[c.marlowe]}" \
  --change-address "${ROLE_ADDR[c.marlowe]}" \
  --manual-sign tx-4.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_4 = $TX_4"
```

```
TX_4 = 2dd9c58f3c30274dbe570e8ff5a193d819b67f133b0a225bf2f9212514ab49fc
```

Ký và gửi giao dịch.

In \[23]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-4.unsigned \
  --required-signer "${ROLE_SKEY[c.marlowe]}" \
  --timeout 600
```

```
TxId "2dd9c58f3c30274dbe570e8ff5a193d819b67f133b0a225bf2f9212514ab49fc"
```

Ta thấy Christopher Marlowe đã rút thành công 50 Djed từ địa chỉ vai trò.

In \[24]:

```
echo "https://cardanoscan.io/transaction/$TX_4?tab=utxo"
```

```
https://cardanoscan.io/transaction/2dd9c58f3c30274dbe570e8ff5a193d819b67f133b0a225bf2f9212514ab49fc?tab=utxo
```

### Giao dịch số 5: Jane Lumley rút 1 BearGarden của mình từ địa chỉ thanh toán của vai trò.

In \[25]:

```
TX_5=$(
marlowe withdraw \
  --contract "$CONTRACT_ID" \
  --role "${ROLE_NAME[j.lumley]}" \
  --change-address "${ROLE_ADDR[j.lumley]}" \
  --manual-sign tx-5.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_5 = $TX_5"
```

```
TX_5 = 7201d7fa362aea4ae0af212b9ebfddb71bceb1e1ac36f5cd9fde5fc3bbde16ef
```

Ký và gửi giao dịch.

In \[26]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-5.unsigned \
  --required-signer "${ROLE_SKEY[j.lumley]}" \
  --timeout 600
```

```
TxId "7201d7fa362aea4ae0af212b9ebfddb71bceb1e1ac36f5cd9fde5fc3bbde16ef"
```

Ta thấy Jane Lumley đã rút thành công 1 BearGarden từ địa chỉ vai trò.

In \[27]:

```
echo "https://cardanoscan.io/transaction/$TX_5?tab=utxo"
```

```
https://cardanoscan.io/transaction/7201d7fa362aea4ae0af212b9ebfddb71bceb1e1ac36f5cd9fde5fc3bbde16ef?tab=utxo
```

### Xem lại toàn bộ hợp đồng <a href="#view-the-whole-history-of-the-contract" id="view-the-whole-history-of-the-contract"></a>

Chúng ta sử dụng câu lệnh `marlowe-pipe` để in toàn bộ lịch sử hợp đồng.

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
              role_token: c.marlowe
            of_token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
            party:
              role_token: c.marlowe
          then:
            timeout: 1676679840000
            timeout_continuation: close
            when:
            - case:
                deposits: 50000000
                into_account:
                  role_token: c.marlowe
                of_token:
                  currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
                  token_name: DjedMicroUSD
                party:
                  role_token: j.lumley
              then:
                from_account:
                  role_token: c.marlowe
                pay: 1
                then: close
                to:
                  party:
                    role_token: j.lumley
                token:
                  currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                  token_name: BearGarden
            - case:
                deposits: 50000000
                into_account:
                  role_token: c.marlowe
                of_token:
                  currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                  token_name: iUSD
                party:
                  role_token: j.lumley
              then:
                from_account:
                  role_token: c.marlowe
                pay: 1
                then: close
                to:
                  party:
                    role_token: j.lumley
                token:
                  currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                  token_name: BearGarden
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
      txId: 561c6ccdeba9505a385c92f3c3b9c3b007c92c34bbffef8f9a88f02eba1bd4de
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps:
- contractId: 561c6ccdeba9505a385c92f3c3b9c3b007c92c34bbffef8f9a88f02eba1bd4de#1
  payouts: []
  redeemer:
  - input_from_party:
      role_token: c.marlowe
    into_account:
      role_token: c.marlowe
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
            deposits: 50000000
            into_account:
              role_token: c.marlowe
            of_token:
              currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
              token_name: DjedMicroUSD
            party:
              role_token: j.lumley
          then:
            from_account:
              role_token: c.marlowe
            pay: 1
            then: close
            to:
              party:
                role_token: j.lumley
            token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
        - case:
            deposits: 50000000
            into_account:
              role_token: c.marlowe
            of_token:
              currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
              token_name: iUSD
            party:
              role_token: j.lumley
          then:
            from_account:
              role_token: c.marlowe
            pay: 1
            then: close
            to:
              party:
                role_token: j.lumley
            token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
      marloweParams:
        rolesCurrency: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
      marloweState:
        accounts:
        - - - address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
            - currency_symbol: ''
              token_name: ''
          - 3000000
        - - - role_token: c.marlowe
            - currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
          - 1
        boundValues: []
        choices: []
        minTime: 1676324461000
    utxo:
      txId: 4b8528698e34e16837649ef68769653196985180e1a71dc874d2fe8a53a8942a
      txIx: 1
  step: apply
  txId: 4b8528698e34e16837649ef68769653196985180e1a71dc874d2fe8a53a8942a
- contractId: 561c6ccdeba9505a385c92f3c3b9c3b007c92c34bbffef8f9a88f02eba1bd4de#1
  payouts:
  - - txId: 8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887
      txIx: 2
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 1236970
        tokens:
        - - policyId: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
            tokenName: DjedMicroUSD
          - 50000000
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: c.marlowe
  - - txId: 8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887
      txIx: 3
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 1211110
        tokens:
        - - policyId: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
            tokenName: BearGarden
          - 1
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: j.lumley
  redeemer:
  - input_from_party:
      role_token: j.lumley
    into_account:
      role_token: c.marlowe
    of_token:
      currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
      token_name: DjedMicroUSD
    that_deposits: 50000000
  scriptOutput: null
  step: apply
  txId: 8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: c.marlowe
  redeemingTx: 2dd9c58f3c30274dbe570e8ff5a193d819b67f133b0a225bf2f9212514ab49fc
  step: payout
  utxo:
    txId: 8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887
    txIx: 2
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: j.lumley
  redeemingTx: 7201d7fa362aea4ae0af212b9ebfddb71bceb1e1ac36f5cd9fde5fc3bbde16ef
  step: payout
  utxo:
    txId: 8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887
    txIx: 3
```

### Gửi trả token BearGarden về faucet <a href="#return-the-beargarden-token-to-the-faucet" id="return-the-beargarden-token-to-the-faucet"></a>

Việc trả lại các token cho faucet là một cách tiện lợi để **dọn dẹp** cho ví dụ này.

In \[29]:

```
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "561c6ccdeba9505a385c92f3c3b9c3b007c92c34bbffef8f9a88f02eba1bd4de#0" \
  --tx-in "855c131e9b59a6607b4d28cfcb3a68b7f3f9672d03805c9252a6515f6e4ab226#0" \
  --tx-in "8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887#4" \
  --tx-in "2dd9c58f3c30274dbe570e8ff5a193d819b67f133b0a225bf2f9212514ab49fc#2" \
  --tx-in "4b8528698e34e16837649ef68769653196985180e1a71dc874d2fe8a53a8942a#3" \
  --tx-in "7201d7fa362aea4ae0af212b9ebfddb71bceb1e1ac36f5cd9fde5fc3bbde16ef#2" \
  --tx-in "855c131e9b59a6607b4d28cfcb3a68b7f3f9672d03805c9252a6515f6e4ab226#2" \
  --tx-in "8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887#5" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+$((100 * DJED)) $DJED_POLICY.$DJED_NAME" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+$((100 * IUSD)) $IUSD_POLICY.$IUSD_NAME" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+500 $FUNGIBLES_POLICY.BearGarden" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "$FAUCET_SKEY" \
  --required-signer "${ROLE_SKEY[c.marlowe]}" \
  --required-signer "${ROLE_SKEY[j.lumley]}" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "a54cdaeaa68371466a6ddd0a8b20c092b9473bf5840e00f9e3dbcbd0c0967e2a"
```
