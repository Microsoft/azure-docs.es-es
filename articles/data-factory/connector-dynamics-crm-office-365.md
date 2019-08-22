---
title: Copia de datos desde y hacia Dynamics CRM o Dynamics 365 (Common Data Service) mediante Azure Data Factory | Microsoft Docs
description: Aprenda a copiar datos de Microsoft Dynamics CRM o Microsoft Dynamics 365 (Common Data Service) en almacenes de datos receptores compatibles, o bien de almacenes de datos de origen compatibles en Dynamics CRM o Dynamics 365 a través de una actividad de copia en una canalización de Azure Data Factory.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 07/01/2019
ms.author: jingwang
ms.openlocfilehash: 3f7bf3ce8c01e82fa69b3b041b573b4b31a719d2
ms.sourcegitcommit: 6cb4dd784dd5a6c72edaff56cf6bcdcd8c579ee7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/02/2019
ms.locfileid: "67514094"
---
# <a name="copy-data-from-and-to-dynamics-365-common-data-service-or-dynamics-crm-by-using-azure-data-factory"></a>Copia de datos desde y hacia Dynamics 365 (Common Data Service) o Dynamics CRM mediante Azure Data Factory

En este artículo se explica el uso de la actividad de copia de Azure Data Factory para copiar datos desde y hacia Microsoft Dynamics 365 o Microsoft Dynamics CRM. El documento se basa en el artículo de [introducción a la actividad de copia](copy-activity-overview.md) que describe información general de esta actividad.

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Puede copiar datos de Dynamics 365 (Common Data Service) o Dynamics CRM en cualquier almacén de datos receptor compatible. También puede copiar datos de cualquier almacén de datos de origen compatible en Dynamics 365 (Common Data Service) o Dynamics CRM. Consulte la tabla de [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver una lista de almacenes de datos que la actividad de copia admite como orígenes o receptores.

Este conector de Dynamics es compatible con la versión 7.x a 9.x de Dynamics, tanto en línea como en el entorno local. Más concretamente:

- La versión 7.x se asigna a Dynamics CRM 2015
- La versión 8.x se asigna a Dynamics CRM 2016 y una versión anterior de Dynamics 365
- La versión 9.x se asigna a la versión más reciente de Dynamics 365

Consulte la tabla siguiente sobre los tipos y configuraciones de autenticación compatibles para las versiones o productos de Dynamics respectivos. IFD es la abreviatura en inglés del término "implementación con conexión a Internet".

| Versiones de Dynamics | Tipos de autenticación | Ejemplos de servicios vinculados |
|:--- |:--- |:--- |
| Dynamics 365 Online <br> Dynamics CRM Online | Office 365 | [Autenticación de Dynamics Online + Office 365](#dynamics-365-and-dynamics-crm-online) |
| Dynamics 365 local con IFD <br> Dynamics CRM 2016 local con IFD <br> Dynamics CRM 2015 local con IFD | IFD | [Dynamics local con autenticación de IFD + IFD](#dynamics-365-and-dynamics-crm-on-premises-with-ifd) |

Para Dynamics 365 en concreto, se admiten los siguientes tipos de aplicación:

- Dynamics 365 for Sales
- Dynamics 365 for Customer Service
- Dynamics 365 for Field Service
- Dynamics 365 for Project Service Automation
- Dynamics 365 for Marketing

Otros tipos de aplicaciones, por ejemplo, Finance and Operations, Talent, etc., no son compatibles con este conector.

Este conector de Dynamics se basa en las [herramientas de Dynamics XRM](https://docs.microsoft.com/dynamics365/customer-engagement/developer/build-windows-client-applications-xrm-tools).

>[!TIP]
>Para copiar datos desde **Dynamics 365 Finance and Operations**, puede usar el [conector de Dynamics AX](connector-dynamics-ax.md).

## <a name="get-started"></a>Primeros pasos

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

En las secciones siguientes se proporcionan detalles sobre las propiedades que se usan para definir entidades de Data Factory específicas de Dynamics.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado de Dynamics.

### <a name="dynamics-365-and-dynamics-crm-online"></a>Dynamics 365 y Dynamics CRM Online

| Propiedad | DESCRIPCIÓN | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type debe establecerse en: **Dynamics**. | Sí |
| deploymentType | El tipo de implementación de la instancia de Dynamics. Debe ser **"Online"** para Dynamics Online. | Sí |
| serviceUri | Dirección URL de la instancia de Dynamics, por ejemplo, `https://adfdynamics.crm.dynamics.com`. | Sí |
| authenticationType | Tipo de autenticación para conectarse a un servidor de Dynamics. Especifique **"Office365"** para Dynamics Online. | Sí |
| username | Especifique el nombre de usuario para conectarse a Dynamics. | Sí |
| password | Especifique la contraseña de la cuenta de usuario que especificó para el nombre de usuario. Marque este campo como SecureString para almacenarlo de forma segura en Data Factory o [para hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). | Sí |
| connectVia | El [entorno de ejecución de integración](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Si no se especifica, se usará Azure Integration Runtime. | "No" para el origen, "Sí" para el receptor si el servicio vinculado al origen no tiene ningún entorno de ejecución de integración. |

>[!NOTE]
>El conector de Dynamics permite utilizar la propiedad opcional "organizationName" para identificar la instancia de Dynamics CRM/365 Online. Mientras siga funcionando, es preferible especificar la nueva propiedad "serviceUri" en su lugar para un mejor rendimiento de detección de la instancia.

**Ejemplo: Dynamics Online mediante la autenticación de Office 365**

```json
{
    "name": "DynamicsLinkedService",
    "properties": {
        "type": "Dynamics",
        "description": "Dynamics online linked service using Office365 authentication",
        "typeProperties": {
            "deploymentType": "Online",
            "serviceUri": "https://adfdynamics.crm.dynamics.com",
            "authenticationType": "Office365",
            "username": "test@contoso.onmicrosoft.com",
            "password": {
                "type": "SecureString",
                "value": "<password>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="dynamics-365-and-dynamics-crm-on-premises-with-ifd"></a>Dynamics 365 y Dynamics CRM local con IFD

*Las propiedades adicionales, en comparación con Dynamics Online, son "hostName" y "port".*

| Propiedad | DESCRIPCIÓN | Obligatorio |
|:--- |:--- |:--- |
| Tipo | La propiedad type debe establecerse en: **Dynamics**. | Sí |
| deploymentType | El tipo de implementación de la instancia de Dynamics. Debe ser **"OnPremisesWithIfd"** para Dynamic local con IFD.| Sí |
| hostName | El nombre de host del servidor local de Dynamics. | Sí |
| puerto | El puerto del servidor local de Dynamics. | No (el valor predeterminado es 443) |
| organizationName | El nombre de la organización de la instancia de Dynamics. | Sí |
| authenticationType | Tipo de autenticación para conectarse al servidor de Dynamics. Especifique **"Ifd"** para Dynamics local con IFD. | Sí |
| nombre de usuario | Especifique el nombre de usuario para conectarse a Dynamics. | Sí |
| contraseña | Especifique la contraseña de la cuenta de usuario que especificó para el nombre de usuario. Puede elegir marcar este campo como SecureString para almacenarlo de forma segura en ADF o almacenar la contraseña en Azure Key Vault y permitir que la actividad de copia incorpore los cambios desde allí al realizar la copia de datos. Obtenga más información sobre el [Almacenamiento de credenciales en Key Vault](store-credentials-in-key-vault.md). | Sí |
| connectVia | El [entorno de ejecución de integración](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Si no se especifica, se usará Azure Integration Runtime. | No para el origen, sí para el receptor |

**Ejemplo: Dynamics local con IFD mediante la autenticación de IFD**

```json
{
    "name": "DynamicsLinkedService",
    "properties": {
        "type": "Dynamics",
        "description": "Dynamics on-premises with IFD linked service using IFD authentication",
        "typeProperties": {
            "deploymentType": "OnPremisesWithIFD",
            "hostName": "contosodynamicsserver.contoso.com",
            "port": 443,
            "organizationName": "admsDynamicsTest",
            "authenticationType": "Ifd",
            "username": "test@contoso.onmicrosoft.com",
            "password": {
                "type": "SecureString",
                "value": "<password>"
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

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md). En esta sección se proporciona una lista de las propiedades que admite el conjunto de datos de Dynamics.

Para copiar datos desde y hacia Dynamics, establezca la propiedad type del conjunto de datos en **DynamicsEntity**. Se admiten las siguientes propiedades.

| Propiedad | DESCRIPCIÓN | Obligatorio |
|:--- |:--- |:--- |
| Tipo | La propiedad type del conjunto de datos debe establecerse en: **DynamicsEntity**. |Sí |
| entityName | El nombre lógico de la entidad que se va a recuperar. | No para el origen (si se especifica "query" en el origen de la actividad); sí para el receptor |

> [!IMPORTANT]
>- Cuando se copian datos desde Dynamics, la sección "structure" es opcional, aunque se recomienda encarecidamente en el conjunto de datos de Dynamics para garantizar un resultado de copia determinista. Esta define el nombre de columna y el tipo de datos de los datos de Dynamics que desea copiar. Para más información, consulte [Estructura del conjunto de datos](concepts-datasets-linked-services.md#dataset-structure-or-schema) y [Asignación de tipos de datos de Dynamics](#data-type-mapping-for-dynamics).
>- Cuando se importa el esquema en la interfaz de usuario de creación, ADF lo deduce mediante un muestreo de las primeras filas de los resultados de la consulta de Dynamics a fin de inicializar la construcción de la estructura, en cuyo caso se omiten las columnas sin valores. El mismo comportamiento se aplica a las ejecuciones de copia si no existe una definición de estructura explícita. Puede revisar y agregar más columnas al esquema o la estructura del conjunto de datos de Dynamics si es necesario, lo que se respeta durante el runtime de copia.
>- Al copiar datos a Dynamics, la sección "structure" es opcional en el conjunto de datos de Dynamics. El esquema de datos de origen determinará qué columnas se copian. Si el origen es un archivo CSV sin encabezado, en el conjunto de datos de entrada, especifique "structure" con el tipo de datos y el nombre de columna. Estos se asignan a los campos del archivo CSV, uno por uno en orden.

**Ejemplo:**

```json
{
    "name": "DynamicsDataset",
    "properties": {
        "type": "DynamicsEntity",
        "structure": [
            {
                "name": "accountid",
                "type": "Guid"
            },
            {
                "name": "name",
                "type": "String"
            },
            {
                "name": "marketingonly",
                "type": "Boolean"
            },
            {
                "name": "modifiedon",
                "type": "Datetime"
            }
        ],
        "typeProperties": {
            "entityName": "account"
        },
        "linkedServiceName": {
            "referenceName": "<Dynamics linked service name>",
            "type": "linkedservicereference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades que admite el tipo de origen y el tipo de receptor de Dynamics.

### <a name="dynamics-as-a-source-type"></a>Dynamics como tipo de origen

Para copiar datos de Dynamics, establezca el tipo de origen de la actividad de copia en **DynamicsSource**. En la sección **source** de la actividad de copia se admiten las siguientes propiedades.

| Propiedad | DESCRIPCIÓN | Obligatorio |
|:--- |:--- |:--- |
| Tipo | La propiedad type del origen de la actividad de copia debe establecerse en: **DynamicsSource**. | Sí |
| query | FetchXML es un lenguaje de consulta patentado que se usa en Dynamics (Online y local). Consulte el ejemplo siguiente. Para más información, consulte [Build queries with FetchXML](https://msdn.microsoft.com/library/gg328332.aspx) (Creación de consultas con FetchXML). | No (si se especifica "entityName" en el conjunto de datos) |

>[!NOTE]
>La columna PK siempre se copiará incluso si la proyección de columna que se configura en la consulta de FetchXML no la contiene.

**Ejemplo:**

```json
"activities":[
    {
        "name": "CopyFromDynamics",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Dynamics input dataset>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "DynamicsSource",
                "query": "<FetchXML Query>"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="sample-fetchxml-query"></a>Consulta FetchXML de ejemplo

```xml
<fetch>
  <entity name="account">
    <attribute name="accountid" />
    <attribute name="name" />
    <attribute name="marketingonly" />
    <attribute name="modifiedon" />
    <order attribute="modifiedon" descending="false" />
    <filter type="and">
      <condition attribute ="modifiedon" operator="between">
        <value>2017-03-10 18:40:00z</value>
        <value>2017-03-12 20:40:00z</value>
      </condition>
    </filter>
  </entity>
</fetch>
```

### <a name="dynamics-as-a-sink-type"></a>Dynamics como tipo de receptor

Para copiar datos en Dynamics, establezca el tipo de receptor de la actividad de copia en **DynamicsSink**. En la sección **sink** de la actividad de copia se admiten las siguientes propiedades.

| Propiedad | DESCRIPCIÓN | Obligatorio |
|:--- |:--- |:--- |
| Tipo | La propiedad type del receptor de la actividad de copia debe establecerse en: **DynamicsSink**. | Sí |
| writeBehavior | El comportamiento de escritura de la operación.<br/>El valor permitido es **"Upsert"** . | Sí |
| writeBatchSize | El recuento de filas de datos escritos en Dynamics en cada lote. | No (el valor predeterminado es 10) |
| ignoreNullValues | Indica si se omiten los valores nulos de los datos de entrada (excepto los campos de clave) durante la operación de escritura.<br/>Los valores permitidos son **true** y **false**.<br>- **True**: deja los datos del objeto de destino sin cambiar cuando realiza una operación upsert/update. Inserta un valor predeterminado definido al realizar una operación insert.<br/>- **False**: actualiza los datos del objeto de destino a NULL cuando realiza una operación upsert/update. Inserta un valor NULL al realizar una operación insert. | No (el valor predeterminado es false) |

>[!NOTE]
>El valor predeterminado del receptor "**writeBatchSize**" y el de la actividad de copia " **[parallelCopies](copy-activity-performance.md#parallel-copy)** " del receptor de Dynamics es 10 en ambos casos. Por lo tanto, se enviarán 100 registros a Dynamics simultáneamente.

Para Dynamics 365 en línea, hay un límite de [2 llamadas simultáneas por lotes por organización](https://msdn.microsoft.com/library/jj863631.aspx#Run-time%20limitations). Si se supera dicho límite, se produce un error de "Servidor ocupado" antes de que la primera solicitud se haya ejecutado siquiera. Mantener el valor "writeBatchSize" menor o igual a 10 evitará tal limitación de las llamadas simultáneas.

La combinación óptima de "**writeBatchSize**" y "**parallelCopies**" depende del esquema de la entidad; por ejemplo, el número de columnas, el tamaño de fila o el número de complementos, flujos de trabajo o actividades de flujo de trabajo enlazados a esas llamadas, etc. El valor predeterminado de 10 writeBatchSize * 10 parallelCopies es el recomendado de acuerdo con el servicio Dynamics, y funcionará con la mayoría de entidades de Dynamics, si bien podría no proporcionar el mejor rendimiento. Para optimizar el rendimiento, ajuste la combinación en su configuración de la actividad de copia.

**Ejemplo:**

```json
"activities":[
    {
        "name": "CopyToDynamics",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Dynamics output dataset>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "DynamicsSink",
                "writeBehavior": "Upsert",
                "writeBatchSize": 10,
                "ignoreNullValues": true
            }
        }
    }
]
```

## <a name="data-type-mapping-for-dynamics"></a>Asignación de tipos datos de Dynamics

Al copiar datos desde Dynamics, se utilizan las siguientes asignaciones de tipos de datos de Dynamics en los tipos de datos provisionales de Data Factory. Consulte el artículo sobre [asignaciones de tipos de datos y esquema](copy-activity-schema-and-type-mapping.md) para más información sobre cómo la actividad de copia asigna el tipo de datos y el esquema de origen al receptor.

Configure el tipo de datos de Data Factory correspondiente en la estructura del conjunto de datos en función del tipo de datos de Dynamics de origen mediante la siguiente tabla de asignación.

| Tipo de datos de Dynamics | Tipo de datos provisionales de Data Factory | Se admite como origen | Se admite como receptor |
|:--- |:--- |:--- |:--- |
| AttributeTypeCode.BigInt | long | ✓ | ✓ |
| AttributeTypeCode.Boolean | Boolean | ✓ | ✓ |
| AttributeType.Customer | Guid | ✓ | |
| AttributeType.DateTime | Datetime | ✓ | ✓ |
| AttributeType.Decimal | Decimal | ✓ | ✓ |
| AttributeType.Double | Double | ✓ | ✓ |
| AttributeType.EntityName | Cadena | ✓ | ✓ |
| AttributeType.Integer | Int32 | ✓ | ✓ |
| AttributeType.Lookup | Guid | ✓ | ✓ (con el destino único asociado) |
| AttributeType.ManagedProperty | Boolean | ✓ | |
| AttributeType.Memo | Cadena | ✓ | ✓ |
| AttributeType.Money | Decimal | ✓ | ✓ |
| AttributeType.Owner | Guid | ✓ | |
| AttributeType.Picklist | Int32 | ✓ | ✓ |
| AttributeType.Uniqueidentifier | Guid | ✓ | ✓ |
| AttributeType.String | Cadena | ✓ | ✓ |
| AttributeType.State | Int32 | ✓ | ✓ |
| AttributeType.Status | Int32 | ✓ | ✓ |

> [!NOTE]
> No se admiten los tipos de datos de AttributeType.CalendarRules, AttributeType.MultiSelectPicklist y AttributeType.PartyList.

## <a name="next-steps"></a>Pasos siguientes
Para ver la lista de almacenes de datos que la actividad de copia de Data Factory admite como orígenes y receptores consulte [Almacenes de datos y formatos que se admiten](copy-activity-overview.md#supported-data-stores-and-formats).
