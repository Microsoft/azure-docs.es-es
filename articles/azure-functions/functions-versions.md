---
title: Introducción a las versiones de tiempo de ejecución de Azure Functions
description: Azure Functions admite varias versiones del runtime. Conozca las diferencias entre ellas y cómo elegir la más adecuada en su caso.
ms.topic: conceptual
ms.date: 12/09/2019
ms.openlocfilehash: 29dd4a1ea60c463655c5b9a17fedcf6d51eee927
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87081685"
---
# <a name="azure-functions-runtime-versions-overview"></a>Introducción a las versiones de tiempo de ejecución de Azure Functions

Actualmente, Azure Functions admite tres versiones del host en tiempo de ejecución: 1.x, 2.x y 3.x. Las tres versiones se admiten en escenarios de producción.  

> [!IMPORTANT]
> La versión 1.x se encuentra en modo de mantenimiento, y solo admite el desarrollo en Azure Portal o localmente en equipos Windows. Las mejoras solo se proporcionan en versiones posteriores. 

En este artículo se detallan algunas de las diferencias entre las distintas versiones, cómo se puede crear cada versión y cómo se cambian las versiones.

## <a name="languages"></a>Lenguajes

A partir de la versión 2.x, el entorno de ejecución usa un modelo de extensibilidad de lenguaje y todas las funciones de una aplicación de funciones deben compartir el mismo lenguaje. El lenguaje de las funciones de una aplicación de funciones se elige cuando se crea la aplicación y se mantiene en el valor [FUNCTIONS\_WORKER\_RUNTIME](functions-app-settings.md#functions_worker_runtime). 

En la siguiente tabla se indican los lenguajes de programación que se admiten actualmente en cada versión del entorno de ejecución.

[!INCLUDE [functions-supported-languages](../../includes/functions-supported-languages.md)]

Para más información, consulte [Lenguajes admitidos](supported-languages.md).

## <a name="run-on-a-specific-version"></a><a name="creating-1x-apps"></a>Ejecución en una versión específica

De forma predeterminada, las aplicaciones de funciones que se crean en Azure Portal y en la CLI de Azure se establecen en la versión 3.x. Puede modificar esta versión según sea necesario. Solo puede cambiar la versión del entorno en tiempo de ejecución a 1.x después de crear la aplicación de funciones, pero antes de agregar funciones.  Se permite el desplazamiento entre 2.x y 3.x incluso con aplicaciones que tienen funciones, pero se recomienda probar primero en una aplicación nueva.

## <a name="migrating-from-1x-to-later-versions"></a>Migración desde la versión 1.x a versiones posteriores

Puede migrar una aplicación existente escrita para usar la versión 1.x del entorno en tiempo de ejecución para que utilice en su lugar una versión más reciente. La mayoría de los cambios que debe realizar están relacionados con cambios en el entorno en tiempo de ejecución del lenguaje, como los cambios en la API de C# entre .NET Framework 4.7 y .NET Core. Deberá asegurarse de que el código y las bibliotecas son compatibles con la versión del entorno de ejecución del lenguaje que utilice. Finalmente, asegúrese de anotar los cambios en el desencadenador, los enlaces y las características que se resaltan a continuación. Para obtener los mejores resultados de migración, debe crear una nueva aplicación de funciones en una nueva versión y portar el código de función de la versión 1.x existente a la nueva aplicación.  

Aunque es posible realizar una actualización "en contexto" actualizando manualmente la configuración de la aplicación, al pasar de 1.x a una versión posterior, se incluyen algunos cambios importantes. Por ejemplo, en C# se modifica el objeto de depuración de `TraceWriter` a `ILogger`. Al crear un nuevo proyecto con la versión 3.x, empieza por las funciones actualizadas en función de las plantillas más recientes de la versión 3.x.

### <a name="changes-in-triggers-and-bindings-after-version-1x"></a>Cambios en los desencadenadores y los enlaces después de la versión 1.x

A partir de la versión 2.x, debe instalar las extensiones para desencadenadores y enlaces específicos que las funciones de la aplicación utilizan. La única excepción son los desencadenadores HTTP y el temporizador, que no requieren extensión.  Para más información, consulte la sección sobre [Registro e instalación de extensiones de enlace](./functions-bindings-register.md).

También hay algunos cambios en *function.json* o los atributos de la función de una versión a otra. Por ejemplo, la propiedad `path` del centro de eventos es ahora `eventHubName`. Consulte la [tabla de enlaces existentes](#bindings) para vínculos a la documentación de cada enlace.

### <a name="changes-in-features-and-functionality-after-version-1x"></a>Cambios en las características y la funcionalidad después de la versión 1.x

Se han quitado, actualizado o reemplazado algunas características después de la versión 1.x. En esta sección se describen los cambios que se observan en versiones posteriores después de haber utilizado la versión 1.x.

En la versión 2.x se han realizado los siguientes cambios:

* Las claves para llamar a los puntos de conexión HTTP siempre se almacenan cifradas en Azure Blob Storage. En la versión 1.x, se almacenaban en Azure File Storage de manera predeterminada. Al actualizar una aplicación de la versión 1.x a la versión 2.x, los secretos existentes que se encuentran en el almacenamiento de archivos se restablecen.

* La versión 2.x del entorno de ejecución no incluye compatibilidad integrada con proveedores de webhooks. Este cambio se realizó para mejorar el rendimiento. Todavía puede usar desencadenadores HTTP como puntos de conexión para los webhooks.

* El archivo de configuración de host (host.json) debe estar vacío o tener la cadena `"version": "2.0"`.

* Para mejorar la supervisión, el panel de WebJobs del portal, que usaba la configuración [`AzureWebJobsDashboard`](functions-app-settings.md#azurewebjobsdashboard) se reemplaza por Azure Application Insights, que usa la configuración [`APPINSIGHTS_INSTRUMENTATIONKEY`](functions-app-settings.md#appinsights_instrumentationkey). Para más información, consulte [Supervisión de Azure Functions](functions-monitoring.md).

* Todas las funciones de una aplicación de función deben compartir el mismo lenguaje. Al crear una aplicación de función, debe elegir una pila del entorno de ejecución para la aplicación. La pila del entorno de ejecución se especifica con el valor [`FUNCTIONS_WORKER_RUNTIME`](functions-app-settings.md#functions_worker_runtime) en la configuración de la aplicación. Este requisito se agregó para mejorar el tiempo de inicio y la superficie de memoria. Al desarrollar localmente, también debe incluir esta configuración en el [archivo local.settings.json](functions-run-local.md#local-settings-file).

* El tiempo de espera predeterminado para las funciones en un plan de App Service ha cambiado a 30 minutos. Puede cambiar manualmente el tiempo de espera de vuelta a ilimitado mediante la configuración [functionTimeout](functions-host-json.md#functiontimeout) de host.json.

* Se implementan de forma predeterminada limitaciones de simultaneidad HTTP para las funciones del plan de consumo, cuyo valor predeterminado es de 100 solicitudes simultáneas por instancia. Puede cambiar esto en la configuración [`maxConcurrentRequests`](functions-host-json.md#http) del archivo host.json.

* A causa de las [limitaciones de .NET Core](https://github.com/Azure/azure-functions-host/issues/3414), se ha eliminado la compatibilidad con las funciones de script (.fsx) de F#. Todavía se admiten las funciones compiladas de F# (.fs).

* El formato de dirección URL de los webhooks de desencadenador de Event Grid ha cambiado a `https://{app}/runtime/webhooks/{triggerName}`.

## <a name="migrating-from-2x-to-3x"></a>Migración de 2.x a 3.x

Azure Functions versión 3.x es una versión muy compatible con la versión 2.x.  Muchas aplicaciones deben poder actualizarse a 3.x sin ningún cambio de código.  Aunque se recomienda pasar a 3.x, asegúrese de ejecutar unas pruebas exhaustivas antes de cambiar la versión principal en las aplicaciones de producción.

### <a name="breaking-changes-between-2x-and-3x"></a>Cambios importantes entre 2.x y 3.x

A continuación se indican los cambios que se deben tener en cuenta antes de actualizar una aplicación de 2.x a 3.x.

#### <a name="javascript"></a>JavaScript

* Los enlaces de salida asignados mediante `context.done` o valores devueltos ahora se comportan igual que cuando se establecen en `context.bindings`.

* El objeto del desencadenador de temporizador es camelCase en lugar de PascalCase

* Las funciones desencadenadas por el centro de eventos con el binario `dataType` recibirán una matriz de `binary` en lugar de `string`.

* Ya no se puede obtener acceso a la carga de la solicitud HTTP mediante `context.bindingData.req`.  Todavía se puede acceder a él como parámetro de entrada, `context.req`, y en `context.bindings`.

* Node.js 8 ya no se admite y no se ejecutará en las funciones de 3.x.

#### <a name="net"></a>.NET

* [Las operaciones de servidor sincrónicas están deshabilitadas de forma predeterminada](/dotnet/core/compatibility/2.2-3.0#http-synchronous-io-disabled-in-all-servers).

### <a name="changing-version-of-apps-in-azure"></a>Cambio de la versión de las aplicaciones en Azure

La versión del sistema en ejecución de Functions que usan las aplicaciones publicadas en Azure viene determinada por la configuración de la aplicación [`FUNCTIONS_EXTENSION_VERSION`](functions-app-settings.md#functions_extension_version). Se admiten los siguientes valores principales de la versión del entorno en tiempo de ejecución:

| Value | Destino del entorno en tiempo de ejecución |
| ------ | -------- |
| `~3` | 3.x |
| `~2` | 2.x |
| `~1` | 1.x |

>[!IMPORTANT]
> No cambie esta configuración sin motivo, ya que puede requerir otros cambios de configuración de la aplicación y cambios en el código de función.

### <a name="locally-developed-application-versions"></a>Versiones de aplicaciones desarrolladas de forma local

Puede hacer que las siguientes actualizaciones de las aplicaciones de funciones cambien localmente las versiones de destino.

#### <a name="visual-studio-runtime-versions"></a>Versiones de Runtime en Visual Studio

En Visual Studio se selecciona la versión del entorno de ejecución al crear un proyecto. Las herramientas de Azure Functions para Visual Studio son compatibles con las tres versiones principales del entorno de ejecución. Se usa la versión correcta al depurar y publicar en función de configuración del proyecto. La configuración de la versión se define en el archivo `.csproj` en las siguientes propiedades:

##### <a name="version-1x"></a>Versión 1.x

```xml
<TargetFramework>net461</TargetFramework>
<AzureFunctionsVersion>v1</AzureFunctionsVersion>
```

##### <a name="version-2x"></a>Versión 2.x

```xml
<TargetFramework>netcoreapp2.1</TargetFramework>
<AzureFunctionsVersion>v2</AzureFunctionsVersion>
```

##### <a name="version-3x"></a>Versión 3.x

```xml
<TargetFramework>netcoreapp3.1</TargetFramework>
<AzureFunctionsVersion>v3</AzureFunctionsVersion>
```

> [!NOTE]
> Azure Functions 3.x y .NET requieren que la extensión `Microsoft.NET.Sdk.Functions` sea al menos `3.0.0`.

###### <a name="updating-2x-apps-to-3x-in-visual-studio"></a>Actualización de las aplicaciones de 2.x a 3.x en Visual Studio

Puede abrir una función existente que tenga como destino 2.x y pasar a 3.x editando el archivo `.csproj` y actualizando los valores anteriores.  Visual Studio administra las versiones del entorno en tiempo de ejecución automáticamente en función de los metadatos del proyecto.  Sin embargo, es posible si nunca ha creado una aplicación 3.x antes de que Visual Studio no tuviera aún las plantillas y el entorno en tiempo de ejecución de 3.x en la máquina.  Esto puede presentar un error como "no hay ningún entorno en tiempo de ejecución de Functions disponible que coincida con la versión especificada en el proyecto".  Para obtener las plantillas y el entorno en tiempo de ejecución más recientes, consulte la experiencia para crear un nuevo proyecto de función.  Cuando llegue a la pantalla de selección de versión y plantilla, espere a que Visual Studio complete la captura de las plantillas más recientes.  Una vez que las plantillas más recientes de .NET Core 3 están disponibles y se muestran, debería poder ejecutar y depurar cualquier proyecto configurado para la versión 3.x.

> [!IMPORTANT]
> Las funciones de la versión 3.x solo se pueden desarrollar en Visual Studio si se usa la Visual Studio versión 16.4 o una más reciente.

#### <a name="vs-code-and-azure-functions-core-tools"></a>VS Code y Azure Functions Core Tools

[Azure Functions Core Tools](functions-run-local.md) se usa para el desarrollo de la línea de comandos, pero también lo usa la [extensión de Azure Functions](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) para Visual Studio Code. Para desarrollar con la versión 3.x, instale la versión 3.x de Core Tools. El desarrollo con la versión 2.x requiere la versión 2.x de Core Tools, etc. Para más información, consulte [Instalación de Azure Functions Core Tools](functions-run-local.md#install-the-azure-functions-core-tools).

Para el desarrollo en Visual Studio Code es posible que deba actualizar la configuración de usuario en `azureFunctions.projectRuntime` para que coincida con la versión de las herramientas instaladas.  Esta configuración también actualiza las plantillas y los lenguajes utilizados durante la creación de la aplicación de función.  Para crear aplicaciones en `~3`, se debe actualizar la configuración de usuario de `azureFunctions.projectRuntime` a `~3`.

![Configuración del entorno en tiempo de ejecución de la extensión de Azure Functions](./media/functions-versions/vs-code-version-runtime.png)

#### <a name="maven-and-java-apps"></a>Aplicaciones de Maven y Java

Puede migrar aplicaciones Java de la versión 2.x a la versión 3.x [instalando la versión 3.x de las herramientas principales](functions-run-local.md#install-the-azure-functions-core-tools) necesaria para ejecutarse localmente.  Después de comprobar que la aplicación se ejecuta correctamente en la versión 3.x, actualice el archivo de `POM.xml` de la aplicación para modificar la configuración de `FUNCTIONS_EXTENSION_VERSION` a `~3`, como en el ejemplo siguiente:

```xml
<configuration>
    <resourceGroup>${functionResourceGroup}</resourceGroup>
    <appName>${functionAppName}</appName>
    <region>${functionAppRegion}</region>
    <appSettings>
        <property>
            <name>WEBSITE_RUN_FROM_PACKAGE</name>
            <value>1</value>
        </property>
        <property>
            <name>FUNCTIONS_EXTENSION_VERSION</name>
            <value>~3</value>
        </property>
    </appSettings>
</configuration>
```

## <a name="bindings"></a>Enlaces

A partir de la versión 2.x, el entorno de ejecución usa un nuevo [modelo de extensibilidad de enlaces](https://github.com/Azure/azure-webjobs-sdk-extensions/wiki/Binding-Extensions-Overview) que ofrece estas ventajas:

* Compatibilidad con extensiones de enlace de terceros.

* Desacoplamiento de tiempo de ejecución y enlaces. Este cambio permite el control de versiones y la publicación de las extensiones de enlace de forma independiente. Por ejemplo, puede elegir actualizar a una versión de una extensión que se basa en una versión más reciente de un SDK subyacente.

* Un entorno de ejecución más ligero, donde solo se conocen y se cargan en tiempo de ejecución los enlaces que están en uso.

A excepción de los desencadenadores HTTP y el temporizador, todos los enlaces deben agregarse explícitamente al proyecto de aplicación de función o registrarse en el portal. Para más información, consulte [Registro de extensiones de enlace](./functions-bindings-expressions-patterns.md).

En la siguiente tabla se indica qué enlaces se admiten en cada versión del entorno de ejecución.

[!INCLUDE [Full bindings table](../../includes/functions-bindings.md)]

[!INCLUDE [Timeout Duration section](../../includes/functions-timeout-duration.md)]

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, consulte los siguientes recursos:

* [Codificación y comprobación de Azure Functions en un entorno local](functions-run-local.md)
* [Cómo elegir las versiones del entorno de ejecución de Azure Functions](set-runtime-version.md)
* [Notas de la versión](https://github.com/Azure/azure-functions-host/releases)
