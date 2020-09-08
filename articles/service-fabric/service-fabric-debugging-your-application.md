---
title: Depuración de la aplicación en Visual Studio
description: Mejore la confiabilidad y el rendimiento de los servicios mediante su desarrollo y depuración en Visual Studio en un clúster de desarrollo local.
author: vturecek
ms.topic: conceptual
ms.date: 11/02/2017
ms.author: vturecek
ms.custom: devx-track-csharp
ms.openlocfilehash: 2bec6667ea44e195ea1fccb5313b45ed060312a1
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89000114"
---
# <a name="debug-your-service-fabric-application-by-using-visual-studio"></a>Depurar la aplicación de Service Fabric con Visual Studio
> [!div class="op_single_selector"]
> * [Visual Studio/CSharp](service-fabric-debugging-your-application.md) 
> * [Eclipse/Java](service-fabric-debugging-your-application-java.md)
>


## <a name="debug-a-local-service-fabric-application"></a>Depurar una aplicación de Service Fabric local
Puede ahorrar tiempo y dinero implementando y depurando su aplicación de Service Fabric de Azure en un clúster de desarrollo del equipo local. Visual Studio 2019 o 2015 pueden implementar la aplicación en el clúster local y conectar automáticamente el depurador a todas las instancias de la aplicación. Visual Studio se debe ejecutar como administrador para conectarse al depurador.

1. Inicie un clúster de desarrollo local siguiendo los pasos descritos en [Configurar el entorno de desarrollo de Service Fabric](service-fabric-get-started.md).
2. Presione **F5** o haga clic en **Depurar** > **Iniciar depuración**.
   
    ![Empezar a depurar una aplicación][startdebugging]
3. Establezca puntos de interrupción en el código y recorra la aplicación haciendo clic en los comandos del menú **Depurar** .
   
   > [!NOTE]
   > Visual Studio se conecta a todas las instancias de su aplicación. Al recorrer el código, se pueden alcanzar los puntos de interrupción por varios procesos, lo que da lugar a sesiones simultáneas. Intente deshabilitar los puntos de interrupción después de que se alcancen, haciendo que el punto de interrupción sea condicional en el id. del subproceso o use los eventos de diagnóstico.
   > 
   > 
4. La ventana **Eventos de diagnóstico** se abrirá automáticamente para que pueda ver los eventos de diagnóstico en tiempo real.
   
    ![Ver eventos de diagnóstico en tiempo real][diagnosticevents]
5. También puede abrir la ventana **Eventos de diagnóstico** en Cloud Explorer.  En **Service Fabric**, haga clic con el botón derecho en cualquier nodo y elija **Ver seguimientos en streaming**.
   
    ![Abrir la ventana de eventos de diagnóstico][viewdiagnosticevents]
   
    Si quiere filtrar los seguimientos para mostrar una aplicación o un servicio específicos, habilite los seguimientos en streaming en ellos.
6. Los eventos de diagnóstico se pueden ver en el archivo **ServiceEventSource.cs** generado automáticamente y se llaman desde el código de la aplicación.
   
    ```csharp
    ServiceEventSource.Current.ServiceMessage(this, "My ServiceMessage with a parameter {0}", result.Value.ToString());
    ```
7. La ventana **Eventos de diagnóstico** admite filtrado, pausas y la inspección de eventos en tiempo real.  El filtro es una búsqueda de cadena simple del mensaje de evento, incluido su contenido.
   
    ![Filtrar, pausar y reanudar, o inspeccionar eventos en tiempo real][diagnosticeventsactions]
8. Depurar servicios es parecido a depurar cualquier otra aplicación. Para facilitar el proceso, lo normal es establecer puntos de interrupción a través de Visual Studio. Aunque las instancias de Reliable Collections se replican en varios nodos, siguen implementando IEnumerable. Esta implementación significa que puede usar la vista de resultados en Visual Studio durante la depuración para ver lo que ha almacenado dentro. Para ello, establezca un punto de interrupción en el código.
   
    ![Empezar a depurar una aplicación][breakpoint]


### <a name="running-a-script-as-part-of-debugging"></a>Ejecutar un script como parte de la depuración
En determinados escenarios, puede que tenga que ejecutar un script como parte del inicio de una sesión de depuración (por ejemplo, cuando no usa los servicios predeterminados).

En Visual Studio, puede agregar un archivo denominado **Start-Service.ps1** en la carpeta **Scripts** del proyecto Aplicación de Service Fabric (.sfproj). Se invocará este script una vez que se haya creado la aplicación en el clúster local.


<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->

## <a name="debug-a-remote-service-fabric-application"></a>Depurar una aplicación de Service Fabric remota
Si las aplicaciones de Service Fabric se están ejecutando en un clúster de Service Fabric de Azure, puede depurarlas de forma remota directamente desde Visual Studio.

> [!NOTE]
> La característica requiere [Service Fabric SDK 2.0](https://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-VS2015) y el [SDK de Azure para .NET 2.9](https://azure.microsoft.com/downloads/).    

<!-- -->
> [!WARNING]
> La depuración remota está diseñada para escenarios de desarrollo y pruebas y no para su uso en entornos de producción, debido al impacto en las aplicaciones en ejecución.

1. Vaya al clúster en **Cloud Explorer**. Haga clic con el botón derecho y elija **Habilitar depuración**.
   
    ![Habilitar la depuración remota][enableremotedebugging]
   
    Esta acción iniciará el proceso de habilitar la extensión de depuración remota en los nodos del clúster y las configuraciones de red necesarias.
2. Haga clic con el botón derecho en el nodo de clúster en **Cloud Explorer** y elija **Adjuntar depurador**
   
    ![Asociar depurador][attachdebugger]
3. En el cuadro de diálogo **Asociar al proceso**, elija el proceso que quiere depurar y haga clic en **Adjuntar**
   
    ![Elegir proceso][chooseprocess]
   
    El nombre del proceso que desea asociar equivale al nombre del ensamblado del proyecto de servicio.
   
    El depurador se asociará a todos los nodos que ejecutan el proceso.
   
   * En caso de que vaya a depurar un servicio sin estado, todas las instancias del servicio en todos los nodos forman parte de la sesión de depuración.
   * Si va a depurar un servicio con estado, solo la réplica principal de cualquier partición estará activa y, por tanto, será la que capture el depurador. Si se mueve la réplica principal durante la sesión de depuración, el procesamiento de esa réplica todavía será parte de la sesión de depuración.
   * Para capturar solo las particiones o instancias pertinentes de un servicio determinado, puede utilizar puntos de interrupción condicionales para interrumpir solo una instancia o una partición específica.
     
     ![Punto de interrupción condicional][conditionalbreakpoint]
     
     > [!NOTE]
     > Actualmente no se admite la depuración de un clúster de Service Fabric con varias instancias del mismo nombre ejecutable de servicio.
     > 
     > 
4. Cuando termine de depurar la aplicación, puede deshabilitar la extensión de depuración remota; para ello, haga clic con el botón derecho en el clúster en **Cloud Explorer** y elija **Deshabilitar depuración**
   
    ![Deshabilitar la depuración remota][disableremotedebugging]

## <a name="streaming-traces-from-a-remote-cluster-node"></a>Seguimientos en streaming desde un nodo de clúster remoto
También puede trasmitir seguimientos directamente desde un nodo de clúster remoto a Visual Studio. Esta característica permite transmitir eventos de seguimiento ETW, generados en un nodo de clúster de Service Fabric.

> [!NOTE]
> Esta característica requiere [Service Fabric SDK 2.0](https://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-VS2015) y el [SDK de Azure para .NET 2.9](https://azure.microsoft.com/downloads/).
> Esta característica solo es compatible con clústeres que se ejecutan en Azure.
> 
> 

<!-- -->
> [!WARNING]
> Los seguimientos en streaming están concebidos para escenarios de desarrollo y pruebas y no para su uso en entornos de producción, debido al impacto en las aplicaciones en ejecución.
> En un escenario de producción, debe depender del reenvío de eventos mediante Diagnósticos de Azure.

1. Vaya al clúster en **Cloud Explorer**. Haga clic con el botón derecho y elija **Habilitar seguimientos en streaming**.
   
    ![Habilitar seguimientos en streaming remotos][enablestreamingtraces]
   
    Esta acción iniciará el proceso de habilitar la extensión de seguimientos en streaming en los nodos del clúster, así como las configuraciones de red necesarias.
2. Expanda el elemento **Nodos** en **Cloud Explorer**, haga clic con el botón derecho en el nodo desde el que desea transmitir seguimientos y elija **Ver seguimientos en streaming**
   
    ![Ver seguimientos en streaming remotos][viewremotestreamingtraces]
   
    Repita el paso 2 en todos los nodos desde los que quiera ver seguimientos. Cada transmisión de nodos se mostrará en una ventana dedicada.
   
    Ahora puede ver los seguimientos emitidos por Service Fabric y sus servicios. Si quiere filtrar los eventos para mostrar solo una aplicación específica, simplemente escriba el nombre de la aplicación en el filtro.
   
    ![Ver seguimientos en streaming][viewingstreamingtraces]
3. Cuando haya realizado los seguimientos en streaming desde el clúster, puede deshabilitar los seguimientos en streaming remotos; para ello, haga clic con el botón derecho en el clúster en **Cloud Explorer** y elija **Deshabilitar seguimientos en streaming**
   
    ![Deshabilitar seguimientos en streaming remotos][disablestreamingtraces]

## <a name="next-steps"></a>Pasos siguientes
* [Pruebe un servicio de Service Fabric](service-fabric-testability-overview.md).
* [Administración de aplicaciones de Service Fabric en Visual Studio](service-fabric-manage-application-in-visual-studio.md).

<!--Image references-->
[startdebugging]: ./media/service-fabric-debugging-your-application/startdebugging.png
[diagnosticevents]: ./media/service-fabric-debugging-your-application/diagnosticevents.png
[viewdiagnosticevents]: ./media/service-fabric-debugging-your-application/viewdiagnosticevents.png
[diagnosticeventsactions]: ./media/service-fabric-debugging-your-application/diagnosticeventsactions.png
[breakpoint]: ./media/service-fabric-debugging-your-application/breakpoint.png
[enableremotedebugging]: ./media/service-fabric-debugging-your-application/enableremotedebugging.png
[attachdebugger]: ./media/service-fabric-debugging-your-application/attachdebugger.png
[chooseprocess]: ./media/service-fabric-debugging-your-application/chooseprocess.png
[conditionalbreakpoint]: ./media/service-fabric-debugging-your-application/conditionalbreakpoint.png
[disableremotedebugging]: ./media/service-fabric-debugging-your-application/disableremotedebugging.png
[enablestreamingtraces]: ./media/service-fabric-debugging-your-application/enablestreamingtraces.png
[viewingstreamingtraces]: ./media/service-fabric-debugging-your-application/viewingstreamingtraces.png
[viewremotestreamingtraces]: ./media/service-fabric-debugging-your-application/viewremotestreamingtraces.png
[disablestreamingtraces]: ./media/service-fabric-debugging-your-application/disablestreamingtraces.png
