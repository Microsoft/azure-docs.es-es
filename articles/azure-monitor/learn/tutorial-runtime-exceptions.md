---
title: Diagnóstico de problemas en tiempo de ejecución mediante Azure Application Insights | Microsoft Docs
description: Tutorial para buscar y diagnosticar excepciones en tiempo de ejecución en un aplicación mediante Azure Application Insights.
ms.subservice: application-insights
ms.topic: tutorial
author: mrbullwinkle
ms.author: mbullwin
ms.date: 09/19/2017
ms.custom: mvc
ms.openlocfilehash: b656774762f5930c5665166a701bda7333940307
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87322438"
---
# <a name="find-and-diagnose-run-time-exceptions-with-azure-application-insights"></a>Búsqueda y diagnóstico de excepciones en tiempo de ejecución con Azure Application Insights

Azure Application Insights recopila datos de telemetría de cualquier aplicación para ayudarle a identificar y diagnosticar excepciones en tiempo de ejecución.  Este tutorial le guiará por este proceso en la aplicación.  Aprenderá a:

> [!div class="checklist"]
> * Modifica el proyecto para habilitar el seguimiento de excepciones
> * Identificar excepciones en los distintos componentes de una aplicación
> * Ver los detalles de una excepción
> * Descargar una instantánea de la excepción en Visual Studio para su depuración
> * Analizar los detalles de las solicitudes con errores mediante un lenguaje de consulta
> * Crear un nuevo elemento de trabajo para corregir el código con errores


## <a name="prerequisites"></a>Prerequisites

Para completar este tutorial:

- Instale [Visual Studio 2019](https://www.visualstudio.com/downloads/) con las cargas de trabajo siguientes:
    - ASP.NET y desarrollo web
    - Desarrollo de Azure
- Descargue e instale [Visual Studio Snapshot Debugger](https://aka.ms/snapshotdebugger).
- Habilite [Visual Studio Snapshot Debugger](../app/snapshot-debugger.md).
- Implemente una aplicación de .NET en Azure y [habilite el SDK de Application Insights](../app/asp-net.md). 
- El tutorial realiza un seguimiento de la identificación de una excepción en la aplicación, así que modifique el código en los entornos de desarrollo o prueba para generar una excepción. 

## <a name="log-in-to-azure"></a>Inicio de sesión en Azure
Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).


## <a name="analyze-failures"></a>Análisis de errores
Application Insights recopila los errores de la aplicación y permite ver su frecuencia en las diferentes operaciones, lo que le ayudarán a centrar sus esfuerzos en los que tienen el mayor impacto.  Más adelante puede profundizar en los detalles de dichos errores para identificar la causa principal.   

1. Seleccione **Application Insights** y, después, la suscripción.  
2. Para abrir el panel **Errores**, seleccione **Errores** en el menú **Investigar** o haga clic en el grafo **Solicitudes con errores**.

    ![Error en las solicitudes](media/tutorial-runtime-exceptions/failed-requests.png)

3. El panel **Solicitudes con errores** muestra el número de solicitudes con errores y el número de usuarios afectados para cada operación en la aplicación.  Si ordenar esta información por usuario, puede identificar los errores que más afectan a los usuarios.  En este ejemplo, **GET Employees/Create** y **GET Customers/Details** son probables candidatos a los que investigar debido al gran número de errores y a la gran cantidad de usuarios afectados.  Al seleccionar cualquier operación, se muestra más información acerca de la misma en el panel derecho.

    ![Panel Solicitudes con errores](media/tutorial-runtime-exceptions/failed-requests-blade.png)

4. Reduzca la ventana de tiempo para acercar el periodo en el que la frecuencia de errores muestra un pico.

    ![Ventana Solicitudes con errores](media/tutorial-runtime-exceptions/failed-requests-window.png)

5. Consulte los ejemplos relacionados haciendo clic en el botón con el número de resultados filtrados. Los ejemplos "sugeridos" tienen telemetría relacionada de todos los componentes, incluso si se ha aplicado el muestreo en cualquiera de ellos. Haga clic en un resultado de búsqueda para ver los detalles del error.

    ![Ejemplos de solicitudes con errores](media/tutorial-runtime-exceptions/failed-requests-search.png)

6. Los detalles de la solicitud con errores muestran el gráfico de Gantt que indica que había dos errores de dependencia en esta transacción, que también se atribuyen a más del 50 % de la duración total de la transacción. Esta experiencia presenta toda la telemetría, a través de los componentes de una aplicación distribuida que están relacionados con este identificador de operación. [Más información acerca de la nueva experiencia](../app/transaction-diagnostics.md). Puede seleccionar cualquiera de los elementos para ver sus detalles en el lado derecho. 

    ![Detalles de las solicitudes con errores](media/tutorial-runtime-exceptions/failed-request-details.png)

7. Los detalles de las operaciones también muestran la excepción FormatException, que parece que es lo que ha generado el error.  Puede ver que se debe a un código postal no válido. Puede abrir la instantánea de depuración para ver información de depuración de nivel de código en Visual Studio.

    ![Detalles de la excepción](media/tutorial-runtime-exceptions/failed-requests-exception.png)

## <a name="identify-failing-code"></a>Identificación de código con errores
Snapshot Debugger recopila instantáneas de las excepciones que se producen con más frecuencia en la aplicación para ayudarle a diagnosticar su causa principal en producción.  Puede ver las instantáneas de depuración en el portal para examinar la pila de llamadas e inspeccionar las variables en cada marco de pila de llamadas. Posteriormente, puede depurar el código fuente descargando la instantánea y abriéndola en Visual Studio 2019 Enterprise.

1. En las propiedades de la excepción, haga clic en **Abrir instantánea de depuración**.
2. Se abre el panel **Depurar instantánea** con la pila de llamadas de la solicitud.  Haga clic en cualquiera de los métodos para ver los valores de todas las variables locales en el momento de la solicitud.  Si empezamos por el primero de los métodos de este ejemplo, podemos ver variables locales que no tienen ningún valor.

    ![Depurar instantánea](media/tutorial-runtime-exceptions/debug-snapshot-01.png)

3. La primera llamada que tiene valores válidos es **ValidZipCode** y podemos ver que se especificó un código postal con letras que no se puede convertir en un número entero.  Parece que este es el error que hay en el código y que debe corregirse.

    ![Depurar instantánea](media/tutorial-runtime-exceptions/debug-snapshot-02.png)

4. Puede descargar esta instantánea en Visual Studio, donde se puede buscar el código que se debe corregir. Para ello, haga clic en **Descargar instantánea**.
5. La instantánea se carga en Visual Studio.
6. A partir de ese momento se puede ejecutar una sesión de depuración en Visual Studio Enterprise que identifique rápidamente la línea de código que provocó la excepción.

    ![Excepción en el código](media/tutorial-runtime-exceptions/exception-code.png)


## <a name="use-analytics-data"></a>Uso de datos de análisis
Todos los datos que recopila Application Insights se almacenan en Azure Log Analytics, lo que proporciona un lenguaje de consulta completo que permite analizar los datos de varias formas.  Dichos datos se pueden usar para analizar las solicitudes que generaron la excepción que investigamos. 

1. Haga clic en la información de CodeLens que encontrará encima del código para ver los datos de telemetría que proporciona Application Insights.

    ![Código](media/tutorial-runtime-exceptions/codelens.png)

1. Haga clic en **Analizar impacto** para abrir Application Insights Analytics.  Contiene varias consultas que proporcionan detalles acerca de las solicitudes con errores, como los usuarios afectados, los exploradores usados y las regiones.<br><br>![Analytics](media/tutorial-runtime-exceptions/analytics.png)<br>

## <a name="add-work-item"></a>Incorporación de elemento de trabajo
Si conecta Application Insights a un sistema de seguimiento, como GitHub y Azure DevOps, puede crear un elemento de trabajo directamente desde Application Insights.

1. Vuelva al panel **Exception Properties** (Propiedades de la excepción) de Application Insights.
2. Haga clic en **Nuevo elemento de trabajo**.
3. Se abre el panel **Nuevo elemento de trabajo** con detalles acerca la excepción ya rellenados.  Además, antes de guardarlo puede agregar toda la información adicional que desee.

    ![Nuevo elemento de trabajo](media/tutorial-runtime-exceptions/new-work-item.png)

## <a name="next-steps"></a>Pasos siguientes
Ahora que ha aprendido a identificar las excepciones en tiempo de ejecución, pase al siguiente tutorial, donde aprenderá a identificar y diagnosticar problemas de rendimiento.

> [!div class="nextstepaction"]
> [Identificar problemas de rendimiento](./tutorial-performance.md)

