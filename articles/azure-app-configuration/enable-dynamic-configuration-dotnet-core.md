---
title: 'Tutorial: Uso de la configuración dinámica en una aplicación de .NET Core'
titleSuffix: Azure App Configuration
description: En este tutorial aprenderá a actualizar dinámicamente los datos de configuración de aplicaciones de .NET Core
services: azure-app-configuration
documentationcenter: ''
author: abarora
manager: zhenlan
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.workload: tbd
ms.devlang: csharp
ms.custom: devx-track-csharp
ms.topic: tutorial
ms.date: 07/01/2019
ms.author: abarora
ms.openlocfilehash: aebe7772c673162d60f35d6a81725ba1452d16a8
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88207066"
---
# <a name="tutorial-use-dynamic-configuration-in-a-net-core-app"></a>Tutorial: Uso de la configuración dinámica en una aplicación de .NET Core

La biblioteca cliente de .NET Core de App Configuration admite la actualización de un conjunto de ajustes de configuración a petición sin causar el reinicio de una aplicación. Para implementarla, obtenga primero una instancia de `IConfigurationRefresher` entre las opciones para el proveedor de configuración y, después, llame a `TryRefreshAsync` en esa instancia en cualquier lugar del código.

Con el fin de mantener la configuración actualizada y evitar demasiadas llamadas al almacén de configuración, se usa una caché para cada configuración. Hasta que haya expirado el valor almacenado en caché de una configuración, la operación no actualiza el valor, incluso cuando el valor cambia en el almacén de configuración. El tiempo de expiración predeterminado para cada solicitud es de 30 segundos, pero puede reemplazarse si es necesario.

Este tutorial le muestra cómo puede implementar las actualizaciones de configuración dinámica en el código. Se basa en la aplicación que se introdujo en los inicios rápidos. Antes de continuar, finalice primero el tutorial [Creación de una aplicación .NET Core con Azure App Configuration](./quickstart-dotnet-core-app.md).

Para realizar los pasos de este tutorial, puede usar cualquier editor de código. [Visual Studio Code](https://code.visualstudio.com/) es una excelente opción que está disponible en las plataformas Windows, macOS y Linux.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Configurar la aplicación de .NET Core para actualizar su configuración en respuesta a los cambios en un almacén de App Configuration.
> * Consuma en la aplicación la configuración más reciente.

## <a name="prerequisites"></a>Prerrequisitos

Para realizar este tutorial, instale el [SDK de .NET Core](https://dotnet.microsoft.com/download).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="reload-data-from-app-configuration"></a>Recarga de datos de App Configuration

Abra *Program.cs* y actualice el archivo para agregar una referencia al espacio de nombres `System.Threading.Tasks`, con el fin de especificar la configuración de la actualización en el método `AddAzureAppConfiguration` y desencadene la actualización manual mediante el método `TryRefreshAsync`.

```csharp
using System;
using System.Threading.Tasks;

namespace TestConsole
{
class Program
{
    private static IConfiguration _configuration = null;
    private static IConfigurationRefresher _refresher = null;

    static void Main(string[] args)
    {
        var builder = new ConfigurationBuilder();
        builder.AddAzureAppConfiguration(options =>
        {
            options.Connect(Environment.GetEnvironmentVariable("ConnectionString"))
                    .ConfigureRefresh(refresh =>
                    {
                        refresh.Register("TestApp:Settings:Message")
                               .SetCacheExpiration(TimeSpan.FromSeconds(10));
                    });
                    
                    _refresher = options.GetRefresher();
        });

        _configuration = builder.Build();
        PrintMessage().Wait();
    }

    private static async Task PrintMessage()
    {
        Console.WriteLine(_configuration["TestApp:Settings:Message"] ?? "Hello world!");

        // Wait for the user to press Enter
        Console.ReadLine();

        await _refresher.TryRefreshAsync();
        Console.WriteLine(_configuration["TestApp:Settings:Message"] ?? "Hello world!");
    }
}
}
```

El método `ConfigureRefresh` se utiliza para especificar la configuración usada para actualizar los datos de configuración con el almacén de App Configuration cuando se desencadena una operación de actualización. Se puede recuperar una instancia de `IConfigurationRefresher` mediante una llamada al método `GetRefresher` en las opciones proporcionadas para el método `AddAzureAppConfiguration` y el método `TryRefreshAsync` en esta instancia podría usarse para desencadenar una operación de actualización en cualquier lugar del código.
    
> [!NOTE]
> La hora de expiración de la caché predeterminada para un valor de configuración es de 30 segundos, pero puede reemplazarse por una llamada al método `SetCacheExpiration` en el inicializador de opciones que se pasa como argumento al método `ConfigureRefresh`.

## <a name="build-and-run-the-app-locally"></a>Compilación y ejecución de la aplicación en un entorno local

1. Establezca una variable de entorno llamada **ConnectionString** y defínala como la clave de acceso a su almacén de App Configuration. Si usa el símbolo del sistema de Windows, ejecute el siguiente comando y reinícielo para que se aplique el cambio:

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

1. Ejecute el siguiente comando para compilar la aplicación de consola:

    ```console
     dotnet build
    ```

1. Una vez que la compilación se haya realizado correctamente, ejecute el siguiente comando para ejecutar la aplicación localmente:

    ```console
     dotnet run
    ```

    ![Inicio de la aplicación del artículo de inicio rápido en un entorno local](./media/quickstarts/dotnet-core-app-run.png)

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Todos los recursos** y seleccione la instancia de almacén de App Configuration que creó en el inicio rápido.

1. Seleccione **Explorador de configuración** y actualice los valores de las claves siguientes:

    | Clave | Value |
    |---|---|
    | TestApp:Settings:Message | Datos de Azure App Configuration, actualizados |

1. Presione la tecla Entrar para desencadenar una actualización e imprimir el valor actualizado en la ventana del símbolo del sistema o de PowerShell.

    ![Inicio rápido de actualización de la aplicación en el entorno local](./media/quickstarts/dotnet-core-app-run-refresh.png)
    
    > [!NOTE]
    > Dado que el tiempo de expiración de la caché se estableció en 10 segundos mediante el método `SetCacheExpiration` al especificar la configuración para la operación de actualización, el valor de la configuración solo se actualizará si han transcurrido al menos 10 segundos desde la última actualización de dicha configuración.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha habilitado la aplicación de .NET Core para actualizar dinámicamente la configuración a partir de App Configuration. Para obtener información sobre cómo usar una identidad administrada de Azure para simplificar el acceso a App Configuration, vaya al siguiente tutorial.

> [!div class="nextstepaction"]
> [Integración de identidades administradas](./howto-integrate-azure-managed-service-identity.md)
