---
title: Incorporación de la fuente de distribución de datos a Metrics Advisor
titleSuffix: Azure Cognitive Services
description: En este artículo se explica cómo incorporar fuentes de distribución de datos a Metrics Advisor.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: conceptual
ms.date: 09/14/2020
ms.author: aahi
ms.openlocfilehash: 83ff710804b43837657ea0da7c8f44c245017c7e
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932411"
---
# <a name="how-to-onboard-your-metric-data-to-metrics-advisor"></a>Procedimiento: Incorporación de datos de métricas a Metrics Advisor

Consulte este artículo para saber cómo incorporar datos a Metrics Advisor. 

## <a name="data-schema-requirements-and-configuration"></a>Configuración y requisitos del esquema de datos

[!INCLUDE [data schema requirements](../includes/data-schema-requirements.md)]

## <a name="avoid-loading-partial-data"></a>Evitar la carga de datos parciales

Los datos parciales se deben a incoherencias entre los datos almacenados en Metrics Advisor y el origen de datos. Esto puede ocurrir cuando se actualiza el origen de datos después de que Metrics Advisor haya terminado de extraer datos. Metrics Advisor solo extrae datos de un origen de datos determinado una vez.

Por ejemplo, si se ha incorporado una métrica a Metrics Advisor con fines de supervisión. Metrics Advisor obtiene correctamente los datos de métricas en la marca de tiempo A y realiza la detección de anomalías en ellos. Sin embargo, si los datos de métricas correspondientes a esa marca de tiempo concreta se han actualizado después de la ingesta de los datos, no se recuperará el nuevo valor de los datos.

Puede intentar [reponer ](manage-data-feeds.md#backfill-your-data-feed) los datos históricos (descritos más adelante) para mitigar las incoherencias, pero esto no desencadenará nuevas alertas de anomalías si ya se han desencadenado alertas para esas marcas de tiempo. Este proceso puede agregar más carga de trabajo al sistema; además, no es automático.

Para evitar que se carguen datos parciales, se recomiendan dos enfoques.

* Generar datos en una transacción.

    Asegúrese de que los valores de métricas de todas las combinaciones de dimensiones en la misma marca de tiempo se almacenen en el origen de datos, en una sola transacción. En el ejemplo anterior, espere hasta que estén listos los datos de todos los orígenes de datos y, a continuación, cárguelos en Metrics Advisor en una transacción. Metrics Advisor puede sondear la fuente de distribución de datos periódicamente hasta que los datos se recuperen por completo (o parcialmente).

* Retrasar la ingesta de datos estableciendo un valor adecuado para el parámetro **Ingestion time offset** (Desfase en hora de ingesta).

    Establezca el parámetro **Ingestion time offset** (Desfase en hora de ingesta) de la fuente de distribución de datos para retrasar la ingesta hasta que los datos estén totalmente preparados. Esto puede ser útil para algunos orígenes de datos que no admiten transacciones, como Azure Table Storage. Consulte [Configuración avanzada](manage-data-feeds.md#advanced-settings) para obtener más información.

## <a name="add-a-data-feed-using-the-web-based-workspace"></a>Adición de una fuente de distribución de datos mediante el área de trabajo basada en la Web

Después de iniciar sesión en el portal de Metrics Advisor y elegir el área de trabajo, haga clic en **Get started** (Comenzar). A continuación, en la página principal del área de trabajo, haga clic en **Add data feed** (Agregar fuente de distribución de datos), en el menú de la izquierda.

### <a name="add-connection-settings"></a>Adición de la configuración de conexión

A continuación, deberá especificar un conjunto de parámetros para conectar el origen de datos de series temporales. 
* **Source Type** (Tipo de origen): El tipo de origen de datos donde se almacenan los datos de series temporales.
* **Granularity** (Granularidad): El intervalo entre los puntos de datos consecutivos de los datos de series temporales. Actualmente, Metrics Advisor admite los siguientes: Yearly (Anual), Monthly (Mensual), Weekly (Semanal), Daily (Diario), Hourly (Cada hora) y Custom (Personalizado). El intervalo más bajo compatible con la opción de personalización es de 60 segundos.
  * **Seconds** (Segundos): El número de segundos durante los que *granularityName* se establece en *Customize* (Personalizar).
* **Ingest data since (UTC)** (Ingesta de datos desde [UTC]): La hora de inicio de la línea de base para la ingesta de datos. *startOffsetInSeconds* se usa a menudo para agregar un desfase que ayude a que los datos sean coherentes.

A continuación, deberá especificar la información de conexión para el origen de datos y las consultas personalizadas que se usan para convertir los datos en el esquema requerido. Para obtener más información sobre los demás campos y la conexión de distintos tipos de orígenes de datos, vea [Adición de fuentes de distribución de datos de diferentes orígenes de datos](../data-feeds-from-different-sources.md).

[!INCLUDE [query requirements](../includes/query-requirements.md)]

### <a name="verify-and-get-schema"></a>Comprobación y obtención del esquema

Después de establecer la cadena de conexión y la cadena de consulta, seleccione **Verify and get schema** (Comprobar y obtener el esquema) para comprobar la conexión y ejecutar la consulta con el fin de obtener el esquema de datos del origen de datos. Normalmente, se tarda unos segundos en función de la conexión del origen de datos. Si se produce un error en este paso, confirme lo siguiente.

* La cadena de conexión y la consulta son correctas.
* La instancia de Metrics Advisor puede conectarse al origen de datos si hay una configuración de firewall.

### <a name="schema-configuration"></a>Configuración del esquema

Una vez cargado el esquema de datos, seleccione los campos correspondientes.

Si se omite la marca de tiempo de un punto de datos, Metrics Advisor usará la marca de tiempo cuando se ingiera el punto de datos. Para cada fuente de distribución de datos, puede especificar como máximo una columna como una marca de tiempo. Si recibe un mensaje que indica que no se puede especificar una columna como marca de tiempo, compruebe la consulta o el origen de datos, independientemente de que haya varias marcas de tiempo en el resultado de la consulta, no solo en la vista previa de los datos. Al realizar la ingesta de datos, Metrics Advisor solo puede consumir un fragmento (por ejemplo, un día o una hora, según la granularidad) de los datos de series temporales del origen determinado cada vez.

|Selección  |Descripción  |Notas  |
|---------|---------|---------|
| **Nombre para mostrar** | El nombre que se mostrará en el área de trabajo en lugar del nombre de la columna original. | |
|**Timestamp**     | La marca de tiempo de un punto de datos. Si se omite, Metrics Advisor usará la marca de tiempo cuando se ingiera el punto de datos. Para cada fuente de distribución de datos, puede especificar como máximo una columna como una marca de tiempo.        | Opcional. Debe especificarse con una columna como máximo. Si obtiene un error **column cannot be specified as Timestamp** (La columna no se puede especificar como marca de tiempo), compruebe la consulta o el origen de datos para ver si hay marcas de tiempo duplicadas.      |
|**Measure** (Medida)     |  Los valores numéricos de la fuente de distribución de datos. Para cada fuente de distribución de datos, puede especificar varias medidas, pero debe seleccionar al menos una columna como medida.        | Debe especificarse con al menos una columna.        |
|**Dimensión**     | Los valores de las categorías. Una combinación de valores diferentes identifica una serie temporal de una única dimensión determinada; por ejemplo: Country (País), Language (Idioma) y Tenant (Inquilino). Puede seleccionar cero o más columnas como dimensiones. Nota: Tenga cuidado al seleccionar una columna que no sea de cadena como dimensión. | Opcional.        |
|**Omitir**     | Omite la columna seleccionada.        | Opcional. Vea el texto siguiente.       |

Si desea omitir las columnas, se recomienda actualizar la consulta o el origen de datos para excluir esas columnas. También puede omitir las columnas seleccionando **Ignore columns** (Omitir columnas) y, después, **Ignore** (Omitir) en las columnas específicas. Si una columna debe ser una dimensión y se establece erróneamente como *Ignored* (Omitida), Metrics Advisor puede acabar ingiriendo datos parciales. Por ejemplo, supongamos que los datos de la consulta son los siguientes:

| Identificador de fila | Timestamp | Country (País) | Lenguaje | Income |
| --- | --- | --- | --- | --- |
| 1 | 2019/11/10 | China | ZH-CN | 10000 |
| 2 | 2019/11/10 | China | EN-US | 1000 |
| 3 | 2019/11/10 | US | ZH-CN | 12000 |
| 4 | 2019/11/11 | US | EN-US | 23000 |
| ... | ...| ... | ... | ... |

Si *Country* (País) es una dimensión y *Language* (Idioma) se establece como *Ignored* (Omitido), la primera y la segunda fila tendrán las mismas dimensiones. Metrics Advisor usará arbitrariamente un valor de las dos filas. En este caso, Metrics Advisor no agregará las filas.

### <a name="automatic-roll-up-settings"></a>Configuración de la acumulación automática

> [!IMPORTANT]
> Si desea habilitar el análisis de la causa principal y otras funcionalidades de diagnóstico, es necesario configurar los **valores de acumulación automática**. Una vez habilitado, no se puede cambiar la configuración de la acumulación automática.

Metrics Advisor puede realizar automáticamente la agregación (por ejemplo, SUM, MAX y MIN) en cada dimensión durante la ingesta y, a continuación, crear una jerarquía que se usará en el análisis de la causa principal y en otras características de diagnóstico. 

Considere los siguientes escenarios:

* *No necesito incluir el análisis de acumulación en mis datos.*

    No necesita usar la característica de acumulación de Metrics Advisor.

* *Mis datos ya se han acumulado y el valor de dimensión se representa mediante NULL o Empty (NULL o Vacío), que es el valor predeterminado, NULL only (Solo NULL) y Others (Otros).*

    Esta opción significa que Metrics Advisor no necesita acumular los datos porque las filas ya se han sumado. Por ejemplo, si selecciona *NULL only* (Solo NULL), la segunda fila de datos del ejemplo siguiente se verá como una agregación de todos los países y del idioma *EN-US*; la cuarta fila de datos que tiene un valor vacío para *Country* (País) se considerará, sin embargo, una fila normal que podría indicar datos incompletos.
    
    | Country (País) | Lenguaje | Income |
    |---------|----------|--------|
    | China   | ZH-CN    | 10000  |
    | (NULL)  | EN-US    | 999999 |
    | US      | EN-US    | 12000  |
    |         | EN-US    | 5000   |

* *Necesito que Metrics Advisor acumule los datos calculando Sum, Max, Min, Avg y Count, y que los represente mediante <some string>* .

    Algunos orígenes de datos, como Cosmos DB o Azure Blob Storage, no admiten ciertos cálculos como *agrupar por* o *cubo*. Metrics Advisor proporciona la opción de acumulación para generar automáticamente un cubo de datos durante la ingesta.
    Esta opción significa que necesita Metrics Advisor para calcular la acumulación mediante el algoritmo que ha seleccionado y usar la cadena especificada para representar la acumulación en Metrics Advisor. Esto no cambiará ningún dato en el origen de datos.
    Por ejemplo, supongamos que tiene un conjunto de series temporales que representa las métricas Sales con la dimensión (Country, Region) (País, Región). Para una marca de tiempo determinada, podría ser similar a lo siguiente:


    | Country (País)       | Region           | Sales |
    |---------------|------------------|-------|
    | Canadá        | Alberta          | 100   |
    | Canadá        | Columbia Británica | 500   |
    | Estados Unidos | Montana          | 100   |


    Después de habilitar la acumulación automática con *SUM*, Metrics Advisor calculará las combinaciones de dimensiones y sumará las métricas durante la ingesta de datos. El resultado podría ser el siguiente:

    | Country (País)       | Region           | Sales |
    | ------------ | --------------- | ---- |
    | Canadá        | Alberta          | 100   |
    | NULL          | Alberta          | 100   |
    | Canadá        | Columbia Británica | 500   |
    | NULL          | Columbia Británica | 500   |
    | Estados Unidos | Montana          | 100   |
    | NULL          | Montana          | 100   |
    | NULL          | NULL             | 700   |
    | Canadá        | NULL             | 600   |
    | Estados Unidos | NULL             | 100   |

    `(Country=Canada, Region=NULL, Sales=600)` significa que la suma de Sales (Ventas) en Canadá (todas las regiones) es 600.

    Lo siguiente es la transformación en lenguaje SQL.

    ```mssql
    SELECT
        dimension_1,
        dimension_2,
        ...
        dimension_n,
        sum (metrics_1) AS metrics_1,
        sum (metrics_2) AS metrics_2,
        ...
        sum (metrics_n) AS metrics_n
    FROM
        each_timestamp_data
    GROUP BY
        CUBE (dimension_1, dimension_2, ..., dimension_n);
    ```

    Tenga en cuenta lo siguiente antes de usar la característica de acumulación automática.

    * Si desea usar *SUM* para agregar los datos, asegúrese de que las métricas se suman en cada dimensión. Estos son algunos ejemplos de métricas *no aditivas*.
      * Métricas basadas en fracciones. Se incluyen la proporción, el porcentaje, etc. Por ejemplo, en EE. UU., no debe agregar la tasa de desempleo de cada estado para calcular la tasa de desempleo de todo el país.
      * Superposición en una dimensión. Por ejemplo, no debe agregar el número de personas de cada deporte para calcular el número de personas a las que les gusta el deporte, ya que hay una superposición entre ellas: a una persona le pueden gustar varios deportes.
    * Para garantizar el buen estado de todo el sistema, el tamaño del cubo es limitado. Actualmente, el límite es 1 000 000. Si los datos superan ese límite, se producirá un error en la ingesta en esa marca de tiempo.

## <a name="advanced-settings"></a>Configuración avanzada

Hay varios valores de configuración avanzada para habilitar la ingesta de datos de forma personalizada; por ejemplo, especificando el desfase en la ingesta o la simultaneidad. Para obtener más información, consulte la sección [Configuración avanzada](manage-data-feeds.md#advanced-settings) del artículo sobre la administración de fuentes de distribución de datos.

## <a name="specify-a-name-for-the-data-feed-and-check-the-ingestion-progress"></a>Especificación de un nombre para la fuente de distribución de datos y comprobación del progreso de la ingesta
 
Proporcione un nombre personalizado a la fuente de distribución de datos, que se mostrará en el área de trabajo. Luego, haga clic en **Submit** (Enviar). En la página de detalles de la fuente de distribución de datos, puede usar la barra de progreso de la ingesta para ver la información del estado.

:::image type="content" source="../media/datafeeds/ingestion-progress.png" alt-text="Barra de progreso de la ingesta" lightbox="../media/datafeeds/ingestion-progress.png":::


Para comprobar los detalles del error de la ingesta: 

1. Haga clic en **Show Details** (Mostrar detalles).
2. Haga clic en **Status** (Estado) y seleccione **Failed** (Fallida) o **Error**.
3. Mantenga el mouse sobre una ingesta fallida y vea el mensaje de detalles que aparece.

:::image type="content" source="../media/datafeeds/check-failed-ingestion.png" alt-text="Error en la comprobación de la ingesta":::

Un estado *Failed* (Fallida) indica que la ingesta de este origen de datos se volverá a intentar más tarde.
Un estado *Error* indica que Metrics Advisor no volverá a intentarlo en el origen de datos. Para recargar los datos, debe desencadenar manualmente un proceso de reposición o recarga.

También puede recargar el progreso de una ingesta haciendo clic en **Refresh Progress** (Actualizar progreso). Una vez completada la ingesta de datos, puede hacer clic en las métricas y comprobar los resultados de la detección de anomalías.

## <a name="next-steps"></a>Pasos siguientes
- [Administración de las fuentes de distribución de datos](manage-data-feeds.md)
- [Configuraciones para distintos orígenes de datos](../data-feeds-from-different-sources.md)
- [Configuración de métricas y ajuste de la configuración de la detección](configure-metrics.md)
