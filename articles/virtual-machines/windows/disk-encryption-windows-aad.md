---
title: Azure Disk Encryption con Azure AD para máquinas virtuales Windows (versión anterior)
description: En este artículo se proporcionan instrucciones sobre cómo habilitar Microsoft Azure Disk Encryption para máquinas virtuales IaaS de Windows.
author: msmbaldwin
ms.service: virtual-machines
ms.subservice: security
ms.topic: how-to
ms.author: mbaldwin
ms.date: 03/15/2019
ms.custom: seodec18
ms.openlocfilehash: 0e8ea218aa9c557fb109aee0dba318cfd5f605c7
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87836248"
---
# <a name="azure-disk-encryption-with-azure-ad-for-windows-vms-previous-release"></a>Azure Disk Encryption con Azure AD para máquinas virtuales Windows (versión anterior)

**La nueva versión de Azure Disk Encryption elimina la necesidad de proporcionar un parámetro de aplicación de Azure AD para habilitar el cifrado de disco de máquina virtual. Con la nueva versión, ya no es necesario proporcionar credenciales de Azure AD durante el paso de habilitar el cifrado. Todas las nuevas máquinas virtuales deben estar cifradas sin los parámetros de aplicación de Azure AD con la nueva versión. Para instrucciones sobre cómo habilitar el cifrado de disco de máquina virtual con la nueva versión, consulte [Azure Disk Encryption para máquinas virtuales de Windows](disk-encryption-windows.md). Las máquinas virtuales que ya se han cifrado con parámetros de aplicación de Azure AD se siguen admitiendo y se deben seguir manteniendo con la sintaxis de AAD.**


Puede habilitar muchos escenarios de cifrado de disco, y los pasos pueden variar en función del escenario. En las secciones siguientes se tratan con más detalle los escenarios de máquinas virtuales IaaS de Windows. Para poder usar el cifrado de disco, es necesario satisfacer los [requisitos previos de Azure Disk Encryption](disk-encryption-overview-aad.md). 


>[!IMPORTANT]
> - Es preciso [hacer una instantánea](snapshot-copy-managed-disk.md) o crear una copia de seguridad antes de cifrar los discos. Las copias de seguridad garantizan que, en caso de un error inesperado durante el cifrado, sea posible una opción de recuperación. Las máquinas virtuales con discos administrados requieren una copia de seguridad antes del cifrado. Una vez realizada la copia de seguridad, puede usar el [cmdlet Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension) para cifrar los discos administrados mediante la especificación del parámetro -skipVmBackup. Para más información sobre cómo realizar la copia de seguridad y la restauración de máquinas virtuales cifradas, consulte [Copia de seguridad y restauración de máquinas virtuales de Azure cifradas](../../backup/backup-azure-vms-encryption.md). 
>
> - Cifrar o deshabilitar el cifrado puede provocar el reinicio de una máquina virtual.

## <a name="enable-encryption-on-new-iaas-vms-created-from-the-marketplace"></a>Habilitación del cifrado en nuevas máquinas virtuales IaaS creadas en Marketplace
Puede habilitar el cifrado de disco en nuevas máquinas virtuales IaaS de Windows desde Marketplace en Azure mediante la plantilla de Resource Manager. La plantilla crea una máquina virtual Windows cifrada mediante la imagen de la galería de Windows Server 2012.

1. En la [plantilla de Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-create-new-vm-gallery-image), haga clic en **Deploy to Azure** (Implementar en Azure).

2. Seleccione la suscripción, el grupo de recursos, la ubicación del grupo de recursos, los parámetros, los términos legales y el contrato. Haga clic en **Comprar** para implementar una nueva máquina virtual IaaS donde esté habilitado el cifrado.

3. Después de implementar la plantilla, compruebe el estado de cifrado de la máquina virtual con su método preferido:
     - Realice la comprobación con la CLI de Azure con el comando [az vm encryption show](/cli/azure/vm/encryption#az-vm-encryption-show). 

         ```azurecli-interactive 
         az vm encryption show --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup"
         ```

     - Realice la comprobación con Azure PowerShell a través del comando [Get-AzVmDiskEncryptionStatus](/powershell/module/az.compute/get-azvmdiskencryptionstatus). 

         ```azurepowershell-interactive
         Get-AzVmDiskEncryptionStatus -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM'
         ```

     -  Seleccione la máquina virtual y haga clic en **Discos** en el encabezado **Configuración** para comprobar el estado de cifrado en el portal. En el gráfico que aparece en **Cifrado**, verá si está habilitado. 
           ![Azure Portal: cifrado de disco habilitado](../media/disk-encryption/disk-encryption-fig2.png)

En la tabla siguiente figuran los parámetros de la plantilla de Resource Manager para nuevas máquinas virtuales en el escenario de Marketplace con el identificador de cliente de Azure AD:

| Parámetro | Descripción | 
| --- | --- |
| adminUserName | Nombre de usuario administrador para la máquina virtual. |
| adminPassword | Contraseña de usuario administrador para la máquina virtual. |
| newStorageAccountName | Nombre de la cuenta de almacenamiento que almacena el sistema operativo y los VHD de datos. |
| vmSize | Tamaño de la máquina virtual. Actualmente, solo se admiten las series A, D y G estándar. |
| virtualNetworkName | Nombre de la red virtual a la que debería pertenecer la NIC de la máquina virtual. |
| subnetName | Nombre de la subred en la red virtual a la que debería pertenecer la NIC de la máquina virtual. |
| AADClientID | Identificador de cliente de la aplicación de Azure AD que tiene permisos para escribir secretos en el almacén de claves. |
| AADClientSecret | Secreto de cliente de la aplicación de Azure AD que tiene permisos para escribir secretos en el almacén de claves. |
| keyVaultURL | Dirección URL del almacén de claves en el que se que debe cargar la clave de BitLocker. Puede obtenerla mediante el cmdlet `(Get-AzKeyVault -VaultName "MyKeyVault" -ResourceGroupName "MyKeyVaultResourceGroupName").VaultURI` o la CLI de Azure `az keyvault show --name "MySecureVault" --query properties.vaultUri` |
| keyEncryptionKeyURL | Dirección URL de la clave de cifrado de claves que se utiliza para cifrar la clave generada por BitLocker (opcional). </br> </br>KeyEncryptionKeyURL es un parámetro opcional. Puede aportar su propia KEK para proteger aún más la clave de cifrado de datos (frase de contraseña secreta) en el almacén de claves. |
| keyVaultResourceGroup | Grupo de recursos del almacén de claves. |
| vmName | Nombre de la máquina virtual en que se va a realizar la operación de cifrado. |


## <a name="enable-encryption-on-existing-or-running-iaas-windows-vms"></a><a name="bkmk_RunningWinVM"></a> Habilitación del cifrado en máquinas virtuales IaaS de Windows existentes o en ejecución
En este escenario, puede habilitar el cifrado mediante una plantilla, los cmdlets de PowerShell o los comandos de la CLI. En las siguientes secciones se explica con más detalle cómo habilitar Azure Disk Encryption. 


### <a name="enable-encryption-on-existing-or-running-vms-with-azure-powershell"></a><a name="bkmk_RunningWinVMPSH"></a> Habilitación del cifrado en máquinas virtuales existentes o en ejecución con Azure PowerShell 
Use el cmdlet [Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension) para habilitar el cifrado en una máquina virtual IaaS en ejecución en Azure. Para obtener información acerca de cómo habilitar el cifrado con Azure Disk Encryption mediante el uso de cmdlets de PowerShell, vea las entradas de blog [Explore Azure Disk Encryption with Azure PowerShell - Part 1](/archive/blogs/azuresecurity/explore-azure-disk-encryption-with-azure-powershell) (Exploración de Azure Disk Encryption con Azure PowerShell - Parte 1) y [Explore Azure Disk Encryption with Azure PowerShell - Part 2](/archive/blogs/azuresecurity/explore-azure-disk-encryption-with-azure-powershell-part-2) (Exploración de Azure Disk Encryption con Azure PowerShell - Parte 2).

-  **Cifrar una máquina virtual en ejecución mediante un secreto de cliente:** el siguiente script inicializa las variables y ejecuta el cmdlet Set-AzVMDiskEncryptionExtension. Ya se deben haber satisfecho los requisitos previos de crear el grupo de recursos, la máquina virtual, el almacén de claves, la aplicación de AAD y el secreto de cliente. Reemplace MyKeyVaultResourceGroup, MyVirtualMachineResourceGroup, MySecureVM, MySecureVault, My-AAD-client-ID y My-AAD-client-secret por sus propios valores.
     ```azurepowershell
      $KVRGname = 'MyKeyVaultResourceGroup';
      $VMRGName = 'MyVirtualMachineResourceGroup';
      $vmName = 'MySecureVM';
      $aadClientID = 'My-AAD-client-ID';
      $aadClientSecret = 'My-AAD-client-secret';
      $KeyVaultName = 'MySecureVault';
      $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
      $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
      $KeyVaultResourceId = $KeyVault.ResourceId;

      Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId;
    ```
- **Cifrar una máquina virtual en ejecución con KEK para encapsular el secreto de cliente:** Azure Disk Encryption le permite especificar una clave existente en el almacén de claves para encapsular los secretos de cifrado de disco que se generaron al habilitar el cifrado. Cuando se especifica una clave de cifrado de claves, Azure Disk Encryption usa esa clave para encapsular los secretos de cifrado antes de escribirlos en Key Vault. 

     ```azurepowershell
     $KVRGname = 'MyKeyVaultResourceGroup';
     $VMRGName = 'MyVirtualMachineResourceGroup';
     $vmName = ‘MyExtraSecureVM’;
     $aadClientID = 'My-AAD-client-ID';
     $aadClientSecret = 'My-AAD-client-secret';
     $KeyVaultName = 'MySecureVault';
     $keyEncryptionKeyName = 'MyKeyEncryptionKey';
     $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
     $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
     $KeyVaultResourceId = $KeyVault.ResourceId;
     $keyEncryptionKeyUrl = (Get-AzKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;

     Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGname -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId;

     ```
     
   >[!NOTE]
   > La sintaxis del valor del parámetro disk-encryption-keyvault es la cadena completa del identificador: /subscriptions/[subscription-id-guid]/resourceGroups/[resource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name]</br> La sintaxis del valor del parámetro key-encryption-key es el URI completo de KEK como en: https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id] 

- **Comprobar que los discos están cifrados:** para comprobar el estado de cifrado de una máquina virtual de IaaS, use el cmdlet [Get-AzVmDiskEncryptionStatus](/powershell/module/az.compute/get-azvmdiskencryptionstatus). 
     ```azurepowershell-interactive
     Get-AzVmDiskEncryptionStatus -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM'
     ```
    
- **Deshabilitar el cifrado de disco:** para deshabilitar el cifrado, use el cmdlet [Disable-AzureRmVMDiskEncryption](/powershell/module/az.compute/disable-azvmdiskencryption). 
     ```azurepowershell-interactive
     Disable-AzVMDiskEncryption -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM'
     ```

### <a name="enable-encryption-on-existing-or-running-vms-with--azure-cli"></a><a name="bkmk_RunningWinVMCLI"></a>Habilitación del cifrado en máquinas virtuales existentes o en ejecución con la CLI de Azure
Use el comando [az vm encryption enable](/cli/azure/vm/encryption#az-vm-encryption-enable) para habilitar el cifrado en una máquina virtual IaaS en ejecución en Azure.

- **Cifrar una máquina virtual en ejecución mediante un secreto de cliente:**

    ```azurecli-interactive
    az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --aad-client-id "<my spn created with CLI/my Azure AD ClientID>"  --aad-client-secret "My-AAD-client-secret" --disk-encryption-keyvault "MySecureVault" --volume-type [All|OS|Data]
    ```

- **Cifrar una máquina virtual en ejecución con KEK para encapsular el secreto de cliente:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --aad-client-id "<my spn created with CLI which is the Azure AD ClientID>"  --aad-client-secret "My-AAD-client-secret" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type [All|OS|Data]
     ```

     >[!NOTE]
     > La sintaxis del valor del parámetro disk-encryption-keyvault es la cadena completa del identificador: /subscriptions/[subscription-id-guid]/resourceGroups/[resource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name] </br> La sintaxis del valor del parámetro key-encryption-key es el URI completo de KEK como en: https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id] 

- **Comprobar que los discos están cifrados:** para comprobar el estado de cifrado de una máquina virtual IaaS, use el comando [az vm encryption show](/cli/azure/vm/encryption#az-vm-encryption-show). 

     ```azurecli-interactive
     az vm encryption show --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup"
     ```

- **Deshabilitar el cifrado:** para deshabilitar el cifrado, use el comando [az vm encryption disable](/cli/azure/vm/encryption#az-vm-encryption-disable). 
     ```azurecli-interactive
     az vm encryption disable --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup" --volume-type [ALL, DATA, OS]
     ```


### <a name="using-the-resource-manager-template"></a><a name="bkmk_RunningWinVMwRM"> </a>Uso de la plantilla de Resource Manager
Puede habilitar el cifrado de disco en máquinas virtuales IaaS de Windows existentes o en ejecución en Azure mediante la [plantilla de Resource Manager para cifrar una máquina virtual Windows en ejecución](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-windows-vm).


1. En la plantilla de inicio rápido de Azure, haga clic en **Deploy to Azure** (Implementar en Azure).

2. Seleccione la suscripción, el grupo de recursos, la ubicación del grupo de recursos, los parámetros, los términos legales y el contrato. Haga clic en **Comprar** para habilitar el cifrado en la máquina virtual IaaS existente o en ejecución.

En la tabla siguiente figuran los parámetros de la plantilla de Resource Manager para máquinas virtuales existentes o en ejecución que usan un identificador de cliente de Azure AD:

| Parámetro | Descripción |
| --- | --- |
| AADClientID | Identificador de cliente de la aplicación de Azure AD que tiene permisos para escribir secretos en el almacén de claves. |
| AADClientSecret | Secreto de cliente de la aplicación de Azure AD que tiene permisos para escribir secretos en el almacén de claves. |
| keyVaultName | Nombre del almacén de claves en el que se que debe cargar la clave de BitLocker. Puede obtenerlo mediante el cmdlet `(Get-AzKeyVault -ResourceGroupName <MyKeyVaultResourceGroupName>). Vaultname` o el comando de la CLI de Azure `az keyvault list --resource-group "MySecureGroup"`|
|  keyEncryptionKeyURL | Dirección URL de la clave de cifrado de claves que se utiliza para cifrar la clave generada por BitLocker. Este parámetro es opcional si selecciona **nokek** en la lista desplegable de UseExistingKek. Si selecciona **kek** en la lista desplegable de UseExistingKek, debe proporcionar el valor de _keyEncryptionKeyURL_. |
| volumeType | Tipo de volumen en que se realiza la operación de cifrado. Los valores válidos son _SO_, _Datos_ y _Todo_. |
| sequenceVersion | Versión de la secuencia de la operación de BitLocker. Incremente el número de versión cada vez que se realice una operación de cifrado de disco en la misma máquina virtual. |
| vmName | Nombre de la máquina virtual en que se va a realizar la operación de cifrado. |


## <a name="new-iaas-vms-created-from-customer-encrypted-vhd-and-encryption-keys"></a><a name="bkmk_VHDpre"> </a>Nuevas máquinas virtuales IaaS creadas a partir de discos duros virtuales cifrados por el cliente y claves de cifrado
En este escenario, se puede habilitar el cifrado mediante la plantilla de Resource Manager, cmdlets de PowerShell o comandos de la CLI. Las siguientes secciones explican con más detalle la plantilla de Resource Manager y los comandos de la CLI. 

Use las instrucciones del apéndice para preparar imágenes previamente cifradas que se pueden usar en Azure. Una vez creada la imagen, puede seguir los pasos de la sección siguiente para crear una VM cifrada de Azure.

* [Preparación de un VHD con Windows precifrado](disk-encryption-sample-scripts.md#prepare-a-pre-encrypted-windows-vhd)


### <a name="encrypt-vms-with-pre-encrypted-vhds-with-azure-powershell"></a><a name="bkmk_VHDprePSH"> </a>Cifrado de máquinas virtuales con discos duros virtuales previamente cifrados con Azure PowerShell
Puede habilitar el cifrado de disco en el disco duro virtual cifrado mediante el cmdlet [Set-AzVMOSDisk](/powershell/module/az.compute/set-azvmosdisk#examples) de PowerShell. En el ejemplo siguiente se proporcionan algunos parámetros comunes. 

```powershell
$VirtualMachine = New-AzVMConfig -VMName "MySecureVM" -VMSize "Standard_A1"
$VirtualMachine = Set-AzVMOSDisk -VM $VirtualMachine -Name "SecureOSDisk" -VhdUri "os.vhd" Caching ReadWrite -Windows -CreateOption "Attach" -DiskEncryptionKeyUrl "https://mytestvault.vault.azure.net/secrets/Test1/514ceb769c984379a7e0230bddaaaaaa" -DiskEncryptionKeyVaultId "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myKVresourcegroup/providers/Microsoft.KeyVault/vaults/mytestvault"
New-AzVM -VM $VirtualMachine -ResourceGroupName "MyVirtualMachineResourceGroup"
```

## <a name="enable-encryption-on-a-newly-added-data-disk"></a>Habilitación del cifrado en un disco de datos recién agregado
También puede [agregar un nuevo disco a una máquina virtual Windows con PowerShell](attach-disk-ps.md) o [mediante Azure Portal](attach-managed-disk-portal.md). 

### <a name="enable-encryption-on-a-newly-added-disk-with-azure-powershell"></a>Habilitación del cifrado en un disco recién agregado con Azure PowerShell
 Al usar PowerShell para cifrar un nuevo disco para máquinas virtuales Windows, se debe especificar una nueva versión de secuencia. La versión de secuencia debe ser única. El siguiente script genera un GUID para la versión de secuencia. En algunos casos, un disco de datos recién agregado se podría cifrar automáticamente mediante la extensión Azure Disk Encryption. El cifrado automático suele producirse cuando se reinicia la máquina virtual después de que el nuevo disco se pone en línea. Esto se suele producir porque se especificó "All" para el tipo de volumen cuando el cifrado de disco se ejecutó anteriormente en la máquina virtual. Si el cifrado automático se produce en un disco de datos recién agregado, se recomienda volver a ejecutar el cmdlet Set-AzVmDiskEncryptionExtension con la nueva versión de secuencia. Si el nuevo disco de datos es de cifrado automático y no desea cifrarlo, descifre todas las unidades en primer lugar y, a continuación, vuelva a cifrarlas con una nueva versión de secuencia que especifique el sistema operativo para el tipo de volumen. 
 

-  **Cifrar una máquina virtual en ejecución mediante un secreto de cliente:** el siguiente script inicializa las variables y ejecuta el cmdlet Set-AzVMDiskEncryptionExtension. Ya se deben haber satisfecho los requisitos previos de crear el grupo de recursos, la máquina virtual, el almacén de claves, la aplicación de AAD y el secreto de cliente. Reemplace MyKeyVaultResourceGroup, MyVirtualMachineResourceGroup, MySecureVM, MySecureVault, My-AAD-client-ID y My-AAD-client-secret por sus propios valores. Este ejemplo utiliza "All" para el parámetro -VolumeType, que incluye los volúmenes de sistema operativo y de datos. Si solo quiere cifrar el volumen del sistema operativo, use "OS" para el parámetro VolumeType. 

     ```azurepowershell
      $sequenceVersion = [Guid]::NewGuid();
      $KVRGname = 'MyKeyVaultResourceGroup';
      $VMRGName = 'MyVirtualMachineResourceGroup';
      $vmName = 'MySecureVM';
      $aadClientID = 'My-AAD-client-ID';
      $aadClientSecret = 'My-AAD-client-secret';
      $KeyVaultName = 'MySecureVault';
      $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
      $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
      $KeyVaultResourceId = $KeyVault.ResourceId;

      Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGname -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType 'all' –SequenceVersion $sequenceVersion;
    ```
- **Cifrar una máquina virtual en ejecución con KEK para encapsular el secreto de cliente:** Azure Disk Encryption le permite especificar una clave existente en el almacén de claves para encapsular los secretos de cifrado de disco que se generaron al habilitar el cifrado. Cuando se especifica una clave de cifrado de claves, Azure Disk Encryption usa esa clave para encapsular los secretos de cifrado antes de escribirlos en Key Vault. Este ejemplo utiliza "All" para el parámetro -VolumeType, que incluye los volúmenes de sistema operativo y de datos. Si solo quiere cifrar el volumen del sistema operativo, use "OS" para el parámetro VolumeType. 

     ```azurepowershell
     $sequenceVersion = [Guid]::NewGuid();
     $KVRGname = 'MyKeyVaultResourceGroup';
     $VMRGName = 'MyVirtualMachineResourceGroup';
     $vmName = 'MyExtraSecureVM';
     $aadClientID = 'My-AAD-client-ID';
     $aadClientSecret = 'My-AAD-client-secret';
     $KeyVaultName = 'MySecureVault';
     $keyEncryptionKeyName = 'MyKeyEncryptionKey';
     $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
     $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
     $KeyVaultResourceId = $KeyVault.ResourceId;
     $keyEncryptionKeyUrl = (Get-AzKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;

     Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGname -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId -VolumeType 'all' –SequenceVersion $sequenceVersion;

     ```

    >[!NOTE]
    > La sintaxis del valor del parámetro disk-encryption-keyvault es la cadena completa del identificador: /subscriptions/[subscription-id-guid]/resourceGroups/[resource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name]</br> La sintaxis del valor del parámetro key-encryption-key es el URI completo de KEK como en: https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id] 

### <a name="enable-encryption-on-a-newly-added-disk-with-azure-cli"></a>Habilitación del cifrado en un disco recién agregado con la CLI de Azure
  El comando de la CLI de Azure proporciona automáticamente una nueva versión de secuencia al ejecutarlo para habilitar el cifrado. Los valores aceptables para el parámetro volumen-type son All, OS y Data. Es posible que deba cambiar el parámetro volume-type para SO o Data si solo se está cifrando un tipo de disco para la VM. Los ejemplos usan "All" para el parámetro de volume-type. 

-  **Cifrar una máquina virtual en ejecución mediante un secreto de cliente:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --aad-client-id "<my spn created with CLI/my Azure AD ClientID>"  --aad-client-secret "My-AAD-client-secret" --disk-encryption-keyvault "MySecureVault" --volume-type "All"
     ```

- **Cifrar una máquina virtual en ejecución con KEK para encapsular el secreto de cliente:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --aad-client-id "<my spn created with CLI which is the Azure AD ClientID>"  --aad-client-secret "My-AAD-client-secret" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type "all"
     ```


## <a name="enable-encryption-using-azure-ad-client-certificate-based-authentication"></a>Habilite el cifrado mediante la autenticación basada en certificados de cliente de Azure AD.
Puede usar la autenticación de certificado de cliente con o sin KEK. Antes de usar los scripts de PowerShell, ya debe tener el certificado cargado en el almacén de claves e implementado en la máquina virtual. Si también usa KEK, ya debe existir KEK. Para más información, consulte la sección [Autenticación basada en certificados de Azure AD](disk-encryption-key-vault-aad.md#certificate-based-authentication-optional) del artículo de requisitos previos.


### <a name="enable-encryption-using-certificate-based-authentication-with-azure-powershell"></a>Habilitación del cifrado mediante la autenticación basada en certificados con Azure PowerShell

```powershell
## Fill in 'MyVirtualMachineResourceGroup', 'MyKeyVaultResourceGroup', 'My-AAD-client-ID', 'MySecureVault, and ‘MySecureVM’.

$VMRGName = 'MyVirtualMachineResourceGroup'
$KVRGname = 'MyKeyVaultResourceGroup';
$AADClientID ='My-AAD-client-ID';
$KeyVaultName = 'MySecureVault';
$VMName = 'MySecureVM';
$KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
$diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
$KeyVaultResourceId = $KeyVault.ResourceId;

# Fill in the certificate path and the password so the thumbprint can be set as a variable. 

$certPath = '$CertPath = "C:\certificates\mycert.pfx';
$CertPassword ='Password'
$Cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($CertPath, $CertPassword)
$aadClientCertThumbprint = $cert.Thumbprint;

# Enable disk encryption using the client certificate thumbprint

Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $VMName -AadClientID $AADClientID -AadClientCertThumbprint $AADClientCertThumbprint -DiskEncryptionKeyVaultUrl $DiskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId
```
  
### <a name="enable-encryption-using-certificate-based-authentication-and-a-kek-with-azure-powershell"></a>Habilitación del cifrado mediante la autenticación basada en certificados y KEK con Azure PowerShell

```powershell
# Fill in 'MyVirtualMachineResourceGroup', 'MyKeyVaultResourceGroup', 'My-AAD-client-ID', 'MySecureVault,, 'MySecureVM’, and "KEKName.

$VMRGName = 'MyVirtualMachineResourceGroup';
$KVRGname = 'MyKeyVaultResourceGroup';
$AADClientID ='My-AAD-client-ID';
$KeyVaultName = 'MySecureVault';
$VMName = 'MySecureVM';
$keyEncryptionKeyName ='KEKName';
$KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
$diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
$KeyVaultResourceId = $KeyVault.ResourceId;
$keyEncryptionKeyUrl = (Get-AzKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;

## Fill in the certificate path and the password so the thumbprint can be read and set as a variable.

$certPath = '$CertPath = "C:\certificates\mycert.pfx';
$CertPassword ='Password'
$Cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($CertPath, $CertPassword)
$aadClientCertThumbprint = $cert.Thumbprint;

# Enable disk encryption using the client certificate thumbprint and a KEK

Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $VMName -AadClientID $AADClientID -AadClientCertThumbprint $AADClientCertThumbprint -DiskEncryptionKeyVaultUrl $DiskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId
```

## <a name="disable-encryption"></a>Deshabilitación del cifrado
Puede deshabilitar el cifrado con Azure PowerShell, la CLI de Azure o una plantilla de Resource Manager. 

- **Deshabilitar el cifrado de disco con Azure PowerShell:** para deshabilitar el cifrado, use el cmdlet [Disable-AzureRmVMDiskEncryption](/powershell/module/az.compute/disable-azvmdiskencryption). 
     ```azurepowershell-interactive
     Disable-AzVMDiskEncryption -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM'
     ```

- **Deshabilitar el cifrado con la CLI de Azure:** para deshabilitar el cifrado, use el comando [az vm encryption disable](/cli/azure/vm/encryption#az-vm-encryption-disable). 
     ```azurecli-interactive
     az vm encryption disable --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup" --volume-type [ALL, DATA, OS]
     ```
- **Deshabilitar el cifrado con una plantilla de Resource Manager:** 

    1. Haga clic en **Deploy to Azure** (Implementar en Azure) en la plantilla [Deshabilitar el cifrado de disco en una máquina virtual Windows en ejecución](https://github.com/Azure/azure-quickstart-templates/tree/master/201-decrypt-running-windows-vm).
    2. Seleccione la suscripción, el grupo de recursos, la ubicación, la máquina virtual, los términos legales y el contrato.
    3.  Haga clic en **Comprar** para deshabilitar el cifrado de disco en una máquina virtual Windows en ejecución. 

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Introducción a Azure Disk Encryption](disk-encryption-overview.md)