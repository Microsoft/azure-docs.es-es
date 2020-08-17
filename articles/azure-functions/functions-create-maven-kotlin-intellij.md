---
title: Creación de una función de Azure con Kotlin e IntelliJ
description: Aprenda a crear y publicar una aplicación sencilla sin servidor desencadenada por HTTP en Azure con Kotlin e IntelliJ.
author: dglover
ms.service: azure-functions
ms.topic: quickstart
ms.date: 03/25/2020
ms.author: dglover
ms.openlocfilehash: fa834e3ac4946c4f617e857342f850445eebfd30
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87055453"
---
# <a name="quickstart-create-your-first-http-triggered-function-with-kotlin-and-intellij"></a>Inicio rápido: Creación de la primera función desencadenada por HTTP con Kotlin e IntelliJ

En este artículo se muestra cómo crear un proyecto de función [sin servidor](https://azure.microsoft.com/overview/serverless-computing/) con IntelliJ IDEA y Apache Maven. También se muestra cómo depurar localmente el código de función en el entorno de desarrollo integrado (IDE) y, luego, implementar el proyecto de función en Azure.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="set-up-your-development-environment"></a>Configurado su entorno de desarrollo

Para desarrollar una función con Kotlin e IntelliJ, instale el software siguiente:

- [Kit de desarrolladores de Java](https://aka.ms/azure-jdks) (JDK), versión 8
- [Apache Maven](https://maven.apache.org), versión 3.0 o posterior
- [IntelliJ IDEA](https://www.jetbrains.com/idea/download), versión Community o Ultimate con Maven
- [CLI de Azure](/cli/azure)
- [Versión 2.x](functions-run-local.md#v2) de Azure Functions Core Tools. que proporciona un entorno de desarrollo local para escribir, ejecutar y depurar Azure Functions.

> [!IMPORTANT]
> La variable de entorno JAVA_HOME se debe establecer en la ubicación de instalación del JDK para completar los pasos de este artículo.

## <a name="create-a-functions-project"></a>Creación de un proyecto de Functions

1. En IntelliJ IDEA, seleccione **Create New Project** (Crear nuevo proyecto).  
1. En la ventana **New Project** (Nuevo proyecto), seleccione **Maven** en el panel izquierdo.
1. Active la casilla **Create from archetype** (Crear desde arquetipo) y, luego, seleccione **Add Archetype** (Agregar arquetipo) para [azure-functions-kotlin-archetype](https://mvnrepository.com/artifact/com.microsoft.azure/azure-functions-kotlin-archetype).
1. En la ventana **Add Archetype** (Agregar arquetipo), complete los campos como se indica a continuación:
    - _GroupId_: com.microsoft.azure
    - _ArtifactId_: azure-functions-kotlin-archetype
    - _Versión_: use la versión más reciente del [repositorio central](https://mvnrepository.com/artifact/com.microsoft.azure/azure-functions-kotlin-archetype)
    ![Creación de un proyecto de Maven a partir de arquetipo en IntelliJ IDEA](media/functions-create-first-kotlin-intellij/functions-create-intellij.png).  
1. Seleccione **Aceptar** y después **Siguiente**.
1. Especifique los detalles del proyecto actual y seleccione **Finish** (Finalizar).

Maven crea los archivos del proyecto en una carpeta nueva con el mismo nombre que el valor _ArtifactId_. El código generado en el proyecto es una función [desencadenada por HTTP](./functions-bindings-http-webhook.md) simple que devuelve el cuerpo de la solicitud HTTP desencadenadora.

## <a name="run-functions-locally-in-the-ide"></a>Ejecución local de funciones en el IDE

> [!NOTE]
> Para ejecutar y depurar localmente las funciones, asegúrese de que [Azure Functions Core Tools, versión 2](functions-run-local.md#v2) está instalado.

1. Importe los cambios manualmente o habilite la [importación automática](https://www.jetbrains.com/help/idea/creating-and-optimizing-imports.html).
1. Abra la barra de herramientas **Maven Projects** (Proyectos de Maven).
1. Expanda **Lifecycle** (Ciclo de vida) y, a continuación, abra **package** (paquete). La solución se compila y empaqueta en un directorio de destino recién creado.
1. Expanda **Plugins** > **azure-functions** (Complementos > azure-functions) y abra **azure-functions:run** para iniciar el entorno de ejecución local de Azure Functions.  
  ![Barra de herramientas de Maven para Azure Functions](media/functions-create-first-kotlin-intellij/functions-intellij-kotlin-maven-toolbar.png)  

1. Cierre el cuadro de diálogo de ejecución cuando termine de probar la función. Solo puede haber un host de función activo y en ejecución en el entorno local a la vez.

## <a name="debug-the-function-in-intellij"></a>Depuración de la función en IntelliJ

1. Para iniciar el host de función en modo de depuración, agregue **-DenableDebug** como argumento al ejecutar la función. Puede cambiar la configuración en [maven goals](https://www.jetbrains.com/help/idea/maven-support.html#run_goal) (objetivos de maven) o ejecutar el siguiente comando en una ventana de terminal:  

   ```
   mvn azure-functions:run -DenableDebug
   ```

   Este comando hace que el host de función abra un puerto de depuración en 5005.

1. En el menú **Run** (Ejecutar), seleccione **Edit Configurations** (Editar configuraciones).
1. Seleccione **(+)** para agregar un elemento **Remote** (Remoto).
1. Complete los campos _Name_ (Nombre) y _Settings_ (Configuración) y, a continuación, seleccione **OK** para guardar la configuración.
1. Después de la configuración, seleccione **Debug < Remote Configuration Name >** (Depurar <Nombre de configuración remota>) o presione MAYÚS + F9 en el teclado para iniciar la depuración.

   ![Depuración de funciones en IntelliJ](media/functions-create-first-kotlin-intellij/debug-configuration-intellij.PNG)

1. Cuando termine, detenga el depurador y el proceso en ejecución. Solo puede haber un host de función activo y en ejecución en el entorno local a la vez.

## <a name="deploy-the-function-to-azure"></a>Implementación de la función en Azure

1. Antes de poder implementar su función en Azure, debe [iniciar sesión mediante la CLI de Azure](/cli/azure/authenticate-azure-cli?view=azure-cli-latest).

   ``` azurecli
   az login
   ```

1. Implemente el código en una función nueva con el destino de Maven `azure-functions:deploy`. También puede seleccionar la opción **azure-functions:deploy** en la ventana de proyectos de Maven.

   ```
   mvn azure-functions:deploy
   ```

1. Busque la dirección URL de la función en la salida de la CLI de Azure después de que la función se haya implementado correctamente.

   ``` output
   [INFO] Successfully deployed Function App with package.
   [INFO] Deleting deployment package from Azure Storage...
   [INFO] Successfully deleted deployment package fabrikam-function-20170920120101928.20170920143621915.zip
   [INFO] Successfully deployed Function App at https://fabrikam-function-20170920120101928.azurewebsites.net
   [INFO] ------------------------------------------------------------------------
   ```

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha implementado la primera función de Kotlin en Azure, revise la [guía para desarrolladores de Functions en Java](functions-reference-java.md) para más información sobre el desarrollo de funciones de Java y Kotlin.
- Agregue funciones adicionales con desencadenadores distintos en el proyecto con el destino de Maven `azure-functions:add`.
