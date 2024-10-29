# Hợp đồng thanh toán bản quyền NFT

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

### Hợp đồng thanh toán bản quyền NFT

Bán và trao đổi một token để nhận stablecoin iUSD và thanh toán tiền bản quyền cho nghệ sĩ NFT. Xem CIP-27 để biết thêm thông tin về tiền bản quyền NFT trong Cardano.

Ví dụ này bao gồm sáu giao dịch:

1. Christopher Marlowe tạo hợp đồng Marlowe bán token.
2. Christopher Marlowe gửi 1 token BearGarden vào hợp đồng.
3. Jane Lumley gửi 50.175 iUSD vào hợp đồng, khiến hợp đồng thanh toán token cho cô và iUSD cho Christopher Marlowe và nghệ sĩ Elizabeth Cary.
4. Christopher Marlowe rút 50 iUSD của mình từ địa chỉ thanh toán của Marlowe.
5. Jane Lumley rút 1 BearGarden của mình từ địa chỉ thanh toán của Marlowe.
6. Elizabeth Cary rút 0.175 iUSD tiền bản quyền của mình từ địa chỉ thanh toán của Marlowe.

Dưới đây là hợp đồng ở định dạng Blockly:

![](https://raw.githubusercontent.com/input-output-hk/real-world-marlowe/a288a9f391e68135e59e65a813db695e6223472d/nfts/royalty/contract.png)

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

### Role tokens <a href="#role-tokens" id="role-tokens"></a>

### oken vai trò&#x20;

Hợp đồng này sử dụng Ada Handles làm token vai trò:

* Christopher Marlowe = $c.marlowe
* Jane Lumley = $j.lumley
* Elizabeth Carey = $e.cary

Lưu ý: Chỉ sử dụng token đã được phát hành trước làm token vai trò Marlowe nếu bạn đã xem xét chính sách tiền tệ để tìm kiếm các lỗ hổng bảo mật.

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

### Policy IDs cho iUSD <a href="#policy-ids-for-iusd" id="policy-ids-for-iusd"></a>

Đây là policy cho stablecoin mà chúng ta sử dụng.

In \[5]:

```
echo "IUSD_POLICY = $IUSD_POLICY"
echo "IUSD_NAME = $IUSD_NAME"
```

```
IUSD_POLICY = f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
IUSD_NAME = iUSD
```

### Gửi token ban đầu <a href="#initial-funding" id="initial-funding"></a>

Gửi token BearGarden từ faucet cho Christopher Marlowe và stablecoin cho Jane Lumley.

In \[6]:

```
ADA=1000000
IUSD=1000000
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "a54cdaeaa68371466a6ddd0a8b20c092b9473bf5840e00f9e3dbcbd0c0967e2a#2" \
  --tx-in "a54cdaeaa68371466a6ddd0a8b20c092b9473bf5840e00f9e3dbcbd0c0967e2a#3" \
  --tx-out "${ROLE_ADDR[j.lumley]}+$((2 * ADA))+$((100 * IUSD)) $IUSD_POLICY.$IUSD_NAME" \
  --tx-out "${ROLE_ADDR[c.marlowe]}+$((2 * ADA))+500 $FUNGIBLES_POLICY.BearGarden" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "$FAUCET_SKEY" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "fc76cd9764ec44756afae5e1f83d5f5049daf94df67e3ac1d3c18b654dc5558b"
```

### Hợp đồng Marlowe <a href="#the-marlowe-contract" id="the-marlowe-contract"></a>

Hợp đồng Marlowe thực tế chỉ cần download file JSON được biên dịch từ hợp đồng này dạng Blockly đã được thiết kế sẵn trong [Marlowe Playground](https://play.marlowe.iohk.io/#/).

In \[7]:

```
json2yaml contract.json
```

```
be: 3500
let: Royalty PPM
then:
  be: 50000000
  let: Price
  then:
    be:
      by: 1000000
      divide:
        multiply:
          use_value: Price
        times:
          use_value: Royalty PPM
    let: Royalty
    then:
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
              deposits:
                add:
                  use_value: Price
                and:
                  use_value: Royalty
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
              then:
                from_account:
                  role_token: c.marlowe
                pay:
                  use_value: Price
                then:
                  from_account:
                    role_token: c.marlowe
                  pay:
                    use_value: Royalty
                  then: close
                  to:
                    party:
                      role_token: e.cary
                  token:
                    currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                    token_name: iUSD
                to:
                  party:
                    role_token: c.marlowe
                token:
                  currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                  token_name: iUSD
              to:
                party:
                  role_token: j.lumley
              token:
                currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                token_name: BearGarden
```

### Phân tích hợp đồng <a href="#analyze-the-contract" id="analyze-the-contract"></a>

Hợp đồng này khá phức tạp và chúng ta cần kiểm tra kỹ nếu nó có thể thực thi thành công trên blockchain Cardano.

Đầu tiên, tạo 1 file bao gồm các trạng thái ban đầu của hợp đồng.

In \[8]:

```
yaml2json << EOI > state.json
accounts:
- - - address: "$FAUCET_ADDR"
    - currency_symbol: ''
      token_name: ''
  - 3000000
boundValues: []
choices: []
minTime: 1
EOI
cat state.json
```

```
{"accounts":[[[{"address":"addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7"},{"currency_symbol":"","token_name":""}],3000000]],"boundValues":[],"choices":[],"minTime":1}
```

Tiếp theo, Chuyển thông tin về trạng thái và hợp đồng vào file JSON để phân tích.

In \[9]:

```
marlowe-cli run initialize \
  --mainnet \
  --contract-file contract.json \
  --state-file state.json \
  --roles-currency "$ROLES_CURRENCY" \
  --at-address "$REFERENCE_ADDR" \
  --out-file marlowe.json
```

Cuối cùng, hãy phân tích file JSON bao gồm tất cả thông tin về hợp đồng.

In \[10]:

```
marlowe-cli run analyze \
  --mainnet \
  --marlowe-file marlowe.json
```

```
Note that path-based analysis ignore the initial state of the contract and instead start with an empty state.
Starting search for execution paths . . .
 . . . found 3 execution paths.
- Preconditions:
    Duplicate accounts: []
    Duplicate bound values: []
    Duplicate choices: []
    Invalid account parties: []
    Invalid account tokens: []
    Invalid choice parties: []
    Invalid roles currency: false
    Non-positive account balances: []
- Role names:
    Blank role names: false
    Invalid role names: []
- Tokens:
    Invalid tokens: []
- Maximum value:
    Actual: 144
    Invalid: false
    Maximum: 5000
    Percentage: 2.88
    Unit: byte
- Minimum UTxO:
    Requirement:
      lovelace: 1474020
- Execution cost:
    Memory:
      Actual: 11045416
      Invalid: false
      Maximum: 14000000
      Percentage: 78.89582857142857
    Steps:
      Actual: 2836866405
      Invalid: false
      Maximum: 10000000000
      Percentage: 28.36866405
- Transaction size:
    Actual: 1903
    Invalid: false
    Maximum: 16384
    Percentage: 11.614990234375
```

Trong phần trên, chúng ta thấy rằng không có giá trị không hợp lệ hoặc vượt quá giới hạn giao thức, vì vậy hợp đồng an toàn để chạy.&#x20;

_Lưu ý: một thực hành an toàn hơn là chạy tất cả các đường dẫn thực thi của hợp đồng trên một mạng thử nghiệm._

### Giao dịch số 1: Tạo hợp đồng <a href="#transaction-1.-create-the-contract" id="transaction-1.-create-the-contract"></a>

Chúng ta sử dụng công cụ _Marlowe Runtime's command-line_ để xây dựng giao dịch tạo hợp đồng.

In \[11]:

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
CONTRACT_ID = bc6896d9e69d61058355f44fdb6d2343385999f26c894e58b6c5b3c7649df58f#1
```

The contract can be signed an submitted with any wallet or service. For convenience, we use `marlowe-cli` here.

In \[12]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-1.unsigned \
  --required-signer "$FAUCET_SKEY" \
  --timeout 600
```

```
TxId "bc6896d9e69d61058355f44fdb6d2343385999f26c894e58b6c5b3c7649df58f"
```

In \[13]:

```
echo "https://cardanoscan.io/transaction/${CONTRACT_ID%%#1}?tab=utxo"
```

```
https://cardanoscan.io/transaction/bc6896d9e69d61058355f44fdb6d2343385999f26c894e58b6c5b3c7649df58f?tab=utxo
```

We can use a tool such as `marlowe-pipe` to fetch the contract from the blockchain and display it.

In \[14]:

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
        be: 3500
        let: Royalty PPM
        then:
          be: 50000000
          let: Price
          then:
            be:
              by: 1000000
              divide:
                multiply:
                  use_value: Price
                times:
                  use_value: Royalty PPM
            let: Royalty
            then:
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
                      deposits:
                        add:
                          use_value: Price
                        and:
                          use_value: Royalty
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
                      then:
                        from_account:
                          role_token: c.marlowe
                        pay:
                          use_value: Price
                        then:
                          from_account:
                            role_token: c.marlowe
                          pay:
                            use_value: Royalty
                          then: close
                          to:
                            party:
                              role_token: e.cary
                          token:
                            currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                            token_name: iUSD
                        to:
                          party:
                            role_token: c.marlowe
                        token:
                          currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                          token_name: iUSD
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
      txId: bc6896d9e69d61058355f44fdb6d2343385999f26c894e58b6c5b3c7649df58f
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps: []
```

### Transaction 2. Christopher Marlowe deposits the BearGarden token into the contract <a href="#transaction-2.-christopher-marlowe-deposits-the-beargarden-token-into-the-contract" id="transaction-2.-christopher-marlowe-deposits-the-beargarden-token-into-the-contract"></a>

The logic of the contract dictates that Christopher Marlowe deposits one BearGarden token into his account in the Marlowe contract.

In \[15]:

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
TX_2 = 5ddaf981b9191f607c5f8e901f3655f876f994bb3aab7d385d4fee0b7bc30f73
```

Sign and submit the transaction.

In \[16]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-2.unsigned \
  --required-signer "${ROLE_SKEY[c.marlowe]}" \
  --timeout 600
```

```
TxId "5ddaf981b9191f607c5f8e901f3655f876f994bb3aab7d385d4fee0b7bc30f73"
```

In \[17]:

```
echo "https://cardanoscan.io/transaction/$TX_2?tab=utxo"
```

```
https://cardanoscan.io/transaction/5ddaf981b9191f607c5f8e901f3655f876f994bb3aab7d385d4fee0b7bc30f73?tab=utxo
```

View the output to the Marlowe contract to see that it now holds 1 BearGarden token.

In \[18]:

```
cardano-cli query utxo --mainnet --tx-in "$TX_2#1"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
5ddaf981b9191f607c5f8e901f3655f876f994bb3aab7d385d4fee0b7bc30f73     1        3000000 lovelace + 1 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d.4265617247617264656e + TxOutDatumHash ScriptDataInBabbageEra "06fcf8fbb9256df94ee3f58533c631d5b51e61dfb49d9dc0c1e0db9606e06fbf"
```

### Transaction 3. Jane Lumley deposits 50.175 iUSD into the contract, causing it to pay the parties. <a href="#transaction-3.-jane-lumley-deposits-50.175-iusd-into-the-contract-causing-it-to-pay-the-parties" id="transaction-3.-jane-lumley-deposits-50.175-iusd-into-the-contract-causing-it-to-pay-the-parties"></a>

Depositing the 50.175 iUSD causes the contract to close as it pays 1 BearGarden to Marlowe's role-payout address for the benefit of Jane Lumley, 50 iUSD for the benefit of Christopher Marlowe, and 0.175 iUSD for the benefit of Elizabeth Cary.

In \[19]:

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
        be: 3500
        let: Royalty PPM
        then:
          be: 50000000
          let: Price
          then:
            be:
              by: 1000000
              divide:
                multiply:
                  use_value: Price
                times:
                  use_value: Royalty PPM
            let: Royalty
            then:
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
                      deposits:
                        add:
                          use_value: Price
                        and:
                          use_value: Royalty
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
                      then:
                        from_account:
                          role_token: c.marlowe
                        pay:
                          use_value: Price
                        then:
                          from_account:
                            role_token: c.marlowe
                          pay:
                            use_value: Royalty
                          then: close
                          to:
                            party:
                              role_token: e.cary
                          token:
                            currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                            token_name: iUSD
                        to:
                          party:
                            role_token: c.marlowe
                        token:
                          currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                          token_name: iUSD
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
      txId: bc6896d9e69d61058355f44fdb6d2343385999f26c894e58b6c5b3c7649df58f
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps:
- contractId: bc6896d9e69d61058355f44fdb6d2343385999f26c894e58b6c5b3c7649df58f#1
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
            deposits:
              add:
                use_value: Price
              and:
                use_value: Royalty
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
            then:
              from_account:
                role_token: c.marlowe
              pay:
                use_value: Price
              then:
                from_account:
                  role_token: c.marlowe
                pay:
                  use_value: Royalty
                then: close
                to:
                  party:
                    role_token: e.cary
                token:
                  currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                  token_name: iUSD
              to:
                party:
                  role_token: c.marlowe
              token:
                currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                token_name: iUSD
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
        boundValues:
        - - Royalty PPM
          - 3500
        - - Price
          - 50000000
        - - Royalty
          - 175000
        choices: []
        minTime: 1676326882000
    utxo:
      txId: 5ddaf981b9191f607c5f8e901f3655f876f994bb3aab7d385d4fee0b7bc30f73
      txIx: 1
  step: apply
  txId: 5ddaf981b9191f607c5f8e901f3655f876f994bb3aab7d385d4fee0b7bc30f73
```

In \[20]:

```
TX_3=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[j.lumley]}" \
  --to-party "${ROLE_NAME[c.marlowe]}" \
  --currency "$IUSD_POLICY" \
  --token-name "$IUSD_NAME" \
  --quantity "$((50 * IUSD + 175000))" \
  --change-address "${ROLE_ADDR[j.lumley]}" \
  --manual-sign tx-3.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_3 = $TX_3"
```

```
TX_3 = ef7ba4b05d4f8bed493aaf881fcd7b4aac3d396e92e4977a23f69b3b0e194a7a
```

Sign and submit the transaction.

In \[21]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-3.unsigned \
  --required-signer "${ROLE_SKEY[j.lumley]}" \
  --timeout 600
```

```
TxId "ef7ba4b05d4f8bed493aaf881fcd7b4aac3d396e92e4977a23f69b3b0e194a7a"
```

In \[22]:

```
echo "https://cardanoscan.io/transaction/$TX_3?tab=utxo"
```

```
https://cardanoscan.io/transaction/ef7ba4b05d4f8bed493aaf881fcd7b4aac3d396e92e4977a23f69b3b0e194a7a?tab=utxo
```

See that Chrisopher Marlowe still holds his role token.

In \[23]:

```
cardano-cli query utxo --mainnet --address "${ROLE_ADDR[c.marlowe]}"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
5ddaf981b9191f607c5f8e901f3655f876f994bb3aab7d385d4fee0b7bc30f73     0        135165043 lovelace + TxOutDatumNone
5ddaf981b9191f607c5f8e901f3655f876f994bb3aab7d385d4fee0b7bc30f73     2        5000000 lovelace + 1 f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a.632e6d61726c6f7765 + TxOutDatumNone
5ddaf981b9191f607c5f8e901f3655f876f994bb3aab7d385d4fee0b7bc30f73     3        1180940 lovelace + 499 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d.4265617247617264656e + TxOutDatumNone
ca16af8573a2b844b8a3ea8d8299da37d6c63d83582a8f54eac927198884aae6     0        12201100 lovelace + TxOutDatumNone
```

See that Jane Lumley still holds her role token.

In \[24]:

```
cardano-cli query utxo --mainnet --address "${ROLE_ADDR[j.lumley]}"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
ef7ba4b05d4f8bed493aaf881fcd7b4aac3d396e92e4977a23f69b3b0e194a7a     0        34433457 lovelace + TxOutDatumNone
ef7ba4b05d4f8bed493aaf881fcd7b4aac3d396e92e4977a23f69b3b0e194a7a     1        5000000 lovelace + 1 f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a.6a2e6c756d6c6579 + TxOutDatumNone
ef7ba4b05d4f8bed493aaf881fcd7b4aac3d396e92e4977a23f69b3b0e194a7a     6        1163700 lovelace + 49825000 f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880.69555344 + TxOutDatumNone
```

See that Marlowe's payout address holds the 50 iUSD, 0.175 iUSD, and the 1 BearGarden.

In \[25]:

```
cardano-cli query utxo --mainnet --tx-in "$TX_3#2" --tx-in "$TX_3#3" --tx-in "$TX_3#4"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
ef7ba4b05d4f8bed493aaf881fcd7b4aac3d396e92e4977a23f69b3b0e194a7a     2        1202490 lovelace + 50000000 f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880.69555344 + TxOutDatumHash ScriptDataInBabbageEra "ea0afe598fe417d62bee6191c1838aaadb7d7aaa2849fe9bff19abeb5233199b"
ef7ba4b05d4f8bed493aaf881fcd7b4aac3d396e92e4977a23f69b3b0e194a7a     3        1202490 lovelace + 175000 f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880.69555344 + TxOutDatumHash ScriptDataInBabbageEra "33753770f23489161e4c7a12b9d09f77eb29b4d2320ea906092a23b07821fb5f"
ef7ba4b05d4f8bed493aaf881fcd7b4aac3d396e92e4977a23f69b3b0e194a7a     4        1211110 lovelace + 1 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d.4265617247617264656e + TxOutDatumHash ScriptDataInBabbageEra "9a0959b845f659fd39e6be40797247b4b1f2a1af78710d69adc9eb48f983a789"
```

### Transaction 4. Christopher Marlowe withdraws his 50 iUSD from the role-payout address <a href="#transaction-4.-christopher-marlowe-withdraws-his-50-iusd-from-the-role-payout-address" id="transaction-4.-christopher-marlowe-withdraws-his-50-iusd-from-the-role-payout-address"></a>

In \[26]:

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
TX_4 = 4e30cb6ca15dd6f830e6f1a78d5d2e40a9c98abe0bedcd87e85db1727ae94453
```

Sign and submit the transaction.

In \[27]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-4.unsigned \
  --required-signer "${ROLE_SKEY[c.marlowe]}" \
  --timeout 600
```

```
TxId "4e30cb6ca15dd6f830e6f1a78d5d2e40a9c98abe0bedcd87e85db1727ae94453"
```

See that Christopher Marlowe has successfully withdrawn the 50 iUSD from the role-payout address.

In \[28]:

```
echo "https://cardanoscan.io/transaction/$TX_4?tab=utxo"
```

```
https://cardanoscan.io/transaction/4e30cb6ca15dd6f830e6f1a78d5d2e40a9c98abe0bedcd87e85db1727ae94453?tab=utxo
```

### Transaction 5. Jane Lumley withdraws her 1 BearGarden from the role-payout address <a href="#transaction-5.-jane-lumley-withdraws-her-1-beargarden-from-the-role-payout-address" id="transaction-5.-jane-lumley-withdraws-her-1-beargarden-from-the-role-payout-address"></a>

In \[29]:

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
TX_5 = 32748b6b714afc4c1721224917b501b2886a306d196c4ab839ec33bd6a6c38c4
```

Sign and submit the transaction.

In \[30]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-5.unsigned \
  --required-signer "${ROLE_SKEY[j.lumley]}" \
  --timeout 600
```

```
TxId "32748b6b714afc4c1721224917b501b2886a306d196c4ab839ec33bd6a6c38c4"
```

See that Jane Lumley has successfully withdrawn the 1 BearGarden from the role-payout address.

In \[31]:

```
echo "https://cardanoscan.io/transaction/$TX_5?tab=utxo"
```

```
https://cardanoscan.io/transaction/32748b6b714afc4c1721224917b501b2886a306d196c4ab839ec33bd6a6c38c4?tab=utxo
```

### Transaction 6. Elizabeth Cary withdraws her 0.175 iUSD royalty from the role-payout address <a href="#transaction-6.-elizabeth-cary-withdraws-her-0.175-iusd-royalty-from-the-role-payout-address" id="transaction-6.-elizabeth-cary-withdraws-her-0.175-iusd-royalty-from-the-role-payout-address"></a>

In \[32]:

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
TX_6 = 9c0e6575a3b69dd9435949c6587584032858f709b3e74bcafb8a2a35bdbb45d9
```

Sign and submit the transaction.

In \[33]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-6.unsigned \
  --required-signer "${ROLE_SKEY[e.cary]}" \
  --timeout 600
```

```
TxId "9c0e6575a3b69dd9435949c6587584032858f709b3e74bcafb8a2a35bdbb45d9"
```

See that Elizabeth Cary has successfully withdrawn the 0.175 iUSD from the role-payout address.

In \[34]:

```
echo "https://cardanoscan.io/transaction/$TX_6?tab=utxo"
```

```
https://cardanoscan.io/transaction/9c0e6575a3b69dd9435949c6587584032858f709b3e74bcafb8a2a35bdbb45d9?tab=utxo
```

### View the whole history of the contract <a href="#view-the-whole-history-of-the-contract" id="view-the-whole-history-of-the-contract"></a>

We use `marlowe-pipe` to print the whole history of this contract.

In \[35]:

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
        be: 3500
        let: Royalty PPM
        then:
          be: 50000000
          let: Price
          then:
            be:
              by: 1000000
              divide:
                multiply:
                  use_value: Price
                times:
                  use_value: Royalty PPM
            let: Royalty
            then:
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
                      deposits:
                        add:
                          use_value: Price
                        and:
                          use_value: Royalty
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
                      then:
                        from_account:
                          role_token: c.marlowe
                        pay:
                          use_value: Price
                        then:
                          from_account:
                            role_token: c.marlowe
                          pay:
                            use_value: Royalty
                          then: close
                          to:
                            party:
                              role_token: e.cary
                          token:
                            currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                            token_name: iUSD
                        to:
                          party:
                            role_token: c.marlowe
                        token:
                          currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                          token_name: iUSD
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
      txId: bc6896d9e69d61058355f44fdb6d2343385999f26c894e58b6c5b3c7649df58f
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps:
- contractId: bc6896d9e69d61058355f44fdb6d2343385999f26c894e58b6c5b3c7649df58f#1
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
            deposits:
              add:
                use_value: Price
              and:
                use_value: Royalty
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
            then:
              from_account:
                role_token: c.marlowe
              pay:
                use_value: Price
              then:
                from_account:
                  role_token: c.marlowe
                pay:
                  use_value: Royalty
                then: close
                to:
                  party:
                    role_token: e.cary
                token:
                  currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                  token_name: iUSD
              to:
                party:
                  role_token: c.marlowe
              token:
                currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                token_name: iUSD
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
        boundValues:
        - - Royalty PPM
          - 3500
        - - Price
          - 50000000
        - - Royalty
          - 175000
        choices: []
        minTime: 1676326882000
    utxo:
      txId: 5ddaf981b9191f607c5f8e901f3655f876f994bb3aab7d385d4fee0b7bc30f73
      txIx: 1
  step: apply
  txId: 5ddaf981b9191f607c5f8e901f3655f876f994bb3aab7d385d4fee0b7bc30f73
- contractId: bc6896d9e69d61058355f44fdb6d2343385999f26c894e58b6c5b3c7649df58f#1
  payouts:
  - - txId: ef7ba4b05d4f8bed493aaf881fcd7b4aac3d396e92e4977a23f69b3b0e194a7a
      txIx: 2
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 1202490
        tokens:
        - - policyId: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
            tokenName: iUSD
          - 50000000
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: c.marlowe
  - - txId: ef7ba4b05d4f8bed493aaf881fcd7b4aac3d396e92e4977a23f69b3b0e194a7a
      txIx: 3
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 1202490
        tokens:
        - - policyId: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
            tokenName: iUSD
          - 175000
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: e.cary
  - - txId: ef7ba4b05d4f8bed493aaf881fcd7b4aac3d396e92e4977a23f69b3b0e194a7a
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
        tokenName: j.lumley
  redeemer:
  - input_from_party:
      role_token: j.lumley
    into_account:
      role_token: c.marlowe
    of_token:
      currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
      token_name: iUSD
    that_deposits: 50175000
  scriptOutput: null
  step: apply
  txId: ef7ba4b05d4f8bed493aaf881fcd7b4aac3d396e92e4977a23f69b3b0e194a7a
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: c.marlowe
  redeemingTx: 4e30cb6ca15dd6f830e6f1a78d5d2e40a9c98abe0bedcd87e85db1727ae94453
  step: payout
  utxo:
    txId: ef7ba4b05d4f8bed493aaf881fcd7b4aac3d396e92e4977a23f69b3b0e194a7a
    txIx: 2
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: j.lumley
  redeemingTx: 32748b6b714afc4c1721224917b501b2886a306d196c4ab839ec33bd6a6c38c4
  step: payout
  utxo:
    txId: ef7ba4b05d4f8bed493aaf881fcd7b4aac3d396e92e4977a23f69b3b0e194a7a
    txIx: 4
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: e.cary
  redeemingTx: 9c0e6575a3b69dd9435949c6587584032858f709b3e74bcafb8a2a35bdbb45d9
  step: payout
  utxo:
    txId: ef7ba4b05d4f8bed493aaf881fcd7b4aac3d396e92e4977a23f69b3b0e194a7a
    txIx: 3
```

### Return the BearGarden and iUSD tokens to the faucet <a href="#return-the-beargarden-and-iusd-tokens-to-the-faucet" id="return-the-beargarden-and-iusd-tokens-to-the-faucet"></a>

Returning the token to the faucet is convenient housekeeping for this example.

In \[36]:

```
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "32748b6b714afc4c1721224917b501b2886a306d196c4ab839ec33bd6a6c38c4#2" \
  --tx-in "4e30cb6ca15dd6f830e6f1a78d5d2e40a9c98abe0bedcd87e85db1727ae94453#2" \
  --tx-in "5ddaf981b9191f607c5f8e901f3655f876f994bb3aab7d385d4fee0b7bc30f73#3" \
  --tx-in "9c0e6575a3b69dd9435949c6587584032858f709b3e74bcafb8a2a35bdbb45d9#2" \
  --tx-in "ef7ba4b05d4f8bed493aaf881fcd7b4aac3d396e92e4977a23f69b3b0e194a7a#6" \
  --tx-in "bc6896d9e69d61058355f44fdb6d2343385999f26c894e58b6c5b3c7649df58f#0" \
  --tx-in "ef7ba4b05d4f8bed493aaf881fcd7b4aac3d396e92e4977a23f69b3b0e194a7a#5" \
  --tx-in "fc76cd9764ec44756afae5e1f83d5f5049daf94df67e3ac1d3c18b654dc5558b#0" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+$((100 * IUSD)) $IUSD_POLICY.$IUSD_NAME" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+500 $FUNGIBLES_POLICY.BearGarden" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "$FAUCET_SKEY" \
  --required-signer "${ROLE_SKEY[c.marlowe]}" \
  --required-signer "${ROLE_SKEY[j.lumley]}" \
  --required-signer "${ROLE_SKEY[e.cary]}" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "357298faa4c200ce3f1696671c0455be01866bdbcbfc80a3893d62e394b4a9c2"
```
