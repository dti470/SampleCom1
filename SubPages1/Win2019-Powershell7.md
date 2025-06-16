・前提
```
　Windows Server 2019 Datacenter Editionで確認
　AD未参加
　6/16 までのアップデート適用
　作業全部Build-in administrator
　MSI 64bit でインストールwget使えない
```

・結論
```
　MSIをデフォルト値でインストールすれば問題ない。
　本体/モジュール/プロファイルは、基本分離されている。
```

・ポイント
```
　システムデフォルトの PS5.1 とは別に PS7 はインストールされる。
　PowerBI への接続だけでは、PS5.1/7 の差はなく、どちらも警告なし。
　PS7は、PS5.1でインストールしたモジュールも読み込んでくれる。
　Moduleのインストール先については、下記公開資料通りに動作してない可能性があるので注意
　※ PS7は、デフォルトでカレントユーザで保存された。
　※ Moduleインストール先は、PS5.1 と PS7で完全分離ではないので注意する。
    ○ PS5.1 で PowreBIのモジュールをインストールするとPS7でも利用できる。
    ○ PS7 で、Az モジュールをCurrent/Allいずれでインストールしても、PS5.1で Get-Module -ListAvailable で検索できない。
    × PS7 で、PowerBIのPowerShellをCurrent/Allいずれで再インストールしても追加されなかった。
    → PS5.1 でインストールしたモジュールを上書きした可能性がある。
    　※フォルダ/ファイルのタイムスタンプは更新されなかった。
```

・参考<br>
　Windows への PowerShell のインストール<br>
　https://learn.microsoft.com/ja-jp/powershell/scripting/install/installing-powershell-on-windows?view=powershell-7.5

　→　インストール方法はこちら<br>

　Windows PowerShell 5.1 から PowerShell 7 への移行<br>
　https://learn.microsoft.com/ja-jp/powershell/scripting/whats-new/migrating-from-windows-powershell-51-to-powershell-7?view=powershell-7.5

　→　併存に関する情報はこちら<br>

　PowerBI PowerShellはこちら<br>
　https://learn.microsoft.com/ja-jp/powershell/power-bi/overview?view=powerbi-ps


・インストールメモ<br>
1)PowerShell 7 インストール前設定確認<br>
![事前設定確認](https://dti470.github.io/SampleCom1/SubPages1/Pic/01.Win2019-PS7-Before.png)

2)PowerBI Powershellインストール<br>
![インストール](https://dti470.github.io/SampleCom1/SubPages1/Pic/02.Win2019-PS7-Install-PBI.png)

3)PowerBI 接続 Powershell 5.1<br>
![接続](https://dti470.github.io/SampleCom1/SubPages1/Pic/03.Win2019-PS5.1-Connect-PBI.png)

4)PowerShell 7 インストール ※再起動要求無し<br>
![1](https://dti470.github.io/SampleCom1/SubPages1/Pic/04.Win2019-PS7-Install1.png)
![2](https://dti470.github.io/SampleCom1/SubPages1/Pic/05.Win2019-PS7-Install2.png)
![3](https://dti470.github.io/SampleCom1/SubPages1/Pic/06.Win2019-PS7-Install3.png)
![4](https://dti470.github.io/SampleCom1/SubPages1/Pic/07.Win2019-PS7-Install4.png)
![5](https://dti470.github.io/SampleCom1/SubPages1/Pic/08.Win2019-PS7-Install5.png)
![6](https://dti470.github.io/SampleCom1/SubPages1/Pic/09.Win2019-PS7-Install6.png)
![7](https://dti470.github.io/SampleCom1/SubPages1/Pic/09.Win2019-PS7-Install7.png)
![8](https://dti470.github.io/SampleCom1/SubPages1/Pic/10.Win2019-PS7-Install7.png)

5)PowerShell 7 インストール後設定確認<br>
![再起動前](https://dti470.github.io/SampleCom1/SubPages1/Pic/11.Win2019-PS7-BeforeRestart.png)
![再起動後](https://dti470.github.io/SampleCom1/SubPages1/Pic/12.Win2019-PS7-AfterRestart.png)
![PSPath](https://dti470.github.io/SampleCom1/SubPages1/Pic/13.Win2019-PS7-PSPath.png)

6)PowerBI 接続 Powershell 7<br>
![接続](https://dti470.github.io/SampleCom1/SubPages1/Pic/14.Win2019-PS7Connect-PBI-.png)

以上