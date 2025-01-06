事前にどのリソースへプロジェクト作成するか？
そのリソースへプロジェクト作成する手順はあってるか？
を確認します。

Entra → WriteBack → AD → GWS 反映されます。

同期に使用するアカウントは、事前に下記へログインして利用規約へ承諾が必要です。
https://console.cloud.google.com/
https://admin.google.com/
MFAも有効にしよう。
※Python実行アカウントと異なっても良い。両方とも特権管理者アカウントです。

1) サービスアカウント作成が許可されてるかチェック
特権管理者で以下へログイン
https://console.cloud.google.com/

ここで画面上位を確認
「組織なし」：NG
「ドメイン名」：OK
「特定のプロジェクト」：NG

→　取り合えずこのまま進める

2) メニュー移動と選択
左ペイン
IAMと管理

左ペイン下
リソースを管理

右ペイン
ドメイン名→・・・設定

ここで画面上位を確認
「組織なし」：NG
「ドメイン名」：OK
「特定のプロジェクト」：NG

3) 自分に組織ポリシー管理者を付与
左ペイン
IAMと管理
IAM
→　左から「組織ポリシー」(結構下の方にある)選択後、右から「組織ポリシー管理者」を追加付与する

4) サービスアカウントキー作成OKか確認
左ペイン
IAMと管理
組織のポリシー
以下両方非アクティブならOK

新しい方多分非アクティブ
iam.managed.disableServiceAccountKeyCreation	

古い方多分アクティブ→非アクティブへ変更
iam.disableServiceAccountKeyCreation

→　これでPythonスクリプト通ります。

5) Python スクリプト実行
画面右上の Cloud Shellを起動
プロジェクトセットのメッセージは無視

python3 <(curl -s -S -L https://git.io/password-sync-create-service-account)

1回エンター
→承認
→認証
→「リンククリック」　※これもらしても進んでエラーになるので注意
→認証→承認→エンタ
ー→JSONダウンロード→エンター
→exit

このJSONは、厳重に管理する。

・その他
Password Sync プログラムインストール後、すぐ再起動メッセージになります。

Entraパスワード変更URL
https://mysignins.microsoft.com/security-info/password/change

GWS初回ログイン時のパスワード変更を無効化しないとせっかくEntraで変更しても、
また変更してくれと言われます。

[回避設定](https://dti470.github.io/SampleCom1/SubPages1/Pic/ForceChangePass.png)