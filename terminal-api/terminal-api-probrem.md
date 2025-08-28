# 問題点メモ

ソフトウェア更新の状態が端末ステータスから確認できるのか。

【降った】transaction get から、取引通番が確認できない。

keystore の扱いが MaintenanceApp は違う。統一しておきたい。

【降った】MC相互認証前に PostPaymentLooper.kt が動作し、売り上げ送信するとアプリケーションがクラッシュする


設定が変更されていない場合に自動更新が動いていないのでは

 MainNotifier.setOnUpdateApkListener {
     mainViewModel.resetTimer()
 }


- payments の応答に通番がソース上ある。→ もともとっぽい。絶対に返却されないようなのでただの無駄コード。

- 取り消しは非接触NG？



# スタンドアロン

別アプリから Intent で起こされていなければスタンドアロン判定。

# 取り消し

Edy と * はできない

# 名前

history_slip は印字データ。

history_uri は売上データ。
