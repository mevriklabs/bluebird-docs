# Resource

**File Location:**  
`projectRoot/app/Services/ApiResponse/ApiResources.php`

This file contains all API functions that map to various bot use cases, intents, and corresponding GP APIs.

---

| Sl | Bot Use Case | Intent Name | GP APIs |
|----|---------------|-------------|----------|
| 1 | Recharge | recharge, payment | recharge |
| 2 | Star Status | FIRST ACTIVATION, UPGRADE_STAR_STATUS |  |
| 3 | Billing | billingAddress, FINALIZE_CHANGED_BILL, FINALIZE_ENTRY_BILL, emailAddress, CHANGE_EMAIL_CONFIRMATION, ENTRY_EMAIL_CONFIRMATION, FINALIZE_CHANGED_EMAIL, FINALIZE_ENTRY_EMAIL, eBillStatus, POSTPAID_TOTAL_DUE, TOTAL_DUE_LAST_INVOICE, CURRENT_DUE_AMOUNT, EBILL STATUS, BILLING ADDRESS | balanceQuery, postPaidAccountSummary, productOrdering, bscsAddressGet |
| 4 | Recharge History | POSTPAID_LAST_PAYMENT, POSTPAID_LAST_PAID_AMOUNT_DATE, POSTPAID_LAST_PAYMENT_INVOICE, LAST_RECHARGE_HISTORY | balanceQuery, queryRechargeHistory |
| 5 | Account Switch | MEVRIK_AUTH_OTP, MEVRIK_AUTH_TIMEOUT_SWAP, MEVRIK_AUTH_ATTEMPT, AGENT_SERVICE | skittoStatusCheck, sendSMS |
| 6 | Purchase | purchase, purchaseCmp | balanceQuery, dadetails |
| 7 | FNF Migration | FNF_MIGRATION | changeRatePlan, balanceQuery, queryFnfCbl |
| 8 | Sim Package Status | sim_package_status | changeRatePlan |
| 9 | Greeting | WELCOME_MESSAGE |  |
| 10 | Micro Offer | MICRO DATA OFFER, MICRO VOICE OFFER, MICRO OFFER | offers |
| 11 | FNF SuperFNF Status | CHECK ELIGIBILITY | changeRatePlan, balanceQuery, queryFnfCbl |
| 12 | Balance Check | BALANCE | balanceQuery, dadetails |
| 13 | Talktime | TALKTIME | dadetails, balanceQuery |
| 14 | Offers | EXCLUSIVE_VOICE_PACKS, EXCLUSIVE_DATA_PACKS, EXCLUSIVE_SMS_PACKS, BUNDLE OFFER, SOCIAL PACK | productOrdering |
| 15 | Account Check | POSTPAID_LAST_PAID_AMOUNT_DATE, TOTAL_DUE_LAST_INVOICE, CURRENT_DUE_AMOUNT, POSTPAID_BILL_CYCLE, POSTPAID_LAST_PAYMENT_INVOICE, POSTPAID_TOTAL_DUE, POSTPAID_ACCOUNT_PACKAGE_STATUS, POSTPAID_LOCAL_CREDIT_BALANCE, CHECK SECURITY DEPOSIT | postPaidAccountSummary, balanceQuery |
| 16 | FNF Status Check | PRE_FNF_SUPFNF_STATUS | changeRatePlan |
| 17 | Bioscope | Bioscope Download, Bioscope Series, Bioscope Access |  |
| 18 | Loyalty Point | Loyalty Point |  |
| 19 | Balance Transfer | Balance Transfer Eligibility, Balance Transfer | balanceQuery |
| 20 | Churn Back | New Sim Offer, Close Sim Offer | balanceQuery |
| 21 | GP TouchPoint | GP TouchPoint |  |
| 22 | Flexiplan | Flexiplan Purchase, Flexiplan Gift, Flexiplan Validity |  |
| 23 | GPay | GPay Registration, GPay Pin Reset, Gpay Wallet Recharge, GPay Service Charge, Recharge by GPay, GPay Bill Process |  |
| 24 | GP Info | GP Info |  |
| 25 | GP Star | GP Star, Star Usage, Star Offer | customerStatus |
| 26 | Emergency Balance | Emergency Balance, Emergency Credit Limit, Emergency Balance Usages | balanceQuery |
| 27 | DND | DND Status, DND Activation, DND DeActivation |  |
| 28 | Lost SIM | Unblock Lost SIM, Lost Phone |  |
| 29 | Skitto | Skitto SIM Price, Skitto Call Rate, Skitto2GP, GP2Skitto, Skitto Sim Replace, Skitto Hotline |  |
| 30 | Roaming | Roaming Status, Roaming Activation, Roaming Packages, Roaming SD, Roaming Deactivation, Roaming Offer, Roaming Total Due | internationalRoaming, queryRoamingPack, balanceQuery, postPaidAccountSummary |
| 31 | Manual Network | Manual Network |  |
| 32 | Train Tracking | Train Tracking |  |
| 33 | Voice Chat Service | Voice Chat Service Activation, Voice Chat Service DeActivation |  |
| 34 | Call Block Service | Call Block Service Activation, Call Block Service DeActivation |  |
| 35 | CP | CP Activation, CP DeActivation, CP Overcharging |  |
| 36 | MNP | MNP Port in, MNP Port out, MNP Info, MNP Issue |  |
| 37 | Sim Card Issue | Sim Card Issue, Puk Recover, Foreigner Sim Issue |  |
| 38 | Owner Transfer | Transfer One2One, Transfer C2B, Death Case, Transfer B2C |  |
| 39 | Device Activity | Device Activity, Log In Issue |  |
| 40 | MyGP Uses History | Call Record |  |
| 41 | MyGP | NID Reg Sim, Reward Point, Data Mismatch, Package Migration |  |
| 42 | Biometric | NID Remove |  |
| 43 | GP Alo | gp_alo, gpalo_products, alo_app, alo_where_to_get |  |
| 44 | GPFI | gpfi |  |
| 45 | Welcome Tune | WT Song Add, WT Song Delete, WT Song Copy Process, WT Random, WT Activation & Deactivation Status | wtStatusCheck, balanceQuery, productOrdering |

---
