# 端末系

## 端末系：端末情報取得

- HTTPメソッド  
  GET

- 要求データ (URL)
  ```
  /vi/terminal
  ```

  なし

- 応答データ
  ```
  {
    "terminal_no": "3080902100007",
    "device_no": "107",
    "staff_no": "9999",
    "product_code": "0000270",
    "receipt_tel_no": "000-0000-0570",
    "receipt_branch_office_name": "あんどう支社",
    "receipt_sales_office_name": "あんどう営業所A",
    "receipt_tax": 0.1,
    "invoice_no": "T1234500000543",
    "battery": 100,
    "radio": 0,
    "app_version": "1.0.0",
    "storage_free_space": 9486,
    "memory_free_space": 934,
    "memory_usage": 845,
    "uptime": 668,
    "serial_no": "9320001597",
    "model": "NEW9310Pro",
    "os_version": "Android 13",
    "firm_version": "bengal_32go-userdebug 13 TKQ1.221013.002 eng.panjp.20250814.194747 release-keys",
    "payment_methods": [
      {
        "enabled": true,
        "method": "credit"
      },
      {
        "enabled": true,
        "method": "suica"
      },
      {
        "enabled": true,
        "method": "id"
      },
      {
        "enabled": true,
        "method": "waon"
      },
      {
        "enabled": true,
        "method": "nanaco"
      },
      {
        "enabled": true,
        "method": "edy"
      },
      {
        "enabled": true,
        "method": "quicpay"
      },
      {
        "enabled": false,
        "method": "okica"
      },
      {
        "enabled": true,
        "method": "qr"
      }
    ],
    "wifi_info": {
      "ip_address": "192.168.251.25",
      "on": true,
      "signal_level": 4,
      "signal_strength": -41,
      "ssid": "MCI-AP01-WPA2"
    },
    "ethernet_info": {
      "ip_address": ""
    }
  }
  ```
  |  |  |
  |---|---|
  | terminal_no | 出荷時設定された端末番号 |
  | device_no | 出荷時設定された任意機器番号 |
  | staff_no | 出荷時設定された係員番号 （現在変更方法なし） |
  | product_code | 商品区分コード（クレジット決済に使用する） |
  | receipt_tel_no | 出荷時設定された組織の電話番号（レシートに使用する）| 
  | receipt_branch_office_name | 出荷時設定された組織の支社名（レシートに使用する）| 
  | receipt_sales_office_name | 出荷時設定された組織の営業所名（レシートに使用する） |
  | receipt_tax | 出荷時設定された組織の消費税率（レシートに使用する） |
  | invoice_no | 出荷時設定された組織のインボイス番号（レシートに使用する） |
  | battery | バッテリー残量（％） |
  | radio | 電波強度（SIM利用時、アプリが前面時に更新） |
  | app_version | アプリケーションのバージョン（ビルド時のversionName） |
  | storage_free_space | ストレージ空き容量（MB） |
  | memory_free_space | メモリ空き容量（MB） |
  | memory_usage | メモリ使用量（MB） |
  | uptime | ブートからの経過時間（秒） |
  | serial_no | シリアル番号 |
  | model | 端末モデル（"NEW9310Pro" など） |
  | os_version | 端末のOSバージョン （"Android 13" など）|
  | firm_version | OSのビルド情報 |
  | mdns | 現状含まれない。バグ |
  | payment_methods | マネーの利用可否（以下、リソース Object1 の配列） |
  | wifi_info | wifi情報（以下、リソース WifiInfo ） |
  | ethernet_info | イーサネット情報（以下、リソース EthernetInfo ） |

##### Object1
  |||
  |---|---|
  | enabled | 利用できるなら true |
  | method | credit, suica, id, waon, nanaco, edy, quicpay, okica, qr のいずれか |
  
##### WifiInfo
  |||
  |---|---|
  | on | Wifiが on か |
  | ssid | 接続中のSSID |
  | signal_strength | 電波強度 RSSI （–30 dBm … 最高レベル –90 dBm以下 … 通信困難） |
  | signal_level | 電波レベル （RSSI を 0~4 に変換したもの） | 
  | ip_address | WIFIネットワーク上のIPアドレス |
  
##### EthernetInfo
  |||
  |---|---|
  | ip_address | LANネットワーク上のIPアドレス |
  

#### 説明

- 端末の基本情報を取得する。
- 取得できるデータはあらかじめ設定されたマスタデータから、リソース状態などリアルタイムに変化するものまで様々である。
- レシート情報などをクライアントが自前で構築する場合は、このAPIで得られる情報は  
  必要な情報となる。
- センターからの取得値を返しているため、通信不良時は特定の項目が空で応答される。


## 端末系：端末ステータス取得

- HTTPメソッド  
  GET

- 要求データ (URL)
  ```
  /vi/terminal/status
  ```

  なし

- 応答データ
  ```
  {
    "biz_status": "started",
    "biz_started_at": "2025-09-26T10:08:38Z",
    "open_status": "opened",
    "open_at": "2025-09-26T10:08:38Z",
    "soft_update_status": "latest"
  }
  ```

  |  |  |
  |---|---|
  | biz_status | 業務状態 (stopped, starting, started, stopping) |
  | biz_started_at | started になった時間 yyyy-MM-ddTHH:mm:ssZ |
  | open_status | 開局状態 (closed, opening, opened, expired) |
  | opened_at | opened になった時間 yyyy-MM-ddTHH:mm:ssZ |
  | soft_update_status | ソフト更新状態 （latest, has_update, updating）| 


#### 説明
- 端末のステータスを取得。  
  ステータスとは 業務状態 などを指す模様。
- 業務状態と開局状態はほぼ同じ意味。
- 業務状態は 起動時 stopeed で始まり、開局が終わると started になる。  
  stopping になることはない。
- 開局状態も 起動時 closed で始まり、業務状態 started と同じタイミングで opened になる。  
  開局状態は ２４時間経過すると expired になる。  
  設定により再開局が24時間後に行われる。
- このAPIは Preference に書かれているステータスを応答する。


## 端末系：再開局

- HTTPメソッド  
  POST
  
- 要求データ (body)  

  ```
  { }
  ```
  なし

- 応答データ（body）

  ```
  {  }
  ```
  なし

#### 説明
- アプリ起動時に実行される開局処理を手動で再実行する。
- 

#### 利用できない状態
- 業務状態が開始済でないなら、以下のエラーを返す  
  `status code : 400 , error code : INVALID_BIZ_STATUS`

- 開局処理実行中なら、以下のエラーを返す  
  `status code : 400 , error code : INVALID_OPEN_STATUS`


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

- HTTPメソッド  
  POST

- 要求データ（body）
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
- show_confirm に true が指定されている場合、UI で確認ダイアログを挟む動作になる。  
  規定値は true。
- この処理は history_aggregates に終了時刻を記録し、ローテーションする。


#### 利用できない状態

- シャットダウンできない状態ならエラーを返す。

  `status code : 400 , error code : INVALID_SOFT_STATUS`

- "シャットダウンできない状態" とは、具体的には以下を指す。
  - 決済 / 取消処理中
  - 残高照会中


## 端末系：ソフトウェア更新

- HTTPメソッド  
  POST

- 要求データ（URL）
  ```
  /v1/terminal/actions/softUpdate
  ```

  なし

- 応答データ  

  　ストリーム配信

    ```
    : ping
    progress:0
    progress:25
    progress:50
    progress:75
    progress:100
    install:start
    note:app-will-restart
    completed
    ```

#### 説明

- TerminalAPI のソフトウェア更新を試行する。
- このAPIはまずサーバに最新バージョンを問い合わせる。   
- サーバに公開されているものと比較し、自アプリの VersionCode が低いなら   
  アップデート処理が行われる。
- 同じか、それ以上ならこのAPIは何もしない。

#### 利用できない状態

- 既に更新ファイルダウンロード中なら以下のエラーを返す。  

  `status code : 400 , error code : INVALID_SOFT_STATUS`



## 端末系：保守メニュー表示

- HTTPメソッド  
  POST

- 要求データ (URL)
  ```
  /vi/terminal/actions/showMaintenance
  ```

  なし

- 応答データ
  ```
  {
  }
  ```
  なし

#### 説明

- 端末にインストールされている `MaintenanceAPP` を起動し、全面表示する。
- この操作により、TerminalAPI アプリは MaintenanceAPP の裏側に隠れる形となる。  
  MaintenanceAPP 上で閉じるボタンを押すと、特殊なUI操作を行っていない限りは、  
  TerminalAPI が再度最前面にくる。

- この操作は Android の Intent を用いて行われるが、その際引数として、  
  TerminalAPI が所持している TerminalInfo を受け渡す。

- 呼ばれた側の Activity は 初回なら onCreate() が実行される。
  既に保守メニューが前面にある場合、何も起きない。


