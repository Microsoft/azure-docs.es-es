---
title: Script del flujo de datos de asignación
description: Información general sobre el lenguaje de código subyacente de un script de flujo de datos de Data Factory
author: kromerm
ms.author: nimoolen
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 07/29/2020
ms.openlocfilehash: d28cd7a7edd5d6405761bf21ee87ec39dc9ec9cb
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87448539"
---
# <a name="data-flow-script-dfs"></a>Script de flujo de datos (DFS)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

El script de flujo de datos (DFS) son los metadatos subyacentes, similares a un lenguaje de codificación, que se usan para ejecutar las transformaciones que se incluyen en un flujo de datos de asignación. Cada transformación se representa mediante una serie de propiedades que proporcionan la información necesaria para ejecutar el trabajo correctamente. El script se puede ver y editar en ADF haciendo clic en el botón "Script" de la cinta de opciones superior de la interfaz de usuario del explorador.

![Botón Script](media/data-flow/scriptbutton.png "Botón Script")

Por ejemplo, `allowSchemaDrift: true,` en una transformación de origen indica al servicio que incluya todas las columnas del conjunto de datos de origen en el flujo de datos, incluso si no están incluidas en la proyección del esquema.

## <a name="use-cases"></a>Casos de uso
El DFS lo genera automáticamente la interfaz de usuario. Puede hacer clic en el botón Script para ver y personalizar el script. También puede generar scripts fuera de la interfaz de usuario de ADF y después pasarlos al cmdlet de PowerShell. Al depurar flujos de datos complejos, es posible que le resulte más fácil examinar el código subyacente del script en lugar de examinar la representación del gráfico de la interfaz de usuario de los flujos.

Estos son algunos casos de uso de ejemplo:
- Generación mediante programación de muchos flujos de datos que sean bastante similares, es decir, "sellar" flujos de trabajo.
- Expresiones complejas que son difíciles de administrar en la interfaz de usuario o que producen problemas de validación.
- Depuración y mejor reconocimiento de distintos errores devueltos durante la ejecución.

Al compilar un script de flujo de datos para usarlo con PowerShell o una API, debe contraer el texto con formato a una sola línea. Puede mantener las tabulaciones y las líneas nuevas como caracteres de escape. Pero se debe dar formato al texto para que quepa en una propiedad JSON. Hay un botón en la interfaz de usuario del editor de scripts en la parte inferior que dará formato al script como una sola línea.

![Botón Copiar](media/data-flow/copybutton.png "botón Copiar")

## <a name="how-to-add-transforms"></a>Incorporación de transformaciones
La incorporación de transformaciones requiere tres pasos básicos: agregar los datos de la transformación principal, volver a enrutar el flujo de entrada y, después, volver a enrutar el flujo de salida. Esto puede verse más fácilmente en un ejemplo.
Supongamos que empezamos con un origen sencillo para recibir un flujo de datos como el siguiente:

```
source(output(
        movieId as string,
        title as string,
        genres as string
    ),
    allowSchemaDrift: true,
    validateSchema: false) ~> source1
source1 sink(allowSchemaDrift: true,
    validateSchema: false) ~> sink1
```

Si decidimos agregar una transformación derivada, primero es necesario crear el texto de transformación básico, que tiene una expresión simple para agregar una nueva columna en mayúsculas denominada `upperCaseTitle`:
```
derive(upperCaseTitle = upper(title)) ~> deriveTransformationName
```

A continuación, tomamos el DFS existente y agregamos la transformación:
```
source(output(
        movieId as string,
        title as string,
        genres as string
    ),
    allowSchemaDrift: true,
    validateSchema: false) ~> source1
derive(upperCaseTitle = upper(title)) ~> deriveTransformationName
source1 sink(allowSchemaDrift: true,
    validateSchema: false) ~> sink1
```

Y ahora volvemos a enrutar el flujo entrante al identificar qué transformación queremos que sea la nueva transformación que venga después (en este caso, `source1`) y copiar el nombre del flujo en la nueva transformación:
```
source(output(
        movieId as string,
        title as string,
        genres as string
    ),
    allowSchemaDrift: true,
    validateSchema: false) ~> source1
source1 derive(upperCaseTitle = upper(title)) ~> deriveTransformationName
source1 sink(allowSchemaDrift: true,
    validateSchema: false) ~> sink1
```

Por último, identificamos la transformación que deseamos que venga después de esta nueva transformación y reemplazamos su flujo de entrada (en este caso, `sink1`) por el nombre del flujo de salida de la nueva transformación:
```
source(output(
        movieId as string,
        title as string,
        genres as string
    ),
    allowSchemaDrift: true,
    validateSchema: false) ~> source1
source1 derive(upperCaseTitle = upper(title)) ~> deriveTransformationName
deriveTransformationName sink(allowSchemaDrift: true,
    validateSchema: false) ~> sink1
```

## <a name="dfs-fundamentals"></a>Aspectos básicos de DFS
El DFS se compone de una serie de transformaciones conectadas, que incluyen orígenes, receptores y varios usuarios que pueden agregar nuevas columnas, filtrar datos, combinar datos y mucho más. Normalmente, el script comienza con uno o más orígenes seguidos de muchas transformaciones y termina con uno o más receptores.

Todos los orígenes tienen la misma construcción básica:
```
source(
  source properties
) ~> source_name
```

Por ejemplo, un origen simple con tres columnas (movieId, title y genres) sería el siguiente:
```
source(output(
        movieId as string,
        title as string,
        genres as string
    ),
    allowSchemaDrift: true,
    validateSchema: false) ~> source1
```

Todas las transformaciones que no sean orígenes tienen la misma construcción básica:
```
name_of_incoming_stream transformation_type(
  properties
) ~> new_stream_name
```

Por ejemplo, una transformación simple derivada que toma una columna (title) y la sobrescribe con una versión en mayúsculas sería la siguiente:
```
source1 derive(
  title = upper(title)
) ~> derive1
```

Y un receptor sin esquema sería sencillamente:
```
derive1 sink(allowSchemaDrift: true,
    validateSchema: false) ~> sink1
```

## <a name="script-snippets"></a>Script de fragmentos de código

Los fragmentos de script son código compartible del script de flujo de datos que puede usar para compartir entre flujos de datos. En el siguiente vídeo se habla de cómo usar fragmentos de script y el uso del script de flujo de datos para copiar y pegar partes del script detrás de los gráficos de flujo de datos:

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4tA9b]


### <a name="aggregated-summary-stats"></a>Estadísticas de resumen agregadas
Añada una transformación agregado al flujo de datos denominada "SummaryStats" y, a continuación, pegue el siguiente código para la función de agregado en el script, reemplazando el SummaryStats existente. Esto proporcionará un patrón genérico para las estadísticas de resumen de perfil de datos.

```
aggregate(each(match(true()), $$+'_NotNull' = countIf(!isNull($$)), $$ + '_Null' = countIf(isNull($$))),
        each(match(type=='double'||type=='integer'||type=='short'||type=='decimal'), $$+'_stddev' = round(stddev($$),2), $$ + '_min' = min ($$), $$ + '_max' = max($$), $$ + '_average' = round(avg($$),2), $$ + '_variance' = round(variance($$),2)),
        each(match(type=='string'), $$+'_maxLength' = max(length($$)))) ~> SummaryStats
```
También puede usar el ejemplo siguiente para contar el número de filas únicas y el número de filas distintas de los datos. El ejemplo siguiente se puede pegar en un flujo de datos con la transformación agregada llamada ValueDistAgg. En este ejemplo se usa una columna denominada "title". Asegúrese de reemplazar "title" por la columna de cadena de los datos que desea usar para obtener los recuentos de valores.

```
aggregate(groupBy(title),
    countunique = count()) ~> ValueDistAgg
ValueDistAgg aggregate(numofunique = countIf(countunique==1),
        numofdistinct = countDistinct(title)) ~> UniqDist
```

### <a name="include-all-columns-in-an-aggregate"></a>Incluir todas las columnas en un agregado
Se trata de un patrón de agregado genérico que muestra cómo se pueden mantener las columnas restantes en los metadatos de salida al compilar agregados. En este caso, usamos la ```first()``` función para elegir el primer valor de cada columna cuyo nombre no sea "movie". Para usar esto, cree una transformación agregada llamada DistinctRows y, a continuación, péguela en el script que existe sobre el script agregado DistinctRows.

```
aggregate(groupBy(movie),
    each(match(name!='movie'), $$ = first($$))) ~> DistinctRows
```

### <a name="create-row-hash-fingerprint"></a>Crear huella digital en código hash 
Use este código en el script de flujo de datos para crear una nueva columna derivada denominada ```DWhash``` que genere una ```sha1``` hash de tres columnas.

```
derive(DWhash = sha1(Name,ProductNumber,Color))
```

También puede usar este script para generar un hash de filas con todas las columnas presentes en la secuencia, sin necesidad de asignar un nombre a cada columna:

```
derive(DWhash = sha1(columns()))
```

### <a name="string_agg-equivalent"></a>Equivalente de string_agg
Este código actuará como la función ```string_agg()``` de T-SQL y agregará valores de cadena a una matriz. Después, puede convertir esa matriz en una cadena para usarla con destinos SQL.

```
source1 aggregate(groupBy(year),
    string_agg = collect(title)) ~> Aggregate1
Aggregate1 derive(string_agg = toString(string_agg)) ~> DerivedColumn2
```

### <a name="count-number-of-updates-upserts-inserts-deletes"></a>Recuento del número de actualizaciones, upserts, inserciones y eliminaciones
Cuando se usa una transformación Alter Row, es posible que quiera contar el número de actualizaciones, upserts, inserciones y eliminaciones que resultan de las directivas Alter Row. Agregue una transformación Aggregate después de Alter Row y pegue este script de Data Flow en la definición de agregado para esos recuentos.

```
aggregate(updates = countIf(isUpdate(), 1),
        inserts = countIf(isInsert(), 1),
        upserts = countIf(isUpsert(), 1),
        deletes = countIf(isDelete(),1)) ~> RowCount
```

### <a name="distinct-row-using-all-columns"></a>Fila DISTINCT que utiliza todas las columnas
Este fragmento de código agregará una nueva transformación Aggregate al flujo de datos que recogerá todas las columnas de entrada, generará un hash que se usará para agrupar y eliminar duplicados y, a continuación, proporcionará el primer caso de cada duplicado como salida. No es necesario asignar un nombre explícito a las columnas, ya que se generarán automáticamente a partir del flujo de datos de entrada.

```
aggregate(groupBy(mycols = sha2(256,columns())),
    each(match(true()), $$ = first($$))) ~> DistinctRows
```

## <a name="next-steps"></a>Pasos siguientes

Explore los flujos de datos comenzando por el [artículo de información general sobre los flujos de datos](concepts-data-flow-overview.md)
