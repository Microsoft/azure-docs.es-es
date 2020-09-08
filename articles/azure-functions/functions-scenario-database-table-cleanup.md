---
title: Uso de Azure Functions para realizar una tarea de limpieza de base de datos
description: Use Azure Functions para programar una tarea que se conecte a Azure SQL Database a fin de limpiar filas de forma periódica.
ms.assetid: 076f5f95-f8d2-42c7-b7fd-6798856ba0bb
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 10/02/2019
ms.openlocfilehash: 0b5e255d7d108eb063ece4e5489a8762261a0bed
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88207267"
---
# <a name="use-azure-functions-to-connect-to-an-azure-sql-database"></a>Uso de Azure Functions para conectarse a una base de datos de Azure SQL Database

En este artículo se indica cómo usar Azure Functions para crear un trabajo programado que se conecte a una instancia de Azure SQL Database o una instancia administrada de Azure SQL. El código de función limpia las filas de una tabla de la base de datos. La nueva función de C# se crea a partir de una plantilla de desencadenador del temporizador predefinida de Visual Studio 2019. Para que este escenario sea posible, también debe establecer una cadena de conexión de base de datos como una configuración de aplicación en la aplicación de función. En el caso de la instancia administrada de Azure SQL, debe [habilitar el punto de conexión público](../azure-sql/managed-instance/public-endpoint-configure.md) para poder conectarse desde Azure Functions. En este escenario se utiliza una operación masiva en la base de datos. 

Si es la primera vez que trabaja con funciones de C#, debe leer la [referencia para desarrolladores de C# de Azure Functions](functions-dotnet-class-library.md).

## <a name="prerequisites"></a>Requisitos previos

+ Complete los pasos del artículo [Creación de la primera función mediante Visual Studio](functions-create-your-first-function-visual-studio.md) para crear una aplicación de funciones local que tenga como destino la versión 2.x o una posterior del entorno en tiempo de ejecución. También debe haber publicado el proyecto en una aplicación de función en Azure.

+ En este artículo se realiza una demostración de un comando de Transact-SQL que ejecuta una operación de limpieza masiva en la tabla **SalesOrderHeader** de la base de datos AdventureWorksLT de ejemplo. Para crear la base de datos de ejemplo AdventureWorksLT, efectúe los pasos indicados en el artículo [Creación de una base de datos de Azure SQL Database mediante Azure Portal](../azure-sql/database/single-database-create-quickstart.md).

+ Debe agregar una [regla de firewall de nivel de servidor](../azure-sql/database/firewall-create-server-level-portal-quickstart.md) para la dirección IP pública del equipo que usa para seguir este inicio rápido. Esta regla debe ser capaz de acceder a la instancia de SQL Database desde el equipo local.  

## <a name="get-connection-information"></a>Obtención de información sobre la conexión

Deberá obtener la cadena de conexión de la base de datos que creó una vez concluidos los pasos de [Creación de una base de datos en Azure SQL Database mediante Azure Portal](../azure-sql/database/single-database-create-quickstart.md).

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. Seleccione **SQL Database** en el menú de la izquierda y seleccione la base de datos en la página **SQL Database**.

1. Seleccione **Cadenas de conexión**  en **Configuración** y copie la cadena de conexión de **ADO.NET** completa. Para la instancia administrada de Azure SQL, copie la cadena de conexión para el punto de conexión público.

    ![Copie la cadena de conexión de ADO.NET.](./media/functions-scenario-database-table-cleanup/adonet-connection-string.png)

## <a name="set-the-connection-string"></a>Establecimiento de la cadena de conexión

Una aplicación de función hospeda la ejecución de sus funciones en Azure. Como práctica de seguridad recomendada, almacene las cadenas de conexión y otros secretos en la configuración de la aplicación de función. El uso de la configuración de la aplicación impide divulgaciones accidentales de la cadena de conexión con el código. Puede acceder a la configuración de la aplicación para la aplicación de función directamente desde Visual Studio.

Anteriormente debe haber publicado la aplicación en Azure. Si todavía no lo ha hecho, [publique la aplicación de función en Azure](functions-develop-vs.md#publish-to-azure).

1. En el Explorador de soluciones, haga clic con el botón derecho en el proyecto de aplicación de funciones y elija **Publicar** > **Editar configuración de Azure App Service**. Seleccione **Agregar configuración**, en **Nuevo nombre de configuración de aplicación**, escriba `sqldb_connection` y seleccione **Aceptar**.

    ![Configuración de la aplicación para la aplicación de función.](./media/functions-scenario-database-table-cleanup/functions-app-service-add-setting.png)

1. En la nueva configuración **sqldb_connection**, pegue la cadena de conexión que copió en la sección anterior en el campo **Local** y reemplace los marcadores de posición `{your_username}` y `{your_password}` por valores reales. Seleccione **Insertar un valor desde Local** para copiar el valor actualizado en el campo **Remoto** y seleccione **Aceptar**.

    ![Incorporación de una configuración de cadena de conexión SQL.](./media/functions-scenario-database-table-cleanup/functions-app-service-settings-connection-string.png)

    Las cadenas de conexión se almacenan cifradas en Azure (**Remoto**). Para evitar la pérdida de secretos, el archivo de proyecto local.settings.json (**Local**) se debe excluir de control de código fuente, por ejemplo, utilizando un archivo .gitignore.

## <a name="add-the-sqlclient-package-to-the-project"></a>Incorporación del paquete SqlClient al proyecto

Necesita agregar el paquete NuGet que contiene la biblioteca de SqlClient. Esta biblioteca de acceso de datos es necesaria para conectarse a SQL Database.

1. Abra el proyecto de aplicación de funciones local en Visual Studio 2019.

1. En el Explorador de soluciones, haga clic con el botón derecho en proyecto de la aplicación de función y elija **Administrar paquetes NuGet**.

1. En la pestaña **Examinar**, busque ```System.Data.SqlClient``` y selecciónelo.

1. En la página **System.Data.SqlClient**, seleccione la versión `4.5.1` y, a continuación, haga clic en **Instalar**.

1. Cuando finalice la instalación, revise los cambios y, a continuación, haga clic en **Aceptar** para cerrar la ventana **Vista previa**.

1. Si aparece una ventana de **Aceptación de licencia**, haga clic en **Acepto**.

Ahora, puede agregar el código de función de C# que conecta con SQL Database.

## <a name="add-a-timer-triggered-function"></a>Adición de una función desencadenada por el temporizador

1. En el Explorador de soluciones, haga clic con el botón derecho en proyecto de la aplicación de función y elija **Agregar** > **Nueva función de Azure**.

1. Con la plantilla **Azure Functions** seleccionada, asigne un nombre al nuevo elemento parecido a `DatabaseCleanup.cs` y seleccione **Agregar**.

1. En el diálogo cuadro **Nueva función de Azure**, elija **Desencadenador de temporizador** y luego **Aceptar**. Este cuadro de diálogo crea un archivo de código para la función desencadenada por el temporizador.

1. Abra el nuevo archivo de código y agregue las siguientes instrucciones using en la parte superior de dicho archivo:

    ```cs
    using System.Data.SqlClient;
    using System.Threading.Tasks;
    ```

1. Reemplace la función `Run` existente por el código siguiente:

    ```cs
    [FunctionName("DatabaseCleanup")]
    public static async Task Run([TimerTrigger("*/15 * * * * *")]TimerInfo myTimer, ILogger log)
    {
        // Get the connection string from app settings and use it to create a connection.
        var str = Environment.GetEnvironmentVariable("sqldb_connection");
        using (SqlConnection conn = new SqlConnection(str))
        {
            conn.Open();
            var text = "UPDATE SalesLT.SalesOrderHeader " +
                    "SET [Status] = 5  WHERE ShipDate < GetDate();";

            using (SqlCommand cmd = new SqlCommand(text, conn))
            {
                // Execute the command and log the # rows affected.
                var rows = await cmd.ExecuteNonQueryAsync();
                log.LogInformation($"{rows} rows were updated");
            }
        }
    }
    ```

    Esta función se ejecuta cada 15 segundos para actualizar la columna `Status` según la fecha de envío. Para más información sobre el desencadenador de temporizador, consulte [Desencadenador de temporizador para Azure Functions](functions-bindings-timer.md).

1. Presione **F5** para iniciar la aplicación de función. Se abrirá la ventana de ejecución de [Azure Functions Core Tools](functions-develop-local.md) detrás de Visual Studio.

1. A los 15 segundos del inicio, se ejecuta la función. Observe la salida y tenga en cuenta el número de filas actualizadas en la tabla **SalesOrderHeader**.

    ![Vista de los registros de función.](./media/functions-scenario-database-table-cleanup/function-execution-results-log.png)

    En la primera ejecución, debería actualizar 32 filas de datos. Las ejecuciones siguientes no actualizan ninguna fila de datos, a menos que realice cambios en los datos de la tabla SalesOrderHeader para que la instrucción `UPDATE` seleccione más filas.

Si tiene previsto [publicar esta función](functions-develop-vs.md#publish-to-azure), no olvide cambiar el atributo `TimerTrigger` a una [programación cron](functions-bindings-timer.md#ncrontab-expressions) más razonable que cada quince segundos.

## <a name="next-steps"></a>Pasos siguientes

A continuación, obtenga información sobre cómo usar Functions con Logic Apps para la integración con otros servicios.

> [!div class="nextstepaction"]
> [Creación de una función que se integre con Logic Apps](functions-twitter-email.md)

Para más información sobre Functions, vea los siguientes artículos:

+ [Referencia para desarrolladores de Azure Functions](functions-reference.md)  
  contiene las referencias del programador para codificar funciones y definir desencadenadores y enlaces.
+ [Prueba de Azure Functions](functions-test-a-function.md)  
  describe las diversas herramientas y técnicas para probar sus funciones.  
