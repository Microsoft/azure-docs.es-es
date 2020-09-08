---
title: Creación de una entidad de servicio de Azure
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 09/01/2020
ms.author: pafarley
ms.openlocfilehash: 5cff103d17138608f4932e36444847437f6da236
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89321652"
---
## <a name="create-an-azure-service-principal"></a>Creación de una entidad de servicio de Azure

Para que la aplicación interactúe con su cuenta de Azure, necesita una entidad de servicio de Azure para administrar los permisos. Siga las instrucciones de [Creación de una entidad de servicio de Azure](https://docs.microsoft.com/powershell/azure/create-azure-service-principal-azureps?view=azps-4.4.0&viewFallbackFrom=azps-3.3.0).

Al crear una entidad de servicio, verá que tiene un valor secreto, un identificador y un identificador de aplicación. Guarde el identificador de aplicación y el secreto en una ubicación temporal para los pasos posteriores.