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
| Terminal | /v1/terminal/actions/shutdown | POST | 業務終了 |
| Terminal | /v1/terminal/actions/inquireBalance | POST | 残高照会 | 
| Terminal | /v1/terminal/actions/charge | POST | チャージ |
| Terminal | /v1/terminal/actions/softUpdate | POST | ソフトウェア更新 |
| Terminal | /v1/terminal/actions/showMaintenance | POST | 保守メニュー表示 |


## 共通事項

### 存在しないURL
URL が定義されていないものの場合は、以下のエラーを返却する。

`status code : 404 , error code : NOT_FOUND`

### アクティベーション
すべてのAPIは、利用するために事前に端末がアクティベーションされている必要がある。  
もし、未アクティベーションの場合、以下のエラーを返却する。

`status code : 400 , error code : NOT_ACTIVATED`

なお、未アクティベーションの場合 TerminalAPI を立ち上げると Maintenanceアプリを立ち上げようとする。

### 更新中
アプリケーションのアップデート中（apkダウンロード後のインストール中）は、以下のエラーを返却する。

`status code : 400 , error code : APP_INSTALLING`

だが、実際は更新と同時にアプリケーションが終了し、 HTTP 待ち受けが閉じるので、
このコードが返却されることはない。


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
- 
- 業務開始状態でない場合は以下のエラーを返す  
  `status code : 400 , error code : APP_INSTALLING`
- 既に別の決済が実行中の場合は以下のエラーを返す  
  `status code : 400 , error code : SERVICE_NOT_AVAILABLE`

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
