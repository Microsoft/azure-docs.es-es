---
title: Actualización de Media Services después de sustituir las claves de acceso de almacenamiento | Microsoft Docs
description: En este artículo se proporcionan instrucciones sobre cómo actualizar Media Services tras rotar las claves de acceso de almacenamiento.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: a892ebb0-0ea0-4fc8-b715-60347cc5c95b
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2019
ms.author: juliako
ms.reviewer: milanga;cenkdin
ms.openlocfilehash: d0a449a84bebcc0a296bde51234b0ceb128b2d23
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87000136"
---
# <a name="update-media-services-after-rolling-storage-access-keys"></a>Actualización de Media Services después de sustituir las claves de acceso de almacenamiento 

Cuando crea una nueva cuenta de Azure Media Services (AMS), también se le pide que seleccione una cuenta de Azure Storage que se usa para almacenar el contenido multimedia. Puede agregar más de una cuenta de almacenamiento a su cuenta de Media Services. En este artículo se explica cómo rotar las claves de almacenamiento. También se explica cómo agregar cuentas de almacenamiento a una cuenta multimedia. 

Para realizar las acciones descritas en este artículo, debe usar las [API de Azure Resource Manager](/rest/api/media/operations/azure-media-services-rest-api-reference) y [PowerShell](/powershell/module/az.media).  Para más información, vea [How to manage Azure resources with PowerShell and Resource Manager](../../azure-resource-manager/management/manage-resource-groups-powershell.md) (Administración de recursos de Azure con PowerShell y Resource Manager).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="overview"></a>Información general

Cuando se crea una nueva cuenta de almacenamiento, Azure genera dos claves de acceso de almacenamiento de 512 bits, que se usan para autenticar el acceso a su cuenta de almacenamiento. Para aumentar la seguridad de sus conexiones de almacenamiento, se recomienda regenerar y rotar periódicamente su claves de acceso de almacenamiento. Se proporcionan dos claves de acceso (principal y secundaria) con el fin de permitirle mantener conexiones con la cuenta de almacenamiento mediante el uso de una clave de acceso mientras regenera la otra. Este procedimiento se conoce también como "rotación de claves de acceso".

Media Services depende de una clave de almacenamiento que se le ofrece. En concreto, los localizadores que se usan para transmitir o descargar sus activos dependen de la clave de acceso de almacenamiento. Cuando se crea una cuenta de AMS, toma una dependencia en la clave de acceso de almacenamiento principal de forma predeterminada, pero, como usuario, puede actualizar la clave de almacenamiento que AMS tiene. Debe asegurarse de que Media Services conocen qué clave usar siguiendo los pasos descritos en este artículo.  

>[!NOTE]
> Si tiene varias cuentas de almacenamiento, realizaría este procedimiento con cada una. El orden en que se rotan las claves de almacenamiento no es fijo. Puede rotar primero la clave secundaria y después la primaria, o viceversa.
>
> Antes de ejecutar los pasos que se describen en este artículo en una cuenta de producción, asegúrese de probarlos en una cuenta de ensayo.
>

## <a name="steps-to-rotate-storage-keys"></a>Pasos para rotar claves de almacenamiento 
 
 1. Cambie la clave principal de la cuenta de almacenamiento mediante el cmdlet de PowerShell o en [Azure](https://portal.azure.com/) Portal.
 2. Llame al cmdlet Sync-AzMediaServiceStorageKeys con los parámetros adecuados para forzar que la cuenta multimedia seleccione las claves de la cuenta de almacenamiento.
 
    En el ejemplo siguiente se muestra cómo sincronizar las claves para las cuentas de almacenamiento.
  
    `Sync-AzMediaServiceStorageKeys -ResourceGroupName $resourceGroupName -AccountName $mediaAccountName -StorageAccountId $storageAccountId`
  
 3. Espere una hora más o menos. Verifique que los escenarios de streaming estén funcionando.
 4. Cambie la clave secundaria de la cuenta de almacenamiento con el cmdlet de PowerShell o en Azure Portal.
 5. Llame al cmdlet Sync-AzMediaServiceStorageKeys de PowerShell con los parámetros adecuados para forzar que la cuenta multimedia seleccione las nuevas claves de la cuenta de almacenamiento. 
 6. Espere una hora más o menos. Verifique que los escenarios de streaming estén funcionando.
 
### <a name="a-powershell-cmdlet-example"></a>Un ejemplos de cmdlet de PowerShell 

En el ejemplo siguiente se muestra cómo obtener la cuenta de almacenamiento y sincronizarla con la cuenta de AMS.

```console
$regionName = "West US"
$resourceGroupName = "SkyMedia-USWest-App"
$mediaAccountName = "sky"
$storageAccountName = "skystorage"
$storageAccountId = "/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.Storage/storageAccounts/$storageAccountName"

Sync-AzMediaServiceStorageKeys -ResourceGroupName $resourceGroupName -AccountName $mediaAccountName -StorageAccountId $storageAccountId
```
 
## <a name="steps-to-add-storage-accounts-to-your-ams-account"></a>Pasos para agregar cuentas de almacenamiento a la cuenta de AMS

En el siguiente artículo se muestra cómo agregar cuentas de almacenamiento a la cuenta de AMS: [Asociar varias cuentas de almacenamiento a una cuenta de Media Services](./media-services-managing-multiple-storage-accounts.md).

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

### <a name="acknowledgments"></a>Agradecimientos
Nos gustaría mencionar a las siguientes personas que han contribuido a crear este documento: Cenk Dingiloglu, Milan Gada y Seva Titov.
