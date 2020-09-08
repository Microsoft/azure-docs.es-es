---
title: Uso de PowerShell para administrar datos en nubes independientes de Azure
titleSuffix: Azure Storage
description: Administración del almacenamiento en las nubes de China, Alemania y de administración pública mediante Azure PowerShell.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 12/04/2019
ms.author: tamram
ms.subservice: common
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: b610a5537d110a4046bd42ac86f5c938aeafe953
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89072968"
---
# <a name="managing-storage-in-the-azure-independent-clouds-using-powershell"></a>Administración del almacenamiento en las nubes independientes mediante PowerShell

La mayoría de los usuarios utiliza la nube pública de Azure para una implementación global. También hay algunas implementaciones independientes de Microsoft Azure por motivos de soberanía, etc. Estas implementaciones independientes se conocen como "entornos". La siguiente lista detalla las nubes independientes disponibles actualmente.

* [Azure Government Cloud (Nube de Azure Government)](https://azure.microsoft.com/features/gov/)
* [Nube Azure China 21Vianet controlada por 21Vianet en China](http://www.windowsazure.cn/)
* [Nube de Alemania de Azure](../../germany/germany-welcome.md)

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="using-an-independent-cloud"></a>Uso de una nube independiente

Para usar Azure Storage en una de las nubes independientes, conéctese a ella, en lugar de a la nube pública de Azure. Para usar una de las nubes independientes en lugar de la nube pública de Azure:

* Especifique el *entorno* al que se va a conectar.
* Puede determinar y usar las regiones disponibles.
* Utilice el sufijo de punto de conexión correcto, que es diferente en el caso de la nube pública de Azure.

Los ejemplos requieren la versión 0.7 o posterior del módulo Az de Azure PowerShell. En una ventana de PowerShell, ejecute `Get-Module -ListAvailable Az` para buscar la versión. Si no aparece ninguna o necesita una actualización, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-Az-ps).

## <a name="log-in-to-azure"></a>Inicio de sesión en Azure

Ejecute el cmdlet [Get-AzEnvironment](/powershell/module/az.accounts/get-azenvironment) para ver los entornos de Azure disponibles:

```powershell
Get-AzEnvironment
```

Inicie sesión en la cuenta que tenga acceso a la nube a la que desea conectarse y establezca el entorno. Este ejemplo le muestra cómo iniciar sesión en una cuenta que utiliza la nube de administración pública de Azure.   

```powershell
Connect-AzAccount –Environment AzureUSGovernment
```

Para acceder a la nube de China, use el entorno **AzureChinaCloud**. Para acceder a la nube de Alemania, use **AzureGermanCloud**.

En este momento, si necesita la lista de ubicaciones para crear una cuenta de almacenamiento o algún otro recurso, puede consultar las ubicaciones disponibles de la nube seleccionada mediante [Get-AzLocation](/powershell/module/az.resources/get-azlocation).

```powershell
Get-AzLocation | select Location, DisplayName
```

La siguiente tabla muestra las ubicaciones devueltas para la nube de Alemania.

|Location | Display Name (Nombre para mostrar) |
|----|----|
| `germanycentral` | Centro de Alemania|
| `germanynortheast` | Nordeste de Alemania |


## <a name="endpoint-suffix"></a>Sufijo de punto de conexión

El sufijo de punto de conexión para cada uno de estos entornos es diferente del punto de conexión de la nube pública de Azure. Por ejemplo, el sufijo de punto de conexión de blobs para la nube pública de Azure es **blob.core.windows.net**. Para la nube de administración pública, el sufijo de punto de conexión de blobs es **blob.core.usgovcloudapi.net**.

### <a name="get-endpoint-using-get-azenvironment"></a>Obtención del punto de conexión mediante Get-AzEnvironment

Puede recuperar el sufijo del punto de conexión mediante [Get-AzEnvironment](/powershell/module/az.accounts/get-azenvironment). El punto de conexión es la propiedad *StorageEndpointSuffix* del entorno.

En los fragmentos de código siguientes se muestra cómo recuperar el sufijo del punto de conexión. Todos estos comandos devuelven algo parecido a "core.cloudapp.net" o "core.cloudapi.de", etc. Adjunte este sufijo al servicio de almacenamiento para acceder a ese servicio. Por ejemplo, "queue.core.cloudapi.de" accederá al servicio de cola en la nube de Alemania.

Este fragmento de código recupera todos los entornos y el sufijo de punto de conexión para cada uno de ellos.

```powershell
Get-AzEnvironment | select Name, StorageEndpointSuffix 
```

Este comando devuelve los siguientes resultados.

| Nombre| StorageEndpointSuffix|
|----|----|
| AzureChinaCloud | core.chinacloudapi.cn|
| AzureCloud | core.windows.net |
| AzureGermanCloud | core.cloudapi.de|
| AzureUSGovernment | core.usgovcloudapi.net |

Para recuperar todas las propiedades del entorno especificado, llame a **Get-AzEnvironment** y especifique el nombre de la nube. Este fragmento de código devuelve una lista de propiedades. Busque **StorageEndpointSuffix** en la lista. El ejemplo siguiente es para la nube de Alemania.

```powershell
Get-AzEnvironment -Name AzureGermanCloud
```

Los resultados son similares a los valores siguientes:

|Nombre de propiedad|Value|
|----|----|
| Nombre | `AzureGermanCloud` |
| EnableAdfsAuthentication | `False` |
| ActiveDirectoryServiceEndpointResourceI | `http://management.core.cloudapi.de/` |
| GalleryURL | `https://gallery.cloudapi.de/` |
| ManagementPortalUrl | `https://portal.microsoftazure.de/` |
| ServiceManagementUrl | `https://manage.core.cloudapi.de/` |
| PublishSettingsFileUrl| `https://manage.microsoftazure.de/publishsettings/index` |
| ResourceManagerUrl | `http://management.microsoftazure.de/` |
| SqlDatabaseDnsSuffix | `.database.cloudapi.de` |
| **StorageEndpointSuffix** | `core.cloudapi.de` |
| … | … |

Para recuperar solo la propiedad del sufijo de punto de conexión de almacenamiento, recupere la nube específica y pida solo esa propiedad.

```powershell
$environment = Get-AzEnvironment -Name AzureGermanCloud
Write-Host "Storage EndPoint Suffix = " $environment.StorageEndpointSuffix
```

Este comando devuelve la siguiente información:

`Storage Endpoint Suffix = core.cloudapi.de`

### <a name="get-endpoint-from-a-storage-account"></a>Obtención del punto de conexión desde una cuenta de almacenamiento

También puede examinar las propiedades de una cuenta de almacenamiento para recuperar los puntos de conexión:

```powershell
# Get a reference to the storage account.
$resourceGroup = "myexistingresourcegroup"
$storageAccountName = "myexistingstorageaccount"
$storageAccount = Get-AzStorageAccount `
  -ResourceGroupName $resourceGroup `
  -Name $storageAccountName 
  # Output the endpoints.
Write-Host "blob endpoint = " $storageAccount.PrimaryEndPoints.Blob 
Write-Host "file endpoint = " $storageAccount.PrimaryEndPoints.File
Write-Host "queue endpoint = " $storageAccount.PrimaryEndPoints.Queue
Write-Host "table endpoint = " $storageAccount.PrimaryEndPoints.Table
```

En el caso de una cuenta de almacenamiento en la nube de administración pública, este comando devolverá la siguiente salida:

```
blob endpoint = http://myexistingstorageaccount.blob.core.usgovcloudapi.net/
file endpoint = http://myexistingstorageaccount.file.core.usgovcloudapi.net/
queue endpoint = http://myexistingstorageaccount.queue.core.usgovcloudapi.net/
table endpoint = http://myexistingstorageaccount.table.core.usgovcloudapi.net/
```

## <a name="after-setting-the-environment"></a>Después de configurar el entorno

Ahora puede usar PowerShell para administrar las cuentas de almacenamiento y acceder a los datos de blobs, colas, archivos y tablas. Para más información, consulte [Az.Storage](/powershell/module/az.storage).

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ha creado un nuevo grupo de recursos y una cuenta de almacenamiento para este ejercicio, puede quitar ambos recursos eliminando el grupo de recursos. Al eliminar un grupo de recursos se eliminan todos los recursos contenidos en él.

```powershell
Remove-AzResourceGroup -Name $resourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

* [Conservación de inicios de sesión de usuario entre sesiones de PowerShell](/powershell/azure/context-persistence)
* [Almacenamiento de Azure Government](../../azure-government/documentation-government-services-storage.md)
* [Guía para desarrolladores de Microsoft Azure Government](../../azure-government/documentation-government-developer-guide.md)
* [Notas para desarrolladores para las aplicaciones de Azure China 21Vianet](https://msdn.microsoft.com/library/azure/dn578439.aspx)
* [Documentación sobre Azure Alemania](../../germany/germany-welcome.md)
