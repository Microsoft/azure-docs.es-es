---
title: Copia y transformación de datos en Azure Blob Storage
description: Aprenda a copiar datos en y desde Blob Storage, y a transformar datos en Blob Storage mediante Data Factory.
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: craigg
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 08/18/2020
ms.openlocfilehash: 489d05f19fe9fafd4c32f988e9b0b90f95c0b5b2
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88718363"
---
# <a name="copy-and-transform-data-in-azure-blob-storage-by-using-azure-data-factory"></a>Copia y transformación de datos en Azure Blob Storage mediante Azure Data Factory

> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que esté usando:"]
> * [Versión 1](v1/data-factory-azure-blob-connector.md)
> * [Versión actual](connector-azure-blob-storage.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se resume el uso de la actividad de copia en Azure Data Factory para copiar datos con Azure Blob Storage como origen y destino. También se describe cómo usar la actividad de Data Flow para transformar datos en Azure Blob Storage. Para información sobre Azure Data Factory, lea el [artículo de introducción](introduction.md).

>[!TIP]
>Para obtener información sobre un escenario de migración de un lago de datos o un almacenamiento de datos, consulte [Uso de Azure Data Factory para migrar datos del lago de datos y el almacenamiento de datos a Azure](data-migration-guidance-overview.md).

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Este conector de Azure Blob Storage es compatible con las actividades siguientes:

- [Actividad de copia](copy-activity-overview.md) con [matriz de origen o receptor compatible](copy-activity-overview.md)
- [Asignación de Data Flow](concepts-data-flow-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)
- [Actividad GetMetadata](control-flow-get-metadata-activity.md)
- [Actividad de eliminación](delete-activity.md)

En el caso de la actividad de copia, este conector de Blob Storage admite:

- Copia de blobs con cuentas de Azure Storage de uso general y almacenamiento de blobs en frío y en caliente como orígenes y destinos. 
- Copia de blobs mediante el uso de una clave de cuenta, una firma de acceso compartido de servicio (SAS), una entidad de servicio o identidades administradas paras las autenticaciones de recurso de Azure.
- Copia de blobs desde blobs en bloques, blobs con anexos o blobs en páginas y copia de datos solo a blobs en bloques.
- Copia de blobs tal cual, o análisis o generación de los mismos con [códecs de compresión y formatos de archivo compatibles](supported-file-formats-and-compression-codecs.md).
- [Conservación de los metadatos de archivo durante la copia](#preserving-metadata-during-copy).

>[!IMPORTANT]
>Si habilita la opción **Permitir que los servicios de Microsoft de confianza accedan a esta cuenta de almacenamiento** en la configuración de firewall de Azure Storage y quiere usar Azure Integration Runtime para conectarse a Blob Storage, debe usar la [autenticación de identidad administrada](#managed-identity).

## <a name="get-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

En las secciones siguientes se proporcionan detalles acerca de las propiedades que se usan para definir entidades de Data Factory específicas de Blob Storage.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

El conector de Blob Storage admite los siguientes tipos de autenticación. Consulte las secciones correspondientes para más información.

- [Autenticación de clave de cuenta](#account-key-authentication)
- [Autenticación con firma de acceso compartido](#shared-access-signature-authentication)
- [Autenticación de entidad de servicio](#service-principal-authentication)
- [Identidades administradas para la autenticación del recurso de Azure](#managed-identity)

>[!NOTE]
>Cuando use PolyBase para cargar datos en SQL Data Warehouse, si la instancia de origen o almacenamiento provisional de Blob Storage está configurada con un punto de conexión de Azure Virtual Network, deberá usar la autenticación de identidad administrada como requiere PolyBase. También debe usar el entorno de ejecución de integración autohospedado, con la versión 3.18 o posterior. Consulte la sección sobre [autenticación de identidad administrada](#managed-identity) para conocer más requisitos previos de configuración.

>[!NOTE]
>Las actividades de Azure HDInsights y Azure Machine Learning solo admiten la autenticación que usa las claves de cuenta de Azure Blob Storage.

### <a name="account-key-authentication"></a>Autenticación de clave de cuenta

Data Factory admite las siguientes propiedades para la autenticación de clave de cuenta de almacenamiento:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type** debe establecerse en **AzureBlobStorage** (recomendable) o **AzureStorage** (consulte las siguientes notas). |Sí |
| connectionString | Especifique la información necesaria para conectarse a Storage para la propiedad **connectionString**. <br/> También puede colocar la clave de cuenta en Azure Key Vault y extraer la configuración `accountKey` de la cadena de conexión. Para obtener más información, consulte los siguientes ejemplos y el artículo [Almacenamiento de credenciales en Azure Key Vault](store-credentials-in-key-vault.md). |Sí |
| connectVia | El [entorno de ejecución de integración](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Se puede usar Azure Integration Runtime o un entorno de ejecución de integración autohospedado (si el almacén de datos está en una red privada). Si no se especifica esta propiedad, el servicio usa el valor predeterminado de Azure Integration Runtime. |No |

>[!NOTE]
>No se admite un punto de conexión de Blob service secundario al usar la autenticación de clave de cuenta. Puede usar otros tipos de autenticación.

>[!NOTE]
>Si está usando el tipo de servicio vinculado "AzureStorage", todavía se admite tal cual. Pero se recomienda usar el nuevo tipo de servicio vinculado "AzureBlobStorage" en el futuro.

**Ejemplo**:

```json
{
    "name": "AzureBlobStorageLinkedService",
    "properties": {
        "type": "AzureBlobStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Ejemplo: Almacenamiento de la clave de cuenta en Azure Key Vault**

```json
{
    "name": "AzureBlobStorageLinkedService",
    "properties": {
        "type": "AzureBlobStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;",
            "accountKey": { 
                "type": "AzureKeyVaultSecret", 
                "store": { 
                    "referenceName": "<Azure Key Vault linked service name>", 
                    "type": "LinkedServiceReference" 
                }, 
                "secretName": "<secretName>" 
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }            
    }
}
```

### <a name="shared-access-signature-authentication"></a>Autenticación con firma de acceso compartido

Una firma de acceso compartido ofrece acceso delegado a recursos en la cuenta de almacenamiento. Puede utilizar una firma de acceso compartido para conceder a un cliente permisos limitados a los objetos de su cuenta de almacenamiento durante un periodo de tiempo especificado. 

No tiene que compartir las claves de acceso de su cuenta. La firma de acceso compartido es un URI que incluye en sus parámetros de consulta toda la información necesaria para el acceso autenticado a un recurso de almacenamiento. Para obtener acceso a los recursos de almacenamiento con la firma de acceso compartido, el cliente solo tiene que pasar la firma de acceso compartido al método o constructor adecuados. 

Para obtener más información sobre las firmas de acceso compartido, consulte [Uso de firmas de acceso compartido (SAS): Comprender el modelo de firma de acceso compartido](../storage/common/storage-dotnet-shared-access-signature-part-1.md).

> [!NOTE]
>- Data Factory ahora admite *firmas de acceso compartido de servicio* y *firmas de acceso compartido de cuenta*. Para obtener más información sobre las firmas de acceso compartido, consulte [Otorgar acceso limitado a recursos de Azure Storage con firmas de acceso compartido](../storage/common/storage-sas-overview.md).
>- En la configuración posterior del conjunto de datos, la ruta de la carpeta es la ruta absoluta a partir del nivel del contenedor. Deberá configurar una alineada con la ruta de acceso del URI de SAS.

Data Factory admite las siguientes propiedades para usar la autenticación con firma de acceso compartido:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type** debe establecerse en **AzureBlobStorage** (recomendable) o **AzureStorage** (consulte la siguiente nota). |Sí |
| sasUri | Especifique el URI de firma de acceso compartido a los recursos de Storage como blob o contenedor. <br/>Marque este campo como **SecureString** para almacenarlo de forma segura en Data Factory. También puede colocar el token de SAS en Azure Key Vault para usar la rotación automática y quitar la parte del token. Para obtener más información, consulte los siguientes ejemplos y el [Almacenamiento de credenciales en Azure Key Vault](store-credentials-in-key-vault.md). |Sí |
| connectVia | El [entorno de ejecución de integración](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Se puede usar Azure Integration Runtime o un entorno de ejecución de integración autohospedado (si el almacén de datos está en una red privada). Si no se especifica esta propiedad, el servicio usa el valor predeterminado de Azure Integration Runtime. |No |

>[!NOTE]
>Si usa el tipo de servicio vinculado "AzureStorage", todavía se admite tal cual. Pero se recomienda usar el nuevo tipo de servicio vinculado "AzureBlobStorage" en el futuro.

**Ejemplo**:

```json
{
    "name": "AzureBlobStorageLinkedService",
    "properties": {
        "type": "AzureBlobStorage",
        "typeProperties": {
            "sasUri": {
                "type": "SecureString",
                "value": "<SAS URI of the Azure Storage resource e.g. https://<accountname>.blob.core.windows.net/?sv=<storage version>&st=<start time>&se=<expire time>&sr=<resource>&sp=<permissions>&sip=<ip range>&spr=<protocol>&sig=<signature>>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Ejemplo: Almacenamiento de la clave de cuenta en Azure Key Vault**

```json
{
    "name": "AzureBlobStorageLinkedService",
    "properties": {
        "type": "AzureBlobStorage",
        "typeProperties": {
            "sasUri": {
                "type": "SecureString",
                "value": "<SAS URI of the Azure Storage resource without token e.g. https://<accountname>.blob.core.windows.net/>"
            },
            "sasToken": { 
                "type": "AzureKeyVaultSecret", 
                "store": { 
                    "referenceName": "<Azure Key Vault linked service name>", 
                    "type": "LinkedServiceReference" 
                }, 
                "secretName": "<secretName with value of SAS token e.g. ?sv=<storage version>&st=<start time>&se=<expire time>&sr=<resource>&sp=<permissions>&sip=<ip range>&spr=<protocol>&sig=<signature>>" 
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

Cuando cree un URI de firma de acceso compartido, tenga en cuenta lo siguiente:

- Establezca los permisos de lectura y escritura adecuados en los objetos, en función de cómo se utilizará el servicio vinculado (lectura, escritura, lectura y escritura) en la factoría de datos.
- Establezca la **hora de expiración** adecuadamente. Asegúrese de que el acceso a los objetos de Storage no expirará durante el período activo de la canalización.
- El URI debe crearse en el nivel correcto del contenedor o blob, en función de lo que se necesite. Un URI de firma de acceso compartido a un blob permite a Data Factory tener acceso a ese blob determinado. Un URI de firma de acceso compartido a un contenedor de Blob Storage permite a Data Factory recorrer en iteración los blobs de ese contenedor. Para proporcionar acceso a más o menos objetos más adelante, o actualizar el URI de firma de acceso compartida, no olvide actualizar el servicio vinculado con el nuevo URI.

### <a name="service-principal-authentication"></a>Autenticación de entidad de servicio

Para obtener información general sobre la autenticación con entidad de servicio de Azure Storage, consulte [Autenticación del acceso a Azure Storage con Azure Active Directory](../storage/common/storage-auth-aad.md).

Antes de usar la autenticación de entidad de servicio, siga estos pasos:

1. Registre una entidad de aplicación en Azure Active Directory (Azure AD) como se indica en [Registro de la aplicación con un inquilino de Azure AD](../storage/common/storage-auth-aad-app.md#register-your-application-with-an-azure-ad-tenant). Anote estos valores; los usará para definir el servicio vinculado:

    - Identificador de aplicación
    - Clave de la aplicación
    - Id. de inquilino

2. Conceda a la entidad de servicio el permiso adecuado en Azure Blob Storage. Para más información sobre los roles, consulte [Administración de los derechos de acceso a los datos de Azure Storage con RBAC](../storage/common/storage-auth-aad-rbac.md).

    - **Como origen**, en **Control de acceso (IAM)** , conceda al menos el rol **Lector de datos de Storage Blob**.
    - **Como receptor**, en el **control de acceso (IAM)** , conceda al menos el rol **Colaborador de datos de Storage Blob**.

Estas propiedades son compatibles con un servicio vinculado de Azure Blob Storage:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type** debe establecerse en **AzureBlobStorage**. |Sí |
| serviceEndpoint | Especifique el punto de conexión de servicio de Azure Blob Storage con el patrón `https://<accountName>.blob.core.windows.net/`. |Sí |
| servicePrincipalId | Especifique el id. de cliente de la aplicación. | Sí |
| servicePrincipalKey | Especifique la clave de la aplicación. Marque este campo como [SecureString](store-credentials-in-key-vault.md) para almacenarlo de forma segura en Data Factory, o bien **para hacer referencia a un secreto almacenado en Azure Key Vault**. | Sí |
| tenant | Especifique la información del inquilino (nombre de dominio o identificador de inquilino) en el que reside la aplicación. Para recuperarlo, mantenga el mouse en la esquina superior derecha de Azure Portal. | Sí |
| azureCloudType | Para la autenticación de la entidad del servicio, especifique el tipo de entorno de nube de Azure en el que está registrada la aplicación Azure Active Directory. <br/> Los valores permitidos son **AzurePublic**, **AzureChina**, **AzureUsGovernment** y **AzureGermany**. De forma predeterminada, se usa el entorno de nube de la factoría de datos. | No |
| connectVia | El [entorno de ejecución de integración](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Se puede usar Azure Integration Runtime o un entorno de ejecución de integración autohospedado (si el almacén de datos está en una red privada). Si no se especifica esta propiedad, el servicio usa el valor predeterminado de Azure Integration Runtime. |No |

>[!NOTE]
>Solo el servicio vinculado tipo "AzureBlobStorage" admite la autenticación con la entidad de servicio, mientras que el servicio vinculado tipo "AzureStorage" anterior no la admite.

**Ejemplo**:

```json
{
    "name": "AzureBlobStorageLinkedService",
    "properties": {
        "type": "AzureBlobStorage",
        "typeProperties": {            
            "serviceEndpoint": "https://<accountName>.blob.core.windows.net/",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": {
                "type": "SecureString",
                "value": "<service principal key>"
            },
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>" 
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="managed-identities-for-azure-resource-authentication"></a><a name="managed-identity"></a> Identidades administradas para la autenticación del recurso de Azure

Una factoría de datos se puede asociar con una [identidad administrada para recursos de Azure](data-factory-service-identity.md), que representa esa factoría de datos concreta. Puede usar directamente esta identidad administrada para la autenticación de Blob Storage, que similar a como usa su propia entidad de servicio. Permite que esta factoría designada acceda y copie los datos con Blob Storage como origen y destino.

Para obtener información general sobre la autenticación de Azure Storage, consulte [Autenticación del acceso a Azure Storage con Azure Active Directory](../storage/common/storage-auth-aad.md). Para usar identidades administradas para la autenticación de recursos de Azure, siga estos pasos:

1. [Recupere la información de la identidad administrada de Data Factory](data-factory-service-identity.md#retrieve-managed-identity) mediante la copia del valor de Id. del objeto de identidad administrada que se ha generado junto con la factoría.

2. Conceda el permiso de identidad administrada en Azure Blob Storage. Para más información sobre los roles, consulte [Administración de los derechos de acceso a los datos de Azure Storage con RBAC](../storage/common/storage-auth-aad-rbac.md).

    - **Como origen**, en **Control de acceso (IAM)** , conceda al menos el rol **Lector de datos de Storage Blob**.
    - **Como receptor**, en el **Control de acceso (IAM)** , conceda al menos el rol **Colaborador de datos de blobs de almacenamiento**.

>[!IMPORTANT]
>Si usa PolyBase para cargar datos desde Blob Storage (como origen o como almacenamiento provisional) en SQL Data Warehouse, cuando use la autenticación de identidad administrada para Blob Storage, asegúrese de seguir los pasos 1 y 2 de [esta guía](../azure-sql/database/vnet-service-endpoint-rule-overview.md#impact-of-using-vnet-service-endpoints-with-azure-storage). En estos pasos se registrará el servidor con Azure AD y se asignará el rol Colaborador de datos de Storage Blob en el servidor. Data Factory controla el resto. Si ha configurado la instancia de Blob Storage con un punto de conexión de Azure Virtual Network, para usar PolyBase para cargar datos desde este deberá usar la autenticación de identidad administrada como requiere PolyBase.

Estas propiedades son compatibles con un servicio vinculado de Azure Blob Storage:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type** debe establecerse en **AzureBlobStorage**. |Sí |
| serviceEndpoint | Especifique el punto de conexión de servicio de Azure Blob Storage con el patrón `https://<accountName>.blob.core.windows.net/`. |Sí |
| connectVia | El [entorno de ejecución de integración](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Se puede usar Azure Integration Runtime o un entorno de ejecución de integración autohospedado (si el almacén de datos está en una red privada). Si no se especifica esta propiedad, el servicio usa el valor predeterminado de Azure Integration Runtime. |No |

> [!NOTE]
> Solo el servicio vinculado del tipo "AzureBlobStorage" admite las identidades administradas para la autenticación de recursos de Azure, pero el servicio vinculado del tipo "AzureStorage" anterior no las admite.

**Ejemplo**:

```json
{
    "name": "AzureBlobStorageLinkedService",
    "properties": {
        "type": "AzureBlobStorage",
        "typeProperties": {            
            "serviceEndpoint": "https://<accountName>.blob.core.windows.net/"
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

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

Las propiedades siguientes se admiten para Azure Blob Storage en la configuración `location` del conjunto de datos basado en formato:

| Propiedad   | Descripción                                                  | Obligatorio |
| ---------- | ------------------------------------------------------------ | -------- |
| type       | La propiedad **type** de la ubicación del conjunto de datos se debe establecer en **AzureBlobStorageLocation**. | Sí      |
| contenedor  | Contenedor de blobs.                                          | Sí      |
| folderPath | Ruta de acceso a la carpeta bajo el contenedor especificado. Si quiere usar un carácter comodín para filtrar la carpeta, omita este valor y especifíquelo en la configuración del origen de actividad. | No       |
| fileName   | Nombre de archivo en el contenedor y ruta de carpeta indicados. Si quiere usar el carácter comodín para filtrar los archivos, omita este valor y especifíquelo en la configuración del origen de actividad. | No       |

**Ejemplo**:

```json
{
    "name": "DelimitedTextDataset",
    "properties": {
        "type": "DelimitedText",
        "linkedServiceName": {
            "referenceName": "<Azure Blob Storage linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, auto retrieved during authoring > ],
        "typeProperties": {
            "location": {
                "type": "AzureBlobStorageLocation",
                "container": "containername",
                "folderPath": "folder/subfolder"
            },
            "columnDelimiter": ",",
            "quoteChar": "\"",
            "firstRowAsHeader": true,
            "compressionCodec": "gzip"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades que admite el origen y receptor de Blob Storage.

### <a name="blob-storage-as-a-source-type"></a>Blob Storage como un tipo de origen

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

Las propiedades siguientes se admiten para Azure Blob Storage en la configuración `storeSettings` en un origen de copia basado en formato:

| Propiedad                 | Descripción                                                  | Obligatorio                                      |
| ------------------------ | ------------------------------------------------------------ | --------------------------------------------- |
| type                     | La propiedad **type** de `storeSettings` se debe establecer en **AzureBlobStorageReadSettings**. | Sí                                           |
| ***Buscar los archivos que se van a copiar:*** |  |  |
| OPCIÓN 1: ruta de acceso estática<br> | Copia de la ruta de acceso de carpeta/archivo o de contenedor especificada en el conjunto de datos. Si quiere copiar todos los blobs de un contenedor o una carpeta, especifique también `wildcardFileName` como `*`. |  |
| OPCIÓN 2: prefijo de blob<br>- prefix | Prefijo para el nombre de blob en el contenedor dado configurado en un conjunto de datos para filtrar los blobs de origen. Se seleccionan los blobs cuyo nombre comienza con `container_in_dataset/this_prefix`. Usa el filtro del lado de servicio para Blob Storage, que proporciona un mejor rendimiento que un filtro de caracteres comodín. | No                                                          |
| OPCIÓN 3: carácter comodín<br>- wildcardFolderPath | Ruta de acceso de carpeta con caracteres comodín en el contenedor especificado configurado en un conjunto de datos para filtrar las carpetas de origen. <br>Los caracteres comodín permitidos son: `*` (equivale a cero o a varios caracteres) y `?` (equivale a cero o a un único carácter). Use `^` como escape si el nombre de la carpeta contiene un carácter comodín o este carácter de escape. <br>Ver más ejemplos en [Ejemplos de filtros de carpetas y archivos](#folder-and-file-filter-examples). | No                                            |
| OPCIÓN 3: carácter comodín<br>- wildcardFileName | Nombre de archivo con caracteres comodín en el contenedor y la ruta de carpeta (o ruta de carpeta con carácter comodín) indicada para filtrar los archivos de origen. <br>Los caracteres comodín permitidos son: `*` (equivale a cero o a varios caracteres) y `?` (equivale a cero o a un único carácter). Use `^` como escape si el nombre de la carpeta contiene un carácter comodín o este carácter de escape. Ver más ejemplos en [Ejemplos de filtros de carpetas y archivos](#folder-and-file-filter-examples). | Sí |
| OPCIÓN 4: una lista de archivos<br>- fileListPath | Indica que se copie un conjunto de archivos determinado. Apunte a un archivo de texto que incluya una lista de los archivos que quiere copiar, con un archivo por línea, que sea la ruta de acceso relativa a la ruta de acceso configurada en el conjunto de datos.<br/>Al usar esta opción, no especifique un nombre de archivo en el conjunto de datos. Ver más ejemplos en [Ejemplos de lista de archivos](#file-list-examples). |No |
| ***Configuración adicional:*** |  | |
| recursive | Indica si los datos se leen de forma recursiva de las subcarpetas o solo de la carpeta especificada. Tenga en cuenta que cuando **recursive** se establece en **true** y el receptor es un almacén basado en archivos, no se crea una carpeta o una subcarpeta vacía en el receptor. <br>Los valores permitidos son: **True** (valor predeterminado) y **False**.<br>Esta propiedad no se aplica al configurar `fileListPath`. |No |
| deleteFilesAfterCompletion | Indica si los archivos binarios se eliminarán del almacén de origen después de moverse correctamente al almacén de destino. Cada archivo se elimina individualmente, de modo que cuando se produzca un error en la actividad de copia, algunos archivos ya se habrán copiado al destino y se habrán eliminado del origen, mientras que otros seguirán aún en el almacén de origen. <br/>Esta propiedad solo es válida en el escenario de copia de archivos binarios, en el que los almacenes de origen de datos son Blob, ADLS Gen1, ADLS Gen2, S3, Google Cloud Storage, File, Azure File, SFTP o FTP. El valor predeterminado es false. |No |
| modifiedDatetimeStart    | Los archivos se filtran en función del atributo Last Modified. <br>Los archivos se seleccionarán si la hora de su última modificación está dentro del intervalo de tiempo entre `modifiedDatetimeStart` y `modifiedDatetimeEnd`. La hora se aplica a una zona horaria UTC en el formato "2018-12-01T05:00:00Z". <br> Las propiedades pueden ser **NULL**, lo que significa que no se aplica ningún filtro de atributo de archivo al conjunto de datos.  Cuando `modifiedDatetimeStart` tiene un valor de fecha y hora, pero `modifiedDatetimeEnd` es **NULL**, significa que se seleccionarán los archivos cuyo último atributo modificado sea mayor o igual que el valor de fecha y hora.  Cuando `modifiedDatetimeEnd` tiene un valor de fecha y hora, pero `modifiedDatetimeStart` es **NULL**, significa que se seleccionarán los archivos cuyo último atributo modificado sea menor que el valor de fecha y hora.<br/>Esta propiedad no se aplica al configurar `fileListPath`. | No                                            |
| modifiedDatetimeEnd      | Igual que el anterior.                                               | No                                            |
| maxConcurrentConnections | Número de conexiones simultáneas al almacenamiento. Solo se especifica cuando se quieren limitar las conexiones simultáneas al almacén de datos. | No                                            |

> [!NOTE]
> Para el formato de texto delimitado o Parquet, aún se admite el tipo **BlobSource** del origen de copia de seguridad mencionado en la sección siguiente para la compatibilidad con versiones anteriores. Se recomienda usar el nuevo modelo hasta que la interfaz de usuario de creación de Data Factory haya comenzado a generar nuevos tipos.

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyFromBlob",
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
                    "type": "DelimitedTextReadSettings",
                    "skipLineCount": 10
                },
                "storeSettings":{
                    "type": "AzureBlobStorageReadSettings",
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

### <a name="blob-storage-as-a-sink-type"></a>Blob Storage como un tipo de receptor

[!INCLUDE [data-factory-v2-file-sink-formats](../../includes/data-factory-v2-file-sink-formats.md)] 

Las propiedades siguientes se admiten para Azure Blob Storage en la configuración `storeSettings` en un receptor de copia basado en formato:

| Propiedad                 | Descripción                                                  | Obligatorio |
| ------------------------ | ------------------------------------------------------------ | -------- |
| type                     | La propiedad **type** de `storeSettings` se debe establecer en **AzureBlobStorageWriteSettings**. | Sí      |
| copyBehavior             | Define el comportamiento de copia cuando el origen son archivos de un almacén de datos basados en archivos.<br/><br/>Los valores permitidos son:<br/><b>- PreserveHierarchy (valor predeterminado)</b>: conserva la jerarquía de archivos en la carpeta de destino. La ruta de acceso relativa del archivo de origen a la carpeta de origen es idéntica que la ruta de acceso relativa del archivo de destino a la carpeta de destino.<br/><b>- FlattenHierarchy</b>: todos los archivos de la carpeta de origen están en el primer nivel de la carpeta de destino. Los archivos de destino tienen nombres generados automáticamente. <br/><b>- MergeFiles</b>: combina todos los archivos de la carpeta de origen en un archivo. Si se especifica el nombre del blob o del archivo, el nombre de archivo combinado es el nombre especificado. De lo contrario, es un nombre de archivo generado automáticamente. | No       |
| blockSizeInMB | Especifique el tamaño de bloque en megabytes que se usa para escribir datos en blobs en bloques. Más información [sobre los blobs en bloques](https://docs.microsoft.com/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs#about-block-blobs). <br/>El valor permitido está *entre 4 MB y 100 MB*. <br/>De forma predeterminada, Data Factory determina automáticamente el tamaño de bloque en función del tipo y los datos del almacén de origen. En el caso de una copia no binaria en Blob Storage, el tamaño de bloque predeterminado es de 100 MB, para que pueda ajustarse (como máximo) a 4,95 TB de datos. Este tamaño podría no ser óptimo cuando los datos no son grandes, en especial cuando se usa el entorno de ejecución de integración autohospedado con conexiones de red deficientes, lo que genera problemas de tiempo de espera de operación o rendimiento. Puede especificar explícitamente un tamaño de bloque, a la vez que garantiza que `blockSizeInMB*50000` sea lo suficientemente grande como para almacenar los datos. De lo contrario, se generará un error en la ejecución de la actividad de copia. | No |
| maxConcurrentConnections | Número de conexiones simultáneas al almacenamiento. Solo se especifica cuando se quieren limitar las conexiones simultáneas al almacén de datos. | No       |

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyFromBlob",
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
                    "type": "AzureBlobStorageWriteSettings",
                    "copyBehavior": "PreserveHierarchy"
                }
            }
        }
    }
]
```

### <a name="folder-and-file-filter-examples"></a>Ejemplos de filtros de carpetas y archivos

Esta sección describe el comportamiento resultante de la ruta de acceso de la carpeta y el nombre de archivo con los filtros de carácter comodín.

| folderPath | fileName | recursive | Resultado de estructura de carpeta de origen y filtro (se recuperan los archivos en **negrita**)|
|:--- |:--- |:--- |:--- |
| `container/Folder*` | (vacío, usar el valor predeterminado) | false | contenedor<br/>&nbsp;&nbsp;&nbsp;&nbsp;FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| `container/Folder*` | (vacío, usar el valor predeterminado) | true | contenedor<br/>&nbsp;&nbsp;&nbsp;&nbsp;FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File4.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File5.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| `container/Folder*` | `*.csv` | false | contenedor<br/>&nbsp;&nbsp;&nbsp;&nbsp;FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| `container/Folder*` | `*.csv` | true | contenedor<br/>&nbsp;&nbsp;&nbsp;&nbsp;FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File5.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |

### <a name="file-list-examples"></a>Ejemplos de lista de archivos

En esta sección se describe el comportamiento resultante de usar una ruta de acceso de la lista de archivos en el origen de la actividad de copia.

Suponga que tiene la siguiente estructura de carpetas de origen y quiere copiar los archivos en negrita:

| Estructura de origen de ejemplo                                      | Contenido de FileListToCopy.txt                             | Configuración de Data Factory                                            |
| ------------------------------------------------------------ | --------------------------------------------------------- | ------------------------------------------------------------ |
| contenedor<br/>&nbsp;&nbsp;&nbsp;&nbsp;FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File5.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Metadatos<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;FileListToCopy.txt | File1.csv<br>Subfolder1/File3.csv<br>Subfolder1/File5.csv | **En el conjunto de datos:**<br>- Contenedor: `container`<br>- Ruta de acceso de la carpeta: `FolderA`<br><br>**En el origen de la actividad de copia:**<br>- Ruta de acceso de la lista de archivos: `container/Metadata/FileListToCopy.txt` <br><br>La ruta de acceso de la lista de archivos apunta a un archivo de texto en el mismo almacén de datos que incluye una lista de archivos que se quieren copiar, con un archivo por línea, con la ruta de acceso relativa a la ruta de acceso configurada en el conjunto de datos. |

### <a name="some-recursive-and-copybehavior-examples"></a>Algunos ejemplos de recursive y copyBehavior

En esta sección se describe el comportamiento resultante de la operación de copia para diferentes combinaciones de valores **recursive** y **copyBehavior**.

| recursive | copyBehavior | Estructura de carpetas de origen | Destino resultante |
|:--- |:--- |:--- |:--- |
| true |preserveHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | La carpeta de destino Folder1 se crea con la misma estructura que la de origen:<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 |
| true |flattenHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | La carpeta de destino Folder1 se crea con la misma estructura que la de origen: <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;nombre de archivo generado automáticamente para File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;nombre de archivo generado automáticamente para File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;nombre de archivo generado automáticamente para File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;nombre de archivo generado automáticamente para File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;nombre de archivo generado automáticamente para File5 |
| true |mergeFiles | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | La carpeta de destino Folder1 se crea con la misma estructura que la de origen: <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;El contenido de File1 + File2 + File3 + File4 + File5 se combina en un archivo con un nombre de archivo generado automáticamente. |
| false |preserveHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | La carpeta de destino Folder1 se crea con la misma estructura que la de origen: <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/><br/>No se selecciona la subcarpeta Subfolder1, con File3, File4 y File5. |
| false |flattenHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | La carpeta de destino Folder1 se crea con la misma estructura que la de origen: <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;nombre de archivo generado automáticamente para File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;nombre de archivo generado automáticamente para File2<br/><br/>No se selecciona la subcarpeta Subfolder1, con File3, File4 y File5. |
| false |mergeFiles | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | La carpeta de destino Folder1 se crea con la misma estructura que la de origen:<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;El contenido de File1 + File2 se combina en un archivo con un nombre de archivo generado automáticamente. nombre de archivo generado automáticamente para File1<br/><br/>No se selecciona la subcarpeta Subfolder1, con File3, File4 y File5. |

## <a name="preserving-metadata-during-copy"></a>Conservación de los metadatos durante la copia

Al copiar archivos de Amazon S3, Azure Blob Storage o Azure Data Lake Storage Gen2 a Azure Data Lake Storage Gen2 o Azure Blob Storage, puede optar por conservar los metadatos de archivo junto con los datos. Más información en [Conservación de metadatos](copy-activity-preserve-metadata.md#preserve-metadata).

## <a name="mapping-data-flow-properties"></a>Propiedades de Asignación de instancias de Data Flow

Al transformar datos en los flujos de datos de asignación, puede leer y escribir archivos de Azure Blob Storage en los siguientes formatos:
* [Avro](format-avro.md#mapping-data-flow-properties)
* [Texto delimitado](format-delimited-text.md#mapping-data-flow-properties)
* [Delta](format-delta.md#mapping-data-flow-properties)
* [Excel](format-excel.md#mapping-data-flow-properties)
* [JSON](format-json.md#mapping-data-flow-properties)
* [Parquet](format-parquet.md#mapping-data-flow-properties)

La configuración específica de formato se encuentra en la documentación de ese formato. Para obtener más información, consulte los artículos sobre la [transformación de origen en un flujo de datos de asignación](data-flow-source.md) y la [transformación de receptor en un flujo de datos de asignación](data-flow-sink.md).

### <a name="source-transformation"></a>Transformación de origen

En la transformación de origen, puede leer de un contenedor, una carpeta o un archivo individual en Azure Blob Storage. Use la pestaña **Opciones de origen** para administrar cómo se leen los archivos. 

![Opciones de origen](media/data-flow/sourceOptions1.png "Opciones de origen")

**Rutas de acceso con carácter comodín:** El uso de un patrón de caracteres comodín indicará a Data Factory que recorra todos los archivos y carpetas que coincidan en una única transformación del origen. Se trata de una manera eficaz de procesar varios archivos en un único flujo. Agregue varios patrones de coincidencia de caracteres comodín con el signo más que aparece al desplazar el puntero sobre el patrón de caracteres comodín existente.

En el contenedor de origen, elija una serie de archivos que coincidan con un patrón. Solo se puede especificar un contenedor en el conjunto de datos. La ruta de acceso con carácter comodín, por tanto, también debe incluir la ruta de acceso de la carpeta de la carpeta raíz.

Ejemplos de caracteres comodín:

* ```*``` Representa cualquier conjunto de caracteres.
* ```**``` Representa el anidamiento recursivo de directorios.
* ```?``` Reemplaza un carácter.
* ```[]``` Coincide con al menos uno de los caracteres entre corchetes.

* ```/data/sales/**/*.csv``` Obtiene todos los archivos .csv que se encuentran en /data/sales.
* ```/data/sales/20??/**/``` Obtiene todos los archivos del siglo XX.
* ```/data/sales/*/*/*.csv``` Obtiene los archivos .csv dos niveles debajo de /data/sales.
* ```/data/sales/2004/*/12/[XY]1?.csv``` Obtiene todos los archivos .csv de diciembre de 2004 que comienzan con X o Y precedido por un número de dos dígitos.

**Ruta de acceso raíz de la partición:** Si tiene carpetas con particiones en el origen de archivo con un formato ```key=value``` (por ejemplo, `year=2019`), puede asignar el nivel superior del árbol de carpetas de la partición a un nombre de columna del flujo de datos.

En primer lugar, establezca un comodín que incluya todas las rutas de acceso que sean carpetas con particiones y, además, los archivos de hoja que quiera leer.

![Configuración del archivo de origen de partición](media/data-flow/partfile2.png "Configuración del archivo de partición")

Use el valor de **Partition root path** (Ruta de acceso de la raíz de la partición) para definir cuál es el nivel superior de la estructura de carpetas. Cuando vea el contenido de los datos mediante una vista previa, verá que Data Factory agregará las particiones resueltas que se encuentran en cada uno de los niveles de las carpetas.

![Ruta de acceso raíz de la partición](media/data-flow/partfile1.png "Vista previa de la ruta de acceso raíz de la partición")

**Lista de archivos**: Se trata de un conjunto de archivos. Cree un archivo de texto que incluya una lista de archivos de ruta de acceso relativa para procesar. Apunte a este archivo de texto.

**Column to store file name:** (Columna para almacenar el nombre de archivo) Almacene el nombre del archivo de origen en una columna de los datos. Escriba aquí el nombre de una nueva columna para almacenar la cadena de nombre de archivo.

**After completion:** (Tras finalizar) Elija no hacer nada con el archivo de origen después de que se ejecute el flujo de datos o bien elimine o mueva el archivo de origen. Las rutas de acceso para mover los archivos de origen son relativas.

Para mover archivos de origen a otra ubicación posterior al procesamiento, primero seleccione "Mover" para la operación de archivo. A continuación, establezca el directorio "from". Si no usa ningún carácter comodín para la ruta de acceso, la configuración de "from" será la misma carpeta que la carpeta de origen.

Si tiene una ruta de acceso de origen con un comodín, su sintaxis será así:

```/data/sales/20??/**/*.csv```

Puede especificar "from" como:

```/data/sales```

Y puede especificar "to" como:

```/backup/priorSales```

En este caso, todos los subdirectorios cuyo origen se encuentra en /data/sales se mueven a /backup/priorSales.

> [!NOTE]
> Las operaciones de archivo solo se ejecutan cuando el flujo de datos se inicia desde una ejecución de canalización (depuración o ejecución de canalización) que usa la actividad de ejecución de Data Flow de una canalización. Las operaciones de archivo *no* se ejecutan en modo de depuración de Data Flow.

**Filter by last modified:** (Filtrar últimos modificados) Puede filtrar los archivos que desea procesar especificando un intervalo de fechas de la última vez que se modificaron. Todos los valores datetime están en formato UTC. 

### <a name="sink-properties"></a>Propiedades del receptor

En la transformación del receptor, puede escribir en un contenedor o carpeta en Azure Blob Storage. Use la pestaña **Configuración** para administrar cómo se escriben los archivos.

![Opciones del receptor](media/data-flow/file-sink-settings.png "opciones de receptor")

**Clear the folder** (Borrar la carpeta): determina si se borra o no la carpeta de destino antes de escribir los datos.

**File name option** (Opción de nombre de archivo): determina cómo se denominan los archivos de destino en la carpeta de destino. Las opciones de nombre de archivo son:
   * **Valor predeterminado**: permita que Spark nombre los archivos según los valores predeterminados de PART.
   * **Patrón**: escriba un patrón que enumere los archivos de salida por partición. Por ejemplo, **loans[n].csv** creará loans1.csv, loans2.csv, etc.
   * **Por partición**: escriba un nombre de archivo por partición.
   * **Como datos de columna**: establezca el archivo de salida en el valor de una columna. La ruta de acceso es relativa al contenedor del conjunto de datos, no a la carpeta de destino. Si tiene una ruta de acceso de carpeta en el conjunto de datos, se invalidará.
   * **Salida en un solo archivo**: combine los archivos de salida con particiones en un solo archivo con nombre. La ruta de acceso es relativa a la carpeta del conjunto de datos. Tenga en cuenta que la operación Merge probablemente produzca un error en función del tamaño del nodo. No se recomienda esta opción para conjuntos de datos de gran tamaño.

**Quote all** (Entrecomillar todo): determina si todos los valores se deben entrecomillar.

## <a name="lookup-activity-properties"></a>Propiedades de la actividad de búsqueda

Para obtener información detallada sobre las propiedades, consulte [Actividad de búsqueda](control-flow-lookup-activity.md).

## <a name="getmetadata-activity-properties"></a>Propiedades de la actividad GetMetadata

Para información detallada sobre las propiedades, consulte [la actividad GetMetadata](control-flow-get-metadata-activity.md). 

## <a name="delete-activity-properties"></a>Propiedades de la actividad de eliminación

Para información detallada sobre las propiedades, consulte [Actividad de eliminación](delete-activity.md).

## <a name="legacy-models"></a>Modelos heredados

>[!NOTE]
>Estos modelos siguen siendo compatibles con versiones anteriores. Se recomienda usar el nuevo modelo mencionado anteriormente. La interfaz de usuario de creación en Data Factory ha cambiado para generar el nuevo modelo.

### <a name="legacy-dataset-model"></a>Modelo de conjunto de datos heredado

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type** del conjunto de datos tiene que establecerse en **AzureBlob**. |Sí |
| folderPath | Ruta de acceso para el contenedor y la carpeta en Blob Storage. <br/><br/>Un filtro de comodín es compatible con la ruta de acceso sin incluir el nombre del contenedor. Los caracteres comodín permitidos son: `*` (equivale a cero o a varios caracteres) y `?` (equivale a cero o a un único carácter). Use `^` como escape si el nombre de la carpeta contiene un carácter comodín o este carácter de escape. <br/><br/>Un ejemplo: myblobcontainer/myblobfolder/. Ver más ejemplos en [Ejemplos de filtros de carpetas y archivos](#folder-and-file-filter-examples). |Sí, para la actividad de copia o búsqueda; no, para la actividad GetMetadata |
| fileName | Filtro de nombre o carácter comodín para los blobs de la ruta **folderPath** especificada. Si no especifica ningún valor para esta propiedad, el conjunto de datos apunta a todos los blobs de la carpeta. <br/><br/>Para el filtro, los caracteres comodín permitidos son: `*` (equivale a cero o a varios caracteres) y `?` (equivale a cero o a un único carácter).<br/>- Ejemplo 1: `"fileName": "*.csv"`<br/>- Ejemplo 2: `"fileName": "???20180427.txt"`<br/>Use `^` como escape si el nombre de archivo contiene un carácter comodín o este carácter de escape.<br/><br/>Cuando **fileName** no se especifica para un conjunto de datos de salida y **preserveHierarchy** no se determina en el receptor de la actividad, la actividad de copia generará automáticamente el nombre de blob con el siguiente patrón: "*Data.[GUID de id. de ejecución de actividad].[GUID, si FlattenHierarchy].[formato, si está configurado].[compresión, si está configurada]* ". Por ejemplo: "Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt.gz". <br/><br/>Si realiza una copia desde el origen tabular usando el nombre de una tabla en lugar de una consulta, el patrón del nombre es " *[nombre de tabla].[formato].[compresión, si está configurada]* ". Por ejemplo: "MyTable.csv". |No |
| modifiedDatetimeStart | Los archivos se filtran en función del atributo Last Modified. Los archivos se seleccionarán si la hora de su última modificación está dentro del intervalo de tiempo entre `modifiedDatetimeStart` y `modifiedDatetimeEnd`. La hora se aplica a la zona horaria UTC en el formato "2018-12-01T05:00:00Z". <br/><br/> Tenga en cuenta que al habilitar esta configuración se verá afectado el rendimiento general del movimiento de datos cuando quiera filtrar grandes cantidades de archivos. <br/><br/> Las propiedades pueden ser **NULL**, lo que significa que no se aplica ningún filtro de atributo de archivo al conjunto de datos.  Cuando `modifiedDatetimeStart` tiene un valor de fecha y hora, pero `modifiedDatetimeEnd` es **NULL**, significa que se seleccionarán los archivos cuyo último atributo modificado sea mayor o igual que el valor de fecha y hora.  Cuando `modifiedDatetimeEnd` tiene un valor de fecha y hora, pero `modifiedDatetimeStart` es **NULL**, significa que se seleccionarán los archivos cuyo último atributo modificado sea menor que el valor de fecha y hora.| No |
| modifiedDatetimeEnd | Los archivos se filtran en función del atributo Last Modified. Los archivos se seleccionarán si la hora de su última modificación está dentro del intervalo de tiempo entre `modifiedDatetimeStart` y `modifiedDatetimeEnd`. La hora se aplica a la zona horaria UTC en el formato "2018-12-01T05:00:00Z". <br/><br/> Tenga en cuenta que al habilitar esta configuración se verá afectado el rendimiento general del movimiento de datos cuando quiera filtrar grandes cantidades de archivos. <br/><br/> Las propiedades pueden ser **NULL**, lo que significa que no se aplica ningún filtro de atributo de archivo al conjunto de datos.  Cuando `modifiedDatetimeStart` tiene un valor de fecha y hora, pero `modifiedDatetimeEnd` es **NULL**, significa que se seleccionarán los archivos cuyo último atributo modificado sea mayor o igual que el valor de fecha y hora.  Cuando `modifiedDatetimeEnd` tiene un valor de fecha y hora, pero `modifiedDatetimeStart` es **NULL**, significa que se seleccionarán los archivos cuyo último atributo modificado sea menor que el valor de fecha y hora.| No |
| format | Si desea copiar los archivos tal cual entre los almacenes basados en archivos (copia binaria), omita la sección de formato en las definiciones de los conjuntos de datos de entrada y salida.<br/><br/>Si quiere analizar o generar archivos con un formato concreto, se admiten los siguientes tipos de formato de archivo: **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat** y **ParquetFormat**. Establezca la propiedad **type** en **format** en uno de los siguientes valores. Para más información, consulte las secciones [Formato de texto](supported-file-formats-and-compression-codecs-legacy.md#text-format), [Formato JSON](supported-file-formats-and-compression-codecs-legacy.md#json-format), [Formato AVRO](supported-file-formats-and-compression-codecs-legacy.md#avro-format), [Formato ORC](supported-file-formats-and-compression-codecs-legacy.md#orc-format) y [Formato Parquet](supported-file-formats-and-compression-codecs-legacy.md#parquet-format). |No (solo para el escenario de copia binaria) |
| compression | Especifique el tipo y el nivel de compresión de los datos. Para más información, consulte el artículo sobre [códecs de compresión y formatos de archivo compatibles](supported-file-formats-and-compression-codecs-legacy.md#compression-support).<br/>Los tipos admitidos son **GZip**, **Deflate**, **BZip2** y **ZipDeflate**.<br/>Niveles admitidos son **Optimal** y **Fastest**. |No |

>[!TIP]
>Para copiar todos los blobs en una carpeta, especifique solo **folderPath**.<br>Para copiar un único blob con un nombre determinado, especifique **folderPath** para la sección de la carpeta y **fileName** para el nombre de archivo.<br>Para copiar un subconjunto de blobs en una carpeta, especifique **folderPath** para la sección de la carpeta y **fileName** con un filtro de comodín. 

**Ejemplo**:

```json
{
    "name": "AzureBlobDataset",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": {
            "referenceName": "<Azure Blob storage linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "folderPath": "mycontainer/myfolder",
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

### <a name="legacy-source-model-for-the-copy-activity"></a>Modelo de origen heredado para la actividad de copia

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type** del origen de la actividad de copia tiene que establecerse en **BlobSource**. |Sí |
| recursive | Indica si los datos se leen de forma recursiva de las subcarpetas o solo de la carpeta especificada. Tenga en cuenta que cuando **recursive** se establece en **true** y el receptor es un almacén basado en archivos, no se crea una carpeta o una subcarpeta vacía en el receptor.<br/>Los valores permitidos son: **True** (valor predeterminado) y **False**. | No |
| maxConcurrentConnections | Número de conexiones simultáneas al almacenamiento. Solo se especifica cuando se quieren limitar las conexiones simultáneas al almacén de datos. | No |

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyFromBlob",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Azure Blob input dataset name>",
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
                "type": "BlobSource",
                "recursive": true
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="legacy-sink-model-for-the-copy-activity"></a>Modelo de receptor heredado para la actividad de copia

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type** del receptor de la actividad de copia debe establecerse en **BlobSink**. |Sí |
| copyBehavior | Define el comportamiento de copia cuando el origen son archivos de un almacén de datos basados en archivos.<br/><br/>Los valores permitidos son:<br/><b>- PreserveHierarchy (valor predeterminado)</b>: conserva la jerarquía de archivos en la carpeta de destino. La ruta de acceso relativa del archivo de origen que apunta a la carpeta de origen es idéntica a la ruta de acceso relativa del archivo de destino que apunta a la carpeta de destino.<br/><b>- FlattenHierarchy</b>: todos los archivos de la carpeta de origen están en el primer nivel de la carpeta de destino. Los archivos de destino tienen nombres generados automáticamente. <br/><b>- MergeFiles</b>: combina todos los archivos de la carpeta de origen en un archivo. Si se especifica el nombre del blob o del archivo, el nombre de archivo combinado es el nombre especificado. De lo contrario, es un nombre de archivo generado automáticamente. | No |
| maxConcurrentConnections | Número de conexiones simultáneas al almacenamiento. Solo se especifica cuando se quieren limitar las conexiones simultáneas al almacén de datos. | No |

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyToBlob",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Azure Blob output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "BlobSink",
                "copyBehavior": "PreserveHierarchy"
            }
        }
    }
]
```

## <a name="next-steps"></a>Pasos siguientes

Para ver una lista de los almacenes de datos que la actividad de copia de Data Factory admite como orígenes y receptores, consulte los [almacenes de datos que se admiten](copy-activity-overview.md#supported-data-stores-and-formats).
