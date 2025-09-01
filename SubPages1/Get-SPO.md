【接続】<br>
```
$adminUrl = "管理サイトトップ最後/有"
$siteUrl = "各サイトトップサイト/無"

Connect-SPOService -Url $adminUrl
```

【画面表示だけ】<br>
```
Get-SPOUser -Limit All -Site $siteUrl
```
```
Get-SPOSiteGroup -Limit 2147483647 -Site $siteUrl
```


【CSV出力GUIDになる】<br>
```
Get-SPOUser -Limit All -Site $siteUrl | Export-Csv -NoTypeInformation -Encoding UTF8 -Path "C:\work\SPOU.csv"
```
```
Get-SPOSiteGroup -Limit 2147483647 -Site $siteUrl | Export-Csv -NoTypeInformation -Encoding UTF8 -Path "C:\work\SPOG.csv"
```


【CSV出力GUID変換】<br>
#ユーザー一覧<br>
```
Get-SPOUser -Limit All -Site $siteUrl | 
    Select-Object DisplayName, LoginName, @{Name='Groups'; Expression={ ($_.Groups | ForEach-Object { $_.ToString() }) -join '; '}} | 
    Export-Csv -Path "C:\work\SPOU.csv" -NoTypeInformation -Encoding UTF8
```

#グループ一覧<br>
```
$allGroups = Get-SPOSiteGroup -Limit 2147483647 -Site $siteUrl
$results = @()
foreach ($group in $allGroups) {
    $userListString = ($group.Users | ForEach-Object { $_.ToString() }) -join '; '

    $object = [PSCustomObject]@{
        Title     = $group.Title
        LoginName = $group.LoginName
        Users     = $userListString
    }

    $results += $object
}

$results | Export-Csv -Path "C:\work\SPOG.csv" -NoTypeInformation -Encoding UTF8
```
【切断】<br>
```
Disconnect-SPOService
```

【削除】<br>
#サイトから削除<br>
```
$userLoginName = "<ユーザーUPN>"
Remove-SPOUser -Site $siteUrl -LoginName $userLoginName
```

#グループから削除<br>
```
$groupName = "<GroupTitle>"
Remove-SPOUser -Site $siteUrl -Group $groupName -LoginName $userLoginName
```

【CSVインポート削除】<br>
site_remove_users.csv
```
UserPrincipalName
<メアド並べる>
```

Remove_SPOUser_From_Site.ps1
```
# =============================================================================
# [設定項目] ご自身の環境に合わせて、ここから下の項目を必ず変更してください
# =============================================================================

# 1. SharePoint管理センターのURL
$adminUrl = "<管理URL最後/有>"

# 2. 対象のSharePointサイトのURL
$siteUrl = "各サイト最後/無"

# 3. 読み込むCSVファイルのパス (削除対象のユーザーリスト)
$csvInputPath = "C:\work\site_remove_users.csv"

# 4. ログを出力するCSVファイルのパス (結果がここに保存されます)
$logOutputPath = "C:\work\site_remove_users_log.csv"

# 5. 入力CSVファイル内のUPNが記載されている列のヘッダー名
$upnColumnName = "UserPrincipalName"

# =============================================================================
# [スクリプト本体] ここから下は変更不要です
# =============================================================================

# 処理結果を格納する配列を初期化
$logRecords = @()

# SharePoint Onlineへの接続
try {
    Write-Host "SharePoint Onlineに接続しています..." -ForegroundColor Cyan
    Connect-SPOService -Url $adminUrl
}
catch {
    Write-Host "SharePoint Onlineへの接続に失敗しました。URLや認証情報を確認してください。" -ForegroundColor Red
    # スクリプトを停止
    return
}

# 入力CSVファイルの存在チェック
if (-not (Test-Path $csvInputPath)) {
    Write-Host "入力CSVファイルが見つかりません: $csvInputPath" -ForegroundColor Red
    Disconnect-SPOService
    return
}

# CSVファイルから削除対象のユーザーをインポート
$usersToRemove = Import-Csv -Path $csvInputPath

Write-Host "合計 $($usersToRemove.Count) 件のユーザー削除処理を開始します。" -ForegroundColor Green

# 各ユーザーに対してループ処理を実行
foreach ($user in $usersToRemove) {
    # CSVからユーザーのUPNを取得
    $userLoginName = $user.$upnColumnName

    # 必須項目が空でないかチェック
    if ([string]::IsNullOrWhiteSpace($userLoginName)) {
        Write-Warning "CSVファイルに行が空のUPNがあります。スキップします。"
        continue
    }

    Write-Host "処理中: $userLoginName をサイト '$siteUrl' から削除します..."

    try {
        # ユーザー削除コマンドを実行 (-Group を指定しない)
        Remove-SPOUser -Site $siteUrl -LoginName $userLoginName

        # 成功ログを記録
        $logEntry = [PSCustomObject]@{
            Timestamp         = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
            UserPrincipalName = $userLoginName
            Status            = "Success"
            Message           = "サイトから正常に削除されました。"
        }
        $logRecords += $logEntry
        Write-Host "成功: $userLoginName" -ForegroundColor Green
    }
    catch {
        # 失敗ログを記録 (エラーメッセージを取得)
        $logEntry = [PSCustomObject]@{
            Timestamp         = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
            UserPrincipalName = $userLoginName
            Status            = "Failure"
            Message           = "エラー: $($_.Exception.Message.Trim())"
        }
        $logRecords += $logEntry
        Write-Host "失敗: $userLoginName - $($_.Exception.Message.Trim())" -ForegroundColor Red
    }
}

# ログをCSVファイルに出力
if ($logRecords.Count -gt 0) {
    $logRecords | Export-Csv -Path $logOutputPath -NoTypeInformation -Encoding UTF8
    Write-Host "全ての処理が完了しました。ログファイルが出力されました: $logOutputPath" -ForegroundColor Cyan
}
else {
    Write-Host "処理対象のユーザーがいませんでした。" -ForegroundColor Yellow
}

# SharePoint Onlineから切断
Disconnect-SPOService
```
