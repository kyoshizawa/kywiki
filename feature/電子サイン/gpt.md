# 推奨アーキテクチャ（鍵を“画像ごと”に払い出す）

## 全体フロー
1. **鍵要求（TerminalAPI → サーバ）**  
   - サーバは **KMS GenerateDataKey(256bit)** で乱数鍵を生成  
   - 返すもの：**平文鍵（短期利用）**＋**CiphertextBlob**（KMSで復号可能な暗号化鍵）＋`imageKeyId`（内部ID）

2. **HMAC 計算（端末内）**  
   - TerminalAPI は受け取った平文鍵で `HMAC-SHA256(file.png)` をストリーム計算  
   - 平文鍵はメモリ上のみで使用（すぐゼロ化/GC）

3. **アップロード**  
   - 送信：`file.png`、`mac(hex)`、`imageKeyId`、`ciphertextBlob`、（必要なら）`transactionId`

4. **サーバ検証 & 保存**  
   - KMS `Decrypt(ciphertextBlob)` → 平文鍵  
   - サーバ側でも `HMAC-SHA256(file)` を再計算して `mac` と一致確認  
   - 保存：S3 等に `file.png`、DBに `transactionId / imageId / mac / ciphertextBlob / meta`

---

# API 仕様（案）

## 1. 鍵払い出し
`POST /v1/transactions/{transactionId}/images/key`

- Req: `{ "mime":"image/png", "size":123456 }`
- Res:
  ```json
  {
    "imageKeyId": "imgk_...",
    "ciphertextBlob": "BASE64...",
    "plaintextKey": "BASE64...",
    "expiresAt": "2025-09-09T01:23:45Z"
  }
  ```

## 2. 画像＋MAC アップロード
`POST /v1/transactions/{transactionId}/images`

- multipart/form-data:
  - `file`: バイナリ（png）
  - `macHex`: 文字列（HMAC-SHA256）
  - `imageKeyId`: 文字列
  - `ciphertextBlob`: 文字列（BASE64）

## 3. 検証（任意の再検証API）
`POST /v1/images/{imageId}/verify`

---

# Android（TerminalAPI）側 実装ポイント
- `plaintextKey` は**永続化しない**
- 計算直後に送信・破棄
- 画像ファイルは既存の保存仕様に合わせて扱う

---

# サーバ側 実装ポイント（要約）
- **キー管理**: `GenerateDataKey` を採用  
- **保存**: 画像は S3、DB は RDS/DynamoDB  
- **検証**: 受信時に都度検証。不一致は即 400/422  

---

# ハッシュアルゴリズム
- **HMAC-SHA256**（要件通り）  
- 送受は **hex** 推奨

---

# 工数見積（目安）

## Android（TerminalAPI）側：3.5 ～ 6.5 人日
## サーバ側：5 ～ 9 人日
### 合計：8.5 ～ 15.5 人日

---

# リスク & 決めどころ
- **鍵配布の有効期限**: 短いほど安全だが、通信不安定で再取得が増える  
- **画像サイズ/回線**: 大容量時のタイムアウト対策（チャンク/再送）  
- **KMS コスト/スロットリング**: GenerateDataKey/Decrypt の呼び出し頻度管理  
- **削除/再アップロード**時の整合性: `imageKeyId` の再生成ポリシー  
- **監査ログ**: 鍵操作（発行/復号）の記録をどの粒度で残すか
