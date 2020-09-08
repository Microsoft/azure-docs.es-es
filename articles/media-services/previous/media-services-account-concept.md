---
title: Administración de cuentas v2 de Azure Media Services | Microsoft Docs
description: Para iniciar la administración, el cifrado, la codificación, el análisis y el streaming de contenido multimedia en Azure, debe crear una cuenta de Media Services. En este artículo se explica cómo administrar las cuentas de Azure Media Services v2.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 07/05/2019
ms.author: juliako
ms.openlocfilehash: 167028cda6962445373b22c5f3d0a9e56c3e608a
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89255417"
---
# <a name="manage-azure-media-services-v2-accounts"></a>Administración de cuentas v2 de Azure Media Services

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Para iniciar la administración, el cifrado, la codificación, el análisis y el streaming de contenido multimedia en Azure, debe crear una cuenta de Media Services. Al crear una cuenta de Media Services, debe proporcionar el nombre de un recurso de cuenta de Azure Storage. La cuenta de almacenamiento especificada está asociada a su cuenta de Media Services. La cuenta de Media Services y todas las cuentas de almacenamiento asociadas deben estar en la misma suscripción de Azure.  

## <a name="moving-a-media-services-account-between-subscriptions"></a>Traslado de una cuenta de Media Services entre suscripciones

Si necesita trasladar una cuenta de Media Services a una nueva suscripción, primero debe trasladar todo el grupo de recursos que contiene la cuenta de Media Services a la nueva suscripción. Debe trasladar todos los recursos asociados: Cuentas de Azure Storage, perfiles de Azure CDN, etc. Para obtener más información, consulte [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../../azure-resource-manager/management/move-resource-group-and-subscription.md). Al igual que con los recursos de Azure, los traslados del grupo de recursos pueden tardar algún tiempo en completarse.

Media Services v2 no es compatible con el modelo multiinquilino. Si tiene que mover una cuenta de Media Services a una suscripción en un nuevo inquilino, cree una nueva aplicación de Azure Active Directory (Azure AD) en el nuevo inquilino. A continuación, traslade la cuenta a la suscripción del nuevo inquilino. Una vez finalizada la migración del inquilino, puede empezar a usar una aplicación de Azure AD desde el nuevo inquilino para acceder a la cuenta de Media Services mediante las API de v2.

> [!IMPORTANT]
> Tiene que restablecer la información de [Autenticación de Azure AD](media-services-portal-get-started-with-aad.md) para acceder a la API de Media Services v2.
  
### <a name="considerations"></a>Consideraciones

* Cree copias de seguridad de todos los datos en su cuenta antes de migrar a otra suscripción.
* Debe detener todos los puntos de conexión de streaming y los recursos de streaming en vivo. Los usuarios no podrán acceder a su contenido durante el traslado del grupo de recursos.

> [!IMPORTANT]
> No inicie el punto de conexión de streaming hasta que la migración se haya completado correctamente.

### <a name="troubleshoot"></a>Solución de problemas

Si una cuenta de Media Services o una cuenta de Azure Storage asociada se "desconecta" después del traslado del grupo de recursos, intente rotar las claves de la cuenta de almacenamiento. Si la rotación de las claves de la cuenta de almacenamiento no resuelve el estado "desconectado" de la cuenta de Media Services, presente una nueva solicitud de soporte técnico desde el menú "Soporte técnico y solución de problemas" de la cuenta de Media Services.  

## <a name="next-steps"></a>Pasos siguientes

[Crear una cuenta](media-services-portal-create-account.md)
