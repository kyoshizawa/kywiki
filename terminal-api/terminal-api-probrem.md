# 問題点メモ

ソフトウェア更新の状態が端末ステータスから確認できるのか。

【降った】transaction get から、取引通番が確認できない。

keystore の扱いが MaintenanceApp は違う。統一しておきたい。

【降った】MC相互認証前に PostPaymentLooper.kt が動作し、売り上げ送信するとアプリケーションがクラッシュする


設定が変更されていない場合に自動更新が動いていないのでは

 MainNotifier.setOnUpdateApkListener {
     mainViewModel.resetTimer()
 }


# スタンドアロン

別アプリから Intent で起こされていなければスタンドアロン判定。

# 取り消し

Edy と * はできない

