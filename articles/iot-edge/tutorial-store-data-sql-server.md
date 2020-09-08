---
title: 'Tutorial: Almacenamiento de datos con el módulo de SQL mediante Azure IoT Edge'
description: En este tutorial se indica cómo almacenar datos localmente en el dispositivo IoT Edge con un módulo de SQL Server
services: iot-edge
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 08/04/2020
ms.topic: tutorial
ms.service: iot-edge
ms.custom: mvc, devx-track-csharp
ms.openlocfilehash: 5de5224ca85d2421182c9721d3b6394e10b43863
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88997955"
---
# <a name="tutorial-store-data-at-the-edge-with-sql-server-databases"></a>Tutorial: Almacenamiento de datos en el perímetro con bases de datos de SQL Server

Implemente un módulo de SQL Server para almacenar datos en un dispositivo Linux en el que se ejecuta Azure IoT Edge.

Use Azure IoT Edge y SQL Server para almacenar y consultar datos en el perímetro. Azure IoT Edge incluye funcionalidades de almacenamiento básicas que almacenan mensajes en la memoria caché cuando un dispositivo se queda sin conexión y que, posteriormente, los reenvían cuando se restablece esta. No obstante, puede que también quiera funcionalidades de almacenamiento más avanzadas como, por ejemplo, poder consultar datos de forma local. Los dispositivos de IoT Edge pueden usar bases de datos locales para realizar procesos más complejos sin tener que mantener una conexión con IoT Hub.

Este artículo proporciona instrucciones para implementar una base de datos de SQL Server en un dispositivo IoT Edge. Azure Functions, ejecutándose en el dispositivo IoT Edge, estructura los datos de entrada y a continuación los envía a la base de datos. Los pasos descritos en este artículo también pueden aplicarse a otras bases de datos que funcionan en contenedores, como MySQL o PostgreSQL.

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Usar Visual Studio Code para crear una instancia de Azure Functions
> * Implementar una base de datos SQL en el dispositivo IoT Edge
> * Usar Visual Studio Code para compilar módulos e implementarlos en el dispositivo IoT Edge
> * Visualización de datos generados

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar este tutorial, debe haber realizado el anterior para configurar el entorno de desarrollo de contenedores de Linux: [Desarrollo de módulos IoT Edge para dispositivos Linux](tutorial-develop-for-linux.md). Al completar ese tutorial, se deben cumplir los siguientes requisitos previos:

* Una instancia de [IoT Hub](../iot-hub/iot-hub-create-through-portal.md) de nivel estándar o gratis en Azure.
* Un [dispositivo Linux que ejecute Azure IoT Edge](quickstart-linux.md) que sea AMD64.
  * Los dispositivos ARM, como Raspberry PI, no pueden ejecutar SQL Server. Si quiere usar SQL en un dispositivo ARM, puede registrarse para probar la versión preliminar de [Azure SQL Edge](https://azure.microsoft.com/services/sql-edge/).
* Un registro de contenedor, como [Azure Container Registry](https://docs.microsoft.com/azure/container-registry/).
* [Visual Studio Code](https://code.visualstudio.com/) configurado con [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).
* [CE de Docker](https://docs.docker.com/install/) configurado para ejecutar contenedores de Linux.

En este tutorial se usa un módulo de Azure Functions para enviar datos a SQL Server. Para desarrollar un módulo de IoT Edge con Azure Functions, instale los siguientes requisitos previos adicionales en la máquina de desarrollo:

* [Extensión de C# para Visual Studio Code (con tecnología de OmniSharp) para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp).
* [SDK de .NET Core 2.1](https://www.microsoft.com/net/download).

## <a name="create-a-function-project"></a>Creación de un proyecto de aplicación de una función

Para enviar datos a una base de datos, necesita un módulo que pueda estructurar los datos correctamente y que, después, los almacene en una tabla.

### <a name="create-a-new-project"></a>Creación de un nuevo proyecto

En los siguientes pasos puede ver cómo crear una función de IoT Edge mediante Visual Studio Code y Azure IoT Tools.

1. Abra Visual Studio Code.

2. Para abrir la paleta de comandos de VS Code, seleccione **View** > **Command palette** (Ver > Paleta de comandos).

3. En la paleta de comandos, escriba y ejecute el comando **Azure IoT Edge: New IoT Edge solution** (Azure IoT Edge: nueva solución de IoT Edge). En la paleta de comandos, proporcione la siguiente información para crear la solución:

   | Campo | Value |
   | ----- | ----- |
   | Seleccionar carpeta | Elija la ubicación en el equipo de desarrollo en la que VS Code creará los archivos de la solución. |
   | Proporcionar un nombre de la solución | Escriba un nombre descriptivo para la solución, como **SqlSolution** o acepte el valor predeterminado. |
   | Seleccionar plantilla del módulo | Seleccione **Azure Functions: C#** . |
   | Proporcionar un nombre de módulo | Asigne el nombre **sqlFunction** al módulo. |
   | Proporcionar repositorio de imágenes de Docker del módulo | Un repositorio de imágenes incluye el nombre del registro de contenedor y el nombre de la imagen de contenedor. La imagen de contenedor se rellena previamente a partir del último paso. Reemplace **localhost:5000** por el valor de **Servidor de inicio de sesión** del registro de contenedor de Azure. Puede recuperar el servidor de inicio de sesión en la página de información general del registro de contenedor en Azure Portal. <br><br>La cadena final es como esta: \<registry name\>.azurecr.io/sqlfunction. |

   El área de trabajo de la solución de IoT Edge se carga en la ventana de Visual Studio Code.

### <a name="add-your-registry-credentials"></a>Adición de las credenciales del Registro

El archivo del entorno almacena las credenciales del registro de contenedor y las comparte con el runtime de IoT Edge. El entorno de ejecución necesita estas credenciales para extraer las imágenes privadas e insertarlas en el dispositivo IoT Edge.

La extensión de IoT Edge intenta extraer de Azure las credenciales del registro del contenedor y rellenar con ellas el archivo de entorno. Compruebe si las credenciales ya están incluidas. Si no lo están, agréguelas ahora:

1. En el explorador de VS Code, abra el archivo .env.
2. Actualice los campos con los valores de **nombre de usuario** y **contraseña** que ha copiado del Registro de contenedor de Azure.
3. Guarde este archivo.

### <a name="select-your-target-architecture"></a>Selección de la arquitectura de destino

Actualmente, Visual Studio Code puede desarrollar módulos de C para dispositivos Linux AMD64 y Linux ARM32v7. Debe seleccionar qué arquitectura tiene como destino con cada solución, dado que el contenedor se crea y ejecuta de manera diferente para cada tipo de arquitectura. La opción predeterminada es Linux AMD64.

1. Abra la paleta de comandos y busque **Azure IoT Edge: Set Default Target Platform for Edge Solution** (Establecer la plataforma de destino predeterminada para la solución perimetral), o bien seleccione el icono de acceso directo en la barra lateral en la parte inferior de la ventana.

2. En la paleta de comandos, seleccione la arquitectura de destino en la lista de opciones. Para este tutorial, usamos una máquina virtual Ubuntu como dispositivo IoT Edge, por lo que mantendrá el valor predeterminado **amd64**.

### <a name="update-the-module-with-custom-code"></a>Actualización del módulo con código personalizado

1. En el explorador de VS Code, abra **modules** > **sqlFunction** > **sqlFunction.csproj**.

2. Busque el grupo de referencias de paquete y agregue uno nuevo que incluya a SqlClient.

   ```csproj
   <PackageReference Include="System.Data.SqlClient" Version="4.5.1"/>
   ```

3. Guarde el archivo **sqlFunction.csproj**.

4. Abra el archivo **sqlFunction.cs**.

5. Reemplace todo el contenido del archivo con el código siguiente:

   ```csharp
   using System;
   using System.Collections.Generic;
   using System.IO;
   using System.Text;
   using System.Threading.Tasks;
   using Microsoft.Azure.Devices.Client;
   using Microsoft.Azure.WebJobs;
   using Microsoft.Azure.WebJobs.Extensions.EdgeHub;
   using Microsoft.Azure.WebJobs.Host;
   using Microsoft.Extensions.Logging;
   using Newtonsoft.Json;
   using Sql = System.Data.SqlClient;

   namespace Functions.Samples
   {
       public static class sqlFunction
       {
           [FunctionName("sqlFunction")]
           public static async Task FilterMessageAndSendMessage(
               [EdgeHubTrigger("input1")] Message messageReceived,
               [EdgeHub(OutputName = "output1")] IAsyncCollector<Message> output,
               ILogger logger)
           {
               const int temperatureThreshold = 20;
               byte[] messageBytes = messageReceived.GetBytes();
               var messageString = System.Text.Encoding.UTF8.GetString(messageBytes);

               if (!string.IsNullOrEmpty(messageString))
               {
                   logger.LogInformation("Info: Received one non-empty message");
                   // Get the body of the message and deserialize it.
                   var messageBody = JsonConvert.DeserializeObject<MessageBody>(messageString);

                   //Store the data in SQL db
                   const string str = "<sql connection string>";
                   using (Sql.SqlConnection conn = new Sql.SqlConnection(str))
                   {
                       conn.Open();
                       var insertMachineTemperature = "INSERT INTO MeasurementsDB.dbo.TemperatureMeasurements VALUES (CONVERT(DATETIME2,'" + messageBody.timeCreated + "', 127), 'machine', " + messageBody.machine.temperature + ");";
                       var insertAmbientTemperature = "INSERT INTO MeasurementsDB.dbo.TemperatureMeasurements VALUES (CONVERT(DATETIME2,'" + messageBody.timeCreated + "', 127), 'ambient', " + messageBody.ambient.temperature + ");";
                       using (Sql.SqlCommand cmd = new Sql.SqlCommand(insertMachineTemperature + "\n" + insertAmbientTemperature, conn))
                       {
                           //Execute the command and log the # rows affected.
                           var rows = await cmd.ExecuteNonQueryAsync();
                           logger.LogInformation($"{rows} rows were updated");
                       }
                   }

                   if (messageBody != null && messageBody.machine.temperature > temperatureThreshold)
                   {
                       // Send the message to the output as the temperature value is greater than the threashold.
                       using (var filteredMessage = new Message(messageBytes))
                       {
                            // Copy the properties of the original message into the new Message object.
                            foreach (KeyValuePair<string, string> prop in messageReceived.Properties)
                            {filteredMessage.Properties.Add(prop.Key, prop.Value);}
                            // Add a new property to the message to indicate it is an alert.
                            filteredMessage.Properties.Add("MessageType", "Alert");
                            // Send the message.
                            await output.AddAsync(filteredMessage);
                            logger.LogInformation("Info: Received and transferred a message with temperature above the threshold");
                       }
                   }
               }
           }
       }
       //Define the expected schema for the body of incoming messages.
       class MessageBody
       {
           public Machine machine {get; set;}
           public Ambient ambient {get; set;}
           public string timeCreated {get; set;}
       }
       class Machine
       {
           public double temperature {get; set;}
           public double pressure {get; set;}
       }
       class Ambient
       {
           public double temperature {get; set;}
           public int humidity {get; set;}
       }
   }
   ```

6. En la línea 35, reemplace la cadena **\<sql connection string\>** por la cadena siguiente. La propiedad **Origen de datos** hace referencia al contenedor de SQL Server, que todavía no existe. Lo creará con el nombre **SQL** en la sección siguiente.

   ```csharp
   Data Source=tcp:sql,1433;Initial Catalog=MeasurementsDB;User Id=SA;Password=Strong!Passw0rd;TrustServerCertificate=False;Connection Timeout=30;
   ```

7. Guarde el archivo **sqlFunction.cs**.

## <a name="add-the-sql-server-container"></a>Incorporación de un contenedor de SQL Server

Un [manifiesto de implementación](module-composition.md) declara qué módulos del entorno de ejecución de IoT Edge se instalarán en el dispositivo IoT Edge. En la sección anterior proporcionó el código para hacer un módulo de función personalizado, pero el módulo de SQL Server ya está integrado y disponible en Azure Marketplace. Solo debe indicar el entorno de ejecución de IoT Edge para incluirlo y configurarlo en el dispositivo.

1. En Visual Studio Code, abra la paleta de comandos; para ello, seleccione **View** > **Command palette** (Ver > Paleta de comandos).

2. En la paleta de comandos, escriba y ejecute el comando **Azure IoT Edge: Add IoT Edge Module** (Azure IoT Edge: agregar módulo IoT Edge). En la paleta de comandos, proporcione la siguiente información para agregar un módulo nuevo:

   | Campo | Value |
   | ----- | ----- |
   | Seleccionar archivo de la plantilla de implementación | La paleta de comandos resalta el archivo **deployment.template.json** en su carpeta de soluciones actual. Seleccione ese archivo.  |
   | Seleccionar plantilla del módulo | Seleccione **Módulo de Azure Marketplace**. |

3. En Marketplace de módulos de Azure IoT Edge, busque y seleccione **Módulo de SQL Server**.

4. Cambie el nombre del módulo por **sql**, en minúsculas. Este nombre coincide con el del contenedor declarado en la cadena de conexión en el archivo sqlFunction.cs.

5. Seleccione **Importar** para agregar el módulo a la solución.

6. En la carpeta de la solución, abra el archivo **deployment.template.json**.

7. Busque la sección **modules**. Debería ver tres módulos. El módulo *SimulatedTemperatureSensor* se incluye de forma predeterminada en nuevas soluciones y proporciona datos de prueba para usar con los otros módulos. El módulo *sqlFunction* es el módulo que ha creado inicialmente y se actualiza con nuevo código. Por último, el módulo *sql* se importó desde Azure Marketplace.

   >[!Tip]
   >El módulo de SQL Server viene con una contraseña predeterminada establecida en las variables de entorno del manifiesto de implementación. Cada vez que cree un contenedor de SQL Server en un entorno de producción, debería [cambiar la contraseña de administrador del sistema predeterminada](https://docs.microsoft.com/sql/linux/quickstart-install-connect-docker).

8. Cierre el archivo **deployment.template.json**.

## <a name="build-your-iot-edge-solution"></a>Compilación de la solución de IoT Edge

En las secciones anteriores, ha creado una solución con un módulo y, después, agregó otro a la plantilla del manifiesto de implementación. Microsoft hospeda públicamente el módulo de SQL Server, pero debe incluir el código en el módulo Functions. En esta sección, compilará la solución, creará imágenes de contenedor para el módulo sqlFunction e insertará las imágenes en el registro de contenedor.

1. En Visual Studio Code, abra el terminal integrado; para ello, seleccione **Ver** > **Terminal**.  

1. Inicie sesión en el registro de contenedor en Visual Studio Code para que pueda insertar las imágenes en el registro. Use las mismas credenciales de Azure Container Registry (ACR) que ha agregado al archivo .env. En el terminal integrado, escriba el siguiente comando:

    ```csh/sh
    docker login -u <ACR username> -p <ACR password> <ACR login server>
    ```

    También puede ver una advertencia de seguridad que recomienda el uso del parámetro --password-stdin. Cuando su uso esté fuera del ámbito de este artículo, recomendamos seguir este procedimiento recomendado. Para más información, vea la referencia del comando [docker login](https://docs.docker.com/engine/reference/commandline/login/#provide-a-password-using-stdin).

1. En el explorador de VS Code, haga clic con el botón derecho en el archivo **deployment.template.json** y seleccione **Build and Push IoT Edge solution** (Compilar e insertar solución de IoT Edge).

   El comando de compilación e inserción inicia tres operaciones. En primer lugar, se crea una nueva carpeta en la solución llamada **config**, que contiene los archivos del manifiesto de la implementación completa, con la información de la plantilla de implementación y otros archivos de la solución. En segundo lugar, ejecuta `docker build` para generar la imagen de contenedor basándose en el Dockerfile adecuado para la arquitectura de destino. A continuación, ejecuta `docker push` para insertar el repositorio de imágenes en el registro de contenedor.

   Este proceso puede tardar varios minutos la primera vez, pero es más rápido la próxima vez que ejecute los comandos.

   Puede comprobar que el módulo sqlFunction se insertó correctamente en el registro de contenedor. En Azure Portal, vaya al registro de contenedor. Seleccione **repositorios** y busque **sqlFunction**. Los otros dos módulos, SimulatedTemperatureSensor y SQL, no se insertarán en el registro de contenedor porque sus repositorios ya están en los Registros de Microsoft.

## <a name="deploy-the-solution-to-a-device"></a>Implementación de la solución en un dispositivo

Puede establecer módulos en un dispositivo con IoT Hub, pero también puede acceder a este y a los dispositivos mediante Visual Studio Code. En esta sección puede configurar el acceso a IoT Hub y, posteriormente, usar VS Code para implementar la solución en el dispositivo IoT Edge.

1.  En el explorador de Visual Studio Code, en la sección **Azure IoT Hub**, expanda **Dispositivos** para ver la lista de dispositivos IoT.

2. Haga clic con el botón derecho en el dispositivo que desea que sea el destino de su implementación y seleccione **Create Deployment for Single Device** (Crear una implementación para un dispositivo individual).

3. Seleccione el archivo **deployment.amd64.json** en la carpeta **config** y, a continuación, haga clic en **Select Edge Deployment Manifest** (Seleccionar manifiesto de implementación de Edge). No utilice el archivo deployment.template.json.

4. En el dispositivo, expanda **Módulos** para ver una lista de módulos implementados y en ejecución. Haga clic en el botón Actualizar. Debería ver que los módulos nuevos **sql** y **sqlFunction** se ejecutan junto con el módulo **SimulatedTemperatureSensor** y **$edgeAgent** y **$edgeHub**.

    También puede comprobar que todos los módulos están activados y ejecutándose en el dispositivo. En el dispositivo IoT Edge, ejecute el siguiente comando para ver el estado de los módulos.

   ```cmd/sh
   iotedge list
   ```

    Los módulos pueden tardar unos minutos en iniciarse. El entorno de ejecución de Azure IoT Edge necesita recibir su nuevo manifiesto de implementación, extraer las imágenes de los módulos del entorno de ejecución del contenedor y, después, iniciar cada nuevo módulo.

## <a name="create-the-sql-database"></a>Creación de la base de datos SQL

Cuando aplica el manifiesto de implementación al dispositivo, consigue la ejecución de tres módulos. El módulo SimulatedTemperatureSensor genera datos en un entorno simulado. El módulo sqlFunction toma los datos y les da un formato para una base de datos. Esta sección le guiará en el proceso de configuración de la base de datos SQL para almacenar los datos de temperatura.

Ejecute los siguientes comandos en el dispositivo de IoT Edge. Estos comandos se conectan al módulo **sql** que se ejecuta en el dispositivo y crean una base de datos y una tabla donde se almacenarán los datos de temperatura que se le envían.

1. En una herramienta de línea de comandos en el dispositivo de IoT Edge, conéctese a la base de datos.

      ```bash
      sudo docker exec -it sql bash
      ```

2. Abra la herramienta de comando SQL.

      ```bash
      /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P 'Strong!Passw0rd'
      ```

3. Cree la base de datos:

      ```sql
      CREATE DATABASE MeasurementsDB
      ON
      (NAME = MeasurementsDB, FILENAME = '/var/opt/mssql/measurementsdb.mdf')
      GO
      ```

4. Defina la tabla.

   ```sql
   CREATE TABLE MeasurementsDB.dbo.TemperatureMeasurements (measurementTime DATETIME2, location NVARCHAR(50), temperature FLOAT)
   GO
   ```

Puede personalizar el archivo de Docker de SQL Server para configurar automáticamente SQL Server para que se implemente en varios dispositivos IoT Edge. Para más información, consulte el [proyecto de demostración de contenedor de Microsoft SQL Server](https://github.com/twright-msft/mssql-node-docker-demo-app).

## <a name="view-the-local-data"></a>Visualización de los datos locales

Después de que la tabla se crea, el módulo sqlFunction empieza a almacenar datos en una base de datos local de SQL Server 2017 del dispositivo IoT Edge.

Desde dentro de la herramienta de comando SQL, ejecute el siguiente comando para ver los datos de tabla con formato:

   ```sql
   SELECT * FROM MeasurementsDB.dbo.TemperatureMeasurements
   GO
   ```

   ![Visualización del contenido de la base de datos local](./media/tutorial-store-data-sql-server/view-data.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Si prevé seguir con el siguiente artículo recomendado, puede mantener los recursos y las configuraciones que ya ha creado y volverlos a utilizar. También puede seguir usando el mismo dispositivo de IoT Edge como dispositivo de prueba.

En caso contrario, para evitar gastos, puede eliminar las configuraciones locales y los recursos de Azure que creó en este artículo.

[!INCLUDE [iot-edge-clean-up-cloud-resources](../../includes/iot-edge-clean-up-cloud-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado un módulo de Azure Functions que contiene código para filtrar datos sin procesar generados por el dispositivo IoT Edge. Cuando esté listo para compilar sus propios módulos, consulte [Develop Azure Functions with Azure IoT Edge for Visual Studio Code](how-to-develop-csharp-function.md) (Desarrollo de Azure Functions con Azure IoT Edge para Visual Studio Code).

Si desea utilizar otro método de almacenamiento en el perímetro, obtenga información sobre cómo usar Azure Blob Storage en IoT Edge.

> [!div class="nextstepaction"]
> [Almacenamiento de datos en el perímetro con Azure Blob Storage en IoT Edge](how-to-store-data-blob.md)
