# Intune Chrome OAM-URI参考値
コピペする場合は、" が ” になってないか注意

```
シークレットモードの有効化
./Device/Vendor/MSFT/Policy/Config/Chrome~Policy~googlechrome/IncognitoModeAvailability
文字列
値:
<enabled/><data id="IncognitoModeAvailability" value="0"/>
```

```
ChromeタスクマネージャでのChromeプロセス無効化許可
./Device/Vendor/MSFT/Policy/Config/Chrome~Policy~googlechrome/TaskManagerEndProcessEnabled
文字列
<enabled/>
```

[IncognitoModeAvailability-YesGWS](https://dti470.github.io/SampleCom1/SubPages1/Pic/IncognitoModeAvailability-YesGWS.png)
[IncognitoModeAvailability-NoGWS](https://dti470.github.io/SampleCom1/SubPages1/Pic/IncognitoModeAvailability-NoGWS.png)
[TaskManagerEndProcessEnabled-YesGWS](https://dti470.github.io/SampleCom1/SubPages1/Pic/TaskManagerEndProcessEnabled-YesGWS.png)
[TaskManagerEndProcessEnabled-NoGWS](https://dti470.github.io/SampleCom1/SubPages1/Pic/TaskManagerEndProcessEnabled-NoGWS.png)