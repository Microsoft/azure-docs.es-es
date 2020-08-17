---
title: Configuración del análisis de aplicaciones web para ASP.NET con Azure Application Insights | Microsoft Docs
description: Configure las herramientas de análisis del rendimiento, la disponibilidad y de comportamiento de usuarios para un sitio web de ASP.NET, hospedado localmente o en Azure.
ms.topic: conceptual
ms.date: 05/08/2019
ms.openlocfilehash: acfba63cba520631831888a1480929be3b1897f0
ms.sourcegitcommit: 5f7b75e32222fe20ac68a053d141a0adbd16b347
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87475539"
---
# <a name="set-up-application-insights-for-your-aspnet-website"></a>Configuración de Application Insights para un sitio web de ASP.NET

Este procedimiento configura una aplicación web de ASP.NET para que envíe datos de telemetría al servicio [Azure Application Insights](./app-insights-overview.md). Funciona en las aplicaciones ASP.NET que se hospedan en su propio servidor IIS local o en la nube. Obtenga gráficos y un eficaz lenguaje de consulta que le ayudarán a conocer el rendimiento de la aplicación y cómo la usan las personas, más alertas automáticas cuando aparezcan errores o haya problemas de rendimiento. Muchos desarrolladores consideran que estas características son excelentes tal y como están, pero si necesita ampliar y personalizar la telemetría, puede hacerlo.

Su instalación se realiza desde Visual Studio con unos pocos clics. Tiene la opción para evitar cargos. Para ello, solo debe limitar el volumen de la telemetría. Esta funcionalidad le permite probar y depurar un sitio con no muchos usuarios, o incluso supervisarlo. Si decide que desea supervisar su sitio de producción, no le costará trabajo aumentar el límite más adelante.

## <a name="prerequisites"></a>Prerrequisitos
Para agregar Application Insights al sitio web de ASP.NET, necesita:

- Instale [Visual Studio 2019 para Windows](https://www.visualstudio.com/downloads/) con las cargas de trabajo siguientes:
    - Desarrollo de ASP.NET y web (no desactive los componentes opcionales)
    - Desarrollo de Azure

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="step-1-add-the-application-insights-sdk"></a><a name="ide"></a> Paso 1: Agregue el SDK de Application Insights

> [!IMPORTANT]
> Las capturas de pantalla de este ejemplo están basadas en Visual Studio 2017, versión 15.9.9 y posteriores. La experiencia de adición de Application Insights varía en las versiones de Visual Studio, así como el tipo de plantilla de ASP.NET. Las versiones anteriores pueden tener texto alternativo, como "Configurar Application Insights".

Haga clic con el botón derecho en el nombre de la aplicación web en el Explorador de soluciones y seleccione **Agregar** > **Telemetría de Application Insights**.

![Captura de pantalla del Explorador de soluciones, con Configurar Application Insights resaltado](./media/asp-net/add-telemetry-new.png)

(En función de la versión del SDK de Application Insights que tenga, puede que se le solicite que actualice a la versión más reciente. Si se le solicita, seleccione **Actualizar SDK**.)

![Captura de pantalla: Hay una nueva versión disponible del SDK de Microsoft Application Insights. Actualización del SDK resaltado](./media/asp-net/0002-update-sdk.png)

Pantalla Configuración de Application Insights:

Seleccione **Comenzar**.

![Captura de pantalla de la página Registre la aplicación en Application Insights](./media/asp-net/00004-start-free.png)

Si desea establecer el grupo de recursos o la ubicación en que se almacenan los datos, haga clic en **Parámetros de configuración**. Los grupos de recursos se utilizan para controlar el acceso a los datos. Por ejemplo, si tiene varias aplicaciones que forman parte del mismo sistema, puede poner sus datos de Application Insights en el mismo grupo de recursos.

 Seleccione **Registrar**.

![Captura de pantalla de la página Registre la aplicación en Application Insights](./media/asp-net/00005-register-ed.png)

 Seleccione **Proyecto** > **Administrar paquetes NuGet** > **Origen de paquete: nuget.org** > Confirme que tiene la versión estable más reciente del SDK de Application Insights.

 La telemetría se enviará a [Azure Portal](https://portal.azure.com), durante la depuración y después de que se haya publicado la aplicación.
> [!NOTE]
> Si no desea enviar datos de telemetría al portal mientras se lleva a cabo una depuración, agregue el SDK de Application Insights a la aplicación, pero no configure ningún recurso en el portal. Durante la depuración puede ver la telemetría en Visual Studio. Posteriormente, puede volver a esta página de configuración, o bien puede esperar hasta que haya implementado la aplicación y [activar la telemetría en tiempo de ejecución](./status-monitor-v2-overview.md).

## <a name="step-2-run-your-app"></a><a name="run"></a> Paso 2: Ejecutar la aplicación
Ejecute la aplicación con F5. Abra distintas páginas para generar telemetría.

En Visual Studio, verá un recuento de los eventos que se han registrado.

![Captura de pantalla de Visual Studio. El botón Application Insights se muestra durante la depuración.](./media/asp-net/00006-Events.png)

## <a name="step-3-see-your-telemetry"></a>Paso 3: Visualización de los datos de telemetría
La telemetría se puede ver en Visual Studio o en el portal web de Application Insights. Busque telemetría en Visual Studio, ya que le ayudará a depurar la aplicación. Supervise el rendimiento y uso en el portal web cuando el sistema esté activo. 

### <a name="see-your-telemetry-in-visual-studio"></a>Visualización de datos de telemetría en Visual Studio

En Visual Studio, para ver los datos de Application Insights.  Seleccione **Explorador de soluciones** > **Servicios conectados** > haga clic con el botón derecho en **Application Insights**y, a continuación, haga clic en **Buscar telemetría activa**.

En la ventana de búsqueda de Application Insights de Visual Studio, puede ver los datos de la aplicación para la telemetría generada en el lado servidor de la aplicación. Experimente con los filtros y haga clic en cualquier evento para ver más información al respecto.

![Captura de pantalla de la vista de datos de la sesión de depuración en la ventana de Application Insights.](./media/asp-net/55.png)

> [!Tip]
> Si no ve datos, asegúrese de que el intervalo de tiempo es correcto y haga clic en el icono de búsqueda.

[Más información acerca de las herramientas de Application Insights en Visual Studio](./visual-studio.md).

<a name="monitor"></a>
### <a name="see-telemetry-in-web-portal"></a>Visualización de datos de telemetría en el portal web

La telemetría también se puede ver en el portal web de Application Insights (salvo que se elija instalar solo el SDK). El portal tiene más gráficos, herramientas de análisis y vistas de componentes que Visual Studio. El portal también ofrece alertas.

Abra el recurso de Application Insights. Inicie sesión en [Azure Portal](https://portal.azure.com/) y búsquelo allí o seleccione **Explorador de soluciones** > **Servicios conectados** > haga clic con el botón derecho en  **Application Insights** > **Abrir portal de Application Insights** y deje que le lleve allí.

El portal se abrirá en una vista de los datos de telemetría de su aplicación.

![Captura de pantalla de la página de información general de Application Insights](./media/asp-net/007.png)

En el portal, haga clic en cualquier icono para ver su contenido con mayor detalle.

## <a name="step-4-publish-your-app"></a>Paso 4: Publicación de la aplicación
Publique su aplicación en el servidor IIS o en Azure. Consulte [Secuencia de métricas en directo](./live-stream.md) para asegurarse de que todo está ejecutándose sin problemas.

La telemetría se crea en el portal de Application Insights, donde puede supervisar las métricas y buscar la telemetría. También puede usar el eficaz [lenguaje de consulta de Kusto](/azure/kusto/query/) para analizar el uso y rendimiento o para buscar eventos concretos.

También puede seguir analizando la telemetría en [Visual Studio](./visual-studio.md) con herramientas como búsqueda de diagnóstico y las [tendencias](./visual-studio-trends.md).

> [!NOTE]
> Si la aplicación envía suficiente telemetría que se acerque a las [limitaciones de peticiones](./pricing.md#limits-summary), se activará el [muestreo](./sampling.md) automático. El muestreo reduce la cantidad de datos de telemetría enviados desde su aplicación, a la vez que conserva los datos correlacionados para fines de diagnóstico.
>
>

## <a name="youre-all-set"></a><a name="land"></a>Ya está listo

Felicidades. Ha instalado el paquete de Application Insights en la aplicación y lo ha configurado para que envíe telemetría al servicio Application Insights en Azure.

El recurso de Azure que recibe la telemetría de la aplicación se identifica mediante una *clave de instrumentación*, que se encuentra en el archivo ApplicationInsights.config.


## <a name="upgrade-to-future-sdk-versions"></a>Actualización a futuras versiones del SDK

* [Notas de la versión](./release-notes.md)

Para actualizar a una nueva versión del SDK, abra el **Administrador de paquetes NuGet** y filtre por los paquetes instalados. Seleccione **Microsoft.ApplicationInsights.Web** y elija **Actualizar**.

Si ha hecho alguna personalización en ApplicationInsights.config, guarde una copia del mismo antes de realizar la actualización. Luego, combine los cambios en la nueva versión.

## <a name="next-steps"></a>Pasos siguientes

Si está interesado, puede examinar otros temas:

* [Instrumentación de aplicaciones web en tiempo de ejecución con Application Insights](./monitor-performance-live-website-now.md)
* [Azure Cloud Services](./cloudservices.md)

### <a name="more-telemetry"></a>Más telemetría

* **[Datos de carga de página y explorador](./javascript.md)** : inserte un fragmento de código en las páginas web.
* **[Obtenga una supervisión más detallada de las dependencias y excepciones](./monitor-performance-live-website-now.md)** : instale el Monitor de estado en el servidor.
* **[Codifique eventos personalizados](./api-custom-events-metrics.md)** para contar, cronometrar y medir las acciones de los usuarios.
* **[Obtenga datos del registro](./asp-net-trace-logs.md)** : ponga en correlación los datos del registro con la telemetría.

### <a name="analysis"></a>Análisis

* **[Trabajo con Application Insights en Visual Studio](./visual-studio.md)**<br/>Incluye información acerca de la depuración con telemetría, la búsqueda de diagnóstico y la profundización en el código.
* **[Analytics](../log-query/get-started-portal.md)** : eficaz lenguaje de consulta.

### <a name="alerts"></a>Alertas

* [Pruebas de disponibilidad](./monitor-web-app-availability.md): cree pruebas para asegurarse de que el sitio sea visible en la Web.
* [Diagnósticos inteligentes](./proactive-diagnostics.md): estas pruebas se realizan automáticamente, por lo que no es preciso hacer nada para configurarlas. Le indican si la aplicación tiene una tasa de solicitudes con error inusual.
* [Alertas de métricas](../platform/alerts-log.md): Establezca alertas que le adviertan si una métrica supera un umbral. Puede establecerlas en las métricas personalizadas que codifique en la aplicación.

### <a name="automation"></a>Automation

* [Creación de recursos de Application Insights mediante PowerShell](./powershell.md)

