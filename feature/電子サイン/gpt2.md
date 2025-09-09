
非接触IC：StateCLProc/StateMsOnlineAuth で _emvCLProc.getCVM() の値で判定
接触IC：StateIcProc → EmvProcessing#emvIcProc() → emvCheckCardholderVerifyResult() でPIN/サイン/バイパス等を判定
磁気：StateMsOnlineAuth でIC/CL以外の場合に、金額や条件で CreditSettlement#setSignatureFlag() で署名要否を設定
どの方式も最終的には CreditSettlement#setSignatureFlag() で「サイン要否」を管理しています。






## メモ
