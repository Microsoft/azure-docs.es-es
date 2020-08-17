---
title: Uso de Java y Gradle para publicar una función en Azure
description: Cree y publique una función desencadenada por HTTP en Azure con Java y Gradle.
author: KarlErickson
ms.custom: devx-track-java
ms.author: karler
ms.topic: how-to
ms.date: 04/08/2020
ms.openlocfilehash: ce2169838faaee8da459f80979fa31318e9d50c9
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87322183"
---
# <a name="use-java-and-gradle-to-create-and-publish-a-function-to-azure"></a>Uso de Java y Gradle para crear y publicar una función en Azure

En este artículo se muestra cómo compilar y publicar un proyecto de una función de Java en Azure Functions con la herramienta de línea de comandos de Gradle. Una vez que haya terminado, el código de función se ejecutará en Azure en un [plan de hospedaje sin servidor](functions-scale.md#consumption-plan) y se desencadenará mediante una solicitud HTTP. 

> [!NOTE]
> Si Gradle no es la herramienta de desarrollo preferida, consulte nuestros tutoriales análogos para desarrolladores de Java con [Maven](./functions-create-first-azure-function-azure-cli.md?pivots=programming-language-java), [IntelliJ IDEA](/azure/developer/java/toolkit-for-intellij/quickstart-functions) y [VS Code](./functions-create-first-function-vs-code.md?pivots=programming-language-java).

## <a name="prerequisites"></a>Requisitos previos

Para desarrollar funciones con Java, debe tener instalado lo siguiente:

- [Kit para desarrolladores de Java](https://aka.ms/azure-jdks), versión 8
- [CLI de Azure]
- [Azure Functions Core Tools](./functions-run-local.md#v2), versión 2.6.666 u otra posterior
- [Gradle](https://gradle.org/), versión 4.10 y superior

Necesita también una suscripción de Azure activa. [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

> [!IMPORTANT]
> La variable de entorno JAVA_HOME se debe establecer en la ubicación de instalación del JDK para completar esta guía de inicio rápido.

## <a name="prepare-a-functions-project"></a>Preparación de un proyecto de Functions

Use el comando siguiente para clonar el proyecto de ejemplo:

```bash
git clone https://github.com/Azure-Samples/azure-functions-samples-java.git
cd azure-functions-samples-java/
```

Abra `build.gradle` y cambie el valor de `appName` en la sección siguiente por un nombre único para evitar conflictos de nombres de dominio al realizar la implementación en Azure. 

```gradle
azurefunctions {
    resourceGroup = 'java-functions-group'
    appName = 'azure-functions-sample-demo'
    pricingTier = 'Consumption'
    region = 'westus'
    runtime {
      os = 'windows'
    }
    localDebug = "transport=dt_socket,server=y,suspend=n,address=5005"
}
```

Abra el nuevo archivo Function.java desde la ruta de acceso *src/main/java*  en un editor de texto y revise el código generado. Este código es una función [desencadenada por HTTP](functions-bindings-http-webhook.md) que devuelve el cuerpo de la solicitud. 

> [!div class="nextstepaction"]
> [He tenido un problema](https://www.research.net/r/javae2e?tutorial=functions-create-first-java-gradle&step=generate-project)

## <a name="run-the-function-locally"></a>Ejecución local de la función

Ejecute el siguiente comando para realizar la compilación y, luego, ejecutar el proyecto de función:

```bash
gradle jar --info
gradle azureFunctionsRun
```
Verá una salida similar a la siguiente desde Azure Functions Core Tools cuando ejecute el proyecto localmente:

<pre>
...

Now listening on: http://0.0.0.0:7071
Application started. Press Ctrl+C to shut down.

Http Functions:

    HttpExample: [GET,POST] http://localhost:7071/api/HttpExample
...
</pre>

Desencadene la función desde la línea de comandos mediante el comando cURL siguiente en una ventana de terminal nueva:

```bash
curl -w "\n" http://localhost:7071/api/HttpExample --data AzureFunctions
```

El resultado esperado es el siguiente:

<pre>
Hello, AzureFunctions
</pre>

> [!NOTE]
> Si establece authLevel en `FUNCTION` o `ADMIN`, no se requiere la [clave de función](functions-bindings-http-webhook-trigger.md#authorization-keys) cuando se ejecuta de manera local.  

Use `Ctrl+C` en el terminal para detener el código de función.

> [!div class="nextstepaction"]
> [He tenido un problema](https://www.research.net/r/javae2e?tutorial=functions-create-first-java-gradle&step=local-run)

## <a name="deploy-the-function-to-azure"></a>Implementación de la función en Azure

La primera vez que crea una aplicación de funciones, se crea dicha función y los recursos relacionados en Azure. Antes de poder implementar, use el comando [az login](/cli/azure/authenticate-azure-cli) de la CLI de Azure para iniciar sesión en la suscripción de Azure. 

```azurecli
az login
```

> [!TIP]
> Si su cuenta puede acceder a varias suscripciones, use [az account set](/cli/azure/account#az-account-set) para establecer la suscripción predeterminada de esta sesión. 

Use el siguiente comando para implementar el proyecto en una nueva aplicación de funciones. 

```bash
gradle azureFunctionsDeploy
```

De este modo, se crean los siguientes recursos en Azure, en función de los valores del archivo build.gradle:

+ Un grupo de recursos Se le ha asignado el nombre de _resourceGroup_ que proporcionó anteriormente.
+ Una cuenta de almacenamiento Necesaria con Functions. El nombre se genera aleatoriamente según los requisitos de nombre de la cuenta de almacenamiento.
+ Plan de App Service. El hospedaje del plan de consumo sin servidor de la aplicación de funciones de la región _appRegion_ especificada. El nombre se genera aleatoriamente.
+ Aplicación de funciones. Una aplicación de funciones es la unidad de implementación y ejecución de las funciones. El nombre es _appName_, anexado a un número generado de forma aleatoria. 

La implementación también empaqueta los archivos de proyecto y los implementa en la nueva aplicación de funciones mediante la [implementación de archivos zip](functions-deployment-technologies.md#zip-deploy), con el modo de ejecución desde el paquete habilitado.

La propiedad authLevel para el desencadenador HTTP en el proyecto de ejemplo es `ANONYMOUS`, que omitirá la autenticación. Sin embargo, si usa otra propiedad authLevel como `FUNCTION` o `ADMIN`, debe obtener la clave de función para llamar al punto de conexión de la función a través de HTTP. La forma más fácil de obtener la clave de función es desde [Azure Portal].

> [!div class="nextstepaction"]
> [He tenido un problema](https://www.research.net/r/javae2e?tutorial=functions-create-first-java-gradle&step=deploy)

## <a name="get-the-http-trigger-url"></a>Obtención de la dirección URL del desencadenador HTTP

Puede obtener la dirección URL necesaria para desencadenar la función, con la clave de función, en Azure Portal. 

1. Vaya a [Azure Portal], inicie sesión, escriba el nombre de _appName_ para la aplicación de funciones en **Buscar** en la parte superior de la página y presione Entrar.
 
1. En la aplicación de funciones, seleccione **Funciones**, elija la función y, a continuación, haga clic en **</> Obtener la dirección URL de la función** en la parte superior derecha. 

    :::image type="content" source="./media/functions-create-first-java-gradle/get-function-url-portal.png" alt-text="Copiar la dirección URL de la función desde Azure Portal":::

1. Seleccione **valor predeterminado (clave de función)**  y seleccione **Copiar**. 

Ahora puede usar la dirección URL copiada para acceder a la función.

## <a name="verify-the-function-in-azure"></a>Comprobación de la función en Azure

Para comprobar la aplicación de funciones que se ejecuta en Azure mediante `cURL`, reemplace la dirección URL del ejemplo siguiente por la dirección URL que copió del portal.

```console
curl -w "\n" http://azure-functions-sample-demo.azurewebsites.net/api/HttpExample --data AzureFunctions
```

Esto envía una solicitud POST al punto de conexión de la función con `AzureFunctions` en el cuerpo de la solicitud. Puede ver la siguiente respuesta.

<pre>
Hello, AzureFunctions
</pre>

> [!div class="nextstepaction"]
> [He tenido un problema](https://www.research.net/r/javae2e?tutorial=functions-create-first-java-gradle&step=verify-deployment)

## <a name="next-steps"></a>Pasos siguientes

Ha creado un proyecto de funciones de Java con una función desencadenada por HTTP, lo ha ejecutado en la máquina local y lo ha implementado en Azure. Ahora, amplíe la función como sigue...

> [!div class="nextstepaction"]
> [Adición de un enlace de salida de la cola de Azure Storage](functions-add-output-binding-storage-queue-java.md)


[CLI de Azure]: /cli/azure
[Azure Portal]: https://portal.azure.com
