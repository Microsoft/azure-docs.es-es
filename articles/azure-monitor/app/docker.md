---
title: Supervisión de aplicaciones de Docker en Azure Application Insights | Microsoft Docs
description: En Application Insights, se pueden mostrar los contadores de rendimiento, los eventos y las excepciones de Docker, además de la telemetría de las aplicaciones en contenedor.
ms.topic: conceptual
ms.date: 03/14/2019
ms.openlocfilehash: 1cbb2968fec68eb750ce3c9b6cac09f23a1d36c5
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87324427"
---
# <a name="monitor-docker-applications-in-application-insights-deprecated"></a>Supervisar aplicaciones de Docker en Application Insights (en desuso)

> [!NOTE]
> Esta solución está en desuso. Para más información sobre nuestras inversiones actuales en la supervisión de contenedores, se recomienda comprobar [Azure Monitor para contenedores](../insights/container-insights-overview.md).

Los eventos de ciclo de vida y los contadores de rendimiento de los contenedores [Docker](https://www.docker.com/) pueden mostrarse en gráficos en Application Insights. Instale la imagen de [Application Insights](https://hub.docker.com/r/microsoft/applicationinsights/) en un contenedor del host y se mostrarán los contadores de rendimiento del host, así como de las demás imágenes.

Con Docker, distribuye sus aplicaciones en contenedores ligeros junto con todas las dependencias. Estos contenedores se ejecutan en cualquier equipo host que ejecute un motor Docker.

Al ejecutar la [imagen de Application Insights](https://hub.docker.com/r/microsoft/applicationinsights/) en el host Docker, obtendrá estas ventajas:

* Telemetría del ciclo de vida sobre todos los contenedores que se ejecutan en el host: start, stop, etc.
* Contadores de rendimiento de todos los contenedores. CPU, memoria, uso de red y mucho más.
* Si [instaló el SDK de Application Insights](./java-get-started.md) en las aplicaciones que se ejecutan en los contenedores, toda la telemetría de dichas aplicaciones tendrán propiedades adicionales que identifiquen el contenedor y el equipo host. De esta forma, si tiene, por ejemplo, instancias de una aplicación que se ejecuta en más de un host, podrá filtrar fácilmente los datos de telemetría de la aplicación en función del host.

## <a name="set-up-your-application-insights-resource"></a>Configurar el recurso de Application Insights

1. Inicie sesión en [Microsoft Azure Portal](https://azure.com) y abra el recurso de Application Insights de la aplicación; o [cree uno nuevo](./create-new-resource.md). 
   
    *¿Qué recurso debería usar?* Si las aplicaciones que se ejecutan en el host las desarrolló otra persona, deberá [crear un nuevo recurso de Application Insights](./create-new-resource.md). Es el sitio donde verá y analizará los datos de telemetría. Seleccione General para el tipo de aplicación.
   
    Pero si usted es el desarrollador de las aplicaciones, esperamos que [haya agregado el SDK de Application Insights](./java-get-started.md) a cada una de ellas. Si todos son realmente componentes de una aplicación empresarial única, puede configurarlas todas para que envíen telemetría a un recurso y usará ese mismo recurso para mostrar los datos de rendimiento y de ciclo de vida de Docker. 
   
    Un tercer escenario es que haya desarrollado la mayoría de las aplicaciones, pero use recursos independientes para mostrar sus datos de telemetría. En ese caso, probablemente también querrá crear un recurso independiente para los datos de Docker.

2. Haga clic en la lista desplegable **Essentials** y copie la clave de instrumentación. Se usará para indicar al SDK dónde puede enviar sus datos de telemetría.

Tenga a mano la ventana del explorador, volverá a ella pronto para examinar los datos de telemetría.

## <a name="run-the-application-insights-monitor-on-your-host"></a>Ejecutar la supervisión de Application Insights en el host

Ahora que tiene un lugar donde mostrar los datos de telemetría, puede configurar la aplicación del contenedor que los recopilará y enviará.

1. Conéctese al host Docker.
2. Edite la clave de instrumentación de este comando y después ejecútelo:
   
   ```
   
   docker run -v /var/run/docker.sock:/docker.sock -d microsoft/applicationinsights ikey=000000-1111-2222-3333-444444444
   ```

Solo es necesaria una imagen de Application Insights por cada host de Docker. Si la aplicación se implementa en varios hosts de Docker, repita el comando en todos los hosts.

## <a name="update-your-app"></a>Actualización de la aplicación
Si la aplicación se instrumenta con el [SDK de Application Insights para Java`<TelemetryInitializers>`, agregue la siguiente línea en el archivo ApplicationInsights.xml del proyecto, en el elemento ](./java-get-started.md):

```xml

    <Add type="com.microsoft.applicationinsights.extensibility.initializer.docker.DockerContextInitializer"/> 
```

Esto agrega información de Docker como el contenedor y el identificador de host a cada uno de los elementos de telemetría que se envían desde la aplicación.

## <a name="view-your-telemetry"></a>Visualización de la telemetría
Vuelva al recurso de Application Insights en el Portal de Azure.

Haga clic en el icono de Docker.

Pronto verá que llegan datos desde la aplicación de Docker, especialmente si tiene otros contenedores que se ejecutan en el motor de Docker.

### <a name="docker-container-events"></a>Eventos de contenedor Docker
![ejemplo](./media/docker/13.png)

Para investigar eventos individuales, haga clic en [Buscar](./diagnostic-search.md). Busque y filtre para encontrar los eventos deseados. Haga clic en cualquier evento para más detalles.

### <a name="exceptions-by-container-name"></a>Excepciones por el nombre del contenedor
![ejemplo](./media/docker/14.png)

### <a name="docker-context-added-to-app-telemetry"></a>Contexto de Docker agregado a los datos de telemetría de la aplicación
Los datos de telemetría de solicitud enviados desde la aplicación instrumentada con el SDK de AI están enriquecidos con la información de contexto de Docker.

## <a name="q--a"></a>Preguntas y respuestas
*¿Qué me da Application Insights que no me dé Docker?*

* Desglose detallado de los contadores de rendimiento por contenedor e imagen.
* Integración de datos de contenedores y aplicaciones en un panel.
* [Exportación de la telemetría](export-telemetry.md) , para su análisis posterior en una base de datos, Power BI u otro panel.

*¿Cómo se obtienen datos de telemetría de la propia aplicación?*

* Instale el SDK de Application Insights en la aplicación. Más información sobre: [aplicaciones web de Java](./java-get-started.md) y [aplicaciones web de Windows](./asp-net.md).

## <a name="video"></a>Vídeo

> [!VIDEO https://channel9.msdn.com/events/Connect/2016/100/player]

## <a name="next-steps"></a>Pasos siguientes

* [Application Insights para Java](./java-get-started.md)
* [Application Insights para Node.js](./nodejs.md)
* [Application Insights para ASP.NET](./asp-net.md)

