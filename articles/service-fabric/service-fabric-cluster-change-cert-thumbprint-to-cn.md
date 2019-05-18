---
title: Actualizar un clúster de Azure Service Fabric para usar un nombre común del certificado | Microsoft Docs
description: Obtenga información acerca de cómo puede hacer que un clúster de Service Fabric use el nombre común del certificado en vez de las huellas digitales del mismo.
services: service-fabric
documentationcenter: .net
author: aljo-microsoft
manager: chackdan
editor: aljo
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 01/01/2019
ms.author: aljo
ms.openlocfilehash: c199bd7314cb076def497bc18030f783eb23f4be
ms.sourcegitcommit: 3675daec6c6efa3f2d2bf65279e36ca06ecefb41
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65620231"
---
# <a name="change-cluster-from-certificate-thumbprint-to-common-name"></a>Cambiar el clúster para que use un nombre común en vez de la huella digital del certificado
No hay dos certificados que puedan tener la misma huella digital, lo que dificulta la sustitución o administración del certificado de clúster. Sin embargo, varios certificados pueden tener el mismo nombre o asunto común.  Si cambia un clúster implementado para que use nombres comunes del certificado en vez de las huellas digitales del mismo, será mucho más fácil administrar los certificados. En este artículo se describe cómo actualizar un clúster activo de Service Fabric para que use un nombre común del certificado en lugar de la huella digital del certificado.

>[!NOTE]
> Si tiene dos huellas digitales declaradas en la plantilla, deberá realizar dos implementaciones.  La primera implementación se realiza antes de seguir los pasos descritos en este artículo.  La primera implementación establece la propiedad **thumbprint** en la plantilla para el certificado que se usa y quita la propiedad **thumbprintSecondary**.  Para la segunda implementación, siga los pasos descritos en este artículo.
 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="get-a-certificate"></a>Obtener un certificado
Primero, obtenga un certificado de una [entidad de certificación (CA)](https://wikipedia.org/wiki/Certificate_authority).  El nombre común del certificado debe ser el nombre de host del clúster.  Por ejemplo, "myclustername.southcentralus.cloudapp.azure.com".  

Si quiere realizar pruebas, puede obtener un certificado que haya firmado una CA de una entidad de certificación gratuita o pública.

> [!NOTE]
> Recuerde que no se admiten certificados autofirmados (incluyendo aquellos que se crean al implementar un clúster de Service Fabric en Azure Portal).

## <a name="upload-the-certificate-and-install-it-in-the-scale-set"></a>Cargar el certificado e instalarlo en el conjunto de escalado
En Azure, se implementa un clúster de Service Fabric en un conjunto de escalado de máquinas virtuales.  Cargue el certificado a un almacén de claves e instálelo en el conjunto de escalado de máquinas virtuales en el que se está ejecutando el clúster.

```powershell
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser -Force

$SubscriptionId  =  "<subscription ID>"

# Sign in to your Azure account and select your subscription
Login-AzAccount -SubscriptionId $SubscriptionId

$region = "southcentralus"
$KeyVaultResourceGroupName  = "mykeyvaultgroup"
$VaultName = "mykeyvault"
$certFilename = "C:\users\sfuser\myclustercert.pfx"
$certname = "myclustercert"
$Password  = "P@ssw0rd!123"
$VmssResourceGroupName     = "myclustergroup"
$VmssName                  = "prnninnxj"

# Create new Resource Group 
New-AzResourceGroup -Name $KeyVaultResourceGroupName -Location $region

# Create the new key vault
$newKeyVault = New-AzKeyVault -VaultName $VaultName -ResourceGroupName $KeyVaultResourceGroupName `
    -Location $region -EnabledForDeployment 
$resourceId = $newKeyVault.ResourceId 

# Add the certificate to the key vault.
$PasswordSec = ConvertTo-SecureString -String $Password -AsPlainText -Force
$KVSecret = Import-AzureKeyVaultCertificate -VaultName $vaultName -Name $certName `
    -FilePath $certFilename -Password $PasswordSec

$CertificateThumbprint = $KVSecret.Thumbprint
$CertificateURL = $KVSecret.SecretId
$SourceVault = $resourceId
$CommName    = $KVSecret.Certificate.SubjectName.Name

Write-Host "CertificateThumbprint    :"  $CertificateThumbprint
Write-Host "CertificateURL           :"  $CertificateURL
Write-Host "SourceVault              :"  $SourceVault
Write-Host "Common Name              :"  $CommName    

Set-StrictMode -Version 3
$ErrorActionPreference = "Stop"

$certConfig = New-AzVmssVaultCertificateConfig -CertificateUrl $CertificateURL -CertificateStore "My"

# Get current VM scale set 
$vmss = Get-AzVmss -ResourceGroupName $VmssResourceGroupName -VMScaleSetName $VmssName

# Add new secret to the VM scale set.
$vmss = Add-AzVmssSecret -VirtualMachineScaleSet $vmss -SourceVaultId $SourceVault `
    -VaultCertificate $certConfig

# Update the VM scale set 
Update-AzVmss -ResourceGroupName $VmssResourceGroupName -Verbose `
    -Name $VmssName -VirtualMachineScaleSet $vmss 
```

>[!NOTE]
> Los secretos de un conjunto de escalado no admiten el mismo identificador de recurso para dos secretos independientes, porque cada secreto es una versión única de un recurso. 

## <a name="download-and-update-the-template-from-the-portal"></a>Descargar y actualizar la plantilla desde el portal
El certificado se ha instalado en el conjunto de escalado subyacente, pero también debe actualizar el clúster de Service Fabric para que use ese certificado y su nombre común.  A continuación, descargue la plantilla para implementar el clúster.  Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya al grupo de recursos que hospeda el clúster.  En **Configuración**, seleccione **Implementaciones**.  Seleccione la implementación más reciente y haga clic en **Ver plantilla**.

![Ver plantillas][image1]

Descargue los archivos JSON de la plantilla y los parámetros en el equipo local.

En primer lugar, abra el archivo de parámetros en el editor de texto y agregue el siguiente valor de parámetro:
```json
"certificateCommonName": {
    "value": "myclustername.southcentralus.cloudapp.azure.com"
},
```

A continuación, abra el archivo de plantilla en el editor de texto y realice tres actualizaciones para admitir el nombre común del certificado.

1. En la sección **Parámetros**, agregue un parámetro *certificateCommonName* parámetro:
    ```json
    "certificateCommonName": {
        "type": "string",
        "metadata": {
            "description": "Certificate Commonname"
        }
    },
    ```

    También puede quitar el *certificateThumbprint*, ya no se puede hacer referencia en la plantilla de Resource Manager.

2. En el recurso **Microsoft.Compute/virtualMachineScaleSets**, actualice la extensión de máquina virtual para usar el nombre común en la configuración del certificado en lugar de la huella digital.  En **virtualMachineProfile**->**extensionProfile**->**extensiones**->**propiedades**->**configuración**->**certificado**, agregue `"commonNames": ["[parameters('certificateCommonName')]"],` y quite `"thumbprint": "[parameters('certificateThumbprint')]",`.
    ```json
        "virtualMachineProfile": {
        "extensionProfile": {
            "extensions": [
                {
                    "name": "[concat('ServiceFabricNodeVmExt','_vmNodeType0Name')]",
                    "properties": {
                        "type": "ServiceFabricNode",
                        "autoUpgradeMinorVersion": true,
                        "protectedSettings": {
                            "StorageAccountKey1": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('supportLogStorageAccountName')),'2015-05-01-preview').key1]",
                            "StorageAccountKey2": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('supportLogStorageAccountName')),'2015-05-01-preview').key2]"
                        },
                        "publisher": "Microsoft.Azure.ServiceFabric",
                        "settings": {
                            "clusterEndpoint": "[reference(parameters('clusterName')).clusterEndpoint]",
                            "nodeTypeRef": "[variables('vmNodeType0Name')]",
                            "dataPath": "D:\\SvcFab",
                            "durabilityLevel": "Bronze",
                            "enableParallelJobs": true,
                            "nicPrefixOverride": "[variables('subnet0Prefix')]",
                            "certificate": {
                                "commonNames": [
                                    "[parameters('certificateCommonName')]"
                                ],
                                "x509StoreName": "[parameters('certificateStoreValue')]"
                            }
                        },
                        "typeHandlerVersion": "1.0"
                    }
                },
    ```

3.  En el recurso **Microsoft.ServiceFabric/clusters**, actualice la versión de API a "2018-02-01".  A continuación, agregue un valor de configuración **certificateCommonNames** con la propiedad **commonNames** y quite la configuración del **certificado** (con la propiedad de la huella digital), tal como se indica en el siguiente ejemplo:
    ```json
    {
        "apiVersion": "2018-02-01",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        "dependsOn": [
            "[concat('Microsoft.Storage/storageAccounts/', variables('supportLogStorageAccountName'))]"
        ],
        "properties": {
            "addonFeatures": [
                "DnsService",
                "RepairManager"
            ],
            "certificateCommonNames": {
                "commonNames": [
                    {
                        "certificateCommonName": "[parameters('certificateCommonName')]",
                        "certificateIssuerThumbprint": ""
                    }
                ],
                "x509StoreName": "[parameters('certificateStoreValue')]"
            },
        ...
    ```

## <a name="deploy-the-updated-template"></a>Implementar la plantilla actualizada
Vuelva a implementar la plantilla actualizada después de realizar los cambios.

```powershell
$groupname = "sfclustertutorialgroup"

New-AzResourceGroupDeployment -ResourceGroupName $groupname -Verbose `
    -TemplateParameterFile "C:\temp\cluster\parameters.json" -TemplateFile "C:\temp\cluster\template.json" 
```

## <a name="next-steps"></a>Pasos siguientes
* Obtenga más información sobre la [seguridad del clúster](service-fabric-cluster-security.md).
* Obtenga información acerca de cómo [sustituir un certificado de clúster](service-fabric-cluster-rollover-cert-cn.md)
* [Actualizar y administrar certificados del clúster](service-fabric-cluster-security-update-certs-azure.md)

[image1]: ./media/service-fabric-cluster-change-cert-thumbprint-to-cn/PortalViewTemplates.png
