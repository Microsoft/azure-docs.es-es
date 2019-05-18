---
title: Inicio rápido de Azure App Configuration con .NET Framework | Microsoft Docs
description: Un inicio rápido para el uso de Azure App Configuration con aplicaciones de .NET Framework
services: azure-app-configuration
documentationcenter: ''
author: yegu-ms
manager: balans
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.devlang: csharp
ms.topic: quickstart
ms.tgt_pltfrm: .NET
ms.workload: tbd
ms.date: 02/24/2019
ms.author: yegu
ms.openlocfilehash: acf1608257bc5416ff52ed9fbd4209008712f2d7
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65408705"
---
# <a name="quickstart-create-a-net-framework-app-with-azure-app-configuration"></a>Inicio rápido: Creación de una aplicación de .NET Framework con Azure App Configuration

Azure App Configuration es un servicio de configuración administrado de Azure. Puede usarlo para almacenar y administrar fácilmente toda la configuración de una aplicación en un solo lugar que esté separado del código. En este artículo de inicio rápido se muestra cómo incorporar el servicio en una aplicación de consola de escritorio de Windows basada en .NET Framework.

![Inicio rápido completo local](./media/quickstarts/dotnet-fx-app-run.png)

## <a name="prerequisites"></a>Requisitos previos

Para completar este artículo de inicio rápido, instale [Visual Studio 2017](https://visualstudio.microsoft.com/vs) y [.NET Framework 4.7.1](https://dotnet.microsoft.com/download), o cualquier versión posterior, si no lo ha hecho aún.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="create-an-app-configuration-store"></a>Creación de un almacén de configuración de aplicaciones

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-create.md)]

6. Seleccione **Explorador de configuración** > **+ Crear** para agregar los siguientes pares clave-valor:

    | Clave | Valor |
    |---|---|
    | TestApp:Settings:Message | Datos de Azure App Configuration |

    Deje **Etiqueta** y **Tipo de contenido** en blanco, por ahora.

## <a name="create-a-net-console-app"></a>Creación de una aplicación de consola de .NET

1. Inicie Visual Studio y seleccione **Archivo** > **Nuevo** > **Proyecto**.

2. En **Nuevo proyecto**, seleccione **Instalado** > **Visual C#** > **Escritorio clásico de Windows**. Seleccione **Aplicación de consola (.NET Framework)** y escriba un nombre para el proyecto. Seleccione **.NET Framework 4.7.1** u otra versión posterior y seleccione **Aceptar**.

## <a name="connect-to-an-app-configuration-store"></a>Conexión a un almacén de configuración de aplicaciones

1. Haga clic con el botón derecho en el proyecto y seleccione **Administrar paquetes NuGet**. En la pestaña **Examinar**, busque y agregue los siguientes paquetes NuGet al proyecto. Si no los encuentra, seleccione la casilla **Incluir versión preliminar**.

    ```
    Microsoft.Configuration.ConfigurationBuilders.AzureAppConfiguration 1.0.0 preview or later
    Microsoft.Configuration.ConfigurationBuilders.Environment 2.0.0 preview or later
    ```

2. Actualice el archivo *App.config* del proyecto como sigue:

    ```xml
    <configSections>
        <section name="configBuilders" type="System.Configuration.ConfigurationBuildersSection, System.Configuration, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" restartOnExternalChanges="false" requirePermission="false" />
    </configSections>

    <configBuilders>
        <builders>
            <add name="MyConfigStore" mode="Greedy" connectionString="${ConnectionString}" type="Microsoft.Configuration.ConfigurationBuilders.AzureAppConfigurationBuilder, Microsoft.Configuration.ConfigurationBuilders.AzureAppConfiguration" />
            <add name="Environment" mode="Greedy" type="Microsoft.Configuration.ConfigurationBuilders.EnvironmentConfigBuilder, Microsoft.Configuration.ConfigurationBuilders.Environment" />
        </builders>
    </configBuilders>

    <appSettings configBuilders="Environment,MyConfigStore">
        <add key="AppName" value="Console App Demo" />
        <add key="ConnectionString" value ="Set via an environment variable - for example, dev, test, staging, or production connection string." />
    </appSettings>
    ```

   La cadena de conexión del almacén de configuración de la aplicación se lee en la variable de entorno `ConnectionString`. Agregue el generador de configuración `Environment` antes de `MyConfigStore` en la propiedad `configBuilders` de la sección `appSettings`.

3. Abra *Program.cs* y actualice el método `Main` para usar App Configuration mediante una llamada a `ConfigurationManager`.

    ```csharp
    static void Main(string[] args)
    {
        string message = ConfigurationManager.AppSettings["TestApp:Settings:Message"];

        Console.WriteLine(message);
    }
    ```

## <a name="build-and-run-the-app-locally"></a>Compilación y ejecución de la aplicación en un entorno local

1. Establezca una variable de entorno llamada **ConnectionString** en la cadena de conexión del almacén de configuración de aplicaciones. Si usa el símbolo del sistema de Windows, ejecute el siguiente comando:

        setx ConnectionString "connection-string-of-your-app-configuration-store"

    Si usa Windows PowerShell, ejecute el siguiente comando:

        $Env:ConnectionString = "connection-string-of-your-app-configuration-store"

2. Reinicie Visual Studio para permitir que el cambio surta efecto. Presione Ctrl + F5 para compilar y ejecutar la aplicación de consola.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado un almacén de configuración de aplicaciones y lo ha usado con una aplicación de consola de .NET Framework. Para más información acerca del uso de App Configuration, continúe con el siguiente tutorial, ya que en él se muestra cómo realizar la autenticación.

> [!div class="nextstepaction"]
> [Integración de identidades administradas](./howto-integrate-azure-managed-service-identity.md)
