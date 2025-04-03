# Entra-GWS-SAML Script 参考

# Federation時
```
# 接続
Connect-MgGraph -Scopes "Directory.AccessAsUser.All", "Domain.ReadWrite.All","User.ReadWrite.All"

# 確認
Get-MgDomain

# Fedrationするドメイン名
$DomainName    = "<ドメイン名>"

# Fedration設定(EntraMFA要求時の動作)
# https://learn.microsoft.com/ja-jp/graph/api/resources/internaldomainfederation?view=graph-rest-1.0#federatedidpmfabehavior-values
#$FederatedIdpMfaBehavior = "acceptIfMfaDoneByFederatedIdp"
#$FederatedIdpMfaBehavior = "enforceMfaByFederatedIdp"
$FederatedIdpMfaBehavior = "rejectMfaByFederatedIdp"

# GWS側で表示されるSSO URLを記載
$PassiveLogOnUrl = "<GWSで表示されるURL>"
$ActiveLogOnUri = "<GWSで表示されるURL>"

# GWS側で表示される証明書
$SigningCertificate    = "-----BEGIN <省略こんな感じで改行消して一行にして貼り付け>-----END CERTIFICATE-----"

# GWS側で表示されるエンティティID(URL)
$IssuerURI = "<GWSで表示されるURL>"

# GWS側で表示されるログアウトURL
$LogOffUri = "<GWSで表示されるURL>"

# saml or wsFed
$PreferredAuthenticationProtocol = "saml"

New-MgDomainFederationConfiguration -DomainId $DomainName `
                                    -DisplayName $DomainName `
                                    -FederatedIdpMfaBehavior $FederatedIdpMfaBehavior `
                                    -PassiveSignInUri $PassiveLogOnUrl `
                                    -ActiveSignInUri $ActiveLogOnUri `
                                    -SigningCertificate $SigningCertificate `
                                    -IssuerUri $IssuerURI `
                                    -SignOutUri $LogOffUri `
                                    -PreferredAuthenticationProtocol $PreferredAuthenticationProtocol ;

# 確認
Get-MgDomain

Disconnect-MgGraph
```

# Manage時 (Get-MgDomain で設定確認可能)
```
# 接続
Connect-MgGraph -Scopes "Directory.AccessAsUser.All", "Domain.ReadWrite.All","User.ReadWrite.All"

# 確認
Get-MgDomain

# Managedにするドメイン名
$DomainName    = "<ドメイン名>"

$DomainId = (Get-MgDomainFederationConfiguration -DomainId $DomainName).Id

Remove-MgDomainFederationConfiguration -DomainId $DomainName -InternalDomainFederationId $DomainId

# 確認
Get-MgDomain

Disconnect-MgGraph
```
