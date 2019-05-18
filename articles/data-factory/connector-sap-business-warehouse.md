---
title: Copia de datos desde SAP BW mediante Azure Data Factory | Microsoft Docs
description: Obtenga información sobre cómo copiar datos desde SAP Business Warehouse a almacenes de datos receptores compatibles a través de una actividad de copia de una canalización de Azure Data Factory.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 08/07/2018
ms.author: jingwang
ms.openlocfilehash: 9a0abcd70b4aeb2369604bafa924136122206e0a
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2019
ms.locfileid: "60309120"
---
# <a name="copy-data-from-sap-business-warehouse-using-azure-data-factory"></a>Copia de datos desde SAP Business Warehouse mediante Azure Data Factory
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [Versión 1](v1/data-factory-sap-business-warehouse-connector.md)
> * [Versión actual](connector-sap-business-warehouse.md)

En este artículo se explica el uso de la actividad de copia de Azure Data Factory para copiar datos desde una instancia de SAP Business Warehouse (BW). El documento se basa en el artículo de [introducción a la actividad de copia](copy-activity-overview.md) que describe información general de la actividad de copia.

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Puede copiar los datos desde SAP Business Warehouse hasta cualquier almacén de datos de receptores que se admita. Consulte la tabla de [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver una lista de almacenes de datos que la actividad de copia admite como orígenes o receptores.

Concretamente, este conector SAP Business Warehouse admite:

- SAP Business Warehouse **versión 7.x**.
- La copia de datos de **InfoCubes y QueryCubes** (incluidas las consultas BEx) mediante consultas MDX.
- Copiar datos con la autenticación básica.

## <a name="prerequisites"></a>Requisitos previos

Para usar este conector SAP Business Warehouse, necesita hacer lo siguiente:

- Configurar un entorno Integration Runtime autohospedado. Consulte el artículo sobre [Integration Runtime autohospedado](create-self-hosted-integration-runtime.md) para más información.
- Instalar la **biblioteca SAP NetWeaver** en la máquina de Integration Runtime. Puede obtener la biblioteca SAP Netweaver desde el administrador de SAP o directamente desde el [centro de descarga de software de SAP](https://support.sap.com/swdc). Busque la **nota 1025361 de SAP** para obtener la ubicación de descarga de la versión más reciente. Asegúrese de elegir la biblioteca de SAP NetWeaver de **64 bits**, que coincida con la instalación de Integration Runtime. A continuación, instale todos los archivos incluidos en SAP NetWeaver RFC SDK según la nota de SAP. La biblioteca SAP NetWeaver también se incluye en la instalación de las herramientas de cliente de SAP.

>[!TIP]
>Para solucionar problemas de conectividad a SAP BW, asegúrese de lo siguiente:
>- Todas las bibliotecas de dependencias extraídas de NetWeaver RFC SDK están en la carpeta %windir%\system32. Normalmente contiene icudt34.dll, icuin34.dll, icuuc34.dll, libicudecnumber.dll, librfc32.dll, libsapucum.dll, sapcrypto.dll, sapcryto_old.dll, sapnwrfc.dll.
>- Los puertos necesarios usados para conectarse a SAP Server están habilitados en la máquina de IR autohospedado, que normalmente son 3300 y 3201.

## <a name="getting-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Las secciones siguientes proporcionan detalles sobre las propiedades que se usan para definir entidades de Data Factory específicas del conector SAP Business Warehouse.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado SAP Business Warehouse (BW):

| Propiedad | DESCRIPCIÓN | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type debe establecerse en: **SapBw** | Sí |
| server | Nombre del servidor en el que reside la instancia de SAP BW. | Sí |
| systemNumber | Número del sistema de SAP BW.<br/>Valor permitido: número decimal de dos dígitos que se representa en forma de cadena. | Sí |
| clientId | Identificador del cliente en el sistema SAP W.<br/>Valor permitido: número decimal de tres dígitos que se representa en forma de cadena. | Sí |
| userName | Nombre del usuario que tiene acceso al servidor SAP. | Sí |
| password | Contraseña del usuario. Marque este campo como SecureString para almacenarlo de forma segura en Data Factory o [para hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). | Sí |
| connectVia | El entorno [Integration Runtime](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Tal y como se mencionó en los [requisitos previos](#prerequisites), se requiere un entorno Integration Runtime autohospedado. |Sí |

**Ejemplo:**

```json
{
    "name": "SapBwLinkedService",
    "properties": {
        "type": "SapBw",
        "typeProperties": {
            "server": "<server name>",
            "systemNumber": "<system number>",
            "clientId": "<client id>",
            "userName": "<SAP user>",
            "password": {
                "type": "SecureString",
                "value": "<Password for SAP user>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre conjuntos de datos. En esta sección se proporciona una lista de las propiedades que admite el conjunto de datos de SAP BW.

Para copiar datos desde SAP BW, establezca la propiedad type del conjunto de datos en **RelationalTable**. No hay ninguna propiedad específica del tipo compatible con el conjunto de datos de SAP BW de tipo RelationalTable.

**Ejemplo:**

```json
{
    "name": "SAPBWDataset",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": {
            "referenceName": "<SAP BW linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {}
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades que admite el origen de SAP BW.

### <a name="sap-bw-as-source"></a>SAP BW como origen

Para copiar datos desde SAP BW, establezca el tipo de origen de la actividad de copia como **RelationalSource**. Se admiten las siguientes propiedades en la sección **source** de la actividad de copia:

| Propiedad | DESCRIPCIÓN | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del origen de la actividad de copia debe establecerse en: **RelationalSource** | Sí |
| query | Especifica la consulta MDX para leer datos de la instancia de SAP BW. | Sí |

**Ejemplo:**

```json
"activities":[
    {
        "name": "CopyFromSAPBW",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<SAP BW input dataset name>",
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
                "type": "RelationalSource",
                "query": "<MDX query for SAP BW>"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="data-type-mapping-for-sap-bw"></a>Asignación de tipos de datos para SAP BW

Al copiar datos desde SAP BW, se usan las siguientes asignaciones de tipos de datos de SAP BW en los tipos de datos provisionales de Azure Data Factory. Consulte el artículo sobre [asignaciones de tipos de datos y esquema](copy-activity-schema-and-type-mapping.md) para información sobre cómo la actividad de copia asigna el tipo de datos y el esquema de origen al receptor.

| Tipo de datos de SAP BW | Tipo de datos provisionales de Data Factory |
|:--- |:--- |
| ACCP | Int |
| CHAR | String |
| CLNT | String |
| CURR | Decimal |
| CUKY | String |
| DEC | Decimal |
| FLTP | Double |
| INT1 | Byte |
| INT2 | Int16 |
| INT4 | Int |
| LANG | String |
| LCHR | String |
| LRAW | Byte[] |
| PREC | Int16 |
| QUAN | Decimal |
| RAW | Byte[] |
| RAWSTRING | Byte[] |
| STRING | String |
| UNIT | String |
| DATS | String |
| NUMC | String |
| TIMS | String |


## <a name="next-steps"></a>Pasos siguientes
Consulte los [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver la lista de almacenes de datos que la actividad de copia de Azure Data Factory admite como orígenes y receptores.
