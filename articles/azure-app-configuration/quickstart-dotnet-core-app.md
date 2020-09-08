---
title: Inicio rápido de Azure App Configuration con .NET Core | Microsoft Docs
description: En este inicio rápido se crea una aplicación de .NET Core con Azure App Configuration para centralizar el almacenamiento y la administración de la configuración de la aplicación de forma independiente del código.
services: azure-app-configuration
author: lisaguthrie
ms.service: azure-app-configuration
ms.topic: quickstart
ms.custom: devx-track-csharp
ms.date: 1/9/2019
ms.author: lcozzens
ms.openlocfilehash: 57202752a965ad0405a04b2eba4cdf42070c94bf
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89319515"
---
# <a name="quickstart-create-a-net-core-app-with-app-configuration"></a>Inicio rápido: Creación de una aplicación .NET Core con App Configuration

En este inicio rápido incorporará Azure App Configuration a una aplicación de consola de .NET Core para centralizar el almacenamiento y la administración de la configuración de la aplicación de forma independiente del código.

## <a name="prerequisites"></a>Prerrequisitos

- Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/)
- [SDK de .NET Core](https://dotnet.microsoft.com/download): también está disponible en [Azure Cloud Shell](https://shell.azure.com).

## <a name="create-an-app-configuration-store"></a>Creación de un almacén de App Configuration

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-create.md)]

6. Seleccione **Explorador de configuración** > **Crear** > **Clave-valor** para agregar los siguientes pares clave-valor:

    | Clave | Value |
    |---|---|
    | TestApp:Settings:Message | Datos de Azure App Configuration |

    Deje **Etiqueta** y **Tipo de contenido** en blanco, por ahora.

7. Seleccione **Aplicar**.

## <a name="create-a-net-core-console-app"></a>Creación de una aplicación de consola de .NET Core

Va a utilizar la [interfaz de la línea de comandos (CLI) de .NET Core](https://docs.microsoft.com/dotnet/core/tools/) para crear un proyecto de aplicación de consola de .NET Core. La ventaja de usar la CLI de .NET Core frente a Visual Studio es que está disponible en las plataformas Windows, macOS y Linux.  También puede usar las herramientas preinstaladas disponibles en [Azure Cloud Shell](https://shell.azure.com).

1. Cree una carpeta nueva para su proyecto.

2. En la nueva carpeta, ejecute el siguiente comando para crear un nuevo proyecto de aplicación de la consola de .NET Core:

    ```dotnetcli
    dotnet new console
    ```

## <a name="connect-to-an-app-configuration-store"></a>Conexión a un almacén de App Configuration

1. Para agregar una referencia al paquete NuGet `Microsoft.Extensions.Configuration.AzureAppConfiguration`, ejecute el comando siguiente:

    ```dotnetcli
    dotnet add package Microsoft.Extensions.Configuration.AzureAppConfiguration
    ```

2. Ejecute el siguiente comando para restaurar los paquetes para el proyecto:

    ```dotnetcli
    dotnet restore
    ```

3. Abra *Program.cs* y agregue una referencia al proveedor de configuración de la aplicación .NET Core.

    ```csharp
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.Configuration.AzureAppConfiguration;
    ```

4. Actualice el método `Main` para usar App Configuration; para ello, llame al método `builder.AddAzureAppConfiguration()`.

    ```csharp
    static void Main(string[] args)
    {
        var builder = new ConfigurationBuilder();
        builder.AddAzureAppConfiguration(Environment.GetEnvironmentVariable("ConnectionString"));

        var config = builder.Build();
        Console.WriteLine(config["TestApp:Settings:Message"] ?? "Hello world!");
    }
    ```

## <a name="build-and-run-the-app-locally"></a>Compilación y ejecución de la aplicación en un entorno local

1. Establezca una variable de entorno llamada **ConnectionString** y defínala como la clave de acceso a su almacén de App Configuration. En la línea de comandos, ejecute el siguiente comando:

    ```cmd
    setx ConnectionString "connection-string-of-your-app-configuration-store"
    ```

    Si usa Windows PowerShell, ejecute el siguiente comando:

    ```azurepowershell
    $Env:ConnectionString = "connection-string-of-your-app-configuration-store"
    ```

    Si usa macOS o Linux, ejecute el siguiente comando:

    ```console
    export ConnectionString='connection-string-of-your-app-configuration-store'
    ```

    Reinicie el símbolo del sistema para permitir que el cambio surta efecto. Imprima el valor de la variable de entorno para comprobar que se ha establecido correctamente.

2. Ejecute el siguiente comando para compilar la aplicación de consola:

    ```dotnetcli
    dotnet build
    ```

3. Una vez que la compilación se haya realizado correctamente, ejecute el siguiente comando para ejecutar la aplicación localmente:

    ```dotnetcli
    dotnet run
    ```

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado un almacén de App Configuration y lo ha usado con una aplicación de consola de .NET Core a través del [proveedor de App Configuration](https://go.microsoft.com/fwlink/?linkid=2074664). Para aprender a configurar una aplicación de .NET Core para actualizar dinámicamente la configuración, vaya al siguiente tutorial.

> [!div class="nextstepaction"]
> [Habilitación de la configuración dinámica](./enable-dynamic-configuration-dotnet-core.md)
