# 問題点メモ

ソフトウェア更新の状態が端末ステータスから確認できるのか。

transaction get から、取引通番が確認できない。

keystore の扱いが MaintenanceApp は違う。統一しておきたい。

【降った】MC相互認証前に PostPaymentLooper.kt が動作し、売り上げ送信するとアプリケーションがクラッシュする



# これは自分で

iCASDevice.java
edy だけコミットしたので、これを参考に残りも


# スタンドアロン

別アプリから Intent で起こされていなければスタンドアロン判定。

# 取り消し
Edy と * はできない
