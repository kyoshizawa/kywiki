# 取引系


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


