# 決済系

## 決済系：決済実行

- ヘッダ  
  ```
  Idempotency-Key: {string}
  ```
  |  |  |
  |---|---|
  | Idempotency-Key | 電文重複キー |

- 要求データ
  ```
  {
    "type": "suica",
    "amount": 1,
    "showCancelButton": true
  }
  ```
  |  |  |
  |---|---|
  | type | 金種 |
  | amount | 支払金額 |
  | showCancelButton | キャンセルボタンを表示する |

- 応答データ
  ```
  {
    "id": "20250820110145",
    "amount": 1,
    "status": "processing",
    "transaction_at": "2025-08-20T11:01:45Z"
  }
  ```
  |  |  |
  |---|---|
  | id | 取引ID |
  | amount | 支払金額 |
  | status | 状態 |
  | transaction_at | 取引日時 |


- 定数値
type に使用できるデータ。

||
|---|
| credit |
| suica |
| id |
| waon |
| nanaco |
| edy |
| quicpay |
| okica |
| qr |


#### 説明

- このAPIをコールすると決済を開始し、TerminalAPI上のUIで支払い待ち画面を表示する。
- その際、DB: `transactions` を新規登録する。初期状態は "processing"。

- 支払い待ち画面のタイムアウト値は３０秒。

- 支払操作が完了（成功）すると  `transactions` を "complated" に更新し、金種固有の情報と共に保存を行う。  
  また、DB: `history_uris`, `history_slips` を作成する。
  `history_slips`.id は transactions に関連付けされる。

- 作成された `history_uris` は売上データである。作成後、BackgroundTask で直ちにセンター送信される。

- 通番の最新値は preference に保存されている。  
  設定名は "term_sequence"。


#### 異常ケース

- 支払が失敗すると  `transactions` を "failed" に更新する。  
  失敗時に DB: `history_uris`, `history_slips` が作成されるかは、処理の進捗による。

  1. オーソリ送信前の失敗： 作成されない
  2. オーソリ送信以降の失敗： 作成される

- 支払い待ち画面で何らかの要因でキャンセルすると、 `transactions` を "stopped" に更新する。  
 DB: `history_uris`, `history_slips` は作成されない。

  キャンセル要因は以下。
  1. タイムアウト
  2. キャンセルボタンの押下。
  3. PIN入力画面でキャンセルボタンの押下。

- PIN 入力画面でタイムアウトすると `transactions` を "failed" に更新する。   
  PIN 入力画面のタイムアウト値は６０秒。

#### 利用できない状態

- 業務開始状態でない場合は以下のエラーを返す  
  `status code : 400 , error code : INVALID_OPEN_STATUS`
- 既に別の決済が実行中の場合は以下のエラーを返す  
  `status code : 409 , error code : TRANSACTION_IN_PROGRESS`
- 指定の金種が使えない状態の場合以下のエラーを返す  
  `status code : 400 , error code : PAYMENT_METHOD_UNAVAILABLE`
- 電文重複キーが同一のものが送信された場合、以下のエラーを返す
  `status code : 400 , error code : TRANSACTION_EXECUTED`


#### データサンプル

- 本APIで出力されるデータのサンプルを以下に記載している。

[サンプル（EXCEL）](./files/db_sample1.xlsx)





## 	/v1/terminal/actions/inquireBalance	POST	残高照会

?type=suica

iD は対応していない


##	/v1/payments/{id}/cancel	POST	決済の取消


Idempotency-Key: 

{}


{
"id": "20250820131604",
"amount": 1,
"status": "processing",
"transaction_at": "2025-08-20T13:16:04Z"
}

Edy と nanaco は対応してない


## 端末系：業務終了

- 要求データ
  ```
  {
    "show_confirm": true
  }
  ```
  |  |  |
  |---|---|
  | show_confirm | 確認画面を表示する |

- 応答データ
  ```
  {}
  ```

#### 説明

- 終了時の処理を行い、端末をシャットダウンする。
- この処理は history_aggregates に終了時刻を記録し、ローテーションする。
  