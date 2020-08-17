---
title: Copia de datos de Salesforce Marketing Cloud
description: Obtenga información sobre cómo copiar datos desde Salesforce Marketing Cloud en almacenes de datos receptores compatibles a través de una actividad de copia de una canalización de Azure Data Factory.
services: data-factory
documentationcenter: ''
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 07/17/2020
ms.openlocfilehash: 1f0fb1ee8580c0c7f6eb30228b65e0a3780ef0a8
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87076798"
---
# <a name="copy-data-from-salesforce-marketing-cloud-using-azure-data-factory"></a>Copia de datos de Salesforce Marketing Cloud con Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se explica el uso de la actividad de copia de Azure Data Factory para copiar datos desde una base de datos Salesforce Marketing Cloud. El documento se basa en el artículo de [introducción a la actividad de copia](copy-activity-overview.md) que describe información general de la actividad de copia.

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Este conector de Salesforce Marketing Cloud es compatible con las actividades siguientes:

- [Actividad de copia](copy-activity-overview.md) con [matriz de origen o receptor compatible](copy-activity-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)

Puede copiar datos desde Salesforce Marketing Cloud a cualquier almacén de datos receptor compatible. Consulte la tabla de [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver una lista de almacenes de datos que la actividad de copia admite como orígenes o receptores.

El conector de Salesforce Marketing Cloud es compatible con la autenticación OAuth 2 y admite tipos de paquetes heredados y mejorados. Este conector se crea en la parte superior de la [API REST de Salesforce Marketing Cloud](https://developer.salesforce.com/docs/atlas.en-us.mc-apis.meta/mc-apis/index-api.htm).

>[!NOTE]
>Este conector no admite la recuperación de objetos personalizados o extensiones de datos personalizadas.

## <a name="getting-started"></a>Introducción

Puede crear una canalización con la actividad de copia mediante el SDK de .NET, el SDK de Python, Azure PowerShell, la API de REST o la plantilla de Azure Resource Manager. Consulte el [tutorial de actividad de copia](quickstart-create-data-factory-dot-net.md) para obtener instrucciones paso a paso para crear una canalización con una actividad de copia.

Las secciones siguientes proporcionan detalles sobre las propiedades que se usan para definir entidades de Data Factory específicas para el conector Salesforce Marketing Cloud.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado Salesforce Marketing Cloud:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type debe establecerse en: **SalesforceMarketingCloud** | Sí |
| connectionProperties | Un grupo de propiedades que define cómo conectarse a Salesforce Marketing Cloud. | Sí |
| ***En`connectionProperties`:*** | | |
| authenticationType | especifica el método de autenticación que se va a utilizar. Los valores permitidos son: `Enhanced sts OAuth 2.0` o `OAuth_2.0`.<br><br>El paquete heredado de Salesforce Marketing Cloud solo admite `OAuth_2.0`, mientras que el paquete mejorado necesita `Enhanced sts OAuth 2.0`. <br>Desde el 1 de agosto de 2019, Salesforce Marketing Cloud ha eliminado la capacidad de crear paquetes heredados. Todos los paquetes nuevos son paquetes mejorados. | Sí |
| host | Para un paquete mejorado, el host debe ser el [subdominio](https://developer.salesforce.com/docs/atlas.en-us.mc-apis.meta/mc-apis/your-subdomain-tenant-specific-endpoints.htm) que se representa mediante una cadena de 28 caracteres que empieza por las letras "mc", por ejemplo `mc563885gzs27c5t9-63k636ttgm`. <br>Para un paquete heredado, especifique `www.exacttargetapis.com`. | Sí |
| clientId | El identificador de cliente asociado a la aplicación Salesforce Marketing Cloud.  | Sí |
| clientSecret | El secreto de cliente asociado a la aplicación Salesforce Marketing Cloud. Puede elegir marcar este campo como SecureString para almacenarlo de forma segura en ADF o almacenar el secreto en Azure Key Vault y permitir que la actividad de copia de ADF incorpore los cambios desde allí al realizar la copia de datos. Obtenga más información sobre el [Almacenamiento de credenciales en Key Vault](store-credentials-in-key-vault.md). | Sí |
| useEncryptedEndpoints | Especifica si los puntos de conexión de origen de datos se cifran mediante HTTPS. El valor predeterminado es true.  | No |
| useHostVerification | Especifica si se requiere que el nombre de host del certificado del servidor coincida con el nombre de host del servidor al conectarse a través de TLS. El valor predeterminado es true.  | No |
| usePeerVerification | Especifica si se debe verificar la identidad del servidor al conectarse a través de TLS. El valor predeterminado es true.  | No |

**Ejemplo: uso de la autenticación OAuth 2 de STS para un paquete mejorado** 

```json
{
    "name": "SalesforceMarketingCloudLinkedService",
    "properties": {
        "type": "SalesforceMarketingCloud",
        "typeProperties": {
            "connectionProperties": {
                "host": "<subdomain e.g. mc563885gzs27c5t9-63k636ttgm>",
                "authenticationType": "Enhanced sts OAuth 2.0",
                "clientId": "<clientId>",
                "clientSecret": {
                     "type": "SecureString",
                     "value": "<clientSecret>"
                },
                "useEncryptedEndpoints": true,
                "useHostVerification": true,
                "usePeerVerification": true
            }
        }
    }
}

```

**Ejemplo: uso de la autenticación OAuth 2 para un paquete heredado** 

```json
{
    "name": "SalesforceMarketingCloudLinkedService",
    "properties": {
        "type": "SalesforceMarketingCloud",
        "typeProperties": {
            "connectionProperties": {
                "host": "www.exacttargetapis.com",
                "authenticationType": "OAuth_2.0",
                "clientId": "<clientId>",
                "clientSecret": {
                     "type": "SecureString",
                     "value": "<clientSecret>"
                },
                "useEncryptedEndpoints": true,
                "useHostVerification": true,
                "usePeerVerification": true
            }
        }
    }
}

```

Si usaba el servicio Salesforce Marketing Cloud vinculado a la siguiente carga, todavía se admite tal cual, aunque se aconseja usar la nueva en el futuro, que agrega compatibilidad con paquetes mejorados.

```json
{
    "name": "SalesforceMarketingCloudLinkedService",
    "properties": {
        "type": "SalesforceMarketingCloud",
        "typeProperties": {
            "clientId": "<clientId>",
            "clientSecret": {
                 "type": "SecureString",
                 "value": "<clientSecret>"
            },
            "useEncryptedEndpoints": true,
            "useHostVerification": true,
            "usePeerVerification": true
        }
    }
}

```

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md). En esta sección se proporciona una lista de las propiedades que admite el conjunto de datos de Salesforce Marketing Cloud.

Para copiar datos desde y hacia Salesforce Marketing Cloud, establezca la propiedad type del conjunto de datos en **SalesforceMarketingCloudObject**. Se admiten las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del conjunto de datos debe establecerse en: **SalesforceMarketingCloudObject** | Sí |
| tableName | Nombre de la tabla. | No (si se especifica "query" en el origen de la actividad) |

**Ejemplo**

```json
{
    "name": "SalesforceMarketingCloudDataset",
    "properties": {
        "type": "SalesforceMarketingCloudObject",
        "typeProperties": {},
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<SalesforceMarketingCloud linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades admitidas por el origen y el receptor de Salesforce Marketing Cloud.

### <a name="salesforce-marketing-cloud-as-source"></a>Salesforce Marketing Cloud como origen

Para copiar datos desde y hacia Salesforce Marketing Cloud, establezca el tipo de origen de la actividad de copia en **SalesforceMarketingCloudObject**. Se admiten las siguientes propiedades en la sección **source** de la actividad de copia:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del origen de la actividad de copia debe establecerse en: **SalesforceMarketingCloudSource** | Sí |
| Query | Use la consulta SQL personalizada para leer los datos. Por ejemplo: `"SELECT * FROM MyTable"`. | No (si se especifica "tableName" en el conjunto de datos) |

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyFromSalesforceMarketingCloud",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<SalesforceMarketingCloud input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "SalesforceMarketingCloudSource",
                "query": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="lookup-activity-properties"></a>Propiedades de la actividad de búsqueda

Para obtener información detallada sobre las propiedades, consulte [Actividad de búsqueda](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Pasos siguientes
Consulte los [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver la lista de almacenes de datos que la actividad de copia de Azure Data Factory admite como orígenes y receptores.
