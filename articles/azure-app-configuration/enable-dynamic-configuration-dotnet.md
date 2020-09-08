---
title: 'Tutorial de .NET Framework: Configuración dinámica de Azure App Configuration'
description: En este tutorial aprenderá a actualizar dinámicamente los datos de configuración de aplicaciones de .NET Framework mediante Azure App Configuration.
services: azure-app-configuration
author: lisaguthrie
ms.service: azure-app-configuration
ms.devlang: csharp
ms.custom: devx-track-csharp
ms.topic: tutorial
ms.date: 10/21/2019
ms.author: lcozzens
ms.openlocfilehash: 49ab9090dd2f1860bed84d0494cb862212840e82
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88207043"
---
# <a name="tutorial-use-dynamic-configuration-in-a-net-framework-app"></a>Tutorial: Uso de la configuración dinámica en una aplicación de .NET Framework

La biblioteca cliente de .NET de App Configuration admite la actualización de un conjunto de valores de configuración a petición sin causar el reinicio de una aplicación. Para implementarla, obtenga primero una instancia de `IConfigurationRefresher` entre las opciones para el proveedor de configuración y, después, llame a `TryRefreshAsync` en esa instancia en cualquier lugar del código.

Con el fin de mantener la configuración actualizada y evitar demasiadas llamadas al almacén de configuración, se usa una caché para cada configuración. Hasta que haya expirado el valor almacenado en caché de una configuración, la operación no actualiza el valor, incluso cuando el valor cambia en el almacén de configuración. El tiempo de expiración predeterminado para cada solicitud es de 30 segundos, pero puede reemplazarse si es necesario.

Este tutorial le muestra cómo puede implementar las actualizaciones de configuración dinámica en el código. Se basa en la aplicación que se introdujo en los inicios rápidos. Antes de continuar, finalice primero el tutorial [Creación de una aplicación de .NET Framework con Azure App Configuration](./quickstart-dotnet-app.md).

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Configurar la aplicación de .NET Framework para actualizar su configuración en respuesta a los cambios en un almacén de App Configuration.
> * Insertar la configuración más reciente en la aplicación.
## <a name="prerequisites"></a>Prerrequisitos

- Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/)
- [Visual Studio 2019](https://visualstudio.microsoft.com/vs)
- [.NET Framework 4.7.1 o posterior](https://dotnet.microsoft.com/download)

## <a name="create-an-app-configuration-store"></a>Creación de un almacén de App Configuration

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-create.md)]

6. Seleccione **Explorador de configuración** >  **+ Crear** > **Clave-valor** para agregar los siguientes pares clave-valor:

    | Clave | Value |
    |---|---|
    | TestApp:Settings:Message | Datos de Azure App Configuration |

    Deje **Etiqueta** y **Tipo de contenido** en blanco, por ahora.

7. Seleccione **Aplicar**.

## <a name="create-a-net-framework-console-app"></a>Creación de una aplicación de consola de .NET Framework

1. Inicie Visual Studio y seleccione **Archivo** > **Nuevo** > **Proyecto**.

1. En **Crear un proyecto**, filtre por el tipo de proyecto **Consola** y haga clic en **Aplicación de consola (.NET Framework)** . Haga clic en **Next**.

1. En **Configurar el nuevo proyecto**, asígnele al proyecto un nombre. En **Marco**, seleccione **.NET Framework 4.7.1** o una versión posterior. Haga clic en **Crear**.

## <a name="reload-data-from-app-configuration"></a>Recarga de datos de App Configuration
1. Haga clic con el botón derecho en el proyecto y seleccione **Administrar paquetes NuGet**. En la pestaña **Examinar**, busque y agregue el paquete NuGet *Microsoft.Extensions.Configuration.AzureAppConfiguration* al proyecto. Si no lo encuentra, seleccione la casilla **Incluir versión preliminar**.

1. Abra *Program.cs* y agregue una referencia al proveedor de configuración de la aplicación .NET Core.

    ```csharp
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.Configuration.AzureAppConfiguration;
    ```

1. Agregue dos variables para almacenar los objetos relacionados con la configuración.

    ```csharp
    private static IConfiguration _configuration = null;
    private static IConfigurationRefresher _refresher = null;
    ```

1. Actualice el método `Main` para conectarse a App Configuration con las opciones de actualización especificadas.

    ```csharp
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
    ```
    El método `ConfigureRefresh` se utiliza para especificar la configuración usada para actualizar los datos de configuración con el almacén de App Configuration cuando se desencadena una operación de actualización. Se puede recuperar una instancia de `IConfigurationRefresher` mediante una llamada al método `GetRefresher` en las opciones proporcionadas para el método `AddAzureAppConfiguration`; además, el método `TryRefreshAsync` de esta instancia puede usarse para desencadenar una operación de actualización en cualquier lugar del código.

    > [!NOTE]
    > La hora de expiración de la caché predeterminada para un valor de configuración es de 30 segundos, pero puede reemplazarse por una llamada al método `SetCacheExpiration` en el inicializador de opciones que se pasa como argumento al método `ConfigureRefresh`.

1. Agregue un método denominado `PrintMessage()` que desencadene una actualización manual de los datos de configuración de App Configuration.

    ```csharp
    private static async Task PrintMessage()
    {
        Console.WriteLine(_configuration["TestApp:Settings:Message"] ?? "Hello world!");

        // Wait for the user to press Enter
        Console.ReadLine();

        await _refresher.TryRefreshAsync();
        Console.WriteLine(_configuration["TestApp:Settings:Message"] ?? "Hello world!");
    }
    ```

## <a name="build-and-run-the-app-locally"></a>Compilación y ejecución de la aplicación en un entorno local

1. Establezca una variable de entorno llamada **ConnectionString** y defínala como la clave de acceso a su almacén de App Configuration. Si usa el símbolo del sistema de Windows, ejecute el siguiente comando y reinícielo para que se aplique el cambio:

    ```console
        setx ConnectionString "connection-string-of-your-app-configuration-store"
    ```

    Si usa Windows PowerShell, ejecute el siguiente comando:

    ```powershell
        $Env:ConnectionString = "connection-string-of-your-app-configuration-store"
    ```

1. Reinicie Visual Studio para permitir que el cambio surta efecto. 

1. Presione Ctrl + F5 para compilar y ejecutar la aplicación de consola.

    ![Inicio local de la aplicación](./media/dotnet-app-run.png)

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Todos los recursos** y seleccione la instancia de almacén de App Configuration que creó en el inicio rápido.

1. Seleccione **Explorador de configuración** y actualice los valores de las claves siguientes:

    | Clave | Value |
    |---|---|
    | TestApp:Settings:Message | Datos de Azure App Configuration, actualizados |

1. De vuelta en la aplicación en ejecución, presione la tecla Entrar para desencadenar una actualización e imprimir el valor actualizado en la ventana del símbolo del sistema o de PowerShell.

    ![Actualización local de la aplicación](./media/dotnet-app-run-refresh.png)
    
    > [!NOTE]
    > Dado que el tiempo de expiración de la caché se estableció en 10 segundos mediante el método `SetCacheExpiration` al especificar la configuración para la operación de actualización, el valor de la configuración solo se actualizará si han transcurrido al menos 10 segundos desde la última actualización de dicha configuración.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha habilitado la aplicación de .NET Framework para actualizar dinámicamente la configuración a partir de App Configuration. Para obtener información sobre cómo usar una identidad administrada de Azure para simplificar el acceso a App Configuration, vaya al siguiente tutorial.

> [!div class="nextstepaction"]
> [Integración de identidades administradas](./howto-integrate-azure-managed-service-identity.md)
