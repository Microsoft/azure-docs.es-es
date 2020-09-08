---
author: aahill
ms.author: aahi
ms.service: cognitive-services
ms.topic: include
ms.date: 06/20/2019
ms.openlocfilehash: 9d46b304d598b4830cf325909f77eea6b68af757
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89304096"
---
>[!NOTE]
> Los puntos de conexión de los recursos creados después del 1 de julio de 2019 usan el formato de subdominio personalizado que se muestra a continuación. Para más información y para obtener una lista completa de los puntos de conexión regionales, consulte [Nombres de subdominios personalizados para Cognitive Services.](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-custom-subdomains) 

Los servicios de Azure Cognitive Services se representan por medio de recursos de Azure a los que se suscribe. Cree un recurso para Ink Recognizer mediante [Azure Portal](../../cognitive-services-apis-create-account.md).

Después de crear un recurso, obtenga el punto de conexión y la clave; para ello, abra el recurso en [Azure Portal](https://ms.portal.azure.com#blade/HubsExtension/BrowseResourceGroupBlade) y haga clic en **Inicio rápido**.

Cree dos [variables de entorno](../../cognitive-services-apis-create-account.md#get-the-keys-for-your-resource):

* `INK_RECOGNITION_SUBSCRIPTION_KEY`: la clave de suscripción para autenticar las solicitudes. 

* `INK_RECOGNITION_ENDPOINT`: el punto de conexión del recurso. Tendrá el siguiente aspecto: <br> `https://<your-custom-subdomain>.api.cognitive.microsoft.com`   
