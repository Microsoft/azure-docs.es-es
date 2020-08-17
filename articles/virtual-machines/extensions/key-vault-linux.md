---
title: Extensión de máquina virtual de Azure Key Vault para Linux
description: Implemente un agente que realice la actualización automática de certificados de Key Vault en máquinas virtuales mediante una extensión de máquina virtual.
services: virtual-machines-linux
author: msmbaldwin
tags: keyvault
ms.service: virtual-machines-linux
ms.topic: article
ms.date: 12/02/2019
ms.author: mbaldwin
ms.openlocfilehash: d2deb59b5a10177b1a6e57046c013ec9dac0fb06
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87010808"
---
# <a name="key-vault-virtual-machine-extension-for-linux"></a>Extensión de máquina virtual de Key Vault para Linux

La extensión de máquina virtual de Key Vault proporciona la actualización automática de certificados almacenados en un almacén de claves de Azure. En concreto, la extensión supervisa la lista de certificados observados almacenados en los almacenes de claves.  Al detectar un cambio, la extensión recupera e instala los certificados correspondientes. Microsoft publica la extensión de máquina virtual de Key Vault y es compatible con esta, actualmente en máquinas virtuales Linux. En este documento se especifican las plataformas compatibles, configuraciones y opciones de implementación de la extensión de máquina virtual de Key Vault para Linux. 

### <a name="operating-system"></a>Sistema operativo

La extensión de máquina virtual de Key Vault admite estas distribuciones de Linux:

- Ubuntu-1604
- Ubuntu-1804
- Debian-9
- Suse-15 

### <a name="supported-certificate-content-types"></a>Tipos de contenido de certificado admitidos

- PKCS #12
- PEM

## <a name="extension-schema"></a>Esquema de extensión

El siguiente JSON muestra el esquema para la extensión de máquina virtual de Key Vault. La extensión no requiere configuración protegida, ya que todas sus opciones se consideran información sin riesgo de seguridad. La extensión requiere una lista de secretos supervisados, frecuencia de sondeo y el almacén de certificados de destino. Concretamente:  
```json
    {
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "name": "KVVMExtensionForLinux",
      "apiVersion": "2019-07-01",
      "location": "<location>",
      "dependsOn": [
          "[concat('Microsoft.Compute/virtualMachines/', <vmName>)]"
      ],
      "properties": {
      "publisher": "Microsoft.Azure.KeyVault",
      "type": "KeyVaultForLinux",
      "typeHandlerVersion": "1.0",
      "autoUpgradeMinorVersion": true,
      "settings": {
        "secretsManagementSettings": {
          "pollingIntervalInS": <polling interval in seconds, e.g. "3600">,
          "certificateStoreName": <It is ignored on Linux>,
          "linkOnRenewal": <Not available on Linux e.g.: false>,
          "certificateStoreLocation": <disk path where certificate is stored, default: "/var/lib/waagent/Microsoft.Azure.KeyVault">,
          "requireInitialSync": <initial synchronization of certificates e..g: true>,
          "observedCertificates": <list of KeyVault URIs representing monitored certificates, e.g.: "https://myvault.vault.azure.net/secrets/mycertificate"
        },
        "authenticationSettings": {
                "msiEndpoint":  <Optional MSI endpoint e.g.: "http://169.254.169.254/metadata/identity">,
                "msiClientId":  <Optional MSI identity e.g.: "c7373ae5-91c2-4165-8ab6-7381d6e75619">
        }
       }
      }
    }
```

> [!NOTE]
> Las direcciones URL de los certificados observados deben tener el formato `https://myVaultName.vault.azure.net/secrets/myCertName`.
> 
> Esto se debe a que la ruta de acceso `/secrets` devuelve el certificado completo, incluida la clave privada, mientras que la ruta de acceso `/certificates` no. Se puede encontrar más información sobre los certificados aquí: [Certificados de Key Vault](../../key-vault/general/about-keys-secrets-certificates.md)

> [!NOTE]
> La propiedad "authenticationSettings" es opcional para los escenarios en los que la máquina virtual tiene varias identidades asignadas.
> Permite especificar la identidad que se usará para la autenticación en Key Vault.


### <a name="property-values"></a>Valores de propiedad

| Nombre | Valor / ejemplo | Tipo de datos |
| ---- | ---- | ---- |
| apiVersion | 2019-07-01 | date |
| publisher | Microsoft.Azure.KeyVault | string |
| type | KeyVaultForLinux | string |
| typeHandlerVersion | 1.0 | int |
| pollingIntervalInS | 3600 | string |
| certificateStoreName | Se omite en Linux. | string |
| linkOnRenewal | false | boolean |
| certificateStoreLocation  | /var/lib/waagent/Microsoft.Azure.KeyVault | string |
| requiredInitialSync | true | boolean |
| observedCertificates  | ["https://myvault.vault.azure.net/secrets/mycertificate"] | matriz de cadenas
| msiEndpoint | http://169.254.169.254/metadata/identity | string |
| msiClientId | c7373ae5-91c2-4165-8ab6-7381d6e75619 | string |


## <a name="template-deployment"></a>Implementación de plantilla

Las extensiones de VM de Azure pueden implementarse con plantillas de Azure Resource Manager. Las plantillas son ideales al implementar una o varias máquinas virtuales que requieren la actualización de los certificados tras la implementación. La extensión se puede implementar en máquinas virtuales individuales o en conjuntos de escalado de máquinas virtuales. El esquema y la configuración son comunes para ambos tipos de plantilla. 

La configuración de JSON para una extensión de máquina virtual debe estar anidada dentro del fragmento de recursos de máquina virtual de la plantilla, en concreto en el objeto `"resources": []` de la plantilla de máquina virtual y, en el caso de un conjunto de escalado de máquinas virtuales, en el objeto `"virtualMachineProfile":"extensionProfile":{"extensions" :[]`.

```json
    {
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "name": "KeyVaultForLinux",
      "apiVersion": "2019-07-01",
      "location": "<location>",
      "dependsOn": [
          "[concat('Microsoft.Compute/virtualMachines/', <vmName>)]"
      ],
      "properties": {
      "publisher": "Microsoft.Azure.KeyVault",
      "type": "KeyVaultForLinux",
      "typeHandlerVersion": "1.0",
      "autoUpgradeMinorVersion": true,
      "settings": {
          "secretsManagementSettings": {
          "pollingIntervalInS": <polling interval in seconds, e.g. "3600">,
          "certificateStoreName": <ingnored on linux>,
          "certificateStoreLocation": <disk path where certificate is stored, default: "/var/lib/waagent/Microsoft.Azure.KeyVault">,
          "observedCertificates": <list of KeyVault URIs representing monitored certificates, e.g.: "https://myvault.vault.azure.net/secrets/mycertificate"
        }      
      }
      }
    }
```


## <a name="azure-powershell-deployment"></a>Implementación de Azure PowerShell

Azure PowerShell puede usarse para implementar la extensión de máquina virtual de Key Vault en una máquina virtual o un conjunto de escalado de máquinas virtuales. 

* Para implementar la extensión en una máquina virtual:
    
    ```powershell
        # Build settings
        $settings = '{"secretsManagementSettings": 
        { "pollingIntervalInS": "' + <pollingInterval> + 
        '", "certificateStoreName": "' + <certStoreName> + 
        '", "certificateStoreLocation": "' + <certStoreLoc> + 
        '", "observedCertificates": ["' + <observedCerts> + '"] } }'
        $extName =  "KeyVaultForLinux"
        $extPublisher = "Microsoft.Azure.KeyVault"
        $extType = "KeyVaultForLinux"
       
    
        # Start the deployment
        Set-AzVmExtension -TypeHandlerVersion "1.0" -ResourceGroupName <ResourceGroupName> -Location <Location> -VMName <VMName> -Name $extName -Publisher $extPublisher -Type $extType -SettingString $settings
    
    ```

* Para implementar la extensión en un conjunto de escalado de máquinas virtuales:

    ```powershell
    
        # Build settings
        $settings = '{"secretsManagementSettings": 
        { "pollingIntervalInS": "' + <pollingInterval> + 
        '", "certificateStoreName": "' + <certStoreName> + 
        '", "certificateStoreLocation": "' + <certStoreLoc> + 
        '", "observedCertificates": ["' + <observedCerts> + '"] } }'
        $extName = "KeyVaultForLinux"
        $extPublisher = "Microsoft.Azure.KeyVault"
        $extType = "KeyVaultForLinux"
        
        # Add Extension to VMSS
        $vmss = Get-AzVmss -ResourceGroupName <ResourceGroupName> -VMScaleSetName <VmssName>
        Add-AzVmssExtension -VirtualMachineScaleSet $vmss  -Name $extName -Publisher $extPublisher -Type $extType -TypeHandlerVersion "1.0" -Setting $settings

        # Start the deployment
        Update-AzVmss -ResourceGroupName <ResourceGroupName> -VMScaleSetName <VmssName> -VirtualMachineScaleSet $vmss 
    
    ```

## <a name="azure-cli-deployment"></a>Implementación de la CLI de Azure

La CLI de Azure puede usarse para implementar la extensión de máquina virtual de Key Vault en una máquina virtual o un conjunto de escalado de máquinas virtuales. 
 
* Para implementar la extensión en una máquina virtual:
    
    ```azurecli
       # Start the deployment
         az vm extension set -n "KeyVaultForLinux" `
         --publisher Microsoft.Azure.KeyVault `
         -g "<resourcegroup>" `
         --vm-name "<vmName>" `
         --settings '{\"secretsManagementSettings\": { \"pollingIntervalInS\": \"<pollingInterval>\", \"certificateStoreName\": \"<certStoreName>\", \"certificateStoreLocation\": \"<certStoreLoc>\", \"observedCertificates\": [\ <observedCerts>\"] }}'
    ```

* Para implementar la extensión en un conjunto de escalado de máquinas virtuales:

   ```azurecli
        # Start the deployment
        az vmss extension set -n "KeyVaultForLinux" `
        --publisher Microsoft.Azure.KeyVault `
        -g "<resourcegroup>" `
        --vm-name "<vmName>" `
        --settings '{\"secretsManagementSettings\": { \"pollingIntervalInS\": \"<pollingInterval>\", \"certificateStoreName\": \"<certStoreName>\", \"certificateStoreLocation\": \"<certStoreLoc>\", \"observedCertificates\": [\ <observedCerts>\"] }}'
    ```

Tenga en cuenta las restricciones y los requisitos siguientes:
- Restricciones de Key Vault:
  - Debe existir en el momento de la implementación. 
  - La directiva de acceso de Key Vault está establecida para la identidad de VM/VMSS mediante una identidad administrada. Consulte [Autenticación de Key Vault con una identidad administrada](../../key-vault/general/managed-identity.md).


## <a name="troubleshoot-and-support"></a>Solución de problemas y asistencia

### <a name="troubleshoot"></a>Solución de problemas

Los datos sobre el estado de las implementaciones de extensiones pueden recuperarse desde Azure Portal y mediante Azure PowerShell. Para ver el estado de implementación de las extensiones de una máquina virtual determinada, ejecute el comando siguiente con Azure PowerShell.

## <a name="azure-powershell"></a>Azure PowerShell
```powershell
Get-AzVMExtension -VMName <vmName> -ResourceGroupname <resource group name>
```

## <a name="azure-cli"></a>Azure CLI
```azurecli
 az vm get-instance-view --resource-group <resource group name> --name  <vmName> --query "instanceView.extensions"
```
### <a name="logs-and-configuration"></a>Registros y configuración

```
/var/log/waagent.log
/var/log/azure/Microsoft.Azure.KeyVault.KeyVaultForLinux/*
/var/lib/waagent/Microsoft.Azure.KeyVault.KeyVaultForLinux-<most recent version>/config/*
```

### <a name="support"></a>Soporte técnico

Si necesita más ayuda con cualquier aspecto de este artículo, puede ponerse en contacto con los expertos de Azure en los [foros de MSDN Azure o Stack Overflow](https://azure.microsoft.com/support/forums/). Como alternativa, puede registrar un incidente de soporte técnico de Azure. Vaya al [sitio de soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione Obtener soporte. Para obtener información sobre el uso del soporte técnico, lea las [Preguntas más frecuentes de soporte técnico de Microsoft Azure](https://azure.microsoft.com/support/faq/).
