---
title: Copia de datos de QuickBooks Online con Azure Data Factory (versión preliminar) | Microsoft Docs
description: Obtenga información sobre cómo copiar datos de QuickBooks Online en almacenes de datos receptores compatibles a través de una actividad de copia de una canalización de Azure Data Factory.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 12/07/2018
ms.author: jingwang
ms.openlocfilehash: 8f5e3958588a597bde04ae1c8e4873006b281458
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2019
ms.locfileid: "60405824"
---
# <a name="copy-data-from-quickbooks-online-using-azure-data-factory-preview"></a>Copia de datos de QuickBooks Online con Azure Data Factory (versión preliminar)

En este artículo se explica el uso de la actividad de copia de Azure Data Factory para copiar datos de QuickBooks One. El documento se basa en el artículo de [introducción a la actividad de copia](copy-activity-overview.md) que describe información general de la actividad de copia.

> [!IMPORTANT]
> Este conector está actualmente en versión preliminar. Puede probarlo y enviarnos sus comentarios. Si desea depender de los conectores de versión preliminar en la solución, póngase en contacto con el [soporte técnico de Azure](https://azure.microsoft.com/support/).

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Puede copiar datos de QuickBooks Online en cualquier almacén de datos de receptor compatible. Consulte la tabla de [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver una lista de almacenes de datos que la actividad de copia admite como orígenes o receptores.

Azure Data Factory proporciona un controlador integrado para habilitar la conectividad. Por lo tanto, no es necesario instalar manualmente ningún controlador mediante este conector.

Actualmente este conector solo es compatible con 1.0a, lo que significa que el usuario debe tener una cuenta de desarrollador con aplicaciones creadas antes del 17 de julio de 2017.

## <a name="getting-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

En las secciones siguientes se proporcionan detalles sobre las propiedades que se usan para definir entidades de Data Factory específicas para el conector QuickBooks.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado de QuickBooks:

| Propiedad | DESCRIPCIÓN | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type debe establecerse en: **QuickBooks** | Sí |
| endpoint | Punto de conexión del servidor de QuickBooks Online. (es decir, quickbooks.api.intuit.com)  | Sí |
| companyId | Id. de empresa de la compañía de QuickBooks que se debe autorizar. Para obtener información sobre cómo buscar el identificador de empresa, consulte [How do I find my Company ID?](https://quickbooks.intuit.com/community/Getting-Started/How-do-I-find-my-Company-ID/m-p/185551) (¿Cómo puedo buscar el identificador de empresa?). | Sí |
| consumerKey | La clave del consumidor para la autenticación OAuth 1.0. | Sí |
| consumerSecret | Secreto del consumidor para la autenticación OAuth 1.0. Marque este campo como SecureString para almacenarlo de forma segura en Data Factory o [para hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). | Sí |
| accessToken | Token de acceso para la autenticación de OAuth 1.0. Marque este campo como SecureString para almacenarlo de forma segura en Data Factory o [para hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). | Sí |
| accessTokenSecret | Secreto del token de acceso para la autenticación de OAuth 1.0. Marque este campo como SecureString para almacenarlo de forma segura en Data Factory o [para hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). | Sí |
| useEncryptedEndpoints | Especifica si los puntos de conexión de origen de datos se cifran mediante HTTPS. El valor predeterminado es true.  | Sin  |

**Ejemplo:**

```json
{
    "name": "QuickBooksLinkedService",
    "properties": {
        "type": "QuickBooks",
        "typeProperties": {
            "endpoint" : "quickbooks.api.intuit.com",
            "companyId" : "<companyId>",
            "consumerKey": "<consumerKey>",
            "consumerSecret": {
                "type": "SecureString",
                "value": "<consumerSecret>"
            },
            "accessToken": {
                 "type": "SecureString",
                 "value": "<accessToken>"
            },
            "accessTokenSecret": {
                 "type": "SecureString",
                 "value": "<accessTokenSecret>"
            },
            "useEncryptedEndpoints" : true
        }
    }
}
```

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md). En esta sección se proporciona una lista de las propiedades compatibles con el conjunto de datos de QuickBooks.

Para copiar datos de QuickBooks Online, establezca la propiedad type del conjunto de datos en **QuickBooksObject**. Se admiten las siguientes propiedades:

| Propiedad | DESCRIPCIÓN | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del conjunto de datos debe establecerse en: **QuickBooksObject** | Sí |
| tableName | Nombre de la tabla. | No (si se especifica "query" en el origen de la actividad) |

**Ejemplo**

```json
{
    "name": "QuickBooksDataset",
    "properties": {
        "type": "QuickBooksObject",
        "linkedServiceName": {
            "referenceName": "<QuickBooks linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {}
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades compatibles con el origen de QuickBooks.

### <a name="quickbooks-as-source"></a>QuickBooks como origen

Para copiar datos de QuickBooks Online, establezca el tipo de origen de la actividad de copia en **QuickBooksSource**. Se admiten las siguientes propiedades en la sección **source** de la actividad de copia:

| Propiedad | DESCRIPCIÓN | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del origen de la actividad de copia debe establecerse en: **QuickBooksSource** | Sí |
| query | Use la consulta SQL personalizada para leer los datos. Por ejemplo: `"SELECT * FROM "Bill" WHERE Id = '123'"`. | No (si se especifica "tableName" en el conjunto de datos) |

**Ejemplo:**

```json
"activities":[
    {
        "name": "CopyFromQuickBooks",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<QuickBooks input dataset name>",
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
                "type": "QuickBooksSource",
                "query": "SELECT * FROM \"Bill\" WHERE Id = '123' "
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```
## <a name="copy-data-from-quickbooks-desktop"></a>Copia de datos de QuickBooks Desktop

La actividad de copia en Azure Data Factory no puede copiar datos directamente desde Quickbooks Desktop. Para copiar datos desde Quickbooks Desktop, exporte los datos de Quickbooks en un archivo de valores separados por comas (CSV) y, a continuación, cargue el archivo en Azure Blob Storage. Desde ahí puede usar Data Factory para copiar los datos al receptor que prefiera.

## <a name="next-steps"></a>Pasos siguientes
Consulte los [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver la lista de almacenes de datos que la actividad de copia de Azure Data Factory admite como orígenes y receptores.
