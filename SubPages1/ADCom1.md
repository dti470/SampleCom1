```
#0 以下4つのスクリプトを c:\work ディレクトリへ保存します。
#PowerShellを管理者権限で起動し、各ps1スクリプトを実行するか、
#PowerShellISEを管理者権限で起動し、スクリプトを読み込んで実行します。
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
#2 本スクリプトで修正対象外ユーザ(MailAddress無)を抽出1 02-ADData-NoEmailAddress.ps1
# 後続の#5でGivenName空の場合、GivenNameは修正される可能性がある
# 個別調査対象アカウント扱いになる
# 作業パス
$WORKDIR = "C:\work\"
# 入力ファイル
$INPUTPATH = $WORKDIR + "BeforeAllUserList1.csv"
# 出力ファイル
$OUTPUTPATH = $WORKDIR + "NoEmailEmailUsers.csv"

# 入力ファイルをCSV取り込み
$DATALIST = Import-Csv $INPUTPATH -Encoding UTF8

# 入力ファイルから本スクリプトで修正外(必要に応じて手修正予定)ユーザの抽出、条件は以下の3つ
# 1) 有効なアカウントである
# 2) EmailAddress が空でである
# 3) クリティカルシステムアカウントではない ※ 管理者アカウントなどを除外 isCriticalSystemObjectは、AADCなどでTrueの場合は、デフォルトで同期対象外となる属性
$DATALIST | Where-Object { $_.Enabled -eq "True" 
                            -and ([string]::IsNullOrEmpty($_.EmailAddress)) 
                            -and -not $_.isCriticalSystemObject -eq "True" 
                            } | Export-Csv -Path $OUTPUTPATH -NoTypeInformation -Encoding UTF8
```

```
#3 本スクリプトで修正対象外ユーザ(Surname無)を抽出2 03-ADData-NoSurname.ps1
# 後続の#5でGivenName空の場合、GivenNameは修正されない
# 個別調査対象アカウントになる
# 作業パス
$WORKDIR = "C:\work\"
# 入力ファイル
$INPUTPATH = $WORKDIR + "BeforeAllUserList1.csv"
# 出力ファイル
$OUTPUTPATH = $WORKDIR + "NoSurnameUsers.csv"

# 入力ファイルをCSV取り込み
$DATALIST = Import-Csv $OUTPUTPATH -Encoding UTF8

# 入力ファイルから本スクリプトで修正外(必要に応じて手修正予定)ユーザの抽出、条件は以下の5つ
# 1) Surname(姓)が空である
# 2) 有効なアカウントである
# 3) GivenName(名)が空である
# 4) EmailAddress が空でない
# 5) クリティカルシステムアカウントではない ※ 管理者アカウントなどを除外 isCriticalSystemObjectは、AADCなどでTrueの場合は、デフォルトで同期対象外となる属性
$EXTRACTION = $DATALIST | Where-Object { ([string]::IsNullOrEmpty($_.Surname))
                                         -and $_.Enabled -eq "True"
                                         -and ([string]::IsNullOrEmpty($_.GivenName))
                                         -and -not ([string]::IsNullOrEmpty($_.EmailAddress))
                                         -and -not $_.isCriticalSystemObject -eq "True"
                                          } | Export-Csv -Path $OUTPUTPATH -NoTypeInformation -Encoding UTF8
```

```
#4 設定変更対象の抽出と更新列の作成 04-ADData-Extraction-Create.ps1
# 作業パス
$WORKDIR = "C:\work\"
# 入力ファイル
$INPUTPATH = $WORKDIR + "BeforeAllUserList1.csv"
# 出力ファイル
$OUTPUTPATH = $WORKDIR + "ChangeActiveAndNoGivenName.csv"

# 入力ファイルをCSV取り込み
$DATALIST = Import-Csv $INPUTPATH -Encoding UTF8

# 入力ファイルから修正対象行を抽出、条件は下記4つ
# 1) Surname(姓)が空でない
# 2) 有効なアカウントである
# 3) GivenName(名)が空である
# 4) EmailAddress が空でない
# 5) クリティカルシステムアカウントではない ※ 管理者アカウントなどを除外 isCriticalSystemObjectは、AADCなどでTrueの場合は、デフォルトで同期対象外となる属性 
$EXTRACTION = $DATALIST | Where-Object { not ([string]::IsNullOrEmpty($_.Surname))
                                         -and $_.Enabled -eq "True"
                                         -and ([string]::IsNullOrEmpty($_.GivenName))
                                         -and -not ([string]::IsNullOrEmpty($_.EmailAddress))
                                         -and -not $_.isCriticalSystemObject -eq "True"
                                          }

# 抽出した対象行の Surname(性)の最後の一文字を除いた文字を抽出し、新しい列を行に追加(更新後のSurname(姓)になる)。文字が一文字しかない場合は、新しい列にはそのまま一文字が入り、空にはならない。
# 抽出した対象行の Surname(性)の最後の一文字を抽出し、新しい列を行に追加(更新後のGivenName(名)になる)。
$EXTRACTION | ForEach-Object {
    $WITHOUTLASTCHAR = if ($_.Surname.Length -gt 1) { $_.Surname.Substring(0, $_.Surname.Length - 1) } else { $_Surname }
    $_ | Add-Member -MemberType NoteProperty -Name "SurnameWithoutLastCharNewSurname" -Value $WITHOUTLASTCHAR

    $LASTCHAR = $_.Surname[-1]
    $_ | Add-Member -MemberType NoteProperty -Name "SurnameLastcharNewGiveName" -Value $LASTCHAR

    $_
} | Export-Csv -Path $OUTPUTPATH -NoTypeInformation -Encoding UTF8
```

```
#5 設定更新と作業後の情報出力 05-ADData-Update.ps1
# 作業パス
$WORKDIR = "C:\work\"
# 入力ファイル
$INPUTPATH = $WORKDIR + "ChangeActiveAndNoGivenName.csv"
# 出力ファイル
$OUTPUTPATH = $WORKDIR + "AfterAllUserList1.csv"

# 入力ファイルをCSV取り込み
$DATALIST = Import-Csv $INPUTPATH -Encoding UTF8

# 入力ファイルCSVのSamAccountName と一致するアカウントを更新
# Surname(姓)とGivenName(名)のみ更新
foreach ($USER in $DATALIST) {
    $SAMACCOUNTNAME = $USER.SamAccountName

    Set-ADUser -Identity $SAMACCOUNTNAME -Surname $USER.SurnameWithoutLastCharNewSurname -GivenName $USER.SurnameLastcharNewGiveName

    Write-Host "Updated user $SAMACCOUNTNAME with Surname: $($USER.SurnameWithoutLastCharNewSurname) and GivenName: $($USER.SurnameLastcharNewGiveName)"
}

get-ADUser -filter {objectClass -eq "user"} -Properties * | Export-Csv -Path $OUTPUTPATH -NoTypeInformation -Encoding UTF8
```

```
#6 DisplayName にずれがないかチェック 06-ADData-Diff-DisplayName.ps1
# 作業パス
$WORKDIR = "C:\work\"
# 入力ファイル1 作業前
$INPUTPATH1 = $WORKDIR + "BeforeAllUserList1.csv"
# 入力ファイル2 作業後
$INPUTPATH2 = $WORKDIR + "AfterAllUserList1.csv"
# 入力ファイル1 作業前をCSV取り込み
$INPUTFILE1 = Import-CSV -Path $INPUTPATH1 -Encoding UTF8
# 入力ファイル2 作業前をCSV取り込み
$INPUTFILE2 = Import-CSV -Path $INPUTPATH2 -Encoding UTF8
# 出力ファイル
$OUTPUTPATH = $WORKDIR + "DiffCheck.csv"

# 比較結果を格納するリスト
$COMPARISONRESULTS = @()


# ファイル1 と ファイル2 の各行を比較
# 一致キーは、SamAccountName
# 比較する値は、DisplayName
# DisplayNameMatch が、Trueなら問題なし。Falseなら要確認
foreach ($ROW1 in $INPUTFILE1) {

    $ROW2 = $INPUTFILE2 | Where-Object { $_.SamAccountName -eq $ROW1.SamAccountName }

    if ($ROW2) {
        $isDisplayNameMatch = $ROW1.DisplayName -eq $ROW2.DisplayName

        # 比較結果を格納
        $COMPARISONRESULTS += [PSCustomObject]@{
            SamAccountName = $ROW1.SamAccountName
            DisplayNameMatch = $isDisplayNameMatch
        }
    }
    else {
        # ID が一致しない場合は "Not Found" と表示
        $COMPARISONRESULTS += [PSCustomObject]@{
            SamAccountName = "Not Found"
            DisplayNameMatch = "Not Found"
        }
    }
}

# 結果を表示 数が多い場合は以下の通りコメントアウト
# $COMPARISONRESULTS | Format-Table -Property SamAccountName, DisplayNameMatch

# 結果をCSVファイルへ出力
$COMPARISONRESULTS | Export-Csv -Path $OUTPUTPATH -NoTypeInformation -Encoding UTF8
```


<pre>
  <code id="myCode">
  // ここにコピーしたいコードを記述

# テスト
# 結果をCSVファイルへ出力
$COMPARISONRESULTS | Export-Csv -Path $OUTPUTPATH -NoTypeInformation -Encoding UTF8

  </code>
  <button onclick="copyToClipboard('myCode')">Copy</button>
</pre>
