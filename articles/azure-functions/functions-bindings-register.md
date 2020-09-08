---
title: Registro de las extensiones de enlace de Azure Functions
description: Aprenda a registrar una extensión de enlace de Azure Functions basada en su entorno.
author: craigshoemaker
ms.topic: reference
ms.date: 08/16/2020
ms.author: cshoe
ms.openlocfilehash: 942ca3229808b57894598c3477e9dc97e40e8c80
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88689578"
---
# <a name="register-azure-functions-binding-extensions"></a>Registro de las extensiones de enlace de Azure Functions

A partir de Azure Functions versión 2.x, el entorno de ejecución de Functions solo incluye desencadenadores HTTP y de temporizador de forma predeterminada. Hay disponibles otros [desencadenadores y enlaces](./functions-triggers-bindings.md) como paquetes independientes.

Las aplicaciones de funciones de la biblioteca de clases .NET usan enlaces que se instalan en el proyecto como paquetes NuGet. Los conjuntos de extensiones permiten a las aplicaciones de funciones que no son .NET utilizar los mismos enlaces sin tener que tratar con la infraestructura de .NET.

En la tabla siguiente se indica cuándo y cómo registrar los enlaces.

| Entorno de desarrollo |Registro<br/> en Functions 1.x  |Registro<br/> en Functions 3.x/2.x  |
|-------------------------|------------------------------------|------------------------------------|
|Portal de Azure|Automático|Automático<sup>*</sup>|
|Lenguajes que no son .NET|Automático|Use [conjuntos de extensiones](#extension-bundles) (recomendado) o [instale las extensiones de forma explícita](#explicitly-install-extensions).|
|Biblioteca de clases de C# con Visual Studio|[Uso de herramientas NuGet](#vs)|[Uso de herramientas NuGet](#vs)|
|Biblioteca de clases de C# con Visual Studio Code|N/D|[Uso de la CLI de .NET Core](#vs-code)|

<sup>*</sup> El portal usa conjuntos de extensiones.

## <a name="access-extensions-in-non-net-languages"></a>Acceso a las extensiones en lenguajes que no son .NET

Para Java, JavaScript, PowerShell, Python y las aplicaciones de funciones con controlador personalizado, se recomienda usar conjuntos de extensiones para acceder a los enlaces. En los casos en los que no se pueden usar conjuntos de extensiones, puede instalar explícitamente las extensiones de enlaces.

### <a name="extension-bundles"></a><a name="extension-bundles"></a>Conjuntos de extensiones

Los conjuntos de extensiones son una manera de agregar un conjunto compatible de extensiones de enlace a la aplicación de funciones. Los conjuntos de extensiones se habilitan en el archivo *host.json* de la aplicación.

Puede usar agrupaciones de extensiones con la versión 2.x y versiones posteriores del tiempo de ejecución de Functions.

Los conjuntos de extensiones tienen control de versiones. Cada versión contiene un conjunto específico de extensiones de enlace que se ha comprobado que funcionan juntas. Seleccione una versión del conjunto en función de las extensiones que necesite en la aplicación.

Para agregar un conjunto de extensiones a la aplicación de funciones, agregue la sección `extensionBundle` al archivo *host.json*. En muchos casos, Visual Studio Code y Azure Functions Core Tools lo agregarán automáticamente.

[!INCLUDE [functions-extension-bundles-json](../../includes/functions-extension-bundles-json.md)]

En la tabla siguiente se enumeran las versiones disponibles actualmente del conjunto *Microsoft.Azure.Functions.ExtensionBundle* predeterminado y los vínculos a las extensiones que incluyen.

| Versión del conjunto | Versión en host.json | Extensiones incluidas |
| --- | --- | --- |
| 1.x | `[1.*, 2.0.0)` | Consulte el archivo [extensions.json](https://github.com/Azure/azure-functions-extension-bundles/blob/v1.x/src/Microsoft.Azure.Functions.ExtensionBundle/extensions.json) utilizado para generar el conjunto. |
| 2.x | `[2.*, 3.0.0)` | Consulte el archivo [extensions.json](https://github.com/Azure/azure-functions-extension-bundles/blob/v2.x/src/Microsoft.Azure.Functions.ExtensionBundle/extensions.json) utilizado para generar el conjunto. |

> [!NOTE]
> Aunque puede especificar un intervalo de versiones personalizado en el archivo host.json, se recomienda usar un valor de versión de esta tabla.

### <a name="explicitly-install-extensions"></a><a name="explicitly-install-extensions"></a>Instalación explícita de extensiones

[!INCLUDE [functions-extension-register-core-tools](../../includes/functions-extension-register-core-tools.md)]

## <a name="install-extensions-from-nuget-in-net-languages"></a><a name="local-csharp"></a>Instalación de las extensiones desde NuGet en los lenguajes .NET

En un proyecto de funciones basados en la biblioteca de clases de C#, se deben instalar las extensiones directamente. Los conjuntos de extensiones se han diseñado específicamente para proyectos que no están basados en la biblioteca de clases de C#.

### <a name="c-class-library-with-visual-studio"></a><a name="vs"></a> Biblioteca de clases de C\# con Visual Studio

En **Visual Studio**, puede instalar paquetes desde la Consola del Administrador de paquetes mediante el comando [Install-Package](/nuget/tools/ps-ref-install-package), tal como se muestra en el ejemplo siguiente:

```powershell
Install-Package Microsoft.Azure.WebJobs.Extensions.ServiceBus -Version <TARGET_VERSION>
```

El nombre del paquete que se usa para un enlace determinado se proporciona en el artículo de referencia de ese enlace. Para obtener un ejemplo, consulte la [sección de paquetes del artículo de referencia de enlace de Service Bus](functions-bindings-service-bus.md#functions-1x).

Reemplace `<TARGET_VERSION>` en el ejemplo con una versión específica del paquete, como `3.0.0-beta5`. Las versiones válidas se enumeran en las páginas individuales del paquete en [NuGet.org](https://nuget.org). Las versiones principales que corresponden al tiempo de ejecución de Functions 1.x o 2.x se especifican en el artículo de referencia del enlace.

Si usa `Install-Package` para hacer referencia a un enlace, no es necesario usar [agrupaciones de extensiones](#extension-bundles). Este enfoque es específico de las bibliotecas de clases compiladas en Visual Studio.

### <a name="c-class-library-with-visual-studio-code"></a><a name="vs-code"></a> Biblioteca de clases de C# con Visual Studio Code

En **Visual Studio Code**, instale paquetes para un proyecto de biblioteca de clases de C# desde el símbolo del sistema mediante el comando [dotnet add package](/dotnet/core/tools/dotnet-add-package) de la CLI de .NET Core. En el ejemplo siguiente se muestra cómo agregar un enlace:

```terminal
dotnet add package Microsoft.Azure.WebJobs.Extensions.<BINDING_TYPE_NAME> --version <TARGET_VERSION>
```

La CLI de .NET Core solo puede utilizarse para el desarrollo de Azure Functions 2.x.

Reemplace `<BINDING_TYPE_NAME>` por el nombre del paquete que contiene el enlace que necesita. Puede encontrar el artículo de referencia de enlace deseado en la [lista de enlaces admitidos](./functions-triggers-bindings.md#supported-bindings).

Reemplace `<TARGET_VERSION>` en el ejemplo con una versión específica del paquete, como `3.0.0-beta5`. Las versiones válidas se enumeran en las páginas individuales del paquete en [NuGet.org](https://nuget.org). Las versiones principales que corresponden al tiempo de ejecución de Functions 1.x o 2.x se especifican en el artículo de referencia del enlace.

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Ejemplo de desencadenador y enlace de Azure Functions](./functions-bindings-example.md)
