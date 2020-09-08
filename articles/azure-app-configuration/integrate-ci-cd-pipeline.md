---
title: Integración de Azure App Configuration mediante una canalización de entrega e integración continuas
description: Aprenda a implementar una entrega e integración continuas mediante Azure App Configuration
services: azure-app-configuration
author: lisaguthrie
ms.service: azure-app-configuration
ms.topic: tutorial
ms.custom: devx-track-csharp
ms.date: 01/30/2020
ms.author: lcozzens
ms.openlocfilehash: f0d1e57c7e212fefc6e17a8170e3b4537b190f60
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88211639"
---
# <a name="integrate-with-a-cicd-pipeline"></a>Integración con una canalización de CI/CD

En este artículo se explica cómo usar datos de Azure App Configuration en un sistema de integración e implementación continuas.

## <a name="use-app-configuration-in-your-azure-devops-pipeline"></a>Use App Configuration en la canalización de Azure DevOps

Si tiene una canalización de Azure DevOps, puede capturar valores de clave de App Configuration y establecerlos como variables de la tarea. La [extensión de DevOps de Azure App Configuration](https://go.microsoft.com/fwlink/?linkid=2091063) es un módulo adicional que proporciona esta funcionalidad. Para usar la extensión en una secuencia de tareas de una compilación o versión, siga sus instrucciones.

## <a name="deploy-app-configuration-data-with-your-application"></a>Implementación de datos de App Configuration con la aplicación

La aplicación puede no ejecutarse si depende de Azure App Configuration y no puede acceder a él. Mejore la resistencia de la aplicación mediante el empaquetado de los datos de configuración en un archivo que se implementa con la aplicación y se carga localmente durante el inicio de la aplicación. Este enfoque garantiza que la aplicación tenga al menos los valores de configuración predeterminados en el inicio. Estos valores se sobrescriben con los cambios más recientes en un almacén de App Configuration cuando está disponible.

Mediante la función [Exportar](./howto-import-export-data.md#export-data) de Azure App Configuration, puede automatizar el proceso de recuperar los datos de configuración actuales como un único archivo. A continuación, puede insertar este archivo en un paso de compilación o implementación en la canalización de integración continua e implementación continua (CI/CD).

En el ejemplo siguiente se muestra cómo incluir los datos de App Configuration como un paso de compilación para la aplicación web que se introdujo en los inicios rápidos. Antes de continuar, finalice primero el tutorial [Creación de una aplicación ASP.NET Core con Azure App Configuration](./quickstart-aspnet-core-app.md).

Para realizar los pasos de este tutorial, puede usar cualquier editor de código. [Visual Studio Code](https://code.visualstudio.com/) es una excelente opción disponible en las plataformas Windows, macOS y Linux.

### <a name="prerequisites"></a>Requisitos previos

Si compila localmente, descargue e instale la [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) si aún no lo ha hecho.

Para realizar una compilación en la nube, por ejemplo, con Azure DevOps, asegúrese de que la [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) está instalada en el sistema de compilación.

### <a name="export-an-app-configuration-store"></a>Exportación de un almacén de App Configuration

1. Abra su archivo *.csproj* y agregue el siguiente script:

    ```xml
    <Target Name="Export file" AfterTargets="Build">
        <Message Text="Export the configurations to a temp file. " />
        <Exec WorkingDirectory="$(MSBuildProjectDirectory)" Condition="$(ConnectionString) != ''" Command="az appconfig kv export -d file --path $(OutDir)\azureappconfig.json --format json --separator : --connection-string $(ConnectionString)" />
    </Target>
    ```
1. Abra *Program.cs* y actualice el método `CreateWebHostBuilder` para que use el archivo JSON exportado, para lo que es preciso llamar al método `config.AddJsonFile()`.  Agregue también el espacio de nombres `System.Reflection`.

    ```csharp
    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .ConfigureAppConfiguration((hostingContext, config) =>
            {
                var directory = System.IO.Path.GetDirectoryName(Assembly.GetExecutingAssembly().Location);
                var settings = config.Build();

                config.AddJsonFile(Path.Combine(directory, "azureappconfig.json"));
                config.AddAzureAppConfiguration(settings["ConnectionStrings:AppConfig"]);
            })
            .UseStartup<Startup>();
    ```

### <a name="build-and-run-the-app-locally"></a>Compilación y ejecución de la aplicación en un entorno local

1. Establezca una variable de entorno llamada **ConnectionString** y defínala como la clave de acceso a su almacén de App Configuration. 
    Si usa el símbolo del sistema de Windows, ejecute el siguiente comando y reinícielo para que se aplique el cambio:

    ```console
     setx ConnectionString "connection-string-of-your-app-configuration-store"
    ```

    Si usa Windows PowerShell, ejecute el siguiente comando:

    ```powershell
     $Env:ConnectionString = "connection-string-of-your-app-configuration-store"
    ```

    Si usa macOS o Linux, ejecute el siguiente comando:

    ```console
     export ConnectionString='connection-string-of-your-app-configuration-store'
    ```

2. Para compilar la aplicación mediante la CLI de .NET Core, ejecute el siguiente comando en el shell de comandos:

    ```console
     dotnet build
    ```

3. Una vez que la compilación se haya realizado correctamente, ejecute el siguiente comando para ejecutar la aplicación web localmente:

    ```console
     dotnet run
    ```

4. Inicie una ventana del explorador y vaya a `http://localhost:5000`, que es la dirección URL predeterminada de la aplicación web hospedada localmente.

    ![Inicio de la aplicación del artículo de inicio rápido en un entorno local](./media/quickstarts/aspnet-core-app-launch-local.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha exportado los datos de Azure App Configuration para su uso en una canalización de implementación. Para más información sobre App Configuration, continúe con los ejemplos de la CLI de Azure.

> [!div class="nextstepaction"]
> [CLI de Azure](https://docs.microsoft.com/cli/azure/appconfig?view=azure-cli-latest)
