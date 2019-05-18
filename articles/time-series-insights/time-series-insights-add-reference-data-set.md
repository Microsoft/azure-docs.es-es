---
title: Adición de un conjunto de datos de referencia al entorno de Azure Time Series Insights | Microsoft Docs
description: En este artículo se describe cómo agregar un conjunto de datos de referencia para aumentar los datos al entorno de Azure Time Series Insights.
ms.service: time-series-insights
services: time-series-insights
author: ashannon7
ms.author: anshan
manager: cshankar
ms.reviewer: jasonh, kfile, anshan
ms.workload: big-data
ms.topic: conceptual
ms.date: 05/13/2019
ms.custom: seodec18
ms.openlocfilehash: c0b9db05effe42880465310f6eb5c38e3e832266
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65606914"
---
# <a name="create-a-reference-data-set-for-your-time-series-insights-environment-using-the-azure-portal"></a>Creación de un conjunto de datos de referencia para el entorno de Time Series Insights mediante Azure Portal

En este artículo se describe cómo agregar un conjunto de datos de referencia al entorno de Azure Time Series Insights. Los datos de referencia son útiles para unirse a los datos de origen y aumentar los valores.

Un conjunto de datos de referencia es una colección de elementos que aumentan los eventos de un origen de eventos. El motor de entrada de Time Series Insights combina cada evento del origen de eventos con la fila de datos correspondiente en el conjunto de datos de referencia. A partir de ese momento, este evento aumentado está disponible para consultas. Esta combinación se basa en las columnas de clave principal definidas en el conjunto de datos de referencia.

Los datos de referencia no se combinan de manera retroactiva. Por lo tanto, se coincide con los datos de entrada solo actuales y futuros y se une al conjunto de datos de referencia una vez que se ha configurado y cargado.

## <a name="video"></a>Vídeo

### <a name="learn-about-time-series-insights-reference-data-modelbr"></a>Obtenga información sobre el modelo de datos de referencia de Time Series Insights.</br>

> [!VIDEO https://www.youtube.com/embed/Z0NuWQUMv1o]

## <a name="add-a-reference-data-set"></a>Adición de un conjunto de datos de referencia

1. Inicie sesión en el [Azure Portal](https://portal.azure.com).

1. Busque su entorno de Time Series Insights existente. Haga clic en **Todos los recursos** en el menú izquierdo de Azure Portal. Seleccione el entorno de Time Series Insights.

1. Seleccione la página **Información general**. Busque la **URL del explorador de Time Series Insights** y abra el vínculo.  

   Consulte el explorador para ver el entorno de TSI.

1. Expanda el selector de entorno en el explorador de TSI. Elija el entorno activo. Seleccione el icono de los datos de referencia en la esquina superior derecha de la página del explorador.

   [![Agregar datos de referencia](media/add-reference-data-set/add_reference_data.png)](media/add-reference-data-set/add_reference_data.png#lightbox)

1. Seleccione el botón **+ Agregar un conjunto de datos** para comenzar a agregar un conjunto de datos nuevo.

   [![Agregar conjunto de datos](media/add-reference-data-set/add_data_set.png)](media/add-reference-data-set/add_data_set.png#lightbox)

1. En la página **Nuevo conjunto de datos de referencia**, elija el formato de los datos:
   - Elija **CSV** para los datos delimitados por coma. La primera fila se considera como una fila de encabezado.
   - Elija **Matriz JSON** para los datos con formato de notación de objetos JavaScript (JSON).

   [![Elija el formato de datos.](media/add-reference-data-set/add_data.png)](media/add-reference-data-set/add_data.png#lightbox)

1. Proporcione los datos con uno de estos dos métodos:
   - Pegue los datos en el editor de texto. Luego, seleccione el botón **Analizar datos de referencia**.
   - Seleccione el botón **Elegir archivo** para agregar datos desde un archivo de texto local.

   Por ejemplo, pegue los datos de CSV: [![Datos CSV pegados](media/add-reference-data-set/csv_data_pasted.png)](media/add-reference-data-set/csv_data_pasted.png#lightbox)

   Por ejemplo, pegue los datos de la matriz JSON: [![Pegar datos JSON](media/add-reference-data-set/json_data_pasted.png)](media/add-reference-data-set/json_data_pasted.png#lightbox)

   Si hay un error al analizar los valores de datos, el error aparece en rojo en la parte inferior de la página, como `CSV parsing error, no rows extracted`.

1. Una vez que los datos se analizan correctamente, aparece una cuadrícula de datos que muestra las columnas y las filas que representan los datos.  Revise la cuadrícula de datos para asegurarse de que esté correcta.

   [![Agregar datos de referencia](media/add-reference-data-set/parse_data.png)](media/add-reference-data-set/parse_data.png#lightbox)

1. Revise cada columna para ver el tipo de datos asumido y cambie el tipo de datos en caso de ser necesario.  Seleccione el símbolo del tipo de datos en el encabezado de columna: **#** para doble (datos numéricos), **T|F** para booleano o **Abc** para cadena.

   [![Elija los tipos de datos en los encabezados de columna.](media/add-reference-data-set/choose_datatypes.png)](media/add-reference-data-set/choose_datatypes.png#lightbox)

1. Cambie el nombre de los encabezados de columna si es necesario. El nombre de la columna de clave es necesario para combinarlo con la propiedad correspondiente en el origen del evento. Asegúrese de que los nombres de la columna de clave de datos de referencia coincidan exactamente con el nombre del evento en los datos entrantes, incluida la distinción entre mayúsculas y minúsculas. Los nombres de la columna no de clave se usan para aumentar los datos entrantes con los valores de datos de referencia correspondientes.

1. Haga clic en **Agregar una fila** o en **Agregar una columna** para agregar más valores de datos de referencia, según sea necesario.

1. Escriba un valor en el campo **Filtrar las filas...** para revisar filas específicas según sea necesario. El filtro es útil para revisar los datos, pero no se aplica al cargar los datos.

1. Para asignarle un nombre al conjunto de datos, rellene el campo **Nombre del conjunto de datos** que está sobre la cuadrícula de los datos.

    [![Nombre del conjunto de datos.](media/add-reference-data-set/name_reference_dataset.png)](media/add-reference-data-set/name_reference_dataset.png#lightbox)

1. Para proporcionar la columna **Clave principal** en el conjunto de datos, seleccione la lista desplegable sobre la cuadrícula de datos.

    [![Seleccione las columnas de clave.](media/add-reference-data-set/set_primary_key.png)](media/add-reference-data-set/set_primary_key.png#lightbox)

    De manera opcional, seleccione el botón **+** para agregar una columna de clave secundaria, como una clave principal compuesta. Si necesita deshacer la selección, elija el valor vacío en la lista desplegable para quitar la clave secundaria.

1. Para cargar los datos, seleccione el botón **Cargar filas**.

    [![Cargar](media/add-reference-data-set/upload_rows.png)](media/add-reference-data-set/upload_rows.png#lightbox)

    La página confirma que la carga se completó y muestra el mensaje **El conjunto de datos se ha cargado correctamente**.

## <a name="next-steps"></a>Pasos siguientes

* [Administración de datos de referencia](time-series-insights-manage-reference-data-csharp.md) mediante programación.

* Para obtener una referencia completa a la API, consulte el documento [API de datos de referencia](/rest/api/time-series-insights/ga-reference-data-api).
