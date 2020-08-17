---
title: Libros de Azure Monitor con parámetros personalizados
description: Simplifique la creación de informes complejos con libros parametrizados creados previamente y personalizados.
services: azure-monitor
author: mrbullwinkle
manager: carmonm
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 07/20/2020
ms.author: mbullwin
ms.openlocfilehash: 33da3cd8a72bb4d93011c348db65c5b4d9e687ed
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87461470"
---
# <a name="interactive-workbooks"></a>Libros interactivos

Los libros permiten a los autores crear experiencias e informes interactivos para sus consumidores. La interactividad se admite de varias maneras.

## <a name="parameter-changes"></a>Cambios de parámetros

Cuando un usuario de un libro actualiza un parámetro, todos los controles que usen el parámetro se actualizan automáticamente y se vuelven a dibujar para reflejar el nuevo estado. Así es como la mayoría de los informes de Azure Portal admiten interactividad. En los libros se proporciona de forma sencilla y con un esfuerzo mínimo por parte del usuario.

Más información sobre los [parámetros de los libros](workbooks-parameters.md)

## <a name="grid-tile-chart-selections"></a>Selecciones de cuadrícula, mosaico y gráfico

Los libros permiten a los autores construir escenarios en los que al hacer clic en una fila de una cuadrícula se actualizan los gráficos posteriores según el contenido de la fila.

Por ejemplo, un usuario puede tener una cuadrícula que muestre una lista de solicitudes y algunas estadísticas, como recuentos de errores. Esta cuadrícula se podría configurar de modo que al hacer clic en una fila correspondiente a una solicitud, el resultado fueran gráficos detallados que se actualizan para filtrar solo esa solicitud.

### <a name="setting-up-interactivity-on-grid-row-click"></a>Configuración de la interactividad al hacer clic en la fila de cuadrícula

1. Para cambiar el libro al modo de edición, haga clic en el elemento de la barra de herramientas _Editar_.
2. Use el vínculo _Agregar consulta_ para agregar un control de consulta de registro al libro.
3. Seleccione el tipo de consulta _Registro_, el tipo de recurso (por ejemplo, Application Insights) y los recursos de destino.
4. Use el editor de consultas para especificar el KQL para el análisis.

    ```kusto
    requests
    | summarize AllRequests = count(), FailedRequests = countif(success == false) by Request = name
    | order by AllRequests desc
    ```

5. Use `Run query` para ver los resultados.
6. Seleccione el icono _Configuración avanzada_ en el pie de página de la consulta (el icono es similar a un engranaje). Se abre el panel Configuración avanzada.
7. Active la opción `When an item is selected, export a parameter`.
8. En la opción que ha activado, seleccione *Agregar parámetro* y rellénelo con la información siguiente.
    1. Campo para exportar: `Request`
    2. Nombre de parámetro: `SelectedRequest`
    3. Valor predeterminado: `All requests`
9. Seleccionar Guardar

    ![Captura de pantalla que muestra el editor avanzado con la configuración para exportar campos como parámetros.](./media/workbooks-interactive/export-parameters-add.png)

10. Seleccione `Done Editing`.
11. Agregue otro control de consulta con los pasos 2 y 3.
12. Use el editor de consultas para especificar el KQL para el análisis.
    ```kusto
    requests
    | where name == '{SelectedRequest}' or 'All Requests' == '{SelectedRequest}'
    | summarize ['{SelectedRequest}'] = count() by bin(timestamp, 1h)
    ```
13. Use `Run query` para ver los resultados.
14. Cambie _Visualización_ a `Area chart`.
15. Elija una fila para seleccionarla en la primera cuadrícula. Observe cómo el gráfico de área que aparece debajo se filtra a la solicitud seleccionada.

El informe resultante tiene el siguiente aspecto en el modo de edición:

![Captura de pantalla de las dos primeras consultas en el modo de edición.](./media/workbooks-interactive//interactivity-grid-create.png)

La imagen siguiente muestra un informe interactivo más elaborado en modo de lectura basado en los mismos principios. El informe usa clics de cuadrícula para exportar parámetros, que a su vez se usan en dos gráficos y en un bloque de texto.

![Captura de pantalla de un informe mediante clics de cuadrícula en modo de lectura.](./media/workbooks-interactive/interactivity-grid-read-mode.png)

### <a name="exporting-the-contents-of-an-entire-row"></a>Exportación del contenido de una fila completa

A veces es conveniente exportar todo el contenido de la fila seleccionada en lugar de una sola columna determinada. En tales casos, deje la propiedad `Field to export` sin establecer en el paso 7.1 anterior. Los libros exportarán todo el contenido de la fila como JSON al parámetro.

En el control KQL al que se hace referencia, use la función `todynamic` para analizar el código JSON y acceder a las columnas individuales.

## <a name="grid-cell-clicks"></a>Clics de celda de cuadrícula

Los libros permiten a los autores agregar interactividad mediante un tipo especial de representador de columnas de cuadrícula denominado `link renderer`. Un representador de vínculos convierte una celda de cuadrícula en un hipervínculo basado en el contenido de la celda. Los libros admiten muchos tipos de representadores de vínculos, incluidos los que permiten abrir hojas de información general de recursos, visores de contenedores de propiedades, búsqueda de App Insights, uso, seguimiento de transacciones, etc.

### <a name="setting-up-interactivity-using-grid-cell-clicks"></a>Configuración de la interactividad mediante clics de celda de cuadrícula

1. Para cambiar el libro al modo de edición, haga clic en el elemento de la barra de herramientas _Editar_.
2. Use el vínculo _Agregar consulta_ para agregar un control de consulta de registro al libro.
3. Seleccione el tipo de consulta _Registro_, el tipo de recurso (por ejemplo, Application Insights) y los recursos de destino.
4. Use el editor de consultas para especificar el KQL para el análisis.

    ```kusto
    requests
    | summarize Count = count(), Sample = any(pack_all()) by Request = name
    | order by Count desc
    ```

5. Use `Run query` para ver los resultados.
6. Seleccione _Configuración de columnas_ para abrir el panel de configuración.
7. En la sección _Columnas_, establezca:
    1. _Sample_ - Representador de columnas: `Link`, Vista para abrir: `Cell Details`, Etiqueta de vínculo: `Sample`
    2. _Count_ - Representador de columnas: `Bar`, Paleta de colores: `Blue`, Valor mínimo: `0`
    3. _Request_ - Representador de columnas: `Automatic`
    4. Seleccione _Guardar y cerrar_ para aplicar los cambios.

    ![Captura de pantalla de la pestaña de configuración de columna.](./media/workbooks-interactive/column-settings.png)

8. Haga clic en uno de los vínculos `Sample` de la cuadrícula. Se abre un panel con los detalles de una solicitud muestreada.

    ![Captura de pantalla del panel de detalles de la solicitud muestreada.](./media/workbooks-interactive/details.png)

### <a name="link-renderer-actions"></a>Acciones del representador de vínculos

| Acción de vínculo | Acción al hacer clic |
|:------------- |:-------------|
| `Generic Details` | Muestra los valores de fila de una pestaña de contexto de cuadrícula de propiedades. |
| `Cell Details` | Muestra el valor de celda de una pestaña de contexto de cuadrícula de propiedades. Resulta útil cuando la celda contiene un tipo dinámico con información (por ejemplo, JSON con propiedades de solicitud, como ubicación, instancia de rol, etc.). |
| `Cell Details` | Muestra el valor de celda de una pestaña de contexto de cuadrícula de propiedades. Resulta útil cuando la celda contiene un tipo dinámico con información (por ejemplo, JSON con propiedades de solicitud, como ubicación, instancia de rol, etc.). |
| `Custom Event Details` | Abre los detalles de búsqueda de Application Insights con el identificador de evento personalizado (`itemId`) en la celda. |
| `* Details` | Similar a los detalles de eventos personalizados, exceptuando las dependencias, las excepciones, las vistas de página, las solicitudes y los seguimientos. |
| `Custom Event User Flows` | Abre la experiencia Flujos de usuario de Application Insights dinamizada en el nombre del evento personalizado de la celda. |
| `* User Flows` | Similar a Flujos de usuario de eventos personalizados, exceptuando las excepciones, las vistas de página y las solicitudes. |
| `User Timeline` | Abre la escala de tiempo del usuario con el identificador de usuario (user_Id) de la celda. |
| `Session Timeline` | Abre la experiencia de búsqueda de Application Insights para el valor de la celda (por ejemplo, buscar texto "abc" donde abc es el valor de la celda). |
| `Resource overview` | Abre la información general del recurso en el portal en función del valor del identificador del recurso de la celda. |

## <a name="conditional-visibility"></a>Visibilidad condicional

Los libros permiten a los usuarios hacer que determinados controles aparezcan o desaparezcan en función de los valores de los parámetros. De esta forma, los autores pueden hacer que los informes tengan una apariencia diferente en función de la entrada del usuario o del estado de la telemetría. Un ejemplo sería mostrar a los consumidores un resumen cuando las cosas van bien, pero mostrar detalles completos cuando algo va mal.

### <a name="setting-up-interactivity-using-conditional-visibility"></a>Configuración de la interactividad mediante la visibilidad condicional

1. Siga los pasos de la sección [Configuración de la interactividad al hacer clic en la fila de cuadrícula](#setting-up-interactivity-on-grid-row-click) para configurar dos controles interactivos.
2. Agregue un nuevo parámetro en la parte superior:
    1. Nombre: `ShowDetails`
    2. Tipo de parámetro: `Drop down`
    3. Obligatorio: `checked`
    4. Obtener datos de: `JSON`
    5. Entrada JSON: `["Yes", "No"]`
    6. Guarde para confirmar los cambios.

    ![Después de seleccionar el botón Agregar parámetro, se muestra el panel para editar el parámetro.](./media/workbooks-interactive/edit-parameter.png)

3. Establezca el valor del parámetro en `Yes`.

    ![Encima del botón de edición finalizada se encuentra la lista desplegable que le permitirá establecer el valor del parámetro.](./media/workbooks-interactive/set-parameter.png)

4. En el control de consulta con el gráfico de áreas, seleccione el icono _Configuración avanzada_ (icono de engranaje).
5. Marque el valor `Make this item conditionally visible`.
    1. Este elemento está visible si el valor del parámetro `ShowDetails` `equals` `Yes`.
6. Seleccione _Edición finalizada_ para confirmar los cambios.
7. Seleccione _Edición finalizada_ en la barra de herramientas del libro para entrar en el modo de lectura.
8. Cambie el valor del parámetro `ShowDetails` a `No`. Observe que el gráfico siguiente desaparece.

En la imagen siguiente se muestra el caso visible en el que `ShowDetails` es `Yes`.

![Captura de pantalla que muestra la visibilidad condicional donde el gráfico está visible](./media/workbooks-interactive/interactivity-conditional-visibility-visible.png)

En la imagen siguiente se muestra el caso oculto en el que `ShowDetails` es `No`.

![Captura de pantalla que muestra la visibilidad condicional donde el gráfico está oculto](./media/workbooks-interactive/interactivity-conditional-visibility-invisible.png)

## <a name="interactivity-with-multiple-selections-in-grids-and-charts"></a>Interactividad con varias selecciones en cuadrículas y gráficos

Los pasos de consulta y métricas también pueden exportar uno o varios parámetros cuando se selecciona una o varias filas.

![Captura de pantalla que muestra la configuración de los parámetros de exportación con varios parámetros. ](./media/workbooks-interactive/interactivity-export-parameters.png)

1. En el paso de consulta que muestra la cuadrícula, vaya a la configuración avanzada.
2. Active la casilla `When items are selected, export parameters`. Aparecerán controles adicionales.
3. Active la casilla `allow selection of multiple values`.
    1. La visualización mostrada permitirá que los valores del parámetro de selección múltiple y exportados sean matrices de valores, igual que cuando se usan parámetros desplegables de selección múltiple.
    2. Si no está activada, la visualización solo respetará el último elemento seleccionado. Solo se exporta un valor único cada vez.
4. Para cada parámetro que desee exportar, use el botón *Agregar parámetro*. Aparecerá una ventana emergente que contiene la configuración del parámetro que se vaya a exportar.

Cuando la selección única está habilitada, el autor puede especificar qué campo de los datos originales se va a exportar. Los campos son el nombre del parámetro, el tipo de parámetro y el valor predeterminado que se usarán si no se selecciona nada (opcional).

Cuando la selección múltiple está habilitada, el autor especifica qué campo de los datos originales se va a exportar. Los campos son el nombre del parámetro, el tipo de parámetro, la comilla y el delimitador. Los valores de las comillas y del delimitador se usan al convertir los valores de las flechas en texto cuando se reemplazan en una consulta. Si no se selecciona ningún valor en la selección múltiple, de forma predeterminada es una matriz vacía.

> [!NOTE]
> En el caso de la selección múltiple, solo se exportarán los valores únicos; no verá los valores de la matriz de salida como "1,1,2,1", sino que obtendrá "1,2" como valores de salida.

Puede dejar la opción "Campo para exportar" vacía en la configuración de exportación. Si lo hace, todos los campos disponibles en los datos se exportarán como un objeto JSON de pares clave:valor que son cadenas. En el caso de las cuadrículas y los títulos, serán todos los campos de la cuadrícula. En el caso de los gráficos, los campos disponibles serán x, y, series y etiqueta (dependiendo del tipo de gráfico).

Aunque el comportamiento predeterminado es exportar un parámetro como texto, si sabe que el campo es un identificador de suscripción o recurso, úselo como tipo de parámetro de exportación. De esta forma, podrá usar el parámetro después en lugares que requieran esos tipos de parámetros.

## <a name="next-steps"></a>Pasos siguientes

* [Comience](workbooks-visualizations.md) a aprender más sobre las muchas opciones de visualizaciones enriquecidas de los libros.
* [Controle](workbooks-access-control.md) y comparta el acceso a los recursos del libro.
