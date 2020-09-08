---
title: Transformación de origen en flujo de datos de asignación
description: Aprenda a configurar una transformación de origen en Mapping Data Flow.
author: kromerm
ms.author: makromer
manager: anandsub
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 08/18/2020
ms.openlocfilehash: 0f6b1789a5148af37ddd2d0895cec348c27f663a
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89182503"
---
# <a name="source-transformation-in-mapping-data-flow"></a>Transformación de origen en flujo de datos de asignación 

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Una transformación de origen configura el origen de datos para el flujo de datos. Al diseñar flujos de datos, el primer paso será siempre configurar una transformación de origen. Para agregar un origen, haga clic en el cuadro **Agregar origen** en el lienzo de Data Flow.

Cada flujo de datos requiere al menos una transformación de origen, pero puede agregar tantos orígenes como sea necesario para completar las transformaciones de datos. Puede combinar esos orígenes con una transformación de combinación, búsqueda o unión.

Cada transformación de origen se asocia exactamente con un conjunto de datos o servicio vinculado. El conjunto de datos define la forma y la ubicación de los datos que quiere escribir o leer. Si va a utilizar un conjunto de datos basado en archivos, puede usar caracteres comodín y listas de archivos en el origen para trabajar con más de un archivo a la vez.

## <a name="inline-datasets"></a>Conjuntos de datos en línea

La primera decisión que se toma al crear una transformación de origen es si la información de origen se define dentro de un objeto de conjunto de datos o dentro de la transformación de origen. La mayoría de los formatos solo están disponibles en una opción o en la otra. Consulte el documento adecuado del conector para obtener información sobre cómo usar un conector específico.

Cuando un formato se admita tanto en la opción en línea como en un objeto de conjunto de datos, existen ventajas para ambos. Los objetos de conjunto de datos son entidades reutilizables que se pueden aprovechar en otros flujos de datos y actividades, como en la copia. Son especialmente útiles cuando se usa un esquema protegido. Los conjuntos de datos no se basan en Spark y, en ocasiones, es posible que necesite reemplazar determinados valores o la proyección del esquema en la transformación de origen.

Se recomiendan los conjuntos de datos en línea cuando se usan esquemas flexibles, instancias de origen único u orígenes con parámetros. Si el origen contiene muchos parámetros, los conjuntos de datos en línea permiten no crear un objeto "ficticio". Los conjuntos de datos en línea se basan en Spark y sus propiedades son nativas para el flujo de datos.

Para usar un conjunto de datos en línea, seleccione el formato deseado en el selector **Tipo de origen**. En lugar de seleccionar un conjunto de datos de origen, seleccione el servicio vinculado al que desea conectarse.

![Conjunto de datos en línea](media/data-flow/inline-selector.png "Conjunto de datos en línea")

##  <a name="supported-source-types"></a><a name="supported-sources"></a> Tipos de orígenes admitidos

Mapping Data Flow sigue un enfoque de extracción, carga y transformación (ELT) y funciona con conjuntos de datos de un *almacenamiento provisional* que están todos en Azure. Actualmente, se pueden usar los siguientes conjuntos de datos en una transformación de origen:

| Conector | Formato | Conjunto de datos/en línea |
| --------- | ------ | -------------- |
| [Azure Blob Storage](connector-azure-blob-storage.md#mapping-data-flow-properties) | [Avro](format-avro.md#mapping-data-flow-properties)<br>[Texto delimitado](format-delimited-text.md#mapping-data-flow-properties)<br>[Delta (versión preliminar)](format-delta.md)<br>[Excel](format-excel.md#mapping-data-flow-properties)<br>[JSON](format-json.md#mapping-data-flow-properties) <br>[Parquet](format-parquet.md#mapping-data-flow-properties)<br>[XML](format-xml.md#mapping-data-flow-properties) | ✓/-<br>✓/-<br>-/✓<br>✓/✓<br/>✓/-<br>✓/-<br>✓/✓ |
| [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md#mapping-data-flow-properties) | [Avro](format-avro.md#mapping-data-flow-properties)<br>[Texto delimitado](format-delimited-text.md#mapping-data-flow-properties)<br>[Excel](format-excel.md#mapping-data-flow-properties)<br>[JSON](format-json.md#mapping-data-flow-properties)<br>[Parquet](format-parquet.md#mapping-data-flow-properties)<br>[XML](format-xml.md#mapping-data-flow-properties)  | ✓/-<br>✓/-<br>✓/✓<br/>✓/-<br>✓/-<br>✓/✓ |
| [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#mapping-data-flow-properties) | [Avro](format-avro.md#mapping-data-flow-properties)<br>[Common Data Model (versión preliminar)](format-common-data-model.md#source-properties)<br>[Texto delimitado](format-delimited-text.md#mapping-data-flow-properties)<br>[Delta (versión preliminar)](format-delta.md)<br>[Excel](format-excel.md#mapping-data-flow-properties)<br>[JSON](format-json.md#mapping-data-flow-properties)<br>[Parquet](format-parquet.md#mapping-data-flow-properties)<br>[XML](format-xml.md#mapping-data-flow-properties) | ✓/-<br/>-/✓<br>✓/-<br>-/✓<br>✓/✓<br>✓/-<br/>✓/-<br>✓/✓ |
| [Azure Synapse Analytics](connector-azure-sql-data-warehouse.md#mapping-data-flow-properties) | | ✓/- |
| [Azure SQL Database](connector-azure-sql-database.md#mapping-data-flow-properties) | | ✓/- |
| [Azure Cosmos DB (API de SQL)](connector-azure-cosmos-db.md#mapping-data-flow-properties) | | ✓/- |
| [Snowflake](connector-snowflake.md) | | ✓/✓ |

La configuración específica de estos conectores se encuentra en la pestaña **Source options** (Opciones de origen). La información y algunos ejemplos de script de flujo de datos sobre esta configuración se encuentran en la documentación del conector. 

Azure Data Factory tiene acceso a más de [90 conectores nativos](connector-overview.md). Para incluir datos de esos otros orígenes en el flujo de datos, use la herramienta de actividad de copia para cargar esos datos en una de las áreas de almacenamiento provisional compatibles.

## <a name="source-settings"></a>Configuración de origen

Una vez que haya agregado un origen, configúrelo mediante la pestaña **Configuración de origen**. Aquí puede elegir o crear el conjunto de datos al que apunta el origen. También puede seleccionar las opciones de muestreo y esquema para los datos.

![Pestaña de configuración de origen](media/data-flow/source1.png "Pestaña de configuración de origen")

**Nombre de la secuencia de salida:** El nombre de la transformación de origen.

**Tipo de origen:** Elija si desea utilizar un conjunto de datos en línea o un objeto de conjunto de datos existente.

**Probar conexión:** pruebe si el servicio Spark del flujo de datos puede conectarse correctamente al servicio vinculado que se usa en el conjunto de datos de origen. El modo de depuración debe estar activado para habilitar esta característica.

**Desfase de esquema:** El [desfase de esquema](concepts-data-flow-schema-drift.md) es la capacidad de Data Factory de administrar de forma nativa los esquemas flexibles en los flujos de datos sin necesidad de definir explícitamente los cambios en las columnas.

* Si las columnas de origen van a cambiar con frecuencia, seleccione la casilla **Allow schema drift** (Permitir el desfase de esquema). Esta opción permite que todos los campos de origen entrantes fluyan hasta el receptor a través de las transformaciones.

* Al elegir **Infer drifted column types** (Inferir tipos de columnas desfasadas), se indica a Data Factory que detecte y defina los tipos de datos para cada nueva columna detectada. Con esta característica desactivada, todas las columnas desfasadas serán del tipo cadena.

**Validate schema:** (Validar esquema) Si Validar esquema está seleccionado, el flujo de datos no se podrá ejecutar si los datos de origen entrantes no coinciden con el esquema definido del conjunto de datos.

**Skip line count:** (Número de líneas para omitir) Este campo especifica el número de líneas que se van a omitir al principio del conjunto de datos.

**Muestreo:** Habilite el muestreo para limitar el número de filas del origen. Use esta configuración al probar o muestrear datos del origen con fines de depuración.

Para validar si el origen está configurado correctamente, active el modo de depuración y capture una vista previa de los datos. Para más información, consulte [Modo de depuración](concepts-data-flow-debug-mode.md).

> [!NOTE]
> Cuando se activa el modo de depuración, el valor del límite de filas de la configuración de depuración sobrescribirá el valor de muestreo en el origen durante la vista previa de los datos.

## <a name="source-options"></a>Opciones de origen

La pestaña de opciones de origen contiene la configuración específica del conector y el formato elegidos. Para obtener más información y ejemplos, consulte la [documentación pertinente del conector](#supported-sources).

## <a name="projection"></a>Proyección

Al igual que los esquemas en los conjuntos de datos, la proyección de un origen define las columnas, los tipos y los formatos de datos de los datos de origen. Para la mayoría de los tipos de conjuntos de datos, como SQL y Parquet, la proyección en un origen se corrige para que refleje el esquema definido en un conjunto de datos. Cuando los archivos de origen no están fuertemente tipados (por ejemplo, archivos sin formato en lugar de archivos Parquet), puede definir los tipos de datos de cada campo en la transformación de origen.

![Configuración de la pestaña de proyección](media/data-flow/source3.png "Proyección")

Si el archivo de texto no tiene ningún esquema definido, seleccione **Detectar tipo de datos** para que Data Factory muestree e infiera los tipos de datos. Seleccione **Definir formato predeterminado** para detectar automáticamente los formatos de datos predeterminados.

**Reset schema** (Restablecer esquema) restablece la proyección a la definición del conjunto de datos de referencia.

Puede modificar los tipos de datos de columna en una transformación de columna derivada de un nivel inferior. Use una transformación de selección para modificar los nombres de columna.

### <a name="import-schema"></a>Importar esquema

El botón **Import Schema** (Importar esquema) de la pestaña **Proyección** permite usar un clúster de depuración activo para crear una proyección de esquema. Disponible en cada tipo de origen, la importación del esquema aquí invalidará la proyección definida en el conjunto de datos. El objeto del conjunto de datos no se cambiará.

Resulta útil en los conjuntos de datos como Avro y Azure Cosmos DB, que admiten estructuras de datos complejas y no requieren la existencia de definiciones de esquemas en el conjunto de datos. Para los conjuntos de datos en línea, esta es la única manera de hacer referencia a los metadatos de columna sin desfase de esquema.

## <a name="optimize-the-source-transformation"></a>Optimización de la transformación de origen

La pestaña **Optimizar** permite la edición de la información de partición en cada paso de la transformación. En la mayoría de los casos, **Use current partitioning** (Usar la creación de particiones actual) optimizará la estructura de la creación de particiones ideal para un origen.

Si va a leer un origen de Azure SQL Database, la creación de particiones de **Origen** personalizada probablemente leerá los datos con mayor rapidez. ADF leerá las consultas grandes realizando conexiones a la base de datos en paralelo. Esta creación de particiones de origen se puede realizar en una columna o mediante una consulta.

![Configuración de la partición de origen](media/data-flow/sourcepart3.png "creación de particiones")

Para más información sobre la optimización en Mapping Data Flow, consulte la [pestaña de optimización](concepts-data-flow-overview.md#optimize).

## <a name="next-steps"></a>Pasos siguientes

Comience a compilar el flujo de datos con una [transformación de columna derivada](data-flow-derived-column.md) y una [transformación de selección](data-flow-select.md).
