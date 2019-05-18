---
title: Copia de datos de ServiceNow con Azure Data Factory (beta) | Microsoft Docs
description: Obtenga información sobre cómo copiar datos de ServiceNow en almacenes de datos receptores compatibles a través de una actividad de copia de una canalización de Azure Data Factory.
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
ms.openlocfilehash: 234b78a97c2663121d0d585154695887a58b9522
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2019
ms.locfileid: "60203421"
---
# <a name="copy-data-from-servicenow-using-azure-data-factory"></a>Copia de datos de ServiceNow con Azure Data Factory

En este artículo se explica el uso de la actividad de copia de Azure Data Factory para copiar datos de ServiceNow. El documento se basa en el artículo de [introducción a la actividad de copia](copy-activity-overview.md) que describe información general de la actividad de copia.

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Puede copiar datos de ServiceNow en cualquier almacén de datos de receptor compatible. Consulte la tabla de [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver una lista de almacenes de datos que la actividad de copia admite como orígenes o receptores.

Azure Data Factory proporciona un controlador integrado para habilitar la conectividad. Por lo tanto, no es necesario instalar manualmente ningún controlador mediante este conector.

## <a name="getting-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

En las secciones siguientes se proporcionan detalles sobre las propiedades que se usan para definir entidades de Data Factory específicas para el conector de ServiceNow.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado de ServiceNow:

| Propiedad | DESCRIPCIÓN | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type debe establecerse en: **ServiceNow** | Sí |
| endpoint | El punto de conexión del servidor de ServiceNow (`http://<instance>.service-now.com`).  | Sí |
| authenticationType | Tipo de autenticación que se debe usar. <br/>Los valores permitidos son: **Basic** y **OAuth2** | Sí |
| username | Nombre de usuario utilizado para conectarse al servidor de ServiceNow para la autenticación Basic y OAuth2.  | Sí |
| password | Contraseña correspondiente al nombre de usuario para la autenticación Basic y OAuth2. Marque este campo como SecureString para almacenarlo de forma segura en Data Factory o [para hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). | Sí |
| clientId | Id. de cliente para la autenticación OAuth2.  | Sin  |
| clientSecret | Secreto de cliente para la autenticación OAuth2. Marque este campo como SecureString para almacenarlo de forma segura en Data Factory o [para hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). | Sin  |
| useEncryptedEndpoints | Especifica si los puntos de conexión de origen de datos se cifran mediante HTTPS. El valor predeterminado es true.  | Sin  |
| useHostVerification | Especifica si se requiere que el nombre de host del certificado del servidor coincida con el nombre de host del servidor al conectarse a través de SSL. El valor predeterminado es true.  | Sin  |
| usePeerVerification | Especifica si se debe verificar la identidad del servidor al conectarse a través de SSL. El valor predeterminado es true.  | Sin  |

**Ejemplo:**

```json
{
    "name": "ServiceNowLinkedService",
    "properties": {
        "type": "ServiceNow",
        "typeProperties": {
            "endpoint" : "http://<instance>.service-now.com",
            "authenticationType" : "Basic",
            "username" : "<username>",
            "password": {
                 "type": "SecureString",
                 "value": "<password>"
            }
        }
    }
}
```

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md). En esta sección se proporciona una lista de las propiedades compatibles con el conjunto de datos de ServiceNow.

Para copiar datos de ServiceNow, establezca la propiedad type del conjunto de datos en **ServiceNowObject**. Se admiten las siguientes propiedades:

| Propiedad | DESCRIPCIÓN | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del conjunto de datos debe establecerse en: **ServiceNowObject** | Sí |
| tableName | Nombre de la tabla. | No (si se especifica "query" en el origen de la actividad) |

**Ejemplo**

```json
{
    "name": "ServiceNowDataset",
    "properties": {
        "type": "ServiceNowObject",
        "linkedServiceName": {
            "referenceName": "<ServiceNow linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {}
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades compatibles con el origen de ServiceNow.

### <a name="servicenow-as-source"></a>ServiceNow como origen

Para copiar datos de ServiceNow, establezca el tipo de origen de la actividad de copia en **ServiceNowSource**. Se admiten las siguientes propiedades en la sección **source** de la actividad de copia:

| Propiedad | DESCRIPCIÓN | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del origen de la actividad de copia debe establecerse en: **ServiceNowSource** | Sí |
| query | Use la consulta SQL personalizada para leer los datos. Por ejemplo: `"SELECT * FROM Actual.alm_asset"`. | No (si se especifica "tableName" en el conjunto de datos) |

Tenga en cuenta lo siguiente cuando especifique el esquema y la columna para ServiceNow en la consulta, y **consulte los [consejos de rendimiento](#performance-tips) en la implicación de rendimiento de copia**.

- **Esquema:** especifica el esquema como `Actual` o `Display` en la consulta de ServiceNow, que puede considerar como el parámetro de `sysparm_display_value` verdadero o falso al llamar a las [API Restful de ServiceNow](https://developer.servicenow.com/app.do#!/rest_api_doc?v=jakarta&id=r_AggregateAPI-GET). 
- **Columna:** el nombre de columna del valor real del esquema `Actual` es `[column name]_value`, mientras que el valor de visualización del esquema `Display` es `[column name]_display_value`. Tenga en cuenta que el nombre de columna se debe asignar al esquema que se usa en la consulta.

**Consulta de ejemplo:**
`SELECT col_value FROM Actual.alm_asset` OR 
`SELECT col_display_value FROM Display.alm_asset`

**Ejemplo:**

```json
"activities":[
    {
        "name": "CopyFromServiceNow",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<ServiceNow input dataset name>",
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
                "type": "ServiceNowSource",
                "query": "SELECT * FROM Actual.alm_asset"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```
## <a name="performance-tips"></a>Consejos de rendimiento

### <a name="schema-to-use"></a>Esquema que se va a usar

ServiceNow tiene dos esquemas diferentes: uno se denomina **"Real"**, ya que devuelve los datos reales y el otro se denomina **"Mostrar"**, ya que devuelve los valores de visualización de datos. 

Si tiene un filtro en su consulta, use el esquema "Real" que tenga un mejor rendimiento de copia. Al realizar una consulta con el esquema "Real", ServiceNow admite de forma nativa el filtro al obtener los datos que usará para devolver solo el conjunto de resultados filtrado; asimismo, al consultar el esquema "Mostrar", ADF recupera todos los datos y aplica el filtro internamente.

### <a name="index"></a>Índice

El índice de la tabla de ServiceNow puede ayudarle a mejorar el rendimiento de las consultas; para ello, consulte [Create a table index](https://docs.servicenow.com/bundle/geneva-servicenow-platform/page/administer/table_administration/task/t_CreateCustomIndex.html) (Crear un índice de tabla).

## <a name="next-steps"></a>Pasos siguientes
Consulte los [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver la lista de almacenes de datos que la actividad de copia de Azure Data Factory admite como orígenes y receptores.
