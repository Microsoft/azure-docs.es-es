---
title: Uso de una memoria caché externa en Azure API Management | Microsoft Docs
description: Obtenga información sobre cómo configurar y usar una memoria caché externa en Azure API Management.
services: api-management
documentationcenter: ''
author: mikebudzynski
manager: erikre
editor: ''
ms.assetid: 740f6a27-8323-474d-ade2-828ae0c75e7a
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/15/2019
ms.author: apimpm
ms.openlocfilehash: e2362d06fa0ef795122a2d47a7a621b66fdd9470
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65780353"
---
# <a name="use-an-external-azure-cache-for-redis-in-azure-api-management"></a>Uso de una instancia externa de Azure Redis Cache en Azure API Management

Además de utilizar la memoria caché integrada, Azure API Management permite también almacenar en caché las respuestas en una instancia externa de Azure Redis Cache.

El uso de una memoria caché externa permite superar algunas limitaciones de la memoria caché integrada. Resulta especialmente útil si desea:

* Evitar que la memoria caché se borre periódicamente durante las actualizaciones de API Management
* Tener más control sobre la configuración de la memoria caché
* Almacenar en memoria caché más datos de los que permite el plan de API Management
* Usar el almacenamiento en caché con el nivel de consumo de API Management

Para más información acerca del almacenamiento en caché, consulte [Directivas de almacenamiento en caché de API Management](api-management-caching-policies.md) y [Almacenamiento en caché personalizado en Azure API Management](api-management-sample-cache-by-key.md).

![Traiga su propia memoria caché a APIM](media/api-management-howto-cache-external/overview.png)

Temas que se abordarán:

> [!div class="checklist"]
> * Agregar una memoria caché externa en API Management

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, necesita:

+ [Creación de una instancia de Azure API Management](get-started-create-service-instance.md)
+ Comprender el [almacenamiento en memoria caché en Azure API Management](api-management-howto-cache.md)

## <a name="create-cache"> </a> Creación de una instancia de Azure Redis Cache

En esta sección se explica cómo crear una instancia de Azure Redis Cache en Azure. Si ya tiene una instancia de Azure Redis Cache, dentro o fuera de Azure, puede <a href="#add-external-cache">pasar</a> a la sección siguiente.

[!INCLUDE [redis-cache-create](../../includes/redis-cache-create.md)]

## <a name="add-external-cache"> </a> Adición de una memoria caché externa

Siga estos pasos para agregar una instancia externa de Azure Redis Cache en Azure API Management.

![Traiga su propia memoria caché a APIM](media/api-management-howto-cache-external/add-external-cache.png)

> [!NOTE]
> El **usar desde** configuración especifica que se comunicarán las implementaciones regionales de API Management con la memoria caché configurada en el caso de una configuración regional múltiple de API Management. Las memorias caché especificadas como **Predeterminada** serán reemplazadas por las memorias caché con un valor regional.
>
> Por ejemplo, si API Management está hospedado en las regiones Este de EE. UU., Sudeste Asiático y Europa Occidental y hay dos memorias caché configuradas, una como **Predeterminada** y otra para **Sudeste Asiático**, la instancia de API Management de  **Sudeste Asiático** usará su propia memoria caché, mientras que las otras dos regiones utilizará la entrada de caché **Predeterminada**.

### <a name="add-an-azure-cache-for-redis-from-the-same-subscription"></a>Incorporación de una instancia de Azure Redis Cache desde la misma suscripción

1. Vaya a la instancia de API Management en Azure Portal.
2. Seleccione la pestaña **Caché externa** en el menú de la izquierda.
3. Haga clic en el botón **+ Agregar**.
4. Seleccione la memoria caché en el campo de lista desplegable **Instancia de caché**.
5. Seleccione **predeterminado** o especificar la región deseada en el **usar desde** campo de lista desplegable.
6. Haga clic en **Save**(Guardar).

### <a name="add-an-azure-cache-for-redis-hosted-outside-of-the-current-azure-subscription-or-azure-in-general"></a>Incorporación de una instancia de Azure Redis Cache hospedada fuera de la suscripción de Azure actual o de Azure en general

1. Vaya a la instancia de API Management en Azure Portal.
2. Seleccione la pestaña **Caché externa** en el menú de la izquierda.
3. Haga clic en el botón **+ Agregar**.
4. Seleccione **Personalizada** en el campo de lista desplegable **Instancia de caché**.
5. Seleccione **predeterminado** o especificar la región deseada en el **usar desde** campo de lista desplegable.
6. Proporcione la cadena de conexión de Azure Redis Cache en el campo **Cadena de conexión**.
7. Haga clic en **Save**(Guardar).

## <a name="use-the-external-cache"></a>Uso de la memoria caché externa

Una vez que la memoria caché externa está configurada en Azure API Management, se puede usar mediante directivas de almacenamiento en caché. Consulte [Incorporación del almacenamiento en caché para mejorar el rendimiento en Azure API Management](api-management-howto-cache.md) para conocer los pasos detallados.

## <a name="next-steps"> </a>Pasos siguientes

* Para más información sobre las directivas de almacenamiento en caché, consulte [Caching policies][Caching policies] (Directivas de almacenamiento en caché) en [API Management policy reference][API Management policy reference] (Referencia de la directiva de API Management).
* Para obtener información sobre el almacenamiento en caché de los elementos por parte de la clave mediante expresiones de directiva, consulte [Custom caching in Azure API Management](api-management-sample-cache-by-key.md).

[API Management policy reference]: https://msdn.microsoft.com/library/azure/dn894081.aspx
[Caching policies]: https://msdn.microsoft.com/library/azure/dn894086.aspx
