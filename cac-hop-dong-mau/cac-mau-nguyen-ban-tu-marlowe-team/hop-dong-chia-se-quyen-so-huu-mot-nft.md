# Hợp đồng chia sẻ quyền sở hữu một NFT

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

### Mua chung một NFT, với tùy chọn mua lại phần sở hữu của các đồng sở hữu khác.

Ba bên cùng góp một khoản tiền bằng nhau để mua một NFT mà họ sở hữu chung. Vào bất kỳ thời điểm nào, một trong số họ có thể trả cho hai bên còn lại giá trị của token để họ có thể sở hữu token đó hoàn toàn.

Ví dụ này bao gồm mười giao dịch:

1. Elizabeth Cary gửi 100 Ada vào hợp đồng.
2. Jane Lumley gửi 100 Ada vào hợp đồng.
3. Mary Herbert gửi 100 Ada vào hợp đồng.
4. Christopher Marlowe gửi 1 token BearGarden vào hợp đồng, dẫn đến việc hợp đồng thanh toán 300 Ada cho lợi ích của anh ấy.
5. Christopher Marlowe rút 300 Ada từ địa chỉ thanh toán của vai trò.
6. Sau đó, Mary Herbert gửi 300 Ada vào hợp đồng để cô ấy có thể sở hữu token hoàn toàn; việc này dẫn đến việc 150 Ada mỗi người được thanh toán cho Elizabeth Cary và Jane Lumley.
7. Elizabeth Cary rút 150 Ada từ địa chỉ thanh toán của Marlowe.
8. Jane Lumley rút 150 Ada từ địa chỉ thanh toán của Marlowe.
9. Mary Herbert rút 1 token BearGarden từ địa chỉ thanh toán của Marlowe.

Dưới đây là hợp đồng ở định dạng Blockly:

![Shared ownership of an NFT](https://raw.githubusercontent.com/input-output-hk/real-world-marlowe/a288a9f391e68135e59e65a813db695e6223472d/nfts/shared/contract.png)

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

* Christopher Marlowe = [$c.marlowe](https://pool.pm/asset1z2xzfc6lu63jfmfffe2w3nyf6420eylv8e2xjp)
* Elizabeth Carey = [$e.cary](https://pool.pm/asset1tx4euajkdczmkawgkjy342agaq33885dlvp0jl)
* Mary Herbert = [$m.herbert](https://pool.pm/asset1a38nhu84xquj7whe3xqr80uyf99mh2r7hzf277)
* Jane Lumley = [$j.lumley](https://pool.pm/asset1kujmmryzmxyqz6utp2slrmwfq4dmmnvwhkh7gkm)

_Lưu ý: Chỉ sử dụng token đã được phát hành trước làm token vai trò Marlowe nếu bạn đã xem xét chính sách tiền tệ để tìm kiếm các lỗ hổng bảo mật._

Dưới đây là ký hiệu tiền tệ cho Ada handles trên mainnet:

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

### Initial funding <a href="#initial-funding" id="initial-funding"></a>

Gửi token BearGarden từ faucet tới Christopher Marlowe.

In \[5]:

```
cardano-cli query utxo --mainnet --address $FAUCET_ADDR
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
0afae60d6282c0f4d2f91eb3046b2e7c22489b70c97e4c18b92c0968403be754     1        3000000 lovelace + 100000000 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61.446a65644d6963726f555344 + TxOutDatumNone
a1e8562663b89e7d793470a2773ea721f59819ca5028d76a012fdf638338214f     1        3000000 lovelace + 100000000 f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880.69555344 + TxOutDatumNone
a1e8562663b89e7d793470a2773ea721f59819ca5028d76a012fdf638338214f     2        3000000 lovelace + 499 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d.4265617247617264656e + TxOutDatumNone
efc74dd7dfe0f033e17c26e93ab2db01b356f6642e4b0eb66ebd353771e98ff4     0        312096021 lovelace + TxOutDatumNone
efc74dd7dfe0f033e17c26e93ab2db01b356f6642e4b0eb66ebd353771e98ff4     1        3000000 lovelace + 500 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d.476c6f6265 + TxOutDatumNone
efc74dd7dfe0f033e17c26e93ab2db01b356f6642e4b0eb66ebd353771e98ff4     2        3000000 lovelace + 500 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d.5377616e + TxOutDatumNone
```

In \[6]:

```
ADA=1000000
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "a1e8562663b89e7d793470a2773ea721f59819ca5028d76a012fdf638338214f#2" \
  --tx-in "efc74dd7dfe0f033e17c26e93ab2db01b356f6642e4b0eb66ebd353771e98ff4#0" \
  --tx-out "${ROLE_ADDR[c.marlowe]}+$((2 * ADA))+1 $FUNGIBLES_POLICY.BearGarden" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "$FAUCET_SKEY" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "a84c33bce6722f805120252b764e6aed68d1e6f4702dadca49967db2456d5107"
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
    deposits: 100000000
    into_account:
      role_token: e.cary
    of_token:
      currency_symbol: ''
      token_name: ''
    party:
      role_token: e.cary
  then:
    timeout: 1676679840000
    timeout_continuation: close
    when:
    - case:
        deposits: 100000000
        into_account:
          role_token: j.lumley
        of_token:
          currency_symbol: ''
          token_name: ''
        party:
          role_token: j.lumley
      then:
        timeout: 1676679850000
        timeout_continuation: close
        when:
        - case:
            deposits: 100000000
            into_account:
              role_token: m.herbert
            of_token:
              currency_symbol: ''
              token_name: ''
            party:
              role_token: m.herbert
          then:
            timeout: 1676679860000
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
                  role_token: c.marlowe
              then:
                from_account:
                  role_token: e.cary
                pay: 100000000
                then:
                  from_account:
                    role_token: m.herbert
                  pay: 100000000
                  then:
                    from_account:
                      role_token: j.lumley
                    pay: 100000000
                    then:
                      timeout: 1676679870000
                      timeout_continuation: close
                      when:
                      - case:
                          deposits: 300000000
                          into_account:
                            role_token: e.cary
                          of_token:
                            currency_symbol: ''
                            token_name: ''
                          party:
                            role_token: e.cary
                        then:
                          from_account:
                            role_token: e.cary
                          pay: 1
                          then:
                            from_account:
                              role_token: e.cary
                            pay: 150000000
                            then:
                              from_account:
                                role_token: e.cary
                              pay: 150000000
                              then: close
                              to:
                                party:
                                  role_token: j.lumley
                              token:
                                currency_symbol: ''
                                token_name: ''
                            to:
                              party:
                                role_token: m.herbert
                            token:
                              currency_symbol: ''
                              token_name: ''
                          to:
                            party:
                              role_token: e.cary
                          token:
                            currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                            token_name: BearGarden
                      - case:
                          deposits: 300000000
                          into_account:
                            role_token: m.herbert
                          of_token:
                            currency_symbol: ''
                            token_name: ''
                          party:
                            role_token: m.herbert
                        then:
                          from_account:
                            role_token: m.herbert
                          pay: 150000000
                          then:
                            from_account:
                              role_token: e.cary
                            pay: 1
                            then:
                              from_account:
                                role_token: m.herbert
                              pay: 150000000
                              then: close
                              to:
                                party:
                                  role_token: j.lumley
                              token:
                                currency_symbol: ''
                                token_name: ''
                            to:
                              party:
                                role_token: m.herbert
                            token:
                              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                              token_name: BearGarden
                          to:
                            party:
                              role_token: e.cary
                          token:
                            currency_symbol: ''
                            token_name: ''
                      - case:
                          deposits: 300000000
                          into_account:
                            role_token: j.lumley
                          of_token:
                            currency_symbol: ''
                            token_name: ''
                          party:
                            role_token: j.lumley
                        then:
                          from_account:
                            role_token: j.lumley
                          pay: 150000000
                          then:
                            from_account:
                              role_token: j.lumley
                            pay: 150000000
                            then:
                              from_account:
                                role_token: e.cary
                              pay: 1
                              then: close
                              to:
                                party:
                                  role_token: j.lumley
                              token:
                                currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                                token_name: BearGarden
                            to:
                              party:
                                role_token: m.herbert
                            token:
                              currency_symbol: ''
                              token_name: ''
                          to:
                            party:
                              role_token: e.cary
                          token:
                            currency_symbol: ''
                            token_name: ''
                    to:
                      party:
                        role_token: c.marlowe
                    token:
                      currency_symbol: ''
                      token_name: ''
                  to:
                    party:
                      role_token: c.marlowe
                  token:
                    currency_symbol: ''
                    token_name: ''
                to:
                  party:
                    role_token: c.marlowe
                token:
                  currency_symbol: ''
                  token_name: ''
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
CONTRACT_ID = e9b4608d23b955783bce458c02687353e9497c30980ae8ad941256a74840cd5b#1
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
TxId "e9b4608d23b955783bce458c02687353e9497c30980ae8ad941256a74840cd5b"
```

Xem hợp đồng qua trình khám phá dữ liệu blockchain Cardano (ví dụ: Cardanoscan.io).

In \[10]:

```
echo "https://cardanoscan.io/transaction/${CONTRACT_ID%%#1}?tab=utxo"
```

```
https://cardanoscan.io/transaction/e9b4608d23b955783bce458c02687353e9497c30980ae8ad941256a74840cd5b?tab=utxo
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
            deposits: 100000000
            into_account:
              role_token: e.cary
            of_token:
              currency_symbol: ''
              token_name: ''
            party:
              role_token: e.cary
          then:
            timeout: 1676679840000
            timeout_continuation: close
            when:
            - case:
                deposits: 100000000
                into_account:
                  role_token: j.lumley
                of_token:
                  currency_symbol: ''
                  token_name: ''
                party:
                  role_token: j.lumley
              then:
                timeout: 1676679850000
                timeout_continuation: close
                when:
                - case:
                    deposits: 100000000
                    into_account:
                      role_token: m.herbert
                    of_token:
                      currency_symbol: ''
                      token_name: ''
                    party:
                      role_token: m.herbert
                  then:
                    timeout: 1676679860000
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
                          role_token: c.marlowe
                      then:
                        from_account:
                          role_token: e.cary
                        pay: 100000000
                        then:
                          from_account:
                            role_token: m.herbert
                          pay: 100000000
                          then:
                            from_account:
                              role_token: j.lumley
                            pay: 100000000
                            then:
                              timeout: 1676679870000
                              timeout_continuation: close
                              when:
                              - case:
                                  deposits: 300000000
                                  into_account:
                                    role_token: e.cary
                                  of_token:
                                    currency_symbol: ''
                                    token_name: ''
                                  party:
                                    role_token: e.cary
                                then:
                                  from_account:
                                    role_token: e.cary
                                  pay: 1
                                  then:
                                    from_account:
                                      role_token: e.cary
                                    pay: 150000000
                                    then:
                                      from_account:
                                        role_token: e.cary
                                      pay: 150000000
                                      then: close
                                      to:
                                        party:
                                          role_token: j.lumley
                                      token:
                                        currency_symbol: ''
                                        token_name: ''
                                    to:
                                      party:
                                        role_token: m.herbert
                                    token:
                                      currency_symbol: ''
                                      token_name: ''
                                  to:
                                    party:
                                      role_token: e.cary
                                  token:
                                    currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                                    token_name: BearGarden
                              - case:
                                  deposits: 300000000
                                  into_account:
                                    role_token: m.herbert
                                  of_token:
                                    currency_symbol: ''
                                    token_name: ''
                                  party:
                                    role_token: m.herbert
                                then:
                                  from_account:
                                    role_token: m.herbert
                                  pay: 150000000
                                  then:
                                    from_account:
                                      role_token: e.cary
                                    pay: 1
                                    then:
                                      from_account:
                                        role_token: m.herbert
                                      pay: 150000000
                                      then: close
                                      to:
                                        party:
                                          role_token: j.lumley
                                      token:
                                        currency_symbol: ''
                                        token_name: ''
                                    to:
                                      party:
                                        role_token: m.herbert
                                    token:
                                      currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                                      token_name: BearGarden
                                  to:
                                    party:
                                      role_token: e.cary
                                  token:
                                    currency_symbol: ''
                                    token_name: ''
                              - case:
                                  deposits: 300000000
                                  into_account:
                                    role_token: j.lumley
                                  of_token:
                                    currency_symbol: ''
                                    token_name: ''
                                  party:
                                    role_token: j.lumley
                                then:
                                  from_account:
                                    role_token: j.lumley
                                  pay: 150000000
                                  then:
                                    from_account:
                                      role_token: j.lumley
                                    pay: 150000000
                                    then:
                                      from_account:
                                        role_token: e.cary
                                      pay: 1
                                      then: close
                                      to:
                                        party:
                                          role_token: j.lumley
                                      token:
                                        currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                                        token_name: BearGarden
                                    to:
                                      party:
                                        role_token: m.herbert
                                    token:
                                      currency_symbol: ''
                                      token_name: ''
                                  to:
                                    party:
                                      role_token: e.cary
                                  token:
                                    currency_symbol: ''
                                    token_name: ''
                            to:
                              party:
                                role_token: c.marlowe
                            token:
                              currency_symbol: ''
                              token_name: ''
                          to:
                            party:
                              role_token: c.marlowe
                          token:
                            currency_symbol: ''
                            token_name: ''
                        to:
                          party:
                            role_token: c.marlowe
                        token:
                          currency_symbol: ''
                          token_name: ''
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
      txId: e9b4608d23b955783bce458c02687353e9497c30980ae8ad941256a74840cd5b
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps: []
```

### Giao dịch số 2: Elizabeth Cary nạp 100 Ada <a href="#transaction-2.-elizabeth-cary-deposits-100-ada" id="transaction-2.-elizabeth-cary-deposits-100-ada"></a>

In \[12]:

```
TX_2=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[e.cary]}" \
  --to-party "${ROLE_NAME[e.cary]}" \
  --lovelace "$((100 * ADA))" \
  --change-address "${ROLE_ADDR[e.cary]}" \
  --manual-sign tx-2.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_2 = $TX_2"
```

```
TX_2 = 72e4916c2cb6626af2102062a592f4ba8f2d2f649ea9e8934191e2f875fca204
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
TxId "72e4916c2cb6626af2102062a592f4ba8f2d2f649ea9e8934191e2f875fca204"
```

Kiểm tra xem hợp đồng đã nhận được 100 Ada?

In \[14]:

```
echo "https://cardanoscan.io/transaction/${TX_2}?tab=utxo"
```

```
https://cardanoscan.io/transaction/72e4916c2cb6626af2102062a592f4ba8f2d2f649ea9e8934191e2f875fca204?tab=utxo
```

### Giao dịch số 3: Jane Lumley nạp 100 Ada <a href="#transaction-3.-jane-lumley-deposits-100-ada" id="transaction-3.-jane-lumley-deposits-100-ada"></a>

In \[15]:

```
TX_3=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[j.lumley]}" \
  --to-party "${ROLE_NAME[j.lumley]}" \
  --lovelace "$((100 * ADA))" \
  --change-address "${ROLE_ADDR[j.lumley]}" \
  --manual-sign tx-3.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_3 = $TX_3"
```

```
TX_3 = 751155b2ccf8f810cbaf9f1f5ac59e30f1b7c5a5667306eb5878ed2e85227fe9
```

Ký và gửi giao dịch.

In \[16]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-3.unsigned \
  --required-signer "${ROLE_SKEY[j.lumley]}" \
  --timeout 600
```

```
TxId "751155b2ccf8f810cbaf9f1f5ac59e30f1b7c5a5667306eb5878ed2e85227fe9"
```

Kiểm tra xem hợp đồng đã nhận được 200 Ada?

In \[17]:

```
echo "https://cardanoscan.io/transaction/${TX_3}?tab=utxo"
```

```
https://cardanoscan.io/transaction/751155b2ccf8f810cbaf9f1f5ac59e30f1b7c5a5667306eb5878ed2e85227fe9?tab=utxo
```

### Giao dịch số 4: Mary Herbert nạp 100 Ada <a href="#transaction-4.-mary-herbert-deposits-100-ada" id="transaction-4.-mary-herbert-deposits-100-ada"></a>

In \[18]:

```
TX_4=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[m.herbert]}" \
  --to-party "${ROLE_NAME[m.herbert]}" \
  --lovelace "$((100 * ADA))" \
  --change-address "${ROLE_ADDR[m.herbert]}" \
  --manual-sign tx-4.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_4 = $TX_4"
```

```
TX_4 = 76cdca5b9b4efcad402c9c0eb61ef08898b9f854b392d33f9f57b52a137b1fbe
```

Ký và gửi giao dịch.

In \[19]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-4.unsigned \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --timeout 600
```

```
TxId "76cdca5b9b4efcad402c9c0eb61ef08898b9f854b392d33f9f57b52a137b1fbe"
```

Kiểm tra xem hợp đồng đã nhận được 300 Ada?

In \[20]:

```
echo "https://cardanoscan.io/transaction/${TX_4}?tab=utxo"
```

```
https://cardanoscan.io/transaction/76cdca5b9b4efcad402c9c0eb61ef08898b9f854b392d33f9f57b52a137b1fbe?tab=utxo
```

### Giao dịch số 5: Christopher Marlowe nạp token BearGarden <a href="#transaction-5.-christopher-marlowe-deposits-the-beargarden-token" id="transaction-5.-christopher-marlowe-deposits-the-beargarden-token"></a>

Giao dịch này dẫn tới 300 Ada sẽ được trả như là phần lợi ích của anh ấy.

In \[21]:

```
TX_5=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[c.marlowe]}" \
  --to-party "${ROLE_NAME[e.cary]}" \
  --currency "$FUNGIBLES_POLICY" \
  --token-name BearGarden \
  --quantity 1 \
  --change-address "${ROLE_ADDR[c.marlowe]}" \
  --manual-sign tx-5.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_5 = $TX_5"
```

```
TX_5 = 19459abed8961d91ba258648a0616e0a9932c0d04863a8b5d92075db16b8b97e
```

Ký và gửi giao dịch.

In \[22]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-5.unsigned \
  --required-signer "${ROLE_SKEY[c.marlowe]}" \
  --timeout 600
```

```
TxId "19459abed8961d91ba258648a0616e0a9932c0d04863a8b5d92075db16b8b97e"
```

Kiểm tra xem hợp đồng đã nhận được token và đã trả 300 Ada.

In \[23]:

```
echo "https://cardanoscan.io/transaction/${TX_5}?tab=utxo"
```

```
https://cardanoscan.io/transaction/19459abed8961d91ba258648a0616e0a9932c0d04863a8b5d92075db16b8b97e?tab=utxo
```

### Giao dịch số 6: Christopher Marlowe rút 300 Ada <a href="#transaction-6.-christopher-marlowe-withdraws-his-300-ada" id="transaction-6.-christopher-marlowe-withdraws-his-300-ada"></a>

In \[24]:

```
TX_6=$(
marlowe withdraw \
  --contract "$CONTRACT_ID" \
  --role "${ROLE_NAME[c.marlowe]}" \
  --change-address "${ROLE_ADDR[c.marlowe]}" \
  --manual-sign tx-6.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_6 = $TX_6"
```

```
TX_6 = e14270e0ea927b313b3e0d7b56f373037aa84b140536f3881da84f6019f94d6b
```

Ký và gửi giao dịch

In \[25]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-6.unsigned \
  --required-signer "${ROLE_SKEY[c.marlowe]}" \
  --timeout 600
```

```
TxId "e14270e0ea927b313b3e0d7b56f373037aa84b140536f3881da84f6019f94d6b"
```

Kiểm tra xem Ada đã được rút thành công?

In \[26]:

```
echo "https://cardanoscan.io/transaction/${TX_6}?tab=utxo"
```

```
https://cardanoscan.io/transaction/e14270e0ea927b313b3e0d7b56f373037aa84b140536f3881da84f6019f94d6b?tab=utxo
```

### Giao dịch số 7: Mary Herbet nạp 300 Ada để nhận lại token của cô ấy. <a href="#transaction-7.-mary-herbet-deposits-300-ada-to-claim-the-token-for-herself" id="transaction-7.-mary-herbet-deposits-300-ada-to-claim-the-token-for-herself"></a>

Điều này dẫn đến việc thanh toán 150 Ada cho mỗi đồng sở hữu và token được thanh toán cho Mary Herbert, tất cả đều tại địa chỉ thanh toán của vai trò.

In \[27]:

```
TX_7=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[m.herbert]}" \
  --to-party "${ROLE_NAME[m.herbert]}" \
  --lovelace "$((300 * ADA))" \
  --change-address "${ROLE_ADDR[m.herbert]}" \
  --manual-sign tx-7.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_7 = $TX_7"
```

```
TX_7 = d86636353bc234b416d43b4c754f9c55bae6643abba25eaa812359574006074d
```

Ký và gửi giao dịch.

In \[28]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-7.unsigned \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --timeout 600
```

```
TxId "d86636353bc234b416d43b4c754f9c55bae6643abba25eaa812359574006074d"
```

Kiểm tra xem hợp đồng đã được đóng và trả Ada và token.

In \[29]:

```
echo "https://cardanoscan.io/transaction/${TX_7}?tab=utxo"
```

```
https://cardanoscan.io/transaction/d86636353bc234b416d43b4c754f9c55bae6643abba25eaa812359574006074d?tab=utxo
```

### Giao dịch số 8: Elizabeth Cary rút 150 Ada <a href="#transaction-8.-elizabeth-cary-withdraws-her-150-ada" id="transaction-8.-elizabeth-cary-withdraws-her-150-ada"></a>

In \[30]:

```
TX_8=$(
marlowe withdraw \
  --contract "$CONTRACT_ID" \
  --role "${ROLE_NAME[e.cary]}" \
  --change-address "${ROLE_ADDR[e.cary]}" \
  --manual-sign tx-8.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_8 = $TX_8"
```

```
TX_8=493637cec0d089348a3f0f9c3064bc4978a5571d699db6e7ef0d1b6b625e59cc
```

Ký và gửi giao dịch.

In \[31]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-8.unsigned \
  --required-signer "${ROLE_SKEY[e.cary]}" \
  --timeout 600
```

```
TxId "493637cec0d089348a3f0f9c3064bc4978a5571d699db6e7ef0d1b6b625e59cc"
```

Kiểm tra xem Ada của Cary đã được rút thành công?

In \[32]:

```
echo "https://cardanoscan.io/transaction/${TX_8}?tab=utxo"
```

```
https://cardanoscan.io/transaction/493637cec0d089348a3f0f9c3064bc4978a5571d699db6e7ef0d1b6b625e59cc?tab=utxo
```

### Giao dịch số 9: Jane Lumley rút 150 Ada <a href="#transaction-9.-jane-lumley-withdraws-her-150-ada" id="transaction-9.-jane-lumley-withdraws-her-150-ada"></a>

In \[33]:

```
TX_9=$(
marlowe withdraw \
  --contract "$CONTRACT_ID" \
  --role "${ROLE_NAME[j.lumley]}" \
  --change-address "${ROLE_ADDR[j.lumley]}" \
  --manual-sign tx-9.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_9 = $TX_9"
```

```
TX_9 = ab576de1aae77fd5fee0892eefb3062fc4a2e68394d752e528722f0a3ca3623a
```

Ký và gửi giao dịch.

In \[34]:

```
marlowe-cli transaction submit \
 --mainnet \
  --tx-body-file tx-9.unsigned \
  --required-signer "${ROLE_SKEY[j.lumley]}" \
  --timeout 600
```

```
TxId "ab576de1aae77fd5fee0892eefb3062fc4a2e68394d752e528722f0a3ca3623a"
```

Kiểm tra xem Ada đã được rút thành công?

In \[35]:

```
echo "https://cardanoscan.io/transaction/${TX_9}?tab=utxo"
```

```
https://cardanoscan.io/transaction/ab576de1aae77fd5fee0892eefb3062fc4a2e68394d752e528722f0a3ca3623a?tab=utxo
```

### Giao dịch số 10: Mary Herbert rút token BearGarden của cô ấy <a href="#transaction-10.-mary-herbert-withdraws-her-beargarden-token" id="transaction-10.-mary-herbert-withdraws-her-beargarden-token"></a>

In \[36]:

```
TX_10=$(
marlowe withdraw \
  --contract "$CONTRACT_ID" \
  --role "${ROLE_NAME[m.herbert]}" \
  --change-address "${ROLE_ADDR[m.herbert]}" \
  --manual-sign tx-10.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_10 = $TX_10"
```

```
TX_10 = e0155b5b3cad34a758ba7be22408500b15b8c9b81b2365212df9b542cfb1127c
```

Ký và gửi giao dịch.

In \[37]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-10.unsigned \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --timeout 600
```

```
TxId "e0155b5b3cad34a758ba7be22408500b15b8c9b81b2365212df9b542cfb1127c"
```

In \[38]:

```
echo "https://cardanoscan.io/transaction/${TX_10}?tab=utxo"
```

```
https://cardanoscan.io/transaction/e0155b5b3cad34a758ba7be22408500b15b8c9b81b2365212df9b542cfb1127c?tab=utxo
```

### Xem lại toàn bộ lịch sử của hợp đồng <a href="#view-the-whole-history-of-the-contract" id="view-the-whole-history-of-the-contract"></a>

Chúng ta sử dụng `marlowe-pipe` để in ra toàn bộ lịch sử của hợp đồng.

In \[39]:

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
            deposits: 100000000
            into_account:
              role_token: e.cary
            of_token:
              currency_symbol: ''
              token_name: ''
            party:
              role_token: e.cary
          then:
            timeout: 1676679840000
            timeout_continuation: close
            when:
            - case:
                deposits: 100000000
                into_account:
                  role_token: j.lumley
                of_token:
                  currency_symbol: ''
                  token_name: ''
                party:
                  role_token: j.lumley
              then:
                timeout: 1676679850000
                timeout_continuation: close
                when:
                - case:
                    deposits: 100000000
                    into_account:
                      role_token: m.herbert
                    of_token:
                      currency_symbol: ''
                      token_name: ''
                    party:
                      role_token: m.herbert
                  then:
                    timeout: 1676679860000
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
                          role_token: c.marlowe
                      then:
                        from_account:
                          role_token: e.cary
                        pay: 100000000
                        then:
                          from_account:
                            role_token: m.herbert
                          pay: 100000000
                          then:
                            from_account:
                              role_token: j.lumley
                            pay: 100000000
                            then:
                              timeout: 1676679870000
                              timeout_continuation: close
                              when:
                              - case:
                                  deposits: 300000000
                                  into_account:
                                    role_token: e.cary
                                  of_token:
                                    currency_symbol: ''
                                    token_name: ''
                                  party:
                                    role_token: e.cary
                                then:
                                  from_account:
                                    role_token: e.cary
                                  pay: 1
                                  then:
                                    from_account:
                                      role_token: e.cary
                                    pay: 150000000
                                    then:
                                      from_account:
                                        role_token: e.cary
                                      pay: 150000000
                                      then: close
                                      to:
                                        party:
                                          role_token: j.lumley
                                      token:
                                        currency_symbol: ''
                                        token_name: ''
                                    to:
                                      party:
                                        role_token: m.herbert
                                    token:
                                      currency_symbol: ''
                                      token_name: ''
                                  to:
                                    party:
                                      role_token: e.cary
                                  token:
                                    currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                                    token_name: BearGarden
                              - case:
                                  deposits: 300000000
                                  into_account:
                                    role_token: m.herbert
                                  of_token:
                                    currency_symbol: ''
                                    token_name: ''
                                  party:
                                    role_token: m.herbert
                                then:
                                  from_account:
                                    role_token: m.herbert
                                  pay: 150000000
                                  then:
                                    from_account:
                                      role_token: e.cary
                                    pay: 1
                                    then:
                                      from_account:
                                        role_token: m.herbert
                                      pay: 150000000
                                      then: close
                                      to:
                                        party:
                                          role_token: j.lumley
                                      token:
                                        currency_symbol: ''
                                        token_name: ''
                                    to:
                                      party:
                                        role_token: m.herbert
                                    token:
                                      currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                                      token_name: BearGarden
                                  to:
                                    party:
                                      role_token: e.cary
                                  token:
                                    currency_symbol: ''
                                    token_name: ''
                              - case:
                                  deposits: 300000000
                                  into_account:
                                    role_token: j.lumley
                                  of_token:
                                    currency_symbol: ''
                                    token_name: ''
                                  party:
                                    role_token: j.lumley
                                then:
                                  from_account:
                                    role_token: j.lumley
                                  pay: 150000000
                                  then:
                                    from_account:
                                      role_token: j.lumley
                                    pay: 150000000
                                    then:
                                      from_account:
                                        role_token: e.cary
                                      pay: 1
                                      then: close
                                      to:
                                        party:
                                          role_token: j.lumley
                                      token:
                                        currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                                        token_name: BearGarden
                                    to:
                                      party:
                                        role_token: m.herbert
                                    token:
                                      currency_symbol: ''
                                      token_name: ''
                                  to:
                                    party:
                                      role_token: e.cary
                                  token:
                                    currency_symbol: ''
                                    token_name: ''
                            to:
                              party:
                                role_token: c.marlowe
                            token:
                              currency_symbol: ''
                              token_name: ''
                          to:
                            party:
                              role_token: c.marlowe
                          token:
                            currency_symbol: ''
                            token_name: ''
                        to:
                          party:
                            role_token: c.marlowe
                        token:
                          currency_symbol: ''
                          token_name: ''
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
      txId: e9b4608d23b955783bce458c02687353e9497c30980ae8ad941256a74840cd5b
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps:
- contractId: e9b4608d23b955783bce458c02687353e9497c30980ae8ad941256a74840cd5b#1
  payouts: []
  redeemer:
  - input_from_party:
      role_token: e.cary
    into_account:
      role_token: e.cary
    of_token:
      currency_symbol: ''
      token_name: ''
    that_deposits: 100000000
  scriptOutput:
    address: 716a9391d6aa51af28dd876ebb5565b69d1e83e5ac7861506bd29b56b0
    assets:
      ada: 103000000
      tokens: []
    datum:
      marloweContract:
        timeout: 1676679840000
        timeout_continuation: close
        when:
        - case:
            deposits: 100000000
            into_account:
              role_token: j.lumley
            of_token:
              currency_symbol: ''
              token_name: ''
            party:
              role_token: j.lumley
          then:
            timeout: 1676679850000
            timeout_continuation: close
            when:
            - case:
                deposits: 100000000
                into_account:
                  role_token: m.herbert
                of_token:
                  currency_symbol: ''
                  token_name: ''
                party:
                  role_token: m.herbert
              then:
                timeout: 1676679860000
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
                      role_token: c.marlowe
                  then:
                    from_account:
                      role_token: e.cary
                    pay: 100000000
                    then:
                      from_account:
                        role_token: m.herbert
                      pay: 100000000
                      then:
                        from_account:
                          role_token: j.lumley
                        pay: 100000000
                        then:
                          timeout: 1676679870000
                          timeout_continuation: close
                          when:
                          - case:
                              deposits: 300000000
                              into_account:
                                role_token: e.cary
                              of_token:
                                currency_symbol: ''
                                token_name: ''
                              party:
                                role_token: e.cary
                            then:
                              from_account:
                                role_token: e.cary
                              pay: 1
                              then:
                                from_account:
                                  role_token: e.cary
                                pay: 150000000
                                then:
                                  from_account:
                                    role_token: e.cary
                                  pay: 150000000
                                  then: close
                                  to:
                                    party:
                                      role_token: j.lumley
                                  token:
                                    currency_symbol: ''
                                    token_name: ''
                                to:
                                  party:
                                    role_token: m.herbert
                                token:
                                  currency_symbol: ''
                                  token_name: ''
                              to:
                                party:
                                  role_token: e.cary
                              token:
                                currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                                token_name: BearGarden
                          - case:
                              deposits: 300000000
                              into_account:
                                role_token: m.herbert
                              of_token:
                                currency_symbol: ''
                                token_name: ''
                              party:
                                role_token: m.herbert
                            then:
                              from_account:
                                role_token: m.herbert
                              pay: 150000000
                              then:
                                from_account:
                                  role_token: e.cary
                                pay: 1
                                then:
                                  from_account:
                                    role_token: m.herbert
                                  pay: 150000000
                                  then: close
                                  to:
                                    party:
                                      role_token: j.lumley
                                  token:
                                    currency_symbol: ''
                                    token_name: ''
                                to:
                                  party:
                                    role_token: m.herbert
                                token:
                                  currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                                  token_name: BearGarden
                              to:
                                party:
                                  role_token: e.cary
                              token:
                                currency_symbol: ''
                                token_name: ''
                          - case:
                              deposits: 300000000
                              into_account:
                                role_token: j.lumley
                              of_token:
                                currency_symbol: ''
                                token_name: ''
                              party:
                                role_token: j.lumley
                            then:
                              from_account:
                                role_token: j.lumley
                              pay: 150000000
                              then:
                                from_account:
                                  role_token: j.lumley
                                pay: 150000000
                                then:
                                  from_account:
                                    role_token: e.cary
                                  pay: 1
                                  then: close
                                  to:
                                    party:
                                      role_token: j.lumley
                                  token:
                                    currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                                    token_name: BearGarden
                                to:
                                  party:
                                    role_token: m.herbert
                                token:
                                  currency_symbol: ''
                                  token_name: ''
                              to:
                                party:
                                  role_token: e.cary
                              token:
                                currency_symbol: ''
                                token_name: ''
                        to:
                          party:
                            role_token: c.marlowe
                        token:
                          currency_symbol: ''
                          token_name: ''
                      to:
                        party:
                          role_token: c.marlowe
                      token:
                        currency_symbol: ''
                        token_name: ''
                    to:
                      party:
                        role_token: c.marlowe
                    token:
                      currency_symbol: ''
                      token_name: ''
      marloweParams:
        rolesCurrency: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
      marloweState:
        accounts:
        - - - address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
            - currency_symbol: ''
              token_name: ''
          - 3000000
        - - - role_token: e.cary
            - currency_symbol: ''
              token_name: ''
          - 100000000
        boundValues: []
        choices: []
        minTime: 1676397329000
    utxo:
      txId: 72e4916c2cb6626af2102062a592f4ba8f2d2f649ea9e8934191e2f875fca204
      txIx: 1
  step: apply
  txId: 72e4916c2cb6626af2102062a592f4ba8f2d2f649ea9e8934191e2f875fca204
- contractId: e9b4608d23b955783bce458c02687353e9497c30980ae8ad941256a74840cd5b#1
  payouts: []
  redeemer:
  - input_from_party:
      role_token: j.lumley
    into_account:
      role_token: j.lumley
    of_token:
      currency_symbol: ''
      token_name: ''
    that_deposits: 100000000
  scriptOutput:
    address: 716a9391d6aa51af28dd876ebb5565b69d1e83e5ac7861506bd29b56b0
    assets:
      ada: 203000000
      tokens: []
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
              currency_symbol: ''
              token_name: ''
            party:
              role_token: m.herbert
          then:
            timeout: 1676679860000
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
                  role_token: c.marlowe
              then:
                from_account:
                  role_token: e.cary
                pay: 100000000
                then:
                  from_account:
                    role_token: m.herbert
                  pay: 100000000
                  then:
                    from_account:
                      role_token: j.lumley
                    pay: 100000000
                    then:
                      timeout: 1676679870000
                      timeout_continuation: close
                      when:
                      - case:
                          deposits: 300000000
                          into_account:
                            role_token: e.cary
                          of_token:
                            currency_symbol: ''
                            token_name: ''
                          party:
                            role_token: e.cary
                        then:
                          from_account:
                            role_token: e.cary
                          pay: 1
                          then:
                            from_account:
                              role_token: e.cary
                            pay: 150000000
                            then:
                              from_account:
                                role_token: e.cary
                              pay: 150000000
                              then: close
                              to:
                                party:
                                  role_token: j.lumley
                              token:
                                currency_symbol: ''
                                token_name: ''
                            to:
                              party:
                                role_token: m.herbert
                            token:
                              currency_symbol: ''
                              token_name: ''
                          to:
                            party:
                              role_token: e.cary
                          token:
                            currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                            token_name: BearGarden
                      - case:
                          deposits: 300000000
                          into_account:
                            role_token: m.herbert
                          of_token:
                            currency_symbol: ''
                            token_name: ''
                          party:
                            role_token: m.herbert
                        then:
                          from_account:
                            role_token: m.herbert
                          pay: 150000000
                          then:
                            from_account:
                              role_token: e.cary
                            pay: 1
                            then:
                              from_account:
                                role_token: m.herbert
                              pay: 150000000
                              then: close
                              to:
                                party:
                                  role_token: j.lumley
                              token:
                                currency_symbol: ''
                                token_name: ''
                            to:
                              party:
                                role_token: m.herbert
                            token:
                              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                              token_name: BearGarden
                          to:
                            party:
                              role_token: e.cary
                          token:
                            currency_symbol: ''
                            token_name: ''
                      - case:
                          deposits: 300000000
                          into_account:
                            role_token: j.lumley
                          of_token:
                            currency_symbol: ''
                            token_name: ''
                          party:
                            role_token: j.lumley
                        then:
                          from_account:
                            role_token: j.lumley
                          pay: 150000000
                          then:
                            from_account:
                              role_token: j.lumley
                            pay: 150000000
                            then:
                              from_account:
                                role_token: e.cary
                              pay: 1
                              then: close
                              to:
                                party:
                                  role_token: j.lumley
                              token:
                                currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                                token_name: BearGarden
                            to:
                              party:
                                role_token: m.herbert
                            token:
                              currency_symbol: ''
                              token_name: ''
                          to:
                            party:
                              role_token: e.cary
                          token:
                            currency_symbol: ''
                            token_name: ''
                    to:
                      party:
                        role_token: c.marlowe
                    token:
                      currency_symbol: ''
                      token_name: ''
                  to:
                    party:
                      role_token: c.marlowe
                  token:
                    currency_symbol: ''
                    token_name: ''
                to:
                  party:
                    role_token: c.marlowe
                token:
                  currency_symbol: ''
                  token_name: ''
      marloweParams:
        rolesCurrency: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
      marloweState:
        accounts:
        - - - address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
            - currency_symbol: ''
              token_name: ''
          - 3000000
        - - - role_token: e.cary
            - currency_symbol: ''
              token_name: ''
          - 100000000
        - - - role_token: j.lumley
            - currency_symbol: ''
              token_name: ''
          - 100000000
        boundValues: []
        choices: []
        minTime: 1676397626000
    utxo:
      txId: 751155b2ccf8f810cbaf9f1f5ac59e30f1b7c5a5667306eb5878ed2e85227fe9
      txIx: 1
  step: apply
  txId: 751155b2ccf8f810cbaf9f1f5ac59e30f1b7c5a5667306eb5878ed2e85227fe9
- contractId: e9b4608d23b955783bce458c02687353e9497c30980ae8ad941256a74840cd5b#1
  payouts: []
  redeemer:
  - input_from_party:
      role_token: m.herbert
    into_account:
      role_token: m.herbert
    of_token:
      currency_symbol: ''
      token_name: ''
    that_deposits: 100000000
  scriptOutput:
    address: 716a9391d6aa51af28dd876ebb5565b69d1e83e5ac7861506bd29b56b0
    assets:
      ada: 303000000
      tokens: []
    datum:
      marloweContract:
        timeout: 1676679860000
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
              role_token: c.marlowe
          then:
            from_account:
              role_token: e.cary
            pay: 100000000
            then:
              from_account:
                role_token: m.herbert
              pay: 100000000
              then:
                from_account:
                  role_token: j.lumley
                pay: 100000000
                then:
                  timeout: 1676679870000
                  timeout_continuation: close
                  when:
                  - case:
                      deposits: 300000000
                      into_account:
                        role_token: e.cary
                      of_token:
                        currency_symbol: ''
                        token_name: ''
                      party:
                        role_token: e.cary
                    then:
                      from_account:
                        role_token: e.cary
                      pay: 1
                      then:
                        from_account:
                          role_token: e.cary
                        pay: 150000000
                        then:
                          from_account:
                            role_token: e.cary
                          pay: 150000000
                          then: close
                          to:
                            party:
                              role_token: j.lumley
                          token:
                            currency_symbol: ''
                            token_name: ''
                        to:
                          party:
                            role_token: m.herbert
                        token:
                          currency_symbol: ''
                          token_name: ''
                      to:
                        party:
                          role_token: e.cary
                      token:
                        currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                        token_name: BearGarden
                  - case:
                      deposits: 300000000
                      into_account:
                        role_token: m.herbert
                      of_token:
                        currency_symbol: ''
                        token_name: ''
                      party:
                        role_token: m.herbert
                    then:
                      from_account:
                        role_token: m.herbert
                      pay: 150000000
                      then:
                        from_account:
                          role_token: e.cary
                        pay: 1
                        then:
                          from_account:
                            role_token: m.herbert
                          pay: 150000000
                          then: close
                          to:
                            party:
                              role_token: j.lumley
                          token:
                            currency_symbol: ''
                            token_name: ''
                        to:
                          party:
                            role_token: m.herbert
                        token:
                          currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                          token_name: BearGarden
                      to:
                        party:
                          role_token: e.cary
                      token:
                        currency_symbol: ''
                        token_name: ''
                  - case:
                      deposits: 300000000
                      into_account:
                        role_token: j.lumley
                      of_token:
                        currency_symbol: ''
                        token_name: ''
                      party:
                        role_token: j.lumley
                    then:
                      from_account:
                        role_token: j.lumley
                      pay: 150000000
                      then:
                        from_account:
                          role_token: j.lumley
                        pay: 150000000
                        then:
                          from_account:
                            role_token: e.cary
                          pay: 1
                          then: close
                          to:
                            party:
                              role_token: j.lumley
                          token:
                            currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                            token_name: BearGarden
                        to:
                          party:
                            role_token: m.herbert
                        token:
                          currency_symbol: ''
                          token_name: ''
                      to:
                        party:
                          role_token: e.cary
                      token:
                        currency_symbol: ''
                        token_name: ''
                to:
                  party:
                    role_token: c.marlowe
                token:
                  currency_symbol: ''
                  token_name: ''
              to:
                party:
                  role_token: c.marlowe
              token:
                currency_symbol: ''
                token_name: ''
            to:
              party:
                role_token: c.marlowe
            token:
              currency_symbol: ''
              token_name: ''
      marloweParams:
        rolesCurrency: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
      marloweState:
        accounts:
        - - - address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
            - currency_symbol: ''
              token_name: ''
          - 3000000
        - - - role_token: e.cary
            - currency_symbol: ''
              token_name: ''
          - 100000000
        - - - role_token: j.lumley
            - currency_symbol: ''
              token_name: ''
          - 100000000
        - - - role_token: m.herbert
            - currency_symbol: ''
              token_name: ''
          - 100000000
        boundValues: []
        choices: []
        minTime: 1676397740000
    utxo:
      txId: 76cdca5b9b4efcad402c9c0eb61ef08898b9f854b392d33f9f57b52a137b1fbe
      txIx: 1
  step: apply
  txId: 76cdca5b9b4efcad402c9c0eb61ef08898b9f854b392d33f9f57b52a137b1fbe
- contractId: e9b4608d23b955783bce458c02687353e9497c30980ae8ad941256a74840cd5b#1
  payouts:
  - - txId: 19459abed8961d91ba258648a0616e0a9932c0d04863a8b5d92075db16b8b97e
      txIx: 3
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 300000000
        tokens: []
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: c.marlowe
  redeemer:
  - input_from_party:
      role_token: c.marlowe
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
        timeout: 1676679870000
        timeout_continuation: close
        when:
        - case:
            deposits: 300000000
            into_account:
              role_token: e.cary
            of_token:
              currency_symbol: ''
              token_name: ''
            party:
              role_token: e.cary
          then:
            from_account:
              role_token: e.cary
            pay: 1
            then:
              from_account:
                role_token: e.cary
              pay: 150000000
              then:
                from_account:
                  role_token: e.cary
                pay: 150000000
                then: close
                to:
                  party:
                    role_token: j.lumley
                token:
                  currency_symbol: ''
                  token_name: ''
              to:
                party:
                  role_token: m.herbert
              token:
                currency_symbol: ''
                token_name: ''
            to:
              party:
                role_token: e.cary
            token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
        - case:
            deposits: 300000000
            into_account:
              role_token: m.herbert
            of_token:
              currency_symbol: ''
              token_name: ''
            party:
              role_token: m.herbert
          then:
            from_account:
              role_token: m.herbert
            pay: 150000000
            then:
              from_account:
                role_token: e.cary
              pay: 1
              then:
                from_account:
                  role_token: m.herbert
                pay: 150000000
                then: close
                to:
                  party:
                    role_token: j.lumley
                token:
                  currency_symbol: ''
                  token_name: ''
              to:
                party:
                  role_token: m.herbert
              token:
                currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                token_name: BearGarden
            to:
              party:
                role_token: e.cary
            token:
              currency_symbol: ''
              token_name: ''
        - case:
            deposits: 300000000
            into_account:
              role_token: j.lumley
            of_token:
              currency_symbol: ''
              token_name: ''
            party:
              role_token: j.lumley
          then:
            from_account:
              role_token: j.lumley
            pay: 150000000
            then:
              from_account:
                role_token: j.lumley
              pay: 150000000
              then:
                from_account:
                  role_token: e.cary
                pay: 1
                then: close
                to:
                  party:
                    role_token: j.lumley
                token:
                  currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                  token_name: BearGarden
              to:
                party:
                  role_token: m.herbert
              token:
                currency_symbol: ''
                token_name: ''
            to:
              party:
                role_token: e.cary
            token:
              currency_symbol: ''
              token_name: ''
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
        minTime: 1676397771000
    utxo:
      txId: 19459abed8961d91ba258648a0616e0a9932c0d04863a8b5d92075db16b8b97e
      txIx: 1
  step: apply
  txId: 19459abed8961d91ba258648a0616e0a9932c0d04863a8b5d92075db16b8b97e
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: c.marlowe
  redeemingTx: e14270e0ea927b313b3e0d7b56f373037aa84b140536f3881da84f6019f94d6b
  step: payout
  utxo:
    txId: 19459abed8961d91ba258648a0616e0a9932c0d04863a8b5d92075db16b8b97e
    txIx: 3
- contractId: e9b4608d23b955783bce458c02687353e9497c30980ae8ad941256a74840cd5b#1
  payouts:
  - - txId: d86636353bc234b416d43b4c754f9c55bae6643abba25eaa812359574006074d
      txIx: 2
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 150000000
        tokens: []
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: e.cary
  - - txId: d86636353bc234b416d43b4c754f9c55bae6643abba25eaa812359574006074d
      txIx: 3
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 150000000
        tokens: []
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: j.lumley
  - - txId: d86636353bc234b416d43b4c754f9c55bae6643abba25eaa812359574006074d
      txIx: 4
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
  - input_from_party:
      role_token: m.herbert
    into_account:
      role_token: m.herbert
    of_token:
      currency_symbol: ''
      token_name: ''
    that_deposits: 300000000
  scriptOutput: null
  step: apply
  txId: d86636353bc234b416d43b4c754f9c55bae6643abba25eaa812359574006074d
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: e.cary
  redeemingTx: 493637cec0d089348a3f0f9c3064bc4978a5571d699db6e7ef0d1b6b625e59cc
  step: payout
  utxo:
    txId: d86636353bc234b416d43b4c754f9c55bae6643abba25eaa812359574006074d
    txIx: 2
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: j.lumley
  redeemingTx: ab576de1aae77fd5fee0892eefb3062fc4a2e68394d752e528722f0a3ca3623a
  step: payout
  utxo:
    txId: d86636353bc234b416d43b4c754f9c55bae6643abba25eaa812359574006074d
    txIx: 3
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: m.herbert
  redeemingTx: e0155b5b3cad34a758ba7be22408500b15b8c9b81b2365212df9b542cfb1127c
  step: payout
  utxo:
    txId: d86636353bc234b416d43b4c754f9c55bae6643abba25eaa812359574006074d
    txIx: 4
```

### Trả lại token BearGarden về faucet <a href="#return-the-beargarden-tokens-to-the-faucet" id="return-the-beargarden-tokens-to-the-faucet"></a>

Trả lại token cho faucet là một cách quản lý tiện lợi cho ví dụ này.

In \[40]:

```
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "e0155b5b3cad34a758ba7be22408500b15b8c9b81b2365212df9b542cfb1127c#2" \
  --tx-in "e9b4608d23b955783bce458c02687353e9497c30980ae8ad941256a74840cd5b#2" \
  --tx-in "e9b4608d23b955783bce458c02687353e9497c30980ae8ad941256a74840cd5b#0" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+499 $FUNGIBLES_POLICY.BearGarden" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "$FAUCET_SKEY" \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "6b4faac1384ac24db7df0c8728fc78d1b1364b368045fe080e79f0b14f869c10"
```
