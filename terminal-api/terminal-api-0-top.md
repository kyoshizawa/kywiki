# TerminalAPI攻略

アプリケーション "TerminalAPI" の攻略ガイドです。

## 概要
- アンドロイドアプリ
  - 対象バージョン 13 , 5
- 決済端末 newXXXX シリーズの機能を利用した決済処理提供を大まかな役割とする。

## 外観

![アイコン](./img/outlook-1.png)

![起動後](./img/outlook-2.png)

## 利用時の構成

- TerminalAPI 単独 (+ MaintenanceAPP)

![構成](./img/structure-1.png)


  - 導入予定：エイジィ xxx  
    - ホテル向け無人決済機を構築したい。  
    - 利用者はエイジィの端末を操作するが、決済機能は Terminal API を利用したい。


- 【没】 決済アプリ + TerminalAPI
  
![構成２](./img/structure-2.png)

    - 没になった構成案。
    - この構成の場合 MultiPaymentProject でも連携のための対応が必要。  
     `MultiPaymentProject: feature/v1/terminal` ブランチに対応ソースが残されている（未マージ）
    - が、おそらく使うことはない。
    - TerminalAPI には OKICA の種別指定ができるAPIがあるが、単独で開局はしていないため  
      OKICA はこの構成ありきで組まれている。


## ビルド

- リポジトリ  
  https://github.com/MobileCreate/pay_terminal.git  
  `develop` が開発用ブランチ

- JDK  
  gradle の設定を JDK 17 使用し、デバッグ実行を確認。  



## 内部構成


(TBD)  


## 機能一覧

TerminalAPI は主に以下の種類の処理が実装されている。  
詳細は各リンク先へ

- HTTPサーバ（WebAPI）
- mDNS サービス
- Wifi Direct 接続（親）
- 各種バックグラウンド処理  
  [リンク](./terminal-api-4-background.md)
　　
　　
　