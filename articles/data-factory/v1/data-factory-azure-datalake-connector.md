---
title: Copia de datos hacia y desde Azure Data Lake Storage Gen1
description: Aprenda a copiar datos hacia y desde Data Lake Store mediante Azure Data Factory
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.assetid: 25b1ff3c-b2fd-48e5-b759-bb2112122e30
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: jingwang
ms.custom: devx-track-csharp
robots: noindex
ms.openlocfilehash: fe3401354d4853b875cdd001d5074ebdf0d3377b
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89019545"
---
# <a name="copy-data-to-and-from-data-lake-storage-gen1-by-using-data-factory"></a>Copia de datos hacia y desde Data Lake Storage Gen1 mediante Data Factory
> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](data-factory-azure-datalake-connector.md)
> * [Versión 2 (versión actual)](../connector-azure-data-lake-store.md)

> [!NOTE]
> Este artículo se aplica a la versión 1 de Data Factory. Si usa la versión actual del servicio Data Factory, vea [Azure Data Lake Storage Gen1 connector in V2](../connector-azure-data-lake-store.md) (Conector de Azure Data Lake Storage Gen1 en V2).

En este artículo se explica cómo usar la actividad de copia de Azure Data Factory para mover datos con Azure Data Lake Storage Gen1 (anteriormente conocido como Azure Data Lake Store) como origen o destino. Se basa en el artículo [Movimiento de datos con la actividad de copia](data-factory-data-movement-activities.md), una introducción al movimiento de datos con la actividad de copia.

## <a name="supported-scenarios"></a>Escenarios admitidos
Puede copiar datos **desde Azure Data Lake Store** hacia los siguientes almacenes de datos:

[!INCLUDE [data-factory-supported-sinks](../../../includes/data-factory-supported-sinks.md)]

Puede copiar datos desde los siguientes almacenes de datos **hacia Azure Data Lake Store**:

[!INCLUDE [data-factory-supported-sources](../../../includes/data-factory-supported-sources.md)]

> [!NOTE]
> Cree una cuenta de Data Lake Store antes de crear una canalización con la actividad de copia. Para más información, consulte [Introducción a Azure Data Lake Store](../../data-lake-store/data-lake-store-get-started-portal.md).

## <a name="supported-authentication-types"></a>Tipos de autenticación que se admiten
El conector de Data Lake Store admite estos tipos de autenticación:
* Autenticación de entidad de servicio
* Autenticación de credenciales de usuario (OAuth)

Se recomienda que use la autenticación de la entidad de servicio, en especial para una copia de datos programada. Con la autenticación se credenciales de usuario puede darse la situación de que expiren los tokens. Para información sobre los detalles de configuración, consulte la sección [Propiedades del servicio vinculado](#linked-service-properties).

## <a name="get-started"></a>Introducción
Puede crear una canalización con actividad de copia que mueva los datos hacia Azure Data Lake Store y desde este servicio mediante el uso de diferentes herramientas o API.

La manera más fácil de crear una canalización es usar el **Asistente para copia**. Para ver un tutorial sobre la creación de una canalización mediante el Asistente para copia, consulte [Tutorial: crear una canalización con el Asistente para copia](data-factory-copy-data-wizard-tutorial.md).

Puede usar las siguientes herramientas para crear una canalización: **Visual Studio**, **Azure PowerShell**, una **plantilla de Azure Resource Manager**, la **API de .NET** y **API REST**. Consulte el [tutorial de actividad de copia](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) para obtener instrucciones paso a paso para crear una canalización con una actividad de copia.

Tanto si usa las herramientas como las API, realice los pasos siguientes para crear una canalización que mueva datos de un almacén de datos de origen a un almacén de datos receptor:

1. Crear una **factoría de datos**. Una factoría de datos puede contener una o más canalizaciones.
2. Cree **servicios vinculados** para vincular almacenes de datos de entrada y salida a la factoría de datos. Por ejemplo, si va a copiar datos desde una instancia de Azure Blob Storage hacia una instancia de Azure Data Lake Store, creará dos servicios vinculados para vincular la cuenta de Azure Storage y la instancia de Azure Data Lake Store a su factoría de datos. Para información sobre las propiedades de los servicios vinculados que son específicas de Azure Data Lake Store, consulte la sección [Propiedades del servicio vinculado](#linked-service-properties).
2. Cree **conjuntos de datos** con el fin de representar los datos de entrada y salida para la operación de copia. En el ejemplo mencionado en el último paso, se crea un conjunto de datos para especificar el contenedor de blobs y la carpeta que contiene los datos de entrada. Además, se crea otro conjunto de datos para especificar la ruta de acceso al archivo y la carpeta en Data Lake Store que contiene los datos copiados de Blob Storage. Para información sobre las propiedades del conjunto de datos que son específicas de Azure Data Lake Store, consulte la sección [Propiedades del conjunto de datos](#dataset-properties).
3. Cree una **canalización** con una actividad de copia que tome como entrada un conjunto de datos y un conjunto de datos como salida. En el ejemplo que se ha mencionado anteriormente, se usa BlobSource como origen y AzureDataLakeStoreSink como receptor para la actividad de copia. De igual forma, si va a copiar desde Azure Data Lake Store hacia Azure Blob Storage, usará AzureDataLakeStoreSource y BlobSink en la actividad de copia. Para información sobre las propiedades de la actividad de copia que son específicas de Azure Data Lake Store, consulte la sección [Propiedades de la actividad de copia](#copy-activity-properties). Para obtener más información sobre cómo usar un almacén de datos como origen o receptor, haga clic en el vínculo de la sección anterior para el almacén de datos.

Cuando se usa el Asistente, se crean automáticamente definiciones de JSON para estas entidades de Data Factory (servicios vinculados, conjuntos de datos y la canalización). Al usar herramientas o API (excepto la API de .NET), se definen estas entidades de Data Factory con el formato JSON. Para obtener ejemplos con definiciones de JSON para entidades de Data Factory que se utilizan para copiar datos a Azure Data Lake Store y desde este servicio, consulte la sección [Ejemplos de JSON](#json-examples-for-copying-data-to-and-from-data-lake-store) de este artículo.

En las secciones siguientes se proporcionan detalles sobre las propiedades JSON que se usan para definir entidades de Data Factory específicas de Data Lake Store.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado
Un servicio vinculado vincula un almacén de datos a una factoría de datos. Creará un servicio vinculado de tipo **AzureDataLakeStore** para vincular datos de Data Lake Store a la factoría de datos. En la tabla siguiente se describen los elementos JSON específicos de los servicios vinculados Data Lake Store. Puede elegir entre la autenticación de la entidad de servicio y la autenticación de credenciales de usuario.

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| **type** | La propiedad type debe establecerse en: **AzureDataLakeStore**. | Sí |
| **dataLakeStoreUri** | Información sobre la cuenta de Azure Data Lake Store. Esta información adopta uno de los siguientes formatos: `https://[accountname].azuredatalakestore.net/webhdfs/v1` o `adl://[accountname].azuredatalakestore.net/`. | Sí |
| **subscriptionId** | Id. de suscripción de Azure al que pertenece la cuenta de Data Lake Store. | Necesario para el receptor |
| **resourceGroupName** | Nombre del grupo de recursos de Azure al que pertenece la cuenta de Data Lake Store. | Necesario para el receptor |

### <a name="service-principal-authentication-recommended"></a>Autenticación de la entidad de servicio (recomendada)
Para usar la autenticación de la entidad de servicio, registre una entidad de aplicación en Azure Active Directory (AAD) y concédale acceso a Data Lake Store. Consulte [Autenticación entre servicios](../../data-lake-store/data-lake-store-authenticate-using-active-directory.md) para ver los pasos detallados. Anote los siguientes valores; los usará para definir el servicio vinculado:
* Identificador de aplicación
* Clave de la aplicación
* Id. de inquilino

> [!IMPORTANT]
> Asegúrese de que concede el permiso adecuado principal del servicio en Azure Data Lake Store:
>- **Para usar Data Lake Store como origen**, conceda al menos el permiso de acceso a datos de **lectura y ejecución** para enumerar y copiar el contenido de una carpeta o permiso de **lectura** para copiar un único archivo. No hay ningún requisito en el control de acceso de nivel de cuenta.
>- **Para usar Data Lake Store como receptor**, garantice al menos el permiso de acceso a datos de **escritura y ejecución** para crear elementos secundarios en la carpeta. Además, si usa Azure IR para autorizar la copia (tanto el origen como el receptor están en la nube), para permitir a Data Factory detectar la región de Data Lake Storage, conceda al menos el rol de **lector** en el control de acceso de cuenta (IAM). Si desea evitar este rol IAM, [especifique executionLocation](data-factory-data-movement-activities.md#global) con la ubicación de Data Lake Store en la actividad de copia.
>- Si utiliza el **Asistente de copia para crear canalizaciones**, conceda al menos el rol de **lector** en el control de acceso a cuentas (IAM). Además, conceda al menos el permiso de **lectura y ejecución** para la raíz de Data Lake Store ("/") y sus elementos secundarios. De lo contrario, puede que aparezca el mensaje "Las credenciales proporcionadas no son válidas".

Para usar la autenticación de la entidad de servicio, especifique las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| **servicePrincipalId** | Especifique el id. de cliente de la aplicación. | Sí |
| **servicePrincipalKey** | Especifique la clave de la aplicación. | Sí |
| **tenant** | Especifique la información del inquilino (nombre de dominio o identificador de inquilino) en el que reside la aplicación. Para recuperarlo, mantenga el puntero del mouse en la esquina superior derecha de Azure Portal. | Sí |

**Ejemplo: Autenticación de entidad de servicio**
```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "subscriptionId": "<subscription of ADLS>",
            "resourceGroupName": "<resource group of ADLS>"
        }
    }
}
```

### <a name="user-credential-authentication"></a>Autenticación de credenciales de usuario
También puede usar la autenticación de credenciales de usuario para copiar hacia o desde Data Lake Store mediante la especificación de las propiedades siguientes:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| **authorization** | Haga clic en el botón **Autorizar** de Data Factory Editor y escriba sus credenciales, que asignan la dirección URL de autorización generada automáticamente a esta propiedad. | Sí |
| **sessionId** | Id. de sesión de OAuth de la sesión de autorización de OAuth. Cada id. de sesión es único y solo se puede usar una vez. Esta configuración se genera automáticamente al usar Data Factory Editor. | Sí |

> [!IMPORTANT]
> Asegúrese de que concede el permiso adecuado al usuario en Azure Data Lake Store:
>- **Para usar Data Lake Store como origen**, conceda al menos el permiso de acceso a datos de **lectura y ejecución** para enumerar y copiar el contenido de una carpeta o permiso de **lectura** para copiar un único archivo. No hay ningún requisito en el control de acceso de nivel de cuenta.
>- **Para usar Data Lake Store como receptor**, garantice al menos el permiso de acceso a datos de **escritura y ejecución** para crear elementos secundarios en la carpeta. Además, si usa Azure IR para autorizar la copia (tanto el origen como el receptor están en la nube), para permitir a Data Factory detectar la región de Data Lake Storage, conceda al menos el rol de **lector** en el control de acceso de cuenta (IAM). Si desea evitar este rol IAM, [especifique executionLocation](data-factory-data-movement-activities.md#global) con la ubicación de Data Lake Store en la actividad de copia.
>- Si utiliza el **Asistente de copia para crear canalizaciones**, conceda al menos el rol de **lector** en el control de acceso a cuentas (IAM). Además, conceda al menos el permiso de **lectura y ejecución** para la raíz de Data Lake Store ("/") y sus elementos secundarios. De lo contrario, puede que aparezca el mensaje "Las credenciales proporcionadas no son válidas".

**Ejemplo: Autenticación de credenciales de usuario**
```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "sessionId": "<session ID>",
            "authorization": "<authorization URL>",
            "subscriptionId": "<subscription of ADLS>",
            "resourceGroupName": "<resource group of ADLS>"
        }
    }
}
```

#### <a name="token-expiration"></a>Expiración del token
El código de autorización que se genera con el botón **Autorizar** expira transcurrido un período de tiempo determinado. El siguiente mensaje significa que expiró el token de autenticación:

Error de operación de credencial: invalid_grant - AADSTS70002: error al validar las credenciales. AADSTS70008: la concesión de acceso proporcionada expiró o se revocó. Identificador de seguimiento: d18629e8-af88-43c5-88e3-d8419eb1fca1 Identificador de correlación: fac30a0c-6be6-4e02-8d69-a776d2ffefd7 Marca de tiempo: 2015-12-15 21-09-31Z.

En la tabla siguiente se muestran los tiempos de expiración de los distintos tipos de cuentas de usuario:

| Tipo de usuario | Expira después de |
|:--- |:--- |
| Cuentas de usuario *no* administradas por Azure Active Directory (por ejemplo, @hotmail.com o @live.com) |12 horas |
| Cuentas de usuario administradas por Azure Active Directory |14 días después de la ejecución del último segmento. <br/><br/>90 días, si un segmento basado en un servicio vinculado basado en OAuth se ejecuta al menos una vez cada 14 días. |

Si cambia la contraseña antes de la hora de expiración del token, el token expira inmediatamente. Verá el mensaje mencionado anteriormente en esta sección.

Puede volver a autorizar la cuenta mediante el botón **Autorizar** cuando expire el token para volver a implementar el servicio vinculado. También puede generar valores para las propiedades **sessionId** y **authorization** mediante programación por medio del código siguiente:


```csharp
if (linkedService.Properties.TypeProperties is AzureDataLakeStoreLinkedService ||
    linkedService.Properties.TypeProperties is AzureDataLakeAnalyticsLinkedService)
{
    AuthorizationSessionGetResponse authorizationSession = this.Client.OAuth.Get(this.ResourceGroupName, this.DataFactoryName, linkedService.Properties.Type);

    WindowsFormsWebAuthenticationDialog authenticationDialog = new WindowsFormsWebAuthenticationDialog(null);
    string authorization = authenticationDialog.AuthenticateAAD(authorizationSession.AuthorizationSession.Endpoint, new Uri("urn:ietf:wg:oauth:2.0:oob"));

    AzureDataLakeStoreLinkedService azureDataLakeStoreProperties = linkedService.Properties.TypeProperties as AzureDataLakeStoreLinkedService;
    if (azureDataLakeStoreProperties != null)
    {
        azureDataLakeStoreProperties.SessionId = authorizationSession.AuthorizationSession.SessionId;
        azureDataLakeStoreProperties.Authorization = authorization;
    }

    AzureDataLakeAnalyticsLinkedService azureDataLakeAnalyticsProperties = linkedService.Properties.TypeProperties as AzureDataLakeAnalyticsLinkedService;
    if (azureDataLakeAnalyticsProperties != null)
    {
        azureDataLakeAnalyticsProperties.SessionId = authorizationSession.AuthorizationSession.SessionId;
        azureDataLakeAnalyticsProperties.Authorization = authorization;
    }
}
```
Para más información sobre las clases de Data Factory que se usan en el código, consulte los temas [AzureDataLakeStoreLinkedService Class](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.azuredatalakestorelinkedservice.aspx), [AzureDataLakeAnalyticsLinkedService Class](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.azuredatalakeanalyticslinkedservice.aspx) y [AuthorizationSessionGetResponse Class](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.authorizationsessiongetresponse.aspx). Agregue una referencia a la versión `2.9.10826.1824` de `Microsoft.IdentityModel.Clients.ActiveDirectory.WindowsForms.dll` para la clase `WindowsFormsWebAuthenticationDialog` usada en el código.

## <a name="troubleshooting-tips"></a>Sugerencias de solución de problemas

**Síntoma**: cuando se copian datos **en** Azure Data Lake Store, la actividad de copia produce el siguiente error:

  ```
  Failed to detect the region for Azure Data Lake account {your account name}. Please make sure that the Resource Group name: {resource group name} and subscription ID: {subscription ID} of this Azure Data Lake Store resource are correct.
  ```

**Causa principal:** existen 2 razones posibles.

1. El elemento `resourceGroupName` o `subscriptionId` especificado en el servicio vinculado de Azure Data Lake Store no es correcto.
2. El usuario o la entidad de servicio no tienen el permiso necesario.

**Resolución:**

1. Asegúrese de que los elementos `subscriptionId` y `resourceGroupName` que se especifican en el servicio vinculado `typeProperties` son los que pertenecen a su cuenta de Data Lake.

2. Asegúrese de que se concede al menos el rol de **lector** al usuario o a la entidad de servicio en la cuenta de Data Lake. Aquí se muestra cómo hacerlo:

    1. Vaya a Azure Portal -> su cuenta de Data Lake Store.
    2. Haga clic en **Control de acceso (IAM)** en la hoja de Data Lake Store.
    3. Haga clic en **Agregar asignación de roles**.
    4. Establezca **Rol** en **Lector** y seleccione el usuario o la entidad de servicio que usará a modo de copia para conceder acceso.

3. Si no desea conceder el rol **Lector** al usuario o a la entidad de servicio, puede especificar [explícitamente una ubicación de ejecución](data-factory-data-movement-activities.md#global) en la actividad de copia junto con la ubicación de Data Lake Store. Ejemplo:

    ```json
    {
      "name": "CopyToADLS",
      "type": "Copy",
      ......
      "typeProperties": {
        "source": {
          "type": "<source type>"
        },
        "sink": {
          "type": "AzureDataLakeStoreSink"
        },
        "exeuctionLocation": "West US"
      }
    }
    ```

## <a name="dataset-properties"></a>Propiedades del conjunto de datos
Para especificar un conjunto de datos para representar datos de entrada en Data Lake Store, establezca la propiedad **type** del conjunto de datos en **AzureDataLakeStore**. Establezca la propiedad **linkedServiceName** del conjunto de datos en el nombre del servicio vinculado Data Lake Store. Para ver una lista completa de las secciones y propiedades JSON disponibles para definir conjuntos de datos, consulte el artículo [Creación de conjuntos de datos](data-factory-create-datasets.md). Las secciones de un conjunto de datos de JSON, como **structure**, **availability** y **policy**, son similares para todos los tipos de datos (base de datos de Azure SQL, blob de Azure y tabla de Azure, por ejemplo). La sección **typeProperties** es diferente para cada tipo de conjunto de datos y proporciona información como la ubicación y el formato de los datos del almacén de datos.

La sección **typeProperties** de un conjunto de datos de tipo **AzureDataLakeStore** contiene las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| **folderPath** |Ruta de acceso al contenedor y la carpeta de Data Lake Store. |Sí |
| **fileName** |Nombre del archivo en Azure Data Lake Store. La propiedad **fileName** es opcional y distingue mayúsculas de minúsculas. <br/><br/>Si especifica **fileName**, la actividad (incluida la copia) funciona en el archivo específico.<br/><br/>Cuando no se especifica **fileName**, la copia incluye todos los archivos de **folderPath** en el conjunto de datos de entrada.<br/><br/>Cuando **fileName** no se especifica para un conjunto de datos de salida y **preserveHierarchy** no se especifica en el receptor de la actividad, el nombre del archivo generado está en el formato `Data._Guid_.txt`. Por ejemplo: Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt. |No |
| **partitionedBy** |La propiedad **partitionedBy** es opcional. Puede usarla para especificar una ruta de acceso dinámica y un nombre de archivo para datos de series temporales. Por ejemplo, se puede parametrizar **folderPath** por cada hora de datos. Para más información y ejemplos, consulte La propiedad partitionedBy. |No |
| **format** | Se admiten los tipos de formato siguientes: **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat** y **ParquetFormat**. Establezca la propiedad **type** en **format** en uno de los siguientes valores. Para más información, consulte las secciones [Formato de texto](data-factory-supported-file-and-compression-formats.md#text-format), [Formato JSON](data-factory-supported-file-and-compression-formats.md#json-format), [Formato Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [Formato ORC](data-factory-supported-file-and-compression-formats.md#orc-format) y [Formato Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format) del artículo [Formatos de compresión de archivos admitidos por Azure Data Factory](data-factory-supported-file-and-compression-formats.md). <br><br> Si desea copiar los archivos tal cual entre los almacenes basados en archivos (copia binaria), omita la sección `format` en las definiciones de los conjuntos de datos de entrada y salida. |No |
| **compression** | Especifique el tipo y el nivel de compresión de los datos. Los tipos admitidos son **GZip**, **Deflate**, **BZip2** y **ZipDeflate**. Niveles admitidos son **Optimal** y **Fastest**. Para más información, consulte el artículo sobre [Formatos de compresión de archivos admitidos por Azure Data Factory](data-factory-supported-file-and-compression-formats.md#compression-support). |No |

### <a name="the-partitionedby-property"></a>La propiedad partitionedBy
Puede especificar las propiedades dinámicas **folderPath** y **fileName** para los datos de series temporales con la propiedad **partitionedBy**, funciones de Data Factory y variables del sistema. Para más información, consulte el artículo [Azure Data Factory: funciones y variables del sistema](data-factory-functions-variables.md).


En el ejemplo siguiente, `{Slice}` se reemplaza por el valor de la variable del sistema de Data Factory `SliceStart` en el formato especificado (`yyyyMMddHH`). El nombre `SliceStart` hace referencia a la hora de inicio del segmento. La propiedad `folderPath` es diferente para cada segmento, como en `wikidatagateway/wikisampledataout/2014100103` o `wikidatagateway/wikisampledataout/2014100104`.

```JSON
"folderPath": "wikidatagateway/wikisampledataout/{Slice}",
"partitionedBy":
[
    { "name": "Slice", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyyMMddHH" } },
],
```

En el ejemplo siguiente, el año, el mes, el día y la hora de `SliceStart` se extraen en variables independientes que usan las propiedades `folderPath` y `fileName`:
```JSON
"folderPath": "wikidatagateway/wikisampledataout/{Year}/{Month}/{Day}",
"fileName": "{Hour}.csv",
"partitionedBy":
[
    { "name": "Year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
    { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } },
    { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } },
    { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "hh" } }
],
```
Para más información sobre los conjuntos de datos, la programación y los segmentos de serie temporal, consulte [Conjuntos de datos en Azure Data Factory](data-factory-create-datasets.md) y [Programación y ejecución de Data Factory](data-factory-scheduling-and-execution.md).


## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia
Para ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo [Creación de canalizaciones](data-factory-create-pipelines.md). Las propiedades (como nombre, descripción, tablas de entrada y salida, y directivas) están disponibles para todos los tipos de actividades.

Las propiedades disponibles en la sección **typeProperties** de una actividad varían con cada tipo de actividad. En una actividad de copia, varían en función de los tipos de orígenes y receptores.

**AzureDataLakeStoreSource** admite la siguiente propiedad en la sección **typeProperties**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| **recursive** |Indica si los datos se leen de forma recursiva de las subcarpetas o solo de la carpeta especificada. |True (valor predeterminado), False |No |

**AzureDataLakeStoreSink** admite las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| **copyBehavior** |Especifica el comportamiento de copia. |<b>PreserveHierarchy</b>: conserva la jerarquía de archivos en la carpeta de destino. La ruta de acceso relativa del archivo de origen que apunta a la carpeta de origen es idéntica a la ruta de acceso relativa del archivo de destino que apunta a la carpeta de destino.<br/><br/><b>FlattenHierarchy</b>: todos los archivos de la carpeta de origen se crean en el primer nivel de la carpeta de destino. Los archivos de destino se crean con nombres generados automáticamente.<br/><br/><b>MergeFiles</b>: combina todos los archivos de la carpeta de origen en un archivo. Si se especifica el nombre del blob o del archivo, el nombre de archivo combinado es el nombre especificado. En caso contrario, el nombre de archivo se genera automáticamente. |No |

### <a name="recursive-and-copybehavior-examples"></a>Ejemplos de recursive y copyBehavior
En esta sección se describe el comportamiento resultante de la operación de copia para diferentes combinaciones de valores recursive y copyBehavior.

| recursive | copyBehavior | Comportamiento resultante |
| --- | --- | --- |
| true |preserveHierarchy |Si la carpeta de origen Folder1 tiene esta estructura: <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>la carpeta de destino Folder1 se crea con la misma estructura que la de origen<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5. |
| true |flattenHierarchy |Si la carpeta de origen Folder1 tiene esta estructura: <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>la carpeta de destino Folder1 se crea con la estructura siguiente: <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Nombre de archivo generado automáticamente para File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Nombre de archivo generado automáticamente para File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Nombre de archivo generado automáticamente para File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;Nombre de archivo generado automáticamente para File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;Nombre de archivo generado automáticamente para File5 |
| true |mergeFiles |Si la carpeta de origen Folder1 tiene esta estructura: <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>la carpeta de destino Folder1 se crea con la estructura siguiente: <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;El contenido de File1 + File2 + File3 + File4 + File 5 se combina en un archivo con un nombre de archivo generado automáticamente |
| false |preserveHierarchy |Si la carpeta de origen Folder1 tiene esta estructura: <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>la carpeta de destino Folder1 se crea con la estructura siguiente<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/><br/><br/>No se selecciona la subcarpeta Subfolder1, que contiene los archivos File3, File4 y File5. |
| false |flattenHierarchy |Si la carpeta de origen Folder1 tiene esta estructura:<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>la carpeta de destino Folder1 se crea con la estructura siguiente<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Nombre de archivo generado automáticamente para File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Nombre de archivo generado automáticamente para File2<br/><br/><br/>No se selecciona la subcarpeta Subfolder1, que contiene los archivos File3, File4 y File5. |
| false |mergeFiles |Si la carpeta de origen Folder1 tiene esta estructura:<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>la carpeta de destino Folder1 se crea con la estructura siguiente<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;El contenido de File1 + File2 se combina en un archivo con un nombre de archivo generado automáticamente. Nombre de archivo generado automáticamente para File1<br/><br/>No se selecciona la subcarpeta Subfolder1, que contiene los archivos File3, File4 y File5. |

## <a name="supported-file-and-compression-formats"></a>Formatos de archivo y de compresión admitidos
Para más información, consulte el artículo [Formatos de compresión de archivos admitidos por Azure Data Factory](data-factory-supported-file-and-compression-formats.md).

## <a name="json-examples-for-copying-data-to-and-from-data-lake-store"></a>Ejemplos de JSON para copiar datos hacia y desde Data Lake Store
En los ejemplos siguientes se proporcionan definiciones de JSON de ejemplo. Puede usar estas definiciones de ejemplo para crear una canalización mediante [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) o [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). En los ejemplos se muestra cómo copiar datos entre Azure Data Lake Store y Azure Blob Storage. En cambio, los datos pueden copiarse _directamente_ desde cualquiera de los orígenes a cualquiera de los receptores admitidos. Para más información, consulte la sección "Almacenes de datos y formatos que se admiten" del artículo [Movimiento de datos con la actividad de copia](data-factory-data-movement-activities.md).

### <a name="example-copy-data-from-azure-blob-storage-to-azure-data-lake-store"></a>Ejemplo: copia de datos desde Azure Blob Storage hacia Azure Data Lake Store
El código de ejemplo de esta sección muestra:

* Un servicio vinculado de tipo [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)
* Un servicio vinculado de tipo [AzureDataLakeStore](#linked-service-properties).
* Un [conjunto de datos](data-factory-create-datasets.md) de entrada de tipo [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
* Un [conjunto de datos](data-factory-create-datasets.md) de salida de tipo [AzureDataLakeStore](#dataset-properties).
* Una [canalización](data-factory-create-pipelines.md) con una actividad de copia que usa [BlobSource](data-factory-azure-blob-connector.md#copy-activity-properties) y [AzureDataLakeStoreSink](#copy-activity-properties).

En los ejemplos se muestra cómo los datos de serie temporal de Azure Blob Storage se copian en Data Lake Store cada hora.

**Servicio vinculado de Azure Storage**

```JSON
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```

**Servicio vinculado de Azure Data Lake Store**

```JSON
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "subscriptionId": "<subscription of ADLS>",
            "resourceGroupName": "<resource group of ADLS>"
        }
    }
}
```

> [!NOTE]
> Para información sobre los detalles de configuración, consulte la sección [Propiedades del servicio vinculado](#linked-service-properties).
>

**Conjunto de datos de entrada de blob de Azure:**

En el siguiente ejemplo, los datos se seleccionan de un nuevo blob cada hora (`"frequency": "Hour", "interval": 1`). La ruta de acceso de la carpeta y el nombre de archivo para el blob se evalúan dinámicamente según la hora de inicio del segmento que se está procesando. La ruta de acceso a la carpeta usa la parte de año, mes y día de la hora de inicio. El nombre de archivo usa la parte de hora de la hora de inicio. El valor `"external": true` informa al servicio Data Factory de que la tabla es externa a la factoría de datos y no la produce ninguna actividad de dicha factoría.

```JSON
{
  "name": "AzureBlobInput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}",
      "partitionedBy": [
        {
          "name": "Year",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "yyyy"
          }
        },
        {
          "name": "Month",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "MM"
          }
        },
        {
          "name": "Day",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "dd"
          }
        },
        {
          "name": "Hour",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "HH"
          }
        }
      ]
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      "externalData": {
        "retryInterval": "00:01:00",
        "retryTimeout": "00:10:00",
        "maximumRetry": 3
      }
    }
  }
}
```

**Conjunto de datos de salida de Azure Data Lake Store**

En el siguiente ejemplo se copian los datos en Data Lake Store. Los nuevos datos se copian en Data Lake Store cada hora.

```JSON
{
    "name": "AzureDataLakeStoreOutput",
    "properties": {
        "type": "AzureDataLakeStore",
        "linkedServiceName": "AzureDataLakeStoreLinkedService",
        "typeProperties": {
            "folderPath": "datalake/output/"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

**Actividad de copia en una canalización con un origen de blob y el receptor de Azure Data Lake Store**

En el ejemplo siguiente, la canalización contiene una actividad de copia que está configurada para usar los conjuntos de datos de entrada y de salida. La actividad de copia está programada para ejecutarse cada hora. En la definición de JSON de la canalización, el tipo `source` está establecido en `BlobSource` y el tipo `sink` está establecido en `AzureDataLakeStoreSink`.

```json
{
    "name":"SamplePipeline",
    "properties":
    {
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-01T19:00:00",
        "description":"pipeline with copy activity",
        "activities":
        [
            {
                "name": "AzureBlobtoDataLake",
                "description": "Copy Activity",
                "type": "Copy",
                "inputs": [
                    {
                        "name": "AzureBlobInput"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureDataLakeStoreOutput"
                    }
                ],
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                    },
                    "sink": {
                        "type": "AzureDataLakeStoreSink"
                    }
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "policy": {
                    "concurrency": 1,
                    "executionPriorityOrder": "OldestFirst",
                    "retry": 0,
                    "timeout": "01:00:00"
                }
            }
        ]
    }
}
```

### <a name="example-copy-data-from-azure-data-lake-store-to-an-azure-blob"></a>Ejemplo: copia de datos desde Azure Data Lake Store hacia un blob de Azure
El código de ejemplo de esta sección muestra:

* Un servicio vinculado de tipo [AzureDataLakeStore](#linked-service-properties).
* Un servicio vinculado de tipo [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)
* Un [conjunto de datos](data-factory-create-datasets.md) de entrada de tipo [AzureDataLakeStore](#dataset-properties).
* Un [conjunto de datos](data-factory-create-datasets.md) de salida de tipo [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
* Una [canalización](data-factory-create-pipelines.md) con una actividad de copia que usa [AzureDataLakeStoreSource](#copy-activity-properties) y [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties).

El código copia los datos de series temporales desde Data Lake Store hacia un blob de Azure cada hora.

**Servicio vinculado de Azure Data Lake Store**

```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>"
        }
    }
}
```

> [!NOTE]
> Para información sobre los detalles de configuración, consulte la sección [Propiedades del servicio vinculado](#linked-service-properties).
>

**Servicio vinculado de Azure Storage**

```JSON
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```
**Conjunto de datos de entrada de Azure Data Lake**

Si se establece `"external"` en `true`, se informa al servicio Data Factory de que la tabla es externa a la factoría de datos y no la produce ninguna actividad de dicha factoría.

```json
{
    "name": "AzureDataLakeStoreInput",
    "properties":
    {
        "type": "AzureDataLakeStore",
        "linkedServiceName": "AzureDataLakeStoreLinkedService",
        "typeProperties": {
            "folderPath": "datalake/input/",
            "fileName": "SearchLog.tsv",
            "format": {
                "type": "TextFormat",
                "rowDelimiter": "\n",
                "columnDelimiter": "\t"
            }
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```
**Conjunto de datos de salida de blob de Azure**

En el ejemplo siguiente, los datos se escriben en un nuevo blob cada hora (`"frequency": "Hour", "interval": 1`). La ruta de acceso de la carpeta para el blob se evalúa dinámicamente según la hora de inicio del segmento que se está procesando. La ruta de acceso de la carpeta usa las partes year, month, day y hours de la hora de inicio.

```JSON
{
  "name": "AzureBlobOutput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
      "partitionedBy": [
        {
          "name": "Year",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "yyyy"
          }
        },
        {
          "name": "Month",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "MM"
          }
        },
        {
          "name": "Day",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "dd"
          }
        },
        {
          "name": "Hour",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "HH"
          }
        }
      ],
      "format": {
        "type": "TextFormat",
        "columnDelimiter": "\t",
        "rowDelimiter": "\n"
      }
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```

**Una actividad de copia en una canalización con un origen de Azure Data Lake Store y un receptor de blob**

En el ejemplo siguiente, la canalización contiene una actividad de copia que está configurada para usar los conjuntos de datos de entrada y de salida. La actividad de copia está programada para ejecutarse cada hora. En la definición de JSON de la canalización, el tipo `source` está establecido en `AzureDataLakeStoreSource` y el tipo `sink` está establecido en `BlobSink`.

```json
{
    "name":"SamplePipeline",
    "properties":{
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-01T19:00:00",
        "description":"pipeline for copy activity",
        "activities":[
            {
                "name": "AzureDakeLaketoBlob",
                "description": "copy activity",
                "type": "Copy",
                "inputs": [
                    {
                        "name": "AzureDataLakeStoreInput"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureBlobOutput"
                    }
                ],
                "typeProperties": {
                    "source": {
                        "type": "AzureDataLakeStoreSource",
                    },
                    "sink": {
                        "type": "BlobSink"
                    }
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "policy": {
                    "concurrency": 1,
                    "executionPriorityOrder": "OldestFirst",
                    "retry": 0,
                    "timeout": "01:00:00"
                }
            }
        ]
    }
}
```

En la definición de la actividad de copia, también puede asignar columnas del conjunto de datos de origen a columnas del conjunto de datos receptor. Para más información, consulte [Mapping dataset columns in Azure Data Factory](data-factory-map-columns.md) (Asignación de columnas de conjunto de datos de Azure Data Factory).

## <a name="performance-and-tuning"></a>Rendimiento y optimización
Para conocer los factores que afectan al rendimiento de la actividad de copia y cómo optimizarla, consulte el artículo [Guía de optimización y rendimiento de la actividad de copia](data-factory-copy-activity-performance.md).
