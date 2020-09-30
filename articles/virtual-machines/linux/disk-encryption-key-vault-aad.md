---
title: Creación y configuración de un almacén de claves para Azure Disk Encryption con Azure AD (versión anterior)
description: En este artículo se proporcionan los requisitos previos para usar Microsoft Azure Disk Encryption para máquinas virtuales Linux.
author: msmbaldwin
ms.service: virtual-machines-linux
ms.subservice: security
ms.topic: conceptual
ms.author: mbaldwin
ms.date: 03/15/2019
ms.custom: seodec18
ms.openlocfilehash: 4b533fa23d3c128b5f9f75737fb88d39aec94905
ms.sourcegitcommit: e69bb334ea7e81d49530ebd6c2d3a3a8fa9775c9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88950075"
---
# <a name="creating-and-configuring-a-key-vault-for-azure-disk-encryption-with-azure-ad-previous-release-for-linux-vms"></a>Creación y configuración de un almacén de claves para Azure Disk Encryption con Azure AD (versión anterior) para máquinas virtuales Linux

**La nueva versión de Azure Disk Encryption elimina la necesidad de proporcionar un parámetro de aplicación de Azure AD para habilitar el cifrado de disco de máquina virtual. Con la nueva versión, ya no es necesario proporcionar credenciales de Azure AD durante el paso de habilitar el cifrado. Todas las nuevas máquinas virtuales deben estar cifradas sin los parámetros de aplicación de Azure AD con la nueva versión. Para ver las instrucciones necesarias para habilitar el cifrado de disco de máquina virtual con la nueva versión, consulte [Azure Disk Encryption](disk-encryption-overview.md). Las máquinas virtuales que ya se han cifrado con parámetros de aplicación de Azure AD se siguen admitiendo y se deben seguir manteniendo con la sintaxis de AAD.**

Azure Disk Encryption usa Azure Key Vault para controlar y administrar las claves y los secretos de cifrado de discos.  Para más información sobre los almacenes de claves, consulte [Introducción a Azure Key Vault](../../key-vault/general/overview.md) y [Protección de un almacén de claves](../../key-vault/general/secure-your-key-vault.md). 

Para crear y configurar un almacén de claves para Azure Disk Encryption con Azure AD (versión anterior) debe seguir estos tres pasos:

1. Cree un almacén de claves. 
2. Configure una aplicación de Azure AD y una entidad de servicio.
3. Establezca la directiva de acceso del almacén de claves para la aplicación de Azure AD.
4. Configure las directivas de acceso avanzado del almacén de claves.
 
Si lo desea, también puede generar o importar una clave de cifrado de claves (KEK).

En el artículo [Creación y configuración de un almacén de claves para Azure Disk Encryption](disk-encryption-key-vault.md) encontrará los pasos necesarios para [instalar las herramientas y conectarse a Azure](disk-encryption-key-vault.md#install-tools-and-connect-to-azure).

> [!Note]
> Los pasos que se describen en este artículo se automatizan en el [script de la CLI de requisitos previos de Azure Disk Encryption](https://github.com/ejarvi/ade-cli-getting-started) y en el [script de PowerShell de requisitos previos de Azure Disk Encryption](https://github.com/Azure/azure-powershell/tree/master/src/Compute/Compute/Extension/AzureDiskEncryption/Scripts).


## <a name="create-a-key-vault"></a>Creación de un Almacén de claves 
Azure Disk Encryption se integra con [Azure Key Vault](https://azure.microsoft.com/documentation/services/key-vault/) para controlar y administrar los secretos y las claves de cifrado de los discos en la suscripción de Key Vault. Puede crear un almacén de claves o usar uno existente para Azure Disk Encryption. Para más información sobre los almacenes de claves, consulte [Introducción a Azure Key Vault](../../key-vault/general/overview.md) y [Protección de un almacén de claves](../../key-vault/general/secure-your-key-vault.md). Puede usar una plantilla de Resource Manager, Azure PowerShell o la CLI de Azure para crear un almacén de claves. 


>[!WARNING]
>Con el fin de garantizar que los secretos de cifrado no traspasen los límites regionales, Azure Disk Encryption necesita que Key Vault y las máquinas virtuales estén ubicadas conjuntamente en la misma región. Cree y use una instancia de Key Vault que se encuentre en la misma región que la máquina virtual que se va a cifrar. 


### <a name="create-a-key-vault-with-powershell"></a><a name="bkmk_KVPSH"></a> Crear un almacén de claves con PowerShell

Puede crear un almacén de claves con Azure PowerShell mediante el cmdlet [New-AzKeyVault](/powershell/module/az.keyvault/New-azKeyVault). Para otros cmdlets de Key Vault, consulte [Az.KeyVault](/powershell/module/az.keyvault/). 

1. Cree un nuevo grupo de recursos, si fuera necesario, con [New-AzResourceGroup](/powershell/module/az.Resources/New-azResourceGroup).  Para enumerar las ubicaciones de los centro de datos, use [Get-AzLocation](/powershell/module/az.resources/get-azlocation). 
     
     ```azurepowershell-interactive
     # Get-AzLocation 
     New-AzResourceGroup –Name 'MyKeyVaultResourceGroup' –Location 'East US'
     ```

1. Cree un nuevo almacén de claves con [New-AzKeyVault](/powershell/module/az.keyvault/New-azKeyVault).
    
      ```azurepowershell-interactive
     New-AzKeyVault -VaultName 'MySecureVault' -ResourceGroupName 'MyKeyVaultResourceGroup' -Location 'East US'
     ```

4. Tenga en cuenta el **nombre del almacén**, el **nombre del grupo de recursos**, el **id. de recurso**, el **URI de almacén** y el **id. de objeto** que se devuelven para su uso posterior al cifrar los discos. 


### <a name="create-a-key-vault-with-azure-cli"></a><a name="bkmk_KVCLI"></a> Crear un almacén de claves con la CLI de Azure
Puede administrar el almacén de claves con la CLI de Azure mediante el comando [az keyvault](/cli/azure/keyvault#commands). Para crear un almacén de claves, use [crear az keyvault](/cli/azure/keyvault#az-keyvault-create).

1. Cree un nuevo grupo de recursos, si es necesario, con [crear grupo az](/cli/azure/group#az-group-create). Para enumerar las ubicaciones, use [az account list-locations](/cli/azure/account#az-account-list). 
     
     ```azurecli-interactive
     # To list locations: az account list-locations --output table
     az group create -n "MyKeyVaultResourceGroup" -l "East US"
     ```

3. Cree un nuevo almacén de claves con [crear az keyvault](/cli/azure/keyvault#az-keyvault-create).
    
     ```azurecli-interactive
     az keyvault create --name "MySecureVault" --resource-group "MyKeyVaultResourceGroup" --location "East US"
     ```

4. Tenga en cuenta el **nombre del almacén** (nombre), el **nombre del grupo de recursos**, el **id. de recurso** (identificador), el **URI de almacén** y el **id. de objeto** que se devuelven para su uso posterior. 

### <a name="create-a-key-vault-with-a-resource-manager-template"></a><a name="bkmk_KVRM"></a> Crear un almacén de claves con una plantilla de Resource Manager

Puede crear un almacén de claves con la [plantilla de Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/101-key-vault-create).

1. En la plantilla de inicio rápido de Azure, haga clic en **Deploy to Azure** (Implementar en Azure).
2. Seleccione la suscripción, el grupo de recursos, la ubicación del grupo de recursos, el nombre del almacén de claves, el id. de objeto, los términos legales y el contrato, y luego haga clic en **Comprar**. 


## <a name="set-up-an-azure-ad-app-and-service-principal"></a><a name="bkmk_ADapp"></a> Configurar una aplicación de Azure AD y una entidad de servicio 
Si es preciso habilitar el cifrado en una máquina virtual en ejecución en Azure, Azure Disk Encryption genera y escribe las claves de cifrado en su almacén de claves. La administración de claves de cifrado en el almacén de claves necesita la autenticación de Azure AD. Para ello, cree una aplicación de Azure AD. Para realizar la autenticación, se pueden usar la autenticación basada en secreto de cliente o la [autenticación de Azure AD basada en certificado del cliente](../../active-directory/authentication/active-directory-certificate-based-authentication-get-started.md).


### <a name="set-up-an-azure-ad-app-and-service-principal-with-azure-powershell"></a><a name="bkmk_ADappPSH"></a> Configurar una aplicación de Azure AD y una entidad de servicio con Azure PowerShell 
Para ejecutar los siguientes comandos, obtenga y use el [módulo de PowerShell de Azure AD](/powershell/azure/active-directory/install-adv2). 

1. Use el cmdlet [New-AzADApplication](/powershell/module/az.resources/new-azadapplication) de PowerShell para crear una aplicación de Azure AD. MyApplicationHomePage y MyApplicationUri pueden tener los valores que desee.

     ```azurepowershell
     $aadClientSecret = "My AAD client secret"
     $aadClientSecretSec = ConvertTo-SecureString -String $aadClientSecret -AsPlainText -Force
     $azureAdApplication = New-AzADApplication -DisplayName "My Application Display Name" -HomePage "https://MyApplicationHomePage" -IdentifierUris "https://MyApplicationUri" -Password $aadClientSecretSec
     $servicePrincipal = New-AzADServicePrincipal –ApplicationId $azureAdApplication.ApplicationId
     ```

3. $azureAdApplication.ApplicationId es el identificador de cliente de Azure AD y $aadClientSecret es el secreto de cliente que usará posteriormente para habilitar Azure Disk Encryption. Proteja el secreto del cliente de Azure AD apropiadamente. Ejecutar `$azureAdApplication.ApplicationId` mostrará el valor de ApplicationID.


### <a name="set-up-an-azure-ad-app-and-service-principal-with-azure-cli"></a><a name="bkmk_ADappCLI"></a> Configurar una aplicación de Azure AD y una entidad de servicio con la CLI de Azure

Puede administrar las entidades de servicio con la CLI de Azure mediante el comando [az ad sp](/cli/azure/ad/sp). Para más información, consulte [Creación de una entidad de servicio de Azure](/cli/azure/create-an-azure-service-principal-azure-cli).

1. Cree una nueva entidad de servicio.
     
     ```azurecli-interactive
     az ad sp create-for-rbac --name "ServicePrincipalName" --password "My-AAD-client-secret" --skip-assignment 
     ```
3.  El valor de appId devuelto es el identificador de cliente de Azure AD que se usa en otros comandos. También es el SPN que se va a usar para az keyvault set-policy. La contraseña es el secreto de cliente que se debe usar posteriormente para habilitar Azure Disk Encryption. Proteja el secreto del cliente de Azure AD apropiadamente.
 
### <a name="set-up-an-azure-ad-app-and-service-principal-though-the-azure-portal"></a><a name="bkmk_ADappRM"></a> Configurar una aplicación de Azure AD y una entidad de servicioa través de Azure Portal
Siga los pasos que aparecen en el artículo [Uso del portal para crear una aplicación de Azure Active Directory y una entidad de servicio con acceso a los recursos](../../active-directory/develop/howto-create-service-principal-portal.md) para crear una aplicación de Azure AD. Cada paso que se enumera a continuación lo llevará directamente a la sección del artículo que hay que completar. 

1. [Comprobar los permisos requeridos](../../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app)
2. [Crear una aplicación de Azure Active Directory](../../active-directory/develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal) 
     - Puede utilizar cualquier nombre y dirección URL de inicio de sesión que desee al crear la aplicación.
3. [Obtener el identificador de aplicación y la clave de autenticación](../../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in) 
     - La clave de autenticación es el secreto de cliente y se usa como el objeto AadClientSecret para Set-AzVMDiskEncryptionExtension. 
        - La aplicación usa la clave de autenticación como una credencial para iniciar sesión en Azure AD. En Azure Portal, este secreto se denomina "claves", pero no tiene relación con los almacenes de claves. Proteja adecuadamente este secreto. 
     - El identificador de aplicación se usará más adelante como objeto AadClientId para Set-AzVMDiskEncryptionExtension y como objeto ServicePrincipalName para Set-AzKeyVaultAccessPolicy. 

## <a name="set-the-key-vault-access-policy-for-the-azure-ad-app"></a><a name="bkmk_KVAP"></a> Establecer la directiva de acceso del almacén de claves para la aplicación de Azure AD
Para escribir los secretos de cifrado en un almacén de claves especificado, Azure Disk Encryption necesita el identificador de cliente y el secreto de cliente de la aplicación de Azure Active Directory que tenga permisos para escribir secretos en Key Vault. 

> [!NOTE]
> Azure Disk Encryption requiere que configure las siguientes directivas de acceso a la aplicación de cliente de Azure AD: los permisos _WrapKey_ y _Set_.

### <a name="set-the-key-vault-access-policy-for-the-azure-ad-app-with-azure-powershell"></a><a name="bkmk_KVAPPSH"></a> Establecer la directiva de acceso del almacén de claves para la aplicación de Azure AD con Azure PowerShell
La aplicación de Azure AD necesita derechos de acceso a las claves o secretos del almacén. Use el cmdlet [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy) para conceder permisos a la aplicación, con el identificador de cliente (que se generó cuando se registró la aplicación) como valor del parámetro _–ServicePrincipalName_. Para obtener más información, consulte la entrada de blog [Azure Key Vault - Step by Step](/archive/blogs/kv/azure-key-vault-step-by-step) (Azure Key Vault - Paso a paso). 

1. Establezca la directiva de acceso del almacén de claves para la aplicación de AD con PowerShell.

     ```azurepowershell
     $keyVaultName = 'MySecureVault'
     $aadClientID = 'MyAadAppClientID'
     $KVRGname = 'MyKeyVaultResourceGroup'
     Set-AzKeyVaultAccessPolicy -VaultName $keyVaultName -ServicePrincipalName $aadClientID -PermissionsToKeys 'WrapKey' -PermissionsToSecrets 'Set' -ResourceGroupName $KVRGname
     ```

### <a name="set-the-key-vault-access-policy-for-the-azure-ad-app-with-azure-cli"></a><a name="bkmk_KVAPCLI"></a> Establecer la directiva de acceso del almacén de claves para la aplicación de Azure AD con la CLI de Azure
Use [az keyvault set-policy](/cli/azure/keyvault#az-keyvault-set-policy) para establecer la directiva de acceso. Para más información, consulte [Administración de Key Vault mediante CLI 2.0](../../key-vault/general/manage-with-cli2.md#authorizing-an-application-to-use-a-key-or-secret).

Conceda a la entidad de servicio que creó mediante la CLI de Azure accesp para obtener los secretos y encapsular las claves con el comando siguiente:

```azurecli-interactive
az keyvault set-policy --name "MySecureVault" --spn "<spn created with CLI/the Azure AD ClientID>" --key-permissions wrapKey --secret-permissions set
```

### <a name="set-the-key-vault-access-policy-for-the-azure-ad-app-with-the-portal"></a><a name="bkmk_KVAPRM"></a> Establecer la directiva de acceso del almacén de claves para la aplicación de Azure AD con el portal

1. Abra el grupo de recursos con el almacén de claves.
2. Seleccione el almacén de claves, vaya a **Directivas de acceso** y haga clic en **Agregar nuevo**.
3. En **Select principal** (Seleccionar la entidad de seguridad), busque la aplicación de Azure AD que creó y selecciónela. 
4. Para **Permisos de claves**, active **Encapsular clave** en **Operaciones criptográficas**.
5. Para **Permisos de secretos**, active **Establecer** en **Operaciones de administración de secretos**.
6. Haga clic en **Aceptar** para guardar la directiva de acceso. 

![Operaciones criptográficas de Azure Key Vault: Encapsular clave](./media/disk-encryption/keyvault-portal-fig3.png)

![Permisos de secretos de Azure Key Vault: Establecer](./media/disk-encryption/keyvault-portal-fig3b.png)

## <a name="set-key-vault-advanced-access-policies"></a><a name="bkmk_KVper"></a> Configurar las directivas de acceso avanzado del almacén de claves
La plataforma Azure necesita acceso a las claves de cifrado o secretos del almacén de claves para ponerlos a disposición de la máquina virtual para el proceso de arranque y descifrado de los volúmenes. Habilite el cifrado de disco en el almacén de claves o se producirá un error en las implementaciones.  

### <a name="set-key-vault-advanced-access-policies-with-azure-powershell"></a><a name="bkmk_KVperPSH"></a> Establecer las directivas de acceso avanzado del almacén de claves con Azure PowerShell
 Use el cmdlet [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy) de PowerShell del almacén de claves para habilitar el cifrado de disco para el almacén de claves.

  - **Habilitar Key Vault para el cifrado de disco:** se requiere EnabledForDiskEncryption para el cifrado de Azure Disk.
      
     ```azurepowershell-interactive 
     Set-AzKeyVaultAccessPolicy -VaultName 'MySecureVault' -ResourceGroupName 'MyKeyVaultResourceGroup' -EnabledForDiskEncryption
     ```

  - **Habilitar Key Vault para la implementación, si es necesario:** permite que el proveedor de recursos Microsoft.Compute recupere los secretos de este almacén de claves cuando se hace referencia al almacén de claves en la creación de recursos, por ejemplo, cuando se crea una máquina virtual.

     ```azurepowershell-interactive
      Set-AzKeyVaultAccessPolicy -VaultName 'MySecureVault' -ResourceGroupName 'MyKeyVaultResourceGroup' -EnabledForDeployment
     ```

  - **Habilitar Key Vault para la implementación de plantillas, si es necesario:** permite que Azure Resource Manager obtenga los secretos de este almacén de claves cuando se hace referencia al almacén de claves en una implementación de plantilla.

     ```azurepowershell-interactive             
     Set-AzKeyVaultAccessPolicy -VaultName 'MySecureVault' -ResourceGroupName 'MyKeyVaultResourceGroup' -EnabledForTemplateDeployment
     ```

### <a name="set-key-vault-advanced-access-policies-using-the-azure-cli"></a><a name="bkmk_KVperCLI"></a> Establecer directivas de acceso avanzado del almacén de claves mediante la CLI de Azure
Use [az keyvault update](/cli/azure/keyvault#az-keyvault-update) para habilitar el cifrado de disco para el almacén de claves. 

 - **Habilitar Key Vault para el cifrado de disco:** Es necesario Enabled-for-disk-encryption. 

     ```azurecli-interactive
     az keyvault update --name "MySecureVault" --resource-group "MyKeyVaultResourceGroup" --enabled-for-disk-encryption "true"
     ```  

 - **Habilitar Key Vault para la implementación, si es necesario:** permite que Virtual Machines recupere certificados almacenados como secretos del almacén.
     ```azurecli-interactive
     az keyvault update --name "MySecureVault" --resource-group "MyKeyVaultResourceGroup" --enabled-for-deployment "true"
     ``` 

 - **Habilitar Key Vault para la implementación de plantillas, si es necesario:** permite que Resource Manager recupere secretos del almacén.
     ```azurecli-interactive  
     az keyvault update --name "MySecureVault" --resource-group "MyKeyVaultResourceGroup" --enabled-for-template-deployment "true"
     ```


### <a name="set-key-vault-advanced-access-policies-through-the-azure-portal"></a><a name="bkmk_KVperrm"></a> Establecer directivas de acceso avanzado del almacén de claves a través de Azure Portal

1. Seleccione el almacén de claves, vaya a **Directivas de acceso** y **Haga clic para mostrar las directivas de acceso avanzado**.
2. Active la casilla etiquetada **Habilitar el acceso a Azure Disk Encryption para el cifrado de volúmenes**.
3. Seleccione **Habilitar el acceso a Azure Virtual Machines para la implementación** o **Habilitar el acceso a Azure Resource Manager para la implementación de plantillas**, si es necesario. 
4. Haga clic en **Save**(Guardar).

![Directivas de acceso avanzado de Azure Key Vault](./media/disk-encryption/keyvault-portal-fig4.png)


## <a name="set-up-a-key-encryption-key-optional"></a><a name="bkmk_KEK"></a> Configurar una clave de cifrado de claves (opcional)
Si desea usar una clave de cifrado de claves (KEK) para una brindar una capa adicional de seguridad para las claves de cifrado, agregue una KEK a su almacén de claves. Use el cmdlet [Add-AzKeyVaultKey](/powershell/module/az.keyvault/add-azkeyvaultkey) para crear una clave de cifrado de claves en el almacén de claves. También puede importar una KEK en el HSM de administración de claves local. Para más información, consulte la [documentación de Key Vault](../../key-vault/keys/hsm-protected-keys.md). Cuando se especifica una clave de cifrado de claves, Azure Disk Encryption usa esa clave para encapsular los secretos de cifrado antes de escribirlos en Key Vault. 

* Al generar las claves, use un tipo de clave RSA. Azure Disk Encryption no admite aún el uso de claves de curva elíptica.

* El secreto del almacén de claves y las direcciones URL de KEK deben tener versiones. Azure exige esta restricción del control de versiones. Para ver direcciones URL de KEK y de secretos válidas, vea los ejemplos siguientes:

  * Ejemplo de una dirección URL secreta válida: *https://contosovault.vault.azure.net/secrets/EncryptionSecretWithKek/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx*
  * Ejemplo de una dirección URL de KEK válida: *https://contosovault.vault.azure.net/keys/diskencryptionkek/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx*

* Azure Disk Encryption no admite la especificación de números de puerto como parte de los secretos del almacén de claves y las direcciones URL de KEK. Para ver ejemplos de direcciones URL de almacén de claves admitidas y no admitidas, consulte los siguientes:

  * Dirección URL de almacén de claves inaceptable *https://contosovault.vault.azure.net:443/secrets/contososecret/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx*
  * Dirección URL de almacén de claves aceptable: *https://contosovault.vault.azure.net/secrets/contososecret/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx*

### <a name="set-up-a-key-encryption-key-with-azure-powershell"></a><a name="bkmk_KEKPSH"></a> Configurar una clave de cifrado de claves con Azure PowerShell 
Antes de usar el script de PowerShell, debe estar familiarizado con los requisitos previos de Azure Disk Encryption para entender los pasos en el script. Es posible que el script de ejemplo requiera cambios para adaptarse al entorno en cuestión. Este script crea todos los requisitos previos de Azure Disk Encryption y cifra una máquina virtual IaaS existente, encapsulando la clave de cifrado de disco mediante el uso de una clave de cifrado de claves. 

 ```powershell
 # Step 1: Create a new resource group and key vault in the same location.
     # Fill in 'MyLocation', 'MyKeyVaultResourceGroup', and 'MySecureVault' with your values.
     # Use Get-AzLocation to get available locations and use the DisplayName.
     # To use an existing resource group, comment out the line for New-AzResourceGroup
     
     $Loc = 'MyLocation';
     $KVRGname = 'MyKeyVaultResourceGroup';
     $KeyVaultName = 'MySecureVault'; 
     New-AzResourceGroup –Name  $KVRGname –Location $Loc;
     New-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname -Location $Loc;
     $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName  $KVRGname;
     $KeyVaultResourceId = (Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName  $KVRGname).ResourceId;
     $diskEncryptionKeyVaultUrl = (Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName  $KVRGname).VaultUri;
     
 # Step 2: Create the AD application and service principal.
     # Fill in 'MyAADClientSecret', "<My Application Display Name>", "<https://MyApplicationHomePage>", and "<https://MyApplicationUri>" with your values.
     # MyApplicationHomePage and the MyApplicationUri can be any values you wish.
     
     $aadClientSecret =  'MyAADClientSecret';
     $aadClientSecretSec = ConvertTo-SecureString -String $aadClientSecret -AsPlainText -Force;
     $azureAdApplication = New-AzADApplication -DisplayName "<My Application Display Name>" -HomePage "<https://MyApplicationHomePage>" -IdentifierUris "<https://MyApplicationUri>" -Password $aadClientSecretSec
     $servicePrincipal = New-AzADServicePrincipal –ApplicationId $azureAdApplication.ApplicationId;
     $aadClientID = $azureAdApplication.ApplicationId;
     
 #Step 3: Enable the vault for disk encryption and set the access policy for the Azure AD application.
     
     Set-AzKeyVaultAccessPolicy -VaultName $KeyVaultName -ResourceGroupName $KVRGname -EnabledForDiskEncryption;
     Set-AzKeyVaultAccessPolicy -VaultName $keyVaultName -ServicePrincipalName $aadClientID -PermissionsToKeys 'WrapKey' -PermissionsToSecrets 'Set' -ResourceGroupName  $KVRGname;
     
 #Step 4: Create a new key in the key vault with the Add-AzKeyVaultKey cmdlet.
     # Fill in 'MyKeyEncryptionKey' with your value.
     
     $keyEncryptionKeyName = 'MyKeyEncryptionKey';
     Add-AzKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName -Destination 'Software';
     $keyEncryptionKeyUrl = (Get-AzKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;
     
 #Step 5: Encrypt the disks of an existing IaaS VM
     # Fill in 'MySecureVM' and 'MyVirtualMachineResourceGroup' with your values. 
     
     $VMName = 'MySecureVM';
      $VMRGName = 'MyVirtualMachineResourceGroup';
     Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId;
```

## <a name="certificate-based-authentication-optional"></a><a name="bkmk_Cert"></a> Autenticación basada en certificados (opcional)
Si desea usar la autenticación de certificado, puede cargar uno en el almacén de claves e implementarlo en el cliente. Antes de usar el script de PowerShell, debe estar familiarizado con los requisitos previos de Azure Disk Encryption para entender los pasos en el script. Es posible que el script de ejemplo requiera cambios para adaptarse al entorno en cuestión.

     
 ```powershell

 # Fill in "MyKeyVaultResourceGroup", "MySecureVault", and 'MyLocation' ('My location' only if needed)

   $KVRGname = 'MyKeyVaultResourceGroup'
   $KeyVaultName= 'MySecureVault'

   # Create a key vault and set enabledForDiskEncryption property on it. 
   # Comment out the next three lines if you already have an existing key vault enabled for encryption. No need to set 'My location' in this case.

   $Loc = 'MyLocation'
   New-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname -Location $Loc
   Set-AzKeyVaultAccessPolicy -VaultName $KeyVaultName -ResourceGroupName $KVRGname -EnabledForDiskEncryption

   #Setting some variables with the key vault information 
   $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname
   $DiskEncryptionKeyVaultUrl = $KeyVault.VaultUri
   $KeyVaultResourceId = $KeyVault.ResourceId

   # Create the Azure AD application and associate the certificate with it. 
   # Fill in "C:\certificates\mycert.pfx", "Password", "<My Application Display Name>", "<https://MyApplicationHomePage>", and "<https://MyApplicationUri>" with your values.
   # MyApplicationHomePage and the MyApplicationUri can be any values you wish

   $CertPath = "C:\certificates\mycert.pfx"
   $CertPassword = "Password"
   $Cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($CertPath, $CertPassword)
   $CertValue = [System.Convert]::ToBase64String($cert.GetRawCertData())

   $AzureAdApplication = New-AzADApplication -DisplayName "<My Application Display Name>" -HomePage "<https://MyApplicationHomePage>" -IdentifierUris "<https://MyApplicationUri>" -CertValue $CertValue 
   $ServicePrincipal = New-AzADServicePrincipal -ApplicationId $AzureAdApplication.ApplicationId

   $AADClientID = $AzureAdApplication.ApplicationId
   $aadClientCertThumbprint= $cert.Thumbprint

   Set-AzKeyVaultAccessPolicy -VaultName $keyVaultName -ServicePrincipalName $aadClientID -PermissionsToKeys 'WrapKey' -PermissionsToSecrets 'Set' -ResourceGroupName $KVRGname
   
   # Upload the pfx file to the key vault. 
   # Fill in "MyAADCert".  

   $KeyVaultSecretName = "MyAADCert"
   $FileContentBytes = get-content $CertPath -Encoding Byte
   $FileContentEncoded = [System.Convert]::ToBase64String($fileContentBytes)
           $JSONObject = @"
           { 
               "data" : "$filecontentencoded", 
               "dataType" : "pfx", 
               "password" : "$CertPassword" 
           } 
"@

   $JSONObjectBytes = [System.Text.Encoding]::UTF8.GetBytes($jsonObject)
   $JSONEncoded = [System.Convert]::ToBase64String($jsonObjectBytes)

   #Set the secret and set the key vault policy for -EnabledForDeployment

   $Secret = ConvertTo-SecureString -String $JSONEncoded -AsPlainText -Force
   Set-AzKeyVaultSecret -VaultName $KeyVaultName -Name $KeyVaultSecretName -SecretValue $Secret
   Set-AzKeyVaultAccessPolicy -VaultName $KeyVaultName -ResourceGroupName $KVRGname -EnabledForDeployment

   # Deploy the certificate to the VM
   # Fill in 'MySecureVM' and 'MyVirtualMachineResourceGroup' with your values.

   $VMName = 'MySecureVM'
   $VMRGName = 'MyVirtualMachineResourceGroup'
   $CertUrl = (Get-AzKeyVaultSecret -VaultName $KeyVaultName -Name $KeyVaultSecretName).Id
   $SourceVaultId = (Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGName).ResourceId
   $VM = Get-AzVM -ResourceGroupName $VMRGName -Name $VMName 
   $VM = Add-AzVMSecret -VM $VM -SourceVaultId $SourceVaultId -CertificateStore "My" -CertificateUrl $CertUrl
   Update-AzVM -VM $VM -ResourceGroupName $VMRGName 

   #Enable encryption on the VM using Azure AD client ID and the client certificate thumbprint

   Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $VMName -AadClientID $AADClientID -AadClientCertThumbprint $AADClientCertThumbprint -DiskEncryptionKeyVaultUrl $DiskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId
 ```

## <a name="certificate-based-authentication-and-a-kek-optional"></a><a name="bkmk_CertKEK"></a> Autenticación basada en certificados y una KEK (opcional)

Si desea usar la autenticación de certificado y encapsular la clave de cifrado con una KEK, puede usar el siguiente script como ejemplo. Antes de usar el script de PowerShell, debe estar familiarizado con todos los requisitos previos de Azure Disk Encryption anteriores para conocer los pasos en el script. Es posible que el script de ejemplo requiera cambios para adaptarse al entorno en cuestión.

> [!IMPORTANT]
> Actualmente no se admite la autenticación basada en certificado de Azure AD en máquinas virtuales con Linux.



     
 ```powershell
# Fill in 'MyKeyVaultResourceGroup', 'MySecureVault', and 'MyLocation' (if needed)

   $KVRGname = 'MyKeyVaultResourceGroup'
   $KeyVaultName= 'MySecureVault'

   # Create a key vault and set enabledForDiskEncryption property on it. 
   # Comment out the next three lines if you already have an existing key vault enabled for encryption.

   $Loc = 'MyLocation'
   New-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname -Location $Loc
   Set-AzKeyVaultAccessPolicy -VaultName $KeyVaultName -ResourceGroupName $KVRGname -EnabledForDiskEncryption

   # Create the Azure AD application and associate the certificate with it.  
   # Fill in "C:\certificates\mycert.pfx", "Password", "<My Application Display Name>", "<https://MyApplicationHomePage>", and "<https://MyApplicationUri>" with your values.
   # MyApplicationHomePage and the MyApplicationUri can be any values you wish

   $CertPath = "C:\certificates\mycert.pfx"
   $CertPassword = "Password"
   $Cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($CertPath, $CertPassword)
   $CertValue = [System.Convert]::ToBase64String($cert.GetRawCertData())

   $AzureAdApplication = New-AzADApplication -DisplayName "<My Application Display Name>" -HomePage "<https://MyApplicationHomePage>" -IdentifierUris "<https://MyApplicationUri>" -CertValue $CertValue 
   $ServicePrincipal = New-AzADServicePrincipal -ApplicationId $AzureAdApplication.ApplicationId

   $AADClientID = $AzureAdApplication.ApplicationId
   $aadClientCertThumbprint= $cert.Thumbprint

   ## Give access for setting secrets and wraping keys
   Set-AzKeyVaultAccessPolicy -VaultName $keyVaultName -ServicePrincipalName $aadClientID -PermissionsToKeys 'WrapKey' -PermissionsToSecrets 'Set' -ResourceGroupName $KVRGname

   # Upload the pfx file to the key vault. 
   # Fill in "MyAADCert". 

   $KeyVaultSecretName = "MyAADCert"
   $FileContentBytes = get-content $CertPath -Encoding Byte
   $FileContentEncoded = [System.Convert]::ToBase64String($fileContentBytes)
           $JSONObject = @"
           { 
               "data" : "$filecontentencoded", 
               "dataType" : "pfx", 
               "password" : "$CertPassword" 
           } 
"@

   $JSONObjectBytes = [System.Text.Encoding]::UTF8.GetBytes($jsonObject)
   $JSONEncoded = [System.Convert]::ToBase64String($jsonObjectBytes)

   #Set the secret and set the key vault policy for deployment

   $Secret = ConvertTo-SecureString -String $JSONEncoded -AsPlainText -Force
   Set-AzKeyVaultSecret -VaultName $KeyVaultName -Name $KeyVaultSecretName -SecretValue $Secret
   Set-AzKeyVaultAccessPolicy -VaultName $KeyVaultName -ResourceGroupName $KVRGname -EnabledForDeployment

   #Setting some variables with the key vault information and generating a KEK 
   # FIll in 'KEKName'
   
   $KEKName ='KEKName'
   $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname
   $DiskEncryptionKeyVaultUrl = $KeyVault.VaultUri
   $KeyVaultResourceId = $KeyVault.ResourceId
   $KEK = Add-AzKeyVaultKey -VaultName $KeyVaultName -Name $KEKName -Destination "Software"
   $KeyEncryptionKeyUrl = $KEK.Key.kid



   # Deploy the certificate to the VM
   # Fill in 'MySecureVM' and 'MyVirtualMachineResourceGroup' with your values.

   $VMName = 'MySecureVM';
   $VMRGName = 'MyVirtualMachineResourceGroup';
   $CertUrl = (Get-AzKeyVaultSecret -VaultName $KeyVaultName -Name $KeyVaultSecretName).Id
   $SourceVaultId = (Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGName).ResourceId
   $VM = Get-AzVM -ResourceGroupName $VMRGName -Name $VMName 
   $VM = Add-AzVMSecret -VM $VM -SourceVaultId $SourceVaultId -CertificateStore "My" -CertificateUrl $CertUrl
   Update-AzVM -VM $VM -ResourceGroupName $VMRGName

   #Enable encryption on the VM using Azure AD client ID and the client certificate thumbprint

   Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $VMName -AadClientID $AADClientID -AadClientCertThumbprint $AADClientCertThumbprint -DiskEncryptionKeyVaultUrl $DiskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId
```

 
## <a name="next-steps"></a>Pasos siguientes

[Habilitación de Azure Disk Encryption con Azure AD en máquinas virtuales Linux (versión anterior)](disk-encryption-linux-aad.md)
