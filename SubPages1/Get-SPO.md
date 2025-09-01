【接続】<br>
```
$adminUrl = "管理サイトトップ最後/有"
$siteUrl = "各サイトトップサイト/無"

Connect-SPOService -Url $adminUrl
```

【画面表示だけ】<br>
```
Get-SPOUser -Limit All -Site $siteUrl | 
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
