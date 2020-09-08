---
title: Creación de un recurso compartido de archivos de Azure
titleSuffix: Azure Files
description: Creación de un recurso compartido de archivos de Azure mediante Azure Portal, PowerShell y la CLI de Azure.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 2/22/2020
ms.author: rogarana
ms.subservice: files
ms.custom: devx-track-azurecli, references_regions
ms.openlocfilehash: 236134887728ebc3dd4d03fa4c9d9d450b39eac2
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88930679"
---
# <a name="create-an-azure-file-share"></a>Creación de un recurso compartido de archivos de Azure
Para crear un recurso compartido de archivos de Azure, debe responder a tres preguntas sobre cómo lo usará:

- **¿Cuáles son los requisitos de rendimiento para el recurso compartido de archivos de Azure?**  
    Azure Files ofrece recursos compartidos de archivos estándar, que se hospedan en hardware basado en disco duro (HDD), y recursos compartidos de archivos prémium, que se hospedan en hardware basado en disco de estado sólido (SSD).

- **¿Qué tamaño de recurso compartido de archivos necesita?**  
    Los recursos compartidos de archivos estándar pueden abarcar hasta 100 TiB, pero esta característica no está habilitada de forma predeterminada. Si necesita un recurso compartido de archivos de más de 5 TiB, deberá habilitar la característica de recurso compartido de archivos de gran tamaño para su cuenta de almacenamiento. Los recursos compartidos de archivos prémium pueden abarcar hasta 100 TiB sin ninguna configuración especial; sin embargo, los recursos compartidos de archivos prémium se aprovisionan en lugar de pagarse por uso como sucede con los recursos compartidos de archivos estándar. Esto significa que el aprovisionamiento de un recurso compartido de archivos mucho más grande de lo necesario aumentará el costo total del almacenamiento.

- **¿Cuáles son los requisitos de redundancia para el recurso compartido de archivos de Azure?**  
    Los recursos compartidos de archivos estándar ofrecen almacenamiento con redundancia local (LRS), redundancia de zona (ZRS), redundancia geográfica (GRS) o redundancia de zona geográfica (GZRS). Sin embargo, la característica de recursos compartido de archivos de gran tamaño solo se admite en los recursos compartidos de archivos con redundancia local y redundancia de zona. Los recursos compartidos de archivos prémium no admiten ninguna forma de redundancia geográfica.

    Los recursos compartidos de archivos prémium están disponibles con redundancia local en la mayoría de las regiones que ofrecen cuentas de almacenamiento y con redundancia de zona en un subconjunto de regiones más pequeño. Para averiguar si los recursos compartidos de archivos prémium están disponibles actualmente en su región, consulte la página [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=storage) para Azure. Para obtener información sobre las regiones que admiten ZRS, consulte [Redundancia de Azure Storage](../common/storage-redundancy.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

Para más información sobre estas tres opciones, consulte [Planeamiento de una implementación de Azure Files](storage-files-planning.md).

## <a name="prerequisites"></a>Prerrequisitos
- En este artículo se supone que ya ha creado una suscripción a Azure. Si todavía no tiene una suscripción, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.
- Si planea usar Azure PowerShell, [instale la versión más reciente](https://docs.microsoft.com/powershell/azure/install-az-ps).
- Si planea usar la CLI de Azure, [instale la versión más reciente](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest).

## <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento
Los recursos compartidos de archivos de Azure se implementan en *cuentas de almacenamiento*, que son objetos de nivel superior que representan un grupo compartido de almacenamiento. Este grupo de almacenamiento se puede usar para implementar varios recursos compartidos de archivos. 

Azure admite varios tipos de cuentas de almacenamiento para los distintos escenarios de almacenamiento que pueden tener los clientes, pero hay dos tipos principales de cuentas de almacenamiento para Azure Files. El tipo de cuenta de almacenamiento que debe crear depende de si desea crear un recurso compartido de archivos estándar o un recurso compartido de archivos prémium: 

- **Cuentas de almacenamiento de uso general, versión 2 (GPv2)** : Las cuentas de almacenamiento de GPv2 permiten implementar recursos compartidos de archivos de Azure en hardware estándar o basado en disco duro (HDD). Además de almacenar recursos compartidos de archivos de Azure, las cuentas de almacenamiento de GPv2 pueden almacenar otros recursos de almacenamiento, como contenedores de blobs, colas o tablas. 

- **Cuentas de almacenamiento FileStorage**: Las cuentas de almacenamiento FileStorage permiten implementar recursos compartidos de archivos de Azure en hardware prémium o basado en unidades de estado sólido (SSD). Las cuentas FileStorage solo se pueden usar para almacenar recursos compartidos de archivos de Azure. No se puede implementar ningún otro recurso de almacenamiento (contenedores de blobs, colas, tablas, etc.) en una cuenta FileStorage.

# <a name="portal"></a>[Portal](#tab/azure-portal)
Para crear una cuenta de almacenamiento mediante Azure Portal, seleccione **+ Crear un recurso** en el panel. En la ventana de búsqueda de Azure Marketplace que aparece, busque **cuenta de almacenamiento** y seleccione el resultado de la búsqueda resultante. Esta operación le dirigirá a una página de información general de las cuentas de almacenamiento. Seleccione **Crear** para continuar con el Asistente para crear cuentas de almacenamiento.

![Captura de pantalla de la opción creación rápida de una cuenta de almacenamiento en un explorador](media/storage-how-to-create-file-share/create-storage-account-0.png)

#### <a name="the-basics-section"></a>Sección Datos básicos
La primera sección que se debe rellenar para crear una cuenta de almacenamiento se denomina **Datos básicos**. Contiene todos los campos obligatorios para crear una cuenta de almacenamiento. Para crear una cuenta de almacenamiento de GPv2, asegúrese de que el botón de radio **Rendimiento** está establecido en *Estándar* y que la lista desplegable **Tipo de cuenta** tiene seleccionada la opción *StorageV2 (general purpose v2)* StorageV2 (uso general V2).

![Captura de pantalla del botón de radio Rendimiento con la opción Estándar seleccionada y Tipo de cuenta con la opción StorageV2 seleccionada](media/storage-how-to-create-file-share/create-storage-account-1.png)

Para crear una cuenta de almacenamiento FileStorage, asegúrese de que el botón de radio **Rendimiento** está establecido en *Premium* y que la lista desplegable **Tipo de cuenta** tiene seleccionada la opción *FileStorage*.

![Captura de pantalla del botón de radio Rendimiento con la opción Premium seleccionada y Tipo de cuenta con la opción FileStorage seleccionada](media/storage-how-to-create-file-share/create-storage-account-2.png)

Los demás campos de datos básicos son independientes de la elección de la cuenta de almacenamiento:
- **Suscripción**: suscripción de la cuenta de almacenamiento donde se va a realizar la implementación. 
- **Grupo de recursos**: grupo de recursos de la cuenta de almacenamiento donde se va a realizar la implementación. Puede crear un grupo de recursos nuevo o seleccionar uno existente. Un grupo de recursos es un contenedor lógico para agrupar servicios de Azure. Al crear una cuenta de almacenamiento, puede elegir entre crear un grupo de recursos o usar uno existente.
- **Nombre de la cuenta de almacenamiento**: nombre del recurso de la cuenta de almacenamiento que se va a crear. Este nombre debe ser único globalmente, pero puede ser cualquier nombre que desee. El nombre de la cuenta de almacenamiento se usará como el nombre del servidor al montar un recurso compartido de archivos de Azure a través de SMB.
- **Ubicación**: región de la cuenta de almacenamiento donde se va a realizar la implementación. Puede ser la región asociada al grupo de recursos o cualquier otra región disponible.
- **Replication** (Replicación): aunque se trata de la replicación etiquetada, este campo significa **redundancia** en realidad; este es el nivel de redundancia deseado: redundancia local (LRS), redundancia de zona (ZRS), redundancia geográfica (GRS) y redundancia de zona geográfica. Esta lista desplegable también contiene redundancia geográfica con acceso de lectura (RA-GRS) y redundancia de zona geográfica con acceso de lectura (RA-GZRS), que no se aplican a los recursos compartidos de archivos de Azure. Los recursos compartidos de archivos creados en una cuenta de almacenamiento con estas opciones seleccionadas tendrán redundancia geográfica o redundancia de zona geográfica, respectivamente. En función de la región o el tipo de cuenta de almacenamiento seleccionado, es posible que algunas opciones de redundancia no se permitan.
- **Nivel de acceso**: este campo no se aplica a Azure Files, por lo que puede elegir uno de los botones de radio.

#### <a name="the-networking-blade"></a>Hoja Redes
La sección Redes le permite configurar las opciones de redes. Esta configuración es opcional para la creación de la cuenta de almacenamiento y se puede configurar más adelante si lo desea. Para obtener más información sobre estas opciones, vea [Consideraciones de redes para Azure Files](storage-files-networking-overview.md).

#### <a name="the-advanced-blade"></a>Hoja Opciones avanzadas
La sección Opciones avanzadas contiene varias opciones de configuración importantes para los recursos compartidos de archivos de Azure:

- **Se requiere transferencia segura**: este campo indica si la cuenta de almacenamiento requiere cifrado en tránsito para la comunicación con la cuenta de almacenamiento. Se recomienda que esta opción esté habilitada; sin embargo, si necesita compatibilidad con SMB 2.1, debe deshabilitarla. Si se deshabilita el cifrado, se recomienda restringir el acceso de la cuenta de almacenamiento a una red virtual con puntos de conexión de servicio o puntos de conexión privados.
- **Recursos compartidos de archivos grandes**: este campo habilita la cuenta de almacenamiento para los recursos compartidos de archivos de hasta 100 TiB. Al habilitar esta característica, se limita la cuenta de almacenamiento a las opciones de almacenamiento con redundancia local y redundancia de zona solamente. Una vez habilitada una cuenta de almacenamiento de GPv2 para los recursos compartidos de archivos grandes, no se puede deshabilitar la funcionalidad de recurso compartido de archivos grande. Las cuentas de almacenamiento de FileStorage (cuentas de almacenamiento para recursos compartidos de archivos prémium) no tienen esta opción, ya que todos los recursos compartidos de archivos prémium se pueden escalar hasta 100 TiB. 

![Captura de pantalla de la configuración avanzada importante que se aplica a Azure Files](media/storage-how-to-create-file-share/create-storage-account-3.png)

Los demás valores de configuración que están disponibles en la pestaña Opciones avanzadas (eliminación temporal de blobs, espacio de nombres jerárquico para Azure Data Lake Storage Gen 2 y NFSv3 para Blob Storage) no se aplican a Azure Files.

#### <a name="tags"></a>Etiquetas
Las etiquetas son pares nombre-valor que permiten categorizar los recursos y ver una facturación consolidada mediante la aplicación de la misma etiqueta en varios recursos y grupos de recursos. Son opcionales y se pueden aplicar después de la creación de la cuenta de almacenamiento.

#### <a name="review--create"></a>Revisar y crear
El paso final para crear la cuenta de almacenamiento es seleccionar el botón **Crear** en la pestaña **Revisar y crear**. Este botón no estará disponible si no se rellenan todos los campos obligatorios para una cuenta de almacenamiento.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Para crear una cuenta de almacenamiento mediante PowerShell, usaremos el cmdlet `New-AzStorageAccount`. Este cmdlet tiene muchas opciones (solo se muestran las necesarias). Para obtener más información sobre las opciones avanzadas, consulte la [documentación del cmdlet `New-AzStorageAccount`](/powershell/module/az.storage/new-azstorageaccount).

Para simplificar la creación de la cuenta de almacenamiento y el recurso compartido de archivos subsiguiente, almacenaremos varios parámetros en variables. Puede reemplazar el contenido de la variable por los valores que desee, pero tenga en cuenta que el nombre de la cuenta de almacenamiento debe ser único globalmente.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$storageAccountName = "mystorageacct$(Get-Random)"
$region = "westus2"
```

Para crear una cuenta de almacenamiento capaz de almacenar recursos compartidos de archivos de Azure estándar, usaremos el siguiente comando. El parámetro `-SkuName` está relacionado con el tipo de redundancia deseado; si desea una cuenta de almacenamiento con redundancia geográfica o con redundancia de zona geográfica, también debe quitar el parámetro `-EnableLargeFileShare`.

```azurepowershell-interactive
$storAcct = New-AzStorageAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $storageAccountName `
    -SkuName Standard_LRS `
    -Location $region `
    -Kind StorageV2 `
    -EnableLargeFileShare
```

Para crear una cuenta de almacenamiento capaz de almacenar recursos compartidos de archivos de Azure prémium, usaremos el siguiente comando. Observe que el parámetro `-SkuName` ha cambiado para incluir tanto `Premium` como el nivel de redundancia deseado de redundancia local (`LRS`). El parámetro `-Kind` es `FileStorage` en lugar de `StorageV2` porque los recursos compartidos de archivos prémium deben crearse en una cuenta de almacenamiento de FileStorage en lugar de en una cuenta de almacenamiento de GPv2.

```azurepowershell-interactive
$storAcct = New-AzStorageAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $storageAccountName `
    -SkuName Premium_LRS `
    -Location $region `
    -Kind FileStorage 
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
Para crear una cuenta de almacenamiento con la CLI de Azure, usaremos el comando az storage account create. Este comando tiene muchas opciones (solo se muestran las necesarias). Para obtener más información sobre las opciones avanzadas, consulte la [documentación del comando `az storage account create`](/cli/azure/storage/account).

Para simplificar la creación de la cuenta de almacenamiento y el recurso compartido de archivos subsiguiente, almacenaremos varios parámetros en variables. Puede reemplazar el contenido de la variable por los valores que desee, pero tenga en cuenta que el nombre de la cuenta de almacenamiento debe ser único globalmente.

```azurecli-interactive
resourceGroupName="myResourceGroup"
storageAccountName="mystorageacct$RANDOM"
region="westus2"
```

Para crear una cuenta de almacenamiento capaz de almacenar recursos compartidos de archivos de Azure estándar, usaremos el siguiente comando. El parámetro `--sku` está relacionado con el tipo de redundancia deseado; si desea una cuenta de almacenamiento con redundancia geográfica o con redundancia de zona geográfica, también debe quitar el parámetro `--enable-large-file-share`.

```azurecli-interactive
az storage account create \
    --resource-group $resourceGroupName \
    --name $storageAccountName \
    --kind StorageV2 \
    --sku Standard_ZRS \
    --enable-large-file-share \
    --output none
```

Para crear una cuenta de almacenamiento capaz de almacenar recursos compartidos de archivos de Azure prémium, usaremos el siguiente comando. Observe que el parámetro `--sku` ha cambiado para incluir tanto `Premium` como el nivel de redundancia deseado de redundancia local (`LRS`). El parámetro `--kind` es `FileStorage` en lugar de `StorageV2` porque los recursos compartidos de archivos prémium deben crearse en una cuenta de almacenamiento de FileStorage en lugar de en una cuenta de almacenamiento de GPv2.

```azurecli-interactive
az storage account create \
    --resource-group $resourceGroupName \
    --name $storageAccountName \
    --kind FileStorage \
    --sku Premium_LRS \
    --output none
```

---

## <a name="create-file-share"></a>Creación de un recurso compartido de archivos
Una vez que haya creado su cuenta de almacenamiento, todo lo que queda es crear el recurso compartido de archivos. Este proceso es prácticamente el mismo, independientemente de si usa un recurso compartido de archivos prémium o un recurso compartido de archivos estándar. La diferencia principal es la **cuota** y lo que representa.

En el caso de los recursos compartidos de archivos estándar, es un límite superior del recurso compartido de archivos de Azure, que los usuarios finales no pueden superar. El objetivo principal de la cuota para un recurso compartido de archivos estándar es presupuestario: "no quiero que este recurso compartido de archivos crezca más allá de este punto". Si no se especifica una cuota, el recurso compartido de archivos estándar puede abarcar hasta 100 TiB (o 5 TiB si no se establece la propiedad de recursos compartidos de archivos grandes para una cuenta de almacenamiento).

En el caso de recursos compartidos de archivos prémium, la cuota se sobrecarga para indicar **tamaño aprovisionado**. El tamaño aprovisionado es la cantidad que se facturará, independientemente del uso real. Al aprovisionar un recurso compartido de archivos prémium, se deben tener en cuenta dos factores: 1) el crecimiento futuro del recurso compartido desde una perspectiva de uso del espacio y 2) las IOPS necesarias para la carga de trabajo. Cada GiB aprovisionado le da derecho a IOPS reservadas y de ráfaga adicionales. Para obtener más información sobre cómo planear un recurso compartido de archivos prémium, consulte el tema sobre el [aprovisionamiento de recursos compartidos de archivos prémium](storage-files-planning.md#understanding-provisioning-for-premium-file-shares).

# <a name="portal"></a>[Portal](#tab/azure-portal)
Si acaba de crear la cuenta de almacenamiento, puede navegar a esta desde la pantalla de implementación. Para ello, seleccione **Ir al recurso**. Si ya ha creado la cuenta de almacenamiento anteriormente, puede navegar a esta a través del grupo de recursos que la contiene. Una vez en la cuenta de almacenamiento, seleccione el icono con la etiqueta **Recursos compartidos de archivos** (también puede navegar a **Recursos compartidos de archivos** a través de la tabla de contenido de la cuenta de almacenamiento).

![Captura de pantalla del icono Recursos compartidos de archivos](media/storage-how-to-create-file-share/create-file-share-1.png)

En la lista de recursos compartidos de archivos, debería ver los recursos compartidos de archivos creados previamente en esta cuenta de almacenamiento; se muestra una tabla vacía si aún no se han creado recursos compartidos de archivos. Seleccione **+ Recurso compartido de archivos** para crear un recurso compartido de archivos.

La hoja Nuevo recurso compartido de archivos debería aparecer en la pantalla. Complete los campos de la hoja Nuevo recurso compartido de archivos para crear un recurso compartido de archivos:

- **Nombre**: nombre del recurso compartido de archivos que se va a crear.
- **Cuota**: cuota del recurso compartido de archivos para los recursos compartidos de archivos estándar; tamaño aprovisionado del recurso compartido de archivos para los recursos compartidos de archivos prémium.

Seleccione **Crear** para terminar de crear el nuevo recurso compartido de archivos. Tenga en cuenta que si la cuenta de almacenamiento está en una red virtual, no podrá crear correctamente un recurso compartido de archivos de Azure, a menos que el cliente también esté en la red virtual. También puede resolver esta limitación puntual mediante el cmdlet `New-AzRmStorageShare` de Azure PowerShell.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Puede crear el recurso compartido de archivos de Azure con el cmdlet de [`New-AzRmStorageShare`](/powershell/module/az.storage/New-AzRmStorageShare). Los siguientes comandos de PowerShell suponen que ha establecido las variables `$resourceGroupName` y `$storageAccountName` como se ha definido anteriormente en la sección sobre la creación de una cuenta de almacenamiento con Azure PowerShell. 

> [!Important]  
> En el caso de los recursos compartidos de archivos prémium, el parámetro `-QuotaGiB` hace referencia al tamaño aprovisionado del recurso compartido de archivos. El tamaño aprovisionado del recurso compartido de archivos es la cantidad que se facturará, independientemente del uso. Los recursos compartidos de archivos estándar se facturan en función del uso en lugar de basarse en el tamaño aprovisionado.

```azurepowershell-interactive
$shareName = "myshare"

New-AzRmStorageShare `
    -ResourceGroupName $resourceGroupName `
    -StorageAccountName $storageAccountName `
    -Name $shareName `
    -QuotaGiB 1024 | Out-Null
```

> [!Note]  
> El nombre del recurso compartido de archivos debe estar en minúsculas. Para obtener detalles completos sobre cómo asignar un nombre a recursos compartidos y archivos, consulte  [Asignación de nombres y referencia a recursos compartidos, directorios, archivos y metadatos](https://msdn.microsoft.com/library/azure/dn167011.aspx).

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
Para poder crear un recurso compartido de archivos de Azure con la CLI de Azure, debe obtener una clave de cuenta de almacenamiento para autorizar la operación de creación del recurso compartido de archivos. Esto puede hacerse con el comando [`az storage account keys list`](/cli/azure/storage/account/keys):

```azurecli-interactive
storageAccountKey=$(az storage account keys list \
    --resource-group $resourceGroupName \
    --account-name $storageAccountName \
    --query "[0].value" | tr -d '"')
```

Una vez que tenga la clave de la cuenta de almacenamiento, puede crear el recurso compartido de archivos de Azure con el comando de [`az storage share create`](/cli/azure/storage/share). 

> [!Important]  
> En el caso de los recursos compartidos de archivos prémium, el parámetro `--quota` hace referencia al tamaño aprovisionado del recurso compartido de archivos. El tamaño aprovisionado del recurso compartido de archivos es la cantidad que se facturará, independientemente del uso. Los recursos compartidos de archivos estándar se facturan en función del uso en lugar de basarse en el tamaño aprovisionado.

```azurecli-interactive
shareName="myshare"

az storage share create \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --name $shareName \
    --quota 1024 \
    --output none
```

Se producirá un error en este comando si la cuenta de almacenamiento está incluida en una red virtual y el equipo desde el que lo invoca no forma parte de dicha red virtual. Para solucionar esta limitación puntual puede usar el cmdlet `New-AzRmStorageShare` de Azure PowerShell como se describió anteriormente, o ejecutar la CLI de Azure desde un equipo que forme parte de la red virtual, incluso a través de una conexión VPN.

---

> [!Note]  
> El nombre del recurso compartido de archivos debe estar en minúsculas. Para obtener detalles completos sobre cómo asignar un nombre a recursos compartidos y archivos, consulte  [Asignación de nombres y referencia a recursos compartidos, directorios, archivos y metadatos](https://msdn.microsoft.com/library/azure/dn167011.aspx).

### <a name="create-a-hot-or-cool-file-share"></a>Creación de un recurso compartido de archivos de acceso frecuente o esporádico
Un recurso compartido de archivos de una **cuenta de almacenamiento de uso general v2 (GPv2)** puede contener recursos compartidos de archivos de acceso frecuente o esporádico u optimizados para transacciones (o una combinación de ellos). Los recursos compartidos optimizados para transacciones están disponibles en todas las regiones de Azure, pero los recursos compartidos de archivos de acceso frecuente y esporádico solo están disponibles [en un subconjunto de regiones](storage-files-planning.md#storage-tiers). Puede crear un recurso compartido de archivos de acceso frecuente o esporádico mediante el módulo en versión preliminar de Azure PowerShell o la CLI de Azure. 

# <a name="portal"></a>[Portal](#tab/azure-portal)
Azure Portal no admite todavía la creación de recursos compartidos de archivos de acceso frecuente y esporádico, o el traslado de los recursos compartidos de archivos optimizados para transacciones existentes a acceso frecuente o esporádico. Consulte las instrucciones para crear un recurso compartido de archivos con PowerShell o la CLI de Azure.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
```PowerShell
# Update the Azure storage module to use the preview version. You may need to close and 
# reopen PowerShell before running this command. If you are running PowerShell 5.1, ensure 
# the following:
# - Run the below cmdlets as an administrator.
# - Have PowerShellGet 2.2.3 or later. Uncomment the following line to check.
# Get-Module -ListAvailable -Name PowerShellGet
Remove-Module -Name Az.Storage -ErrorAction SilentlyContinue
Uninstall-Module -Name Az.Storage
Install-Module -Name Az.Storage -RequiredVersion "2.1.1-preview" -AllowClobber -AllowPrerelease 

# Assuming $resourceGroupName and $storageAccountName from earlier in this document have already
# been populated. The access tier parameter may be TransactionOptimized, Hot, or Cool for GPv2 
# storage accounts. Standard tiers are only available in standard storage accounts. 
$shareName = "myhotshare"

New-AzRmStorageShare `
    -ResourceGroupName $resourceGroupName `
    -StorageAccountName $storageAccountName `
    -Name $shareName `
    -AccessTier Hot

# You can also change an existing share's tier.
Update-AzRmStorageShare `
    -ResourceGroupName $resourceGroupName `
    -StorageAccountName $storageAccountName `
    -Name $shareName `
    -AccessTier Cool
```

> [!Note]  
> La capacidad de establecer y cambiar los niveles mediante PowerShell se proporciona en la versión preliminar del módulo Az.Storage de PowerShell. Puede que estos cmdlets o sus salidas cambien antes de su publicación en la versión con disponibilidad general del módulo Az.Storage de PowerShell, por lo que debe crear scripts con esto en mente.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
La funcionalidad para crear o mover un recurso compartido de archivos a un nivel específico está disponible en la actualización más reciente de la CLI de Azure. La actualización de la CLI de Azure es específica de la distribución de Linux o del sistema operativo que está usando. Para obtener instrucciones sobre cómo actualizar la CLI de Azure en el sistema, vea [Instalación de la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli).

```bash
# Assuming $resourceGroupName and $storageAccountName from earlier in this document have already
# been populated. The access tier parameter may be TransactionOptimized, Hot, or Cool for GPv2
# storage accounts. Standard tiers are only available in standard storage accounts.
shareName="myhotshare"

az storage share-rm create \
    --resource-group $resourceGroupName \
    --storage-account $storageAccountName \
    --name $shareName \
    --access-tier "Hot"
```

> [!Note]  
> La capacidad de establecer un nivel con el parámetro `--access-tier` se proporciona en una versión preliminar del último paquete de la CLI de Azure. Este comando o su salida pueden cambiar antes de pasar a tener disponibilidad general, por lo que debe crear scripts con esto en mente.

---

## <a name="next-steps"></a>Pasos siguientes
- [Planeamiento de una implementación de Azure Files](storage-files-planning.md) o [una implementación de Azure File Sync](storage-sync-files-planning.md). 
- [Introducción a las redes](storage-files-networking-overview.md).
- Conexión y montaje de un recurso compartido de archivos en [Windows](storage-how-to-use-files-windows.md), [macOS](storage-how-to-use-files-mac.md) y [Linux](storage-how-to-use-files-linux.md).