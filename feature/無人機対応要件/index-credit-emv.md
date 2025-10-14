# クレジット(EMV仕様)

## 端末タイプ

Terminal Type = 9F35

22 

## 端末能力

Terminal Capabilities = 9F33

端末がもつ能力を示す定義：

| エリア | ビット | 名前 | 意味 |
|---|---|---|---|
| 1バイト目 | b8 |  Manual key entry | カード番号の手打ち入力をサポートするか | 
| | b7 | Magnetic stripe | 磁気リーダーがあり、機能するか |
| | b6 | IC with contacts | 接触ICリーダーがあり、機能するか |
| | b5 | RFU | 将来用に予約済 |
| | b4 | Contactless chip | 非接触リーダー(EMV)があり、機能するか |
| | b3 | Contactless magstripe | 非接触リーダー(磁気入力モード)があり、機能するか |
| | b2 | RFU | 将来用に予約済 |
| | b1 | RFU | 将来用に予約済 |
| 2バイト目 | b8 |  Plaintext PIN for ICC | 入力されたPINを平文でカードに渡せるか | 
| | b7 | Enciphered PIN for online | 入力されたPINを暗号化しオンラインで検証できるか |
| | b6 | Signature | 署名記載欄の出力を端末がサポートするか |
| | b5 | Enciphered PIN for offline | 入力されたPINを暗号化しオフラインで検証できるか |
| | b4 | Enciphered PIN + plaintext PIN | 入力されたPINを平文＋暗号化の両方でカードに渡しオフラインで突合検証できるか |
| | b3 | No CVM required | CVMなしカードを端末がサポートするか |
| | b2 | RFU | 将来用に予約済 |
| | b1 | RFU | 将来用に予約済 |
| 3バイト目 | b8 | SDA | 静的データ認証をサポートするか |
| | b7 | DDA | 動的データ認証をサポートするか |
| | b6 | Card capture | 端末にカード回収機が存在するか |
| | b5 | TRM: floor limit | フロアリミットを使ったリスク管理をサポートするか |
| | b4 | TRM: random selection | ランダム抽出によるリスク管理をサポートするか |
| | b3 | TRM: velocity checking | 速度チェックによるリスク管理をサポートするか |
| | b2 | Issuer authentication | イシュア認証をサポートするか |
| | b1 | RFU | 将来用に予約済 |

※ 静的データ認証 ： カードの固定データから作成したハッシュのようなものが、端末でCA公開鍵から作成したものと一致するか検証する方式  
※ 動的データ認証 ： 端末が送ったランダムデータからカードがハッシュのようなものを作成し、端末で検証、正規鍵をカードが持っているか検証する方式  
※ ランダム抽出：フロアリミット内でも確率でオンラインにする。オフライン用の機能  
※ 速度チェック：一定期間内に同じカードが複数回処理されるとオンライン判断する。オフライン用の機能  
※ イシュア認証：オンラインにてイシュアからの応答に含まれる tag 0x91 を端末がカードに渡して検証する方式


### 実際の設定は

No CVM の Terminal Capabilities : E008C8

