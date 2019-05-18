---
title: Copia de datos a Azure Data Lake Storage Gen1 o desde él mediante Azure Data Factory | Microsoft Docs
description: Aprenda a copiar datos desde almacenes de datos de origen compatibles a Azure Data Lake Store o desde Data Lake Store a almacenes de receptores compatibles mediante Data Factory.
services: data-factory
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: ''
ms.devlang: ''
ms.topic: conceptual
ms.date: 05/13/2019
ms.author: jingwang
ms.openlocfilehash: 94fb3fbe9def034dc36467d219cb3e8b45c40dad
ms.sourcegitcommit: 179918af242d52664d3274370c6fdaec6c783eb6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65560610"
---
# <a name="copy-data-to-or-from-azure-data-lake-storage-gen1-by-using-azure-data-factory"></a>Copia de datos con Azure Data Lake Storage Gen1 como origen o destino mediante Azure Data Factory
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [Versión 1](v1/data-factory-azure-datalake-connector.md)
> * [Versión actual](connector-azure-data-lake-store.md)

Este artículo describe cómo copiar datos hacia y desde Azure Data Lake Storage Gen1 (Gen1 ADLS). Para información sobre Azure Data Factory, lea el [artículo de introducción](introduction.md).

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Este conector de Azure Data Lake Storage Gen1 es compatible con las siguientes actividades:

- [Actividad de copia](copy-activity-overview.md) con [admite la matriz de origen/receptor](copy-activity-overview.md)
- [Asignación de flujo de datos](concepts-data-flow-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)
- [Actividad GetMetadata](control-flow-get-metadata-activity.md)

En concreto, este conector admite las siguientes funcionalidades:

- La copia de archivos mediante uno de los siguientes métodos de autenticación: **entidad de servicio** o **identidades administradas para recursos de Azure**.
- La copia de archivos tal cual, o bien el análisis o generación de archivos con los [códecs de compresión y los formatos de archivo compatibles](supported-file-formats-and-compression-codecs.md).

> [!IMPORTANT]
> Si copia datos con el entorno de ejecución de integración autohospedado, configure el firewall corporativo para que permita el tráfico saliente a `<ADLS account name>.azuredatalakestore.net` y a `login.microsoftonline.com/<tenant>/oauth2/token` en el puerto 443. El último es Azure Security Token Service, con el que el entorno de ejecución de integración necesita comunicarse para obtener el token de acceso.

## <a name="get-started"></a>Introducción

> [!TIP]
> Para ver un tutorial del uso de conector de Azure Data Lake Store, consulte [Carga de datos en Azure Data Lake Storage Gen1 mediante Azure Data Factory](load-azure-data-lake-store.md).

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

En las secciones siguientes se proporcionan detalles acerca de las propiedades que se usan para definir entidades de Data Factory específicas para Azure Data Lake Store.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado de Azure Data Lake Store:

| Propiedad | DESCRIPCIÓN | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad `type` debe establecerse en **AzureDataLakeStore**. | Sí |
| dataLakeStoreUri | Información sobre la cuenta de Azure Data Lake Store. Esta información adopta uno de los siguientes formatos: `https://[accountname].azuredatalakestore.net/webhdfs/v1` o `adl://[accountname].azuredatalakestore.net/`. | Sí |
| subscriptionId | El identificador de la suscripción de Azure al que pertenece la cuenta de Data Lake Store. | Necesario para el receptor |
| resourceGroupName | El nombre del grupo de recursos de Azure al que pertenece la cuenta de Data Lake Store. | Necesario para el receptor |
| connectVia | El [entorno de ejecución de integración](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Puede usar el entorno de ejecución de integración de Azure o el entorno de ejecución de integración autohospedado (si el almacén de datos se encuentra en una red privada). Si no se especifica esta propiedad, usa el entorno de ejecución de integración de Azure predeterminado. |Sin  |

### <a name="use-service-principal-authentication"></a>Uso de la autenticación de entidad de servicio

Para usar la autenticación de entidad de servicio, registre una entidad de aplicación en Azure Active Directory y concédale acceso a Data Lake Store. Consulte [Autenticación entre servicios](../data-lake-store/data-lake-store-authenticate-using-active-directory.md) para ver los pasos detallados. Anote los siguientes valores; los usará para definir el servicio vinculado:

- Identificador de aplicación
- Clave de la aplicación
- Id. de inquilino

>[!IMPORTANT]
> Asegúrese de que concede el permiso adecuado a la entidad de servicio en Data Lake Store:
>- **Como origen**: En **Explorador de datos** > **Acceso**, conceda al menos permiso de **lectura y ejecución** permiso para enumerar y copiar los archivos de las carpetas y subcarpetas. O bien, puede conceder permiso de **lectura** para copiar un único archivo. Puede elegir agregarlo a **Esta carpeta y todos los elementos secundarios** si desea que sea recursivo y agregarlo como **un permiso de acceso y una entrada de permiso predeterminada**. No hay ningún requisito en el control de acceso de nivel de cuenta (IAM).
>- **Como receptor**: En **Explorador de datos** > **Acceso**, conceda al menos permiso de **escritura y ejecución** para crear elementos secundarios en la carpeta. Puede elegir agregarlo a **Esta carpeta y todos los elementos secundarios** si desea que sea recursivo y agregarlo como **un permiso de acceso y una entrada de permiso predeterminada**. Si usa Azure Integration Runtime para realizar la copia (tanto el origen como el receptor están en la nube), en IAM, conceda al menos el rol de **lector** para que Data Factory pueda detectar la región de Data Lake Store. Si desea no usar este rol de IAM, [cree explícitamente una instancia de Azure Integration Runtime](create-azure-integration-runtime.md#create-azure-ir) con la ubicación de Data Lake Store. Por ejemplo, si el Store del lago de datos está en Europa occidental, crear un entorno de ejecución de integración de Azure con la ubicación establecida en "Europa occidental". Asociar ellos en el servicio vinculado de Data Lake Store como en el ejemplo siguiente.

>[!NOTE]
>En la lista de carpetas que comienza en la raíz, debe establecer el permiso de la entidad de servicio que se concede **a nivel raíz con el permiso de "ejecución"**. Esto es válido cuando se usa la:
>- **Herramienta Copy Data** para crear la canalización de la copia.
>- **Interfaz de usuario de Data Factory** para probar la conexión y el desplazamiento por las carpetas durante la creación.
>Si dispone de preocupación sobre la concesión de permiso de nivel raíz, puede omitir ruta de acceso de entrada y de conexión de prueba manualmente durante la creación. Actividad de copia seguirá funcionando siempre y cuando se le concede la entidad de servicio con los permisos adecuados en los archivos que se va a copiar.

Se admiten las siguientes propiedades:

| Propiedad | DESCRIPCIÓN | Obligatorio |
|:--- |:--- |:--- |
| servicePrincipalId | Especifique el id. de cliente de la aplicación. | Sí |
| servicePrincipalKey | Especifique la clave de la aplicación. Marque este campo como `SecureString`para almacenarlo de forma segura en Data Factory o [para hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). | Sí |
| tenant | Especifique la información del inquilino (nombre de dominio o identificador de inquilino) en el que reside la aplicación. Para recuperarlo, mantenga el puntero del mouse en la esquina superior derecha de Azure Portal. | Sí |

**Ejemplo:**

```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": {
                "type": "SecureString",
                "value": "<service principal key>"
            },
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "subscriptionId": "<subscription of ADLS>",
            "resourceGroupName": "<resource group of ADLS>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="managed-identity"></a> Uso de identidades administradas para la autenticación de recursos de Azure

Una factoría de datos se puede asociar con una [identidad administrada para recursos de Azure](data-factory-service-identity.md), que representa esa factoría de datos concreta. Puede usar directamente esta identidad administrada para la autenticación de Data Lake Store, similar a usar a su propia entidad de servicio. Permite que esta fábrica designada acceda a datos los copie a o desde Data Lake Store.

Para usar identidades administradas para la autenticación de recursos de Azure:

1. [Recuperar la información de identidad de data factory administra](data-factory-service-identity.md#retrieve-managed-identity) copiando el valor el "servicio de identidad del identificador de aplicación" generado junto con la factoría.
2. Conceder el acceso de identidad administrada a Data Lake Store, la misma manera que para la entidad de servicio, siga estas notas.

>[!IMPORTANT]
> Asegúrese de que conceder el permiso adecuado de data factory administra identidad en Data Lake Store:
>- **Como origen**: En **Explorador de datos** > **Acceso**, conceda al menos permiso de **lectura y ejecución** permiso para enumerar y copiar los archivos de las carpetas y subcarpetas. O bien, puede conceder permiso de **lectura** para copiar un único archivo. Puede elegir agregarlo a **Esta carpeta y todos los elementos secundarios** si desea que sea recursivo y agregarlo como **un permiso de acceso y una entrada de permiso predeterminada**. No hay ningún requisito en el control de acceso de nivel de cuenta (IAM).
>- **Como receptor**: En **Explorador de datos** > **Acceso**, conceda al menos permiso de **escritura y ejecución** para crear elementos secundarios en la carpeta. Puede elegir agregarlo a **Esta carpeta y todos los elementos secundarios** si desea que sea recursivo y agregarlo como **un permiso de acceso y una entrada de permiso predeterminada**. Si usa Azure Integration Runtime para realizar la copia (tanto el origen como el receptor están en la nube), en IAM, conceda al menos el rol de **lector** para que Data Factory pueda detectar la región de Data Lake Store. Si desea no usar este rol de IAM, [cree explícitamente una instancia de Azure Integration Runtime](create-azure-integration-runtime.md#create-azure-ir) con la ubicación de Data Lake Store. Asócielo en el servicio vinculado de Data Lake Storage como se muestra en el ejemplo siguiente.

>[!NOTE]
>En la lista de carpetas a partir de la raíz, debe establecer el permiso de la identidad administrada que se concede al **nivel raíz con el permiso "Execute"**. Esto es válido cuando se usa la:
>- **Herramienta Copy Data** para crear la canalización de la copia.
>- **Interfaz de usuario de Data Factory** para probar la conexión y el desplazamiento por las carpetas durante la creación.
>Si dispone de preocupación sobre la concesión de permiso de nivel raíz, puede omitir ruta de acceso de entrada y de conexión de prueba manualmente durante la creación. Actividad de copia seguirá funcionando siempre y cuando se concede la identidad administrada con los permisos adecuados en los archivos que se va a copiar.

En Azure Data Factory no es necesario especificar ninguna propiedad, más allá de la información general de Data Lake Store, en el servicio vinculado.

**Ejemplo:**

```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "subscriptionId": "<subscription of ADLS>",
            "resourceGroupName": "<resource group of ADLS>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md). 

- Para **Parquet y formato de texto delimitado**, consulte [conjunto de datos con formato Parquet y texto delimitado](#parquet-and-delimited-text-format-dataset) sección.
- Para otros formatos como **formato ORC/Avro/JSON/binario**, consulte [otro conjunto de datos de formato](#other-format-dataset) sección.

### <a name="parquet-and-delimited-text-format-dataset"></a>Conjunto de datos con formato parquet y texto delimitado

Para copiar datos hacia y desde Gen1 de ADLS en **Parquet o formato de texto delimitado**, consulte [formato Parquet](format-parquet.md) y [formato de texto delimitado](format-delimited-text.md) artículo en el conjunto de datos de formato y configuraciones admitidas. Las siguientes propiedades son compatibles con ADLS Gen1 en `location` configuración en el conjunto de datos basado en el formato:

| Propiedad   | DESCRIPCIÓN                                                  | Obligatorio |
| ---------- | ------------------------------------------------------------ | -------- |
| type       | La propiedad type en `location` en el conjunto de datos debe establecerse en **AzureDataLakeStoreLocation**. | Sí      |
| folderPath | La ruta de acceso a la carpeta. Si desea usar el carácter comodín a la carpeta de filtro, omita esta configuración y especificar en la configuración del origen de actividad. | Sin        |
| fileName   | El nombre de archivo en folderPath determinado. Si desea usar el carácter comodín para filtrar los archivos, omita esta configuración y especificar en la configuración del origen de actividad. | Sin        |

> [!NOTE]
>
> **AzureDataLakeStoreFile** todavía se admite el tipo de conjunto de datos con formato Parquet/texto que se mencionan en la siguiente sección como-es para la actividad de copia/búsqueda/GetMetadata para compatibilidad con versiones anteriores, pero no funciona con la asignación de flujo de datos. Se sugieren para usar este nuevo modelo a partir de ahora, y ha cambiado la interfaz de usuario de creación de ADF para generar estos nuevos tipos.

**Ejemplo:**

```json
{
    "name": "DelimitedTextDataset",
    "properties": {
        "type": "DelimitedText",
        "linkedServiceName": {
            "referenceName": "<ADLS Gen1 linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, auto retrieved during authoring > ],
        "typeProperties": {
            "location": {
                "type": "AzureDataLakeStoreLocation",
                "folderPath": "root/folder/subfolder"
            },
            "columnDelimiter": ",",
            "quoteChar": "\"",
            "firstRowAsHeader": true,
            "compressionCodec": "gzip"
        }
    }
}
```

### <a name="other-format-dataset"></a>Otro conjunto de datos de formato

Para copiar datos hacia y desde Gen1 de ADLS en **formato ORC/Avro/JSON/binario**, se admiten las siguientes propiedades:

| Propiedad | DESCRIPCIÓN | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del conjunto de datos debe establecerse en: **AzureDataLakeStoreFile** |Sí |
| folderPath | Ruta de acceso a la carpeta de Data Lake Store. Si no se especifica, apunta a la raíz. <br/><br/>Se admite el filtro de comodín, los caracteres comodín permitidos son: `*` (coincide con cero o más caracteres) y `?` (coincide con cero o carácter individual); use `^` para el escape si el nombre real de la carpeta tiene un carácter comodín o este carácter de escape dentro. <br/><br/>Ejemplos: rootfolder/subfolder/ver más en [Ejemplos de filtros de carpetas y archivos](#folder-and-file-filter-examples). |Sin  |
| fileName | **Filtro de nombre o de comodín** para los archivos de la ruta "folderPath" especificada. Si no especifica ningún valor para esta propiedad, el conjunto de datos apunta a todos los archivos de la carpeta. <br/><br/>Para filtrar, los caracteres comodín permitidos son: `*` (equivale a cero o a varios caracteres) y `?` (equivale a cero o a un único carácter).<br/>- Ejemplo 1: `"fileName": "*.csv"`<br/>- Ejemplo 2: `"fileName": "???20180427.txt"`<br/>Use `^` como escape si el nombre de archivo real contiene un comodín o este carácter de escape.<br/><br/>Cuando fileName no se especifica para un conjunto de datos de salida y **preserveHierarchy** no se determina en el receptor de la actividad, la actividad de copia generará automáticamente el nombre de archivo con el siguiente patrón: "*Datos. [identificador GUID de ejecución de actividad]. [GUID si FlattenHierarchy]. [formato si configura]. [Si configura la compresión]* ". Un ejemplo es "Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt.gz". Si realiza una copia desde el origen tabular utilizando el nombre de la tabla en lugar de la consulta, el patrón del nombre es "*[nombre de tabla].[formato].[compresión, si está configurada]*". Por ejemplo, "MyTable.csv". |Sin  |
| modifiedDatetimeStart | Filtro de archivos basado en el atributo: Última modificación. Los archivos se seleccionarán si la hora de su última modificación está dentro del intervalo de tiempo entre `modifiedDatetimeStart` y `modifiedDatetimeEnd`. La hora se aplica a la zona horaria UTC en el formato "2018-12-01T05:00:00Z". <br/><br/> Tenga en cuenta que el rendimiento general del movimiento de datos se verán afectado por si habilita a esta configuración cuando desee del filtro de archivos de grandes cantidades de archivos. <br/><br/> Las propiedades pueden ser NULL que significa que no se aplicará ningún filtro de atributo de archivo para el conjunto de datos.  Cuando `modifiedDatetimeStart` tiene el valor de fecha y hora, pero `modifiedDatetimeEnd` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea mayor o igual que el valor de fecha y hora.  Cuando `modifiedDatetimeEnd` tiene el valor de fecha y hora, pero `modifiedDatetimeStart` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea inferior al valor de fecha y hora.| Sin  |
| modifiedDatetimeEnd | Filtro de archivos basado en el atributo: Última modificación. Los archivos se seleccionarán si la hora de su última modificación está dentro del intervalo de tiempo entre `modifiedDatetimeStart` y `modifiedDatetimeEnd`. La hora se aplica a la zona horaria UTC en el formato "2018-12-01T05:00:00Z". <br/><br/> Tenga en cuenta que el rendimiento general del movimiento de datos se verán afectado por si habilita a esta configuración cuando desee del filtro de archivos de grandes cantidades de archivos. <br/><br/> Las propiedades pueden ser NULL que significa que no se aplicará ningún filtro de atributo de archivo para el conjunto de datos.  Cuando `modifiedDatetimeStart` tiene el valor de fecha y hora, pero `modifiedDatetimeEnd` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea mayor o igual que el valor de fecha y hora.  Cuando `modifiedDatetimeEnd` tiene el valor de fecha y hora, pero `modifiedDatetimeStart` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea inferior al valor de fecha y hora.| Sin  |
| format | Si desea **copiar los archivos tal cual** entre los almacenes basados en archivos (copia binaria), omita la sección de formato en las definiciones de los conjuntos de datos de entrada y salida.<br/><br/>Si desea analizar o generar archivos con un formato concreto, se admiten los siguientes tipos de formato de archivo: **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat**, **ParquetFormat**. Establezca la propiedad **type** de formato en uno de los siguientes valores. Para más información, consulte las secciones [Formato de texto](supported-file-formats-and-compression-codecs.md#text-format), [Formato Json](supported-file-formats-and-compression-codecs.md#json-format), [Formato Avro](supported-file-formats-and-compression-codecs.md#avro-format), [Formato Orc](supported-file-formats-and-compression-codecs.md#orc-format) y [Formato Parquet](supported-file-formats-and-compression-codecs.md#parquet-format). |No (solo para el escenario de copia binaria) |
| compression | Especifique el tipo y el nivel de compresión de los datos. Para más información, consulte el artículo sobre [códecs de compresión y formatos de archivo compatibles](supported-file-formats-and-compression-codecs.md#compression-support).<br/>Estos son los tipos que se admiten: **GZip**, **Deflate**, **BZip2** y **ZipDeflate**.<br/>Estos son los niveles que se admiten: **Optimal** y **Fastest**. |Sin  |


>[!TIP]
>Para copiar todos los archivos en una carpeta, especifique solo **folderPath**.<br>Para copiar un único archivo con un nombre concreto, especifique **folderPath** con la parte de la carpeta y **fileName** con el nombre de archivo.<br>Para copiar un subconjunto de archivos en una carpeta, especifique **folderPath** con el elemento de carpeta y **fileName** con el filtro de comodín. 

**Ejemplo:**

```json
{
    "name": "ADLSDataset",
    "properties": {
        "type": "AzureDataLakeStoreFile",
        "linkedServiceName":{
            "referenceName": "<ADLS linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "folderPath": "datalake/myfolder/",
            "fileName": "*",
            "modifiedDatetimeStart": "2018-12-01T05:00:00Z",
            "modifiedDatetimeEnd": "2018-12-01T06:00:00Z",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ",",
                "rowDelimiter": "\n"
            },
            "compression": {
                "type": "GZip",
                "level": "Optimal"
            }
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Para ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte [Canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades que admiten el receptor y el origen de Azure Data Lake Store.

### <a name="azure-data-lake-store-as-source"></a>Azure Data Lake Store como origen

- Para copiar desde **Parquet y formato de texto delimitado**, consulte [Parquet y origen del formato de texto delimitado](#parquet-and-delimited-text-format-source) sección.
- Para copiar desde otros formatos como **formato ORC/Avro/JSON/binario**, consulte [otro origen de formato](#other-format-source) sección.

#### <a name="parquet-and-delimited-text-format-source"></a>Parquet y origen del formato de texto delimitado

Para copiar datos de ADLS Gen1 en **Parquet o formato de texto delimitado**, consulte [formato Parquet](format-parquet.md) y [formato de texto delimitado](format-delimited-text.md) artículo sobre el origen de la actividad de copia basada en el formato y configuraciones admitidas. Las siguientes propiedades son compatibles con ADLS Gen1 en `storeSettings` configuración en el origen de copia basada en el formato:

| Propiedad                 | DESCRIPCIÓN                                                  | Obligatorio                                      |
| ------------------------ | ------------------------------------------------------------ | --------------------------------------------- |
| type                     | La propiedad type en `storeSettings` debe establecerse en **AzureDataLakeStoreReadSetting**. | Sí                                           |
| recursive                | Indica si los datos se leen de forma recursiva de las subcarpetas o solo de la carpeta especificada. Tenga en cuenta que cuando recursive se establece en true y el receptor es un almacén basado en archivos, no se crea una carpeta o una subcarpeta vacía en el receptor. Los valores permitidos son: **True** (valor predeterminado) y **False**. | Sin                                             |
| wildcardFolderPath       | La ruta de acceso de carpeta con caracteres comodín para filtrar las carpetas de origen. <br>Los caracteres comodín permitidos son: `*` (coincide con cero o más caracteres) y `?` (coincide con cero o carácter individual); use `^` para el escape si el nombre real de la carpeta tiene un carácter comodín o este carácter de escape dentro. <br>Ver más ejemplos en [Ejemplos de filtros de carpetas y archivos](#folder-and-file-filter-examples). | Sin                                             |
| wildcardFileName         | El nombre de archivo con caracteres comodín en el folderPath/wildcardFolderPath determinado para filtrar los archivos de origen. <br>Los caracteres comodín permitidos son: `*` (coincide con cero o más caracteres) y `?` (coincide con cero o carácter individual); use `^` para el escape si el nombre real de la carpeta tiene un carácter comodín o este carácter de escape dentro.  Ver más ejemplos en [Ejemplos de filtros de carpetas y archivos](#folder-and-file-filter-examples). | Sí si `fileName` no se especifica en el conjunto de datos |
| modifiedDatetimeStart    | Filtro de archivos basado en el atributo: Última modificación. Los archivos se seleccionarán si la hora de su última modificación está dentro del intervalo de tiempo entre `modifiedDatetimeStart` y `modifiedDatetimeEnd`. La hora se aplica a la zona horaria UTC en el formato "2018-12-01T05:00:00Z". <br> Las propiedades pueden ser NULL, lo que significa que no se aplicará ningún filtro de atributo de archivo al conjunto de datos.  Cuando `modifiedDatetimeStart` tiene el valor de fecha y hora, pero `modifiedDatetimeEnd` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea mayor o igual que el valor de fecha y hora.  Cuando `modifiedDatetimeEnd` tiene el valor de fecha y hora, pero `modifiedDatetimeStart` es NULL, significa que se seleccionarán los archivos cuyo último atributo modificado sea inferior al valor de fecha y hora. | Sin                                             |
| modifiedDatetimeEnd      | Igual que el anterior.                                               | Sin                                             |
| maxConcurrentConnections | El número de conexiones para conectarse al almacén de almacenamiento al mismo tiempo. Especifique solo cuando desee limitar las conexiones simultáneas al almacén de datos. | No                                            |

> [!NOTE]
> Para Parquet/texto delimitado con **AzureDataLakeStoreSource** todavía se admite como origen de la actividad de copia tipo mencionado en la siguiente sección: sirve para la compatibilidad con versiones anteriores. Se sugieren para usar este nuevo modelo a partir de ahora, y ha cambiado la interfaz de usuario de creación de ADF para generar estos nuevos tipos.

**Ejemplo:**

```json
"activities":[
    {
        "name": "CopyFromADLSGen1",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Delimited text input dataset name>",
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
                "type": "DelimitedTextSource",
                "formatSettings":{
                    "type": "DelimitedTextReadSetting",
                    "skipLineCount": 10
                },
                "storeSettings":{
                    "type": "AzureDataLakeStoreReadSetting",
                    "recursive": true,
                    "wildcardFolderPath": "myfolder*A",
                    "wildcardFileName": "*.csv"
                }
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

#### <a name="other-format-source"></a>Otro origen de formato

Para copiar datos de ADLS Gen1 en **formato ORC/Avro/JSON/binario**, se admiten las siguientes propiedades en la actividad de copia **origen** sección:

| Propiedad | DESCRIPCIÓN | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad `type` del origen de la actividad de copia debe establecerse en: **AzureDataLakeStoreSource**. |Sí |
| recursive | Indica si los datos se leen de forma recursiva de las subcarpetas o solo de la carpeta especificada. Tenga en cuenta que cuando `recursive` se establece en true y el receptor es un almacén basado en archivos, no se copia ni crea ninguna carpeta o subcarpeta vacías en el receptor. Los valores permitidos son: **true** (valor predeterminado) y **false**. | Sin  |
| maxConcurrentConnections | El número de conexiones para conectarse al almacén de datos al mismo tiempo. Especifique solo cuando desee limitar las conexiones simultáneas al almacén de datos. | Sin  |

**Ejemplo:**

```json
"activities":[
    {
        "name": "CopyFromADLSGen1",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<ADLS Gen1 input dataset name>",
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
                "type": "AzureDataLakeStoreSource",
                "recursive": true
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="azure-data-lake-store-as-sink"></a>Azure Data Lake Store como receptor

- Para copiar en **Parquet y formato de texto delimitado**, consulte [Parquet y receptores de formato de texto delimitado](#parquet-and-delimited-text-format-sink) sección.
- Para copiar en otros formatos, como **formato ORC/Avro/JSON/binario**, consulte [otros receptores de formato](#other-format-sink) sección.

#### <a name="parquet-and-delimited-text-format-sink"></a>Parquet y receptores de formato de texto delimitado

Para copiar datos en ADLS Gen1 en **Parquet o formato de texto delimitado**, consulte [formato Parquet](format-parquet.md) y [formato de texto delimitado](format-delimited-text.md) artículo en el receptor de la actividad de copia basada en el formato y configuraciones admitidas. Las siguientes propiedades son compatibles con ADLS Gen1 en `storeSettings` configuración del receptor de copia basada en el formato:

| Propiedad                 | DESCRIPCIÓN                                                  | Obligatorio |
| ------------------------ | ------------------------------------------------------------ | -------- |
| type                     | La propiedad type en `storeSettings` debe establecerse en **AzureDataLakeStoreWriteSetting**. | Sí      |
| copyBehavior             | Define el comportamiento de copia cuando el origen son archivos de un almacén de datos basados en archivos.<br/><br/>Los valores permitidos son:<br/><b>- PreserveHierarchy (valor predeterminado)</b>: conserva la jerarquía de archivos en la carpeta de destino. La ruta de acceso relativa del archivo de origen que apunta a la carpeta de origen es idéntica a la ruta de acceso relativa del archivo de destino que apunta a la carpeta de destino.<br/><b>- FlattenHierarchy</b>: todos los archivos de la carpeta de origen están en el primer nivel de la carpeta de destino. Los archivos de destino tienen nombres generados automáticamente. <br/><b>- MergeFiles</b>: combina todos los archivos de la carpeta de origen en un archivo. Si se especifica el nombre del archivo, el nombre de archivo combinado es el nombre especificado. De lo contrario, es un nombre de archivo generado automáticamente. | Sin        |
| maxConcurrentConnections | El número de conexiones para conectarse al almacén de datos al mismo tiempo. Especifique solo cuando desee limitar las conexiones simultáneas al almacén de datos. | No       |

> [!NOTE]
> Para Parquet/texto delimitado con **AzureDataLakeStoreSink** todavía se admite como receptor de la actividad de copia tipo mencionado en la siguiente sección: sirve para la compatibilidad con versiones anteriores. Se sugieren para usar este nuevo modelo a partir de ahora, y ha cambiado la interfaz de usuario de creación de ADF para generar estos nuevos tipos.

**Ejemplo:**

```json
"activities":[
    {
        "name": "CopyToADLSGen1",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Parquet output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "ParquetSink",
                "storeSettings":{
                    "type": "AzureDataLakeStoreWriteSetting",
                    "copyBehavior": "PreserveHierarchy"
                }
            }
        }
    }
]
```

#### <a name="other-format-sink"></a>Otros receptores de formato

Para copiar datos en ADLS Gen1 en **formato ORC/Avro/JSON/binario**, se admiten las siguientes propiedades en el **receptor** sección:

| Propiedad | DESCRIPCIÓN | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad `type` del receptor de la actividad de copia debe establecerse en: **AzureDataLakeStoreSink** |Sí |
| copyBehavior | Define el comportamiento de copia cuando el origen son archivos de un almacén de datos basados en archivos.<br/><br/>Los valores permitidos son:<br/><b>- PreserveHierarchy (valor predeterminado)</b>: conserva la jerarquía de archivos en la carpeta de destino. La ruta de acceso relativa del archivo de origen a la carpeta de origen es idéntica que la ruta de acceso relativa del archivo de destino a la carpeta de destino.<br/><b>- FlattenHierarchy</b>: todos los archivos de la carpeta de origen están en el primer nivel de la carpeta de destino. Los archivos de destino tienen nombres generados automáticamente. <br/><b>- MergeFiles</b>: combina todos los archivos de la carpeta de origen en un archivo. Si se especifica el nombre del archivo, el nombre de archivo combinado es el nombre especificado. En caso contrario, el nombre de archivo se genera automáticamente. | Sin  |
| maxConcurrentConnections | El número de conexiones para conectarse al almacén de datos al mismo tiempo. Especifique solo cuando desee limitar las conexiones simultáneas al almacén de datos. | Sin  |

**Ejemplo:**

```json
"activities":[
    {
        "name": "CopyToADLSGen1",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<ADLS Gen1 output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "AzureDataLakeStoreSink",
                "copyBehavior": "PreserveHierarchy"
            }
        }
    }
]
```

### <a name="folder-and-file-filter-examples"></a>Ejemplos de filtros de carpetas y archivos

Esta sección describe el comportamiento resultante de la ruta de acceso de la carpeta y el nombre de archivo con los filtros de carácter comodín.

| folderPath | fileName | recursive | Resultado de estructura de carpeta de origen y filtro (se recuperan los archivos en **negrita**)|
|:--- |:--- |:--- |:--- |
| `Folder*` | (vacío, usar el valor predeterminado) | false | FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5.csv<br/>AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| `Folder*` | (vacío, usar el valor predeterminado) | true | FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File4.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File5.csv**<br/>AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| `Folder*` | `*.csv` | false | FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5.csv<br/>AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| `Folder*` | `*.csv` | true | FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File5.csv**<br/>AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |

### <a name="examples-of-behavior-of-the-copy-operation"></a>Ejemplos de comportamiento de la operación de copia

En esta sección se describe el comportamiento resultante de la operación de copia para diferentes combinaciones de los valores `recursive` y `copyBehavior`.

| recursive | copyBehavior | Estructura de carpetas de origen | Destino resultante |
|:--- |:--- |:--- |:--- |
| true |preserveHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  &nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  &nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  &nbsp;File5 | El destino Folder1 se crea con la misma estructura que el origen:<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  &nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  &nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  &nbsp;File5. |
| true |flattenHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  &nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  &nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  &nbsp;File5 | La carpeta de destino Folder1 se crea con la estructura siguiente: <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Nombre de archivo generado automáticamente para File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Nombre de archivo generado automáticamente para File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Nombre de archivo generado automáticamente para File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;Nombre de archivo generado automáticamente para File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;Nombre de archivo generado automáticamente para File5 |
| true |mergeFiles | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  &nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  &nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  &nbsp;File5 | La carpeta de destino Folder1 se crea con la estructura siguiente: <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;El contenido de File1 + File2 + File3 + File4 + File 5 se combina en un solo archivo, cuyo nombre se genera automáticamente. |
| false |preserveHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  &nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  &nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  &nbsp;File5 | La carpeta de destino Folder1 se crea con la estructura siguiente:<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/><br/>No se selecciona la subcarpeta Subfolder1, que contiene los archivos File3, File4 y File5. |
| false |flattenHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  &nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  &nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  &nbsp;File5 | La carpeta de destino Folder1 se crea con la estructura siguiente:<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Nombre de archivo generado automáticamente para File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Nombre de archivo generado automáticamente para File2<br/><br/>No se selecciona la subcarpeta Subfolder1, que contiene los archivos File3, File4 y File5. |
| false |mergeFiles | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  &nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  &nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  &nbsp;File5 | La carpeta de destino Folder1 se crea con la estructura siguiente:<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;El contenido de File1 + File2 se combina en un archivo con un nombre de archivo generado automáticamente. Nombre de archivo generado automáticamente para File1<br/><br/>No se selecciona la subcarpeta Subfolder1, que contiene los archivos File3, File4 y File5. |

## <a name="preserve-acls-to-data-lake-storage-gen2"></a>Conservar las ACL en Data Lake Storage Gen2

Si desea replicar las ACL junto con los archivos de datos al actualizar desde Data Lake Storage Gen1 a Gen2, consulte [conservar las ACL de Data Lake Storage Gen1](connector-azure-data-lake-storage.md#preserve-acls-from-data-lake-storage-gen1).

## <a name="mapping-data-flow-properties"></a>Propiedades de flujo de datos de asignación

Obtenga información detallada de [transformación del origen](data-flow-source.md) y [receptor transformación](data-flow-sink.md) en asignación de flujo de datos.

## <a name="next-steps"></a>Pasos siguientes

Para ver la lista de almacenes de datos que la actividad de copia de Azure Data Factory admite como orígenes y receptores, consulte los [almacenes de datos compatibles](copy-activity-overview.md##supported-data-stores-and-formats).
