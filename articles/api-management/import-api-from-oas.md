---
title: Importación de una especificación OpenAPI mediante Azure Portal | Microsoft Docs
description: Aprenda a importar una especificación de OpenAPI con API Management y, a continuación, probar la API en los portales de Azure y para desarrolladores.
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: tutorial
ms.date: 04/20/2020
ms.author: apimpm
ms.openlocfilehash: 346156c624a4c8b1c1d6cc490fc0559f364b4b9d
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87905574"
---
# <a name="import-an-openapi-specification"></a>Importación de una especificación OpenAPI

En este artículo se explica cómo importar una API de back-end de Especificación OpenAPI que reside en https://conferenceapi.azurewebsites.net?format=json. Esta API de back-end la proporciona Microsoft y se hospeda en Azure. El artículo también muestra cómo probar la API de APIM.

En este artículo aprenderá a:

> [!div class="checklist"]
> * Importación de una API de back-end de la especificación OpenAPI
> * Prueba de la API en Azure Portal
> * Pruebe la API en el Portal para desarrolladores

## <a name="prerequisites"></a>Prerrequisitos

Complete el siguiente inicio rápido: [Creación de una instancia de Azure API Management](get-started-create-service-instance.md)

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="import-and-publish-a-back-end-api"></a><a name="create-api"> </a>Importación y publicación de una API de back-end

1. Vaya al servicio API Management en Azure Portal y seleccione **API** en el menú.
2. Seleccione **Especificación OpenAPI** de la lista **Add a new API** (Agregar una nueva API).

    ![Especificación de OpenAPI](./media/import-api-from-oas/oas-api.png)
3. Escriba los valores de la API. Puede establecer los valores durante la creación o luego accediendo a la pestaña **Ajustes**. Los valores de configuración se explican en el tutorial [Importación y publicación de la primera API](import-and-publish.md#-import-and-publish-a-backend-api).
4. Seleccione **Crear**.

> [!NOTE]
> Las limitaciones de la importación de API se documentan en [otro artículo](api-management-api-import-restrictions.md).

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-append-apis.md)]

[!INCLUDE [api-management-define-api-topics.md](../../includes/api-management-define-api-topics.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Transformación y protección de una API publicada](transform-api.md)
