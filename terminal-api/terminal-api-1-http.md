# HTTPサーバ

- Terminal API は起動直後、HTTP の listen を開始する。  
  このエンドポイントに対し、APIをコールできる。  
- 待ち受けポートは `8080` 固定。  
  当然アプリケーションを終了させると待ち受けも解除される。  
- API によっては Terminal API アプリ上の UI が操作されるものがある。  
  アプリケーションが前面に表示されている場合は、UI 表示に反映するが、  
  他のアプリケーションが前面に来ている場合、UI 表示は Android バージョンにより以下の挙動となる。

  - Android 13    
    利用者によりアプリケーションが前面に持ってこられた時点でUIに反映する。
    ※ OS の機能制限により、強制的に前面にもってくることができない。

  - Android 5  
    アプリケーションが強制的に前面表示される。

  


以下は API URL の一覧。

参考のIF仕様は v6

| 種類 | 和名 | URL | Method | 概要 | 
|---|---|---|---|---|
| - | ヘルスチェック | /v1/health-check | - | | 
| Payments | 決済系：決済実行 | /v1/payments | POST | |
| Payments | /v1/payments/{id} | GET | 決済の取得 |
| Payments | /v1/payments/{id}/cancel | POST | 決済の取消 |
| Payments | /v1/payments/{id}/stop | POST | 決済の中断 |
| Transactions | /v1/transactions/{id} | GET | 取引の取得 |
| Transactions | /v1/transactions | GET | 取引の一覧取得 |
| Transactions | /v1/transactions/{id}/receipt | POST | 取引のレシート印刷 |
| Terminal | /v1/terminal | GET | 端末情報取得 | 
| Terminal | /v1/terminal/status | GET | 端末ステータス取得 | 
| Terminal | /v1/terminal/actions/reopen | POST | 再開局 | 
| Terminal | 端末系：業務終了 | /v1/terminal/actions/shutdown | POST | |
| Terminal | /v1/terminal/actions/inquireBalance | POST | 残高照会 | 
| Terminal | /v1/terminal/actions/charge | POST | チャージ |
| Terminal | /v1/terminal/actions/softUpdate | POST | ソフトウェア更新 |
| Terminal | /v1/terminal/actions/showMaintenance | POST | 保守メニュー表示 |


## 共通事項

### 存在しないURL
- URL が定義されていないものの場合は、以下のエラーを返却する。

  `status code : 404 , error code : NOT_FOUND`

### アクティベーション
- すべてのAPIは、利用するために事前に端末がアクティベーションされている必要がある。  
  もし、未アクティベーションの場合、以下のエラーを返却する。

  `status code : 400 , error code : NOT_ACTIVATED`

  なお、未アクティベーションの場合 TerminalAPI を立ち上げると Maintenanceアプリを立ち上げようとする。

### 更新中
- アプリケーションのアップデート中（apkダウンロード後のインストール中）は、以下のエラーを返却する。

  `status code : 400 , error code : APP_INSTALLING`

- だが、実際は更新と同時にアプリケーションが終了し、 HTTP 待ち受けが閉じるので、このコード返却が再現されることはない。


## 業務状態
- 決済関連のAPIは業務状態というステータスをもつ。
  業務状態が "開局" でなければ、APIの実行はできない。

- "開局" とは、決済の利用にあたり必要なシンクラセンター認証などの前処理を指す。

- 業務状態は以下のバリエーションを持つ。  

  |||
  |---|---|
  | OPENING | 開局中 |
  | OPENED | 開局 |
  | CLOSED | 閉局 | 
  | EXPIRED | 期限切れ | 


### 開局

- 開局は以下のタイミングで行われる  
  - アプリケーション起動時
  - API : "再開局" の呼び出し時
  - 設定時の自動再開局

- 開局後２４時間経過すると、自動的に "期限切れ" に更新される。

- `history_aggregate` というテーブルがあるが、こちらは 「起動」～「業務終了APIコール」 までの時刻が記録されるもので、開局時刻ではないことに注意。


### 自動再開局
- アプリケーション設定で "前回の開局から24時間以内に自動で再開局" を設定すると、
  タイマー処理により２４時間後に再処理が行われる。

  ※ これは起動時と同様 paymentInitializerUseCase により実行される。

- 設定が無効であれば、再開局は実行されない。  
  その場合、運用としては API : 再開局" の呼び出しか、アプリケーションの再起動を実行する
   
## エンドポイント別

### ヘルスチェック

- 要求データ
  ```  
  なし
  ```
- 応答データ
  ```
  {
     "message": "OK"
  }
  ```

#### 説明

特に何も処理せず、固定の応答を返す。  
通信確認用。


### 決済系：決済実行

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


#### 説明

- このAPIをコールすると決済を開始し、TerminalAPI上のUIで支払待ち画面を表示する。
- その際、DB: `transactions` を新規登録する。初期状態は "processing"。

- 支払い待ち画面のタイムアウト値は３０秒。

- 支払が完了すると  `transactions` を "complated" に更新し、金種固有の情報と共に保存を行う。  
  また、DB: `history_uris`, `history_slips` を作成する。
  `history_slips`.id は transactions に関連付けされる。

- 作成された `history_uris` は売上データである。作成後、直ちにセンター送信される。

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
  

#### 利用できない状態

- 業務開始状態でない場合は以下のエラーを返す  
  `status code : 400 , error code : INVALID_OPEN_STATUS`
- 既に別の決済が実行中の場合は以下のエラーを返す  
  `status code : 409 , error code : TRANSACTION_IN_PROGRESS`
- 指定の金種が使えない状態の場合以下のエラーを返す  
  `status code : 400 , error code : PAYMENT_METHOD_UNAVAILABLE`
- 電文重複キーが同一のものが送信された場合、以下のエラーを返す
  `status code : 400 , error code : TRANSACTION_EXECUTED`


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
  