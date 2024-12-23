```
#1 作業前の情報出力
#  最後の2行はSurname空がいるかの事前チェック、空でもシステムアカウントなら処理しないから問題ない。
$WORKDIR = "C:\work\"
$OUTPUTPATH = $WORKDIR + "BeforeAllUserList1.csv"

get-ADUser -filter {objectClass -eq "user"} -Properties * | Export-Csv -Path $OUTPUTPATH -NoTypeInformation -Encoding UTF8

$DATALIST = Import-Csv $OUTPUTPATH -Encoding UTF8
$DATALIST | Where-Object { ([string]::IsNullOrEmpty($_.Surname)) } | select SamAccountName, EmailAddress
```

```
#2 設定変更対象の抽出と更新列の作成
$WORKDIR = "C:\work\"
$INPUTPATH = $WORKDIR + "BeforeAllUserList1.csv"
$OUTPUTPATH = $WORKDIR + "ChangeActiveAndNoGivenName.csv"

$DATALIST = Import-Csv $INPUTPATH -Encoding UTF8

$EXTRACTION = $DATALIST | Where-Object { $_.Enabled -eq "True" -and ([string]::IsNullOrEmpty($_.GivenName)) -and -not ([string]::IsNullOrEmpty($_.EmailAddress)) -and -not $_.isCriticalSystemObject -eq "True" }

$EXTRACTION | ForEach-Object {
    $WITHOUTLASTCHAR = if ($_.Surname.Length -gt 1) { $_.Surname.Substring(0, $_.Surname.Length - 1) } else { $_Surname }
    $_ | Add-Member -MemberType NoteProperty -Name "SurnameWithoutLastCharNewSurname" -Value $WITHOUTLASTCHAR

    $LASTCHAR = $_.Surname[-1]
    $_ | Add-Member -MemberType NoteProperty -Name "SurnameLastcharNewGiveName" -Value $LASTCHAR


    $_
} | Export-Csv -Path $OUTPUTPATH -NoTypeInformation -Encoding UTF8
```

```
#3 設定更新と作業後の情報出力
$WORKDIR = "C:\work\"
$INPUTPATH = $WORKDIR + "ChangeActiveAndNoGivenName.csv"
$OUTPUTPATH = $WORKDIR + "AfterAllUserList1.csv"

$DATALIST = Import-Csv $INPUTPATH -Encoding UTF8

foreach ($USER in $DATALIST) {
    $SAMACCOUNTNAME = $USER.SamAccountName

    Set-ADUser -Identity $SAMACCOUNTNAME -Surname $USER.SurnameWithoutLastCharNewSurname -GivenName $USER.SurnameLastcharNewGiveName

    Write-Host "Updated user $SAMACCOUNTNAME with Surname: $($USER.SurnameWithoutLastCharNewSurname) and GivenName: $($USER.SurnameLastcharNewGiveName)"
}

get-ADUser -filter {objectClass -eq "user"} -Properties * | Export-Csv -Path $OUTPUTPATH -NoTypeInformation -Encoding UTF8
```