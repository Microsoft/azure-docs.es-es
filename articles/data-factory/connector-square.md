---
title: Copia de datos de Square (versión preliminar)
description: Obtenga información sobre cómo copiar datos de Square en almacenes de datos receptores compatibles a través de una actividad de copia de una canalización de Azure Data Factory.
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
ms.date: 08/03/2020
ms.openlocfilehash: 2bfe9115f38c79618924379837dda8014ee31ed5
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87529371"
---
# <a name="copy-data-from-square-using-azure-data-factory-preview"></a>Copiar datos de Square con Azure Data Factory (versión preliminar)
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se explica el uso de la actividad de copia de Azure Data Factory para copiar datos de Square. El documento se basa en el artículo de [introducción a la actividad de copia](copy-activity-overview.md) que describe información general de la actividad de copia.

> [!IMPORTANT]
> Este conector está actualmente en versión preliminar. Puede probarlo y enviarnos sus comentarios. Si desea depender de los conectores de versión preliminar en la solución, póngase en contacto con el [soporte técnico de Azure](https://azure.microsoft.com/support/).

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Este conector Square es compatible con las actividades siguientes:

- [Actividad de copia](copy-activity-overview.md) con [matriz de origen o receptor compatible](copy-activity-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)

Puede copiar datos de Square en cualquier almacén de datos de receptor admitido. Consulte la tabla de [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver una lista de almacenes de datos que la actividad de copia admite como orígenes o receptores.

Azure Data Factory proporciona un controlador integrado para habilitar la conectividad. Por lo tanto, no es necesario instalar manualmente ningún controlador mediante este conector.

## <a name="getting-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

En las secciones siguientes se proporcionan detalles sobre las propiedades que se usan para definir entidades de Data Factory específicas del conector de Square.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado de Square:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type debe establecerse en: **Square** | Sí |
| connectionProperties | Grupo de propiedades que define cómo conectarse a Square. | Sí |
| ***En`connectionProperties`:*** | | |
| host | Dirección URL de la instancia de Square. (es decir, mystore.mysquare.com)  | Sí |
| clientId | Id. de cliente asociado a la aplicación Square.  | Sí |
| clientSecret | Secreto de cliente asociado a la aplicación Square. Marque este campo como SecureString para almacenarlo de forma segura en Data Factory o [para hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). | Sí |
| accessToken | Token de acceso obtenido de Square. Concede acceso limitado a una cuenta de Square solicitando permisos explícitos a un usuario autenticado. Los tokens de acceso de OAuth expiran 30 días después de emitirse, pero los tokens de actualización no expiran. Los tokens de acceso se pueden actualizar mediante un token de actualización.<br>Marque este campo como SecureString para almacenarlo de forma segura en Data Factory o [para hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md).  | Sí |
| refreshToken | Token de actualización obtenido de Square. Se usa para obtener nuevos tokens de acceso cuando expira el actual.<br>Marque este campo como SecureString para almacenarlo de forma segura en Data Factory o [para hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). | No |
| useEncryptedEndpoints | Especifica si los puntos de conexión de origen de datos se cifran mediante HTTPS. El valor predeterminado es true.  | No |
| useHostVerification | Especifica si se requiere que el nombre de host del certificado del servidor coincida con el nombre de host del servidor al conectarse a través de TLS. El valor predeterminado es true.  | No |
| usePeerVerification | Especifica si se debe verificar la identidad del servidor al conectarse a través de TLS. El valor predeterminado es true.  | No |

Square admite dos tipos de tokens de acceso: **personal** y **OAuth**.

- Los tokens de acceso personal se usan para que Connect API tenga acceso ilimitado a los recursos de su propia cuenta de Square.
- Los tokens de acceso de OAuth se usan para que Connect API tenga acceso autenticado y por ámbito a cualquier cuenta de Square. Úselos cuando la aplicación acceda a recursos de otras cuentas de Square en nombre de los propietarios de la cuenta. Los tokens de acceso de OAuth también se pueden usar para acceder a recursos de su propia cuenta de Square.

En Data Factory, la autenticación mediante el token de acceso personal solo necesita `accessToken`, mientras que la autenticación mediante OAuth necesita `accessToken` y `refreshToken`. Obtenga información sobre cómo recuperar el token de acceso [aquí](https://developer.squareup.com/docs/build-basics/access-tokens).

**Ejemplo**:

```json
{
    "name": "SquareLinkedService",
    "properties": {
        "type": "Square",
        "typeProperties": {
            "connectionProperties": {
                "host": "<e.g. mystore.mysquare.com>", 
                "clientId": "<client ID>", 
                "clientSecrect": {
                    "type": "SecureString",
                    "value": "<clientSecret>"
                }, 
                "accessToken": {
                    "type": "SecureString",
                    "value": "<access token>"
                }, 
                "refreshToken": {
                    "type": "SecureString",
                    "value": "<refresh token>"
                }, 
                "useEncryptedEndpoints": true, 
                "useHostVerification": true, 
                "usePeerVerification": true 
            }
        }
    }
}
```

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md). En esta sección se proporciona una lista de las propiedades compatibles con el conjunto de datos de Square.

Para copiar datos de Square, establezca la propiedad type del conjunto de datos en **SquareObject**. Se admiten las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del conjunto de datos debe establecerse en: **SquareObject** | Sí |
| tableName | Nombre de la tabla. | No (si se especifica "query" en el origen de la actividad) |

**Ejemplo**

```json
{
    "name": "SquareDataset",
    "properties": {
        "type": "SquareObject",
        "typeProperties": {},
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<Square linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades compatibles con el origen de Square.

### <a name="square-as-source"></a>Square como origen

Si va a copiar datos de Square, establezca el tipo de origen de la actividad de copia en **SquareSource**. Se admiten las siguientes propiedades en la sección **source** de la actividad de copia:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del origen de la actividad de copia debe establecerse en: **SquareSource** | Sí |
| Query | Use la consulta SQL personalizada para leer los datos. Por ejemplo: `"SELECT * FROM Business"`. | No (si se especifica "tableName" en el conjunto de datos) |

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyFromSquare",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Square input dataset name>",
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
                "type": "SquareSource",
                "query": "SELECT * FROM Business"
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
