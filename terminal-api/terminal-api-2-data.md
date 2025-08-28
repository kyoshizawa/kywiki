# データ構造

## DB

### テーブル一覧

| | |
|---|---|
| history_aggregates | 業務状態 |
| main_app_datas | アプリデータ |


### history_aggregates

- 集計履歴。  
  集計開始時刻 ~ 終了時刻 間 の *** を管理する
- 起動するごとに（業務開始ごとに）連番 = 0 のレコードが作成される。  
  前回のレコードは　連番 = 1 以降にスライドされる。
- 最大で６レコードまで保管され、連番 = 5 が最古となる。

#### フィールド

| シンボル名 | 備考 |
|---|---|
| id | 自動採番 |
| aggregate_history_order | 連番 |
| aggregate_start_datetime | 集計開始時刻 |
| aggregate_end_datetime | 集計終了時刻 |

- aggregate_history_order  
  先頭を 0 として扱っている


### main_app_datas

- アプリデータ。UIの起動時に削除 / 新規登録される。
- 参照されている場所は見当たらない。
  

#### フィールド

| シンボル名 | 備考 |
|---|---|
| id | |
| startupType | | 
| model | |
| driverCd | | 
| driverName | |
| carNo | |
| amount | |
| organizationId | |
| deviceId | |
| isInput1yen | | 
| prepaidServiceDomain | |
| prepaidServiceKey | |
| isDemoMode | |
| organizationName | |
| organizationParentName | |

