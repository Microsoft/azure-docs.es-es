---
title: Ejecución de un contenedor de Face en Azure Container Instances
titleSuffix: Azure Cognitive Services
description: Implemente el contenedor de Face en una instancia de Azure Container Instances y pruébelo en un explorador web.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: conceptual
ms.date: 07/16/2020
ms.author: aahi
ms.openlocfilehash: 5fbe316580cfa2ca280a2587536df037146e8d02
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86538129"
---
# <a name="deploy-the-face-container-to-azure-container-instances"></a>Implementar un contenedor de Face en Azure Container Instances

> [!IMPORTANT]
> Se ha alcanzado el límite de usuarios del contenedor de Face. Actualmente no aceptamos nuevas aplicaciones para el contenedor de Face.

Aprenda a implementar el contenedor de [Face](../face-how-to-install-containers.md) de Cognitive Services en una instancia de [Azure Container Instances](https://docs.microsoft.com/azure/container-instances/). Este procedimiento muestra la creación de un recurso de Azure Face. Luego se trata la extracción de la imagen de contenedor asociada. Por último, se resalta la posibilidad de aprovechar la orquestación de los dos desde un explorador. El uso de contenedores puede desviar la atención de los desarrolladores de la administración de la infraestructura y centrarla en el desarrollo de aplicaciones.

[!INCLUDE [Prerequisites](../../containers/includes/container-preview-prerequisites.md)]

[!INCLUDE [Create a Cognitive Services Face resource](../includes/create-face-resource.md)]

[!INCLUDE [Create an Face container on Azure Container Instances](../../containers/includes/create-container-instances-resource-from-azure-cli.md)]

[!INCLUDE [API documentation](../../../../includes/cognitive-services-containers-api-documentation.md)]

[!INCLUDE [Containers Next Steps](../../containers/includes/containers-next-steps.md)]