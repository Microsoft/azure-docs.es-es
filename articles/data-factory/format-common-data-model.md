---
title: Formato Common Data Model
description: Transformación de datos mediante el sistema de metadatos de Common Data Model
author: djpmsft
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 08/05/2020
ms.author: daperlov
ms.openlocfilehash: 483e26cf4044b909c8d7923cfd74bd6fcf871e2a
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87905312"
---
# <a name="common-data-model-format-in-azure-data-factory"></a>Formato de Common Data Model en Azure Data Factory
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

El sistema de metadatos de Common Data Model (CDM) permite que los datos y su significado se compartan fácilmente entre las aplicaciones y los procesos empresariales. Para obtener más información, consulte la información general de [Common Data Model](https://docs.microsoft.com/common-data-model/).

En Azure Data Factory, los usuarios pueden transformar datos las entidades CDM tanto en model.json como en formato de manifiesto almacenadas en [Azure Data Lake Store Gen2](connector-azure-data-lake-storage.md) (ADLS Gen2) mediante flujo de datos de asignación. También puede recibir datos en formato CDM mediante referencias de entidad CDM que van a colocar los datos en formato CSV o Parquet en las carpetas con particiones. 

> [!NOTE]
> El conector de formato de Common Data Model (CDM) para flujos de datos de ADF está disponible actualmente como vista previa pública.

## <a name="mapping-data-flow-properties"></a>Propiedades de Asignación de instancias de Data Flow

Common Data Model está disponible como [conjunto de datos en línea](data-flow-source.md#inline-datasets) en el flujo de datos de asignación como origen y receptor.

> [!NOTE]
> Al escribir entidades CDM, debe tener una definición de entidad CDM existente (esquema de metadatos) ya definida para usarla como referencia. El receptor de flujo de datos de ADF leerá ese archivo de entidad CDM e importará el esquema en el receptor para la asignación de campos.

### <a name="source-properties"></a>Propiedades de origen

En la tabla siguiente se enumeran las propiedades que admite un origen CDM. Puede editar estas propiedades en la pestaña **Source options** (Opciones del origen).

| Nombre | Descripción | Obligatorio | Valores permitidos | Propiedad de script de flujo de datos |
| ---- | ----------- | -------- | -------------- | ---------------- |
| Formato | El formato debe ser `cdm`. | sí | `cdm` | format |
| Formato de metadatos | Donde se encuentra la referencia de entidad de los datos. Si utiliza la versión 1.0 de CDM, elija manifiesto. Si usa una versión de CDM anterior a la 1.0, elija model.json. | Sí | `'manifest'` o `'model'` | manifestType |
| Ubicación raíz: contenedor | Nombre del contenedor de la carpeta CDM | sí | String | fileSystem |
| Ubicación raíz: ruta de acceso de la carpeta | Ubicación de la carpeta raíz de la carpeta CDM | sí | String | folderPath |
| Archivo de manifiesto: Ruta de acceso de entidad | Ruta de acceso de la carpeta de la entidad dentro de la carpeta raíz | no | String | entityPath |
| Archivo de manifiesto: Nombre del manifiesto | Nombre del archivo de manifiesto. El valor predeterminado es "default"  | No | String | manifestName |
| Filtrar por última modificación | Elija si desea filtrar los archivos en función de cuándo se modificaron por última vez. | no | Timestamp | modifiedAfter <br> modifiedBefore | 
| Servicio vinculado de esquema | El servicio vinculado en el que se encuentra el corpus | Sí, si se utiliza el manifiesto | `'adlsgen2'` o `'github'` | corpusStore | 
| Contenedor de referencia de entidad | El contenedor corpus está en | sí, si utiliza el manifiesto y el Corpus en ADLS Gen2 | String | adlsgen2_fileSystem |
| Repositorio de referencia de entidad | Nombre del repositorio de GitHub | sí, si utiliza el manifiesto y el Corpus en GitHub | String | github_repository |
| Rama de referencia de entidad | Rama del repositorio de GitHub | sí, si utiliza el manifiesto y el corpus en GitHub | String |  github_branch |
| Carpeta Corpus | la ubicación raíz de Corpus | Sí, si se utiliza el manifiesto | String | corpusPath |
| Entidad Corpus | Ruta de acceso a la referencia de entidad | sí | String | Entidad |
| No permitir que se encuentren archivos | Si es true, no se devuelve un error si no se encuentra ningún archivo. | no | `true` o `false` | ignoreNoFilesFound |

### <a name="sink-settings"></a>Configuración del receptor

* Seleccione el archivo de referencia de la entidad CDM que contiene la definición de la entidad que quiere escribir.

![entity settings](media/data-flow/common-data-model-111.png "Referencia de entidad")

* Defina la ruta de acceso de la partición y el formato de los archivos de salida que desea que ADF use para escribir las entidades.

![entity format](media/data-flow/common-data-model-222.png "Formato de entidad")

* Establezca la ubicación del archivo de salida y la ubicación y el nombre del archivo de manifiesto.

![cdm location](media/data-flow/common-data-model-333.png "Ubicación CDM")


#### <a name="import-schema"></a>Importar esquema

CDM solo está disponible como un conjunto de datos insertado y, de forma predeterminada, no tiene un esquema asociado. Para obtener los metadatos de columna, haga clic en el botón **Importar esquema** en la pestaña **Proyección**. Esto le permitirá hacer referencia a los nombres de columna y los tipos de datos especificados por corpus. Para importar el esquema, una [sesión de depuración de flujo de datos](concepts-data-flow-debug-mode.md) debe estar activa y debe tener un archivo de definición de entidad CDM existente al que apuntar.

Al asignar columnas de flujo de datos a las propiedades de entidad en la transformación del receptor, haga clic en la pestaña "Asignación" y seleccione "Importar esquema". ADF leerá la referencia a la entidad a la que apunta en las opciones del receptor, permitiéndole asignar al esquema CDM de destino.

![Configuración del receptor CDM](media/data-flow/common-data-model-444.png "Asignación CDM")

> [!NOTE]
>  Cuando se usa el tipo de origen model.json que se origina en Power BI o en los flujos de datos de Power Platform, es posible que se produzcan errores de "la ruta de acceso de Corpus es null o está vacía" de la transformación de origen. Esto se debe probablemente a problemas de formato de la ruta de acceso de la partición en el archivo model.json. Para solucionarlo, siga estos pasos. 

1. Abra el archivo de model.json con un editor de texto
2. Busque las propiedades partitions.Location 
3. Cambie "blob.core.windows.net" a "dfs.core.windows.net"
4. Corrija cualquier codificación "% 2F" de la dirección URL a "/"
 

### <a name="cdm-source-data-flow-script-example"></a>Ejemplo de script de flujo de datos del origen de CDM

```
source(output(
        ProductSizeId as integer,
        ProductColor as integer,
        CustomerId as string,
        Note as string,
        LastModifiedDate as timestamp
    ),
    allowSchemaDrift: true,
    validateSchema: false,
    entity: 'Product.cdm.json/Product',
    format: 'cdm',
    manifestType: 'manifest',
    manifestName: 'ProductManifest',
    entityPath: 'Product',
    corpusPath: 'Products',
    corpusStore: 'adlsgen2',
    adlsgen2_fileSystem: 'models',
    folderPath: 'ProductData',
    fileSystem: 'data') ~> CDMSource
```

### <a name="sink-properties"></a>Propiedades del receptor

En la tabla siguiente se enumeran las propiedades que admite un receptor CDM. Puede editar estas propiedades en la pestaña **Configuración**.

| Nombre | Descripción | Obligatorio | Valores permitidos | Propiedad de script de flujo de datos |
| ---- | ----------- | -------- | -------------- | ---------------- |
| Formato | El formato debe ser `cdm`. | sí | `cdm` | format |
| Ubicación raíz: contenedor | Nombre del contenedor de la carpeta CDM | sí | String | fileSystem |
| Ubicación raíz: ruta de acceso de la carpeta | Ubicación de la carpeta raíz de la carpeta CDM | sí | String | folderPath |
| Archivo de manifiesto: Ruta de acceso de entidad | Ruta de acceso de la carpeta de la entidad dentro de la carpeta raíz | no | String | entityPath |
| Archivo de manifiesto: Nombre del manifiesto | Nombre del archivo de manifiesto. El valor predeterminado es "default" | No | String | manifestName |
| Servicio vinculado de esquema | El servicio vinculado en el que se encuentra el corpus | sí | `'adlsgen2'` o `'github'` | corpusStore | 
| Contenedor de referencia de entidad | El contenedor corpus está en | sí, si el corpus está en ADLS Gen2 | String | adlsgen2_fileSystem |
| Repositorio de referencia de entidad | Nombre del repositorio de GitHub | sí, si el corpus está en GitHub | String | github_repository |
| Rama de referencia de entidad | Rama del repositorio de GitHub | sí, si el corpus está en GitHub | String |  github_branch |
| Carpeta Corpus | la ubicación raíz del corpus | sí | String | corpusPath |
| Entidad Corpus | Ruta de acceso a la referencia de entidad | sí | String | Entidad |
| Ruta de acceso a la partición | Ubicación donde se escribirá la partición | no | String | partitionPath |
| Borrar la carpeta | Si la carpeta de destino se borra antes de escribir. | no | `true` o `false` | truncate |
| Tipo de formato | Elija esta opción para especificar el formato Parquet | no | `parquet` si se especifica | subformat |
| Delimitador de columna | Si se escribe en DelimitedText, cómo se delimitan las columnas | sí, si escribe en DelimitedText | String | columnDelimiter |
| Primera fila como encabezado | Si se utiliza DelimitedText, si los nombres de las columnas se agregan como un encabezamiento | no | `true` o `false` | columnNamesAsHeader |

### <a name="cdm-sink-data-flow-script-example"></a>Ejemplo de script de flujo de datos del receptor de CDM

El script de flujo de datos asociado es:

```
CDMSource sink(allowSchemaDrift: true,
    validateSchema: false,
    entity: 'Product.cdm.json/Product',
    format: 'cdm',
    entityPath: 'ProductSize',
    manifestName: 'ProductSizeManifest',
    corpusPath: 'Products',
    partitionPath: 'adf',
    folderPath: 'ProductSizeData',
    fileSystem: 'cdm',
    subformat: 'parquet',
    corpusStore: 'adlsgen2',
    adlsgen2_fileSystem: 'models',
    truncate: true,
    skipDuplicateMapInputs: true,
    skipDuplicateMapOutputs: true) ~> CDMSink

```

## <a name="next-steps"></a>Pasos siguientes

Cree una [transformación de origen](data-flow-source.md) en flujo de datos de asignación.
