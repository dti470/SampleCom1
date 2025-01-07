'''
# Microsoft Graphモジュールがインストールされているか確認
$moduleName = "Microsoft.Graph"
$module = Get-InstalledModule -Name $moduleName -ErrorAction SilentlyContinue

# Microsoft Graphモジュールがインストールされていない場合
if (-not $module) {
    Write-Host "Microsoft Graphモジュールがインストールされていません。インストールを開始します..." -ForegroundColor Cyan
    # サイレントインストール
    Install-Module -Name $moduleName -Force -Scope CurrentUser -Confirm:$false
    Write-Host "Microsoft Graphモジュールのインストールが完了しました。" -ForegroundColor Green
} else {
    # インストールされているバージョンを取得
    $installedVersion = $module.Version
    Write-Host "インストールされているMicrosoft Graphモジュールのバージョン: $installedVersion" -ForegroundColor Cyan

    # 最新バージョンの取得
    $latestVersion = Find-Module -Name $moduleName | Select-Object -ExpandProperty Version

    # 最新バージョンがインストールされていない場合、アップデートを実施
    if ($installedVersion -lt $latestVersion) {
        Write-Host "新しいバージョンが利用可能です。アップデートを開始します..." -ForegroundColor Cyan
        # サイレントアップデート
        Update-Module -Name $moduleName -Force -Scope CurrentUser -Confirm:$false
        Write-Host "Microsoft Graphモジュールのアップデートが完了しました。" -ForegroundColor Green
    } else {
        Write-Host "最新バージョンのMicrosoft Graphモジュールがすでにインストールされています。" -ForegroundColor Green
    }
}
'''

'''
# Microsoft Graphモジュールをインポート
# Import-Module Microsoft.Graph

# Graph APIに接続
Connect-MgGraph -Scopes "Group.ReadWrite.All", "User.Read.All"

# ログ保存用のフォルダを作成
$logFolder = "C:\work\logs"
if (-not (Test-Path -Path $logFolder)) {
    New-Item -ItemType Directory -Path $logFolder
}

# ログファイルのパス
$logFile = "$logFolder\GroupUserAddition_$(Get-Date -Format 'yyyyMMdd_HHmmss').log"

# グループとユーザーのCSVファイルを読み込み
$groups = Import-Csv -Path "c:\work\groups.csv"  # グループ名を含むCSV
$users = Import-Csv -Path "c:\work\users.csv"    # ユーザーUPNを含むCSV

# 成功件数と失敗件数のカウンター
$successCount = 0
$failureCount = 0

# グループ名からグループIDを取得し、ユーザーを追加
foreach ($group in $groups) {
    try {
        # グループ名を使ってグループIDを取得
        $groupInfo = Get-MgGroup -Filter "displayName eq '$($group.GroupName)'" -ErrorAction Stop
        $groupId = $groupInfo.Id
        
        Write-Host "グループ '$($group.GroupName)' のID: $groupId を取得しました。" -ForegroundColor Cyan
        # ログに出力
        $logMessage = "$(Get-Date -Format 'yyyy-MM-dd HH:mm:ss') - 成功: グループ '$($group.GroupName)' のID取得"
        Add-Content -Path $logFile -Value $logMessage
        Write-Host $logMessage

        foreach ($user in $users) {
            try {
                # ユーザーUPNを使ってユーザーIDを取得
                $userInfo = Get-MgUser -UserId $user.UserPrincipalName -ErrorAction Stop
                $userId = $userInfo.Id

                # ユーザーをグループに追加
                New-MgGroupMember -GroupId $groupId -DirectoryObjectId $userId
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
    } catch {
        # グループ情報取得失敗時の処理
        $failureCount++

        Write-Host "エラー: グループ '$($group.GroupName)' の情報取得に失敗しました。" -ForegroundColor Red
        $logMessage = "$(Get-Date -Format 'yyyy-MM-dd HH:mm:ss') - 失敗 ($failureCount): グループ '$($group.GroupName)' の情報取得に失敗"
        Add-Content -Path $logFile -Value $logMessage
        Write-Host $logMessage
    }
}

# 結果のまとめを画面とログに出力
Write-Host "処理終了。成功件数: $successCount, 失敗件数: $failureCount" -ForegroundColor Yellow
$logMessage = "$(Get-Date -Format 'yyyy-MM-dd HH:mm:ss') - 処理終了。成功件数: $successCount, 失敗件数: $failureCount"
Add-Content -Path $logFile -Value $logMessage
Write-Host $logMessage

# Microsoft Graphから切断
Disconnect-MgGraph
'''

[参考1](https://dti470.github.io/SampleCom1/SubPages1/Pic/1.png "参考1")
[参考2](https://dti470.github.io/SampleCom1/SubPages1/Pic/2.png "参考2")
[参考3](https://dti470.github.io/SampleCom1/SubPages1/Pic/3.png "参考3")
[参考4](https://dti470.github.io/SampleCom1/SubPages1/Pic/4.png "参考4")