# 取引系

## 取引の取得

- 要求データ（URL）  

```
/vi/transactions/{id}
```

|  |  |
|---|---|
| id | 取引ID |

- 応答データ

```
{
  "id": "20250916180555",
  "exec": "payment",
  "amount": 1,
  "status": "canceled",
  "transactionAt": "2025-09-16T18:05:55Z",
  "method": "credit",
  "term_sequence": 1,
  "idempotency_key": "20250916-06",
  "credit": {
    "brand": "クレジット",
    "card_company": "AMEX CARD",
    "card_no": "377784*****0903",
    "card_exp_date": "XX/XX",
    "credit_type": "IC",
    "approval_no": "37",
    "arc": "00",
    "aid": "A000000025010402",
    "apl": "AMEX"
  }
}
```

|  |  |  |
|---|---|---|
| id | 取引ID | |
| exec | 取引内容 | "payment", "cancel" |
| amount | 取引金額 | |
| status | 取引の状態 | "processing", "completed", "failed", "canceled", "stopped", "unknown" |
| transaction_at | 取引の日時 |  yyyy-MM-ddTHH:mm:ssZ |
| method | 決済手段 | "credit" など |
| term_sequence | 端末通番 | 1 ~ 999 |
| (oneof) | (金種固有データ) | 決済が行われていないとこのエリアのデータは null  |

- 金種固有データ  
このエリアは決済手段により異なる。

  - credit  
    | | |
    |---|---|
    | brand | クレジットカードブランド "VISA" など |
    | card_company | カード発行会社 |
    | (TBD) | |





- 取引金額は 1 ~ 999999.
- 取引の状態の canceld は取消されると元取引がこの状態になる。
- 

#### 説明

- 指定した取引IDの情報を取得する。
- レシート印字に必要な情報が取得できるため、印字をクライアント依存で行う場合はこの情報を参照する。


#### 利用できない状態

- 指定 id に該当するデータがない場合、以下のエラーを返す。  
  `status code : 404 , error code : TRANSACTION_NOT_FOUND`


## レシート印字


- 要求データ (URL)

  ```
  /vi/transactions/{id}/receipt
  ```
  |  |  |
  |---|---|
  | id | 取引ID |

- 応答データ
  ```
  {
  }
  ```

#### 説明

- 指定した取引IDのレシート印刷を行う。
- APIコール後、お客様控えが印刷される。
- UIにダイアログが表示されるのでそれに同意すると、加盟店控えが印刷される。

#### 異常ケース


#### 利用できない状態

- レシート印字中に再度呼び出すと、以下のエラーを返す。  
  `status code : 409 , error code : TRANSACTION_IN_PROGRESS`

- 指定 id に該当するデータがない場合、以下のエラーを返す。  
  `status code : 404 , error code : TRANSACTION_NOT_FOUND`


