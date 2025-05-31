# Template no2: Contract for consulting on the design of a residential house.

#### &#x20;**Contract Title: Residential Building Design Consulting Contract**

### **Overview:** <a href="#overview" id="overview"></a>

CommentShare feedback on the editorThis contract simulates a consulting agreement for residential architectural design services involving three parties: **Investor (ChuDauTu)**, **Consulting Unit (DonViTuVan)**, and **Verification Unit (DonViThamTra)**. The process includes upfront deposits from all parties, phased payments based on design milestones, evaluation and approval of the design dossier (HSTK), and conditional payouts depending on quality and delivery. The contract ensures transparency, milestone-based commitment, and financial safeguards to align responsibilities and mitigate risks among all participants.CommentShare feedback on the editor

### **Purpose:** <a href="#purpose" id="purpose"></a>

CommentShare feedback on the editorEnsure a transparent, milestone-driven cooperation process among the investor, consultant, and verifier, with clear financial commitments to protect all parties' interests.CommentShare feedback on the editor

### **Parties Involved:** <a href="#parties-involved" id="parties-involved"></a>

CommentShare feedback on the editor

* CommentShare feedback on the editor**Investor**: The client hiring the design consultant.
* CommentShare feedback on the editor**Consulting Unit**: Responsible for preparing the design documents (Design Dossier – _Hồ sơ thiết kế_, or HSTK).
* CommentShare feedback on the editor**Verification Unit**: Evaluates and validates the quality of the design dossier.

#### **Contract Structure & Workflow:** <a href="#contract-structure-and-workflow" id="contract-structure-and-workflow"></a>

CommentShare feedback on the editor

1. CommentShare feedback on the editor**Investor** deposits the design fee (`PhiThietKe`) into the contract.
2. CommentShare feedback on the editor**Consulting Unit** deposits their consulting collateral (`CocTuVan`).
3. CommentShare feedback on the editor**Verification Unit** deposits their verification collateral (`CocThamTra`).
4. CommentShare feedback on the editor**Investor** pays **30% of the design fee** to the consulting unit to initiate the work.
5. CommentShare feedback on the editor**Consulting Unit** submits the design dossier (HSTK).
6. CommentShare feedback on the editor**Verification Unit** reviews the HSTK and makes a decision:CommentShare feedback on the editor
   * CommentShare feedback on the editor✅ **If the design is approved**:CommentShare feedback on the editor
     * CommentShare feedback on the editorInvestor pays an additional **50% of the design fee** to the consultant.
     * CommentShare feedback on the editorInvestor pays **10% of the design fee** to the verifier.
     * CommentShare feedback on the editorUpon project completion and approval, the investor pays the remaining **10%** to the consultant.
     * CommentShare feedback on the editorContract is closed.
   * CommentShare feedback on the editor❌ **If the design is not approved**:CommentShare feedback on the editor
     * CommentShare feedback on the editorConsultant receives a **20% refund of their deposit**.
     * CommentShare feedback on the editorThey may resubmit the revised design dossier.
     * CommentShare feedback on the editorIf the revised dossier still fails after a grace period (`GiahanTienDo`), the deposit may be forfeited.

CommentShare feedback on the editor

**Security Mechanisms:**

CommentShare feedback on the editor

* CommentShare feedback on the editor\

* Collaterals (`CocTuVan`, `CocThamTra`) ensure that the consulting and verification parties fulfill their obligations.
* Payments are milestone-based and conditional on deliverables and quality outcomes.

#### &#x20;**Contract Completion Conditions:**

* Full payments made and all parties have fulfilled their duties.
* Or, early termination occurs if the design consistently fails to meet requirements and the consultant has no further chances to amend.

### Contract flowchart

<figure><img src="../../.gitbook/assets/deepseek_mermaid_20250530_b1008d (1).png" alt=""><figcaption><p>Contract flowchart</p></figcaption></figure>





<figure><img src="../../.gitbook/assets/deepseek_mermaid_20250530_e2a6c3.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/deepseek_mermaid_20250530_0edb24.png" alt=""><figcaption></figcaption></figure>

### Contract in blocky and marlowe format

#### Contract in blocky format

Fully marlowe code please visit here [Marlowe playground](https://tinyurl.com/yc599pc2) (Note: Since the original URL of the Marlowe Playground is very long, I have shortened it.)

<figure><img src="../../.gitbook/assets/Screenshot 2025-05-30 145659.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/Screenshot 2025-05-30 145806.jpg" alt=""><figcaption></figcaption></figure>

#### Contract in marlowe code

```haskell
When
    [Case
        (Deposit
            (Role "ChuDauTu")
            (Role "ChuDauTu")
            (Token "" "")
            (ConstantParam "PhiThietKe")
        )
        (When
            [Case
                (Deposit
                    (Role "DonViTuVan")
                    (Role "DonViTuVan")
                    (Token "" "")
                    (ConstantParam "CocTuVan")
                )
                (When
                    [Case
                        (Deposit
                            (Role "DonViThamTra")
                            (Role "DonViThamTra")
                            (Token "" "")
                            (ConstantParam "CocThamTra")
                        )
                        (Pay
                            (Role "ChuDauTu")
                            (Party (Role "DonViTuVan"))
                            (Token "" "")
                            (DivValue
                                (MulValue
                                    (ConstantParam "PhiThietKe")
                                    (Constant 30)
                                )
                                (Constant 100)
                            )
                            (When
                                [Case
                                    (Choice
                                        (ChoiceId
                                            "HSTKDat"
                                            (Role "DonViThamTra")
                                        )
                                        [Bound 1 1]
                                    )
                                    (Pay
                                        (Role "ChuDauTu")
                                        (Party (Role "DonViTuVan"))
                                        (Token "" "")
                                        (DivValue
                                            (MulValue
                                                (ConstantParam "PhiThietKe")
                                                (Constant 50)
                                            )
                                            (Constant 100)
                                        )
                                        (Pay
                                            (Role "ChuDauTu")
                                            (Party (Role "DonViThamTra"))
                                            (Token "" "")
                                            (DivValue
                                                (MulValue
                                                    (ConstantParam "PhiThietKe")
                                                    (Constant 10)
                                                )
                                                (Constant 100)
                                            )
                                            (When
                                                [Case
                                                    (Choice
                                                        (ChoiceId
                                                            "NghiemThuCongTrinh"
                                                            (Role "ChuDauTu")
                                                        )
                                                        [Bound 1 1]
                                                    )
                                                    (Pay
                                                        (Role "ChuDauTu")
                                                        (Party (Role "DonViTuVan"))
                                                        (Token "" "")
                                                        (DivValue
                                                            (MulValue
                                                                (ConstantParam "PhiThietKe")
                                                                (Constant 10)
                                                            )
                                                            (Constant 100)
                                                        )
                                                        Close 
                                                    )]
                                                (TimeParam "GiamSatTacGia")
                                                Close 
                                            )
                                        )
                                    ), Case
                                    (Choice
                                        (ChoiceId
                                            "HSTKChuaDat"
                                            (Role "DonViThamTra")
                                        )
                                        [Bound 0 0]
                                    )
                                    (When
                                        [Case
                                            (Choice
                                                (ChoiceId
                                                    "HSTKDat"
                                                    (Role "DonViThamTra")
                                                )
                                                [Bound 1 1]
                                            )
                                            (Pay
                                                (Role "ChuDauTu")
                                                (Party (Role "DonViTuVan"))
                                                (Token "" "")
                                                (DivValue
                                                    (MulValue
                                                        (ConstantParam "PhiThietKe")
                                                        (Constant 50)
                                                    )
                                                    (Constant 100)
                                                )
                                                (Pay
                                                    (Role "ChuDauTu")
                                                    (Party (Role "DonViThamTra"))
                                                    (Token "" "")
                                                    (DivValue
                                                        (MulValue
                                                            (ConstantParam "PhiThietKe")
                                                            (Constant 10)
                                                        )
                                                        (Constant 100)
                                                    )
                                                    (When
                                                        [Case
                                                            (Choice
                                                                (ChoiceId
                                                                    "NghiemThuCongTrinh"
                                                                    (Role "ChuDauTu")
                                                                )
                                                                [Bound 1 1]
                                                            )
                                                            (Pay
                                                                (Role "ChuDauTu")
                                                                (Party (Role "DonViTuVan"))
                                                                (Token "" "")
                                                                (DivValue
                                                                    (MulValue
                                                                        (ConstantParam "PhiThietKe")
                                                                        (Constant 10)
                                                                    )
                                                                    (Constant 100)
                                                                )
                                                                Close 
                                                            )]
                                                        (TimeParam "GiamSatTacGia")
                                                        Close 
                                                    )
                                                )
                                            )]
                                        (TimeParam "TVTKNopHSTKChinhsua")
                                        (Pay
                                            (Role "DonViTuVan")
                                            (Party (Role "ChuDauTu"))
                                            (Token "" "")
                                            (ConstantParam "CocTuVan")
                                            Close 
                                        )
                                    )]
                                (TimeParam "TVNopHSTK")
                                (Pay
                                    (Role "DonViTuVan")
                                    (Party (Role "ChuDauTu"))
                                    (Token "" "")
                                    (DivValue
                                        (MulValue
                                            (ConstantParam "CocTuVan")
                                            (Constant 20)
                                        )
                                        (Constant 100)
                                    )
                                    (When
                                        [Case
                                            (Choice
                                                (ChoiceId
                                                    "HSTKDat"
                                                    (Role "DonViThamTra")
                                                )
                                                [Bound 1 1]
                                            )
                                            (Pay
                                                (Role "ChuDauTu")
                                                (Party (Role "DonViTuVan"))
                                                (Token "" "")
                                                (DivValue
                                                    (MulValue
                                                        (ConstantParam "PhiThietKe")
                                                        (Constant 50)
                                                    )
                                                    (Constant 100)
                                                )
                                                (Pay
                                                    (Role "ChuDauTu")
                                                    (Party (Role "DonViThamTra"))
                                                    (Token "" "")
                                                    (DivValue
                                                        (MulValue
                                                            (ConstantParam "PhiThietKe")
                                                            (Constant 10)
                                                        )
                                                        (Constant 100)
                                                    )
                                                    (When
                                                        [Case
                                                            (Choice
                                                                (ChoiceId
                                                                    "NghiemThuCongTrinh"
                                                                    (Role "ChuDauTu")
                                                                )
                                                                [Bound 1 1]
                                                            )
                                                            (Pay
                                                                (Role "ChuDauTu")
                                                                (Party (Role "DonViTuVan"))
                                                                (Token "" "")
                                                                (DivValue
                                                                    (MulValue
                                                                        (ConstantParam "PhiThietKe")
                                                                        (Constant 10)
                                                                    )
                                                                    (Constant 100)
                                                                )
                                                                Close 
                                                            )]
                                                        (TimeParam "GiamSatTacGia")
                                                        Close 
                                                    )
                                                )
                                            )]
                                        (TimeParam "GiahanTienDo")
                                        (Pay
                                            (Role "DonViTuVan")
                                            (Party (Role "ChuDauTu"))
                                            (Token "" "")
                                            (ConstantParam "CocTuVan")
                                            Close 
                                        )
                                    )
                                )
                            )
                        )]
                    (TimeParam "TTNopCoc")
                    Close 
                )]
            (TimeParam "TVNopCoc")
            Close 
        )]
    (TimeParam "CĐTNopPhiThietKe")
    Close 
```
