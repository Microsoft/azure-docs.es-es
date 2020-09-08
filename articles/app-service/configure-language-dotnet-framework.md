---
title: Configuración de las aplicaciones de ASP.NET
description: Cómo configurar una aplicación de ASP.NET en Azure App Service En este artículo se muestran las tareas de configuración más comunes.
ms.devlang: dotnet
ms.custom: devx-track-csharp
ms.topic: article
ms.date: 06/02/2020
ms.openlocfilehash: 30fddaec9ca5d0439beadedf7c5ca6b6c7d51d83
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88961710"
---
# <a name="configure-an-aspnet-app-for-azure-app-service"></a>Configuración de una aplicación de ASP.NET para Azure App Service

> [!NOTE]
> Para ASP.NET Core, vea [Configurar una aplicación de ASP.NET Core para Azure App Service](configure-language-dotnetcore.md)

Las aplicaciones de ASP.NET deben implementarse en Azure App Service como binarios compilados. La herramienta de publicación de Visual Studio compila la solución y, a continuación, implementa los binarios compilados directamente, mientras que el motor de implementación de App Service implementa el repositorio de código en primer lugar y, a continuación, compila los binarios.

Esta guía incluye conceptos clave e instrucciones para los desarrolladores de ASP.NET. Si nunca ha usado Azure App Service, siga primero la [guía de inicio rápido de ASP.NET](quickstart-dotnet-framework.md) y el [tutorial de ASP.NET con SQL Database](app-service-web-tutorial-dotnet-sqldatabase.md).

## <a name="show-supported-net-framework-runtime-versions"></a>Mostrar las versiones del entorno de ejecución de .NET Framework

En App Service, las instancias de Windows ya tienen instaladas todas las versiones compatibles de .NET Framework. Para mostrar las versiones del SDK y .NET Framework Runtime disponibles para usted, vaya a `https://<app-name>.scm.azurewebsites.net/DebugConsole` y ejecute el comando correspondiente en la consola basada en el explorador:

Para las versiones de tiempo de ejecución de CLR 4 (.NET Framework 4 y posteriores):

```CMD
ls "D:\Program Files (x86)\Reference Assemblies\Microsoft\Framework\.NETFramework\"
```

Es posible que la última versión de .NET Framework no esté disponible inmediatamente.

Para las versiones de tiempo de ejecución de CLR 2 (.NET Framework 3.5 y anteriores):

```CMD
ls "D:\Program Files (x86)\Reference Assemblies\Microsoft\Framework\"
```

## <a name="show-current-net-framework-runtime-version"></a>Consulta de la versión de tiempo de ejecución actual de .NET Framework

En [Cloud Shell](https://shell.azure.com), ejecute el comando siguiente:

```azurecli-interactive
az webapp config show --resource-group <resource-group-name> --name <app-name> --query netFrameworkVersion
```

El valor `v4.0` indica que se utiliza la última versión de CLR 4 (.NET Framework 4.x). El valor `v2.0` indica que se utiliza la versión de CLR 2 (.NET Framework 3.5).

## <a name="set-net-framework-runtime-version"></a>Establecimiento de la versión de tiempo de ejecución de .NET Framework

De forma predeterminada, App Service usa la última versión de .NET Framework compatible para ejecutar la aplicación de ASP.NET. Para ejecutar la aplicación con .NET Framework 3.5, en cambio, ejecute el siguiente comando en [Cloud Shell](https://shell.azure.com) (v2.0 significa CLR 2):

```azurecli-interactive
az webapp config set --resource-group <resource-group-name> --name <app-name> --net-framework-version v2.0
```

## <a name="access-environment-variables"></a>Acceso a variables de entorno

En App Service, puede [establecer la configuración de la aplicación](configure-common.md#configure-app-settings) y las cadenas de conexión fuera del código de la aplicación. Luego puede acceder a ellos en cualquier clase mediante el patrón de ASP.NET estándar:

```csharp
using System.Configuration;
...
// Get an app setting
ConfigurationManager.AppSettings["MySetting"];
// Get a connection string
ConfigurationManager.ConnectionStrings["MyConnection"];
}
```

Si configura una opción de aplicación con el mismo nombre en App Service y en *web.config*, el valor de App Service tiene prioridad sobre el valor *web.config*. El valor de *web.config* local permite depurar la aplicación localmente, pero el valor de App Service permite la ejecución de la aplicación en el producto con la configuración de producción. Las cadenas de conexión funcionan de la misma manera. De este modo, puede conservar los secretos de aplicación fuera de su repositorio de código y tener acceso a los valores adecuados sin cambiar el código.

## <a name="deploy-multi-project-solutions"></a>Implementar soluciones de varios proyectos

Cuando una solución de Visual Studio incluye varios proyectos, el proceso de publicación de Visual Studio ya incluye la selección del proyecto que se va a implementar. Cuando se implementa en el motor de implementación de App Service, como con Git o con implementación de archivo ZIP, con la automatización de compilación activada, el motor de implementación de App Service elige el primer sitio web o el proyecto de aplicación web que encuentra como aplicación de App Service. Para especificar el proyecto que App Service debe usar, especifique la configuración de la aplicación `PROJECT`. Por ejemplo, ejecute lo siguiente en [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp config appsettings set --resource-group <resource-group-name> --name <app-name> --settings PROJECT="<project-name>/<project-name>.csproj"
```

## <a name="get-detailed-exceptions-page"></a>Obtener la página de excepciones detalladas

Cuando la aplicación de ASP.NET genera una excepción en el depurador de Visual Studio, el explorador muestra una página de excepciones detalladas, pero en App Service, esa página se sustituye por un mensaje de error genérico. Para mostrar la página de excepción detallada en App Service, abra el archivo *Web.config* y agregue el elemento `<customErrors mode="Off"/>` en el elemento `<system.web>`. Por ejemplo:

```xml
<system.web>
    <customErrors mode="Off"/>
</system.web>
```

Vuelva a implementar la aplicación con *Web.config* actualizado. Ahora debería ver la misma página de excepciones detallada.

## <a name="access-diagnostic-logs"></a>Acceso a los registros de diagnóstico

Puede agregar mensajes de diagnóstico en el código de la aplicación mediante [System.Diagnostics.Trace](/dotnet/api/system.diagnostics.trace). Por ejemplo: 

```csharp
Trace.TraceError("Record not found!"); // Error trace
Trace.TraceWarning("Possible data loss"); // Warning trace
Trace.TraceInformation("GET /Home/Index"); // Information trace
```

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-no-h.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial: Compilación de una aplicación ASP.NET en Azure con SQL Database](app-service-web-tutorial-dotnet-sqldatabase.md)