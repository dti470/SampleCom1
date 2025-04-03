# Entra-GWS-SAML Script 参考

# Federation時
```
# 接続前処理
$clientId = "<クライアントID>"
$clientSecret_plain = "<クライアントシークレット>"
$clientSecret = ConvertTo-SecureString -String $clientSecret_plain -AsPlainText -Force
$tenantId = "<テナントID>"
$scope = "https://graph.microsoft.com/.default"
$oauth = Get-MsalToken -ClientId $clientId -TenantId $tenantId -ClientSecret $clientSecret -Scopes $scope
$accessToken = ConvertTo-SecureString -String $oauth.AccessToken -AsPlainText -Force  

# 接続
Connect-MgGraph -Scopes "Directory.ReadWrite.All", "Domain.ReadWrite.All", "User.ReadWrite.All" -AccessToken $accessToken

# 確認
Get-MgDomain

# Fedrationするドメイン名
$DomainName = "<ドメイン名>"

# 任意の表示名
$DisplayName = "Google Cloud Identity"

# Fedration設定(EntraMFA要求時の動作)
# https://learn.microsoft.com/ja-jp/graph/api/resources/internaldomainfederation?view=graph-rest-1.0#federatedidpmfabehavior-values
#$FederatedIdpMfaBehavior = "acceptIfMfaDoneByFederatedIdp"
#$FederatedIdpMfaBehavior = "enforceMfaByFederatedIdp"
$FederatedIdpMfaBehavior = "rejectMfaByFederatedIdp"

# GWS側で表示されるSSO URLを記載
$PassiveLogOnUrl = "<GWSで表示されるSSOのURL>"
$ActiveLogOnUri = "<GWSで表示されるSSOのURL>"

# GWSで表示される証明書
$SigningCertificate = "-----BEGIN <省略こんな感じで改行消して一行にして貼り付け>-----END CERTIFICATE-----"

# GWSで表示されるエンティティID(URL)
$IssuerURI = "<GWSで表示されるエンティティIDのURL>"

# GWS側固定ログアウトURL
$LogOffUri = "https://accounts.google.com/logout"

# saml or wsFed
$PreferredAuthenticationProtocol = "saml"

New-MgDomainFederationConfiguration -DomainId $DomainName `
                                    -DisplayName $DisplayName `
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

# Manage時
```
# 接続前処理
$clientId = "<クライアントID>"
$clientSecret_plain = "<クライアントシークレット>"
$clientSecret = ConvertTo-SecureString -String $clientSecret_plain -AsPlainText -Force
$tenantId = "<テナントID>"
$scope = "https://graph.microsoft.com/.default"
$oauth = Get-MsalToken -ClientId $clientId -TenantId $tenantId -ClientSecret $clientSecret -Scopes $scope
$accessToken = ConvertTo-SecureString -String $oauth.AccessToken -AsPlainText -Force  

# 接続
Connect-MgGraph -Scopes "Directory.ReadWrite.All", "Domain.ReadWrite.All", "User.ReadWrite.All" -AccessToken $accessToken

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
