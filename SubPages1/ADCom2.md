```
#0 以下1つのスクリプトを c:\work ディレクトリへ保存します。
#PowerShellを管理者権限で起動し、各ps1スクリプトを実行するか、
#PowerShell ISEを管理者権限で起動し、スクリプトを読み込んで実行します。
```

```
#1 作業前の情報出力　01-ADData-Get.ps1
# 作業パス
$WORKDIR = "C:\work\"
# 出力ファイル
$OUTPUTPATH = $WORKDIR + "BeforeAllUserList1.csv"

# 全ユーザの全プロパティを取得し、CSV出力
get-ADUser -filter {objectClass -eq "user"} -Properties * | Export-Csv -Path $OUTPUTPATH -NoTypeInformation -Encoding UTF8
```

```
#2 本スクリプトで修正対象ユーザ(MailAddress無)を抽出1 02-ADData-NoEmailAddress.ps1
# 作業パス
$WORKDIR = "C:\work\"
# 入力ファイル
$INPUTPATH = $WORKDIR + "BeforeAllUserList1.csv"
# 出力ファイル
$OUTPUTPATH = $WORKDIR + "NoEmailEmailUsers.csv"

# 入力ファイルをCSV取り込み
$DATALIST = Import-Csv $INPUTPATH -Encoding UTF8

# 入力ファイルから本スクリプトで修正外(必要に応じて手修正予定)ユーザの抽出、条件は以下の3つ
# 1) EmailAddress が空でである
# 2) 有効なアカウントである
# 3) クリティカルシステムアカウントではない
#    ※ 管理者アカウントなどを除外 isCriticalSystemObjectは、AADCなどでTrueの場合は、デフォルトで同期対象外となる属性
$DATALIST | Where-Object {
                            ([string]::IsNullOrEmpty($_.EmailAddress)) `
                            -and $_.Enabled -eq "True" `
                            -and -not $_.isCriticalSystemObject -eq "True" `
                            } | Export-Csv -Path $OUTPUTPATH -NoTypeInformation -Encoding UTF8
```

```
ここで手作業
NoEmailEmailUsers.csv を「AddMailAddressList1.csv」とコピーして、
コピーしたCSVの一番右の列へ「NewEmailAddress」を追加、
各ユーザのメールアドレスを入力する。
→　自動化できそうなら処理を追加
```

```
#3 設定更新と作業後の情報出力 03-ADData-AddMailAddress.ps1
# Surname と NewEmailAddress 列があるCSVを読み込んで、EmailAddressを追加
# 作業パス
$WORKDIR = "C:\work\"
# 入力ファイル
$INPUTPATH = $WORKDIR + "AddMailAddressList1.csv"
# 出力ファイル
$OUTPUTPATH = $WORKDIR + "AfterAllUserList1.csv"

# 入力ファイルをCSV取り込み
$DATALIST = Import-Csv $INPUTPATH -Encoding UTF8

# 入力ファイルCSVのSamAccountName と一致するアカウントを更新
# EmailAddressが空の場合のみ追加
foreach ($USER in $DATALIST) {
    $SAMACCOUNTNAME = $USER.SamAccountName
    Set-ADUser -Identity $SAMACCOUNTNAME -EmailAddress $USER.NewEmailAddress

    Write-Host "Updated user $SAMACCOUNTNAME with EmailAddress: $($USER.NewEmailAddress)
}

get-ADUser -filter {objectClass -eq "user"} -Properties * | Export-Csv -Path $OUTPUTPATH -NoTypeInformation -Encoding UTF8
```

```
#4 MailAddress が正しく追加されたかチェック 04-ADData-Diff-MailAddress.ps1
# 作業パス
$WORKDIR = "C:\work\"
# 入力ファイル1 作業前
$INPUTPATH1 = $WORKDIR + "BeforeAllUserList1.csv"
# 入力ファイル2 作業後
$INPUTPATH2 = $WORKDIR + "AfterAllUserList1.csv"
# 入力ファイル3 メールアドレス追加用
$INPUTPATH3 = $WORKDIR + "AddMailAddressList1.csv"
# 入力ファイル1 作業前をCSV取り込み
$INPUTFILE1 = Import-CSV -Path $INPUTPATH1 -Encoding UTF8
# 入力ファイル2 作業前をCSV取り込み
$INPUTFILE2 = Import-CSV -Path $INPUTPATH2 -Encoding UTF8
# 入力ファイル3 メールアドレス追加用をCSV取り込み
$INPUTFILE3 = Import-CSV -Path $INPUTPATH3 -Encoding UTF8


# 出力ファイル
$OUTPUTPATH = $WORKDIR + "DiffCheck.csv"

# 比較結果を格納するリスト
$COMPARISONRESULTS = @()

# ファイル1 と ファイル2 の各行を比較
# 一致キーは、SamAccountName
# 比較する値は、MailAddress
# MailAddressMatch が、Trueなら問題なし。Falseなら要確認
foreach ($ROW1 in $INPUTFILE1) {

    $ROW2 = $INPUTFILE2 | Where-Object { $_.SamAccountName -eq $ROW1.SamAccountName }

    if ($ROW2) {
        $isMailAddressMatch = $ROW1.MailAddress -eq $ROW2.MailAddres

        # 一致しなかった場合は、本作業でMailAddressを追加した行なので、AddMailAddressList1と比較する。
        if($isMailAddressMatch) {
            # 比較結果を格納(入力ファイル1と2比較)
            $COMPARISONRESULTS += [PSCustomObject]@{
            SamAccountName = $ROW1.SamAccountName
            MailAddressMatch = $isMailAddressMatch
            }
        }
        else {
            $ROW3 = $INPUTFILE3 | Where-Object { $_.SamAccountName -eq $ROW2.SamAccountName }
            $isMailAddressMatch = $ROW2.MailAddress -eq $ROW3.MailAddres

            # 比較結果を格納(入力ファイル2と3比較)
            $COMPARISONRESULTS += [PSCustomObject]@{
            SamAccountName = $ROW2.SamAccountName
            MailAddressMatch = $isMailAddressMatch
            }
        }
    }
    else {
        # ID が一致しない場合は "Not Found" と表示
        $COMPARISONRESULTS += [PSCustomObject]@{
            SamAccountName = "Not Found"
            MailAddressMatch = "Not Found"
        }
    }
}

# 結果を表示 数が多い場合は以下の通りコメントアウト
# $COMPARISONRESULTS | Format-Table -Property SamAccountName, MailAddressMatch

# 結果をCSVファイルへ出力
$COMPARISONRESULTS | Export-Csv -Path $OUTPUTPATH -NoTypeInformation -Encoding UTF8
```
