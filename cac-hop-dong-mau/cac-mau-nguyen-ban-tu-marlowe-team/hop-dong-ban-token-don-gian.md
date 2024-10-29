# Hợp đồng bán token đơn giản

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

### Hợp đồng bán token đơn giản

Giao dịch token đơn giản nhất là sự trao đổi giữa một token và Ada.&#x20;

Ví dụ này bao gồm năm giao dịch:

1. Christopher Marlowe tạo hợp đồng Marlowe cho việc bán token.
2. Christopher Marlowe gửi 1 token BearGarden vào hợp đồng.
3. Jane Lumley gửi 50 Ada vào hợp đồng, khiến hợp đồng trả token cho cô ấy và Ada cho Christopher Marlowe.
4. Christopher Marlowe rút 50 Ada từ địa chỉ trả thưởng của Marlowe.
5. Jane Lumley rút 1 BearGarden từ địa chỉ trả thưởng của Marlowe.

Dưới đây là hợp đồng ở định dạng Blockly:

![Hợp đồng bán token đơn giản](https://raw.githubusercontent.com/input-output-hk/real-world-marlowe/a288a9f391e68135e59e65a813db695e6223472d/nfts/simple/contract.png)

### Thiết lập <a href="#set-up" id="set-up"></a>

Sử dụng `mainnet`.

In \[1]:

```
. ../../mainnet.env
```

Use the standard example roles.

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

Chúng tôi đã trước đó phát hành token BearGarden với chính sách sau đây. In \[4]:

```
echo "FUNGIBLES_POLICY = $FUNGIBLES_POLICY"
```

```
FUNGIBLES_POLICY = 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
```

### Initial funding <a href="#initial-funding" id="initial-funding"></a>

Gửi token BearGarden từ faucet tới Christopher Marlowe.

In \[5]:

```
ADA=1000000
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "332f37fbcd87aaf56dd3596ec7dfb8567dfac3e5d5d344553fdbaf3977d1560d#0" \
  --tx-in "332f37fbcd87aaf56dd3596ec7dfb8567dfac3e5d5d344553fdbaf3977d1560d#3" \
  --tx-out "${ROLE_ADDR[c.marlowe]}+$((3 * ADA))+1 $FUNGIBLES_POLICY.BearGarden" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+499 $FUNGIBLES_POLICY.BearGarden" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "$FAUCET_SKEY" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "c2970d098870bbf9cd8d7f94fba69a7dd1f08171b580fc53a1e7aae57069311a"
```

### Hợp đồng Marlowe <a href="#the-marlowe-contract" id="the-marlowe-contract"></a>

Hợp đồng Marlowe thực tế chỉ cần download file JSON được biên dịch từ hợp đồng này dạng Blockly đã được thiết kế sẵn trong [Marlowe Playground](https://play.marlowe.iohk.io/#/).

In \[6]:

```
json2yaml contract.json
```

```bash
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
          currency_symbol: ''
          token_name: ''
        party:
          role_token: j.lumley
      then:
        from_account:
          role_token: c.marlowe
        pay: 1
        then:
          from_account:
            role_token: c.marlowe
          pay: 50000000
          then: close
          to:
            party:
              role_token: c.marlowe
          token:
            currency_symbol: ''
            token_name: ''
        to:
          party:
            role_token: j.lumley
        token:
          currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
          token_name: BearGarden
```

### Giao dịch số 1: Tạo hợp đồng <a href="#transaction-1.-create-the-contract" id="transaction-1.-create-the-contract"></a>

Chúng ta sử dụng công cụ _Marlowe Runtime's command-line_ để xây dựng giao dịch tạo hợp đồng.

```bash
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
CONTRACT_ID = 20dbec7afca02f0ac99fbd00a050a660b24cfc6998ec22529d50ba5c2e60e9f1#1
```

Hợp đồng có thể được ký và gửi bằng bất kỳ ví hoặc dịch vụ nào. Để thuận tiện, chúng ta sử dụng `marlowe-cli`.

```bash
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-1.unsigned \
  --required-signer "$FAUCET_SKEY" \
  --timeout 600
```

```
TxId "20dbec7afca02f0ac99fbd00a050a660b24cfc6998ec22529d50ba5c2e60e9f1"
```

Xem hợp đồng qua trình khám phá dữ liệu blockchain Cardano (ví dụ: Cardanoscan.io).

```
echo "https://cardanoscan.io/transaction/${CONTRACT_ID%%#1}?tab=utxo"
```

```
https://cardanoscan.io/transaction/20dbec7afca02f0ac99fbd00a050a660b24cfc6998ec22529d50ba5c2e60e9f1?tab=utxo
```

Chúng ta cũng có thể sử dụng công cụ như `marlowe-pipe` để truy xuất hợp đồng từ blockchain và hiển thị nó.

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
                  currency_symbol: ''
                  token_name: ''
                party:
                  role_token: j.lumley
              then:
                from_account:
                  role_token: c.marlowe
                pay: 1
                then:
                  from_account:
                    role_token: c.marlowe
                  pay: 50000000
                  then: close
                  to:
                    party:
                      role_token: c.marlowe
                  token:
                    currency_symbol: ''
                    token_name: ''
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
      txId: 20dbec7afca02f0ac99fbd00a050a660b24cfc6998ec22529d50ba5c2e60e9f1
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps: []
```

### Giao dịch số 2: Christopher Marlowe gửi token BearGarden vào hợp đồng.

Logic của hợp đồng quy định rằng Christopher Marlowe sẽ gửi một token BearGarden vào tài khoản của mình trong hợp đồng Marlowe.

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
TX_2 = ca16af8573a2b844b8a3ea8d8299da37d6c63d83582a8f54eac927198884aae6
```

Ký và gửi giao dịch

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-2.unsigned \
  --required-signer "${ROLE_SKEY[c.marlowe]}" \
  --timeout 600
```

```
TxId "ca16af8573a2b844b8a3ea8d8299da37d6c63d83582a8f54eac927198884aae6"
```

Kiểm tra xem token đã được chuyển vào hợp đồng.

```
echo "https://cardanoscan.io/transaction/$TX_2?tab=utxo"
```

```
https://cardanoscan.io/transaction/ca16af8573a2b844b8a3ea8d8299da37d6c63d83582a8f54eac927198884aae6?tab=utxo
```

Ta có thể thấy output trong hợp đồng Marlowe đã có 1 token BearGarden.

```
cardano-cli query utxo --mainnet --tx-in "$TX_2#1"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
ca16af8573a2b844b8a3ea8d8299da37d6c63d83582a8f54eac927198884aae6     1        3000000 lovelace + 1 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d.4265617247617264656e + TxOutDatumHash ScriptDataInBabbageEra "ea15d2cfbca32664eb64dee4607d2e82996834480dc2f32d5d69fddf3d6d6b61"
```

### Giao dịch số 3: Jane Lumley gửi 50 Ada vào hợp đồng, dẫn đến việc hợp đồng thanh toán cho các bên.

&#x20;Việc gửi 50 Ada khiến hợp đồng đóng lại vì nó thanh toán 1 BearGarden cho địa chỉ thanh toán của Marlowe để phục vụ cho lợi ích của Jane Lumley và 50 Ada cho lợi ích của Christopher Marlowe.

```
TX_3=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[j.lumley]}" \
  --to-party "${ROLE_NAME[c.marlowe]}" \
  --lovelace "$((50 * ADA))" \
  --change-address "${ROLE_ADDR[j.lumley]}" \
  --manual-sign tx-3.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_3 = $TX_3"
```

```
TX_3 = c1246f0f0d47804be9ee4c0439065c438a5c03c852781a1c3e390a0683697885
```

Ký và gửi giao dịch.

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-3.unsigned \
  --required-signer "${ROLE_SKEY[j.lumley]}" \
  --timeout 600
```

```
TxId "c1246f0f0d47804be9ee4c0439065c438a5c03c852781a1c3e390a0683697885"
```

Kiểm tra xem hợp đồng đã được đóng thông qua việc thanh toán cho các bên liên quan.

```
echo "https://cardanoscan.io/transaction/$TX_3?tab=utxo"
```

```
https://cardanoscan.io/transaction/c1246f0f0d47804be9ee4c0439065c438a5c03c852781a1c3e390a0683697885?tab=utxo
```

Ta thấy Chrisopher Marlowe vẫn đang giữ token vai trò của anh ấy

```
cardano-cli query utxo --mainnet --address "${ROLE_ADDR[c.marlowe]}"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
4e047764afd5f346c8ac13bc676e1f15f5ca1e0dae0b66a0ab4f8f6976f6a923     7        85000000 lovelace + TxOutDatumNone
ca16af8573a2b844b8a3ea8d8299da37d6c63d83582a8f54eac927198884aae6     0        12201100 lovelace + TxOutDatumNone
ca16af8573a2b844b8a3ea8d8299da37d6c63d83582a8f54eac927198884aae6     2        5000000 lovelace + 1 f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a.632e6d61726c6f7765 + TxOutDatumNone
```

Ta thấy Jane Lumley vẫn đang giữ token vai trò của cô ấy.

```
cardano-cli query utxo --mainnet --address "${ROLE_ADDR[j.lumley]}"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
3afe677d9749efe356292a140b8c74dfe0f1ad6f6db385b638ec5ea99b3c7a4a     3        10000000 lovelace + TxOutDatumNone
c1246f0f0d47804be9ee4c0439065c438a5c03c852781a1c3e390a0683697885     0        32897476 lovelace + TxOutDatumNone
c1246f0f0d47804be9ee4c0439065c438a5c03c852781a1c3e390a0683697885     1        5000000 lovelace + 1 f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a.6a2e6c756d6c6579 + TxOutDatumNone
```

Ta thấy địa chỉ thanh toán Marlowe's vẫn đang giữ 50 Ada và 1 BearGarden.

```
cardano-cli query utxo --mainnet --tx-in "$TX_3#2" --tx-in "$TX_3#3"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
c1246f0f0d47804be9ee4c0439065c438a5c03c852781a1c3e390a0683697885     2        50000000 lovelace + TxOutDatumHash ScriptDataInBabbageEra "ea0afe598fe417d62bee6191c1838aaadb7d7aaa2849fe9bff19abeb5233199b"
c1246f0f0d47804be9ee4c0439065c438a5c03c852781a1c3e390a0683697885     3        1211110 lovelace + 1 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d.4265617247617264656e + TxOutDatumHash ScriptDataInBabbageEra "9a0959b845f659fd39e6be40797247b4b1f2a1af78710d69adc9eb48f983a789"
```

### Giao dịch số 4: Christopher Marlowe rút 50 Ada của mình từ địa chỉ thanh toán của vai trò.

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
TX_4 = 1c455a1dd18468c20f2d502186f6a99cfbf1677a1614bf0e1ec40a514e3dafe3
```

Ký và gửi giao dịch.

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-4.unsigned \
  --required-signer "${ROLE_SKEY[c.marlowe]}" \
  --timeout 600
```

```
TxId "1c455a1dd18468c20f2d502186f6a99cfbf1677a1614bf0e1ec40a514e3dafe3"
```

Ta thấy Christopher Marlowe đã rút thành công 50 Ada từ địa chỉ vai trò

```
echo "https://cardanoscan.io/transaction/$TX_4?tab=utxo"
```

```
https://cardanoscan.io/transaction/1c455a1dd18468c20f2d502186f6a99cfbf1677a1614bf0e1ec40a514e3dafe3?tab=utxo
```

### Giao dịch số 5: Jane Lumley rút 1 BearGarden của mình từ địa chỉ thanh toán của vai trò.

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
TX_5 = 60ca9f7a9ed5248d833e9a8d24a78136ee73fa3004b29fc5f20bf0a87303c64b
```

Ký và gửi giao dịch

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-5.unsigned \
  --required-signer "${ROLE_SKEY[j.lumley]}" \
  --timeout 600
```

```
TxId "60ca9f7a9ed5248d833e9a8d24a78136ee73fa3004b29fc5f20bf0a87303c64b"
```

Ta thấy Jane Lumley đã rút thành công 1 BearGarden từ địa chỉ vai trò

```
echo "https://cardanoscan.io/transaction/$TX_5?tab=utxo"
```

```
https://cardanoscan.io/transaction/60ca9f7a9ed5248d833e9a8d24a78136ee73fa3004b29fc5f20bf0a87303c64b?tab=utxo
```

### Xem lại toàn bộ hợp đồng <a href="#view-the-whole-history-of-the-contract" id="view-the-whole-history-of-the-contract"></a>

Chúng ta sử dụng câu lệnh `marlowe-pipe` để in toàn bộ lịch sử hợp đồng.

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
                  currency_symbol: ''
                  token_name: ''
                party:
                  role_token: j.lumley
              then:
                from_account:
                  role_token: c.marlowe
                pay: 1
                then:
                  from_account:
                    role_token: c.marlowe
                  pay: 50000000
                  then: close
                  to:
                    party:
                      role_token: c.marlowe
                  token:
                    currency_symbol: ''
                    token_name: ''
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
      txId: 20dbec7afca02f0ac99fbd00a050a660b24cfc6998ec22529d50ba5c2e60e9f1
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps:
- contractId: 20dbec7afca02f0ac99fbd00a050a660b24cfc6998ec22529d50ba5c2e60e9f1#1
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
              currency_symbol: ''
              token_name: ''
            party:
              role_token: j.lumley
          then:
            from_account:
              role_token: c.marlowe
            pay: 1
            then:
              from_account:
                role_token: c.marlowe
              pay: 50000000
              then: close
              to:
                party:
                  role_token: c.marlowe
              token:
                currency_symbol: ''
                token_name: ''
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
        minTime: 1676250375000
    utxo:
      txId: ca16af8573a2b844b8a3ea8d8299da37d6c63d83582a8f54eac927198884aae6
      txIx: 1
  step: apply
  txId: ca16af8573a2b844b8a3ea8d8299da37d6c63d83582a8f54eac927198884aae6
- contractId: 20dbec7afca02f0ac99fbd00a050a660b24cfc6998ec22529d50ba5c2e60e9f1#1
  payouts:
  - - txId: c1246f0f0d47804be9ee4c0439065c438a5c03c852781a1c3e390a0683697885
      txIx: 2
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 50000000
        tokens: []
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: c.marlowe
  - - txId: c1246f0f0d47804be9ee4c0439065c438a5c03c852781a1c3e390a0683697885
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
      currency_symbol: ''
      token_name: ''
    that_deposits: 50000000
  scriptOutput: null
  step: apply
  txId: c1246f0f0d47804be9ee4c0439065c438a5c03c852781a1c3e390a0683697885
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: c.marlowe
  redeemingTx: 1c455a1dd18468c20f2d502186f6a99cfbf1677a1614bf0e1ec40a514e3dafe3
  step: payout
  utxo:
    txId: c1246f0f0d47804be9ee4c0439065c438a5c03c852781a1c3e390a0683697885
    txIx: 2
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: j.lumley
  redeemingTx: 60ca9f7a9ed5248d833e9a8d24a78136ee73fa3004b29fc5f20bf0a87303c64b
  step: payout
  utxo:
    txId: c1246f0f0d47804be9ee4c0439065c438a5c03c852781a1c3e390a0683697885
    txIx: 3
```

### Gửi trả token BearGarden về faucet <a href="#return-the-beargarden-token-to-the-faucet" id="return-the-beargarden-token-to-the-faucet"></a>

Việc trả lại token cho faucet là một cách tiện lợi để **dọn dẹp** cho ví dụ này.

In \[28]:

```
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "60ca9f7a9ed5248d833e9a8d24a78136ee73fa3004b29fc5f20bf0a87303c64b#0" \
  --tx-in "60ca9f7a9ed5248d833e9a8d24a78136ee73fa3004b29fc5f20bf0a87303c64b#2" \
  --tx-in "c1246f0f0d47804be9ee4c0439065c438a5c03c852781a1c3e390a0683697885#0" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+1 $FUNGIBLES_POLICY.BearGarden" \
  --change-address "${ROLE_ADDR[j.lumley]}" \
  --required-signer "${ROLE_SKEY[j.lumley]}" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "3ade722a48e57a6d59b310ed520df6f90114481712ec313cbd24cd0b60ecefc1"
```
