---
title: Tutorial para la integración con una canalización de integración y entrega continuas mediante Azure App Configuration | Microsoft Docs
description: En este tutorial, aprenderá a generar un archivo de configuración mediante los datos de Azure App Configuration durante la integración y entrega continuas.
services: azure-app-configuration
documentationcenter: ''
author: yegu-ms
manager: balans
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.topic: tutorial
ms.date: 02/24/2019
ms.author: yegu
ms.custom: mvc
ms.openlocfilehash: 26bd49af7245d6e6dde3162a2e1d95c54f13e35b
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65415935"
---
# <a name="integrate-with-a-cicd-pipeline"></a>Integración con una canalización de CI/CD

En este artículo se describen distintas formas de usar datos de Azure App Configuration en un sistema de integración e implementación continuas.

## <a name="use-app-configuration-in-your-azure-devops-pipeline"></a>Use App Configuration en la canalización de Azure DevOps

Si tiene una canalización de Azure DevOps, puede capturar valores de clave de App Configuration y establecerlos como variables de la tarea. La [extensión de DevOps de Azure App Configuration](https://go.microsoft.com/fwlink/?linkid=2091063) es un módulo adicional que proporciona esta funcionalidad. Para usar la extensión en una secuencia de tareas de una compilación o versión no tiene más que seguir sus instrucciones.

## <a name="deploy-app-configuration-data-with-your-application"></a>Implementación de datos de App Configuration con la aplicación

La aplicación puede no ejecutarse si depende de Azure App Configuration y no puede acceder a él. Puede mejorar la resistencia de la aplicación para tratar con este tipo de evento, aunque es poco probable que ocurra. Para ello, empaquete los datos de configuración actuales en un archivo que se implemente con la aplicación y se cargue localmente durante su inicio. Este enfoque garantiza que la aplicación tenga al menos los valores de configuración predeterminados. Estos valores se sobrescriben con los cambios más recientes en un almacén de configuración de aplicaciones cuando está disponible.

Mediante la función [Exportar](./howto-import-export-data.md#export-data) de Azure App Configuration, puede automatizar el proceso de recuperar los datos de configuración actuales como un único archivo. A continuación, inserte este archivo en un paso de compilación o implementación en la canalización de implementación continua e integración continua (CI/CD).

En el ejemplo siguiente se muestra cómo incluir los datos de App Configuration como un paso de compilación para la aplicación web que se introdujo en los inicios rápidos. Antes de continuar, finalice primero el tutorial [Creación de una aplicación ASP.NET Core con Azure App Configuration](./quickstart-aspnet-core-app.md).

Para realizar los pasos de este tutorial, puede usar cualquier editor de código. [Visual Studio Code](https://code.visualstudio.com/) es una excelente opción disponible en las plataformas Windows, macOS y Linux.

### <a name="prerequisites"></a>Requisitos previos

Si compila localmente, descargue e instale la [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) si aún no lo ha hecho.

Para realizar una compilación en la nube, por ejemplo, con Azure DevOps, asegúrese de que la [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) está instalada en el sistema de compilación.

### <a name="export-an-app-configuration-store"></a>Exportación de un almacén de configuración de aplicaciones

1. Abra su archivo *.csproj* y agregue el siguiente script:

    ```xml
    <Target Name="Export file" AfterTargets="Build">
        <Message Text="Export the configurations to a temp file. " />
        <Exec WorkingDirectory="$(MSBuildProjectDirectory)" Condition="$(ConnectionString) != ''" Command="az appconfig kv export -f $(OutDir)\azureappconfig.json --format json --separator : --connection-string $(ConnectionString)" />
    </Target>
    ```

    Agregue el valor de *ConnectionString* asociado al almacén de configuración de aplicaciones como una variable de entorno.

2. Abra *Program.cs* y actualice el método `CreateWebHostBuilder` para que use el archivo JSON exportado, para lo que es preciso llamar al método `config.AddJsonFile()`.

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

1. Establezca una variable de entorno llamada **ConnectionString** y defínala como la clave de acceso a su almacén de configuración de aplicaciones. Si usa el símbolo del sistema de Windows, ejecute el siguiente comando y reinícielo para que se aplique el cambio:

        setx ConnectionString "connection-string-of-your-app-configuration-store"

    Si usa Windows PowerShell, ejecute el siguiente comando:

        $Env:ConnectionString = "connection-string-of-your-app-configuration-store"

    Si usa macOS o Linux, ejecute el siguiente comando:

        export ConnectionString='connection-string-of-your-app-configuration-store'

2. Para compilar la aplicación mediante la CLI de .NET Core, ejecute el siguiente comando en el shell de comandos:

        dotnet build

3. Una vez que la compilación se haya realizado correctamente, ejecute el siguiente comando para ejecutar la aplicación web localmente:

        dotnet run

4. Inicie una ventana del explorador y vaya a `http://localhost:5000`, que es la dirección URL predeterminada de la aplicación web hospedada localmente.

    ![Inicio de la aplicación del artículo de inicio rápido en un entorno local](./media/quickstarts/aspnet-core-app-launch-local.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha exportado los datos de Azure App Configuration para su uso en una canalización de implementación. Para más información sobre App Configuration, continúe con los ejemplos de la CLI de Azure.

> [!div class="nextstepaction"]
> [Integración de identidades administradas](./howto-integrate-azure-managed-service-identity.md)
