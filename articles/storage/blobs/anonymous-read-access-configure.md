---
title: Configuración de acceso de lectura público anónimo a contenedores y blobs
titleSuffix: Azure Storage
description: Aprenda a permitir o no permitir el acceso anónimo a los datos de blobs en la cuenta de almacenamiento. Establezca la configuración del acceso público a contenedores para que tanto los contenedores como los blobs estén disponibles para el acceso anónimo.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 08/02/2020
ms.author: tamram
ms.reviewer: fryu
ms.openlocfilehash: 3a45f185a20345dac00bd459789afc9d53bd48f7
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87534318"
---
# <a name="configure-anonymous-public-read-access-for-containers-and-blobs"></a>Configuración de acceso de lectura público anónimo a contenedores y blobs

Azure Storage admite el acceso de lectura público anónimo opcional a contenedores y blobs. De forma predeterminada, nunca se permite el acceso anónimo a los datos. A menos que habilite explícitamente el acceso anónimo, todas las solicitudes a un contenedor y sus blobs deben estar autorizadas. Si se configura el nivel de acceso público de un contenedor para permitir el acceso anónimo, los clientes pueden leer los datos de dicho contenedor sin autorizar la solicitud.

> [!WARNING]
> Cuando se configura un contenedor para el acceso público, cualquier cliente puede leer los datos del mismo. El acceso público presenta un riesgo de seguridad potencial, por lo que si el escenario no lo requiere, Microsoft recomienda no permitirlo para la cuenta de almacenamiento. Para más información, consulte el artículo en el que se explica cómo [impedir el acceso de lectura público anónimo a contenedores y blobs](anonymous-read-access-prevent.md).

En este artículo se describe cómo configurar el acceso público anónimo de lectura para un contenedor y sus blobs. Para más información sobre cómo acceder a los datos de blobs de forma anónima desde una aplicación cliente, consulte [Acceso anónimo a contenedores y blobs públicos con .NET](anonymous-read-access-client.md).

## <a name="about-anonymous-public-read-access"></a>Acerca del acceso de lectura público anónimo

El acceso público a los datos siempre está prohibido de forma predeterminada. Hay dos configuraciones independientes que afectan al acceso público:

1. **Permitir el acceso público para la cuenta de almacenamiento.** De forma predeterminada, una cuenta de almacenamiento permite a un usuario con los permisos adecuados configurar el acceso público a un contenedor. Los datos de blobs no están disponibles para el acceso público a menos que el usuario lleve a cabo el paso adicional para configurar explícitamente la configuración de acceso público del contenedor.
1. **Configurar el acceso público del contenedor.** De forma predeterminada, la configuración de acceso público de un contenedor está deshabilitada, lo que significa que se requiere autorización para cada solicitud al contenedor o sus datos. Un usuario con los permisos adecuados puede modificar la configuración de acceso público de un contenedor para habilitar el acceso anónimo solo si se permite el acceso anónimo para la cuenta de almacenamiento.

En la tabla siguiente se resume el modo en que ambas opciones afectan a acceso público a un contenedor.

| Configuración de acceso público | El acceso público está deshabilitado para un contenedor (configuración predeterminada) | El acceso público para un contenedor está establecido en Contenedor | El acceso público para un contenedor está establecido en Blob |
|--|--|--|--|
| El acceso público no está permitido para la cuenta de almacenamiento | No hay acceso público a ningún contenedor de la cuenta de almacenamiento. | No hay acceso público a ningún contenedor de la cuenta de almacenamiento. La configuración de la cuenta de almacenamiento invalida la configuración del contenedor. | No hay acceso público a ningún contenedor de la cuenta de almacenamiento. La configuración de la cuenta de almacenamiento invalida la configuración del contenedor. |
| El acceso público está permitido para la cuenta de almacenamiento (configuración predeterminada) | No hay acceso público a este contenedor (configuración predeterminada). | Se permite el acceso público a este contenedor y sus blobs. | Se permite el acceso público a los blobs de este contenedor, pero no al propio contenedor. |

## <a name="allow-or-disallow-public-read-access-for-a-storage-account"></a>Habilitación o deshabilitación del permiso de acceso de lectura público a una cuenta de almacenamiento

De forma predeterminada, una cuenta de almacenamiento está configurada para permitir a los usuarios con los permisos adecuados configurar el acceso público a contenedores y blobs. Cuando se permite el acceso público, un usuario con los permisos adecuados puede modificar la configuración de acceso público de un contenedor para permitir el acceso público anónimo a los datos de ese contenedor. Los datos de blobs no están nunca disponibles para el acceso público a menos que el usuario lleve a cabo el paso adicional para configurar explícitamente la configuración de acceso público del contenedor.

Tenga en cuenta que el acceso público a un contenedor está siempre desactivado de forma predeterminada y se debe configurar explícitamente para permitir solicitudes anónimas. Independientemente de la configuración de la cuenta de almacenamiento, los datos nunca estarán disponibles para el acceso público a menos que un usuario con los permisos adecuados lleve a cabo este paso adicional para habilitar el acceso público en el contenedor.

No permitir el acceso público para la cuenta de almacenamiento impide el acceso anónimo a todos los contenedores y blobs de esa cuenta. Cuando no se permite el acceso público a la cuenta, no es posible configurar el acceso público para un contenedor para permitir el acceso anónimo. Para mejorar la seguridad, Microsoft recomienda no permitir el acceso público para las cuentas de almacenamiento, salvo que el escenario requiera que los usuarios accedan a los recursos de los blobs de forma anónima.

> [!IMPORTANT]
> Cuando no se permite el acceso público a una cuenta de almacenamiento, se invalida la configuración de acceso público de todos los contenedores de esta. Además, todas las solicitudes anónimas futuras a esa cuenta generarán un error. Antes de cambiar esta configuración, asegúrese de que entiende el impacto en las aplicaciones cliente que pueden tener acceso a los datos de la cuenta de almacenamiento de forma anónima. Para más información, consulte el artículo en el que se explica cómo [impedir el acceso de lectura público anónimo a contenedores y blobs](anonymous-read-access-prevent.md).

Para permitir o denegar el acceso público a una cuenta de almacenamiento, configure la propiedad **AllowBlobPublicAccess** de la cuenta. Esta propiedad está disponible para todas las cuentas de almacenamiento que se crean con el modelo de implementación de Azure Resource Manager. Para más información, consulte [Introducción a las cuentas de almacenamiento](../common/storage-account-overview.md).

La propiedad **AllowBlobPublicAccess** no se establece de forma predeterminada y no devuelve un valor hasta que se establece de forma explícita. La cuenta de almacenamiento permite el acceso público cuando el valor de la propiedad es **NULL** o **true**.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

Para permitir o no permitir el acceso público a una cuenta de almacenamiento en Azure Portal, siga estos pasos:

1. Vaya a la cuenta de almacenamiento en Azure Portal.
1. Busque la opción **Configuración** en **Configuración**.
1. Defina **Blob public access** (Acceso público a blobs) como **Habilitado** o **Deshabilitado**.

    :::image type="content" source="media/anonymous-read-access-configure/blob-public-access-portal.png" alt-text="Captura de pantalla que muestra cómo permitir o no permitir el acceso público a blobs en una cuenta":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para permitir o denegar el acceso público de una cuenta de almacenamiento con PowerShell, instale [Azure PowerShell, versión 4.4.0](https://www.powershellgallery.com/packages/Az/4.4.0) o posterior. A continuación, configure la propiedad **AllowBlobPublicAccess** para una cuenta de almacenamiento nueva o existente.

En el ejemplo siguiente se crea una cuenta de almacenamiento y se establece explícitamente la propiedad **AllowBlobPublicAccess** en **true**. A continuación, se actualiza la cuenta de almacenamiento para establecer la propiedad **AllowBlobPublicAccess** en **false**. En este ejemplo también se recupera el valor de la propiedad en cada caso. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores:

```powershell
$rgName = "<resource-group>"
$accountName = "<storage-account>"
$location = "<location>"

# Create a storage account with AllowBlobPublicAccess set to true (or null).
New-AzStorageAccount -ResourceGroupName $rgName `
    -AccountName $accountName `
    -Location $location `
    -SkuName Standard_GRS
    -AllowBlobPublicAccess $false

# Read the AllowBlobPublicAccess property for the newly created storage account.
(Get-AzStorageAccount -ResourceGroupName $rgName -Name $accountName).AllowBlobPublicAccess

# Set AllowBlobPublicAccess set to false
Set-AzStorageAccount -ResourceGroupName $rgName `
    -AccountName $accountName `
    -AllowBlobPublicAccess $false

# Read the AllowBlobPublicAccess property.
(Get-AzStorageAccount -ResourceGroupName $rgName -Name $accountName).AllowBlobPublicAccess
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para permitir o denegar el acceso público de una cuenta de almacenamiento con la CLI de Azure, instale la CLI de Azure, versión 2.9.0 o posterior. Para más información, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). A continuación, configure la propiedad **allowBlobPublicAccess** en una cuenta de almacenamiento nueva o existente.

En el ejemplo siguiente se crea una cuenta de almacenamiento y se establece explícitamente la propiedad **allowBlobPublicAccess** en **true**. A continuación, se actualiza la cuenta de almacenamiento para establecer la propiedad **allowBlobPublicAccess** en **false**. En este ejemplo también se recupera el valor de la propiedad en cada caso. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores:

```azurecli-interactive
az storage account create \
    --name <storage-account> \
    --resource-group <resource-group> \
    --kind StorageV2 \
    --location <location> \
    --allow-blob-public-access true

az storage account show \
    --name <storage-account> \
    --resource-group <resource-group> \
    --query allowBlobPublicAccess \
    --output tsv

az storage account update \
    --name <storage-account> \
    --resource-group <resource-group> \
    --allow-blob-public-access false

az storage account show \
    --name <storage-account> \
    --resource-group <resource-group> \
    --query allowBlobPublicAccess \
    --output tsv
```

# <a name="template"></a>[Plantilla](#tab/template)

Para permitir o denegar el acceso público de una cuenta de almacenamiento con una plantilla, cree una plantilla con la propiedad **AllowBlobPublicAccess** establecida en **true** o **false**. En los siguientes pasos se muestra cómo crear una plantilla en Azure Portal.

1. En Azure Portal, elija **Crear un recurso**.
1. En **Buscar en Marketplace**, escriba **implementación de plantillas** y, después, presione **ENTRAR**.
1. Elija la opción de **implementación de plantillas (implementación con plantillas personalizadas; versión preliminar)** , seleccione **Crear** y, a continuación, elija **Compilar su propia plantilla en el editor**.
1. En el editor de plantillas, pegue el siguiente código JSON para crear una cuenta nueva y establecer la propiedad **AllowBlobPublicAccess** en **true** o **false**. No olvide reemplazar los valores de los marcadores de posición entre corchetes angulares con sus propios valores.

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {},
        "variables": {
            "storageAccountName": "[concat(uniqueString(subscription().subscriptionId), 'template')]"
        },
        "resources": [
            {
            "name": "[variables('storageAccountName')]",
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-06-01",
            "location": "<location>",
            "properties": {
                "allowBlobPublicAccess": false
            },
            "dependsOn": [],
            "sku": {
              "name": "Standard_GRS"
            },
            "kind": "StorageV2",
            "tags": {}
            }
        ]
    }
    ```

1. Guarde la plantilla.
1. Especifique el parámetro del grupo de recursos y, a continuación, pulse el botón **Revisar y crear** para implementar la plantilla y crear una cuenta de almacenamiento con la propiedad **allowBlobPublicAccess** configurada.

---

> [!NOTE]
> No permitir el acceso público en una cuenta de almacenamiento no afecta a los sitios web estáticos hospedados en dicha cuenta. Al contenedor **$web** siempre se puede acceder de forma pública.
>
> Después de actualizar la configuración de acceso público de la cuenta de almacenamiento, el cambio puede tardar hasta 30 segundos en propagarse por completo.

Recuerde que permitir o denegar el acceso público al blob requiere la versión 2019-04-01 o posterior del proveedor de recursos de Azure Storage. Para obtener más información, consulte la [API de REST del proveedor de recursos de Azure Storage](/rest/api/storagerp/).

En los ejemplos de esta sección se ha mostrado cómo leer la propiedad **AllowBlobPublicAccess** de la cuenta de almacenamiento para determinar si actualmente se permite el acceso público o no. Para obtener más información sobre cómo comprobar que la configuración de acceso público de una cuenta está configurada para evitar accesos anónimos, consulte [Corregir el acceso público anónimo](anonymous-read-access-prevent.md#remediate-anonymous-public-access).

## <a name="set-the-public-access-level-for-a-container"></a>Establecimiento del nivel de acceso público en un contenedor

Para conceder a usuarios anónimos acceso de lectura a un contenedor y sus blobs, primero hay que permitir el acceso público en la cuenta de almacenamiento y, después, establecer el nivel de acceso público del contenedor. Si el acceso público no está permitido en la cuenta de almacenamiento, no podrá configurar un acceso público en un contenedor.

Cuando está permitido el acceso público para una cuenta de almacenamiento, puede configurar un contenedor con los siguientes permisos:

- **Sin acceso de lectura público:** Solo puede acceder al contenedor y a sus blobs con una solicitud autorizada. Esta opción es el valor predeterminado para todos los contenedores nuevos.
- **Acceso de lectura público solo para blobs:** Los blobs del contenedor los puede leer una solicitud anónima, pero los datos del contenedor no están disponibles de forma anónima. Los clientes anónimos no pueden enumerar los blobs dentro del contenedor.
- **Acceso de lectura público para contenedores y blobs:** Los datos de los contenedores y los blobs los puede leer una solicitud anónima, excepto la configuración de los permisos de un contenedor y los metadatos de un contenedor. Los clientes pueden enumerar los blobs del contenedor a través de una solicitud anónima, pero no pueden enumerar los contenedores que están en la cuenta de almacenamiento.

No puede cambiar el nivel de acceso público para un blob individual. El nivel de acceso público se establece en el nivel de contenedor. El nivel de acceso público del contenedor se puede establecer al crear el contenedor o al actualizar este valor de configuración en un contenedor existente.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

Para actualizar el nivel de acceso público en uno o varios contenedores desde Azure Portal, siga estos pasos:

1. Vaya a la información general de su cuenta de almacenamiento en Azure Portal.
1. En **Blob service** en la hoja de menú, seleccione **Contenedores**.
1. Seleccione los contenedores para el que quiere establecer el nivel de acceso público.
1. Use el botón **Cambiar nivel de acceso** para mostrar la configuración de acceso público.
1. Seleccione el nivel de acceso público deseado en la lista desplegable **Nivel de acceso público** y haga clic en el botón Aceptar para aplicar el cambio a los contenedores seleccionados.

    ![Captura de pantalla que muestra cómo establecer el nivel de acceso público en el portal](./media/anonymous-read-access-configure/configure-public-access-container.png)

Cuando el acceso público no está permitido en la cuenta de almacenamiento, no es posible establecer el nivel de acceso público de ningún contenedor. Si intenta establecer el nivel de acceso público del contenedor, verá que el valor está deshabilitado porque el acceso público no está permitido para la cuenta.

:::image type="content" source="media/anonymous-read-access-configure/container-public-access-blocked.png" alt-text="Captura de pantalla que muestra que el valor del nivel de acceso público del contenedor está bloqueado cuando no se permite el acceso público":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para actualizar el nivel de acceso público de uno o varios contenedores con PowerShell, llame al comando [Set-AzStorageContainerAcl](/powershell/module/az.storage/set-azstoragecontaineracl). Para autorizar esta operación, use la clave de su cuenta, una cadena de conexión o una firma de acceso compartido (SAS). La operación [Set Container ACL](/rest/api/storageservices/set-container-acl) (Establecer ACL de contenedor) que establece el nivel de acceso público del contenedor no admite la autorización con Azure AD. Para obtener más información, consulte [Permisos para llamar a operaciones de datos de blobs y colas](/rest/api/storageservices/authorize-with-azure-active-directory#permissions-for-calling-blob-and-queue-data-operations).

En el ejemplo siguiente se crea un contenedor con el acceso público deshabilitado y, a continuación, se actualiza la configuración de acceso público del contenedor para permitir el acceso anónimo al contenedor y sus blobs. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores:

```powershell
# Set variables.
$rgName = "<resource-group>"
$accountName = "<storage-account>"

# Get context object.
$storageAccount = Get-AzStorageAccount -ResourceGroupName $rgName -Name $accountName
$ctx = $storageAccount.Context

# Create a new container with public access setting set to Off.
$containerName = "<container>"
New-AzStorageContainer -Name $containerName -Permission Off -Context $ctx

# Read the container's public access setting.
Get-AzStorageContainerAcl -Container $containerName -Context $ctx

# Update the container's public access setting to Container.
Set-AzStorageContainerAcl -Container $containerName -Permission Container -Context $ctx

# Read the container's public access setting.
Get-AzStorageContainerAcl -Container $containerName -Context $ctx
```

Cuando el acceso público no está permitido en la cuenta de almacenamiento, no es posible establecer el nivel de acceso público de ningún contenedor. Si intenta establecer el nivel de acceso público del contenedor, Azure Storage devuelve un error que indica que no se permite el acceso público en la cuenta de almacenamiento.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para actualizar el nivel de acceso público de uno o varios contenedores con la CLI de Azure, llame al comando [az storage container set permission](/cli/azure/storage/container#az-storage-container-set-permission). Para autorizar esta operación, use la clave de su cuenta, una cadena de conexión o una firma de acceso compartido (SAS). La operación [Set Container ACL](/rest/api/storageservices/set-container-acl) (Establecer ACL de contenedor) que establece el nivel de acceso público del contenedor no admite la autorización con Azure AD. Para obtener más información, consulte [Permisos para llamar a operaciones de datos de blobs y colas](/rest/api/storageservices/authorize-with-azure-active-directory#permissions-for-calling-blob-and-queue-data-operations).

En el ejemplo siguiente se crea un contenedor con el acceso público deshabilitado y, a continuación, se actualiza la configuración de acceso público del contenedor para permitir el acceso anónimo al contenedor y sus blobs. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores:

```azurecli-interactive
az storage container create \
    --name <container-name> \
    --account-name <account-name> \
    --resource-group <resource-group>
    --public-access off \
    --account-key <account-key> \
    --auth-mode key

az storage container show-permission \
    --name <container-name> \
    --account-name <account-name> \
    --account-key <account-key> \
    --auth-mode key

az storage container set-permission \
    --name <container-name> \
    --account-name <account-name> \
    --public-access container \
    --account-key <account-key> \
    --auth-mode key

az storage container show-permission \
    --name <container-name> \
    --account-name <account-name> \
    --account-key <account-key> \
    --auth-mode key
```

Cuando el acceso público no está permitido en la cuenta de almacenamiento, no es posible establecer el nivel de acceso público de ningún contenedor. Si intenta establecer el nivel de acceso público del contenedor, Azure Storage devuelve un error que indica que no se permite el acceso público en la cuenta de almacenamiento.

# <a name="template"></a>[Plantilla](#tab/template)

N/D

---

## <a name="check-the-public-access-setting-for-a-set-of-containers"></a>Comprobación de la configuración del acceso público de un conjunto de contenedores

Para comprobar qué contenedores de una o varias cuentas de almacenamiento están configurados para el acceso público, puede enumerar los contenedores y consultar la configuración del acceso público. Este enfoque es una opción práctica cuando una cuenta de almacenamiento no contiene un elevado número de contenedores, o bien cuando se comprueba la configuración de un pequeño número de cuentas de almacenamiento. Sin embargo, el rendimiento puede empeorar si intenta enumerar un elevado número de contenedores.

En el ejemplo siguiente se usa PowerShell para obtener la configuración del acceso público de todos los contenedores de una cuenta de almacenamiento. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores:

```powershell
$rgName = "<resource-group>"
$accountName = "<storage-account>"

$storageAccount = Get-AzStorageAccount -ResourceGroupName $rgName -Name $accountName
$ctx = $storageAccount.Context

Get-AzStorageContainer -Context $ctx | Select Name, PublicAccess
```

## <a name="next-steps"></a>Pasos siguientes

- [Impedir el acceso de lectura público anónimo a contenedores y blobs](anonymous-read-access-prevent.md)
- [Acceso anónimo a contenedores y blobs públicos con .NET](anonymous-read-access-client.md)
- [Autorización del acceso a Azure Storage](../common/storage-auth.md)
