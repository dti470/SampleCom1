この設定を
```
./Device/Vendor/MSFT/Policy/Config/Chrome~Policy~googlechrome~Extensions/ExtensionInstallForcelist
```

下記設定から
```
<enabled/><data id="ExtensionInstallForcelistDesc" value="1&#xF000;ppnbnpeolgkicgegkbkbjmhlideopiji;https://clients2.google.com/service/update2/crx"/>
```

以下の設定へ変更
```
<enabled/><data id="ExtensionInstallForcelistDesc" value="1&#xF000;ppnbnpeolgkicgegkbkbjmhlideopiji;https://clients2.google.com/service/update2/crx&#xF000;2&#xF000;callobklhcbilhphinckomhgkigmfocg;https://clients2.google.com/service/update2/crx"/>
```
