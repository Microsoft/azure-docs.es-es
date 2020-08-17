---
title: Transformación Dinamizar en el flujo de datos de asignación
description: Dinamice los datos de filas a columnas mediante la transformación Dinamizar del flujo de datos de asignación de Azure Data Factory
author: kromerm
ms.author: makromer
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 07/17/2020
ms.openlocfilehash: e098182c000cbe05df533434a41c55b797ef876f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87086683"
---
# <a name="pivot-transformation-in-mapping-data-flow"></a>Transformación Dinamizar en el flujo de datos de asignación


[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Use la transformación Dinamizar para crear varias columnas a partir de los valores únicos de fila de una sola columna. "Dinamizar" es una transformación de agregación en la que debe seleccionar las columnas Agrupar por y generar columnas dinámicas mediante [funciones de agregado](data-flow-expression-functions.md#aggregate-functions).

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4C4YN]

## <a name="configuration"></a>Configuración

La transformación Dinamizar requiere tres entradas diferentes: las columnas Agrupar por, la clave dinámica y cómo generar las columnas dinamizadas.

### <a name="group-by"></a>Agrupar por

![Opciones de Agrupar por](media/data-flow/pivot2.png "Opciones de Agrupar por")

Seleccione las columnas sobre las que se deben agregar las columnas dinamizadas. Los datos de salida agruparán todas las filas con los mismos valores Agrupar por en una fila. La agregación realizada en la columna dinamizada se realizará en cada grupo.

Esta sección es opcional. Si no se selecciona ninguna columna Agrupar por, se agregará todo el flujo de datos y solo se generará una fila.

### <a name="pivot-key"></a>Clave dinámica

![Clave dinámica](media/data-flow/pivot3.png "Clave dinámica")

La clave dinámica es la columna cuyos valores de fila se dinamizan en nuevas columnas. De forma predeterminada, la transformación dinámica creará una nueva columna para cada valor de fila único.

En la sección con la etiqueta **Valor**, puede especificar valores de fila específicos para dinamizarlos. Solo se dinamizarán los valores de fila especificados en esta sección. Al habilitar la opción **Valor nulo**, se creará una columna dinamizada para los valores nulos de la columna.

### <a name="pivoted-columns"></a>Columnas dinamizadas

![Columnas dinamizadas](media/data-flow/pivot4.png "Columnas dinamizadas")

Para cada valor de clave dinámica único que se convierta en una columna, genere un valor de fila agregado para cada grupo. Puede crear varias columnas por cada clave dinámica. Cada columna dinámica debe contener al menos una [función de agregado](data-flow-expression-functions.md#aggregate-functions).

**Patrón de nombre de columna:** seleccione cómo dar formato al nombre de la columna de cada columna dinámica. El nombre de la columna de salida será una combinación del valor de la clave dinámica, el prefijo de la columna y los caracteres opcionales de prefijo, de sufijo e intermedios. 

**Organización de columnas:** si genera más de una columna dinámica por cada clave dinámica, elija cómo quiere que se ordenen las columnas. 

**Prefijo de columna:** si genera más de una columna dinámica por cada clave dinámica, escriba un prefijo de columna para cada columna. Esta configuración es opcional si solo tiene una columna dinamizada.

## <a name="help-graphic"></a>Gráfico de ayuda

En el gráfico de ayuda siguiente se muestra cómo interactúan entre sí los distintos componentes dinámicos.

![Gráficos de ayuda de los componentes dinámicos](media/data-flow/pivot5.png "Gráficos de ayuda de los componentes dinámicos")

## <a name="pivot-metadata"></a>Metadatos dinámicos

Si no se especifica ningún valor en la configuración de la clave dinámica, las columnas dinamizadas se generarán dinámicamente en tiempo de ejecución. El número de columnas dinamizadas será igual al número de valores de clave dinámica únicos multiplicado por el número de columnas dinámicas. Dado que este número puede ser cambiante, la experiencia de usuario no mostrará los metadatos de columna en la pestaña **Inspeccionar** y no habrá ninguna propagación de columna. Para transformar estas columnas, use las funcionalidades de los [patrones de columna](concepts-data-flow-column-pattern.md) del flujo de datos de asignación. 

Si se establecen valores específicos de clave dinámica, las columnas dinamizadas aparecerán en los metadatos. Los nombres de columna estarán a su disposición en la asignación de la inspección y el receptor.

### <a name="generate-metadata-from-drifted-columns"></a>Generación de metadatos a partir de columnas desfasadas

Dinamizar genera nuevos nombres de columna de forma dinámica en función de los valores de fila. Puedes agregar estas nuevas columnas en los metadatos a los que se puede hacer referencia posteriormente en el flujo de datos. Para ello, use la acción rápida [Map drifted](concepts-data-flow-schema-drift.md#map-drifted-columns-quick-action) (Asignar desfasadas) de la vista previa de los datos. 

![Columnas dinámicas](media/data-flow/newpivot1.png "Asignar columnas dinámicas desplazadas")

### <a name="sinking-pivoted-columns"></a>Recepción de columnas dinamizadas

Aunque las columnas dinamizadas son dinámicas, también se pueden escribir en el almacén de datos de destino. Habilite **Permitir el desfase de esquema** en la configuración del receptor. De este modo, podrá escribir columnas que no estén incluidas en los metadatos. No verá los nombres dinámicos nuevos en los metadatos de la columna, pero la opción de desviación del esquema le permitirá el aterrizaje de los datos.

### <a name="rejoin-original-fields"></a>Recombinación de los campos originales

La transformación dinámica solo proyectará las columnas Agrupar por y las dinamizadas. Si quiere que los datos de salida incluyan otras columnas de entrada, use un patrón de [autocombinación](data-flow-join.md#self-join).

## <a name="data-flow-script"></a>Script de flujo de datos

### <a name="syntax"></a>Sintaxis

```
<incomingStreamName>
    pivot(groupBy(Tm),
        pivotBy(<pivotKeyColumn, [<specifiedColumnName1>,...,<specifiedColumnNameN>]),
        <pivotColumnPrefix> = <pivotedColumnValue>,
        columnNaming: '< prefix >< $N | $V ><middle >< $N | $V >< suffix >',
        lateral: { 'true' | 'false'}
    ) ~> <pivotTransformationName
```
### <a name="example"></a>Ejemplo

Las pantallas que aparecen en la sección de configuración tienen el siguiente script de flujo de datos:

```
BasketballPlayerStats pivot(groupBy(Tm),
    pivotBy(Pos),
    {} = count(),
    columnNaming: '$V$N count',
    lateral: true) ~> PivotExample

```

## <a name="next-steps"></a>Pasos siguientes

Pruebe la [transformación Anular dinamización](data-flow-unpivot.md) para convertir los valores de columna en los valores de fila. 
