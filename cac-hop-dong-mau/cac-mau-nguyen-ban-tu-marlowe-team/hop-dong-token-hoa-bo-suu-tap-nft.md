# Hợp đồng token hóa bộ sưu tập NFT

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

### Hợp đồng token hóa bộ sưu tập NFT

**Tóm tắt:**

Marlowe có thể được sử dụng để tập hợp nhiều NFT vào một bộ sưu tập, bản thân nó cũng là một NFT. Điều này tạo hiệu quả chuyển đổi nhiều NFT thành một đơn vị duy nhất.&#x20;

Hợp đồng được mô tả ở đây thu thập ba NFT vào một hợp đồng Marlowe, được điều khiển bởi một NFT thứ tư đại diện cho bộ sưu tập của ba NFT này. Người nắm giữ NFT thứ tư sở hữu và điều khiển ba NFT còn lại.&#x20;

Tuy nhiên, hợp đồng quy định rằng người tạo hợp đồng (có thể là nghệ sĩ của các NFT) sẽ nhận được 500 Ada nếu chủ sở hữu NFT phá vỡ bộ sưu tập trước năm 2030. Điều này khuyến khích chủ sở hữu giữ bộ sưu tập nguyên vẹn và trả tiền bản quyền cho nghệ sĩ nếu bộ sưu tập bị phá vỡ sớm.&#x20;

Chúng tôi sẽ trình bày cách xây dựng hợp đồng này và kiểm tra nó trên mạng prepod trước khi triển khai trên `mainnet`.

**Nguyên tắc minh họa:**

* Sử dụng Marlowe CLI để mint NFT.
* Sử dụng Marlowe CLI và Marlowe Runtime để thực thi và xem hợp đồng Marlowe.
* Cách khởi động một hợp đồng Marlowe ở trạng thái ban đầu đã chứa hỗn hợp Ada và token.
* Cách kiểm tra hợp đồng trên testnet trước khi triển khai trên mainnet.

Video tham khảo [https://youtu.be/v4KtJb4k0Jc](https://youtu.be/v4KtJb4k0Jc).

### Thiết kế NFTs. <a href="#design-the-nfts" id="design-the-nfts"></a>

#### Chúng ta tạo 4 NFT. <a href="#we-create-four-nfts" id="we-create-four-nfts"></a>

* `YoTr2016-0` vừa là một NFT vừa là token vai trò Marlowe cho hợp đồng mà chúng ta sẽ tạo. Nó đại diện cho bộ sưu tập của ba NFT còn lại.
* `YoTr2016-1`, `YoTr2016-2`, and `YoTr2016-3` là ba NFT trong bộ sưu tập sẽ được gộp vào hợp đồng Marlowe.

| YoTr2016-0                                                                                                                                                                       | YoTr2016-1                                                                                                                                                                                               | YoTr2016-2                                                                                                                                                                                                             | YoTr20160-3                                                                                                                                                                                                                                                                   |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ![Yogācāra Triptych](https://raw.githubusercontent.com/input-output-hk/real-world-marlowe/a288a9f391e68135e59e65a813db695e6223472d/nfts/collection/images/yogacara-triptych.png) | ![The imagination of the unreal](https://raw.githubusercontent.com/input-output-hk/real-world-marlowe/a288a9f391e68135e59e65a813db695e6223472d/nfts/collection/images/the-imagination-of-the-unreal.png) | ![The dependence on flowing conditions](https://raw.githubusercontent.com/input-output-hk/real-world-marlowe/a288a9f391e68135e59e65a813db695e6223472d/nfts/collection/images/the-dependence-on-flowing-conditions.png) | ![The external non-existence of what appears in the way it appears](https://raw.githubusercontent.com/input-output-hk/real-world-marlowe/a288a9f391e68135e59e65a813db695e6223472d/nfts/collection/images/the-external-nonexistence-of-what-appears-in-the-way-it-appears.png) |
| Yogācāra Triptych                                                                                                                                                                | The imagination of the unreal                                                                                                                                                                            | The dependence on flowing conditions                                                                                                                                                                                   | The external non-existence of what appears in the way it appears                                                                                                                                                                                                              |

Dưới đây là metadata tiêu chuẩn CIP-25 cho các NFT.

In \[1]:

```
json2yaml metadata.json
```

```
YoTr2016-0:
  artist: Brian W Bush
  contract: 'A Marlowe role token: <https://marlowe.iohk.io/>.'
  copyright: (c) 2016 Brian W Bush
  description:
  - A series of three original artworks on the subject of Yogācāra
  - philosophy.
  image: ipfs://Qma2TDVkJPwt14WheDnzvwyyutnsabWhn64ey3f4imqLXo
  mediaType: image/png
  name: Yogācāra Triptych
  year: 2016
YoTr2016-1:
  artist: Brian W Bush
  copyright: (c) 2016 Brian W Bush
  description:
  - 'Yogācāra Triptych #1: The imagination of the unreal, 2016,'
  - acrylic, aluminum mesh, and papier-mâché on wood panel,
  - 12″ × 14″.
  dimensions: 12″ × 14″
  image: ipfs://QmVpfNww1KKQ4SWDJaeHoApjt2KWLXraV2xWdXhhVPLTEG
  mediaType: image/png
  medium: acrylic, aluminum mesh, and papier-mâché on wood panel
  name: The imagination of the unreal
  number: 1
  series: Yogācāra Triptych
  year: 2016
YoTr2016-2:
  artist: Brian W Bush
  copyright: (c) 2016 Brian W Bush
  description:
  - 'Yogācāra Triptych #2: The dependence on flowing conditions,'
  - 2016, alabaster, 3″ × 2.5″ × 2.5″.
  dimensions: 3″ × 2.5″ × 2.5″
  image: ipfs://QmTBNjkoZ6wC9erfVCoabMVueQmQTzCGFXDvK47U3p3KgR
  mediaType: image/png
  medium: alabaster
  name: The dependence on flowing conditions
  number: 2
  series: Yogācāra Triptych
  year: 2016
YoTr2016-3:
  artist: Brian W Bush
  copyright: (c) 2016 Brian W Bush
  description:
  - 'Yogācāra Triptych #3: The external non-existence of what'
  - appears in the way it appears, 2016, acrylic and vanilla
  - extract on canvas board, 16″ × 20″.
  dimensions: 16″ × 20″
  image: ipfs://QmW9EEdgxz9RYn52dt3Ynnw39ToN5npcgr4KNwVKQMSXDR
  mediaType: image/png
  medium: acrylic and vanilla extract on canvas board
  name: The external non-existence of what appears in the way it appears
  number: 3
  series: Yogācāra Triptych
  year: 2016
```

#### Gán tên của các token khác nhau cho các biến như sau:

In \[2]:

```
TOKEN_NAMES=($(jq -r 'to_entries | .[].key' metadata.json))
echo "TOKEN_NAMES = ${TOKEN_NAMES[@]}"
ROLE_NAME="${TOKEN_NAMES[0]}"
echo "ROLE_NAME = $ROLE_NAME"
COLLECTION_TOKENS=(${TOKEN_NAMES[@]:1})
echo "COLLECTION_TOKENS = ${COLLECTION_TOKENS[@]}"
```

```
TOKEN_NAMES = YoTr2016-0 YoTr2016-1 YoTr2016-2 YoTr2016-3
ROLE_NAME = YoTr2016-0
COLLECTION_TOKENS = YoTr2016-1 YoTr2016-2 YoTr2016-3
```

#### Thiết kế hợp đồng <a href="#design-the-contract" id="design-the-contract"></a>

* 5 Ada sẽ được gửi vào hợp đồng khi tạo, cùng với ba NFT trong bộ sưu tập.&#x20;
* Token vai trò `(role)` sẽ là NFT đại diện cho ba NFT còn lại.&#x20;
* Hợp đồng sẽ giữ ba NFT này đến năm 2030, trừ khi `role` trả 500 Ada để lấy lại ba NFT, phá vỡ bộ sưu tập.

Về mặt khái niệm, chúng ta sẽ gửi `YoTr2016-1`, `YoTr2016-2`, và `YoTr2016-3` vào một hợp đồng Marlowe, hợp đồng này sẽ giải phóng các token đó cho vai trò `YoTr2016-0` nếu vai trò đó đã thanh toán 500 Ada để giải phóng chúng.&#x20;

Điều này sẽ tạo thành hợp đồng như sau.

![Expanded contract for collecting and holding NFTs](https://raw.githubusercontent.com/input-output-hk/real-world-marlowe/a288a9f391e68135e59e65a813db695e6223472d/nfts/collection/images/expanded-contract.png)

Tuy nhiên, hợp đồng này khá lãng phí vì phải trả phí ba lần để gửi các token từng cái một. Ngôn ngữ Marlowe không cho phép gửi nhiều token cùng lúc, nhưng chúng ta có thể khắc phục bằng cách khởi tạo hợp đồng Marlowe ở trạng thái ban đầu đã chứa các token được gửi vào tài khoản của vai trò `YoTr2016-0`.&#x20;

Giao dịch tạo hợp đồng sẽ thực hiện việc gửi này trước khi hợp đồng bắt đầu chạy. Khi hợp đồng đạt đến trạng thái `Close`, nó sẽ thanh toán các NFT. Đây là hợp đồng đơn giản hóa mà chúng ta sẽ sử dụng ở đây.

![Simplified contract that begins with the NFTs in its initial state](https://raw.githubusercontent.com/input-output-hk/real-world-marlowe/a288a9f391e68135e59e65a813db695e6223472d/nfts/collection/images/contract.png)

#### Thiết lập các thông số cho hợp đồng Marlowe <a href="#set-the-parameters-for-the-marlowe-contract" id="set-the-parameters-for-the-marlowe-contract"></a>

In \[3]:

```
ADA=1000000
MIN_ADA=$((2 * ADA))
echo "MIN_ADA = $MIN_ADA"
SEPARATION_COST=$((500 * ADA))
echo "SEPARATION_COST = $SEPARATION_COST"
```

```
MIN_ADA = 2000000
SEPARATION_COST = 500000000
```

In \[4]:

```
TIMEOUT=$((1000 * $(date -d "2030-01-01" -u +%s)))
echo "TIMEOUT = $TIMEOUT"
```

```
TIMEOUT = 1893456000000
```

### Kiểm thử trên mạng pre-production <a href="#test-on-the-pre-production-network" id="test-on-the-pre-production-network"></a>

Không nên tạo hợp đồng Marlowe mới trên mainnet trừ khi nó đã được thử nghiệm trên một mạng thử nghiệm!

#### Chọn mạng

In \[5]:

```
export CARDANO_NODE_SOCKET_PATH=preprod.socket
MAGIC=(--testnet-magic 1)
NETWORK=testnet
```

Chúng ta đặt vị trí của các dịch vụ backend Marlowe Runtime.

In \[6]:

```
export MARLOWE_RT_HISTORY_HOST=192.168.0.12
export MARLOWE_RT_HISTORY_COMMAND_PORT=13717
export MARLOWE_RT_HISTORY_QUERY_PORT=13718
export MARLOWE_RT_HISTORY_SYNC_PORT=13719
                                                                                                                                     
export MARLOWE_RT_DISCOVERY_HOST=192.168.0.12
export MARLOWE_RT_DISCOVERY_QUERY_PORT=13721
export MARLOWE_RT_DISCOVERY_SYNC_PORT=13722

export MARLOWE_RT_TX_HOST=192.168.0.12
export MARLOWE_RT_TX_COMMAND_PORT=13723
```

#### Thiết lập địa chỉ và khóa ký cho người tạo hợp đồng Marlowe.

In \[7]:

```
PAYMENT_SKEY=payment.skey
PAYMENT_ADDR=$(cat payment.$NETWORK.address)
echo "PAYMENT_ADDR = $PAYMENT_ADDR"
```

```
PAYMENT_ADDR = addr_test1vq9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupczgtm9j
```

#### Để mint các NFT, sử dụng chính sách tiền tệ cấm việc mint thêm sau hai ngày kể từ bây giờ.

In \[8]:

```
LOCKING_SLOT=$(($(cardano-cli query tip "${MAGIC[@]}" | jq -r '.slot') + 2 * 24 * 60 * 60))
echo "LOCKING_SLOT = $LOCKING_SLOT"
```

```
LOCKING_SLOT = 19516471
```

#### Đúc các NFT. <a href="#mint-the-nfts" id="mint-the-nfts"></a>

In \[9]:

```
POLICY_ID=$(
marlowe-cli util mint \
  "${MAGIC[@]}" \
  --issuer "$PAYMENT_ADDR:$PAYMENT_SKEY" \
  --metadata-file metadata.json \
  --expires "$LOCKING_SLOT" \
  --out-file /dev/null \
  --submit 600 \
  $(for n in ${TOKEN_NAMES[@]}; do echo "$n:$PAYMENT_ADDR"; done) \
| sed -e 's/^PolicyID "\(.*\)"$/\1/'
)
echo "POLICY_ID = $POLICY_ID"
```

```
Fee: Lovelace 268593
Size: 2381 / 16384 = 14%
Execution units:
  Memory: 0 / 14000000 = 0%
  Steps: 0 / 10000000000 = 0%
POLICY_ID = 9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05
```

Chúng ta có thể xem các NFT qua đường dẫn:

In \[10]:

```
echo "https://preprod.cexplorer.io/policy/$POLICY_ID"
```

```
https://preprod.cexplorer.io/policy/9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05
```

#### Khởi tạo trạng thái ban đầu của hợp đồng. <a href="#create-the-initial-state-of-the-contract" id="create-the-initial-state-of-the-contract"></a>

Không như các hợp đồng thông thường, trạng thái ban đầu của hợp đồng này sẽ bao gồm một số non-ADA token.

In \[11]:

```
jq 'keys | del(.[0]) | [.[] | [[{role_token : "'"$ROLE_NAME"'"}, {currency_symbol : "'"$POLICY_ID"'", token_name : .}], 1]] | {accounts : ([[[{role_token : "'"$ROLE_NAME"'"}, {currency_symbol : "", token_name : ""}], '"$MIN_ADA"']] + .), choices : [], boundValues : [], minTime : 1}' metadata.json > state.json
json2yaml state.json
```

```
accounts:
- - - role_token: YoTr2016-0
    - currency_symbol: ''
      token_name: ''
  - 2000000
- - - role_token: YoTr2016-0
    - currency_symbol: 9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05
      token_name: YoTr2016-1
  - 1
- - - role_token: YoTr2016-0
    - currency_symbol: 9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05
      token_name: YoTr2016-2
  - 1
- - - role_token: YoTr2016-0
    - currency_symbol: 9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05
      token_name: YoTr2016-3
  - 1
boundValues: []
choices: []
minTime: 1
```

#### Tạo hợp đồng. <a href="#create-the-contract" id="create-the-contract"></a>

Hợp đồng chỉ đơn giản chờ đợi khoản gửi và sau đó đóng lại, dẫn đến việc các token được phân phối cho địa chỉ thanh toán.

In \[12]:

```
yaml2json << EOI > contract.json
when:
- case:
    party:
      role_token: $ROLE_NAME
    deposits: $SEPARATION_COST
    of_token:
      currency_symbol: ''
      token_name: ''
    into_account:
      address: $PAYMENT_ADDR
  then: close
timeout: $TIMEOUT
timeout_continuation: close
EOI
json2yaml contract.json
```

```
timeout: 1893456000000
timeout_continuation: close
when:
- case:
    deposits: 500000000
    into_account:
      address: addr_test1vq9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupczgtm9j
    of_token:
      currency_symbol: ''
      token_name: ''
    party:
      role_token: YoTr2016-0
  then: close
```

Khởi tạo dữ liệu off-chain cho hợp đồng Marlowe.&#x20;

Tệp `.marlowe` chứa thông tin cần thiết để thực thi hợp đồng

In \[13]:

```
marlowe-cli run initialize \
  "${MAGIC[@]}" \
  --roles-currency "$POLICY_ID" \
  --contract-file contract.json \
  --state-file state.json \
  --permanently-without-staking \
  --out-file tx-1.marlowe \
  --print-stats
```

```
Searching for reference script at address: addr_test1vrw0tuh8l95thdqr65dmpcfqnmcw0en7v7vhgegck7gzqgswa07sw

Expected reference script hash: "6a9391d6aa51af28dd876ebb5565b69d1e83e5ac7861506bd29b56b0"

Searching for reference script at address: addr_test1vpa36uuyf95kxpcleldsncedlhjru6vdmh2vnpkdrsz4u6cll9zas

Expected reference script hash: "49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3"

Validator size: 12505
Base-validator cost: ExBudget {exBudgetCPU = ExCPU 18515100, exBudgetMemory = ExMemory 80600}
```

Phân tích hợp đồng Marlowe để đảm bảo rằng nó có thể được chạy thành công trên mạng Cardano.

&#x20;Chỉ vì hợp đồng hợp lệ về mặt ngữ nghĩa không có nghĩa là nó sẽ chạy trên blockchain!

In \[14]:

```
marlowe-cli run analyze \
  "${MAGIC[@]}" \
  --marlowe-file tx-1.marlowe
```

```
Note that path-based analysis ignore the initial state of the contract and instead start with an empty state.
Starting search for execution paths . . .
 . . . found 2 execution paths.
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
    Actual: 88
    Invalid: false
    Maximum: 5000
    Percentage: 1.76
    Unit: byte
- Minimum UTxO:
    Requirement:
      lovelace: 1120600
- Execution cost:
    Memory:
      Actual: 5803588
      Invalid: false
      Maximum: 14000000
      Percentage: 41.4542
    Steps:
      Actual: 1505731602
      Invalid: false
      Maximum: 10000000000
      Percentage: 15.05731602
- Transaction size:
    Actual: 1058
    Invalid: false
    Maximum: 16384
    Percentage: 6.45751953125
```

Phân tích cho thấy hợp đồng không có vấn đề gì ngăn cản việc chạy. Tuy nhiên, phiên bản này của `marlowe-cli run analyze` không tính đến trạng thái ban đầu không chuẩn mà chúng ta đang sử dụng, vì vậy chúng ta cần thực sự chạy hợp đồng trên mạng thử nghiệm để đảm bảo rằng nó an toàn để chạy trên mainnet.

Thực hiện giao dịch để tạo hợp đồng. Đáng tiếc là Marlowe Runtime không hỗ trợ tạo hợp đồng với trạng thái ban đầu không mặc định. Thay vào đó, chúng ta sử dụng `marlowe-cli` để tạo hợp đồng.

In \[15]:

```
TX_1=$(
marlowe-cli run auto-execute \
  "${MAGIC[@]}" \
  --change-address "$PAYMENT_ADDR" \
  --required-signer "$PAYMENT_SKEY" \
  --marlowe-out-file tx-1.marlowe \
  --out-file /dev/null \
  --submit 600 \
  --print-stats \
| sed -e 's/^TxId "\(.*\)"$/\1/' \
)
echo "TX_1 = $TX_1"
CONTRACT_ID="$TX_1#1"
echo "CONTRACT_ID = $CONTRACT_ID"
```

```
Fee: Lovelace 222789
Size: 883 / 16384 = 5%
Execution units:
  Memory: 0 / 14000000 = 0%
  Steps: 0 / 10000000000 = 0%
TX_1 = 7935febf14cd7519f0d419c0e112891c6be1bdf1301e0334095b732024afeedf
CONTRACT_ID = 7935febf14cd7519f0d419c0e112891c6be1bdf1301e0334095b732024afeedf#1
```

Xem UTxO của hợp đồng trên blockchain.

In \[16]:

```
cardano-cli query utxo  "${MAGIC[@]}" --tx-in "$CONTRACT_ID"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
7935febf14cd7519f0d419c0e112891c6be1bdf1301e0334095b732024afeedf     1        2000000 lovelace + 1 9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05.596f5472323031362d31 + 1 9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05.596f5472323031362d32 + 1 9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05.596f5472323031362d33 + TxOutDatumHash ScriptDataInBabbageEra "e3c7281f48e683cb4253aafad5bbb50ff7d0cdb2366bafaf4f7d6d4af68c09b3"
```

Sử dụng Marlowe Runtime để xem hợp đồng trên blockchain.

In \[17]:

```
marlowe log --show-contract "$CONTRACT_ID"
```

```
transaction 7935febf14cd7519f0d419c0e112891c6be1bdf1301e0334095b732024afeedf (creation)
ContractId:      7935febf14cd7519f0d419c0e112891c6be1bdf1301e0334095b732024afeedf#1
SlotNo:          19344129
BlockNo:         572415
BlockId:         c3b99130628d9fd3ac09b93aac61d4114e9286dbf00c3fd3bd76a6f0a30de5b7
ScriptAddress:   addr_test1wp4f8ywk4fg672xasahtk4t9k6w3aql943uxz5rt62d4dvqu3c6jv
Marlowe Version: 1

    When [
      (Case
         (Deposit (Address "addr_test1vq9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupczgtm9j") (Role "YoTr2016-0")
            (Token "" "")
            (Constant 500000000)) Close)] 1893456000000 Close

```

Xem giao dịch qua trình khám phá dữ liệu blockchain

In \[18]:

```
echo "https://preprod.cardanoscan.io/transaction/$TX_1?tab=utxo"
```

```
https://preprod.cardanoscan.io/transaction/7935febf14cd7519f0d419c0e112891c6be1bdf1301e0334095b732024afeedf?tab=utxo
```

#### Chuẩn bị một giao dịch để loại bỏ các NFT khỏi hợp đồng.&#x20;

Việc này yêu cầu chủ sở hữu token vai trò gửi 500 Ada vào hợp đồng.

In \[19]:

```
TX_2=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "$ROLE_NAME" \
  --to-party "$PAYMENT_ADDR" \
  --lovelace "$SEPARATION_COST" \
  --change-address "$PAYMENT_ADDR" \
  --manual-sign tx-2.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_2 = $TX_2"
```

```
TX_2 = d6ffa4f8e2841a19e17e04b9ed57b88c576c7dcf5c2ec94064531976898adba2
```

#### Thực thi giao dịch để đóng hợp đồng và loại bỏ các token vai trò. <a href="#execute-the-transaction-to-close-the-contract-and-remove-the-role-tokens" id="execute-the-transaction-to-close-the-contract-and-remove-the-role-tokens"></a>

In \[20]:

```
marlowe-cli transaction submit \
  "${MAGIC[@]}" \
  --tx-body-file tx-2.unsigned \
  --required-signer "$PAYMENT_SKEY" \
  --timeout 600
```

```
TxId "d6ffa4f8e2841a19e17e04b9ed57b88c576c7dcf5c2ec94064531976898adba2"
```

Sử dụng Marlowe Runtime để kiểm tra xem hợp đồng đã được đóng?

In \[21]:

```
marlowe log --show-contract "$CONTRACT_ID"
```

```
transaction 7935febf14cd7519f0d419c0e112891c6be1bdf1301e0334095b732024afeedf (creation)
ContractId:      7935febf14cd7519f0d419c0e112891c6be1bdf1301e0334095b732024afeedf#1
SlotNo:          19344129
BlockNo:         572415
BlockId:         c3b99130628d9fd3ac09b93aac61d4114e9286dbf00c3fd3bd76a6f0a30de5b7
ScriptAddress:   addr_test1wp4f8ywk4fg672xasahtk4t9k6w3aql943uxz5rt62d4dvqu3c6jv
Marlowe Version: 1

    When [
      (Case
         (Deposit (Address "addr_test1vq9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupczgtm9j") (Role "YoTr2016-0")
            (Token "" "")
            (Constant 500000000)) Close)] 1893456000000 Close

transaction d6ffa4f8e2841a19e17e04b9ed57b88c576c7dcf5c2ec94064531976898adba2 (close)
ContractId: 7935febf14cd7519f0d419c0e112891c6be1bdf1301e0334095b732024afeedf#1
SlotNo:     19344275
BlockNo:    572421
BlockId:    77620cdb13a8efe63144d9cec98d00652b472a102ba4ea58acb7ba597b981d08
Inputs:     [NormalInput (IDeposit "\"addr_test1vq9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupczgtm9j\"" "YoTr2016-0" (Token "" "") 500000000)]


```

Xem giao dịch qua trình khám phá dữ liệu blockchain

In \[22]:

```
echo "https://preprod.cardanoscan.io/transaction/$TX_2?tab=utxo"
```

```
https://preprod.cardanoscan.io/transaction/d6ffa4f8e2841a19e17e04b9ed57b88c576c7dcf5c2ec94064531976898adba2?tab=utxo
```

#### Rút các NFT từ địa chỉ thanh toán vai trò. <a href="#withdraw-the-nfts-from-marlowes-role-payout-address" id="withdraw-the-nfts-from-marlowes-role-payout-address"></a>

Khi một hợp đồng Marlowe thanh toán cho một vai trò, các khoản tiền sẽ được gửi đến địa chỉ thanh toán của vai trò, từ đó người nắm giữ token vai trò có thể rút tiền.

Xem UTxO tại địa chỉ thanh toán của vai trò.

In \[23]:

```
cardano-cli query utxo "${MAGIC[@]}" --tx-in "$TX_2#2"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
d6ffa4f8e2841a19e17e04b9ed57b88c576c7dcf5c2ec94064531976898adba2     2        2000000 lovelace + 1 9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05.596f5472323031362d31 + 1 9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05.596f5472323031362d32 + 1 9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05.596f5472323031362d33 + TxOutDatumHash ScriptDataInBabbageEra "8ddc57b7cbf71fefe4b3cf200982a8181e8c369929c0b4719c69f4848193f018"
```

#### Bây giờ rút các NFT.

In \[24]:

```
TX_3=$(
marlowe withdraw \
  --contract "$CONTRACT_ID" \
  --change-address "$PAYMENT_ADDR" \
  --role "$ROLE_NAME" \
  --manual-sign tx-3.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_3 = $TX_3"
```

```
TX_3 = 11e792d140b2d1749a505b70dd76ae45bf8cd5abfc5b8acbbf38789e4395b8bd
```

In \[25]:

```
marlowe-cli transaction submit \
  "${MAGIC[@]}" \
  --tx-body-file tx-3.unsigned \
  --required-signer "$PAYMENT_SKEY" \
  --timeout 600
```

```
TxId "11e792d140b2d1749a505b70dd76ae45bf8cd5abfc5b8acbbf38789e4395b8bd"
```

Kiểm tra các NFT hiện đã nằm trong ví ban đầu.

In \[26]:

```
cardano-cli query utxo "${MAGIC[@]}" --address "$PAYMENT_ADDR"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
11e792d140b2d1749a505b70dd76ae45bf8cd5abfc5b8acbbf38789e4395b8bd     0        29473874451 lovelace + TxOutDatumNone
11e792d140b2d1749a505b70dd76ae45bf8cd5abfc5b8acbbf38789e4395b8bd     1        1051640 lovelace + 1 9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05.596f5472323031362d30 + TxOutDatumNone
11e792d140b2d1749a505b70dd76ae45bf8cd5abfc5b8acbbf38789e4395b8bd     2        1155080 lovelace + 1 9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05.596f5472323031362d31 + 1 9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05.596f5472323031362d32 + 1 9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05.596f5472323031362d33 + TxOutDatumNone
d6ffa4f8e2841a19e17e04b9ed57b88c576c7dcf5c2ec94064531976898adba2     3        500000000 lovelace + TxOutDatumNone
```

In \[27]:

```
echo "https://preprod.cardanoscan.io/transaction/$TX_3?tab=utxo"
```

```
https://preprod.cardanoscan.io/transaction/11e792d140b2d1749a505b70dd76ae45bf8cd5abfc5b8acbbf38789e4395b8bd?tab=utxo
```

#### Làm sạch bằng cách "đốt" các NFT. <a href="#clean-up-by-burning-the-nfts" id="clean-up-by-burning-the-nfts"></a>

In \[28]:

```
marlowe-cli util burn \
  "${MAGIC[@]}" \
  --issuer "$PAYMENT_ADDR:$PAYMENT_SKEY" \
  --metadata-file metadata.json \
  --expires "$LOCKING_SLOT" \
  --out-file /dev/null \
  --submit 600 \
  $(for n in ${TOKEN_NAMES[@]}; do echo "--token-provider $PAYMENT_ADDR:$PAYMENT_SKEY"; done)
```

```
Fee: Lovelace 271893
Size: 2153 / 16384 = 13%
Execution units:
  Memory: 0 / 14000000 = 0%
  Steps: 0 / 10000000000 = 0%
PolicyID "9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05"
```

Bây giờ các ví đầu tiên sẽ không còn chứa các NFT.

In \[29]:

```
cardano-cli query utxo "${MAGIC[@]}" --address "$PAYMENT_ADDR"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
288fbf2a4e177c56a0bedcfbf935ac107c208073f53513b3cbbca2074c0230b1     0        29975809278 lovelace + TxOutDatumNone
```

### Chạy trên `mainnet` <a href="#run-on-mainnet" id="run-on-mainnet"></a>

Bây giờ chúng ta đã biết hợp đồng là an toàn, bắt đầu chạy trên `mainnet`.

#### Chọn mạng  <a href="#select-the-network" id="select-the-network"></a>

In \[30]:

```
export CARDANO_NODE_SOCKET_PATH=mainnet.socket
MAGIC=(--mainnet)
NETWORK=mainnet
```

Chúng ta đặt vị trí của các dịch vụ backend Marlowe Runtime.

In \[31]:

```
export MARLOWE_RT_HISTORY_HOST=192.168.0.12
export MARLOWE_RT_HISTORY_COMMAND_PORT=53717
export MARLOWE_RT_HISTORY_QUERY_PORT=53718
export MARLOWE_RT_HISTORY_SYNC_PORT=53719
                                                                                                                                     
export MARLOWE_RT_DISCOVERY_HOST=192.168.0.12
export MARLOWE_RT_DISCOVERY_QUERY_PORT=53721
export MARLOWE_RT_DISCOVERY_SYNC_PORT=53722

export MARLOWE_RT_TX_HOST=192.168.0.12
export MARLOWE_RT_TX_COMMAND_PORT=53723
```

#### Thiết lập địa chỉ và khóa ký cho người tạo hợp đồng Marlowe.

In \[32]:

```
PAYMENT_SKEY=payment.skey
PAYMENT_ADDR=$(cat payment.$NETWORK.address)
echo "PAYMENT_ADDR = $PAYMENT_ADDR"
```

```
PAYMENT_ADDR = addr1vy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupceql82h
```

Chúng ta cũng sử dụng địa chỉ đã biết của script **tham chiếu** Marlowe trên `mainnet`.

In \[33]:

```
REFERENCE_ADDR=addr1z9l4w7djneh0kss4drg2php6ynflsvmal7x3w5nrc95uvhz7e4q926apsvcd6kn33cpx95k8jsmrj7v0k62rczvz8urqrl2z0l
```

#### Để mint các NFT, sử dụng chính sách tiền tệ cấm việc mint thêm sau hai ngày kể từ bây giờ.

In \[34]:

```
LOCKING_SLOT=$(($(cardano-cli query tip "${MAGIC[@]}" | jq -r '.slot') + 2 * 24 * 60 * 60))
echo "LOCKING_SLOT = $LOCKING_SLOT"
```

```
LOCKING_SLOT = 83634311
```

#### Đúc các NFT. <a href="#mint-the-nfts" id="mint-the-nfts"></a>

In \[35]:

```
POLICY_ID=$(
marlowe-cli util mint \
  "${MAGIC[@]}" \
  --issuer "$PAYMENT_ADDR:$PAYMENT_SKEY" \
  --metadata-file metadata.json \
  --expires "$LOCKING_SLOT" \
  --out-file /dev/null \
  --submit 600 \
  $(for n in ${TOKEN_NAMES[@]}; do echo "$n:$PAYMENT_ADDR"; done) \
| sed -e 's/^PolicyID "\(.*\)"$/\1/'
)
echo "POLICY_ID = $POLICY_ID"
```

```
Fee: Lovelace 268417
Size: 2377 / 16384 = 14%
Execution units:
  Memory: 0 / 14000000 = 0%
  Steps: 0 / 10000000000 = 0%
POLICY_ID = 76d2abf2f510fe2d2ddd990fd87f68f14360d303b6366cf5a7482ad4
```

Chúng ta có thể xem các NFT qua đường dẫn:

In \[36]:

```
echo "https://cexplorer.io/policy/$POLICY_ID"
```

```
https://cexplorer.io/policy/76d2abf2f510fe2d2ddd990fd87f68f14360d303b6366cf5a7482ad4
```

#### Khởi tạo trạng thái ban đầu của hợp đồng. <a href="#create-the-initial-state-of-the-contract" id="create-the-initial-state-of-the-contract"></a>

Không như các hợp đồng thông thường, trạng thái ban đầu của hợp đồng này sẽ bao gồm một số non-ADA token.

In \[37]:

```
jq 'keys | del(.[0]) | [.[] | [[{role_token : "'"$ROLE_NAME"'"}, {currency_symbol : "'"$POLICY_ID"'", token_name : .}], 1]] | {accounts : ([[[{role_token : "'"$ROLE_NAME"'"}, {currency_symbol : "", token_name : ""}], '"$MIN_ADA"']] + .), choices : [], boundValues : [], minTime : 1}' metadata.json > state.json
json2yaml state.json
```

```
accounts:
- - - role_token: YoTr2016-0
    - currency_symbol: ''
      token_name: ''
  - 2000000
- - - role_token: YoTr2016-0
    - currency_symbol: 76d2abf2f510fe2d2ddd990fd87f68f14360d303b6366cf5a7482ad4
      token_name: YoTr2016-1
  - 1
- - - role_token: YoTr2016-0
    - currency_symbol: 76d2abf2f510fe2d2ddd990fd87f68f14360d303b6366cf5a7482ad4
      token_name: YoTr2016-2
  - 1
- - - role_token: YoTr2016-0
    - currency_symbol: 76d2abf2f510fe2d2ddd990fd87f68f14360d303b6366cf5a7482ad4
      token_name: YoTr2016-3
  - 1
boundValues: []
choices: []
minTime: 1
```

#### Tạo hợp đồng. <a href="#create-the-contract" id="create-the-contract"></a>

Hợp đồng chỉ đơn giản chờ đợi khoản gửi và sau đó đóng lại, dẫn đến việc các token được phân phối cho địa chỉ thanh toán.

In \[38]:

```
yaml2json << EOI > contract.json
when:
- case:
    party:
      role_token: $ROLE_NAME
    deposits: $SEPARATION_COST
    of_token:
      currency_symbol: ''
      token_name: ''
    into_account:
      address: $PAYMENT_ADDR
  then: close
timeout: $TIMEOUT
timeout_continuation: close
EOI
json2yaml contract.json
```

```
timeout: 1893456000000
timeout_continuation: close
when:
- case:
    deposits: 500000000
    into_account:
      address: addr1vy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupceql82h
    of_token:
      currency_symbol: ''
      token_name: ''
    party:
      role_token: YoTr2016-0
  then: close
```

#### Khởi tạo dữ liệu off-chain cho hợp đồng Marlowe.&#x20;

Tệp `.marlowe` chứa thông tin cần thiết để thực thi hợp đồng

In \[39]:

```
marlowe-cli run initialize \
  "${MAGIC[@]}" \
  --roles-currency "$POLICY_ID" \
  --contract-file contract.json \
  --state-file state.json \
  --at-address "$REFERENCE_ADDR" \
  --out-file tx-1.marlowe \
  --print-stats
```

```
Searching for reference script at address: addr1z9l4w7djneh0kss4drg2php6ynflsvmal7x3w5nrc95uvhz7e4q926apsvcd6kn33cpx95k8jsmrj7v0k62rczvz8urqrl2z0l

Expected reference script hash: "6a9391d6aa51af28dd876ebb5565b69d1e83e5ac7861506bd29b56b0"

Searching for reference script at address: addr1z9l4w7djneh0kss4drg2php6ynflsvmal7x3w5nrc95uvhz7e4q926apsvcd6kn33cpx95k8jsmrj7v0k62rczvz8urqrl2z0l

Expected reference script hash: "49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3"

Validator size: 12505
Base-validator cost: ExBudget {exBudgetCPU = ExCPU 18515100, exBudgetMemory = ExMemory 80600}
```

#### Thực hiện giao dịch để tạo hợp đồng.&#x20;

Đáng tiếc là Marlowe Runtime không hỗ trợ việc tạo hợp đồng với trạng thái ban đầu không mặc định. Thay vào đó, chúng ta sẽ sử dụng `marlowe-cli` để tạo hợp đồng.

In \[40]:

```
TX_1=$(
marlowe-cli run auto-execute \
  "${MAGIC[@]}" \
  --change-address "$PAYMENT_ADDR" \
  --required-signer "$PAYMENT_SKEY" \
  --marlowe-out-file tx-1.marlowe \
  --out-file /dev/null \
  --submit 600 \
  --print-stats \
| sed -e 's/^TxId "\(.*\)"$/\1/' \
)
echo "TX_1 = $TX_1"
CONTRACT_ID="$TX_1#1"
echo "CONTRACT_ID = $CONTRACT_ID"
```

```
Fee: Lovelace 222613
Size: 875 / 16384 = 5%
Execution units:
  Memory: 0 / 14000000 = 0%
  Steps: 0 / 10000000000 = 0%
TX_1 = ac64d6f7ed327ab9b0f561aebcb004967fbd9ff5ab377964b88874efd6c8b33b
CONTRACT_ID = ac64d6f7ed327ab9b0f561aebcb004967fbd9ff5ab377964b88874efd6c8b33b#1
```

Xem UTxO của hợp đồng trên blockchain.

In \[41]:

```
cardano-cli query utxo  "${MAGIC[@]}" --tx-in "$CONTRACT_ID"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
ac64d6f7ed327ab9b0f561aebcb004967fbd9ff5ab377964b88874efd6c8b33b     1        2000000 lovelace + 1 76d2abf2f510fe2d2ddd990fd87f68f14360d303b6366cf5a7482ad4.596f5472323031362d31 + 1 76d2abf2f510fe2d2ddd990fd87f68f14360d303b6366cf5a7482ad4.596f5472323031362d32 + 1 76d2abf2f510fe2d2ddd990fd87f68f14360d303b6366cf5a7482ad4.596f5472323031362d33 + TxOutDatumHash ScriptDataInBabbageEra "cfe093e26ba90bbfadfcbbd0d11a2dfa8441b0e5fdfe4921ad8c03f8192ab2a1"
```

Sử dụng Marlowe Runtime để xem hợp đồng trên blockchain.

In \[42]:

```
marlowe log --show-contract "$CONTRACT_ID"
```

```
transaction ac64d6f7ed327ab9b0f561aebcb004967fbd9ff5ab377964b88874efd6c8b33b (creation)
ContractId:      ac64d6f7ed327ab9b0f561aebcb004967fbd9ff5ab377964b88874efd6c8b33b#1
SlotNo:          83461652
BlockNo:         8333588
BlockId:         a90e1ab45d88d9781b5a1b623965e26342db0d629162a808dc1e9c85ae35bb87
ScriptAddress:   addr1w94f8ywk4fg672xasahtk4t9k6w3aql943uxz5rt62d4dvq8evxaf
Marlowe Version: 1

    When [
      (Case
         (Deposit (Address "addr1vy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupceql82h") (Role "YoTr2016-0")
            (Token "" "")
            (Constant 500000000)) Close)] 1893456000000 Close

```

Kiểm tra giao dịch qua trình khám phá dữ liệu blockchain:

In \[43]:

```
echo "https://cardanoscan.io/transaction/$TX_1?tab=utxo"
```

```
https://cardanoscan.io/transaction/ac64d6f7ed327ab9b0f561aebcb004967fbd9ff5ab377964b88874efd6c8b33b?tab=utxo
```

#### Chuẩn bị một giao dịch để loại bỏ các NFT khỏi hợp đồng.&#x20;

Việc này yêu cầu chủ sở hữu token vai trò gửi 500 Ada vào hợp đồng.

In \[44]:

```
TX_2=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "$ROLE_NAME" \
  --to-party "$PAYMENT_ADDR" \
  --lovelace "$SEPARATION_COST" \
  --change-address "$PAYMENT_ADDR" \
  --manual-sign tx-2.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_2 = $TX_2"
```

```
TX_2 = ff16318eb0a3ad7eb9dc4096796dfd461ad42e459076f6eb06fa898f0989c335
```

_Chúng ta sẽ không gửi giao dịch này, nhưng thật tốt khi biết rằng chúng ta có thể thực hiện thành công nếu chúng ta muốn đóng hợp đồng và loại bỏ các token khỏi đó._

### Kết luận:

Giờ đây, chúng ta có NFT đại diện cho bộ sưu tập trong ví của chủ sở hữu và ba NFT cấu thành bộ sưu tập trong một hợp đồng Marlowe được kiểm soát bởi chủ sở hữu.

Có thể sử dụng một trình khám phá để xem rằng NFT cho toàn bộ bộ sưu tập nằm trong ví.

In \[45]:

```
echo "https://pool.pm/$PAYMENT_ADDR"
```

```
https://pool.pm/addr1vy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupceql82h
```

In \[46]:

```
MARLOWE_ADDR=$(marlowe-cli contract address "${MAGIC[@]}")
echo "MARLOWE_ADDR = $MARLOWE_ADDR"
```

```
MARLOWE_ADDR = addr1w94f8ywk4fg672xasahtk4t9k6w3aql943uxz5rt62d4dvq8evxaf
```

In \[47]:

```
echo "https://pool.pm/$MARLOWE_ADDR"
```

```
https://pool.pm/addr1w94f8ywk4fg672xasahtk4t9k6w3aql943uxz5rt62d4dvq8evxaf
```
