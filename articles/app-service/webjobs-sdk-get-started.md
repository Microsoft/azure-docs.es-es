---
title: Introducción al SDK de WebJobs
description: Introducción al SDK de WebJobs para el procesamiento en segundo plano basado en eventos. Aprenda a acceder a los datos de servicios de Azure y de terceros.
author: ggailey777
ms.devlang: dotnet
ms.topic: article
ms.date: 02/18/2019
ms.author: glenga
ms.openlocfilehash: a3b3e52b8386f212521f0fa1ca97cbd658374209
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88078315"
---
# <a name="get-started-with-the-azure-webjobs-sdk-for-event-driven-background-processing"></a>Introducción al SDK de Azure WebJobs para el procesamiento en segundo plano basado en eventos

En este artículo se muestra cómo usar Visual Studio 2019 para crear un proyecto de SDK de Azure WebJobs, ejecutarlo de forma local e implementarlo en [Azure App Service](overview.md). La versión 3.x del SDK de WebJobs es compatible con las aplicaciones de consola de .NET Core y .NET Framework. Para más información sobre cómo trabajar con el SDK de WebJobs, consulte [Uso del SDK de Azure WebJobs para el procesamiento en segundo plano basado en eventos](webjobs-sdk-how-to.md).

En este artículo se muestra cómo implementar WebJobs como una aplicación de consola de .NET Core. Para implementar WebJobs como una aplicación de consola de .NET Framework, consulte [WebJobs as .NET Framework console apps](webjobs-dotnet-deploy-vs.md#webjobs-as-net-framework-console-apps) (WebJobs como aplicaciones de consola de .NET Framework). Si está interesado en la versión 2.x del SDK de WebJobs, que solo es compatible con .NET Framework, consulte [Desarrollo e implementación de WebJobs mediante Visual Studio - Azure App Service](webjobs-dotnet-deploy-vs.md).

## <a name="prerequisites"></a>Prerrequisitos

* [Instale Visual Studio 2019](/visualstudio/install/) con la carga de trabajo de **desarrollo de Azure**. Si ya tiene Visual Studio pero no tiene esa carga de trabajo, agréguela mediante la selección de **Herramientas > Obtener herramientas y características**.

* Debe tener [una cuenta de Azure](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) para publicar el proyecto del SDK de WebJobs en Azure.

## <a name="create-a-project"></a>Crear un proyecto

1. En Visual Studio, seleccione **Crear un proyecto nuevo**.

2. Luego, seleccione **Aplicación de consola (.NET Core)** .

3. Asigne al proyecto el nombre *WebJobsSDKSample* y seleccione **Crear**.

   ![Cuadro de diálogo Nuevo proyecto](./media/webjobs-sdk-get-started/new-project.png)

## <a name="webjobs-nuget-packages"></a>Paquetes NuGet de WebJobs

1. Instale la versión 3.x estable más reciente del [`Microsoft.Azure.WebJobs.Extensions` paquete NuGet](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions/), que incluye `Microsoft.Azure.WebJobs`.

     Este es el comando de la **Consola del Administrador de paquetes**:

     ```powershell
     Install-Package Microsoft.Azure.WebJobs.Extensions -version <3_X_VERSION>
     ```

    En este comando, reemplace `<3_X_VERSION>` por una versión compatible del paquete. 

## <a name="create-the-host"></a>Creación del host

El host es el contenedor en tiempo de ejecución de las funciones que escucha a los desencadenadores y llama a las funciones. En los pasos siguientes se crea un host que implementa [`IHost`](/dotnet/api/microsoft.extensions.hosting.ihost), que es el host genérico en ASP.NET Core.

1. En *Program.cs*, agregue una instrucción `using`:

    ```cs
    using System.Threading.Tasks;
    using Microsoft.Extensions.Hosting;
    ```

1. Reemplace el método `Main` por el código siguiente:

    ```cs
    static async Task Main()
    {
        var builder = new HostBuilder();
        builder.ConfigureWebJobs(b =>
                {
                    b.AddAzureStorageCoreServices();
                });
        var host = builder.Build();
        using (host)
        {
            await host.RunAsync();
        }
    }
    ```

En ASP.NET Core, las configuraciones de host se establecen mediante la llamada a los métodos de la instancia [`HostBuilder`](/dotnet/api/microsoft.extensions.hosting.hostbuilder). Para más información, consulte [Host genérico .NET](/aspnet/core/fundamentals/host/generic-host). El método de extensión `ConfigureWebJobs` inicializa el host de WebJobs. En `ConfigureWebJobs`, se inicializan extensiones específicas de WebJobs y se establecen las propiedades de esas extensiones.  

## <a name="enable-console-logging"></a>Habilitación del registro de consola

En esta sección, se configura el registro de la consola que usa la [plataforma de registro de ASP.NET Core](/aspnet/core/fundamentals/logging).

1. Instale la versión estable más reciente del [`Microsoft.Extensions.Logging.Console` paquete NuGet](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Console/), que incluye `Microsoft.Extensions.Logging`.

   Este es el comando de la **Consola del Administrador de paquetes**:

   ```powershell
   Install-Package Microsoft.Extensions.Logging.Console -version <3_X_VERSION>
   ```
   En este comando, reemplace `<3_X_VERSION>` por una versión 3. x compatible del paquete.

1. En *Program.cs*, agregue una instrucción `using`:

   ```cs
   using Microsoft.Extensions.Logging;
   ```

1. Llame al método [`ConfigureLogging`](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.configurelogging) en [`HostBuilder`](/dotnet/api/microsoft.extensions.hosting.hostbuilder). El método [`AddConsole`](/dotnet/api/microsoft.extensions.logging.consoleloggerextensions.addconsole) agrega el registro de la consola a la configuración.

    ```cs
    builder.ConfigureLogging((context, b) =>
    {
        b.AddConsole();
    });
    ```

    El método `Main` tiene el aspecto siguiente:

    ```cs
    static async Task Main()
    {
        var builder = new HostBuilder();
        builder.ConfigureWebJobs(b =>
                {
                    b.AddAzureStorageCoreServices();
                });
        builder.ConfigureLogging((context, b) =>
                {
                    b.AddConsole();
                });
        var host = builder.Build();
        using (host)
        {
            await host.RunAsync();
        }
    }
    ```

    Esta actualización hace lo siguiente:

    * Deshabilita el [registro del panel](https://github.com/Azure/azure-webjobs-sdk/wiki/Queues#logs). El panel es una herramienta de supervisión heredada y no se recomienda el registro del panel para escenarios de producción de alto rendimiento.
    * Agrega el proveedor de consola con el [filtrado](webjobs-sdk-how-to.md#log-filtering) predeterminado.

Ahora, puede agregar una función desencadenada por los mensajes que llegan a una cola de Azure Storage.

## <a name="install-the-storage-binding-extension"></a>Instalación de la extensión de enlaces de Storage

A partir de la versión 3.x, es necesario instalar explícitamente la extensión de enlace de almacenamiento que requiere el SDK de WebJobs. En versiones anteriores, los enlaces de almacenamiento se incluían en el SDK.

1. Instale la versión estable más reciente del paquete NuGet [Microsoft.Azure.WebJobs.Extensions.Storage](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Storage), versión 3.x. 

    Este es el comando de la **Consola del Administrador de paquetes**:

    ```powershell
    Install-Package Microsoft.Azure.WebJobs.Extensions.Storage -Version <3_X_VERSION>
    ```
    
    En este comando, reemplace `<3_X_VERSION>` por una versión compatible del paquete. 

2. En el método de extensión `ConfigureWebJobs`, llame al método `AddAzureStorage` en la instancia [`HostBuilder`](/dotnet/api/microsoft.extensions.hosting.hostbuilder) para inicializar la extensión Storage. En este momento, el método `ConfigureWebJobs` es similar al ejemplo siguiente:

    ```cs
    builder.ConfigureWebJobs(b =>
                    {
                        b.AddAzureStorageCoreServices();
                        b.AddAzureStorage();
                    });
    ```

## <a name="create-a-function"></a>Creación de una función

1. Haga clic con el botón derecho en el proyecto, seleccione **Agregar** > **Nuevo elemento...** , elija **Clase**, asigne al nuevo archivo de clase C# el nombre *Functions.cs* y seleccione **Agregar**.

1. En Functions.cs, reemplace la plantilla generada por el código siguiente:
    
    ```cs
    using Microsoft.Azure.WebJobs;
    using Microsoft.Extensions.Logging;
    
    namespace WebJobsSDKSample
    {
        public class Functions
        {
            public static void ProcessQueueMessage([QueueTrigger("queue")] string message, ILogger logger)
            {
                logger.LogInformation(message);
            }
        }
    }
    ```

   El atributo `QueueTrigger` indica el tiempo de ejecución para llamar a esta función cuando se escribe un mensaje nuevo en una cola de Azure Storage denominada `queue`. El contenido del mensaje de cola se proporciona para el código del método en el parámetro `message`. El cuerpo del método es donde se procesan los datos de desencadenador. En este ejemplo, el código solo registra el mensaje.

   El parámetro `message` no tiene que ser una cadena. También puede enlazar a un objeto JSON, una matriz de bytes o un objeto [CloudQueueMessage](https://docs.microsoft.com/dotnet/api/microsoft.azure.storage.queue.cloudqueuemessage). Consulte [Uso del desencadenador de cola](/azure/azure-functions/functions-bindings-storage-queue-trigger?tabs=csharp#usage). Cada tipo de enlace (como colas, blobs o tablas) tiene un conjunto diferente de tipos de parámetros con los que puede enlazar.

## <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

El emulador de Azure Storage que se ejecuta localmente no tiene todas las características que necesita el SDK de WebJobs. Por lo tanto, en esta sección creará una cuenta de almacenamiento en Azure y configurará el proyecto para utilizarla. Si ya tiene una cuenta de almacenamiento, vaya al paso 6.

1. Abra el **Explorador de servidores** en Visual Studio e inicie sesión en Azure. Haga clic con el botón derecho en el nodo **Azure** y, después, seleccione **Conectar a la suscripción de Microsoft Azure**.

   ![Inicio de sesión en Azure](./media/webjobs-sdk-get-started/sign-in.png)

1. En el nodo **Azure**, en el **Explorador de servidores**, haga clic con el botón derecho en **Almacenamiento** y seleccione luego **Crear cuenta de almacenamiento**.

   ![Menú Crear cuenta de almacenamiento](./media/webjobs-sdk-get-started/create-storage-account-menu.png)

1. En el cuadro de diálogo **Crear cuenta de almacenamiento**, escriba un nombre único para la cuenta de almacenamiento.

1. Elija la misma **Región** en la que ha creado la aplicación de App Service o una región cercana.

1. Seleccione **Crear**.

   ![Crear cuenta de almacenamiento](./media/webjobs-sdk-get-started/create-storage-account.png)

1. En el nodo **Storage**, en el **Explorador de servidores**, seleccione la nueva cuenta de almacenamiento. En la ventana **Propiedades**, seleccione los puntos suspensivos ( **...** ) a la derecha del campo de valor **Cadena de conexión**.

   ![Puntos suspensivos de Cadena de conexión](./media/webjobs-sdk-get-started/conn-string-ellipsis.png)

1. Copie la cadena de conexión y guarde este valor en algún lugar del que pueda copiarlo fácilmente de nuevo.

   ![Copia de la cadena de conexión](./media/webjobs-sdk-get-started/copy-key.png)

## <a name="configure-storage-to-run-locally"></a>Configuración del almacenamiento para ejecutarse localmente

El SDK de WebJobs busca la cadena de conexión de almacenamiento en la Configuración de la aplicación en Azure. Cuando se ejecuta localmente, busca este valor en el archivo de configuración local o en variables de entorno.

1. Haga clic con el botón derecho en el proyecto, seleccione **Agregar** > **Nuevo elemento...** , elija **Archivo de configuración JSON de JavaScript**, asigne al nuevo archivo el nombre *appsettings.json* y seleccione **Agregar**. 

1. En el nuevo archivo, agregue un campo `AzureWebJobsStorage`, como en el ejemplo siguiente:

    ```json
    {
        "AzureWebJobsStorage": "{storage connection string}"
    }
    ```

1. Reemplace *{storage connection string}* (cadena de conexión de almacenamiento) por la cadena de conexión copiada anteriormente.

1. Seleccione el archivo *appsettings.json* en el Explorador de soluciones y, en la ventana **Propiedades**, establezca **Copiar en el directorio de salida** en **Copiar si es posterior**.

Más adelante, agregará la misma configuración de aplicación de cadena de conexión a la aplicación en Azure App Service.

## <a name="test-locally"></a>Prueba local

En esta sección, va a crear y ejecutar el proyecto localmente, y desencadenar la función mediante la creación de un mensaje de cola.

1. Presione **Ctrl+F5** para ejecutar el proyecto.

   La consola muestra que el tiempo de ejecución encuentra la función y está esperando mensajes de cola para desencadenarla. El host v3.x genera la siguiente salida:

   ```console
    info: Microsoft.Azure.WebJobs.Hosting.JobHostService[0]
          Starting JobHost
    info: Host.Startup[0]
          Found the following functions:
          WebJobsSDKSample.Functions.ProcessQueueMessage

    info: Host.Startup[0]
          Job host started
    Application started. Press Ctrl+C to shut down.
    Hosting environment: Development
    Content root path: C:\WebJobsSDKSample\WebJobsSDKSample\bin\Debug\netcoreapp2.1\
   ```

1. Cierre la ventana de la consola.

1. En el **Explorador de servidores** en Visual Studio, expanda el nodo de la nueva cuenta de almacenamiento y, después, haga clic con el botón derecho en **Colas**.

1. Seleccione **Crear cola**.

1. Escriba *queue* (cola) como nombre de la cola y seleccione luego **Aceptar**.

   ![Crear cola](./media/webjobs-sdk-get-started/create-queue.png)

1. Haga clic con el botón derecho en el nodo de la nueva cola y seleccione **Ver cola**.

1. Seleccione el icono **Agregar mensaje**.

   ![Crear cola](./media/webjobs-sdk-get-started/create-queue-message.png)

1. En el cuadro de diálogo **Agregar mensaje**, escriba *¡Hola mundo!* como **Texto del mensaje**y, a continuación, seleccione **Aceptar**. Ahora hay un mensaje en la cola.

   ![Crear cola](./media/webjobs-sdk-get-started/hello-world-text.png)

1. Vuelva a ejecutar el proyecto.

   Debido a que ha utilizado el atributo `QueueTrigger` en la función `ProcessQueueMessage`, el tiempo de ejecución del SDK de WeJobs escucha los mensajes de cola cuando se inicia. Encuentra un nuevo mensaje de cola en la cola denominada *queue* (cola) y llama a la función.

   Debido al [retroceso exponencial del sondeo de la cola](/azure/azure-functions/functions-bindings-storage-queue-trigger?tabs=csharp#polling-algorithm), el tiempo de ejecución podría tardar hasta 2 minutos en encontrar el mensaje e invocar la función. Este tiempo de espera se puede reducir mediante la ejecución en [modo de desarrollo](webjobs-sdk-how-to.md#host-development-settings).

   La salida de la consola tendrá un aspecto similar al siguiente:

   ```console
    info: Function.ProcessQueueMessage[0]
          Executing 'Functions.ProcessQueueMessage' (Reason='New queue message detected on 'queue'.', Id=2c319369-d381-43f3-aedf-ff538a4209b8)
    info: Function.ProcessQueueMessage[0]
          Trigger Details: MessageId: b00a86dc-298d-4cd2-811f-98ec39545539, DequeueCount: 1, InsertionTime: 1/18/2019 3:28:51 AM +00:00
    info: Function.ProcessQueueMessage.User[0]
          Hello World!
    info: Function.ProcessQueueMessage[0]
          Executed 'Functions.ProcessQueueMessage' (Succeeded, Id=2c319369-d381-43f3-aedf-ff538a4209b8)
   ```

1. Cierre la ventana de la consola. 

1. Vuelva a la ventana Cola y actualícela. El mensaje desaparece, puesto que lo ha procesado la función que se ejecuta localmente. 

## <a name="add-application-insights-logging"></a>Incorporación del registro de Application Insights

Cuando el proyecto se ejecuta en Azure, no puede supervisar la ejecución de la función mediante la visualización de la salida de la consola. La solución de supervisión recomendada es [Application Insights](../azure-monitor/app/app-insights-overview.md). Para más información, consulte [Supervisión de Azure Functions](../azure-functions/functions-monitoring.md).

En esta sección, realiza las tareas siguientes para configurar el registro de Application Insights antes de implementarlo en Azure:

* Asegúrese de tener una aplicación de App Service y una instancia de Application Insights para trabajar con ellas.
* Configure la aplicación de App Service para usar la instancia de Application Insights y la cuenta de almacenamiento creadas anteriormente.
* Configure el proyecto para el registro de Application Insights.

### <a name="create-app-service-app-and-application-insights-instance"></a>Creación de la aplicación de App Service y la instancia de Application Insights

1. Si aún no tiene una aplicación de App Service que pueda usar, [cree una](quickstart-dotnet-framework.md). Cuando se crea la aplicación, también puede crear un recurso de Application Insights conectado. Al hacerlo, `APPINSIGHTS_INSTRUMENTATIONKEY` se establece automáticamente en la aplicación.

1. Si aún no tiene un recurso de Application Insights que pueda utilizar, [cree uno](../azure-monitor/app/create-new-resource.md ). Establezca **Tipo de aplicación** en **General** y omita las secciones que siguen a **Copy the instrumentation key** (Copiar la clave de instrumentación).

1. Si ya tiene un recurso de Application Insights que desee usar, [copie la clave de instrumentación](../azure-monitor/app/create-new-resource.md#copy-the-instrumentation-key).

### <a name="configure-app-settings"></a>Configuración de aplicaciones 

1. En el **Explorador de servidores** en Visual Studio, expanda el nodo **App Service** en **Azure**.

1. Expanda el grupo de recursos que se encuentra en la aplicación de App Service y, a continuación, haga clic con el botón derecho en la aplicación de App Service.

1. Seleccione **Ver configuración**.

1. En el cuadro de diálogo **Cadenas de conexión**, agregue la entrada siguiente.

   |Nombre  |Cadena de conexión  |Tipo de base de datos|
   |---------|---------|------|
   |AzureWebJobsStorage | {cadena de conexión de almacenamiento copiada anteriormente}|Personalizado|

1. Si el cuadro de diálogo **Configuración de la aplicación** no tiene una clave de instrumentación de Application Insights, agregue la que ha copiado anteriormente. (Es posible que la clave de instrumentación ya esté allí, en función de cómo haya creado la aplicación de App Service).

   |Nombre  |Value  |
   |---------|---------|
   |APPINSIGHTS_INSTRUMENTATIONKEY | {clave de instrumentación} |

1. Reemplace *{clave de instrumentación}* por la clave de instrumentación del recurso de Application Insights que esté utilizando.

1. Seleccione **Guardar**.

1. Agregue la conexión de Application Insights al proyecto para que pueda ejecutarlo localmente. En el archivo *appsettings.json*, agregue un campo `APPINSIGHTS_INSTRUMENTATIONKEY`, como en el ejemplo siguiente:

    ```json
    {
        "AzureWebJobsStorage": "{storage connection string}",
        "APPINSIGHTS_INSTRUMENTATIONKEY": "{instrumentation key}"
    }
    ```

    Reemplace *{clave de instrumentación}* por la clave de instrumentación del recurso de Application Insights que esté utilizando.

1. Guarde los cambios.

### <a name="add-application-insights-logging-provider"></a>Incorporación del proveedor de registro de Application Insights

Para aprovechar el registro de [Application Insights](../azure-monitor/app/app-insights-overview.md), actualice el código de registro para que haga lo siguiente:

* Agregue un proveedor de registro de Application Insights con [filtrado](webjobs-sdk-how-to.md#log-filtering) predeterminado. Cuando se ejecuta localmente, toda la información y los registros de nivel superior se escriben en la consola y en Application Insights.
* Coloque el objeto [LoggerFactory](./webjobs-sdk-how-to.md#logging-and-monitoring) en un bloque `using`para asegurarse de que la salida del registro se vacía al cerrarse el host.

1. Instale la versión 3.x estable más reciente del [`Microsoft.Azure.WebJobs.Logging.ApplicationInsights` paquete NuGet](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Logging.ApplicationInsights/).

   Este es el comando de la **Consola del Administrador de paquetes**:

   ```powershell
   Install-Package Microsoft.Azure.WebJobs.Logging.ApplicationInsights -Version <3_X_VERSION>
   ```
    En este comando, reemplace `<3_X_VERSION>` por una versión compatible del paquete.

1. Abra el archivo *Program.cs* y sustituya el código del método `Main` por el siguiente:

    ```cs
    static async Task Main()
    {
        var builder = new HostBuilder();
        builder.UseEnvironment(EnvironmentName.Development);
        builder.ConfigureWebJobs(b =>
                {
                    b.AddAzureStorageCoreServices();
                    b.AddAzureStorage();
                });
        builder.ConfigureLogging((context, b) =>
                {
                    b.AddConsole();

                    // If the key exists in settings, use it to enable Application Insights.
                    string instrumentationKey = context.Configuration["APPINSIGHTS_INSTRUMENTATIONKEY"];
                    if (!string.IsNullOrEmpty(instrumentationKey))
                    {
                        b.AddApplicationInsightsWebJobs(o => o.InstrumentationKey = instrumentationKey);
                    }
                });
        var host = builder.Build();
        using (host)
        {
            await host.RunAsync();
        }
    }
    ```

    Esto agrega el proveedor de Application Insights al registro mediante la clave que usted agregó anteriormente a la configuración de la aplicación.

## <a name="test-application-insights-logging"></a>Prueba del registro de Application Insights

En esta sección, volverá a realizar una ejecución local para comprobar que los datos de registro van ahora a Application Insights, así como a la consola.

1. Use el **Explorador de servidores** en Visual Studio para crear un mensaje de cola, de la misma manera que lo hizo [anteriormente](#test-locally), excepto en que va a escribir *¡Hola, App Insights!* como texto del mensaje.

1. Ejecute el proyecto.

   El SDK de WebJobs procesa el mensaje de cola y le permite ver los registros en la ventana de la consola.

1. Cierre la ventana de la consola.

1. Vaya a [Azure Portal](https://portal.azure.com/) para ver el recurso de Application Insights. Busque y seleccione **Application Insights**.

1. Elija una instancia de Application Insights.

1. Seleccione **Buscar**.

   ![Selección de Buscar](./media/webjobs-sdk-get-started/select-search.png)

1. Si no ve el mensaje *¡Hola App Insights!* , seleccione **Actualizar** periódicamente durante varios minutos. (Los registros no aparecen inmediatamente porque el cliente de Application Insights tarda un tiempo en vaciar los registros que procesa).

   ![Registros de Application Insights](./media/webjobs-sdk-get-started/logs-in-ai.png)

1. Cierre la ventana de la consola.

## <a name="deploy-to-azure"></a><a name="deploy-as-a-webjob"></a>Implementación en Azure

Durante la implementación, cree una instancia de App Service en la que se ejecutarán las funciones. Al publicar una aplicación de consola de .NET Core en App Service de Azure, este se ejecuta automáticamente como un WebJob. Para obtener más información sobre la publicación, consulte [Desarrollo e implementación de WebJobs mediante Visual Studio](webjobs-dotnet-deploy-vs.md).

[!INCLUDE [webjobs-publish-net-core](../../includes/webjobs-publish-net-core.md)]

## <a name="trigger-the-function-in-azure"></a>Desencadenamiento de la función en Azure

1. Asegúrese de que no está en una ejecución local (cierre la ventana la de consola si aún está abierta). En caso contrario, la instancia local podría ser la primera que va a procesar los mensajes de cola que cree.

1. En la página **Cola** de Visual Studio, agregue un mensaje a la cola como antes.

1. Actualice la página **Cola** y el nuevo mensaje desaparecerá porque se ha procesado por la función que se ejecuta en Azure.

   > [!TIP]
   > Cuando está realizando pruebas en Azure, utilice el [modo de desarrollo](webjobs-sdk-how-to.md#host-development-settings) para garantizar que se invoca a una función de desencadenador de cola de inmediato y evitar retrasos debido al [retroceso exponencial del sondeo de la cola](/azure/azure-functions/functions-bindings-storage-queue-trigger?tabs=csharp#polling-algorithm).

### <a name="view-logs-in-application-insights"></a>Visualización de registros en Application Insights

1. Abra [Azure Portal](https://portal.azure.com/) y vaya al recurso de Application Insights.

1. Seleccione **Buscar**.

1. Si no ve el mensaje *¡Hola Azure!* seleccione **Actualizar** periódicamente durante varios minutos.

   Ve los registros de la función que se ejecuta en un WebJob, incluido el texto *¡Hola Azure!* que ha escrito en la sección anterior.

## <a name="add-an-input-binding"></a>Incorporación de un enlace de entrada

Los enlaces de entrada simplifican el código que lee los datos. En este ejemplo, el mensaje de cola será un nombre de blob, y usará el nombre del blob para buscar y leer un blob en Azure Storage.

1. En *Functions.cs*, reemplace el método `ProcessQueueMessage` por el código siguiente:

   ```cs
   public static void ProcessQueueMessage(
       [QueueTrigger("queue")] string message,
       [Blob("container/{queueTrigger}", FileAccess.Read)] Stream myBlob,
       ILogger logger)
   {
       logger.LogInformation($"Blob name:{message} \n Size: {myBlob.Length} bytes");
   }
   ```

   En este código, `queueTrigger` es una [expresión de enlace](../azure-functions/functions-bindings-expressions-patterns.md), lo que significa que se resuelve con un valor diferente en tiempo de ejecución.  En tiempo de ejecución, tiene el contenido del mensaje de cola.

1. Agregue `using`:

   ```cs
   using System.IO;
   ```

1. Cree un contenedor de blobs en la cuenta de almacenamiento.

   a. En el **Explorador de servidores** en Visual Studio, expanda el nodo de la cuenta de almacenamiento, haga clic con el botón derecho en **Blobs** y, a continuación, seleccione **Crear contenedor de blobs**.

   b. En el cuadro de diálogo **Crear contenedor de blobs**, especifique *contenedor* como nombre para el contenedor y, después, haga clic en **Aceptar**.

1. Cargue el archivo *Program.cs* en el contenedor de blobs. (Este archivo se usa aquí como ejemplo; podría cargar cualquier archivo de texto y crear un mensaje de cola con el nombre del archivo).

   a. En el **Explorador de servidores**, haga doble clic en el nodo correspondiente al contenedor que ha creado.

   b. En la ventana **Contenedor**, seleccione el botón **Cargar**.

   ![Botón de carga de blobs](./media/webjobs-sdk-get-started/blob-upload-button.png)

   c. Busque y seleccione *Program.cs* y seleccione luego **Aceptar**.

1. Cree un mensaje de cola en la cola que ha creado anteriormente, con *Program.cs* como texto del mensaje.

   ![Mensaje de cola Program.cs](./media/webjobs-sdk-get-started/queue-msg-program-cs.png)

1. Ejecute el proyecto de forma local.

   El mensaje de cola desencadena la función, que lee el blob y registra su longitud. La salida de la consola tendrá un aspecto similar al siguiente:

   ```console
   Found the following functions:
   ConsoleApp1.Functions.ProcessQueueMessage
   Job host started
   Executing 'Functions.ProcessQueueMessage' (Reason='New queue message detected on 'queue'.', Id=5a2ac479-de13-4f41-aae9-1361f291ff88)
   Blob name:Program.cs
   Size: 532 bytes
   Executed 'Functions.ProcessQueueMessage' (Succeeded, Id=5a2ac479-de13-4f41-aae9-1361f291ff88)
   ```

## <a name="add-an-output-binding"></a>Adición de un enlace de salida

Los enlaces de salida simplifican el código que escribe los datos. Este ejemplo modifica el ejemplo anterior al escribir una copia del blob en lugar de registrar su tamaño. Los enlaces de Blob Storage se incluyen en el paquete de extensiones de Azure Storage que se instaló anteriormente.

1. Reemplace el método `ProcessQueueMessage` por el código siguiente:

   ```cs
   public static void ProcessQueueMessage(
       [QueueTrigger("queue")] string message,
       [Blob("container/{queueTrigger}", FileAccess.Read)] Stream myBlob,
       [Blob("container/copy-{queueTrigger}", FileAccess.Write)] Stream outputBlob,
       ILogger logger)
   {
       logger.LogInformation($"Blob name:{message} \n Size: {myBlob.Length} bytes");
       myBlob.CopyTo(outputBlob);
   }
   ```

1. Cree otro mensaje de cola con *Program.cs* como texto del mensaje.

1. Ejecute el proyecto de forma local.

   El mensaje de cola desencadena la función, que lee el blob, registra su longitud y crea un nuevo blob. La salida de la consola es la misma pero cuando vaya a la ventana del contenedor de blobs y seleccione **Actualizar**, verá un nuevo blob denominado *copy-Program.cs*.

## <a name="republish-the-updates-to-azure"></a>Nueva publicación de las actualizaciones en Azure

1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto y seleccione **Publicar**.

1. En el cuadro de diálogo **Publicar**, asegúrese de que el perfil actual esté seleccionado y elija **Publicar**. Los resultados de la publicación se detallan en la ventana **Salida**.
 
1. Compruebe la función en Azure volviendo a cargar un archivo en el contenedor de blobs y agregando un mensaje a la cola que es el nombre del archivo cargado. Verá que el mensaje se quita de la cola y una copia del archivo creado en el contenedor de blobs. 

## <a name="next-steps"></a>Pasos siguientes

En este artículo se ha mostrado cómo crear, ejecutar e implementar un proyecto del SDK 3.x de WebJobs.

> [!div class="nextstepaction"]
> [Más información sobre el SDK de WebJobs](webjobs-sdk-how-to.md)
