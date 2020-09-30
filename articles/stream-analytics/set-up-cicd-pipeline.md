---
title: Uso de Azure DevOps para crear una canalización de CI/CD para un trabajo de Stream Analytics
description: En este artículo se describe cómo configurar una canalización de integración continua y entrega continua (CI/CD) para un trabajo de Azure Stream Analytics en Azure DevOps
services: stream-analytics
author: su-jie
ms.author: sujie
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 09/10/2020
ms.openlocfilehash: 23ac1e241c0811944a943c3c3fef3116eff68a67
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931999"
---
# <a name="use-azure-devops-to-create-a-cicd-pipeline-for-a-stream-analytics-job"></a>Uso de Azure DevOps para crear una canalización de CI/CD para un trabajo de Stream Analytics

En este artículo, aprenderá a crear canalizaciones de [compilación](/devops/pipelines/get-started-designer) y [versión](/devops/pipelines/release/define-multistage-release-process) de Azure DevOps mediante herramientas de CI/CD de Azure Stream Analytics.

## <a name="commit-your-stream-analytics-project"></a>Confirmación del proyecto de Stream Analytics

Antes de comenzar, confirme los proyectos de Stream Analytics completados como archivos de origen para un repositorio de [Azure DevOps](/devops/user-guide/source-control). Puede hacer referencia a este [repositorio de ejemplo](https://dev.azure.com/wenyzou/azure-streamanalytics-cicd-demo) y al [código fuente del proyecto de Stream Analytics](https://dev.azure.com/wenyzou/_git/azure-streamanalytics-cicd-demo?path=%2FmyASAProject) en Azure Pipelines.

En los pasos de este artículo se usa un proyecto de Visual Studio Code para Stream Analytics. Si usa un proyecto de Visual Studio, siga los pasos descritos en [Automatización de las compilaciones, las pruebas y las implementaciones de un trabajo de Azure Stream Analytics mediante herramientas de CI/CD](cicd-tools.md).

## <a name="create-a-build-pipeline"></a>Creación de una canalización de compilación

En esta sección, aprenderá a crear una canalización de compilación. Puede hacer referencia a la [canalización de compilación y prueba automática](https://dev.azure.com/wenyzou/_git/azure-streamanalytics-cicd-demo?path=%2FmyASAProject) de este ejemplo en Azure DevOps.

1. Abra un explorador web y vaya al proyecto en Azure DevOps.  

1. En **Pipelines**, en el menú de navegación izquierdo, seleccione **Compilaciones**. Luego, seleccione **Nueva canalización**.

   :::image type="content" source="media/set-up-cicd-pipeline/new-pipeline.png" alt-text="Creación de una instancia de Azure Pipeline":::

1. Seleccione **Usar el editor clásico** para crear una canalización sin YAML.

1. Seleccione el tipo de origen, el proyecto del equipo y el repositorio. Después, seleccione **Continuar**.

   :::image type="content" source="media/set-up-cicd-pipeline/select-repo.png" alt-text="Selección del proyecto de Azure Stream Analytics":::

1. En la página **Elegir una plantilla**, seleccione **Trabajo vacío**.

## <a name="install-npm-package"></a>Instalación del paquete de npm

1. En la página **Tareas**, seleccione el signo más que aparece al lado de **Agent job 1**. Escriba *npm* en la búsqueda de tareas y seleccione **npm**.

   :::image type="content" source="media/set-up-cicd-pipeline/search-npm.png" alt-text="Selección de la tarea npm":::

2. Asigne a la tarea un **nombre para mostrar**. Cambie la opción **Comando** a *personalizar* y escriba el siguiente comando en **Comando y argumentos**. Deje las restantes opciones predeterminadas.

   ```bash
   install -g azure-streamanalytics-cicd
   ```

   :::image type="content" source="media/set-up-cicd-pipeline/npm-config.png" alt-text="Especificación de configuraciones para la tarea npm":::

## <a name="add-a-build-task"></a>Adición de una tarea de compilación

1. En la página **Variables**, en **Variables de canalización**, seleccione **+ Agregar**. Agregue las variables siguientes. Establezca los valores siguientes según sus preferencias:

   |Nombre de la variable|Valor|
   |-|-|
   |projectRootPath|[YourProjectName]|
   |outputPath|Resultados|
   |deployPath|Implementar|

2. En la página **Tareas**, seleccione el signo más que aparece al lado de **Agent job 1**. Busque **Línea de comandos**.

3. Asigne a la tarea un **nombre para mostrar** y escriba el siguiente script. Modifique el script con el nombre del repositorio y el nombre del proyecto.

   ```bash
   azure-streamanalytics-cicd build -project $(projectRootPath)/asaproj.json -outputpath $(projectRootPath)/$(outputPath)/$(deployPath)
   ```

   En la imagen siguiente se usa como ejemplo un proyecto de Visual Studio Code para Stream Analytics.

   :::image type="content" source="media/set-up-cicd-pipeline/command-line-config-build.png" alt-text="Especificación de configuraciones para la tarea de línea de comandos en Visual Studio Code":::

## <a name="add-a-test-task"></a>Adición de una tarea de prueba

1. En la página **Variables**, en **Variables de canalización**, seleccione **+ Agregar**. Agregue las variables siguientes. Cambie los valores por su ruta de acceso de salida y el nombre del repositorio.

   |Nombre de la variable|Valor|
   |-|-|
   |testPath|Prueba|

   :::image type="content" source="media/set-up-cicd-pipeline/pipeline-variables-test.png" alt-text="Adición de variables de canalización":::

2. En la página **Tareas**, seleccione el signo más que aparece al lado de **Agent job 1**. Busque **Línea de comandos**.

3. Asigne a la tarea un **nombre para mostrar** y escriba el siguiente script. Modifique el script con su nombre del archivo de proyecto y la ruta de acceso al archivo de configuración de prueba. 

   Para más información sobre cómo agregar y configurar casos de prueba, consulte las [instrucciones de pruebas automatizadas](cicd-tools.md#automated-test).

   ```bash
   azure-streamanalytics-cicd test -project $(projectRootPath)/asaproj.json -outputpath $(projectRootPath)/$(outputPath)/$(testPath) -testConfigPath $(projectRootPath)/test/testConfig.json 
   ```

   :::image type="content" source="media/set-up-cicd-pipeline/command-line-config-test.png" alt-text="Especificación de configuraciones para la tarea de línea de comandos":::

## <a name="add-a-copy-files-task"></a>Adición de una tarea de copia de archivos

Se debe agregar una tarea de copia de archivos para copiar el archivo de resumen de prueba y los archivos de la plantilla de Azure Resource Manager en la carpeta de artefactos. 

1. En la página **Tareas**, seleccione el signo **+** que aparece al lado de **Trabajo de agente 1**. Busque **Copiar archivos**. Luego, escriba las siguientes configuraciones. Al asignar `**` a **Contenido**, se copian todos los archivos de los resultados de prueba.

   |Parámetro|Entrada|
   |-|-|
   |Nombre para mostrar|Copiar archivos a: $(build.artifactstagingdirectory)|
   |Carpeta de origen|`$(system.defaultworkingdirectory)/$(outputPath)/`|
   |Contenido| `**` |
   |Carpeta de destino| `$(build.artifactstagingdirectory)`|

2. Expanda **Opciones de control**. En **Ejecutar esta tarea**, seleccione **Incluso si una tarea anterior ha dado error, a menos que se haya cancelado la compilación**.

   :::image type="content" source="media/set-up-cicd-pipeline/copy-config.png" alt-text="Especificación de configuraciones para la tarea de copia":::

## <a name="add-a-publish-build-artifacts-task"></a>Adición de una tarea de publicación de artefactos de compilación

1. En la página **Tareas**, seleccione el signo más que aparece al lado de **Agent job 1**. Busque **Publicar artefactos de compilación** y seleccione la opción con el icono de flecha negra.

2. Expanda **Opciones de control**. En **Ejecutar esta tarea**, seleccione **Incluso si una tarea anterior ha dado error, a menos que se haya cancelado la compilación**.

   :::image type="content" source="media/set-up-cicd-pipeline/publish-config.png" alt-text="Especificación de configuraciones para la tarea de publicación":::

## <a name="save-and-run"></a>Guardar y ejecutar

Cuando haya terminado de agregar el paquete npm, la línea de comandos y las tareas de copia de archivos y publicación de artefactos de compilación, seleccione **Guardar y poner en cola**. Cuando se le pida, escriba un comentario de guardado y seleccione **Guardar y ejecutar**. Puede descargar los resultados de las pruebas de la página **Resumen** de la canalización.

## <a name="check-the-build-and-test-results"></a>Comprobación de los resultados de compilación y prueba

El archivo de resumen de pruebas y los archivos de la plantilla de Azure Resource Manager se pueden encontrar en la carpeta **Published**.

   :::image type="content" source="media/set-up-cicd-pipeline/check-build-test-result.png" alt-text="Comprobación de los resultados de compilación y prueba":::

   :::image type="content" source="media/set-up-cicd-pipeline/check-drop-folder.png" alt-text="Comprobación de los artefactos":::

## <a name="release-with-azure-pipelines"></a>Publicación con Azure Pipelines

En esta sección, aprenderá a crear una canalización de versión. Puede hacer referencia a la [canalización de versión](https://dev.azure.com/wenyzou/azure-streamanalytics-cicd-demo/_release?_a=releases&view=mine&definitionId=2&preserve-view=true) de este ejemplo en Azure DevOps.

Abra un explorador web y vaya al proyecto de Visual Studio Code de Azure Stream Analytics.

1. En **Pipelines**, en el menú de navegación izquierdo, seleccione **Versiones**. Luego seleccione **Nueva canalización**.

2. Seleccione **Empezar con un trabajo vacío**.

3. En el cuadro de **Artefactos**, seleccione **+ Agregar un artefacto**. En **Origen**, seleccione la canalización de compilación que ha creado y elija **Agregar**.

   :::image type="content" source="media/set-up-cicd-pipeline/build-artifact.png" alt-text="Especificar artefacto de canalización de compilación":::

4. Cambie el nombre de **Stage 1** a **Deploy job to test environment** (Implementar trabajo en entorno de prueba).

5. Agregue una nueva fase y asígnele el nombre **Implementar trabajo en entorno de producción**.

### <a name="add-deploy-tasks"></a>Adición de tareas de implementación

1. En la lista desplegable de tareas, seleccione **Deploy job to test environment** (Implementar trabajo en entorno de prueba).

2. Seleccione el signo **+** junto a **Trabajo de agente** y busque **implementación de plantilla de ARM**. Escriba los siguientes parámetros:

   |Parámetro|Value|
   |-|-|
   |Nombre para mostrar| *Implementación de myASAProject*|
   |Suscripción de Azure| Elija su suscripción.|
   |Acción| *Crear o actualizar grupo de recursos*|
   |Resource group| Elija un nombre para el grupo de recursos de prueba que contendrá su trabajo de Stream Analytics.|
   |Location|Elija la ubicación de su grupo de recursos de prueba.|
   |Ubicación de la plantilla| Artefacto vinculado|
   |Plantilla| $(System.DefaultWorkingDirectory)/_azure-streamanalytics-cicd-demo-CI-Deploy/drop/myASAProject.JobTemplate.json |
   |Parámetros de plantilla|$(System.DefaultWorkingDirectory)/_azure-streamanalytics-cicd-demo-CI-Deploy/drop/myASAProject.JobTemplate.parameters.json |
   |Reemplazo de los parámetros de plantilla|-<arm_template_parameter> "su valor". Puede definir los parámetros mediante **variables**.|
   |Modo de implementación|Incremental|

3. En la lista desplegable de tareas, seleccione **Implementar trabajo en entorno de producción**.

4. Seleccione el signo **+** junto a **Trabajo de agente** y busque *implementación de plantilla de ARM*. Escriba los siguientes parámetros:

   |Parámetro|Value|
   |-|-|
   |Nombre para mostrar| *Implementación de myASAProject*|
   |Suscripción de Azure| Elija su suscripción.|
   |Acción| *Crear o actualizar grupo de recursos*|
   |Resource group| Elija un nombre para el grupo de recursos de producción que contendrá su trabajo de Stream Analytics.|
   |Location|Elija la ubicación del grupo de recursos de producción.|
   |Ubicación de la plantilla| *Artefacto vinculado*|
   |Plantilla| $(System.DefaultWorkingDirectory)/_azure-streamanalytics-cicd-demo-CI-Deploy/drop/myASAProject.JobTemplate.json |
   |Parámetros de plantilla|$(System.DefaultWorkingDirectory)/_azure-streamanalytics-cicd-demo-CI-Deploy/drop/myASAProject.JobTemplate.parameters.json |
   |Reemplazo de los parámetros de plantilla|-<arm_template_parameter> "su valor"|
   |Modo de implementación|Incremental|

### <a name="create-a-release"></a>Creación de una versión

Para crear una versión, seleccione **Crear versión** en la esquina superior derecha.

:::image type="content" source="media/set-up-cicd-pipeline/create-release.png" alt-text="Crear una versión mediante Azure Pipelines":::

## <a name="next-steps"></a>Pasos siguientes

* [Integración continua e implementación continua en Azure Stream Analytics](cicd-overview.md)
* [Automatización de las compilaciones, las pruebas y las implementaciones de un trabajo de Azure Stream Analytics mediante herramientas de CI/CD](cicd-tools.md)