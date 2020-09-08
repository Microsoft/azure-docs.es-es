---
title: Comprobación de la propiedad Hora de la última sincronización de una cuenta de almacenamiento
titleSuffix: Azure Storage
description: Aprenda cómo comprobar la propiedad Hora de la última sincronización para una cuenta de almacenamiento con replicación geográfica. La propiedad Hora de la última sincronización indica la hora más reciente en que todas las operaciones de escritura de la región primaria se escribieron correctamente en la secundaria.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 05/28/2020
ms.author: tamram
ms.reviewer: artek
ms.subservice: common
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: cad80a554db0424ad8635e81cbfb4225705e9c89
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89073019"
---
# <a name="check-the-last-sync-time-property-for-a-storage-account"></a>Comprobación de la propiedad Hora de la última sincronización de una cuenta de almacenamiento

Cuando se configura una cuenta de almacenamiento, se puede definir que los datos se copien en una región secundaria que se encuentra a cientos de kilómetros de la región primaria. La replicación geográfica proporciona durabilidad para los datos en caso de que se produzca una interrupción importante en la región primaria, como un desastre natural. Si, además, habilita el acceso de lectura a la región secundaria, los datos seguirán disponibles para las operaciones de lectura si la región primaria deja de estar disponible. Puede diseñar la aplicación para que cambie sin problemas a la lectura desde la región secundaria si la región primaria no responde.

Tanto el almacenamiento con redundancia geográfica (GRS) como el almacenamiento con redundancia de zona geográfica (GZRS) replican los datos de forma asincrónica en una región secundaria. Para obtener acceso de lectura a la región secundaria, habilite el almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS) o el almacenamiento con redundancia de zona geográfica con acceso de lectura (RA-GZRS). Para obtener más información sobre las diversas opciones de redundancia que se ofrecen en Azure Storage, consulte [Redundancia de Azure Storage](storage-redundancy.md).

En este artículo se describe cómo comprobar la propiedad **Hora de la última sincronización** de la cuenta de almacenamiento para que pueda evaluar cualquier discrepancia entre las regiones primaria y secundaria.

## <a name="about-the-last-sync-time-property"></a>Acerca de la propiedad Hora de la última sincronización

Dado que la replicación geográfica es asincrónica, es posible que los datos escritos en la región primaria no se hayan escrito aún en la región secundaria en el momento en que se produce un apagón. La propiedad **Hora de la última sincronización** indica la hora más reciente en que los datos de la región primaria se escribieron correctamente en la secundaria. Todas las operaciones de escritura realizadas en la región primaria antes de la hora de la última sincronización pueden leerse desde la ubicación secundaria. Las operaciones de escritura realizadas en la región primaria después de la propiedad Hora de la última sincronización pueden o no estar todavía disponibles para operaciones de lectura.

La propiedad **Hora de la última sincronización** es un valor de fecha y hora GMT.

## <a name="get-the-last-sync-time-property"></a>Obtención de la propiedad Hora de la última sincronización

Puede usar PowerShell o la CLI de Azure para recuperar el valor de la propiedad **Hora de la última sincronización**.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para obtener la hora de la última sincronización de la cuenta de almacenamiento con PowerShell, instale la versión 1.11.0 o posterior del módulo [Az.Storage](https://www.powershellgallery.com/packages/Az.Storage). A continuación, compruebe la propiedad **GeoReplicationStats.LastSyncTime** de la cuenta de almacenamiento. Recuerde reemplazar los valores de marcador de posición por los propios:

```powershell
$lastSyncTime = $(Get-AzStorageAccount -ResourceGroupName <resource-group> `
    -Name <storage-account> `
    -IncludeGeoReplicationStats).GeoReplicationStats.LastSyncTime
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para obtener la hora de la última sincronización para la cuenta de almacenamiento con la CLI de Azure, compruebe la propiedad **geoReplicationStats.lastSyncTime** de la cuenta de almacenamiento. Use el parámetro `--expand` para devolver valores para las propiedades anidadas bajo **geoReplicationStats**. Recuerde reemplazar los valores de marcador de posición por los propios:

```azurecli-interactive
$lastSyncTime=$(az storage account show \
    --name <storage-account> \
    --resource-group <resource-group> \
    --expand geoReplicationStats \
    --query geoReplicationStats.lastSyncTime \
    --output tsv)
```

---

## <a name="see-also"></a>Consulte también

- [Redundancia de Azure Storage](storage-redundancy.md)
- [Cambio de la opción de redundancia para una cuenta de almacenamiento](redundancy-migration.md)
- [Uso de redundancia geográfica para diseñar aplicaciones de alta disponibilidad](geo-redundant-design.md)
