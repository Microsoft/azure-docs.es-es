---
title: 'Administración de varios inquilinos con Video Indexer: Azure'
description: En este artículo se indican diferentes opciones de integración para administrar varios inquilinos con Video Indexer.
services: media-services
documentationcenter: ''
author: ika-microsoft
manager: femila
editor: ''
ms.service: media-services
ms.subservice: video-indexer
ms.workload: ''
ms.topic: article
ms.custom: ''
ms.date: 05/15/2019
ms.author: ikbarmen
ms.openlocfilehash: 2919e021d6b70ce82a6ff6b1d1972dd89de95104
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65799486"
---
# <a name="manage-multiple-tenants"></a>Administración de varios inquilinos

En este artículo se analizan diferentes opciones para administrar varios inquilinos con Video Indexer. Elija el método que sea más adecuado para su escenario:

* Una cuenta de Video Indexer por cada inquilino
* Una única cuenta de Video Indexer para todos los inquilinos
* Una suscripción de Azure por cada inquilino

## <a name="video-indexer-account-per-tenant"></a>Una cuenta de Video Indexer por cada inquilino

Cuando se usa esta arquitectura, se crea una cuenta de Video Indexer para cada inquilino. Los inquilinos tienen un aislamiento completo en el nivel persistente y en el de proceso.  

![Una cuenta de Video Indexer por cada inquilino](./media/manage-multiple-tenants/video-indexer-account-per-tenant.png)

### <a name="considerations"></a>Consideraciones

* Los clientes no comparten cuentas de almacenamiento (a menos que el cliente las configure manualmente).
* Los clientes no comparten la capacidad de proceso (unidades reservadas) y los tiempos de procesamiento de los trabajos de otros usuarios no se ven afectados.
* Puede quitar fácilmente un inquilino del sistema mediante la eliminación de la cuenta de Video Indexer.
* No hay ninguna posibilidad de compartir modelos personalizados entre los inquilinos.

    Asegúrese de que no hay ningún requisito empresarial para compartir modelos personalizados.
* Más difícil de administrar debido a las múltiples cuentas de Video Indexer (y las instancias de Media Services asociadas) por cada inquilino.

> [!TIP]
> Cree un usuario administrador para el sistema en el [Portal para desarrolladores de Video Indexer](https://api-portal.videoindexer.ai/) y use la API de autorización para proporcionar a los inquilinos el correspondiente [token de acceso de la cuenta](https://api-portal.videoindexer.ai/docs/services/authorization/operations/Get-Account-Access-Token).

## <a name="single-video-indexer-account-for-all-users"></a>Una única cuenta de Video Indexer para todos los usuarios

Cuando se usa esta arquitectura, el cliente es responsable del aislamiento de los inquilinos. Todos los inquilinos tienen que usar una única cuenta de Video Indexer con una única cuenta de Azure Media Services. Al cargar, buscar o eliminar contenido, el cliente deberá filtrar los resultados adecuados para ese inquilino.

![Una única cuenta de Video Indexer para todos los usuarios](./media/manage-multiple-tenants/single-video-indexer-account-for-all-users.png)

Con esta opción, se pueden compartir los modelos de personalización (persona, idioma y marcas) o se pueden aislar entre inquilinos filtrando los modelos por inquilino.

Al [cargar vídeos](https://api-portal.videoindexer.ai/docs/services/operations/operations/Upload-video?), puede especificar un atributo de partición diferente por cada inquilino. Esto permitirá el aislamiento en la [API de búsqueda](https://api-portal.videoindexer.ai/docs/services/operations/operations/Search-videos?). Si especifica el atributo de partición en la API de búsqueda, solo obtendrá los resultados de la partición especificada. 

### <a name="considerations"></a>Consideraciones

* Posibilidad de compartir los modelos de contenido y personalización entre los inquilinos.
* Un inquilino afecta al rendimiento de otros inquilinos.
* El cliente necesita crear un nivel de administración complejo por encima de Video Indexer.

> [!TIP]
> Puede usar el atributo [priority](upload-index-videos.md) para clasificar los trabajos de los inquilinos.

## <a name="azure-subscription-per-tenant"></a>Una suscripción de Azure por cada inquilino 

Si se usa esta arquitectura, cada inquilino tiene su propia suscripción de Azure. Para cada usuario, creará una nueva cuenta de Video Indexer en la suscripción del inquilino.

![Una suscripción de Azure por cada inquilino](./media/manage-multiple-tenants/azure-subscription-per-tenant.png)

### <a name="considerations"></a>Consideraciones

* Esta es la única opción que permite la separación de la facturación.
* Esta integración tiene más sobrecarga de administración que la del escenario con la cuenta de Video Indexer por cada inquilino. Si la facturación no es un requisito, se recomienda utilizar una de las otras opciones que se describen en este artículo.

## <a name="next-steps"></a>Pasos siguientes

[Información general](video-indexer-overview.md)
