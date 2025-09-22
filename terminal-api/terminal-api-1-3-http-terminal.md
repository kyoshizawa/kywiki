# 端末系

## 端末系：端末情報取得

- HTTPメソッド  
  GET

- 要求データ (URL)
  ```
  /vi/terminal
  ```

  なし






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

- 呼ばれた側の Activity は 初回なら onCreate() , 起動済なら onNewIntent() が実行される。

