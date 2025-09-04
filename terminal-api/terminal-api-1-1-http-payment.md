# 決済系

## 決済系独自仕様

### 操作可否

決済APIは取り扱う金種ごとに可能な操作が定義されている。  
以下は、取り扱える金種と利用可否をまとめたもの。  

| 金種 | 支払 | 取消 | 残額照会 | チャージ |
|---|---|---|---|---|
| クレジット | 〇 | 〇 | 　 | 　 |
| Edy | 〇 | 　 | 〇 | 　 |
| iD | 〇 | 〇 | 　 | 　 |
| nanaco | 〇 | 　 | 〇 | 　 |
| QuicPay | 〇 | 〇 | 　 | 　 |
| 交通系 | 〇 | 〇 | 〇 | 　 |
| WAON | 〇 | 〇 | 〇 | 　 |
| QR | 〇 | 〇 | 　 | 　 |

- OKICA は現状 TerminalAPI 単独の構成では使用できない。  
※ 開局処理の関係  
- チャージは OKICA のみ対応している。

### 端末通番
アプリケーション上で採番される。  
範囲は 1 ~ 999 でローテーションする。
- 通番の最新値は preference に保存されている。  
  設定名は "term_sequence"。

### 売上送信
売上送信は　Background Task で行われている。

`history_uris` に登録されている全件を対象におおむね 1分に一度の間隔で動作する。  
※ looper を使っているので厳密な時間ではない。

- 電文は１０件ずつ送信する
- 失敗したデータに関しては、１件ずつ再送する
- 再送も失敗した場合は送信不可電文として、専用のインターフェースに送信する
- 送信が完了した `history_uris` は物理削除する。


## 決済系：決済実行

- ヘッダ  
  ```
  Idempotency-Key: {string}
  ```
  |  |  |
  |---|---|
  | Idempotency-Key | 電文重複キー |

- 要求データ (body)
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




## 決済系：決済取得


- 要求データ (URL)
  ```
  /vi/payments/{id}
  ```
  |  |  |
  |---|---|
  | id | 取引ID |

- 応答データ
  ```
  {
    "id": "20250820110145",
    "amount": 1,
    "status": "processing",
    "transaction_at": "2025-08-20T11:01:45Z",
    "term_sequence": "1"
  }
  ```
  |  |  |
  |---|---|
  | id | 取引ID |
  | amount | 支払金額 |
  | status | 状態 |
  | transaction_at | 取引日時 |
  | term_sequence | 端末通番 |


#### 説明

- 指定した取引ID の transactions を検索し、返却する。
- 取引の状態を確認するのが主な使い方。  
  この API でポーリングし、"completed" になったら、クライアントは次の処理に進める。
  など

#### 利用できない状態

- 指定 id に該当するデータがない場合、以下のエラーを返す。  
  `status code : 404 , error code : TRANSACTION_NOT_FOUND`



## 決済系：決済取消

- ヘッダ  
  ```
  Idempotency-Key: {string}
  ```
  |  |  |
  |---|---|
  | Idempotency-Key | 電文重複キー |


- 要求データ (URL)
  ```
  /vi/payments/{id}/cancel
  ```
  |  |  |
  |---|---|
  | id | 取引ID |


- 要求データ (body)  

  ```
  { }
  ```
  なし

- 応答データ

  ```
  {
    "id": "1",
    "amount": 10000,
    "status": "processing",
    "transaction_at": "2024-02-20T12:00:00Z",
    "term_sequence": 123
  }
  ```



#### 説明

- 指定した取引IDの取消を行うために、TerminalAPI上のUIで取消待ち画面を表示する。  
  画面には 種別： 取消 の文字が出る点が決済と違う。

- 取消できるデータには条件がある。  
  以下の "利用できない状態" の項を参考。

- 基本的は決済と同様の操作の流れとなる。  
  画面表示寺院 DB: transactions を新規登録する。初期状態は "processing"。ただし種別は取り消しのものとなる。("cancel")

- 金額や金種は指定した取引のものが採用される。
- 

#### 異常ケース

- 当然だが、媒体は支払い時と同様のものを使う必要がある。
  別媒体で取り消そうとするとエラーとなる。

- 支払と同様に、失敗すると  `transactions` を "failed" に更新する。  
  失敗時に DB: `history_uris`, `history_slips` が作成されるかは、処理の進捗による。

  1. オーソリ送信前の失敗： 作成されない
  2. オーソリ送信以降の失敗： 作成される

- 支払と同様に、何らかの要因でキャンセルすると、 `transactions` を "stopped" に更新する。  
 DB: `history_uris`, `history_slips` は作成されない。

  キャンセル要因は以下。
  1. タイムアウト
  2. キャンセルボタンの押下。
  3. PIN入力画面でキャンセルボタンの押下。

#### 利用できない状態

- 業務開始状態でない場合は以下のエラーを返す  
  `status code : 400 , error code : INVALID_OPEN_STATUS`
- 既に別の決済が実行中の場合は以下のエラーを返す  
  `status code : 409 , error code : TRANSACTION_IN_PROGRESS`
- 電文重複キーが同一のものが送信された場合、以下のエラーを返す  
  `status code : 400 , error code : TRANSACTION_EXECUTED`
- 指定 id に該当するデータがない場合、以下のエラーを返す。  
  `status code : 400 , error code : TRANSACTION_NOT_FOUND`
- 取消できないブランドの場合以下のエラーを返す  
  `status code : 400 , error code : REFUND_NOT_SUPPORTED`
- 支払でない取引が指定された場合以下のエラーを返す
  `status code : 400 , error code : INVALID_TRANSACTION_TYPE`
- 完了でない取引が指定された場合以下のエラーを返す  
  `status code : 400 , error code : PAYMENT_NOT_COMPLETED`


## 	/v1/terminal/actions/inquireBalance	POST	残高照会

?type=suica

iD は対応していない


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
  