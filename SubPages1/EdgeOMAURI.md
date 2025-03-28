# Edge OAM-URI参考値

```
EdgeADMXインポート
./Device/Vendor/MSFT/Policy/ConfigOperations/ADMXInstall/Edge/Policy/EdgeAdmx
文字列
値は、EdgeEnterprise.cab を解凍 の .admx ファイルの中身をそのまま全部貼り付け
https://www.microsoft.com/ja-jp/edge/business/download?form=MA13FJ
※ Windows 64bit ポリシーをダウンロード
```

```
Edgeホームボタン表示
./Device/Vendor/MSFT/Policy/Config/Edge~Policy~microsoft_edge~Startup/ShowHomeButton
文字列
<enabled/>
```

```
Edgeホームボタン押下時のURL(でも新しいタブのまま)
./Device/Vendor/MSFT/Policy/Config/Edge~Policy~microsoft_edge_recommended~Startup_recommended/HomepageLocation_recommended
文字列
<enabled/><data id="HomepageLocation" value="https://www.mizukinana.jp/"/>
```

```
Edge起動時の動作設定
./Device/Vendor/MSFT/Policy/Config/Edge~Policy~microsoft_edge~Startup/RestoreOnStartup
文字列
<enabled/><data id="RestoreOnStartup" value="4"/>
```

```
Edge起動時のURL指定上記4の場合に動作
./Device/Vendor/MSFT/Policy/Config/Edge~Policy~microsoft_edge_recommended~Startup_recommended/RestoreOnStartupURLs_recommended
文字列
<enabled/><data id="RestoreOnStartupURLsDesc" value="1&#xF000;https://www.mizukinana.jp/"/>
```

```
Edge起動時URL指定を追加許可
./Device/Vendor/MSFT/Policy/Config/Edge~Policy~microsoft_edge~Startup/RestoreOnStartupUserURLsEnabled
文字列
<enabled/>
```

```
Edge新しいタブ押下した時のURL指定(エンドユーザ変更できない)
./Device/Vendor/MSFT/Policy/Config/Edge~Policy~microsoft_edge_recommended~Startup_recommended/NewTabPageLocation_recommended
文字列
<enabled/><data id="NewTabPageLocation" value="https://www.mizukinana.jp/"/>
```


