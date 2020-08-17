---
title: 'Recuperación de una API de oferta específica: Azure Marketplace'
description: API para recuperar la oferta especificada en el espacio de nombres del editor.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: reference
author: mingshen-ms
ms.author: mingshen
ms.date: 07/14/2020
ms.openlocfilehash: 9f3ba6b2f13b9f2bb1d538db84723e3a9baaef12
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87271848"
---
# <a name="retrieve-a-specific-offer"></a>Recuperación de una oferta específica

> [!NOTE]
> Las API de Cloud Partner Portal se integran con el Centro de partners y seguirán funcionando en este. La transición presenta pequeños cambios. Revise los cambios que se muestran en [Referencia de API de Cloud Partner Portal](./cloud-partner-portal-api-overview.md) para asegurarse de que el código sigue funcionando después de la transición al Centro de partners. Las API de CPP solo deben usarse para los productos existentes que ya estaban integrados antes de la transición al Centro de partners. Los nuevos productos deben usar las API de envío del Centro de partners.

Recupera la oferta especificada en el espacio de nombres del anunciante.  

También puede recuperar una versión concreta de la oferta o recuperar la oferta en espacios de borrador, visualización o producción. Si no se especifica un espacio, el valor predeterminado es `draft`. Si intenta recuperar una oferta de la que no hay una versión preliminar o que no se ha publicado, se producirá un error `404 Not Found`.

> [!WARNING]
> Esta API no recuperará los valores del secreto de los campos de tipo de secreto.

``` http
    GET https://cloudpartner.azure.com/api/publishers/<publisherId>/offers/<offerId>?api-version=2017-10-31

    GET https://cloudpartner.azure.com/api/publishers/<publisherId>/offers/<offerId>/versions/<version>?api-version=2017-10-31

    GET https://cloudpartner.azure.com/api/publishers/<publisherId>/offers/<offerId>/slot/<slotId>?api-version=2017-10-31

```

## <a name="uri-parameters"></a>Parámetros del identificador URI

| **Nombre**    | **Descripción**                                                                          | **Tipo de datos** |
|-------------|------------------------------------------------------------------------------------------|---------------|
| publisherId | publisherId. Por ejemplo, Contoso.                                                        | String        |
| offerId     | GUID que identifica de manera única la oferta.                                                 | String        |
| version     | Versión de la oferta que se va a recuperar. De forma predeterminada, se recupera la versión más reciente de la oferta. | Entero       |
| slotId      | El espacio del que se va a recuperar la oferta. Puede ser uno de estos:      <br/>  - `Draft` (valor predeterminado), recupera la versión de la oferta que está actualmente en modo borrador.  <br/>  -  `Preview` recupera la versión de la oferta que está actualmente en la etapa de versión preliminar.     <br/>  -  `Production` recupera la versión de la oferta que está actualmente en la fase de producción.          |      enum |
| api-version | Última versión de la API.                                                                    | Date          |
|  |  |  |

## <a name="header"></a>Encabezado

|  **Nombre**          |   **Valor**            |
|  ---------------   |  --------------        |
|  Content-Type      | `application/json`     |
|  Authorization     | `Bearer YOUR_TOKEN`    |
|  |  |

## <a name="body-example"></a>Ejemplo de cuerpo

### <a name="response"></a>Response

``` json
{
    "offerTypeId": "microsoft-azure-virtualmachines",
    "publisherId": "contoso",
    "status": "failed",
    "id": "059afc24-07de-4126-b004-4e42a51816fe",
    "version": 5,
    "definition": {
        "displayText": "Contoso Virtual Machine Offer",
        "offer": {
            "microsoft-azure-marketplace-testdrive.enabled": false,
            "microsoft-azure-marketplace-testdrive.videos": [],
            "microsoft-azure-marketplace.title": "Contoso App",
            "microsoft-azure-marketplace.summary": "Contoso App makes dev ops a breeze",
            "microsoft-azure-marketplace.longSummary": "Contoso App makes dev ops a breeze",
            "microsoft-azure-marketplace.description": "Contoso App makes dev ops a breeze",
            "microsoft-azure-marketplace.offerMarketingUrlIdentifier": "contosoapp",
            "microsoft-azure-marketplace.allowedSubscriptions": [
                "59160c40-2e25-4dcf-a2fd-6514cb08bf08"
            ],
            "microsoft-azure-marketplace.usefulLinks": [
            {
                "linkTitle": "Contoso App for Azure",
                "linkUrl": "https://azuremarketplace.microsoft.com"
            }
            ],
                "microsoft-azure-marketplace.categoryMap": [
                {
                "categoryL1": "analytics",
                "categoryL2-analytics": [
                "visualization-and-reporting"
                ]
                },
                {
                "categoryL1": "ai-plus-machine-learning",
                "categoryL2-ai-plus-machine-learning": [
                "bot-services",
                "cognitive-services",
                "other"
                ]
                }
            ],
            "microsoft-azure-marketplace.smallLogo": "https://publishingapistore.blob.core.windows.net/testcontent/D6191_publishers_contoso/contosovirtualmachine/6218c455-9cbc-450c-9920-f2e7a69ee132.png?sv=2014-02-14&sr=b&sig=6O8MM9dgiJ48VK0MwddkyVbprRAnBszyhVkVHGShhkI%3D&se=2019-03-28T19%3A46%3A50Z&sp=r",
            "microsoft-azure-marketplace.mediumLogo": "https://publishingapistore.blob.core.windows.net/testcontent/D6191_publishers_contoso/contosovirtualmachine/557e714b-2f31-4e12-b0cc-e48dd840edf4.png?sv=2014-02-14&sr=b&sig=NwL67NTQf9Gc9VScmZehtbHXpYmxhwZc2foy3o4xavs%3D&se=2019-03-28T19%3A46%3A49Z&sp=r",
            "microsoft-azure-marketplace.largeLogo": "https://publishingapistore.blob.core.windows.net/testcontent/D6191_publishers_contoso/contosovirtualmachine/142485da-784c-44cb-9523-d4f396446258.png?sv=2014-02-14&sr=b&sig=xaMxhwx%2FlKYfz33mJGIg8UBdVpsOwVvqhjTJ883o0iY%3D&se=2019-03-28T19%3A46%3A49Z&sp=r",
            "microsoft-azure-marketplace.wideLogo": "https://publishingapistore.blob.core.windows.net/testcontent/D6191_publishers_contoso/contosovirtualmachine/48af9013-1df7-4c94-8da8-4626e5039ce0.png?sv=2014-02-14&sr=b&sig=%2BnN7f2tprkrqb45ID6JlT01zXcy1PMTkWXtLKD6nfoE%3D&se=2019-03-28T19%3A46%3A49Z&sp=r",
            "microsoft-azure-marketplace.screenshots": [],
            "microsoft-azure-marketplace.videos": [],
            "microsoft-azure-marketplace.leadDestination": "None",
            "microsoft-azure-marketplace.tableLeadConfiguration": {},
            "microsoft-azure-marketplace.blobLeadConfiguration": {},
            "microsoft-azure-marketplace.salesForceLeadConfiguration": {},
            "microsoft-azure-marketplace.crmLeadConfiguration": {},
            "microsoft-azure-marketplace.httpsEndpointLeadConfiguration": {},
            "microsoft-azure-marketplace.marketoLeadConfiguration": {},
            "microsoft-azure-marketplace.privacyURL": "https://azuremarketplace.microsoft.com",
            "microsoft-azure-marketplace.termsOfUse": "Terms of use",
            "microsoft-azure-marketplace.engineeringContactName": "Jon Doe",
            "microsoft-azure-marketplace.engineeringContactEmail": "jondoe@outlook.com",
            "microsoft-azure-marketplace.engineeringContactPhone": "555-555-5555",
            "microsoft-azure-marketplace.supportContactName": "Jon Doe",
            "microsoft-azure-marketplace.supportContactEmail": "jondoe@outlook.com",
            "microsoft-azure-marketplace.supportContactPhone": "555-555-5555",
            "microsoft-azure-marketplace.publicAzureSupportUrl": "",
            "microsoft-azure-marketplace.fairfaxSupportUrl": ""
            },
            "plans": [
            {
                "planId": "contososkuidentifier",
                "microsoft-azure-virtualmachines.skuTitle": "Contoso App",
                "microsoft-azure-virtualmachines.skuSummary": "Contoso App makes dev ops a breeze.",
                "microsoft-azure-virtualmachines.skuDescription": "This is a description for the Contoso App that makes dev ops a breeze.",
                "microsoft-azure-virtualmachines.hideSKUForSolutionTemplate": false,
                "microsoft-azure-virtualmachines.cloudAvailability": [
                    "PublicAzure"
                ],
                "microsoft-azure-virtualmachines.certificationsFairfax": [],
                "virtualMachinePricing": {
                    "isByol": true,
                    "freeTrialDurationInMonths": 0
                },
                "microsoft-azure-virtualmachines.operatingSystemFamily": "Windows",
                "microsoft-azure-virtualmachines.windowsOSType": "Other",
                "microsoft-azure-virtualmachines.operationSystem": "Contoso App",
                "microsoft-azure-virtualmachines.recommendedVMSizes": [
                    "a0-basic",
                    "a0-standard",
                    "a1-basic",
                    "a1-standard",
                    "a2-basic",
                    "a2-standard"
                ],
                "microsoft-azure-virtualmachines.openPorts": [],
                "microsoft-azure-virtualmachines.vmImages": {
                    "1.0.1": {
                    "osVhdUrl": "http://contosoteststorage.blob.core.windows.net/test/contosoVM.vhd?sv=2014-02-14&sig=WlDo6Q4xwYH%2B5QEJbItPUVdgHhBcrVxPBmntZ2vU96w%3D&st=2016-06-25T18%3A30%3A00Z&se=2017-06-25T18%3A30%3A00Z&sp=rl",
                    "lunVhdDetails": []
                    }
                },
                "regions": [
                    "DZ",
                    "AR"
                ]
            }
            ]
        },
        "changedTime": "2017-06-07T06:15:39.7349221Z"
    }
}
```

### <a name="response-body-properties"></a>Propiedades del cuerpo de la respuesta

|  **Nombre**       |   **Descripción**                                                                                                               |
|  -------------  |   -----------------------------------------------------------------------------------------------------                         |
|  offerTypeId    | Identifica el tipo de oferta                                                                                                    |
|  publisherId    | Identificador único del anunciante                                                                                              |
|  status         | Estado de la oferta. Para obtener la lista de valores posibles, consulte la sección [Estado de la oferta](#offer-status) a continuación.                                  |
|  Identificador             | GUID que identifica de manera única la oferta                                                                                         |
|  version        | Versión actual de la oferta. El cliente no puede modificar la propiedad de versión. Se incrementa después de cada publicación.    |
|  definición     | Definición real de la carga de trabajo                                                                                               |
|  changedTime    | Fecha y hora UTC en la que se modificó por última vez la oferta                                                                                   |
|  |  |

### <a name="response-status-codes"></a>Códigos de estado de respuesta

| **Código**  | **Descripción**                                                                                                                 |
|  ------   | ------------------------------------------------------------------------------------------------------------------------------- |
|  200      | `OK`: La solicitud se ha procesado correctamente y todas las ofertas del anunciante se han devuelto al cliente.               |
|  400      | `Bad/Malformed request`: El cuerpo de la respuesta del error puede contener más información.                                                 |
|  403      | `Forbidden`: El cliente no tiene acceso al espacio de nombres especificado.                                                        |
|  404      | `Not found`: La entidad especificada no existe. El cliente debe comprobar publisherId, offerId y version (si se especifica).      |
|  |  |

### <a name="offer-status"></a>Estado de la oferta

|  **Nombre**                   |   **Descripción**                             |
| --------------------------- |  -------------------------------------------- |
|  NeverPublished             | La oferta nunca se ha publicado.               |
|  NotStarted                 | La oferta es nueva y no se ha iniciado.              |
|  WaitingForPublisherReview  | La oferta está esperando la aprobación del anunciante.      |
|  En ejecución                    | Se está procesando el envío de la oferta.          |
|  Correcto                  | Se terminó de procesar el envío de la oferta.    |
|  Canceled                   | Se canceló el envío de la oferta.                |
|  Con error                     | Error en el envío de la oferta.                      |
|  |  |
