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
| idempotency_key | 重複キー | |
| (oneof) | (金種固有データ) | 決済が行われていないとこのエリアのデータは null  |

- 金種固有データ  
このエリアは決済手段により異なる。  

  - credit  
    | | | |
    |---|---|---|
    | brand | クレジットカードブランド "VISA" など | BrandSign の変換値 | 
    | card_company | カード会社 | Mst_CreditBinRange.PrintText |
    | card_no | カード番号 | マスク済 |
    | card_exp_date | カード有効期限 | マスク済 |
    | credit_type | カード種別 | "IC" など |
    | approval_no | 承認番号 | |
    | arc | オーソリ結果コード | "00" など|
    | aid | カード定義アプリケーションID | "A000000025010402"など カードから取得 |
    | apl | カード定義アプリケーションラベル | "AMEX" など |  

  - edy  
    | | | |
    |---|---|---|
    | card_no | カード番号 | マスク済 | 
    | before_balance | 取引前残高 |  |
    | after_balance | 取引後残高 |  |
    | edy_transaction_no | Edy専用取引番号 | "917140501" など シンクラ応答値 |

  - id_card 
    | | | |
    |---|---|---|
    | card_no | カード番号 | マスク済 | 
    | card_exp_date | カード有効期限 | マスク済 |

  - nanaco
    | | | |
    |---|---|---|
    | card_no | カード番号 | マスク済 | 
    | before_balance | 取引前残高 |  |
    | after_balance | 取引後残高 |  |

  - quicpay
    | | | |
    |---|---|---|
    | card_no | カード番号 | マスク済 | 

  - suica
    | | | |
    |---|---|---|
    | card_no | カード番号 | マスク済 | 
    | before_balance | 取引前残高 |  |
    | after_balance | 取引後残高 |  |

  - WAON
    | | | |
    |---|---|---|
    | card_no | カード番号 | マスク済 | 
    | before_balance | 取引前残高 |  |
    | after_balance | 取引後残高 |  |

- 取引金額は 1 ~ 999999.
- 取引の状態の canceld は取消されると元取引がこの状態になる。
- iD だけ プロパティ名が "id_card"。

#### 説明

- 指定した取引IDの情報を取得する。
- レシート印字に必要な情報が取得できるため、印字をクライアント依存で行う場合はこの情報を参照する。


#### 利用できない状態

- 指定 id に該当するデータがない場合、以下のエラーを返す。  
  `status code : 404 , error code : TRANSACTION_NOT_FOUND`


## 取引一覧取得

- 要求データ（URL）  

```
/v1/transactions?offset=0&limit=10
```

| | | |
|---|---|---|
| offset | 取得開始位置 | 任意：デフォルト 0 範囲 0 ~ |
| limit | 取得件数 | 任意：デフォルト 10 範囲 1 ~ 1000 |


- 応答データ

```
{
  "transactions": [
    {
      "amount": 1,
      "exec": "cancel",
      "id": "20250917163403",
      "idempotency_key": "20250917-17",
      "method": "credit",
      "status": "failed",
      "transactionAt": "2025-09-17T16:34:03Z"
    }
  ],
  "offset": 0,
  "limit": 1,
  "total": 18,
  "has_next": true
}
```

|  |  |  |
|---|---|---|
| offset | 取得開始位置 | |
| limit | 取得件数 | |
| count | 総件数 | |
| has_next | 続きがあるか | offset + limit < count |
| transactions | 取得したデータ | 取引取得と同様のスキーマ  |

#### 説明

- 取引情報を一覧で取得する。
- 取得順は ID の降順となる。

#### 利用できない状態

- 不正なパラメータ指定の場合、以下のエラーを返す。  
  `status code : 400 , error code : INVALID_PARAMS`

  不正なパラメータとは以下を指す
  - offset or limit が数値でない
  - offset or limit が範囲外


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
  なし

#### 説明

- 指定した取引IDのレシート印刷を行う。レシート出力できる状態になっていない (= 正常・処理未了以外の) 取引が指定されるとエラーを返却する。
 
- APIコール後、まず加盟店控えが印刷される。
- UI上にダイアログが表示されるのでそれに同意すると、続けてお客様控えが印刷される。
- 上記は正常取引の場合。処理未了の場合は専用のレシートが一度に出力され、同意ダイアログは出力されない。


- 印刷されるレイアウトは指定されたＩＤの取引の金種・取引内容（支払 / 取消）により変化する。

- 処理未了の取引を指定した場合、処理未了レシートが出力される。

処理未了レシートが出力される可能性のある金種は以下。

| 金種 |
|---|
| edy （未確認） |
| id （未確認）|
| nanaco |
| quicpay （未確認）|
| suica |
| WAON |
| qr |

~~iD は処理未了がない。~~ iD , Edy, QuicPay は発生させることができなかった。  


#### 異常ケース


#### 利用できない状態

- レシート印字中に再度呼び出すと、以下のエラーを返す。  
  `status code : 409 , error code : TRANSACTION_IN_PROGRESS`

- 指定 id に該当するデータがない場合、以下のエラーを返す。  
  `status code : 404 , error code : TRANSACTION_NOT_FOUND`

- 正常終了または処理未了以外のデータを出力しようとした場合、以下のエラーを返す。  
  `status code : 400 , error code : RECEIPT_UNAVAILABLE`
