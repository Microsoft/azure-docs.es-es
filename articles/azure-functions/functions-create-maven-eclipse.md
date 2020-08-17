---
title: Creación de una aplicación de funciones de Azure con Java y Eclipse
description: Guía de procedimientos para crear y publicar una aplicación sencilla sin servidor desencadenada por HTTP con Java y Eclipse en Azure Functions.
author: jeffhollan
ms.topic: how-to
ms.date: 07/01/2018
ms.author: jehollan
ms.custom: mvc, devcenter, devx-track-java
ms.openlocfilehash: a4e240837dd3c95efc4dc83a7491f2ad9163435a
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87373324"
---
# <a name="create-your-first-function-with-java-and-eclipse"></a>Creación de la primera función con Java y Eclipse 

En este artículo se muestra cómo crear un proyecto de función [sin servidor](https://azure.microsoft.com/solutions/serverless/) con el IDE de Eclipse y Apache Maven, probarlo y depurarlo, e implementarlo en Azure Functions. 

<!-- TODO ![Access a Hello World function from the command line with cURL](media/functions-create-java-maven/hello-azure.png) -->

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="set-up-your-development-environment"></a>Configurado su entorno de desarrollo

Para desarrollar una aplicación de función con Java y Eclipse, debe tener instalado lo siguiente:

-  [Kit para desarrolladores de Java](https://www.azul.com/downloads/zulu/), versión 8.
-  [Apache Maven](https://maven.apache.org), versión 3.0 o posterior.
-  [Eclipse](https://www.eclipse.org/downloads/packages/), con compatibilidad con Java y Maven.
-  [CLI de Azure](/cli/azure)

> [!IMPORTANT] 
> La variable de entorno JAVA_HOME se debe establecer en la ubicación de instalación del JDK para completar esta guía de inicio rápido.

Se recomienda encarecidamente instalar también [Azure Functions Core Tools, versión 2](functions-run-local.md#v2), que proporciona un entorno de desarrollo local para ejecutar y depurar Azure Functions. 

## <a name="create-a-functions-project"></a>Creación de un proyecto de Functions

1. En Eclipse, seleccione el menú **File** (Archivo) y, luego, seleccione **New -&gt; Maven Project** (Nuevo - Proyecto Maven). 
1. Acepte los valores predeterminados en el cuadro de diálogo **New Maven Project** (Nuevo proyecto de Maven) y seleccione **Next** (Siguiente).
1. Busque y seleccione [azure-functions-archetype](https://mvnrepository.com/artifact/com.microsoft.azure/azure-functions-archetype) y haga clic en **Next** (Siguiente).
1. Asegúrese de rellenar los valores de todos los campos, incluidos `resourceGroup`, `appName`y `appRegion` (use un valor de appName distinto de **fabrikam-function-20170920120101928**) y, por último, seleccione **Finalizar**.
    ![Creación de Maven en Eclipse 2](media/functions-create-first-java-eclipse/functions-create-eclipse2.png)  

Maven crea los archivos del proyecto en una carpeta nueva con el nombre _artifactId_. El código generado en el proyecto es una función [desencadenada por HTTP](./functions-bindings-http-webhook.md) simple que devuelve el cuerpo de la solicitud HTTP desencadenadora.

## <a name="run-functions-locally-in-the-ide"></a>Ejecución local de funciones en el IDE

> [!NOTE]
> [Azure Functions Core Tools, versión 2](functions-run-local.md#v2), debe estar instalado para ejecutar y depurar localmente las funciones.

1. Haga clic con el botón derecho en el proyecto generado y, luego, elija **Ejecutar como** y **Maven build** (compilación de Maven).
1. En el cuadro de diálogo **Edit Configuration** (Editar configuración), escriba `package` en los campos **Goals** (Objetivos) y **Name** (Nombres); luego, seleccione **Run** (Ejecutar). Con esto, se compila y empaqueta el código de función.
1. Una vez completada la compilación, cree otra configuración de ejecución como las anteriores utilizando `azure-functions:run` como el objetivo y el nombre. Seleccione **Run** (Ejecutar) para ejecutar la función en el IDE.

Finalice el tiempo de ejecución en la ventana de consola cuando haya terminado de probar la función. Solo puede haber un host de función activo y en ejecución en el entorno local a la vez.

### <a name="debug-the-function-in-eclipse"></a>Depuración de la función en Eclipse

En la configuración de **Run As** (Ejecutar como) establecida en el paso anterior, cambie `azure-functions:run` a `azure-functions:run -DenableDebug` y ejecute la configuración actualizada para iniciar la aplicación de función en el modo de depuración.

Seleccione el menú **Run** (Ejecutar) y abra **Debug Configurations** (Configuraciones de depuración). Elija **Remote Java Application** (Aplicación de Java remota) y cree una. Asigne un nombre a la configuración y complete la configuración. El puerto debe ser el mismo que el del puerto de depuración que abrió el host de función, cuyo valor predeterminado es `5005`. Después de la instalación, haga clic en `Debug` para iniciar la depuración.

![Depuración de funciones en Eclipse](media/functions-create-first-java-eclipse/debug-configuration-eclipse.PNG)

Establezca puntos de interrupción e inspeccione los objetos de la función mediante el IDE. Cuando termine, detenga el depurador y el host de la función en ejecución. Solo puede haber un host de función activo y en ejecución en el entorno local a la vez.

## <a name="deploy-the-function-to-azure"></a>Implementación de la función en Azure

El proceso de implementación en Azure Functions usa las credenciales de cuenta desde la CLI de Azure. [Inicie sesión con la CLI de Azure](/cli/azure/authenticate-azure-cli?view=azure-cli-latest) antes de seguir usando el símbolo del sistema de su equipo.

```azurecli
az login
```

Implemente el código en una nueva aplicación de función con el objetivo de Maven `azure-functions:deploy` en una nueva configuración de **Run As** (Ejecutar como).

Cuando se complete la implementación, verá la dirección URL que puede usar para acceder a la aplicación de función de Azure:

```output
[INFO] Successfully deployed Function App with package.
[INFO] Deleting deployment package from Azure Storage...
[INFO] Successfully deleted deployment package fabrikam-function-20170920120101928.20170920143621915.zip
[INFO] Successfully deployed Function App at https://fabrikam-function-20170920120101928.azurewebsites.net
[INFO] ------------------------------------------------------------------------
```

## <a name="next-steps"></a>Pasos siguientes

- Revise la [guía para desarrolladores de Java Functions](functions-reference-java.md) para más información sobre cómo desarrollar funciones de Java.
- Agregue funciones adicionales con desencadenadores distintos en el proyecto con el destino de Maven `azure-functions:add`.
