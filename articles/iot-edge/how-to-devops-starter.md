---
title: 'Canalización de CI/CD con Azure DevOps Starter: Azure IoT Edge | Microsoft Docs'
description: Azure DevOps Starter facilita la introducción a Azure. En pocos y rápidos pasos le ayuda a iniciar la aplicación de Azure IoT Edge que prefiera.
author: shizn
ms.author: xshi
ms.date: 08/25/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 7251e84af001422de2c0e33ca04fdfdb309f7afd
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89301129"
---
# <a name="create-a-cicd-pipeline-for-iot-edge-with-azure-devops-starter"></a>Creación de una canalización de CI/CD para IoT Edge con Azure DevOps Starter

Configure la integración continua (CI) y la integración continua (CD) para la aplicación de IoT Edge con DevOps Projects. DevOps Starter simplifica la configuración inicial de una canalización de compilación y de versión en Azure Pipelines.

Si no tiene una suscripción activa a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free) antes de comenzar.

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

DevOps Starter crea una canalización de CI/CD en Azure DevOps. Puede crear una organización de Azure DevOps nueva o usar una existente. DevOps Starter también crea recursos de Azure en la suscripción a Azure que prefiera.

1. Inicie sesión en [Microsoft Azure Portal](https://portal.azure.com).

1. En el panel izquierdo, seleccione **Crear un recurso** y, después, busque **DevOps Starter**.  

1. Seleccione **Crear**.

## <a name="create-a-new-application-pipeline"></a>Creación de una nueva canalización de aplicaciones

1. Los módulos de Azure IoT Edge se pueden escribir en [C#](tutorial-csharp-module.md), [Node.js](tutorial-node-module.md), [Python](tutorial-python-module.md), [C](tutorial-c-module.md) y [Java](tutorial-java-module.md). Seleccione el lenguaje que prefiera para iniciar una aplicación nueva: **.NET**, **Node.js**, **Python**, **C** o **Java**. Seleccione **Next** (Siguiente) para continuar.

   ![Selección del lenguaje para crear una aplicación](./media/how-to-devops-starter/select-language.png)

2. Seleccione **IoT simple** como marco de la aplicación y haga clic en **Siguiente**.

   ![Selección de la plataforma IoT simple](media/how-to-devops-starter/select-iot.png)

3. Seleccione **IoT Edge** como servicio de Azure que implementa la aplicación y, a continuación, seleccione **Siguiente**.

   ![Selección del servicio IoT Edge](media/how-to-devops-starter/select-iot-edge.png)

4. Cree una organización de Azure DevOps gratuita o elija una existente.

   1. Proporcione un nombre para el proyecto.

   2. Seleccione la organización de Azure DevOps. Si no tiene una organización existente, seleccione **Configuración adicional** para crear una nueva.

   3. Seleccione su suscripción a Azure.

   4. Use el nombre de IoT Hub generado por el nombre del proyecto o proporcione uno.

   5. Acepte la ubicación predeterminada o elija una que se encuentre próxima.

   6. Seleccione **Configuración adicional** para configurar los recursos de Azure que DevOps Starter crea automáticamente.

   7. Seleccione **Listo** para terminar de crear el proyecto.

   ![Asignación de nombre y creación del proyecto](media/how-to-devops-starter/create-project.png)

En unos minutos, el panel de DevOps Starter se muestra en Azure Portal. Seleccione el nombre del proyecto para ver el progreso. Es posible que deba actualizar la página. Una aplicación IoT Edge de ejemplo se configura en un repositorio en la organización de Azure DevOps, se ejecuta una compilación y la aplicación se implementa en el dispositivo IoT Edge. Este panel proporciona visibilidad sobre el repositorio de código, la canalización de CI/CD y la aplicación de Azure.

   ![Ver el proyecto en Azure Portal](./media/how-to-devops-starter/portal.png)

## <a name="commit-code-changes-and-execute-cicd"></a>Confirmación de los cambios de código y ejecución de CI/CD

DevOps Starter ha creado un repositorio Git para el proyecto en Azure Repos. En esta sección, puede ver el repositorio y realizar cambios en la aplicación.

1. Para navegar en el repositorio creado para el proyecto, seleccione **Repositorios** en el menú del panel del proyecto. Este vínculo abre una pestaña del explorador y el repositorio de Azure DevOps del nuevo proyecto.

   ![Consulte el repositorio generado en Azure Repos](./media/how-to-devops-starter/view-repositories.png)

> [!NOTE]
> Los pasos siguientes sirven como guía para el uso del explorador web para realizar cambios de código. Si quiere clonar el repositorio localmente en su lugar, seleccione **Clonar** en la parte superior derecha de la ventana. Use la dirección URL proporcionada para clonar el repositorio de Git en Visual Studio Code o la herramienta de desarrollo que prefiera.

2. El repositorio ya incluye código para un módulo denominado **FilterModule** según el idioma de la aplicación que eligió en el proceso de creación. Abra el archivo **modules/FilterModule/module.json**.

   ![Archivo module.json abierto en Azure Repos](./media/how-to-devops-starter/open-module-json.png)

3. Tenga en cuenta que este archivo usa [variables de compilación de Azure DevOps](https://docs.microsoft.com/azure/devops/pipelines/build/variables?view=vsts#build-variables) en el parámetro de **versión**. Esta configuración garantiza la creación de una nueva versión del módulo cada vez que se ejecute una nueva compilación.

## <a name="examine-the-cicd-pipeline"></a>Examen de la canalización de CI/CD

En las secciones anteriores, Azure DevOps Starter configuró automáticamente una canalización de CI/CD completa para la aplicación de IoT Edge. Ahora, explore y personalice la canalización según sea necesario. Para familiarizarse con las canalizaciones de compilación y versión de Azure DevOps, use estos pasos.

1. Para ver las canalizaciones de compilación en el proyecto de DevOps, seleccione **Compilar canalizaciones** en el menú del panel del proyecto. Este vínculo abre una pestaña del explorador y la canalización de compilación de Azure DevOps del nuevo proyecto.

   ![Ver canalizaciones de compilación en Azure Pipelines](./media/how-to-devops-starter/view-build-pipelines.png)

2. Abra la canalización de compilación generada automáticamente y seleccione **Editar** en la parte superior derecha.

    ![Edición de la canalización de compilación](media/how-to-devops-starter/click-edit-button.png)

3. En el panel que se abre, puede examinar las tareas que se producen cuando se ejecuta la canalización de compilación. La canalización de compilación realiza varias tareas que se usan para las implementaciones, como capturar orígenes del repositorio de Git, crear imágenes de módulo IoT Edge, insertar módulos IoT Edge en un registro de contenedores y publicar las salidas usadas. Para más información sobre las tareas de Azure IoT Edge en Azure DevOps, consulte [Configuración de Azure Pipelines para integración continua](how-to-continuous-integration-continuous-deployment-classic.md#create-a-build-pipeline-for-continuous-integration).

4. Seleccione el encabezado **Canalización** en la parte superior de la canalización de compilación para abrir los detalles de la canalización. Cambie el nombre de la canalización de compilación por algo más descriptivo.

   ![Edición de los detalles de canalización](./media/how-to-devops-starter/edit-build-pipeline.png)

5. Seleccione **Guardar y poner en cola** y, después, seleccione **Guardar**. Se puede comentar opcionalmente.

6. Seleccione **Desencadenadores** en el menú de canalización de compilación. DevOps Starter creó automáticamente un desencadenador de integración continua y cada confirmación al repositorio inicia una nueva compilación.  Si lo desea, puede elegir incluir o excluir ramas del proceso de CI.

7. Seleccione **Retención**. Siga el vínculo que le redirigirá a la configuración del proyecto, donde se encuentran las directivas de retención. En función del escenario, puede especificar directivas para conservar o quitar un determinado número de compilaciones.

8. Seleccione **Historial**. Este panel incluye un registro de auditoría de los cambios recientes de la compilación. Azure Pipelines realiza un seguimiento de los cambios que se realizan en la canalización de compilación y permite comparar versiones.

9. Cuando haya terminado la exploración de la canalización de compilación, vaya a la canalización de versión correspondiente. Seleccione **Versiones** en **Canalizaciones** y, a continuación, seleccione **Editar** para ver los detalles de la canalización.

    ![Visualización de la canalización de versión](media/how-to-devops-starter/release-pipeline.png)

10. En **Artefactos**, seleccione **Colocar**. La salida que se usa para el artefacto es la salida de la canalización de compilación que ha examinado en los pasos anteriores.

11. Al lado del icono **Colocar**, seleccione el **Desencadenador de implementación continua** que parece un rayo. Esta canalización de versión tiene el desencadenador habilitado, que ejecuta una implementación cada vez que hay un nuevo artefacto de compilación disponible. Opcionalmente, puede deshabilitar el desencadenador, con lo que las implementaciones van a requerir una ejecución manual.  

12. En el menú de la canalización de versión, seleccione **Tareas** y, a continuación, elija la fase **dev** en la lista desplegable. DevOps Projects ha creado una fase de versión que crea un centro de IoT y un dispositivo IoT Edge en ese centro, que implementa el módulo de ejemplo de la canalización de compilación y que aprovisiona una máquina virtual para ejecutar como dispositivo IoT Edge. Para más información sobre las tareas de Azure IoT Edge para CD, consulte [Configuración de Azure Pipelines para implementación continua](how-to-continuous-integration-continuous-deployment-classic.md#create-a-release-pipeline-for-continuous-deployment).

    ![Visualización de las tareas de implementación continua](media/how-to-devops-starter/choose-release.png)

13. A la derecha, seleccione **Ver versiones**. Esta vista muestra un historial de versiones.

14. Seleccione el nombre de una versión para ver más información sobre ella.

## <a name="clean-up-resources"></a>Limpieza de recursos

Azure App Service y otros recursos relacionados que haya creado se pueden eliminar cuando no se necesiten. Use la funcionalidad **Eliminación** del panel de DevOps Starter.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre las tareas de Azure IoT Edge en Azure DevOps en [Integración continua e implementación continua en Azure IoT Edge](how-to-continuous-integration-continuous-deployment.md)
* Puede encontrar información sobre la implementación de IoT Edge en [Descripción de las implementaciones de IoT Edge en un único dispositivo o a escala](module-deployment-monitoring.md).
* Siga los pasos para crear, actualizar o eliminar una implementación en [Implementación y supervisión de módulos de IoT Edge a escala](how-to-deploy-at-scale.md).
