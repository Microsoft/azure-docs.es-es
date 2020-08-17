---
title: 'Adición de un conjunto de datos de referencia al entorno: Azure Time Series Insights | Microsoft Docs'
description: En este artículo se describe cómo agregar un conjunto de datos de referencia para aumentar los datos al entorno de Azure Time Series Insights.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.reviewer: jasonh, kfile
ms.workload: big-data
ms.topic: conceptual
ms.date: 08/12/2020
ms.custom: seodec18
ms.openlocfilehash: 57e99dfd0cec4ee396cc90173f1f22715c7ba760
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88168191"
---
# <a name="create-a-reference-data-set-for-your-azure-time-series-insights-gen1-environment-using-the-azure-portal"></a>Creación de un conjunto de datos de referencia para el entorno de Azure Time Series Insights Gen1 mediante Azure Portal

En este artículo se describe cómo agregar un conjunto de datos de referencia al entorno de Azure Time Series Insights. Los datos de referencia son útiles para unirse a los datos de origen y aumentar los valores.

Un conjunto de datos de referencia es una colección de elementos que aumentan los eventos de un origen de eventos. El motor de entrada de Azure Time Series Insights combina cada evento del origen de eventos con la fila de datos correspondiente del conjunto de datos de referencia. A partir de ese momento, este evento aumentado está disponible para consultas. Esta combinación se basa en las columnas de clave principal definidas en el conjunto de datos de referencia.

Los datos de referencia no se combinan de manera retroactiva. Por lo tanto, solo los datos de entrada actuales y futuros se combinan y unen con el conjunto de datos de referencia una vez que se configuran y cargan.

## <a name="video"></a>Vídeo

### <a name="learn-about-time-series-insights-reference-data-modelbr"></a>Obtenga información sobre el modelo de datos de referencia de Time Series Insight.</br>

> [!VIDEO <https://www.youtube.com/embed/Z0NuWQUMv1o>]

## <a name="add-a-reference-data-set"></a>Adición de un conjunto de datos de referencia

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Busque su entorno de Azure Time Series Insights existente. Seleccione **Todos los recursos** en el menú izquierdo de Azure Portal. Seleccione el entorno de Azure Time Series Insights.

1. Seleccione la página **Información general**. Expanda la sección **Essentials** situada cerca de la parte superior de la página para buscar la **dirección URL del Explorador de Time Series Insights** y abra el vínculo.  

   [![Expandir la sección Información esencial](media/add-reference-data-set/essentials.png)](media/add-reference-data-set/essentials.png#lightbox)

   Vea el Explorador del entorno de Azure Time Series Insights.

1. Expanda el selector de entornos en el Explorador de Azure Time Series Insights. Elija el entorno activo. Seleccione el icono de datos de referencia en la esquina superior derecha de la página del Explorador.

   [![Agregar datos de referencia](media/add-reference-data-set/tsi-select-environment-and-data-icons.png)](media/add-reference-data-set/tsi-select-environment-and-data-icons.png#lightbox)

1. Seleccione el botón **+ Agregar un conjunto de datos** para comenzar a agregar un conjunto de datos nuevo.

   [![Agregar conjunto de datos](media/add-reference-data-set/tsi-add-a-reference-data-set.png)](media/add-reference-data-set/tsi-add-a-reference-data-set.png#lightbox)

1. En la página **Nuevo conjunto de datos de referencia**, elija el formato de los datos:

   - Elija **CSV** para los datos delimitados por coma. La primera fila se considera como una fila de encabezado.
   - Elija **Matriz JSON** para los datos con formato de notación de objetos JavaScript (JSON).

   [![Elija el formato de los datos.](media/add-reference-data-set/tsi-select-data-upload-option.png)](media/add-reference-data-set/tsi-select-data-upload-option.png#lightbox)

1. Proporcione los datos con uno de estos dos métodos:

   - Pegue los datos en el editor de texto. Luego, seleccione el botón **Analizar datos de referencia**.
   - Seleccione el botón **Elegir archivo** para agregar datos desde un archivo de texto local.

   Por ejemplo, pegue los datos de CSV: [![Datos de CSV pegados](media/add-reference-data-set/select-csv-and-enter-data.png)](media/add-reference-data-set/select-csv-and-enter-data.png#lightbox)

   Por ejemplo, pegue los datos de la matriz JSON: [![Pegar los datos de JSON](media/add-reference-data-set/select-json-option-and-enter-data.png)](media/add-reference-data-set/select-json-option-and-enter-data.png#lightbox)

   Si hay un error al analizar los valores de datos, el error aparece en rojo en la parte inferior de la página, como `CSV parsing error, no rows extracted`.

1. Una vez que los datos se analizan correctamente, aparece una cuadrícula de datos que muestra las columnas y las filas que representan los datos. Revise la cuadrícula de datos para asegurarse de que esté correcta.

   [![Referencia de revisiones](media/add-reference-data-set/review-displayed-data-grid.png)](media/add-reference-data-set/review-displayed-data-grid.png#lightbox)

1. Revise cada columna para comprender el tipo de datos asumido y cambie el tipo de datos en caso de ser necesario.  Seleccione el símbolo del tipo de datos en el encabezado de columna: **#** para doble (datos numéricos), **T|F** para booleano o **Abc** para cadena.

   [![Elija los tipos de datos en los encabezados de columna.](media/add-reference-data-set/select-column-types.png)](media/add-reference-data-set/select-column-types.png#lightbox)

1. Cambie el nombre de los encabezados de columna si es necesario. El nombre de la columna de clave es necesario para combinarlo con la propiedad correspondiente en el origen del evento.

   > [!IMPORTANT]
   > Asegúrese de que los nombres de la columna de clave de datos de referencia coincidan exactamente con el nombre del evento en los datos entrantes, incluida la distinción entre mayúsculas y minúsculas. Los nombres de la columna no de clave se usan para aumentar los datos entrantes con los valores de datos de referencia correspondientes.

1. Escriba un valor en el campo **Filtrar las filas...** para revisar filas específicas según sea necesario. El filtro es útil para revisar los datos, pero no se aplica al cargar los datos.

1. Para asignarle un nombre al conjunto de datos, rellene el campo **Nombre del conjunto de datos** que está sobre la cuadrícula de los datos.

    [![Asigne un nombre al conjunto de datos.](media/add-reference-data-set/enter-reference-data-set-name.png)](media/add-reference-data-set/enter-reference-data-set-name.png#lightbox)

1. Para proporcionar la columna **Clave principal** en el conjunto de datos, seleccione la lista desplegable sobre la cuadrícula de datos.

    [![Seleccione las columnas de clave.](media/add-reference-data-set/select-primary-key-column.png)](media/add-reference-data-set/select-primary-key-column.png#lightbox)

    **(Opcional)** Seleccione e botón **+** para agregar una columna de clave secundaria, como una clave principal compuesta. Si necesita deshacer la selección, elija el valor vacío en la lista desplegable para quitar la clave secundaria.

1. Para cargar los datos, seleccione el botón **Cargar filas**.

    [![Cargar filas y confirmar datos.](media/add-reference-data-set/confirm-upload-reference-data.png)](media/add-reference-data-set/confirm-upload-reference-data.png#lightbox)

    La página confirma que la carga se completó y muestra el mensaje **El conjunto de datos se ha cargado correctamente**.

    > [!WARNING]
    > Las columnas o propiedades compartidas entre conjuntos de datos de referencia mostrarán un error de carga **Nombre de propiedad duplicado**. El error no impedirá la carga correcta de los conjuntos de datos de referencia. Se puede eliminar mediante la combinación de filas que comparten el nombre de propiedad duplicado.

1. Seleccione **Agregar una fila**, **Importación en bloque de filas** o **Agregar una columna** para agregar más valores de datos de referencia, según sea necesario.

    [![Agregar una fila, Importación en bloque de filas o Agregar una columna.](media/add-reference-data-set/add-row-or-bulk-upload.png)](media/add-reference-data-set/add-row-or-bulk-upload.png#lightbox)

   > [!IMPORTANT]
   > Se reemplazarán las columnas de cualquier fila que comparta una clave única con otra fila por la última fila agregada que comparta esa clave única.

   > [!NOTE]
   > Las filas agregadas **no** deben ser necesariamente *rectangulares*: pueden tener menos columnas, más columnas o variar columnas respecto a otras entradas del conjunto de datos de referencia.

## <a name="next-steps"></a>Pasos siguientes

- [Administración de datos de referencia](time-series-insights-manage-reference-data-csharp.md) mediante programación.

- Para obtener una referencia completa de la API, lea el documento [API de datos de referencia](https://docs.microsoft.com/rest/api/time-series-insights/gen1-reference-data-api).
