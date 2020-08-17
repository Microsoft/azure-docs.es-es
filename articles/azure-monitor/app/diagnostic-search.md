---
title: Uso de Búsqueda en Azure Application Insights | Microsoft Docs
description: Busque y filtre los datos de telemetría sin procesar que envía la aplicación web.
ms.topic: conceptual
ms.date: 07/30/2019
ms.openlocfilehash: f88e0fb8fbd7e9605635ede1e8f71b57de84b74e
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87324461"
---
# <a name="using-search-in-application-insights"></a>Uso de Búsqueda en Application Insights

Búsqueda es una característica de [Application Insights](./app-insights-overview.md) que se usa para buscar y explorar elementos de telemetría individuales, como vistas de páginas, excepciones o solicitudes web. Y puede ver los seguimientos de registros y eventos que haya codificado.

(Para consultas más complejas sobre los datos, use [Analytics](../log-query/get-started-portal.md)).

## <a name="where-do-you-see-search"></a>¿Dónde verá Search?

### <a name="in-the-azure-portal"></a>En el Portal de Azure

Puede abrir la búsqueda de diagnóstico en la pestaña Información general de Application Insights de la aplicación (que se encuentra en la barra superior) o en Investigar a la izquierda.

![Pestaña de búsqueda](./media/diagnostic-search/view-custom-events.png)

Vaya al menú desplegable Tipos de evento para ver una lista de elementos de telemetría: solicitudes de servidor, vistas de página, eventos personalizados que haya codificado, etc. En la parte superior de la lista de resultados hay un gráfico de resumen que muestra los recuentos de eventos con el paso del tiempo.

Haga clic en el menú desplegable o en Actualizar para obtener los nuevos eventos.

### <a name="in-visual-studio"></a>En Visual Studio

En Visual Studio, también hay una ventana de Búsqueda de Application Insights. Esta ventana resulta muy útil para mostrar eventos de telemetría generados por la aplicación que está depurando. Pero también puede mostrar los eventos recopilados desde la aplicación publicada en Azure Portal.

Abra la ventana de búsqueda en Visual Studio:

![Búsqueda de Application Insights abierta en Visual Studio](./media/diagnostic-search/32.png)

La ventana de búsqueda tiene características similares al portal web:

![Ventana de búsqueda de Application Insights en Visual Studio](./media/diagnostic-search/34.png)

La pestaña Track Operation (Seguimiento de operaciones) está disponible cuando se abre una solicitud o una vista de página. Una "operación" es una secuencia de eventos que está asociada a una sola solicitud o vista de página. Por ejemplo, llamadas de dependencia, excepciones, registros de seguimiento y eventos personalizados pueden ser parte de una única operación. La pestaña Track Operation (Seguimiento de operaciones) muestra gráficamente el tiempo y la duración de estos eventos con respecto a la solicitud o vista de página.

## <a name="inspect-individual-items"></a>Inspección de elementos individuales

Seleccione cualquier elemento de telemetría para ver los campos clave y los elementos relacionados.

![Captura de pantalla de una solicitud de dependencia individual](./media/diagnostic-search/telemetry-item.png)

Esto iniciará la vista de detalles de una transacción completa.

## <a name="filter-event-types"></a>Filtro de los tipos de evento

Abra el menú desplegable Tipos de evento y elija los tipos de evento que quiere ver. (Si, posteriormente, quiere restaurar los filtros, haga clic en Restablecer).

Los tipos de evento son:

* **Seguimiento** - [registros de diagnóstico](./asp-net-trace-logs.md), como llamadas a TrackTrace, log4Net, NLog y System.Diagnostic.Trace.
* **Solicitud** - solicitudes HTTP recibidas por la aplicación de servidor, como páginas, scripts, imágenes, archivos de estilo y datos. Estos eventos se utilizan para crear los gráficos de información general de solicitudes y respuestas.
* **Vista de página** - [es la telemetría enviada por el cliente web](./javascript.md) , que se usa para crear informes de vistas de página.
* **Evento personalizado** - si ha insertado llamadas a TrackEvent() [para supervisar el uso](./api-custom-events-metrics.md) , puede buscarlas aquí.
* **Excepción** - [excepciones no detectadas en el servidor](./asp-net-exceptions.md) y las que se registran mediante TrackException().
* **Dependencia:**  - [ llamadas desde su aplicación de servidor](./asp-net-dependencies.md) a otros servicios, como API de REST o bases de datos, y llamadas de AJAX desde su [código de cliente](./javascript.md).
* **Disponibilidad**: resultados de [pruebas de disponibilidad](./monitor-web-app-availability.md).

## <a name="filter-on-property-values"></a>Filtro de los valores de propiedad

Puede filtrar eventos por los valores de sus propiedades. Las propiedades disponibles dependen de los tipos de evento que haya seleccionado. Haga clic en el icono de filtro ![Icono de Filtro](./media/diagnostic-search/filter-icon.png) para comenzar.

El hecho de no elegir ningún valor de una propiedad en particular tiene el mismo efecto que elegir todos los valores. Se desactiva el filtrado en esa propiedad.

Observe que los recuentos a la derecha de los valores de filtro muestran cuántas repeticiones hay en el conjunto filtrado actual.

## <a name="find-events-with-the-same-property"></a>Búsqueda de eventos con la misma propiedad

Para buscar todos los elementos con el mismo valor de propiedad, escríbalo en la barra de búsqueda o haga clic en la casilla al examinar las propiedades en la pestaña de filtro.

![Haga clic en la casilla de una propiedad en la pestaña de filtro](./media/diagnostic-search/filter-property.png)

## <a name="search-the-data"></a>Búsqueda de los datos

> [!NOTE]
> Para escribir consultas más complejas, abra [**Registros (Analytics)** ](../log-query/get-started-portal.md) desde la parte superior de la hoja Buscar.
>

Puede buscar términos en cualquiera de los valores de propiedad. Esto es útil si ha escrito [eventos personalizados](./api-custom-events-metrics.md) con valores de propiedad.

Quizás desee establecer un intervalo de tiempo, dado que las búsquedas en un intervalo más corto son más rápidas.

![Open diagnostic search](./media/diagnostic-search/search-property.png)

Busque palabras completas, no subcadenas. Use comillas con los caracteres especiales.

| String | *No* encontrado | Encontrado |
| --- | --- | --- |
| HomeController.About |`home`<br/>`controller`<br/>`out` | `homecontroller`<br/>`about`<br/>`"homecontroller.about"`|
|Estados Unidos|`Uni`<br/>`ted`|`united`<br/>`states`<br/>`united AND states`<br/>`"united states"`

A continuación se muestran las expresiones de búsqueda que puede utilizar:

| Consulta de ejemplo | Efecto |
| --- | --- |
| `apple` |Busca todos los eventos del intervalo de tiempo cuyos campos incluyen la palabra "apple". |
| `apple AND banana` <br/>`apple banana` |Busca eventos que contienen ambos términos. Utilizar "AND" en mayúsculas, no "and". <br/>Forma abreviada. |
| `apple OR banana` |Buscar eventos que contengan cualquiera de los dos términos. Usar "OR" no "or". |
| `apple NOT banana` |Buscar eventos que contengan un término pero no el otro. |

## <a name="sampling"></a>muestreo

Si la aplicación genera una gran cantidad de datos de telemetría (y está usando la versión 2.0.0-beta3 o posterior del SDK de ASP.NET), el módulo de muestreo adaptable reduce automáticamente el volumen que se envía al portal mediante el envío de solamente una fracción representativa de los eventos. Sin embargo, los eventos relacionados con la misma solicitud se seleccionan o deseleccionan como grupo, por lo que puede navegar entre ellos.

[Más información sobre el muestreo](./sampling.md).

## <a name="create-work-item"></a>Creación de elemento de trabajo

Puede crear un error en GitHub o en Azure DevOps, con información de cualquier elemento de telemetría.

Para ir a la vista de detalles de la transacción de un extremo a otro, haga clic en cualquier elemento de telemetría y seleccione **Crear elemento de trabajo**.

![Haga clic en Nuevo elemento de trabajo, modifique los campos y, a continuación, haga clic en Aceptar.](./media/diagnostic-search/work-item.png)

La primera vez que lo haga, se le pedirá que configure un vínculo a la organización y el proyecto de Azure DevOps.

(También puede configurar el vínculo en la pestaña Elementos de trabajo).

## <a name="send-more-telemetry-to-application-insights"></a>Envío de más telemetría a Application Insights

Además de la telemetría inmediata enviada por el SDK de Application Insights, puede:

* Capturar seguimientos de registros de su plataforma de registro de favoritos en [.NET](./asp-net-trace-logs.md) o [Java](./java-trace-logs.md). Esto significa que puede buscar en los seguimientos de registros y correlacionarlos con vistas de página, excepciones y otros eventos.
* [Escribir código](./api-custom-events-metrics.md) para enviar eventos personalizados, vistas de página y excepciones.

[Aprender a enviar registros y telemetría personalizada a Application Insights](./asp-net-trace-logs.md).

## <a name="q--a"></a><a name="questions"></a>Preguntas y respuestas

### <a name="how-much-data-is-retained"></a><a name="limits"></a>¿Qué cantidad de datos se conserva?

Consulte [Resumen de límites](./pricing.md#limits-summary).

### <a name="how-can-i-see-post-data-in-my-server-requests"></a>¿Cómo puedo ver datos POST en mis solicitudes de servidor?

Aunque no registramos los datos POST automáticamente, puede usar [TrackTrace o llamadas de registro](./asp-net-trace-logs.md). Coloque los datos POST en el parámetro de mensaje. No puede filtrar por el mensaje de la misma forma que con las propiedades, pero el límite de tamaño es mayor.

## <a name="next-steps"></a><a name="add"></a>Pasos siguientes

* [Escribir consultas complejas en Analytics](../log-query/get-started-portal.md)
* [Envío de registros y telemetría personalizada a Application Insights](./asp-net-trace-logs.md).
* [Configuración de pruebas de disponibilidad y de capacidad de respuesta](./monitor-web-app-availability.md)
* [Solución de problemas](../faq.md)

