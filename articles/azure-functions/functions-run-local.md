---
title: Uso de Azure Functions Core Tools | Microsoft Docs
description: Aprenda a codificar y probar funciones de Azure en el símbolo del sistema o terminal en la máquina local antes de ejecutarlas en Azure Functions.
services: functions
documentationcenter: na
author: ggailey777
manager: jeconnoc
ms.assetid: 242736be-ec66-4114-924b-31795fd18884
ms.service: azure-functions
ms.devlang: multiple
ms.topic: conceptual
ms.date: 03/13/2019
ms.author: glenga
ms.custom: 80e4ff38-5174-43
ms.openlocfilehash: 7c6e7d8bb407b0ffeb320ebfe9e2639feb303800
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65603414"
---
# <a name="work-with-azure-functions-core-tools"></a>Uso de Azure Functions Core Tools

Azure Functions Core Tools le permite desarrollar y probar funciones en el equipo local desde el símbolo del sistema o terminal. Las funciones locales pueden conectarse a servicios de Azure en directo, y puede depurar sus funciones en el equipo local con el tiempo de ejecución de Functions completo. Incluso puede implementar una aplicación de función en su suscripción a Azure.

[!INCLUDE [Don't mix development environments](../../includes/functions-mixed-dev-environments.md)]

Desarrollo de funciones en el equipo local y publicarlas en Azure mediante herramientas principales sigue estos pasos básicos:

> [!div class="checklist"]
> * [Instale las herramientas de Core y las dependencias.](#v2)
> * [Crear un proyecto de aplicación de función desde una plantilla específica del lenguaje.](#create-a-local-functions-project)
> * [Registrar extensiones de enlace y desencadenador.](#register-extensions)
> * [Definir el almacenamiento y otras conexiones.](#local-settings-file)
> * [Cree una función de un desencadenador y una plantilla específica del lenguaje.](#create-func)
> * [Ejecute la función localmente](#start)
> * [Publicar el proyecto en Azure](#publish)

## <a name="core-tools-versions"></a>Versiones de Core Tools

Hay dos versiones de Azure Functions Core Tools. La versión que use depende del entorno de desarrollo local, la [elección del lenguaje](supported-languages.md) y el nivel de compatibilidad necesario:

+ Versión 1.x: admite la versión 1.x del runtime. Esta versión de las herramientas solo se admite en equipos con Windows y se instala desde un [paquete npm](https://docs.npmjs.com/getting-started/what-is-npm). Con esta versión, puede crear funciones en lenguajes experimentales que no se admiten oficialmente. Para más información, consulte [Lenguajes admitidos en Azure Functions](supported-languages.md).

+ [Versión 2.x](#v2): admite la [versión 2.x del entorno en tiempo de ejecución](functions-versions.md). Esta versión admite [Windows](#windows-npm), [macOS](#brew) y [Linux](#linux). Usa administradores de paquetes específicos de la plataforma o npm para la instalación.

A menos que se indique lo contrario, los ejemplos de este artículo son para la versión 2.x.

## <a name="install-the-azure-functions-core-tools"></a>Instalación de Azure Functions Core Tools

[Azure Functions Core Tools] incluye una versión del mismo tiempo de ejecución de Azure Functions que puede ejecutar en el equipo de desarrollo local. También proporciona comandos para crear funciones, conectarse a Azure e implementar proyectos de funciones.

### <a name="v2"></a>Versión 2.x

La versión 2.x de las herramientas usa el entorno en tiempo de ejecución 2.x de Azure Functions, que se basa en .NET Core. Esta versión se admite en todas las plataformas que admiten .NET Core 2.x, incluidas [Windows](#windows-npm), [macOS](#brew) y [Linux](#linux). En primer lugar debe instalar el SDK de .NET Core 2.x.

#### <a name="windows-npm"></a>Windows

Los pasos siguientes utilizan npm para instalar Core Tools en Windows. También puede usar [Chocolatey](https://chocolatey.org/). Para más información, consulte el [archivo Léame de Core Tools](https://github.com/Azure/azure-functions-core-tools/blob/master/README.md#windows).

1. Instale [el SDK de .NET Core 2.x para Windows](https://www.microsoft.com/net/download/windows).

2. Instale [Node.js], que incluye npm. Para la versión 2.x de las herramientas, solo se admite Node.js 8.5 y versiones posteriores.

3. Instale el paquete de Core Tools:

    ```bash
    npm install -g azure-functions-core-tools
    ```

#### <a name="brew"></a>MacOS con Homebrew

Los pasos siguientes utilizan Homebrew para instalar Core Tools en macOS.

1. Instale [el SDK de .NET Core 2.x para macOS](https://www.microsoft.com/net/download/macos).

2. Instale [Homebrew](https://brew.sh/), si aún no está instalado.

3. Instale el paquete de Core Tools:

    ```bash
    brew tap azure/functions
    brew install azure-functions-core-tools
    ```

#### <a name="linux"></a> Linux (Ubuntu/Debian) con APT

Los siguientes pasos usan [APT](https://wiki.debian.org/Apt) para instalar Core Tools en la distribución de Ubuntu/Debian Linux. Para otras distribuciones de Linux, consulte el [archivo Léame de Core Tools](https://github.com/Azure/azure-functions-core-tools/blob/master/README.md#linux).

1. Instale [el SDK de .NET Core 2.x para Linux](https://www.microsoft.com/net/download/linux).

2. Registre la clave de producto de Microsoft como de confianza:

    ```bash
    curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
    sudo mv microsoft.gpg /etc/apt/trusted.gpg.d/microsoft.gpg
    ```

3. Compruebe que el servidor de Ubuntu ejecuta una de las versiones adecuadas de la siguiente tabla. Para agregar el origen apt, ejecute:

    ```bash
    sudo sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-ubuntu-$(lsb_release -cs)-prod $(lsb_release -cs) main" > /etc/apt/sources.list.d/dotnetdev.list'
    sudo apt-get update
    ```

    | Distribución de Linux | Version |
    | --------------- | ----------- |
    | Ubuntu 18.10    | `cosmic`    |
    | Ubuntu 18.04    | `bionic`    |
    | Ubuntu 17.04    | `zesty`     |
    | Ubuntu 16.04/Linux Mint 18    | `xenial`  |

4. Instale el paquete de Core Tools:

    ```bash
    sudo apt-get install azure-functions-core-tools
    ```

## <a name="create-a-local-functions-project"></a>Creación de un proyecto local de Functions

Un directorio de proyecto de funciones contiene los archivos [host.json](functions-host-json.md) y [local.settings.json](#local-settings-file), junto con las subcarpetas que contienen el código de funciones individuales. Este directorio es el equivalente a una aplicación de función en Azure. Para obtener más información sobre la estructura de carpetas de Functions, vea la [Guía para desarrolladores de Azure Functions](functions-reference.md#folder-structure).

Para la versión 2.x deberá seleccionar un lenguaje predeterminado para el proyecto cuando se inicializa, y todas las funciones de agregan con plantillas de lenguaje predeterminado. En la versión 1.x, debe especificar el lenguaje cada vez que crea una función.

En la ventana de terminal o desde un símbolo del sistema, ejecute el siguiente comando para crear el proyecto y el repositorio de Git local:

```bash
func init MyFunctionProj
```

Al especificar un nombre de proyecto, se crea una carpeta con dicho nombre y posteriormente se inicializa. En caso contrario, se inicializa la carpeta actual.  
En la versión 2.x, cuando ejecute el comando, debe elegir un tiempo de ejecución para el proyecto. Si va a desarrollar funciones de JavaScript, elija **node**:

```output
Select a worker runtime:
dotnet
node
```

Use las flechas arriba/abajo para elegir un lenguaje, a continuación, presione ENTRAR. La salida tendrá un aspecto similar al siguiente ejemplo de un proyecto de JavaScript:

```output
Select a worker runtime: node
Writing .gitignore
Writing host.json
Writing local.settings.json
Writing C:\myfunctions\myMyFunctionProj\.vscode\extensions.json
Initialized empty Git repository in C:/myfunctions/myMyFunctionProj/.git/
```

`func init` admite las siguientes opciones, que corresponden solo a la versión 2.x, a menos que se indique lo contrario:

| Opción     | DESCRIPCIÓN                            |
| ------------ | -------------------------------------- |
| **`--csx`** | Inicializa un proyecto de script de C# (.csx). Debe especificar `--csx` en los siguientes comandos. |
| **`--docker`** | Crea un archivo Dockerfile para un contenedor con una imagen base en función del `--worker-runtime` elegido. Use esta opción cuando vaya a publicar en un contenedor Linux personalizado. |
| **`--force`** | Inicializa el proyecto incluso cuando hay archivos existentes en el proyecto. Este valor sobrescribe los archivos existentes con el mismo nombre. Los otros archivos de la carpeta del proyecto no se ven afectados. |
| **`--no-source-control -n`** | Impide la creación de forma predeterminada de un repositorio de Git en la versión 1.x. En la versión 2.x, el repositorio de git no se crea de forma predeterminada. |
| **`--source-control`** | Controla si se crea un repositorio de git. De forma predeterminada, no se crea un repositorio. Cuando es `true`, se crea un repositorio. |
| **`--worker-runtime`** | Establece el entorno de ejecución del lenguaje del proyecto. Los valores admitidos son `dotnet`, `node` (JavaScript), `java` y `python`. Si no se establece, deberá elegir el entorno de ejecución durante la inicialización. |

> [!IMPORTANT]
> De manera predeterminada, la versión 2.x de Core Tools crea proyectos de aplicación de función para el runtime de .NET como [proyectos de clase de C#](functions-dotnet-class-library.md) (.csproj). Estos proyectos de C#, que se pueden usar con Visual Studio o con Visual Studio Code, se compilan durante las pruebas y al publicar en Azure. Si en su lugar desea crear y trabajar con los mismos archivos de script de C# (.csx) creados en la versión 1.x y en el portal, debe incluir el parámetro `--csx` cuando cree e implemente las funciones.

## <a name="register-extensions"></a>Registro de las extensiones

En la versión 2.x del entorno de ejecución de Azure Functions, debe registrar explícitamente las extensiones de enlace (tipos de enlace) que utilice en la aplicación de función.

[!INCLUDE [Register extensions](../../includes/functions-core-tools-install-extension.md)]

Para más información, consulte [Conceptos básicos sobre los enlaces y desencadenadores de Azure Functions](./functions-bindings-expressions-patterns.md).

## <a name="local-settings-file"></a>Archivo de configuración local

El archivo local.settings.json almacena la configuración de la aplicación, las cadenas de conexión y la configuración de Azure Functions Core Tools. Las herramientas de Functions solo usan las opciones de configuración de dicho archivo cuando las herramientas se ejecutan localmente. De manera predeterminada, estas opciones de configuración no se migran automáticamente cuando el proyecto se publica en Azure. Use el conmutador `--publish-local-settings` [al publicarlo](#publish) para asegurarse de que la configuración se agregue a la aplicación de función en Azure. Tenga en cuenta que los valores de **ConnectionStrings** nunca se publican. El archivo tiene la siguiente estructura:

```json
{
  "IsEncrypted": false,
  "Values": {
    "FUNCTIONS_WORKER_RUNTIME": "<language worker>",
    "AzureWebJobsStorage": "<connection-string>",
    "AzureWebJobsDashboard": "<connection-string>",
    "MyBindingConnection": "<binding-connection-string>"
  },
  "Host": {
    "LocalHttpPort": 7071,
    "CORS": "*"
  },
  "ConnectionStrings": {
    "SQLConnectionString": "<sqlclient-connection-string>"
  }
}
```

| Configuración      | DESCRIPCIÓN                            |
| ------------ | -------------------------------------- |
| **`IsEncrypted`** | Cuando se establece en `true`, todos los valores se cifran mediante una clave del equipo local. Se usa con los comandos `func settings`. El valor predeterminado es `false`. |
| **`Values`** | Colección de opciones de configuración de la aplicación y las cadenas de conexión que se usan en la ejecución local. Estos valores se corresponden con la configuración de la aplicación en la aplicación de función en Azure, como [ `AzureWebJobsStorage` ]. Muchos desencadenadores y enlaces tienen una propiedad que hace referencia a una configuración de la aplicación de cadena de conexión, como `Connection` para el [desencadenador de Blob storage](functions-bindings-storage-blob.md#trigger---configuration). Para estas propiedades, se necesita una configuración de aplicación definido en el `Values` matriz. <br/>[`AzureWebJobsStorage`] es una aplicación requiere la configuración para los desencadenadores que no sean HTTP. <br/>Versión 2.x de Functions runtime requiere la [ `FUNCTIONS_WORKER_RUNTIME` ] configuración, que se genera para el proyecto de Core Tools. <br/> Cuando tenga el [emulador de Azure storage](../storage/common/storage-use-emulator.md) instalado localmente, puede establecer [ `AzureWebJobsStorage` ] a `UseDevelopmentStorage=true` y Core Tools usa el emulador. Esto es útil durante el desarrollo, pero debe probar con una conexión de almacenamiento real antes de la implementación. |
| **`Host`** | La configuración que se muestra esta sección permite personalizar el proceso de host de Functions cuando se ejecuta localmente. |
| **`LocalHttpPort`** | Establece el puerto predeterminado que se usa cuando al ejecutar el host de Functions local (`func host start` y `func run`). La opción de línea de comandos `--port` tiene prioridad sobre este valor. |
| **`CORS`** | Define los orígenes permitidos para el [uso compartido de recursos entre orígenes (CORS)](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing). Los orígenes se proporcionan en una lista de valores separados por comas y sin espacios. Se admite el valor comodín (\*), lo que permite realizar solicitudes desde cualquier origen. |
| **`ConnectionStrings`** | No utilice esta colección para las cadenas de conexión que utilizan los enlaces de función. Esta colección solo se usa por marcos de trabajo que normalmente se obtienen las cadenas de conexión desde el `ConnectionStrings` sección de configuración de una archivo, como [Entity Framework](https://msdn.microsoft.com/library/aa937723(v=vs.113).aspx). Las cadenas de conexión de este objeto se agregan al entorno con el tipo de proveedor de [System.Data.SqlClient](https://msdn.microsoft.com/library/system.data.sqlclient(v=vs.110).aspx). Los elementos de esta colección no se publican en Azure con otra configuración de aplicación. Debe agregar explícitamente estos valores para el `Connection strings` colección de la configuración de function app. Si está creando un [ `SqlConnection` ](https://msdn.microsoft.com/library/system.data.sqlclient.sqlconnection(v=vs.110).aspx) en el código de función, debe almacenar el valor de cadena de conexión **configuración de la aplicación** en el portal con las otras conexiones. |

Esta configuración de la aplicación de función también se puede leer en el código como variables de entorno. Para más información, consulte la sección Variables de entorno de estos temas de referencia específicos del lenguaje:

* [C# precompilado](functions-dotnet-class-library.md#environment-variables)
* [Script de C# (.csx)](functions-reference-csharp.md#environment-variables)
* [Script de F# (.fsx)](functions-reference-fsharp.md#environment-variables)
* [Java](functions-reference-java.md#environment-variables)
* [JavaScript](functions-reference-node.md#environment-variables)

Cuando no se establece ninguna cadena de conexión de almacenamiento válida para [ `AzureWebJobsStorage` ] y no se usa el emulador, se muestra el mensaje de error siguiente:

> Missing value for AzureWebJobsStorage in local.settings.json. This is required for all triggers other than HTTP. You can run 'func azure functionapp fetch-app-settings \<functionAppName\>' or specify a connection string in local.settings.json (Puede ejecutar "func azure functionapp fetch-app-settings <functionAppName>" o especificar una cadena de conexión en local.settings.json).

### <a name="get-your-storage-connection-strings"></a>Obtención de las cadenas de conexión de almacenamiento

Incluso cuando se usa el emulador de almacenamiento para tareas de desarrollo, recomendamos probar con una conexión de almacenamiento real. Suponiendo que ya [creó una cuenta de almacenamiento](../storage/common/storage-create-storage-account.md), puede obtener una cadena de conexión de almacenamiento válida de una de las maneras siguientes:

+ Desde [Azure Portal]. Navegue a su cuenta de almacenamiento, seleccione **Claves de acceso** en **Configuración** y, a continuación, copie uno de los valores de **Cadena de conexión**.

  ![Copia de una cadena de conexión desde Azure Portal](./media/functions-run-local/copy-storage-connection-portal.png)

+ Use [Explorador de Azure Storage](https://storageexplorer.com/) para conectarse a su cuenta de almacenamiento de Azure. En el **Explorador**, expanda su suscripción, seleccione la cuenta de almacenamiento y copie la cadena de conexión principal o secundaria.

  ![Copia de la cadena de conexión desde el Explorador de Azure Storage](./media/functions-run-local/storage-explorer.png)

+ Utilice Core Tools para descargar la cadena de conexión de Azure con uno de los siguientes comandos:

  + Descargue toda la configuración de una aplicación de función existente:

    ```bash
    func azure functionapp fetch-app-settings <FunctionAppName>
    ```
  + Obtenga la cadena de conexión de una cuenta de almacenamiento concreta:

    ```bash
    func azure storage fetch-connection-string <StorageAccountName>
    ```

    Si aún no ha iniciado sesión en Azure, se le pedirá que lo haga.

## <a name="create-func"></a>Creación de una función

Para crear una función, ejecute el siguiente comando:

```bash
func new
```

En la versión 2.x, al ejecutar `func new` se le pedirá que elija una plantilla en el lenguaje predeterminado de la aplicación de función y, después, también se le pedirá que elija un nombre para la función. En la versión 1.x, también se le pedirá que elija el lenguaje.

```output
Select a language: Select a template:
Blob trigger
Cosmos DB trigger
Event Grid trigger
HTTP trigger
Queue trigger
SendGrid
Service Bus Queue trigger
Service Bus Topic trigger
Timer trigger
```

El código de la función se genera en una subcarpeta con el nombre proporcionado de la función, como se aprecia en la siguiente salida de desencadenador de cola:

```output
Select a language: Select a template: Queue trigger
Function name: [QueueTriggerJS] MyQueueTrigger
Writing C:\myfunctions\myMyFunctionProj\MyQueueTrigger\index.js
Writing C:\myfunctions\myMyFunctionProj\MyQueueTrigger\readme.md
Writing C:\myfunctions\myMyFunctionProj\MyQueueTrigger\sample.dat
Writing C:\myfunctions\myMyFunctionProj\MyQueueTrigger\function.json
```

También puede especificar estas opciones en el comando con los argumentos siguientes:

| Argumento     | DESCRIPCIÓN                            |
| ------------------------------------------ | -------------------------------------- |
| **`--csx`** | (Versión 2.x) Genera las mismas plantillas de script de C# (.csx) que se usan en la versión 1.x y en el portal. |
| **`--language -l`**| Lenguaje de programación de la plantilla, como C#, F# o JavaScript. Esta opción es obligatoria en la versión 1.x. En la versión 2.x, no utilice esta opción o elija un lenguaje que coincida con el entorno de ejecución del trabajo. |
| **`--name -n`** | Nombre de la función. |
| **`--template -t`** | Use el comando `func templates list` para ver la lista completa de plantillas disponibles para cada lenguaje compatible.   |

Por ejemplo, para crear un desencadenador HTTP de JavaScript en un único comando, ejecute:

```bash
func new --template "Http Trigger" --name MyHttpTrigger
```

Para crear una función desencadenada por la cola en un único comando, ejecute:

```bash
func new --template "Queue Trigger" --name QueueTriggerJS
```

## <a name="start"></a>Ejecución local de funciones

Para ejecutar un proyecto de Functions, ejecute el host de Functions. El host habilita desencadenadores para todas las funciones del proyecto:

```bash
func host start
```

El comando `host` solo es necesario en la versión 1.x.

`func host start` admite las siguientes opciones:

| Opción     | DESCRIPCIÓN                            |
| ------------ | -------------------------------------- |
| **`--no-build`** | No compile del proyecto actual antes de su ejecución. Solo para proyectos de dotnet. El valor predeterminado se establece en false. Solo versión 2.x. |
| **`--cert`** | La ruta de acceso a un archivo .pfx que contiene una clave privada. Solo se usa con `--useHttps`. Solo versión 2.x. |
| **`--cors-credentials`** | Permitir solicitudes autenticadas de varios orígenes (es decir, cookies y el encabezado de autenticación) Solo versión 2.x. |
| **`--cors`** | Lista separada por comas de orígenes CORS, sin espacios en blanco. |
| **`--language-worker`** | Argumentos para configurar el trabajo del lenguaje. Solo versión 2.x. |
| **`--nodeDebugPort -n`** | Puerto del depurador de nodo que se va a usar. Valor predeterminado: un valor de launch.json o 5858. Solo versión 1.x. |
| **`--password`** | La contraseña o un archivo que contenga la contraseña de un archivo. pfx. Solo se usa con `--cert`. Solo versión 2.x. |
| **`--port -p`** | Puerto local en el que se escucha. Valor predeterminado: 7071. |
| **`--pause-on-error`** | Se pone en pausa en espera de entrada adicional antes de salir del proceso. Se utiliza solo cuando se inicia Core Tools desde un entorno de desarrollo integrado (IDE).|
| **`--script-root --prefix`** | Se usa para especificar la ruta de acceso a la raíz de la aplicación de función que se va a ejecutar o implementar. Esto se usa para los proyectos compilados que generan archivos de proyecto en una subcarpeta. Por ejemplo, cuando se compila un proyecto de biblioteca de clases de C#, los archivos host.json, local.settings.json y function.json se generan en una subcarpeta *raíz* con una ruta de acceso similar a `MyProject/bin/Debug/netstandard2.0`. En este caso, establezca el prefijo como `--script-root MyProject/bin/Debug/netstandard2.0`. Esta es la raíz de la aplicación de función cuando se ejecuta en Azure. |
| **`--timeout -t`** | Tiempo de espera en segundos para que se inicie el host de Functions. Valor predeterminado: 20 segundos.|
| **`--useHttps`** | Enlace con `https://localhost:{port}` en lugar de con `http://localhost:{port}`. De forma predeterminada, esta opción crea un certificado de confianza en el equipo.|

En el caso de un proyecto de la biblioteca de clases de C# (.csproj), debe incluir la opción `--build` para generar el archivo .dll de biblioteca.

Cuando se inicia el host de Functions, devuelve la dirección URL de las funciones desencadenadas por HTTP:

```output
Found the following functions:
Host.Functions.MyHttpTrigger

Job host started
Http Function MyHttpTrigger: http://localhost:7071/api/MyHttpTrigger
```

>[!IMPORTANT]
>Cuando se ejecuta localmente, no se aplica la autenticación para puntos de conexión HTTP. Esto significa que todas las solicitudes HTTP locales se tratan como `authLevel = "anonymous"`. Para obtener más información, consulte el artículo sobre [enlaces HTTP](functions-bindings-http-webhook.md#authorization-keys).

### <a name="passing-test-data-to-a-function"></a>Paso de datos de prueba a una función

Para probar sus funciones localmente, [inicie el host de Functions](#start) y llame a puntos de conexión del servidor local mediante solicitudes HTTP. El punto de conexión al que llama depende del tipo de función.

>[!NOTE]
> En los ejemplos de este tema se usa la herramienta cURL para enviar solicitudes HTTP desde el terminal o un símbolo del sistema. Puede usar una herramienta de su elección para enviar solicitudes HTTP al servidor local. La herramienta cURL está disponible de forma predeterminada en los sistemas basados en Linux. En Windows, primero debe descargar e instalar la [herramienta cURL](https://curl.haxx.se/).

Para obtener información más general sobre cómo probar funciones, consulte [Estrategias para probar el código en Azure Functions](functions-test-a-function.md).

#### <a name="http-and-webhook-triggered-functions"></a>Funciones desencadenadas por HTTP y webhook

Llama al siguiente punto de conexión para ejecutar de forma local funciones desencadenadas por HTTP y webhook:

    http://localhost:{port}/api/{function_name}

Asegúrese de usar el mismo nombre del servidor y puerto en el que escucha el host de Functions. Puede ver esto en la salida generada al iniciar el host de Functions. Puede llamar a esta dirección URL mediante cualquier método HTTP admitido por el desencadenador.

El siguiente comando cURL desencadena la función de inicio rápido `MyHttpTrigger` desde una solicitud GET con el parámetro _name_ transferido en la cadena de consulta.

```bash
curl --get http://localhost:7071/api/MyHttpTrigger?name=Azure%20Rocks
```

En el siguiente ejemplo está la misma función a la que se llama desde una solicitud POST que transfiere _name_ en el cuerpo de la solicitud:

```bash
curl --request POST http://localhost:7071/api/MyHttpTrigger --data '{"name":"Azure Rocks"}'
```

Puede realizar solicitudes GET desde un explorador que transfiere datos en la cadena de consulta. Para todos los demás métodos HTTP, debe usar cURL, Fiddler, Postman o una herramienta de pruebas HTTP similar.

#### <a name="non-http-triggered-functions"></a>Funciones no desencadenadas por HTTP

En el caso de todos los tipos de funciones distintas de los desencadenadores HTTP y HTTP webhooks, puede probar sus funciones localmente llamando a un punto de conexión de administración. Al llamar a este punto de conexión con una solicitud HTTP POST en el servidor local se desencadena esta función. Opcionalmente, puede transferir los datos de prueba a la ejecución en el cuerpo de la solicitud POST. Esta funcionalidad es similar a la pestaña **Prueba** de Azure Portal.

Se llama al siguiente punto de conexión de administrador para desencadenar funciones ajenas a HTTP:

    http://localhost:{port}/admin/functions/{function_name}

Para transferir datos de prueba al punto de conexión de administrador de una función, debe proporcionar los datos en el cuerpo de un mensaje de solicitud POST. Es necesario que el cuerpo del mensaje tenga el siguiente formato JSON:

```JSON
{
    "input": "<trigger_input>"
}
```

El valor `<trigger_input>` contiene datos en un formato esperado por la función. El siguiente ejemplo de cURL es una solicitud POST dirigida a una función `QueueTriggerJS`. En este caso, la entrada es una cadena que equivale al mensaje que se espera encontrar en la cola.

```bash
curl --request POST -H "Content-Type:application/json" --data '{"input":"sample queue data"}' http://localhost:7071/admin/functions/QueueTriggerJS
```

#### <a name="using-the-func-run-command-in-version-1x"></a>Uso del comando `func run` en la versión 1.x

>[!IMPORTANT]
> El comando `func run` no se admite en la versión 2.x de las herramientas. Para obtener más información, consulte el tema [How to target Azure Functions runtime versions](set-runtime-version.md) (Cómo seleccionar un destino para versiones en tiempo de ejecución de Azure Functions).

También puede invocar una función directamente con `func run <FunctionName>` y proporcionar datos de entrada para la función. Este comando es similar a la ejecución de una función con la pestaña **Prueba** de Azure Portal.

`func run` admite las siguientes opciones:

| Opción     | DESCRIPCIÓN                            |
| ------------ | -------------------------------------- |
| **`--content -c`** | Contenido alineado. |
| **`--debug -d`** | Se asocia un depurador al proceso de host antes de ejecutar la función.|
| **`--timeout -t`** | Tiempo de espera (en segundos) hasta que el host local de Functions está listo.|
| **`--file -f`** | Nombre del archivo que se usa como contenido.|
| **`--no-interactive`** | No pide entrada. Resulta útil en escenarios de automatización.|

Por ejemplo, para llamar a una función desencadenada por HTTP y pasar cuerpo del contenido, ejecute el siguiente comando:

```bash
func run MyHttpTrigger -c '{\"name\": \"Azure\"}'
```

## <a name="publish"></a>Publicación en Azure

Core Tools admite dos tipos de implementación: implementar los archivos del proyecto de función directamente en la aplicación de función e implementar un contenedor Linux personalizado, lo que se admite únicamente en la versión 2.x. Previamente, debe haber [creado una aplicación de función en la suscripción de Azure](functions-cli-samples.md#create).

En la versión 2.x, debe haber [registrado las extensiones](#register-extensions) en el proyecto antes de la publicación. Se deben compilar los proyectos que lo requieran para poder implementar los archivos binarios.

### <a name="project-file-deployment"></a>Implementación del archivo de proyecto

El método de implementación más común implica el uso de Core Tools para empaquetar el proyecto de aplicación de función, archivos binarios y dependencias e implementar el paquete en la aplicación de función. También puede [ejecutar las funciones directamente desde el paquete de implementación](run-functions-from-deployment-package.md).

Para publicar un proyecto de Functions en una aplicación de función en Azure, use el comando `publish`:

```bash
func azure functionapp publish <FunctionAppName>
```

Este comando se publica en una aplicación de función existente en Azure. Se produce un error cuando `<FunctionAppName>` no existe en la suscripción. Para obtener información sobre cómo crear una aplicación de función desde el símbolo del sistema o la ventana de Terminal mediante la CLI de Azure, consulte [Creación de una instancia de Function App para la ejecución sin servidor](./scripts/functions-cli-create-serverless.md).

El comando `publish` carga el contenido del directorio del proyecto de Functions. Si elimina archivos localmente, el comando `publish` no los eliminará de Azure. Puede eliminar archivos de Azure con la [herramienta Kudu](functions-how-to-use-azure-function-app-settings.md#kudu) de [Azure Portal].

>[!IMPORTANT]
> Cuando se crea una aplicación de función en Azure Portal, se usa la versión 2.x del entorno de ejecución de Functions de forma predeterminada. Para hacer que la aplicación de función utilice la versión 1.x del entorno de ejecución, siga las instrucciones de [Ejecución en la versión 1.x](functions-versions.md#creating-1x-apps).
> No se puede cambiar la versión del entorno de ejecución de una aplicación de función que tiene funciones existentes.

Las siguientes opciones de publicación del proyecto se aplican a ambas versiones, 1.x y 2.x:

| Opción     | DESCRIPCIÓN                            |
| ------------ | -------------------------------------- |
| **`--publish-local-settings -i`** |  Se publica la configuración de local.settings.json en Azure, se pide que se sobrescriba si la configuración ya existe. Si usa el Emulador de Storage, cambie la configuración de la aplicación a una [conexión de almacenamiento real](#get-your-storage-connection-strings). |
| **`--overwrite-settings -y`** | Suprime el mensaje de sobrescritura de la configuración de la aplicación cuando se utiliza `--publish-local-settings -i`.|

Las siguientes opciones de publicación del proyecto solo se admiten en la versión 2.x:

| Opción     | DESCRIPCIÓN                            |
| ------------ | -------------------------------------- |
| **`--publish-settings-only -o`** |  Solo se publica la configuración y se omite el contenido. El valor predeterminado es Preguntar. |
|**`--list-ignored-files`** | Muestra una lista de archivos que se omiten durante la publicación, según el archivo .funcignore. |
| **`--list-included-files`** | Muestra una lista de archivos que se publican, según el archivo .funcignore. |
| **`--nozip`** | Desactiva el modo `Run-From-Zip` predeterminado. |
| **`--build-native-deps`** | Omite la generación de la carpeta .wheels al publicar aplicaciones de función de Python. |
| **`--additional-packages`** | Lista de paquetes para instalar al crear dependencias nativas. Por ejemplo: `python3-dev libevent-dev`. |
| **`--force`** | Omite la comprobación previa a la publicación en determinados escenarios. |
| **`--csx`** | Publica un proyecto de script de C# (.csx). |
| **`--no-build`** | Omite la compilación de funciones de dotnet. |
| **`--dotnet-cli-params`** | Al publicar funciones de C# compiladas (.csproj), Core Tools llama a "dotnet build --output bin/publish". Todos los parámetros pasados se anexarán a la línea de comandos. |

### <a name="custom-container-deployment"></a>Implementación en un contenedor personalizado

Functions permite implementar el proyecto de función en un contenedor Linux personalizado. Para más información, consulte [Creación de una función en Linux con una imagen personalizada](functions-create-function-linux-custom-image.md). La versión 2.x de Core Tools admite la implementación de un contenedor personalizado. Los contenedores personalizados deben tener un archivo Dockerfile. Utilice la opción --dockerfile en `func init`.

```bash
func deploy
```

Están disponibles las siguientes opciones de implementación de contenedor personalizado:

| Opción     | DESCRIPCIÓN                            |
| ------------ | -------------------------------------- |
| **`--registry`** | Nombre de un registro de Docker en el que el usuario actual ha iniciado sesión. |
| **`--platform`** | Plataforma de hospedaje de la aplicación de función. Las opciones válidas son `kubernetes` |
| **`--name`** | Nombre de la aplicación de función. |
| **`--max`**  | Opcionalmente, establece el número máximo de instancias de la aplicación de función que se va a implementar. |
| **`--min`**  | Opcionalmente, establece el número mínimo de instancias de la aplicación de función que se va a implementar. |
| **`--config`** | Establece un archivo de configuración de implementación opcional. |

## <a name="monitoring-functions"></a>Supervisión de funciones

Es la manera recomendada para supervisar la ejecución de las funciones mediante la integración con Azure Application Insights. De forma predeterminada, cuando crea una aplicación de función en Azure Portal, esta integración se realiza automáticamente. Sin embargo, al crear la aplicación de función mediante la CLI de Azure, no se lleva a cabo su integración en Azure.

Para habilitar Application Insights para la aplicación de función:

[!INCLUDE [functions-connect-new-app-insights.md](../../includes/functions-connect-new-app-insights.md)]

Para más información, consulte [Supervisión de Azure Functions](functions-monitoring.md).
## <a name="next-steps"></a>Pasos siguientes

Azure Functions Core Tools es [código abierto que se hospeda en GitHub](https://github.com/azure/azure-functions-cli).  
Para notificar un error o realizar una solicitud de característica, [abra un problema de GitHub](https://github.com/azure/azure-functions-cli/issues).

<!-- LINKS -->

[Azure Functions Core Tools]: https://www.npmjs.com/package/azure-functions-core-tools
[Azure Portal]: https://portal.azure.com 
[Node.js]: https://docs.npmjs.com/getting-started/installing-node#osx-or-windows
[`FUNCTIONS_WORKER_RUNTIME`]: functions-app-settings.md#functions_worker_runtime
[`AzureWebJobsStorage`]: functions-app-settings.md#azurewebjobsstorage
