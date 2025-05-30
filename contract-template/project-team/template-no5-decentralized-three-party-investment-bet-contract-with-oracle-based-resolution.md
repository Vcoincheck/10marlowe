# Template No5: Decentralized Three-Party Investment Bet Contract with Oracle-Based Resolution

### **Title:** Decentralized Three-Party Investment Bet Contract with Oracle-Based Resolution

***

### **Overview:**

This contract models a decentralized conditional investment (or bet) between three parties — `B`, `C`, and `D` — who each deposit an equal amount of ADA into the contract. After deposit, each party makes a private price prediction for a specific asset (e.g., token, stock, crypto). At a designated time, a trusted oracle (`Pricemarket`) submits the actual market price. The contract then evaluates which party's prediction was closest and automatically distributes the combined pool accordingly.

### **Contract Steps (Process Summary):**

1. **Initial Deposits:**
   * Party `B` deposits `SoluongADA` ADA × 1\_000\_000 lovelace.
   * Party `C` does the same after `B`.
   * Party `D` does the same after `C`.
2. **Prediction Phase:**
   * Each party (`B`, `C`, `D`) makes a price prediction by selecting a value via a `Choice`.
3. **Oracle Price Submission:**
   * At time `ThoiGianLayGiaMarket`, the oracle (`Pricemarket`) submits the actual market price.
4. **Evaluation Phase:**
   * The contract calculates the difference between each party's prediction and the oracle price.
   * The winner is the party whose prediction is closest (evaluated using `SubValue` and conditional comparisons).
5. **Payout Phase:**
   * If Party `C` is the winner:
     * `C` receives funds from both `B` and `D`.
   * If Party `B` is the winner:
     * `B` receives funds from both `C` and `D`.
   * If Party `D` is the winner:
     * `D` receives funds from both `B` and `C`.
6. **Timeout Handling:**
   * If any deposit or choice is not made before respective deadlines (`ThoiHanBnoptien`, `ThoiHanCnoptien`, `ThoiHanDnoptien`, etc.), the contract terminates via `Close`.

#### **Roles Involved:**

* **`B`, `C`, `D`**\
  Each acts as both an investor and predictor. All three must deposit funds and submit a price prediction.
* **`Pricemarket` (Oracle)**\
  A trusted role responsible for submitting the actual market price used for evaluating winner(s).

### Contract flowchart

<figure><img src="../../.gitbook/assets/deepseek_mermaid_20250530_ba72aa.png" alt=""><figcaption></figcaption></figure>

#### Key Components Explained:

1. **Deposit Phase (Yellow)**:
   * Sequential deposits from B, C, D
   * Each deposits `SoluongADA × 1,000,000` lovelace
   * Strict time limits at each step (`ThoiHanXnoptien`)
2. **Prediction Phase (Blue)**:
   * Participants submit price predictions in sequence
   * B → C → D submission order
   * Time-bound submissions (`ThoiGianXChon`)
3. **Oracle Phase (Teal)**:
   * Market price oracle submits value
   * Must occur before `ThoiGianLayGiaMarket`
4. **Winner Determination (Red)**:
   *   Compares absolute differences:

       CopyDownload

       ```
       |Participant Prediction - Oracle Value|
       ```
   * Complex tie-breaking logic:
     * Compares secondary differences
     * Uses nested conditionals
5. **Payout Phase (Green)**:
   * Winner receives combined deposits from other two
   * Three possible outcomes:
     * B wins: Receives C+D deposits
     * C wins: Receives B+D deposits
     * D wins: Receives B+C deposits
6. **Timeout Handling (Red)**:
   * Contract closes if any deadline is missed
   * No partial payments or refunds
   * Strict enforcement at every phase

### Contract in blockly and Marlowe code

#### Contract in blocky

Fully marlowe code please visit here [Marlowe playground](https://tinyurl.com/p8984sks) (Note: Since the original URL of the Marlowe Playground is very long, I have shortened it.)

<figure><img src="../../.gitbook/assets/Screenshot 2025-05-30 180721.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/Screenshot 2025-05-30 180741.jpg" alt=""><figcaption></figcaption></figure>

#### Contract in Marlowe code

```
When
    [Case
        (Deposit
            (Role "B")
            (Role "B")
            (Token "" "")
            (MulValue
                (ConstantParam "SoluongADA")
                (Constant 1000000)
            )
        )
        (When
            [Case
                (Deposit
                    (Role "C")
                    (Role "C")
                    (Token "" "")
                    (MulValue
                        (ConstantParam "SoluongADA")
                        (Constant 1000000)
                    )
                )
                (When
                    [Case
                        (Deposit
                            (Role "D")
                            (Role "D")
                            (Token "" "")
                            (MulValue
                                (ConstantParam "SoluongADA")
                                (Constant 1000000)
                            )
                        )
                        (When
                            [Case
                                (Choice
                                    (ChoiceId
                                        "B Chon"
                                        (Role "B")
                                    )
                                    [Bound 1 1000000000000]
                                )
                                (When
                                    [Case
                                        (Choice
                                            (ChoiceId
                                                "C Chon"
                                                (Role "C")
                                            )
                                            [Bound 1 1000000000000]
                                        )
                                        (When
                                            [Case
                                                (Choice
                                                    (ChoiceId
                                                        "D Chon"
                                                        (Role "D")
                                                    )
                                                    [Bound 1 1000000000000]
                                                )
                                                (When
                                                    []
                                                    (TimeParam "Thoigiandoiketqua")
                                                    (When
                                                        [Case
                                                            (Choice
                                                                (ChoiceId
                                                                    "Oracle"
                                                                    (Role "Pricemarket")
                                                                )
                                                                [Bound 1 1000000000000]
                                                            )
                                                            (If
                                                                (ValueGE
                                                                    (Cond
                                                                        (ValueGE
                                                                            (ChoiceValue
                                                                                (ChoiceId
                                                                                    "C Chon"
                                                                                    (Role "C")
                                                                                ))
                                                                            (ChoiceValue
                                                                                (ChoiceId
                                                                                    "Oracle"
                                                                                    (Role "Pricemarket")
                                                                                ))
                                                                        )
                                                                        (SubValue
                                                                            (ChoiceValue
                                                                                (ChoiceId
                                                                                    "C Chon"
                                                                                    (Role "C")
                                                                                ))
                                                                            (ChoiceValue
                                                                                (ChoiceId
                                                                                    "Oracle"
                                                                                    (Role "Pricemarket")
                                                                                ))
                                                                        )
                                                                        (SubValue
                                                                            (ChoiceValue
                                                                                (ChoiceId
                                                                                    "Oracle"
                                                                                    (Role "Pricemarket")
                                                                                ))
                                                                            (ChoiceValue
                                                                                (ChoiceId
                                                                                    "C Chon"
                                                                                    (Role "C")
                                                                                ))
                                                                        )
                                                                    )
                                                                    (Cond
                                                                        (ValueGE
                                                                            (ChoiceValue
                                                                                (ChoiceId
                                                                                    "B Chon"
                                                                                    (Role "B")
                                                                                ))
                                                                            (ChoiceValue
                                                                                (ChoiceId
                                                                                    "Oracle"
                                                                                    (Role "Pricemarket")
                                                                                ))
                                                                        )
                                                                        (SubValue
                                                                            (ChoiceValue
                                                                                (ChoiceId
                                                                                    "B Chon"
                                                                                    (Role "B")
                                                                                ))
                                                                            (ChoiceValue
                                                                                (ChoiceId
                                                                                    "Oracle"
                                                                                    (Role "Pricemarket")
                                                                                ))
                                                                        )
                                                                        (SubValue
                                                                            (ChoiceValue
                                                                                (ChoiceId
                                                                                    "Oracle"
                                                                                    (Role "Pricemarket")
                                                                                ))
                                                                            (ChoiceValue
                                                                                (ChoiceId
                                                                                    "B Chon"
                                                                                    (Role "B")
                                                                                ))
                                                                        )
                                                                    )
                                                                )
                                                                (If
                                                                    (ValueGE
                                                                        (Cond
                                                                            (ValueGE
                                                                                (ChoiceValue
                                                                                    (ChoiceId
                                                                                        "D Chon"
                                                                                        (Role "D")
                                                                                    ))
                                                                                (ChoiceValue
                                                                                    (ChoiceId
                                                                                        "Oracle"
                                                                                        (Role "Pricemarket")
                                                                                    ))
                                                                            )
                                                                            (SubValue
                                                                                (ChoiceValue
                                                                                    (ChoiceId
                                                                                        "D Chon"
                                                                                        (Role "D")
                                                                                    ))
                                                                                (ChoiceValue
                                                                                    (ChoiceId
                                                                                        "Oracle"
                                                                                        (Role "Pricemarket")
                                                                                    ))
                                                                            )
                                                                            (SubValue
                                                                                (ChoiceValue
                                                                                    (ChoiceId
                                                                                        "Oracle"
                                                                                        (Role "Pricemarket")
                                                                                    ))
                                                                                (ChoiceValue
                                                                                    (ChoiceId
                                                                                        "D Chon"
                                                                                        (Role "D")
                                                                                    ))
                                                                            )
                                                                        )
                                                                        (Cond
                                                                            (ValueGE
                                                                                (ChoiceValue
                                                                                    (ChoiceId
                                                                                        "B Chon"
                                                                                        (Role "B")
                                                                                    ))
                                                                                (ChoiceValue
                                                                                    (ChoiceId
                                                                                        "Oracle"
                                                                                        (Role "Pricemarket")
                                                                                    ))
                                                                            )
                                                                            (SubValue
                                                                                (ChoiceValue
                                                                                    (ChoiceId
                                                                                        "B Chon"
                                                                                        (Role "B")
                                                                                    ))
                                                                                (ChoiceValue
                                                                                    (ChoiceId
                                                                                        "Oracle"
                                                                                        (Role "Pricemarket")
                                                                                    ))
                                                                            )
                                                                            (SubValue
                                                                                (ChoiceValue
                                                                                    (ChoiceId
                                                                                        "Oracle"
                                                                                        (Role "Pricemarket")
                                                                                    ))
                                                                                (ChoiceValue
                                                                                    (ChoiceId
                                                                                        "B Chon"
                                                                                        (Role "B")
                                                                                    ))
                                                                            )
                                                                        )
                                                                    )
                                                                    (Pay
                                                                        (Role "C")
                                                                        (Party (Role "B"))
                                                                        (Token "" "")
                                                                        (MulValue
                                                                            (ConstantParam "SoluongADA")
                                                                            (Constant 1000000)
                                                                        )
                                                                        (Pay
                                                                            (Role "D")
                                                                            (Party (Role "B"))
                                                                            (Token "" "")
                                                                            (MulValue
                                                                                (ConstantParam "SoluongADA")
                                                                                (Constant 1000000)
                                                                            )
                                                                            Close 
                                                                        )
                                                                    )
                                                                    (Pay
                                                                        (Role "B")
                                                                        (Party (Role "D"))
                                                                        (Token "" "")
                                                                        (MulValue
                                                                            (ConstantParam "SoluongADA")
                                                                            (Constant 1000000)
                                                                        )
                                                                        (Pay
                                                                            (Role "C")
                                                                            (Party (Role "D"))
                                                                            (Token "" "")
                                                                            (MulValue
                                                                                (ConstantParam "SoluongADA")
                                                                                (Constant 1000000)
                                                                            )
                                                                            Close 
                                                                        )
                                                                    )
                                                                )
                                                                (If
                                                                    (ValueGE
                                                                        (Cond
                                                                            (ValueGE
                                                                                (ChoiceValue
                                                                                    (ChoiceId
                                                                                        "D Chon"
                                                                                        (Role "D")
                                                                                    ))
                                                                                (ChoiceValue
                                                                                    (ChoiceId
                                                                                        "Oracle"
                                                                                        (Role "Pricemarket")
                                                                                    ))
                                                                            )
                                                                            (SubValue
                                                                                (ChoiceValue
                                                                                    (ChoiceId
                                                                                        "D Chon"
                                                                                        (Role "D")
                                                                                    ))
                                                                                (ChoiceValue
                                                                                    (ChoiceId
                                                                                        "Oracle"
                                                                                        (Role "Pricemarket")
                                                                                    ))
                                                                            )
                                                                            (SubValue
                                                                                (ChoiceValue
                                                                                    (ChoiceId
                                                                                        "Oracle"
                                                                                        (Role "Pricemarket")
                                                                                    ))
                                                                                (ChoiceValue
                                                                                    (ChoiceId
                                                                                        "D Chon"
                                                                                        (Role "D")
                                                                                    ))
                                                                            )
                                                                        )
                                                                        (Cond
                                                                            (ValueGE
                                                                                (ChoiceValue
                                                                                    (ChoiceId
                                                                                        "C Chon"
                                                                                        (Role "C")
                                                                                    ))
                                                                                (ChoiceValue
                                                                                    (ChoiceId
                                                                                        "Oracle"
                                                                                        (Role "Pricemarket")
                                                                                    ))
                                                                            )
                                                                            (SubValue
                                                                                (ChoiceValue
                                                                                    (ChoiceId
                                                                                        "C Chon"
                                                                                        (Role "C")
                                                                                    ))
                                                                                (ChoiceValue
                                                                                    (ChoiceId
                                                                                        "Oracle"
                                                                                        (Role "Pricemarket")
                                                                                    ))
                                                                            )
                                                                            (SubValue
                                                                                (ChoiceValue
                                                                                    (ChoiceId
                                                                                        "Oracle"
                                                                                        (Role "Pricemarket")
                                                                                    ))
                                                                                (ChoiceValue
                                                                                    (ChoiceId
                                                                                        "C Chon"
                                                                                        (Role "C")
                                                                                    ))
                                                                            )
                                                                        )
                                                                    )
                                                                    (Pay
                                                                        (Role "D")
                                                                        (Party (Role "C"))
                                                                        (Token "" "")
                                                                        (MulValue
                                                                            (ConstantParam "SoluongADA")
                                                                            (Constant 1000000)
                                                                        )
                                                                        (Pay
                                                                            (Role "B")
                                                                            (Party (Role "C"))
                                                                            (Token "" "")
                                                                            (MulValue
                                                                                (ConstantParam "SoluongADA")
                                                                                (Constant 1000000)
                                                                            )
                                                                            Close 
                                                                        )
                                                                    )
                                                                    (Pay
                                                                        (Role "C")
                                                                        (Party (Role "D"))
                                                                        (Token "" "")
                                                                        (MulValue
                                                                            (ConstantParam "SoluongADA")
                                                                            (Constant 1000000)
                                                                        )
                                                                        (Pay
                                                                            (Role "B")
                                                                            (Party (Role "D"))
                                                                            (Token "" "")
                                                                            (MulValue
                                                                                (ConstantParam "SoluongADA")
                                                                                (Constant 1000000)
                                                                            )
                                                                            Close 
                                                                        )
                                                                    )
                                                                )
                                                            )]
                                                        (TimeParam "ThoiGianLayGiaMarket")
                                                        Close 
                                                    )
                                                )]
                                            (TimeParam "ThoiGianDChon")
                                            Close 
                                        )]
                                    (TimeParam "ThoiGianCChon")
                                    Close 
                                )]
                            (TimeParam "ThoiGianBChon")
                            Close 
                        )]
                    (TimeParam "ThoiHanDnoptien")
                    Close 
                )]
            (TimeParam "ThoiHanCnoptien")
            Close 
        )]
    (TimeParam "ThoiHanBnoptien")
    Close 
```
