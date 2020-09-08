---
title: Uso de una memoria caché externa en Azure API Management | Microsoft Docs
description: Obtenga información sobre cómo configurar y usar una memoria caché externa en Azure API Management. El uso de una caché externa permite superar algunas limitaciones de la caché integrada.
services: api-management
documentationcenter: ''
author: mikebudzynski
manager: erikre
editor: ''
ms.assetid: 740f6a27-8323-474d-ade2-828ae0c75e7a
ms.service: api-management
ms.topic: conceptual
ms.date: 04/26/2020
ms.author: apimpm
ms.openlocfilehash: cfb7dd7a3831d90235b25af9598cfbc137ffcb3d
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87904962"
---
# <a name="use-an-external-redis-compatible-cache-in-azure-api-management"></a>Uso de una memoria caché compatible con Redis externa en Azure API Management

Además de utilizar la memoria caché integrada, Azure API Management permite almacenar en caché las respuestas en una memoria caché compatible con Redis, por ejemplo, Azure Cache for Redis.

El uso de una caché externa permite superar algunas limitaciones de la caché integrada:

* Evitar que la memoria caché se borre periódicamente durante las actualizaciones de API Management
* Tener más control sobre la configuración de la memoria caché
* Almacenar en memoria caché más datos de los que permite el plan de API Management
* Usar el almacenamiento en caché con el nivel de consumo de API Management
* Habilitar el almacenamiento en caché tal como se explica para las [puertas de enlace autohospedadas de API Management](self-hosted-gateway-overview.md)

Para más información acerca del almacenamiento en caché, consulte [Directivas de almacenamiento en caché de API Management](api-management-caching-policies.md) y [Almacenamiento en caché personalizado en Azure API Management](api-management-sample-cache-by-key.md).

![Traiga su propia memoria caché a APIM](media/api-management-howto-cache-external/overview.png)

Temas que se abordarán:

> [!div class="checklist"]
> * Agregar una memoria caché externa en API Management

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, necesita:

+ [Creación de una instancia de Azure API Management](get-started-create-service-instance.md)
+ Comprender el [almacenamiento en memoria caché en Azure API Management](api-management-howto-cache.md)

## <a name="create-azure-cache-for-redis"></a><a name="create-cache"> </a> Creación de una instancia de Azure Redis Cache

En esta sección se explica cómo crear una instancia de Azure Redis Cache en Azure. Si ya tiene una instancia de Azure Redis Cache, dentro o fuera de Azure, puede <a href="#add-external-cache">pasar</a> a la sección siguiente.

[!INCLUDE [redis-cache-create](../../includes/redis-cache-create.md)]

## <a name="deploy-redis-cache-to-kubernetes"></a><a name="create-cache"> </a> Implementación de Redis Cache en Kubernetes

En el caso del almacenamiento en caché, las puertas de enlace autohospedadas se basan exclusivamente en memorias caché externas. Para que el almacenamiento en caché sea eficaz, las puertas de enlace autohospedadas y la caché en la que se basan, debe estar cerca unas de otras para minimizar las latencias de búsqueda y almacenamiento. La implementación de una memoria Redis Cache en el mismo clúster de Kubernetes o en un clúster independiente cercano son las mejores opciones. Siga este [vínculo](https://github.com/kubernetes/examples/tree/master/guestbook) para aprender a implementar Redis Cache en un clúster de Kubernetes.

## <a name="add-an-external-cache"></a><a name="add-external-cache"> </a> Adición de una memoria caché externa

Siga estos pasos para agregar una instancia externa de Azure Redis Cache en Azure API Management.

![Traiga su propia memoria caché a APIM](media/api-management-howto-cache-external/add-external-cache.png)

> [!NOTE]
> La configuración **Utilizar desde** especifica una región de Azure o una ubicación de puerta de enlace autohospedada que usará la caché configurada. Las memorias caché configuradas como **Predeterminadas** se reemplazarán por memorias caché con un valor de región o ubicación coincidente específico.
>
> Por ejemplo, si API Management está hospedado en las regiones Este de EE. UU., Sudeste de Asia y Oeste de Europa y hay dos memorias caché configuradas, una como **Predeterminada** y otra para **Sudeste de Asia**, la instancia de API Management de ** Sudeste de Asia** usará su propia memoria caché, mientras que las otras dos regiones utilizará la entrada de caché **Predeterminada**.

### <a name="add-an-azure-cache-for-redis-from-the-same-subscription"></a>Incorporación de una instancia de Azure Redis Cache desde la misma suscripción

1. Vaya a la instancia de API Management en Azure Portal.
2. Seleccione la pestaña **Caché externa** en el menú de la izquierda.
3. Haga clic en el botón **+ Agregar**.
4. Seleccione la memoria caché en el campo de lista desplegable **Instancia de caché**.
5. Seleccione **Predeterminada** o especifique la región deseada en el campo de lista desplegable **Utilizar desde**.
6. Haga clic en **Save**(Guardar).

### <a name="add-an-azure-cache-for-redis-hosted-outside-of-the-current-azure-subscription-or-azure-in-general"></a>Incorporación de una instancia de Azure Redis Cache hospedada fuera de la suscripción de Azure actual o de Azure en general

1. Vaya a la instancia de API Management en Azure Portal.
2. Seleccione la pestaña **Caché externa** en el menú de la izquierda.
3. Haga clic en el botón **+ Agregar**.
4. Seleccione **Personalizada** en el campo de lista desplegable **Instancia de caché**.
5. Seleccione **Predeterminada** o especifique la región deseada en el campo de lista desplegable **Utilizar desde**.
6. Proporcione la cadena de conexión de Azure Redis Cache en el campo **Cadena de conexión**.
7. Haga clic en **Save**(Guardar).

### <a name="add-a-redis-cache-to-a-self-hosted-gateway"></a>Incorporación de Redis Cache a una puerta de enlace autohospedada

1. Vaya a la instancia de API Management en Azure Portal.
2. Seleccione la pestaña **Caché externa** en el menú de la izquierda.
3. Haga clic en el botón **+ Agregar**.
4. Seleccione **Personalizada** en el campo de lista desplegable **Instancia de caché**.
5. Especifique la ubicación de puerta de enlace autohospedada que desee o **Predeterminada** en el campo desplegable **Utilizar desde**.
6. Proporcione la cadena de conexión de Redis Cache en el campo **Cadena de conexión**.
7. Haga clic en **Save**(Guardar).

## <a name="use-the-external-cache"></a>Uso de la memoria caché externa

Una vez que la memoria caché externa está configurada en Azure API Management, se puede usar mediante directivas de almacenamiento en caché. Consulte [Incorporación del almacenamiento en caché para mejorar el rendimiento en Azure API Management](api-management-howto-cache.md) para conocer los pasos detallados.

## <a name="next-steps"></a><a name="next-steps"> </a>Pasos siguientes

* Para más información sobre las directivas de almacenamiento en caché, consulte [Caching policies][Caching policies] (Directivas de almacenamiento en caché) en [API Management policy reference][API Management policy reference] (Referencia de la directiva de Administración de API).
* Para obtener información sobre el almacenamiento en caché de los elementos por parte de la clave mediante expresiones de directiva, consulte [Custom caching in Azure API Management](api-management-sample-cache-by-key.md).

[API Management policy reference]: ./api-management-policies.md
[Caching policies]: ./api-management-caching-policies.md
