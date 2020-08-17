---
title: 'Configuración de la terminación TLS con certificados de Key Vault: PowerShell'
titleSuffix: Azure Application Gateway
description: Obtenga información sobre cómo puede integrar Azure Application Gateway con Key Vault para certificados de servidor que se adjuntan a los clientes de escucha con HTTPS habilitado.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 05/26/2020
ms.author: victorh
ms.openlocfilehash: 4a872bc63be33ebed0a8ba9d89383cdfc9feef28
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87386289"
---
# <a name="configure-tls-termination-with-key-vault-certificates-using-azure-powershell"></a>Configuración de la terminación TLS con certificados de Key Vault mediante Azure PowerShell

[Azure Key Vault](../key-vault/general/overview.md) es un almacén de secretos administrado por la plataforma que puede usar para proteger los secretos, las claves y los certificados TLS/SSL. Azure Application Gateway admite la integración con Key Vault para certificados de servidor adjuntos a clientes de escucha con HTTPS habilitado. Esta compatibilidad solo es aplicable a la versión 2 de la SKU de Application Gateway.

Para obtener más información, consulte [Terminación TLS con certificados de Key Vault](key-vault-certs.md).

Este artículo muestra cómo usar un script de Azure PowerShell para integrar el almacén de claves con la puerta de enlace de aplicaciones para los certificados de terminación TLS/SSL.

En este artículo se requiere la versión 1.0.0 del módulo de Azure PowerShell u otra de versión posterior. Para encontrar la versión, ejecute `Get-Module -ListAvailable Az`. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Para ejecutar los comandos de este tutorial, también debe crear una conexión con Azure ejecutando `Connect-AzAccount`.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar, debe tener instalado el módulo ManagedServiceIdentity:

```azurepowershell
Install-Module -Name Az.ManagedServiceIdentity
Connect-AzAccount
Select-AzSubscription -Subscription <your subscription>
```

## <a name="example-script"></a>Script de ejemplo

### <a name="set-up-variables"></a>Configurar las variables

```azurepowershell
$rgname = "KeyVaultTest"
$location = "East US"
$kv = "<your key vault name>"
$appgwName = "AppGwKVIntegration"
```
> [!IMPORTANT]
> El nombre del almacén de claves debe ser único globalmente.

### <a name="create-a-resource-group-and-a-user-managed-identity"></a>Crear un grupo de recursos y una identidad administrada por el usuario

```azurepowershell
$resourceGroup = New-AzResourceGroup -Name $rgname -Location $location
$identity = New-AzUserAssignedIdentity -Name "appgwKeyVaultIdentity" `
  -Location $location -ResourceGroupName $rgname
```

### <a name="create-a-key-vault-policy-and-certificate-to-be-used-by-the-application-gateway"></a>Crear un almacén de claves, la directiva y el certificado que usará la puerta de enlace de aplicaciones

```azurepowershell
$keyVault = New-AzKeyVault -Name $kv -ResourceGroupName $rgname -Location $location -EnableSoftDelete 
Set-AzKeyVaultAccessPolicy -VaultName $kv -PermissionsToSecrets get -ObjectId $identity.PrincipalId

$policy = New-AzKeyVaultCertificatePolicy -ValidityInMonths 12 `
  -SubjectName "CN=www.contoso11.com" -IssuerName self `
  -RenewAtNumberOfDaysBeforeExpiry 30
Set-AzKeyVaultAccessPolicy -VaultName $kv -EmailAddress <your email address> -PermissionsToCertificates create,get,list
$certificate = Add-AzKeyVaultCertificate -VaultName $kv -Name "cert1" -CertificatePolicy $policy
$certificate = Get-AzKeyVaultCertificate -VaultName $kv -Name "cert1"
$secretId = $certificate.SecretId.Replace($certificate.Version, "")
```
> [!NOTE]
> La marca -EnableSoftDelete debe usarse para que la terminación TLS funcione correctamente. Si está configurando [la eliminación temporal de Key Vault mediante el portal](../key-vault/general/soft-delete-overview.md#soft-delete-behavior), el período de retención debe mantenerse en 90 días, el valor predeterminado. Application Gateway aún no es compatible con otro período de retención. 

### <a name="create-a-virtual-network"></a>Creación de una red virtual

```azurepowershell
$sub1 = New-AzVirtualNetworkSubnetConfig -Name "appgwSubnet" -AddressPrefix "10.0.0.0/24"
$sub2 = New-AzVirtualNetworkSubnetConfig -Name "backendSubnet" -AddressPrefix "10.0.1.0/24"
$vnet = New-AzvirtualNetwork -Name "Vnet1" -ResourceGroupName $rgname -Location $location `
  -AddressPrefix "10.0.0.0/16" -Subnet @($sub1, $sub2)
```

### <a name="create-a-static-public-virtual-ip-vip-address"></a>Crear una dirección IP virtual (VIP) pública y estática

```azurepowershell
$publicip = New-AzPublicIpAddress -ResourceGroupName $rgname -name "AppGwIP" `
  -location $location -AllocationMethod Static -Sku Standard
```

### <a name="create-pool-and-front-end-ports"></a>Crear un grupo y puertos de front-end

```azurepowershell
$gwSubnet = Get-AzVirtualNetworkSubnetConfig -Name "appgwSubnet" -VirtualNetwork $vnet

$gipconfig = New-AzApplicationGatewayIPConfiguration -Name "AppGwIpConfig" -Subnet $gwSubnet
$fipconfig01 = New-AzApplicationGatewayFrontendIPConfig -Name "fipconfig" -PublicIPAddress $publicip
$pool = New-AzApplicationGatewayBackendAddressPool -Name "pool1" `
  -BackendIPAddresses testbackend1.westus.cloudapp.azure.com, testbackend2.westus.cloudapp.azure.com
$fp01 = New-AzApplicationGatewayFrontendPort -Name "port1" -Port 443
$fp02 = New-AzApplicationGatewayFrontendPort -Name "port2" -Port 80
```

### <a name="point-the-tlsssl-certificate-to-your-key-vault"></a>Seleccionar el certificado TLS/SSL en el almacén de claves

```azurepowershell
$sslCert01 = New-AzApplicationGatewaySslCertificate -Name "SSLCert1" -KeyVaultSecretId $secretId
```

### <a name="create-listeners-rules-and-autoscale"></a>Crear agentes de escucha, reglas y escalado automático

```azurepowershell
$listener01 = New-AzApplicationGatewayHttpListener -Name "listener1" -Protocol Https `
  -FrontendIPConfiguration $fipconfig01 -FrontendPort $fp01 -SslCertificate $sslCert01
$listener02 = New-AzApplicationGatewayHttpListener -Name "listener2" -Protocol Http `
  -FrontendIPConfiguration $fipconfig01 -FrontendPort $fp02
$poolSetting01 = New-AzApplicationGatewayBackendHttpSetting -Name "setting1" -Port 80 `
  -Protocol Http -CookieBasedAffinity Disabled
$rule01 = New-AzApplicationGatewayRequestRoutingRule -Name "rule1" -RuleType basic `
  -BackendHttpSettings $poolSetting01 -HttpListener $listener01 -BackendAddressPool $pool
$rule02 = New-AzApplicationGatewayRequestRoutingRule -Name "rule2" -RuleType basic `
  -BackendHttpSettings $poolSetting01 -HttpListener $listener02 -BackendAddressPool $pool
$autoscaleConfig = New-AzApplicationGatewayAutoscaleConfiguration -MinCapacity 3
$sku = New-AzApplicationGatewaySku -Name Standard_v2 -Tier Standard_v2
```

### <a name="assign-the-user-managed-identity-to-the-application-gateway"></a>Asignar la identidad administrada por el usuario a la puerta de enlace de aplicaciones

```azurepowershell
$appgwIdentity = New-AzApplicationGatewayIdentity -UserAssignedIdentityId $identity.Id
```

### <a name="create-the-application-gateway"></a>Creación de la puerta de enlace de aplicaciones

```azurepowershell
$appgw = New-AzApplicationGateway -Name $appgwName -Identity $appgwIdentity -ResourceGroupName $rgname `
  -Location $location -BackendAddressPools $pool -BackendHttpSettingsCollection $poolSetting01 `
  -GatewayIpConfigurations $gipconfig -FrontendIpConfigurations $fipconfig01 `
  -FrontendPorts @($fp01, $fp02) -HttpListeners @($listener01, $listener02) `
  -RequestRoutingRules @($rule01, $rule02) -Sku $sku `
  -SslCertificates $sslCert01 -AutoscaleConfiguration $autoscaleConfig
```

## <a name="next-steps"></a>Pasos siguientes

[Más información sobre la terminación TLS](ssl-overview.md)
