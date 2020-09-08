---
title: Administración de cuentas v3 de Azure Media Services | Microsoft Docs
description: Para iniciar la administración, el cifrado, la codificación, el análisis y el streaming de contenido multimedia en Azure, debe crear una cuenta de Media Services. En este artículo se explica cómo administrar las cuentas de Azure Media Services v3.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: 7c2cbaaf4866edc20f9745abe6759861db7834c3
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89291337"
---
# <a name="manage-azure-media-services-v3-accounts"></a>Administración de cuentas v3 de Azure Media Services

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Para iniciar la administración, el cifrado, la codificación, el análisis y el streaming de contenido multimedia en Azure, debe crear una cuenta de Media Services. Al crear una cuenta de Media Services, debe proporcionar el nombre de un recurso de cuenta de Azure Storage. La cuenta de almacenamiento especificada está asociada a su cuenta de Media Services. La cuenta de Media Services y todas las cuentas de almacenamiento asociadas deben estar en la misma suscripción de Azure. Para más información, consulte [Cuentas de almacenamiento](storage-account-concept.md).

## <a name="moving-a-media-services-account-between-subscriptions"></a>Traslado de una cuenta de Media Services entre suscripciones 

Si necesita trasladar una cuenta de Media Services a una nueva suscripción, primero debe trasladar todo el grupo de recursos que contiene la cuenta de Media Services a la nueva suscripción. Debe trasladar todos los recursos asociados: Cuentas de Azure Storage, perfiles de Azure CDN, etc. Para obtener más información, consulte [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../../azure-resource-manager/management/move-resource-group-and-subscription.md). Al igual que con los recursos de Azure, los traslados del grupo de recursos pueden tardar algún tiempo en completarse.

> [!NOTE]
> Media Services v3 admite el modelo de varios inquilinos.

### <a name="considerations"></a>Consideraciones

* Cree copias de seguridad de todos los datos en su cuenta antes de migrar a otra suscripción.
* Debe detener todos los puntos de conexión de streaming y los recursos de streaming en vivo. Los usuarios no podrán acceder a su contenido durante el traslado del grupo de recursos. 

> [!IMPORTANT]
> No inicie el punto de conexión de streaming hasta que la migración se haya completado correctamente.

### <a name="troubleshoot"></a>Solución de problemas 

Si una cuenta de Media Services o una cuenta de Azure Storage asociada se "desconecta" después del traslado del grupo de recursos, intente rotar las claves de la cuenta de almacenamiento. Si la rotación de las claves de la cuenta de almacenamiento no resuelve el estado "desconectado" de la cuenta de Media Services, presente una nueva solicitud de soporte técnico desde el menú "Soporte técnico y solución de problemas" de la cuenta de Media Services.  

## <a name="next-steps"></a>Pasos siguientes

[Crear una cuenta](./create-account-howto.md)
