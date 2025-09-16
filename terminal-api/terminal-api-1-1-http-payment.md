# 決済系

## 決済系留意事項

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
全ての金種で同様の動作。

`history_uris` に登録されている全件を対象におおむね 1分に一度の間隔で動作する。  
※ looper を使っているので厳密な時間ではない。

- 電文は１０件ずつ送信する
- 失敗したデータに関しては、１件ずつ再送する
- 再送も失敗した場合は送信不可電文として、専用のインターフェースに送信する
- 送信が完了した `history_uris` は物理削除する。

### データサンプル

- 本APIで出力されるデータのサンプルを以下に記載している。

[サンプル（EXCEL）](./files/db_sample1.xlsx)



## 決済系：決済実行

- HTTPメソッド  
  POST

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
  UIは指定された金種により異なる。

- 画面表示時に、DB: `transactions` を新規登録する。初期状態は "processing"。

- 支払い待ち画面のタイムアウト値は３０秒。

- 支払操作が完了（成功）すると  `transactions` を "complated" に更新し、金種固有の情報と共に保存を行う。  
  また、DB: `history_uris`, `history_slips` を作成する。
  `history_slips`.id は transactions に関連付けされる。

- 作成された `history_uris` は売上データである。作成後、BackgroundTask で直ちにセンター送信される。



#### 異常ケース（クレジット）

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

#### 異常ケース（電子マネー）

- 支払が失敗すると  `transactions` を "failed" に更新する。   
  電子マネーはタイムアウトでも "failed" になる。

  DB: `history_uris`, `history_slips` が作成されるのは以下の場合。

  1. 成功
  2. 処理未了

- 支払い待ち画面でキャンセルボタンの押下をすると、 `transactions` を "stopped" に更新する。  
 DB: `history_uris`, `history_slips` は作成されない。


#### 異常ケース（QR）

- 支払が失敗すると  `transactions` を "failed" に更新する。 
- 通信不良などの場合は、結果不明とし、結果を再度問い合わせる画面となる。  
  この照会の結果により `transactions` が "completed" | "failed" に更新される。　　
- 再照会も結果不明の場合、再度問い合わせ画面を表示。以降ループ。  
- 再度問い合わせる画面を中断すると  `transactions` を "completed" に更新する。  
  この場合も `history_uris`、`history_slips` は作成されるが処理未了フラグが立つ。

- カメラ読み取りのタイムアウトはない。

  DB: `history_uris`, `history_slips` が作成されるのは以下の場合。

  1. 成功
  2. 処理未了

- カメラ待ち画面でキャンセルボタンの押下をすると、 `transactions` を "stopped" に更新する。  
 DB: `history_uris`, `history_slips` は作成されない。


#### 利用できない状態

- 業務開始状態でない場合は以下のエラーを返す  
  `status code : 400 , error code : INVALID_OPEN_STATUS`
- 既に別の決済が実行中の場合は以下のエラーを返す  
  `status code : 409 , error code : TRANSACTION_IN_PROGRESS`
- 指定の金種が使えない状態の場合以下のエラーを返す  
  `status code : 400 , error code : PAYMENT_METHOD_UNAVAILABLE`
- 電文重複キーが同一のものが送信された場合、以下のエラーを返す
  `status code : 400 , error code : TRANSACTION_EXECUTED`


## 決済系：決済取得

- HTTPメソッド  
  GET

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



## 決済系：決済の取消

- HTTPメソッド  
  POST

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
    "id": "20250911115240",
    "amount": 1,
    "status": "processing",
    "transaction_at": "2025-09-11T11:52:40Z"
  }
  ```



#### 説明

- 指定した取引IDの取消を行うために、TerminalAPI上のUIで取消待ち画面を表示する。  
  画面には 種別： 取消 の文字が出る点が決済と違う。

- 取消できるデータには条件がある。  
  以下の "利用できない状態" の項を参考。

- 取消に失敗しても操作が完了していない場合、再度実行が可能。

- 基本的は決済と同様の操作の流れとなる。  
  画面表示時に DB: transactions を新規登録する。初期状態は "processing"。  
  ただし種別は取消のものとなる。("cancel")

- 取消待ち画面のタイムアウト値は３０秒。

- 金額や金種は指定した取引のものが採用される。

- QRの場合は再度の読み取りは求めず、APIをコール後、即処理が実行される。


#### 異常ケース（クレジット）

- 当然だが、媒体は支払い時と同様のものを使う必要がある。
  別媒体で取り消そうとするとエラーとなる。

- 支払と同様に、失敗すると  `transactions` を "failed" に更新する。  
  取消では、失敗時に DB: `history_uris`, `history_slips` は作成されない。

- 支払と同様に、何らかの要因でキャンセルすると、 `transactions` を "stopped" に更新する。  
 DB: `history_uris`, `history_slips` は作成されない。

  キャンセル要因は以下。
  1. タイムアウト
  2. キャンセルボタンの押下。
  3. PIN入力画面でキャンセルボタンの押下。

#### 異常ケース（電子マネー）

- 支払と同様に、失敗すると `transactions` を "failed" に更新する。   
  電子マネーはタイムアウトでも "failed" になる。

  DB: `history_uris`, `history_slips` が作成されるのは以下の場合。

  1. 成功
  2. 処理未了

- 待ち画面でキャンセルボタンの押下をすると、 `transactions` を "stopped" に更新する。  
 DB: `history_uris`, `history_slips` は作成されない。

#### 異常ケース（QR）

- 取消が失敗すると  `transactions` を "failed" に更新する。 
- 通信不良などの場合は、結果不明とし、結果を再度問い合わせる画面となる。  
  この照会の結果により `transactions` が "completed" | "failed" に更新される。　　
- 再照会も結果不明の場合、再度問い合わせ画面を表示。以降ループ。  
- 再度問い合わせる画面を中断すると  `transactions` を "completed" に更新する。  
  この場合も `history_uris`、`history_slips` は作成されるが処理未了フラグが立つ。

- カメラ読み取りのタイムアウトはない。

  DB: `history_uris`, `history_slips` が作成されるのは以下の場合。

  1. 成功
  2. 処理未了


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
- すでに取り消しされている場合以下のエラーを返す
  `status code : 400 , error code : PAYMENT_CANCELED`



## 決済系：決済の中断

- HTTPメソッド  
  POST

- 要求データ (URL)
  ```
  /vi/payments/{id}/stop
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
    "id": "20250911115240",
    "amount": 1,
    "status": "processing",
    "transaction_at": "2025-09-11T11:52:40Z"
  }
  ```

#### 説明

- 指定した取引IDの処理前中止を行う。  
  カードかざし待ち画面の状態で有効。
- 中断された取引のステータスは "stopped" となる。
- 支払にも取消にも使用可能。


#### 利用できない状態

- 指定 id に該当するデータがない場合、以下のエラーを返す。  
  `status code : 404 , error code : TRANSACTION_NOT_FOUND`
- 処理中でない取引が指定された場合以下のエラーを返す  
  `status code : 400 , error code : TRANSACTION_NOT_IN_PROGRESS`

