---
title: 'Control de simultaneidad: Azure Marketplace'
description: Estrategias de control de simultaneidad para las API de publicación de Cloud Partner Portal.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: reference
author: mingshen-ms
ms.author: mingshen
ms.date: 07/14/2020
ms.openlocfilehash: 2ec2e701a1d9cb449939416e65cf645f8bc01887
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87287913"
---
# <a name="concurrency-control"></a>Control de simultaneidad

> [!NOTE]
> Las API de Cloud Partner Portal se integran con el Centro de partners y seguirán funcionando en este. La transición presenta pequeños cambios. Revise los cambios que se muestran en [Referencia de API de Cloud Partner Portal](./cloud-partner-portal-api-overview.md) para asegurarse de que el código sigue funcionando después de la transición al Centro de partners. Las API de CPP solo deben usarse para los productos existentes que ya estaban integrados antes de la transición al Centro de partners. Los nuevos productos deben usar las API de envío del Centro de partners.

Cada llamada a la API de publicación de Cloud Partner Portal debe especificar explícitamente la estrategia de control de simultaneidad que se usará. Si no se proporciona el encabezado **If-Match**, se producirá una respuesta de error HTTP 400. Ofrecemos dos estrategias para el control de simultaneidad.

-   **Optimista**: el cliente que realiza que la actualización comprueba si los datos han cambiado desde la última lectura de los datos.
-   **El último gana**: el cliente actualiza directamente los datos, independientemente de si otra aplicación los ha modificado desde la última lectura.

<a name="optimistic-concurrency-workflow"></a>Flujo de trabajo de simultaneidad optimista
-------------------------------

Se recomienda usar la estrategia de simultaneidad optimista, con el siguiente flujo de trabajo, para garantizar que no se realizan modificaciones inesperadas a los recursos.

1.  Recupere una entidad mediante las API. La respuesta incluye un valor de ETag que identifica la versión de la entidad almacenada actualmente (en el momento de la respuesta).
2.  En el momento de la actualización, incluya este valor de ETag en el encabezado de solicitud **If-Match** obligatorio.
3.  La API compara el valor ETag recibido en la solicitud con el valor ETag actual de la entidad en una transacción atómica.
    *   Si los valores de ETag son diferentes, la API devuelve un respuesta HTTP `412 Precondition Failed`. Este error indica que cualquier otro proceso ha actualizado la entidad desde que el cliente la recuperó por última vez, o que el valor de ETag especificado en la solicitud es incorrecto.
    *  Si los valores de ETag son iguales, o el encabezado **If-Match** contiene el carácter comodín asterisco (`*`), la API realiza la operación solicitada. La operación de API también actualiza el valor de ETag almacenado de la entidad.


> [!NOTE]
> Especificar el carácter comodín (*) en el encabezado **If-Match** lleva a que la API use la estrategia de simultaneidad El último gana. En este caso, no se realiza la comparación de ETag y el recurso se actualiza sin ninguna comprobación. 
