---
title: Creación de una función de Azure con Java e IntelliJ
description: Aprenda a crear y publicar una aplicación sencilla sin servidor desencadenada por HTTP en Azure con Java e IntelliJ.
author: jeffhollan
ms.topic: how-to
ms.date: 07/01/2018
ms.author: jehollan
ms.custom: mvc, devcenter, devx-track-java
ms.openlocfilehash: 38ccc99de72af8f989cca0fefae2f962fb178f0e
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87373375"
---
# <a name="create-your-first-azure-function-with-java-and-intellij"></a>Creación de la primera función de Azure con Java e IntelliJ

Este artículo le muestra:
- Cómo crear un proyecto de función [sin servidor](https://azure.microsoft.com/overview/serverless-computing/) con IntelliJ IDEA
- Pasos para probar y depurar la función en el entorno de desarrollo integrado (IDE) en su propio equipo
- Instrucciones para implementar el proyecto de función en Azure Functions

<!-- TODO ![Access a Hello World function from the command line with cURL](media/functions-create-java-maven/hello-azure.png) -->

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="set-up-your-development-environment"></a>Configurado su entorno de desarrollo

Para desarrollar una función con Java e IntelliJ, instale el software siguiente:

+ Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
+ Un [kit de desarrollo de Java (JDK) compatible con Azure](https://aka.ms/azure-jdks) para Java 8
+ [IntelliJ IDEA](https://www.jetbrains.com/idea/download/) Ultimate Edition o Community Edition instalado
+ [Maven 3.5.0+](https://maven.apache.org/download.cgi)
+ La versión más reciente de [Azure Functions Core Tools](https://github.com/Azure/azure-functions-core-tools)


## <a name="installation-and-sign-in"></a>Instalación e inicio de sesión

1. En el cuadro de diálogo de configuración y preferencias del IntelliJ IDEA (Ctrl+Alt+S), seleccione **Complementos**. A continuación, busque **Azure Toolkit for IntelliJ** en **Marketplace** y haga clic en **Instalar**. Una vez instalado, haga clic en **Reiniciar** para activar el complemento. 

    ![Complemento Azure Toolkit for IntelliJ en Marketplace][marketplace]

2. Para iniciar sesión en la cuenta de Azure, abra **Azure Explorer** y, a continuación, haga clic en el icono **Inicio de sesión de Azure** en la barra de la parte superior (o en el menú IDEA **Herramientas/Azure/Inicio de sesión de Azure**).
    ![El comando de inicio de sesión en Azure IntelliJ][intellij-azure-login]

3. En la ventana **Inicio de sesión de Azure**, seleccione **Inicio de sesión de dispositivo** y, a continuación, haga clic en **Iniciar sesión** ([otras opciones de inicio de sesión](/azure/developer/java/toolkit-for-intellij/sign-in-instructions)).

   ![Ventana Inicio de sesión de Azure con el inicio de sesión de dispositivo seleccionado][intellij-azure-popup]

4. Haga clic en **Copiar y abrir** en el cuadro de diálogo **Inicio de sesión de dispositivo de Azure**.

   ![El cuadro de diálogo Inicio de sesión en Microsoft Azure][intellij-azure-copycode]

5. En el explorador, pegue el código del dispositivo (que se ha copiado al hacer clic en **Copiar y abrir** en el último paso) y, a continuación, haga clic en **Siguiente**.

   ![Explorador de inicio de sesión de dispositivo][intellij-azure-link-ms-account]

6. En el cuadro de diálogo **Seleccionar suscripciones**, elija las suscripciones que desea usar y, luego, haga clic en **Aceptar**.

   ![Cuadro de diálogo Seleccionar suscripciones][intellij-azure-login-select-subs]
   
## <a name="create-your-local-project"></a>Creación del proyecto local

En esta sección se usa Azure Toolkit for IntelliJ para crear un proyecto local de Azure Functions. Más adelante en este artículo, publicará el código de función en Azure. 

1. Abra el cuadro de diálogo de bienvenida de IntelliJ, seleccione *Create New Project* (Crear nuevo proyecto) para abrir el asistente para un nuevo proyecto y seleccione *Azure Functions*.

    ![Creación de un proyecto de Functions](media/functions-create-first-java-intellij/create-functions-project.png)

1. Seleccione *Http Trigger* (Desencadenador HTTP), haga clic en *Next* (Siguiente) y siga el asistente para recorrer todas las configuraciones de las páginas siguientes, confirme la ubicación del proyecto y haga clic en *Finish* (Finalizar); Intellj IDEA abrirá el nuevo proyecto.

    ![Finalización de la creación de un proyecto de Functions](media/functions-create-first-java-intellij/create-functions-project-finish.png)

## <a name="run-the-function-app-locally"></a>Ejecución de la aplicación de funciones de forma local

1. Vaya a `src/main/java/org/example/functions/HttpTriggerFunction.java` para ver el código generado. Al lado de la línea *17*, observará que hay un botón verde *Run* (Ejecutar); haga clic en él y seleccione *Run "azure-function-exam..."* (Ejecutar "azure-function-exam..."). Verá que la aplicación de función se ejecuta localmente y tiene algunos registros.

    ![Ejecución local del proyecto de Functions](media/functions-create-first-java-intellij/local-run-functions-project.png)

    ![Salida de la ejecución local del proyecto de Functions](media/functions-create-first-java-intellij/local-run-functions-output.png)

1. Puede probar la función mediante el acceso desde el explorador al punto de conexión que se imprime; por ejemplo, `http://localhost:7071/api/HttpTrigger-Java?name=Azure`.

    ![Resultado de prueba de la ejecución local de la función](media/functions-create-first-java-intellij/local-run-functions-test.png)

1. El registro también se imprime en IDEA; ahora, para detener la función, haga clic en el botón *Stop* (Detener).

    ![Registro de la prueba de la ejecución local de la función](media/functions-create-first-java-intellij/local-run-functions-log.png)

## <a name="debug-the-function-app-locally"></a>Depuración de la aplicación de funciones de forma local

1. Vamos a intentar depurar la aplicación de funciones de forma local; para ello, haga clic en el botón *Debug* (Depurar) de la barra de herramientas (si no lo ve, haga clic en *View -> Appearance -> Toolbar* [Vista -> Apariencia -> Barra de herramientas] para habilitar la barra de herramientas).

    ![Botón de depuración de funciones de forma local](media/functions-create-first-java-intellij/local-debug-functions-button.png)

1. Haga clic en la línea *20* del archivo `src/main/java/org/example/functions/HttpTriggerFunction.java` para agregar un punto de interrupción; acceda de nuevo al punto de conexión `http://localhost:7071/api/HttpTrigger-Java?name=Azure` y verá que se alcanza el punto de interrupción; puede probar más características de depuración como *step* (paso), *watch* (inspección), *evaluation* (evaluación). Para detener la sesión de depuración, haga clic en el botón Stop (Detener).

    ![Punto de interrupción de la depuración de funciones de forma local](media/functions-create-first-java-intellij/local-debug-functions-break.png)

## <a name="deploy-your-function-app-to-azure"></a>Implementación de la aplicación de funciones en Azure

1. Haga clic con el botón derecho en el proyecto en el explorador de proyectos de IntelliJ y seleccione *Azure -> Deploy to Azure Functions* (Azure -> Implementar en Azure Functions).

    ![Implementación de funciones en Azure](media/functions-create-first-java-intellij/deploy-functions-to-azure.png)

1. Si aún no tiene ninguna aplicación de funciones, haga clic en *No available function, click to create a new one* (No hay ninguna función disponible; haga clic para crear una nueva).

    ![Crear aplicación en la implementación de funciones en Azure](media/functions-create-first-java-intellij/deploy-functions-create-app.png)

1. Escriba el nombre de la aplicación de funciones y elija la suscripción, la plataforma, el grupo de recursos y el plan de App Service adecuados. También puede crear un grupo de recursos y un plan de App Service aquí. A continuación, mantenga la configuración de la aplicación sin cambios, haga clic en *OK* (Aceptar) y espere unos minutos hasta que se cree la nueva función. Después de mostrar *Creating New Function App...* (Creando nueva aplicación de funciones...), la barra de progreso desaparece.

    ![Asistente para crear aplicación en la implementación de funciones en Azure](media/functions-create-first-java-intellij/deploy-functions-create-app-wizard.png)

1. Seleccione la aplicación de funciones en la que quiere realizar la implementación (la nueva aplicación de funciones que acaba de crear se seleccionará automáticamente). Haga clic en *Run* (Ejecutar) para implementar las funciones.

    ![Botón Ejecutar en la implementación de funciones en Azure](media/functions-create-first-java-intellij/deploy-functions-run.png)

    ![Registro de la implementación de funciones en Azure](media/functions-create-first-java-intellij/deploy-functions-log.png)

## <a name="manage-azure-functions-from-idea"></a>Administración de Azure Functions desde IDEA

1. Puede administrar sus funciones con *Azure Explorer* en el entorno de IDEA; haga clic en *Function App* (Aplicación de funciones) para ver todas las funciones.

    ![Visualización de las funciones en el explorador](media/functions-create-first-java-intellij/explorer-view-functions.png)

1. Haga clic para seleccionar una de las funciones, haga clic con el botón derecho y seleccione *Show Properties* (Mostrar propiedades) para abrir la página de detalles. 

    ![Mostrar propiedades de las funciones](media/functions-create-first-java-intellij/explorer-functions-show-properties.png)

1. Haga clic con el botón derecho en el elemento *HttpTrigger-Java* y seleccione *Trigger Function* (Desencadenar función); verá que el explorador se abre con la dirección URL del desencadenador.

    ![Ejecución de la implementación de funciones en Azure](media/functions-create-first-java-intellij/explorer-trigger-functions.png)

## <a name="add-more-functions-to-the-project"></a>Adición de más funciones al proyecto

1. Haga clic con el botón derecho en el paquete *org.example.functions* y seleccione *New -> Azure Function Class* (Nuevo > Clase de función de Azure). 

    ![Entrada de la adición de funciones al proyecto](media/functions-create-first-java-intellij/add-functions-entry.png)

1. Escriba *HttpTest* en el nombre de la clase y seleccione *HttpTrigger* en el asistente para crear clases de función y haga clic en *OK* (Aceptar) para crearla. De esta manera, puede crear tantas nuevas funciones como desee.

    ![Adición de funciones al desencadenador seleccionado del proyecto](media/functions-create-first-java-intellij/add-functions-trigger.png)
    
    ![Salida de la adición de funciones al proyecto](media/functions-create-first-java-intellij/add-functions-output.png)

## <a name="cleaning-up-functions"></a>Limpieza de funciones

1. Eliminación de funciones de Azure Functions en Azure Explorer
      
      ![Adición de funciones al desencadenador seleccionado del proyecto](media/functions-create-first-java-intellij/delete-function.png)
      

## <a name="next-steps"></a>Pasos siguientes

Ha creado un proyecto de funciones de Java con una función desencadenada por HTTP, lo ha ejecutado en la máquina local y lo ha implementado en Azure. Ahora, amplíe la función como sigue...

> [!div class="nextstepaction"]
> [Adición de un enlace de salida de la cola de Azure Storage](./functions-add-output-binding-storage-queue-java.md)


[marketplace]:./media/functions-create-first-java-intellij/marketplace.png
[intellij-azure-login]: media/functions-create-first-java-intellij/intellij-azure-login.png
[intellij-azure-popup]: media/functions-create-first-java-intellij/intellij-azure-login-popup.png
[intellij-azure-copycode]: media/functions-create-first-java-intellij/intellij-azure-login-copyopen.png
[intellij-azure-link-ms-account]: media/functions-create-first-java-intellij/intellij-azure-login-linkms-account.png
[intellij-azure-login-select-subs]: media/functions-create-first-java-intellij/intellij-azure-login-selectsubs.png
