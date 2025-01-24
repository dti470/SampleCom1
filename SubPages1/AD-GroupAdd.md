```
# ADサーバ指定
$ADServer1 = "<サーバ名>"

# ログ保存用のフォルダを作成
$logFolder = "C:\work\logs"
if (-not (Test-Path -Path $logFolder)) {
    New-Item -ItemType Directory -Path $logFolder
}

# ログファイルのパス
$logFile = "$logFolder\GroupUserAddition_$(Get-Date -Format 'yyyyMMdd_HHmmss').log"

# グループとユーザーのCSVファイルを読み込み
$groups = Import-Csv -Path "c:\work\groups.csv"  # グループ名を含むCSV 先頭列属性名 GroupName
$users = Import-Csv -Path "c:\work\users.csv"    # ユーザーUPN(メールアドレス)を含むCSV 先頭列属性名 UserPrincipalName

# 成功件数と失敗件数のカウンター
$successCount = 0
$failureCount = 0

# グループ名からグループIDを取得し、ユーザーを追加
foreach ($group in $groups) {
        foreach ($user in $users) {
            try {
                # メールアドレスを変数へ設定
                $TargetMail=$user.UserPrincipalName 

                # メールアドレスを使ってユーザーDNを取得
                $UserDN = (Get-ADUser -LDAPFilter "(mail=$TargetMail)" -Server $ADServer1).DistinguishedName
                
                # グループ名を変数へ設定
                $TargetGroup=$group.GroupName

                # ユーザーをグループに追加
                Add-ADGroupMember -Identity $TargetGroup -Members $UserDN -Server $ADServer1
                $successCount++
                # 成功メッセージを画面に出力
                Write-Host "ユーザー '$($user.UserPrincipalName)' をグループ '$($group.GroupName)' に追加しました。 (成功: $successCount)" -ForegroundColor Green
                
                # ログに出力
                $logMessage = "$(Get-Date -Format 'yyyy-MM-dd HH:mm:ss') - 成功 ($successCount): ユーザー '$($user.UserPrincipalName)' をグループ '$($group.GroupName)' に追加"
                Add-Content -Path $logFile -Value $logMessage
                Write-Host $logMessage
            } catch {
                $failureCount++

                # 失敗メッセージを画面に出力
                Write-Host "エラー: ユーザー '$($user.UserPrincipalName)' のグループ '$($group.GroupName)' への追加に失敗しました。 (失敗: $failureCount)" -ForegroundColor Red
                
                # ログに出力
                $logMessage = "$(Get-Date -Format 'yyyy-MM-dd HH:mm:ss') - 失敗 ($failureCount): ユーザー '$($user.UserPrincipalName)' のグループ '$($group.GroupName)' への追加に失敗"
                Add-Content -Path $logFile -Value $logMessage
                Write-Host $logMessage
            }
        }
}

# 結果のまとめを画面とログに出力
Write-Host "処理終了。成功件数: $successCount, 失敗件数: $failureCount" -ForegroundColor Yellow
$logMessage = "$(Get-Date -Format 'yyyy-MM-dd HH:mm:ss') - 処理終了。成功件数: $successCount, 失敗件数: $failureCount"
Add-Content -Path $logFile -Value $logMessage
Write-Host $logMessage
```