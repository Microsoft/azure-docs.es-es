---
title: 'Azure Key Vault: Uso de la eliminación temporal con PowerShell'
description: Aprenda a usar Azure PowerShell para utilizar la característica de eliminación temporal de Azure Key Vault que permite la recuperación de almacenes de claves y objetos de Key Vault.
services: key-vault
author: ShaneBala-keyvault
manager: ravijan
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 08/11/2020
ms.author: sudbalas
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 251159c4fcf27ceb4d8b0dddeecbe501c7346e9e
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89073240"
---
# <a name="how-to-use-key-vault-soft-delete-with-powershell"></a>Uso de la eliminación temporal de Key Vault con PowerShell

La característica de eliminación temporal de Azure Key Vault permite recuperar almacenes y objetos de almacenes eliminados. En concreto, la eliminación temporal trata los siguientes escenarios:

- Compatibilidad con la eliminación recuperable de una instancia de Key Vault
- Compatibilidad con la eliminación recuperable de objetos de Key Vault: claves, secretos y certificados

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

- Azure PowerShell 1.0.0 o posterior: si aún no lo tiene instalado, instale Azure PowerShell y asócielo con la suscripción de Azure. Para ello, consulte [How to install and configure Azure PowerShell](https://docs.microsoft.com/powershell/azure/) (Instalación y configuración de Azure PowerShell). 

>[!NOTE]
> Hay una versión no actualizada del archivo de formato de salida de PowerShell de Key Vault que se **podría** cargar en el entorno en lugar de la versión correcta. Se espera una versión actualizada de PowerShell que contenga la corrección necesaria para el formato de salida y este tema se actualizará en ese momento. La solución actual, si se encuentra con este problema de formato, es:
> - Use la siguiente consulta si no ve la propiedad Habilitar la eliminación temporal descrita en este tema: `$vault = Get-AzKeyVault -VaultName myvault; $vault.EnableSoftDelete`.


Para obtener más información específica de referencia sobre Key Vault para PowerShell, consulte la [referencia de Azure Key Vault para PowerShell](/powershell/module/az.keyvault).

## <a name="required-permissions"></a>Permisos necesarios

Las operaciones de Key Vault se administran por separado mediante los permisos de control de acceso basado en roles (RBAC) como se indica a continuación:

| Operación | Descripción | Permiso del usuario |
|:--|:--|:--|
|List|Enumera los almacenes de claves eliminados.|Microsoft.KeyVault/deletedVaults/read|
|Recover|Restaura un almacén de claves eliminado.|Microsoft.KeyVault/vaults/write|
|Purgar|Elimina permanentemente un almacén de claves eliminado y todo su contenido.|Microsoft.KeyVault/locations/deletedVaults/purge/action|

Para más información sobre los permisos y el control de acceso, consulte [Protección de un almacén de claves](secure-your-key-vault.md).

## <a name="enabling-soft-delete"></a>Habilitar la eliminación temporal

La "eliminación temporal" se habilita para permitir la recuperación de un almacén de claves eliminado o de los objetos almacenados en un almacén de claves.

> [!IMPORTANT]
> La acción de habilitar la "eliminación temporal" en un almacén de claves es irreversible. Una vez que la propiedad de eliminación temporal se ha establecido en "true", no se puede modificar ni eliminar.  

### <a name="existing-key-vault"></a>Almacén de claves existente

Para un almacén de claves existente denominado ContosoVault, habilite la eliminación temporal como se indica a continuación. 

```powershell
($resource = Get-AzResource -ResourceId (Get-AzKeyVault -VaultName "ContosoVault").ResourceId).Properties | Add-Member -MemberType "NoteProperty" -Name "enableSoftDelete" -Value "true"

Set-AzResource -resourceid $resource.ResourceId -Properties $resource.Properties
```

### <a name="new-key-vault"></a>Almacén de claves nuevo

La eliminación temporal se activa automáticamente de forma predeterminada para todos los nuevos almacenes de claves. El 31 de diciembre de 2020 ya no será posible deshabilitar la eliminación temporal en ningún almacén de claves. 

### <a name="verify-soft-delete-enablement"></a>Comprobación de la habilitación de la eliminación temporal

Para comprobar que un almacén de claves tiene habilitada la eliminación temporal, ejecute el comando *show* y busque el atributo "Soft Delete Enabled?" :

```powershell
Get-AzKeyVault -VaultName "ContosoVault"
```

## <a name="deleting-a-soft-delete-protected-key-vault"></a>Eliminación de un almacén de claves protegido por eliminación temporal

El comando para eliminar un almacén de claves cambia de comportamiento, según si está habilitada la eliminación temporal.

> [!IMPORTANT]
>Si ejecuta el comando anterior para un almacén de claves que no tiene habilitada la eliminación temporal, este almacén de claves y todo su contenido se eliminarán definitivamente sin ninguna oportunidad de recuperación.

```powershell
Remove-AzKeyVault -VaultName 'ContosoVault'
```

### <a name="how-soft-delete-protects-your-key-vaults"></a>Uso de la eliminación temporal para proteger los almacenes de claves

Con la eliminación temporal habilitada:

- El almacén de claves eliminado se quita de su grupo de recursos y se coloca en un espacio de nombres reservado, asociado con la ubicación donde se creó. 
- Los objetos eliminados, como las claves, los secretos y los certificados, son inaccesibles mientras el almacén de claves que los contenga esté en estado eliminado. 
- El nombre DNS de un almacén de claves eliminado es reservado, así se impide que se cree un almacén de claves con el mismo nombre.  

Puede ver los almacenes de claves en estado eliminado asociados a su suscripción con el comando siguiente:

```powershell
Get-AzKeyVault -InRemovedState 
```

- El campo *ID* se puede usar para identificar el recurso durante la recuperación o purga. 
- *Resource ID* (Identificador de recurso) es el identificador de recurso original de este almacén. Puesto que este almacén de claves está ahora en estado eliminado, no existe ningún recurso con ese identificador de recurso. 
- *Scheduled Purge Date* (Fecha de purga programada) es la fecha en la que el almacén se eliminará definitivamente si no se realiza ninguna acción. El período de retención predeterminado que se utiliza para calcular la *fecha de purga programada* es de 90 días.

## <a name="recovering-a-key-vault"></a>Recuperación de un almacén de claves

Para recuperar un almacén de claves, debe especificar el nombre del almacén de claves, el grupo de recursos y la ubicación. Anote la ubicación y el grupo de recursos del almacén de claves eliminado ya que los necesitará para un proceso de recuperación.

```powershell
Undo-AzKeyVaultRemoval -VaultName ContosoVault -ResourceGroupName ContosoRG -Location westus
```

Cuando se recupera un almacén de claves, se crea un recurso con el identificador de recurso original del almacén de claves. Si se quita el grupo de recursos original, se debe crear uno con el mismo nombre antes de intentar la recuperación.

## <a name="deleting-and-purging-key-vault-objects"></a>Eliminación y purga de objetos del almacén de claves

El siguiente comando elimina la clave "ContosoFirstKey", en un almacén de claves denominado "ContosoVault", que tiene habilitada la eliminación temporal:

```powershell
Remove-AzKeyVaultKey -VaultName ContosoVault -Name ContosoFirstKey
```

Con el almacén de claves habilitado para la eliminación temporal, una clave eliminada todavía se muestra para eliminar, a no ser que enumere explícitamente las claves eliminadas. La mayoría de las operaciones que se realicen en una clave en estado eliminado producirán error, excepto las operaciones de enumeración, recuperación o purga. 

Por ejemplo, el siguiente comando enumera las claves eliminadas en el almacén de claves "ContosoVault":

```powershell
Get-AzKeyVaultKey -VaultName ContosoVault -InRemovedState
```

### <a name="transition-state"></a>Estado de transición 

Cuando se elimina una clave de un almacén de claves con eliminación temporal habilitada, puede tardar unos segundos en completarse la transición. Durante esta transición, puede parecer que la clave no está ni en estado activo ni en estado eliminado. 

### <a name="using-soft-delete-with-key-vault-objects"></a>Uso de la eliminación temporal con objetos de almacén de claves

Al igual que con los almacenes de claves, una clave, secreto o certificado eliminados permanecerán en ese estado durante 90 días a menos que los recupere o los purgue. 

#### <a name="keys"></a>Claves

Para recuperar una clave eliminada temporalmente:

```powershell
Undo-AzKeyVaultKeyRemoval -VaultName ContosoVault -Name ContosoFirstKey
```

Para eliminar definitivamente (lo que se conoce también como purga) una clave eliminada temporalmente:

> [!IMPORTANT]
> Al purgar una clave se elimina definitivamente, lo que significa que no se podrá recuperar de nuevo. 

```powershell
Remove-AzKeyVaultKey -VaultName ContosoVault -Name ContosoFirstKey -InRemovedState
```

Las acciones de **recuperación** y **purga** tienen permisos propios asociados a una directiva de acceso al almacén de claves. Para que un usuario o una entidad de servicio puedan ejecutar una acción de **recuperación** o **purga**, deben tener el permiso correspondiente para ese almacén o secreto. De forma predeterminada, el permiso de **purga** no se agrega a la directiva de acceso del almacén de claves cuando se usa el acceso directo "todos" para conceder todos los permisos. Debe conceder específicamente el permiso de **purga**. 

#### <a name="set-a-key-vault-access-policy"></a>Establecimiento de una directiva de acceso del almacén de claves

El siguiente comando concede a user@contoso.com el permiso para realizar varias operaciones en las claves de *ContosoVault*, incluida la **purga**.

```powershell
Set-AzKeyVaultAccessPolicy -VaultName ContosoVault -UserPrincipalName user@contoso.com -PermissionsToKeys get,create,delete,list,update,import,backup,restore,recover,purge
```

>[!NOTE] 
> Si tiene un almacén de claves existente para el que se acaba de habilitar la eliminación temporal, puede que no tenga los permisos de **recuperación** y **purga**.

#### <a name="secrets"></a>Secretos

Al igual que las claves, los secretos se administran con sus propios comandos:

- Elimine un secreto llamado SQLPassword: 
  ```powershell
  Remove-AzKeyVaultSecret -VaultName ContosoVault -name SQLPassword
  ```

- Enumere todos los secretos eliminados de un almacén de claves: 
  ```powershell
  Get-AzKeyVaultSecret -VaultName ContosoVault -InRemovedState
  ```

- Recupere un secreto en el estado eliminado: 
  ```powershell
  Undo-AzKeyVaultSecretRemoval -VaultName ContosoVault -Name SQLPAssword
  ```

- Purgue un secreto en el estado eliminado: 

  > [!IMPORTANT]
  > Si purga un secreto se eliminará definitivamente y no se podrá volver a recuperar.

  ```powershell
  Remove-AzKeyVaultSecret -VaultName ContosoVault -InRemovedState -name SQLPassword
  ```

#### <a name="certificates"></a>Certificados

Puede administrar certificados mediante los siguientes comandos:

- Eliminar un certificado: 
  ```powershell
  Remove-AzKeyVaultCertificate -VaultName ContosoVault -Name 'MyCert'
  ```

- Enumerar todos los certificados eliminados de un almacén de claves: 
  ```powershell
  Get-AzKeyVaultCertificate -VaultName ContosoVault -InRemovedState
  ```

- Recuperar un certificado en estado eliminado: 
  ```powershell
  Undo-AzKeyVaultCertificateRemoval -VaultName ContosoVault -Name 'MyCert'
  ```

- Purgar un certificado en estado eliminado: 

  > [!IMPORTANT]
  > Si purga un certificado, se eliminará definitivamente y no se podrá recuperar.

  ```powershell
  Remove-AzKeyVaultcertificate -VaultName ContosoVault -Name 'MyCert' -InRemovedState 
  ```
  
## <a name="purging-a-soft-delete-protected-key-vault"></a>Purga de un almacén de claves protegido por eliminación temporal

> [!IMPORTANT]
> Si purga un almacén de claves o uno de los objetos que contiene, se eliminará definitivamente, lo que significa que no se podrá recuperar.

La función de purga se usa para eliminar permanentemente un objeto del almacén de claves o un almacén de claves completo que anteriormente se habían eliminado temporalmente. Como se muestra en la sección anterior, los objetos almacenados en un almacén de claves con la característica de eliminación temporal habilitada pueden pasar por varios estados:
- **Activo**: antes de la eliminación.
- **Eliminado temporalmente**: después de la eliminación, puede mostrarse y recuperarse para volver al estado activo.
- **Eliminado permanentemente**: después de la purga, no se puede recuperar.


Lo mismo ocurre con el almacén de claves. Con el fin de eliminar permanentemente un almacén de claves eliminado temporalmente y su contenido, debe purgar el propio almacén de claves.

### <a name="purging-a-key-vault"></a>Purga de un almacén de claves

Cuando se purga un almacén de claves, todo su contenido, incluidas las claves, los secretos y los certificados, se eliminan definitivamente. Para purgar un almacén de claves eliminado de forma temporal, use el comando `Remove-AzKeyVault` con la opción `-InRemovedState` y especifique la ubicación del almacén de claves eliminado con el argumento `-Location location`. Puede encontrar la ubicación de un almacén eliminado mediante el comando `Get-AzKeyVault -InRemovedState`.

```powershell
Remove-AzKeyVault -VaultName ContosoVault -InRemovedState -Location westus
```

### <a name="purge-permissions-required"></a>Se requieren permisos de purga
- Para purgar un almacén de claves eliminado, el usuario debe tener el permiso de RBAC para la operación *Microsoft.KeyVault/locations/deletedVaults/purge/action*. 
- Para mostrar un almacén de claves eliminado, el usuario necesita el permiso de RBAC para la operación *Microsoft.KeyVault/deletedVaults/read*. 
- Solo un administrador de suscripción tiene estos permisos. 

### <a name="scheduled-purge"></a>Purga programada

La lista de objetos del almacén de claves eliminado también muestra cuándo se han programado para su purga mediante Key Vault. El campo *Scheduled Purge Date* (Fecha de purga programada) indica cuándo se eliminará definitivamente un objeto de almacén de claves si no se realiza ninguna acción. De forma predeterminada, el período de retención para un objeto de almacén de claves eliminado es de 90 días.

>[!IMPORTANT]
>Un objeto de almacén purgado, desencadenado por el valor de su campo de *fecha de purga programada*, se eliminará permanentemente. No se podrá volver a recuperar.

## <a name="enabling-purge-protection"></a>Habilitación de la protección de purgas

Cuando la protección de purgas está activada, un almacén o un objeto en estado eliminado no se puede purgar hasta que ha transcurrido el período de retención. El almacén u objeto todavía se puede recuperar. Esta característica ofrece mayor seguridad de que un almacén u objeto nunca se va a eliminar de forma permanente hasta que haya transcurrido el período de retención. El período de retención predeterminado es de 90 días pero, durante la creación del almacén de claves, es posible establecer el intervalo de la directiva de retención en un valor de 7 a 90 días. La directiva de retención de protección de purgas usa el mismo intervalo. Una vez establecido, el intervalo de la directiva de retención no se puede cambiar.

Puede habilitar la protección de purgas solo si también está habilitada la eliminación temporal. No se admite la deshabilitación de la protección de purgas. 

Para activar la eliminación temporal y la protección de purgas al crear un almacén, use el cmdlet [New-AzKeyVault](/powershell/module/az.keyvault/new-azkeyvault?view=azps-1.5.0):

```powershell
New-AzKeyVault -Name ContosoVault -ResourceGroupName ContosoRG -Location westus -EnableSoftDelete -EnablePurgeProtection
```

Para agregar protección de purgas a un almacén existente (que ya tenga habilitada la eliminación temporal), use los cmdlets [Get-AzKeyVault](/powershell/module/az.keyvault/Get-AzKeyVault?view=azps-1.5.0), [Get-AzResource](/powershell/module/az.resources/get-azresource?view=azps-1.5.0) y [Set-AzResource](/powershell/module/az.resources/set-azresource?view=azps-1.5.0):

```
($resource = Get-AzResource -ResourceId (Get-AzKeyVault -VaultName "ContosoVault").ResourceId).Properties | Add-Member -MemberType "NoteProperty" -Name "enablePurgeProtection" -Value "true"

Set-AzResource -resourceid $resource.ResourceId -Properties $resource.Properties
```

## <a name="other-resources"></a>Otros recursos

- Para ver una introducción a la característica de eliminación temporal de Key Vault, consulte [Introducción a la eliminación temporal de Azure Key Vault](soft-delete-overview.md).
- Para ver una introducción al uso de Azure Key Vault, consulte [¿Qué es Azure Key Vault?](overview.md).
