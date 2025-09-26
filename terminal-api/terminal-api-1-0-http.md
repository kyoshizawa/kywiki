# HTTPサーバ

- Terminal API は起動直後、HTTP の listen を開始する。  
  このエンドポイントに対し、通信を行うことでAPIをコールできる。  
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
| Payments | 決済系：決済取得 | /v1/payments/{id} | GET | |
| Payments | 決済系：決済の取消 | /v1/payments/{id}/cancel | POST | |
| Payments | 決済系：決済の中断 | /v1/payments/{id}/stop | POST | |
| Transactions | 取引系：取引の取得 | /v1/transactions/{id} | GET | |
| Transactions | 取引系：取引一覧取得 | /v1/transactions | GET | |
| Transactions | 取引系：レシート印刷 | /v1/transactions/{id}/receipt | POST | |
| Terminal | 端末系：端末情報取得 | /v1/terminal | GET |  | 　　
| Terminal | 端末系：端末ステータス取得 | /v1/terminal/status | GET |  | 
| Terminal | 端末系：再開局 | /v1/terminal/actions/reopen | POST |  | 
| Terminal | 端末系：業務終了 | /v1/terminal/actions/shutdown | POST | |
| Terminal | /v1/terminal/actions/inquireBalance | POST | 残高照会 | 
| Terminal | 端末系：チャージ | /v1/terminal/actions/charge | POST |  |
| Terminal | ソフトウェア更新 | POST | /v1/terminal/actions/softUpdate | |
| Terminal | 保守メニュー表示 | POST | /v1/terminal/actions/showMaintenance | |


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


### 業務状態
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
   


## 決済系仕様

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

### 処理未了

- 電子マネー、および、QR では処理未了の特殊動作が実装されている。

#### 基本ケース
- 処理未了が発生すると、"もう一度タッチしてください" のアナウンスと共に再かざしを求める。
- 再かざしの結果、処理成功すると通常のデータとなる。
- 取消の場合は、再かざしを求めるが、未了データは発生しない。

#### マネー毎特殊運用

- edy   
  edy はタイムアウトが確認できなかった。  
  しばらくすると再かざし画面に中止ボタンが現れ、押されると普通に中止の処理となる。  
  残額照会とからめて運用する？

- iD  
  処理未了がない。再かざしに遷移しない。

- nanaco   
  nanaco はタイムアウトが確認できなかった。  
  しばらくすると再かざし画面に中止ボタンが現れ、押されると処理未了となる。  
  処理未了取引は status = "unknown" となる。

- suica  
  再かざしがタイムアウトまで行われなかった場合、処理未了取引となる。  
  処理未了取引は status = "unknown" となる。

- WAON
  しばらくすると再かざし画面に中止ボタンが現れ、押されると処理未了となる。  
  処理未了取引は status = "stopped" となる。（バグ？





## データサンプル

- 本APIで出力されるデータのサンプルを以下に記載している。

  [サンプル（EXCEL）](./files/db_sample1.xlsx)



## エンドポイント別解説


### 決済系

[こちらから](./terminal-api-1-1-http-payment.md)

### 取引系

[こちらから](./terminal-api-1-2-http-transaction.md)

### 端末系

[こちらから](./terminal-api-1-3-http-terminal.md)

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