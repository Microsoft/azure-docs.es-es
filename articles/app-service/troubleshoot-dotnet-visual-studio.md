---
title: Solución de problemas con Visual Studio
description: Obtenga información sobre cómo solucionar problemas de una aplicación de App Service con herramientas de registro, seguimiento y depuración remota que están integradas en Visual Studio 2013.
ms.assetid: def8e481-7803-4371-aa55-64025d116c97
ms.devlang: dotnet
ms.topic: article
ms.date: 08/29/2016
ms.custom: devx-track-csharp, seodec18
ms.openlocfilehash: de39789a45856211421e3ec5638a2df94d49976c
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88958718"
---
# <a name="troubleshoot-an-app-in-azure-app-service-using-visual-studio"></a>Solución de problemas de una aplicación en Azure App Service con Visual Studio
## <a name="overview"></a>Información general
En este tutorial se muestra cómo usar herramientas de Visual Studio para ayudar a depurar una aplicación en [App Service](https://go.microsoft.com/fwlink/?LinkId=529714), ya sea en [modo de depuración](/visualstudio/debugger/) de manera remota o consultando los registros del servidor web o de aplicación.

Aprenderá a realizar los siguientes procedimientos:

* Qué funciones de administración de aplicaciones de Azure están disponibles en Visual Studio.
* Uso de la vista remota de Visual Studio para realizar cambios rápidos en una aplicación remota.
* Ejecución del modo de depuración de manera remota mientras un proyecto se ejecuta en Azure, tanto para una aplicación como para un WebJob.
* Creación de registros de seguimiento de aplicación y su visualización mientras los crea la aplicación.
* Visualización de registros de servidor web, incluidos mensajes de error detallados y seguimiento de solicitudes con error.
* Envío de registros de diagnóstico a una cuenta de Azure Storage y su visualización aquí.

Si tiene Visual Studio Ultimate, también puede usar [IntelliTrace](/visualstudio/debugger/intellitrace) para la depuración. IntelliTrace no se trata en este tutorial.

## <a name="prerequisites"></a><a name="prerequisites"></a>Requisitos previos
Este tutorial funciona con el entorno de desarrollo, proyecto web y aplicación App Service que configuró en [Creación de una aplicación web de ASP.NET Framework en Azure](quickstart-dotnet-framework.md). Para las secciones de WebJobs, necesitará la aplicación que creó en [Introducción al SDK de Azure WebJobs][GetStartedWJ].

Los ejemplos de código que aparecen en este tutorial son para una aplicación web C# MVC, pero los procedimientos de solución de problemas son los mismos para las aplicaciones de Web Forms y Visual Basic.

En el tutorial se supone que está usando Visual Studio 2019. 

La característica de registros de streaming solo funciona para aplicaciones dirigidas a .NET Framework 4 o superior.

## <a name="app-configuration-and-management"></a><a name="sitemanagement"></a>Administración y configuración de la aplicación
Visual Studio proporciona acceso a un subconjunto de las funciones de administración de aplicaciones y los ajustes de configuración disponibles en [Azure Portal](https://go.microsoft.com/fwlink/?LinkId=529715). En esta sección podrá ver las opciones y funciones disponibles mediante el **Explorador de servidores**. Para ver las últimas características de integración de Azure, pruebe también **Cloud Explorer** . Puede abrir dos ventanas del menú **Ver** .

1. Si aún no ha iniciado sesión en Azure en Visual Studio, haga clic con el botón derecho en **Azure** y seleccione Conectar a la **suscripción de Microsoft Azure** en el **Explorador de servidores**.

    Una alternativa es instalar un certificado de administración que permita el acceso a su cuenta. Si elige instalar un certificado, haga clic con el botón derecho en el nodo **Azure** del **Explorador de servidores** y, a continuación, seleccione **Administrar y filtrar suscripciones** en el menú contextual. En el cuadro de diálogo **Administrar suscripciones de Microsoft Azure**, haga clic en la pestaña **Certificados** y, luego, en **Importar**. Siga las instrucciones para descargar y luego importar un archivo de suscripción (también conocido como archivo *.publishsettings* ) para su cuenta de Azure.

   > [!NOTE]
   > Si descarga un archivo de suscripción, guárdelo en una carpeta ajena a los directorios de código fuente (por ejemplo, en la carpeta Descargas) y elimínelo una vez que finalice la importación. Si un usuario malintencionado obtuviera acceso al archivo de suscripción, podría editar, crear y eliminar servicios de Azure.
   >
   >

    Para obtener más información acerca de la conexión a recursos de Azure desde Visual Studio, consulte [Administración de cuentas, suscripciones y roles administrativos](https://go.microsoft.com/fwlink/?LinkId=324796#BKMK_AccountVCert).
2. En el **Explorador de servidores**, expanda **Azure** y, a continuación, **App Service**.
3. Expanda el grupo de recursos que incluye la aplicación que creó en [Creación de una aplicación web de ASP.NET Framework en Azure](quickstart-dotnet-framework.md) y luego haga clic con el botón derecho en el nodo de la aplicación y haga clic en **Ver configuración**.

    ![Ver configuración en el Explorador de servidores](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-viewsettings.png)

    Aparecerá la pestaña **Aplicación web de Azure** y podrá ver las tareas de administración y configuración de aplicaciones disponibles en Visual Studio.

    ![Ventana de aplicaciones web de Azure](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-configtab.png)

    En este tutorial, usará los menús desplegables de registro y seguimiento. También utilizará la depuración remota, pero va a utilizar un método distinto para habilitarla.

    Para obtener información sobre los cuadros Configuración de la aplicación y Cadenas de conexión de esta ventana, vea [Azure App Service: How Application Strings and Connection Strings Work](https://azure.microsoft.com/blog/windows-azure-web-sites-how-application-strings-and-connection-strings-work/) (Funcionamiento de las cadenas de aplicación y de las cadenas de conexión).

    Si desea realizar una tarea de administración de aplicaciones que no se pueda hacer en esta ventana, haga clic en **Abrir en Portal de administración** para abrir una ventana del explorador con Azure Portal.

## <a name="access-app-files-in-server-explorer"></a><a name="remoteview"></a>Acceso a archivos de aplicaciones en el Explorador de servidores
Normalmente implementa un sitio con la marca `customErrors` en el archivo Web.config establecida en `On` o en `RemoteOnly`, lo que significa que no recibirá un práctico mensaje de error cuando algo se realice de manera incorrecta. Para muchos errores, todo lo que verá será una página como alguna de las siguientes:

**Error del servidor en la aplicación '/':**

![Página de error poco práctica](./media/web-sites-dotnet-troubleshoot-visual-studio/genericerror.png)

**Se produjo un error:**

![Página de error poco práctica](./media/web-sites-dotnet-troubleshoot-visual-studio/genericerror1.png)

**El sitio web no puede mostrar la página**

![Página de error poco práctica](./media/web-sites-dotnet-troubleshoot-visual-studio/genericerror2.png)

Con frecuencia, la manera más fácil de encontrar la causa de un error es habilitar los mensajes detallados de error, una acción que se explica en la primera de las capturas de pantalla anteriores. Eso requiere un cambio en el archivo Web.config implementado. Puede editar el archivo *Web.config* en el proyecto y volver a implementar el proyecto, o bien crear una [transformación de Web.config](https://www.asp.net/mvc/tutorials/deployment/visual-studio-web-deployment/web-config-transformations) e implementar una compilación de depuración, pero hay una forma más rápida: en el **Explorador de soluciones** puede ver y editar archivos en la aplicación remota utilizando directamente la característica de *vista remota*.

1. En el **Explorador de servidores**, expanda **Azure**, expanda **App Service**, expanda el grupo de recursos en el que se encuentra la aplicación y después expanda el nodo de la aplicación.

    Puede ver los nodos que le proporcionan acceso a los archivos de registro y de contenido de la aplicación.
2. Expanda el nodo **Archivos** y haga doble clic en el archivo *Web.config* .

    ![Abrir Web.config](./media/web-sites-dotnet-troubleshoot-visual-studio/webconfig.png)

    Visual Studio abre el archivo Web.config desde la aplicación remota y muestra [Remote] junto al nombre de archivo en la barra de título.
3. Agregue la siguiente línea al elemento `system.web` :

    `<customErrors mode="Off"></customErrors>`

    ![Editar Web.config](./media/web-sites-dotnet-troubleshoot-visual-studio/webconfigedit.png)
4. Actualice el explorador que actualmente muestra el mensaje de error poco práctico y ahora podrá ver un mensaje detallado del error, tal como muestra el siguiente ejemplo:

    ![Mensajes de error detallados](./media/web-sites-dotnet-troubleshoot-visual-studio/detailederror.png)

    (El error que aparece se generó al agregar la línea en rojo a *Views\Home\Index.cshtml*).

Editar el archivo Web.config es solo un ejemplo de las situaciones en las que la capacidad de leer y editar archivos en la aplicación de App Service facilita la solución de problemas.

## <a name="remote-debugging-apps"></a><a name="remotedebug"></a>Aplicaciones de depuración remota
Si el mensaje de error detallado no proporciona información suficiente y no es posible recrear el error de manera local, otra forma de solucionar problemas es ejecutar remotamente en modo de depuración. Puede definir puntos de interrupción, manipular directamente la memoria, revisar el código e, incluso, cambiar la ruta del código.

La depuración remota no funciona en ediciones Express de Visual Studio.

En esta sección se muestra cómo realizar una depuración remota mediante el proyecto creado en [Creación de una aplicación web de ASP.NET Framework en Azure](quickstart-dotnet-framework.md).

1. Abra el proyecto web que creó en [Creación de una aplicación web de ASP.NET Framework en Azure](quickstart-dotnet-framework.md).

1. Abra *Controllers\HomeController.cs*.

1. Elimine el método `About()` e inserte el siguiente código en su lugar.

    ```csharp
    public ActionResult About()
    {
        string currentTime = DateTime.Now.ToLongTimeString();
        ViewBag.Message = "The current time is " + currentTime;
        return View();
    }
    ```

1. [Establezca un punto de interrupción](https://docs.microsoft.com/visualstudio/debugger/) en la línea `ViewBag.Message`.

1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto y, luego, haga clic en **Publicar**.

1. En la lista desplegable **Perfil**, seleccione el mismo perfil que usó en [Creación de una aplicación web de ASP.NET Framework en Azure](quickstart-dotnet-framework.md). A continuación, haga clic en Configuración.

1. En el cuadro de diálogo **Publicar**, haga clic en la pestaña **Configuración** y, a continuación, cambie la **Configuración** a **Depurar** y, luego, haga clic en **Guardar**.

    ![Publicar en modo de depuración](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-publishdebug.png)

1. Haga clic en **Publicar**. Una vez que haya finalizado la implementación y que el explorador abre la dirección URL de Azure de su aplicación, cierre el explorador.

1. En el **Explorador de servidores**, haga clic con el botón derecho en la aplicación y luego haga clic en **Asociar depurador**.

    ![Asociar depurador](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-attachdebugger.png)

    El explorador se abre automáticamente en su página inicial en ejecución en Azure. Es posible que deba esperar unos 20 segundos mientras Azure configura el servidor para la depuración. Esta demora solo se produce la primera vez que ejecuta el modo de depuración en una aplicación en un período de 48 horas. Al iniciar la depuración de nuevo en el mismo período, no hay ningún retraso.

    > [!NOTE] 
    > Si tiene algún problema para iniciar el depurador, intente hacerlo con **Cloud Explorer** en lugar de usar el **Explorador de servidores**.
    >

1. Haga clic en **Acerca de** en el menú.

    Visual Studio se detiene en el punto de interrupción y el código se ejecuta en Azure, no en su equipo local.

1. Mantenga el puntero sobre la variable `currentTime` para ver el valor de tiempo.

    ![Ver la variable en ejecución en modo de depuración en Azure](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-debugviewinwa.png)

    La hora que ve es la hora del servidor Azure, que puede estar en una zona horaria distinta a la de su equipo local.

1. Escriba un valor nuevo para la variable `currentTime` , como "Ahora en ejecución en Azure".

1. Presione F5 para continuar la ejecución.

     La página About que se ejecuta en Azure muestra el valor nuevo que escribió en la variable currentTime.

     ![Página About con el valor nuevo](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-debugchangeinwa.png)

## <a name="remote-debugging-webjobs"></a><a name="remotedebugwj"></a> WebJobs de depuración remota
En esta sección se muestra cómo depurar de forma remota mediante el proyecto y la aplicación que creó en [Get Started with the Azure WebJobs SDK](https://github.com/Azure/azure-webjobs-sdk/wiki) (Introducción al SDK de Azure WebJobs).

Las características mostradas en esta sección solo están disponibles en Visual Studio 2013 con Update 4 o superior.

La depuración remota solo funciona con WebJobs continuos. Los WebJobs bajo demanda y programados no admiten la depuración.

1. Abra el proyecto web que creó en [Introducción al SDK de Azure WebJobs][GetStartedWJ].

2. En el proyecto ContosoAdsWebJob, abra *Functions.cs*.

3. [Establezca un punto de interrupción](https://docs.microsoft.com/visualstudio/debugger/) en la primera instrucción del método `GnerateThumbnail`.

    ![Establecimiento de punto de interrupción](./media/web-sites-dotnet-troubleshoot-visual-studio/wjbreakpoint.png)

4. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto web (no el proyecto WebJob) y, luego, haga clic en **Publicar**.

5. En la lista desplegable **Perfil** , seleccione el mismo perfil que utilizó en [Introducción al SDK de Azure WebJobs](https://github.com/Azure/azure-webjobs-sdk/wiki).

6. Haga clic en la pestaña **Configuración**, cambie la **Configuración** a **Depurar** y, luego, haga clic en **Publicar**.

    Visual Studio implementará los proyectos web y de WebJobs, y el explorador se abrirá en la dirección URL de Azure de su aplicación.

7. En el **Explorador de servidores**, expanda **Azure > App Service > su grupo de recursos > su aplicación > WebJobs > Continuo** y después haga clic con el botón derecho en **ContosoAdsWebJob**.

8. Haga clic en **Adjuntar el depurador**.

    ![Asociar depurador](./media/web-sites-dotnet-troubleshoot-visual-studio/wjattach.png)

    El explorador se abre automáticamente en su página inicial en ejecución en Azure. Es posible que deba esperar unos 20 segundos mientras Azure configura el servidor para la depuración. Esta demora solo se produce la primera vez que ejecuta el modo de depuración en una aplicación en un período de 48 horas. Al iniciar la depuración de nuevo en el mismo período, no hay ningún retraso.

9. En el explorador web que se abre en la página principal de Contoso Ads, cree un nuevo anuncio.

    Al crear un anuncio, se crea un mensaje de la cola, que WebJob recogerá y procesará. Cuando el SDK de WebJobs llama a la función para procesar el mensaje de la cola, el código alcanza el punto de interrupción.

10. Cuando el depurador se interrumpe en el punto de interrupción, puede examinar y cambiar los valores de las variables mientras está ejecutando el programa en la nube. En la siguiente ilustración, el depurador muestra el contenido del objeto blobInfo que se pasó al método `GenerateThumbnail`.

     ![Objeto blobInfo en el depurador](./media/web-sites-dotnet-troubleshoot-visual-studio/blobinfo.png)

11. Presione F5 para continuar la ejecución.

     El método `GenerateThumbnail` finaliza la creación de la miniatura.

12. En el explorador, actualice la página de índice y vea la miniatura.

13. En Visual Studio, presione MAYÚS+F5 para detener la depuración.

14. En el **Explorador de servidores**, haga clic con el botón derecho en el nodo ContosoAdsWebJob y, luego, haga clic en **Ver panel**.

15. Inicie sesión con sus credenciales de Azure y, a continuación, haga clic en el nombre de WebJob para ir a la página de su WebJob.

     ![Clic en ContosoAdsWebJob](./media/web-sites-dotnet-troubleshoot-visual-studio/clickcaw.png)

     El panel muestra la función `GenerateThumbnail` ejecutada recientemente.

     (La próxima vez que haga clic en **Ver panel**, no tendrá que iniciar sesión y el explorador irá directamente a la página de su WebJob).

16. Haga clic en el nombre de función para ver detalles acerca de la ejecución de la función.

     ![Detalles de la función](./media/web-sites-dotnet-troubleshoot-visual-studio/funcdetails.png)

Si la función [escribió registros](https://github.com/Azure/azure-webjobs-sdk/wiki), puede hacer clic en **ToggleOutput** para verlos.

## <a name="notes-about-remote-debugging"></a>Notas acerca de la depuración remota

* No se recomienda ejecutar el modo de depuración en producción. Si la aplicación de producción no está escalada horizontalmente a varias instancias de servidor, la depuración impide que el servidor web responda a otras solicitudes. Si tiene varias instancias de servidor web, cuando se asocie al depurador recibirá una instancia aleatoria y no tendrá forma de asegurarse de que las solicitudes del explorador siguientes se dirijan a la misma instancia. Además, normalmente no implementará una compilación de depuración en producción, y las optimizaciones de compilador para compilaciones de versión podrían hacer imposible mostrar lo que ocurre línea a línea en su código fuente. Para solucionar problemas de producción, su mejor recurso son los registros de servidor web y de seguimiento de la aplicación.
* Evite detenciones prolongadas en los puntos de interrupción durante la depuración remota. Azure considera un proceso detenido durante más de unos minutos como un proceso sin respuesta y lo apaga.
* Mientras realiza la depuración, el servidor envía datos a Visual Studio, lo que podría afectar a los cargos de ancho de banda. Para obtener información acerca de las tarifas de ancho de banda, consulte [Precios de Azure](https://azure.microsoft.com/pricing/calculator/).
* Asegúrese de que el atributo `debug` del elemento `compilation` en el archivo *Web.config* esté establecido en true. Este es el valor predeterminado cuando publica una configuración de compilación de depuración.

    ```xml
    <system.web>
      <compilation debug="true" targetFramework="4.5" />
      <httpRuntime targetFramework="4.5" />
    </system.web>
    ```
* Si encuentra que el depurador no entra en el código que desea depurar, es posible que deba cambiar la configuración de Solo mi código.  Para más información, consulte [Specify whether to debug only user code using Just My Code in Visual Studio](https://docs.microsoft.com/visualstudio/debugger/just-my-code) (Elección de depurar solo código de usuario con Solo mi código en Visual Studio).
* Cuando habilita la característica de depuración remota, se inicia un temporizador en el servidor y 48 horas después la característica se desactiva automáticamente. Este límite de 48 horas es por motivos de seguridad y rendimiento. Puede activar fácilmente la característica las veces que lo desee. Recomendamos dejarla deshabilitada cuando no esté realizando activamente una depuración.
* Puede asociar manualmente el depurador a cualquier proceso, no solo al proceso de la aplicación (w3wp.exe). Para obtener más información acerca de cómo utilizar el modo de depuración en Visual Studio, consulte [Depuración en Visual Studio](/visualstudio/debugger/debugging-in-visual-studio).

## <a name="diagnostic-logs-overview"></a><a name="logsoverview"></a>Información general de registros de diagnóstico
Una aplicación ASP.NET que se ejecuta en una aplicación de App Service puede crear las siguientes clases de registros:

* **Registros de seguimiento de aplicación**<br/>
  La aplicación crea estos registros mediante la llamada a métodos de la clase [System.Diagnostics.Trace](/dotnet/api/system.diagnostics.trace) .
* **Registros de servidor web**<br/>
  El servidor web crea una entrada de registro para cada solicitud HTTP a la aplicación.
* **Registros de mensaje de error detallados**<br/>
  El servidor web crea una página HTML con algo de información adicional para solicitudes HTTP con error (las solicitudes que generan el código de error 400 o superior).
* **Registros de seguimiento de solicitudes con error**<br/>
  El servidor web crea un archivo XML con información de seguimiento detallada para solicitudes HTTP con error. El servidor web también proporciona un archivo XSL para dar formato al XML en un explorador.

El registro afecta el rendimiento de la aplicación, por lo que Azure le ofrece la posibilidad de habilitar o deshabilitar cada tipo de registro según sea necesario. En el caso de registros de aplicaciones, puede especificar que solo se escriban los registros por encima de un determinado nivel de gravedad. Cuando crea una aplicación, todos los registros están deshabilitados de manera predeterminada.

Los registros se escriben en archivos en una carpeta *LogFiles* en el sistema de archivos del sitio y son accesibles a través de FTP. Los registros de servidor web y registros de aplicaciones también se pueden escribir en una cuenta de Azure Storage. Puede conservar un volumen mayor de registros en una cuenta de almacenamiento de lo que es posible en el sistema de archivos. Cuando utiliza el sistema de archivos, tiene un límite máximo de 100 megabytes de registros. (Los registros del sistema de archivos solo sirven para la conservación a corto plazo. Azure elimina los archivos de registro antiguos para dejar espacio para los nuevos una vez que se alcanza el límite).  

## <a name="create-and-view-application-trace-logs"></a><a name="apptracelogs"></a>Creación y visualización de registros de seguimiento de aplicación
En esta sección, realizará las siguientes tareas:

* Agregue instrucciones de seguimiento para el proyecto web que creó en [Creación de una aplicación web de ASP.NET Framework en Azure](quickstart-dotnet-framework.md).
* Ver los registros cuando ejecute localmente el proyecto.
* Ver los registros a medida que los genera la aplicación que se ejecuta en Azure.

Para obtener información acerca de cómo crear registros de aplicaciones en WebJobs, consulte [Trabajo con el almacenamiento de cola de Azure mediante el SDK de WebJobs - Cómo escribir registros](https://github.com/Azure/azure-webjobs-sdk/wiki). Las instrucciones siguientes para ver los registros y controlar cómo se almacenan en Azure se aplican también a los registros de aplicación creados por WebJobs.

### <a name="add-tracing-statements-to-the-application"></a>Adición de instrucciones de seguimiento a la aplicación
1. Abra *Controllers\HomeController.cs* y reemplace los métodos `Index`, `About` y `Contact` por el siguiente código para agregar instrucciones `Trace` y una instrucción `using` para `System.Diagnostics`:

    ```csharp
    public ActionResult Index()
    {
        Trace.WriteLine("Entering Index method");
        ViewBag.Message = "Modify this template to jump-start your ASP.NET MVC application.";
        Trace.TraceInformation("Displaying the Index page at " + DateTime.Now.ToLongTimeString());
        Trace.WriteLine("Leaving Index method");
        return View();
    }
    
    public ActionResult About()
    {
        Trace.WriteLine("Entering About method");
        ViewBag.Message = "Your app description page.";
        Trace.TraceWarning("Transient error on the About page at " + DateTime.Now.ToShortTimeString());
        Trace.WriteLine("Leaving About method");
        return View();
    }
    
    public ActionResult Contact()
    {
        Trace.WriteLine("Entering Contact method");
        ViewBag.Message = "Your contact page.";
        Trace.TraceError("Fatal error on the Contact page at " + DateTime.Now.ToLongTimeString());
        Trace.WriteLine("Leaving Contact method");
        return View();
    }        
    ```

1. Agregue una instrucción `using System.Diagnostics;` a la parte superior del archivo.

### <a name="view-the-tracing-output-locally"></a>Visualización del resultado de seguimiento localmente
1. Presione F5 para ejecutar la aplicación en modo de depuración.

    La escucha de seguimiento predeterminada escribe todo el resultado del seguimiento en la ventana **Resultados** , además del resultado de la depuración. La siguiente ilustración muestra el resultado de las instrucciones de seguimiento que agregó al método `Index` .

    ![Ventana de seguimiento en depuración](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-debugtracing.png)

    Los pasos siguientes muestran cómo ver el resultado del seguimiento en una página web, sin compilar en modo de depuración.
1. Abra el archivo Web.config de la aplicación (el que está ubicado en la carpeta del proyecto) y agregue un elemento `<system.diagnostics>` al final del archivo, justo antes del elemento `</configuration>` de cierre:

    ``` xml
    <system.diagnostics>
    <trace>
      <listeners>
        <add name="WebPageTraceListener"
            type="System.Web.WebPageTraceListener,
            System.Web,
            Version=4.0.0.0,
            Culture=neutral,
            PublicKeyToken=b03f5f7f11d50a3a" />
      </listeners>
    </trace>
    </system.diagnostics>
    ```

`WebPageTraceListener` le permite ver los resultados de seguimiento si navega a `/trace.axd`.
1. Agregue un <a href="https://msdn.microsoft.com/library/vstudio/6915t83k(v=vs.100).aspx">elemento de seguimiento</a> bajo `<system.web>` en el archivo Web.config, como en el siguiente ejemplo:

    ``` xml
    <trace enabled="true" writeToDiagnosticsTrace="true" mostRecent="true" pageOutput="false" />
    ```

1. Presione CTRL+F5 para ejecutar la aplicación.
1. En la barra de direcciones de la ventana del explorador, agregue *trace.axd* a la dirección URL y, luego, presione ENTRAR (la dirección URL es similar a `http://localhost:53370/trace.axd`).
1. En la página **Seguimiento de la aplicación**, haga clic en **Ver detalles** en la primera línea (no en la línea BrowserLink).

    ![trace.axd](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-traceaxd1.png)

    Aparece la página **Detalles de la solicitud** y en la sección **Información de seguimiento**, podrá ver el resultado de las instrucciones de seguimiento que agregó al método `Index`.

    ![trace.axd](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-traceaxd2.png)

    De manera predeterminada, `trace.axd` solo está disponible localmente. Si desea que esté disponible desde una aplicación remota, puede agregar `localOnly="false"` al elemento `trace` en el archivo *Web.config*, tal como aparece en el siguiente ejemplo:

    ```xml
    <trace enabled="true" writeToDiagnosticsTrace="true" localOnly="false" mostRecent="true" pageOutput="false" />
    ```

    Pero no se recomienda habilitar `trace.axd` en una aplicación de producción por motivos de seguridad. En las siguientes secciones, verá una forma más sencilla de leer registros de seguimiento en una aplicación de App Service.

### <a name="view-the-tracing-output-in-azure"></a>Visualización del resultado de seguimiento en Azure
1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto web y, a continuación, en **Publicar**.
2. En el cuadro de diálogo **Publicación web**, haga clic en **Publicar**.

    Después de que Visual Studio publica su actualización, se abre una ventana del explorador en su página principal (suponiendo que no desactivó **Dirección URL de destino** en la pestaña **Conexión**).
3. En el **Explorador de servidores**, haga clic con el botón derecho en la aplicación y seleccione **Ver registros de streaming**.

    ![Ver registros de streaming en el menú contextual](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-viewlogsmenu.png)

    La ventana **Resultados** muestra que está conectado al servicio de streaming de registros y agrega una línea de notificación cada minuto que pasa sin un registro para mostrar.

    ![Ver registros de streaming en el menú contextual](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-nologsyet.png)
4. En la ventana del explorador que muestra la página principal de su aplicación, haga clic en **Contact**.

    En unos pocos segundos, el resultado del seguimiento en el nivel de error que agregó al método `Contact` aparecerá en la ventana **Resultados**.

    ![Seguimiento de errores en la ventana Output](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-errortrace.png)

    Visual Studio muestra solo seguimientos en el nivel de errores porque es el valor predeterminado cuando habilita el servicio de supervisión de registros. Cuando crea una aplicación de App Service, todos los registros están deshabilitados de manera predeterminada, tal como vio cuando abrió anteriormente la página de configuración:

    ![Registro de aplicación desactivado](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-apploggingoff.png)

    Sin embargo, cuando selecciona **Ver registros de streaming**, Visual Studio cambia automáticamente **Registro de la aplicación (sistema de archivos)** a **Error**, lo que significa que los registros de errores se notifican. Para ver todos los registros de seguimiento, puede cambiar esta configuración a **Detallado**. Cuando selecciona un nivel de gravedad inferior al error, también se notifican todos los registros para niveles de gravedad más altos. Por lo tanto, cuando selecciona detallado, también verá registros de errores, advertencias e información.  

5. En el **Explorador de servidores**, haga clic con el botón derecho en la aplicación y después haga clic en **Ver configuración** como hizo antes.
6. Cambie **Registro de la aplicación (sistema de archivos)** a **Detallado** y haga clic en **Guardar**.

    ![Definir el nivel de seguimiento como Verbose](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-applogverbose.png)
7. En la ventana del explorador que ahora muestra su página **Contacto**, haga clic en **Inicio**, en **Información** y, a continuación, haga clic en **Contacto**.

    En unos pocos segundos, la ventana **Resultados** mostrará todos los resultados del seguimiento.

    ![Resultados detallados del seguimiento](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-verbosetraces.png)

    En esta sección, aprendió a habilitar y deshabilitar el registro mediante la configuración de la aplicación. También puede habilitar y deshabilitar escuchas de seguimiento si modifica el archivo Web.config. Pero modificar el archivo Web.config hace que el dominio de la aplicación se recicle, mientras que habilitar el registro a través de la aplicación no provoca eso. Si el problema tarda mucho en reproducirse, o bien es intermitente, reciclar el dominio de la aplicación podría "arreglarlo" y forzarle a esperar hasta que vuelva a ocurrir. Permitir el diagnóstico en Azure le permite comenzar inmediatamente a capturar información sobre el error sin reciclar el dominio de aplicación.

### <a name="output-window-features"></a>Características de la ventana de salida
La pestaña **Registros de Microsoft Azure** de la ventana **Resultados** tiene varios botones y un cuadro de texto:

![Botones de la pestaña de registros](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-icons.png)

Estos elementos realizan las siguientes funciones:

* Limpiar la ventana **Resultado** .
* Habilitar o deshabilitar el ajuste automático de línea.
* Iniciar o detener la supervisión de los registros.
* Especificar los registros para supervisar.
* Descargar registros.
* Filtrar los registros según una cadena de búsqueda o una expresión regular.
* Cierre la ventana **Resultados** .

Si especificó una cadena de búsqueda o una expresión regular, Visual Studio filtra la información de registro en el cliente. Esto significa que puede especificar los criterios después de que aparezcan los registros en la ventana **Resultados** y que puede cambiar esos criterios de filtrado sin tener que volver a generar los registros.

## <a name="view-web-server-logs"></a><a name="webserverlogs"></a>Visualización de registros de servidor web
Los registros de servidor web registran toda la actividad HTTP para la aplicación. Para verlos en la ventana **Resultados**, debe habilitarlos en la aplicación e indicarle a Visual Studio que desea supervisarlos.

1. En la pestaña **Configuración de aplicaciones web de Azure** que abrió desde el **Explorador de servidores**, cambie el registro de servidor web a **Activado** y haga clic en **Guardar**.

    ![Habilitar el registro de servidor web](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-webserverloggingon.png)
2. En la ventana **Resultados**, haga clic en el botón **Especificar los registros de Microsoft Azure para supervisar**.

    ![Especificar los registros de Azure para supervisar](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-specifylogs.png)
3. En el cuadro de diálogo **Opciones de registro de Microsoft Azure**, seleccione **Registros de servidor web** y haga clic en **Aceptar**.

    ![Supervisar los registros de servidor web](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-monitorwslogson.png)
4. En la ventana del explorador que muestra la aplicación, haga clic en **Inicio**, en **Información** y después en **Contacto**.

    Los registros de aplicación generalmente aparecen primero y luego aparecen los registros de servidor web. Es posible que deba esperar un poco hasta que aparezcan.

    ![Registros de servidor web en la ventana Output](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-wslogs.png)

De manera predeterminada, cuando habilita por primera vez los registros de servidor web mediante el uso de Visual Studio, Azure escribe los registros en el sistema de archivos. De manera alternativa, puede utilizar el Portal de Azure para especificar que los registros de servidor web se deben escribir en un contenedor de blobs en una cuenta de almacenamiento.

Si utiliza el portal para habilitar el registro de servidor web para una cuenta de almacenamiento de Azure y luego deshabilitar el registro en Visual Studio, cuando vuelva a habilitar el registro en Visual Studio, se restaurará la configuración de su cuenta de almacenamiento.

## <a name="view-detailed-error-message-logs"></a><a name="detailederrorlogs"></a>Visualización de registros de mensajes de error detallados
Los registros de error detallados proporcionan información adicional acerca de las solicitudes HTTP que generaron códigos de respuesta con error (400 o superiores). Para verlos en la ventana **Resultados**, debe habilitarlos en la aplicación e indicarle a Visual Studio que desea supervisarlos.

1. En la pestaña **Configuración de aplicaciones web de Azure** que abrió desde el **Explorador de servidores**, cambie **Mensajes de error detallados** a **Activado** y haga clic en **Guardar**.

    ![Habilitar los mensajes de error detallados](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-detailedlogson.png)

2. En la ventana **Resultados**, haga clic en el botón **Especificar los registros de Microsoft Azure para supervisar**.

3. En el cuadro de diálogo **Opciones de registro de Microsoft Azure**, haga clic en **Todos los registros** y, a continuación, en **Aceptar**.

    ![Supervisar todos los registros](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-monitorall.png)

4. En la barra de direcciones de la ventana del explorador, agregue un carácter adicional a la dirección URL para generar un error 404 (por ejemplo, `http://localhost:53370/Home/Contactx`) y presione Entrar.

    Después de varios segundos, el registro detallado del error aparecerá en la ventana **Resultados** de Visual Studio.

    ![Registro de error detallado: ventana Resultados](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-detailederrorlog.png)

    Presione Ctrl y haga clic en el vínculo para ver el resultado del registro formateado en un explorador:

    ![Registro de error detallado: ventana del explorador](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-detailederrorloginbrowser.png)

## <a name="download-file-system-logs"></a><a name="downloadlogs"></a>Descarga de registros del sistema de archivos
Todos los registros que se pueden supervisar en la ventana **Resultados** también pueden descargarse como un archivo *.zip*

1. En la ventana **Resultados**, haga clic en **Descargar registros de streaming**.

    ![Botones de la pestaña de registros](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-downloadicon.png)

    El Explorador de archivos abrirá la carpeta *Descargas* con el archivo descargado seleccionado.

    ![Archivo descargado](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-downloadedfile.png)
2. Extraiga el archivo *.zip* y verá la siguiente estructura de carpetas:

    ![Archivo descargado](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-logfilefolders.png)

   * Los registros de seguimiento de aplicación están en archivos *.txt* en la carpeta *LogFiles\Application*.
   * Los registros de servidor web están en archivos *.log* en la carpeta *LogFiles\http\RawLogs*. Puede utilizar una herramienta como [Analizador del registro](https://www.microsoft.com/download/details.aspx?displaylang=en&id=24659) para ver y manipular estos archivos.
   * Los registros de mensajes de error detallados están en los archivos *.html* de la carpeta *LogFiles\DetailedErrors*.

     (La carpeta de *implementaciones* es para archivos creados por la publicación del control del código fuente; no tiene ninguna relación con la publicación en Visual Studio. La carpeta *Git* es para seguimientos relacionados con la publicación de control de origen y el servicio de transmisión de archivos de registro).  

<!-- ## <a name="storagelogs"></a>View storage logs
Application tracing logs can also be sent to an Azure storage account, and you can view them in Visual Studio. To do that you'll create a storage account, enable storage logs in the Azure portal, and view them in the **Logs** tab of the **Azure Web App** window.

You can send logs to any or all of three destinations:

* The file system.
* Storage account tables.
* Storage account blobs.

You can specify a different severity level for each destination.

Tables make it easy to view details of logs online, and they support streaming; you can query logs in tables and see new logs as they are being created. Blobs make it easy to download logs in files and to analyze them using HDInsight, because HDInsight knows how to work with blob storage. For more information, see **Hadoop and MapReduce** in [Data Storage Options (Building Real-World Cloud Apps with Azure)](https://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/data-storage-options).

You currently have file system logs set to verbose level; the following steps walk you through setting up information level logs to go to storage account tables. Information level means all logs created by calling `Trace.TraceInformation`, `Trace.TraceWarning`, and `Trace.TraceError` will be displayed, but not logs created by calling `Trace.WriteLine`.

Storage accounts offer more storage and longer-lasting retention for logs compared to the file system. Another advantage of sending application tracing logs to storage is that you get some additional information with each log that you don't get from file system logs.

1. Right-click **Storage** under the Azure node, and then click **Create Storage Account**.

![Create Storage Account](./media/web-sites-dotnet-troubleshoot-visual-studio/createstor.png)

1. In the **Create Storage Account** dialog, enter a name for the storage account.

    The name must be must be unique (no other Azure storage account can have the same name). If the name you enter is already in use you'll get a chance to change it.

    The URL to access your storage account will be *{name}*.core.windows.net.
2. Set the **Region or Affinity Group** drop-down list to the region closest to you.

    This setting specifies which Azure datacenter will host your storage account. For this tutorial your choice won't make a noticeable difference, but for a production web app you want your web server and your storage account to be in the same region to minimize latency and data egress charges. The web app (which you'll create later) should run in a region as close as possible to the browsers accessing your web app in order to minimize latency.
3. Set the **Replication** drop-down list to **Locally redundant**.
   
    When geo-replication is enabled for a storage account, the stored content is replicated to a secondary datacenter to enable failover to that location in case of a major disaster in the primary location. Geo-replication can incur additional costs. For test and development accounts, you generally don't want to pay for geo-replication. For more information, see [Create, manage, or delete a storage account](../storage/common/storage-create-storage-account.md).
4. Click **Create**.

    ![New storage account](./media/web-sites-dotnet-troubleshoot-visual-studio/newstorage.png)    
5. In the Visual Studio **Azure Web App** window, click the **Logs** tab, and then click **Configure Logging in Management Portal**.

     <!-- todo:screenshot of new portal if the VS page link goes to new portal -- >
    ![Configure logging](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-configlogging.png)

    This opens the **Configure** tab in the portal for your web app.
6. In the portal's **Configure** tab, scroll down to the application diagnostics section, and then change **Application Logging (Table Storage)** to **On**.
7. Change **Logging Level** to **Information**.
8. Click **Manage Table Storage**.

    ![Click Manage TableStorage](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-stgsettingsmgmtportal.png)

    In the **Manage table storage for application diagnostics** box, you can choose your storage account if you have more than one. You can create a new table or use an existing one.

    ![Manage table storage](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-choosestorageacct.png)
9. In the **Manage table storage for application diagnostics** box, click the check mark to close the box.
10. In the portal's **Configure** tab, click **Save**.
11. In the browser window that displays the application web app, click **Home**, then click **About**, and then click **Contact**.

     The logging information produced by browsing these web pages is written to the storage account.
12. In the **Logs** tab of the **Azure Web App** window in Visual Studio, click **Refresh** under **Diagnostic Summary**.

     ![Click Refresh](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-refreshstorage.png)

     The **Diagnostic Summary** section shows logs for the last 15 minutes by default. You can change the period to see more logs.

     (If you get a "table not found" error, verify that you browsed to the pages that do the tracing after you enabled **Application Logging (Storage)** and after you clicked **Save**.)

     ![Storage logs](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-storagelogs.png)

     Notice that in this view you see **Process ID** and **Thread ID** for each log, which you don't get in the file system logs. You can see additional fields by viewing the Azure storage table directly.
13. Click **View all application logs**.

     The trace log table appears in the Azure storage table viewer.

     (If you get a "sequence contains no elements" error, open **Server Explorer**, expand the node for your storage account under the **Azure** node, and then right-click **Tables** and click **Refresh**.)

     ![Storage logs in table view](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-tracelogtableview.png)

     This view shows additional fields you don't see in any other views. This view also enables you to filter logs by using special Query Builder UI for constructing a query. For more information, see Working with Table Resources - Filtering Entities in [Browsing Storage Resources with Server Explorer](https://msdn.microsoft.com/library/ff683677.aspx).
14. To look at the details for a single row, double-click one of the rows.

     ![Trace table in Server Explorer](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-tracetablerow.png)
 -->
## <a name="view-failed-request-tracing-logs"></a><a name="failedrequestlogs"></a>Visualización de registros de seguimiento de solicitudes con error
Los registros de seguimiento de solicitudes con error son útiles cuando necesita comprender los detalles de cómo IIS maneja una solicitud HTTP, en situaciones tales como problemas de autenticación o reconfiguración de URL.

Las aplicaciones de App Service usan la misma funcionalidad de seguimiento de solicitudes con error que ha estado disponible con IIS 7.0 y posterior. Sin embargo, no tiene acceso a la configuración de IIS que determina los errores que se registran. Cuando habilita el seguimiento de solicitudes con error, se capturan todos los errores.

Puede habilitar el seguimiento de solicitudes con error a través de Visual Studio, pero no puede verlas ahí. Estos registros son archivos XML. El servicio de registro de streaming solo supervisa archivos que se pueden leer en modo de texto sin formato: *.txt*, *.html* y *.log*.

Puede ver los registros de seguimiento de solicitudes en un explorador directamente vía FTP o de manera local después de utilizar una herramienta FTP para descargarlos en el equipo local. En esta sección, los verá directamente en un explorador.

1. En la pestaña **Configuración** de la ventana **Aplicación web de Azure** que abrió desde el **Explorador de servidores**, cambie **Seguimiento de solicitudes con error** a **Activado** y haga clic en **Guardar**.

    ![Habilitar el seguimiento de solicitudes con error](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-failedrequeston.png)
2. En la barra de direcciones de la ventana del explorador que muestra la aplicación, agregue un carácter adicional a la dirección URL y haga clic en Entrar para generar un error 404.

    Esto hará que se cree un registro de seguimiento de solicitud con error; los pasos siguientes muestran cómo ver o descargar el registro.

3. En Visual Studio, en la pestaña **Configuración** de la ventana **Aplicación web de Azure**, haga clic en **Abrir en el Portal de administración**.

4. En la hoja [Azure Portal](https://portal.azure.com)**Configuración** de la aplicación, haga clic en **Credenciales de implementación** y luego especifique un nuevo nombre de usuario y contraseña.

    ![Nuevo nombre de usuario y contraseña de FTP](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-enterftpcredentials.png)

    > [!NOTE]
    > Cuando inicie sesión, debe utilizar el nombre de usuario completo, con el nombre de la aplicación como prefijo. Por ejemplo, si escribe "myid" como nombre de usuario y el sitio es "myexample", inicie sesión como "myexample\myid".
    >

5. En una ventana nueva del explorador, vaya a la dirección URL que aparece en **Nombre de host de FTP** o **Nombre de host de FTPS** en la página **Información general** de la aplicación.

6. Inicie sesión con las credenciales de FTP que creó anteriormente (el nombre de usuario debe incluir como prefijo el nombre de la aplicación).

    El explorador muestra la carpeta raíz de la aplicación.

7. Abra la carpeta *LogFiles* .

    ![Abrir la carpeta LogFiles](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-logfilesfolder.png)

8. Abra la carpeta llamada W3SVC más un valor numérico.

    ![Abrir la carpeta W3SVC](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-w3svcfolder.png)

    La carpeta contiene archivos XML de todo error que se haya registrado después de haber habilitado el seguimiento de solicitudes con error, además de un archivo XSL que un explorador puede utilizar para dar formato al XML.

    ![Carpeta W3SVC](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-w3svcfoldercontents.png)

9. Haga clic en el archivo XML de la solicitud con error para la que desea ver información de seguimiento.

    La siguiente ilustración muestra parte de la información de seguimiento de un error de muestra.

    ![Seguimiento de solicitudes con error en un explorador](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-failedrequestinbrowser.png)

## <a name="next-steps"></a><a name="nextsteps"></a>Pasos siguientes
Ha visto cómo Visual Studio facilita la visualización de registros creados por una aplicación de App Service. Las secciones siguientes proporcionan vínculos a más recursos sobre temas relacionados:

* Solución de problemas de App Service
* Depuración en Visual Studio
* Depuración remota en Azure
* Seguimiento en aplicaciones de ASP.NET
* Análisis de registros de servidor web
* Análisis de registros de seguimiento de solicitudes con error
* Depuración de Cloud Services

### <a name="app-service-troubleshooting"></a>Solución de problemas de App Service
Para obtener más información sobre cómo solucionar problemas de aplicaciones en Azure App Service, vea los siguientes recursos:

* [Control de Supervisión de aplicaciones en Azure App Service](web-sites-monitor.md)
* [Investigating Memory Leaks in Azure App Service with Visual Studio 2013](https://devblogs.microsoft.com/devops/investigating-memory-leaks-in-azure-web-sites-with-visual-studio-2013/) (Investigación de fugas de memoria en Azure App Service con Visual Studio 2013). Artículo del blog ALM de Microsoft sobre las características de Visual Studio para el análisis de problemas de memoria administrada.
* [Azure App Service online tools you should know about](https://azure.microsoft.com/blog/2014/03/28/windows-azure-websites-online-tools-you-should-know-about-2/) (Herramientas en línea de Azure App Service que debe conocer). Publicación en el blog de Amit Apple.

Si desea obtener ayuda para una pregunta específica sobre la solución de problemas, comience una conversación en alguno de los siguientes foros:

* [El foro sobre Azure en el sitio de ASP.NET](https://forums.asp.net/1247.aspx/1?Azure+and+ASP+NET).
* [El foro de Azure en Microsoft Q&A](https://docs.microsoft.com/answers/topics/azure-webapps.html).
* [StackOverflow.com](https://www.stackoverflow.com).

### <a name="debugging-in-visual-studio"></a>Depuración en Visual Studio
Para más información acerca del uso del modo de depuración en Visual Studio, consulte [Depuración en Visual Studio](/visualstudio/debugger/debugging-in-visual-studio) y [Sugerencias de depuración de Visual Studio 2010](https://weblogs.asp.net/scottgu/archive/2010/08/18/debugging-tips-with-visual-studio-2010.aspx).

### <a name="remote-debugging-in-azure"></a>Depuración remota en Azure
Para obtener más información sobre la depuración remota para aplicaciones de App Service y WebJobs, vea los siguientes recursos:

* [Introduction to Remote Debugging Azure App Service](https://azure.microsoft.com/blog/2014/05/06/introduction-to-remote-debugging-on-azure-web-sites/) (Introducción a la depuración remota de Azure App Service).
* [Introduction to Remote Debugging Azure App Service part 2 - Inside Remote debugging](https://azure.microsoft.com/blog/2014/05/07/introduction-to-remote-debugging-azure-web-sites-part-2-inside-remote-debugging/) (Introducción a la depuración remota en Azure App Service, Parte 2: detalles de la depuración remota)
* [Introduction to Remote Debugging on Azure App Service part 3 - Multi-Instance environment and GIT](https://azure.microsoft.com/blog/2014/05/08/introduction-to-remote-debugging-on-azure-web-sites-part-3-multi-instance-environment-and-git/) (Introducción a la depuración remota en Azure App Service, Parte 3: entorno de varias instancias y GIT)
* [Depuración de WebJobs](https://www.youtube.com/watch?v=ncQm9q5ZFZs&list=UU_SjTh-ZltPmTYzAybypB-g&index=1)

Si su aplicación utiliza una API web de Azure o el back-end de Mobile Services y necesita depurarlo, vea [Debugging .NET Backend in Visual Studio](/archive/blogs/azuremobile/debugging-net-backend-in-visual-studio) (Depuración de back-end de .NET en Visual Studio).

### <a name="tracing-in-aspnet-applications"></a>Seguimiento en aplicaciones de ASP.NET
No hay introducciones rigurosas ni actualizadas al seguimiento de ASP.NET disponibles en Internet. Lo mejor que puede hacer es comenzar con materiales de introducción antiguos escritos para Web Forms, porque MVC todavía no existía, y complementarlos con las publicaciones más recientes en blogs que se centran en temas específicos. Algunos buenos sitios para comenzar son los siguientes recursos:

* [Supervisión y telemetría (crear aplicaciones en la nube para el mundo real con Azure)](https://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/monitoring-and-telemetry).<br>
  Capítulo de un libro electrónico con recomendaciones para realizar seguimiento en aplicaciones de la nube de Azure.
* [Seguimiento de ASP.NET](/previous-versions/dotnet/articles/ms972204(v=msdn.10))<br/>
  Un recurso antiguo, pero siempre útil, con una introducción básica al tema.
* [Agentes de escucha de seguimiento](/dotnet/framework/debug-trace-profile/trace-listeners)<br/>
  Brinda información acerca de los agentes de escucha de seguimiento, pero no menciona [WebPageTraceListener](/dotnet/api/system.web.webpagetracelistener).
* [Tutorial: Integración del seguimiento de ASP.NET con el seguimiento de System.Diagnostics](/previous-versions/b0ectfxd(v=vs.140))<br/>
  Este artículo también es antiguo, pero incluye cierta información adicional que el artículo de introducción no incluye.
* [Seguimiento en vistas Razor de ASP.NET MVC](https://devblogs.microsoft.com/aspnet/tracing-in-asp-net-mvc-razor-views/)<br/>
  Además del seguimiento en vistas de Razor, la publicación también explica cómo crear un filtro de error para registrar todas las excepciones no manejadas en una aplicación MVC. Si desea obtener información sobre cómo registrar todas las excepciones no controladas en una aplicación de Web Forms, consulte el ejemplo Global.asax en [Ejemplo completo de controladores de errores en MSDN](/previous-versions/bb397417(v=vs.140)) . Ya sea en MVC o en Web Forms, si desea registrar ciertas excepciones, pero permitir que la gestión del marco predeterminado las afecte, puede volver a intentarlo como en el siguiente ejemplo:

    ```csharp
    try
    {
       // Your code that might cause an exception to be thrown.
    }
    catch (Exception ex)
    {
        Trace.TraceError("Exception: " + ex.ToString());
        throw;
    }
    ```

* [Registro de seguimiento de diagnóstico de streaming desde la línea de comandos de Azure (con Glimpse)](https://www.hanselman.com/blog/StreamingDiagnosticsTraceLoggingFromTheAzureCommandLinePlusGlimpse.aspx)<br/>
  Uso de la línea de comandos para hacer lo que muestra este tutorial en Visual Studio. [Glimpse](https://www.hanselman.com/blog/IfYoureNotUsingGlimpseWithASPNETForDebuggingAndProfilingYoureMissingOut.aspx) es una herramienta de depuración de aplicaciones ASP.NET.
* [Uso de diagnósticos y registro de Web Apps - con David Ebbo](https://azure.microsoft.com/documentation/videos/azure-web-site-logging-and-diagnostics/) y [Registros de streaming desde Web Apps - con David Ebbo](https://azure.microsoft.com/documentation/videos/log-streaming-with-azure-web-sites/)<br>
  Vídeos de Scott Hanselman y David Ebbo.

Para el registro de errores, una alternativa a escribir su propio código de seguimiento es utilizar un marco de registro de código abierto, como [ELMAH](https://nuget.org/packages/elmah/). Para obtener más información, consulte [Publicaciones de blog de Scott Hanselman sobre ELMAH](https://www.hanselman.com/blog/NuGetPackageOfTheWeek7ELMAHErrorLoggingModulesAndHandlersWithSQLServerCompact.aspx).

Además, no es necesario realizar un seguimiento de ASP.NET o de `System.Diagnostics` para obtener registros de streaming de Azure. El servicio de registros de streaming de la aplicación de App Service transmite todo archivo *.txt*, *.html* o *.log* que encuentra en la carpeta *LogFiles*. Por tanto, podría crear su propio sistema de registro que escriba en el sistema de archivos de la aplicación y su archivo se transmitirá y descargará automáticamente. Todo lo que tiene que hacer es escribir código de aplicación que cree archivos en la carpeta *d:\home\logfiles*.

### <a name="analyzing-web-server-logs"></a>Análisis de registros de servidor web
Para obtener más información acerca del análisis de registros de servidor web, consulte los siguientes recursos:

* [LogParser](https://www.microsoft.com/download/details.aspx?id=24659)<br/>
  Una herramienta para visualizar datos en registros de servidor web (archivos *.log* ).
* [Solución de problemas de rendimiento de IIS o errores de aplicación al usar LogParser](https://www.iis.net/learn/troubleshoot/performance-issues/troubleshooting-iis-performance-issues-or-application-errors-using-logparser)<br/>
  Una introducción a la herramienta de analizador del registro que puede utilizar para analizar registros de servidor web.
* [Publicaciones en el blog de Robert McMurray sobre el uso de LogParser](/archive/blogs/robert_mcmurray/using-logparser-with-ftp-7-x-sessions)<br/>
* [El código de estado HTTP en IIS 7.0, IIS 7.5 y IIS 8.0](https://support.microsoft.com/kb/943891)

### <a name="analyzing-failed-request-tracing-logs"></a>Análisis de registros de seguimiento de solicitudes con error
El sitio web de Microsoft TechNet incluye una sección llamada [Uso de seguimiento de solicitudes erróneas](https://www.iis.net/learn/troubleshoot/using-failed-request-tracing), que puede ser muy práctica para comprender cómo utilizar estos registros. Pero esta documentación se centra en la configuración del seguimiento de solicitudes con error en IIS, algo que no puede hacer en Azure App Service.

[GetStarted]: quickstart-dotnetcore.md?pivots=platform-windows
[GetStartedWJ]: https://github.com/Azure/azure-webjobs-sdk/wiki