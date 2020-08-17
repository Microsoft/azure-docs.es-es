---
title: 'Azure Key Vault: Uso de la eliminación temporal con la CLI'
description: Ejemplos de casos de uso de eliminación temporal con fragmentos de código de la CLI
services: key-vault
author: msmbaldwin
manager: rkarlin
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 08/12/2019
ms.author: mbaldwin
ms.openlocfilehash: 7023a2301b6b6137f7a0485523c68f21d72d67a9
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87385745"
---
# <a name="how-to-use-key-vault-soft-delete-with-cli"></a>Uso de la eliminación temporal de Key Vault con la CLI

La característica de eliminación temporal de Azure Key Vault permite recuperar almacenes y objetos de almacenes eliminados. En concreto, la eliminación temporal trata los siguientes escenarios:

- Compatibilidad con la eliminación recuperable de una instancia de Key Vault
- Compatibilidad con la eliminación recuperable de objetos de Key Vault: claves, secretos y certificados

## <a name="prerequisites"></a>Requisitos previos

- CLI de Azure: si no tiene esta configuración para su entorno, consulte [Administración de Key Vault mediante la CLI de Azure](manage-with-cli2.md).

Para obtener información de referencia específica sobre Key Vault para la CLI, consulte la [referencia de Key Vault para la CLI de Azure](https://docs.microsoft.com/cli/azure/keyvault).

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

```azurecli
az keyvault update -n ContosoVault --enable-soft-delete true
```

### <a name="new-key-vault"></a>Almacén de claves nuevo

La habilitación de la eliminación temporal para un nuevo almacén de claves se realiza en el momento de la creación agregando el indicador de habilitación de eliminación temporal al comando create.

```azurecli
az keyvault create --name ContosoVault --resource-group ContosoRG --enable-soft-delete true --location westus
```

### <a name="verify-soft-delete-enablement"></a>Comprobación de la habilitación de la eliminación temporal

Para comprobar que un almacén de claves tiene habilitada la eliminación temporal, ejecute el comando *show* y busque el atributo "Soft Delete Enabled?" :

```azurecli
az keyvault show --name ContosoVault
```

## <a name="deleting-a-soft-delete-protected-key-vault"></a>Eliminación de un almacén de claves protegido por eliminación temporal

El comando para eliminar un almacén de claves cambia de comportamiento, según si está habilitada la eliminación temporal.

> [!IMPORTANT]
>Si ejecuta el comando anterior para un almacén de claves que no tiene habilitada la eliminación temporal, este almacén de claves y todo su contenido se eliminarán definitivamente sin ninguna oportunidad de recuperación.

```azurecli
az keyvault delete --name ContosoVault
```

### <a name="how-soft-delete-protects-your-key-vaults"></a>Uso de la eliminación temporal para proteger los almacenes de claves

Con la eliminación temporal habilitada:

- El almacén de claves eliminado se quita de su grupo de recursos y se coloca en un espacio de nombres reservado, asociado con la ubicación donde se creó. 
- Los objetos eliminados, como las claves, los secretos y los certificados, son inaccesibles mientras el almacén de claves que los contenga esté en estado eliminado. 
- El nombre DNS de un almacén de claves eliminado es reservado, así se impide que se cree un almacén de claves con el mismo nombre.  

Puede ver los almacenes de claves en estado eliminado asociados a su suscripción con el comando siguiente:

```azurecli
az keyvault list-deleted
```
- El campo *ID* se puede usar para identificar el recurso durante la recuperación o purga. 
- *Resource ID* (Identificador de recurso) es el identificador de recurso original de este almacén. Puesto que este almacén de claves está ahora en estado eliminado, no existe ningún recurso con ese identificador de recurso. 
- *Scheduled Purge Date* (Fecha de purga programada) es la fecha en la que el almacén se eliminará definitivamente si no se realiza ninguna acción. El período de retención predeterminado que se utiliza para calcular la *fecha de purga programada* es de 90 días.

## <a name="recovering-a-key-vault"></a>Recuperación de un almacén de claves

Para recuperar un almacén de claves, debe especificar el nombre del almacén de claves, el grupo de recursos y la ubicación. Anote la ubicación y el grupo de recursos del almacén de claves eliminado ya que los necesitará para un proceso de recuperación.

```azurecli
az keyvault recover --location westus --resource-group ContosoRG --name ContosoVault
```

Cuando se recupera un almacén de claves, se crea un recurso con el identificador de recurso original del almacén de claves. Si se quita el grupo de recursos original, se debe crear uno con el mismo nombre antes de intentar la recuperación.

## <a name="deleting-and-purging-key-vault-objects"></a>Eliminación y purga de objetos del almacén de claves

El siguiente comando elimina la clave "ContosoFirstKey", en un almacén de claves denominado "ContosoVault", que tiene habilitada la eliminación temporal:

```azurecli
az keyvault key delete --name ContosoFirstKey --vault-name ContosoVault
```

Con el almacén de claves habilitado para la eliminación temporal, una clave eliminada seguirá apareciendo como eliminada excepto si enumera o recupera explícitamente las claves eliminadas. La mayoría de las operaciones que se realicen en una clave en el estado eliminado producirán error, excepto las operaciones de enumeración, recuperación o purga. 

Por ejemplo, para solicitar la enumeración de las claves eliminadas de un almacén de claves, use el comando siguiente:

```azurecli
az keyvault key list-deleted --vault-name ContosoVault
```

### <a name="transition-state"></a>Estado de transición 

Cuando se elimina una clave de un almacén de claves con eliminación temporal habilitada, puede tardar unos segundos en completarse la transición. Durante esta transición, puede parecer que la clave no está ni en estado activo ni en estado eliminado. 

### <a name="using-soft-delete-with-key-vault-objects"></a>Uso de la eliminación temporal con objetos de almacén de claves

Al igual que con los almacenes de claves, una clave, secreto o certificado eliminados permanecerán en ese estado durante 90 días a menos que los recupere o los purgue.

#### <a name="keys"></a>Claves

Para recuperar una clave eliminada temporalmente:

```azurecli
az keyvault key recover --name ContosoFirstKey --vault-name ContosoVault
```

Para eliminar definitivamente (lo que se conoce también como purga) una clave eliminada temporalmente:

> [!IMPORTANT]
> Al purgar una clave se elimina definitivamente, lo que significa que no se podrá recuperar de nuevo. 

```azurecli
az keyvault key purge --name ContosoFirstKey --vault-name ContosoVault
```

Las acciones de **recuperación** y **purga** tienen permisos propios asociados a una directiva de acceso al almacén de claves. Para que un usuario o una entidad de servicio puedan ejecutar una acción de **recuperación** o **purga**, deben tener el permiso correspondiente para ese almacén o secreto. De forma predeterminada, el permiso de **purga** no se agrega a la directiva de acceso del almacén de claves cuando se usa el acceso directo "todos" para conceder todos los permisos. Debe conceder específicamente el permiso de **purga**. 

#### <a name="set-a-key-vault-access-policy"></a>Establecimiento de una directiva de acceso del almacén de claves

El siguiente comando concede a user@contoso.com el permiso para realizar varias operaciones en las claves de *ContosoVault*, incluida la **purga**.

```azurecli
az keyvault set-policy --name ContosoVault --key-permissions get create delete list update import backup restore recover purge
```

>[!NOTE] 
> Si tiene un almacén de claves existente para el que se acaba de habilitar la eliminación temporal, puede que no tenga los permisos de **recuperación** y **purga**.

#### <a name="secrets"></a>Secretos

Al igual que las claves, los secretos se administran con sus propios comandos:

- Elimine un secreto llamado SQLPassword: 
  ```azurecli
  az keyvault secret delete --vault-name ContosoVault -name SQLPassword
  ```

- Enumere todos los secretos eliminados de un almacén de claves: 
  ```azurecli
  az keyvault secret list-deleted --vault-name ContosoVault
  ```

- Recupere un secreto en el estado eliminado: 
  ```azurecli
  az keyvault secret recover --name SQLPassword --vault-name ContosoVault
  ```

- Purgue un secreto en el estado eliminado: 

  > [!IMPORTANT]
  > Si purga un secreto se eliminará definitivamente y no se podrá volver a recuperar. 

  ```azurecli
  az keyvault secret purge --name SQLPAssword --vault-name ContosoVault
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

Cuando se purga un almacén de claves, todo su contenido, incluidas las claves, los secretos y los certificados, se eliminan definitivamente. Para purgar un almacén de claves eliminado temporalmente, use el comando `az keyvault purge`. Puede encontrar la ubicación de los almacenes de claves eliminados de la suscripción mediante el comando `az keyvault list-deleted`.

```azurecli
az keyvault purge --location westus --name ContosoVault
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

Cuando la protección de purgas está activada, un almacén o un objeto en estado eliminado no se puede purgar hasta que ha transcurrido el período de retención de 90 días. El almacén u objeto todavía se puede recuperar. Esta característica ofrece mayor seguridad de que un almacén u objeto nunca se va a eliminar de forma permanente hasta que haya transcurrido el período de retención.

Puede habilitar la protección de purgas solo si también está habilitada la eliminación temporal. No se admite la deshabilitación de la protección de purga.

Para activar la eliminación temporal y la protección de purgas al crear un almacén, use el comando [az keyvault create](/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-create):

```azurecli
az keyvault create --name ContosoVault --resource-group ContosoRG --location westus --enable-soft-delete true --enable-purge-protection true
```

Para agregar la protección de purgas a un almacén existente (que ya tenga habilitada la eliminación temporal), use el comando [az keyvault update](/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-update):

```azurecli
az keyvault update --name ContosoVault --resource-group ContosoRG --enable-purge-protection true
```

## <a name="other-resources"></a>Otros recursos

- Para ver una introducción a la característica de eliminación temporal de Key Vault, consulte [Introducción a la eliminación temporal de Azure Key Vault](soft-delete-overview.md).
- Para ver una introducción al uso de Azure Key Vault, consulte [¿Qué es Azure Key Vault?](overview.md).

