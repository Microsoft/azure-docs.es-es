---
title: Diagnóstico de problemas de rendimiento mediante Azure Application Insights | Microsoft Docs
description: Tutorial para buscar y diagnosticar problemas de rendimiento en un aplicación mediante Azure Application Insights.
ms.subservice: application-insights
ms.topic: tutorial
author: mrbullwinkle
ms.author: mbullwin
ms.date: 06/15/2020
ms.custom: mvc
ms.openlocfilehash: 3c7185e07190895dfcc97555c6603049ed41c18c
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87322506"
---
# <a name="find-and-diagnose-performance-issues-with-azure-application-insights"></a>Búsqueda y diagnóstico de problemas de rendimiento con Azure Application Insights

Azure Application Insights recopila datos de telemetría de cualquier aplicación para ayudarle a analizar su funcionamiento y rendimiento.  Esta información se puede usar para identificar los problemas que se pueden producir o las mejoras en la aplicación que más afectarían a los usuarios.  En este tutorial se recorre todo el proceso de análisis del rendimiento tanto de los componentes de servidor de una aplicación como de la perspectiva del cliente.  Aprenderá a:

> [!div class="checklist"]
> * Identificar el rendimiento de las operaciones del servidor
> * Analizar las operaciones del servidor para determinar la causa principal de un bajo rendimiento
> * Identificar las operaciones del cliente más lentas
> * Analizar los detalles de las vistas de página mediante el lenguaje de consulta


## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial:

- Instale [Visual Studio 2019](https://www.visualstudio.com/downloads/) con las cargas de trabajo siguientes:
    - ASP.NET y desarrollo web
    - Desarrollo de Azure
- Implemente una aplicación de .NET en Azure y [habilite el SDK de Application Insights](../app/asp-net.md).
- [Habilite el generador de perfiles de Application Insights](../app/profiler.md#installation) de la aplicación.

## <a name="log-in-to-azure"></a>Inicio de sesión en Azure
Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

## <a name="identify-slow-server-operations"></a>Identificación de operaciones lentas del servidor
Application Insights recopila datos de rendimiento de las distintas operaciones en la aplicación. Mediante la identificación de las operaciones que más tardan en completarse, es posible diagnosticar problemas potenciales o enfocar mejor el desarrollo en curso para mejorar el rendimiento general de la aplicación.

1. Seleccione **Application Insights** y, a continuación, seleccione la suscripción.  
1. Para abrir el panel **Rendimiento**, seleccione **Rendimiento** en el menú **Investigar** o haga clic en el grafo **Tiempo de respuesta del servidor**.

    ![Rendimiento](media/tutorial-performance/1-overview.png)

2. El panel **Rendimiento** muestra el número de operaciones de la aplicación y la duración media de cada una de ellas.  Esta información se puede usar para identificar las operaciones que más afectan a los usuarios. En este ejemplo, **GET Customers/Details** y **GET Home/Index** son probables candidatos a los que investigar, ya que su duración es relativamente alta y se las llama en numerosas ocasiones.  Otras operaciones pueden tener una duración superior, pero rara vez se les llama, por lo que el efecto de su mejora sería mínimo.  

    ![Panel del servidor de rendimiento](media/tutorial-performance/2-server-operations.png)

3. Actualmente, el grafo muestra la duración media de las operaciones a lo largo del tiempo. Puede cambiar al percentil 95 para detectar los problemas de rendimiento. Para agregar las operaciones que le interesen, ánclelas al grafo.  Esto muestra que hay algunos valores máximos que merece la pena investigar.  Y para aislarlos aún más, puede reducir la ventana de tiempo del grafo.

    ![Anclar operaciones](media/tutorial-performance/3-server-operations-95th.png)

4.  El panel de rendimiento de la derecha muestra la distribución de duraciones de distintas solicitudes para la operación seleccionada.  Reduzca la ventana para empezar alrededor del percentil 95. La tarjeta de información "3 dependencias principales" puede indicarle de un vistazo que las dependencias externas probablemente estén coadyuvando a que se produzcan las transacciones lentas.  Haga clic en el botón con el número de ejemplos para ver una lista de los ejemplos. A continuación, puede seleccionar cualquier ejemplo para ver los detalles de transacción.

5.  Puede ver a simple vista que la llamada a la tabla de Azure Fabrikamaccount es la que más contribuye a la duración total de la transacción. También puede ver que una excepción ha provocado un error. Puede hacer clic en cualquier elemento de la lista para ver sus detalles en el lado derecho. [Más información acerca de la experiencia de diagnósticos de transacción](../app/transaction-diagnostics.md)

    ![Detalles de la operación de un extremo a otro](media/tutorial-performance/4-end-to-end.png)
    

6.  [**Profiler**](../app/profiler-overview.md) ayuda a profundizar en los diagnósticos de nivel de código al mostrar el código real que se ejecutó para la operación y el tiempo necesario para cada paso. Es posible que no se pueda realizar un seguimiento de algunas operaciones, ya que el generador de perfiles se ejecuta periódicamente.  Con el tiempo, más operaciones deben tener seguimientos.  Para iniciar el generador de perfiles de la operación, haga clic en **Seguimientos de Profiler**.
5.  El seguimiento muestra los eventos individuales de cada operación, por lo que puede diagnosticar la causa principal por la duración de la operación global.  Haga clic en uno de los ejemplos superiores, que son los que tienen una duración mayor.
6.  Haga clic en **Ruta de acceso activa** para resaltar la ruta de acceso específica de los eventos que más contribuyen a la duración total de la operación.  En este ejemplo, puede ver que la llamada más lenta procede del método *FabrikamFiberAzureStorage.GetStorageTableData*. La parte que tiene la mayor parte del tiempo es el método *CloudTable.CreateIfNotExist*. Si esta línea de código se ejecuta cada vez que se llama a la función, se consumirán recursos de CPU y llamadas de red innecesarios. La mejor manera de corregir el código es poner esta línea en algún método de inicio que solo se ejecute una vez.

    ![Detalles de Profiler](media/tutorial-performance/5-hot-path.png)

7.  La sugerencia que se muestra en **Consejo de rendimiento**, en la parte superior de la pantalla, es compatible con la evaluación de que la duración excesiva se debe a la espera.  Haga clic en el vínculo de **espera** para obtener documentación acerca de cómo interpretar los diferentes tipos de eventos.

    ![Consejo de rendimiento](media/tutorial-performance/6-perf-tip.png)

8.   Para un análisis más profundo, haga clic en **Descargar seguimiento** para descargar el seguimiento. Puede ver estos datos mediante [PerfView](https://github.com/Microsoft/perfview#perfview-overview).

## <a name="use-logs-data-for-server"></a>Uso de datos de registros para el servidor
 Los registros proporcionan un lenguaje de consulta completo que le permite analizar todos los datos recopilados por Application Insights. Con ello se puede realizar un análisis profundo de los datos de solicitud y rendimiento.

1. Vuelva al panel de detalles de la operación y haga clic en el ![icono Registros](media/tutorial-performance/app-viewinlogs-icon.png)**Ver en registros (Analytics)**

2. Los registros se abren con una consulta en cada una de las vistas del panel.  Dichas consultas se pueden ejecutar tal cual, o bien se pueden modificar para ajustarlas a los requisitos.  La primera consulta muestra la duración de esta operación a lo largo del tiempo.

    ![consulta de registros](media/tutorial-performance/7-request-time-logs.png)


## <a name="identify-slow-client-operations"></a>Identificación de operaciones lentas del cliente
Además de identificar los procesos del servidor que se deben optimizar, Application Insights puede analizar la perspectiva de los exploradores cliente.  Esto puede ayudarle a identificar posibles mejoras en los componentes de cliente, e incluso identificar problemas con distintos exploradores o ubicaciones diferentes.

1. Seleccione **Explorador** en **Investigar** y, después, haga clic en **Rendimiento del explorador** o seleccione **Rendimiento** en **Investigar** y cambie a la pestaña **Explorador**; para ello, haga clic en el botón de alternancia servidor/explorador situado en la parte superior derecha para abrir el resumen del rendimiento del explorador. Esto proporciona un resumen visual de los distintos datos de telemetría de la aplicación desde la perspectiva del explorador.

    ![Resumen de explorador](media/tutorial-performance/8-browser.png)

2. Seleccione uno de los nombres de la operación y, después, haga clic en el botón de ejemplos azul en la parte inferior derecha y seleccione una operación. Se mostrarán los detalles de la transacción completa y, en el lado derecho, podrá ver las **propiedades de la vista de página**. Esto te permite ver los detalles del cliente que solicita la página incluido, lo que incluye el tipo de explorador y su ubicación. Esta información puede ayudarle a determinar si ciertos tipos de clientes tienen problemas de rendimiento.

    ![Vista de página](media/tutorial-performance/9-page-view-properties.png)

## <a name="use-logs-data-for-client"></a>Uso de datos de registros para el cliente
Al igual que los datos recopilados relativos al rendimiento del servidor, Application Insights hace que todos los datos de cliente estén disponibles para un análisis profundo mediante Registros.

1. Vuelva al resumen del explorador y haga clic en el icono ![Registros](media/tutorial-performance/app-viewinlogs-icon.png) **Ver en Registros (Analytics)**

2. Los registros se abren con una consulta en cada una de las vistas del panel. La primera consulta muestra la duración de distintas vistas de la página a lo largo del tiempo.

    ![consulta de registros](media/tutorial-performance/10-page-view-logs.png)

3.  Smart Diagnostics es una característica de Registros que identifica patrones únicos en los datos. Al hacer clic en el punto de Smart Diagnostics en el gráfico de líneas, la misma consulta se ejecuta sin los registros que causaron las anomalías. Los detalles de dichos registros se muestran en la sección de comentarios de consulta para que pueda identificar las propiedades de las vistas de página que provocan que la duración sea excesiva.

    ![Registros con Smart Diagnostics](media/tutorial-performance/11-page-view-logs-dsmart.png)


## <a name="next-steps"></a>Pasos siguientes
Ahora que ha aprendido a identificar las excepciones en tiempo de ejecución, pase al siguiente tutorial, donde aprenderá a crear alertas en respuesta a errores.

> [!div class="nextstepaction"]
> [Alerta por el estado de una aplicación](./tutorial-alert.md)

