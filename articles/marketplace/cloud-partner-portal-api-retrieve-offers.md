---
title: 'API Recuperación de ofertas: Azure Marketplace'
description: La API para recuperar una lista resumida de ofertas de un espacio de nombres del anunciante.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: reference
author: mingshen-ms
ms.author: mingshen
ms.date: 07/14/2020
ms.openlocfilehash: 39d07751c708d5555799ecbb3b3bc66d3f44f43a
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87271967"
---
# <a name="retrieve-offers"></a>Recuperación de ofertas

> [!NOTE]
> Las API de Cloud Partner Portal se integran en el Centro de partners y seguirán funcionando en este. La transición presenta pequeños cambios. Revise los cambios que se muestran en la [referencia de API de Cloud Partner Portal](./cloud-partner-portal-api-overview.md) para asegurarse de que el código siga funcionando después de la transición al Centro de partners. Las API de CPP solo deben usarse para los productos existentes que ya estaban integrados antes de la transición al Centro de partners. Los nuevos productos deben usar las API de envío del Centro de partners.

Recupera una lista resumida de ofertas de un espacio de nombres del anunciante.

 `GET https://cloudpartner.azure.com/api/publishers/<publisherId>/offers?api-version=2017-10-31`

## <a name="uri-parameters"></a>Parámetros del identificador URI

| **Nombre**         |  **Descripción**                         |  **Tipo de datos** |
| -------------    |  ------------------------------------    |  -----------   |
|  publisherId     | Identificador del publicador, por ejemplo `contoso`. |   String    |
|  api-version     | Última versión de la API.                    |    Date        |
|  |  |

## <a name="header"></a>Encabezado

|  **Nombre**        |         **Valor**       |
|  --------------- |       ----------------  |
|  Content-Type    | `application/json`      |
|  Authorization   | `Bearer YOUR_TOKEN`     |
|  |  |

## <a name="body-example"></a>Ejemplo de cuerpo

### <a name="response"></a>Response

``` json
  200 OK 
  [ 
      {  
          "offerTypeId": "microsoft-azure-virtualmachines",
          "publisherId": "contoso",
          "status": "published",
          "id": "059afc24-07de-4126-b004-4e42a51816fe",
          "version": 1,
          "definition": {
              "displayText": "Contoso Virtual Machine"
          },
          "changedTime":"2017-05-23T23:33:47.8802283Z"
      }
  ]
```

### <a name="response-body-properties"></a>Propiedades del cuerpo de la respuesta

|  **Nombre**       |       **Descripción**                                                                                                  |
|  -------------  |      --------------------------------------------------------------------------------------------------------------    |
|  offerTypeId    | Identifica el tipo de oferta                                                                                           |
|  publisherId    | El identificador que distingue de manera exclusiva al anunciante                                                                      |
|  status         | Estado de la oferta. Para obtener la lista de valores posibles, consulte la sección [Estado de la oferta](#offer-status) a continuación.                         |
|  id             | GUID que identifica de forma exclusiva a la oferta del espacio de nombres del anunciante.                                                    |
|  version        | Versión actual de la oferta. El cliente no puede modificar la propiedad de versión. Se incrementa después de cada publicación. |
|  definición     | Contiene una vista resumida de la definición real de la carga de trabajo. Para obtener una definición detallada, use la API de [Recuperación de una oferta específica](./cloud-partner-portal-api-retrieve-specific-offer.md). |
|  changedTime    | Hora UTC en la que se modificó por última vez la oferta                                                                              |
|  |  |

### <a name="response-status-codes"></a>Códigos de estado de respuesta

| **Código**  |  **Descripción**                                                                                                   |
| -------   |  ----------------------------------------------------------------------------------------------------------------- |
|  200      | `OK`: La solicitud se ha procesado correctamente y todas las ofertas del anunciante se han devuelto al cliente.  |
|  400      | `Bad/Malformed request`: El cuerpo de la respuesta del error puede contener más información.                                    |
|  403      | `Forbidden`: El cliente no tiene acceso al espacio de nombres especificado.                                          |
|  404      | `Not found`: La entidad especificada no existe.                                                                 |
|  |  |

### <a name="offer-status"></a>Estado de la oferta

|  **Nombre**                    | **Descripción**                                  |
|  ------------------------    | -----------------------------------------------  |
|  NeverPublished              | La oferta nunca se ha publicado.                  |
|  NotStarted                  | La oferta es nueva y no se ha iniciado.                 |
|  WaitingForPublisherReview   | La oferta está esperando la aprobación del anunciante.         |
|  En ejecución                     | Se está procesando el envío de la oferta.             |
|  Correcto                   | Se terminó de procesar el envío de la oferta.       |
|  Canceled                    | Se canceló el envío de la oferta.                   |
|  Con error                      | Error en el envío de la oferta.                         |
|  |  |
