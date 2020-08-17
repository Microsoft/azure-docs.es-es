---
title: 'Publicación de una oferta: Azure Marketplace'
description: API para publicar la oferta especificada.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: reference
author: mingshen-ms
ms.author: mingshen
ms.date: 07/14/2020
ms.openlocfilehash: 60e75aff79913896bdf1dcdc8754b6ecf5620b06
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87272052"
---
# <a name="publish-an-offer"></a>Publicación de una oferta

> [!NOTE]
> Las API de Cloud Partner Portal se integran con el Centro de partners y seguirán funcionando en este. La transición presenta pequeños cambios. Revise los cambios que se muestran en [Referencia de API de Cloud Partner Portal](./cloud-partner-portal-api-overview.md) para asegurarse de que el código sigue funcionando después de la transición al Centro de partners. Las API de CPP solo deben usarse para los productos existentes que ya estaban integrados antes de la transición al Centro de partners. Los nuevos productos deben usar las API de envío del Centro de partners.

Inicia el proceso de publicación de la oferta especificada. Esta llamada es una operación de larga duración.

  `POST  https://cloudpartner.azure.com/api/publishers/<publisherId>/offers/<offerId>/publish?api-version=2017-10-31`

## <a name="uri-parameters"></a>Parámetros del identificador URI
--------------

|  **Nombre**      |    **Descripción**                               |  **Tipo de datos** |
|  ------------- |  ------------------------------------            |   -----------  |
|  publisherId   | Identificador del publicador, por ejemplo `contoso`.      |   String       |
|  offerId       | Identificador de la oferta.                                 |   String       |
|  api-version   | La versión más reciente de la API                        |   Date         |
|  |  |

## <a name="header"></a>Encabezado
------

|  **Nombre**        |    **Valor**          |
|  --------        |    ---------          |
|  Content-Type    | `application/json`    |
|  Authorization   |  `Bearer YOUR_TOKEN`  |
|  |  |


## <a name="body-example"></a>Ejemplo de cuerpo
------------

### <a name="request"></a>Solicitud

``` json
  { 
      'metadata': 
          { 
              'notification-emails': 'jdoe@contoso.com'
          } 
  }
```

### <a name="request-body-properties"></a>Propiedades del cuerpo de la solicitud

|  **Nombre**               |   **Descripción**                                                                                 |
|  ---------------------  | ------------------------------------------------------------------------------------------------- |
|  notification-emails    | Lista de direcciones de correo electrónico separadas por comas para recibir una notificación del progreso de la operación de publicación. |
|  |  |

### <a name="response"></a>Response

#### <a name="migrated-offers"></a>Ofertas migradas

`Location: /api/publishers/contoso/offers/contoso-offer/operations/56615b67-2185-49fe-80d2-c4ddf77bb2e8?api-version=2017-10-31`

#### <a name="non-migrated-offers"></a>Ofertas no migradas

`Location: /api/operations/contoso$contoso-offer$2$preview?api-version=2017-10-31`

### <a name="response-header"></a>Encabezado de respuesta

|  **Nombre**             |    **Valor**                                                                 |
|  -------------------- | ---------------------------------------------------------------------------- |
| Location    | La ruta de acceso relativa para recuperar el estado de esta operación.     |
|  |  |

### <a name="response-status-codes"></a>Códigos de estado de respuesta

| **Código** |  **Descripción**                                                                                                                           |
| ------   |  ----------------------------------------------------------------------------------------------------------------------------------------- |
| 202   | `Accepted`: La solicitud se aceptó correctamente. La respuesta contiene una ubicación que se puede usar para realizar un seguimiento de la operación que se inicia. |
| 400   | `Bad/Malformed request`: el cuerpo de la respuesta del error puede proporcionar más información.                                                               |
| 422   | `Un-processable entity`: indica que la entidad que se va a publicar no se pudo validar.                                                        |
| 404   | `Not found`: la entidad especificada por el cliente no existe.                                                                              |
|  |  |
