---
title: Flujos de datos de limpieza y transformación en Azure Data Factory
description: Introducción a los flujos de datos de limpieza y transformación en Azure Data Factory
author: djpmsft
ms.author: daperlov
ms.reviewer: gamal
ms.service: data-factory
ms.topic: conceptual
ms.date: 11/01/2019
ms.openlocfilehash: 5d0393ab5f2582efa2a4b727543bc799cdb1143e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87082875"
---
# <a name="what-are-wrangling-data-flows"></a>¿Qué son los flujos de datos de limpieza y transformación?

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]


Las organizaciones deben encargarse de la preparación, y la limpieza y transformación de datos para un análisis preciso de los datos complejos que crecen cada día más. La preparación de datos es necesaria para que las organizaciones puedan usar los datos en diversos procesos empresariales y reducir el plazo de amortización.

Los flujos de datos de limpieza y transformación en Azure Data Factory permiten la preparación de datos sin código en la escala de nube de forma iterativa. Los flujos de datos de limpieza y transformación se integran en [Power Query Online](https://docs.microsoft.com/power-query/) y ponen las funciones de Power Query M a disposición de los usuarios de Data Factory.

El flujo de datos de limpieza y transformación traduce el código M generado por el editor de mashups de Power Query Online en el código de Spark para la ejecución a escala de nube.

Los flujos de datos de limpieza y transformación son especialmente útiles para los ingenieros de datos o "integradores de datos de la ciudadanía".

> [!NOTE]
> El flujo de datos de limpieza y transformación está actualmente disponible en versión preliminar pública.

## <a name="use-cases"></a>Casos de uso

### <a name="fast-interactive-data-exploration-and-preparation"></a>Exploración y preparación rápidas de datos interactivos

Varios ingenieros de datos e integradores de datos de ciudadanos pueden explorar y preparar interactivamente conjuntos de datos a escala de nube. Con el aumento del volumen, la variedad y la velocidad de los datos en los lagos de datos, los usuarios necesitan una manera eficaz de explorar y preparar los conjuntos de datos. Por ejemplo, puede que necesite crear un conjunto de datos que "tenga toda la información demográfica de los clientes para los nuevos clientes desde 2017". No está asignando a un destino conocido. Está explorando, limpiando y transformando, y preparando los conjuntos de datos para que cumplan un requisito antes de publicarlos en el lago. Los flujos de datos de limpieza y transformación se usan a menudo para escenarios de análisis menos formales. Los conjuntos de datos preparados se pueden usar para realizar transformaciones y operaciones de aprendizaje automático de nivel inferior.

### <a name="code-free-agile-data-preparation"></a>Preparación de datos ágil sin código

Los integradores de datos de los ciudadanos invierten más del 60 % de su tiempo en buscar y preparar los datos. Buscan hacerlo sin código para mejorar la productividad operativa. Permitir a los integradores de datos de ciudadanos enriquecer, dar forma y publicar datos mediante herramientas conocidas, como Power Query Online, de forma escalable mejora drásticamente su productividad. El flujo de datos de limpieza y transformación en Azure Data Factory permite que el editor de mashup de Power Query Online conocido permita a los integradores de datos de ciudadanos corregir errores rápidamente, estandarizar datos y generar datos de alta calidad para permitir las decisiones empresariales.

### <a name="data-validation"></a>Validación de datos

Analice visualmente los datos sin código para quitar los valores atípicos y las anomalías, y hacer que cumplan con una forma para el análisis rápido.

## <a name="supported-sources"></a>Orígenes compatibles

| Conector | Formato de datos | Tipo de autenticación |
| -- | -- | --|
| [Azure Blob Storage](connector-azure-blob-storage.md) | CSV, Parquet | Clave de cuenta |
| [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md) | CSV | Entidad de servicio |
| [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md) | CSV, Parquet | Clave de cuenta, Entidad de servicio |
| [Azure SQL Database](connector-azure-sql-database.md) | - | Autenticación SQL |
| [Azure Synapse Analytics](connector-azure-sql-data-warehouse.md) | - | Autenticación SQL |

## <a name="the-mashup-editor"></a>Editor de mashup

Cuando se crea un flujo de datos de limpieza y transformación, todos los conjuntos de datos de origen se convierten en consultas de conjuntos de datos y se colocan en la carpeta **ADFResource**. De forma predeterminada, UserQuery apuntará a la primera consulta del conjunto de datos. Todas las transformaciones deben realizarse en UserQuery, ya que no se admiten ni se conservan los cambios en las consultas del conjunto de datos. Actualmente no se admiten las opciones para cambiar el nombre o agregar y eliminar consultas.

![Limpieza y transformación](media/wrangling-data-flow/editor.png)

Actualmente no se admiten todas las funciones de Power Query M para la limpieza y transformación de datos, a pesar de estar disponibles durante la creación. Al compilar los flujos de datos de limpieza y transformación, aparecerá el siguiente mensaje de error si no se admite una función:

`The wrangling data flow is invalid. Expression.Error: The transformation logic isn't supported. Please try a simpler expression`

Para obtener más información sobre las transformaciones admitidas, consulte [Funciones del flujo de datos de limpieza y transformación](wrangling-data-flow-functions.md).

Actualmente, el flujo de datos de limpieza y transformación solo admite la escritura en un receptor.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo [crear un flujo de datos de limpieza y transformación](wrangling-data-flow-tutorial.md).
