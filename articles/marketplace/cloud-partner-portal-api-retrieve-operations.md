---
title: 'API Retrieve operations: Azure Marketplace'
description: API para recuperar todas las operaciones en la oferta o para obtener una determinada operación para el operationId especificado.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: reference
author: mingshen-ms
ms.author: mingshen
ms.date: 07/14/2020
ms.openlocfilehash: cb44d977407a7e854603e6bbacf3591752b109c2
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87271950"
---
# <a name="retrieve-operations"></a>Retrieve operations

> [!NOTE]
> Las API de Cloud Partner Portal están integradas y seguirán funcionando en el Centro de partners. La transición presenta pequeños cambios. Revise los cambios que se muestran en [Referencia de API de Cloud Partner Portal](./cloud-partner-portal-api-overview.md) para asegurarse de que el código sigue funcionando después de la transición al Centro de partners. Las API de CPP solo deben usarse para los productos existentes que ya estaban integrados antes de la transición al Centro de partners. Los nuevos productos deben usar las API de envío del Centro de partners.

Recupera todas las operaciones en la oferta u obtiene una determinada operación para el operationId especificado. El cliente puede utilizar los parámetros de consulta para filtrar según las operaciones de ejecución.

``` https

  GET https://cloudpartner.azure.com/api/publishers/<publisherId>/offers/<offerId>/submissions/?api-version=2017-10-31&status=<filteredStatus>

  GET https://cloudpartner.azure.com/api/publishers/<publisherId>/offers/<offerId>/operations/<operationId>?api-version=2017-10-31

```

## <a name="uri-parameters"></a>Parámetros del identificador URI

|  **Nombre**          |      **Descripción**                                                                                           | **Tipo de datos** |
|  ----------------  |     --------------------------------------------------------------------------------------------------------   |  -----------  |
|  publisherId       |  Identificador del publicador, por ejemplo `Contoso`.                                                                   |  String       |
|  offerId           |  Identificador de la oferta.                                                                                              |  String       |
|  operationId       |  GUID que identifica de manera única la operación de la oferta. El operationId se puede recuperar mediante el uso de esta API y también se puede devolver en el encabezado HTTP de la respuesta para cualquier operación de larga ejecución, como la API de [publicación de oferta](./cloud-partner-portal-api-publish-offer.md).  |   Guid   |
|  api-version       | Última versión de la API. |    Date      |
|  |  |  |

## <a name="header"></a>Encabezado

|  **Nombre**          |  **Valor**           |
|  ---------------   | -------------------- |
|  Content-Type      | `application/json`   |
|  Authorization     | `Bearer YOUR_TOKEN`  |
|  |  |

## <a name="body-example"></a>Ejemplo de cuerpo

### <a name="response"></a>Response

#### <a name="get-operations"></a>Operaciones GET

``` json
    [
        {
            "id": "5a63deb5-925b-4ee0-938b-7c86fbf287c5",
            "offerId": "56615b67-2185-49fe-80d2-c4ddf77bb2e8",
            "offerVersion": 1,
            "offerTypeId": "microsoft-azure-virtualmachines",
            "publisherId": "contoso",
            "submissionType": "publish",
            "submissionState": "running",
            "publishingVersion": 2,
            "slot": "staging",
            "version": 636576975611768314,
            "definition": {
                "metadata": {
                    "emails": "jdoe@contoso.com"
                }
            },
            "changedTime": "2018-03-26T21:46:01.179948Z"
        }
    ]
```

#### <a name="get-operation"></a>Operación GET

``` json
    [
        {
            "status" : "running",
            "messages" : [],
            "publishingVersion" : 2,
            "offerVersion" : 1,
            "cancellationRequestState": "canCancel",
            "steps": [
                        {
                            "estimatedTimeFrame": "< 15 min",
                            "id": "displaydummycertify",
                            "stepName": "Validate Pre-Requisites",
                            "description": "Offer settings provided are validated",
                            "status": "complete",
                            "messages": 
                            [
                                {
                                    "messageHtml": "Step completed.",
                                    "level": "information",
                                    "timestamp": "2017-03-28T19:50:36.500052Z"
                                }
                            ],
                            "progressPercentage": 100
                        },
                        {
                            "estimatedTimeFrame": "< 5 day",
                            "id": "displaycertify",
                            "stepName": "Certification",
                            "description": "Your offer is analyzed by our certification systems for issues.",
                            "status": "blocked",
                            "messages": 
                            [
                                {
                                    "messageHtml": "No virtual machine image was found for the plan contoso.",
                                    "level": "error",
                                    "timestamp": "2017-03-28T19:50:39.5506018Z"
                                },
                                {
                                    "messageHtml": "This step has not started yet.",
                                    "level": "information",
                                    "timestamp": "2017-03-28T19:50:39.5506018Z"
                                }
                            ],
                            "progressPercentage": 0
                        },
                        {
                            "estimatedTimeFrame": "< 1 day",
                            "id": "displayprovision",
                            "stepName": "Provisioning",
                            "description": "Your virtual machine is being replicated in our production systems.",
                            "status": "notStarted",
                            "messages": [],
                            "progressPercentage": 0
                        },
                        {
                            "estimatedTimeFrame": "< 1 hour",
                            "id": "displaypackage",
                            "stepName": "Packaging and Lead Generation Registration",
                            "description": "Your virtual machine is packaged for being shown to your customers. Additionally, we hookup our lead generation systems to send leads for your offer.",
                            "status": "notStarted",
                            "messages": [],
                            "progressPercentage": 0
                        },
                        {
                            "id": "publisher-signoff",
                            "stepName": "Publisher signoff",
                            "description": "Offer is available to preview. Ensure that everything looks good before making your offer live.",
                            "status": "notStarted",
                            "messages": [],
                            "progressPercentage": 0
                        },
                        {
                            "estimatedTimeFrame": "~2-5 days",
                            "id": "live",
                            "stepName": "Live",
                            "description": "Offer is publicly visible and is available for purchase.",
                            "status": "notStarted",
                            "messages": [],
                            "progressPercentage": 0
                        }
                    ],
                "previewLinks": [],
                "liveLinks": [],
            }
        }
    ]
```

### <a name="response-body-properties"></a>Propiedades del cuerpo de la respuesta

|  **Nombre**                    |  **Descripción**                                                                                  |
|  --------------------        |  ------------------------------------------------------------------------------------------------ |
|  id                          | GUID que identifica de manera única la operación.                                                       |
|  submissionType              | Identifica el tipo de operación sobre el que informa la oferta, por ejemplo `Publish/GoLive`.      |
|  createdDateTime             | Datetime de UTC cuando se creó la operación.                                                       |
|  lastActionDateTime          | Datetime de UTC cuando se realizó la última actualización en la operación.                                       |
|  status                      | Estado de la operación, que puede ser `not started` \| `running` \| `failed` \| `completed`. Solo una operación puede tener el estado `running` a la vez. |
|  error                       | Mensaje de error de las operaciones incorrectas                                                               |
|  |  |

### <a name="response-step-properties"></a>Propiedades de paso de respuesta

|  **Nombre**                    |  **Descripción**                                                                                  |
|  --------------------        |  ------------------------------------------------------------------------------------------------ |
| estimatedTimeFrame | La duración estimada de esta operación. |
| id | El identificador único del proceso de paso. |
| description | Descripción del paso |
| stepName | El nombre descriptivo del paso. |
| status | El estado del paso, either `notStarted` \| `running` \| `failed` \| `completed` |
| messages | Cualquier notificación o advertencia detectada durante el paso. Matriz de cadenas |
| progressPercentage | Entero de 0 al 100 que indica la progresión del paso. |
| | |

### <a name="response-status-codes"></a>Códigos de estado de respuesta

| **Código**  |   **Descripción**                                                                                  |
|  -------- |   -------------------------------------------------------------------------------------------------|
|  200      | `OK`: la solicitud se ha procesado correctamente y se han devuelto las operaciones que se solicitaron.        |
|  400      | `Bad/Malformed request`: El cuerpo de la respuesta del error puede contener más información.                    |
|  403      | `Forbidden`: El cliente no tiene acceso al espacio de nombres especificado.                          |
|  404      | `Not found`: La entidad especificada no existe.                                                 |
|  |  |
