---
title: Transformación de receptor en el flujo de datos de asignación
description: Aprenda a configurar una transformación de receptor en el flujo de datos de asignación.
author: kromerm
ms.author: makromer
ms.reviewer: daperlov
manager: anandsub
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 08/18/2020
ms.openlocfilehash: b2a356f8ee6c8e3d11548a3c7c17a868e4c057c4
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89179188"
---
# <a name="sink-transformation-in-mapping-data-flow"></a>Transformación de receptor en el flujo de datos de asignación

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cuando termine de transformar los datos, escríbalos en un almacén de destino mediante la transformación del receptor. Cada flujo de datos requiere al menos una transformación de receptor, pero puede agregar tantos receptores como sea necesario para completar el flujo de transformación. Para escribir en receptores adicionales, cree nuevas secuencias a través de nuevas ramas y divisiones condicionales.

Cada transformación del receptor se asocia exactamente con un servicio vinculado o un objeto de conjunto de objetos de Azure Data Factory. La transformación del receptor determina la forma y la ubicación de los datos en los que se desea escribir.

## <a name="inline-datasets"></a>Conjuntos de datos insertado

Al crear una transformación de receptor, elija si la información del receptor se define dentro de un objeto de conjunto de datos o en la transformación. La mayoría de los formatos solo están disponibles en una opción o en la otra. Consulte el documento adecuado del conector para obtener información sobre cómo usar un conector específico.

Cuando un formato se admite tanto en la opción insertada como en un objeto de conjunto de datos, supone ventajas para ambos. Los objetos de conjunto de datos son entidades reutilizables que se pueden aprovechar en otros flujos de datos y actividades, como en la copia. Resultan especialmente útiles cuando se usa un esquema protegido. Los conjuntos de datos no se basan en Spark y, en ocasiones, es posible que tenga que reemplazar determinados valores o la proyección del esquema en la transformación de receptor.

Se recomiendan los conjuntos de datos insertados cuando se usan esquemas flexibles, instancias de origen único u receptores con parámetros. Si el receptor contiene muchos parámetros, los conjuntos de datos insertados permiten no crear un objeto "ficticio". Los conjuntos de datos insertados se basan en Spark y sus propiedades son nativas para el flujo de datos.

Para usar un conjunto de datos insertado, seleccione el formato deseado en el selector **Tipo de receptor**. En lugar de seleccionar un conjunto de datos de receptor, seleccione el servicio vinculado al que desee conectarse.

![Conjunto de datos insertado](media/data-flow/inline-selector.png "Conjunto de datos insertado")

##  <a name="supported-sink-types"></a><a name="supported-sinks"></a> Tipos de receptores admitidos

Mapping Data Flow sigue un enfoque de extracción, carga y transformación (ELT) y funciona con conjuntos de datos de un *almacenamiento provisional* que están todos en Azure. Actualmente, se pueden usar los siguientes conjuntos de datos en una transformación de origen:

| Conector | Formato | Conjunto de datos/insertado |
| --------- | ------ | -------------- |
| [Azure Blob Storage](connector-azure-blob-storage.md#mapping-data-flow-properties) | [JSON](format-json.md#mapping-data-flow-properties) <br> [Avro](format-avro.md#mapping-data-flow-properties) <br> [Texto delimitado](format-delimited-text.md#mapping-data-flow-properties) <br> [Delta (versión preliminar)](format-delta.md) <br> [Parquet](format-parquet.md#mapping-data-flow-properties) | ✓/- <br> ✓/- <br> ✓/- <br> -/✓ <br> ✓/- |
| [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md#mapping-data-flow-properties) | [JSON](format-json.md#mapping-data-flow-properties) <br> [Avro](format-avro.md#mapping-data-flow-properties) <br> [Texto delimitado](format-delimited-text.md#mapping-data-flow-properties) <br> [Parquet](format-parquet.md#mapping-data-flow-properties)  | ✓/- <br> ✓/- <br> ✓/- <br> ✓/- |
| [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#mapping-data-flow-properties) | [JSON](format-json.md#mapping-data-flow-properties) <br> [Avro](format-avro.md#mapping-data-flow-properties) <br> [Texto delimitado](format-delimited-text.md#mapping-data-flow-properties) <br> [Delta (versión preliminar)](format-delta.md) <br> [Parquet](format-parquet.md#mapping-data-flow-properties)  <br> [Common Data Model (versión preliminar)](format-common-data-model.md#sink-properties) | ✓/- <br> ✓/- <br> ✓/- <br> -/✓ <br> ✓/- <br> -/✓ |
| [Azure Synapse Analytics](connector-azure-sql-data-warehouse.md#mapping-data-flow-properties) | | ✓/- |
| [Azure SQL Database](connector-azure-sql-database.md#mapping-data-flow-properties) | | ✓/- |
| [Azure Cosmos DB (SQL API)](connector-azure-cosmos-db.md#mapping-data-flow-properties) | | ✓/- |
| [Snowflake](connector-snowflake.md) | | ✓/✓ |

La configuración específica de estos conectores se encuentra en la pestaña **Configuración**. La información y algunos ejemplos de script de flujo de datos sobre esta configuración se encuentran en la documentación del conector. 

Azure Data Factory tiene acceso a más de [90 conectores nativos](connector-overview.md). Para escribir datos en esos otros orígenes desde el flujo de datos, use la actividad de copia para cargar los datos desde un receptor compatible.

## <a name="sink-settings"></a>Configuración del receptor

Una vez que haya agregado un receptor, configúrelo a través de la pestaña **Receptor**. Aquí puede elegir o crear el conjunto de datos en el que escribe el receptor. A continuación, se muestra un vídeo en el que se explican varias opciones de receptor diferentes para los tipos de archivo delimitados de texto:

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4tf7T]

![Configuración del receptor](media/data-flow/sink-settings.png "Configuración del receptor")

**Desfase de esquema:** El [desfase de esquema](concepts-data-flow-schema-drift.md) es la capacidad de Data Factory de administrar de forma nativa los esquemas flexibles en los flujos de datos sin necesidad de definir explícitamente los cambios en las columnas. Habilite **Permitir el desfase de esquema** para escribir columnas adicionales sobre lo que se define en el esquema de datos del receptor.

**Validate schema:** (Validar esquema) Si se selecciona que se valide el esquema, se producirá un error en el flujo de datos si no se encuentra ninguna columna del esquema de origen entrante en la proyección de origen o si los tipos de datos no coinciden. Use esta opción para exigir que los datos de origen cumplan el contrato de la proyección definida. Es muy útil en escenarios de origen de base de datos para indicar que los nombres o los tipos de columna han cambiado.

## <a name="field-mapping"></a>Asignación de campos

De forma similar a la transformación Selección, en la pestaña **Asignación** del receptor, puede decidir qué columnas de entrada se escribirán. De forma predeterminada, se asignan todas las columnas de entrada, incluidas las columnas desfasadas. Esto se conoce como **registro automático**.

Cuando desactive la asignación automática, tendrá la opción de agregar asignaciones basadas en columnas o basadas en reglas. Las asignaciones basada en reglas le permiten escribir expresiones con coincidencia de patrones, mientras que la asignación fija asignará nombres de columna lógicos y físicos. Para más información sobre la asignación basada en reglas, consulte [Patrones de columna en flujos de datos de asignación](concepts-data-flow-column-pattern.md#rule-based-mapping-in-select-and-sink).

## <a name="custom-sink-ordering"></a>Ordenación de receptores personalizados

De forma predeterminada, los datos se escriben en varios receptores en un orden no determinista. El motor de ejecución escribirá datos en paralelo a medida que se complete la lógica de transformación y el orden de los receptores puede variar en cada ejecución. Para especificar una ordenación de receptores exacta, habilite la opción **Custom sink ordering** (Ordenación de receptores personalizada) en la pestaña General del flujo de datos. Una vez habilitada, los receptores se escribirán secuencialmente en orden ascendente.

![Ordenación de receptores personalizada](media/data-flow/custom-sink-ordering.png "Ordenación de receptores personalizados")

## <a name="data-preview-in-sink"></a>Vista previa de los datos en el receptor

Al obtener una vista previa de los datos en un clúster de depuración, no se escribirá ningún dato en el receptor. Se devolverá una instantánea de la apariencia de los datos, pero no se escribirá nada en el destino. Para probar la escritura de datos en el receptor, ejecute una depuración de canalización desde el lienzo de canalización.

## <a name="next-steps"></a>Pasos siguientes
Ahora que ha creado el flujo de datos, agregue una [actividad de Data Flow a la canalización](concepts-data-flow-overview.md).
