# データ構造

## DB

### テーブル一覧

フレームワークなどが作成するメタ情報は除く  

| | |
|---|---|
| history_aggregates | 業務状態 |
| history_okica_uris |  |
| history_slips | レシート情報用履歴 |
| history_uris | 売上情報 |
| main_app_datas | アプリデータ |
| transactions | 取引実行履歴 |


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


### history_okica_uris

(TBD)

### history_slips

- レシート用履歴情報  
- 取引の処理完了をもって作成される。取引成功か、処理未了でできる
- 自動削除されない  

#### フィールド

| シンボル名                    | 備考 |
| ---------------------------- | -- |
| id                           |  transactions.historySlipId  |
| trans_brand                  |    |
| trans_type                   |    |
| trans_type_code              |    |
| trans_result                 |  2 が処理未了  |
| trans_result_detail          |    |
| print_cnt                    |    |
| old_aggregate_order          |    |
| encrypt_type                 |    |
| cancel_flg                   |    |
| trans_id                     |    |
| merchant_name                |    |
| merchant_office              |    |
| merchant_telnumber           |    |
| car_id                       |    |
| driver_id                    |    |
| term_id                      |    |
| term_sequence                |    |
| trans_date                   |    |
| card_company                 |    |
| card_id_merchant             |    |
| card_id_customer             |    |
| card_exp_date                |    |
| card_trans_number            |    |
| edy_trans_number             |    |
| nanaco_slip_number           |    |
| slip_number                  |    |
| old_slip_number              |    |
| auth_id                      |    |
| auth_sequence_number         |    |
| commodity_code               |    |
| installment                  |    |
| point                        |    |
| point_grant_type             |    |
| point_grant_msg_one          |    |
| point_grant_msg_two          |    |
| unique_id                    |    |
| term_ident_id                |    |
| trans_amount                 |    |
| trans_specified_amount       |    |
| trans_meter_amount           |    |
| trans_adj_amount             |    |
| trans_cash_together_amount   |    |
| trans_other_amount_one_type  |    |
| trans_other_amount_one       |    |
| trans_other_amount_two_type  |    |
| trans_other_amount_two       |    |
| trans_before_balance         |    |
| trans_after_balance          |    |
| common_name                  |    |
| credit_type                  |    |
| credit_arc                   |    |
| credit_aid                   |    |
| credit_apl                   |    |
| credit_signature_flg         |    |
| codetrans_order_id           |    |
| codetrans_pay_type_name      |    |
| free_count_one               |    |
| free_count_two               |    |
| credit_kid                   |    |
| trans_complete_amount        |    |
| printing_authid              |    |
| transaction_terminal_type    |    |
| card_category                |    |
| card_brand_code              |    |
| card_brand_name              |    |
| purchased_ticket_deal_id     |    |
| trip_reservation_id          |    |
| send_cancel_purchased_ticket |    |
| watari_point                 |    |
| watari_sum_point             |    |
| watari_calidity_period       |    |


### history_uris

- 売上情報  
- 取引の処理完了をもって作成される（拒否や処理未了の場合もある  
- センター送信で削除される  

#### フィールド

| シンボル名                           | 備考 |
| ------------------------------- | -- |
| id                              |  slipId とは必ずとも一致しない  |
| pos_send                        |    |
| car_id                          |    |
| driver_id                       |    |
| term_id                         |    |
| common_name                     |    |
| trans_brand                     |    |
| trans_date                      |    |
| old_trans_date                  |    |
| trans_type                      |    |
| trans_result                    |  2 が処理未了  |
| trans_result_detail             |    |
| term_sequence                   |    |
| old_term_sequence               |    |
| trans_id                        |    |
| old_trans_id                    |    |
| card_id                         |    |
| installment                     |    |
| trans_amount                    |    |
| trans_specified_amount          |    |
| trans_meter_amount              |    |
| trans_adj_amount                |    |
| trans_cash_together_amount      |    |
| trans_other_amount_one_type     |    |
| trans_other_amount_one          |    |
| trans_other_amount_two_type     |    |
| trans_other_amount_two          |    |
| trans_before_balance            |    |
| trans_after_balance             |    |
| trans_time                      |    |
| trans_input_pin_time            |    |
| term_latitude                   |    |
| term_longitude                  |    |
| term_network_type               |    |
| term_radio_level                |    |
| encrypt_type                    |    |
| unionpay_send_date              |    |
| unionpay_proc_number            |    |
| old_unionpay_send_date          |    |
| old_unionpay_proc_number        |    |
| credit_acq_id                   |    |
| credit_ms_ic                    |    |
| credit_on_off                   |    |
| credit_chip_cc                  |    |
| credit_forced_online            |    |
| credit_forced_approval          |    |
| credit_commodity_code           |    |
| credit_aid                      |    |
| credit_entry_mode               |    |
| credit_pan_sequence_number      |    |
| credit_icterm_flag              |    |
| credit_brand_id                 |    |
| credit_key_type                 |    |
| credit_key_ver                  |    |
| credit_encryption_data          |    |
| ic_err_code                     |    |
| ic_idm                          |    |
| ic_sprwid                       |    |
| ic_statement_id                 |    |
| ic_sequence                     |    |
| ic_sflog_id                     |    |
| ic_old_statement_id             |    |
| ic_old_sflog_id                 |    |
| id_slip_number                  |    |
| id_old_slip_number              |    |
| id_error_code                   |    |
| id_term_ident_id                |    |
| id_sequence_number              |    |
| id_recognition_number           |    |
| waon_slip_number                |    |
| waon_old_slip_number            |    |
| waon_err_code                   |    |
| waon_idm                        |    |
| waon_term_ident_id              |    |
| waon_card_through_num           |    |
| waon_point_trade_value          |    |
| waon_point_grant_type           |    |
| waon_add_point_total            |    |
| waon_total_point                |    |
| nanaco_slip_number              |    |
| nanaco_err_code                 |    |
| nanaco_card_trans_number        |    |
| nanaco_term_ident_id            |    |
| rakuten_edy_err_code            |    |
| rakuten_edy_trans_number        |    |
| rakuten_edy_card_trans_number   |    |
| rakuten_edy_term_ident_id       |    |
| quicpay_slip_number             |    |
| quicpay_old_slip_number         |    |
| quicpay_err_code                |    |
| quicpay_term_ident_id           |    |
| quicpay_dealings_through_number |    |
| codetrans_order_id              |    |
| codetrans_old_order_id          |    |
| codetrans_pay_type_code         |    |
| wallet                          |    |




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


### transactions

- 取引実行履歴。
- /v1/payments を実行すると、（入力チェックエラーでない限り）成否にかかわらず作成される。
- idempotencyKey は 1日経過で NULL に更新される。

#### フィールド

| シンボル名          | 備考 |
| -------------- | -- |
| id             |    |
| idempotencyKey |    |
| exec           |    |
| amount         |    |
| status         |    |
| transactionAt  |    |
| method         |    |
| dataMethod     |    |
| historySlipId  |    |
