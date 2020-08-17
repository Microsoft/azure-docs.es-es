---
title: 'Creación o modificación de una oferta: Azure Marketplace'
description: API para crear una oferta o actualizar una existente.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: reference
author: mingshen-ms
ms.author: mingshen
ms.date: 07/29/2020
ms.openlocfilehash: dba47073a6676b17c091af048f583ca3e7712999
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87420233"
---
# <a name="create-or-modify-an-offer"></a>Creación o modificación de una oferta

> [!NOTE]
> Las API de Cloud Partner Portal se integran con el Centro de partners y seguirán funcionando en este. La transición presenta pequeños cambios. Revise los cambios que se muestran en [Referencia de API de Cloud Partner Portal](./cloud-partner-portal-api-overview.md) para asegurarse de que el código sigue funcionando después de la transición al Centro de partners. Las API de CPP solo deben usarse para los productos existentes que ya estaban integrados antes de la transición al Centro de partners. Los nuevos productos deben usar las API de envío del Centro de partners.

Esta llamada actualiza una oferta específica dentro del espacio de nombres del editor o crea una oferta.

  `PUT https://cloudpartner.azure.com/api/publishers/<publisherId>/offers/<offerId>?api-version=2017-10-31`

## <a name="uri-parameters"></a>Parámetros del identificador URI

|  **Nombre**         |  **Descripción**                      |  **Tipo de datos**  |
|  --------         |  ----------------                     |  -------------  |
| publisherId       |  Identificador del publicador, por ejemplo `contoso`. |   String |
| offerId           |  Identificador de la oferta.                     |   String        |
| api-version       |  La versión más reciente de la API            |   Date           |
|  |  |  |

## <a name="header"></a>Encabezado

|  **Nombre**        |  **Valor**               |
|  ---------       |  ----------              | 
| Content-Type     | `application/json`       |
| Authorization    | `Bearer YOUR_TOKEN`      |
|  |  |

## <a name="body-example"></a>Ejemplo de cuerpo

En el ejemplo siguiente se crea una oferta con offerID de `contosovirtualmachine`.

### <a name="request"></a>Solicitud

``` json
  {
      "publisherId": "contoso",
      "offerTypeId": "microsoft-azure-virtualmachines",
      "id": "contosovirtualmachine",
      "offerTypeVersions": {
          "microsoft-azure-virtualmachines": 87,
          "microsoft-azure-marketplace": 39
      },
      "definition": {
          "displayText": "Contoso Virtual Machine Offer",
          "offer": {
          "microsoft-azure-marketplace.title": "Contoso App",
          "microsoft-azure-marketplace.summary": "Contoso App makes dev ops a breeze",
          "microsoft-azure-marketplace.longSummary": "Contoso App makes dev ops a breeze",
          "microsoft-azure-marketplace.description": "Contoso App makes dev ops a breeze",
          "microsoft-azure-marketplace.offerMarketingUrlIdentifier": "contosoapp",
          "microsoft-azure-marketplace.allowedSubscriptions": ["59160c40-2e25-4dcf-a2fd-6514cb08bf08"],
          "microsoft-azure-marketplace.usefulLinks": [{
              "linkTitle": "Contoso App for Azure",
              "linkUrl": "https://azuremarketplace.microsoft.com"
          }],
          "microsoft-azure-marketplace.categoryMap": [
                {
                    "categoryL1": "analytics",
                    "categoryL2-analytics": [
                        "visualization-and-reporting"
                    ]
                },
                {
                    "categoryL1": "ai-plus-machine-learning",
                    "categoryL2-ai-plus-machine-learning":[
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
          "microsoft-azure-marketplace.heroLogo": "https://publishingapistore.blob.core.windows.net/testcontent/D6191_publishers_contoso/contosovirtualmachine/c46ec74d-d214-4fb5-9082-3cee55200eba.png?sv=2014-02-14&sr=b&sig=RfDvjowFGpP4WZGAHylbF2CuXwO2NXOrwycrXEJvJI4%3D&se=2019-03-28T19%3A46%3A49Z&sp=r",
          "microsoft-azure-marketplace.screenshots": [],
          "microsoft-azure-marketplace.videos": [],
          "microsoft-azure-marketplace.leadDestination": "None",
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
      "plans": 
      [
          {
              "planId": "contososkuidentifier",
              "microsoft-azure-virtualmachines.skuTitle": "Contoso App",
              "microsoft-azure-virtualmachines.skuSummary": "Contoso App makes dev ops a breeze.",
              "microsoft-azure-virtualmachines.skuDescription": "This is a description for the Contoso App that makes dev ops a breeze.",
              "microsoft-azure-virtualmachines.hideSKUForSolutionTemplate": false,
              "microsoft-azure-virtualmachines.cloudAvailability": ["PublicAzure"],
              "virtualMachinePricing": {
                  "isByol": true,
                  "freeTrialDurationInMonths": 0
              },
              "microsoft-azure-virtualmachines.operatingSystemFamily": "Windows",
              "microsoft-azure-virtualmachines.windowsOSType": "Other",
              "microsoft-azure-virtualmachines.operationSystem": "Contoso App",
              "microsoft-azure-virtualmachines.recommendedVMSizes": ["a0-basic", "a0-standard", "a1-basic", "a1-standard", "a2-basic", "a2-standard"],
              "microsoft-azure-virtualmachines.openPorts": [],
              "microsoft-azure-virtualmachines.vmImages": 
              {
                  "1.0.1": 
                  {
                      "osVhdUrl": "http://contosoteststorage.blob.core.windows.net/test/contosoVM.vhd?sv=2014-02-14&sr=c&sig=WlDo6Q4xwYH%2B5QEJbItPUVdgHhBcrVxPBmntZ2vU96w%3D&st=2016-06-25T18%3A30%3A00Z&se=2017-06-25T18%3A30%3A00Z&sp=rl",
                      "lunVhdDetails": []
                  }
              },
              "regions": ["AZ"]
          }
          ]
      },
      "eTag": "W/\"datetime'2017-06-07T06%3A15%3A40.4771399Z'\"",
      "version": 5
  }
```

### <a name="response"></a>Response

``` json
 {
         "offerTypeId": "microsoft-azure-virtualmachines",
         "publisherId": "contoso",
         "status": "neverPublished",
         "id": "contosovirtualmachine",
         "version": 1,
         "definition": {
         "displayText": "Contoso Virtual Machine Offer",
         "offer": 
         {
             "microsoft-azure-marketplace-testdrive.videos": [],
             "microsoft-azure-marketplace.title": "Contoso App",
             "microsoft-azure-marketplace.summary": "Contoso App makes dev ops a breeze",
             "microsoft-azure-marketplace.longSummary": "Contoso App makes dev ops a breeze",
             "microsoft-azure-marketplace.description": "Contoso App makes dev ops a breeze",
             "microsoft-azure-marketplace.offerMarketingUrlIdentifier": "contosoapp",
             "microsoft-azure-marketplace.allowedSubscriptions": 
             [
                 "59160c40-2e25-4dcf-a2fd-6514cb08bf08"
             ],
             "microsoft-azure-marketplace.usefulLinks": 
             [
                 {
                     "linkTitle": "Contoso App for Azure",
                     "linkUrl": "https://azuremarketplace.microsoft.com"
                 }
             ],
             "microsoft-azure-marketplace.categoryMap":
             [
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
             "microsoft-azure-marketplace.smallLogo": "https://publishingstoredm.blob.core.windows.net/prodcontent/D6191_publishers_marketplace:2Dtest/testaoffer/8affcd28-60a5-4839-adf8-c560e069fd61.png?sv=2014-02-14&sr=b&sig=nGErAgn%2BDUecrX892wcmk32kh0MHgIZeJ5jcKyY%2Fuew%3D&se=2020-03-28T22%3A27%3A13Z&sp=r",
             "microsoft-azure-marketplace.mediumLogo": "https://publishingstoredm.blob.core.windows.net/prodcontent/D6191_publishers_marketplace:2Dtest/testaoffer/39550bca-1110-432c-9ea9-e12b3a2288cd.png?sv=2014-02-14&sr=b&sig=4X0hlkXYtuZOmcYq%2BsbYVZz3k5k26kngcFX6yBAJjNI%3D&se=2020-03-28T22%3A27%3A13Z&sp=r",
             "microsoft-azure-marketplace.largeLogo": "https://publishingstoredm.blob.core.windows.net/prodcontent/D6191_publishers_marketplace:2Dtest/testaoffer/ce3576e3-df12-4074-b0a3-0b8d3f329df1.png?sv=2014-02-14&sr=b&sig=mFhtCUQh%2FbFz10nlIWbqsz6jq5MBZ0M%2F5cIREE9P6V0%3D&se=2020-03-28T22%3A27%3A13Z&sp=r",
             "microsoft-azure-marketplace.wideLogo": "https://publishingstoredm.blob.core.windows.net/prodcontent/D6191_publishers_marketplace:2Dtest/testaoffer/476d6edd-12d3-4414-9def-d2970c4a9de4.png?sv=2014-02-14&sr=b&sig=pg4MDSZjAb8w8D%2FrQ9RT%2BodpynSy%2FlYOvpx0yeam2Bw%3D&se=2020-03-28T22%3A27%3A13Z&sp=r",
             "microsoft-azure-marketplace.heroLogo": "https://publishingstoredm.blob.core.windows.net/prodcontent/D6191_publishers_marketplace:2Dtest/testaoffer/46c85b7b-4438-4e0d-8218-24fb5651727a.png?sv=2014-02-14&sr=b&sig=wIsCOO5%2BDj8NsLVSwwzwTgogF71oA7Q1XjKhNB1ni5g%3D&se=2020-03-28T22%3A27%3A13Z&sp=r",
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
         "plans": 
         [
             {
                 "planId": "contososkuidentifier",
                 "microsoft-azure-virtualmachines.skuTitle": "Contoso App (Old Title)",
                 "microsoft-azure-virtualmachines.skuSummary": "Contoso App makes dev ops a breeze.",
                 "microsoft-azure-virtualmachines.skuDescription": "This is a description for the Contoso App that makes dev ops a breeze.",
                 "microsoft-azure-virtualmachines.hideSKUForSolutionTemplate": false,
                 "microsoft-azure-virtualmachines.cloudAvailability": 
                 [
                     "PublicAzure"
                 ],
                 "microsoft-azure-virtualmachines.certificationsFairfax": [],
                 "virtualMachinePricing": {
                     "isByol": true,
                     "freeTrialDurationInMonths": 0
                 },
                 "microsoft-azure-virtualmachines.operatingSystemFamily": "Windows",
                 "microsoft-azure-virtualmachines.operationSystem": "Contoso App",
                 "microsoft-azure-virtualmachines.recommendedVMSizes": 
                 [
                     "a0-basic",
                     "a0-standard",
                     "a1-basic",
                     "a1-standard",
                     "a2-basic",
                     "a2-standard"
                 ],
                 "microsoft-azure-virtualmachines.openPorts": [],
                 "microsoft-azure-virtualmachines.vmImages": 
                 {
                     "1.0.1": 
                     {
                         "osVhdUrl": "http://contosoteststorage.blob.core.windows.net/test/contosoVM.vhd?sv=2014-02-14&sr=c&sig=WlDo6Q4xwYH%2B5QEJbItPUVdgHhBcrVxPBmntZ2vU96w%3D&st=2016-06-25T18%3A30%3A00Z&se=2017-06-25T18%3A30%3A00Z&sp=rl",
                         "lunVhdDetails": []
                     }
                 },
                 "regions":
                 [
                     "AZ"
                 ]
             } 
         ]
     },
     "changedTime": "2018-03-28T22:27:13.8363879Z"
 }
```

> [!NOTE]
> Para modificar esta oferta, agregue un encabezado **If-Match** establecido en * a la solicitud anterior. Use el mismo cuerpo de la solicitud anterior, pero modifique los valores como lo desee. 

### <a name="response-status-codes"></a>Códigos de estado de respuesta

| **Código**  |  **Descripción**                                                                            |
| --------  |  ---------------                                                                            |
|  200      | `OK`. La solicitud se procesó correctamente y la oferta se modificó sin problemas.           |
|  201      | `Created`. La solicitud se procesó correctamente y la oferta se creó sin problemas.   |
|  400      | `Bad/Malformed request`. El cuerpo de la respuesta del error podría proporcionar más información.            |
|  403      | `Forbidden`. El cliente no tiene acceso al espacio de nombres solicitado.                     |
|  404      | `Not found`. La entidad a la que hace referencia el cliente no existe.                           |
|  412      | El servidor no cumple una de las condiciones previas que el solicitante especificó en la solicitud. El cliente debe comparar la ETag que se envió con la solicitud. |
|  |  |

## <a name="uploading-artifacts"></a>Carga de los artefactos

Para compartir los artefactos, como imágenes y logotipos, debe cargarlos en una ubicación accesible en la Web y luego debe incluir cada uno de ellos como un URI en la solicitud PUT, como se muestra en el ejemplo anterior. El sistema detectará que estos archivos no se encuentran en el almacenamiento de Azure Marketplace y los descargará en el almacenamiento.  Como resultado, verá que futuras solicitudes GET devolverán una dirección URL de servicio de Azure Marketplace para estos archivos.

## <a name="categories-and-industries"></a>Categorías y sectores

Al crear una oferta, debe especificar una categoría para ella en el marketplace. Opcionalmente, en el caso de algunos tipos de ofertas, también puede especificar los sectores de la industria. En función del tipo de oferta, proporcione las categorías o sectores aplicables a ella mediante valores de clave específicos de las tablas siguientes.

### <a name="azure-marketplace-categories"></a>Categorías de Azure Marketplace

Estas categorías y sus claves correspondientes son aplicables a los tipos de ofertas de aplicaciones de Azure, Virtual Machines, Core Virtual Machines, contenedores, aplicaciones de contenedor, módulos de IoT Edge y SaaS. Los elementos en negrita cursiva (como ***analytics***) son las categorías y los elementos de texto estándar (como data-insights) son las subcategorías que están debajo de estas. Use los valores de clave exactos, sin cambiar el espaciado o el uso de mayúsculas.

| Category | Claves de SaaS | Claves de aplicaciones de Azure | Virtual Machines, contenedores, aplicaciones de contenedor, módulo de IoT Edge, claves de Core Virtual Machines |
| --- | --- | --- | --- |
| ***Analytics*** | ***analytics*** | ***analytics-azure-apps*** | ***analytics-amp*** |
| Data Insights | data-insights | data-insights | data-insights |
| Análisis de datos | data-analytics | data-analytics | data-analytics |
| Datos de gran tamaño | big-data | bigData | big-data |
| Análisis predictivo | predictive-analytics | predictive-analytics | predictive-analytics |
| Análisis de streaming/en tiempo real | real-time-streaming-analytics | real-time-streaming-analytics | real-time-streaming-analytics |
| Otros | Otros | other-analytics | Otros |
| ***Inteligencia artificial y aprendizaje automático*** | ***ArtificialIntelligence*** | ***ai-plus-machine-learning*** | ***ai-plus-machine-learning*** |
| Servicios de bot | bot-services | bot-services | bot-services |
| Cognitive Services | cognitive-services | cognitive-services | cognitive-services |
| ML Service | ml-service | ml-service | ml-service |
| Machine Learning Automatizado | automated-ml | automated-ml | automated-ml |
| Automatización de procesos empresariales o robóticos | business-robotic-process-automation | business-robotic-process-automation | business-robotic-process-automation |
| Etiquetado de datos | data-labelling | data-labelling | data-labelling |
| Preparación de los datos | data-preparation | data-preparation | data-preparation |
| Minería de conocimientos | knowledge-mining | knowledge-mining | knowledge-mining |
| Operaciones de ML | ml-operations | ml-operations | ml-operations |
| Otros | other-AI-plus-machine-learning | Otros | Otros |
| ***Blockchain*** | ***blockchain*** | ***blockchain*** | ***blockchain*** |
| Aceleradores de aplicación | app-accelerators | app-accelerators | app-accelerators |
| Libro de contabilidad de nodo único | single-node-ledger | single-node-ledger | single-node-ledger |
| Libro de contabilidad de varios nodos | multi-node-ledger | multi-node-ledger | multi-node-ledger |
| Herramientas | herramientas | herramientas | herramientas |
| Otros | Otros | Otros | Otros |
| ***Proceso*** | ***compute-saas*** | ***compute-azure-apps*** | ***compute*** |
| Infraestructura de la aplicación | appInfra | appInfrastructure | application-infrastructure |
| Sistemas operativos | clientOS | clientOS | operating-systems |
| Cache | caché | caché | caché |
| Otros | other-compute | other-compute | Otros |
| ***Contenedores*** | ***containers*** | ***containers*** | ***containers*** |
| Aplicaciones de contenedor | container-apps | container-apps | container-apps |
| Imágenes de contenedor | container-images | container-images | container-images |
| Introducción a los contenedores | get-started-with-containers | get-started-with-containers | get-started-with-containers |
| Otros | Otros | Otros | Otros |
| ***Bases de datos*** | ***databases-saas*** | ***database*** | ***databases*** |
| Bases de datos NoSQL | nosql-databases | nosql-databases | nosql-databases |
| Bases de datos relacionales | relational-databases | relational-databases | relational-databases |
| Bases de datos de cadena de bloques o libro de contabilidad | ledger-blockchain-databases | ledger-blockchain-databases | ledger-blockchain-databases |
| Lagos de datos | data-lakes | data-lakes | data-lakes |
| almacenamiento de datos | data-warehouse | data-warehouse | data-warehouse |
| Otros | other-databases | other-databases | Otros |
| ***Herramientas para desarrolladores*** | ***developer-tools-saas*** | ***developer-tools-azure-apps*** | ***developer-tools*** |
| Herramientas | tools-developer-tools | tools-developer-tools | tools-developer-tools |
| Scripts | scripts | scripts | scripts |
| Servicio para desarrolladores | devService | devService | developer-service |
| Otros | other-developer-tools | other-developer-tools | Otros |
| ***DevOps*** | ***devops*** | ***devops*** | ***devops*** |
| Otros | Otros | Otros | Otros |
| ***Identidad*** | ***identity*** | ***identity*** | ***identity*** |
| Administración de acceso | access-management | access-management | access-management |
| Otros | Otros | Otros | Otros |
| ***Integración*** | ***integration*** | ***integration*** | ***integration*** |
| Mensajería | messaging | messaging | messaging |
| Otros | Otros | Otros | Otros |
| ***Internet de las cosas*** | ***IoT*** | ***internet-of-things-azure-apps*** | ***internet-of-things*** |
| Servicios de IoT Core | N/D | iot-core-services | iot-core-services |
| Módulos de IoT Edge | N/D | iot-edge-modules | iot-edge-modules |
| Soluciones de IoT | iot-solutions | iot-solutions | iot-solutions |
| Visualización y análisis de datos | data-analytics-and-visualization | data-analytics-and-visualization | data-analytics-and-visualization |
| Conectividad de IoT | iot-connectivity | iot-connectivity | iot-connectivity |
| Otros | other-internet-of-things | other-internet-of-things | Otros |
| ***Herramientas de administración y TI*** | ***ITandAdministration*** | ***it-and-management-tools-azure-apps*** | ***it-and-management-tools*** |
| Soluciones de administración | management-solutions | management-solutions | management-solutions |
| Aplicaciones empresariales | businessApplication | businessApplication | business-applications |
| Otros | other-it-management-tools | other-it-management-tools | Otros |
| ***Supervisión y diagnósticos*** | ***monitoring-and-diagnostics*** | ***monitoring-and-diagnostics*** | ***monitoring-and-diagnostics*** |
| Otros | Otros | Otros | Otros |
| ***Elementos multimedia*** | ***media*** | ***media*** | ***media*** |
| Media Services | media-services | media-services | media-services |
| Content Protection | content-protection | content-protection | content-protection |
| Streaming a petición y en vivo | live-and-on-demand-streaming | live-and-on-demand-streaming | live-and-on-demand-streaming |
| Otros | Otros | Otros | Otros |
| ***Migración*** | ***migration*** | ***migration*** | ***migration*** |
| Migración de datos | data-migration | data-migration | data-migration |
| Otros | Otros | Otros | Otros |
| ***Realidad mixta*** | ***mixed-reality*** | ***mixed-reality*** | ***mixed-reality*** |
| Otros | Otros | Otros | Otros |
| ***Redes*** | ***networking*** | ***networking*** | ***networking*** |
| Administradores de dispositivos | appliance-managers | appliance-managers | appliance-managers |
| Conectividad | conectividad | conectividad | conectividad |
| Firewalls | Firewalls | Firewalls | Firewalls |
| Equilibradores de carga | load-balancers | load-balancers | load-balancers |
| Otros | Otros | Otros | Otros |
| ***Seguridad*** | ***security*** | ***security*** | ***security*** |
| Administración de identidad y acceso | identity-and-access-management | identity-and-access-management | identity-and-access-management |
| Protección contra amenazas | threat-protection | threat-protection | threat-protection |
| Information Protection | information-protection | information-protection | information-protection |
| Otros | Otros | Otros | Otros |
| ***Storage*** | ***storage-saas*** | ***storage-azure-apps*** | ***storage*** |
| Copia de seguridad y recuperación | copia de seguridad | copia de seguridad | backup-and-recovery |
| Almacenamiento híbrido empresarial | enterprise-hybrid-storage | enterprise-hybrid-storage | enterprise-hybrid-storage |
| Uso compartido de archivos | file-sharing | file-sharing | file-sharing |
| Administración del ciclo de vida de datos | data-lifecycle-management | data-lifecycle-management | data-lifecycle-management |
| Otros | other-storage | other-storage | Otros |
| ***Web*** | ***web*** | ***web*** | ***web*** |
| Blogs y CMS | blogs-and-cmss | blogs-and-cmss | blogs-and-cmss |
| Aplicaciones web de Starter | starter-web-apps | starter-web-apps | starter-web-apps |
| Comercio electrónico | ecommerce | ecommerce | ecommerce |
| Marcos de Web Apps | web-apps-frameworks | web-apps-frameworks | web-apps-frameworks |
| Web Apps | web-apps | web-apps | web-apps |
| Otros | Otros | Otros | Otros |
||||

### <a name="microsoft-appsource-categories"></a>Categorías de Microsoft AppSource

Estas categorías y sus claves correspondientes se aplican a los tipos de ofertas de aplicaciones de SaaS y PowerBI, Dynamics 365 Business Central, Dynamics 365 for Customer Engagement y Dynamics 365 for Operations. Los elementos en negrita cursiva (como ***analytics***) son las categorías y los elementos de texto estándar (advanced-analytics) son las subcategorías que están debajo de estas. Use los valores de clave exactos, sin cambiar el espaciado o el uso de mayúsculas.

| Category | Claves de SaaS | Claves de Dynamics 365 Business Central, Dynamics 365 for Customer Engagement, Dynamics 365 for Operations | Claves de aplicación de PowerBI |
| --- | --- | --- | --- |
| ***Analytics*** | ***analytics*** | ***Analytics*** | ***Analytics*** |
| Análisis avanzado | advanced-analytics | advanced-analytics | advanced-analytics |
| Visualización e informes | visualization-reporting | visualization-reporting | visualization-reporting |
| Otros | Otros | other-analytics | other-analytics |
| ***Inteligencia artificial y aprendizaje automático*** | ***ArtificialIntelligence*** | ***ai-plus-machine-learning-dynamics*** | ***ai-plus-machine-learning-appsource*** |
| Inteligencia artificial para empresas | ai-for-business | ai-for-business | ai-for-business |
| Aplicaciones de bot | bot-apps | bot-apps | bot-apps |
| Otros | other-ai-plus-machine-learning | other-ai-plus-machine-learning | other-ai-plus-machine-learning |
| ***Colaboración*** | ***Colaboración*** | ***Colaboración*** | ***collaboration*** |
| Contacto y personas | contact-people | contact-people | contact-and-people |
| Administración de reuniones | meeting-management | meeting-management | meeting-management |
| Administración y diseño de sitios | site-design-management | site-design-management | site-design-and-management |
| Administración de proyectos y tareas | task-project-management | task-project-management | task-and-project-management |
| Conferencia de voz o videoconferencia | voice-video-conferencing | voice-video-conferencing | voice-and-video-conferencing |
| Otros | other-collaboration | other-collaboration | Otros |
| ***Cumplimiento y legalidad*** | ***compliance*** | ***compliance*** | ***compliance-and-legal*** |
| Auditoría e impuestos | tax-audit | tax-audit | tax-and-audit |
| Información legal | Información legal | Información legal | legal |
| Datos, gobernanza y privacidad | data-governance-privacy | data-governance-privacy | data-governance-and-privacy |
| Salud y seguridad | health-safety | health-safety | health-and-safety |
| Otros | other-compliance-legal | other-compliance-legal | Otros |
| ***Servicio de atención al cliente*** | ***CustomerService*** | ***CustomerService*** | ***customer-service*** |
| Centro de contacto | contact-center | contact-center | contact-center |
| Servicio presencial | face-to-face-service | face-to-face-service | face-to-face-service |
| Área de operaciones y servicio para empleados | back-office-employee-service | back-office-employee-service | back-office-and-employee-service |
| Administración de casos y conocimientos | knowledge-case-management | knowledge-case-management | knowledge-and-case-management |
| Redes sociales e interacción omnicanal | social-media-omnichannel-engagement | social-media-omnichannel-engagement | social-media-and-omnichannel-engagement |
| Otros | other-customer-service | other-customer-service | Otros |
| ***Sector financiero*** | ***Sector financiero*** | ***Sector financiero*** | ***finance*** |
| Control | accounting | accounting | accounting |
| Administración de recursos | asset-management | asset-management | asset-management |
| Análisis, consolidación e informes | analytics-consolidation-reporting | analytics-consolidation-reporting | analytics-consolidation-and-reporting |
| Colecciones y crédito | credit-collections | credit-collections | credit-and-collections |
| Administración de riesgos y cumplimiento | compliance-risk-management | compliance-risk-management | compliance-and-risk-management |
| Otros | other-finance | other-finance | Otros |
| ***Recursos humanos*** | ***HumanResources*** | ***HumanResources*** | ***human-resources*** |
| Adquisición de talentos | talent-acquisition | talent-acquisition | talent-acquisition |
| Administración de talentos | talent-management | talent-management | talent-management |
| Operaciones de recursos humanos | hr-operations | hr-operations | hr-operations |
| Análisis y planeamiento de personal | workforce-planning-analytics | workforce-planning-analytics | workforce-planning-and-analytics |
| Otros | other-human-resources | other-human-resources | Otros |
| ***Internet de las cosas*** | ***IoT*** | ***internet-of-things-dynamics*** | ***internet-of-things-appsource*** |
| Operaciones y administración de recursos | asset-management-operations | asset-management-operations | asset-management-and-operations |
| Productos conectados | connected-products | connected-products | connected-products |
| Cadena de suministro inteligente | intelligent-supply-chain | intelligent-supply-chain | intelligent-supply-chain |
| Mantenimiento predictivo | predictive-maintenance | predictive-maintenance | predictive-maintenance |
| Supervisión remota | remote-monitoring | remote-monitoring | remote-monitoring |
| Seguridad y protección | safety-security | safety-security | safety-and-security |
| Recursos e infraestructuras inteligentes | smart-infrastructure-resources | smart-infrastructure-resources | smart-infrastructure-and-resources |
| Vehículos y movilidad | vehicles-mobility | vehicles-mobility | vehicles-and-mobility |
| Otros | other-internet-of-things | other-internet-of-things | Otros |
| ***Herramientas de administración y TI*** | ***ITandAdministration*** | ***ITandAdministration*** | ***it-and-management-tools*** |
| Soluciones de administración | management-solutions | management-solutions | management-solutions |
| Aplicaciones empresariales | businessApplication | businessApplication | business-applications |
| Otros | other-it-management-tools | other-it-management-tools | Otros |
| ***Marketing*** | ***Marketing*** | ***Marketing*** | ***marketing*** |
| Anuncio | advertisement | advertisement | advertisement |
| Análisis | analytics-marketing | analytics-marketing | analytics-marketing |
| Automatización y administración de campañas | campaign-management-automation | campaign-management-automation | campaign-management-and-automation |
| Marketing por correo electrónico | email-marketing | email-marketing | email-marketing |
| Administración de recursos y eventos L2 | events-resource-management | events-resource-management | events-and-resource-management |
| Investigación y análisis | research-analytics | research-analytics | research-and-analysis |
| Medios sociales | social-media | social-media | social-media |
| Otros | other-marketing | other-marketing | Otros |
| ***Cadena de operaciones y suministros*** | ***OperationsSupplyChain*** | ***OperationsSupplyChain*** | ***operations-and-supply-chain*** |
| Administración de producción y recursos | asset-production-management | asset-production-management | asset-and-production-management |
| Previsión de la demanda | demand-forecasting | demand-forecasting | demand-forecasting |
| Conectividad y administración de la información | information-management-connectivity | information-management-connectivity | information-management-and-connectivity |
| Planeamiento, compra e informes | planning-purchasing-reporting | planning-purchasing-reporting | planning-purchasing-and-reporting |
| Administración de servicios y calidad | quality-service-management | quality-service-management | quality-and-service-management |
| Administración de pedidos y ventas | sales-order-management | sales-order-management | sales-and-order-management |
| Administración de transporte y almacén | transportation-warehouse-management | transportation-warehouse-management | transportation-and-warehouse-management |
| Otros | other-operations-supply-chain | other-operations-supply-chain | Otros |
| ***Productividad*** | ***Productividad*** | ***Productividad*** | ***productivity*** |
| Administración y creación de contenido | content-creation-management | content-creation-management | content-creation-and-management |
| Idioma y traducción | language-translation | language-translation | language-and-translation |
| Administración de documentos | document-management | document-management | document-management |
| Administración del correo electrónico | email-management | email-management | email-management |
| Búsqueda y referencia | search-reference | search-reference | search-and-reference |
| Otros | other-productivity | other-productivity | Otros |
| Ludificación | Ludificación | Ludificación | gamification |
| ***Ventas*** | ***Ventas*** | ***Ventas*** | ***Ventas*** |
| Televentas | telesales | telesales | telesales |
| Configuración, precio, oferta (CPQ) | configure-price-quote | configure-price-quote | configure-price-quote |
| Administración de contratos | contract-management | contract-management | contract-management |
| CRM | crm | crm | crm |
| Comercio electrónico | e-commerce | e-commerce | e-commerce |
| Enriquecimiento de datos empresariales | business-data-enrichment | business-data-enrichment | business-data-enrichment |
| Habilitación de ventas | sales-enablement | sales-enablement | sales-enablement |
| Otros | other-sales | other-sales | other-sales |
| ***Geolocalización*** | ***geolocation*** | ***geolocation*** | ***geolocation*** |
| Mapas | maps | maps | maps |
| Noticias y el tiempo | news-and-weather | news-and-weather | news-and-weather |
| Otros | other-geolocation | other-geolocation | other-geolocation |
||||

### <a name="microsoft-appsource-industries"></a>Sectores de Microsoft AppSource

Estos sectores y sus claves correspondientes se aplican a los tipos de ofertas de aplicaciones de SaaS y PowerBI, Dynamics 365 Business Central, Dynamics 365 for Customer Engagement y Dynamics 365 for Operations. Los elementos en negrita cursiva (como ***Automotive***) son las categorías y los elementos de texto estándar (como AutomotiveL2) son las subcategorías que están debajo de estas. Use los valores de clave exactos, sin cambiar el espaciado o el uso de mayúsculas.

| Sector | Claves de SaaS, Dynamics 365 Business Central, Dynamics 365 for Customer Engagement, Dynamics 365 for Operations | Claves de aplicaciones de PowerBI |
| --- | --- | --- |
| ***Automoción*** | ***Automoción*** | ***automotive*** |
| Automoción | AutomotiveL2 | AutomotiveL2 |
| ***Agricultura*** | ***Agricultura*** | ***agriculture*** |
| Otros: sin segmentar | Agriculture\_OtherUnsegmented | other-unsegmented |
| ***Distribución*** | ***Distribución*** | ***distribution*** |
| Venta al por mayor | Venta al por mayor | wholesale |
| Envío de paquetes | ParcelAndPackageShipping | parcel-and-package-shipping |
| ***Education*** | ***Education*** | ***education*** |
| Educación superior | HigherEducation | higher-education |
| Educación primaria y secundaria/K-12 | PrimaryAndSecondaryEducationK12 | primary-and-secondary-education |
| Bibliotecas y museos | LibrariesAndMuseums | libraries-and-museums |
| ***Servicios financieros*** | ***FinancialServices*** | ***financial-services*** |
| Banca y mercados de capital | BankingAndCapitalMarkets | banking-and-capital-markets |
| Seguros | Seguros | seguros |
| ***Gobierno*** | ***Gobierno*** | ***government*** |
| Defensa e inteligencia | DefenseAndIntelligence | defense-and-intelligence |
| Justicia y seguridad pública | PublicSafetyAndJustice | public-safety-and-justice |
| Gobierno civil | CivilianGovernment | civilian-government |
| ***Atención sanitaria*** | ***HealthCareandLifeSciences*** | ***healthcare*** |
| Responsable de los pagos de mantenimiento | HealthPayor | health-payor |
| Proveedor de asistencia sanitaria | HealthProvider | health-provider |
| Productos farmacéuticos | Productos farmacéuticos | productos farmacéuticos |
| ***Fabricación y recursos*** | ***Manufacturing*** | ***manufacturing-and-resources*** |
| Química y agroquímica | ChemicalAndAgrochemical | chemical-and-agrochemical |
| Fabricación discreta | DiscreteManufacturing | discrete-manufacturing |
| Sector energético | Sector energético | energy |
| ***Minoristas y bienes de consumo*** | ***RetailandConsumerGoods*** | ***retail-and-consumer-goods*** |
| Bienes de consumo | ConsumerGoods | consumer-goods |
| Minoristas | Minoristas | retailers |
| ***Elementos multimedia y comunicaciones*** | ***MediaAndCommunications*** | ***media-and-communications*** |
| Elementos multimedia y entretenimiento | MediaandEntertainment | media-and-entertainment |
| Telecomunicaciones | Telecomunicaciones | telecommunications |
| ***Servicios profesionales*** | ***ProfessionalServices*** | ***professional-services*** |
| Información legal | Información legal | legal |
| Servicios profesionales de partner | PartnerProfessionalServices | partner-professional-services |
| ***Arquitectura y construcción*** | ***ArchitectureAndConstruction*** | ***architecture-and-construction*** |
| Otros: sin segmentar | ArchitectureAndConstruction\_OtherUnsegmented | other-unsegmented |
| ***Hostelería y viajes*** | ***HospitalityandTravel*** | ***hospitality-and-travel*** |
|    Hoteles y ocio | HotelsAndLeisure | hotels-and-leisure |
| Viajes y transporte | TravelAndTransportation | travel-and-transportation |
| Servicio de restaurantes y hostelería | RestaurantsAndFoodServices | restaurants-and-food-services |
| ***Otras industrias del sector público*** | ***OtherPublicSectorIndustries*** | ***other-public-sector-industries*** |
| Silvicultura y pesca | ForestryAndFishing | forestry-and-fishing |
| Nonprofits | Nonprofits | nonprofits |
| ***Bienes inmuebles*** | ***RealEstate*** | ***real-estate*** |
| Otros: sin segmentar | RealEstate\_OtherUnsegmented | other-unsegmented |
|||
