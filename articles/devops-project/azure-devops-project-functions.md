---
title: 'Tutorial: Implementación de aplicaciones de ASP.NET en Azure Functions con Azure DevOps Starter'
description: Azure DevOps Starter facilita la introducción a Azure. Con DevOps Starter puede implementar una aplicación de ASP.NET en Azure Functions en pocos pasos.
ms.author: mlearned
ms.manager: gwallace
ms.prod: devops
ms.technology: devops-cicd
ms.topic: tutorial
ms.date: 03/24/2020
author: mlearned
ms.custom: devx-track-csharp
ms.openlocfilehash: 79e2726e26ff7883d5022e57fff8fa8961af2c98
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88163686"
---
# <a name="deploy-to-azure-functions-with-devops-starter"></a>Implementación en Azure Functions con DevOps Starter

Azure DevOps Starter ofrece una experiencia simplificada en la que puede utilizar su código existente y el repositorio de Git, o elegir una aplicación de ejemplo para crear una canalización de integración continua (CI) y entrega continua (CD) en Azure.

DevOps Starter también:

* Crea automáticamente recursos de Azure, como Azure Functions.

* Crea y configura una canalización de versión en Azure DevOps para CI/CD.

En este tutorial, aprenderá lo siguiente:

> [!div class="checklist"]
>* A usar DevOps Starter para implementar una aplicación de ASP.NET en Azure Functions
>* Configuración de Azure DevOps y una suscripción de Azure
>* Examinar la instancia de Azure Functions
>* Examen de la canalización de CI
>* Examen de la canalización de CD
>* Confirmación de los cambios en Git y su implementación automática en Azure
>* Limpieza de recursos

Actualmente, los entornos de ejecución compatibles con Functions son **.NET** y **Node.js**. En este tutorial, usaremos .NET para la implementación en Azure Functions. 

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Puede obtener una de forma gratuita mediante [Visual Studio Dev Essentials](https://visualstudio.microsoft.com/dev-essentials/)

## <a name="use-devops-starter-to-deploy-an-aspnet-app-to-azure-functions"></a>Uso de DevOps Starter para implementar una aplicación de ASP.NET en Azure Functions

DevOps Starter crea una canalización de CI/CD en Azure Pipelines. Puede crear una organización de Azure DevOps nueva o usar una existente. DevOps Projects también crea recursos de Azure, como una instancia de IoT Hub, en la suscripción de Azure que prefiera.

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com)

1. En el cuadro de búsqueda, escriba **DevOps Starter** y, a continuación, seleccione. Haga clic en **Agregar** para crear un recurso.

    ![Panel de DevOps Starter](_img/azure-devops-starter-aks/search-devops-starter.png)

1. Seleccione **.NET** y después **Siguiente**. En **Elegir un marco de trabajo de la aplicación**, seleccione **ASP.NET** y haga clic en **Siguiente**.

1. Seleccione **Function App** y, a continuación, **Siguiente**.

## <a name="configure-azure-devops-and-azure-subscription"></a>Configuración de Azure DevOps y una suscripción de Azure

1. Escriba el nombre del proyecto de Azure DevOps.

1. Cree una organización de Azure DevOps nueva o seleccione una existente.

1. Seleccione su suscripción a Azure.

1. Para ver más opciones de configuración de Azure e identificar el plan de tarifa y la ubicación, haga clic en **Configuración adicional**. Este panel muestra varias opciones para configurar el plan de tarifa y la ubicación de los servicios de Azure.

1. Salga del área de configuración de Azure y seleccione **Listo**.

1. Pocos minutos después, el proceso se ha completado. Una aplicación ASP.NET Core de ejemplo se configura en un repositorio de Git de su organización de Azure DevOps, se crea una instancia de Function App y de Application Insights, se ejecuta una canalización de CI/CD y la aplicación se implementa en Azure.

   Cuando todo esto finaliza, se muestra el panel de Azure DevOps Starter en Azure Portal. También puede ir al panel de DevOps Starter directamente desde la opción **Todos los recursos** de Azure Portal.

   Este panel proporciona visibilidad sobre el repositorio de código de Azure DevOps, la canalización de CI/CD y la instancia de Azure Functions. En la canalización de Azure DevOps se pueden configurar otras opciones de CI/CD. A la derecha, seleccione **Aplicación de funciones** para que se vea.

## <a name="examine-the-function-app"></a>Examen de la aplicación de funciones

DevOps Starter configura automáticamente una aplicación de función que puede explorar y personalizar. Para familiarizarse con la aplicación de funciones, haga lo siguiente:

1. Vaya al panel de inicio de DevOps Starter.

    ![Panel de DevOps Projects](_img/azure-devops-project-functions/fapp-dashboard.png)

1. A la derecha, seleccione la aplicación de funciones. Se abre un panel para la aplicación de funciones. En esta vista puede realizar varias acciones, como la supervisión de operaciones y la búsqueda de registros.

    ![Function App](_img/azure-devops-project-functions/function-app.png)

## <a name="examine-the-ci-pipeline"></a>Examen de la canalización de CI

DevOps Starter configura automáticamente una canalización de CI/CD de Azure completa en su organización de Azure DevOps. Puede explorar y personalizar la canalización. Para familiarizarse con ella, siga estos pasos:

1. Vaya al panel de inicio de DevOps Starter.

1. Haga clic en el hipervínculo en **Compilar**. Una pestaña del explorador muestra la canalización de compilación del nuevo proyecto.

    ![Build](_img/azure-devops-project-functions/build.png)

1. Seleccione **Editar**. En este panel puede examinar las distintas tareas de la canalización de compilación. La compilación ejecuta varias tareas, como capturar códigos fuente del repositorio de Git, compilar la aplicación, ejecutar pruebas unitarias y publicar salidas que se usan para implementaciones.

1. Seleccione **Desencadenadores**. DevOps Starter crea automáticamente un desencadenador de integración continua y cada confirmación al repositorio inicia una compilación. Opcionalmente, puede elegir incluir o excluir ramas del proceso de integración continua.

1. Seleccione **Retención**. En función del escenario, puede especificar directivas para conservar o quitar un determinado número de compilaciones.

1. En la parte superior de la canalización de compilación, seleccione el nombre de esta.

1. Cambie el nombre de la canalización de compilación por otro más descriptivo y seleccione **Guardar** en el menú desplegable **Guardar y poner en cola**.

1. En el nombre de la canalización de compilación, seleccione **Historial**. Este panel muestra un registro de auditoría de los cambios recientes de la compilación. Azure DevOps realiza un seguimiento de los cambios realizados en la canalización de compilación y permite comparar las versiones.

## <a name="examine-the-cd-release-pipeline"></a>Examen de la canalización de versión de la implementación continua

DevOps Starter crea y configura automáticamente los pasos necesarios para implementar desde la organización de Azure DevOps en la suscripción de Azure. Dichos pasos incluyen la configuración de una conexión de servicio de Azure para realizar la autenticación de Azure DevOps en la suscripción de Azure. La automatización también crea una canalización de versión, lo que proporciona la implementación continua en Azure. Para más información acerca de la canalización de versión, siga estos pasos:

1. Vaya a **Canalizaciones | Versiones**.

1. Haga clic en **Editar**.

1. En **Artefactos**, seleccione **Colocar**. La canalización de compilación que ha examinado en los pasos anteriores genera la salida que se usa para el artefacto.

1. A la derecha del icono **Colocar**, seleccione **Desencadenador de implementación continua**. Esta canalización de versión tiene un desencadenador de implementación continua habilitado, que ejecuta una implementación cada vez que hay un nuevo artefacto de compilación disponible. Opcionalmente, puede deshabilitar el desencadenador, con lo que las implementaciones van a requerir una ejecución manual.

1. A la derecha, seleccione **Ver versiones** para mostrar un historial de las versiones.

1. Haga clic en la versión que mostrará la canalización. Haga clic en cualquier entorno para comprobar los **Elementos de trabajo** asociados al **resumen y confirmaciones** de la versión.

1. Seleccione **Confirmaciones**. Esta vista muestra las confirmaciones de código que están asociadas a esta implementación. Compare las versiones para ver las diferencias de confirmación entre las implementaciones.

1. Seleccione **Ver registros**. Los registros contienen información útil sobre el proceso de implementación. Se pueden ver tanto durante las implementaciones como después de ellas.

## <a name="commit-code-changes-and-execute-cicd"></a>Confirmación de los cambios de código y ejecución de CI/CD

> [!NOTE]
> El siguiente procedimiento prueba la canalización de CI/CD. Para ello, realiza un cambio de texto simple.

A partir de ese momento ya puede empezar a colaborar con un equipo en una aplicación mediante el uso de un proceso de CI/CD que implemente automáticamente el trabajo más reciente en Azure Functions. Cada cambio que se realiza en el repositorio de Git inicia una compilación en Azure DevOps y una canalización de CD ejecuta una implementación en Azure. Siga el procedimiento descrito en esta sección o utilice otra técnica para confirmar los cambios en el repositorio. Por ejemplo, puede clonar el repositorio de Git en su herramienta favorita o IDE, y luego insertar los cambios en este repositorio.

1. En el menú de Azure DevOps, seleccione **Repositorio | Archivos** y, a continuación, vaya al repositorio.

1. El repositorio ya incluye código denominado **SampleFunctionApp** según el idioma de la aplicación que eligió en el proceso de creación. Abra el archivo **Application/SampleFunctionApp/Function1.cs**.

1. Seleccione **Editar** y, a continuación, haga un cambio en el **número de línea 31**. Por ejemplo, puede actualizarlo a **Hola. Bienvenido a Azure Functions con DevOps Starter**

1. En la parte superior derecha, seleccione **Confirmar** y, después, seleccione **Confirmar** de nuevo para insertar el cambio.

1. Abra el archivo **Application/SampleFunctionApp.Test/Function1TestRunner.cs**. 

1. Seleccione **Editar** y, a continuación, haga un cambio en el **número de línea 21**. Por ejemplo, puede actualizarlo a **Hola. Bienvenido a Azure Functions con Azure DevOps Starter**.

     Tras unos instantes, se inicia una compilación se inicia en Azure DevOps y se ejecuta una versión para implementar los cambios. Supervise el estado de la compilación en el panel de DevOps Starter o en el explorador con la organización de Azure DevOps.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando no necesite los recursos relacionados que haya creado puede eliminarlos. Use la funcionalidad **Eliminación** del panel de DevOps Starter.

## <a name="next-steps"></a>Pasos siguientes

Si lo desea, puede modificar estas canalizaciones de compilación y de versión para satisfacer las necesidades de su equipo. También puede usar este patrón de CI/CD como plantilla para las demás canalizaciones. En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * A usar DevOps Starter para implementar una aplicación de ASP.NET Core en Azure Functions
> * Configuración de Azure DevOps y una suscripción de Azure 
> * Examinar la instancia de Azure Functions
> * Examen de la canalización de CI
> * Examen de la canalización de CD
> * Confirmación de los cambios en Git y su implementación automática en Azure
> * Limpieza de recursos

