---
title: 'Tutorial: Ejecución de Azure Functions en trabajos de Azure Stream Analytics'
description: En este tutorial aprenderá a configurar Azure Functions como un receptor de salida para los trabajos de Stream Analytics.
author: mamccrea
ms.author: mamccrea
ms.service: stream-analytics
ms.topic: tutorial
ms.custom: mvc, devx-track-csharp
ms.date: 01/27/2020
ms.openlocfilehash: 70ea5ec9ee91fdba8023b9c6af1ce65b691a17fb
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89006897"
---
# <a name="tutorial-run-azure-functions-from-azure-stream-analytics-jobs"></a>Tutorial: Ejecución de Azure Functions desde trabajos de Azure Stream Analytics 

Puede ejecutar Azure Functions desde Azure Stream Analytics mediante la configuración de Functions como uno de los receptores de salida para el trabajo de Stream Analytics. Functions es una experiencia de procesos a petición orientada a eventos que permite implementar el código desencadenado por los eventos que se producen en servicios de Azure o en servicios de terceros. Esta capacidad que tiene Functions para responder a los desencadenadores hace que sea una salida natural para los trabajos de Stream Analytics.

Stream Analytics llama a Functions mediante desencadenadores HTTP. El adaptador de salida de Functions permite a los usuarios conectar instancias de Functions a Stream Analytics, de manera que los eventos se puedan desencadenar basándose en consultas de Stream Analytics. 

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear y ejecutar un trabajo de Stream Analytics.
> * Creación de una instancia de Azure Redis Cache
> * Creación de una Función de Azure
> * Búsqueda de resultados en Azure Redis Cache

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="configure-a-stream-analytics-job-to-run-a-function"></a>Configuración de un trabajo de Stream Analytics para que ejecute una función 

En esta sección se muestra cómo configurar un trabajo de Stream Analytics para ejecutar una función que escribe datos en Azure Redis Cache. El trabajo de Stream Analytics lee eventos de Azure Event Hubs y ejecuta una consulta que invoca a la función. Esta función lee los datos del trabajo de Stream Analytics y los escribe en Azure Redis Cache.

![Diagrama que muestra las relaciones entre los servicios de Azure](./media/stream-analytics-with-azure-functions/image1.png)

## <a name="create-a-stream-analytics-job-with-event-hubs-as-input"></a>Creación de un trabajo de Stream Analytics con Event Hubs como entrada

Siga el tutorial [Detección de fraudes en tiempo real](stream-analytics-real-time-fraud-detection.md) para crear un centro de eventos, iniciar la aplicación del generador de eventos y crear un trabajo de Stream Analytics. Omita los pasos para crear la consulta y la salida. En su lugar, consulte las secciones siguientes para configurar la salida de Azure Functions.

## <a name="create-an-azure-cache-for-redis-instance"></a>Creación de una instancia de Azure Redis Cache

1. Cree una caché en Azure Redis Cache mediante los pasos descritos en [Creación de una caché](../azure-cache-for-redis/cache-dotnet-how-to-use-azure-redis-cache.md#create-a-cache).  

2. Después de crear la caché, en **Configuración**, seleccione **Teclas de acceso**. Anote la **cadena de conexión principal**.

   ![Captura de pantalla de la cadena de conexión de Azure Redis Cache](./media/stream-analytics-with-azure-functions/image2.png)

## <a name="create-a-function-in-azure-functions-that-can-write-data-to-azure-cache-for-redis"></a>Creación de una función en Azure Functions que pueda escribir datos en Azure Redis Cache

1. Consulte la sección [Creación de una aplicación de función](../azure-functions/functions-create-first-azure-function.md#create-a-function-app) de la documentación de Functions. En esta sección se explica cómo crear una aplicación de funciones y una [función desencadenada por HTTP en Azure Functions](../azure-functions/functions-create-first-azure-function.md#create-function) mediante el lenguaje CSharp.  

2. Vaya a la función **run.csx**. Actualícela con el código siguiente. Reemplace **"\<your Azure Cache for Redis connection string goes here\>"** por la cadena de conexión principal de Azure Cache for Redis que recuperó en la sección anterior. 

    ```csharp
    using System;
    using System.Net;
    using System.Threading.Tasks;
    using StackExchange.Redis;
    using Newtonsoft.Json;
    using System.Configuration;

    public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
    {
        log.Info($"C# HTTP trigger function processed a request. RequestUri={req.RequestUri}");
    
        // Get the request body
        dynamic dataArray = await req.Content.ReadAsAsync<object>();

        // Throw an HTTP Request Entity Too Large exception when the incoming batch(dataArray) is greater than 256 KB. Make sure that the size value is consistent with the value entered in the Stream Analytics portal.

        if (dataArray.ToString().Length > 262144)
        {
            return new HttpResponseMessage(HttpStatusCode.RequestEntityTooLarge);
        }
        var connection = ConnectionMultiplexer.Connect("<your Azure Cache for Redis connection string goes here>");
        log.Info($"Connection string.. {connection}");
    
        // Connection refers to a property that returns a ConnectionMultiplexer
        IDatabase db = connection.GetDatabase();
        log.Info($"Created database {db}");
    
        log.Info($"Message Count {dataArray.Count}");

        // Perform cache operations using the cache object. For example, the following code block adds few integral data types to the cache
        for (var i = 0; i < dataArray.Count; i++)
        {
            string time = dataArray[i].time;
            string callingnum1 = dataArray[i].callingnum1;
            string key = time + " - " + callingnum1;
            db.StringSet(key, dataArray[i].ToString());
            log.Info($"Object put in database. Key is {key} and value is {dataArray[i].ToString()}");
       
            // Simple get of data types from the cache
            string value = db.StringGet(key);
            log.Info($"Database got: {value}");
        }

        return req.CreateResponse(HttpStatusCode.OK, "Got");
    }

   ```

   Cuando Stream Analytics recibe una excepción "La entidad de solicitud HTTP es demasiado grande" de una función, disminuye el tamaño de los lotes que envía a Azure Functions. El código siguiente garantiza que Stream Analytics no envía lotes demasiado grandes. Asegúrese de que los valores de tamaño y número máximo de lotes que se usan en la función sean coherentes con los valores que se ingresan en el portal de Stream Analytics.

    ```csharp
    if (dataArray.ToString().Length > 262144)
        {
            return new HttpResponseMessage(HttpStatusCode.RequestEntityTooLarge);
        }
   ```

3. En el editor de texto de su elección, cree un archivo JSON denominado **project.json**. Pegue el código siguiente y guárdelo en el equipo local. Este archivo contiene las dependencias de paquetes NuGet requeridas por la función de C#.  
   
    ```json
    {
        "frameworks": {
            "net46": {
                "dependencies": {
                    "StackExchange.Redis":"1.1.603",
                    "Newtonsoft.Json": "9.0.1"
                }
            }
        }
    }

   ```
 
4. Vuelva a Azure Portal. Desde la pestaña **Características de la plataforma**, vaya a la función. En **Herramientas de desarrollo**, seleccione **Editor de App Service**. 
 
   ![Captura de pantalla del editor de App Service](./media/stream-analytics-with-azure-functions/image3.png)

5. En Editor de App Service, haga clic con el botón derecho en el directorio raíz y cargue el archivo **project.json**. Cuando la carga se haya realizado correctamente, actualice la página. Ahora debería ver un archivo generado automáticamente llamado **project.lock.json**. El archivo generado automáticamente contiene referencias a los archivos .dll que se especifican en el archivo project.json.  

   ![Captura de pantalla del editor de App Service](./media/stream-analytics-with-azure-functions/image4.png)

## <a name="update-the-stream-analytics-job-with-the-function-as-output"></a>Actualización del trabajo de Stream Analytics con la función como salida

1. En Azure Portal, abra el trabajo de Stream Analytics.  

2. Vaya a la función y seleccione **Información general** > **Salidas** > **Agregar**. Para agregar una nueva salida, seleccione **Función de Azure** para la opción de receptor. El adaptador de salida de Functions tiene las propiedades siguientes:  

   |**Nombre de propiedad**|**Descripción**|
   |---|---|
   |Alias de salida| Nombre descriptivo que se usará en la consulta del trabajo para hacer referencia a la salida. |
   |Opción de importación| Puede utilizar la función de la suscripción actual o proporcionar la configuración manualmente si la función se encuentra en otra suscripción. |
   |Function App| Nombre de la aplicación de Functions |
   |Función| Nombre de la función de la aplicación de Functions (nombre de la función run.csx).|
   |Tamaño máximo de lote|Establece el tamaño máximo de cada lote de salida que se envía a la función, en bytes. De manera predeterminada, este valor se establece en 262 144 bytes (256 KB).|
   |Número máximo de lotes|Especifica el número máximo de eventos de cada lote que se envía a la función. El valor predeterminado es 100. Esta propiedad es opcional.|
   |Clave|Permite usar una función de otra suscripción. Proporcione el valor de clave para acceder a la función. Esta propiedad es opcional.|

3. Proporcione un nombre para el alias de salida. En este tutorial, se denomina **saop1**, pero puede usar el nombre que prefiera. Rellene otros detalles.

4. Abra el trabajo de Stream Analytics y actualice la consulta a lo siguiente. Si no asignó el nombre **saop1** al receptor de salida, recuerde modificarlo en la consulta.  

   ```sql
    SELECT
            System.Timestamp as Time, CS1.CallingIMSI, CS1.CallingNum as CallingNum1,
            CS2.CallingNum as CallingNum2, CS1.SwitchNum as Switch1, CS2.SwitchNum as Switch2
        INTO saop1
        FROM CallStream CS1 TIMESTAMP BY CallRecTime
           JOIN CallStream CS2 TIMESTAMP BY CallRecTime
            ON CS1.CallingIMSI = CS2.CallingIMSI AND DATEDIFF(ss, CS1, CS2) BETWEEN 1 AND 5
        WHERE CS1.SwitchNum != CS2.SwitchNum
   ```

5. Inicie la aplicación telcodatagen.exe ejecutando el comando siguiente en la línea de comandos. El comando usa el formato `telcodatagen.exe [#NumCDRsPerHour] [SIM Card Fraud Probability] [#DurationHours]`.  
   
   ```cmd
   telcodatagen.exe 1000 0.2 2
   ```
    
6.  Inicie el trabajo de Stream Analytics.

## <a name="check-azure-cache-for-redis-for-results"></a>Búsqueda de resultados en Azure Redis Cache

1. Vaya a Azure Portal y encuentre su instancia de Azure Redis Cache. Seleccione **Consola**.  

2. Use [ Azure Redis Cache](https://redis.io/commands) para comprobar que los datos están Azure Redis Cache. (El comando toma el formato Get {key}). Por ejemplo:

   **Get "12/19/2017 21:32:24 - 123414732"**

   Este comando debe imprimir el valor de la clave especificada:

   ![Captura de pantalla de la salida de Azure Redis Cache](./media/stream-analytics-with-azure-functions/image5.png)

## <a name="error-handling-and-retries"></a>Control de errores y reintentos

Si se produce algún error al enviar eventos a Azure Functions, Stream Analytics reintentará la mayoría de las operaciones. Todas las excepciones HTTP se reintentan hasta que se ejecutan correctamente con la excepción del error HTTP 413 (la entidad es demasiado grande). Un error de que una entidad demasiado grande se trata como un error de datos, que está sujeto a la [directiva de reintento o de eliminación](stream-analytics-output-error-policy.md).

> [!NOTE]
> El tiempo de espera de las solicitudes HTTP de Stream Analytics a Azure Functions se establece en 100 segundos. Si la aplicación de Azure Functions tarda más de 100 segundos en procesar un lote, Stream Analytics genera un error.

## <a name="known-issues"></a>Problemas conocidos

En Azure Portal, al intentar restablecer el valor de Tamaño máximo de lotes o Número máximo de lotes en valor vacío (predeterminado), el valor vuelve a cambiar al valor especificado anteriormente al guardar. Especifique manualmente los valores predeterminados para estos campos en este caso.

El uso de [enrutamiento HTTP](https://docs.microsoft.com/sandbox/functions-recipes/routes?tabs=csharp) en Azure Functions actualmente no es compatible con Stream Analytics.

No está habilitada la compatibilidad con la conexión a Azure Functions hospedado en una red virtual.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando no los necesite, elimine el grupo de recursos, el trabajo de streaming y todos los recursos relacionados. La eliminación del trabajo evita la facturación de las unidades de streaming utilizadas por el trabajo. Si piensa utilizar el trabajo en el futuro, puede detenerlo y volver a iniciarlo más adelante cuando sea necesario. Si no va a seguir usando este trabajo, siga estos pasos para eliminar todos los recursos creados en esta guía de inicio rápido:

1. En el menú de la izquierda de Azure Portal, haga clic en **Grupos de recursos** y en el nombre del recurso que creó.  
2. En la página del grupo de recursos, haga clic en **Eliminar**, escriba en el cuadro de texto el nombre del recurso que quiere eliminar y haga clic en **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado un trabajo de Stream Analytics simple que ejecuta una función una instancia de Azure Functions. Para más información sobre los trabajos de Stream Analytics, continúe con el tutorial siguiente:

> [!div class="nextstepaction"]
> [Ejecución de funciones definidas por el usuario en JavaScript en los trabajos de Stream Analytics](stream-analytics-javascript-user-defined-functions.md)
