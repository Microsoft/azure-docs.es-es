---
title: 'Inicio rápido: Supervisión de sitios web con Azure Monitor Application Insights'
description: En este inicio rápido aprenderá a configurar la supervisión de sitios web en el lado cliente/explorador con Azure Monitor Application Insights.
ms.topic: quickstart
ms.date: 07/15/2019
ms.custom: mvc
ms.openlocfilehash: eb6c0a8319257949cee8f35be6cdfac22f1fe5d4
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87323441"
---
# <a name="quickstart-start-monitoring-your-website-with-azure-monitor-application-insights"></a>Inicio rápido: Empezar a supervisar un sitio web con Azure Monitor Application Insights

En este inicio rápido, aprenderá a agregar el SDK de JavaScript para Application Insights de código abierto a un sitio web. También aprenderá a conocer mejor la experiencia de lado cliente/explorador de los visitantes de su sitio web.

Con Application Insights de Azure Monitor puede supervisar fácilmente la disponibilidad, el rendimiento y el uso del sitio web. También puede identificar y diagnosticar errores en la aplicación rápidamente sin tener que esperar a que un usuario informe de ellos. Application Insights proporciona funcionalidades de supervisión tanto del lado servidor como del lado cliente/explorador.

## <a name="prerequisites"></a>Prerrequisitos

* Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
* Un sitio web al que puede agregar el SDK de JavaScript para Application Insights.

## <a name="enable-application-insights"></a>Habilitación de Application Insights

Application Insights puede recopilar datos de telemetría desde cualquier aplicación conectada a Internet que se ejecute tanto en la nube como en un entorno local. Para ver estos datos, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
1. Seleccione **Crear un recurso** > **Herramientas de administración** > **Application Insights**.

   > [!NOTE]
   >Si es la primera vez que crea un recurso de Application Insights, consulte [Creación de recursos en Application Insights](./create-new-resource.md).
1. Cuando aparezca el cuadro de configuración, use la tabla siguiente para completar los campos de entrada:

    | Configuración        | Value           | Descripción  |
   | ------------- |:-------------|:-----|
   | **Nombre**      | Nombre único global | El nombre que identifica la aplicación que está supervisando. |
   | **Grupo de recursos**     | myResourceGroup      | El nombre del nuevo grupo de recursos que hospeda los datos de Application Insights. puede crear un grupo de recursos o use uno existente. |
   | **Ubicación** | Este de EE. UU. | Elija una ubicación cerca de usted o de donde se hospeda la aplicación. |
1. Seleccione **Crear**.

## <a name="create-an-html-file"></a>Creación de un archivo HTML

1. En el equipo local, cree un archivo llamado ``hello_world.html``. En este ejemplo, cree el archivo en la raíz de la unidad C, así que debe ser algo como ``C:\hello_world.html``.
1. Copie y pegue el siguiente script en ``hello_world.html``:

    ```html
    <!DOCTYPE html>
    <html>
    <head>
    <title>Azure Monitor Application Insights</title>
    </head>
    <body>
    <h1>Azure Monitor Application Insights Hello World!</h1>
    <p>You can use the Application Insights JavaScript SDK to perform client/browser-side monitoring of your website. To learn about more advanced JavaScript SDK configurations, visit the <a href="https://github.com/Microsoft/ApplicationInsights-JS/blob/master/API-reference.md" title="API Reference">API reference</a>.</p>
    </body>
    </html>
    ```

## <a name="configure-application-insights-sdk"></a>Configuración del SDK de Application Insights

1. Seleccione **Información general** > **Información esencial** y copie la **clave de instrumentación** de la aplicación.

   ![Formulario Nuevo recurso de Application Insights](media/website-monitoring/instrumentation-key-001.png)

1. Agregue el siguiente script al archivo ``hello_world.html`` antes de la etiqueta de cierre ``</head>``:

   ```javascript
   <script type="text/javascript">
      var sdkInstance="appInsightsSDK";window[sdkInstance]="appInsights";var aiName=window[sdkInstance],aisdk=window[aiName]||function(e){function n(e){t[e]=function(){var n=arguments;t.queue.push(function(){t[e].apply(t,n)})}}var t={config:e};t.initialize=!0;var i=document,a=window;setTimeout(function(){var n=i.createElement("script");n.src=e.url||"https://az416426.vo.msecnd.net/scripts/b/ai.2.min.js",i.getElementsByTagName("script")[0].parentNode.appendChild(n)});try{t.cookie=i.cookie}catch(e){}t.queue=[],t.version=2;for(var r=["Event","PageView","Exception","Trace","DependencyData","Metric","PageViewPerformance"];r.length;)n("track"+r.pop());n("startTrackPage"),n("stopTrackPage");var s="Track"+r[0];if(n("start"+s),n("stop"+s),n("setAuthenticatedUserContext"),n("clearAuthenticatedUserContext"),n("flush"),!(!0===e.disableExceptionTracking||e.extensionConfig&&e.extensionConfig.ApplicationInsightsAnalytics&&!0===e.extensionConfig.ApplicationInsightsAnalytics.disableExceptionTracking)){n("_"+(r="onerror"));var o=a[r];a[r]=function(e,n,i,a,s){var c=o&&o(e,n,i,a,s);return!0!==c&&t["_"+r]({message:e,url:n,lineNumber:i,columnNumber:a,error:s}),c},e.autoExceptionInstrumented=!0}return t}(
      {
         instrumentationKey:"INSTRUMENTATION_KEY"
      }
      );window[aiName]=aisdk,aisdk.queue&&0===aisdk.queue.length&&aisdk.trackPageView({});
   </script>
   ```

1. Modifique ``hello_world.html`` y agregue la clave de instrumentación.

1. Abra ``hello_world.html`` en una sesión de explorador local. Esta acción crea una vista de página individual. Puede actualizar el explorador para generar varias vistas de páginas de prueba.

## <a name="monitor-your-website-in-the-azure-portal"></a>Supervisión de un sitio web en Azure Portal

1. Vuelva a abrir la página de **información general** de Application Insights en Azure Portal para ver los detalles de la aplicación en ejecución. La página de **información general** es donde ha recuperado la clave de instrumentación.

   Los cuatro gráficos predeterminados de la página de introducción se extienden a los datos de la aplicación del lado servidor. Dado que vamos a instrumentar las interacciones del lado cliente/explorador con el SDK de JavaScript, esta vista concreta no se aplica, salvo que también tengamos instalado un SDK del lado servidor.

1. Seleccione **Analytics** ![Icono de mapa de aplicación](media/website-monitoring/006.png).  Esta acción abre **Analytics**, que proporciona un lenguaje de consulta avanzado para analizar todos los datos recopilados por Application Insights. Para ver los datos relacionados con las solicitudes de explorador del lado cliente, ejecute la siguiente consulta:

    ```kusto
    // average pageView duration by name
    let timeGrain=1s;
    let dataset=pageViews
    // additional filters can be applied here
    | where timestamp > ago(15m)
    | where client_Type == "Browser" ;
    // calculate average pageView duration for all pageViews
    dataset
    | summarize avg(duration) by bin(timestamp, timeGrain)
    | extend pageView='Overall'
    // render result in a chart
    | render timechart
    ```

   ![Gráfico de Analytics con las solicitudes de usuario durante un período de tiempo](./media/website-monitoring/analytics-query.png)

1. Vuelva a la página **Introducción**. En el encabezado **Investigar**, seleccione **Explorador**y, después, seleccione **Rendimiento**.  Aparecen las métricas relacionadas con el rendimiento de su sitio web. Hay una vista correspondiente para analizar los errores y excepciones del sitio web. Puede seleccionar **Ejemplos** para acceder a los [detalles de la transacción de un extremo a otro](./transaction-diagnostics.md).

   ![Gráfico de las métricas del servidor](./media/website-monitoring/browser-performance.png)

1. En el menú principal de Application Insights, en el encabezado **Uso**, seleccione [**Usuarios**](./usage-segmentation.md) para empezar a explorar las [herramientas de análisis del comportamiento del usuario](./usage-overview.md). Dado que las pruebas se realizan desde una sola máquina, solo se verán los datos de un usuario. En el caso de un sitio web en vivo, la distribución de los usuarios podría ser como esta:

     ![Gráfico de usuario](./media/website-monitoring/usage-users.png)

1. En el caso de un sitio web más complejo con varias páginas, puede usar la herramienta [**Flujos de usuario**](./usage-flows.md) para realizar el seguimiento del recorrido que realizan los visitantes en las distintas partes del sitio web.

   ![Visualización de flujos de usuario](./media/website-monitoring/user-flows.png)

Para más información acerca de configuraciones avanzadas para supervisar sitios web, consulte la [referencia a JavaScript SDK API](https://github.com/Microsoft/ApplicationInsights-JS/blob/master/API-reference.md).

## <a name="clean-up-resources"></a>Limpieza de recursos

Si planea seguir trabajando con otros inicios rápidos o tutoriales, no limpie los recursos creados en este inicio rápido. De lo contrario, realice los siguientes pasos para eliminarlos en Azure Portal.

> [!NOTE]
> Si ha usado un grupo de recursos existente, las siguientes instrucciones no funcionarán. En su lugar, puede eliminar el recurso de Application Insights individual. Tenga esto en cuenta que cuando se elimina un grupo de recursos, se eliminan también todos los recursos subyacentes que sean miembros del grupo.

1. En el menú de la izquierda de Azure Portal, seleccione **Grupos de recursos** y luego **myResourceGroup**, o el nombre de un grupo de recursos temporal.
1. En la página del grupo de recursos, seleccione **Eliminar**, escriba **myResourceGroup** en el cuadro de texto y, después, seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Búsqueda y diagnóstico de problemas de rendimiento](../log-query/log-query-overview.md)

