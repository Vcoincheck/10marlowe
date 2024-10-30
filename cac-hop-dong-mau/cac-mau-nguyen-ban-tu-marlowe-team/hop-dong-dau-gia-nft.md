# Hợp đồng đấu giá NFT

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

## Đấu giá kiểu Anh <a href="#english-auction" id="english-auction"></a>

**Tóm tắt**

Đây là một cuộc đấu giá tiếng Anh với năm người tham gia và ba vòng đấu giá, thể hiện việc sử dụng merkleization cho các hợp đồng Marlowe.&#x20;

Do các nhà thầu có thể đấu giá theo bất kỳ thứ tự nào và có thể đấu giá nhiều hơn một lần, trừ khi họ bị loại bởi một lần đấu giá không hợp lệ hoặc hết thời gian, hợp đồng này có sự bùng nổ tổ hợp của các khả năng.&#x20;

Nếu không có **merkleization**, tệp JSON của hợp đồng có kích thước 990MB và chứa 940k câu lệnh Case, nhưng sau khi merkleization, tệp JSON giảm còn 9.8MB và chỉ chứa 1,150 câu lệnh Case đã merkleized.

#### Đặc điểm của hợp đồng

* Một người bán đấu giá một đơn vị tài sản.
* Bất kỳ số lượng người đấu giá nào có thể tham gia vào hợp đồng.
* Các lần đấu giá có thể xảy ra theo bất kỳ thứ tự nào.
* Có một số lần đấu giá cố định (các vòng đấu giá) được cho phép.
* Một lần đấu giá sẽ bị từ chối nếu không cao hơn tất cả các lần đấu giá trước đó.
* Một lần đấu giá sẽ bị từ chối nếu không được theo sau ngay lập tức bởi một khoản tiền đặt cọc Lovelace tương ứng với số tiền đã đấu giá.
* Các quỹ được hoàn trả cho những người đấu giá không thành công.
* Có thời hạn để gửi tài sản.
* Mỗi vòng đấu giá có một thời hạn.

#### Các điểm nổi bật khác

* Sử dụng merkleization.
* Sử dụng Notify để phân chia thực thi thành các giao dịch không vượt quá ngân sách thực thi Plutus.

#### Chuỗi các lần đấu giá trong ví dụ này

1. Christopher Marlowe tạo hợp đồng.
2. Christopher Marlowe gửi token BearGarden.
3. Mary Herbert đấu giá 5 ADA.
4. Mary Herbert gửi ADA để bảo đảm cho lần đấu giá của mình.
5. Elizabeth Cary đấu giá 15 ADA.
6. Elizabeth Cary gửi ADA để bảo đảm cho lần đấu giá của mình.
7. Mary Herbert đấu giá 25 ADA.
8. Mary Herbert gửi ADA bổ sung để bảo đảm cho lần đấu giá của mình.
9. Francis Beaumont đấu giá 30 ADA.
10. Francis Beaumont gửi ADA để bảo đảm cho lần đấu giá của mình.
11. Elizabeth Cary đấu giá 40 ADA.
12. Elizabeth Cary gửi ADA bổ sung để bảo đảm cho lần đấu giá của mình.
13. Mary Herbert đấu giá 50 ADA.
14. Mary Herbert gửi ADA bổ sung để bảo đảm cho lần đấu giá của mình, và việc đấu giá kết thúc.
15. Hợp đồng được thông báo để thanh toán token BearGarden cho địa chỉ thanh toán vai trò nhằm lợi ích của Mary Herbert và số ADA thắng cho tài khoản của Christopher Marlowe.
16. Hợp đồng được thông báo để hoàn trả bất kỳ quỹ nào còn nợ cho Jane Lumley và John Webster về địa chỉ thanh toán vai trò cho lợi ích của họ.
17. Hợp đồng được thông báo để hoàn trả bất kỳ quỹ nào còn nợ cho Elizabeth Cary và Mary Webster về địa chỉ thanh toán vai trò cho lợi ích của họ.
18. Hợp đồng được thông báo để hoàn trả bất kỳ quỹ nào còn nợ cho Christopher Marlowe và Francis Beaumont về địa chỉ thanh toán vai trò cho lợi ích của họ.
19. Christopher Marlowe rút quỹ từ địa chỉ thanh toán vai trò.
20. Francis Beaumont rút quỹ từ địa chỉ thanh toán vai trò.
21. Elizabeth Cary rút quỹ từ địa chỉ thanh toán vai trò.
22. Mary Webster rút token BearGarden và quỹ từ địa chỉ thanh toán vai trò.

Mary Webster là người thắng cuộc trong cuộc đấu giá, Christopher Marlowe nhận được 50 ADA, và các nhà đấu giá khác nhận lại khoản tiền đặt cọc của họ. Jane Lumley và John Webster không tham gia đấu giá.

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
* Francis Beaumont = [$f.beaumont](https://pool.pm/asset1dv4kncr59t9cndrqdhdd28l656eppcq9mlcxq7)
* Elizabeth Carey = [$e.cary](https://pool.pm/asset1tx4euajkdczmkawgkjy342agaq33885dlvp0jl)
* Mary Herbert = [$m.herbert](https://pool.pm/asset1a38nhu84xquj7whe3xqr80uyf99mh2r7hzf277)
* Jane Lumley = [$j.lumley](https://pool.pm/asset1kujmmryzmxyqz6utp2slrmwfq4dmmnvwhkh7gkm)
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

### Policy ID cho token BearGarden <a href="#policy-id-for-the-beargarden-token" id="policy-id-for-the-beargarden-token"></a>

Chúng tôi đã trước đó phát hành token BearGarden với chính sách sau đây.&#x20;

In \[4]:

```
echo "FUNGIBLES_POLICY = $FUNGIBLES_POLICY"
```

```
FUNGIBLES_POLICY = 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
```

### Gửi ban đầu <a href="#initial-funding" id="initial-funding"></a>

Gửi token BearGarden fungible từ faucet cho Christopher Marlowe.

In \[5]:

```
ADA=1000000
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "6de64605baf977fbb88938d8f51f3ab03ed759a4f3442864c7b8d63916f5ef2e#0" \
  --tx-in "0f8060fd92a3918a9536f911f4e575fcbd9cad889e8d61807878f82d631be43d#1" \
  --tx-out "${ROLE_ADDR[c.marlowe]}+$((2 * ADA))+1 $FUNGIBLES_POLICY.BearGarden" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "$FAUCET_SKEY" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "efda2e04c5c40cb2cb7b6ef49f1e674bf71a48d5e5a0618bb81cd63d581dfe16"
```

#### Thiết lập hợp đồng <a href="#set-up-the-contract" id="set-up-the-contract"></a>

Bắt đầu hợp đồng từ thời điểm hiện tại

In \[6]:

```
NOW=$(($(date -u +%s) * 1000))
echo "NOW = $NOW = $(date -d @$((NOW / 1000)))"
```

```
NOW = 1678573569000 = Sat Mar 11 03:26:09 PM MST 2023
```

Yêu cầu tối thiểu về số ADA liên quan đến một native token.

In \[7]:

```
ADA=1000000
MIN_ADA=$((2 * ADA))
echo "MIN_ADA = $MIN_ADA"
```

```
MIN_ADA = 2000000
```

Tạo một chương trình Haskell sẽ sinh ra một hợp đồng với sáu vòng đấu giá cho ba người tham gia. Chương trình xuất ra tệp [contract.json](https://github.com/input-output-hk/real-world-marlowe/blob/a288a9f391e68135e59e65a813db695e6223472d/nfts/auction/contract.json) và các tiếp tục đã merkle hóa [continuations.json](https://github.com/input-output-hk/real-world-marlowe/blob/a288a9f391e68135e59e65a813db695e6223472d/nfts/auction/continuations.json) của hợp đồng.

In \[8]:

```
HOUR=$((60 * 60 * 1000))

cat << EOI > english-auction.hs

{-# LANGUAGE NumericUnderscores #-}
{-# LANGUAGE OverloadedStrings  #-}
{-# LANGUAGE Trustworthy        #-}

module Main (
-- * Entry point
  main
-- * Contracts
, makeContract
) where

import Control.Monad (foldM)
import Control.Monad.Writer (Writer, runWriter)
import Data.Aeson ((.=), encodeFile, object, toJSON)
import Data.List.Split (chunksOf)
import Data.Map.Strict (toList)
import Data.String (fromString)
import Language.Marlowe.Core.V1.Semantics.Types
import Language.Marlowe.CLI.Merkle (deepMerkleize)
import Language.Marlowe.CLI.Types (Continuations)
import Plutus.V1.Ledger.Api (POSIXTime(..), TokenName(..))

-- | Print the contract.
main :: IO ()
main =
  do
    let
      (contract, continuations) =
        runWriter
          $ makeContract
            6 (Role <$> ["f.beaumont", "e.cary", "m.herbert", "j.lumley", "j.webster"])
            (Bound 2_000_000 1_000_000_000_000)
            (Token "$FUNGIBLES_POLICY" "BearGarden")
            $NOW (5 * $HOUR)
    encodeFile "contract.json" contract
    encodeFile "continuations.json" continuations

-- | A timeout beyond the operation of the contract, just used for merklization.
timeoutFinal :: POSIXTime
timeoutFinal = POSIXTime $((NOW + 6 * HOUR))

-- | Ada.
ada :: Token
ada = Token "" ""

-- | The party that sells the item at auction.
seller :: Party
seller = Role "c.marlowe"

-- | The quantity of items that is auctioned.
assetAmount :: Value Observation
assetAmount = Constant 1

-- | The minimum ada requirement for the asset.
minAda :: Value Observation
minAda = Constant $MIN_ADA

-- | The value of the highest bid.
highestBid :: ValueId
highestBid = "Highest Bid"

-- | Create the Marlowe contract for an English auction.
makeContract :: Int                            -- ^ The number of rounds of bidding.
             -> [Party]                        -- ^ The bidders.
             -> Bound                          -- ^ The range for valid bids, in Lovelace.
             -> Token                          -- ^ The token representing the asset being bid upon.
             -> Integer                        -- ^ The start time of the contract.
             -> Integer                        -- ^ The spacing between bid deadlines.
             -> Writer Continuations Contract  -- ^ Action for creating the merkleized English auction.
makeContract nRounds bidders bidBounds assetToken start delta =
  do
    let
      bids = ChoiceId "Bid" <$> bidders
      deadlines = [POSIXTime $ start + delta * (1 + fromIntegral i) | i <- [1..nRounds]]
    -- Make explicit payments upon completion of bidding.
    close <- foldM (flip makeRefunds) Close (chunksOf 2 $ seller : bidders)
    -- Deposit the asset, then make the bids, but close if no one bids.
    makeAssetDeposit assetToken (POSIXTime $ start + delta)
      -- Do the rounds of bidding and depositing.
      =<< makeBids bidBounds assetToken deadlines bids
      close close

-- | Deposit the asset that is the subject of the bidding.
makeAssetDeposit :: Token                          -- ^ The token representing the asset being bid upon.
                 -> Timeout                        -- ^ The timeout for depositing the asset.
                 -> Contract                       -- ^ The contract to be executed after the asset is deposited.
                 -> Writer Continuations Contract  -- ^ Action for creating the merkleized contract for the asset deposit and subsequent activity.
makeAssetDeposit asset assetDeadline continuation =
  deepMerkleize
    $ When
    [
      -- The seller deposits the asset being auctioned.
      Case (Deposit seller seller asset assetAmount)
        continuation
    ]
    -- The contract ends if the deposit is not made.
    assetDeadline
    Close

-- | Make the contract for bids.
makeBids :: Bound                            -- ^ The range of valid bids, in Lovelace.
         -> Token                            -- ^ The token representing the asset being bid upon.
         -> [Timeout]                        -- ^ The deadlines for the rounds of bidding.
         -> [ChoiceId]                       -- ^ The choices the bidders will make.
         -> Contract                         -- ^ The contract to be executed at the end of the bidding.
         -> Contract                         -- ^ The next stage of the contract, if a valid bid was not made.
         -> Writer Continuations Contract    -- ^ Action for creating the merkleized bidding contract, and its merkleized continuations.
makeBids _ _ [] _ _ continuation = pure continuation
makeBids _ _ _ [] _ continuation = pure continuation
makeBids bounds assetToken (deadline : remainingDeadlines) bids close continuation =
  do
    continuation' <- merkleizeTimeout continuation
    let
      -- Let a bidder make their bid.
      makeBid bid@(ChoiceId _ bidder) =
        do
          let
            bidAmount = ChoiceValue bid
          -- Handle the remaining bids and finalization if the bidder is not disqualified.
          remaining <-
            makeBids bounds assetToken remainingDeadlines bids close
              $ When
                [
                  -- Wait for a notification before continuing.
                  Case (Notify TrueObs)
                    -- Make the payment for the asset.
                    $ Pay bidder (Account seller) ada bidAmount
                    $ Pay seller (Party bidder) assetToken assetAmount
                    -- Close the contract.
                    close
                ]
                timeoutFinal Close
          -- Handle the remaining bids and finalization if the bidder is disqualified.
          disqualify <- makeBids bounds assetToken remainingDeadlines (filter (/= bid) bids) close continuation'
          disqualify' <- merkleizeTimeout disqualify
          -- Require a deposit if a bid was made.
          deposit <-
            deepMerkleize
              $ When
                [
                  -- Deposit the Lovelace for the bid.
                  Case (Deposit bidder bidder ada . SubValue (AddValue bidAmount minAda) $ AvailableMoney bidder ada)
                    -- Record the new highest amount.
                    $ Let highestBid bidAmount
                    -- Handle the remaining bids.
                      remaining
                ]
                -- Ignore the bid and disqualify the bidder if the deposit was not made.
                deadline
                disqualify'
          pure
            $ Case (Choice bid [bounds])
              -- Check if the bid is highest so far.
              $ If (ValueGT bidAmount $ UseValue highestBid)
                  -- Require a deposit if the bid is highest.
                  deposit
                  -- Ignore the bid and disqualify the bidder if it is not highest.
                  disqualify
    cs <- mapM makeBid bids
    deepMerkleize
      $ When cs 
      -- End the bidding if no one bids in this round.
      deadline
      continuation'

-- | Make an explicit refund.
makeRefunds :: [Party]                        -- ^ The parties to refund.
            -> Contract                       -- ^ The contract to be executed after the refund.
            -> Writer Continuations Contract  -- ^ Action to merklieze the continuation.
makeRefunds parties continuation =
  deepMerkleize
    $ When
      [
        -- Wait for a notification before continuing.
        Case (Notify TrueObs)
          -- Pay everyone.
          $ flip (foldr (\party -> Pay party (Party party) ada (AvailableMoney party ada))) parties
            continuation
      ]
      timeoutFinal Close

-- | Merkleize a timeout continuations.
merkleizeTimeout :: Contract                       -- ^ The continuation to be merkleized.
                 -> Writer Continuations Contract  -- ^ Action to merklieze the continuation.
merkleizeTimeout continuation =
  deepMerkleize
    $ When [Case (Notify TrueObs) continuation] timeoutFinal Close

EOI
```

👉 Lưu ý rằng hợp đồng trên có lỗ hổng đa đầu vào, trong đó một nhà thầu có thể gửi tất cả các lần đấu giá của hợp đồng trong một giao dịch duy nhất, do đó khóa các nhà thầu khác không thể tham gia. Để đơn giản thay đổi hợp đồng nhằm ngăn chặn điều này, bạn có thể giới hạn mỗi giao dịch chỉ cho phép một lần đấu giá duy nhất từ một nhà thầu.

Sau đó, hãy chạy chương trình Haskell để sinh ra hợp đồng và các tiếp tục.

In \[9]:

```
time runhaskell english-auction.hs
```

```
real	0m8.243s
user	0m7.586s
sys	0m0.664s
```

Hợp đồng ban đầu rất ngắn gọn.

In \[10]:

```
json2yaml contract.json
```

```
timeout: 1678591569000
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
  merkleized_then: 811ac63b42c08040914c1e057c095ffb66237dfc41f3a368d53ff38f2d1d2fdd
```

Tuy nhiên, có hơn ba nghìn tiếp tục `(continuations)` của hợp đồng.

In \[11]:

```
jq -r '. | length' continuations.json
```

```
3085
```

Các `continuation` Chiếm khoảng bốn megabyte dung lượng lưu trữ, điều này vượt xa khả năng lưu trữ trên chuỗi trong một datum duy nhất.

In \[12]:

```
ls -lh continuations.json
```

```
-rw-rw-r-- 1 bbush bbush-upg 4.1M Mar 11 15:26 continuations.json
```

Dưới đây là một ví dụ về hình dạng của một trong các `continuation`:

In \[13]:

```
jq '. | to_entries | .[0]' continuations.json | json2yaml
```

```
key: 0
value:
- 000be6a30ba558e0bdf60ae859991a48074f114a4044b0b2a702e621d5e6b879
- else:
    timeout: 1678699569000
    timeout_continuation:
      timeout: 1678595169000
      timeout_continuation: close
      when:
      - case:
          notify_if: true
        merkleized_then: 91027f9e2893a55c0db5c7fc38f696516fb2419eb319930bf82f979197644f2d
    when:
    - case:
        choose_between:
        - from: 2000000
          to: 1000000000000
        for_choice:
          choice_name: Bid
          choice_owner:
            role_token: e.cary
      merkleized_then: fca4e7a126a77e415a9f61a4511024918cdc4507afcd03cb9921c299d60db31a
    - case:
        choose_between:
        - from: 2000000
          to: 1000000000000
        for_choice:
          choice_name: Bid
          choice_owner:
            role_token: m.herbert
      merkleized_then: 8c67f0ac70139fa1e5504dafaf22a4d1c34f22cdee7f44d4e4fe32c0894cc282
  if:
    gt:
      use_value: Highest Bid
    value:
      value_of_choice:
        choice_name: Bid
        choice_owner:
          role_token: j.webster
  then:
    timeout: 1678681569000
    timeout_continuation:
      timeout: 1678595169000
      timeout_continuation: close
      when:
      - case:
          notify_if: true
        merkleized_then: 2175194962b7d501a9d45df4fdda77af733f4906f314531b58b7a27d29181a66
    when:
    - case:
        deposits:
          minus:
            amount_of_token:
              currency_symbol: ''
              token_name: ''
            in_account:
              role_token: j.webster
          value:
            add:
              value_of_choice:
                choice_name: Bid
                choice_owner:
                  role_token: j.webster
            and: 2000000
        into_account:
          role_token: j.webster
        of_token:
          currency_symbol: ''
          token_name: ''
        party:
          role_token: j.webster
      merkleized_then: e767ee6b7876937ec0d3140f5679541109650b5644b27f5c02fb97a6b6859472
```

Chúng ta bắt đầu hợp đồng trong trạng thái ban đầu chứa số lượng ADA tối thiểu trong UTxO cho địa chỉ script.

In \[14]:

```
yaml2json << EOI > state.json
accounts:
- - - address: ${ROLE_ADDR[c.marlowe]}
    - currency_symbol: ''
      token_name: ''
  - $MIN_ADA
boundValues: []
choices: []
minTime: 1
EOI
cat state.json
```

```
{"accounts":[[[{"address":"addr1q8tntkszteptml4e9ce9l3fsmgavwv4ywunvdnhxv6nw5ksvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60stayv90"},{"currency_symbol":"","token_name":""}],2000000]],"boundValues":[],"choices":[],"minTime":1}
```

### Giao dịch số 1: Tạo hợp đồng

Chúng ta cung cấp các vai trò tiền tệ khi khởi tạo tệp dữ liệu trạng thái hợp đồng.

In \[15]:

```
marlowe-cli run initialize \
  --mainnet \
  --contract-file contract.json \
  --state-file state.json \
  --roles-currency "$ROLES_CURRENCY" \
  --at-address addr1z9l4w7djneh0kss4drg2php6ynflsvmal7x3w5nrc95uvhz7e4q926apsvcd6kn33cpx95k8jsmrj7v0k62rczvz8urqrl2z0l \
  --out-file tmp.marlowe \
  --print-stats
```

```
Searching for reference script at address: addr1z9l4w7djneh0kss4drg2php6ynflsvmal7x3w5nrc95uvhz7e4q926apsvcd6kn33cpx95k8jsmrj7v0k62rczvz8urqrl2z0l

Expected reference script hash: "2ed2631dbb277c84334453c5c437b86325d371f0835a28b910a91a6e"

Searching for reference script at address: addr1z9l4w7djneh0kss4drg2php6ynflsvmal7x3w5nrc95uvhz7e4q926apsvcd6kn33cpx95k8jsmrj7v0k62rczvz8urqrl2z0l

Expected reference script hash: "e165610232235bbbbeff5b998b233daae42979dec92a6722d9cda989"

Validator size: 12296
Base-validator cost: ExBudget {exBudgetCPU = ExCPU 18745100, exBudgetMemory = ExMemory 81600}
```

Công cụ marlowe-cli vẫn chưa hỗ trợ nhập các `continuation` đã merkle hóa, vì vậy chúng ta sẽ chèn chúng vào tệp dữ liệu trạng thái hợp đồng (**contract-state**).

In \[16]:

```
jq -s '.[0] * {tx : {continuations : .[1]}}' tmp.marlowe continuations.json > marlowe-1.json
```

Gửi giao dịch để tạo hợp đồng.

In \[17]:

```
TX_1=$(
marlowe-cli run auto-execute \
  --mainnet \
  --marlowe-out-file marlowe-1.json \
  --change-address "${ROLE_ADDR[c.marlowe]}" \
  --required-signer "${ROLE_SKEY[c.marlowe]}" \
  --out-file /dev/null \
  --submit 600 \
  --print-stats \
| sed -e 's/^TxId "\(.*\)"$/\1/' \
)
echo "TX_1 = $TX_1"
```

```
Fee: Lovelace 201141
Size: 662 / 16384 = 4%
Execution units:
  Memory: 0 / 14000000 = 0%
  Steps: 0 / 10000000000 = 0%
TX_1 = 15b9ce2788f76dfd867d94abe8e5c9ec88f761cc0b54ca01c4ab31494938b352
```

Xem giao dịch qua trình khám phá dữ liệu blockchain

In \[18]:

```
echo "https://cardanoscan.io/transaction/$TX_1?tab=utxo"
```

```
https://cardanoscan.io/transaction/15b9ce2788f76dfd867d94abe8e5c9ec88f761cc0b54ca01c4ab31494938b352?tab=utxo
```

### Transaction 2. Christopher Marlowe nạp token BearGarden token <a href="#transaction-2.-christopher-marlowe-deposits-the-beargarden-token" id="transaction-2.-christopher-marlowe-deposits-the-beargarden-token"></a>

Chuẩn bị giao dịch để gửi token đang được đấu giá.

In \[19]:

```
marlowe-cli run prepare \
  --marlowe-file marlowe-1.json \
  --out-file marlowe-2.json \
  --deposit-party c.marlowe \
  --deposit-account c.marlowe \
  --deposit-token "$FUNGIBLES_POLICY.BearGarden" \
  --deposit-amount 1 \
  --invalid-before "$((1000 * ($(date -u +%s) - 2 * 60)))" \
  --invalid-hereafter "$((1000 * ($(date -u +%s) + 8 * 60)))" \
  --print-stats
```

```
Rounding  `TransactionInput` txInterval boundries to:(POSIXTime {getPOSIXTime = 1678573702000},POSIXTime {getPOSIXTime = 1678574302999})
TransactionInput {txInterval = (POSIXTime {getPOSIXTime = 1678573702000},POSIXTime {getPOSIXTime = 1678574302999}), txInputs = [NormalInput (IDeposit "c.marlowe" "c.marlowe" (Token "8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d" "BearGarden") 1)]}

Datum size: 718
```

Hiện tại, hợp đồng chứa ADA ban đầu và token gốc, cả hai đều trong tài khoản của người bán.

In \[20]:

```
jq .tx.state.accounts marlowe-2.json | json2yaml
```

```
- - - address: addr1q8tntkszteptml4e9ce9l3fsmgavwv4ywunvdnhxv6nw5ksvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60stayv90
    - currency_symbol: ''
      token_name: ''
  - 2000000
- - - role_token: c.marlowe
    - currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
      token_name: BearGarden
  - 1
```

Gửi giao dịch.

In \[21]:

```
TX_2=$(
marlowe-cli run auto-execute \
  --mainnet \
  --marlowe-in-file marlowe-1.json \
  --marlowe-out-file marlowe-2.json \
  --tx-in-marlowe "$TX_1#1" \
  --change-address "${ROLE_ADDR[c.marlowe]}" \
  --required-signer "${ROLE_SKEY[c.marlowe]}" \
  --out-file /dev/null \
  --submit 600 \
  --print-stats \
| sed -e 's/^TxId "\(.*\)"$/\1/' \
)
echo "TX_2 = $TX_2"
```

```
Fee: Lovelace 940468
Size: 2314 / 16384 = 14%
Execution units:
  Memory: 8545440 / 14000000 = 61%
  Steps: 2299285045 / 10000000000 = 22%
TX_2 = 99bc07aa0284bdb0376eff6437926ceac31e094c8cb8950539de168471a879f1
```

Xem giao dịch qua trình khám phá dữ liệu blockchain.

In \[22]:

```
echo "https://cardanoscan.io/transaction/$TX_2?tab=utxo"
```

```
https://cardanoscan.io/transaction/99bc07aa0284bdb0376eff6437926ceac31e094c8cb8950539de168471a879f1?tab=utxo
```

### Giao dịch số 3: Mary Herbert đặt giá 5 ada <a href="#transaction-3.-mary-herbert-bids-5-ada" id="transaction-3.-mary-herbert-bids-5-ada"></a>

In \[23]:

```
AMOUNT_1=$((5 * ADA))
echo "AMOUNT_1 = $AMOUNT_1"
```

```
AMOUNT_1 = 5000000
```

In \[24]:

```
marlowe-cli run prepare \
  --marlowe-file marlowe-2.json \
  --out-file marlowe-3.json \
  --choice-party m.herbert \
  --choice-name Bid \
  --choice-number "$AMOUNT_1" \
  --invalid-before "$((1000 * ($(date -u +%s) - 2 * 60)))" \
  --invalid-hereafter "$((1000 * ($(date -u +%s) + 8 * 60)))" \
  --print-stats
```

```
Rounding  `TransactionInput` txInterval boundries to:(POSIXTime {getPOSIXTime = 1678573817000},POSIXTime {getPOSIXTime = 1678574417999})
TransactionInput {txInterval = (POSIXTime {getPOSIXTime = 1678573817000},POSIXTime {getPOSIXTime = 1678574417999}), txInputs = [NormalInput (IChoice (ChoiceId "Bid" "m.herbert") 5000000)]}

Datum size: 472
```

Lần đấu giá đầu tiên đã được ghi lại trong trạng thái.

In \[25]:

```
jq '.tx.state | del(.accounts) | del(.minTime)' marlowe-3.json | json2yaml
```

```
boundValues: []
choices:
- - choice_name: Bid
    choice_owner:
      role_token: m.herbert
  - 5000000
```

Gửi giao dịch.

In \[26]:

```
TX_3=$(
marlowe-cli run auto-execute \
  --mainnet \
  --marlowe-in-file marlowe-2.json \
  --marlowe-out-file marlowe-3.json \
  --tx-in-marlowe "$TX_2#1" \
  --change-address "${ROLE_ADDR[m.herbert]}" \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --out-file /dev/null \
  --submit 600 \
  --print-stats \
| sed -e 's/^TxId "\(.*\)"$/\1/' \
)
echo "TX_3 = $TX_3"
```

```
Fee: Lovelace 1051301
Size: 2610 / 16384 = 15%
Execution units:
  Memory: 9765554 / 14000000 = 69%
  Steps: 2741071460 / 10000000000 = 27%
TX_3 = 9b216f1a87352ff04e80e10b69c1c5afe4ad03aa44d5ddbdf3972a18b24734a1
```

Xem giao dịch qua trình khám phá dữ liệu blockchain.

In \[27]:

```
echo "https://cardanoscan.io/transaction/$TX_3?tab=utxo"
```

```
https://cardanoscan.io/transaction/9b216f1a87352ff04e80e10b69c1c5afe4ad03aa44d5ddbdf3972a18b24734a1?tab=utxo
```

### Giao dịch số 4: Mary Herbert nạp 7 ada <a href="#transaction-4.-mary-herbert-deposits-7-ada" id="transaction-4.-mary-herbert-deposits-7-ada"></a>

Người đấu giá phải gửi số ADA mà họ vừa đấu giá, cùng với yêu cầu về ADA tối thiểu.

In \[28]:

```
DELTA_1=$((AMOUNT_1 + $MIN_ADA))
echo "DELTA_1 = $DELTA_1"
```

```
DELTA_1 = 7000000
```

In \[29]:

```
marlowe-cli run prepare \
  --marlowe-file marlowe-3.json \
  --out-file marlowe-4.json \
  --deposit-account m.herbert \
  --deposit-party m.herbert \
  --deposit-amount "$DELTA_1" \
  --invalid-before "$((1000 * ($(date -u +%s) - 2 * 60)))" \
  --invalid-hereafter "$((1000 * ($(date -u +%s) + 8 * 60)))" \
  --print-stats
```

```
Rounding  `TransactionInput` txInterval boundries to:(POSIXTime {getPOSIXTime = 1678573853000},POSIXTime {getPOSIXTime = 1678574453999})
TransactionInput {txInterval = (POSIXTime {getPOSIXTime = 1678573853000},POSIXTime {getPOSIXTime = 1678574453999}), txInputs = [NormalInput (IDeposit "m.herbert" "m.herbert" (Token "" "") 7000000)]}

Datum size: 795
```

Người đấu giá nạp vào tài khoản của họ.

In \[30]:

```
jq '.tx.state | del(.choices) | del(.minTime)' marlowe-4.json | json2yaml
```

```
accounts:
- - - address: addr1q8tntkszteptml4e9ce9l3fsmgavwv4ywunvdnhxv6nw5ksvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60stayv90
    - currency_symbol: ''
      token_name: ''
  - 2000000
- - - role_token: c.marlowe
    - currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
      token_name: BearGarden
  - 1
- - - role_token: m.herbert
    - currency_symbol: ''
      token_name: ''
  - 7000000
boundValues:
- - Highest Bid
  - 5000000
```

Gửi giao dịch.

In \[31]:

```
TX_4=$(
marlowe-cli run auto-execute \
  --mainnet \
  --marlowe-in-file marlowe-3.json \
  --marlowe-out-file marlowe-4.json \
  --tx-in-marlowe "$TX_3#1" \
  --change-address "${ROLE_ADDR[m.herbert]}" \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --out-file /dev/null \
  --submit 600 \
  --print-stats \
| sed -e 's/^TxId "\(.*\)"$/\1/' \
)
echo "TX_4 = $TX_4"
```

```
Fee: Lovelace 1070450
Size: 2587 / 16384 = 15%
Execution units:
  Memory: 10071658 / 14000000 = 71%
  Steps: 2714088187 / 10000000000 = 27%
TX_4 = b97c0555fd0a8773090cdda3a7270a74121945ac5b8b4e688d5a44a350e92277
```

Xem giao dịch qua trình khám phá dữ liệu blockchain.

In \[32]:

```
echo "https://cardanoscan.io/transaction/$TX_4?tab=utxo"
```

```
https://cardanoscan.io/transaction/b97c0555fd0a8773090cdda3a7270a74121945ac5b8b4e688d5a44a350e92277?tab=utxo
```

### Transaction 5. Elizabeth Cary đặt giá 15 ada <a href="#transaction-5.-elizabeth-cary-bids-15-ada" id="transaction-5.-elizabeth-cary-bids-15-ada"></a>

In \[33]:

```
AMOUNT_2=$((15 * ADA))
echo "AMOUNT_2 = $AMOUNT_2"
```

```
AMOUNT_2 = 15000000
```

In \[34]:

```
marlowe-cli run prepare \
  --marlowe-file marlowe-4.json \
  --out-file marlowe-5.json \
  --choice-party e.cary \
  --choice-name Bid \
  --choice-number $AMOUNT_2 \
  --invalid-before "$((1000 * ($(date -u +%s) - 2 * 60)))" \
  --invalid-hereafter "$((1000 * ($(date -u +%s) + 8 * 60)))" \
  --print-stats
```

```
Rounding  `TransactionInput` txInterval boundries to:(POSIXTime {getPOSIXTime = 1678573943000},POSIXTime {getPOSIXTime = 1678574543999})
TransactionInput {txInterval = (POSIXTime {getPOSIXTime = 1678573943000},POSIXTime {getPOSIXTime = 1678574543999}), txInputs = [NormalInput (IChoice (ChoiceId "Bid" "e.cary") 15000000)]}

Datum size: 534
```

Lượt đấu giá số 2 đã được ghi lại.

In \[35]:

```
jq '.tx.state | del(.accounts) | del(.minTime)' marlowe-5.json | json2yaml
```

```
boundValues:
- - Highest Bid
  - 5000000
choices:
- - choice_name: Bid
    choice_owner:
      role_token: m.herbert
  - 5000000
- - choice_name: Bid
    choice_owner:
      role_token: e.cary
  - 15000000
```

Gửi giao dịch.

In \[36]:

```
TX_5=$(
marlowe-cli run auto-execute \
  --mainnet \
  --marlowe-in-file  marlowe-4.json \
  --marlowe-out-file marlowe-5.json \
  --tx-in-marlowe "$TX_4#1" \
  --change-address "${ROLE_ADDR[e.cary]}" \
  --required-signer "${ROLE_SKEY[e.cary]}" \
  --out-file /dev/null \
  --submit 600 \
  --print-stats \
| sed -e 's/^TxId "\(.*\)"$/\1/' \
)
echo "TX_5 = $TX_5"
```

```
Fee: Lovelace 1121322
Size: 2731 / 16384 = 16%
Execution units:
  Memory: 10619980 / 14000000 = 75%
  Steps: 2954615860 / 10000000000 = 29%
TX_5 = 02891f6ab00e8c1f2f7273edbc409646262aab8e1594d40b233898f1720f7dbd
```

Xem giao dịch qua trình khám phá dữ liệu blockchain.

In \[37]:

```
echo "https://cardanoscan.io/transaction/$TX_5?tab=utxo"
```

```
https://cardanoscan.io/transaction/02891f6ab00e8c1f2f7273edbc409646262aab8e1594d40b233898f1720f7dbd?tab=utxo
```

### Transaction 6. Elizabeth Cary nạp 17 ada <a href="#transaction-6.-elizabeth-cary-deposits-17-ada" id="transaction-6.-elizabeth-cary-deposits-17-ada"></a>

nạp 15 ada đấu giá và số ada _tối thiểu_.

In \[38]:

```
DELTA_2=$((AMOUNT_2 + MIN_ADA))
echo "DELTA_2 = $DELTA_2"
```

```
DELTA_2 = 17000000
```

In \[39]:

```
marlowe-cli run prepare \
  --marlowe-file marlowe-5.json \
  --out-file marlowe-6.json \
  --deposit-account e.cary \
  --deposit-party e.cary \
  --deposit-amount "$DELTA_2" \
  --invalid-before "$((1000 * ($(date -u +%s) - 2 * 60)))" \
  --invalid-hereafter "$((1000 * ($(date -u +%s) + 8 * 60)))" \
  --print-stats
```

```
Rounding  `TransactionInput` txInterval boundries to:(POSIXTime {getPOSIXTime = 1678574099000},POSIXTime {getPOSIXTime = 1678574699999})
TransactionInput {txInterval = (POSIXTime {getPOSIXTime = 1678574099000},POSIXTime {getPOSIXTime = 1678574699999}), txInputs = [NormalInput (IDeposit "e.cary" "e.cary" (Token "" "") 17000000)]}

Warnings:
  TransactionShadowing "Highest Bid" 5000000 15000000
Datum size: 845
```

Khoản nạp của người đấu giá đã xuất hiện trong tài khoản.

In \[40]:

```
jq '.tx.state | del(.choices) | del(.minTime)' marlowe-6.json | json2yaml
```

```
accounts:
- - - address: addr1q8tntkszteptml4e9ce9l3fsmgavwv4ywunvdnhxv6nw5ksvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60stayv90
    - currency_symbol: ''
      token_name: ''
  - 2000000
- - - role_token: c.marlowe
    - currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
      token_name: BearGarden
  - 1
- - - role_token: m.herbert
    - currency_symbol: ''
      token_name: ''
  - 7000000
- - - role_token: e.cary
    - currency_symbol: ''
      token_name: ''
  - 17000000
boundValues:
- - Highest Bid
  - 15000000
```

Gửi giao dịch.

In \[41]:

```
TX_6=$(
marlowe-cli run auto-execute \
  --mainnet \
  --marlowe-in-file marlowe-5.json \
  --marlowe-out-file marlowe-6.json \
  --tx-in-marlowe "$TX_5#1" \
  --change-address "${ROLE_ADDR[e.cary]}" \
  --required-signer "${ROLE_SKEY[e.cary]}" \
  --out-file /dev/null \
  --submit 600 \
  --print-stats \
| sed -e 's/^TxId "\(.*\)"$/\1/' \
)
echo "TX_6 = $TX_6"
```

```
Fee: Lovelace 1146666
Size: 2687 / 16384 = 16%
Execution units:
  Memory: 11018788 / 14000000 = 78%
  Steps: 2952179880 / 10000000000 = 29%
TX_6 = 606f10eb120d6a91d23d50a76aae167d3bb4fb1b444d57d9e03b3a133d57b59e
```

Xem giao dịch qua trình khám phá dữ liệu blockchain.

In \[42]:

```
echo "https://cardanoscan.io/transaction/$TX_6?tab=utxo"
```

```
https://cardanoscan.io/transaction/606f10eb120d6a91d23d50a76aae167d3bb4fb1b444d57d9e03b3a133d57b59e?tab=utxo
```

### Giao dịch số 7: Mary Herbert đặt giá 25 ada <a href="#transaction-7.-mary-herbert-now-bids-25-ada" id="transaction-7.-mary-herbert-now-bids-25-ada"></a>

In \[43]:

```
AMOUNT_3=$((25 * ADA))
echo "AMOUNT_3 = $AMOUNT_3"
```

```
AMOUNT_3 = 25000000
```

In \[44]:

```
marlowe-cli run prepare \
  --marlowe-file marlowe-6.json \
  --out-file marlowe-7.json \
  --choice-party m.herbert \
  --choice-name Bid \
  --choice-number "$AMOUNT_3" \
  --invalid-before "$((1000 * ($(date -u +%s) - 2 * 60)))" \
  --invalid-hereafter "$((1000 * ($(date -u +%s) + 8 * 60)))" \
  --print-stats
```

```
Rounding  `TransactionInput` txInterval boundries to:(POSIXTime {getPOSIXTime = 1678574180000},POSIXTime {getPOSIXTime = 1678574780999})
TransactionInput {txInterval = (POSIXTime {getPOSIXTime = 1678574180000},POSIXTime {getPOSIXTime = 1678574780999}), txInputs = [NormalInput (IChoice (ChoiceId "Bid" "m.herbert") 25000000)]}

Datum size: 572
```

Bản ghi các lựa chọn phản ảnh giá đầu đã tăng.

In \[45]:

```
jq '.tx.state | del(.accounts) | del(.minTime)' marlowe-7.json | json2yaml
```

```
boundValues:
- - Highest Bid
  - 15000000
choices:
- - choice_name: Bid
    choice_owner:
      role_token: m.herbert
  - 25000000
- - choice_name: Bid
    choice_owner:
      role_token: e.cary
  - 15000000
```

Gửi giao dịch.

In \[46]:

```
TX_7=$(
marlowe-cli run auto-execute \
  --mainnet \
  --marlowe-in-file marlowe-6.json \
  --marlowe-out-file marlowe-7.json \
  --tx-in-marlowe "$TX_6#1" \
  --change-address "${ROLE_ADDR[m.herbert]}" \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --out-file /dev/null \
  --submit 600 \
  --print-stats \
| sed -e 's/^TxId "\(.*\)"$/\1/' \
)
echo "TX_7 = $TX_7"
```

```
Fee: Lovelace 1227972
Size: 2873 / 16384 = 17%
Execution units:
  Memory: 11883492 / 14000000 = 84%
  Steps: 3274358628 / 10000000000 = 32%
TX_7 = b091fa392846d921c89039efad7b373087eeaf6040d0dc1756d4e877c3a6e147
```

Xem giao dịch qua trình khám phá dữ liệu blockchain.

In \[47]:

```
echo "https://cardanoscan.io/transaction/$TX_7?tab=utxo"
```

```
https://cardanoscan.io/transaction/b091fa392846d921c89039efad7b373087eeaf6040d0dc1756d4e877c3a6e147?tab=utxo
```

### Transaction 8. Mary Herbert nạp 20 ada <a href="#transaction-8.-mary-herbert-deposits-20-ada" id="transaction-8.-mary-herbert-deposits-20-ada"></a>

Giá đấu mới của Mary cao hơn lần đặt trước 20 adaT.

In \[48]:

```
DELTA_3=$((AMOUNT_3 - AMOUNT_1))
echo "DELTA_3 = $DELTA_3"
```

```
DELTA_3 = 20000000
```

In \[49]:

```
marlowe-cli run prepare \
  --marlowe-file marlowe-7.json \
  --out-file marlowe-8.json \
  --deposit-account m.herbert \
  --deposit-party m.herbert \
  --deposit-amount "$DELTA_3" \
  --invalid-before "$((1000 * ($(date -u +%s) - 2 * 60)))" \
  --invalid-hereafter "$((1000 * ($(date -u +%s) + 8 * 60)))" \
  --print-stats
```

```
Rounding  `TransactionInput` txInterval boundries to:(POSIXTime {getPOSIXTime = 1678574209000},POSIXTime {getPOSIXTime = 1678574809999})
TransactionInput {txInterval = (POSIXTime {getPOSIXTime = 1678574209000},POSIXTime {getPOSIXTime = 1678574809999}), txInputs = [NormalInput (IDeposit "m.herbert" "m.herbert" (Token "" "") 20000000)]}

Warnings:
  TransactionShadowing "Highest Bid" 15000000 25000000
Datum size: 845
```

Tài khoản của người đấu giá thứ 3 đã được cập nhật.

In \[50]:

```
jq '.tx.state | del(.choices) | del(.minTime)' marlowe-8.json | json2yaml
```

```
accounts:
- - - address: addr1q8tntkszteptml4e9ce9l3fsmgavwv4ywunvdnhxv6nw5ksvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60stayv90
    - currency_symbol: ''
      token_name: ''
  - 2000000
- - - role_token: c.marlowe
    - currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
      token_name: BearGarden
  - 1
- - - role_token: m.herbert
    - currency_symbol: ''
      token_name: ''
  - 27000000
- - - role_token: e.cary
    - currency_symbol: ''
      token_name: ''
  - 17000000
boundValues:
- - Highest Bid
  - 25000000
```

Gửi giao dịch.

In \[51]:

```
TX_8=$(
marlowe-cli run auto-execute \
  --mainnet \
  --marlowe-in-file marlowe-7.json \
  --marlowe-out-file marlowe-8.json \
  --tx-in-marlowe "$TX_7#1" \
  --change-address "${ROLE_ADDR[m.herbert]}" \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --out-file /dev/null \
  --submit 600 \
  --print-stats \
| sed -e 's/^TxId "\(.*\)"$/\1/' \
)
echo "TX_8 = $TX_8"
```

```
Fee: Lovelace 1184240
Size: 2737 / 16384 = 16%
Execution units:
  Memory: 11484904 / 14000000 = 82%
  Steps: 3069780365 / 10000000000 = 30%
TX_8 = bdc1b62ff68486230fd5af2a7e9aa0b3257a8a9b2aaebbd205c12bbd7964b7e6
```

Xem giao dịch qua trình khám phá dữ liệu blockchain.

In \[52]:

```
echo "https://cardanoscan.io/transaction/$TX_8?tab=utxo"
```

```
https://cardanoscan.io/transaction/bdc1b62ff68486230fd5af2a7e9aa0b3257a8a9b2aaebbd205c12bbd7964b7e6?tab=utxo
```

### Transaction 9. Francis Beaumont đặt giá 30 ada <a href="#transaction-9.-francis-beaumont-bids-30-ada" id="transaction-9.-francis-beaumont-bids-30-ada"></a>

In \[53]:

```
AMOUNT_4=$((30 * ADA))
echo "AMOUNT_4 = $AMOUNT_4"
```

```
AMOUNT_4 = 30000000
```

In \[54]:

```
marlowe-cli run prepare \
  --marlowe-file marlowe-8.json \
  --out-file marlowe-9.json\
  --choice-party f.beaumont \
  --choice-name Bid \
  --choice-number "$AMOUNT_4" \
  --invalid-before "$((1000 * ($(date -u +%s) - 2 * 60)))" \
  --invalid-hereafter "$((1000 * ($(date -u +%s) + 8 * 60)))" \
  --print-stats
```

```
Rounding  `TransactionInput` txInterval boundries to:(POSIXTime {getPOSIXTime = 1678574277000},POSIXTime {getPOSIXTime = 1678574877999})
TransactionInput {txInterval = (POSIXTime {getPOSIXTime = 1678574277000},POSIXTime {getPOSIXTime = 1678574877999}), txInputs = [NormalInput (IChoice (ChoiceId "Bid" "f.beaumont") 30000000)]}

Datum size: 604
```

Khoản đặt mới đã được ghi nhận.

In \[55]:

```
jq '.tx.state | del(.accounts) | del(.minTime)' marlowe-9.json | json2yaml
```

```
boundValues:
- - Highest Bid
  - 25000000
choices:
- - choice_name: Bid
    choice_owner:
      role_token: m.herbert
  - 25000000
- - choice_name: Bid
    choice_owner:
      role_token: e.cary
  - 15000000
- - choice_name: Bid
    choice_owner:
      role_token: f.beaumont
  - 30000000
```

Gửi giao dịch.

In \[56]:

```
TX_9=$(
marlowe-cli run auto-execute \
  --mainnet \
  --marlowe-in-file marlowe-8.json \
  --marlowe-out-file marlowe-9.json \
  --tx-in-marlowe "$TX_8#1" \
  --change-address "${ROLE_ADDR[f.beaumont]}" \
  --required-signer "${ROLE_SKEY[f.beaumont]}" \
  --out-file /dev/null \
  --submit 600 \
  --print-stats \
| sed -e 's/^TxId "\(.*\)"$/\1/' \
)
echo "TX_9 = $TX_9"
```

```
Fee: Lovelace 1179348
Size: 2875 / 16384 = 17%
Execution units:
  Memory: 11302154 / 14000000 = 80%
  Steps: 3125608537 / 10000000000 = 31%
TX_9 = e2371fd4a6c9e3a60fdfb9ee20d3bd53ec66151db6077d676be571a376536ad0
```

Xem giao dịch qua trình khám phá dữ liệu blockchain.

In \[57]:

```
echo "https://cardanoscan.io/transaction/$TX_9?tab=utxo"
```

```
https://cardanoscan.io/transaction/e2371fd4a6c9e3a60fdfb9ee20d3bd53ec66151db6077d676be571a376536ad0?tab=utxo
```

### Transaction 10. Francis Beaumont nạp 32 ada <a href="#transaction-10.-francis-beaumont-deposits-32-ada" id="transaction-10.-francis-beaumont-deposits-32-ada"></a>

Francis nạp 30 ada đấu giá and 2 ada tối thiểu.

In \[58]:

```
DELTA_4=$((AMOUNT_4 + MIN_ADA))
echo "DELTA_4 = $DELTA_4"
```

```
DELTA_4 = 32000000
```

In \[59]:

```
marlowe-cli run prepare \
  --marlowe-file marlowe-9.json \
  --out-file marlowe-10.json \
  --deposit-account f.beaumont \
  --deposit-party f.beaumont \
  --deposit-amount "$DELTA_4" \
  --invalid-before "$((1000 * ($(date -u +%s) - 2 * 60)))" \
  --invalid-hereafter "$((1000 * ($(date -u +%s) + 8 * 60)))" \
  --print-stats
```

```
Rounding  `TransactionInput` txInterval boundries to:(POSIXTime {getPOSIXTime = 1678574433000},POSIXTime {getPOSIXTime = 1678575033999})
TransactionInput {txInterval = (POSIXTime {getPOSIXTime = 1678574433000},POSIXTime {getPOSIXTime = 1678575033999}), txInputs = [NormalInput (IDeposit "f.beaumont" "f.beaumont" (Token "" "") 32000000)]}

Warnings:
  TransactionShadowing "Highest Bid" 25000000 30000000
Datum size: 903
```

Tài khoản đấu giá của họ đã được ghi nhận khoản nạp.

In \[60]:

```
jq '.tx.state | del(.choices) | del(.minTime)' marlowe-10.json | json2yaml
```

```
accounts:
- - - address: addr1q8tntkszteptml4e9ce9l3fsmgavwv4ywunvdnhxv6nw5ksvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60stayv90
    - currency_symbol: ''
      token_name: ''
  - 2000000
- - - role_token: c.marlowe
    - currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
      token_name: BearGarden
  - 1
- - - role_token: m.herbert
    - currency_symbol: ''
      token_name: ''
  - 27000000
- - - role_token: e.cary
    - currency_symbol: ''
      token_name: ''
  - 17000000
- - - role_token: f.beaumont
    - currency_symbol: ''
      token_name: ''
  - 32000000
boundValues:
- - Highest Bid
  - 30000000
```

Gửi giao dịch.

In \[61]:

```
TX_10=$(
marlowe-cli run auto-execute \
  --mainnet \
  --marlowe-in-file marlowe-9.json \
  --marlowe-out-file marlowe-10.json \
  --tx-in-marlowe "$TX_9#1" \
  --change-address "${ROLE_ADDR[f.beaumont]}" \
  --required-signer "${ROLE_SKEY[f.beaumont]}" \
  --out-file /dev/null \
  --submit 600 \
  --print-stats \
| sed -e 's/^TxId "\(.*\)"$/\1/' \
)
echo "TX_10 = $TX_10"
```

```
Fee: Lovelace 1216330
Size: 2831 / 16384 = 17%
Execution units:
  Memory: 11853596 / 14000000 = 84%
  Steps: 3162435568 / 10000000000 = 31%
TX_10 = 2f8e134d5db966374e1b910ab08327a32610a8af2dd1c9300333f23bee8275b3
```

Xem giao dịch qua trình khám phá dữ liệu blockchain.

In \[62]:

```
echo "https://cardanoscan.io/transaction/$TX_10?tab=utxo"
```

```
https://cardanoscan.io/transaction/2f8e134d5db966374e1b910ab08327a32610a8af2dd1c9300333f23bee8275b3?tab=utxo
```

### Transaction 11. Elizabeth Cary đặt 40 ada <a href="#transaction-11.-elizabeth-cary-bids-40-ada" id="transaction-11.-elizabeth-cary-bids-40-ada"></a>

In \[63]:

```
AMOUNT_5=$((40 * ADA))
echo "AMOUNT_5 = $AMOUNT_5"
```

```
AMOUNT_5 = 40000000
```

In \[64]:

```
marlowe-cli run prepare \
  --marlowe-file marlowe-10.json \
  --out-file marlowe-11.json \
  --choice-party e.cary \
  --choice-name Bid \
  --choice-number "$AMOUNT_5" \
  --invalid-before "$((1000 * ($(date -u +%s) - 2 * 60)))" \
  --invalid-hereafter "$((1000 * ($(date -u +%s) + 8 * 60)))" \
  --print-stats
```

```
Rounding  `TransactionInput` txInterval boundries to:(POSIXTime {getPOSIXTime = 1678574481000},POSIXTime {getPOSIXTime = 1678575081999})
TransactionInput {txInterval = (POSIXTime {getPOSIXTime = 1678574481000},POSIXTime {getPOSIXTime = 1678575081999}), txInputs = [NormalInput (IChoice (ChoiceId "Bid" "e.cary") 40000000)]}

Datum size: 618
```

Khoản đặt giá mới đã được ghi nhận.

In \[65]:

```
jq '.tx.state | del(.accounts) | del(.minTime)' marlowe-11.json | json2yaml
```

```
boundValues:
- - Highest Bid
  - 30000000
choices:
- - choice_name: Bid
    choice_owner:
      role_token: m.herbert
  - 25000000
- - choice_name: Bid
    choice_owner:
      role_token: e.cary
  - 40000000
- - choice_name: Bid
    choice_owner:
      role_token: f.beaumont
  - 30000000
```

Gửi giao dịch.

In \[66]:

```
TX_11=$(
marlowe-cli run auto-execute \
  --mainnet \
  --marlowe-in-file marlowe-10.json \
  --marlowe-out-file marlowe-11.json \
  --tx-in-marlowe "$TX_10#1" \
  --change-address "${ROLE_ADDR[e.cary]}" \
  --required-signer "${ROLE_SKEY[e.cary]}" \
  --out-file /dev/null \
  --submit 600 \
  --print-stats \
| sed -e 's/^TxId "\(.*\)"$/\1/' \
)
echo "TX_11 = $TX_11"
```

```
Fee: Lovelace 1289127
Size: 2959 / 16384 = 18%
Execution units:
  Memory: 12640282 / 14000000 = 90%
  Steps: 3464428923 / 10000000000 = 34%
TX_11 = e5da014a844a62e6e3f9add823b8a4f5d25a54ce62194846083f9012bf797c1a
```

Xem giao dịch qua trình khám phá dữ liệu blockchain..

In \[67]:

```
echo "https://cardanoscan.io/transaction/$TX_11?tab=utxo"
```

```
https://cardanoscan.io/transaction/e5da014a844a62e6e3f9add823b8a4f5d25a54ce62194846083f9012bf797c1a?tab=utxo
```

### Giao dịch số 12: Elizabeth Cary nạp 25 ada <a href="#transaction-12.-elizabeth-cary-deposits-25-ada" id="transaction-12.-elizabeth-cary-deposits-25-ada"></a>

Do Elizabeth đã nạp 15 ada trước đó (giao dịch số 5), nên Eli chỉ cần nạp thêm 25 ada để khớp với khoản 40 ada đặt giá mới.

In \[68]:

```
DELTA_5=$((AMOUNT_5 - AMOUNT_2))
echo "DELTA_5 = $DELTA_5"
```

```
DELTA_5 = 25000000
```

In \[69]:

```
marlowe-cli run prepare \
  --marlowe-file marlowe-11.json \
  --out-file marlowe-12.json \
  --deposit-account e.cary \
  --deposit-party e.cary \
  --deposit-amount "$DELTA_5" \
  --invalid-before "$((1000 * ($(date -u +%s) - 2 * 60)))" \
  --invalid-hereafter "$((1000 * ($(date -u +%s) + 8 * 60)))" \
  --print-stats
```

```
Rounding  `TransactionInput` txInterval boundries to:(POSIXTime {getPOSIXTime = 1678574523000},POSIXTime {getPOSIXTime = 1678575123999})
TransactionInput {txInterval = (POSIXTime {getPOSIXTime = 1678574523000},POSIXTime {getPOSIXTime = 1678575123999}), txInputs = [NormalInput (IDeposit "e.cary" "e.cary" (Token "" "") 25000000)]}

Warnings:
  TransactionShadowing "Highest Bid" 30000000 40000000
Datum size: 903
```

Khoản nạp đã được ghi nhận.

In \[70]:

```
jq '.tx.state | del(.choices) | del(.minTime)' marlowe-12.json | json2yaml
```

```
accounts:
- - - address: addr1q8tntkszteptml4e9ce9l3fsmgavwv4ywunvdnhxv6nw5ksvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60stayv90
    - currency_symbol: ''
      token_name: ''
  - 2000000
- - - role_token: c.marlowe
    - currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
      token_name: BearGarden
  - 1
- - - role_token: m.herbert
    - currency_symbol: ''
      token_name: ''
  - 27000000
- - - role_token: e.cary
    - currency_symbol: ''
      token_name: ''
  - 42000000
- - - role_token: f.beaumont
    - currency_symbol: ''
      token_name: ''
  - 32000000
boundValues:
- - Highest Bid
  - 40000000
```

Gửi giao dịch.

In \[71]:

```
TX_12=$(
marlowe-cli run auto-execute \
  --mainnet \
  --marlowe-in-file marlowe-11.json \
  --marlowe-out-file marlowe-12.json \
  --tx-in-marlowe "$TX_11#1" \
  --change-address "${ROLE_ADDR[e.cary]}" \
  --required-signer "${ROLE_SKEY[e.cary]}" \
  --out-file /dev/null \
  --submit 600 \
  --print-stats \
| sed -e 's/^TxId "\(.*\)"$/\1/' \
)
echo "TX_12 = $TX_12"
```

```
Fee: Lovelace 1249145
Size: 2829 / 16384 = 17%
Execution units:
  Memory: 12288198 / 14000000 = 87%
  Steps: 3270986942 / 10000000000 = 32%
TX_12 = 4876991aa2b2586bbc86a920a2cc65a73cc53aaf8fcfb9423c77ebbb7fff7b41
```

Xem giao dịch qua trình khám phá dữ liệu blockchain.

In \[72]:

```
echo "https://cardanoscan.io/transaction/$TX_12?tab=utxo"
```

```
https://cardanoscan.io/transaction/4876991aa2b2586bbc86a920a2cc65a73cc53aaf8fcfb9423c77ebbb7fff7b41?tab=utxo
```

### Giao dịch số 13: Mary Herbert đặt 50 ada <a href="#transaction-13.-mary-herbert-bids-50-ada" id="transaction-13.-mary-herbert-bids-50-ada"></a>

In \[73]:

```
AMOUNT_6=$((50 * ADA))
echo "AMOUNT_6 = $AMOUNT_6"
```

```
AMOUNT_6 = 50000000
```

In \[74]:

```
marlowe-cli run prepare \
  --marlowe-file marlowe-12.json \
  --out-file marlowe-13.json \
  --choice-party m.herbert \
  --choice-name Bid \
  --choice-number "$AMOUNT_6" \
  --invalid-before "$((1000 * ($(date -u +%s) - 2 * 60)))" \
  --invalid-hereafter "$((1000 * ($(date -u +%s) + 8 * 60)))" \
  --print-stats
```

```
Rounding  `TransactionInput` txInterval boundries to:(POSIXTime {getPOSIXTime = 1678574561000},POSIXTime {getPOSIXTime = 1678575161999})
TransactionInput {txInterval = (POSIXTime {getPOSIXTime = 1678574561000},POSIXTime {getPOSIXTime = 1678575161999}), txInputs = [NormalInput (IChoice (ChoiceId "Bid" "m.herbert") 50000000)]}

Datum size: 630
```

Khoản đấu giá mới đã được ghi nhận.

In \[75]:

```
jq '.tx.state | del(.accounts) | del(.minTime)' marlowe-13.json | json2yaml
```

```
boundValues:
- - Highest Bid
  - 40000000
choices:
- - choice_name: Bid
    choice_owner:
      role_token: m.herbert
  - 50000000
- - choice_name: Bid
    choice_owner:
      role_token: e.cary
  - 40000000
- - choice_name: Bid
    choice_owner:
      role_token: f.beaumont
  - 30000000
```

Gửi giao dịch.

In \[76]:

```
TX_13=$(
marlowe-cli run auto-execute \
  --mainnet \
  --marlowe-in-file marlowe-12.json \
  --marlowe-out-file marlowe-13.json \
  --tx-in-marlowe "$TX_12#1" \
  --change-address "${ROLE_ADDR[m.herbert]}" \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --out-file /dev/null \
  --submit 600 \
  --print-stats \
| sed -e 's/^TxId "\(.*\)"$/\1/' \
)
echo "TX_13 = $TX_13"
```

```
Fee: Lovelace 1201223
Size: 2641 / 16384 = 16%
Execution units:
  Memory: 11725706 / 14000000 = 83%
  Steps: 3171209232 / 10000000000 = 31%
TX_13 = fd67859eb6ae0a2a0496460ebca996292a2db3294bd45792d9fcc061294dc55a
```

Xem giao dịch qua trình khám phá dữ liệu blockchain.

In \[77]:

```
echo "https://cardanoscan.io/transaction/$TX_13?tab=utxo"
```

```
https://cardanoscan.io/transaction/fd67859eb6ae0a2a0496460ebca996292a2db3294bd45792d9fcc061294dc55a?tab=utxo
```

### Giao dịch số 14: Mary Herbert nạp 25 ada và phiên đấu giá kết thúc <a href="#transaction-14.-mary-herbert-deposits-25-ada-and-the-bidding-is-over" id="transaction-14.-mary-herbert-deposits-25-ada-and-the-bidding-is-over"></a>

Mary nạp thêm 25 ADA để số tiền gửi phù hợp với mức đấu giá cộng với yêu cầu về ADA tối thiểu.

Cuộc đấu giá được giới hạn ở sáu vòng đấu giá.

In \[78]:

```
DELTA_6=$((AMOUNT_6 - AMOUNT_3))
echo "DELTA_6 = $DELTA_6"
```

```
DELTA_6 = 25000000
```

In \[79]:

```
marlowe-cli run prepare \
  --marlowe-file marlowe-13.json \
  --out-file marlowe-14.json \
  --deposit-account m.herbert \
  --deposit-party m.herbert \
  --deposit-amount "$DELTA_6" \
  --invalid-before "$((1000 * ($(date -u +%s) - 2 * 60)))" \
  --invalid-hereafter "$((1000 * ($(date -u +%s) + 8 * 60)))" \
  --print-stats
```

```
Rounding  `TransactionInput` txInterval boundries to:(POSIXTime {getPOSIXTime = 1678574643000},POSIXTime {getPOSIXTime = 1678575243999})
TransactionInput {txInterval = (POSIXTime {getPOSIXTime = 1678574643000},POSIXTime {getPOSIXTime = 1678575243999}), txInputs = [NormalInput (IDeposit "m.herbert" "m.herbert" (Token "" "") 25000000)]}

Warnings:
  TransactionShadowing "Highest Bid" 40000000 50000000
Datum size: 471
```

Khoản đặt giá mới được ghi nhận.

In \[80]:

```
jq '.tx.state | del(.choices) | del(.minTime)' marlowe-14.json | json2yaml
```

```
accounts:
- - - address: addr1q8tntkszteptml4e9ce9l3fsmgavwv4ywunvdnhxv6nw5ksvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60stayv90
    - currency_symbol: ''
      token_name: ''
  - 2000000
- - - role_token: c.marlowe
    - currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
      token_name: BearGarden
  - 1
- - - role_token: m.herbert
    - currency_symbol: ''
      token_name: ''
  - 52000000
- - - role_token: e.cary
    - currency_symbol: ''
      token_name: ''
  - 42000000
- - - role_token: f.beaumont
    - currency_symbol: ''
      token_name: ''
  - 32000000
boundValues:
- - Highest Bid
  - 50000000
```

Gửi giao dịch.

In \[81]:

```
TX_14=$(
marlowe-cli run auto-execute \
  --mainnet \
  --marlowe-in-file marlowe-13.json \
  --marlowe-out-file marlowe-14.json \
  --tx-in-marlowe "$TX_13#1" \
  --change-address "${ROLE_ADDR[m.herbert]}" \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --out-file /dev/null \
  --submit 600 \
  --print-stats \
| sed -e 's/^TxId "\(.*\)"$/\1/' \
)
echo "TX_14 = $TX_14"
```

```
Fee: Lovelace 1104754
Size: 1989 / 16384 = 12%
Execution units:
  Memory: 10952962 / 14000000 = 78%
  Steps: 2849516495 / 10000000000 = 28%
TX_14 = 7f4a496bfbd83fb68b6637143b8547d607b896d5ebd15d9db9d8f0f22886f199
```

Xem giao dịch qua trình khám phá dữ liệu blockchain.

In \[82]:

```
echo "https://cardanoscan.io/transaction/$TX_14?tab=utxo"
```

```
https://cardanoscan.io/transaction/7f4a496bfbd83fb68b6637143b8547d607b896d5ebd15d9db9d8f0f22886f199?tab=utxo
```

### Giao dịch 15. Thanh toán token BearGarden cho Mary Herbert và ghi "_CÓ"_ tài khoản của Christopher Marlowe với số ADA đã thanh toán cho token đó.

Bất kỳ ai cũng có thể tiến hành hợp đồng bằng cách thông báo cho nó. Những thông báo này là cần thiết để giữ cho chi phí thực thi của hợp đồng dưới giới hạn chi phí thực thi của Plutus.

In \[83]:

```
marlowe-cli run prepare \
  --marlowe-file marlowe-14.json \
  --out-file marlowe-15.json \
  --notify \
  --invalid-before "$((1000 * ($(date -u +%s) - 2 * 60)))" \
  --invalid-hereafter "$((1000 * ($(date -u +%s) + 8 * 60)))" \
  --print-stats
```

```
Rounding  `TransactionInput` txInterval boundries to:(POSIXTime {getPOSIXTime = 1678574676000},POSIXTime {getPOSIXTime = 1678575276999})
TransactionInput {txInterval = (POSIXTime {getPOSIXTime = 1678574676000},POSIXTime {getPOSIXTime = 1678575276999}), txInputs = [NormalInput INotify]}

Datum size: 436
Payment 1
  Acccount: "m.herbert"
  Payee: Account "c.marlowe"
  Ada: Lovelace {getLovelace = 50000000}
Payment 2
  Acccount: "c.marlowe"
  Payee: Party "m.herbert"
  Ada: Lovelace {getLovelace = 0}
  8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d."BearGarden": 1
```

Hợp đồng chứa các bản ghi về quỹ _nợ_ của các bên.

In \[84]:

```
jq .tx.state.accounts marlowe-15.json | json2yaml
```

```
- - - address: addr1q8tntkszteptml4e9ce9l3fsmgavwv4ywunvdnhxv6nw5ksvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60stayv90
    - currency_symbol: ''
      token_name: ''
  - 2000000
- - - role_token: m.herbert
    - currency_symbol: ''
      token_name: ''
  - 2000000
- - - role_token: e.cary
    - currency_symbol: ''
      token_name: ''
  - 42000000
- - - role_token: f.beaumont
    - currency_symbol: ''
      token_name: ''
  - 32000000
- - - role_token: c.marlowe
    - currency_symbol: ''
      token_name: ''
  - 50000000
```

Gửi giao dịch.

In \[85]:

```
TX_15=$(
marlowe-cli run auto-execute \
  --mainnet \
  --marlowe-in-file marlowe-14.json \
  --marlowe-out-file marlowe-15.json \
  --tx-in-marlowe "$TX_14#1" \
  --change-address "${ROLE_ADDR[c.marlowe]}" \
  --required-signer "${ROLE_SKEY[c.marlowe]}" \
  --out-file /dev/null \
  --submit 600 \
  --print-stats \
| sed -e 's/^TxId "\(.*\)"$/\1/' \
)
echo "TX_15 = $TX_15"
```

```
Fee: Lovelace 1149177
Size: 1910 / 16384 = 11%
Execution units:
  Memory: 11678826 / 14000000 = 83%
  Steps: 3039771540 / 10000000000 = 30%
TX_15 = e931ff8931b36b2baac930aed5db39d17434cfb22254565572e94b2795e97af7
```

Xem giao dịch qua trình khám phá dữ liệu blockchain.

In \[86]:

```
echo "https://cardanoscan.io/transaction/$TX_15?tab=utxo"
```

```
https://cardanoscan.io/transaction/e931ff8931b36b2baac930aed5db39d17434cfb22254565572e94b2795e97af7?tab=utxo
```

### Giao dịch số 16: Thanh toán lại cho Jane Lumley và John Webster các khoản ada mà họ đã chuyển. <a href="#transaction-16.-pay-jane-lumley-and-john-webster-any-funds-that-are-owed-back-to-them" id="transaction-16.-pay-jane-lumley-and-john-webster-any-funds-that-are-owed-back-to-them"></a>

Không có gì _nợ_ họ, nhưng thông báo này vẫn là cần thiết.

In \[87]:

```
marlowe-cli run prepare \
  --marlowe-file marlowe-15.json \
  --out-file marlowe-16.json \
  --notify \
  --invalid-before "$((1000 * ($(date -u +%s) - 2 * 60)))" \
  --invalid-hereafter "$((1000 * ($(date -u +%s) + 8 * 60)))" \
  --print-stats
```

```
Rounding  `TransactionInput` txInterval boundries to:(POSIXTime {getPOSIXTime = 1678574767000},POSIXTime {getPOSIXTime = 1678575367999})
TransactionInput {txInterval = (POSIXTime {getPOSIXTime = 1678574767000},POSIXTime {getPOSIXTime = 1678575367999}), txInputs = [NormalInput INotify]}

Warnings:
  TransactionNonPositivePay "j.lumley" (Party "j.lumley") (Token "" "") 0
  TransactionNonPositivePay "j.webster" (Party "j.webster") (Token "" "") 0
Datum size: 436
```

Hợp đồng chứa các bản ghi về quỹ còn lại _nợ_ các bên.

In \[88]:

```
jq .tx.state.accounts marlowe-16.json | json2yaml
```

```
- - - address: addr1q8tntkszteptml4e9ce9l3fsmgavwv4ywunvdnhxv6nw5ksvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60stayv90
    - currency_symbol: ''
      token_name: ''
  - 2000000
- - - role_token: m.herbert
    - currency_symbol: ''
      token_name: ''
  - 2000000
- - - role_token: e.cary
    - currency_symbol: ''
      token_name: ''
  - 42000000
- - - role_token: f.beaumont
    - currency_symbol: ''
      token_name: ''
  - 32000000
- - - role_token: c.marlowe
    - currency_symbol: ''
      token_name: ''
  - 50000000
```

Gửi giao dịch.

In \[89]:

```
TX_16=$(
marlowe-cli run auto-execute \
  --mainnet \
  --marlowe-in-file marlowe-15.json \
  --marlowe-out-file marlowe-16.json \
  --tx-in-marlowe "$TX_15#1" \
  --change-address "${ROLE_ADDR[c.marlowe]}" \
  --required-signer "${ROLE_SKEY[c.marlowe]}" \
  --out-file /dev/null \
  --submit 600 \
  --print-stats \
| sed -e 's/^TxId "\(.*\)"$/\1/' \
)
echo "TX_16 = $TX_16"
```

```
Fee: Lovelace 848895
Size: 1610 / 16384 = 9%
Execution units:
  Memory: 7815094 / 14000000 = 55%
  Steps: 2104942293 / 10000000000 = 21%
TX_16 = bd87106f6b042a4ba6f2f80cadc7ace74e5a125dcc93755b0bc2677d98db9c98
```

Xem giao dịch qua trình khám phá dữ liệu blockchain.

In \[90]:

```
echo "https://cardanoscan.io/transaction/$TX_16?tab=utxo"
```

```
https://cardanoscan.io/transaction/bd87106f6b042a4ba6f2f80cadc7ace74e5a125dcc93755b0bc2677d98db9c98?tab=utxo
```

### Giao dịch số 17: Thanh toán cho Elizabeth Cary và Mary Herbert các khoản ada mà họ đã chuyển. <a href="#transaction-17.-pay-elizabeth-cary-and-mary-herbert-any-funds-that-are-owed-back-to-them" id="transaction-17.-pay-elizabeth-cary-and-mary-herbert-any-funds-that-are-owed-back-to-them"></a>

Elizabeth Cary nhận lại khoản tiền đặt cọc của mình và **Mary Herbert nhận lại số ADA tối thiểu**.

In \[91]:

```
marlowe-cli run prepare \
  --marlowe-file marlowe-16.json \
  --out-file marlowe-17.json \
  --notify \
  --invalid-before "$((1000 * ($(date -u +%s) - 2 * 60)))" \
  --invalid-hereafter "$((1000 * ($(date -u +%s) + 8 * 60)))" \
  --print-stats
```

```
Rounding  `TransactionInput` txInterval boundries to:(POSIXTime {getPOSIXTime = 1678574893000},POSIXTime {getPOSIXTime = 1678575493999})
TransactionInput {txInterval = (POSIXTime {getPOSIXTime = 1678574893000},POSIXTime {getPOSIXTime = 1678575493999}), txInputs = [NormalInput INotify]}

Datum size: 381
Payment 1
  Acccount: "e.cary"
  Payee: Party "e.cary"
  Ada: Lovelace {getLovelace = 42000000}
Payment 2
  Acccount: "m.herbert"
  Payee: Party "m.herbert"
  Ada: Lovelace {getLovelace = 2000000}
```

Hợp đồng chứa các bản ghi về quỹ còn lại nợ các bên.

In \[92]:

```
jq .tx.state.accounts marlowe-17.json | json2yaml
```

```
- - - address: addr1q8tntkszteptml4e9ce9l3fsmgavwv4ywunvdnhxv6nw5ksvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60stayv90
    - currency_symbol: ''
      token_name: ''
  - 2000000
- - - role_token: f.beaumont
    - currency_symbol: ''
      token_name: ''
  - 32000000
- - - role_token: c.marlowe
    - currency_symbol: ''
      token_name: ''
  - 50000000
```

Gửi giao dịch.

In \[93]:

```
TX_17=$(
marlowe-cli run auto-execute \
  --mainnet \
  --marlowe-in-file marlowe-16.json \
  --marlowe-out-file marlowe-17.json \
  --tx-in-marlowe "$TX_16#1" \
  --change-address "${ROLE_ADDR[c.marlowe]}" \
  --required-signer "${ROLE_SKEY[c.marlowe]}" \
  --out-file /dev/null \
  --submit 600 \
  --print-stats \
| sed -e 's/^TxId "\(.*\)"$/\1/' \
)
echo "TX_17 = $TX_17"
```

```
Fee: Lovelace 1179368
Size: 1786 / 16384 = 10%
Execution units:
  Memory: 12060344 / 14000000 = 86%
  Steps: 3183703533 / 10000000000 = 31%
TX_17 = be2a009ed29009d814986904290676d8aa453a91bae98c195a6f124e5667bc4e
```

Xem giao dịch qua trình khám phá dữ liệu blockchain.

In \[94]:

```
echo "https://cardanoscan.io/transaction/$TX_17?tab=utxo"
```

```
https://cardanoscan.io/transaction/be2a009ed29009d814986904290676d8aa453a91bae98c195a6f124e5667bc4e?tab=utxo
```

### Giao dịch số 18: Thanh toán cho Christopher Marlowe và Francis Beaumont các khoản ada của họ. <a href="#transaction-18.-pay-christopher-marlowe-and-francis-beaumont-any-funds-that-are-owed-back-to-them" id="transaction-18.-pay-christopher-marlowe-and-francis-beaumont-any-funds-that-are-owed-back-to-them"></a>

Christopher Marlowe nhận khoản thanh toán cho token `BearGarden` và số ADA tối thiểu mà họ đã gửi ban đầu. Francis Beaumont nhận lại khoản ada đấu giá của mình.

In \[95]:

```
marlowe-cli run prepare \
  --marlowe-file marlowe-17.json \
  --out-file marlowe-18.json \
  --notify \
  --invalid-before "$((1000 * ($(date -u +%s) - 2 * 60)))" \
  --invalid-hereafter "$((1000 * ($(date -u +%s) + 8 * 60)))" \
  --print-stats
```

```
Rounding  `TransactionInput` txInterval boundries to:(POSIXTime {getPOSIXTime = 1678574930000},POSIXTime {getPOSIXTime = 1678575530999})
TransactionInput {txInterval = (POSIXTime {getPOSIXTime = 1678574930000},POSIXTime {getPOSIXTime = 1678575530999}), txInputs = [NormalInput INotify]}

Datum size: 159
Payment 1
  Acccount: "c.marlowe"
  Payee: Party "c.marlowe"
  Ada: Lovelace {getLovelace = 50000000}
Payment 2
  Acccount: "f.beaumont"
  Payee: Party "f.beaumont"
  Ada: Lovelace {getLovelace = 32000000}
Payment 3
  Acccount: "\"addr1q8tntkszteptml4e9ce9l3fsmgavwv4ywunvdnhxv6nw5ksvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60stayv90\""
  Payee: Party "\"addr1q8tntkszteptml4e9ce9l3fsmgavwv4ywunvdnhxv6nw5ksvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60stayv90\""
  Ada: Lovelace {getLovelace = 2000000}
```

Hợp đồng đã được đóng.

In \[96]:

```
jq .tx.contract marlowe-18.json | json2yaml
```

```
close
...
```

Gửi giao dịch.

In \[97]:

```
TX_18=$(
marlowe-cli run auto-execute \
  --mainnet \
  --marlowe-in-file marlowe-17.json \
  --marlowe-out-file marlowe-18.json \
  --tx-in-marlowe "$TX_17#1" \
  --change-address "${ROLE_ADDR[c.marlowe]}" \
  --required-signer "${ROLE_SKEY[c.marlowe]}" \
  --out-file /dev/null \
  --submit 600 \
  --print-stats \
| sed -e 's/^TxId "\(.*\)"$/\1/' \
)
echo "TX_18 = $TX_18"
```

```
Fee: Lovelace 875504
Size: 1298 / 16384 = 7%
Execution units:
  Memory: 8391256 / 14000000 = 59%
  Steps: 2203315015 / 10000000000 = 22%
TX_18 = 2ab49e5d1d6afb0f4b66b7854e2602ba041996f1588192a03e95c8d11949072d
```

Xem giao dịch qua trình khám phá dữ liệu blockchain.

In \[98]:

```
echo "https://cardanoscan.io/transaction/$TX_18?tab=utxo"
```

```
https://cardanoscan.io/transaction/2ab49e5d1d6afb0f4b66b7854e2602ba041996f1588192a03e95c8d11949072d?tab=utxo
```

### Giao dịch số 50: Christopher Marlowe rút 50 ada về địa chỉ thanh toán vai trò <a href="#transaction-19.-christopher-marlowe-withdraws-50-ada-for-the-role-payout-address" id="transaction-19.-christopher-marlowe-withdraws-50-ada-for-the-role-payout-address"></a>

Người bán (Christopher) nhận về 50 ada .

In \[99]:

```
TX_19=$(
marlowe-cli run auto-withdraw \
  --mainnet \
  --role-name c.marlowe \
  --marlowe-file marlowe-1.json \
  --change-address "${ROLE_ADDR[c.marlowe]}" \
  --required-signer "${ROLE_SKEY[c.marlowe]}" \
  --out-file /dev/null \
  --submit 600 \
  --print-stats \
| sed -e 's/^TxId "\(.*\)"$/\1/' \
)
echo "TX_19 = $TX_19"
```

```
Fee: Lovelace 354162
Size: 582 / 16384 = 3%
Execution units:
  Memory: 1952154 / 14000000 = 13%
  Steps: 546040855 / 10000000000 = 5%
TX_19 = ed8e4c9d04ab99e9265484a4cd72512709e53293c3db3294361b37e72f378e75
```

Xem giao dịch qua trình khám phá dữ liệu blockchain.

In \[100]:

```
echo "https://cardanoscan.io/transaction/$TX_19?tab=utxo"
```

```
https://cardanoscan.io/transaction/ed8e4c9d04ab99e9265484a4cd72512709e53293c3db3294361b37e72f378e75?tab=utxo
```

### Giao dịch số 20: Francis Beaumont rút 32 ada đã nạp <a href="#transaction-20.-francis-beaumont-withdraws-their-32-ada-deposit" id="transaction-20.-francis-beaumont-withdraws-their-32-ada-deposit"></a>

Francis nhận về khoản nạp của mình.

In \[101]:

```
TX_20=$(
marlowe-cli run auto-withdraw \
  --mainnet \
  --role-name f.beaumont \
  --marlowe-file marlowe-1.json \
  --change-address "${ROLE_ADDR[f.beaumont]}" \
  --required-signer "${ROLE_SKEY[f.beaumont]}" \
  --out-file /dev/null \
  --submit 600 \
  --print-stats \
| sed -e 's/^TxId "\(.*\)"$/\1/' \
)
echo "TX_20 = $TX_20"
```

```
Fee: Lovelace 354250
Size: 584 / 16384 = 3%
Execution units:
  Memory: 1952154 / 14000000 = 13%
  Steps: 546040855 / 10000000000 = 5%
TX_20 = fa1145c20ba038163a9a6cca6deae4b42a87d138e0280d92a275ba0c4dfcb2b7
```

Xem giao dịch qua trình khám phá dữ liệu blockchain.

In \[102]:

```
echo "https://cardanoscan.io/transaction/$TX_20?tab=utxo"
```

```
https://cardanoscan.io/transaction/fa1145c20ba038163a9a6cca6deae4b42a87d138e0280d92a275ba0c4dfcb2b7?tab=utxo
```

### Giao dịch số 21: Elizabeth Cary rút 42 ada đã nạp <a href="#transaction-21.-elizabeth-cary-withdraws-their-42-ada-deposit" id="transaction-21.-elizabeth-cary-withdraws-their-42-ada-deposit"></a>

In \[103]:

```
TX_21=$(
marlowe-cli run auto-withdraw \
  --mainnet \
  --role-name e.cary \
  --marlowe-file marlowe-1.json \
  --change-address "${ROLE_ADDR[e.cary]}" \
  --required-signer "${ROLE_SKEY[e.cary]}" \
  --out-file /dev/null \
  --submit 600 \
  --print-stats \
| sed -e 's/^TxId "\(.*\)"$/\1/' \
)
echo "TX_21 = $TX_21"
```

```
Fee: Lovelace 349836
Size: 576 / 16384 = 3%
Execution units:
  Memory: 1897354 / 14000000 = 13%
  Steps: 533555810 / 10000000000 = 5%
TX_21 = 5ed6799dd29c92bdca1f208d8a68c33f6e0ddbb1c7c989dce4bc1047e0d25292
```

Xem giao dịch qua trình khám phá dữ liệu blockchain.

In \[104]:

```
echo "https://cardanoscan.io/transaction/$TX_21?tab=utxo"
```

```
https://cardanoscan.io/transaction/5ed6799dd29c92bdca1f208d8a68c33f6e0ddbb1c7c989dce4bc1047e0d25292?tab=utxo
```

### Giao dịch số 22: Mary Herbert rút token BearGarden cùng với số ada _tối thiểu_ <a href="#transaction-22.-mary-herbert-withdraws-the-beargarden-token-along-with-the-minimum-ada" id="transaction-22.-mary-herbert-withdraws-the-beargarden-token-along-with-the-minimum-ada"></a>

Mary nhận token và 2 ada.

In \[105]:

```
TX_22=$(
marlowe-cli run auto-withdraw \
  --mainnet \
  --role-name m.herbert \
  --marlowe-file marlowe-1.json \
  --change-address "${ROLE_ADDR[m.herbert]}" \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --out-file /dev/null \
  --submit 600 \
  --print-stats \
| sed -e 's/^TxId "\(.*\)"$/\1/' \
)
echo "TX_22 = $TX_22"
```

```
Fee: Lovelace 609082
Size: 678 / 16384 = 4%
Execution units:
  Memory: 5197472 / 14000000 = 37%
  Steps: 1425947506 / 10000000000 = 14%
TX_22 = b0544c678cf287fd319edf515ebb0b3c2a7e0ae1f6f0f262aaa4b13a6f1600c5
```

Xem giao dịch qua trình khám phá dữ liệu blockchain.

In \[106]:

```
echo "https://cardanoscan.io/transaction/$TX_22?tab=utxo"
```

```
https://cardanoscan.io/transaction/b0544c678cf287fd319edf515ebb0b3c2a7e0ae1f6f0f262aaa4b13a6f1600c5?tab=utxo
```

### Dọn dẹp <a href="#cleanup" id="cleanup"></a>

Việc trả lại token BearGarden cho faucet là rất tiện lợi để _dọn dẹp_ hợp đồng

In \[107]:

```
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "b0544c678cf287fd319edf515ebb0b3c2a7e0ae1f6f0f262aaa4b13a6f1600c5#0" \
  --tx-in "b0544c678cf287fd319edf515ebb0b3c2a7e0ae1f6f0f262aaa4b13a6f1600c5#1" \
  --tx-out "$FAUCET_ADDR+$((2 * ADA))+1 $FUNGIBLES_POLICY.BearGarden" \
  --change-address "${ROLE_ADDR[m.herbert]}" \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "8fc3dd9bc80b04a0ae0e9f3ba36b04b4b7db46414ddf5e988e363f349d3c2b4f"
```
